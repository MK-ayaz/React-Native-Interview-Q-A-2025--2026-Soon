# Phase 7: Styling & Design Systems 🎨

Welcome to **Phase 7**. Senior engineers don't just "style" components; they build **Design Systems**. This phase covers scalable styling architectures, atomic design, and performance-first UI patterns.

---

## 📋 Table of Contents
1. [Design Tokens & Tokens-first Styling](#1-design-tokens--tokens-first-styling)
2. [Atomic Design in React Native](#2-atomic-design-in-react-native)
3. [Performance: StyleSheet vs. The Rest](#3-performance-stylesheet-vs-the-rest)
4. [Responsive Design Strategies](#4-responsive-design-strategies)
5. [Theming & Dark Mode](#5-theming--dark-mode)

---

## 1. Design Tokens & Tokens-first Styling
Hardcoded values are the enemy of maintainability. Use **Design Tokens** for colors, spacing, and typography.

```javascript
// theme/tokens.ts
export const TOKENS = {
  colors: {
    primary: '#007AFF',
    success: '#4CD964',
    danger: '#FF3B30',
  },
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
  },
};
```

---

## 2. Atomic Design in React Native
Structure your UI components by complexity:
- **Atoms:** Buttons, Inputs, Icons (Single purpose).
- **Molecules:** SearchBar (Input + Icon).
- **Organisms:** Header (Logo + SearchBar + ProfileIcon).
- **Templates/Pages:** High-level layouts.

---

## 3. Performance: StyleSheet vs. The Rest
React Native's `StyleSheet.create` sends styles across the bridge **once** and refers to them by ID.

| Method | Performance | DX |
| :--- | :--- | :--- |
| **StyleSheet** | High (Optimized) | Standard |
| **Inline Styles** | Low (New object every render) | Fast prototyping |
| **Styled Components** | Medium (Runtime overhead) | Excellent (CSS-in-JS) |
| **Tailwind (NativeWind)** | High (Pre-compiled) | Rapid development |

---

## 4. Responsive Design Strategies
React Native uses **Density-Independent Pixels (dp)**. For tablets and different screen sizes, use a combination of:
1. **Flexbox:** Use `flex: 1` for fluid layouts.
2. **Percentage:** Use `%` for relative widths.
3. **Dimensions API:** `useWindowDimensions()` for conditional logic.

> [!TIP]
> **Pro Tip:** Avoid fixed heights/widths for containers. Let the content and flexbox define the layout.

---

## 5. Theming & Dark Mode
Use the `Appearance` API or `useColorScheme` hook to support system-wide dark mode.

```tsx
const theme = useColorScheme() === 'dark' ? darkTheme : lightTheme;

const styles = StyleSheet.create({
  container: {
    backgroundColor: theme.background,
  },
});
```

---

## 🏁 Next Steps
A great UI is static without movement. In **Phase 8**, we bring your components to life with **Animations & Gestures** using Reanimated 3.

[Back to Main README](../README.md) | [Go to Phase 8: Animations & Gestures](./phase8-animations-gestures.md)
