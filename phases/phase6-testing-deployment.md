[⬅️ Back to Home](../README.md)

# 🎯 Phase 06: Testing, Debugging & Deployment

---

// Safe state updates with stale closure protection
  const handleUpdate = useCallback(() => {
    setData(prev => {
      if (!isMounted) return prev; // Prevent state updates after unmount
      return { ...prev, updated: true };
    });
  }, [isMounted]);

  return (
    <View>
      <Text>{data?.message}</Text>
      <Button title="Update" onPress={handleUpdate} />
    </View>
  );
};

// Context cleanup pattern
const DataProvider = ({ children }) => {
  const [data, setData] = useState({});
  const subscriptionsRef = useRef(new Set());

  const subscribe = useCallback((callback) => {
    subscriptionsRef.current.add(callback);

    return () => {
      subscriptionsRef.current.delete(callback);
    };
  }, []);

  const notifySubscribers = useCallback((newData) => {
    subscriptionsRef.current.forEach(callback => {
      try {
        callback(newData);
      } catch (error) {
        console.error('Subscriber error:', error);
      }
    });
  }, []);

  useEffect(() => {
    return () => {
      // Cleanup all subscriptions
      subscriptionsRef.current.clear();
    };
  }, []);

  const contextValue = useMemo(() => ({
    data,
    subscribe,
    updateData: (newData) => {
      setData(newData);
      notifySubscribers(newData);
    },
  }), [data, subscribe, notifySubscribers]);

  return (
    <DataContext.Provider value={contextValue}>
      {children}
    </DataContext.Provider>
  );
};

// Native module cleanup
const useNativeModule = (moduleName) => {
  const nativeModule = useMemo(() => {
    if (TurboModuleRegistry) {
      return TurboModuleRegistry.get(moduleName);
    }
    return null;
  }, [moduleName]);

  useEffect(() => {
    let eventListener = null;

    if (nativeModule && nativeModule.addListener) {
      eventListener = nativeModule.addListener('onEvent', handleEvent);
    }

    return () => {
      if (eventListener) {
        eventListener.remove();
      }
    };
  }, [nativeModule]);

  return nativeModule;
};
```

**Memory Monitoring and Profiling:**
```javascript
// Memory usage monitoring
const useMemoryMonitor = () => {
  const [memoryUsage, setMemoryUsage] = useState({
    jsHeapSizeLimit: 0,
    totalJSHeapSize: 0,
    usedJSHeapSize: 0,
  });

  useEffect(() => {
    if (__DEV__ && global.performance?.memory) {
      const updateMemoryUsage = () => {
        setMemoryUsage(global.performance.memory);
      };

      updateMemoryUsage();
      const interval = setInterval(updateMemoryUsage, 5000);

      return () => clearInterval(interval);
    }
  }, []);

  return memoryUsage;
};

// Performance and memory reporting
const usePerformanceReporter = (componentName: string) => {
  const memoryUsage = useMemoryMonitor();
  const [renderMetrics, setRenderMetrics] = useState({
    renderCount: 0,
    averageRenderTime: 0,
    totalRenderTime: 0,
  });

  const reportMetrics = useCallback(() => {
    if (!__DEV__) {
      analytics.track('component_performance', {
        component: componentName,
        ...renderMetrics,
        memoryUsage,
      });
    }
  }, [componentName, renderMetrics, memoryUsage]);

  useEffect(() => {
    const interval = setInterval(reportMetrics, 60000); // Every minute
    return () => clearInterval(interval);
  }, [reportMetrics]);

  return { renderMetrics, memoryUsage };
};
```

---

*Phase 4 continues with advanced performance topics including bundle optimization, startup performance, network optimization, and performance monitoring strategies. Each question includes real-world implementation examples and performance benchmarking techniques.*

---

## 🎯 Phase 5 — Native Modules & Device APIs

**Device API questions demonstrate production experience** - expect deep dives into permissions, storage, notifications, and hardware integration.

### 5.1 **Comprehensive Permissions Management System**

**Question:** *"Design a robust permissions management system handling camera, location, contacts, and storage permissions across iOS and Android."*

**Perfect Answer Structure:**
```
PERMISSIONS ARCHITECTURE:

🎯 PERMISSION LIFECYCLE:
- Request → Check status → Handle denial → Retry logic
- Platform-specific permission mapping
- Graceful degradation for denied permissions
- Persistent permission state management

🎯 ERROR HANDLING:
- Permission denied scenarios
- "Don't ask again" state handling
- Alternative flows for missing permissions
- User education and guidance

🎯 COMPLIANCE & UX:
- GDPR compliance for data permissions
- Clear permission rationale
- Progressive permission requests
- Permission grouping strategies
```

**Complete Permissions Management Implementation:**
```javascript
// types/permissions.ts
export type PermissionType =
  | 'camera'
  | 'microphone'
  | 'location'
  | 'contacts'
  | 'storage'
  | 'notification'
  | 'calendar'
  | 'photos';

export type PermissionStatus =
  | 'granted'
  | 'denied'
  | 'blocked'
  | 'unavailable'
  | 'limited';

export interface PermissionResult {
  status: PermissionStatus;
  canAskAgain: boolean;
  isRequestable: boolean;
}

// PermissionManager class
class PermissionManager {
  private static instance: PermissionManager;
  private permissionCache = new Map<PermissionType, PermissionResult>();

  static getInstance(): PermissionManager {
    if (!PermissionManager.instance) {
      PermissionManager.instance = new PermissionManager();
    }
    return PermissionManager.instance;
  }

  async requestPermission(
    type: PermissionType,
    options?: { rationale?: string; fallback?: () => void }
  ): Promise<PermissionResult> {
    try {
      // Check current status first
      const currentStatus = await this.checkPermission(type);

      if (currentStatus.status === 'granted') {
        return currentStatus;
      }

      if (!currentStatus.isRequestable) {
        options?.fallback?.();
        return currentStatus;
      }

      // Show rationale if provided
      if (options?.rationale && currentStatus.canAskAgain) {
        await this.showPermissionRationale(type, options.rationale);
      }

      // Request permission
      const result = await this.performPermissionRequest(type);

      // Cache result
      this.permissionCache.set(type, result);

      return result;
    } catch (error) {
      console.error(`Permission request failed for ${type}:`, error);
      return {
        status: 'unavailable',
        canAskAgain: false,
        isRequestable: false,
      };
    }
  }

  async checkPermission(type: PermissionType): Promise<PermissionResult> {
    // Check cache first
    if (this.permissionCache.has(type)) {
      return this.permissionCache.get(type)!;
    }

    try {
      const result = await this.getPlatformPermissionStatus(type);
      this.permissionCache.set(type, result);
      return result;
    } catch (error) {
      console.error(`Permission check failed for ${type}:`, error);
      return {
        status: 'unavailable',
        canAskAgain: false,
        isRequestable: false,
      };
    }
  }

  private async performPermissionRequest(type: PermissionType): Promise<PermissionResult> {
    switch (type) {
      case 'camera':
        return this.requestCameraPermission();
      case 'location':
        return this.requestLocationPermission();
      case 'contacts':
        return this.requestContactsPermission();
      case 'storage':
        return this.requestStoragePermission();
      case 'notification':
        return this.requestNotificationPermission();
      default:
        throw new Error(`Unsupported permission type: ${type}`);
    }
  }

  private async getPlatformPermissionStatus(type: PermissionType): Promise<PermissionResult> {
    switch (type) {
      case 'camera':
        return this.getCameraPermissionStatus();
      case 'location':
        return this.getLocationPermissionStatus();
      case 'contacts':
        return this.getContactsPermissionStatus();
      case 'storage':
        return this.getStoragePermissionStatus();
      case 'notification':
        return this.getNotificationPermissionStatus();
      default:
        return {
          status: 'unavailable',
          canAskAgain: false,
          isRequestable: false,
        };
    }
  }

  // Platform-specific implementations
  private async requestCameraPermission(): Promise<PermissionResult> {
    const { status } = await Camera.requestCameraPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async getCameraPermissionStatus(): Promise<PermissionResult> {
    const { status } = await Camera.getCameraPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async requestLocationPermission(): Promise<PermissionResult> {
    const { status } = await Location.requestForegroundPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async getLocationPermissionStatus(): Promise<PermissionResult> {
    const { status } = await Location.getForegroundPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async requestContactsPermission(): Promise<PermissionResult> {
    const { status } = await Contacts.requestPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async getContactsPermissionStatus(): Promise<PermissionResult> {
    const { status } = await Contacts.getPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async requestStoragePermission(): Promise<PermissionResult> {
    if (Platform.OS === 'ios') {
      // iOS doesn't have storage permission like Android
      return {
        status: 'granted',
        canAskAgain: true,
        isRequestable: true,
      };
    }

    const { status } = await MediaLibrary.requestPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async getStoragePermissionStatus(): Promise<PermissionResult> {
    if (Platform.OS === 'ios') {
      return {
        status: 'granted',
        canAskAgain: true,
        isRequestable: true,
      };
    }

    const { status } = await MediaLibrary.getPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async requestNotificationPermission(): Promise<PermissionResult> {
    const { status } = await Notifications.requestPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async getNotificationPermissionStatus(): Promise<PermissionResult> {
    const { status } = await Notifications.getPermissionsAsync();
    return {
      status: status as PermissionStatus,
      canAskAgain: status === 'denied',
      isRequestable: true,
    };
  }

  private async showPermissionRationale(type: PermissionType, rationale: string): Promise<void> {
    return new Promise((resolve) => {
      Alert.alert(
        'Permission Required',
        rationale,
        [
          { text: 'Cancel', style: 'cancel', onPress: resolve },
          { text: 'Continue', onPress: resolve },
        ]
      );
    });
  }
}

// React hook for permission management
export const usePermission = (type: PermissionType) => {
  const [permission, setPermission] = useState<PermissionResult | null>(null);
  const [loading, setLoading] = useState(false);

  const manager = PermissionManager.getInstance();

  const checkPermission = useCallback(async () => {
    setLoading(true);
    try {
      const result = await manager.checkPermission(type);
      setPermission(result);
      return result;
    } finally {
      setLoading(false);
    }
  }, [type, manager]);

  const requestPermission = useCallback(async (options?: {
    rationale?: string;
    fallback?: () => void;
  }) => {
    setLoading(true);
    try {
      const result = await manager.requestPermission(type, options);
      setPermission(result);
      return result;
    } finally {
      setLoading(false);
    }
  }, [type, manager]);

  useEffect(() => {
    checkPermission();
  }, [checkPermission]);

  return {
    permission,
    loading,
    checkPermission,
    requestPermission,
  };
};

// Usage example
const CameraComponent = () => {
  const { permission, loading, requestPermission } = usePermission('camera');

  const handleTakePhoto = async () => {
    if (permission?.status !== 'granted') {
      const result = await requestPermission({
        rationale: 'Camera access is needed to take photos for your profile',
        fallback: () => {
          Alert.alert('Camera Required', 'Please enable camera permissions in settings');
        },
      });

      if (result.status !== 'granted') {
        return;
      }
    }

    // Take photo logic
    const photo = await Camera.takePictureAsync();
    console.log('Photo taken:', photo);
  };

  if (loading) {
    return <ActivityIndicator />;
  }

  return (
    <View>
      <Button
        title="Take Photo"
        onPress={handleTakePhoto}
        disabled={permission?.status !== 'granted'}
      />
      {permission?.status === 'denied' && (
        <Text>Please grant camera permission to take photos</Text>
      )}
    </View>
  );
};
```

---

### 5.2 **Advanced Storage Solutions Comparison**

**Question:** *"Compare AsyncStorage, MMKV, SQLite, and Realm for different storage use cases in React Native applications."*

**Perfect Answer Structure:**
```
STORAGE SOLUTION COMPARISON:

🎯 AsyncStorage:
- Simple key-value storage
- Built-in, no dependencies
- Limited to 6MB total
- Asynchronous, unencrypted
- Best for: Simple preferences, small data

🎯 MMKV:
- High-performance key-value
- Encrypted storage option
- Cross-platform consistency
- Direct JavaScript object support
- Best for: Performance-critical data, encrypted storage

🎯 SQLite:
- Relational database
- Complex queries and relationships
- Large dataset handling
- SQL knowledge required
- Best for: Structured data, complex queries

🎯 Realm:
- Object database
- Real-time synchronization
- Advanced querying
- Schema-based
- Best for: Complex data models, real-time features
```

**Complete Storage Solutions Implementation:**
```javascript
// Storage abstraction layer
interface StorageInterface {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T): Promise<void>;
  remove(key: string): Promise<void>;
  clear(): Promise<void>;
  getAllKeys(): Promise<string[]>;
}

// AsyncStorage implementation
class AsyncStorageAdapter implements StorageInterface {
  async get<T>(key: string): Promise<T | null> {
    try {
      const value = await AsyncStorage.getItem(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('AsyncStorage get error:', error);
      return null;
    }
  }

  async set<T>(key: string, value: T): Promise<void> {
    try {
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('AsyncStorage set error:', error);
      throw error;
    }
  }

  async remove(key: string): Promise<void> {
    try {
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('AsyncStorage remove error:', error);
    }
  }

  async clear(): Promise<void> {
    try {
      await AsyncStorage.clear();
    } catch (error) {
      console.error('AsyncStorage clear error:', error);
    }
  }

  async getAllKeys(): Promise<string[]> {
    try {
      return await AsyncStorage.getAllKeys();
    } catch (error) {
      console.error('AsyncStorage getAllKeys error:', error);
      return [];
    }
  }
}

// MMKV implementation
class MMKVStorageAdapter implements StorageInterface {
  private instance: MMKV;

  constructor(id = 'default') {
    this.instance = new MMKV({
      id,
      encryptionKey: 'your-encryption-key', // Optional encryption
    });
  }

  async get<T>(key: string): Promise<T | null> {
    try {
      const value = this.instance.getString(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('MMKV get error:', error);
      return null;
    }
  }

  async set<T>(key: string, value: T): Promise<void> {
    try {
      this.instance.set(key, JSON.stringify(value));
    } catch (error) {
      console.error('MMKV set error:', error);
      throw error;
    }
  }

  async remove(key: string): Promise<void> {
    try {
      this.instance.delete(key);
    } catch (error) {
      console.error('MMKV remove error:', error);
    }
  }

  async clear(): Promise<void> {
    try {
      this.instance.clearAll();
    } catch (error) {
      console.error('MMKV clear error:', error);
    }
  }

  async getAllKeys(): Promise<string[]> {
    try {
      return this.instance.getAllKeys();
    } catch (error) {
      console.error('MMKV getAllKeys error:', error);
      return [];
    }
  }
}

// SQLite implementation
class SQLiteStorageAdapter implements StorageInterface {
  private db: SQLiteDatabase;

  constructor(databaseName = 'app.db') {
    this.db = SQLite.openDatabase(databaseName);
    this.initTables();
  }

  private async initTables(): Promise<void> {
    await this.db.executeSql(`
      CREATE TABLE IF NOT EXISTS key_value_store (
        key TEXT PRIMARY KEY,
        value TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
      )
    `);
  }

  async get<T>(key: string): Promise<T | null> {
    try {
      const [results] = await this.db.executeSql(
        'SELECT value FROM key_value_store WHERE key = ?',
        [key]
      );

      if (results.rows.length > 0) {
        return JSON.parse(results.rows.item(0).value);
      }
      return null;
    } catch (error) {
      console.error('SQLite get error:', error);
      return null;
    }
  }

  async set<T>(key: string, value: T): Promise<void> {
    try {
      await this.db.executeSql(
        `INSERT OR REPLACE INTO key_value_store (key, value, updated_at)
         VALUES (?, ?, datetime('now'))`,
        [key, JSON.stringify(value)]
      );
    } catch (error) {
      console.error('SQLite set error:', error);
      throw error;
    }
  }

  async remove(key: string): Promise<void> {
    try {
      await this.db.executeSql(
        'DELETE FROM key_value_store WHERE key = ?',
        [key]
      );
    } catch (error) {
      console.error('SQLite remove error:', error);
    }
  }

  async clear(): Promise<void> {
    try {
      await this.db.executeSql('DELETE FROM key_value_store');
    } catch (error) {
      console.error('SQLite clear error:', error);
    }
  }

  async getAllKeys(): Promise<string[]> {
    try {
      const [results] = await this.db.executeSql(
        'SELECT key FROM key_value_store'
      );

      const keys: string[] = [];
      for (let i = 0; i < results.rows.length; i++) {
        keys.push(results.rows.item(i).key);
      }
      return keys;
    } catch (error) {
      console.error('SQLite getAllKeys error:', error);
      return [];
    }
  }
}

// Storage manager with migration support
class StorageManager {
  private adapters: Map<string, StorageInterface> = new Map();
  private currentAdapter: StorageInterface;

  constructor() {
    // Initialize adapters
    this.adapters.set('asyncstorage', new AsyncStorageAdapter());
    this.adapters.set('mmkv', new MMKVStorageAdapter());
    this.adapters.set('sqlite', new SQLiteStorageAdapter());

    // Default to MMKV for performance
    this.currentAdapter = this.adapters.get('mmkv')!;
  }

  setAdapter(type: 'asyncstorage' | 'mmkv' | 'sqlite'): void {
    const adapter = this.adapters.get(type);
    if (adapter) {
      this.currentAdapter = adapter;
    }
  }

  async migrateData(fromType: string, toType: string): Promise<void> {
    const fromAdapter = this.adapters.get(fromType);
    const toAdapter = this.adapters.get(toType);

    if (!fromAdapter || !toAdapter) {
      throw new Error('Invalid adapter types');
    }

    const keys = await fromAdapter.getAllKeys();

    for (const key of keys) {
      const value = await fromAdapter.get(key);
      if (value !== null) {
        await toAdapter.set(key, value);
      }
    }
  }

  get<T>(key: string): Promise<T | null> {
    return this.currentAdapter.get<T>(key);
  }

  set<T>(key: string, value: T): Promise<void> {
    return this.currentAdapter.set<T>(key, value);
  }

  remove(key: string): Promise<void> {
    return this.currentAdapter.remove(key);
  }

  clear(): Promise<void> {
    return this.currentAdapter.clear();
  }

  getAllKeys(): Promise<string[]> {
    return this.currentAdapter.getAllKeys();
  }
}

// Usage examples
const storageManager = new StorageManager();

// User preferences (small, frequent access) - use MMKV
const userPrefs = {
  theme: 'dark',
  language: 'en',
  notifications: true,
};
await storageManager.set('userPreferences', userPrefs);

// Large dataset (offline data) - use SQLite
const offlineData = {
  posts: [], // Large array of posts
  comments: [],
  userCache: {},
};
await storageManager.set('offlineData', offlineData);

// Simple flags (compatibility) - use AsyncStorage
await storageManager.setAdapter('asyncstorage');
await storageManager.set('firstLaunch', false);
```

---

### 5.3 **Push Notifications Architecture**

**Question:** *"Design a comprehensive push notifications system handling FCM/APNs, deep linking, and background processing."*

**Perfect Answer Structure:**
```
PUSH NOTIFICATION ARCHITECTURE:

🎯 NOTIFICATION LIFECYCLE:
- Registration → Token management → Message handling
- Foreground/background processing
- Deep linking integration
- User interaction tracking

🎯 CROSS-PLATFORM HANDLING:
- FCM for Android, APNs for iOS
- Unified API abstraction
- Platform-specific features
- Token synchronization

🎯 ADVANCED FEATURES:
- Silent notifications
- Rich media attachments
- Action buttons
- Scheduled notifications

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 7 ➡️](phase7-styling-ui.md)