# Phase 6: State Management 🧠

Welcome to **Phase 6**. At the Senior level, state management isn't about which library you use, but **how** you architect your data flow to be predictable, performant, and easy to debug.

---

## 📋 Table of Contents
1. [The State Spectrum](#1-the-state-spectrum)
2. [Context API vs. Redux vs. Zustand](#2-context-api-vs-redux-vs-zustand)
3. [Redux Toolkit (RTK) & RTK Query](#3-redux-toolkit-rtk--rtk-query)
4. [Zustand: The Modern Alternative](#4-zustand-the-modern-alternative)
5. [Server-State vs. Client-State](#5-server-state-vs-client-state)
6. [Optimization: Selectors & Re-renders](#6-optimization-selectors--re-renders)

---

## 1. The State Spectrum
Not all state is created equal. A senior developer categorizes state into:
- **Local State:** `useState` / `useReducer` (UI-only logic).
- **Global State:** Shared across features (User Auth, Settings).
- **Server State:** Data fetched from APIs (Cache, Loading, Error).
- **Form State:** Temporary input data.

---

## 2. Context API vs. Redux vs. Zustand

| Tool | Best For | Why? |
| :--- | :--- | :--- |
| **Context API** | Static/Low-frequency | Re-renders everything below the provider on any change. |
| **Redux (RTK)** | Large, complex apps | Standardized, powerful DevTools, complex middleware. |
| **Zustand** | Most modern apps | Tiny, no providers needed, simple API, high performance. |

---

## 3. Redux Toolkit (RTK) & RTK Query
Stop writing boilerplate. RTK Query handles the entire data fetching lifecycle automatically.

### RTK Query Example
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

---

## 4. Zustand: The Modern Alternative
Zustand is often preferred in React Native for its simplicity and lack of "Provider" nesting.

```javascript
import { create } from 'zustand';

const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
}));
```

---

## 5. Server-State vs. Client-State
One of the biggest shifts in modern React is moving API data out of global stores like Redux and into specialized caches like **React Query** or **RTK Query**.

> [!TIP]
> **Pro Tip:** Keep your global store (Zustand/Redux) strictly for "Client State" (e.g., UI theme, auth token) and use a query library for "Server State".

---

## 6. Optimization: Selectors & Re-renders
The number one cause of performance issues in global state is unnecessary re-renders. Always use **selectors**.

### The Right Way (Zustand)
```javascript
// Only re-renders when 'user.name' changes
const name = useStore((state) => state.user.name);
```

### The Wrong Way
```javascript
// Re-renders when ANY property in the store changes
const { user } = useStore();
```

---

## 🏁 Next Steps
Now that your data flow is solid, let's make it look beautiful. In **Phase 7**, we dive into **Styling & Design Systems** for React Native.

[Back to Main README](../README.md) | [Go to Phase 7: Styling & Design](./phase7-styling-design.md)
