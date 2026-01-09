[⬅️ Back to Home](../README.md)

# 🎯 Phase 08: State Management Patterns

---

expect(getByText('John Doe')).toBeTruthy();
    expect(getByText('john@example.com')).toBeTruthy();
    expect(getByTestId('user-avatar')).toBeTruthy();
  });

  it('calls onPress when pressed', () => {
    const { getByTestId } = render(
      <UserCard user={mockUser} onPress={mockOnPress} />
    );

    fireEvent.press(getByTestId('user-card'));
    expect(mockOnPress).toHaveBeenCalledWith(mockUser);
  });

  it('shows delete button when showDelete is true', () => {
    const { getByTestId } = render(
      <UserCard
        user={mockUser}
        onPress={mockOnPress}
        showDelete
        onDelete={mockOnDelete}
      />
    );

    expect(getByTestId('delete-button')).toBeTruthy();
  });

  it('calls onDelete when delete button is pressed', () => {
    const { getByTestId } = render(
      <UserCard
        user={mockUser}
        onPress={mockOnPress}
        showDelete
        onDelete={mockOnDelete}
      />
    );

    fireEvent.press(getByTestId('delete-button'));
    expect(mockOnDelete).toHaveBeenCalledWith(mockUser.id);
  });

  it('handles loading state', () => {
    const { getByTestId } = render(
      <UserCard user={mockUser} onPress={mockOnPress} loading />
    );

    expect(getByTestId('loading-indicator')).toBeTruthy();
  });

  it('applies correct styles based on theme', () => {
    const { getByTestId } = render(
      <UserCard user={mockUser} onPress={mockOnPress} theme="dark" />
    );

    const card = getByTestId('user-card');
    expect(card.props.style).toContainEqual(
      expect.objectContaining({ backgroundColor: '#333' })
    );
  });
});

// Hook testing example
import { renderHook, act, waitFor } from '@testing-library/react-native';
import { useCounter } from '../useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());

    expect(result.current.count).toBe(0);
  });

  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(5));

    expect(result.current.count).toBe(5);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  it('resets count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.reset();
    });

    expect(result.current.count).toBe(0);
  });

  it('calls onChange callback', () => {
    const mockOnChange = jest.fn();
    const { result } = renderHook(() => useCounter(0, mockOnChange));

    act(() => {
      result.current.increment();
    });

    expect(mockOnChange).toHaveBeenCalledWith(1);
  });
});

// Integration testing example
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { fireEvent, render, waitFor } from '@testing-library/react-native';

const Stack = createStackNavigator();

const mockApi = {
  login: jest.fn(),
  getUserProfile: jest.fn(),
};

jest.mock('../api/auth', () => mockApi);

describe('Authentication Flow', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('navigates from login to home on successful authentication', async () => {
    mockApi.login.mockResolvedValueOnce({ token: 'fake-token' });
    mockApi.getUserProfile.mockResolvedValueOnce({
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
    });

    const { getByPlaceholderText, getByText, queryByText } = render(
      <NavigationContainer>
        <Stack.Navigator>
          <Stack.Screen name="Login" component={LoginScreen} />
          <Stack.Screen name="Home" component={HomeScreen} />
        </Stack.Navigator>
      </NavigationContainer>
    );

    // Enter credentials
    fireEvent.changeText(getByPlaceholderText('Email'), 'john@example.com');
    fireEvent.changeText(getByPlaceholderText('Password'), 'password123');

    // Submit form
    fireEvent.press(getByText('Login'));

    // Wait for navigation
    await waitFor(() => {
      expect(queryByText('Welcome, John Doe')).toBeTruthy();
    });

    expect(mockApi.login).toHaveBeenCalledWith('john@example.com', 'password123');
    expect(mockApi.getUserProfile).toHaveBeenCalled();
  });

  it('shows error message on login failure', async () => {
    mockApi.login.mockRejectedValueOnce(new Error('Invalid credentials'));

    const { getByPlaceholderText, getByText } = render(<LoginScreen />);

    fireEvent.changeText(getByPlaceholderText('Email'), 'john@example.com');
    fireEvent.changeText(getByPlaceholderText('Password'), 'wrongpassword');

    fireEvent.press(getByText('Login'));

    await waitFor(() => {
      expect(getByText('Invalid credentials')).toBeTruthy();
    });
  });
});

// E2E testing with Detox
// e2e/login.test.js
describe('Login Flow', () => {
  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should login successfully', async () => {
    await element(by.id('email-input')).typeText('john@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();

    await expect(element(by.text('Welcome, John Doe'))).toBeVisible();
    await expect(element(by.id('home-screen'))).toBeVisible();
  });

  it('should show error for invalid credentials', async () => {
    await element(by.id('email-input')).typeText('invalid@example.com');
    await element(by.id('password-input')).typeText('wrongpassword');
    await element(by.id('login-button')).tap();

    await expect(element(by.text('Invalid credentials'))).toBeVisible();
  });

  it('should navigate through user profile flow', async () => {
    // Login first
    await element(by.id('email-input')).typeText('john@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();

    // Navigate to profile
    await element(by.id('profile-tab')).tap();
    await expect(element(by.id('user-profile-screen'))).toBeVisible();

    // Edit profile
    await element(by.id('edit-profile-button')).tap();
    await element(by.id('name-input')).typeText('John Smith');
    await element(by.id('save-button')).tap();

    await expect(element(by.text('Profile updated successfully'))).toBeVisible();
  });
});

// Performance testing utilities
export const measureRenderTime = async (component, iterations = 10) => {
  const times = [];

  for (let i = 0; i < iterations; i++) {
    const startTime = performance.now();
    render(component);
    const endTime = performance.now();
    times.push(endTime - startTime);
  }

  const averageTime = times.reduce((a, b) => a + b, 0) / times.length;
  const minTime = Math.min(...times);
  const maxTime = Math.max(...times);

  return { averageTime, minTime, maxTime, allTimes: times };
};

export const expectPerformance = (metrics, threshold) => {
  expect(metrics.averageTime).toBeLessThan(threshold);
};

// Performance test example
describe('Component Performance', () => {
  it('renders within performance budget', async () => {
    const metrics = await measureRenderTime(
      <UserList users={largeUserDataset} />,
      20
    );

    expectPerformance(metrics, 100); // 100ms budget
  });
});
```

---

### 6.2 **Advanced Debugging Techniques and Tools**

**Question:** *"Demonstrate advanced debugging techniques for React Native applications including Flipper, React DevTools, and performance profiling."*

**Perfect Answer Structure:**
```
DEBUGGING TOOLKIT:

🎯 DEVELOPMENT DEBUGGING:
- React DevTools for component inspection
- Flipper for native debugging
- Metro bundler debugging
- Hot reloading strategies

🎯 PRODUCTION DEBUGGING:
- Crash reporting integration
- Performance monitoring
- Error boundary logging
- Remote debugging setup

🎯 PERFORMANCE PROFILING:
- Memory leak detection
- Render performance analysis
- Network request monitoring
- Bundle size analysis
```

**Complete Debugging Implementation:**
```javascript
// Flipper integration setup
// flipper.config.js
module.exports = {
  android: {
    applicationId: 'com.myapp',
  },
  ios: {
    project: 'MyApp.xcworkspace',
    scheme: 'MyApp',
  },
  plugins: [
    'flipper-plugin-react-native-performance',
    'flipper-plugin-async-storage',
    'flipper-plugin-network',
    'flipper-plugin-redux-debugger',
    'flipper-plugin-rn-idle',
  ],
};

// Custom Flipper plugin for app-specific debugging
// plugins/CustomPlugin.js
const { FlipperPlugin } = require('flipper');

class CustomPlugin extends FlipperPlugin {
  static id = 'custom-debug-plugin';

  static defaultPersistedState = {
    logs: [],
    events: [],
  };

  static persistedStateReducer = (state, method, payload) => {
    switch (method) {
      case 'log':
        return {
          ...state,
          logs: [...state.logs, { timestamp: Date.now(), message: payload }],
        };
      case 'event':
        return {
          ...state,
          events: [...state.events, { timestamp: Date.now(), ...payload }],
        };
      case 'clear':
        return { logs: [], events: [] };
      default:
        return state;
    }
  };

  render() {
    return (
      <div>
        <h2>Custom Debug Plugin</h2>
        <button onClick={() => this.send('clear', null)}>
          Clear Logs
        </button>
        <div>
          <h3>Logs</h3>
          <pre>{JSON.stringify(this.props.logs, null, 2)}</pre>
        </div>
        <div>
          <h3>Events</h3>
          <pre>{JSON.stringify(this.props.events, null, 2)}</pre>
        </div>
      </div>
    );
  }
}

// React Native bridge for Flipper communication
import { addPlugin } from 'react-native-flipper';

if (__DEV__) {
  addPlugin({
    id: 'custom-debug-plugin',
    name: 'Custom Debug',
    plugin: () => require('./plugins/CustomPlugin'),
    supportedApp: 'React Native',
  });
}

// Debug utilities for development
export const DebugUtils = {
  log: (message, data = null) => {
    if (__DEV__) {
      console.log(`[DEBUG] ${message}`, data);

      // Send to Flipper if available
      if (global.flipper) {
        global.flipper.send('custom-debug-plugin', 'log', message);
      }
    }
  },

  measurePerformance: (label, fn) => {
    if (!__DEV__) return fn();

    const start = performance.now();
    const result = fn();
    const end = performance.now();

    console.log(`[PERF] ${label}: ${(end - start).toFixed(2)}ms`);

    return result;
  },

  trackRender: (componentName) => {
    if (!__DEV__) return;

    let renderCount = 0;

    return () => {
      renderCount++;
      console.log(`[RENDER] ${componentName}: ${renderCount} times`);
    };
  },

  inspectState: (componentName, state) => {
    if (!__DEV__) return;

    console.group(`[STATE] ${componentName}`);
    console.log('Current state:', state);
    console.groupEnd();
  },
};

// Enhanced error boundary with debugging
class DebugErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({ error, errorInfo });

    // Enhanced error logging
    const errorDetails = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href,
    };

    console.error('Error Boundary Caught:', errorDetails);

    // Send to error monitoring
    if (!__DEV__) {
      errorReporting.captureException(error, {
        extra: errorDetails,
        tags: {
          component: 'ErrorBoundary',
          environment: __DEV__ ? 'development' : 'production',
        },
      });
    }

    // Send to Flipper in development
    if (__DEV__ && global.flipper) {
      global.flipper.send('custom-debug-plugin', 'event', {
        type: 'error_boundary',
        error: errorDetails,
      });
    }
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
  };

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.errorContainer}>
          <Text style={styles.errorTitle}>Something went wrong</Text>
          <Text style={styles.errorMessage}>
            {this.state.error?.message || 'An unexpected error occurred'}
          </Text>

          {__DEV__ && (
            <ScrollView style={styles.errorDetails}>
              <Text style={styles.errorStack}>
                {this.state.error?.stack}
              </Text>
              <Text style={styles.componentStack}>
                {this.state.errorInfo?.componentStack}
              </Text>
            </ScrollView>
          )}

          <View style={styles.errorActions}>
            <Button title="Retry" onPress={this.handleRetry} />
            <Button title="Report Bug" onPress={this.reportBug} />
          </View>
        </View>
      );
    }

    return this.props.children;
  }
}

// Performance monitoring hook
export const usePerformanceMonitor = (componentName) => {
  const renderCountRef = useRef(0);
  const lastRenderTimeRef = useRef(performance.now());
  const totalRenderTimeRef = useRef(0);

  useEffect(() => {
    const now = performance.now();
    const renderTime = now - lastRenderTimeRef.current;
    totalRenderTimeRef.current += renderTime;
    renderCountRef.current += 1;

    if (__DEV__) {
      console.log(`[PERF] ${componentName} render #${renderCountRef.current}: ${renderTime.toFixed(2)}ms`);

      // Warn about slow renders
      if (renderTime > 16.67) { // More than one frame at 60fps
        console.warn(`[PERF] Slow render in ${componentName}: ${renderTime.toFixed(2)}ms`);
      }

      // Send to Flipper
      if (global.flipper) {
        global.flipper.send('custom-debug-plugin', 'event', {
          type: 'performance',
          component: componentName,
          renderTime,
          renderCount: renderCountRef.current,
          averageRenderTime: totalRenderTimeRef.current / renderCountRef.current,
        });
      }
    }

    lastRenderTimeRef.current = now;
  });

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (__DEV__) {
        const averageTime = totalRenderTimeRef.current / renderCountRef.current;
        console.log(`[PERF] ${componentName} unmounted. Average render time: ${averageTime.toFixed(2)}ms`);
      }
    };
  }, [componentName]);

  return {
    renderCount: renderCountRef.current,
    averageRenderTime: totalRenderTimeRef.current / renderCountRef.current,
  };
};

// Network request debugging
class NetworkDebugger {
  private static instance: NetworkDebugger;
  private requests: Map<string, any> = new Map();

  static getInstance(): NetworkDebugger {
    if (!NetworkDebugger.instance) {
      NetworkDebugger.instance = new NetworkDebugger();
    }
    return NetworkDebugger.instance;
  }

  trackRequest(url: string, method: string, startTime: number) {
    const id = `${method}_${url}_${startTime}`;
    this.requests.set(id, {
      id,
      url,
      method,
      startTime,
      status: 'pending',
    });

    if (__DEV__) {
      console.log(`[NETWORK] ${method} ${url} - Started`);
    }
  }

  completeRequest(url: string, method: string, startTime: number, status: number, responseTime: number) {
    const id = `${method}_${url}_${startTime}`;
    const request = this.requests.get(id);

    if (request) {
      request.status = 'completed';
      request.statusCode = status;
      request.duration = responseTime - startTime;

      this.requests.delete(id);

      if (__DEV__) {
        const duration = (responseTime - startTime).toFixed(2);
        console.log(`[NETWORK] ${method} ${url} - ${status} (${duration}ms)`);

        // Warn about slow requests
        if (request.duration > 5000) {
          console.warn(`[NETWORK] Slow request: ${method} ${url} took ${duration}ms`);
        }
      }

      // Send to Flipper
      if (__DEV__ && global.flipper) {
        global.flipper.send('custom-debug-plugin', 'event', {
          type: 'network',
          ...request,
        });
      }
    }
  }
}

// Axios interceptor for network debugging
import axios from 'axios';

const networkDebugger = NetworkDebugger.getInstance();

axios.interceptors.request.use((config) => {
  const startTime = performance.now();
  networkDebugger.trackRequest(config.url!, config.method!.toUpperCase(), startTime);

  // Store start time for response interceptor
  (config as any).startTime = startTime;

  return config;
});

axios.interceptors.response.use((response) => {
  const startTime = (response.config as any).startTime;
  networkDebugger.completeRequest(
    response.config.url!,
    response.config.method!.toUpperCase(),
    startTime,
    response.status,
    performance.now()
  );

  return response;
}, (error) => {
  const startTime = (error.config as any)?.startTime;
  if (startTime) {
    networkDebugger.completeRequest(
      error.config.url,
      error.config.method.toUpperCase(),
      startTime,
      error.response?.status || 0,
      performance.now()
    );
  }

  return Promise.reject(error);
});

// Memory leak detection
export const useMemoryLeakDetector = (componentName: string) => {
  const mountedRef = useRef(true);
  const renderCountRef = useRef(0);

  useEffect(() => {
    renderCountRef.current += 1;
  });

  useEffect(() => {
    return () => {
      mountedRef.current = false;

      if (__DEV__) {
        // Schedule a check to see if component is still referenced
        setTimeout(() => {
          if (!mountedRef.current) {
            console.log(`[MEMORY] ${componentName} unmounted successfully`);
          }
        }, 1000);
      }
    };
  }, [componentName]);

  // In development, add a warning for frequent re-renders
  useEffect(() => {
    if (__DEV__ && renderCountRef.current > 10) {
      console.warn(`[MEMORY] ${componentName} has rendered ${renderCountRef.current} times - possible performance issue`);
    }
  });

  return mountedRef.current;
};

// Usage in components
const DebuggedComponent = () => {
  usePerformanceMonitor('DebuggedComponent');
  useMemoryLeakDetector('DebuggedComponent');

  DebugUtils.log('Component rendered', { timestamp: Date.now() });

  return (
    <DebugErrorBoundary>
      <Text>Debugged Component</Text>
    </DebugErrorBoundary>
  );
};
```

---

### 6.3 **CI/CD Pipeline and Deployment Automation**

**Question:** *"Design a comprehensive CI/CD pipeline for React Native applications with automated testing, building, and deployment to app stores."*

**Perfect Answer Structure:**
```
CI/CD PIPELINE ARCHITECTURE:

🎯 CONTINUOUS INTEGRATION:
- Automated testing on every PR
- Code quality checks (linting, type checking)
- Bundle size monitoring
- Security vulnerability scanning

🎯 CONTINUOUS DEPLOYMENT:
- Automated app store deployments
- Beta testing distribution
- Rollback strategies
- Environment management

🎯 MONITORING & ALERTS:
- Build failure notifications
- Performance regression alerts
- Deployment status tracking
- Error rate monitoring
```

**Complete CI/CD Implementation:**
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'yarn'

    - name: Install dependencies
      run: yarn install --frozen-lockfile

    - name: Type check
      run: yarn typecheck

    - name: Lint code
      run: yarn lint

    - name: Run unit tests
      run: yarn test:unit --coverage --watchAll=false

    - name: Run integration tests
      run: yarn test:integration

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

    - name: Bundle size check
      run: yarn bundle:analyze
      env:
        BUNDLE_ANALYZE_TOKEN: ${{ secrets.BUNDLE_ANALYZE_TOKEN }}

  build-android:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Setup Java
      uses: actions/setup-java@v3
      with:
        distribution: 'temurin'
        java-version: '11'

    - name: Setup Android SDK
      uses: android-actions/setup-android@v2

    - name: Build Android release
      run: |
        cd android
        ./gradlew assembleRelease --no-daemon
      env:
        ORG_GRADLE_PROJECT_NEW_ARCHITECTURE_ENABLED: true

    - name: Upload Android build artifact
      uses: actions/upload-artifact@v3
      with:
        name: android-release
        path: android/app/build/outputs/apk/release/app-release.apk

  build-ios:
    needs: test
    runs-on: macos-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'yarn'

    - name: Install dependencies
      run: yarn install --frozen-lockfile

    - name: Cache CocoaPods
      uses: actions/cache@v3
      with:
        path: ios/Pods
        key: ${{ runner.os }}-pods-${{ hashFiles('**/Podfile.lock') }}

    - name: Install CocoaPods
      run: |
        cd ios
        pod install

    - name: Build iOS release
      run: |
        cd ios
        xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -sdk iphoneos -archivePath $PWD/build/MyApp.xcarchive archive
      env:
        DEVELOPER_DIR: /Applications/Xcode_14.2.app/Contents/Developer

    - name: Export IPA
      run: |
        cd ios
        xcodebuild -exportArchive -archivePath $PWD/build/MyApp.xcarchive -exportOptionsPlist exportOptions.plist -exportPath $PWD/build

    - name: Upload iOS build artifact
      uses: actions/upload-artifact@v3
      with:
        name: ios-release
        path: ios/build/MyApp.ipa

# .github/workflows/deploy.yml
name: Deploy to Stores

on:
  workflow_run:
    workflows: ["CI"]
    branches: [main]
    types:
      - completed

jobs:
  deploy-android:

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 9 ➡️](phase9-animations-gestures.md)