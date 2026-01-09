[⬅️ Back to Home](../README.md)

# 🎯 Phase 03: Navigation & App Architecture

---

)}
    />
  );
};
```

**Performance Monitoring:**
```javascript
// Add performance monitoring in development
if (__DEV__) {
  const renderCountRef = useRef(0);
  renderCountRef.current += 1;

  useEffect(() => {
    console.log(`Component rendered ${renderCountRef.current} times`);
  });
}
```

---

### 2.9 **Props Drilling Solutions and Context Patterns**

**Question:** *"Solve props drilling in a deep component tree using Context and custom hooks."*

**Perfect Answer Structure:**
```
PROPS DRILLING SOLUTIONS:

1️⃣ Context API (Simple cases)
2️⃣ State management libraries (Complex cases)
3️⃣ Component composition
4️⃣ Render props pattern
5️⃣ Custom hooks with context

WHEN TO USE EACH:
- 3-5 levels: Context API
- 5+ levels: Redux/Zustand
- Component-specific: Render props
- Reusable logic: Custom hooks
```

**Complete Implementation:**
```javascript
// Create context with custom hook
const UserContext = React.createContext();

const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
};

const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [preferences, setPreferences] = useState({});

  // Memoize context value to prevent unnecessary re-renders
  const contextValue = useMemo(() => ({
    user,
    preferences,
    setUser,
    setPreferences,
    // Computed values
    isLoggedIn: !!user,
    displayName: user ? `${user.firstName} ${user.lastName}` : 'Guest',
    // Actions
    login: (userData) => setUser(userData),
    logout: () => setUser(null),
    updatePreferences: (newPrefs) => setPreferences(prev => ({ ...prev, ...newPrefs })),
  }), [user, preferences]);

  return (
    <UserContext.Provider value={contextValue}>
      {children}
    </UserContext.Provider>
  );
};

// BEFORE: Props drilling through multiple levels
const App = () => (
  <UserProvider>
    <NavigationContainer>
      <TabNavigator>
        <Tab.Screen name="Home">
          {() => <HomeScreen user={user} preferences={preferences} />}
        </Tab.Screen>
        <Tab.Screen name="Profile">
          {() => <ProfileScreen user={user} setUser={setUser} />}
        </Tab.Screen>
      </TabNavigator>
    </NavigationContainer>
  </UserProvider>
);

// AFTER: Clean component tree with hooks
const HomeScreen = () => {
  const { displayName, isLoggedIn, preferences } = useUser();

  return (
    <View>
      <Text>Welcome {displayName}!</Text>
      {isLoggedIn && <Text>Theme: {preferences.theme}</Text>}
    </View>
  );
};

const ProfileScreen = () => {
  const { user, logout, updatePreferences } = useUser();

  return (
    <View>
      <Text>{user?.email}</Text>
      <Button title="Logout" onPress={logout} />
      <Button
        title="Dark Mode"
        onPress={() => updatePreferences({ theme: 'dark' })}
      />
    </View>
  );
};

// USAGE
const App = () => (
  <UserProvider>
    <NavigationContainer>
      <TabNavigator>
        <Tab.Screen name="Home" component={HomeScreen} />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </TabNavigator>
    </NavigationContainer>
  </UserProvider>
);
```

**Advanced Patterns:**
```javascript
// Context with reducer for complex state
const UserContext = React.createContext();

const userReducer = (state, action) => {
  switch (action.type) {
    case 'LOGIN':
      return { ...state, user: action.payload, isLoggedIn: true };
    case 'LOGOUT':
      return { ...state, user: null, isLoggedIn: false };
    case 'UPDATE_PREFERENCES':
      return { ...state, preferences: { ...state.preferences, ...action.payload } };
    default:
      return state;
  }
};

const UserProvider = ({ children }) => {
  const [state, dispatch] = useReducer(userReducer, {
    user: null,
    isLoggedIn: false,
    preferences: { theme: 'light' },
  });

  const contextValue = useMemo(() => ({
    ...state,
    login: (user) => dispatch({ type: 'LOGIN', payload: user }),
    logout: () => dispatch({ type: 'LOGOUT' }),
    updatePreferences: (prefs) => dispatch({ type: 'UPDATE_PREFERENCES', payload: prefs }),
  }), [state]);

  return (
    <UserContext.Provider value={contextValue}>
      {children}
    </UserContext.Provider>
  );
};
```

---

### 2.10 **Error Boundaries and Error Handling in React Native**

**Question:** *"Implement comprehensive error handling with error boundaries and recovery patterns."*

**Perfect Answer Structure:**
```
ERROR HANDLING STRATEGIES:

🎯 ERROR BOUNDARIES (Class components only):
- Catch JavaScript errors in component tree
- Display fallback UI
- Log errors for monitoring
- Prevent app crashes

🎯 HOOKS-BASED ERROR HANDLING:
- useErrorHandler hook
- Async error boundaries
- Error recovery patterns

🎯 NATIVE ERROR HANDLING:
- Unhandled promise rejections
- Native module errors
- Platform-specific errors
```

**Complete Error Handling Implementation:**
```javascript
// ERROR BOUNDARY COMPONENT
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log error to monitoring service
    console.error('Error caught by boundary:', error, errorInfo);

    this.setState({
      error,
      errorInfo,
    });

    // Report to error monitoring (e.g., Sentry)
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
  };

  render() {
    if (this.state.hasError) {
      return (
        <ErrorFallback
          error={this.state.error}
          onRetry={this.handleRetry}
          onReset={this.props.onReset}
        />
      );
    }

    return this.props.children;
  }
}

// ERROR FALLBACK COMPONENT
const ErrorFallback = ({ error, onRetry, onReset }) => (
  <View style={styles.errorContainer}>
    <Text style={styles.errorTitle}>Something went wrong</Text>
    <Text style={styles.errorMessage}>
      {error?.message || 'An unexpected error occurred'}
    </Text>

    <View style={styles.errorActions}>
      <Button title="Try Again" onPress={onRetry} />
      {onReset && <Button title="Reset App" onPress={onReset} />}
    </View>

    {__DEV__ && (
      <ScrollView style={styles.errorDetails}>
        <Text style={styles.errorStack}>{error?.stack}</Text>
      </ScrollView>
    )}
  </View>
);

// CUSTOM HOOK FOR ASYNC ERROR HANDLING
const useErrorHandler = () => {
  const [error, setError] = useState(null);

  const handleError = useCallback((error) => {
    console.error('Handled error:', error);
    setError(error);

    // Auto-clear error after 5 seconds
    setTimeout(() => setError(null), 5000);
  }, []);

  const clearError = useCallback(() => setError(null), []);

  // Handle unhandled promise rejections
  useEffect(() => {
    const handleUnhandledRejection = (event) => {
      console.error('Unhandled promise rejection:', event.reason);
      handleError(event.reason);
    };

    const handleErrorEvent = (event) => {
      console.error('Global error:', event.error);
      handleError(event.error);
    };

    window.addEventListener('unhandledrejection', handleUnhandledRejection);
    window.addEventListener('error', handleErrorEvent);

    return () => {
      window.removeEventListener('unhandledrejection', handleUnhandledRejection);
      window.removeEventListener('error', handleErrorEvent);
    };
  }, [handleError]);

  return { error, handleError, clearError };
};

// USAGE EXAMPLE
const DataComponent = () => {
  const { error, handleError, clearError } = useErrorHandler();

  const fetchData = async () => {
    try {
      const result = await riskyApiCall();
      // Handle success
    } catch (err) {
      handleError(err);
    }
  };

  if (error) {
    return (
      <View>
        <Text>Error: {error.message}</Text>
        <Button title="Retry" onPress={fetchData} />
        <Button title="Dismiss" onPress={clearError} />
      </View>
    );
  }

  return <Button title="Fetch Data" onPress={fetchData} />;
};

// APP-LEVEL ERROR BOUNDARY
const App = () => {
  const handleGlobalError = (error, errorInfo) => {
    // Send to monitoring service
    analytics.logError(error, errorInfo);
  };

  const handleReset = () => {
    // Reset app state
    AsyncStorage.clear();
    // Restart app or navigate to welcome screen
  };

  return (
    <ErrorBoundary onError={handleGlobalError} onReset={handleReset}>
      <NavigationContainer>
        {/* App content */}
      </NavigationContainer>
    </ErrorBoundary>
  );
};
```

**Error Recovery Patterns:**
```javascript
// RECOVERY COMPONENT
const withErrorRecovery = (Component) => {
  return (props) => {
    const [retryCount, setRetryCount] = useState(0);
    const maxRetries = 3;

    const handleRetry = () => {
      if (retryCount < maxRetries) {
        setRetryCount(prev => prev + 1);
      }
    };

    try {
      return <Component {...props} retryCount={retryCount} />;
    } catch (error) {
      if (retryCount < maxRetries) {
        return (
          <View>
            <Text>Component failed, retrying... ({retryCount + 1}/{maxRetries})</Text>
            <Button title="Retry" onPress={handleRetry} />
          </View>
        );
      }

      throw error; // Let error boundary handle it
    }
  };
};
```

---

*Phase 2 continues with advanced topics covering concurrent features, suspense, advanced hooks patterns, and performance profiling techniques. Each question includes production-ready code examples and real-world implementation strategies.*

---

## 🎯 Phase 3 — Navigation & App Architecture

**Navigation questions are extremely common** - expect 2-3 questions in senior interviews. Focus on React Navigation v6+ and scalable architecture patterns.

### 3.1 **React Navigation Stack vs Tab vs Drawer Navigator**

**Question:** *"Design a complete navigation structure for a social media app with authentication flows."*

**Perfect Answer Structure:**
```
NAVIGATION ARCHITECTURE DESIGN:

🎯 AUTH FLOW (Unauthenticated):
- Stack Navigator (Login → Register → Forgot Password)
- Modal presentation for auth screens
- No tab bar visible

🎯 MAIN APP FLOW (Authenticated):
- Tab Navigator (Home, Search, Profile, Notifications)
- Stack navigators nested in each tab
- Drawer navigator for settings/menu
- Deep linking support

🎯 NAVIGATION STATE MANAGEMENT:
- Global auth state
- Navigation persistence
- Conditional rendering based on auth status
```

**Complete Implementation:**
```javascript
// navigation/AuthNavigator.js
import { createStackNavigator } from '@react-navigation/stack';

const AuthStack = createStackNavigator();

export const AuthNavigator = () => (
  <AuthStack.Navigator
    screenOptions={{
      headerShown: false,
      presentation: 'modal',
    }}
  >
    <AuthStack.Screen name="Login" component={LoginScreen} />
    <AuthStack.Screen name="Register" component={RegisterScreen} />
    <AuthStack.Screen name="ForgotPassword" component={ForgotPasswordScreen} />
  </AuthStack.Navigator>
);

// navigation/HomeStackNavigator.js
const HomeStack = createStackNavigator();

export const HomeStackNavigator = () => (
  <HomeStack.Navigator
    screenOptions={{
      headerStyle: { backgroundColor: '#fff' },
      headerTintColor: '#000',
    }}
  >
    <HomeStack.Screen
      name="Home"
      component={HomeScreen}
      options={{ title: 'Home' }}
    />
    <HomeStack.Screen
      name="PostDetail"
      component={PostDetailScreen}
      options={({ route }) => ({ title: route.params?.post?.title || 'Post' })}
    />
    <HomeStack.Screen
      name="UserProfile"
      component={UserProfileScreen}
      options={{ title: 'Profile' }}
    />
  </HomeStack.Navigator>
);

// navigation/MainTabNavigator.js
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

export const MainTabNavigator = () => (
  <Tab.Navigator
    screenOptions={({ route }) => ({
      tabBarIcon: ({ focused, color, size }) => {
        let iconName;

        if (route.name === 'HomeTab') {
          iconName = focused ? 'home' : 'home-outline';
        } else if (route.name === 'SearchTab') {
          iconName = focused ? 'search' : 'search-outline';
        } else if (route.name === 'ProfileTab') {
          iconName = focused ? 'person' : 'person-outline';
        }

        return <Ionicons name={iconName} size={size} color={color} />;
      },
      tabBarActiveTintColor: '#007AFF',
      tabBarInactiveTintColor: 'gray',
      headerShown: false,
    })}
  >
    <Tab.Screen name="HomeTab" component={HomeStackNavigator} />
    <Tab.Screen name="SearchTab" component={SearchStackNavigator} />
    <Tab.Screen name="ProfileTab" component={ProfileStackNavigator} />
  </Tab.Navigator>
);

// navigation/AppNavigator.js
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

export const AppNavigator = () => {
  const { isAuthenticated } = useAuth();

  return (
    <NavigationContainer>
      {isAuthenticated ? (
        <Drawer.Navigator
          drawerContent={(props) => <CustomDrawerContent {...props} />}
          screenOptions={{
            headerShown: false,
          }}
        >
          <Drawer.Screen name="MainTabs" component={MainTabNavigator} />
          <Drawer.Screen name="Settings" component={SettingsScreen} />
        </Drawer.Navigator>
      ) : (
        <AuthNavigator />
      )}
    </NavigationContainer>
  );
};
```

**Navigation State Persistence:**
```javascript
// hooks/useNavigationPersistence.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useNavigationContainerRef } from '@react-navigation/native';

export const useNavigationPersistence = () => {
  const navigationRef = useNavigationContainerRef();

  const [isReady, setIsReady] = useState(false);
  const [initialState, setInitialState] = useState();

  useEffect(() => {
    const restoreState = async () => {
      try {
        const savedStateString = await AsyncStorage.getItem('NAVIGATION_STATE');
        const state = savedStateString ? JSON.parse(savedStateString) : undefined;

        if (state !== undefined) {
          setInitialState(state);
        }
      } finally {
        setIsReady(true);
      }
    };

    if (!isReady) {
      restoreState();
    }
  }, [isReady]);

  const onStateChange = useCallback((state) => {
    AsyncStorage.setItem('NAVIGATION_STATE', JSON.stringify(state));
  }, []);

  return {
    isReady,
    initialState,
    navigationRef,
    onStateChange,
  };
};

// USAGE
const App = () => {
  const { isReady, initialState, navigationRef, onStateChange } = useNavigationPersistence();

  if (!isReady) {
    return <SplashScreen />;
  }

  return (
    <NavigationContainer
      ref={navigationRef}
      initialState={initialState}
      onStateChange={onStateChange}
    >
      <AppNavigator />
    </NavigationContainer>
  );
};
```

---

### 3.2 **Deep Linking and URL Routing Strategy**

**Question:** *"Implement deep linking for a content-sharing app with push notification handling."*

**Perfect Answer Structure:**
```
DEEP LINKING ARCHITECTURE:

🎯 URL SCHEMES:
- Custom scheme: myapp://
- Universal links (iOS): https://myapp.com/
- App links (Android): https://myapp.com/

🎯 LINK TYPES:
- Content links: /post/123
- User profiles: /user/john
- Actions: /invite/abc123

🎯 FALLBACK HANDLING:
- App not installed → Web fallback
- Invalid links → Error screen
- Authentication required → Login flow
```

**Complete Deep Linking Implementation:**
```javascript
// navigation/LinkingConfiguration.js
import { Linking } from 'react-native';

export const linking = {
  prefixes: [
    'myapp://',
    'https://myapp.com',
    'https://*.myapp.com', // Universal links
  ],

  config: {
    screens: {
      Auth: {
        screens: {
          Login: 'login',
          Register: 'register',
        },
      },
      MainTabs: {
        screens: {
          HomeTab: {
            screens: {
              Home: 'home',
              PostDetail: 'post/:postId',
            },
          },
          ProfileTab: {
            screens: {
              Profile: 'profile',
              UserProfile: 'user/:userId',
            },
          },
          SearchTab: {
            screens: {
              Search: 'search',
              SearchResults: 'search/:query',
            },
          },
        },
      },
      Settings: 'settings',
      NotFound: '*',
    },
  },

  // Custom link handling
  subscribe(listener) {
    const onReceiveURL = ({ url }) => {
      listener(url);
    };

    const subscription = Linking.addEventListener('url', onReceiveURL);

    // Handle initial URL
    Linking.getInitialURL().then((url) => {
      if (url) {
        listener(url);
      }
    });

    return () => {
      subscription?.remove();
    };
  },
};

// navigation/useDeepLinkHandler.js
export const useDeepLinkHandler = () => {
  const navigation = useNavigation();

  useEffect(() => {
    const handleDeepLink = (url) => {
      console.log('Received deep link:', url);

      // Parse custom URL schemes
      if (url.startsWith('myapp://')) {
        const route = url.replace('myapp://', '');
        handleCustomScheme(route);
      }
      // Universal/App links handled by React Navigation automatically
    };

    const handleCustomScheme = (route) => {
      const [screen, params] = parseCustomRoute(route);

      if (screen === 'invite') {
        // Handle invitation links
        navigation.navigate('Auth', {
          screen: 'Register',
          params: { inviteCode: params.code },
        });
      } else if (screen === 'notification') {
        // Handle notification deep links
        handleNotificationLink(params);
      }
    };

    const parseCustomRoute = (route) => {
      // Custom parsing logic for myapp://invite/ABC123
      const parts = route.split('/');
      return [parts[0], { code: parts[1] }];
    };

    const handleNotificationLink = (params) => {
      if (params.type === 'like') {
        navigation.navigate('MainTabs', {
          screen: 'HomeTab',
          params: {
            screen: 'PostDetail',
            params: { postId: params.postId },
          },
        });
      }
    };

    Linking.addEventListener('url', handleDeepLink);

    return () => {
      // Cleanup
    };
  }, [navigation]);
};

// Push Notification Deep Linking
import messaging from '@react-native-firebase/messaging';

export const usePushNotificationHandler = () => {
  const navigation = useNavigation();

  useEffect(() => {
    const unsubscribe = messaging().onMessage(async (remoteMessage) => {
      if (remoteMessage.data?.deepLink) {
        const deepLink = remoteMessage.data.deepLink;

        // Navigate based on notification type
        if (deepLink.includes('/post/')) {
          const postId = deepLink.split('/post/')[1];
          navigation.navigate('PostDetail', { postId });
        }
      }
    });

    return unsubscribe;
  }, [navigation]);
};

// App.js integration
const App = () => {
  useDeepLinkHandler();
  usePushNotificationHandler();

  return (
    <NavigationContainer linking={linking}>
      <AppNavigator />
    </NavigationContainer>
  );
};
```

**URL Generation and Sharing:**
```javascript
// utils/deepLinkUtils.js
export const generateDeepLink = (route, params = {}) => {
  const baseUrl = 'myapp://';

  switch (route) {
    case 'post':
      return `${baseUrl}post/${params.postId}`;
    case 'user':
      return `${baseUrl}user/${params.userId}`;
    case 'invite':
      return `${baseUrl}invite/${params.inviteCode}`;
    default:
      return baseUrl;
  }
};

export const shareContent = async (contentType, contentId) => {
  const link = generateDeepLink(contentType, { [contentType + 'Id']: contentId });

  try {
    const result = await Share.share({
      message: `Check this out: ${link}`,
      url: link, // iOS only
    });

    if (result.action === Share.sharedAction) {
      analytics.track('content_shared', { contentType, contentId });
    }
  } catch (error) {
    console.error('Error sharing:', error);
  }
};
```

---

### 3.3 **Authentication Flow with Navigation Guards**

**Question:** *"Implement protected routes with authentication state management and navigation guards."*

**Perfect Answer Structure:**
```
AUTHENTICATION FLOW ARCHITECTURE:

🎯 NAVIGATION GUARDS:
- Route protection based on auth state
- Automatic redirects (login → protected routes)
- Persisted login state across app restarts

🎯 AUTH STATE MANAGEMENT:
- Secure token storage
- Auto token refresh
- Logout handling with navigation reset

🎯 USER EXPERIENCE:
- Smooth transitions between auth states
- Loading states during auth checks
- Error handling for auth failures
```

**Complete Authentication Implementation:**
```javascript
// context/AuthContext.js
const AuthContext = React.createContext();

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [state, setState] = useState({
    user: null,
    token: null,
    isLoading: true,
    isAuthenticated: false,
  });

  // Initialize auth state from storage
  useEffect(() => {
    const initializeAuth = async () => {
      try {
        const storedToken = await SecureStore.getItemAsync('userToken');
        const storedUser = await SecureStore.getItemAsync('userData');

        if (storedToken && storedUser) {

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 4 ➡️](phase4-performance-memory.md)