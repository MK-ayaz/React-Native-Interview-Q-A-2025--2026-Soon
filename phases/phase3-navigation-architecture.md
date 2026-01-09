[⬅️ Back to Home](../README.md)

# 🎯 Phase 03: Navigation & App Architecture

<div align="center">
  <img src="https://img.shields.io/badge/Level-Senior-red?style=for-the-badge" alt="Senior Level" />
  <img src="https://img.shields.io/badge/Status-Complete-green?style=for-the-badge" alt="Status Complete" />
</div>

---

## 📋 Table of Contents
1. [Stack vs Tab vs Drawer](#31-react-navigation-stack-vs-tab-vs-drawer-navigator)
2. [Deep Linking Strategy](#32-deep-linking-and-url-routing-strategy)
3. [Authentication Flows](#33-implementing-authentication-flows)
4. [Navigation State Persistence](#34-navigation-state-persistence)
5. [Performance Optimization](#35-navigation-performance-optimization)

---

> [!IMPORTANT]
> **Navigation questions are extremely common** - expect 2-3 questions in senior interviews. Focus on React Navigation v6+ and scalable architecture patterns.

---

### 3.1 React Navigation Stack vs Tab vs Drawer Navigator

**Question:** *"Design a complete navigation structure for a social media app with authentication flows."*

#### 🏗️ Architecture Design
A professional app usually combines multiple navigators:
- **Stack Navigator**: For linear flows (e.g., Auth flow, Settings sub-pages).
- **Tab Navigator**: For the main app features (e.g., Home, Search, Profile).
- **Drawer Navigator**: For secondary features or global settings.

#### 🚀 Senior Insight: Nesting vs Flat
Avoid deep nesting of navigators as it complicates state management and can impact performance. Prefer a flatter structure where possible, using a root stack to switch between Auth and Main flows.

#### 💻 Complete Implementation
```javascript
// navigation/AppNavigator.js
const AppNavigator = () => {
  const { isAuthenticated } = useAuth();

  return (
    <NavigationContainer>
      {isAuthenticated ? (
        <MainStack.Navigator>
          <MainStack.Screen name="MainTabs" component={MainTabNavigator} />
          <MainStack.Screen name="Settings" component={SettingsScreen} />
        </MainStack.Navigator>
      ) : (
        <AuthStack.Navigator>
          <AuthStack.Screen name="Login" component={LoginScreen} />
          <AuthStack.Screen name="Register" component={RegisterScreen} />
        </AuthStack.Navigator>
      )}
    </NavigationContainer>
  );
};
```

---

### 3.2 Deep Linking and URL Routing Strategy

**Question:** *"Implement deep linking for a content-sharing app with push notification handling."*

#### 🎯 Linking Configuration
Deep linking allows users to open specific screens via a URL (e.g., `myapp://post/123`).

```javascript
const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      MainTabs: {
        screens: {
          HomeTab: {
            screens: {
              PostDetail: 'post/:postId',
            },
          },
        },
      },
    },
  },
};
```

#### 🪤 Follow-up Trap
> *"How do you handle deep links when the app is already open vs when it's closed?"*
>
> **Answer:** React Navigation handles this via the `linking` prop. If the app is closed, `getInitialURL` is used. If open, `addEventListener` catches the URL.

---

### 3.3 Implementing Authentication Flows

**Question:** *"What is the recommended way to handle authentication-based navigation in React Navigation?"*

#### ✅ The Best Practice
Use conditional rendering of navigators based on the authentication state. Do **not** manually navigate to a "Home" screen after login; instead, let the state change swap the entire navigator. This prevents users from going "back" to the login screen after they've already logged in.

---

### 3.4 Navigation State Persistence

**Question:** *"Why would you persist navigation state, and how do you implement it?"*

#### 🧠 Persistence Strategy
Persisting state allows the app to return to the exact same screen after a crash or a reload during development.

```javascript
const [isReady, setIsReady] = useState(false);
const [initialState, setInitialState] = useState();

useEffect(() => {
  const restoreState = async () => {
    const savedStateString = await AsyncStorage.getItem('NAV_STATE');
    const state = savedStateString ? JSON.parse(savedStateString) : undefined;
    setInitialState(state);
    setIsReady(true);
  };
  restoreState();
}, []);
```

---

### 3.5 Navigation Performance Optimization

**Question:** *"How do you optimize navigation performance in a large app?"*

#### ⚡ Optimization Tips
1. **Freeze Screens**: Use `react-native-screens` to enable native view management.
2. **Lazy Loading**: Use `detachInactiveScreens` (enabled by default in modern RN).
3. **Optimize Headers**: Avoid complex components in headers that re-render frequently.
4. **Memory Management**: Use `useMemo` for navigation options that depend on state.

---

### 🚀 Next Steps
*Phase 3 has covered the structural backbone of your app. In Phase 4, we dive into Performance Optimization and Memory Management.*

---

[⬅️ Back to Home](../README.md) | [Next Phase: Phase 04 ➡️](phase4-performance-memory.md)
