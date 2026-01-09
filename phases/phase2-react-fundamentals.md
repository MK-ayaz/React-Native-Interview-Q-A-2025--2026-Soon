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
    background: linear-gradient(135deg, #2563eb 0%, #1e40af 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(37, 99, 235, 0.2);
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
    background: #dbeafe;
    color: #1e40af;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #fff7ed;
    border-left: 4px solid #f97316;
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
  table {
    width: 100%;
    border-collapse: collapse;
    margin: 24px 0;
  }
  th {
    background: #f1f5f9;
    text-align: left;
    padding: 12px;
    font-weight: 600;
    border-bottom: 2px solid #e2e8f0;
  }
  td {
    padding: 12px;
    border-bottom: 1px solid #e2e8f0;
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
  <span class="phase-number">Phase 02</span>
  <h1 class="phase-title">React Fundamentals Deep Dive</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Mastering Hooks, Lifecycle, State Management, and Performance Optimization</p>
</div>

---

### 📋 Phase Overview
Building on Phase 1's basics, this phase dives deep into React's core concepts with a focus on mobile-specific optimizations. You'll learn everything from basic hooks to advanced patterns that make React Native apps performant and maintainable.

---

<div class="qa-card">
  <span class="question-tag">BASICS</span>
  <h3 style="margin-top: 0;">Q1: What is React and why use it in React Native?</h3>

  <p><strong>React</strong> is a JavaScript library for building user interfaces, created by Facebook. React Native uses React as its foundation because React provides excellent patterns for:</p>

  <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 16px; margin: 24px 0;">
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px;">
      <strong>Component-Based Architecture</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem;">Reusable UI building blocks</p>
    </div>
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px;">
      <strong>Declarative Programming</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem;">Describe what UI should look like</p>
    </div>
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px;">
      <strong>Virtual DOM</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem;">Efficient rendering optimization</p>
    </div>
  </div>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> React's "Learn once, write anywhere" philosophy is perfect for React Native - you learn React concepts once and can build both web and mobile apps.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">COMPONENTS</span>
  <h3 style="margin-top: 0;">Q2: Explain functional vs class components in React.</h3>

  <p>React components can be written as functions or classes, but functional components with hooks are now the standard:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin: 24px 0;">
    <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #166534;">✅ Functional Components (Modern)</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Use hooks for state and lifecycle</li>
        <li>Less code, easier to test</li>
        <li>Better performance (no class overhead)</li>
        <li>React's recommended approach</li>
      </ul>
    </div>
    <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #92400e;">⚠️ Class Components (Legacy)</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Use `this.state` and lifecycle methods</li>
        <li>More verbose code</li>
        <li>Still supported but not recommended</li>
        <li>Being phased out in new code</li>
      </ul>
    </div>
  </div>

  <div class="code-example">Functional vs Class Component</div>
  <pre class="code-block"><code class="language-javascript">// Functional Component (Preferred)
function Counter() {
  const [count, setCount] = useState(0);

  return (
    &lt;TouchableOpacity onPress={() => setCount(count + 1)}>
      &lt;Text>Count: {count}&lt;/Text>
    &lt;/TouchableOpacity>
  );
}

// Class Component (Legacy)
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  render() {
    return (
      &lt;TouchableOpacity onPress={() => this.setState({count: this.state.count + 1})}>
        &lt;Text>Count: {this.state.count}&lt;/Text>
      &lt;/TouchableOpacity>
    );
  }
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">HOOKS BASICS</span>
  <h3 style="margin-top: 0;">Q3: What are React Hooks and why were they introduced?</h3>

  <p><strong>Hooks</strong> are functions that let you "hook into" React state and lifecycle features from functional components. They were introduced to solve problems with class components:</p>

  <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🎯 Problems Hooks Solve:</strong>
    <ul style="margin: 8px 0 0 0; padding-left: 20px;">
      <li>Reusing logic between components without render props or HOCs</li>
      <li>Complex components split across lifecycle methods</li>
      <li>Related code grouped together instead of scattered</li>
      <li>No more confusing `this` binding issues</li>
    </ul>
  </div>

  <div class="code-example">Basic Hooks Usage</div>
  <pre class="code-block"><code class="language-javascript">import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);        // State hook
  const [loading, setLoading] = useState(true);  // Another state hook

  useEffect(() => {                              // Effect hook
    fetchUser(userId).then(setUser);
    setLoading(false);
  }, [userId]);

  if (loading) return &lt;Text>Loading...&lt;/Text&gt;;
  return &lt;Text>Hello {user.name}!&lt;/Text&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">USESTATE</span>
  <h3 style="margin-top: 0;">Q4: Explain useState hook with examples.</h3>

  <p><strong>useState</strong> is the most basic and commonly used React hook. It allows functional components to have state:</p>

  <div class="code-example">useState Basic Usage</div>
  <pre class="code-block"><code class="language-javascript">// Simple counter
function Counter() {
  const [count, setCount] = useState(0);

  return (
    &lt;View&gt;
      &lt;Text&gt;Count: {count}&lt;/Text&gt;
      &lt;TouchableOpacity onPress={() => setCount(count + 1)}>
        &lt;Text&gt;Increment&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// Object state
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  const updateField = (field, value) => {
    setUser(prev => ({
      ...prev,
      [field]: value
    }));
  };

  return (
    &lt;View&gt;
      &lt;TextInput
        placeholder="Name"
        value={user.name}
        onChangeText={(value) => updateField('name', value)}
      /&gt;
    &lt;/View&gt;
  );
}

// Lazy initialization (for expensive computations)
function ExpensiveComponent() {
  const [data, setData] = useState(() => {
    console.log('Computing initial value...');
    return expensiveCalculation();
  });

  return &lt;Text&gt;Data: {data}&lt;/Text&gt;;
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> When updating objects or arrays in state, always use the functional update form (<code>setState(prev => ...)</code>) or spread syntax to avoid stale closure bugs.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">USEEFFECT</span>
  <h3 style="margin-top: 0;">Q5: Master useEffect hook - timing, dependencies, and cleanup.</h3>

  <p><strong>useEffect</strong> handles side effects in functional components. It's like componentDidMount, componentDidUpdate, and componentWillUnmount combined:</p>

  <div class="code-example">useEffect Patterns</div>
  <pre class="code-block"><code class="language-javascript">// 1. Run on every render (rarely needed)
useEffect(() => {
  console.log('Component rendered');
});

// 2. Run once on mount (API calls, subscriptions)
useEffect(() => {
  const fetchData = async () => {
    const data = await api.get('/user');
    setUser(data);
  };
  fetchData();
}, []); // Empty dependency array = run once

// 3. Run when specific values change
useEffect(() => {
  if (userId) {
    fetchUser(userId);
  }
}, [userId]); // Run when userId changes

// 4. Cleanup function (remove listeners, cancel requests)
useEffect(() => {
  const subscription = subscribeToUpdates();
  return () => {
    subscription.unsubscribe(); // Cleanup on unmount
  };
}, []);

// 5. Async effects (common pattern)
useEffect(() => {
  let isMounted = true;

  const loadData = async () => {
    try {
      const result = await fetchData();
      if (isMounted) setData(result);
    } catch (error) {
      if (isMounted) setError(error);
    }
  };

  loadData();

  return () => {
    isMounted = false; // Prevent state updates after unmount
  };
}, []);</code></pre>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> <em>"What happens if I forget the dependency array?"</em><br/>
    <strong>Answer:</strong> The effect runs after every render, which can cause infinite loops, poor performance, or stale closures. Always include all dependencies or use ESLint to catch this.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">USEREDUCER</span>
  <h3 style="margin-top: 0;">Q6: useReducer for complex state logic.</h3>

  <p><strong>useReducer</strong> is an alternative to useState for complex state logic. It follows the Redux pattern with actions and reducers:</p>

  <div class="code-example">useReducer Examples</div>
  <pre class="code-block"><code class="language-javascript">// Counter with useReducer
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return initialState;
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    &lt;View&gt;
      &lt;Text&gt;Count: {state.count}&lt;/Text&gt;
      &lt;TouchableOpacity onPress={() => dispatch({ type: 'increment' })}&gt;
        &lt;Text&gt;+&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// Complex form state
const formReducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return { ...state, [action.field]: action.value };
    case 'RESET_FORM':
      return action.initialState;
    case 'VALIDATE':
      return { ...state, errors: validateForm(state) };
    default:
      return state;
  }
};

function useFormReducer(initialState) {
  return useReducer(formReducer, initialState);
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Use useReducer when you have complex state logic with multiple sub-values or when the next state depends on the previous state in complex ways. For simple state, useState is usually easier.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">USEMEMO</span>
  <h3 style="margin-top: 0;">Q7: useMemo and useCallback for performance.</h3>

  <p><strong>useMemo</strong> memoizes expensive calculations, <strong>useCallback</strong> memoizes function definitions. Both prevent unnecessary re-renders:</p>

  <div class="code-example">useMemo vs useCallback</div>
  <pre class="code-block"><code class="language-javascript">// useMemo: Expensive calculation
const total = useMemo(() =>
  items.reduce((sum, item) => sum + item.price, 0),
[items]);

// useCallback: Function reference
const handleSubmit = useCallback((data) => {
  submitForm(data);
}, []);</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Don't use useMemo everywhere - only for expensive calculations. Simple operations like string concatenation don't need memoization.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">CONTEXT API</span>
  <h3 style="margin-top: 0;">Q8: Context API vs Redux/Zustand.</h3>

  <p><strong>Context API</strong> is built-in but causes all consumers to re-render when values change. Use it for static or low-frequency data like themes and user auth:</p>

  <div class="code-example">Context API Usage</div>
  <pre class="code-block"><code class="language-javascript">// Create context
const ThemeContext = React.createContext('light');

// Provider component
function App() {
  const [theme, setTheme] = useState('light');

  return (
    &lt;ThemeContext.Provider value={{ theme, setTheme }}>
      &lt;HomeScreen /&gt;
    &lt;/ThemeContext.Provider&gt;
  );
}

// Consumer with useContext hook (preferred)
function ThemeButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    &lt;TouchableOpacity
      onPress={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      &lt;Text&gt;Switch to {theme === 'light' ? 'dark' : 'light'} theme&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">RE-RENDERING</span>
  <h3 style="margin-top: 0;">Q9: Component re-rendering rules and optimization.</h3>

  <p>Components re-render when their state, props, or consumed context changes. Understanding this is crucial for performance:</p>

  <div class="code-example">Preventing Unnecessary Re-renders</div>
  <pre class="code-block"><code class="language-javascript">// React.memo for component memoization
const ListItem = React.memo(({ item, onPress }) => (
  &lt;TouchableOpacity onPress={() => onPress(item.id)}>
    &lt;Text&gt;{item.name}&lt;/Text&gt;
  &lt;/TouchableOpacity&gt;
));

// useCallback for stable function references
const handlePress = useCallback((id) => {
  doSomething(id);
}, []); // Empty deps = stable function reference</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">ERROR BOUNDARIES</span>
  <h3 style="margin-top: 0;">Q10: Error boundaries in React Native.</h3>

  <p><strong>Error boundaries</strong> catch JavaScript errors in the component tree, preventing the entire app from crashing. They only catch errors in the React component tree:</p>

  <div class="code-example">Error Boundary Implementation</div>
  <pre class="code-block"><code class="language-javascript">// Class-based error boundary
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        &lt;View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          &lt;Text style={{ fontSize: 18, color: 'red' }}>
            Something went wrong!
          &lt;/Text&gt;
        &lt;/View&gt;
      );
    }

    return this.props.children;
  }
}

// Usage
function App() {
  return (
    &lt;ErrorBoundary>
      &lt;MainScreen /&gt;
    &lt;/ErrorBoundary&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">CUSTOM HOOKS</span>
  <h3 style="margin-top: 0;">Q11: Custom hooks design patterns.</h3>

  <p><strong>Custom hooks</strong> extract component logic into reusable functions. They follow the naming convention of starting with "use":</p>

  <div class="code-example">Custom Hooks Examples</div>
  <pre class="code-block"><code class="language-javascript">// API hook
function useApi(endpoint) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(endpoint)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [endpoint]);

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useApi(`/api/users/${userId}`);

  if (loading) return &lt;Text&gt;Loading...&lt;/Text&gt;;
  if (error) return &lt;Text&gt;Error: {error.message}&lt;/Text&gt;;
  return &lt;Text&gt;Hello {user.name}!&lt;/Text&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PROFILER API</span>
  <h3 style="margin-top: 0;">Q12: React Profiler and performance auditing.</h3>

  <p>The <strong>React Profiler</strong> measures render performance and helps identify components that render too frequently or take too long:</p>

  <div class="code-example">Profiler API Usage</div>
  <pre class="code-block"><code class="language-javascript">// Wrap components to measure performance
import { Profiler } from 'react';

function onRenderCallback(
  id, // the "id" prop of the Profiler tree
  phase, // either "mount" or "update"
  actualDuration, // time spent rendering
  baseDuration, // estimated time without memoization
  startTime, // when render started
  commitTime, // when changes committed
  interactions // interactions that triggered the render
) {
  console.log(`${id} took ${actualDuration}ms to render`);
}

function App() {
  return (
    &lt;Profiler id="App" onRender={onRenderCallback}>
      &lt;ExpensiveComponent /&gt;
    &lt;/Profiler&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">REACT 18 FEATURES</span>
  <h3 style="margin-top: 0;">Q13: React 18 features and concurrent rendering.</h3>

  <p>React 18 brings automatic batching, concurrent features, and better performance:</p>

  <div class="code-example">React 18 Concurrent Features</div>
  <pre class="code-block"><code class="language-javascript">// Automatic batching (multiple updates batched together)
function handleFormSubmit() {
  setName('John');      // These get batched automatically
  setAge(25);          // No need for unstable_batchedUpdates
  setEmail('john@example.com');
}

// Concurrent rendering with startTransition
import { startTransition } from 'react';

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);

  const handleSearch = (term) => {
    setSearchTerm(term);

    // Mark expensive search as non-urgent
    startTransition(() => {
      const filtered = expensiveSearch(term);
      setResults(filtered);
    });
  };

  return (
    &lt;View&gt;
      &lt;TextInput
        placeholder="Search..."
        onChangeText={handleSearch}
        value={searchTerm}
      /&gt;
      {results.map(result => &lt;Text key={result.id}>{result.name}&lt;/Text&gt;)}
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE PATTERNS</span>
  <h3 style="margin-top: 0;">Q14: Advanced performance optimization patterns.</h3>

  <p>Master these patterns for high-performance React Native apps:</p>

  <div class="code-example">Advanced Performance Patterns</div>
  <pre class="code-block"><code class="language-javascript">// 1. React.memo with custom comparison
const ListItem = React.memo(({ item, onPress }) => (
  &lt;TouchableOpacity onPress={() => onPress(item.id)}>
    &lt;Text&gt;{item.name}&lt;/Text&gt;
  &lt;/TouchableOpacity&gt;
), (prevProps, nextProps) => {
  // Custom comparison function
  return prevProps.item.id === nextProps.item.id &&
         prevProps.onPress === nextProps.onPress;
});

// 2. Selective context with atoms pattern
const UserContext = React.createContext();
const ThemeContext = React.createContext();

// Split contexts to avoid unnecessary re-renders
function App() {
  return (
    &lt;UserContext.Provider value={user}>
      &lt;ThemeContext.Provider value={theme}>
        &lt;Screen /&gt;
      &lt;/ThemeContext.Provider&gt;
    &lt;/UserContext.Provider&gt;
  );
}

// 3. useDeferredValue for non-critical updates
import { useDeferredValue } from 'react';

function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query);
  const results = useMemo(() => search(deferredQuery), [deferredQuery]);

  return (
    &lt;FlatList
      data={results}
      renderItem={({ item }) => &lt;Text&gt;{item}&lt;/Text&gt;}
    /&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">FORWARDREF & IMPERATIVE</span>
  <h3 style="margin-top: 0;">Q15: forwardRef and useImperativeHandle patterns.</h3>

  <p><strong>forwardRef</strong> allows parent components to access child refs, while <strong>useImperativeHandle</strong> customizes the exposed API:</p>

  <div class="code-example">Ref Patterns</div>
  <pre class="code-block"><code class="language-javascript">// Custom input with imperative methods
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    blur: () => inputRef.current.blur(),
    getValue: () => inputRef.current._lastNativeText
  }));

  return &lt;TextInput ref={inputRef} {...props} /&gt;;
});

// Usage
function LoginForm() {
  const inputRef = useRef();

  return (
    &lt;View&gt;
      &lt;FancyInput ref={inputRef} placeholder="Username" /&gt;
      &lt;TouchableOpacity onPress={() => inputRef.current.focus()}>
        &lt;Text&gt;Focus Input&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">HOOKS RULES</span>
  <h3 style="margin-top: 0;">Q16: Advanced hooks patterns and Rules of Hooks.</h3>

  <p>Master the Rules of Hooks and advanced patterns:</p>

  <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>⚠️ Rules of Hooks (Never Break These):</strong>
    <ol style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Only call hooks at the top level</strong> - Never in loops, conditions, or nested functions</li>
      <li><strong>Only call hooks from React functions</strong> - Only in components or custom hooks</li>
    </ol>
  </div>

  <div class="code-example">Advanced Hook Patterns</div>
  <pre class="code-block"><code class="language-javascript">// Custom hook with complex logic
function useAsyncOperation(operation, deps = []) {
  const [state, setState] = useState({
    loading: false,
    data: null,
    error: null
  });

  const execute = useCallback(async (...args) => {
    setState({ loading: true, data: null, error: null });
    try {
      const result = await operation(...args);
      setState({ loading: false, data: result, error: null });
      return result;
    } catch (error) {
      setState({ loading: false, data: null, error });
      throw error;
    }
  }, deps);

  return { ...state, execute };
}

// Usage
function UserProfile({ userId }) {
  const { loading, data: user, error, execute } = useAsyncOperation(
    async (id) => await fetchUser(id),
    []
  );

  useEffect(() => {
    execute(userId);
  }, [execute, userId]);

  // ... render logic
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">SUSPENSE</span>
  <h3 style="margin-top: 0;">Q17: Suspense for data fetching and code splitting.</h3>

  <p><strong>Suspense</strong> enables better loading states and lazy loading:</p>

  <div class="code-example">Suspense Patterns</div>
  <pre class="code-block"><code class="language-javascript">// Lazy loading components
const LazyProfileScreen = lazy(() => import('./ProfileScreen'));

function App() {
  return (
    &lt;Suspense fallback={&lt;Text&gt;Loading...&lt;/Text&gt;}>
      &lt;LazyProfileScreen /&gt;
    &lt;/Suspense&gt;
  );
}

// Custom resource for data fetching
function createResource(promise) {
  let status = 'pending';
  let result;

  const suspender = promise.then(
    res => { status = 'success'; result = res; },
    err => { status = 'error'; result = err; }
  );

  return {
    read() {
      if (status === 'pending') throw suspender;
      if (status === 'error') throw result;
      return result;
    }
  };
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">RENDER OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q18: Advanced render optimization techniques.</h3>

  <p>Master advanced techniques to prevent unnecessary re-renders:</p>

  <div class="code-example">Advanced Optimization Techniques</div>
  <pre class="code-block"><code class="language-javascript">// 1. Deep comparison with custom memo
const DeepMemoComponent = React.memo(Component, (prevProps, nextProps) => {
  // Custom comparison function
  return JSON.stringify(prevProps) === JSON.stringify(nextProps);
});

// 2. Event handler optimization
function EventHandlerOptimization() {
  const [items, setItems] = useState([]);

  // ❌ Creates new function every render
  const handleDelete = (id) => setItems(items => items.filter(item => item.id !== id));

  // ✅ Stable function reference
  const handleDelete = useCallback((id) => {
    setItems(items => items.filter(item => item.id !== id));
  }, []); // Empty deps because it doesn't depend on props/state

  return (
    &lt;FlatList
      data={items}
      renderItem={({ item }) => (
        &lt;TouchableOpacity onPress={() => handleDelete(item.id)}>
          &lt;Text&gt;Delete {item.name}&lt;/Text&gt;
        &lt;/TouchableOpacity&gt;
      )}
    /&gt;
  );
}

// 3. Context optimization with selectors
const UserContext = React.createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  const contextValue = useMemo(() => ({
    user,
    setUser,
    // Computed values
    isLoggedIn: !!user,
    displayName: user ? `${user.firstName} ${user.lastName}` : 'Guest'
  }), [user]);

  return (
    &lt;UserContext.Provider value={contextValue}>
      {children}
    &lt;/UserContext.Provider&gt;
  );
}

// Custom hook with selector pattern
function useUser() {
  const context = useContext(UserContext);
  return context;
}

function useUserDisplayName() {
  const { displayName } = useContext(UserContext);
  return displayName; // Only re-renders when displayName changes
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">VIRTUAL DOM</span>
  <h3 style="margin-top: 0;">Q19: Understanding React's Virtual DOM and reconciliation.</h3>

  <p>The <strong>Virtual DOM</strong> is React's lightweight representation of the actual DOM/native tree. Reconciliation is the process of updating it efficiently:</p>

  <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🔄 Reconciliation Process:</strong>
    <ol style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Render:</strong> React creates/updates Virtual DOM</li>
      <li><strong>Diff:</strong> Compares new Virtual DOM with previous</li>
      <li><strong>Reconcile:</strong> Calculates minimal changes needed</li>
      <li><strong>Commit:</strong> Applies changes to actual DOM/native views</li>
    </ol>
  </div>

  <div class="code-example">How Reconciliation Works</div>
  <pre class="code-block"><code class="language-javascript">// Keys help React identify which items changed
function TodoList({ todos }) {
  return (
    &lt;FlatList
      data={todos}
      keyExtractor={(item) => item.id} // Important for reconciliation!
      renderItem={({ item }) => &lt;Text&gt;{item.text}&lt;/Text&gt;}
    /&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">COMMON MISTAKES</span>
  <h3 style="margin-top: 0;">Q20: Common React hooks mistakes and how to avoid them.</h3>

  <p>Avoiding these pitfalls will save you hours of debugging:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin: 24px 0;">
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Stale Closures</h4>
      <p style="margin: 0; font-size: 0.875rem;">useEffect captures values at creation time, not when it runs.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Missing Dependencies</h4>
      <p style="margin: 0; font-size: 0.875rem;">ESLint warnings about missing deps are important!</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Infinite Loops</h4>
      <p style="margin: 0; font-size: 0.875rem;">Objects/arrays in deps cause re-creation every render.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Wrong Hook Order</h4>
      <p style="margin: 0; font-size: 0.875rem;">Hooks must be called in the same order every render.</p>
    </div>
  </div>

  <div class="code-example">Fixing Common Mistakes</div>
  <pre class="code-block"><code class="language-javascript">// ❌ PROBLEM: Stale closure
function BadCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count); // Always logs initial value (0)
    }, 1000);
  }, []); // Empty deps = stale closure

  return &lt;TouchableOpacity onPress={() => setCount(c => c + 1)} /&gt;;
}

// ✅ SOLUTION: Include dependencies
function GoodCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count); // Now logs current value
    }, 1000);
  }, [count]); // Include count in deps

  return &lt;TouchableOpacity onPress={() => setCount(c => c + 1)} /&gt;;
}

// ❌ PROBLEM: Object in dependencies
const [user, setUser] = useState({ name: '', age: 0 });

useEffect(() => {
  saveUser(user);
}, [user]); // user object recreated every render = infinite loop

// ✅ SOLUTION: Use primitive values or useCallback
const [name, setName] = useState('');
const [age, setAge] = useState(0);

useEffect(() => {
  saveUser({ name, age });
}, [name, age]); // Primitive values don't cause loops</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">DEBUGGING HOOKS</span>
  <h3 style="margin-top: 0;">Q21: Debugging React hooks and performance monitoring.</h3>

  <p>Debug hooks issues and monitor performance in production:</p>

  <div class="code-example">Debugging and Monitoring Hooks</div>
  <pre class="code-block"><code class="language-javascript">// 1. Log shared values (development only)
function DebuggableAnimation() {
  const scale = useSharedValue(1);

  // Add debugging in development
  if (__DEV__) {
    useAnimatedReaction(() => scale.value, (value) => {
      console.log('Scale changed to:', value);
    });
  }

  return &lt;Animated.View style={animatedStyle} /&gt;;
}

// 2. Custom performance hook
function usePerformanceMonitor(componentName) {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(0);

  renderCount.current += 1;

  useEffect(() => {
    const now = performance.now();
    const timeSinceLastRender = now - lastRenderTime.current;

    if (__DEV__) {
      console.log(`${componentName} render #${renderCount.current}: ${timeSinceLastRender.toFixed(2)}ms`);
    }

    lastRenderTime.current = now;
  });

  return renderCount.current;
}

// 3. Hook dependency debugger
function useDependencyDebugger(dependencies, label = 'Dependencies') {
  const prevDeps = useRef(dependencies);

  useEffect(() => {
    const changedDeps = dependencies.reduce((acc, dep, index) => {
      if (prevDeps.current[index] !== dep) {
        acc.push(`[${index}]: ${prevDeps.current[index]} → ${dep}`);
      }
      return acc;
    }, []);

    if (changedDeps.length > 0) {
      console.log(`${label} changed:`, changedDeps);
    }

    prevDeps.current = dependencies;
  }, dependencies);
}

// Usage
function ExpensiveComponent({ data, filter }) {
  usePerformanceMonitor('ExpensiveComponent');
  useDependencyDebugger([data, filter], 'ExpensiveComponent deps');

  const filteredData = useMemo(() => {
    return data.filter(item => item.type === filter);
  }, [data, filter]);

  return &lt;FlatList data={filteredData} renderItem={renderItem} /&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">CONTROLLED COMPONENTS</span>
  <h3 style="margin-top: 0;">Q22: Controlled vs uncontrolled components.</h3>

  <p><strong>Controlled components</strong> have their value controlled by React state, while <strong>uncontrolled components</strong> manage their own state internally:</p>

  <div class="code-example">Controlled vs Uncontrolled</div>
  <pre class="code-block"><code class="language-javascript">// ✅ CONTROLLED: React controls the value
function ControlledInput() {
  const [value, setValue] = useState('');

  return (
    &lt;TextInput
      value={value}                    // Controlled by state
      onChangeText={setValue}          // Updates state
      placeholder="Type something..."
    /&gt;
  );
}

// ❌ UNCONTROLLED: Component manages its own state
function UncontrolledInput() {
  const inputRef = useRef();

  const handleSubmit = () => {
    const value = inputRef.current._lastNativeText; // Direct access
    console.log('Input value:', value);
  };

  return (
    &lt;View&gt;
      &lt;TextInput
        ref={inputRef}                  // No value/onChangeText
        placeholder="Type something..."
      /&gt;
      &lt;TouchableOpacity onPress={handleSubmit}>
        &lt;Text&gt;Submit&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// When to use each:
// ✅ Controlled: Form validation, dynamic formatting, controlled UX
// ✅ Uncontrolled: Performance-critical inputs, simple forms, refs needed</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">HOCS VS HOOKS</span>
  <h3 style="margin-top: 0;">Q23: HOCs vs Hooks - composition patterns.</h3>

  <p><strong>Higher-Order Components (HOCs)</strong> and <strong>Hooks</strong> both solve the problem of reusing logic, but Hooks are now the preferred approach:</p>

  <div class="code-example">HOCs vs Hooks</div>
  <pre class="code-block"><code class="language-javascript">// ❌ OLD: Higher-Order Component
function withUserData(WrappedComponent) {
  return class extends Component {
    state = { user: null, loading: true };

    componentDidMount() {
      fetchUser().then(user => this.setState({ user, loading: false }));
    }

    render() {
      return &lt;WrappedComponent {...this.props} {...this.state} /&gt;;
    }
  };
}

// Usage (creates wrapper hell)
const UserProfileWithData = withUserData(UserProfile);

// ✅ NEW: Custom Hook
function useUserData() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser().then(user => {
      setUser(user);
      setLoading(false);
    });
  }, []);

  return { user, loading };
}

// Usage (clean composition)
function UserProfile() {
  const { user, loading } = useUserData();

  if (loading) return &lt;Text&gt;Loading...&lt;/Text&gt;;
  return &lt;Text&gt;Hello {user.name}!&lt;/Text&gt;;
}

// Hooks advantages:
// ✅ No wrapper components in React tree
// ✅ Better TypeScript support
// ✅ Easier to compose multiple hooks
// ✅ Follows normal function rules</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">LIFECYCLE MAPPING</span>
  <h3 style="margin-top: 0;">Q24: Map class lifecycle methods to hooks.</h3>

  <p>Understanding how class lifecycle methods translate to hooks is essential for migrating legacy code:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Class Lifecycle</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Functional Hook Equivalent</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Key Difference</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>constructor</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useState(initialValue)</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">State initialization happens on every render</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>componentDidMount</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useEffect(fn, [])</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Runs after first render/native mount</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>componentDidUpdate</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useEffect(fn, [deps])</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Skips if dependencies haven't changed</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>componentWillUnmount</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useEffect(() => cleanup, [])</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Cleanup function returned from effect</td>
    </tr>
    <tr>
      <td style="padding: 12px;"><code>shouldComponentUpdate</code></td>
      <td style="padding: 12px;"><code>React.memo()</code></td>
      <td style="padding: 12px;">Shallow comparison by default</td>
    </tr>
  </table>
</div>

<div class="qa-card">
  <span class="question-tag">ADVANCED PATTERNS</span>
  <h3 style="margin-top: 0;">Q25: Advanced React patterns for complex applications.</h3>

  <p>Master these patterns for building maintainable large-scale React Native apps:</p>

  <div class="code-example">Advanced Patterns</div>
  <pre class="code-block"><code class="language-javascript">// 1. Higher-Order Components (still useful for some cases)
function withErrorBoundary(WrappedComponent) {
  return class extends Component {
    state = { hasError: false };

    static getDerivedStateFromError() {
      return { hasError: true };
    }

    render() {
      if (this.state.hasError) {
        return &lt;Text&gt;Something went wrong&lt;/Text&gt;;
      }
      return &lt;WrappedComponent {...this.props} /&gt;;
    }
  };
}

// 2. Render Props Pattern
class DataProvider extends Component {
  state = { data: null, loading: true };

  componentDidMount() {
    fetchData().then(data => this.setState({ data, loading: false }));
  }

  render() {
    return this.props.render(this.state);
  }
}

// Usage
&lt;DataProvider
  render={({ data, loading }) => (
    loading ? &lt;Text&gt;Loading...&lt;/Text&gt; : &lt;Text&gt;{data.name}&lt;/Text&gt;
  )}
/&gt;

// 3. Compound Component Pattern
const Modal = ({ children }) => &lt;View style={styles.modal}>{children}&lt;/View&gt;;
Modal.Header = ({ children }) => &lt;Text style={styles.header}>{children}&lt;/Text&gt;;
Modal.Body = ({ children }) => &lt;View style={styles.body}>{children}&lt;/View&gt;;
Modal.Footer = ({ children }) => &lt;View style={styles.footer}>{children}&lt;/View&gt;;

// Usage
function ConfirmDialog() {
  return (
    &lt;Modal&gt;
      &lt;Modal.Header&gt;Confirm Action&lt;/Modal.Header&gt;
      &lt;Modal.Body&gt;Are you sure you want to delete this item?&lt;/Modal.Body&gt;
      &lt;Modal.Footer&gt;
        &lt;Button&gt;Cancel&lt;/Button&gt;
        &lt;Button&gt;Delete&lt;/Button&gt;
      &lt;/Modal.Footer&gt;
    &lt;/Modal&gt;
  );
}</code></pre>
</div>

---

<div class="nav-footer">
  <a href="phase1-core-fundamentals.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 01: Fundamentals</a>
  <a href="phase3-navigation-architecture.md" style="color: #2563eb; text-decoration: none; font-weight: 600;">Phase 03: Navigation ➡️</a>
</div>

</div>