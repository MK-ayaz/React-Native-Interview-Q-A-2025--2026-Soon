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
    background: linear-gradient(135deg, #ec4899 0%, #db2777 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(236, 72, 153, 0.2);
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
    background: #fdf2f8;
    color: #db2777;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #fdf2f8;
    border-left: 4px solid #ec4899;
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
  <span class="phase-number">Phase 07</span>
  <h1 class="phase-title">Styling & Design Systems</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Design Tokens, Theming, Responsive Layouts, and Cross-Platform UI</p>
</div>

---

### 📋 Phase Overview
Master styling in React Native from basic StyleSheet usage to advanced design systems. Learn responsive layouts, theming, accessibility, and performance optimization for beautiful, scalable UIs.

---

<div class="qa-card">
  <span class="question-tag">ARCHITECTURE</span>
  <h3>7.1 Design Tokens vs. Hardcoded Constants</h3>
  
  **Question:** *"Why are Design Tokens critical for large-scale React Native apps, and how do they differ from simple constants?"*
  
  <p><strong>Design Tokens</strong> are the "single source of truth" for your UI. While constants are just values, tokens represent a semantic meaning (e.g., <code>color.primary.main</code> instead of <code>#007AFF</code>).</p>
  
  <p>In a senior-led project, tokens allow for:</p>
  <ul>
    <li><strong>Multi-theme support:</strong> Changing a single token object swaps the entire look (Dark Mode, High Contrast).</li>
    <li><strong>Cross-platform consistency:</strong> Sharing tokens with Web or Native teams.</li>
    <li><strong>Maintainability:</strong> Updating a brand color in one place updates the entire app.</li>
  </ul>

  <div class="senior-insight">
    <strong>💡 Senior Insight: Semantic Naming</strong><br/>
    Avoid names like <code>$blue</code>. Use <code>$action-primary-background</code>. This way, if the brand changes to red, the variable name still makes sense functionally.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE</span>
  <h3>7.2 StyleSheet vs. Inline Styles</h3>
  
  **Question:** *"Explain the performance difference between StyleSheet.create and passing an object directly to the style prop."*
  
  #### ⚡ Optimization & Validation
  `StyleSheet.create` sends styles across the bridge to the native side **once**, returning an integer ID. Using inline objects `style={{ flex: 1 }}` creates a **new object on every render**, forcing the bridge to re-process the style.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Does 'StyleSheet.create' actually make the app faster?"</em><br/>
    <strong>Answer:</strong> On modern devices, the difference is negligible for simple views. However, in a <strong>FlashList</strong> with 1000 items, inline styles can lead to significant memory churn and GC pressure.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ATOMIC DESIGN</span>
  <h3>7.3 Scalable Atomic Architecture</h3>
  
  **Question:** *"How do you prevent 'Prop Drilling' in a deeply nested Atomic Design system?"*
  
  #### 🏗️ Composition over Props
  Atomic design can lead to deep nesting. Instead of passing props through 5 layers (Page -> Template -> Organism -> Molecule -> Atom), use **Component Composition**. Pass the Atom directly into the Molecule as a prop, or use specialized Contexts for theme-related data.

  <div class="senior-insight">
    <strong>Senior Insight: The Folder Structure</strong><br/>
    Don't just group by 'atoms/molecules'. Group by <strong>feature</strong> first, then apply atomic principles within that feature. A global <code>/components/atoms</code> folder quickly becomes a dumping ground that is impossible to maintain.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">RESPONSIVENESS</span>
  <h3>7.4 Screen Density & Scaled Metrics</h3>
  
  **Question:** *"How do you handle 'Pixel Perfection' across a $150 Android phone and an iPhone 16 Pro Max?"*
  
  #### 📏 Relative Scaling
  React Native uses DP, but 16dp on a small screen looks much larger than on a giant screen. A senior approach uses a **Scaling Utility** based on a "Guideline Base Width" (usually 375 for iPhone).

  <div class="code-block-header">
    <span>metrics.ts</span>
  </div>
  ```typescript
  const { width } = Dimensions.get('window');
  const scale = (size: number) => (width / 375) * size;
  ```

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Why is hardcoding 'Dimensions.get' dangerous?"</em><br/>
    <strong>Answer:</strong> It doesn't account for <strong>screen rotation</strong> or <strong>split-screen mode</strong>. Always use the <code>useWindowDimensions</code> hook for reactive layout updates.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">FLEXBOX</span>
  <h3>7.5 Yoga Engine & Layout Performance</h3>
  
  **Question:** *"What is 'Layout Thrashing' in React Native, and how does Yoga handle it?"*
  
  #### 📐 Recursive Calculations
  Yoga calculates layouts recursively. "Thrashing" occurs when multiple state updates trigger multiple layout passes in a single frame. Yoga optimizes this by **caching layout results**, but deeply nested trees still increase the TTI (Time To Interactive).

  <div class="senior-insight">
    <strong>Senior Insight: display: 'none' vs. Conditional Rendering</strong><br/>
    <code>display: 'none'</code> keeps the component in the Yoga tree but skips the drawing phase. Conditional rendering (<code>{show && <Comp />}</code>) removes it entirely. Use conditional rendering for performance unless you need to preserve the component's internal state.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">THEMING</span>
  <h3>7.6 Dynamic Theming & Context</h3>
  
  **Question:** *"How do you prevent a 'Flash of Wrong Theme' on app startup?"*
  
  #### 🌗 Native Synchronization
  If you rely on JS state for theming, the app might render with the light theme for a split second before the dark theme loads from storage.
  **Solution**: Use a **Native Module** to read the system theme before the first JS render, or use a library like `react-native-bootsplash` to hold the splash screen until the theme is resolved.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How do you handle images in Dark Mode?"</em><br/>
    <strong>Answer:</strong> Use a <code>ThemedImage</code> component that selects the source based on the current theme token, rather than hardcoding logic in every screen.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ACCESSIBILITY</span>
  <h3>7.7 Advanced A11y & Font Scaling</h3>
  
  **Question:** *"How do you handle 'Label-in-Label' accessibility for complex UI cards?"*
  
  #### 🛡️ Grouping & Roles
  A senior developer doesn't just add <code>accessibilityLabel</code>. They use <code>accessible={true}</code> on the parent container to group children, and <code>accessibilityRole="button"</code> to tell Screen Readers exactly how to interact with the element.

  <div class="senior-insight">
    <strong>Senior Insight: maxFontSizeMultiplier</strong><br/>
    Unrestricted font scaling can break your UI. Use <code>maxFontSizeMultiplier={1.5}</code> to allow for accessibility while ensuring the text doesn't grow so large that it overflows its container and becomes unreadable.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">CSS-IN-JS</span>
  <h3>7.8 Styled Components & Memory</h3>
  
  **Question:** *"Why might 'Styled Components' cause performance issues in a complex list?"*
  
  #### 📉 Dynamic Wrapper Overhead
  Every <code>styled.View</code> creates a new component wrapper. In a list of 500 items, this adds 500 extra components to the React tree.
  **Optimization**: Use <code>attrs</code> for frequently changing styles (like colors) to avoid re-generating the underlying CSS class/style object.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Is NativeWind faster?"</em><br/>
    <strong>Answer:</strong> Yes, because it maps utility classes to <code>StyleSheet</code> at build time, avoiding the runtime overhead of parsing CSS strings.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">IMAGES</span>
  <h3>7.9 AspectRatio & Layout Stability</h3>
  
  **Question:** *"Why is 'aspectRatio' superior to calculating height manually with Dimensions?"*
  
  #### 🖼️ Declarative Layout
  <code>aspectRatio</code> is a first-class citizen in the Yoga engine. It allows the layout engine to calculate the height based on the available width **before** the image even starts loading, preventing the "layout jump" once the image appears.

  <div class="senior-insight">
    <strong>Senior Insight: Placeholder Strategy</strong><br/>
    Always use a <code>backgroundColor</code> or a <code>BlurHash</code> as a placeholder. This provides a better UX by showing the user the layout structure before the network-heavy assets arrive.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">BORDERS</span>
  <h3>7.10 Hairline Precision</h3>
  
  **Question:** *"Why does 'borderWidth: 0.5' behave inconsistently across devices?"*
  
  #### 📏 Pixel Snapping
  A 0.5dp line might be 1px on an iPhone but 0px (invisible) on a low-end Android. `StyleSheet.hairlineWidth` is a system-defined constant that guarantees a 1-physical-pixel line regardless of the screen's density.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Can you use hairlineWidth for padding?"</em><br/>
    <strong>Answer:</strong> Technically yes, but practically no. It's too small for touch targets or visible spacing; it should only be used for visual separators.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">SHADOWS</span>
  <h3>7.11 Cross-Platform Shadow Architecture</h3>
  
  **Question:** *"How do you implement a design that requires 'Soft Shadows' on both platforms?"*
  
  #### 🌑 The Android Elevation Problem
  Android's `elevation` only supports hard-coded light sources and limited colors.
  **Senior Solution**: Use **SVG-based shadows** (e.g., `react-native-shadow-2`). It draws a blurred SVG shape behind the component, providing 100% consistency and color control on both iOS and Android.

  <div class="senior-insight">
    <strong>Senior Insight: Shadow Performance</strong><br/>
    Native iOS shadows are expensive to render. Use <code>shadowOpacity</code> sparingly and avoid large <code>shadowRadius</code> values in scrolling lists, as it can cause frame drops during GPU composition.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">COMPONENTS</span>
  <h3>7.12 Atomic Component API Design</h3>
  
  **Question:** *"How do you design a Button component that supports 10+ variants without 'Prop Soup'?"*
  
  #### 🎨 Theme-Based Props
  Instead of <code>isPrimary</code>, <code>isSecondary</code>, <code>isLarge</code>, use a <code>variant</code> prop.

  ```typescript
  type ButtonProps = {
    variant: 'primary' | 'secondary' | 'ghost';
    size: 'sm' | 'md' | 'lg';
  }
  ```
  Map these to a style object. This keeps the component API clean and easy to extend.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How do you handle 'Pressable' vs 'TouchableOpacity'?"</em><br/>
    <strong>Answer:</strong> Use <strong>Pressable</strong>. It's the modern API that provides more granular state access (<code>pressed</code>, <code>hovered</code>, <code>focused</code>) and better performance.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">TYPOGRAPHY</span>
  <h3>7.13 Semantic Typography</h3>
  
  **Question:** *"Why should you wrap the built-in 'Text' component in a custom 'AppText'?"*
  
  #### 🛡️ Global Consistency
  Wrapping `Text` allows you to:
  1. Default to a specific `fontFamily`.
  2. Disable font scaling globally (if required).
  3. Implement a custom `variant` system (e.g., `<AppText variant="h1">`).
  4. Handle internationalization (i18n) centrally.

  <div class="senior-insight">
    <strong>Senior Insight: Line Height Ratio</strong><br/>
    Always define <code>lineHeight</code> as a multiple of <code>fontSize</code> (usually 1.2 to 1.5). This ensures that if the font size changes, the vertical rhythm of the app remains intact.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">LAYOUT</span>
  <h3>7.14 Safe Area Insets & Layout</h3>
  
  **Question:** *"How do you handle the 'Home Indicator' on iPhone without adding extra padding on Android?"*
  
  #### 📱 useSafeAreaInsets
  The `SafeAreaView` component is a "dumb" wrapper. The `useSafeAreaInsets` hook gives you the raw numbers (e.g., `bottom: 34`). You can then apply this only where needed (e.g., `marginBottom: insets.bottom`) to ensure a perfect fit on all devices.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What is 'edges' prop in SafeAreaProvider?"</em><br/>
    <strong>Answer:</strong> It allows you to specify which sides to protect (<code>['top', 'bottom']</code>), preventing unnecessary padding on the sides.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">Z-INDEX</span>
  <h3>7.15 zIndex & View Hierarchy</h3>
  
  **Question:** *"Why does a component with zIndex: 999 sometimes appear behind a component with zIndex: 1?"*
  
  #### 🏗️ Stacking Contexts
  In React Native, `zIndex` is scoped to the **parent View**. If Component A and B are in different parents, their z-indices don't interact. To fix this, you must move the component you want on top to a higher-level parent or use a **Portal**.

  <div class="senior-insight">
    <strong>Senior Insight: Android Elevation vs. zIndex</strong><br/>
    On Android, <code>elevation</code> affects the drawing order *and* the shadow. If you use <code>zIndex</code> without <code>elevation</code>, the component might be "above" in logic but "below" in visual layering.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">DYNAMIC STYLES</span>
  <h3>7.16 Style Memoization</h3>
  
  **Question:** *"When is it necessary to use 'useMemo' for a style object?"*
  
  #### 🔄 Preventing Re-renders
  Use `useMemo` when a style object is calculated based on props or state and passed to a component wrapped in `React.memo`. If you don't memoize the style, the child will re-render every time the parent renders because the style object's reference changed.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Does StyleSheet.flatten() help?"</em><br/>
    <strong>Answer:</strong> No, <code>flatten</code> just merges arrays into a single object. It doesn't help with referential stability.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">TABLETS</span>
  <h3>7.17 Adaptive Layouts</h3>
  
  **Question:** *"How do you implement a 'Split-View' layout for Tablets that becomes a 'Stack-View' on Phones?"*
  
  #### 📐 Breakpoint Logic
  Define a `isTablet` constant using `DeviceInfo.isTablet()` or a width-based check (`width > 600`). Use this to conditionally render different top-level components or change the `flexDirection` of your main container.

  <div class="senior-insight">
    <strong>Senior Insight: numColumns in FlatList</strong><br/>
    Don't forget to use the <code>key</code> prop to force a re-render of the FlatList when changing <code>numColumns</code> dynamically (e.g., when rotating the device).
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">SVGS</span>
  <h3>7.18 SVG Performance & Bundling</h3>
  
  **Question:** *"What is the performance cost of using 50+ SVGs on a single screen?"*
  
  #### 🎨 CPU Painting
  Every SVG in `react-native-svg` is parsed and painted on the CPU.
  **Optimization**:
  1. Use **react-native-svg-transformer** to import SVGs as components.
  2. For simple icons, use **Icon Fonts**.
  3. For complex illustrations, use **FastImage** with a PNG/WebP export.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Can you animate SVGs?"</em><br/>
    <strong>Answer:</strong> Yes, using <strong>Reanimated</strong> and <code>createAnimatedComponent(Path)</code>, you can animate the <code>d</code> attribute (path data) for morphing effects.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">DEBUGGING</span>
  <h3>7.19 Visual Debugging Tools</h3>
  
  **Question:** *"How do you identify which component is causing an 'Overflow' or 'Ghost Padding' issue?"*
  
  #### 🔍 The 'Rainbow' Method
  A quick senior trick is to apply temporary background colors to suspected components: `style={{ backgroundColor: 'rgba(255,0,0,0.2)' }}`. This reveals the true boundaries of every View, making it obvious where the extra padding or clipping is happening.

  <div class="senior-insight">
    <strong>Senior Insight: Flipper Layout Inspector</strong><br/>
    Flipper allows you to edit styles (margins, colors, sizes) in **real-time** on a running app without a reload. This is the fastest way to fine-tune a complex UI.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">GRADIENTS</span>
  <h3>7.20 Native Gradients vs. SVGs</h3>
  
  **Question:** *"Why is 'react-native-linear-gradient' preferred over an SVG gradient?"*
  
  #### ⚡ GPU Acceleration
  `react-native-linear-gradient` uses a native `CAGradientLayer` (iOS) or `GradientDrawable` (Android). These are highly optimized by the OS and rendered on the GPU, whereas SVG gradients require complex path parsing on the CPU.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How to create a 'Glassmorphism' effect?"</em><br/>
    <strong>Answer:</strong> Use <strong>@react-native-community/blur</strong> for the frosted glass effect, layered on top of a gradient.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ANIMATED STYLES</span>
  <h3>7.21 Layout vs. Transform Animations</h3>
  
  **Question:** *"Why does animating 'width' cause jank while 'scale' remains smooth?"*
  
  #### 🧵 Thread Blocking
  - **width**: Triggers a full **Yoga layout pass** and re-calculates the positions of all surrounding elements. This runs on the UI thread and can be slow.
  - **scale**: Only affects the **GPU layer** of that specific component. It doesn't change the layout of other elements, making it much cheaper to animate.

  <div class="senior-insight">
    <strong>Senior Insight: LayoutAnimation API</strong><br/>
    If you *must* animate width/height, use the <code>LayoutAnimation</code> API. It handles the transition on the native side more efficiently than JS-based width updates.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">BASIC STYLING</span>
  <h3 style="margin-top: 0;">Q22: Basic styling with StyleSheet in React Native.</h3>

  <p><strong>StyleSheet</strong> is React Native's primary styling API, providing performance optimizations and type safety:</p>

  <div class="code-example">StyleSheet Basics</div>
  <pre class="code-block"><code class="language-javascript">import { StyleSheet, View, Text } from 'react-native';

function BasicStyling() {
  return (
    &lt;View style={styles.container}>
      &lt;Text style={styles.title}>Hello World&lt;/Text>
      &lt;Text style={styles.subtitle}>Welcome to React Native&lt;/Text>
    &lt;/View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
  },
});

// Inline styles (use sparingly)
function InlineStyles() {
  return (
    &lt;Text style={{
      fontSize: 18,
      color: 'blue',
      fontWeight: 'bold'
    }}>
      Inline Styled Text
    &lt;/Text>
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always use StyleSheet.create() instead of plain objects. It provides validation, performance optimizations, and prevents typos in style names.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">LAYOUT BASICS</span>
  <h3 style="margin-top: 0;">Q23: Flexbox layout fundamentals in React Native.</h3>

  <p>React Native uses Flexbox for layouts, but with some key differences from web CSS:</p>

  <div class="code-example">Flexbox Layouts</div>
  <pre class="code-block"><code class="language-javascript">// Basic flex container
const styles = StyleSheet.create({
  container: {
    flex: 1,              // Take full available space
    flexDirection: 'column', // Default: vertical layout
    justifyContent: 'center', // Main axis alignment
    alignItems: 'center',    // Cross axis alignment
    padding: 20,
  },

  // Row layout
  rowContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },

  // Equal width columns
  column: {
    flex: 1, // Each takes equal space
    margin: 5,
  },

  // Fixed width item
  fixedItem: {
    width: 100,
    height: 100,
  },

  // Flexible item
  flexibleItem: {
    flex: 2, // Takes 2x space of flex: 1 items
  },
});

function LayoutExamples() {
  return (
    &lt;View style={styles.container}>
      {/* Vertical centering */}
      &lt;Text style={{ fontSize: 20 }}>Centered Content&lt;/Text&gt;
    &lt;/View>

    // Horizontal layout
    &lt;View style={styles.rowContainer}>
      &lt;Text style={styles.column}>Left&lt;/Text>
      &lt;Text style={styles.column}>Center&lt;/Text&gt;
      &lt;Text style={styles.column}>Right&lt;/Text>
    &lt;/View>
  );
}</code></pre>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> <em>"What's the difference between React Native Flexbox and CSS Flexbox?"</em><br/>
    <strong>Answer:</strong> Default flexDirection is 'column' (not 'row'), and you must specify dimensions explicitly - there's no automatic sizing like in browsers.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">RESPONSIVE DESIGN</span>
  <h3 style="margin-top: 0;">Q24: Creating responsive layouts for different screen sizes.</h3>

  <p>React Native apps need to work across various device sizes and orientations:</p>

  <div class="code-example">Responsive Design Patterns</div>
  <pre class="code-block"><code class="language-javascript">import { Dimensions, useWindowDimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

// Static dimensions (don't update on rotation)
const screenWidth = Dimensions.get('window').width;
const screenHeight = Dimensions.get('window').height;

// Dynamic dimensions (update on rotation)
function ResponsiveComponent() {
  const { width, height } = useWindowDimensions();

  return (
    &lt;View style={{
      width: width * 0.8,  // 80% of screen width
      height: height * 0.5, // 50% of screen height
    }}>
      &lt;Text>Responsive Content&lt;/Text&gt;
    &lt;/View>
  );
}

// Breakpoint system
const breakpoints = {
  small: 320,
  medium: 768,
  large: 1024,
};

function getBreakpoint(width) {
  if (width >= breakpoints.large) return 'large';
  if (width >= breakpoints.medium) return 'medium';
  return 'small';
}

// Responsive styles based on screen size
function ResponsiveStyles() {
  const { width } = useWindowDimensions();
  const breakpoint = getBreakpoint(width);

  const dynamicStyles = {
    container: {
      padding: breakpoint === 'small' ? 10 : 20,
    },
    title: {
      fontSize: breakpoint === 'small' ? 18 : 24,
    },
  };

  return (
    &lt;View style={dynamicStyles.container}>
      &lt;Text style={dynamicStyles.title}>Responsive Title&lt;/Text&gt;
    &lt;/View>
  );
}

// Orientation handling
function OrientationAware() {
  const { width, height } = useWindowDimensions();
  const isPortrait = height > width;

  return (
    &lt;View style={{
      flexDirection: isPortrait ? 'column' : 'row',
    }}>
      &lt;Text>Content adapts to orientation&lt;/Text&gt;
    &lt;/View>
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">THEMING BASICS</span>
  <h3 style="margin-top: 0;">Q25: Implementing light/dark theme switching.</h3>

  <p>Themes allow users to customize the app appearance and support system-wide dark mode:</p>

  <div class="code-example">Theme Implementation</div>
  <pre class="code-block"><code class="language-javascript">// Theme definitions
const lightTheme = {
  colors: {
    background: '#ffffff',
    text: '#000000',
    primary: '#007AFF',
    secondary: '#6c757d',
  },
  spacing: {
    small: 8,
    medium: 16,
    large: 24,
  },
};

const darkTheme = {
  colors: {
    background: '#000000',
    text: '#ffffff',
    primary: '#0A84FF',
    secondary: '#98989D',
  },
  spacing: {
    small: 8,
    medium: 16,
    large: 24,
  },
};

// Theme context
const ThemeContext = React.createContext();

function ThemeProvider({ children }) {
  const [isDark, setIsDark] = useState(false);
  const theme = isDark ? darkTheme : lightTheme;

  return (
    &lt;ThemeContext.Provider value={{ theme, isDark, setIsDark }}>
      {children}
    &lt;/ThemeContext.Provider&gt;
  );
}

// Styled component with theme
function ThemedButton({ title, onPress }) {
  const { theme } = useContext(ThemeContext);

  return (
    &lt;TouchableOpacity
      style={{
        backgroundColor: theme.colors.primary,
        padding: theme.spacing.medium,
        borderRadius: 8,
      }}
      onPress={onPress}
    >
      &lt;Text style={{ color: theme.colors.background }}>
        {title}
      &lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}

// System theme detection
import { useColorScheme } from 'react-native';

function AutoThemeProvider({ children }) {
  const colorScheme = useColorScheme(); // 'light' or 'dark'
  const [isDark, setIsDark] = useState(colorScheme === 'dark');

  // Listen for system theme changes
  useEffect(() => {
    setIsDark(colorScheme === 'dark');
  }, [colorScheme]);

  const theme = isDark ? darkTheme : lightTheme;

  return (
    &lt;ThemeContext.Provider value={{ theme, isDark, setIsDark }}>
      {children}
    &lt;/ThemeContext.Provider&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">ACCESSIBILITY</span>
  <h3 style="margin-top: 0;">Q26: Making React Native apps accessible.</h3>

  <p>Accessibility ensures your app works for users with disabilities:</p>

  <div class="code-example">Accessibility Features</div>
  <pre class="code-block"><code class="language-javascript">// Screen reader support
&lt;TouchableOpacity
  accessible={true}
  accessibilityLabel="Add item to cart"
  accessibilityHint="Double tap to add this item to your shopping cart"
  accessibilityRole="button"
  onPress={addToCart}
>
  &lt;Text>Add to Cart&lt;/Text&gt;
&lt;/TouchableOpacity>

// Form accessibility
&lt;TextInput
  accessible={true}
  accessibilityLabel="Email address"
  accessibilityHint="Enter your email address for account creation"
  autoComplete="email"
  keyboardType="email-address"
  textContentType="emailAddress"
/>

// Images need alt text
&lt;Image
  source={logo}
  accessible={true}
  accessibilityLabel="Company logo"
  accessibilityRole="image"
/>

// Semantic grouping
&lt;View
  accessible={true}
  accessibilityLabel="User profile section"
  accessibilityRole="summary"
>
  &lt;Text accessibilityRole="header">John Doe&lt;/Text&gt;
  &lt;Text accessibilityRole="text">Software Developer&lt;/Text&gt;
&lt;/View>

// Minimum touch targets (44x44 points recommended)
const styles = StyleSheet.create({
  touchableArea: {
    minWidth: 44,
    minHeight: 44,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

// Font scaling support
&lt;Text
  style={{
    fontSize: 16, // Will scale with system font size
  }}
  allowFontScaling={true} // Default is true
  maxFontSizeMultiplier={2} // Limit maximum scaling
>
  Scalable text
&lt;/Text></code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Test your app with VoiceOver (iOS) or TalkBack (Android) enabled. Many accessibility issues are only discovered through actual usage with assistive technologies.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">PLATFORM DIFFERENCES</span>
  <h3 style="margin-top: 0;">Q27: Handling platform-specific styling and components.</h3>

  <p>iOS and Android have different design languages and capabilities:</p>

  <div class="code-example">Platform-Specific Code</div>
  <pre class="code-block"><code class="language-javascript">import { Platform, StyleSheet } from 'react-native';

// Platform detection
const isIOS = Platform.OS === 'ios';
const isAndroid = Platform.OS === 'android';

// Platform-specific styles
const styles = StyleSheet.create({
  container: {
    padding: 20,
    // Different shadows for each platform
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 3.84,
      },
      android: {
        elevation: 5,
      },
    }),
  },

  // Different fonts
  title: {
    fontFamily: Platform.select({
      ios: 'System',
      android: 'Roboto',
    }),
    fontSize: 24,
    fontWeight: 'bold',
  },
});

// Platform-specific components
function PlatformSpecificButton() {
  if (Platform.OS === 'ios') {
    return &lt;TouchableOpacity style={styles.iosButton} /&gt;;
  }

  return &lt;TouchableOpacity style={styles.androidButton} /&gt;;
}

// File-based platform specificity
// MyComponent.ios.js (iOS only)
// MyComponent.android.js (Android only)
// MyComponent.js (shared)

// Version-specific code
const styles = StyleSheet.create({
  container: {
    // Different padding for iOS versions
    paddingTop: Platform.Version >= 11 ? 0 : 20,
  },
});

// Safe area handling
import { SafeAreaView, useSafeAreaInsets } from 'react-native-safe-area-context';

function SafeAreaExample() {
  const insets = useSafeAreaInsets();

  return (
    &lt;SafeAreaView style={{
      flex: 1,
      paddingTop: insets.top,
      paddingBottom: insets.bottom,
      paddingLeft: insets.left,
      paddingRight: insets.right,
    }}>
      &lt;Text>Safe content&lt;/Text&gt;
    &lt;/SafeAreaView&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">DESIGN SYSTEMS</span>
  <h3 style="margin-top: 0;">Q28: Building and maintaining design systems.</h3>

  <p>Design systems ensure consistency across your app and team:</p>

  <div class="code-example">Design System Structure</div>
  <pre class="code-block"><code class="language-javascript">// 📁 design-system/
├── tokens/
│   ├── colors.js
│   ├── typography.js
│   ├── spacing.js
│   └── breakpoints.js
├── components/
│   ├── Button/
│   ├── Input/
│   ├── Card/
│   └── index.js
└── theme/
    ├── light.js
    └── dark.js

// Design tokens
export const colors = {
  primary: {
    50: '#eff6ff',
    500: '#3b82f6',
    900: '#1e3a8a',
  },
  gray: {
    50: '#f9fafb',
    500: '#6b7280',
    900: '#111827',
  },
};

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
};

export const typography = {
  h1: {
    fontSize: 32,
    fontWeight: 'bold',
    lineHeight: 40,
  },
  body: {
    fontSize: 16,
    lineHeight: 24,
  },
};

// Reusable components
function Button({ variant = 'primary', size = 'md', children, ...props }) {
  const themeStyles = useTheme();

  const variantStyles = {
    primary: {
      backgroundColor: themeStyles.colors.primary[500],
    },
    secondary: {
      backgroundColor: themeStyles.colors.gray[500],
    },
  };

  const sizeStyles = {
    sm: { padding: spacing.sm },
    md: { padding: spacing.md },
    lg: { padding: spacing.lg },
  };

  return (
    &lt;TouchableOpacity
      style={[variantStyles[variant], sizeStyles[size]]}
      {...props}
    >
      &lt;Text style={typography.body}>{children}&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}

// Usage
&lt;Button variant="primary" size="lg" onPress={handlePress}>
  Click me
&lt;/Button></code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q29: Style performance optimization techniques.</h3>

  <p>Styling can impact performance - optimize carefully:</p>

  <div class="code-example">Style Performance Tips</div>
  <pre class="code-block"><code class="language-javascript">// ✅ Good: StyleSheet.create for performance
const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  title: { fontSize: 24, fontWeight: 'bold' },
});

// ❌ Bad: Inline styles (recreated every render)
&lt;Text style={{
  fontSize: 24,
  fontWeight: 'bold',
  color: someColor,
}}>Title&lt;/Text>

// ✅ Good: Conditional styles without inline objects
const getTitleStyle = (isActive) => [
  styles.title,
  isActive && styles.titleActive,
];

&lt;Text style={getTitleStyle(isActive)}>Title&lt;/Text>

// ✅ Good: Flatten styles for complex combinations
const combinedStyles = StyleSheet.flatten([
  styles.base,
  isActive && styles.active,
  variant === 'large' && styles.large,
]);

// ❌ Bad: Array syntax creates new objects
&lt;Text style={[styles.base, isActive && styles.active]}>Text&lt;/Text>

// ✅ Good: Pre-computed conditional styles
const conditionalStyles = {
  active: StyleSheet.compose(styles.base, styles.active),
  inactive: styles.base,
};

// Performance monitoring
function useStylePerformance() {
  const renderCount = useRef(0);

  renderCount.current += 1;

  useEffect(() => {
    if (__DEV__ && renderCount.current > 10) {
      console.warn('Component re-rendering frequently - check styles');
    }
  });

  return renderCount.current;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">TESTING STYLES</span>
  <h3 style="margin-top: 0;">Q30: Testing styled components and design systems.</h3>

  <p>Test that your styles work correctly and maintain design consistency:</p>

  <div class="code-example">Testing Styling</div>
  <pre class="code-block"><code class="language-javascript">// Component tests
describe('Button', () => {
  it('applies correct variant styles', () => {
    const { getByText } = render(
      &lt;Button variant="primary"&gt;Click me&lt;/Button&gt;
    );

    const button = getByText('Click me').parent;
    expect(button).toHaveStyle({
      backgroundColor: theme.colors.primary[500],
    });
  });

  it('respects theme changes', () => {
    const { getByText, rerender } = render(
      &lt;ThemeProvider theme={lightTheme}>
        &lt;Button&gt;Test&lt;/Button&gt;
      &lt;/ThemeProvider&gt;
    );

    // Test light theme
    expect(getByText('Test').parent).toHaveStyle({
      backgroundColor: lightTheme.colors.primary,
    });

    // Change to dark theme
    rerender(
      &lt;ThemeProvider theme={darkTheme}>
        &lt;Button&gt;Test&lt;/Button&gt;
      &lt;/ThemeProvider&gt;
    );

    expect(getByText('Test').parent).toHaveStyle({
      backgroundColor: darkTheme.colors.primary,
    });
  });
});

// Design system tests
describe('Design Tokens', () => {
  it('maintains consistent spacing scale', () => {
    expect(spacing.sm).toBe(8);
    expect(spacing.md).toBe(16);
    expect(spacing.lg).toBe(24);
    expect(spacing.xl).toBe(32);
  });

  it('has valid color contrast ratios', () => {
    // Test that text colors meet WCAG contrast requirements
    const contrast = getContrastRatio(colors.primary[500], colors.gray[50]);
    expect(contrast).toBeGreaterThan(4.5); // WCAG AA standard
  });
});

// Snapshot tests for styles
describe('Style Snapshots', () => {
  it('maintains consistent button appearance', () => {
    const { getByText } = render(&lt;Button&gt;Snapshot&lt;/Button&gt;);
    expect(getByText('Snapshot').parent).toMatchSnapshot();
  });
});

// Responsive design tests
describe('Responsive Layouts', () => {
  it('adapts to different screen sizes', () => {
    const { rerender } = render(
      &lt;ResponsiveComponent /&gt;
    );

    // Mock small screen
    Dimensions.get = jest.fn().mockReturnValue({ width: 375, height: 667 });

    // Test layout changes
    expect(screen.getByTestId('container')).toHaveStyle({
      flexDirection: 'column',
    });

    // Mock large screen
    Dimensions.get = jest.fn().mockReturnValue({ width: 1024, height: 768 });
    rerender(&lt;ResponsiveComponent /&gt;);

    expect(screen.getByTestId('container')).toHaveStyle({
      flexDirection: 'row',
    });
  });
});</code></pre>
</div>

---

<div class="nav-footer">
  <a href="phase6-state-management.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 06: State</a>
  <a href="phase8-animations-gestures.md" style="color: #64748b; text-decoration: none; font-weight: 600;">Phase 08: Animations ➡️</a>
</div>

</div>