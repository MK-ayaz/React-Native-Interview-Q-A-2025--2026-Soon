# Phase 4: Performance & Memory Management ⚡

Welcome to **Phase 4**. At the Senior level, knowing how to build a feature is expected; knowing how to make it scale and perform under pressure is what defines your expertise. This phase covers advanced architecture, performance optimization, and memory management.

---

## 📋 Table of Contents
1. [Scalable Architecture](#1-scalable-architecture)
2. [Type-Safe Navigation](#2-type-safe-navigation)
3. [Authentication Flows](#3-authentication-flows)
4. [Performance Optimization](#4-performance-optimization)
5. [Memory Management](#5-memory-management)
6. [Error Handling & Stability](#6-error-handling--stability)

---

## 1. Scalable Architecture
For large-scale applications, a "feature-based" folder structure is superior to a "type-based" one.

> [!TIP]
> **Pro Tip:** Grouping by feature reduces cognitive load and makes it easier for multiple teams to work on the same codebase without constant merge conflicts.

### Recommended Structure
```text
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── index.ts (Public API)
│   ├── profile/
├── shared/ (Common UI, utils, constants)
├── navigation/
└── store/
```

---

## 2. Type-Safe Navigation
Stop using `any` for your navigation props. Senior engineers implement full type safety for navigation parameters.

### Implementation Example
```typescript
// types/navigation.ts
export type RootStackParamList = {
  Auth: undefined;
  MainTabs: undefined;
  ProductDetails: { id: string; name: string };
};

// Use in Component
const navigation = useNavigation<StackNavigationProp<RootStackParamList>>();
```

---

## 3. Authentication Flows
A common mistake is manually checking `token ? <Home /> : <Login />` in every screen. Use a **Switch-like** pattern at the root navigator level.

### Auth Pattern
```tsx
const RootNavigator = () => {
  const { isLoading, userToken } = useAuth();

  if (isLoading) return <SplashScreen />;

  return (
    <Stack.Navigator>
      {userToken == null ? (
        <Stack.Screen name="SignIn" component={SignInScreen} />
      ) : (
        <Stack.Screen name="Main" component={MainNavigator} />
      )}
    </Stack.Navigator>
  );
};
```

---

## 4. Performance Optimization
React Native performance is often gated by the Bridge (in older architecture) or the JS thread's ability to stay at 60fps.

| Technique | When to Use | Why? |
| :--- | :--- | :--- |
| **FlashList** | Large Lists | Reuses cells instead of destroying them (by Shopify). |
| **useMemo/useCallback** | Expensive computations | Prevents unnecessary re-renders of children. |
| **InteractionManager** | After transitions | Defers heavy JS tasks until animations finish. |
| **Image Caching** | Remote assets | Reduces bandwidth and UI flicker (use `react-native-fast-image`). |

---

## 5. Memory Management
Memory leaks in React Native usually stem from three places:
1. **Uncleaned Event Listeners:** Always remove `addListener` in the cleanup function of `useEffect`.
2. **Closures in Timers:** `setTimeout` or `setInterval` keeping references to state after unmount.
3. **Console Logs:** Large logs in production can consume significant memory and slow down the JS thread.

> [!WARNING]
> **Warning:** Never leave large `console.log` statements in production. Use a logger like `react-native-logs` that can be disabled in release builds.

---

## 6. Error Handling & Stability
Don't let the "Red Screen of Death" reach your users. Implement Global Error Boundaries.

```tsx
class GlobalErrorBoundary extends React.Component {
  componentDidCatch(error, info) {
    logErrorToService(error, info); // e.g., Sentry
  }

  render() {
    if (this.state.hasError) return <FallbackUI />;
    return this.props.children;
  }
}
```

---

## 🏁 Next Steps
Now that you've mastered the JS-side performance, it's time to go deeper. In **Phase 5**, we explore the native side: Bridge, JSI, and Custom Native Modules.

[Back to Main README](../README.md) | [Go to Phase 5: Native Modules](./phase5-native-modules-apis.md)
