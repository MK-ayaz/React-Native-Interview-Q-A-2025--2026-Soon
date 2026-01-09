<style>
  .handbook-page {
    background-color: #ffffff;
    color: #1e293b;
    line-height: 1.8;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    max-width: 900px;
    margin: 0 auto;
    padding: 20px;
  }
  .phase-header {
    background: linear-gradient(135deg, #8b5cf6 0%, #6d28d9 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(139, 92, 246, 0.2);
  }
  .phase-number {
    font-weight: 800;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    font-size: 0.875rem;
    opacity: 0.9;
    display: block;
    margin-bottom: 8px;
  }
  .phase-title {
    font-size: 2.5rem;
    font-weight: 800;
    margin: 0;
    line-height: 1.2;
  }
  .qa-card {
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 16px;
    padding: 32px;
    margin-bottom: 32px;
    transition: transform 0.2s ease;
  }
  .question-tag {
    background: #f5f3ff;
    color: #6d28d9;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #f0fdf4;
    border-left: 4px solid #22c55e;
    padding: 20px;
    border-radius: 8px;
    margin: 24px 0;
  }
  .follow-up-trap {
    background: #fef2f2;
    border-left: 4px solid #ef4444;
    padding: 20px;
    border-radius: 8px;
    margin: 24px 0;
  }
  .code-block-header {
    background: #1e293b;
    color: #94a3b8;
    padding: 8px 16px;
    border-top-left-radius: 8px;
    border-top-right-radius: 8px;
    font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
    font-size: 0.75rem;
    display: flex;
    justify-content: space-between;
  }
  .nav-footer {
    display: flex;
    justify-content: space-between;
    margin-top: 60px;
    padding-top: 20px;
    border-top: 1px solid #e2e8f0;
  }
</style>

<div class="handbook-page">

<div class="phase-header">
  <span class="phase-number">Phase 03</span>
  <h1 class="phase-title">Navigation & App Architecture</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Routing, State Management Architecture, and Scalable App Structure</p>
</div>

---

### 📋 Phase Overview
Building on React fundamentals, this phase covers navigation patterns, app architecture, and state management at scale. You'll learn how to structure large React Native applications, implement complex navigation flows, and build maintainable architecture that grows with your app.

---

<div class="qa-card">
  <span class="question-tag">ARCHITECTURE</span>
  <h3>3.1 Scalable Navigation Patterns</h3>
  
  **Question:** *"Design a navigation structure for a large-scale app that includes Auth, Tabs, and Nested Stacks."*

  <div class="senior-insight">
    <strong>Root Navigator Strategy:</strong>
    A senior approach uses a single Root Stack that conditionally renders either an <code>AuthStack</code> or a <code>MainNavigator</code> based on the global authentication state.
  </div>

  <div class="code-block-header">
    <span>navigation/RootNavigator.tsx</span>
    <span>TypeScript</span>
  </div>

  ```tsx
  const RootNavigator = () => {
    const { user, isLoading } = useAuth();
    if (isLoading) return <SplashScreen />;
    return (
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {user ? (
          <Stack.Screen name="Main" component={MainTabNavigator} />
        ) : (
          <Stack.Screen name="Auth" component={AuthStackNavigator} />
        )}
      </Stack.Navigator>
    );
  };
  ```
</div>

<div class="qa-card">
  <span class="question-tag">DEEP LINKING</span>
  <h3>3.2 Deep Linking & Universal Links</h3>
  
  **Question:** *"Explain the difference between Custom URL Schemes and Universal/App Links, and how to handle them."*

  <ul>
    <li><strong>Custom Schemes (<code>myapp://</code>):</strong> Easy to set up but less secure (can be hijacked).</li>
    <li><strong>Universal Links (iOS) / App Links (Android):</strong> Uses standard HTTPS URLs, verified via site association files for high security.</li>
  </ul>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap: Background Links</strong><br/>
    How do you handle a link if the app is already in the background? React Navigation's <code>linking</code> config handles this via <code>subscribe</code>. Cold starts use <code>getInitialURL</code>, while active apps use <code>Linking.addEventListener</code>.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">AUTH</span>
  <h3>3.3 Authentication Flow Implementation</h3>
  
  **Question:** *"What is the best practice for handling login/logout transitions in React Navigation?"*

  <p>Never use <code>navigation.navigate('Home')</code> for login. Instead, update the global Auth state and let the <code>RootNavigator</code> swap stacks. This automatically cleans up the Auth screens from memory and prevents "back to login" bugs.</p>
</div>

<div class="qa-card">
  <span class="question-tag">STATE</span>
  <h3>3.4 Navigation State Persistence</h3>
  
  **Question:** *"Why persist navigation state, and what are the risks in production?"*

  <p>Persistence helps development (reloading on the same screen) but carries risks like stale data or crashes after navigation structure updates. Always use a versioning check for persisted state.</p>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE</span>
  <h3>3.5 Navigation Performance Optimization</h3>
  
  **Question:** *"How do you optimize navigation performance in a large app?"*

  <div class="senior-insight">
    <strong>⚡ Key Tips:</strong>
    <ul>
      <li><strong>React Native Screens:</strong> Use native fragments/view controllers for inactive screens.</li>
      <li><strong>Freeze:</strong> Enable "freeze" to prevent re-renders in background screens.</li>
      <li><strong>Lazy Loading:</strong> Load screens only when they are first accessed.</li>
    </ul>
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">NATIVE</span>
  <h3>3.6 Why use react-native-screens?</h3>
  
  **Question:** *"How does react-native-screens change the way screens are managed at the OS level?"*

  <p>It tells the OS to treat React components as native Fragments (Android) or View Controllers (iOS), significantly reducing memory usage and enabling native animations.</p>
</div>

<div class="qa-card">
  <span class="question-tag">TYPES</span>
  <h3>3.7 Type Safety in Navigation</h3>
  
  **Question:** *"How do you implement strong typing for navigation params in a TypeScript project?"*

  <p>Define a <code>RootStackParamList</code> and use it with <code>StackNavigationProp</code> and <code>RouteProp</code> to ensure params are type-safe across the app.</p>
</div>

<div class="qa-card">
  <span class="question-tag">EVENTS</span>
  <h3>3.8 Navigation Listeners & Events</h3>
  
  **Question:** *"When would you use the 'focus' or 'blur' events in a navigation screen?"*

  <p>Use 'focus' to refresh data when a user returns to a screen, and 'blur' to stop expensive background tasks (like video playback or location tracking).</p>
</div>

<div class="qa-card">
  <span class="question-tag">BUGS</span>
  <h3>3.9 Common Navigation Pitfalls</h3>
  
  **Question:** *"How do you prevent 'double-tap' navigation where the same screen opens twice?"*

  <p>Implement a debounce mechanism or check if the target screen is already at the top of the stack before navigating.</p>
</div>

<div class="qa-card">
  <span class="question-tag">ACTIONS</span>
  <h3>3.10 Navigation Actions & Dispatches</h3>
  
  **Question:** *"What is the difference between navigate() and push() in a Stack Navigator?"*

  <p><code>navigate()</code> will go back to an existing screen if it's already in the stack, while <code>push()</code> always adds a new instance of the screen to the top.</p>
</div>

<div class="qa-card">
  <span class="question-tag">UI</span>
  <h3>3.11 Customizing Transitions</h3>
  
  **Question:** *"How do you implement a custom 'modal' slide-up transition for specific screens?"*

  <p>Use the <code>presentation: 'modal'</code> option or define a custom <code>cardStyleInterpolator</code> in the screen options.</p>
</div>

<div class="qa-card">
  <span class="question-tag">OS</span>
  <h3>3.12 Hardware Back Button (Android)</h3>
  
  **Question:** *"How do you intercept the hardware back button to show an 'Exit App' alert?"*

  <p>Use the <code>BackHandler</code> API combined with the <code>beforeRemove</code> navigation listener to prevent accidental exits.</p>
</div>

<div class="qa-card">
  <span class="question-tag">NOTIFICATIONS</span>
  <h3>3.13 Linking with Push Notifications</h3>
  
  **Question:** *"How do you route a user to a specific screen when they tap a push notification?"*

  <p>Handle the notification in <code>Linking.getInitialURL</code> (cold start) or via the notification library's listener (active app), then use the <code>linking</code> config to map the notification payload to a screen.</p>
</div>

<div class="qa-card">
  <span class="question-tag">HISTORY</span>
  <h3>3.14 Breadcrumbs & Navigation History</h3>
  
  **Question:** *"How do you programmatically reset the navigation history after a specific user action?"*

  <p>Use <code>navigation.reset()</code> to replace the current state with a completely new set of screens, useful for flows like "Order Completion."</p>
</div>

<div class="qa-card">
  <span class="question-tag">TESTING</span>
  <h3>3.15 Testing Navigation Logic</h3>
  
  **Question:** *"How do you unit test a component that uses the useNavigation hook?"*

  <p>Mock the <code>useNavigation</code> hook using <code>jest.mock</code> and verify that the correct navigation methods are called with expected parameters.</p>
</div>

<div class="qa-card">
  <span class="question-tag">BASIC NAVIGATION</span>
  <h3 style="margin-top: 0;">Q16: Basic navigation setup in React Native.</h3>

  <p><strong>React Navigation</strong> is the most popular navigation library. It provides native navigation components:</p>

  <div class="code-example">Basic Navigation Setup</div>
  <pre class="code-block"><code class="language-javascript">// Installation
// npm install @react-navigation/native @react-navigation/stack
// npm install react-native-screens react-native-safe-area-context

import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

function App() {
  return (
    &lt;NavigationContainer&gt;
      &lt;Stack.Navigator initialRouteName="Home"&gt;
        &lt;Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ title: 'Welcome' }}
        /&gt;
        &lt;Stack.Screen
          name="Profile"
          component={ProfileScreen}
          options={{ title: 'My Profile' }}
        /&gt;
      &lt;/Stack.Navigator&gt;
    &lt;/NavigationContainer&gt;
  );
}

// In a screen component
function HomeScreen({ navigation }) {
  return (
    &lt;View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      &lt;Text style={{ fontSize: 24 }}>Home Screen&lt;/Text&gt;
      &lt;TouchableOpacity
        style={{ marginTop: 20, padding: 10, backgroundColor: '#007AFF' }}
        onPress={() => navigation.navigate('Profile')}
      >
        &lt;Text style={{ color: 'white' }}>Go to Profile&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always wrap your app with <code>NavigationContainer</code> at the root. Every screen automatically receives a <code>navigation</code> prop for moving between screens.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">NAVIGATION TYPES</span>
  <h3 style="margin-top: 0;">Q17: Different navigation types and when to use them.</h3>

  <p>React Navigation provides different navigators for different use cases:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin: 24px 0;">
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #1e40af;">📚 Stack Navigator</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Screens stack on top of each other</li>
        <li>Like browser history</li>
        <li>Best for: Detail screens, forms</li>
        <li>Has back button by default</li>
      </ul>
    </div>
    <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #166534;">📱 Tab Navigator</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Bottom tabs for main sections</li>
        <li>Persistent across navigation</li>
        <li>Best for: App main sections</li>
        <li>Icons and labels</li>
      </ul>
    </div>
    <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #92400e;">🗂️ Drawer Navigator</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Side menu slides in</li>
        <li>Good for settings, account screens</li>
        <li>Best for: Secondary navigation</li>
        <li>Customizable content</li>
      </ul>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">🔄 Modal</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Overlays current screen</li>
        <li>For temporary content</li>
        <li>Best for: Confirmations, settings</li>
        <li>Can be dismissed</li>
      </ul>
    </div>
  </div>

  <div class="code-example">Combining Navigators</div>
  <pre class="code-block"><code class="language-javascript">// Tab Navigator inside Stack
const Tab = createBottomTabNavigator();
const Stack = createStackNavigator();

function MainTabNavigator() {
  return (
    &lt;Tab.Navigator&gt;
      &lt;Tab.Screen name="Home" component={HomeStack} /&gt;
      &lt;Tab.Screen name="Settings" component={SettingsStack} /&gt;
    &lt;/Tab.Navigator&gt;
  );
}

function HomeStack() {
  return (
    &lt;Stack.Navigator&gt;
      &lt;Stack.Screen name="Home" component={HomeScreen} /&gt;
      &lt;Stack.Screen name="Details" component={DetailsScreen} /&gt;
    &lt;/Stack.Navigator&gt;
  );
}

function RootStack() {
  return (
    &lt;Stack.Navigator&gt;
      &lt;Stack.Screen
        name="Main"
        component={MainTabNavigator}
        options={{ headerShown: false }}
      /&gt;
      &lt;Stack.Screen name="Modal" component={ModalScreen} /&gt;
    &lt;/Stack.Navigator&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PASSING DATA</span>
  <h3 style="margin-top: 0;">Q18: Passing data between screens.</h3>

  <p>There are several ways to pass data between screens in React Navigation:</p>

  <div class="code-example">Data Passing Methods</div>
  <pre class="code-block"><code class="language-javascript">// Method 1: Route params (recommended)
function HomeScreen({ navigation }) {
  const user = { id: 1, name: 'John' };

  return (
    &lt;TouchableOpacity
      onPress={() => navigation.navigate('Profile', { user })}
    >
      &lt;Text&gt;Go to Profile&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}

function ProfileScreen({ route, navigation }) {
  const { user } = route.params;

  return (
    &lt;View&gt;
      &lt;Text&gt;Profile: {user.name}&lt;/Text&gt;
    &lt;/View&gt;
  );
}

// Method 2: Navigation options (for headers)
function DetailsScreen({ route, navigation }) {
  const { itemId } = route.params;

  useEffect(() => {
    navigation.setOptions({
      title: `Item ${itemId}`,
    });
  }, [navigation, itemId]);

  return &lt;Text&gt;Details for item {itemId}&lt;/Text&gt;;
}

// Method 3: Global state (Context, Redux, Zustand)
const UserContext = React.createContext();

function App() {
  const [currentUser, setCurrentUser] = useState(null);

  return (
    &lt;UserContext.Provider value={{ currentUser, setCurrentUser }}>
      &lt;NavigationContainer&gt;
        &lt;Stack.Navigator&gt;
          &lt;Stack.Screen name="Home" component={HomeScreen} /&gt;
          &lt;Stack.Screen name="Profile" component={ProfileScreen} /&gt;
        &lt;/Stack.Navigator&gt;
      &lt;/NavigationContainer&gt;
    &lt;/UserContext.Provider&gt;
  );
}</code></pre>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> <em>"What's the difference between navigation params and global state?"</em><br/>
    <strong>Answer:</strong> Use params for screen-specific data that doesn't change the app's global state. Use global state for data that affects multiple screens or persists across navigation.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">AUTHENTICATION FLOW</span>
  <h3 style="margin-top: 0;">Q19: Implementing authentication flow with navigation.</h3>

  <p>Authentication flows require conditional navigation based on auth state:</p>

  <div class="code-example">Auth Flow Implementation</div>
  <pre class="code-block"><code class="language-javascript">// Auth context
const AuthContext = React.createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check if user is logged in (async storage, etc.)
    checkAuthStatus().then(user => {
      setUser(user);
      setLoading(false);
    });
  }, []);

  const login = async (email, password) => {
    const user = await loginAPI(email, password);
    setUser(user);
    await AsyncStorage.setItem('user', JSON.stringify(user));
  };

  const logout = async () => {
    await logoutAPI();
    setUser(null);
    await AsyncStorage.removeItem('user');
  };

  return (
    &lt;AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    &lt;/AuthContext.Provider&gt;
  );
}

// Root navigator with conditional rendering
function RootNavigator() {
  const { user, loading } = useContext(AuthContext);

  if (loading) {
    return &lt;SplashScreen /&gt;;
  }

  return (
    &lt;NavigationContainer&gt;
      &lt;Stack.Navigator screenOptions={{ headerShown: false }}>
        {user ? (
          &lt;Stack.Screen name="Main" component={MainTabNavigator} /&gt;
        ) : (
          &lt;Stack.Screen name="Auth" component={AuthStackNavigator} /&gt;
        )}
      &lt;/Stack.Navigator&gt;
    &lt;/NavigationContainer&gt;
  );
}

// Auth stack
function AuthStackNavigator() {
  return (
    &lt;Stack.Navigator&gt;
      &lt;Stack.Screen name="Login" component={LoginScreen} /&gt;
      &lt;Stack.Screen name="Register" component={RegisterScreen} /&gt;
      &lt;Stack.Screen name="ForgotPassword" component={ForgotPasswordScreen} /&gt;
    &lt;/Stack.Navigator&gt;
  );
}

// Main tab navigator
function MainTabNavigator() {
  return (
    &lt;Tab.Navigator&gt;
      &lt;Tab.Screen name="Home" component={HomeScreen} /&gt;
      &lt;Tab.Screen name="Profile" component={ProfileScreen} /&gt;
    &lt;/Tab.Navigator&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always show a loading screen while checking auth status to avoid flash of incorrect content. Store auth tokens securely using AsyncStorage or secure storage libraries.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">DEEP LINKING</span>
  <h3 style="margin-top: 0;">Q20: Implementing deep linking in React Native.</h3>

  <p>Deep linking allows external URLs to open specific screens in your app:</p>

  <div class="code-example">Deep Linking Setup</div>
  <pre class="code-block"><code class="language-javascript">// 1. Configure linking in NavigationContainer
const config = {
  screens: {
    Home: 'home',
    Profile: {
      path: 'user/:userId',
      parse: {
        userId: (userId) => parseInt(userId, 10),
      },
    },
    Settings: 'settings',
  },
};

function App() {
  const linking = {
    prefixes: ['myapp://', 'https://myapp.com'],
    config,
  };

  return (
    &lt;NavigationContainer linking={linking} fallback={&lt;Text&gt;Loading...&lt;/Text&gt;}>
      &lt;Stack.Navigator&gt;
        &lt;Stack.Screen name="Home" component={HomeScreen} /&gt;
        &lt;Stack.Screen name="Profile" component={ProfileScreen} /&gt;
        &lt;Stack.Screen name="Settings" component={SettingsScreen} /&gt;
      &lt;/Stack.Navigator&gt;
    &lt;/NavigationContainer&gt;
  );
}

// 2. Handle deep links programmatically
import { Linking } from 'react-native';

function useDeepLinkHandler() {
  useEffect(() => {
    const handleDeepLink = (event) => {
      const { url } = event;
      // Parse URL and navigate
      if (url.includes('user/')) {
        const userId = url.split('user/')[1];
        navigation.navigate('Profile', { userId: parseInt(userId) });
      }
    };

    // Handle initial URL (app opened from link)
    Linking.getInitialURL().then((url) => {
      if (url) handleDeepLink({ url });
    });

    // Handle URL when app is already open
    const subscription = Linking.addEventListener('url', handleDeepLink);

    return () => subscription?.remove();
  }, []);
}

// 3. Create links in your app
function ShareButton({ itemId }) {
  const shareLink = () => {
    const url = `myapp://item/${itemId}`;
    // Use Share API or copy to clipboard
    Share.share({ message: `Check out this item: ${url}` });
  };

  return (
    &lt;TouchableOpacity onPress={shareLink}>
      &lt;Text&gt;Share&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">NAVIGATION STATE</span>
  <h3 style="margin-top: 0;">Q21: Managing navigation state and persistence.</h3>

  <p>React Navigation can persist and restore navigation state:</p>

  <div class="code-example">Navigation State Persistence</div>
  <pre class="code-block"><code class="language-javascript">// Persist navigation state to AsyncStorage
import AsyncStorage from '@react-native-async-storage/async-storage';

const PERSISTENCE_KEY = 'NAVIGATION_STATE';

function App() {
  const [isReady, setIsReady] = useState(false);
  const [initialState, setInitialState] = useState();

  useEffect(() => {
    const restoreState = async () => {
      try {
        const savedStateString = await AsyncStorage.getItem(PERSISTENCE_KEY);
        const state = savedStateString ? JSON.parse(savedStateString) : undefined;

        if (state !== undefined) {
          setInitialState(state);
        }
      } finally {
        setIsReady(true);
      }
    };

    restoreState();
  }, []);

  if (!isReady) {
    return &lt;SplashScreen /&gt;;
  }

  return (
    &lt;NavigationContainer
      initialState={initialState}
      onStateChange={(state) =>
        AsyncStorage.setItem(PERSISTENCE_KEY, JSON.stringify(state))
      }
    >
      &lt;Stack.Navigator&gt;
        &lt;Stack.Screen name="Home" component={HomeScreen} /&gt;
        &lt;Stack.Screen name="Profile" component={ProfileScreen} /&gt;
      &lt;/Stack.Navigator&gt;
    &lt;/NavigationContainer&gt;
  );
}

// Reset navigation state (useful after login/logout)
function resetNavigationToHome(navigation) {
  navigation.reset({
    index: 0,
    routes: [{ name: 'Home' }],
  });
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Navigation state persistence survives app restarts but can become stale. Always validate that screens still exist when restoring state.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">CUSTOM NAVIGATION</span>
  <h3 style="margin-top: 0;">Q22: Custom navigation transitions and animations.</h3>

  <p>React Navigation allows custom screen transitions:</p>

  <div class="code-example">Custom Transitions</div>
  <pre class="code-block"><code class="language-javascript">// Custom transition spec
const config = {
  animation: 'spring',
  config: {
    stiffness: 1000,
    damping: 500,
    mass: 3,
    overshootClamping: true,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 0.01,
  },
};

// Apply to specific screens
function StackNavigator() {
  return (
    &lt;Stack.Navigator
      screenOptions={{
        transitionSpec: {
          open: config,
          close: config,
        },
        cardStyleInterpolator: CardStyleInterpolators.forHorizontalIOS,
      }}
    >
      &lt;Stack.Screen
        name="Home"
        component={HomeScreen}
        options={{
          transitionSpec: {
            open: TransitionSpecs.ScaleFromCenterAndroid,
            close: TransitionSpecs.ScaleFromCenterAndroid,
          },
        }}
      /&gt;
      &lt;Stack.Screen name="Modal" component={ModalScreen} /&gt;
    &lt;/Stack.Navigator&gt;
  );
}

// Custom header with animations
function CustomHeader() {
  const progress = useHeaderHeight();

  return (
    &lt;Animated.View
      style={{
        height: progress.interpolate({
          inputRange: [0, 1],
          outputRange: [100, 50],
        }),
        backgroundColor: progress.interpolate({
          inputRange: [0, 1],
          outputRange: ['#ff0000', '#0000ff'],
        }),
      }}
    >
      &lt;Text&gt;Custom Header&lt;/Text&gt;
    &lt;/Animated.View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">APP ARCHITECTURE</span>
  <h3 style="margin-top: 0;">Q23: Scalable app architecture patterns.</h3>

  <p>Large React Native apps need careful architecture planning:</p>

  <div class="code-example">Feature-Based Architecture</div>
  <pre class="code-block"><code class="language-javascript">// 📁 src/
├── features/           # Feature-based organization
│   ├── auth/
│   │   ├── components/
│   │   ├── screens/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types.ts
│   ├── products/
│   │   ├── components/
│   │   ├── screens/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types.ts
│   └── settings/
├── shared/            # Shared utilities
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── constants/
├── navigation/
│   ├── navigators/
│   └── types.ts
└── store/            # Global state
    ├── slices/
    └── store.ts

// Feature folder structure
// 📁 features/auth/
├── index.ts          # Public API exports
├── components/
│   ├── LoginForm.tsx
│   └── RegisterForm.tsx
├── screens/
│   ├── LoginScreen.tsx
│   └── RegisterScreen.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useLogin.ts
├── services/
│   ├── authAPI.ts
│   └── tokenStorage.ts
└── types.ts

// Barrel exports for clean imports
// features/auth/index.ts
export { LoginScreen } from './screens/LoginScreen';
export { RegisterScreen } from './screens/RegisterScreen';
export { useAuth } from './hooks/useAuth';
export { login, logout } from './services/authAPI';

// Usage
import { LoginScreen, useAuth } from '@/features/auth';</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">STATE MANAGEMENT</span>
  <h3 style="margin-top: 0;">Q24: Choosing the right state management solution.</h3>

  <p>Different state management tools for different needs:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Solution</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Best For</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Complexity</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Bundle Size</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>useState/useReducer</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Component state, simple global state</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Low</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">None</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Context API</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Medium apps, theme/user settings</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Medium</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Small</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Zustand</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Most apps, great DX, scalable</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Low</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Small</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><strong>Redux Toolkit</strong></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Large teams, complex async logic</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Medium-High</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Medium</td>
    </tr>
    <tr>
      <td style="padding: 12px;"><strong>Recoil/MobX</strong></td>
      <td style="padding: 12px;">Complex state relationships</td>
      <td style="padding: 12px;">High</td>
      <td style="padding: 12px;">Large</td>
    </tr>
  </table>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE & OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q25: Navigation performance optimization techniques.</h3>

  <p>Optimize navigation for better user experience:</p>

  <div class="code-example">Navigation Performance Tips</div>
  <pre class="code-block"><code class="language-javascript">// 1. Lazy loading screens
const LazyProfileScreen = lazy(() => import('./ProfileScreen'));

function ProfileStack() {
  return (
    &lt;Stack.Navigator&gt;
      &lt;Stack.Screen name="Profile" component={LazyProfileScreen} /&gt;
    &lt;/Stack.Navigator&gt;
  );
}

// 2. Preloading important screens
function App() {
  useEffect(() => {
    // Preload heavy screens in background
    const preloadScreens = async () => {
      await Promise.all([
        import('./screens/HeavyScreen1'),
        import('./screens/HeavyScreen2'),
      ]);
    };

    preloadScreens();
  }, []);

  return &lt;NavigationContainer&gt;...&lt;/NavigationContainer&gt;;
}

// 3. Optimize screen options
function OptimizedScreen() {
  return (
    &lt;Stack.Screen
      name="Home"
      component={HomeScreen}
      options={{
        headerShown: false, // Remove header if not needed
        gestureEnabled: false, // Disable swipe gestures if not needed
        animationEnabled: false, // Disable animations for fast screens
      }}
    /&gt;
  );
}

// 4. Use react-native-screens for better performance
// This is enabled by default in React Navigation v6+

// 5. Memoize navigation options
const screenOptions = useMemo(() => ({
  headerStyle: { backgroundColor: theme.colors.primary },
  headerTintColor: theme.colors.text,
}), [theme]);

function StackNavigator() {
  return (
    &lt;Stack.Navigator screenOptions={screenOptions}>
      &lt;Stack.Screen name="Home" component={HomeScreen} /&gt;
    &lt;/Stack.Navigator&gt;
  );
}</code></pre>
</div>

---

<div class="nav-footer">
  <a href="phase2-react-fundamentals.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 02: React Mastery</a>
  <a href="phase4-performance-memory.md" style="color: #3b82f6; text-decoration: none; font-weight: 600;">Phase 04: Performance ➡️</a>
</div>

</div>
