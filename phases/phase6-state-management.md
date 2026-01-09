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
    background: linear-gradient(135deg, #4f46e5 0%, #3730a3 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(79, 70, 229, 0.2);
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
    background: #e0e7ff;
    color: #3730a3;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #f5f3ff;
    border-left: 4px solid #8b5cf6;
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
<span class="phase-number">Phase 06</span>
<h1 class="phase-title">State Management Strategy</h1>
<p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Architecting predictable and performant data flows.</p>
</div>

---

### 📋 Phase Overview
Master state management from basic React state to complex global architectures. Learn when to use Context, Zustand, Redux Toolkit, and advanced patterns for scalable React Native applications.

---

<div class="qa-card">
<span class="question-tag">ARCHITECTURE</span>

### 6.1 Context vs Redux vs Zustand

**Question:** *"When should you avoid using React Context for global state in a high-performance React Native app?"*

**React Context** is built for static or low-frequency updates (like themes or auth user objects). It is NOT a state management tool; it's a dependency injection tool.

**The Problem:** When a value in a Context Provider changes, **every component consuming that context re-renders**. In React Native, where UI thread performance is critical, frequent updates (like a timer or location coordinates) in Context can cause significant frame drops.

<div class="senior-insight">

**💡 Senior Insight: The "Selector" Missing Link**
Unlike Redux or Zustand, Context lacks built-in **selectors**. You cannot "subscribe" to only a piece of the context value. To optimize Context, you'd need to wrap components in `React.memo` or split contexts into many smaller providers, which leads to "Provider Hell."
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "Can we use `useMemo` inside the Provider to stop re-renders?"
**Answer:** No. While `useMemo` can prevent the *value object* from changing, any component calling `useContext(MyContext)` will still re-render if the value changes at all. The only real fix for Context performance is **context splitting** or moving to a library with external stores (Zustand/Redux).
</div>
</div>

<div class="qa-card">
<span class="question-tag">CACHING</span>

### 6.2 Server State vs Client State

**Question:** *"Why is moving API data out of Redux/Zustand into React Query (TanStack Query) considered a Senior architectural move?"*

Traditional state management treats API data as "Global State," leading to massive boilerplate for `loading`, `error`, and `caching` logic.

**Specialized Server State (React Query/RTK Query) provides:**

- **Automatic Caching & Revalidation:** Background syncing without manual dispatch.
- **Request Deduplication:** Multiple components can request the same data without triggering multiple network calls.
- **Persisted State:** Easy "Offline First" implementation.

<div class="senior-insight">

**💡 Senior Insight: The Cache Key Philosophy**
Senior architects treat API data as a **cache**, not a store. By using deterministic cache keys (e.g., `['posts', userId, page]`), you delegate the entire lifecycle of data to the library. This prevents the "Stale Data" bug where a developer forgets to clear a Redux slice after a user logs out or navigates away.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "So we don't need Redux anymore?"**
Don't fall for this. Explain that Redux/Zustand is still vital for **Client State** (UI states, form data across screens, feature flags) while React Query handles **Server State**. A hybrid approach is the modern standard.
</div>
</div>

<div class="qa-card">
<span class="question-tag">ZUSTAND</span>

### 6.3 Zustand Performance Internals

**Question:** *"How does Zustand achieve better performance than Context API in React Native?"*

Zustand uses a **pub/sub** model outside of the React render cycle. It allows components to subscribe to specific slices of state via selectors.

<div class="code-block-header">
<span>store.ts</span>
<span>TypeScript</span>
</div>

```typescript
// Only re-renders when 'user.name' changes
const name = useStore((state) => state.user.name);

// This component will NEVER re-render when other properties 
// in the store (like 'settings' or 'posts') change.
```

In React Native, this precision is vital for maintaining 60FPS in complex lists or animated screens.

<div class="senior-insight">

**💡 Senior Insight: Bypassing the React Tree**
Zustand's state is stored in an external object. When you call `set()`, it notifies subscribers directly. Context, however, is part of the React component tree; when the Provider value changes, React must perform a reconciliation pass on the entire subtree to find consumers, which is much more expensive.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What happens if you use a selector that returns a new object every time, like `state => ({ name: state.name })`?"
**Answer:** It will re-render on every store change because the reference to the object is always new. You must either return a primitive value or use Zustand's `shallow` equality function as the second argument to `useStore`.
</div>
</div>

<div class="qa-card">
<span class="question-tag">STATE CATEGORIZATION</span>

### 6.4 The State Spectrum

**Question:** *"How do you categorize state in a large-scale React Native application?"*

Not all state is created equal. A senior developer categorizes state into four main types to choose the right tool for each:

- **Local State:** UI-only logic (`useState` / `useReducer`).
- **Global State:** Shared across features (User Auth, App Settings).
- **Server State:** Data fetched from APIs (Cache, Loading, Error states).
- **Form State:** Temporary input data during complex user flows.

<div class="senior-insight">

**💡 Senior Insight: The Ephemeral State Rule**
Never put "ephemeral" state (like current input text or a toggle switch) into a global store unless it's needed by other screens. The overhead of the JS dispatch cycle can cause visible latency.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "Where does 'Navigation State' fit in?"
**Answer:** Navigation state is its own beast. It's managed by React Navigation but can be integrated with your global store for analytics or deep linking. However, usually, it should be treated as a separate, specialized layer.
</div>
</div>

<div class="qa-card">
<span class="question-tag">REDUX TOOLKIT</span>

### 6.5 RTK Query Implementation

**Question:** *"Explain how RTK Query handles the data fetching lifecycle automatically."*

RTK Query eliminates the need for manual `loading` and `error` states. It uses a declarative approach to define endpoints and automatically manages caching, revalidation, and provides auto-generated hooks.

<div class="code-block-header">
<span>api/apiSlice.ts</span>
<span>TypeScript</span>
</div>

```typescript
export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], void>({
      query: () => '/posts',
      providesTags: ['Post'],
    }),
  }),
});
```

<div class="senior-insight">

**💡 Senior Insight: Automated Cache Invalidation**
The real power of RTK Query is the **Tags** system. By tagging a query with `providesTags: ['Post']` and an update with `invalidatesTags: ['Post']`, RTK Query will automatically refetch the post list when an update succeeds, ensuring UI consistency without manual store updates.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "How do you handle pagination with RTK Query?"
**Answer:** You pass the page number as an argument to the query hook. RTK Query caches each page separately by default. To implement infinite scroll, you can use `merge` logic in the endpoint definition to append new results to the existing cache.
</div>
</div>

<div class="qa-card">
<span class="question-tag">ZUSTAND</span>

### 6.6 Zustand Implementation Pattern

**Question:** *"Why is Zustand often preferred over Redux in modern React Native apps?"*

Zustand is tiny, has zero boilerplate, and doesn't require wrapping the app in Providers. It's high-performance and allows for easy state access both inside and outside of React components.

<div class="code-block-header">
<span>store/useStore.ts</span>
<span>JavaScript</span>
</div>

```javascript
import { create } from 'zustand';

const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
}));
```

<div class="senior-insight">

**💡 Senior Insight: Transient Updates**
Zustand allows for **transient updates** (updates that don't trigger a re-render) via `subscribe`. This is incredibly useful for high-frequency data like scroll positions or animation values where you want to update a native component directly without going through the React render cycle.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "Is Zustand suitable for very large, complex enterprise applications?"
**Answer:** Yes, but it requires discipline. Without the strict structure of Redux (Actions/Reducers), a Zustand store can become a "dumping ground." Senior teams solve this by splitting the store into **slices** and using TypeScript strictly.
</div>
</div>

<div class="qa-card">
<span class="question-tag">OPTIMIZATION</span>

### 6.7 Avoiding Re-renders with Selectors

**Question:** *"What is the most common performance mistake when using global state libraries?"*

The biggest mistake is subscribing to the entire state object instead of specific slices. This causes the component to re-render whenever *any* part of the state changes.

<div class="senior-insight">

**💡 Senior Insight: The Correct Approach**
Always use selectors to pick only the data your component needs.
- `const name = useStore((state) => state.user.name);` (**Correct**)
- `const { user } = useStore();` (**Incorrect** - re-renders on any user property change)
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What if you need multiple values from the store?"
**Answer:** Either call the hook multiple times with simple selectors, or return an array/object and use a shallow equality comparison (e.g., `useStore(selector, shallow)` in Zustand or `useSelector(selector, shallowEqual)` in Redux).
</div>
</div>

<div class="qa-card">
<span class="question-tag">IMMUTABILITY</span>

### 6.8 Immutability & Structural Sharing

**Question:** *"Why is immutability non-negotiable in React Native state management, and how do libraries like Immer optimize this?"*

#### 🧠 Referential Equality
React's reconciliation process relies on **shallow comparisons** (`prevProps === nextProps`). If you mutate an object, the reference remains the same, and React won't trigger a re-render. Immutability ensures that every update creates a new reference.

<div class="senior-insight">

**💡 Senior Insight: The Immer Proxy Magic**
Immer uses **Copy-on-Write (COW)** via JS Proxies. It tracks which parts of your state tree were actually touched and only creates new objects for those specific branches, while using **structural sharing** to reuse the untouched parts. This is significantly more memory-efficient than deep cloning.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "Is there any performance downside to using Immer?"
**Answer:** There is a slight overhead for the Proxy creation and tracking, but it's negligible compared to the manual overhead of spread operators for deeply nested objects or the risk of bugs from mutation.
</div>
</div>

<div class="qa-card">
<span class="question-tag">OFFLINE SYNC</span>

### 6.9 Robust Offline-First Architecture

**Question:** *"How do you architect a state system that handles intermittent connectivity without losing user data?"*

#### 📡 The Persistence Layer
A senior approach involves three layers:
1. **Memory Store**: Fast access for the UI (Zustand/Redux).
2. **Local Storage**: Synchronous, high-speed persistence (MMKV).
3. **Mutation Queue**: A specialized store that tracks pending API calls and executes them in order once the `NetInfo` state returns to `isConnected`.

<div class="senior-insight">

**💡 Senior Insight: Optimistic UI vs. Reality**
Never wait for the server to update the UI in an offline-first app. Update the local store immediately (Optimistic Update) and generate a unique `tempId`. When the server responds, replace the `tempId` with the real `id`. This ensures the user feels zero latency, regardless of their connection.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What happens if a user makes 5 conflicting edits while offline?"
**Answer:** You must implement a **conflict resolution strategy** (e.g., "Last Write Wins" or "Version Vectoring"). Simply replaying the queue might lead to data corruption if the server state changed in the meantime.
</div>
</div>

<div class="qa-card">
<span class="question-tag">MIDDLEWARE</span>

### 6.10 Custom Middleware & Interceptors

**Question:** *"Give a real-world example where a custom Redux/Zustand middleware is superior to a Hook-based solution."*

#### 🛡️ Cross-Cutting Concerns
Middleware is perfect for logic that needs to run outside the component lifecycle.
**Example**: An **Auth-Expiry Interceptor**. When an API call fails with a 401, the middleware can trigger a global "Session Expired" modal, clear the token from the Keychain, and reset the store—all without a single component being aware of the failure.

<div class="senior-insight">

**💡 Senior Insight: Action-to-Analytics**
Middleware is the cleanest way to implement **event-driven analytics**. You can map specific state actions to tracking events (e.g., `PURCHASE_SUCCESS` -> `Mixpanel.track()`) without polluting your UI logic with analytics code.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "Can middleware modify the action before it reaches the reducer?"
**Answer:** Yes. Middleware can intercept, modify, or even cancel actions. For example, a middleware could add a 'timestamp' or 'correlationId' to every action for debugging purposes.
</div>
</div>

<div class="qa-card">
<span class="question-tag">FORM STATE</span>

### 6.11 Global vs. Local Form State

**Question:** *"When is it a 'Performance Anti-pattern' to put form state into a global store?"*

Updating a global store (Redux/Zustand) on every keystroke in a large app can cause noticeable input lag, especially on low-end Android devices. This is because every keystroke triggers a dispatch, a store update, and a potential re-render of the entire connected component tree.

<div class="senior-insight">

**💡 Senior Insight: The "Controlled" Performance Tax**
In React Native, controlled components (`value` + `onChangeText`) involve a round-trip from the Native thread to the JS thread on every character. If that JS thread is also busy updating a massive Redux store, you get "jumping cursors" or dropped characters. Uncontrolled components (using `refs` or libraries like `react-hook-form`) bypass this entirely.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "How do you share data between Step 1 and Step 3 of a multi-screen form without global state?"
**Answer:** Use **Navigation Params** for small data, or a specialized **Zustand slice** (e.g., `useRegistrationStore`) that is explicitly cleared when the flow completes or the user exits.
</div>
</div>

<div class="qa-card">
<span class="question-tag">HYDRATION</span>

### 6.12 The Hydration Bottleneck

**Question:** *"What is 'State Hydration' and how can it increase your app's TTI (Time To Interactive)?"*

Hydration is the process of reading persisted data from disk (MMKV/AsyncStorage) and populating the JS store on startup. If you persist a massive state tree (e.g., 5MB+ of JSON), the app may freeze for hundreds of milliseconds while the JS thread parses the string and hydrates the store.

<div class="senior-insight">

**💡 Senior Insight: Partial & Lazy Hydration**
Never persist the whole store. Use "persistence blacklists" for temporary UI states. For large datasets, implement **Lazy Hydration**—only load critical "Auth" and "Profile" data on startup, and load "Recent Activity" or "Chat History" only when the user actually navigates to those features.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "How do you handle the UI state while the store is still hydrating?"
**Answer:** Use a `persist/REHYDRATE` gate or a simple `hasHydrated` boolean in your store. Show a splash screen or a skeleton loader until the hydration is complete to prevent the UI from flickering between "Empty" and "Populated" states.
</div>
</div>

<div class="qa-card">
<span class="question-tag">SELECTORS</span>

### 6.13 Memoized Selectors & Reselect

**Question:** *"How do you prevent expensive derived state calculations from running on every re-render?"*

Derived state (e.g., filtering a list of 1,000 items) should never happen directly inside a component. Use `createSelector` from **Reselect** or the built-in selector memoization in Zustand. These libraries cache the output; if the input state hasn't changed, they return the memoized result without re-running the logic.

<div class="senior-insight">

**💡 Senior Insight: Selector Composition**
Senior architects compose selectors into a graph. For example, `selectVisibleItems` might depend on `selectRawItems` and `selectFilters`. This ensures that a change in `selectFilters` only re-runs the visibility logic, not the logic that fetched or processed the raw items.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "Does Zustand need Reselect?"
**Answer:** Not strictly, but it helps. While Zustand supports selectors, `createSelector` provides **multi-input memoization** (re-computing only if A OR B changes) which is harder to do with raw Zustand selectors alone.
</div>
</div>

<div class="qa-card">
<span class="question-tag">DEBUGGING</span>

### 6.14 Debugging Complex State Transitions

**Question:** *"How do you debug a state-related bug that only happens in Production?"*

Since Redux DevTools aren't available in production, senior developers rely on **Production Observability** patterns. This involves logging state transitions and snapshots to remote services like Sentry or Bugsnag.

<div class="senior-insight">

**💡 Senior Insight: The Breadcrumb Pattern**
Configure your store middleware to log every `action.type` and `payload` as a "breadcrumb" in your error reporting tool. When a crash occurs, you can see the exact sequence of user actions and state changes that led to the failure, making "impossible to reproduce" bugs trivial to solve.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What is the primary danger of logging state in production?"
**Answer:** **PII (Personally Identifiable Information)**. You MUST implement a sanitizer in your logging middleware to mask sensitive data like passwords, emails, or credit card numbers before they leave the device.
</div>
</div>

<div class="qa-card">
<span class="question-tag">SINGLETONS</span>

### 6.15 State vs. Pure Singletons

**Question:** *"Why is it sometimes better to use a C++ Singleton (via JSI) instead of a JS State Store?"*

If you have data that needs to be accessed by both JS and Native code (like a real-time WebSocket stream or a Database connection), a JS-only store is insufficient. A **C++ Singleton** exposed via JSI allows both worlds to access the same memory address with zero serialization overhead.

<div class="senior-insight">

**💡 Senior Insight: Persistence Across Reloads**
Native singletons persist across JS bundle reloads (Hot Reloads). This is vital for maintaining a socket connection or a heavy database engine during development. If that state lived in JS, the connection would drop every time you save a file, ruining the developer experience.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What is the biggest downside of using Singletons?"
**Answer:** They are difficult to unit test and create **hidden dependencies**. Always prefer dependency injection where possible, and only use singletons for truly global, cross-runtime resources.
</div>
</div>

<div class="qa-card">
<span class="question-tag">STORAGE</span>

### 6.16 MMKV vs. AsyncStorage Internals

**Question:** *"Why is MMKV significantly faster than AsyncStorage for persisting state?"*

The performance difference comes down to the architecture of the communication between JavaScript and the storage engine. **AsyncStorage** is bridge-based and asynchronous, while **MMKV** is JSI-based and synchronous.

<div class="senior-insight">

**💡 Senior Insight: Memory-Mapped Files**
MMKV uses `mmap` to map a file directly into the process's virtual memory. When JS reads a value via JSI, it's accessing a direct C++ pointer to that memory. There is **no JSON serialization**, **no bridge crossing**, and **no context switching**. It's effectively as fast as reading from a JS object but with instant persistence.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "Since MMKV is synchronous, can it block the JS thread?"
**Answer:** Theoretically, yes. If you try to write a massive 10MB string in a loop on the JS thread, it will block until the write completes. However, for standard state (keys, tokens, small objects), the overhead is so low (microseconds) that it's invisible to the user.
</div>
</div>

<div class="qa-card">
<span class="question-tag">RTK QUERY</span>

### 6.17 Optimistic Update Rollbacks

**Question:** *"Walk through the manual rollback logic for an Optimistic Update in RTK Query."*

Optimistic updates provide the best UX by assuming success. However, handling failures gracefully is what separates seniors from juniors. In RTK Query, this is handled via the `onQueryStarted` lifecycle method.

<div class="senior-insight">

**💡 Senior Insight: The Undo Patch**
When you trigger an optimistic update with `api.util.updateQueryResult`, it returns a `patchResult`. If the `queryFulfilled` promise rejects, you MUST call `patchResult.undo()`. For complex apps, you might need to undo multiple patches across different API slices to maintain a consistent state (e.g., reverting both the 'Post' and the 'User's Post Count').
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What happens if a second mutation starts before the first one's rollback finishes?"
**Answer:** This is a **Race Condition**. Senior devs handle this by using the `fixedCacheKey` or by checking if the current state in the cache is still the one they expect before applying the rollback.
</div>
</div>

<div class="qa-card">
<span class="question-tag">NORMALIZATION</span>

### 6.18 Normalized vs. Denormalized State

**Question:** *"How does data normalization prevent the 'Stale UI' bug in complex apps?"*

In a denormalized store, the same data (e.g., a User object) might be duplicated across multiple slices (Posts, Comments, Profile). If the user updates their avatar, you have to find and update every single instance. Normalization treats the store like a **database**, where each entity is stored once in a lookup table (map) by ID.

<div class="senior-insight">

**💡 Senior Insight: Referential Integrity**
Normalization ensures **referential integrity**. By storing only the `userId` in the `posts` table, you ensure that every part of the app always sees the most up-to-date user info. Tools like `normalizr` or the built-in normalization in RTK Query handle the heavy lifting of flattening nested API responses.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What is the performance cost of normalization?"
**Answer:** The cost is the "Join" operation during rendering. You have to look up the user by ID in the users table. However, with memoized selectors, this lookup is nearly instantaneous (O(1)) and far cheaper than the bug-prone mess of updating duplicate data.
</div>
</div>

<div class="qa-card">
<span class="question-tag">CLIENT STATE</span>

### 6.19 Feature-Sliced State Management

**Question:** *"How do you organize state in a mono-repo or a very large application with many teams?"*

Avoid a single, giant `store.ts`. Use a **Feature-Sliced** approach where each feature (e.g., `features/auth`, `features/checkout`) owns its own store, actions, and selectors. These are then combined at the root level.

<div class="senior-insight">

**💡 Senior Insight: Encapsulation & APIs**
Treat each feature's state as a private implementation detail. Only expose what's necessary via a stable "Public API" (e.g., `features/auth/index.ts`). This prevents Team A from accidentally depending on a internal state structure of Team B, allowing teams to refactor their state without breaking the whole app.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "How do you handle cross-feature state dependencies?"
**Answer:** Use **cross-feature selectors** or **middleware**. If the Checkout feature needs the User's address from the Auth feature, the Checkout selector can reach into the Auth slice. Avoid duplicating the address in the Checkout slice.
</div>
</div>

<div class="qa-card">
<span class="question-tag">SERVER STATE</span>

### 6.20 Prefetching & Cache Warming

**Question:** *"How can you use state management to make an app feel 'Instant' even on slow networks?"*

Don't wait for the user to click. Use **Prefetching** to load data into the store before the user navigates to a screen. For example, when a user hovers over a list item (or starts a press), prefetch the details for that item.

<div class="senior-insight">

**💡 Senior Insight: Intent-Based Fetching**
Senior developers use "Intent-Based Fetching." If a user navigates to the 'Profile' tab, they are likely to check 'Settings' next. Prefetching the settings data while the user is looking at their profile makes the next transition feel instantaneous.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "What are the risks of excessive prefetching?"
**Answer:** **Wasteful Data Usage** and **Memory Pressure**. You must be selective. Only prefetch the most likely next actions and set aggressive cache expiration (TTL) to ensure the data is still fresh when the user eventually sees it.
</div>
</div>

<div class="qa-card">
<span class="question-tag">RACE CONDITIONS</span>

### 6.21 Handling Async Race Conditions

**Question:** *"How do you handle the 'Search Race Condition' in global state?"*

If a user types 'A', then 'AB' quickly, the request for 'A' might finish *after* the request for 'AB', overwriting the search results with the older data. This is a classic async race condition.

<div class="senior-insight">

**💡 Senior Insight: Cancellation & AbortControllers**
Modern state libraries (React Query, RTK Query) handle this automatically by cancelling the previous request when a new one is made. If you're using manual Redux/Zustand, you must use an `AbortController` or a "latest request ID" check to discard stale responses.
</div>

<div class="follow-up-trap">

**⚠️ Follow-up Trap:** "How do you handle this with Redux Sagas?"
**Answer:** Use the `takeLatest` effect. It automatically cancels any previous task if a new action of the same type is dispatched, ensuring only the most recent search request is processed.
</div>
</div>

---

## 🛠️ Comparison Matrix

<table>
<thead>
<tr>
<th>Tool</th>
<th>Best For</th>
<th>Performance Impact</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Context API</strong></td>
<td>Static Data (Theme, Auth)</td>
<td>Medium (Triggers tree re-renders)</td>
</tr>
<tr>
<td><strong>Zustand</strong></td>
<td>Modern Apps / Feature State</td>
<td>High (Atomic subscriptions)</td>
</tr>
<tr>
<td><strong>Redux Toolkit</strong></td>
<td>Complex Enterprise Logic</td>
<td>High (Standardized middleware)</td>
</tr>
<tr>
<td><strong>React Query</strong></td>
<td>API Caching / Server State</td>
<td>Extreme (Reduces network overhead)</td>
  </tr>
</tbody>
</table>

<div class="qa-card">
  <span class="question-tag">BASIC STATE</span>
  <h3 style="margin-top: 0;">Q22: Basic state management with useState.</h3>

  <p><strong>useState</strong> is React's built-in hook for managing local component state:</p>

  <div class="code-example">useState Fundamentals</div>
  <pre class="code-block"><code class="language-javascript">// Basic counter
function Counter() {
  const [count, setCount] = useState(0);

  return (
    &lt;View&gt;
      &lt;Text&gt;Count: {count}&lt;/Text&gt;
      &lt;TouchableOpacity onPress={() => setCount(count + 1)}&gt;
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
      &lt;TextInput
        placeholder="Email"
        value={user.email}
        onChangeText={(value) => updateField('email', value)}
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
    <strong>🟢 Beginner Tip:</strong> When updating objects or arrays, always use the functional update form <code>setState(prev => ...)</code> to avoid stale closure bugs and ensure you're working with the latest state.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">LIFTING STATE UP</span>
  <h3 style="margin-top: 0;">Q23: Lifting state up and prop drilling.</h3>

  <p>When multiple components need to share state, move it up to their common parent:</p>

  <div class="code-example">State Lifting Pattern</div>
  <pre class="code-block"><code class="language-javascript">// ❌ BAD: State scattered across components
function Parent() {
  return (
    &lt;View&gt;
      &lt;Counter /&gt; {/* Has its own count state */}
      &lt;Display /&gt; {/* Has its own display state */}
    &lt;/View&gt;
  );
}

// ✅ GOOD: Lift state to common ancestor
function Parent() {
  const [count, setCount] = useState(0);
  const [display, setDisplay] = useState('counter');

  return (
    &lt;View&gt;
      &lt;Counter count={count} onIncrement={() => setCount(c => c + 1)} /&gt;
      &lt;Display value={count} mode={display} /&gt;
      &lt;Controls
        onDisplayChange={setDisplay}
        onReset={() => setCount(0)}
      /&gt;
    &lt;/View&gt;
  );
}

function Counter({ count, onIncrement }) {
  return (
    &lt;TouchableOpacity onPress={onIncrement}>
      &lt;Text&gt;Count: {count}&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}

// Prop drilling becomes problematic with deep nesting
function DeepComponent({ theme, user, onUpdate, ...props }) {
  // This component receives many props just to pass them down
  return &lt;ChildComponent theme={theme} user={user} onUpdate={onUpdate} /&gt;;
}</code></pre>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> <em>"When does lifting state up become prop drilling?"</em><br/>
    <strong>Answer:</strong> When you pass props through multiple levels of components that don't use them. This is a sign you need Context API or a state management library.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">CONTEXT BASICS</span>
  <h3 style="margin-top: 0;">Q24: React Context API for global state.</h3>

  <p><strong>Context</strong> provides a way to pass data through the component tree without prop drilling:</p>

  <div class="code-example">Context API Usage</div>
  <pre class="code-block"><code class="language-javascript">// 1. Create context
const ThemeContext = React.createContext('light');

// 2. Provider component
function App() {
  const [theme, setTheme] = useState('light');

  return (
    &lt;ThemeContext.Provider value={{ theme, setTheme }}>
      &lt;HomeScreen /&gt;
    &lt;/ThemeContext.Provider&gt;
  );
}

// 3. Consumer with useContext hook (preferred)
function ThemeButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    &lt;TouchableOpacity
      onPress={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      &lt;Text&gt;Switch to {theme === 'light' ? 'dark' : 'light'} theme&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}

// 4. Consumer with Consumer component (legacy)
function LegacyComponent() {
  return (
    &lt;ThemeContext.Consumer>
      {({ theme }) => &lt;Text&gt;Current theme: {theme}&lt;/Text&gt;}
    &lt;/ThemeContext.Consumer&gt;
  );
}

// 5. Custom hook for cleaner usage
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

function BetterThemeButton() {
  const { theme, setTheme } = useTheme(); // Cleaner API
  return &lt;TouchableOpacity onPress={() => setTheme(theme === 'light' ? 'dark' : 'light')}&gt;
    &lt;Text&gt;Toggle Theme&lt;/Text&gt;
  &lt;/TouchableOpacity&gt;;
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always provide a default value when creating context: <code>React.createContext(defaultValue)</code>. This helps with testing and prevents undefined errors.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ZUSTAND BASICS</span>
  <h3 style="margin-top: 0;">Q25: Zustand for lightweight state management.</h3>

  <p><strong>Zustand</strong> is a small, fast state management library with a simple API:</p>

  <div class="code-example">Zustand Store Creation</div>
  <pre class="code-block"><code class="language-javascript">// 1. Create store
import { create } from 'zustand';

const useCounterStore = create((set, get) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
  // Computed values
  doubleCount: () => get().count * 2,
}));

// 2. Use in components
function Counter() {
  const { count, increment, decrement, reset, doubleCount } = useCounterStore();

  return (
    &lt;View&gt;
      &lt;Text&gt;Count: {count}&lt;/Text&gt;
      &lt;Text&gt;Double: {doubleCount()}&lt;/Text&gt;
      &lt;TouchableOpacity onPress={increment}>
        &lt;Text&gt;+&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
      &lt;TouchableOpacity onPress={decrement}>
        &lt;Text&gt;-&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
      &lt;TouchableOpacity onPress={reset}>
        &lt;Text&gt;Reset&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// 3. Async actions
const useUserStore = create((set) => ({
  user: null,
  loading: false,
  error: null,

  fetchUser: async (userId) => {
    set({ loading: true, error: null });
    try {
      const user = await api.getUser(userId);
      set({ user, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
}));

// 4. Selectors for performance
function UserProfile() {
  const user = useUserStore((state) => state.user);
  const loading = useUserStore((state) => state.loading);

  // Only re-renders when user or loading changes
  return loading ? &lt;Text&gt;Loading...&lt;/Text&gt; : &lt;Text&gt;{user?.name}&lt;/Text&gt;;
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Zustand stores are created once and reused across components. Actions are defined inside the store and automatically bound to the state.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">REDUX BASICS</span>
  <h3 style="margin-top: 0;">Q26: Redux Toolkit for complex state management.</h3>

  <p><strong>Redux Toolkit (RTK)</strong> simplifies Redux with less boilerplate:</p>

  <div class="code-example">Redux Toolkit Store</div>
  <pre class="code-block"><code class="language-javascript">// 1. Create slice
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0,
  },
  reducers: {
    increment: (state) => {
      state.value += 1; // RTK allows mutation
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;

// 2. Create store
import { configureStore } from '@reduxjs/toolkit';

export const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
});

// 3. Use in components
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    &lt;View&gt;
      &lt;Text&gt;Count: {count}&lt;/Text&gt;
      &lt;TouchableOpacity onPress={() => dispatch(increment())}&gt;
        &lt;Text&gt;+&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// 4. Async logic with createAsyncThunk
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'users/fetchUser',
  async (userId) => {
    const response = await api.getUser(userId);
    return response.data;
  }
);

// In slice
extraReducers: (builder) => {
  builder
    .addCase(fetchUser.pending, (state) => {
      state.loading = true;
    })
    .addCase(fetchUser.fulfilled, (state, action) => {
      state.user = action.payload;
      state.loading = false;
    })
    .addCase(fetchUser.rejected, (state, action) => {
      state.error = action.error.message;
      state.loading = false;
    });
},</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">WHEN TO USE WHAT</span>
  <h3 style="margin-top: 0;">Q27: Choosing the right state management solution.</h3>

  <p>Different tools for different complexity levels:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Complexity</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Solution</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Use Case</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Bundle Size</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Single Component</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;"><code>useState</code></td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Form inputs, local UI state</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">None</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Component Tree</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Context API</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Theme, user auth, app settings</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Small</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Small App</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Zustand</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Global state, simple async logic</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Small</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Large App</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Redux Toolkit</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Complex state, time-travel debugging</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Medium</td>
    </tr>
    <tr>
      <td style="padding: 12px;">Server State</td>
      <td style="padding: 12px;">React Query</td>
      <td style="padding: 12px;">API data, caching, synchronization</td>
      <td style="padding: 12px;">Medium</td>
    </tr>
  </table>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> <em>"Should I always use Redux for global state?"</em><br/>
    <strong>Answer:</strong> No! Start simple with useState/useContext. Only add complexity when you need it. Premature optimization leads to over-engineering.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ASYNC STATE</span>
  <h3 style="margin-top: 0;">Q28: Handling asynchronous operations in state management.</h3>

  <p>Managing loading, error, and success states for async operations:</p>

  <div class="code-example">Async State Patterns</div>
  <pre class="code-block"><code class="language-javascript">// Zustand async pattern
const useAsyncStore = create((set, get) => ({
  data: null,
  loading: false,
  error: null,

  fetchData: async (params) => {
    set({ loading: true, error: null });

    try {
      const result = await apiCall(params);
      set({ data: result, loading: false });
      return result;
    } catch (error) {
      set({ error: error.message, loading: false });
      throw error;
    }
  },

  // Optimistic updates
  updateItem: async (id, updates) => {
    const prevData = get().data;

    // Optimistically update UI
    set((state) => ({
      data: state.data.map(item =>
        item.id === id ? { ...item, ...updates } : item
      )
    }));

    try {
      await api.updateItem(id, updates);
    } catch (error) {
      // Rollback on failure
      set({ data: prevData, error: error.message });
      throw error;
    }
  }
}));

// Redux async with RTK
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await api.getUser(userId);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.data = action.payload;
        state.loading = false;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.error = action.payload;
        state.loading = false;
      });
  }
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PERSISTENCE</span>
  <h3 style="margin-top: 0;">Q29: Persisting state across app restarts.</h3>

  <p>Using AsyncStorage and other storage solutions for state persistence:</p>

  <div class="code-example">State Persistence</div>
  <pre class="code-block"><code class="language-javascript">// Zustand with persistence
import { persist } from 'zustand/middleware';

const useSettingsStore = create(
  persist(
    (set, get) => ({
      theme: 'light',
      language: 'en',
      notifications: true,

      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      toggleNotifications: () => set((state) => ({
        notifications: !state.notifications
      })),
    }),
    {
      name: 'settings-storage', // AsyncStorage key
      // Only persist these fields
      partialize: (state) => ({
        theme: state.theme,
        language: state.language,
        notifications: state.notifications,
      }),
    }
  )
);

// Redux with redux-persist
import { persistStore, persistReducer } from 'redux-persist';
import AsyncStorage from '@react-native-async-storage/async-storage';

const persistConfig = {
  key: 'root',
  storage: AsyncStorage,
  whitelist: ['user', 'settings'], // Only persist these reducers
};

const persistedReducer = persistReducer(persistConfig, rootReducer);

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }),
});

export const persistor = persistStore(store);

// Custom persistence hook
function usePersistentState(key, initialValue) {
  const [state, setState] = useState(() => {
    try {
      const item = AsyncStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setPersistentState = useCallback(async (value) => {
    try {
      setState(value);
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Failed to persist state:', error);
    }
  }, [key]);

  return [state, setPersistentState];
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">TESTING STATE</span>
  <h3 style="margin-top: 0;">Q30: Testing state management logic.</h3>

  <p>Testing stores, reducers, and stateful components:</p>

  <div class="code-example">Testing State Management</div>
  <pre class="code-block"><code class="language-javascript">// Testing Zustand store
import { act, renderHook } from '@testing-library/react-native';

describe('useCounterStore', () => {
  it('should increment counter', () => {
    const { result } = renderHook(() => useCounterStore());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});

// Testing Redux slice
import counterReducer, { increment, decrement } from './counterSlice';

describe('counter reducer', () => {
  it('should handle increment', () => {
    expect(counterReducer({ value: 0 }, increment())).toEqual({
      value: 1
    });
  });

  it('should handle decrement', () => {
    expect(counterReducer({ value: 1 }, decrement())).toEqual({
      value: 0
    });
  });
});

// Testing async thunks
import { fetchUser } from './userSlice';

describe('fetchUser thunk', () => {
  it('should fetch user successfully', async () => {
    const mockUser = { id: 1, name: 'John' };
    // Mock API call
    api.getUser.mockResolvedValue(mockUser);

    const dispatch = jest.fn();
    const thunk = fetchUser(1);

    await thunk(dispatch, () => ({}), undefined);

    expect(dispatch).toHaveBeenCalledWith(
      expect.objectContaining({
        type: 'user/fetchUser/fulfilled',
        payload: mockUser
      })
    );
  });
});

// Testing components with state
describe('Counter Component', () => {
  it('should display initial count', () => {
    const { getByText } = render(&lt;Counter /&gt;);
    expect(getByText('Count: 0')).toBeTruthy();
  });

  it('should increment on button press', () => {
    const { getByText } = render(&lt;Counter /&gt;);

    fireEvent.press(getByText('Increment'));
    expect(getByText('Count: 1')).toBeTruthy();
  });
});</code></pre>
</div>

---

<div class="nav-footer">
<a href="phase5-native-modules-apis.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 05</a>
<a href="phase7-styling-design.md" style="color: #4f46e5; text-decoration: none; font-weight: 600;">Phase 07: Styling ➡️</a>
</div>

</div>
