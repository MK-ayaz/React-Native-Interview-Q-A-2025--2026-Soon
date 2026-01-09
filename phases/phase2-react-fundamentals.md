[⬅️ Back to Home](../README.md)

# 🎯 Phase 02: React Fundamentals & Hooks Mastery

---


**React Native interviews heavily test React knowledge** - even senior roles expect deep understanding of hooks, components, and performance patterns.

### 2.1 **React Hooks Lifecycle Mapping**

**Question:** *"Map React class lifecycle methods to their functional equivalents using hooks."*

**Perfect Answer Structure:**
```
CLASS LIFECYCLE → HOOKS MAPPING:

🎯 MOUNTING PHASE:
- constructor → useState initializers
- componentDidMount → useEffect(() => {...}, [])

🎯 UPDATING PHASE:
- componentDidUpdate → useEffect(() => {...}, [deps])
- shouldComponentUpdate → React.memo + useMemo/useCallback

🎯 UNMOUNTING PHASE:
- componentWillUnmount → useEffect return function

🎯 ERROR HANDLING:
- componentDidCatch → Error boundaries (class only)
```

**Complete Mapping Example:**
```javascript
// CLASS COMPONENT LIFECYCLE
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  componentDidMount() {
    console.log('Mounted');
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.count !== this.state.count) {
      console.log('Count changed');
    }
  }

  componentWillUnmount() {
    console.log('Unmounted');
  }

  render() { return <Text>{this.state.count}</Text>; }
}

// FUNCTIONAL COMPONENT EQUIVALENT
const MyComponent = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('Mounted');
    return () => console.log('Unmounted');
  }, []);

  useEffect(() => {
    console.log('Count changed');
  }, [count]);

  return <Text>{count}</Text>;
};
```

**Follow-up Trap:** *"Can you replicate getDerivedStateFromProps with hooks?"*

**Answer:** No direct equivalent. Use useEffect with props dependency, but it's generally discouraged.

---

### 2.2 **useState vs useReducer Deep Comparison**

**Question:** *"When should you use useReducer instead of useState? Provide real-world examples."*

**Perfect Answer Structure:**
```
useState vs useReducer Decision Matrix:

🎯 useState FOR:
- Primitive values (strings, numbers, booleans)
- Simple state transitions
- Independent state updates
- Performance-critical simple updates

🎯 useReducer FOR:
- Complex state logic with multiple sub-values
- State transitions depend on current state
- Related state updates (atomic operations)
- Complex state machines
```

**Real-World Examples:**
```javascript
// ❌ BAD: Multiple useState calls for related data
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
const [data, setData] = useState(null);

// Manual state synchronization issues
const fetchData = async () => {
  setLoading(true);
  setError(null);
  try {
    const result = await api.getData();
    setData(result);
  } catch (err) {
    setError(err);
  } finally {
    setLoading(false);
  }
};

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

**Performance Impact:**
- useState: Better for simple cases (less overhead)
- useReducer: Better for complex logic (predictable updates)

---

### 2.3 **useEffect Dependency Array Mastery**

**Question:** *"Explain the four types of useEffect dependency arrays and their use cases."*

**Perfect Answer Structure:**
```
FOUR DEPENDENCY ARRAY PATTERNS:

1️⃣ NO DEPENDENCY ARRAY:
   useEffect(() => {...}) // Runs after every render
   ❌ Use sparingly - causes infinite loops

2️⃣ EMPTY DEPENDENCY ARRAY:
   useEffect(() => {...}, []) // Runs once on mount
   ✅ Perfect for initialization, event listeners

3️⃣ SPECIFIC DEPENDENCIES:
   useEffect(() => {...}, [dep1, dep2]) // Runs when deps change
   ✅ Most common pattern for reactive effects

4️⃣ FUNCTION DEPENDENCIES:
   useEffect(() => {...}, [callback]) // Careful with functions!
   ⚠️ Usually needs useCallback to prevent loops
```

**Common Pitfalls & Solutions:**
```javascript
// ❌ INFINITE LOOP: Missing dependencies
useEffect(() => {
  setCount(count + 1); // Uses count but not in deps
}); // Infinite loop!

// ❌ INFINITE LOOP: Function dependency without memoization
useEffect(() => {
  api.subscribe(onData); // onData recreated every render
}, [onData]); // onData changes every render

// ✅ CORRECT: Memoized callback
const onData = useCallback((data) => {
  setState(data);
}, []);

useEffect(() => {
  api.subscribe(onData);
}, [onData]); // Stable reference
```

**Follow-up Trap:** *"Should you always include all variables used in useEffect in the dependency array?"*
**Answer:** Yes, unless they're guaranteed to be stable (like setState functions from useState).

---

### 2.4 **useMemo vs useCallback Optimization Strategy**

**Question:** *"Design an optimization strategy using useMemo and useCallback for a complex component."*

**Perfect Answer Structure:**
```
OPTIMIZATION HIERARCHY:

🎯 useCallback: Memoizes function references
🎯 useMemo: Memoizes expensive computations
🎯 React.memo: Prevents unnecessary re-renders

STRATEGY:
1. Identify expensive operations
2. Memoize functions passed as props
3. Memoize computed values
4. Memoize components when appropriate
```

**Complete Optimization Example:**
```javascript
// BEFORE: Unoptimized component
const UserList = ({ users, onUserSelect, filter }) => {
  const filteredUsers = users.filter(user =>
    user.name.toLowerCase().includes(filter.toLowerCase())
  );

  const handleSelect = (user) => {
    console.log('User selected:', user);
    onUserSelect(user);
  };

  return (
    <FlatList
      data={filteredUsers}
      renderItem={({ item }) => (
        <TouchableOpacity onPress={() => handleSelect(item)}>
          <Text>{item.name}</Text>
        </TouchableOpacity>
      )}
    />
  );
};

// AFTER: Fully optimized
const UserList = React.memo(({ users, onUserSelect, filter }) => {
  const filteredUsers = useMemo(() => {
    console.log('Filtering users...'); // Expensive operation
    return users.filter(user =>
      user.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [users, filter]);

  const handleSelect = useCallback((user) => {
    console.log('User selected:', user);
    onUserSelect(user);
  }, [onUserSelect]);

  const renderItem = useCallback(({ item }) => (
    <TouchableOpacity onPress={() => handleSelect(item)}>
      <Text>{item.name}</Text>
    </TouchableOpacity>
  ), [handleSelect]);

  return (
    <FlatList
      data={filteredUsers}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
    />
  );
});
```

**Performance Metrics:**
- Filtering: Only runs when users/filter change
- Functions: Stable references prevent child re-renders
- Component: Only re-renders when props actually change

---

### 2.5 **Custom Hooks Design Patterns**

**Question:** *"Design a custom hook for API data fetching with loading, error, and caching states."*

**Perfect Answer Structure:**
```
CUSTOM HOOK DESIGN PRINCIPLES:

🎯 Single Responsibility: One hook, one purpose
🎯 Reusable Logic: Extract common patterns
🎯 Error Boundaries: Handle errors gracefully
🎯 Performance: Include memoization
🎯 Testing: Easy to test in isolation

IMPLEMENTATION:
- Loading states
- Error handling
- Data caching
- Request cancellation
- Retry logic
```

**Complete Custom Hook Implementation:**
```javascript
const useApiData = (url, options = {}) => {
  const [state, dispatch] = useReducer(apiReducer, {
    data: null,
    loading: false,
    error: null,
    cache: new Map(),
  });

  const { enabled = true, retryCount = 3 } = options;

  useEffect(() => {
    if (!enabled || !url) return;

    // Check cache first
    if (state.cache.has(url)) {
      dispatch({ type: 'CACHE_HIT', payload: state.cache.get(url) });
      return;
    }

    let isCancelled = false;

    const fetchData = async (attemptsLeft = retryCount) => {
      dispatch({ type: 'FETCH_START' });

      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);

        const data = await response.json();

        if (!isCancelled) {
          dispatch({ type: 'FETCH_SUCCESS', payload: { data, url } });
        }
      } catch (error) {
        if (attemptsLeft > 0 && !isCancelled) {
          // Exponential backoff retry
          setTimeout(() => fetchData(attemptsLeft - 1), Math.pow(2, retryCount - attemptsLeft) * 1000);
        } else if (!isCancelled) {
          dispatch({ type: 'FETCH_ERROR', payload: error.message });
        }
      }
    };

    fetchData();

    return () => {
      isCancelled = true;
    };
  }, [url, enabled, retryCount]);

  return {
    data: state.data,
    loading: state.loading,
    error: state.error,
    refetch: () => dispatch({ type: 'REFETCH' }),
  };
};

// Reducer for complex state management
function apiReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      const newCache = new Map(state.cache);
      newCache.set(action.payload.url, action.payload.data);
      return {
        ...state,
        loading: false,
        data: action.payload.data,
        cache: newCache,
      };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
    case 'CACHE_HIT':
      return { ...state, loading: false, data: action.payload, error: null };
    case 'REFETCH':
      return { ...state, loading: true, error: null };
    default:
      return state;
  }
}

// USAGE EXAMPLE
const UserProfile = ({ userId }) => {
  const { data: user, loading, error, refetch } = useApiData(
    `https://api.example.com/users/${userId}`,
    { retryCount: 2 }
  );

  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;

  return (
    <View>
      <Text>{user?.name}</Text>
      <Button title="Refresh" onPress={refetch} />
    </View>
  );
};
```

**Testing Strategy:**
```javascript
// __tests__/useApiData.test.js
import { renderHook, waitFor } from '@testing-library/react-native';
import { useApiData } from '../hooks/useApiData';

// Mock fetch
global.fetch = jest.fn();

test('handles successful data fetching', async () => {
  fetch.mockResolvedValueOnce({
    ok: true,
    json: () => Promise.resolve({ id: 1, name: 'John' }),
  });

  const { result } = renderHook(() => useApiData('/api/user'));

  expect(result.current.loading).toBe(true);

  await waitFor(() => {
    expect(result.current.loading).toBe(false);
    expect(result.current.data).toEqual({ id: 1, name: 'John' });
  });
});
```

---

### 2.6 **Controlled vs Uncontrolled Components in React Native**

**Question:** *"Explain controlled vs uncontrolled components with React Native form examples."*

**Perfect Answer Structure:**
```
CONTROLLED COMPONENTS:
- State managed by React
- Predictable, testable
- Immediate validation possible
- Performance overhead for complex forms

UNCONTROLLED COMPONENTS:
- State managed by DOM/native components
- Better performance
- Harder to validate
- Use refs for access
```

**React Native Implementation:**
```javascript
// CONTROLLED COMPONENT
const ControlledInput = () => {
  const [value, setValue] = useState('');

  return (
    <TextInput
      value={value}
      onChangeText={setValue}
      placeholder="Controlled input"
    />
  );
};

// UNCONTROLLED COMPONENT
const UncontrolledInput = () => {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    const value = inputRef.current?.getNativeTextInput()?.text;
    console.log('Input value:', value);
  };

  return (
    <View>
      <TextInput
        ref={inputRef}
        placeholder="Uncontrolled input"
        defaultValue="initial value"
      />
      <Button title="Submit" onPress={handleSubmit} />
    </View>
  );
};

// HYBRID APPROACH (Recommended)
const SmartInput = () => {
  const [value, setValue] = useState('');
  const inputRef = useRef(null);

  // Controlled for validation
  const handleChangeText = (text) => {
    setValue(text);
    // Real-time validation
    if (text.length < 3) {
      // Show error
    }
  };

  // Uncontrolled for performance
  const handleSubmit = () => {
    // Direct native access if needed
    inputRef.current?.blur();
  };

  return (
    <TextInput
      ref={inputRef}
      value={value}
      onChangeText={handleChangeText}
      onSubmitEditing={handleSubmit}
    />
  );
};
```

**When to Use Each:**
```javascript
// Use CONTROLLED for:
- Form validation
- Conditional rendering
- Data transformation
- Testing requirements

// Use UNCONTROLLED for:
- Performance-critical inputs
- Simple data collection
- Integration with native libraries
- File uploads (rare in RN)
```

---

### 2.7 **React Context API vs Redux Comparison**

**Question:** *"Compare React Context API with Redux for state management in React Native apps."*

**Perfect Answer Structure:**
```
CONTEXT API vs REDUX DECISION MATRIX:

🎯 CONTEXT API FOR:
- Small to medium apps
- Simple state trees
- UI theme/state
- Avoid prop drilling
- Learning curve: Low

🎯 REDUX FOR:
- Large, complex apps
- Complex state interactions
- Time-travel debugging
- Middleware requirements
- Learning curve: High

PERFORMANCE DIFFERENCES:
- Context: Re-renders all consumers on change
- Redux: Selective re-renders with connect()
- Context: Better for static data
- Redux: Better for dynamic data
```

**Complete Implementation Comparison:**
```javascript
// CONTEXT API IMPLEMENTATION
const ThemeContext = React.createContext();

const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

const ThemedComponent = () => {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <View style={{ backgroundColor: theme === 'dark' ? 'black' : 'white' }}>
      <Button title="Toggle Theme" onPress={toggleTheme} />
    </View>
  );
};

// REDUX IMPLEMENTATION
// store.js
const themeSlice = createSlice({
  name: 'theme',
  initialState: { mode: 'light' },
  reducers: {
    toggleTheme: (state) => {
      state.mode = state.mode === 'light' ? 'dark' : 'light';
    },
  },
});

export const { toggleTheme } = themeSlice.actions;
export default themeSlice.reducer;

// component
const ThemedComponent = () => {
  const theme = useSelector(state => state.theme.mode);
  const dispatch = useDispatch();

  return (
    <View style={{ backgroundColor: theme === 'dark' ? 'black' : 'white' }}>
      <Button title="Toggle Theme" onPress={() => dispatch(toggleTheme())} />
    </View>
  );
};
```

**Performance Optimization:**
```javascript
// Context API optimization
const ThemeContext = React.createContext();

const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const contextValue = useMemo(() => ({
    theme,
    toggleTheme: () => setTheme(prev => prev === 'light' ? 'dark' : 'light'),
  }), [theme]);

  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  );
};

// Redux optimization (automatic with connect)
const mapStateToProps = state => ({ theme: state.theme.mode });
const mapDispatchToProps = { toggleTheme };
export default connect(mapStateToProps, mapDispatchToProps)(ThemedComponent);
```

---

### 2.8 **Component Re-rendering Rules and Optimization**

**Question:** *"Explain React's re-rendering rules and optimization techniques for React Native."*

**Perfect Answer Structure:**
```
RE-RENDERING TRIGGERS:

1️⃣ State Changes: setState() calls
2️⃣ Props Changes: Parent re-renders
3️⃣ Context Changes: Context value updates
4️⃣ Force Updates: forceUpdate() calls

OPTIMIZATION HIERARCHY:
1. React.memo (prevents unnecessary renders)
2. useMemo (memoizes expensive computations)
3. useCallback (stable function references)
4. Key optimization (list rendering)
```

**Complete Optimization Strategy:**
```javascript
// BEFORE: Unoptimized component
const UserCard = ({ user, onPress, theme }) => {
  console.log('UserCard rendered:', user.name);

  const userDisplayName = user.firstName + ' ' + user.lastName;
  const cardStyle = {
    backgroundColor: theme === 'dark' ? 'black' : 'white',
    padding: 16,
    margin: 8,
  };

  return (
    <TouchableOpacity style={cardStyle} onPress={() => onPress(user)}>
      <Text style={{ color: theme === 'dark' ? 'white' : 'black' }}>
        {userDisplayName}
      </Text>
      <Text>{user.email}</Text>
    </TouchableOpacity>
  );
};

// AFTER: Fully optimized
const UserCard = React.memo(({ user, onPress, theme }) => {
  console.log('UserCard rendered:', user.name);

  const userDisplayName = useMemo(() => {
    return `${user.firstName} ${user.lastName}`;
  }, [user.firstName, user.lastName]);

  const cardStyle = useMemo(() => ({
    backgroundColor: theme === 'dark' ? 'black' : 'white',
    padding: 16,
    margin: 8,
  }), [theme]);

  const textColor = theme === 'dark' ? 'white' : 'black';

  const handlePress = useCallback(() => {
    onPress(user);
  }, [onPress, user]);

  return (
    <TouchableOpacity style={cardStyle} onPress={handlePress}>
      <Text style={{ color: textColor }}>
        {userDisplayName}
      </Text>
      <Text style={{ color: textColor }}>{user.email}</Text>
    </TouchableOpacity>
  );
});

// USAGE WITH KEY OPTIMIZATION
const UserList = ({ users, onUserPress, theme }) => {
  return (
    <FlatList
      data={users}
      keyExtractor={(item) => item.id.toString()}
      renderItem={({ item }) => (
        <UserCard
          user={item}
          onPress={onUserPress}
          theme={theme}
        />

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 3 ➡️](phase3-navigation-architecture.md)