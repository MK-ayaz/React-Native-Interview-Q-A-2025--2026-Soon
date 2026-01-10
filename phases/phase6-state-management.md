# Phase 06: State Management Strategy

Architecting predictable and performant data flows.

---

### 📋 Phase Overview
Master state management from basic React state to complex global architectures. Learn when to use Context, Zustand, Redux Toolkit, and advanced patterns for scalable React Native applications.

---

### Q1: [ARCHITECTURE] Context vs Redux vs Zustand

**Question:** When should you avoid using React Context for global state in a high-performance React Native app?

**React Context** is built for static or low-frequency updates (like themes or auth user objects). It is NOT a state management tool; it's a dependency injection tool.

**The Problem:** When a value in a Context Provider changes, **every component consuming that context re-renders**. In React Native, where UI thread performance is critical, frequent updates (like a timer or location coordinates) in Context can cause significant frame drops.

> [!TIP]
> **Senior Insight: The "Selector" Missing Link**
> Unlike Redux or Zustand, Context lacks built-in **selectors**. You cannot "subscribe" to only a piece of the context value. To optimize Context, you'd need to wrap components in `React.memo` or split contexts into many smaller providers, which leads to "Provider Hell."

> [!WARNING]
> **Follow-up Trap:** "Can we use `useMemo` inside the Provider to stop re-renders?"
> **Answer:** No. While `useMemo` can prevent the *value object* from changing, any component calling `useContext(MyContext)` will still re-render if the value changes at all. The only real fix for Context performance is **context splitting** or moving to a library with external stores (Zustand/Redux).

---

### Q2: [CACHING] Server State vs Client State

**Question:** Why is moving API data out of Redux/Zustand into React Query (TanStack Query) considered a Senior architectural move?

Traditional state management treats API data as "Global State," leading to massive boilerplate for `loading`, `error`, and `caching` logic.

**Specialized Server State (React Query/RTK Query) provides:**

- **Automatic Caching & Revalidation:** Background syncing without manual dispatch.
- **Request Deduplication:** Multiple components can request the same data without triggering multiple network calls.
- **Persisted State:** Easy "Offline First" implementation.

> [!TIP]
> **Senior Insight: The Cache Key Philosophy**
> Senior architects treat API data as a **cache**, not a store. By using deterministic cache keys (e.g., `['posts', userId, page]`), you delegate the entire lifecycle of data to the library. This prevents the "Stale Data" bug where a developer forgets to clear a Redux slice after a user logs out or navigates away.

> [!WARNING]
> **Follow-up Trap: "So we don't need Redux anymore?"**
> Don't fall for this. Explain that Redux/Zustand is still vital for **Client State** (UI states, form data across screens, feature flags) while React Query handles **Server State**. A hybrid approach is the modern standard.

---

### Q3: [ZUSTAND] Zustand Performance Internals

**Question:** How does Zustand achieve better performance than Context API in React Native?

Zustand uses a **pub/sub** model outside of the React render cycle. It allows components to subscribe to specific slices of state via selectors.

```typescript
// store.ts
// Only re-renders when 'user.name' changes
const name = useStore((state) => state.user.name);

// This component will NEVER re-render when other properties 
// in the store (like 'settings' or 'posts') change.
```

In React Native, this precision is vital for maintaining 60FPS in complex lists or animated screens.

> [!TIP]
> **Senior Insight: Bypassing the React Tree**
> Zustand's state is stored in an external object. When you call `set()`, it notifies subscribers directly. Context, however, is part of the React component tree; when the Provider value changes, React must perform a reconciliation pass on the entire subtree to find consumers, which is much more expensive.

> [!WARNING]
> **Follow-up Trap:** "What happens if you use a selector that returns a new object every time, like `state => ({ name: state.name })`?"
> **Answer:** It will re-render on every store change because the reference to the object is always new. You must either return a primitive value or use Zustand's `shallow` equality function as the second argument to `useStore`.

---

### Q4: [STATE CATEGORIZATION] The State Spectrum

**Question:** How do you categorize state in a large-scale React Native application?

Not all state is created equal. A senior developer categorizes state into four main types to choose the right tool for each:

- **Local State:** UI-only logic (`useState` / `useReducer`).
- **Global State:** Shared across features (User Auth, App Settings).
- **Server State:** Data fetched from APIs (Cache, Loading, Error states).
- **Form State:** Temporary input data during complex user flows.

> [!TIP]
> **Senior Insight: The Ephemeral State Rule**
> Never put "ephemeral" state (like current input text or a toggle switch) into a global store unless it's needed by other screens. The overhead of the JS dispatch cycle can cause visible latency.

> [!WARNING]
> **Follow-up Trap:** "Where does 'Navigation State' fit in?"
> **Answer:** Navigation state is its own beast. It's managed by React Navigation but can be integrated with your global store for analytics or deep linking. However, usually, it should be treated as a separate, specialized layer.

---

### Q5: [REDUX TOOLKIT] RTK Query Implementation

**Question:** Explain how RTK Query handles the data fetching lifecycle automatically.

RTK Query eliminates the need for manual `loading` and `error` states. It uses a declarative approach to define endpoints and automatically manages caching, revalidation, and provides auto-generated hooks.

```typescript
// api/apiSlice.ts
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

> [!TIP]
> **Senior Insight: Automated Cache Invalidation**
> The real power of RTK Query is the **Tags** system. By tagging a query with `providesTags: ['Post']` and an update with `invalidatesTags: ['Post']`, RTK Query will automatically refetch the post list when an update succeeds, ensuring UI consistency without manual store updates.

> [!WARNING]
> **Follow-up Trap:** "How do you handle pagination with RTK Query?"
> **Answer:** You pass the page number as an argument to the query hook. RTK Query caches each page separately by default. To implement infinite scroll, you can use `merge` logic in the endpoint definition to append new results to the existing cache.

---

### Q6: [ZUSTAND] Zustand Implementation Pattern

**Question:** Why is Zustand often preferred over Redux in modern React Native apps?

Zustand is tiny, has zero boilerplate, and doesn't require wrapping the app in Providers. It's high-performance and allows for easy state access both inside and outside of React components.

```javascript
// store/useStore.ts
import { create } from 'zustand';

const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
}));
```

> [!TIP]
> **Senior Insight: Transient Updates**
> Zustand allows for **transient updates** (updates that don't trigger a re-render) via `subscribe`. This is incredibly useful for high-frequency data like scroll positions or animation values where you want to update a native component directly without going through the React render cycle.

> [!WARNING]
> **Follow-up Trap:** "Is Zustand suitable for very large, complex enterprise applications?"
> **Answer:** Yes, but it requires discipline. Without the strict structure of Redux (Actions/Reducers), a Zustand store can become a "dumping ground." Senior teams solve this by splitting the store into **slices** and using TypeScript strictly.

---

### Q7: [OPTIMIZATION] Avoiding Re-renders with Selectors

**Question:** What is the most common performance mistake when using global state libraries?

The biggest mistake is subscribing to the entire state object instead of specific slices. This causes the component to re-render whenever *any* part of the state changes.

> [!TIP]
> **Senior Insight: The Correct Approach**
> Always use selectors to pick only the data your component needs.
> - `const name = useStore((state) => state.user.name);` (**Correct**)
> - `const { user } = useStore();` (**Incorrect** - re-renders on any user property change)

> [!WARNING]
> **Follow-up Trap:** "What if you need multiple values from the store?"
> **Answer:** Either call the hook multiple times with simple selectors, or return an array/object and use a shallow equality comparison (e.g., `useStore(selector, shallow)` in Zustand or `useSelector(selector, shallowEqual)` in Redux).

---

### Q8: [IMMUTABILITY] Immutability & Structural Sharing

**Question:** Why is immutability non-negotiable in React Native state management, and how do libraries like Immer optimize this?

#### 🧠 Referential Equality
React's reconciliation process relies on **shallow comparisons** (`prevProps === nextProps`). If you mutate an object, the reference remains the same, and React won't trigger a re-render. Immutability ensures that every update creates a new reference.

> [!TIP]
> **Senior Insight: The Immer Proxy Magic**
> Immer uses **Copy-on-Write (COW)** via JS Proxies. It tracks which parts of your state tree were actually touched and only creates new objects for those specific branches, while using **structural sharing** to reuse the untouched parts. This is significantly more memory-efficient than deep cloning.

> [!WARNING]
> **Follow-up Trap:** "Is there any performance downside to using Immer?"
> **Answer:** There is a slight overhead for the Proxy creation and tracking, but it's negligible compared to the manual overhead of spread operators for deeply nested objects or the risk of bugs from mutation.

---

### Q9: [OFFLINE SYNC] Robust Offline-First Architecture

**Question:** How do you architect a state system that handles intermittent connectivity without losing user data?

#### 📡 The Persistence Layer
A senior approach involves three layers:
1. **Memory Store**: Fast access for the UI (Zustand/Redux).
2. **Local Storage**: Synchronous, high-speed persistence (MMKV).
3. **Mutation Queue**: A specialized store that tracks pending API calls and executes them in order once the `NetInfo` state returns to `isConnected`.

> [!TIP]
> **Senior Insight: Optimistic UI vs. Reality**
> Never wait for the server to update the UI in an offline-first app. Update the local store immediately (Optimistic Update) and generate a unique `tempId`. When the server responds, replace the `tempId` with the real `id`. This ensures the user feels zero latency, regardless of their connection.

> [!WARNING]
> **Follow-up Trap:** "What happens if a user makes 5 conflicting edits while offline?"
> **Answer:** You must implement a **conflict resolution strategy** (e.g., "Last Write Wins" or "Version Vectoring"). Simply replaying the queue might lead to data corruption if the server state changed in the meantime.

---

### Q10: [MIDDLEWARE] Custom Middleware & Interceptors

**Question:** Give a real-world example where a custom Redux/Zustand middleware is superior to a Hook-based solution?

#### 🛡️ Cross-Cutting Concerns
Middleware is perfect for logic that needs to run outside the component lifecycle.
**Example**: An **Auth-Expiry Interceptor**. When an API call fails with a 401, the middleware can trigger a global "Session Expired" modal, clear the token from the Keychain, and reset the store—all without a single component being aware of the failure.

> [!TIP]
> **Senior Insight: Action-to-Analytics**
> Middleware is the cleanest way to implement **event-driven analytics**. You can map specific state actions to tracking events (e.g., `PURCHASE_SUCCESS` -> `Mixpanel.track()`) without polluting your UI logic with analytics code.

> [!WARNING]
> **Follow-up Trap:** "Can middleware modify the action before it reaches the reducer?"
> **Answer:** Yes. Middleware can intercept, modify, or even cancel actions. For example, a middleware could add a 'timestamp' or 'correlationId' to every action for debugging purposes.

---

### Q11: [FORM STATE] Global vs. Local Form State

**Question:** When is it a 'Performance Anti-pattern' to put form state into a global store?

Updating a global store (Redux/Zustand) on every keystroke in a large app can cause noticeable input lag, especially on low-end Android devices. This is because every keystroke triggers a dispatch, a store update, and a potential re-render of the entire connected component tree.

> [!TIP]
> **Senior Insight: The "Controlled" Performance Tax**
> In React Native, controlled components (`value` + `onChangeText`) involve a round-trip from the Native thread to the JS thread on every character. If that JS thread is also busy updating a massive Redux store, you get "jumping cursors" or dropped characters. Uncontrolled components (using `refs` or libraries like `react-hook-form`) bypass this entirely.

> [!WARNING]
> **Follow-up Trap:** "How do you share data between Step 1 and Step 3 of a multi-screen form without global state?"
> **Answer:** Use **Navigation Params** for small data, or a specialized **Zustand slice** (e.g., `useRegistrationStore`) that is explicitly cleared when the flow completes or the user exits.

---

### Q12: [HYDRATION] The Hydration Bottleneck

**Question:** What is 'State Hydration' and how can it increase your app's TTI (Time To Interactive)?

Hydration is the process of reading persisted data from disk (MMKV/AsyncStorage) and populating the JS store on startup. If you persist a massive state tree (e.g., 5MB+ of JSON), the app may freeze for hundreds of milliseconds while the JS thread parses the string and hydrates the store.

> [!TIP]
> **Senior Insight: Partial & Lazy Hydration**
> Never persist the whole store. Use "persistence blacklists" for temporary UI states. For large datasets, implement **Lazy Hydration**—only load critical "Auth" and "Profile" data on startup, and load "Recent Activity" or "Chat History" only when the user actually navigates to those features.

> [!WARNING]
> **Follow-up Trap:** "How do you handle the UI state while the store is still hydrating?"
> **Answer:** Use a `persist/REHYDRATE` gate or a simple `hasHydrated` boolean in your store. Show a splash screen or a skeleton loader until the hydration is complete to prevent the UI from flickering between "Empty" and "Populated" states.

---

### Q13: [SELECTORS] Memoized Selectors & Reselect

**Question:** How do you prevent expensive derived state calculations from running on every re-render?

Derived state (e.g., filtering a list of 1,000 items) should never happen directly inside a component. Use `createSelector` from **Reselect** or the built-in selector memoization in Zustand. These libraries cache the output; if the input state hasn't changed, they return the memoized result without re-running the logic.

> [!TIP]
> **Senior Insight: Selector Composition**
> Senior architects compose selectors into a graph. For example, `selectVisibleItems` might depend on `selectRawItems` and `selectFilters`. This ensures that a change in `selectFilters` only re-runs the visibility logic, not the logic that fetched or processed the raw items.

> [!WARNING]
> **Follow-up Trap:** "Does Zustand need Reselect?"
> **Answer:** Not strictly, but it helps. While Zustand supports selectors, `createSelector` provides **multi-input memoization** (re-computing only if A OR B changes) which is harder to do with raw Zustand selectors alone.

---

### Q14: [DEBUGGING] Debugging Complex State Transitions

**Question:** How do you debug a state-related bug that only happens in Production?

Since Redux DevTools aren't available in production, senior developers rely on **Production Observability** patterns. This involves logging state transitions and snapshots to remote services like Sentry or Bugsnag.

> [!TIP]
> **Senior Insight: The Breadcrumb Pattern**
> Configure your store middleware to log every `action.type` and `payload` as a "breadcrumb" in your error reporting tool. When a crash occurs, you can see the exact sequence of user actions and state changes that led to the failure, making "impossible to reproduce" bugs trivial to solve.

> [!WARNING]
> **Follow-up Trap:** "What is the primary danger of logging state in production?"
> **Answer:** **PII (Personally Identifiable Information)**. You MUST implement a sanitizer in your logging middleware to mask sensitive data like passwords, emails, or credit card numbers before they leave the device.

---

### Q15: [SINGLETONS] State vs. Pure Singletons

**Question:** Why is it sometimes better to use a C++ Singleton (via JSI) instead of a JS State Store?

If you have data that needs to be accessed by both JS and Native code (like a real-time WebSocket stream or a Database connection), a JS-only store is insufficient. A **C++ Singleton** exposed via JSI allows both worlds to access the same memory address with zero serialization overhead.

> [!TIP]
> **Senior Insight: Persistence Across Reloads**
> Native singletons persist across JS bundle reloads (Hot Reloads). This is vital for maintaining a socket connection or a heavy database engine during development. If that state lived in JS, the connection would drop every time you save a file, ruining the developer experience.

> [!WARNING]
> **Follow-up Trap:** "What is the biggest downside of using Singletons?"
> **Answer:** They are difficult to unit test and create **hidden dependencies**. Always prefer dependency injection where possible, and only use singletons for truly global, cross-runtime resources.

---

### Q16: [STORAGE] MMKV vs. AsyncStorage Internals

**Question:** Why is MMKV significantly faster than AsyncStorage for persisting state?

The performance difference comes down to the architecture of the communication between JavaScript and the storage engine. **AsyncStorage** is bridge-based and asynchronous, while **MMKV** is JSI-based and synchronous.

> [!TIP]
> **Senior Insight: Memory-Mapped Files**
> MMKV uses `mmap` to map a file directly into the process's virtual memory. When JS reads a value via JSI, it's accessing a direct C++ pointer to that memory. There is **no JSON serialization**, **no bridge crossing**, and **no context switching**. It's effectively as fast as reading from a JS object but with instant persistence.

> [!WARNING]
> **Follow-up Trap:** "Since MMKV is synchronous, can it block the JS thread?"
> **Answer:** Theoretically, yes. If you try to write a massive 10MB string in a loop on the JS thread, it will block until the write completes. However, for standard state (keys, tokens, small objects), the overhead is so low (microseconds) that it's invisible to the user.

---

### Q17: [RTK QUERY] Optimistic Update Rollbacks

**Question:** Walk through the manual rollback logic for an Optimistic Update in RTK Query.

Optimistic updates provide the best UX by assuming success. However, handling failures gracefully is what separates seniors from juniors. In RTK Query, this is handled via the `onQueryStarted` lifecycle method.

> [!TIP]
> **Senior Insight: The Undo Patch**
> When you trigger an optimistic update with `api.util.updateQueryResult`, it returns a `patchResult`. If the `queryFulfilled` promise rejects, you MUST call `patchResult.undo()`. For complex apps, you might need to undo multiple patches across different API slices to maintain a consistent state (e.g., reverting both the 'Post' and the 'User's Post Count').

> [!WARNING]
> **Follow-up Trap:** "What happens if a second mutation starts before the first one's rollback finishes?"
> **Answer:** This is a **Race Condition**. Senior devs handle this by using the `fixedCacheKey` or by checking if the current state in the cache is still the one they expect before applying the rollback.

---

### Q18: [NORMALIZATION] Normalized vs. Denormalized State

**Question:** How does data normalization prevent the 'Stale UI' bug in complex apps?

In a denormalized store, the same data (e.g., a User object) might be duplicated across multiple slices (Posts, Comments, Profile). If the user updates their avatar, you have to find and update every single instance. Normalization treats the store like a **database**, where each entity is stored once in a lookup table (map) by ID.

> [!TIP]
> **Senior Insight: Referential Integrity**
> Normalization ensures **referential integrity**. By storing only the `userId` in the `posts` table, you ensure that every part of the app always sees the most up-to-date user info. Tools like `normalizr` or the built-in normalization in RTK Query handle the heavy lifting of flattening nested API responses.

> [!WARNING]
> **Follow-up Trap:** "What is the performance cost of normalization?"
> **Answer:** The cost is the "Join" operation during rendering. You have to look up the user by ID in the users table. However, with memoized selectors, this lookup is nearly instantaneous (O(1)) and far cheaper than the bug-prone mess of updating duplicate data.

---

### Q19: [CLIENT STATE] Feature-Sliced State Management

**Question:** How do you organize state in a mono-repo or a very large application with many teams?

Avoid a single, giant `store.ts`. Use a **Feature-Sliced** approach where each feature (e.g., `features/auth`, `features/checkout`) owns its own store, actions, and selectors. These are then combined at the root level.

> [!TIP]
> **Senior Insight: Encapsulation & APIs**
> Treat each feature's state as a private implementation detail. Only expose what's necessary via a stable "Public API" (e.g., `features/auth/index.ts`). This prevents Team A from accidentally depending on a internal state structure of Team B, allowing teams to refactor their state without breaking the whole app.

> [!WARNING]
> **Follow-up Trap:** "How do you handle cross-feature state dependencies?"
> **Answer:** Use **cross-feature selectors** or **middleware**. If the Checkout feature needs the User's address from the Auth feature, the Checkout selector can reach into the Auth slice. Avoid duplicating the address in the Checkout slice.

---

### Q20: [SERVER STATE] Prefetching & Cache Warming

**Question:** How can you use state management to make an app feel 'Instant' even on slow networks?

Don't wait for the user to click. Use **Prefetching** to load data into the store before the user navigates to a screen. For example, when a user hovers over a list item (or starts a press), prefetch the details for that item.

> [!TIP]
> **Senior Insight: Intent-Based Fetching**
> Senior developers use "Intent-Based Fetching." If a user navigates to the 'Profile' tab, they are likely to check 'Settings' next. Prefetching the settings data while the user is looking at their profile makes the next transition feel instantaneous.

> [!WARNING]
> **Follow-up Trap:** "What are the risks of excessive prefetching?"
> **Answer:** **Wasteful Data Usage** and **Memory Pressure**. You must be selective. Only prefetch the most likely next actions and set aggressive cache expiration (TTL) to ensure the data is still fresh when the user eventually sees it.

---

### Q21: [RACE CONDITIONS] Handling Async Race Conditions

**Question:** How do you handle the 'Search Race Condition' in global state?

If a user types 'A', then 'AB' quickly, the request for 'A' might finish *after* the request for 'AB', overwriting the search results with the older data. This is a classic async race condition.

> [!TIP]
> **Senior Insight: Cancellation & AbortControllers**
> Modern state libraries (React Query, RTK Query) handle this automatically by cancelling the previous request when a new one is made. If you're using manual Redux/Zustand, you must use an `AbortController` or a "latest request ID" check to discard stale responses.

> [!WARNING]
> **Follow-up Trap:** "How do you handle this with Redux Sagas?"
> **Answer:** Use the `takeLatest` effect. It automatically cancels any previous task if a new action of the same type is dispatched, ensuring only the most recent search request is processed.

---

## 🛠️ Comparison Matrix

| Tool | Best For | Performance Impact |
| :--- | :--- | :--- |
| **Context API** | Static Data (Theme, Auth) | Medium (Triggers tree re-renders) |
| **Zustand** | Modern Apps / Feature State | High (Atomic subscriptions) |
| **Redux Toolkit** | Complex Enterprise Logic | High (Standardized middleware) |
| **React Query** | API Caching / Server State | Extreme (Reduces network overhead) |

---

> [!TIP]
> **Next Steps:**
> - Master the **New Architecture** (TurboModules/Fabric) in [Phase 5](./phase5-native-modules-apis.md).
> - Explore **Performance Optimization** in [Phase 4](./phase4-performance-memory.md).

[⬅️ Phase 05: Native Modules](phase5-native-modules-apis.md) | [Phase 07: Styling ➡️](phase7-styling-design.md)
