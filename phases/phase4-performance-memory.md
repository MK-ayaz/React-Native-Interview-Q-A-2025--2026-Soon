[⬅️ Back to Home](../README.md)

# 🎯 Phase 04: Performance Optimization & Memory Management

---

const user = JSON.parse(storedUser);

          // Validate token with server
          const isValid = await validateToken(storedToken);

          if (isValid) {
            setState({
              user,
              token: storedToken,
              isLoading: false,
              isAuthenticated: true,
            });
          } else {
            // Token expired, clear storage
            await clearAuthStorage();
            setState(prev => ({ ...prev, isLoading: false }));
          }
        } else {
          setState(prev => ({ ...prev, isLoading: false }));
        }
      } catch (error) {
        console.error('Auth initialization error:', error);
        setState(prev => ({ ...prev, isLoading: false }));
      }
    };

    initializeAuth();
  }, []);

  const login = async (email, password) => {
    try {
      setState(prev => ({ ...prev, isLoading: true }));

      const response = await authAPI.login(email, password);
      const { user, token } = response.data;

      // Store securely
      await SecureStore.setItemAsync('userToken', token);
      await SecureStore.setItemAsync('userData', JSON.stringify(user));

      setState({
        user,
        token,
        isLoading: false,
        isAuthenticated: true,
      });

      return { success: true };
    } catch (error) {
      setState(prev => ({ ...prev, isLoading: false }));
      return { success: false, error: error.message };
    }
  };

  const logout = async () => {
    try {
      setState(prev => ({ ...prev, isLoading: true }));

      // Call logout API if needed
      if (state.token) {
        await authAPI.logout(state.token);
      }

      // Clear storage
      await clearAuthStorage();

      setState({
        user: null,
        token: null,
        isLoading: false,
        isAuthenticated: false,
      });
    } catch (error) {
      console.error('Logout error:', error);
      // Force logout even if API call fails
      await clearAuthStorage();
      setState({
        user: null,
        token: null,
        isLoading: false,
        isAuthenticated: false,
      });
    }
  };

  const clearAuthStorage = async () => {
    await SecureStore.deleteItemAsync('userToken');
    await SecureStore.deleteItemAsync('userData');
  };

  const contextValue = useMemo(() => ({
    ...state,
    login,
    logout,
  }), [state]);

  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
};

// navigation/ProtectedNavigator.js
export const ProtectedNavigator = ({ children }) => {
  const { isAuthenticated, isLoading } = useAuth();
  const navigation = useNavigation();

  useEffect(() => {
    if (!isLoading && !isAuthenticated) {
      // Redirect to auth flow
      navigation.reset({
        index: 0,
        routes: [{ name: 'Auth' }],
      });
    }
  }, [isAuthenticated, isLoading, navigation]);

  if (isLoading) {
    return <LoadingScreen />;
  }

  if (!isAuthenticated) {
    return null; // Will redirect via useEffect
  }

  return children;
};

// navigation/AppNavigator.js
export const AppNavigator = () => {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <SplashScreen />;
  }

  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {isAuthenticated ? (
          // Authenticated user flow
          <Stack.Screen name="MainApp">
            {() => (
              <ProtectedNavigator>
                <Drawer.Navigator>
                  <Drawer.Screen name="MainTabs" component={MainTabNavigator} />
                  <Drawer.Screen name="Settings" component={SettingsScreen} />
                </Drawer.Navigator>
              </ProtectedNavigator>
            )}
          </Stack.Screen>
        ) : (
          // Unauthenticated user flow
          <Stack.Screen name="Auth" component={AuthNavigator} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};

// components/AuthGuard.js (Higher-order component)
export const withAuthGuard = (Component) => {
  return (props) => {
    const { isAuthenticated, isLoading } = useAuth();
    const navigation = useNavigation();

    useEffect(() => {
      if (!isLoading && !isAuthenticated) {
        Alert.alert(
          'Authentication Required',
          'Please log in to access this feature',
          [
            { text: 'Cancel', style: 'cancel' },
            {
              text: 'Login',
              onPress: () => navigation.navigate('Auth'),
            },
          ]
        );
      }
    }, [isAuthenticated, isLoading, navigation]);

    if (isLoading) {
      return <ActivityIndicator size="large" />;
    }

    if (!isAuthenticated) {
      return null;
    }

    return <Component {...props} />;
  };
};

// USAGE
const ProtectedSettingsScreen = withAuthGuard(SettingsScreen);
```

**Token Refresh Implementation:**
```javascript
// utils/authUtils.js
export const setupTokenRefresh = (logout) => {
  let refreshTimeout;

  const scheduleTokenRefresh = (token) => {
    try {
      // Decode token to get expiry time
      const payload = JSON.parse(atob(token.split('.')[1]));
      const expiryTime = payload.exp * 1000; // Convert to milliseconds
      const refreshTime = expiryTime - Date.now() - (5 * 60 * 1000); // 5 minutes before expiry

      if (refreshTime > 0) {
        refreshTimeout = setTimeout(async () => {
          try {
            const newToken = await authAPI.refreshToken(token);

            // Update stored token
            await SecureStore.setItemAsync('userToken', newToken);

            // Schedule next refresh
            scheduleTokenRefresh(newToken);
          } catch (error) {
            console.error('Token refresh failed:', error);
            logout(); // Force logout on refresh failure
          }
        }, refreshTime);
      } else {
        // Token already expired or expires soon
        logout();
      }
    } catch (error) {
      console.error('Token parsing error:', error);
      logout();
    }
  };

  const clearRefreshTimeout = () => {
    if (refreshTimeout) {
      clearTimeout(refreshTimeout);
      refreshTimeout = undefined;
    }
  };

  return { scheduleTokenRefresh, clearRefreshTimeout };
};
```

---

### 3.4 **Navigation Parameters and Type Safety**

**Question:** *"Implement type-safe navigation parameters with validation and error handling."*

**Perfect Answer Structure:**
```
TYPE-SAFE NAVIGATION SYSTEM:

🎯 TYPE DEFINITIONS:
- Screen parameter interfaces
- Navigation prop types
- Route prop extensions

🎯 PARAMETER VALIDATION:
- Runtime parameter validation
- Default values for optional params
- Error handling for invalid params

🎯 TYPE-SAFE NAVIGATION HOOKS:
- Custom navigation hooks
- Parameter extraction utilities
- Type-safe navigation methods
```

**Complete Type-Safe Navigation Implementation:**
```typescript
// types/navigation.ts
export type RootStackParamList = {
  Auth: NavigatorScreenParams<AuthStackParamList>;
  MainTabs: NavigatorScreenParams<MainTabParamList>;
  Settings: undefined;
  NotFound: undefined;
};

export type AuthStackParamList = {
  Login: undefined;
  Register: { inviteCode?: string };
  ForgotPassword: { email?: string };
};

export type MainTabParamList = {
  HomeTab: NavigatorScreenParams<HomeStackParamList>;
  SearchTab: NavigatorScreenParams<SearchStackParamList>;
  ProfileTab: NavigatorScreenParams<ProfileStackParamList>;
};

export type HomeStackParamList = {
  Home: undefined;
  PostDetail: { postId: string; fromNotification?: boolean };
  UserProfile: { userId: string; showFollowButton?: boolean };
  CreatePost: { draft?: PostDraft };
};

export type SearchStackParamList = {
  Search: undefined;
  SearchResults: { query: string; filters?: SearchFilters };
};

export type ProfileStackParamList = {
  Profile: undefined;
  EditProfile: { field?: 'name' | 'bio' | 'photo' };
  Followers: { userId: string; type: 'followers' | 'following' };
};

// Custom navigation hook with type safety
export const useAppNavigation = () => {
  const navigation = useNavigation<NavigationProp<RootStackParamList>>();

  const navigateToPost = useCallback((postId: string, options?: {
    fromNotification?: boolean;
  }) => {
    navigation.navigate('MainTabs', {
      screen: 'HomeTab',
      params: {
        screen: 'PostDetail',
        params: { postId, ...options },
      },
    });
  }, [navigation]);

  const navigateToUser = useCallback((userId: string, options?: {
    showFollowButton?: boolean;
  }) => {
    navigation.navigate('MainTabs', {
      screen: 'HomeTab',
      params: {
        screen: 'UserProfile',
        params: { userId, ...options },
      },
    });
  }, [navigation]);

  const navigateToSearch = useCallback((query: string, filters?: SearchFilters) => {
    navigation.navigate('MainTabs', {
      screen: 'SearchTab',
      params: {
        screen: 'SearchResults',
        params: { query, filters },
      },
    });
  }, [navigation]);

  return {
    ...navigation,
    navigateToPost,
    navigateToUser,
    navigateToSearch,
  };
};

// Type-safe route hook
export const useRouteParams = <T extends Record<string, any>>() => {
  const route = useRoute<RouteProp<Record<string, T>, string>>();

  // Validate required parameters
  const validateParams = useCallback((required: (keyof T)[]) => {
    const missing = required.filter(key => !(key in (route.params || {})));

    if (missing.length > 0) {
      console.warn(`Missing required route parameters: ${missing.join(', ')}`);
      // Could navigate to error screen or provide defaults
    }
  }, [route.params]);

  return {
    params: (route.params as T) || {} as T,
    validateParams,
  };
};

// Screen implementations with type safety
const PostDetailScreen = () => {
  const { params, validateParams } = useRouteParams<HomeStackParamList['PostDetail']>();
  const navigation = useAppNavigation();

  useEffect(() => {
    validateParams(['postId']);
  }, [validateParams]);

  const { postId, fromNotification } = params;

  // Rest of component logic...
};

// Parameter validation utility
export const createParamValidator = <T extends Record<string, any>>(
  schema: { [K in keyof T]: { required?: boolean; defaultValue?: T[K] } }
) => {
  return (params: Partial<T>): T => {
    const validated: Partial<T> = {};

    for (const [key, config] of Object.entries(schema)) {
      const value = params[key];

      if (value === undefined) {
        if (config.required) {
          throw new Error(`Missing required parameter: ${key}`);
        }
        if (config.defaultValue !== undefined) {
          validated[key] = config.defaultValue;
        }
      } else {
        validated[key] = value;
      }
    }

    return validated as T;
  };
};

// Usage in screen
const postDetailSchema = {
  postId: { required: true },
  fromNotification: { required: false, defaultValue: false },
};

const PostDetailScreen = () => {
  const route = useRoute<RouteProp<HomeStackParamList, 'PostDetail'>>();
  const validator = createParamValidator<HomeStackParamList['PostDetail']>(postDetailSchema);

  let params: HomeStackParamList['PostDetail'];

  try {
    params = validator(route.params || {});
  } catch (error) {
    // Handle validation error
    console.error('Invalid route parameters:', error);
    return <ErrorScreen message="Invalid post parameters" />;
  }

  // Component logic with guaranteed valid params
};
```

**Navigation Error Handling:**
```typescript
// utils/navigationErrorHandler.ts
export const handleNavigationError = (
  error: Error,
  navigation: NavigationProp<any>,
  fallbackRoute?: { name: string; params?: any }
) => {
  console.error('Navigation error:', error);

  // Log to analytics
  analytics.track('navigation_error', {
    error: error.message,
    stack: error.stack,
  });

  // Attempt recovery
  if (fallbackRoute) {
    try {
      navigation.reset({
        index: 0,
        routes: [fallbackRoute],
      });
    } catch (resetError) {
      console.error('Fallback navigation failed:', resetError);
      // Last resort: restart app or show error screen
    }
  }
};

// Enhanced navigation hook with error handling
export const useSafeNavigation = () => {
  const navigation = useNavigation();

  const safeNavigate = useCallback(<T extends object>(
    routeName: string,
    params?: T,
    fallback?: { name: string; params?: any }
  ) => {
    try {
      navigation.navigate(routeName as never, params as never);
    } catch (error) {
      handleNavigationError(error, navigation, fallback);
    }
  }, [navigation]);

  const safeReset = useCallback((
    routes: any[],
    fallback?: { name: string; params?: any }
  ) => {
    try {
      navigation.reset({ index: routes.length - 1, routes });
    } catch (error) {
      handleNavigationError(error, navigation, fallback);
    }
  }, [navigation]);

  return {
    ...navigation,
    navigate: safeNavigate,
    reset: safeReset,
  };
};
```

---

### 3.5 **Scalable App Architecture and Folder Structure**

**Question:** *"Design a scalable folder structure and architecture for a large React Native application."*

**Perfect Answer Structure:**
```
SCALABLE APP ARCHITECTURE:

🎯 FOLDER ORGANIZATION:
- Feature-based structure (not type-based)
- Shared/common code separation
- Clear separation of concerns
- Easy navigation and maintenance

🎯 ARCHITECTURAL PATTERNS:
- Container/Presentational components
- Custom hooks for business logic
- Service layer for API calls
- Type-safe interfaces throughout

🎯 SCALING CONSIDERATIONS:
- Code splitting opportunities
- Testing strategy alignment
- Performance optimization hooks
- Team collaboration efficiency
```

**Complete Scalable Architecture:**
```
📁 src/
├── 📁 assets/                 # Images, fonts, icons
│   ├── 📁 images/
│   ├── 📁 fonts/
│   └── 📁 icons/
├── 📁 components/             # Shared/reusable components
│   ├── 📁 ui/                # Basic UI components (Button, Input, etc.)
│   ├── 📁 layout/            # Layout components (Header, Footer, etc.)
│   └── 📁 common/            # Common components used across features
├── 📁 features/              # Feature-based modules
│   ├── 📁 auth/
│   │   ├── 📁 components/    # Feature-specific components
│   │   ├── 📁 hooks/         # Feature-specific hooks
│   │   ├── 📁 services/      # API calls, data fetching
│   │   ├── 📁 types/         # TypeScript interfaces
│   │   ├── 📁 utils/         # Helper functions
│   │   └── 📁 __tests__/     # Unit and integration tests
│   ├── 📁 home/
│   ├── 📁 profile/
│   └── 📁 settings/
├── 📁 navigation/            # Navigation configuration
│   ├── 📁 stacks/
│   ├── 📁 tabs/
│   └── 📁 types.ts
├── 📁 services/              # Global services
│   ├── 📁 api/              # API client, interceptors
│   ├── 📁 storage/          # AsyncStorage, SecureStore
│   ├── 📁 analytics/        # Analytics, tracking
│   └── 📁 notifications/    # Push notifications
├── 📁 context/               # React contexts
│   ├── 📁 AuthContext/
│   └── 📁 ThemeContext/
├── 📁 hooks/                 # Shared custom hooks
├── 📁 utils/                 # Utility functions
├── 📁 types/                 # Global type definitions
├── 📁 constants/             # App constants, config
├── 📁 styles/                # Global styles, themes
└── 📁 lib/                   # Third-party library configurations
```

**Feature-Based Architecture Implementation:**
```typescript
// features/auth/types/index.ts
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
}

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface AuthContextType extends AuthState {
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  register: (userData: RegisterData) => Promise<void>;
}

// features/auth/services/authAPI.ts
export class AuthAPI {
  static async login(credentials: LoginCredentials): Promise<User> {
    const response = await apiClient.post('/auth/login', credentials);
    return response.data.user;
  }

  static async logout(): Promise<void> {
    await apiClient.post('/auth/logout');
  }

  static async refreshToken(): Promise<string> {
    const response = await apiClient.post('/auth/refresh');
    return response.data.token;
  }

  static async getCurrentUser(): Promise<User> {
    const response = await apiClient.get('/auth/me');
    return response.data.user;
  }
}

// features/auth/hooks/useAuth.ts
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

// features/auth/hooks/useLogin.ts
export const useLogin = () => {
  const { login } = useAuth();
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const loginUser = useCallback(async (credentials: LoginCredentials) => {
    try {
      setIsLoading(true);
      setError(null);
      await login(credentials);
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, [login]);

  return { loginUser, isLoading, error };
};

// features/auth/components/LoginForm.tsx
interface LoginFormProps {
  onSuccess?: () => void;
  onRegisterPress?: () => void;
}

export const LoginForm: React.FC<LoginFormProps> = ({
  onSuccess,
  onRegisterPress
}) => {
  const { loginUser, isLoading, error } = useLogin();
  const [credentials, setCredentials] = useState<LoginCredentials>({
    email: '',
    password: '',
  });

  const handleSubmit = async () => {
    try {
      await loginUser(credentials);
      onSuccess?.();
    } catch (err) {
      // Error handled by hook
    }
  };

  return (
    <View style={styles.container}>
      <TextInput
        placeholder="Email"
        value={credentials.email}
        onChangeText={(email) => setCredentials(prev => ({ ...prev, email }))}
        keyboardType="email-address"
        autoCapitalize="none"
      />
      <TextInput
        placeholder="Password"
        value={credentials.password}
        onChangeText={(password) => setCredentials(prev => ({ ...prev, password }))}
        secureTextEntry
      />

      {error && <Text style={styles.error}>{error}</Text>}

      <Button
        title={isLoading ? "Logging in..." : "Login"}
        onPress={handleSubmit}
        disabled={isLoading}
      />

      <Button
        title="Don't have an account?"
        onPress={onRegisterPress}
        variant="secondary"
      />
    </View>
  );
};

// features/auth/components/index.ts
export { LoginForm } from './LoginForm';
export { RegisterForm } from './RegisterForm';
export { ForgotPasswordForm } from './ForgotPasswordForm';

// features/auth/index.ts
export { useAuth, AuthProvider } from './context/AuthContext';
export { useLogin } from './hooks/useLogin';
export { useRegister } from './hooks/useRegister';
export * from './components';
export * from './types';
```

**Shared Utilities and Services:**
```typescript
// services/api/client.ts
import axios, { AxiosInstance, AxiosResponse } from 'axios';

class APIClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: Config.API_BASE_URL,
      timeout: 10000,
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor for auth tokens
    this.client.interceptors.request.use(
      async (config) => {
        const token = await SecureStore.getItemAsync('authToken');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for token refresh
    this.client.interceptors.response.use(
      (response) => response,
      async (error) => {
        if (error.response?.status === 401) {
          // Token expired, try refresh
          try {
            const newToken = await AuthAPI.refreshToken();
            await SecureStore.setItemAsync('authToken', newToken);

            // Retry original request
            error.config.headers.Authorization = `Bearer ${newToken}`;
            return this.client.request(error.config);
          } catch (refreshError) {
            // Refresh failed, logout user
            await SecureStore.deleteItemAsync('authToken');
            // Navigate to login
          }
        }
        return Promise.reject(error);
      }
    );
  }

  async get<T>(url: string): Promise<T> {
    const response: AxiosResponse<T> = await this.client.get(url);
    return response.data;
  }

  async post<T>(url: string, data?: any): Promise<T> {
    const response: AxiosResponse<T> = await this.client.post(url, data);
    return response.data;
  }

  // Other HTTP methods...
}

export const apiClient = new APIClient();

// utils/index.ts
export { formatDate, formatCurrency, debounce } from './formatters';
export { validateEmail, validatePassword } from './validators';
export { cache, Cache } from './cache';

// hooks/index.ts
export { useAsync } from './useAsync';
export { useDebounce } from './useDebounce';
export { useLocalStorage } from './useLocalStorage';
export { useOnlineStatus } from './useOnlineStatus';
```

---

*Phase 3 continues with advanced navigation patterns, modal flows, navigation performance optimization, and testing strategies for navigation components. Each question includes production-ready code with error handling and performance considerations.*

---

## 🎯 Phase 4 — Performance Optimization & Memory Management

**Performance questions separate senior developers from juniors** - expect deep technical discussions about optimization strategies, memory management, and rendering performance.

### 4.1 **React Rendering Optimization Strategies**

**Question:** *"Design a comprehensive performance optimization strategy for a complex React Native component tree."*

**Perfect Answer Structure:**
```
PERFORMANCE OPTIMIZATION HIERARCHY:

🎯 RENDER OPTIMIZATION:
- React.memo for component memoization
- useMemo for expensive computations
- useCallback for stable function references
- Key optimization for list rendering

🎯 MEMORY MANAGEMENT:
- Proper cleanup in useEffect
- Avoiding memory leaks
- Image and asset optimization
- Component unmounting strategies

🎯 RUNTIME OPTIMIZATION:
- Debouncing user input
- Virtualized lists for large datasets
- Lazy loading components
- Background task management
```

**Complete Optimization Implementation:**
```javascript
// BEFORE: Unoptimized component with performance issues
const UserList = ({ users, onUserSelect, searchQuery, theme }) => {
  console.log('UserList rendering...');

  // Expensive computation on every render
  const filteredUsers = users.filter(user =>
    user.name.toLowerCase().includes(searchQuery.toLowerCase())
  );

  // New function reference on every render
  const handleUserPress = (user) => {
    console.log('User pressed:', user.name);
    onUserSelect(user);
  };

  // Complex style object created on every render
  const containerStyle = {
    backgroundColor: theme === 'dark' ? '#000' : '#fff',
    padding: 16,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 5 ➡️](phase5-native-modules-apis.md)