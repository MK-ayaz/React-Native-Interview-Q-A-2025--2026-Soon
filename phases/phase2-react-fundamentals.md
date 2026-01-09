[⬅️ Back to Home](../README.md)

# 🎯 Phase 02: React Fundamentals & Hooks Mastery

<div align="center">
  <img src="https://img.shields.io/badge/Level-Senior-red?style=for-the-badge" alt="Senior Level" />
  <img src="https://img.shields.io/badge/Status-Complete-green?style=for-the-badge" alt="Status Complete" />
</div>

---

## 📋 Table of Contents
1. [Lifecycle Mapping](#21-react-hooks-lifecycle-mapping)
2. [useState vs useReducer](#22-usestate-vs-usereducer-deep-comparison)
3. [useEffect Dependency Mastery](#23-useeffect-dependency-array-mastery)
4. [useMemo vs useCallback](#24-usememo-vs-usecallback-optimization-strategy)
5. [Custom Hooks Design](#25-custom-hooks-design-patterns)
6. [Controlled vs Uncontrolled](#26-controlled-vs-uncontrolled-components-in-react-native)
7. [Context API vs Redux](#27-react-context-api-vs-redux-comparison)
8. [Re-rendering Rules](#28-component-re-rendering-rules-and-optimization)

---

> [!IMPORTANT]
> **React Native interviews heavily test React knowledge** - even senior roles expect deep understanding of hooks, components, and performance patterns. You must be able to explain the "why" behind every optimization.

---

### 2.1 React Hooks Lifecycle Mapping

**Question:** *"Map React class lifecycle methods to their functional equivalents using hooks."*

#### 🎯 Lifecycle Mapping Matrix

| Class Lifecycle | Functional Hook Equivalent |
| :--- | :--- |
| `constructor` | `useState` initializers |
| `componentDidMount` | `useEffect(() => {...}, [])` |
| `componentDidUpdate` | `useEffect(() => {...}, [deps])` |
| `shouldComponentUpdate` | `React.memo` + `useMemo`/`useCallback` |
| `componentWillUnmount` | `useEffect` return cleanup function |
| `componentDidCatch` | **No Hook equivalent** (Use Class Error Boundary) |

#### 💻 Complete Mapping Example
```javascript
// FUNCTIONAL COMPONENT (Modern Standard)
const MyComponent = () => {
  const [count, setCount] = useState(0);

  // componentDidMount + componentWillUnmount
  useEffect(() => {
    console.log('Mounted');
    return () => console.log('Unmounted');
  }, []);

  // componentDidUpdate (filtered by count)
  useEffect(() => {
    console.log('Count changed');
  }, [count]);

  return <Text>{count}</Text>;
};
```

#### 🪤 Follow-up Trap
> *"Can you replicate getDerivedStateFromProps with hooks?"*
>
> **Answer:** There is no direct equivalent. While you can use `useEffect` with props as a dependency, it's generally considered an anti-pattern. Most use cases for `getDerivedStateFromProps` can be handled by simply calculating values during render or using memoization.

---

### 2.2 useState vs useReducer Deep Comparison

**Question:** *"When should you use useReducer instead of useState? Provide real-world examples."*

#### 💡 Decision Matrix
- **Use `useState` for**: Primitive values, simple state transitions, or independent state updates.
- **Use `useReducer` for**: Complex state objects with multiple sub-values, state transitions that depend on previous state, or related state updates that should be atomic.

#### 🚀 Senior Insight: Atomic Operations
Using multiple `useState` calls for related data (e.g., `loading`, `error`, `data`) can lead to "impossible states" where `loading` is true but `data` is also present. `useReducer` ensures all related state changes happen in a single, atomic operation.

#### 💻 Real-World Example: API Fetching
```javascript
// ✅ GOOD: useReducer for atomic operations
const [state, dispatch] = useReducer(dataReducer, {
  loading: false,
  error: null,
  data: null,
});

function dataReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { loading: false, error: null, data: action.payload };
    case 'FETCH_ERROR':
      return { loading: false, error: action.payload, data: null };
    default:
      return state;
  }
}
```

---

### 2.3 useEffect Dependency Array Mastery

**Question:** *"Explain the common pitfalls with the useEffect dependency array and how to solve them."*

#### ⚠️ Common Pitfalls
1. **Missing Dependencies**: Leads to stale closures where the effect uses old values.
2. **Unstable References**: Passing a new object or function in the array on every render causes an infinite loop.
3. **Over-fetching**: Including values that don't actually need to trigger the effect.

#### ✅ The Solution: Memoization
```javascript
// ❌ Problem: onData is recreated every render, triggering useEffect
const onData = (data) => setState(data);
useEffect(() => {
  api.subscribe(onData);
}, [onData]);

// ✅ Solution: useCallback ensures a stable reference
const onData = useCallback((data) => setState(data), []);
useEffect(() => {
  api.subscribe(onData);
}, [onData]);
```

---

### 2.4 useMemo vs useCallback Optimization Strategy

**Question:** *"Design an optimization strategy for a complex component tree."*

#### 🛠️ The Strategy
1. **Memoize Props**: Use `useCallback` for functions and `useMemo` for objects/arrays passed to children.
2. **Memoize Children**: Wrap expensive child components in `React.memo`.
3. **Avoid Inline Objects**: Never define styles or configurations inside the render body unless they are memoized.

```javascript
const UserList = React.memo(({ users, onSelect }) => {
  const filteredUsers = useMemo(() => {
    return users.filter(u => u.isActive);
  }, [users]);

  const handleSelect = useCallback((user) => {
    onSelect(user);
  }, [onSelect]);

  return <FlatList data={filteredUsers} renderItem={...} />;
});
```

---

### 2.5 Custom Hooks Design Patterns

**Question:** *"What are the benefits of custom hooks and how do you design them for testability?"*

#### ✅ Benefits
- **Logic Extraction**: Keep components lean and focused on UI.
- **Reusability**: Share complex logic (e.g., auth, networking) across multiple components.
- **Testability**: Custom hooks can be tested in isolation using `@testing-library/react-hooks`.

---

### 2.6 Controlled vs Uncontrolled Components in React Native

**Question:** *"Explain the trade-offs between controlled and uncontrolled inputs in React Native."*

- **Controlled**: `value` and `onChangeText` are synced with React state. Better for validation but can cause lag on low-end devices during rapid typing.
- **Uncontrolled**: Use `ref` to pull values when needed. Better performance but harder to implement real-time validation.

---

### 2.7 React Context API vs Redux Comparison

**Question:** *"When is Context API enough, and when do you need Redux/Zustand?"*

- **Context API**: Best for low-frequency updates like Themes, User Auth, or Localization.
- **Redux/Zustand**: Best for high-frequency updates, complex state transitions, and large-scale applications with shared data across many screens.

---

### 2.8 Component Re-rendering Rules and Optimization

**Question:** *"Explain React's re-rendering rules and how to prevent unnecessary work in React Native."*

#### 🏗️ Re-rendering Rules
1. A component re-renders if its **State** changes.
2. A component re-renders if its **Props** change.
3. A component re-renders if its **Parent** re-renders (unless wrapped in `React.memo`).
4. A component re-renders if a **Context** it consumes changes.

---

### 🚀 Next Steps
*Phase 2 has mastered the React layer. In Phase 3, we explore Navigation Architecture and Deep Linking.*

---

[⬅️ Back to Home](../README.md) | [Next Phase: Phase 03 ➡️](phase3-navigation-architecture.md)
