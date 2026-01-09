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
    background: linear-gradient(135deg, #06b6d4 0%, #0891b2 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(6, 182, 212, 0.2);
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
    background: #ecfeff;
    color: #0891b2;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #f0fdfa;
    border-left: 4px solid #0d9488;
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
  <span class="phase-number">Phase 09</span>
  <h1 class="phase-title">Testing & Quality Assurance</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Unit Tests, E2E, Automation, and QA Best Practices</p>
</div>

---

### 📋 Phase Overview
Master testing strategies for React Native apps from basic unit tests to advanced E2E automation. Learn to write reliable tests, debug issues, and maintain quality throughout the development lifecycle.

---

<div class="qa-card">
  <span class="question-tag">STRATEGY</span>
  <h3>9.1 The Testing Pyramid vs. The Testing Trophy</h3>
  
  **Question:** *"How do you balance Unit, Integration, and E2E tests in a React Native project?"*
  
  <p>A senior strategy often shifts from the traditional <strong>Testing Pyramid</strong> to the <strong>Testing Trophy</strong> (by Kent C. Dodds), which prioritizes <strong>Integration Tests</strong>. In React Native, this means testing how components interact with their state, navigation, and API clients using RNTL.</p>
  
  <ul>
    <li><strong>Unit Tests:</strong> Pure functions, utilities, and Redux/Zustand logic (Jest).</li>
    <li><strong>Integration Tests:</strong> Screen-level flows, complex hooks, and component trees (RNTL + MSW).</li>
    <li><strong>E2E Tests:</strong> High-value user journeys on real devices/simulators (Detox/Maestro).</li>
    <li><strong>Static:</strong> Catching typos and type errors (TypeScript/ESLint).</li>
  </ul>

  <div class="senior-insight">
    <strong>💡 Senior Insight: The "Confidence per Minute" Metric</strong><br/>
    Integration tests provide the highest ROI. They give you confidence that your feature works without the massive speed and maintenance cost of E2E tests. A senior dev will spend 60% of their testing time here.
  </div>

  <div class="follow-up-trap">
    <strong>⚠️ Follow-up Trap:</strong> "Is 100% test coverage a good goal?"<br/>
    <strong>Answer:</strong> No. It leads to testing implementation details rather than behavior. Focus on <strong>Critical Path Coverage</strong> (e.g., checkout, login, data persistence).
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">UNIT TESTING</span>
  <h3>9.2 Testing Custom Hooks with Asynchronous State</h3>
  
  **Question:** *"How do you test a custom hook that involves asynchronous state updates or API calls?"*
  
  <p>Use <code>renderHook</code> from <code>@testing-library/react-native</code>. The key is to handle the "wait" for the state to update after an async action.</p>

  <div class="code-block-header">Testing Async Hooks</div>

  ```typescript
  test('should fetch and update user data', async () => {
    const { result } = renderHook(() => useUser(1));

    // Initially loading
    expect(result.current.loading).toBe(true);

    // Wait for the async effect to settle
    await waitFor(() => expect(result.current.loading).toBe(false));

    expect(result.current.user.name).toBe('John Doe');
  });
  ```

  <div class="senior-insight">
    <strong>💡 Senior Insight: The "act()" warning</strong><br/>
    If you see "An update to a component... was not wrapped in act()", it usually means your test finished before an asynchronous state update occurred. <code>waitFor</code> internally wraps its checks in <code>act</code>, making it the preferred way to solve this.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">COMPONENT TESTING</span>
  <h3>9.3 RNTL: User-Centric Queries vs. TestID</h3>
  
  **Question:** *"Why does RNTL discourage the use of testID, and when is it acceptable to use it?"*
  
  <p>Queries like <code>getByRole</code>, <code>getByLabelText</code>, and <code>getByText</code> mirror how a user (and a screen reader) interacts with the app. If you change a <code>TouchableOpacity</code> to a <code>Pressable</code>, a role-based query will still work, but an implementation-based one might fail.</p>

  <p><strong>Acceptable testID use cases:</strong></p>
  <ul>
    <li>Non-interactive decorative elements that need to be verified.</li>
    <li>Complex SVG charts where text-based querying is impossible.</li>
    <li>E2E testing (Detox) where native selectors are required.</li>
  </ul>

  <div class="follow-up-trap">
    <strong>⚠️ Follow-up Trap:</strong> "How do you test a component that is hidden behind a conditional?"<br/>
    <strong>Answer:</strong> Use <code>queryByText</code> instead of <code>getByText</code>. <code>getBy</code> throws an error if the element is missing, while <code>queryBy</code> returns <code>null</code>, allowing you to <code>expect(...).toBeNull()</code>.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">E2E TESTING</span>
  <h3>9.4 Detox: Synchronization & Idleness</h3>
  
  **Question:** *"How does Detox's synchronization mechanism work, and how do you handle 'stuck' tests?"*
  
  <p>Detox is a <strong>Gray-box</strong> runner. It monitors the app's internal resources: the main thread, the JS thread, the network (OkHttp/NSURLSession), and timers. It only executes the next test step when the app is "Idle."</p>

  <div class="senior-insight">
    <strong>💡 Senior Insight: The "Infinity Timer" Problem</strong><br/>
    If your app has a looping animation or a polling network request, Detox will never see the app as "Idle" and the test will timeout. Use <code>device.disableSynchronization()</code> for those specific steps, or better yet, mock the polling interval to be very large during tests.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">MOCKING</span>
  <h3>9.5 Advanced Native Module Mocking</h3>
  
  **Question:** *"How do you mock a native module that uses an EventEmitter (e.g., NetInfo or Keyboard)?"*
  
  <p>You must mock both the static methods and the listener registration system.</p>

  <div class="code-block-header">Mocking EventEmitter Modules</div>

  ```javascript
  jest.mock('@react-native-community/netinfo', () => ({
    addEventListener: jest.fn(),
    fetch: jest.fn(() => Promise.resolve({ isConnected: true })),
  }));
  ```

  <div class="follow-up-trap">
    <strong>⚠️ Follow-up Trap:</strong> "What is the danger of mocking everything?"<br/>
    <strong>Answer:</strong> Your tests might pass while the real app fails because the mock's behavior has diverged from the actual native implementation. Periodically verify mocks against real hardware.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">STABILITY</span>
  <h3>9.6 Snapshots vs. Visual Regression</h3>
  
  **Question:** *"Compare Jest Snapshots with Visual Regression tools like Loki or Applitools."*
  
  <ul>
    <li><strong>Jest Snapshots:</strong> Compare the <strong>rendered JSON tree</strong>. Good for catching logic changes in props/state. Useless for catching CSS/Layout bugs (e.g., a button hidden behind another view).</li>
    <li><strong>Visual Regression:</strong> Compare <strong>actual pixel screenshots</strong>. Catches layout shifts, font-rendering issues, and platform-specific styling bugs.</li>
  </ul>

  <div class="senior-insight">
    <strong>💡 Senior Insight: The Snapshot Burden</strong><br/>
    Seniors avoid giant snapshots of whole screens. They are hard to review and often get updated blindly. Prefer small, focused snapshots of specific component states (e.g., "Error State Snapshot").
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">MONITORING</span>
  <h3>9.7 Robust Error Boundaries</h3>
  
  **Question:** *"Beyond showing a fallback UI, what should a senior-level Error Boundary do?"*
  
  <p>It should act as a <strong>diagnostic hub</strong>:</p>
  <ul>
    <li>Capture the component stack trace (via <code>componentDidCatch</code>).</li>
    <li>Log the error to a service (Sentry/Bugsnag) with <strong>user context</strong>.</li>
    <li>Provide a "Hard Reset" that clears <code>AsyncStorage</code>/<code>MMKV</code> if the crash is caused by corrupted local data.</li>
  </ul>

  <div class="code-block-header">Production Error Boundary</div>

  ```typescript
  componentDidCatch(error: Error, info: { componentStack: string }) {
    Sentry.captureException(error, { extra: info });
  }
  ```
</div>

<div class="qa-card">
  <span class="question-tag">OBSERVABILITY</span>
  <h3>9.8 Sentry: Source Maps & Breadcrumbs</h3>
  
  **Question:** *"Why are Source Maps critical for React Native production monitoring?"*
  
  <p>In production, JS is minified and bundled. Without Source Maps, a Sentry stack trace will just point to <code>index.bundle:1:59283</code>. Source Maps allow Sentry to translate that back to your actual TypeScript file and line number.</p>

  <div class="senior-insight">
    <strong>💡 Senior Insight: Custom Breadcrumbs</strong><br/>
    Don't just rely on automatic breadcrumbs. Manually add breadcrumbs for "Business Events" (e.g., <code>Sentry.addBreadcrumb({ category: 'auth', message: 'User clicked Buy Button' })</code>). This makes debugging logic-heavy crashes much easier.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">DEBUGGING</span>
  <h3>9.9 Flipper vs. React Inspector</h3>
  
  **Question:** *"How do you debug a performance issue that only happens on Android?"*
  
  <p>Use the <strong>Flipper "Profiler"</strong>. Unlike the React Inspector which shows component trees, the Profiler shows <strong>native frame drops</strong>. You can see if the bottleneck is on the UI thread (layout thrashing) or the JS thread (heavy computation).</p>

  <div class="follow-up-trap">
    <strong>⚠️ Follow-up Trap:</strong> "Can you use Flipper in a production build?"<br/>
    <strong>Answer:</strong> No. Flipper requires the <code>flipper-plugin</code> native code which is stripped out in release builds for security and size. Use Sentry's "Performance" tab for production profiling.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE</span>
  <h3>9.10 Hermes Sampling Profiler</h3>
  
  **Question:** *"What is the difference between 'Sampling' and 'Tracing' profilers?"*
  
  <ul>
    <li><strong>Sampling (Hermes):</strong> Checks the call stack every few milliseconds. Low overhead, good for finding "Hot Paths" without slowing down the app.</li>
    <li><strong>Tracing:</strong> Records every single function call and its duration. High overhead, can actually cause jank while profiling, but gives 100% accuracy for short bursts.</li>
  </ul>
</div>

<div class="qa-card">
  <span class="question-tag">API MOCKING</span>
  <h3>9.11 MSW (Mock Service Worker)</h3>
  
  **Question:** *"How do you implement MSW to test a complex Redux Toolkit Query flow?"*
  
  <p>MSW intercepts the actual network request. This means your RTK Query hooks think they are talking to a real server, testing your <code>transformResponse</code>, <code>providesTags</code>, and error-handling logic.</p>

  <div class="code-block-header">MSW Handler Example</div>

  ```typescript
  export const handlers = [
    rest.get('/api/user', (req, res, ctx) => {
      return res(ctx.status(200), ctx.json({ name: 'John' }));
    }),
  ];
  ```
</div>

<div class="qa-card">
  <span class="question-tag">CI/CD</span>
  <h3>9.12 CI Optimization: Caching & Parallelization</h3>
  
  **Question:** *"How do you reduce Detox CI time from 40 minutes to 10 minutes?"*
  
  <ul>
    <li><strong>Build Caching:</strong> Cache the <code>ios/build</code> and <code>android/app/build</code> folders using GitHub Actions cache.</li>
    <li><strong>Sharding:</strong> Split your test files and run them in parallel on 4 different macOS runners.</li>
    <li><strong>Pre-built Binaries:</strong> Build the app once, upload the <code>.app</code> or <code>.apk</code> as an artifact, and have the test runners download and use it instead of re-building.</li>
  </ul>
</div>

<div class="qa-card">
  <span class="question-tag">USER INTERACTION</span>
  <h3>9.13 Testing Complex Gestures</h3>
  
  **Question:** *"How do you simulate a multi-touch 'Pinch' gesture in an integration test?"*
  
  <p>Standard RNTL <code>fireEvent.press</code> isn't enough. You must use <code>fireEvent</code> with the specific event data expected by <code>react-native-gesture-handler</code>, simulating the <code>onGestureEvent</code> with multiple pointers and scale values.</p>

  <div class="senior-insight">
    <strong>💡 Senior Insight: Manual Mocks for RNGH</strong><br/>
    Since simulating real touches is hard, seniors often mock the <code>GestureDetector</code> to immediately trigger the <code>onEnd</code> or <code>onUpdate</code> callback with fake data to verify the resulting state change.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ISOLATION</span>
  <h3>9.14 Storybook Interaction Testing</h3>
  
  **Question:** *"How do you combine Storybook with Jest to prevent 'Visual Drift'?"*
  
  <p>Use <strong>Storybook Interaction Tests</strong> (via the <code>@storybook/addon-interactions</code>). It allows you to write "play" functions that simulate user clicks inside the Storybook UI. You can then run these same stories as Jest tests using <code>@storybook/testing-react</code>.</p>
</div>

<div class="qa-card">
  <span class="question-tag">ACCESSIBILITY</span>
  <h3>9.15 Accessibility-First Testing</h3>
  
  **Question:** *"How do you verify that a custom 'CheckBox' is accessible to screen readers?"*
  
  <p>Your test should look for the <code>checkbox</code> role and the <code>checked</code> state, rather than just checking if an icon is rendered.</p>

  <div class="code-block-header">Accessibility Test</div>

  ```typescript
  const checkbox = screen.getByRole('checkbox', { name: /accept terms/i });
  expect(checkbox).toHaveProp('accessibilityState', { checked: true });
  ```
</div>

<div class="qa-card">
  <span class="question-tag">SECURITY</span>
  <h3>9.16 Dependency Scanning</h3>
  
  **Question:** *"What is 'Software Composition Analysis' (SCA) and why is it part of RN QA?"*
  
  <p>SCA tools (like <strong>Snyk</strong> or <strong>GitHub Dependabot</strong>) scan your <code>package.json</code> and <code>yarn.lock</code> for libraries with known security vulnerabilities. In RN, this is critical because a vulnerable native library can expose the entire device's keychain or filesystem.</p>
</div>

<div class="qa-card">
  <span class="question-tag">CODE QUALITY</span>
  <h3>9.17 ESLint for React Native Performance</h3>
  
  **Question:** *"Which ESLint rules are specific to React Native performance and quality?"*
  
  <ul>
    <li><code>react-native/no-inline-styles</code>: Prevents object creation on every render.</li>
    <li><code>react-hooks/exhaustive-deps</code>: Critical for preventing infinite loops in <code>useEffect</code>.</li>
    <li><code>react-native/no-raw-text</code>: Ensures all text is wrapped in a <code><Text></code> component (which would crash on Android otherwise).</li>
  </ul>
</div>

<div class="qa-card">
  <span class="question-tag">MOCK DATA</span>
  <h3>9.18 The Factory Pattern with Fishery</h3>
  
  **Question:** *"Why is the 'Factory Pattern' better than 'Fixture Files' for test data?"*
  
  <p>Fixture files (giant JSONs) are rigid. If you need a user with a slightly different email, you have to create a new file or mutate the object (dangerous). <strong>Factories</strong> allow for dynamic, type-safe generation.</p>

  <div class="code-block-header">Fishery Factory Example</div>

  ```typescript
  const userFactory = Factory.define<User>(({ sequence }) => ({
    id: sequence,
    name: 'Test User',
    email: `user${sequence}@example.com`,
  }));

  const admin = userFactory.build({ role: 'admin' });
  ```
</div>

<div class="qa-card">
  <span class="question-tag">RELIABILITY</span>
  <h3>9.19 Flakiness: Network vs. Timers</h3>
  
  **Question:** *"How do you debug a Detox test that passes locally but fails 20% of the time in CI?"*
  
  <ul>
    <li><strong>Network Flakiness:</strong> Use MSW to remove real network dependency.</li>
    <li><strong>Resource Contention:</strong> CI runners are often slower than local machines. Increase <code>interactTimeout</code> in Detox.</li>
    <li><strong>Artifact Inspection:</strong> Configure CI to record a video of the failed test and upload it. Often, a "Permission Popup" or "System Update" is blocking the UI.</li>
  </ul>
</div>

<div class="qa-card">
  <span class="question-tag">AUTOMATION</span>
  <h3>9.20 Visual Regression: Percy & Loki</h3>
  
  **Question:** *"How do you handle 'False Positives' in Visual Regression (e.g., a flashing cursor or a timestamp)?"*
  
  <p>Use "Ignore Regions." Tools like <strong>Percy</strong> or <strong>Applitools</strong> allow you to draw boxes over dynamic areas (like clocks or random IDs) that should be ignored during the pixel-comparison process.</p>
</div>

<div class="qa-card">
  <span class="question-tag">QUALITY METRICS</span>
  <h3>9.21 Beyond Coverage: Lighthouse for Mobile?</h3>
  
  **Question:** *"How do you measure 'Quality' beyond just test pass rates?"*
  
  <p>Use <strong>FlashList's 'onBlankArea'</strong> to measure list performance, <strong>Sentry's User Misery Score</strong> to measure how many users experience slow loads, and <strong>Play Store/App Store Vitals</strong> (ANR rates, Crash rates) as the ultimate QA metrics.</p>
</div>

<div class="qa-card">
  <span class="question-tag">TESTING BASICS</span>
  <h3 style="margin-top: 0;">Q22: Setting up testing in React Native projects.</h3>

  <p>Every React Native project should have a solid testing foundation:</p>

  <div class="code-example">Testing Setup</div>
  <pre class="code-block"><code class="language-javascript">// 1. Jest configuration (already included with RN)
{
  "jest": {
    "preset": "react-native",
    "setupFilesAfterEnv": ["&lt;rootDir&gt;/jest.setup.js"],
    "testMatch": [
      "**/__tests__/**/*.test.js",
      "**/?(*.)+(spec|test).js"
    ],
    "collectCoverageFrom": [
      "**/*.{js,jsx,ts,tsx}",
      "!**/node_modules/**",
      "!**/coverage/**"
    ]
  }
}

// 2. Jest setup file
import 'react-native-gesture-handler/jestSetup';
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';

// Mock native modules
jest.mock('react-native-reanimated', () => require('react-native-reanimated/mock'));

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);

// Mock react-native-safe-area-context
jest.mock('react-native-safe-area-context', () => ({
  SafeAreaProvider: ({ children }) => children,
  SafeAreaView: ({ children }) => children,
  useSafeAreaInsets: () => ({ top: 0, bottom: 0, left: 0, right: 0 }),
}));

// Silence console warnings during tests
global.console = {
  ...console,
  warn: jest.fn(),
  error: jest.fn(),
};

// 3. Basic test structure
describe('ComponentName', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders correctly', () => {
    // Test implementation
  });

  it('handles user interactions', () => {
    // Test implementation
  });
});</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always mock native modules and async operations in tests. React Native's Jest preset handles most mocking, but you'll need to mock libraries like AsyncStorage, navigation, and third-party native modules.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">UNIT TESTING</span>
  <h3 style="margin-top: 0;">Q23: Writing effective unit tests for React Native components.</h3>

  <p>Unit tests focus on testing individual components in isolation:</p>

  <div class="code-example">Unit Testing Components</div>
  <pre class="code-block"><code class="language-javascript">import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { TextInput } from 'react-native';

// Component to test
function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSubmit = async () => {
    setLoading(true);
    try {
      await onSubmit({ email, password });
    } finally {
      setLoading(false);
    }
  };

  return (
    &lt;View&gt;
      &lt;TextInput
        testID="email-input"
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
      /&gt;
      &lt;TextInput
        testID="password-input"
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      /&gt;
      &lt;TouchableOpacity
        testID="submit-button"
        onPress={handleSubmit}
        disabled={loading}
      >
        &lt;Text>{loading ? 'Loading...' : 'Login'}</Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// Unit tests
describe('LoginForm', () => {
  const mockOnSubmit = jest.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  it('renders all form elements', () => {
    const { getByTestId, getByPlaceholderText } = render(
      &lt;LoginForm onSubmit={mockOnSubmit} /&gt;
    );

    expect(getByTestId('email-input')).toBeTruthy();
    expect(getByTestId('password-input')).toBeTruthy();
    expect(getByTestId('submit-button')).toBeTruthy();
  });

  it('updates email input value', () => {
    const { getByTestId } = render(&lt;LoginForm onSubmit={mockOnSubmit} /&gt;);

    const emailInput = getByTestId('email-input');
    fireEvent.changeText(emailInput, 'test@example.com');

    expect(emailInput.props.value).toBe('test@example.com');
  });

  it('calls onSubmit with form data', async () => {
    const { getByTestId } = render(&lt;LoginForm onSubmit={mockOnSubmit} /&gt;);

    const emailInput = getByTestId('email-input');
    const passwordInput = getByTestId('password-input');
    const submitButton = getByTestId('submit-button');

    fireEvent.changeText(emailInput, 'user@example.com');
    fireEvent.changeText(passwordInput, 'password123');
    fireEvent.press(submitButton);

    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        email: 'user@example.com',
        password: 'password123',
      });
    });
  });

  it('shows loading state during submission', async () => {
    const slowSubmit = jest.fn(() => new Promise(resolve => setTimeout(resolve, 100)));

    const { getByTestId, getByText } = render(
      &lt;LoginForm onSubmit={slowSubmit} /&gt;
    );

    const submitButton = getByTestId('submit-button');
    fireEvent.press(submitButton);

    expect(getByText('Loading...')).toBeTruthy();

    await waitFor(() => {
      expect(getByText('Login')).toBeTruthy();
    });
  });
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">HOOK TESTING</span>
  <h3 style="margin-top: 0;">Q24: Testing custom hooks and async operations.</h3>

  <p>Custom hooks need special testing approaches:</p>

  <div class="code-example">Testing Custom Hooks</div>
  <pre class="code-block"><code class="language-javascript">import { renderHook, act, waitFor } from '@testing-library/react';

// Hook to test
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);

  return { count, increment, decrement, reset };
}

// Hook tests
describe('useCounter', () => {
  it('starts with initial value', () => {
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
    const { result } = renderHook(() => useCounter(2));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(1);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));

    act(() => {
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });
});

// Testing async hooks
function useUserData(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    if (userId) {
      fetchUser();
    }
  }, [userId]);

  return { user, loading, error };
}

// Async hook tests
describe('useUserData', () => {
  beforeEach(() => {
    global.fetch = jest.fn();
  });

  it('fetches user data successfully', async () => {
    const mockUser = { id: 1, name: 'John' };
    global.fetch.mockResolvedValueOnce({
      json: () => Promise.resolve(mockUser),
    });

    const { result } = renderHook(() => useUserData(1));

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.user).toEqual(mockUser);
    expect(result.current.error).toBe(null);
  });

  it('handles fetch errors', async () => {
    global.fetch.mockRejectedValueOnce(new Error('Network error'));

    const { result } = renderHook(() => useUserData(1));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.user).toBe(null);
    expect(result.current.error).toBe('Network error');
  });
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">INTEGRATION TESTING</span>
  <h3 style="margin-top: 0;">Q25: Integration testing with React Native Testing Library.</h3>

  <p>Integration tests verify that components work together correctly:</p>

  <div class="code-example">Integration Testing</div>
  <pre class="code-block"><code class="language-javascript">import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';

// Components to test together
function UserProfile({ navigation }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Simulate API call
    setTimeout(() => {
      setUser({ id: 1, name: 'John Doe', email: 'john@example.com' });
    }, 100);
  }, []);

  if (!user) {
    return &lt;Text&gt;Loading...&lt;/Text&gt;
  }

  return (
    &lt;View&gt;
      &lt;Text testID="user-name">{user.name}&lt;/Text&gt;
      &lt;Text testID="user-email">{user.email}&lt;/Text&gt;
      &lt;TouchableOpacity
        testID="edit-button"
        onPress={() => navigation.navigate('EditProfile', { userId: user.id })}
      >
        &lt;Text&gt;Edit Profile&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// Integration test
describe('UserProfile Integration', () => {
  const mockNavigate = jest.fn();

  beforeEach(() => {
    mockNavigate.mockClear();
  });

  it('displays loading then user data', async () => {
    const { getByText, getByTestId } = render(
      &lt;UserProfile navigation={{ navigate: mockNavigate }} /&gt;
    );

    // Initially shows loading
    expect(getByText('Loading...')).toBeTruthy();

    // After data loads
    await waitFor(() => {
      expect(getByTestId('user-name')).toHaveTextContent('John Doe');
      expect(getByTestId('user-email')).toHaveTextContent('john@example.com');
    });
  });

  it('navigates to edit screen when edit button pressed', async () => {
    const { getByTestId } = render(
      &lt;UserProfile navigation={{ navigate: mockNavigate }} /&gt;
    );

    await waitFor(() => {
      expect(getByTestId('edit-button')).toBeTruthy();
    });

    fireEvent.press(getByTestId('edit-button'));

    expect(mockNavigate).toHaveBeenCalledWith('EditProfile', { userId: 1 });
  });
});

// Testing with navigation context
function TestWrapper({ children }) {
  return (
    &lt;NavigationContainer&gt;
      {children}
    &lt;/NavigationContainer&gt;
  );
}

describe('Navigation Integration', () => {
  it('handles navigation between screens', async () => {
    const { getByText } = render(
      &lt;TestWrapper&gt;
        &lt;Stack.Navigator&gt;
          &lt;Stack.Screen name="Home" component={HomeScreen} /&gt;
          &lt;Stack.Screen name="Profile" component={UserProfile} /&gt;
        &lt;/Stack.Navigator&gt;
      &lt;/TestWrapper&gt;
    );

    // Test navigation flow
    fireEvent.press(getByText('Go to Profile'));
    await waitFor(() => {
      expect(getByText('John Doe')).toBeTruthy();
    });
  });
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">MOCKING STRATEGIES</span>
  <h3 style="margin-top: 0;">Q26: Effective mocking strategies for React Native tests.</h3>

  <p>Proper mocking is essential for reliable and fast tests:</p>

  <div class="code-example">Mocking Strategies</div>
  <pre class="code-block"><code class="language-javascript">// 1. API mocking with jest.mock
import axios from 'axios';

jest.mock('axios');
const mockedAxios = axios;

describe('API Service', () => {
  beforeEach(() => {
    mockedAxios.get.mockClear();
    mockedAxios.post.mockClear();
  });

  it('fetches user data', async () => {
    const mockUser = { id: 1, name: 'John' };
    mockedAxios.get.mockResolvedValueOnce({ data: mockUser });

    const result = await fetchUser(1);

    expect(mockedAxios.get).toHaveBeenCalledWith('/api/users/1');
    expect(result).toEqual(mockUser);
  });
});

// 2. Mocking navigation
const mockNavigate = jest.fn();

jest.mock('@react-navigation/native', () => ({
  useNavigation: () => ({
    navigate: mockNavigate,
    goBack: jest.fn(),
  }),
}));

// 3. Mocking AsyncStorage
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';

jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);

// Usage in tests
describe('Data Persistence', () => {
  beforeEach(async () => {
    await AsyncStorage.clear();
  });

  it('saves and retrieves data', async () => {
    const testData = { userId: 123 };

    await saveUserData(testData);
    const retrieved = await getUserData();

    expect(retrieved).toEqual(testData);
  });
});

// 4. Mocking timers
describe('Timer Component', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.runOnlyPendingTimers();
    jest.useRealTimers();
  });

  it('updates after delay', () => {
    const { getByText } = render(&lt;TimerComponent /&gt;);

    expect(getByText('0 seconds')).toBeTruthy();

    // Fast-forward time
    jest.advanceTimersByTime(1000);

    expect(getByText('1 seconds')).toBeTruthy();
  });
});

// 5. Mocking device features
jest.mock('react-native-device-info', () => ({
  getVersion: jest.fn(() => '1.0.0'),
  getBuildNumber: jest.fn(() => '1'),
  getDeviceId: jest.fn(() => 'test-device-id'),
}));

// 6. Partial mocking (mock only specific methods)
const originalModule = jest.requireActual('react-native-permissions');

jest.mock('react-native-permissions', () => ({
  ...originalModule,
  request: jest.fn(() => Promise.resolve('granted')),
  check: jest.fn(() => Promise.resolve('granted')),
}));</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">E2E TESTING</span>
  <h3 style="margin-top: 0;">Q27: End-to-end testing with Detox.</h3>

  <p>E2E tests simulate real user interactions on actual devices:</p>

  <div class="code-example">Detox E2E Testing</div>
  <pre class="code-block"><code class="language-javascript">// detox.config.js
module.exports = {
  testRunner: {
    args: {
      $0: 'jest',
      config: 'e2e/jest.config.js',
    },
  },
  configurations: {
    ios: {
      type: 'ios.simulator',
      device: {
        type: 'iPhone 13',
      },
      app: 'ios',
    },
    android: {
      type: 'android.emulator',
      device: {
        avdName: 'Pixel_3a_API_30',
      },
      app: 'android',
    },
  },
};

// e2e/firstTest.e2e.js
describe('Login Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should login successfully', async () => {
    await element(by.id('email-input')).typeText('user@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();

    await expect(element(by.text('Welcome!'))).toBeVisible();
  });

  it('should show error for invalid credentials', async () => {
    await element(by.id('email-input')).typeText('invalid@email.com');
    await element(by.id('password-input')).typeText('wrongpassword');
    await element(by.id('login-button')).tap();

    await expect(element(by.text('Invalid credentials'))).toBeVisible();
  });

  it('should navigate between screens', async () => {
    // Login first
    await element(by.id('email-input')).typeText('user@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();

    // Navigate to profile
    await element(by.id('profile-tab')).tap();
    await expect(element(by.id('profile-screen'))).toBeVisible();

    // Go back
    await element(by.id('back-button')).tap();
    await expect(element(by.id('home-screen'))).toBeVisible();
  });
});

// Testing gestures
describe('Swipe Gestures', () => {
  it('should swipe to delete item', async () => {
    await element(by.id('todo-item')).swipe('left');
    await expect(element(by.text('Delete'))).toBeVisible();

    await element(by.text('Delete')).tap();
    await expect(element(by.id('todo-item'))).not.toBeVisible();
  });

  it('should handle pull to refresh', async () => {
    await element(by.id('todo-list')).swipe('down', 'fast', 0.5);
    await expect(element(by.id('refresh-indicator'))).toBeVisible();

    // Wait for refresh to complete
    await waitFor(element(by.id('todo-list'))).toBeVisible().withTimeout(5000);
  });
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">TESTING UTILITIES</span>
  <h3 style="margin-top: 0;">Q28: Testing utilities and best practices.</h3>

  <p>Helper functions and patterns that make testing easier:</p>

  <div class="code-example">Testing Utilities</div>
  <pre class="code-block"><code class="language-javascript">// 1. Custom render function with providers
import { render } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';

function renderWithNavigation(component) {
  return render(
    &lt;NavigationContainer&gt;
      {component}
    &lt;/NavigationContainer&gt;
  );
}

function renderWithProviders(component) {
  return render(
    &lt;ThemeProvider&gt;
      &lt;AuthProvider&gt;
        &lt;NavigationContainer&gt;
          {component}
        &lt;/NavigationContainer&gt;
      &lt;/AuthProvider&gt;
    &lt;/ThemeProvider&gt;
  );
}

// 2. Test data factories
function createMockUser(overrides = {}) {
  return {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'https://example.com/avatar.jpg',
    ...overrides,
  };
}

function createMockTodo(overrides = {}) {
  return {
    id: 1,
    text: 'Buy groceries',
    completed: false,
    createdAt: new Date(),
    ...overrides,
  };
}

// 3. Custom matchers
expect.extend({
  toBeVisible(received) {
    const pass = received.props.style?.opacity !== 0 &&
                 received.props.style?.display !== 'none';
    return {
      message: () => `expected element to ${pass ? 'not ' : ''}be visible`,
      pass,
    };
  },
});

// Usage
expect(component).toBeVisible();

// 4. Async utilities
async function waitForLoadingToComplete() {
  await waitFor(() => {
    expect(screen.queryByText('Loading...')).not.toBeTruthy();
  });
}

async function fillAndSubmitForm(formData) {
  const { email, password } = formData;

  fireEvent.changeText(screen.getByTestId('email-input'), email);
  fireEvent.changeText(screen.getByTestId('password-input'), password);
  fireEvent.press(screen.getByTestId('submit-button'));

  await waitForLoadingToComplete();
}

// 5. Snapshot testing utilities
import 'react-native-testing-library/jest-native';

describe('Button Snapshots', () => {
  it('renders primary button correctly', () => {
    const { toJSON } = render(&lt;Button variant="primary"&gt;Click me&lt;/Button&gt;);
    expect(toJSON()).toMatchSnapshot();
  });

  it('renders disabled button correctly', () => {
    const { toJSON } = render(
      &lt;Button variant="primary" disabled&gt;Click me&lt;/Button&gt;
    );
    expect(toJSON()).toMatchSnapshot();
  });
});

// 6. Accessibility testing
describe('Accessibility Tests', () => {
  it('has proper accessibility labels', () => {
    const { getByA11yLabel } = render(&lt;Button&gt;Save&lt;/Button&gt;);

    expect(getByA11yLabel('Save button')).toBeTruthy();
  });

  it('supports screen reader', () => {
    const { getByA11yRole } = render(&lt;Button&gt;Save&lt;/Button&gt;);

    const button = getByA11yRole('button');
    expect(button).toBeTruthy();
    expect(button).toHaveProp('accessible', true);
  });
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">CI/CD TESTING</span>
  <h3 style="margin-top: 0;">Q29: Setting up automated testing in CI/CD pipelines.</h3>

  <p>Automate testing to catch issues before they reach production:</p>

  <div class="code-example">CI/CD Testing Setup</div>
  <pre class="code-block"><code class="language-javascript">// GitHub Actions workflow
// .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run type checking
      run: npm run type-check

    - name: Run unit tests
      run: npm run test:unit

    - name: Run integration tests
      run: npm run test:integration

    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  e2e-test:
    runs-on: macos-latest
    needs: test

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x

    - name: Install dependencies
      run: npm ci

    - name: Install pods
      run: cd ios && pod install

    - name: Build iOS
      run: npm run build:ios

    - name: Run E2E tests
      run: npm run test:e2e:ios

  # Parallel Android E2E tests
  android-e2e:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x

    - name: Install dependencies
      run: npm ci

    - name: Setup Android SDK
      uses: android-actions/setup-android@v2

    - name: Run Android E2E tests
      uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: 29
        script: npm run test:e2e:android

// Jest configuration for CI
{
  "jest": {
    "preset": "react-native",
    "testEnvironment": "jsdom",
    "collectCoverageFrom": [
      "**/*.{ts,tsx}",
      "!**/*.d.ts",
      "!**/node_modules/**",
      "!**/coverage/**"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    },
    "testTimeout": 10000
  }
}

// Performance testing
describe('Performance Tests', () => {
  it('renders list within time limit', async () => {
    const startTime = performance.now();

    const { getByTestId } = render(&lt;LargeList /&gt;);

    // Wait for list to render
    await waitFor(() => {
      expect(getByTestId('list')).toBeTruthy();
    });

    const endTime = performance.now();
    const renderTime = endTime - startTime;

    // Assert render time is acceptable
    expect(renderTime).toBeLessThan(100); // Less than 100ms
  });
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">TESTING BEST PRACTICES</span>
  <h3 style="margin-top: 0;">Q30: Testing best practices and common pitfalls.</h3>

  <p>Avoid these common testing mistakes and follow best practices:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin: 24px 0;">
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Testing Implementation</h4>
      <p style="margin: 0; font-size: 0.875rem;">Testing how code works instead of what it does.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Brittle Tests</h4>
      <p style="margin: 0; font-size: 0.875rem;">Tests that break when implementation changes.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Testing Libraries</h4>
      <p style="margin: 0; font-size: 0.875rem;">Testing third-party code instead of your logic.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Slow Tests</h4>
      <p style="margin: 0; font-size: 0.875rem;">Tests that take too long to run.</p>
    </div>
  </div>

  <div class="code-example">Best Practices</div>
  <pre class="code-block"><code class="language-javascript">// ✅ Test behavior, not implementation
describe('TodoApp', () => {
  it('adds a new todo when form is submitted', () => {
    // Test what the user sees and can do
    // Don't test internal state or implementation details
  });
});

// ✅ Use descriptive test names
describe('User authentication', () => {
  it('redirects to dashboard after successful login', () => {
    // Clear, descriptive test name
  });

  it('shows error message for invalid credentials', () => {
    // Clear, descriptive test name
  });
});

// ✅ Test edge cases and error conditions
describe('API Error Handling', () => {
  it('shows retry button when network fails', () => {
    // Test error states
  });

  it('handles malformed API responses gracefully', () => {
    // Test unexpected data
  });
});

// ✅ Use test utilities and helpers
function loginUser(user = createMockUser()) {
  // Helper function for common setup
  const utils = render(&lt;LoginForm /&gt;);

  fireEvent.changeText(utils.getByTestId('email'), user.email);
  fireEvent.changeText(utils.getByTestId('password'), user.password);
  fireEvent.press(utils.getByTestId('submit'));

  return utils;
}

describe('User login flow', () => {
  it('navigates to dashboard on success', async () => {
    const mockUser = createMockUser({ email: 'test@example.com' });

    // Mock API success
    jest.spyOn(api, 'login').mockResolvedValue(mockUser);

    const { findByText } = loginUser(mockUser);

    await expect(findByText('Welcome to Dashboard')).toBeTruthy();
  });
});

// ✅ Organize tests by feature, not by component
// 📁 __tests__/features/
├── authentication/
│   ├── login.test.js
│   ├── registration.test.js
│   └── password-reset.test.js
├── todo-management/
│   ├── creating-todos.test.js
│   ├── completing-todos.test.js
│   └── deleting-todos.test.js
└── navigation/
    ├── tab-navigation.test.js
    └── deep-linking.test.js

// ✅ Use meaningful assertions
it('displays user name after login', async () => {
  // ✅ Good: Tests user-visible behavior
  await expect(screen.findByText(`Welcome, ${user.name}`)).toBeVisible();
});

it('calls onUserLogin prop with correct data', () => {
  // ❌ Bad: Tests implementation details
  expect(mockOnUserLogin).toHaveBeenCalledWith(userData);
});

// ✅ Keep tests fast and reliable
describe('Fast Tests', () => {
  // Mock async operations
  beforeEach(() => {
    jest.spyOn(global, 'fetch').mockResolvedValue({
      json: () => Promise.resolve(mockData),
    });
  });

  it('loads data quickly', async () => {
    const start = Date.now();
    render(&lt;DataComponent /&gt;);
    await screen.findByText('Data loaded');
    const duration = Date.now() - start;

    expect(duration).toBeLessThan(100); // Fast!
  });
});</code></pre>
</div>

---

<div class="nav-footer">
  <a href="phase8-animations-gestures.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 08: Animations</a>
  <a href="phase10-security-best-practices.md" style="color: #64748b; text-decoration: none; font-weight: 600;">Phase 10: Security ➡️</a>
</div>

</div>