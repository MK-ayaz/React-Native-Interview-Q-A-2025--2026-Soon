[⬅️ Back to Home](../README.md)

# 🎯 Phase 07: Styling & UI Components System

---

```

**Complete Push Notifications Implementation:**
```javascript
// NotificationManager class
class NotificationManager {
  private static instance: NotificationManager;
  private notificationToken: string | null = null;
  private isInitialized = false;

  static getInstance(): NotificationManager {
    if (!NotificationManager.instance) {
      NotificationManager.instance = new NotificationManager();
    }
    return NotificationManager.instance;
  }

  async initialize(): Promise<void> {
    if (this.isInitialized) return;

    try {
      // Request permissions
      const { status } = await Notifications.requestPermissionsAsync();
      if (status !== 'granted') {
        throw new Error('Notification permission denied');
      }

      // Get device token
      await this.registerForPushNotifications();

      // Set notification handler
      Notifications.setNotificationHandler(this.notificationHandler);

      // Handle incoming notifications
      this.setupNotificationListeners();

      this.isInitialized = true;
    } catch (error) {
      console.error('Notification initialization failed:', error);
      throw error;
    }
  }

  private async registerForPushNotifications(): Promise<void> {
    try {
      let token;

      if (Platform.OS === 'android') {
        // FCM token for Android
        token = await Notifications.getDevicePushTokenAsync();
      } else {
        // APNs token for iOS
        token = await Notifications.getDevicePushTokenAsync();
      }

      this.notificationToken = token.data;
      console.log('Notification token:', this.notificationToken);

      // Send token to server
      await this.sendTokenToServer(this.notificationToken);
    } catch (error) {
      console.error('Failed to get push token:', error);
    }
  }

  private notificationHandler = {
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  };

  private setupNotificationListeners(): void {
    // Handle notification received while app is foregrounded
    const foregroundSubscription = Notifications.addNotificationReceivedListener(
      this.handleNotificationReceived
    );

    // Handle notification opened
    const openedSubscription = Notifications.addNotificationResponseReceivedListener(
      this.handleNotificationOpened
    );

    // Store subscriptions for cleanup
    this.subscriptions = { foregroundSubscription, openedSubscription };
  }

  private handleNotificationReceived = (notification: Notification) => {
    console.log('Notification received:', notification);

    const { data, title, body } = notification.request.content;

    // Handle silent notifications
    if (data?.silent) {
      this.handleSilentNotification(data);
      return;
    }

    // Handle data-only notifications
    if (data?.type) {
      this.handleDataNotification(data);
    }

    // Track analytics
    analytics.track('notification_received', {
      title,
      body,
      data: JSON.stringify(data),
    });
  };

  private handleNotificationOpened = (response: NotificationResponse) => {
    console.log('Notification opened:', response);

    const { notification, actionIdentifier } = response;
    const { data } = notification.request.content;

    // Track user interaction
    analytics.track('notification_opened', {
      action: actionIdentifier,
      data: JSON.stringify(data),
    });

    // Handle deep linking
    if (data?.deepLink) {
      this.handleDeepLink(data.deepLink);
    }

    // Handle action buttons
    if (actionIdentifier !== Notifications.DEFAULT_ACTION_IDENTIFIER) {
      this.handleNotificationAction(actionIdentifier, data);
    }
  };

  private handleSilentNotification(data: any): void {
    // Process silent notification (background data sync, etc.)
    if (data.type === 'sync') {
      // Trigger data synchronization
      syncManager.syncData();
    } else if (data.type === 'update') {
      // Update app data
      dataManager.updateFromServer(data.payload);
    }
  }

  private handleDataNotification(data: any): void {
    // Handle data-only notifications
    switch (data.type) {
      case 'new_message':
        messageManager.addMessage(data.message);
        break;
      case 'friend_request':
        friendManager.addFriendRequest(data.request);
        break;
      case 'update_badge':
        this.updateBadgeCount(data.count);
        break;
    }
  }

  private handleDeepLink(deepLink: string): void {
    // Navigate based on deep link
    const navigation = require('../navigation/NavigationService').default;

    if (deepLink.startsWith('/post/')) {
      const postId = deepLink.split('/post/')[1];
      navigation.navigate('PostDetail', { postId });
    } else if (deepLink.startsWith('/user/')) {
      const userId = deepLink.split('/user/')[1];
      navigation.navigate('UserProfile', { userId });
    }
  }

  private handleNotificationAction(actionIdentifier: string, data: any): void {
    switch (actionIdentifier) {
      case 'accept':
        if (data.type === 'friend_request') {
          friendManager.acceptRequest(data.requestId);
        }
        break;
      case 'decline':
        if (data.type === 'friend_request') {
          friendManager.declineRequest(data.requestId);
        }
        break;
      case 'reply':
        // Open reply interface
        this.showReplyInterface(data);
        break;
    }
  }

  private async sendTokenToServer(token: string): Promise<void> {
    try {
      await apiClient.post('/notifications/register-token', {
        token,
        platform: Platform.OS,
        appVersion: Constants.expoConfig?.version,
      });
    } catch (error) {
      console.error('Failed to send token to server:', error);
    }
  }

  async updateBadgeCount(count: number): Promise<void> {
    try {
      await Notifications.setBadgeCountAsync(count);
    } catch (error) {
      console.error('Failed to update badge count:', error);
    }
  }

  async scheduleNotification(
    title: string,
    body: string,
    data: any,
    trigger: Date
  ): Promise<string> {
    try {
      const notificationId = await Notifications.scheduleNotificationAsync({
        content: {
          title,
          body,
          data,
          sound: 'default',
        },
        trigger,
      });

      return notificationId;
    } catch (error) {
      console.error('Failed to schedule notification:', error);
      throw error;
    }
  }

  async cancelNotification(notificationId: string): Promise<void> {
    try {
      await Notifications.cancelScheduledNotificationAsync(notificationId);
    } catch (error) {
      console.error('Failed to cancel notification:', error);
    }
  }

  async cancelAllNotifications(): Promise<void> {
    try {
      await Notifications.cancelAllScheduledNotificationsAsync();
    } catch (error) {
      console.error('Failed to cancel all notifications:', error);
    }
  }

  getToken(): string | null {
    return this.notificationToken;
  }

  destroy(): void {
    if (this.subscriptions) {
      this.subscriptions.foregroundSubscription?.remove();
      this.subscriptions.openedSubscription?.remove();
    }
    this.isInitialized = false;
  }
}

// React hook for notifications
export const useNotifications = () => {
  const [notification, setNotification] = useState<Notification | null>(null);
  const [notificationResponse, setNotificationResponse] = useState<NotificationResponse | null>(null);

  useEffect(() => {
    const manager = NotificationManager.getInstance();

    // Initialize notifications
    manager.initialize().catch(console.error);

    // Set up listeners for React state
    const receivedSubscription = Notifications.addNotificationReceivedListener(
      setNotification
    );

    const responseSubscription = Notifications.addNotificationResponseReceivedListener(
      setNotificationResponse
    );

    return () => {
      receivedSubscription.remove();
      responseSubscription.remove();
    };
  }, []);

  return {
    notification,
    notificationResponse,
    scheduleNotification: NotificationManager.getInstance().scheduleNotification.bind(NotificationManager.getInstance()),
    cancelNotification: NotificationManager.getInstance().cancelNotification.bind(NotificationManager.getInstance()),
    cancelAllNotifications: NotificationManager.getInstance().cancelAllNotifications.bind(NotificationManager.getInstance()),
  };
};

// Usage example
const NotificationComponent = () => {
  const { notification, notificationResponse } = useNotifications();

  useEffect(() => {
    if (notification) {
      console.log('Received notification:', notification);
    }
  }, [notification]);

  useEffect(() => {
    if (notificationResponse) {
      console.log('Notification response:', notificationResponse);
    }
  }, [notificationResponse]);

  return (
    <View>
      <Text>Notifications enabled</Text>
    </View>
  );
};
```

---

### 5.4 **Background Tasks and App State Management**

**Question:** *"Implement background task processing, app state monitoring, and resource management for battery and data optimization."*

**Perfect Answer Structure:**
```
BACKGROUND PROCESSING ARCHITECTURE:

🎯 APP STATE MANAGEMENT:
- Foreground/background/killed states
- State persistence strategies
- Memory cleanup on backgrounding
- Fast app switching support

🎯 BACKGROUND TASKS:
- Background fetch implementation
- Task scheduling and queuing
- Resource-aware execution
- Error handling and retry logic

🎯 RESOURCE OPTIMIZATION:
- Battery-aware processing
- Network condition monitoring
- Memory pressure handling
- Platform-specific optimizations
```

**Complete Background Tasks Implementation:**
```javascript
// AppStateManager class
class AppStateManager {
  private static instance: AppStateManager;
  private currentState: AppStateStatus = 'active';
  private stateListeners: Set<(state: AppStateStatus) => void> = new Set();

  static getInstance(): AppStateManager {
    if (!AppStateManager.instance) {
      AppStateManager.instance = new AppStateManager();
    }
    return AppStateManager.instance;
  }

  initialize(): void {
    AppState.addEventListener('change', this.handleAppStateChange);
    this.currentState = AppState.currentState;
  }

  private handleAppStateChange = (nextAppState: AppStateStatus): void => {
    const previousState = this.currentState;
    this.currentState = nextAppState;

    console.log(`App state changed: ${previousState} → ${nextAppState}`);

    // Notify listeners
    this.stateListeners.forEach(listener => {
      try {
        listener(nextAppState);
      } catch (error) {
        console.error('App state listener error:', error);
      }
    });

    // Handle state-specific logic
    switch (nextAppState) {
      case 'active':
        this.handleAppActive(previousState);
        break;
      case 'background':
        this.handleAppBackground(previousState);
        break;
      case 'inactive':
        this.handleAppInactive(previousState);
        break;
    }
  };

  private handleAppActive(previousState: AppStateStatus): void {
    // App became active
    analytics.track('app_foreground');

    // Resume paused operations
    BackgroundTaskManager.resumeTasks();

    // Refresh data if needed
    if (previousState === 'background') {
      DataManager.refreshCriticalData();
    }

    // Clear any temporary background optimizations
    MemoryManager.clearBackgroundOptimizations();
  }

  private handleAppBackground(previousState: AppStateStatus): void {
    // App went to background
    analytics.track('app_background');

    // Pause non-critical operations
    BackgroundTaskManager.pauseTasks();

    // Save app state
    StateManager.persistState();

    // Optimize memory usage
    MemoryManager.optimizeForBackground();

    // Schedule background tasks
    this.scheduleBackgroundTasks();
  }

  private handleAppInactive(previousState: AppStateStatus): void {
    // App became inactive (multitasking view)
    analytics.track('app_inactive');
  }

  private scheduleBackgroundTasks(): void {
    // Schedule background fetch if supported
    if (Platform.OS === 'ios') {
      BackgroundFetch.registerTaskAsync('background-sync', {
        minimumInterval: 15 * 60, // 15 minutes
        stopOnTerminate: false,
        startOnBoot: true,
      });
    }
  }

  addStateListener(listener: (state: AppStateStatus) => void): () => void {
    this.stateListeners.add(listener);
    return () => this.stateListeners.delete(listener);
  }

  getCurrentState(): AppStateStatus {
    return this.currentState;
  }

  isActive(): boolean {
    return this.currentState === 'active';
  }

  isBackground(): boolean {
    return this.currentState === 'background';
  }

  destroy(): void {
    AppState.removeEventListener('change', this.handleAppStateChange);
    this.stateListeners.clear();
  }
}

// BackgroundTaskManager class
class BackgroundTaskManager {
  private static instance: BackgroundTaskManager;
  private tasks: Map<string, BackgroundTask> = new Map();
  private isPaused = false;

  static getInstance(): BackgroundTaskManager {
    if (!BackgroundTaskManager.instance) {
      BackgroundTaskManager.instance = new BackgroundTaskManager();
    }
    return BackgroundTaskManager.instance;
  }

  registerTask(
    id: string,
    task: () => Promise<void>,
    options: {
      interval?: number;
      retryOnFailure?: boolean;
      maxRetries?: number;
      priority?: 'low' | 'normal' | 'high';
    } = {}
  ): void {
    const backgroundTask: BackgroundTask = {
      id,
      task,
      options: {
        interval: 5 * 60 * 1000, // 5 minutes default
        retryOnFailure: true,
        maxRetries: 3,
        priority: 'normal',
        ...options,
      },
      isRunning: false,
      nextRun: Date.now() + options.interval || 0,
      retryCount: 0,
    };

    this.tasks.set(id, backgroundTask);
  }

  unregisterTask(id: string): void {
    const task = this.tasks.get(id);
    if (task) {
      this.tasks.delete(id);
    }
  }

  async executeTask(id: string): Promise<void> {
    const task = this.tasks.get(id);
    if (!task || task.isRunning) return;

    task.isRunning = true;

    try {
      await task.task();
      task.retryCount = 0; // Reset retry count on success
      task.nextRun = Date.now() + task.options.interval!;
    } catch (error) {
      console.error(`Background task ${id} failed:`, error);

      if (task.options.retryOnFailure && task.retryCount < task.options.maxRetries!) {
        task.retryCount++;
        // Exponential backoff
        const delay = Math.pow(2, task.retryCount) * 1000;
        task.nextRun = Date.now() + delay;
      } else {
        // Max retries reached or no retry
        task.nextRun = Date.now() + task.options.interval!;
      }
    } finally {
      task.isRunning = false;
    }
  }

  pauseTasks(): void {
    this.isPaused = true;
  }

  resumeTasks(): void {
    this.isPaused = false;
  }

  startScheduler(): void {
    const scheduler = () => {
      if (!this.isPaused) {
        const now = Date.now();

        for (const [id, task] of this.tasks) {
          if (now >= task.nextRun && !task.isRunning) {
            this.executeTask(id);
          }
        }
      }
    };

    // Run scheduler every second
    setInterval(scheduler, 1000);
  }

  getTaskStatus(id: string): BackgroundTask | undefined {
    return this.tasks.get(id);
  }

  getAllTasks(): BackgroundTask[] {
    return Array.from(this.tasks.values());
  }
}

// MemoryManager for resource optimization
class MemoryManager {
  private static instance: MemoryManager;
  private memoryWarnings = 0;

  static getInstance(): MemoryManager {
    if (!MemoryManager.instance) {
      MemoryManager.instance = new MemoryManager();
    }
    return MemoryManager.instance;
  }

  initialize(): void {
    // Listen for memory warnings
    if (Platform.OS === 'ios') {
      // iOS memory pressure
    } else {
      // Android memory info
    }
  }

  optimizeForBackground(): void {
    // Clear image caches
    ImageCache.clear();

    // Cancel pending network requests
    NetworkManager.cancelPendingRequests();

    // Clear non-essential data
    CacheManager.clearNonEssential();

    // Force garbage collection hint
    if (global.gc) {
      global.gc();
    }
  }

  clearBackgroundOptimizations(): void {
    // Restore caches and resume operations
    ImageCache.restore();
    NetworkManager.resumeRequests();
  }

  getMemoryUsage(): Promise<any> {
    // Platform-specific memory monitoring
    if (Platform.OS === 'ios') {
      // iOS memory reporting
    } else {
      // Android memory reporting
    }
  }

  handleMemoryWarning(): void {
    this.memoryWarnings++;

    analytics.track('memory_warning', {
      count: this.memoryWarnings,
      timestamp: Date.now(),
    });

    // Aggressive cleanup
    this.optimizeForBackground();

    // Notify user if critical
    if (this.memoryWarnings > 5) {
      Alert.alert(
        'Memory Warning',
        'The app is running low on memory. Please close other apps.',
        [{ text: 'OK' }]
      );
    }
  }
}

// React hooks for app state and background tasks
export const useAppState = () => {
  const [appState, setAppState] = useState(AppState.currentState);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', setAppState);
    return () => subscription.remove();
  }, []);

  return appState;
};

export const useBackgroundTask = (
  taskId: string,
  task: () => Promise<void>,
  options?: any
) => {
  useEffect(() => {
    const manager = BackgroundTaskManager.getInstance();
    manager.registerTask(taskId, task, options);

    return () => {
      manager.unregisterTask(taskId);
    };
  }, [taskId, task, options]);
};

// Usage examples
const App = () => {
  useEffect(() => {
    const stateManager = AppStateManager.getInstance();
    const taskManager = BackgroundTaskManager.getInstance();
    const memoryManager = MemoryManager.getInstance();

    stateManager.initialize();
    taskManager.startScheduler();
    memoryManager.initialize();

    // Register background tasks
    taskManager.registerTask('sync-data', async () => {
      await syncManager.performBackgroundSync();
    }, { interval: 15 * 60 * 1000 }); // 15 minutes

    taskManager.registerTask('cleanup-cache', async () => {
      await cacheManager.performCleanup();
    }, { interval: 60 * 60 * 1000 }); // 1 hour

    return () => {
      stateManager.destroy();
    };
  }, []);

  return <AppNavigator />;
};

const DataSyncComponent = () => {
  const appState = useAppState();

  useBackgroundTask('component-sync', async () => {
    if (appState === 'background') {
      await performDataSync();
    }
  }, { priority: 'low' });

  return <Text>Data sync active</Text>;
};
```

---

*Phase 5 continues with advanced native module development, hardware sensor integration, biometric authentication, and platform-specific optimizations. Each question includes production-ready implementations with error handling and cross-platform compatibility.*

---

## 🎯 Phase 6 — Testing, Debugging & Deployment

**Production readiness questions filter out juniors** - senior engineers must demonstrate comprehensive testing strategies, debugging expertise, and deployment proficiency.

### 6.1 **Comprehensive Testing Strategy Implementation**

**Question:** *"Design and implement a complete testing strategy for a React Native application covering unit, integration, and E2E tests."*

**Perfect Answer Structure:**
```
TESTING PYRAMID ARCHITECTURE:

🎯 UNIT TESTS (Bottom layer - 70%):
- Component logic testing
- Hook testing
- Utility function testing
- Business logic isolation

🎯 INTEGRATION TESTS (Middle layer - 20%):
- Component integration
- API integration
- Navigation flow testing
- State management testing

🎯 E2E TESTS (Top layer - 10%):
- Critical user journeys
- Cross-platform validation
- Performance regression testing
- Release validation
```

**Complete Testing Implementation:**
```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jsdom',
  testMatch: [
    '**/__tests__/**/*.test.js',
    '**/?(*.)+(spec|test).js',
  ],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
  },
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?|react-navigation|@react-navigation/.*|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-native-svg))',
  ],
};

// jest.setup.js
import 'react-native-gesture-handler/jestSetup';
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';
import mockSafeAreaContext from 'react-native-safe-area-context/jest/mock';

// Mock React Native modules
jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);
jest.mock('react-native-safe-area-context', () => mockSafeAreaContext);

// Mock navigation
jest.mock('@react-navigation/native', () => ({
  useNavigation: () => ({
    navigate: jest.fn(),
    goBack: jest.fn(),
  }),
  useRoute: () => ({
    params: {},
  }),
}));

// Mock API calls
jest.mock('axios');
global.fetch = jest.fn();

// Silence console warnings during tests
global.console = {
  ...console,
  warn: jest.fn(),
  error: jest.fn(),
};

// Custom test utilities
global.testRender = (component, options = {}) => {
  const { wrapper: Wrapper, ...renderOptions } = options;
  const wrappedComponent = Wrapper
    ? <Wrapper>{component}</Wrapper>
    : component;

  return render(wrappedComponent, renderOptions);
};

// Test wrapper for providers
const AllTheProviders = ({ children }) => (
  <ThemeProvider>
    <AuthProvider>
      <NavigationContainer>
        {children}
      </NavigationContainer>
    </AuthProvider>
  </ThemeProvider>
);

const customRender = (ui, options) =>
  render(ui, { wrapper: AllTheProviders, ...options });

global.customRender = customRender;

// Component testing example
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { UserCard } from '../UserCard';

const mockUser = {
  id: '1',
  name: 'John Doe',
  email: 'john@example.com',
  avatar: 'https://example.com/avatar.jpg',
};

describe('UserCard', () => {
  const mockOnPress = jest.fn();
  const mockOnDelete = jest.fn();

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders user information correctly', () => {
    const { getByText, getByTestId } = render(
      <UserCard user={mockUser} onPress={mockOnPress} />
    );

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 8 ➡️](phase8-state-management.md)