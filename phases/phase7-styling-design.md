# Phase 07: Styling & Design Systems
**Design Tokens, Theming, Responsive Layouts, and Cross-Platform UI**

---

### 📋 Phase Overview
Master styling in React Native from basic StyleSheet usage to advanced design systems. Learn responsive layouts, theming, accessibility, and performance optimization for beautiful, scalable UIs.

---

### Q1: [ARCHITECTURE] Design Tokens vs. Hardcoded Constants
**Question:** Why are Design Tokens critical for large-scale React Native apps, and how do they differ from simple constants?

**Design Tokens** are the "single source of truth" for your UI. While constants are just values, tokens represent a semantic meaning (e.g., `color.primary.main` instead of `#007AFF`).

In a senior-led project, tokens allow for:
- **Multi-theme support**: Changing a single token object swaps the entire look (Dark Mode, High Contrast).
- **Cross-platform consistency**: Sharing tokens with Web or Native teams.
- **Maintainability**: Updating a brand color in one place updates the entire app.

> [!TIP]
> **Senior Insight: Semantic Naming**
> Avoid names like `$blue`. Use `$action-primary-background`. This way, if the brand changes to red, the variable name still makes sense functionally.

---

### Q2: [PERFORMANCE] StyleSheet vs. Inline Styles
**Question:** Explain the performance difference between StyleSheet.create and passing an object directly to the style prop.

#### ⚡ Optimization & Validation
`StyleSheet.create` sends styles across the bridge to the native side **once**, returning an integer ID. Using inline objects `style={{ flex: 1 }}` creates a **new object on every render**, forcing the bridge to re-process the style.

> [!WARNING]
> **Follow-up Trap: "Does 'StyleSheet.create' actually make the app faster?"**
> **Answer:** On modern devices, the difference is negligible for simple views. However, in a **FlashList** with 1000 items, inline styles can lead to significant memory churn and GC pressure.

---

### Q3: [ATOMIC DESIGN] Scalable Atomic Architecture
**Question:** How do you prevent 'Prop Drilling' in a deeply nested Atomic Design system?

#### 🏗️ Composition over Props
Atomic design can lead to deep nesting. Instead of passing props through 5 layers (Page -> Template -> Organism -> Molecule -> Atom), use **Component Composition**. Pass the Atom directly into the Molecule as a prop, or use specialized Contexts for theme-related data.

> [!TIP]
> **Senior Insight: The Folder Structure**
> Don't just group by 'atoms/molecules'. Group by **feature** first, then apply atomic principles within that feature. A global `/components/atoms` folder quickly becomes a dumping ground that is impossible to maintain.

---

### Q4: [RESPONSIVENESS] Screen Density & Scaled Metrics
**Question:** How do you handle 'Pixel Perfection' across a $150 Android phone and an iPhone 16 Pro Max?

#### 📏 Relative Scaling
React Native uses DP, but 16dp on a small screen looks much larger than on a giant screen. A senior approach uses a **Scaling Utility** based on a "Guideline Base Width" (usually 375 for iPhone).

```typescript
// metrics.ts
const { width } = Dimensions.get('window');
const scale = (size: number) => (width / 375) * size;
```

> [!WARNING]
> **Follow-up Trap: "Why is hardcoding 'Dimensions.get' dangerous?"**
> **Answer:** It doesn't account for **screen rotation** or **split-screen mode**. Always use the `useWindowDimensions` hook for reactive layout updates.

---

### Q5: [FLEXBOX] Yoga Engine & Layout Performance
**Question:** What is 'Layout Thrashing' in React Native, and how does Yoga handle it?

```mermaid
graph TD
    JS[JS Style Props] --> Bridge[Bridge / JSI]
    Bridge --> Yoga[Yoga Layout Engine (C++)]
    Yoga --> Calc[Calculate Node Tree]
    Calc --> Layout[Generate Layout Coordinates]
    Layout --> Native[Native UI View Updates]
    
    subgraph "Yoga Optimization"
        Calc --> Cache{Layout Cached?}
        Cache -- Yes --> Layout
        Cache -- No --> Process[Recursive Calculation]
        Process --> Layout
    end
```

#### 📐 Recursive Calculations
Yoga calculates layouts recursively. "Thrashing" occurs when multiple state updates trigger multiple layout passes in a single frame. Yoga optimizes this by **caching layout results**, but deeply nested trees still increase the TTI (Time To Interactive).

> [!TIP]
> **Senior Insight: display: 'none' vs. Conditional Rendering**
> `display: 'none'` keeps the component in the Yoga tree but skips the drawing phase. Conditional rendering (`{show && <Comp />}`) removes it entirely. Use conditional rendering for performance unless you need to preserve the component's internal state.

---

### Q6: [THEMING] Dynamic Theming & Context
**Question:** How do you prevent a 'Flash of Wrong Theme' on app startup?

#### 🌗 Native Synchronization
If you rely on JS state for theming, the app might render with the light theme for a split second before the dark theme loads from storage.
**Solution**: Use a **Native Module** to read the system theme before the first JS render, or use a library like `react-native-bootsplash` to hold the splash screen until the theme is resolved.

> [!WARNING]
> **Follow-up Trap: "How do you handle images in Dark Mode?"**
> **Answer:** Use a `ThemedImage` component that selects the source based on the current theme token, rather than hardcoding logic in every screen.

---

### Q7: [ACCESSIBILITY] Advanced A11y & Font Scaling
**Question:** How do you handle 'Label-in-Label' accessibility for complex UI cards?

#### 🛡️ Grouping & Roles
A senior developer doesn't just add `accessibilityLabel`. They use `accessible={true}` on the parent container to group children, and `accessibilityRole="button"` to tell Screen Readers exactly how to interact with the element.

> [!TIP]
> **Senior Insight: maxFontSizeMultiplier**
> Unrestricted font scaling can break your UI. Use `maxFontSizeMultiplier={1.5}` to allow for accessibility while ensuring the text doesn't grow so large that it overflows its container and becomes unreadable.

---

### Q8: [CSS-IN-JS] Styled Components & Memory
**Question:** Why might 'Styled Components' cause performance issues in a complex list?

#### 📉 Dynamic Wrapper Overhead
Every `styled.View` creates a new component wrapper. In a list of 500 items, this adds 500 extra components to the React tree.
**Optimization**: Use `attrs` for frequently changing styles (like colors) to avoid re-generating the underlying CSS class/style object.

> [!WARNING]
> **Follow-up Trap: "Is NativeWind faster?"**
> **Answer:** Yes, because it maps utility classes to `StyleSheet` at build time, avoiding the runtime overhead of parsing CSS strings.

---

### Q9: [IMAGES] AspectRatio & Layout Stability
**Question:** Why is 'aspectRatio' superior to calculating height manually with Dimensions?

#### 🖼️ Declarative Layout
`aspectRatio` is a first-class citizen in the Yoga engine. It allows the layout engine to calculate the height based on the available width **before** the image even starts loading, preventing the "layout jump" once the image appears.

> [!TIP]
> **Senior Insight: Placeholder Strategy**
> Always use a `backgroundColor` or a `BlurHash` as a placeholder. This provides a better UX by showing the user the layout structure before the network-heavy assets arrive.

---

### Q10: [BORDERS] Hairline Precision
**Question:** Why does 'borderWidth: 0.5' behave inconsistently across devices?

#### 📏 Pixel Snapping
A 0.5dp line might be 1px on an iPhone but 0px (invisible) on a low-end Android. `StyleSheet.hairlineWidth` is a system-defined constant that guarantees a 1-physical-pixel line regardless of the screen's density.

> [!WARNING]
> **Follow-up Trap: "Can you use hairlineWidth for padding?"**
> **Answer:** Technically yes, but practically no. It's too small for touch targets or visible spacing; it should only be used for visual separators.

---

### Q11: [SHADOWS] Cross-Platform Shadow Architecture
**Question:** How do you implement a design that requires 'Soft Shadows' on both platforms?

#### 🌑 The Android Elevation Problem
Android's `elevation` only supports hard-coded light sources and limited colors.
**Senior Solution**: Use **SVG-based shadows** (e.g., `react-native-shadow-2`). It draws a blurred SVG shape behind the component, providing 100% consistency and color control on both iOS and Android.

> [!TIP]
> **Senior Insight: Shadow Performance**
> Native iOS shadows are expensive to render. Use `shadowOpacity` sparingly and avoid large `shadowRadius` values in scrolling lists, as it can cause frame drops during GPU composition.

---

### Q12: [COMPONENTS] Atomic Component API Design
**Question:** How do you design a Button component that supports 10+ variants without 'Prop Soup'?

#### 🎨 Theme-Based Props
Instead of `isPrimary`, `isSecondary`, `isLarge`, use a `variant` prop.

```typescript
type ButtonProps = {
  variant: 'primary' | 'secondary' | 'ghost';
  size: 'sm' | 'md' | 'lg';
}
```
Map these to a style object. This keeps the component API clean and easy to extend.

> [!WARNING]
> **Follow-up Trap: "How do you handle 'Pressable' vs 'TouchableOpacity'?"**
> **Answer:** Use **Pressable**. It's the modern API that provides more granular state access (`pressed`, `hovered`, `focused`) and better performance.

---

### Q13: [TYPOGRAPHY] Semantic Typography
**Question:** Why should you wrap the built-in 'Text' component in a custom 'AppText'?

#### 🛡️ Global Consistency
Wrapping `Text` allows you to:
1. Default to a specific `fontFamily`.
2. Disable font scaling globally (if required).
3. Implement a custom `variant` system (e.g., `<AppText variant="h1">`).
4. Handle internationalization (i18n) centrally.

> [!TIP]
> **Senior Insight: Line Height Ratio**
> Always define `lineHeight` as a multiple of `fontSize` (usually 1.2 to 1.5). This ensures that if the font size changes, the vertical rhythm of the app remains intact.

---

### Q14: [LAYOUT] Safe Area Insets & Layout
**Question:** How do you handle the 'Home Indicator' on iPhone without adding extra padding on Android?

#### 📱 useSafeAreaInsets
The `SafeAreaView` component is a "dumb" wrapper. The `useSafeAreaInsets` hook gives you the raw numbers (e.g., `bottom: 34`). You can then apply this only where needed (e.g., `marginBottom: insets.bottom`) to ensure a perfect fit on all devices.

> [!WARNING]
> **Follow-up Trap: "What is 'edges' prop in SafeAreaProvider?"**
> **Answer:** It allows you to specify which sides to protect (`['top', 'bottom']`), preventing unnecessary padding on the sides.

---

### Q15: [Z-INDEX] zIndex & View Hierarchy
**Question:** Why does a component with zIndex: 999 sometimes appear behind a component with zIndex: 1?

#### 🏗️ Stacking Contexts
In React Native, `zIndex` is scoped to the **parent View**. If Component A and B are in different parents, their z-indices don't interact. To fix this, you must move the component you want on top to a higher-level parent or use a **Portal**.

> [!TIP]
> **Senior Insight: Android Elevation vs. zIndex**
> On Android, `elevation` affects the drawing order *and* the shadow. If you use `zIndex` without `elevation`, the component might be "above" in logic but "below" in visual layering.

---

### Q16: [DYNAMIC STYLES] Style Memoization
**Question:** When is it necessary to use 'useMemo' for a style object?

#### 🔄 Preventing Re-renders
Use `useMemo` when a style object is calculated based on props or state and passed to a component wrapped in `React.memo`. If you don't memoize the style, the child will re-render every time the parent renders because the style object's reference changed.

> [!WARNING]
> **Follow-up Trap: "Does StyleSheet.flatten() help?"**
> **Answer:** No, `flatten` just merges arrays into a single object. It doesn't help with referential stability.

---

### Q17: [TABLETS] Adaptive Layouts
**Question:** How do you implement a 'Split-View' layout for Tablets that becomes a 'Stack-View' on Phones?

#### 📐 Breakpoint Logic
Define a `isTablet` constant using `DeviceInfo.isTablet()` or a width-based check (`width > 600`). Use this to conditionally render different top-level components or change the `flexDirection` of your main container.

> [!TIP]
> **Senior Insight: numColumns in FlatList**
> Don't forget to use the `key` prop to force a re-render of the FlatList when changing `numColumns` dynamically (e.g., when rotating the device).

---

### Q18: [SVGS] SVG Performance & Bundling
**Question:** What is the performance cost of using 50+ SVGs on a single screen?

#### 🎨 CPU Painting
Every SVG in `react-native-svg` is parsed and painted on the CPU.
**Optimization**:
1. Use **react-native-svg-transformer** to import SVGs as components.
2. For simple icons, use **Icon Fonts**.
3. For complex illustrations, use **FastImage** with a PNG/WebP export.

> [!WARNING]
> **Follow-up Trap: "Can you animate SVGs?"**
> **Answer:** Yes, using **Reanimated** and `createAnimatedComponent(Path)`, you can animate the `d` attribute (path data) for morphing effects.

---

### Q19: [DEBUGGING] Visual Debugging Tools
**Question:** How do you identify which component is causing an 'Overflow' or 'Ghost Padding' issue?

#### 🔍 The 'Rainbow' Method
A quick senior trick is to apply temporary background colors to suspected components: `style={{ backgroundColor: 'rgba(255,0,0,0.2)' }}`. This reveals the true boundaries of every View, making it obvious where the extra padding or clipping is happening.

> [!TIP]
> **Senior Insight: Flipper Layout Inspector**
> Flipper allows you to edit styles (margins, colors, sizes) in **real-time** on a running app without a reload. This is the fastest way to fine-tune a complex UI.

---

### Q20: [GRADIENTS] Native Gradients vs. SVGs
**Question:** Why is 'react-native-linear-gradient' preferred over an SVG gradient?

#### ⚡ GPU Acceleration
`react-native-linear-gradient` uses a native `CAGradientLayer` (iOS) or `GradientDrawable` (Android). These are highly optimized by the OS and rendered on the GPU, whereas SVG gradients require complex path parsing on the CPU.

> [!WARNING]
> **Follow-up Trap: "How to create a 'Glassmorphism' effect?"**
> **Answer:** Use **@react-native-community/blur** for the frosted glass effect, layered on top of a gradient.

---

### Q21: [ANIMATED STYLES] Layout vs. Transform Animations
**Question:** Why does animating 'width' cause jank while 'scale' remains smooth?

#### 🧵 Thread Blocking
- **width**: Triggers a full **Yoga layout pass** and re-calculates the positions of all surrounding elements. This runs on the UI thread and can be slow.
- **scale**: Only affects the **GPU layer** of that specific component. It doesn't change the layout of other elements, making it much cheaper to animate.

> [!TIP]
> **Senior Insight: LayoutAnimation API**
> If you *must* animate width/height, use the `LayoutAnimation` API. It handles the transition on the native side more efficiently than JS-based width updates.

---

[⬅️ Phase 06: State Management](./phase6-state-management.md) | [Next Phase ➡️](./phase8-animations-gestures.md)
