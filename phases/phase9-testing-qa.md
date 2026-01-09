# Phase 9: Testing & Quality Assurance 🧪

Welcome to **Phase 9**. Senior engineers don't just write code; they write code that is proven to work. This phase covers the full testing pyramid, from unit tests to end-to-end (E2E) automation.

---

## 📋 Table of Contents
1. [The Testing Pyramid](#1-the-testing-pyramid)
2. [Unit Testing with Jest](#2-unit-testing-with-jest)
3. [Component Testing (RNTL)](#3-component-testing-rntl)
4. [End-to-End Testing (Detox)](#4-end-to-end-testing-detox)
5. [Debugging & Profiling](#5-debugging--profiling)
6. [Error Boundaries & Monitoring](#6-error-boundaries--monitoring)

---

## 1. The Testing Pyramid
A balanced testing strategy saves time and prevents regressions.
- **Unit Tests (70%):** Logic, utilities, and hooks.
- **Integration Tests (20%):** Components and navigation flows.
- **E2E Tests (10%):** Critical user paths (Login, Checkout).

---

## 2. Unit Testing with Jest
Jest is the standard runner for React Native. Use it for testing business logic and custom hooks.

### Testing a Custom Hook
```javascript
import { renderHook, act } from '@testing-library/react-native';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter());
  act(() => {
    result.current.increment();
  });
  expect(result.current.count).toBe(1);
});
```

---

## 3. Component Testing (RNTL)
**React Native Testing Library (RNTL)** allows you to test components from the user's perspective without relying on implementation details.

> [!TIP]
> **Pro Tip:** Avoid testing internal state. Instead, test what the user sees (text, buttons) and how they interact with it.

---

## 4. End-to-End Testing (Detox)
Detox is a "gray-box" E2E testing framework specifically for React Native. It runs on real devices or simulators.

```javascript
describe('Login Flow', () => {
  it('should login successfully', async () => {
    await element(by.id('email-input')).typeText('user@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();
    await expect(element(by.text('Welcome'))).toBeVisible();
  });
});
```

---

## 5. Debugging & Profiling
When things go wrong, seniors use professional tools:
- **React DevTools:** Inspect component hierarchy and props.
- **Flipper:** Network inspection, database browsing, and logs.
- **React Native Debugger:** Redux/Zustand state inspection.
- **Performance Monitor:** Tracking FPS and memory usage.

---

## 6. Error Boundaries & Monitoring
In production, you need to know about crashes before your users tell you.
1. **Error Boundaries:** Catch JS errors and show a fallback UI.
2. **Crashlytics:** Track native crashes.
3. **Sentry:** Track JS exceptions and performance bottlenecks.

---

## 🏁 Next Steps
Testing ensures stability. In **Phase 10**, we focus on **Security Best Practices** to protect your users' data and your app's integrity.

[Back to Main README](../README.md) | [Go to Phase 10: Security](./phase10-security-best-practices.md)
