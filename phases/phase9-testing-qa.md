# Phase 09: Testing & Quality Assurance

Master testing strategies for React Native apps from basic unit tests to advanced E2E automation. Learn to write reliable tests, debug issues, and maintain quality throughout the development lifecycle.

---

### Q1: [STRATEGY] The Testing Pyramid vs. The Testing Trophy
*"How do you balance Unit, Integration, and E2E tests in a React Native project?"*

A senior strategy often shifts from the traditional **Testing Pyramid** to the **Testing Trophy**, which prioritizes **Integration Tests**. In React Native, this means testing how components interact with their state, navigation, and API clients using RNTL.

- **Unit Tests:** Pure functions, utilities, and Redux/Zustand logic (Jest).
- **Integration Tests:** Screen-level flows, complex hooks, and component trees (RNTL + MSW).
- **E2E Tests:** High-value user journeys on real devices/simulators (Detox/Maestro).
- **Static:** Catching typos and type errors (TypeScript/ESLint).

> [!TIP]
> **Senior Insight: The "Confidence per Minute" Metric**
> Integration tests provide the highest ROI. They give you confidence that your feature works without the massive speed and maintenance cost of E2E tests. A senior dev will spend 60% of their testing time here.

> [!WARNING]
> **Follow-up Trap:** "Is 100% test coverage a good goal?"
> **Answer:** No. It leads to testing implementation details rather than behavior. Focus on **Critical Path Coverage** (e.g., checkout, login, data persistence).

---

### Q2: [UNIT TESTING] Testing Custom Hooks with Asynchronous State
*"How do you test a custom hook that involves asynchronous state updates or API calls?"*

Use `renderHook` from `@testing-library/react-native`. The key is to handle the "wait" for the state to update after an async action.

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

> [!TIP]
> **Senior Insight: The "act()" warning**
> If you see "An update to a component... was not wrapped in act()", it usually means your test finished before an asynchronous state update occurred. `waitFor` internally wraps its checks in `act`, making it the preferred way to solve this.

---

### Q3: [COMPONENT TESTING] RNTL: User-Centric Queries vs. TestID
*"Why does RNTL discourage the use of testID, and when is it acceptable to use it?"*

Queries like `getByRole`, `getByLabelText`, and `getByText` mirror how a user (and a screen reader) interacts with the app. If you change a `TouchableOpacity` to a `Pressable`, a role-based query will still work, but an implementation-based one might fail.

**Acceptable testID use cases:**
- Non-interactive decorative elements that need to be verified.
- Complex SVG charts where text-based querying is impossible.
- E2E testing (Detox) where native selectors are required.

> [!WARNING]
> **Follow-up Trap:** "How do you test a component that is hidden behind a conditional?"
> **Answer:** Use `queryByText` instead of `getByText`. `getBy` throws an error if the element is missing, while `queryBy` returns `null`, allowing you to `expect(...).toBeNull()`.

---

### Q4: [E2E TESTING] Detox: Synchronization & Idleness
*"How does Detox's synchronization mechanism work, and how do you handle 'stuck' tests?"*

Detox is a **Gray-box** runner. It monitors the app's internal resources: the main thread, the JS thread, the network (OkHttp/NSURLSession), and timers. It only executes the next test step when the app is "Idle."

> [!TIP]
> **Senior Insight: The "Infinity Timer" Problem**
> If your app has a looping animation or a polling network request, Detox will never see the app as "Idle" and the test will timeout. Use `device.disableSynchronization()` for those specific steps, or better yet, mock the polling interval to be very large during tests.

---

### Q5: [MOCKING] Advanced Native Module Mocking
*"How do you mock a native module that uses an EventEmitter (e.g., NetInfo or Keyboard)?"*

You must mock both the static methods and the listener registration system.

```javascript
jest.mock('@react-native-community/netinfo', () => ({
  addEventListener: jest.fn(),
  fetch: jest.fn(() => Promise.resolve({ isConnected: true })),
}));
```

> [!WARNING]
> **Follow-up Trap:** "What is the danger of mocking everything?"
> **Answer:** Your tests might pass while the real app fails because the mock's behavior has diverged from the actual native implementation. Periodically verify mocks against real hardware.

---

### Q6: [STABILITY] Snapshots vs. Visual Regression
*"Compare Jest Snapshots with Visual Regression tools like Loki or Applitools."*

- **Jest Snapshots:** Compare the **rendered JSON tree**. Good for catching logic changes in props/state. Useless for catching CSS/Layout bugs (e.g., a button hidden behind another view).
- **Visual Regression:** Compare **actual pixel screenshots**. Catches layout shifts, font-rendering issues, and platform-specific styling bugs.

> [!TIP]
> **Senior Insight: The Snapshot Burden**
> Seniors avoid giant snapshots of whole screens. They are hard to review and often get updated blindly. Prefer small, focused snapshots of specific component states (e.g., "Error State Snapshot").

---

### Q7: [MONITORING] Robust Error Boundaries
*"Beyond showing a fallback UI, what should a senior-level Error Boundary do?"*

It should act as a **diagnostic hub**:
- Capture the component stack trace (via `componentDidCatch`).
- Log the error to a service (Sentry/Bugsnag) with **user context**.
- Provide a "Hard Reset" that clears `AsyncStorage`/`MMKV` if the crash is caused by corrupted local data.

```typescript
componentDidCatch(error: Error, info: { componentStack: string }) {
  Sentry.captureException(error, { extra: info });
}
```

---

### Q8: [OBSERVABILITY] Sentry: Source Maps & Breadcrumbs
*"Why are Source Maps critical for React Native production monitoring?"*

In production, JS is minified and bundled. Without Source Maps, a Sentry stack trace will just point to `index.bundle:1:59283`. Source Maps allow Sentry to translate that back to your actual TypeScript file and line number.

> [!TIP]
> **Senior Insight: Custom Breadcrumbs**
> Don't just rely on automatic breadcrumbs. Manually add breadcrumbs for "Business Events" (e.g., `Sentry.addBreadcrumb({ category: 'auth', message: 'User clicked Buy Button' })`). This makes debugging logic-heavy crashes much easier.

---

### Q9: [DEBUGGING] Flipper vs. React Inspector
*"How do you debug a performance issue that only happens on Android?"*

Use the **Flipper "Profiler"**. Unlike the React Inspector which shows component trees, the Profiler shows **native frame drops**. You can see if the bottleneck is on the UI thread (layout thrashing) or the JS thread (heavy computation).

> [!WARNING]
> **Follow-up Trap:** "Can you use Flipper in a production build?"
> **Answer:** No. Flipper requires the `flipper-plugin` native code which is stripped out in release builds for security and size. Use Sentry's "Performance" tab for production profiling.

---

### Q10: [PERFORMANCE] Hermes Sampling Profiler
*"What is the difference between 'Sampling' and 'Tracing' profilers?"*

- **Sampling (Hermes):** Checks the call stack every few milliseconds. Low overhead, good for finding "Hot Paths" without slowing down the app.
- **Tracing:** Records every single function call and its duration. High overhead, can actually cause jank while profiling, but gives 100% accuracy for short bursts.

---

### Q11: [API MOCKING] MSW (Mock Service Worker)
*"How do you implement MSW to test a complex Redux Toolkit Query flow?"*

MSW intercepts the actual network request. This means your RTK Query hooks think they are talking to a real server, testing your `transformResponse`, `providesTags`, and error-handling logic.

```typescript
export const handlers = [
  rest.get('/api/user', (req, res, ctx) => {
    return res(ctx.status(200), ctx.json({ name: 'John' }));
  }),
];
```

---

### Q12: [CI/CD] CI Optimization: Caching & Parallelization
*"How do you reduce Detox CI time from 40 minutes to 10 minutes?"*

- **Build Caching:** Cache the `ios/build` and `android/app/build` folders using GitHub Actions cache.
- **Sharding:** Split your test files and run them in parallel on multiple runners.
- **Pre-built Binaries:** Build the app once, upload the `.app` or `.apk` as an artifact, and have the test runners download it instead of re-building.

---

### Q13: [USER INTERACTION] Testing Complex Gestures
*"How do you simulate a multi-touch 'Pinch' gesture in an integration test?"*

Standard RNTL `fireEvent.press` isn't enough. You must use `fireEvent` with the specific event data expected by `react-native-gesture-handler`, simulating the `onGestureEvent` with multiple pointers and scale values.

> [!TIP]
> **Senior Insight: Manual Mocks for RNGH**
> Since simulating real touches is hard, seniors often mock the `GestureDetector` to immediately trigger the `onEnd` or `onUpdate` callback with fake data to verify the resulting state change.

---

### Q14: [ISOLATION] Storybook Interaction Testing
*"How do you combine Storybook with Jest to prevent 'Visual Drift'?"*

Use **Storybook Interaction Tests** (via `@storybook/addon-interactions`). It allows you to write "play" functions that simulate user clicks inside the Storybook UI. You can then run these same stories as Jest tests using `@storybook/testing-react`.

---

### Q15: [ACCESSIBILITY] Accessibility-First Testing
*"How do you verify that a custom 'CheckBox' is accessible to screen readers?"*

Your test should look for the `checkbox` role and the `checked` state, rather than just checking if an icon is rendered.

```typescript
const checkbox = screen.getByRole('checkbox', { name: /accept terms/i });
expect(checkbox).toHaveProp('accessibilityState', { checked: true });
```

---

### Q16: [SECURITY] Dependency Scanning
*"What is 'Software Composition Analysis' (SCA) and why is it part of RN QA?"*

SCA tools (like **Snyk** or **GitHub Dependabot**) scan your `package.json` and `yarn.lock` for libraries with known security vulnerabilities. In RN, this is critical because a vulnerable native library can expose the entire device's keychain or filesystem.

---

### Q17: [CODE QUALITY] ESLint for React Native Performance
*"Which ESLint rules are specific to React Native performance and quality?"*

- `react-native/no-inline-styles`: Prevents object creation on every render.
- `react-hooks/exhaustive-deps`: Critical for preventing infinite loops in `useEffect`.
- `react-native/no-raw-text`: Ensures all text is wrapped in a `<Text>` component (which would crash on Android otherwise).

---

### Q18: [MOCK DATA] The Factory Pattern with Fishery
*"Why is the 'Factory Pattern' better than 'Fixture Files' for test data?"*

Fixture files (giant JSONs) are rigid. If you need a user with a slightly different email, you have to create a new file or mutate the object. **Factories** allow for dynamic, type-safe generation.

```typescript
const userFactory = Factory.define<User>(({ sequence }) => ({
  id: sequence,
  name: 'Test User',
  email: `user${sequence}@example.com`,
}));

const admin = userFactory.build({ role: 'admin' });
```

---

### Q19: [RELIABILITY] Flakiness: Network vs. Timers
*"How do you debug a Detox test that passes locally but fails 20% of the time in CI?"*

- **Network Flakiness:** Use MSW to remove real network dependency.
- **Resource Contention:** CI runners are often slower. Increase `interactTimeout` in Detox.
- **Artifact Inspection:** Configure CI to record a video of the failed test. Often, a "Permission Popup" or "System Update" is blocking the UI.

---

### Q20: [AUTOMATION] Visual Regression: Percy & Loki
*"How do you handle 'False Positives' in Visual Regression (e.g., a flashing cursor or a timestamp)?"*

Use "Ignore Regions." Tools like **Percy** or **Applitools** allow you to draw boxes over dynamic areas (like clocks or random IDs) that should be ignored during the pixel-comparison process.

---

### Q21: [QUALITY METRICS] Beyond Coverage: Lighthouse for Mobile?
*"How do you measure 'Quality' beyond just test pass rates?"*

Use **FlashList's 'onBlankArea'** to measure list performance, **Sentry's User Misery Score** to measure how many users experience slow loads, and **Play Store/App Store Vitals** (ANR rates, Crash rates) as the ultimate QA metrics.

---

### Q22: [BASICS] Jest Configuration & Mocking Native Modules
*"What is the standard way to mock native modules like AsyncStorage or Reanimated in Jest?"*

Most popular libraries provide their own mocks. You should import these in your `jest.setup.js` file to ensure they are available for all tests.

```javascript
// jest.setup.js
import 'react-native-gesture-handler/jestSetup';
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';

// Mock Reanimated
jest.mock('react-native-reanimated', () => require('react-native-reanimated/mock'));

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);
```

---

### Q23: [UNIT TESTING] Writing Effective Component Tests
*"How do you test a component that has multiple states (loading, error, data)?"*

Use `render` and `fireEvent` from RNTL. For async states, use `waitFor` to assert that the UI eventually reaches the expected state.

```typescript
it('shows loading state during submission', async () => {
  const slowSubmit = jest.fn(() => new Promise(resolve => setTimeout(resolve, 100)));
  const { getByTestId, getByText } = render(<LoginForm onSubmit={slowSubmit} />);

  const submitButton = getByTestId('submit-button');
  fireEvent.press(submitButton);

  expect(getByText('Loading...')).toBeTruthy();

  await waitFor(() => {
    expect(getByText('Login')).toBeTruthy();
  });
});
```

---

### Q24: [HOOK TESTING] Testing Custom Hooks and Async Operations
*"What is the difference between `act()` and `waitFor()` when testing hooks?"*

- **act():** Used to wrap synchronous updates (like calling a function that updates state). It ensures all updates are processed before the next assertion.
- **waitFor():** Used to wait for asynchronous updates (like an API call finishing). It repeatedly runs the expectation until it passes or times out.

---

### Q25: [INTEGRATION TESTING] Navigation & Context Integration
*"How do you test a component that depends on React Navigation or a Theme Context?"*

Wrap your component in the necessary providers (e.g., `NavigationContainer` or `ThemeProvider`) inside the `render` function.

```typescript
function TestWrapper({ children }) {
  return (
    <NavigationContainer>
      <ThemeProvider>{children}</ThemeProvider>
    </NavigationContainer>
  );
}
```

---

### Q26: [MOCKING STRATEGIES] Partial vs. Full Mocking
*"When should you use `jest.requireActual()` instead of just mocking the whole module?"*

Use `jest.requireActual()` when you want to mock only **specific functions** of a library while keeping the rest of the library's original functionality intact. This is common for libraries like `react-native-permissions`.

---

### Q27: [E2E TESTING] Detox: Swipe & Gesture Simulation
*"How do you test a 'Pull to Refresh' gesture in Detox?"*

Use the `swipe` method on the list element.

```javascript
it('should handle pull to refresh', async () => {
  await element(by.id('todo-list')).swipe('down', 'fast', 0.5);
  await expect(element(by.id('refresh-indicator'))).toBeVisible();
});
```

---

### Q28: [TESTING UTILITIES] Custom Render Functions
*"Why is it better to create a `renderWithProviders` helper instead of wrapping every test manually?"*

It reduces boilerplate and ensures that if you add a new provider (like a new Redux store or a new Context), you only have to update it in **one place** rather than in hundreds of test files.

---

### Q29: [CI/CD TESTING] Performance Testing in CI
*"How can you prevent performance regressions in your CI/CD pipeline?"*

1. **Render Time Checks:** Assert that critical components render within a certain threshold (e.g., < 100ms).
2. **FlashList Monitoring:** Use `onBlankArea` to log and fail builds if list performance drops.
3. **Bundle Size Checks:** Use tools like `react-native-bundle-visualizer` to fail the build if the JS bundle grows unexpectedly.

---

### Q30: [BEST PRACTICES] Common Pitfalls & Senior Advice
*"What is the single biggest mistake junior developers make when writing tests?"*

**Testing implementation details.** Juniors often test internal state or private methods. If you rename a variable but the UI still works, the test should still pass. If it fails, you are testing implementation, not behavior.

> [!TIP]
> **Senior Insight: Delete Before You Refactor**
> If you are refactoring a complex component and your tests are making it harder rather than easier, don't be afraid to delete the implementation-heavy tests and replace them with a few solid integration tests that verify the user's end goal.

---

[⬅️ Phase 08: Animations](phase8-animations-gestures.md) | [Phase 10: Security ➡️](phase10-security-best-practices.md)
