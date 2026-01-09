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
    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(16, 185, 129, 0.2);
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
    background: #ecfdf5;
    color: #059669;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #f0fdfa;
    border-left: 4px solid #14b8a6;
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
  <span class="phase-number">Phase 04</span>
  <h1 class="phase-title">Performance & Memory Management</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Profiling, Optimization, Memory Leaks, and Threading</p>
</div>

---

### 📋 Phase Overview
Master React Native performance optimization from basic list rendering to advanced memory management and threading. Learn to profile, debug, and optimize your apps for smooth 60fps experiences across all devices.

---

<div class="qa-card">
  <span class="question-tag">LIST RENDERING</span>
  <h3>4.1 FlatList vs FlashList: The Recycling Revolution</h3>
  
  **Question:** *"Why is Shopify's FlashList preferred over FlatList for high-performance apps, and how does it work internally?"*

  #### 🔄 Cell Recycling vs. Destruction
  - **FlatList**: Destroys off-screen components and re-creates them as the user scrolls. This causes "blanking" during fast scrolls because JS can't keep up with component creation.
  - **FlashList**: Reuses (recycles) the underlying native views. It only updates the data (props) of an existing view, which is significantly faster than creating a new one.

  <div class="senior-insight">
    <strong>Senior Insight: Estimated Item Size</strong><br/>
    The most critical prop in FlashList is <code>estimatedItemSize</code>. If this is inaccurate, the list will "jump" or flicker as it re-calculates layout on the fly. For best results, use the average height of your items across all devices.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ANIMATIONS</span>
  <h3>4.2 Off-Main-Thread Animations</h3>

  **Question:** *"Explain the difference between the 'Native Driver' in Animated API and Reanimated 3."*

  #### ⚡ Animation Engines
  1. **Animated (Native Driver)**: Can only animate non-layout properties (Opacity, Transform). It serializes the animation once and runs it on the Native UI thread.
  2. **Reanimated 3 (Worklets)**: Uses JSI to run JS code (Worklets) on a dedicated **Secondary JS Thread**. This allows animating *layout* properties (Width, Height, Flex) without blocking the main JS thread.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Why do we need 'useSharedValue' instead of regular 'useState' for animations?"</em><br/>
    <strong>Answer:</strong> <code>useState</code> updates trigger a full React re-render, which is slow. <code>useSharedValue</code> is a JSI-based reference that can be updated 60 times per second without ever touching the React render cycle.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">PROFILING</span>
  <h3>4.3 Memory Leak Identification</h3>

  **Question:** *"How do you identify and fix a memory leak in a production React Native app?"*

  #### 🔍 The Senior Toolkit
  1. **Hermes Heap Snapshot**: Take snapshots before and after an action (e.g., opening/closing a screen). Look for objects that are never garbage collected.
  2. **Allocation Tracking**: Identify which objects are consuming the most memory in real-time.
  3. **LeakCanary (Android)**: Can be integrated into dev builds to catch native memory leaks automatically.

  <div class="senior-insight">
    <strong>Senior Insight: The Cleanup Pattern</strong><br/>
    Always audit your <code>useEffect</code> hooks. 90% of leaks come from forgotten <code>eventEmitter.remove()</code>, <code>clearInterval()</code>, or <code>subscription.unsubscribe()</code> calls.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">STARTUP</span>
  <h3>4.4 TTI & Startup Optimization</h3>

  **Question:** *"What strategies do you use to reduce app startup time (Time To Interactive)?"*

  #### 🚀 Optimization Strategies:
  1. **Hermes Engine**: Uses AOT (Ahead-of-Time) compilation to turn JS into bytecode during build time, skipping the expensive parsing phase on startup.
  2. **Inline Requires**: Use `babel-plugin-transform-inline-imports-commonjs` to only load modules when they are first called, rather than all at once.
  3. **Asset Optimization**: Pre-fetching critical data and images *after* the initial render to keep the splash screen duration minimal.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What is the danger of large 'index.js' barrel exports?"</em><br/>
    <strong>Answer:</strong> Importing one item from a file with 100 exports forces Metro to load and parse all 100 dependencies, significantly increasing TTI.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">LAYOUT</span>
  <h3>4.5 Yoga Engine Internals</h3>

  **Question:** *"How does the Yoga engine calculate layout, and why does 'over-nesting' hurt performance?"*

  #### 📐 Layout Calculation
  Yoga is a C++ library that implements Flexbox. When you update a prop, Yoga performs a recursive tree traversal to calculate sizes and positions. Deeply nested views (15+ levels) cause exponential increases in layout calculation time.

  <div class="senior-insight">
    <strong>Senior Insight: Flattening the Tree</strong><br/>
    Use tools like the **Inspector** to identify deep trees. Often, multiple <code>View</code> wrappers can be replaced with a single <code>View</code> using complex styles or <code>Fragment</code>.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">THREADS</span>
  <h3>4.6 JS Thread vs. UI Thread</h3>

  **Question:** *"What happens when the JS thread is blocked, and how does it differ from the UI thread being blocked?"*

  #### 🧵 Thread Comparison
  - **JS Thread Blocked**: UI still responds to touches (e.g., a button highlight), but nothing *happens* (no navigation, no data update).
  - **UI Thread Blocked**: The entire app freezes. No animations, no scrolls, no touch highlights. This is much worse for UX.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How do you offload heavy JS tasks?"</em><br/>
    <strong>Answer:</strong> Use <code>InteractionManager.runAfterInteractions()</code> for tasks that can wait, or <strong>Worklets</strong> for tasks that need to run on the UI thread.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">DEFERRING</span>
  <h3>4.7 InteractionManager and Post-Animation Tasks</h3>

  **Question:** *"When and why would you use InteractionManager.runAfterInteractions()?"*

  #### ⏳ Deferring Tasks
  React Native animations (like navigation transitions) need the JS thread to be free to coordinate frames. If you trigger a heavy data fetch or complex state update *during* a transition, you'll see "jank." `runAfterInteractions` defers the task until the animation is complete.

  <div class="senior-insight">
    <strong>Senior Insight: The 500ms Rule</strong><br/>
    If a task takes longer than 500ms, even <code>runAfterInteractions</code> might feel like a delay. In such cases, use a <code>Skeleton Loader</code> to give the user immediate feedback.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">IMAGES</span>
  <h3>4.8 Advanced Image Optimization</h3>

  **Question:** *"How do you handle a list of 1000+ high-res images without crashing the app?"*

  #### 🖼️ Image Strategies
  1. **FastImage**: Uses SDWebImage (iOS) and Glide (Android) for aggressive disk/memory caching.
  2. **ResizeMethod**: On Android, set `resizeMethod="resize"` to reduce memory usage for large images.
  3. **WebP Format**: Use WebP for 30% smaller file sizes compared to PNG/JPG with similar quality.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Why shouldn't you use original 4K images for thumbnails?"</em><br/>
    <strong>Answer:</strong> Even if the display size is 50x50, the full image is decoded into memory. A 4K image can take 24MB of RAM just for decoding.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">BUNDLE</span>
  <h3>4.9 Bundle Analysis & Tree Shaking</h3>

  **Question:** *"How do you audit your JS bundle size, and does React Native support tree shaking?"*

  #### 📦 Bundle Auditing
  Use `react-native-bundle-visualizer` to see which libraries are bloating your bundle. While Metro doesn't support full tree-shaking like Webpack, it does support basic dead-code elimination.

  <div class="senior-insight">
    <strong>Senior Insight: Avoid Lodash/Moment</strong><br/>
    Replace heavy libraries like <code>moment</code> with <code>date-fns</code> and <code>lodash</code> with native ES6 methods to save hundreds of KB in your bundle.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">HERMES</span>
  <h3>4.10 Hermes Garbage Collection (GC)</h3>

  **Question:** *"How does Hermes' generational garbage collector improve performance?"*

  #### ♻️ Generational GC
  Hermes divides memory into "Young" and "Old" generations. It assumes most objects die young. By scanning the "Young" generation frequently and the "Old" one rarely, it minimizes "stop-the-world" pauses that cause dropped frames.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Can you trigger GC manually?"</em><br/>
    <strong>Answer:</strong> No, and you shouldn't. However, you can use <code>gc()</code> in the Hermes debugger for testing.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">RE-RENDERS</span>
  <h3>4.11 Referential Identity & Memoization</h3>

  **Question:** *"Why does 'useCallback' sometimes make performance WORSE?"*

  #### 🔄 The Memoization Overhead
  Memoization isn't free. `useCallback` and `useMemo` have their own execution cost and memory overhead. If the child component is cheap to re-render, adding memoization might actually slow down the app.

  <div class="senior-insight">
    <strong>Senior Insight: The 90% Rule</strong><br/>
    Only memoize components that:
    1. Render complex UI (Lists, Charts).
    2. Are frequently re-rendered by parent state changes.
    3. Have heavy computational logic in the render body.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">BRIDGE</span>
  <h3>4.12 Bridge Traffic & Serialization</h3>

  **Question:** *"What is 'Bridge Traffic', and how does it affect app responsiveness?"*

  #### 🌉 The Serialization Bottleneck
  In the legacy architecture, every call between JS and Native is serialized into JSON and sent across the bridge. If you send large objects (like 1MB JSON) or send messages too frequently (like 60 times/sec for scroll events), the bridge becomes a bottleneck.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How does JSI solve this?"</em><br/>
    <strong>Answer:</strong> JSI allows direct C++ function calls, eliminating the need for JSON serialization entirely.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">JSI</span>
  <h3>4.13 JSI Synchronous Calls</h3>

  **Question:** *"What are the performance benefits of JSI's synchronous execution?"*

  #### ⚡ Instant Access
  With the bridge, getting a value from native always requires an `await`. With JSI, you can get it synchronously. This is critical for animations or real-time data where an asynchronous delay would cause visible "lag."

  <div class="senior-insight">
    <strong>Senior Insight: Use with Caution</strong><br/>
    Synchronous calls block the JS thread. Never perform heavy I/O or network requests synchronously via JSI.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ANIMATIONS</span>
  <h3>4.14 LayoutAnimations vs. Reanimated</h3>

  **Question:** *"When would you use the built-in LayoutAnimation over Reanimated?"*

  #### 🏗️ Layout Transitions
  `LayoutAnimation` is great for simple "automatic" transitions (e.g., items moving when one is deleted). It runs entirely on the native side. However, it's difficult to customize and lacks the fine-grained control of Reanimated's Shared Values.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Is LayoutAnimation thread-safe?"</em><br/>
    <strong>Answer:</strong> It runs on the UI thread, making it very smooth, but it can sometimes conflict with other animations.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">PROFILING</span>
  <h3>4.15 Advanced Profiling: Flipper & Flashlight</h3>

  **Question:** *"How do you use Flipper and Flashlight to measure performance in a CI/CD pipeline?"*

  #### 📊 Automated Profiling
  - **Flipper**: Used for manual inspection of network, logs, and layout.
  - **Flashlight**: A CLI tool that can be used in CI/CD to measure TTI, FPS, and CPU usage on real devices, giving you a "Performance Score" for every PR.

  <div class="senior-insight">
    <strong>Senior Insight: Real Device Testing</strong><br/>
    Never trust Simulator/Emulator performance. Always profile on a low-end Android device (e.g., Moto G) to see the true performance of your app.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">GRAPHICS</span>
  <h3>4.16 SVG vs. PNG in Large Lists</h3>

  **Question:** *"Why might SVGs cause performance issues in a FlatList with 100 items?"*

  #### 🎨 CPU vs. GPU
  - **PNGs**: Decoded once and handled by the GPU. Very efficient for lists.
  - **SVGs**: The `react-native-svg` library has to parse the XML paths and draw them on the CPU for every item. In a fast-scrolling list, this can max out the CPU and cause jank.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How to optimize SVGs?"</em><br/>
    <strong>Answer:</strong> Use **Icon Fonts** for simple icons or convert SVGs to PNG sprites for complex ones used in large lists.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">COMPUTATION</span>
  <h3>4.17 Heavy Computations: Worklets & C++</h3>

  **Question:** *"How do you handle heavy data processing (like image filtering) without freezing the UI?"*

  #### ⚙️ Offloading Work
  1. **Worklets**: Run on the UI thread (good for UI-related calculations).
  2. **Native Modules (C++)**: Perform the processing in C++ and return only the result.
  3. **Web Workers**: While not natively supported, libraries like `react-native-multithreading` can enable worker-like behavior.

  <div class="senior-insight">
    <strong>Senior Insight: Data Serialization</strong><br/>
    Moving large amounts of data between threads can be slow. Try to keep the data on the native side as much as possible and only pass references.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">JSON</span>
  <h3>4.18 Large JSON Parsing Performance</h3>

  **Question:** *"Why is JSON.parse() dangerous for 5MB+ payloads, and how do you optimize it?"*

  #### 📉 Parsing Overhead
  `JSON.parse()` is synchronous and blocking. Parsing a 5MB JSON can freeze the JS thread for 100ms+. 
  **Optimization**: 
  1. Use pagination/GraphQL to fetch smaller chunks.
  2. Use a native module to parse JSON in the background and return a JSI reference.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What about JSON.stringify?"</em><br/>
    <strong>Answer:</strong> Equally dangerous for large objects, especially when sending data across the bridge.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ASSETS</span>
  <h3>4.19 Asset Pre-loading & Caching</h3>

  **Question:** *"How do you implement a robust pre-loading strategy for fonts and images?"*

  #### 📥 Pre-loading
  Use `Asset.loadAsync` (Expo) or manual pre-fetching. For fonts, ensure they are loaded before the first render to avoid "FOIT" (Flash of Invisible Text). For images, use a priority queue to load "above-the-fold" images first.

  <div class="senior-insight">
    <strong>Senior Insight: Priority Loading</strong><br/>
    Don't pre-load everything. Pre-load only the assets needed for the first 3 screens to keep the initial download size and startup time low.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">NETWORK</span>
  <h3>4.20 Network Caching & Perceived Performance</h3>

  **Question:** *"How do RTK Query or TanStack Query improve perceived performance?"*

  #### ⚡ Optimistic UI
  These libraries provide "Stale-While-Revalidate" caching. The user sees old data immediately while the app fetches new data in the background. Combined with **Optimistic Updates**, the app feels instantaneous.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What is the risk of optimistic updates?"</em><br/>
    <strong>Answer:</strong> You must handle rollback logic if the network request fails, or the UI will be out of sync with the server.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">STABILITY</span>
  <h3>4.21 Global Error Boundaries & Crash Reporting</h3>

  **Question:** *"How do you ensure a single JS error doesn't crash the entire app for a user?"*

  #### 🛡️ Error Resilience
  Implement a `Global Error Boundary` at the root. Catch errors, log them to **Sentry** with the full breadcrumb trail, and show a "Friendly Fallback UI" that allows the user to restart the app or go back to the home screen.

<div class="qa-card">
  <span class="question-tag">PERFORMANCE BASICS</span>
  <h3 style="margin-top: 0;">Q1: Why is React Native performance important?</h3>

  <p>React Native performance directly impacts user experience, app store ratings, and business metrics:</p>

  <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 16px; margin: 24px 0;">
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <strong>📱 User Experience</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem;">Slow apps frustrate users, leading to abandonment</p>
    </div>
    <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px;">
      <strong>⭐ App Store Ratings</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem;">Performance issues lead to poor reviews</p>
    </div>
    <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px;">
      <strong>💰 Business Impact</strong>
      <p style="margin: 8px 0 0 0; font-size: 0.875rem;">Better performance = higher retention and revenue</p>
    </div>
  </div>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Performance isn't optional - it's a core requirement. Always profile and optimize before considering features "done".
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">THREADING MODEL</span>
  <h3 style="margin-top: 0;">Q2: Understanding React Native's threading architecture.</h3>

  <p>React Native uses multiple threads to keep your app responsive:</p>

  <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🧵 React Native Threads:</strong>
    <ol style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Main Thread:</strong> Native UI rendering and user interactions</li>
      <li><strong>JS Thread:</strong> JavaScript execution and business logic</li>
      <li><strong>Shadow Thread:</strong> Layout calculations (Yoga layout engine)</li>
      <li><strong>Native Modules Thread:</strong> Background tasks like file I/O</li>
    </ol>
  </div>

  <div class="code-example">Threading Best Practices</div>
  <pre class="code-block"><code class="language-javascript">// ✅ Keep JS thread free for UI
function GoodComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Move heavy work to background
    const processData = async () => {
      const result = await heavyComputation(); // Off main thread
      setData(result);
    };
    processData();
  }, []);

  return &lt;Text&gt;Data: {data}&lt;/Text&gt;;
}

// ❌ Block JS thread (bad!)
function BadComponent() {
  const handlePress = () => {
    // This blocks UI for 5 seconds!
    const result = heavyComputation(); // Synchronous on JS thread
    setData(result);
  };

  return &lt;TouchableOpacity onPress={handlePress} /&gt;;
}

// Use InteractionManager for post-animation tasks
import { InteractionManager } from 'react-native';

function AnimationComponent() {
  const [animating, setAnimating] = useState(false);

  const handlePress = () => {
    setAnimating(true);

    // Animation completes first
    Animated.timing(opacity, {
      toValue: 0,
      duration: 300,
    }).start(() => {
      // Then run heavy task
      InteractionManager.runAfterInteractions(() => {
        heavyTask(); // Won't block animation
      });
    });
  };

  return &lt;Animated.View style={{ opacity }} /&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">BRIDGE COMMUNICATION</span>
  <h3 style="margin-top: 0;">Q3: How does the React Native bridge work and why is it expensive?</h3>

  <p>The bridge is the communication layer between JavaScript and native code:</p>

  <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🌉 Bridge Communication Flow:</strong>
    <ol style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>JS Thread:</strong> JavaScript code runs here</li>
      <li><strong>Serialization:</strong> Data converted to JSON strings</li>
      <li><strong>Async Queue:</strong> Messages sent through single-channel pipe</li>
      <li><strong>Deserialization:</strong> JSON parsed back to native objects</li>
      <li><strong>Main Thread:</strong> Native UI updates applied</li>
    </ol>
  </div>

  <div class="code-example">Bridge Performance Issues</div>
  <pre class="code-block"><code class="language-javascript">// ❌ Expensive: Frequent bridge communication
function BadList({ data }) {
  return data.map((item, index) => (
    &lt;TouchableOpacity
      key={index}
      onPress={() => {
        // Each onPress creates bridge communication
        Vibration.vibrate(100); // Native module call
        Alert.alert('Pressed', item.title); // Another bridge call
      }}
    >
      &lt;Text&gt;{item.title}&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  ));
}

// ✅ Better: Batch bridge communications
function GoodList({ data }) {
  const [selectedItem, setSelectedItem] = useState(null);

  useEffect(() => {
    if (selectedItem) {
      // Single bridge communication batch
      Vibration.vibrate(100);
      Alert.alert('Pressed', selectedItem.title);
      setSelectedItem(null);
    }
  }, [selectedItem]);

  return data.map((item, index) => (
    &lt;TouchableOpacity
      key={index}
      onPress={() => setSelectedItem(item)} // Only state update
    >
      &lt;Text&gt;{item.title}&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  ));
}</code></pre>

  <div class="follow-up-trap">
    <strong>🪤 Follow-up Trap:</strong> <em>"Does the New Architecture eliminate the bridge?"</em><br/>
    <strong>Answer:</strong> No, the bridge still exists but JSI (JavaScript Interface) allows synchronous communication for performance-critical operations, reducing bridge usage by 80-90%.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">RE-RENDERING ISSUES</span>
  <h3 style="margin-top: 0;">Q4: Preventing unnecessary re-renders in React Native.</h3>

  <p>Unnecessary re-renders are the #1 performance killer in React Native:</p>

  <div class="code-example">Re-render Optimization Techniques</div>
  <pre class="code-block"><code class="language-javascript">// 1. React.memo for component memoization
const ListItem = React.memo(({ item, onPress }) => (
  &lt;TouchableOpacity onPress={() => onPress(item.id)}>
    &lt;Text&gt;{item.name}&lt;/Text&gt;
  &lt;/TouchableOpacity&gt;
));

// 2. useMemo for expensive calculations
const processedData = useMemo(() => {
  return data.filter(item => item.active).map(item => ({
    ...item,
    processed: expensiveCalculation(item)
  }));
}, [data]);

// 3. useCallback for stable function references
const handlePress = useCallback((id) => {
  doSomething(id);
}, []); // Empty deps = stable function reference

// 4. Avoid inline objects/functions in render
function BadComponent({ user }) {
  return (
    &lt;View&gt;
      {/* ❌ Creates new object every render */}
      &lt;ChildComponent style={{ margin: 10 }} /&gt;
      {/* ❌ Creates new function every render */}
      &lt;TouchableOpacity onPress={() => handleUser(user.id)} /&gt;
    &lt;/View&gt;
  );
}

function GoodComponent({ user }) {
  const handlePress = useCallback(() => handleUser(user.id), [user.id]);
  const styles = useMemo(() => ({ margin: 10 }), []);

  return (
    &lt;View&gt;
      {/* ✅ Stable object reference */}
      &lt;ChildComponent style={styles} /&gt;
      {/* ✅ Stable function reference */}
      &lt;TouchableOpacity onPress={handlePress} /&gt;
    &lt;/View&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Use React DevTools Profiler to identify re-render causes. Look for components that render more than expected.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">LIST PERFORMANCE</span>
  <h3 style="margin-top: 0;">Q5: Optimizing FlatList and ScrollView performance.</h3>

  <p>Lists are often the biggest performance bottleneck in React Native apps:</p>

  <div class="code-example">FlatList Performance Optimization</div>
  <pre class="code-block"><code class="language-javascript">// ✅ Optimized FlatList configuration
function OptimizedList({ data }) {
  const renderItem = useCallback(({ item, index }) => (
    &lt;ListItem
      item={item}
      index={index}
      onPress={handlePress}
    />
  ), [handlePress]);

  const keyExtractor = useCallback((item, index) => {
    // Use stable keys, not index
    return item.id?.toString() || index.toString();
  }, []);

  const getItemLayout = useCallback((data, index) => ({
    length: ITEM_HEIGHT, // Exact height for better scrolling
    offset: ITEM_HEIGHT * index,
    index,
  }), []);

  return (
    &lt;FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      // Performance props
      initialNumToRender={10}        // Render first 10 items
      maxToRenderPerBatch={10}       // Batch size for rendering
      windowSize={10}                // Items to keep in memory
      removeClippedSubviews={true}   // Remove off-screen views
      // User experience
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      refreshing={refreshing}
      onRefresh={handleRefresh}
      // Optimization
      ListEmptyComponent={&lt;EmptyState /&gt;}
      ListFooterComponent={loading ? &lt;Spinner /&gt; : null}
    /&gt;
  );
}

// Memoized list items
const ListItem = React.memo(({ item, index, onPress }) => {
  const handlePress = useCallback(() => {
    onPress(item.id);
  }, [item.id, onPress]);

  return (
    &lt;TouchableOpacity
      style={styles.item}
      onPress={handlePress}
      activeOpacity={0.7}
    >
      &lt;Text style={styles.title}>{item.title}&lt;/Text&gt;
      &lt;Text style={styles.subtitle}>{item.subtitle}&lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
});

// When to use ScrollView vs FlatList
function PerformanceComparison() {
  // ✅ Use FlatList for:
  // - Large datasets (>50 items)
  // - Variable item heights
  // - Need for pull-to-refresh
  // - Need for infinite scroll

  // ✅ Use ScrollView for:
  // - Small, fixed content
  // - Complex layouts with mixed components
  // - Need for horizontal scrolling
  // - Need for scroll position control

  return (
    &lt;ScrollView&gt;
      &lt;Header /&gt;
      &lt;Content /&gt;
      &lt;Footer /&gt;
    &lt;/ScrollView&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">IMAGE OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q6: Optimizing images for better performance.</h3>

  <p>Images are often the biggest performance bottleneck in React Native apps:</p>

  <div class="code-example">Image Optimization Techniques</div>
  <pre class="code-block"><code class="language-javascript">// 1. Use appropriate sizes
const screenWidth = Dimensions.get('window').width;

// ❌ Bad: Loading full-size image on small screen
&lt;Image source={{uri: 'https://example.com/large-image.jpg'}} /&gt;

// ✅ Good: Resize image based on screen size
const imageSize = screenWidth > 768 ? 'large' : 'small';
&lt;Image source={{uri: `https://example.com/image-${imageSize}.jpg`}} /&gt;

// 2. Use FastImage for better performance
import FastImage from 'react-native-fast-image';

&lt;FastImage
  style={styles.image}
  source={{
    uri: 'https://example.com/image.jpg',
    priority: FastImage.priority.normal,
  }}
  resizeMode={FastImage.resizeMode.contain}
/&gt;

// 3. Implement image caching
const cacheKey = 'user-avatar';
const cachedImage = await CacheManager.get(cacheKey).getPath();

if (cachedImage) {
  // Use cached image
} else {
  // Download and cache
  await CacheManager.get(cacheKey).getPath(imageUrl);
}

// 4. Lazy loading for lists
function LazyImage({ source, style }) {
  const [loaded, setLoaded] = useState(false);

  return (
    &lt;View style={style}>
      {!loaded && &lt;ActivityIndicator style={StyleSheet.absoluteFill} /&gt;}
      &lt;Image
        source={source}
        style={StyleSheet.absoluteFill}
        onLoad={() => setLoaded(true)}
      /&gt;
    &lt;/View&gt;
  );
}

// 5. Progressive image loading
function ProgressiveImage({ source, style }) {
  const [currentSource, setCurrentSource] = useState(source);

  useEffect(() => {
    // Load low quality image first
    const lowQuality = source.uri.replace('/original/', '/thumbnail/');
    setCurrentSource({ uri: lowQuality });

    // Then load high quality
    Image.prefetch(source.uri).then(() => {
      setCurrentSource(source);
    });
  }, [source]);

  return &lt;Image source={currentSource} style={style} /&gt;;
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always compress images before including them in your app. Use tools like ImageOptim or TinyPNG. For network images, consider using a CDN that automatically optimizes images.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">MEMORY MANAGEMENT</span>
  <h3 style="margin-top: 0;">Q7: Common memory leaks and how to prevent them.</h3>

  <p>Memory leaks in React Native can crash your app. Here are the most common causes:</p>

  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin: 24px 0;">
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Event Listeners</h4>
      <p style="margin: 0; font-size: 0.875rem;">Forgetting to remove event listeners from native modules.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Timers</h4>
      <p style="margin: 0; font-size: 0.875rem;">setTimeout/setInterval not cleared on unmount.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Subscriptions</h4>
      <p style="margin: 0; font-size: 0.875rem;">Not unsubscribing from observables or streams.</p>
    </div>
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Closures</h4>
      <p style="margin: 0; font-size: 0.875rem;">Capturing large objects in useEffect dependencies.</p>
    </div>
  </div>

  <div class="code-example">Preventing Memory Leaks</div>
  <pre class="code-block"><code class="language-javascript">// ❌ LEAK: Timer not cleared
function BadTimerComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);

    // No cleanup! Timer keeps running after unmount
  }, []);

  return &lt;Text&gt;Count: {count}&lt;/Text&gt;;
}

// ✅ FIXED: Proper cleanup
function GoodTimerComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);

    return () => clearInterval(timer); // Cleanup on unmount
  }, []);

  return &lt;Text&gt;Count: {count}&lt;/Text&gt;;
}

// ❌ LEAK: Event listener not removed
function BadEventComponent() {
  useEffect(() => {
    const subscription = DeviceEventEmitter.addListener('shake', handleShake);
    // No cleanup! Listener stays active
  }, []);

  return &lt;Text&gt;Shake me!&lt;/Text&gt;;
}

// ✅ FIXED: Proper cleanup
function GoodEventComponent() {
  useEffect(() => {
    const subscription = DeviceEventEmitter.addListener('shake', handleShake);

    return () => subscription.remove(); // Cleanup on unmount
  }, []);

  return &lt;Text&gt;Shake me!&lt;/Text&gt;;
}

// ❌ LEAK: Async operation not cancelled
function BadAsyncComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    let isCancelled = false;

    fetchData().then(result => {
      if (!isCancelled) {  // Check if component still mounted
        setData(result);
      }
    });

    // No cleanup for async operation
  }, []);

  return &lt;Text&gt;{data}&lt;/Text&gt;;
}

// ✅ FIXED: Cancel async operations
function GoodAsyncComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    let isCancelled = false;

    fetchData().then(result => {
      if (!isCancelled) {
        setData(result);
      }
    });

    return () => {
      isCancelled = true; // Cancel on unmount
    };
  }, []);

  return &lt;Text&gt;{data}&lt;/Text&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">BUNDLE SIZE OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q8: Reducing bundle size and improving loading performance.</h3>

  <p>Large bundles slow down app startup. Here's how to optimize:</p>

  <div class="code-example">Bundle Size Optimization</div>
  <pre class="code-block"><code class="language-javascript">// 1. Tree shaking - remove unused code
// Metro automatically tree shakes, but help it:
import { View, Text } from 'react-native'; // Only import what you use
// Avoid: import * as RN from 'react-native';

// 2. Dynamic imports for code splitting
const LazyHeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    &lt;Suspense fallback={&lt;Loading /&gt;}>
      &lt;LazyHeavyComponent /&gt;
    &lt;/Suspense&gt;
  );
}

// 3. Remove development code in production
if (__DEV__) {
  console.log('Debug info');
}

// 4. Use smaller alternatives
// Instead of moment.js (large), use date-fns
import { format, addDays } from 'date-fns';

// Instead of lodash, use lodash-es or specific functions
import debounce from 'lodash/debounce'; // Only import what you need

// 5. Optimize assets
// Compress images, convert to WebP
// Use vector icons instead of PNG icons
// Lazy load non-critical assets

// 6. Bundle analyzer
// npx react-native-bundle-analyzer

// 7. Remove console.logs in production
// metro.config.js
const config = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true,
      },
    }),
    minifierConfig: {
      compress: {
        drop_console: true, // Remove console.logs
        drop_debugger: true,
      },
    },
  },
};</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">STARTUP OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q9: Optimizing app startup time (TTI - Time to Interactive).</h3>

  <p>Users expect apps to launch quickly. Here's how to optimize startup:</p>

  <div class="code-example">Startup Optimization Strategies</div>
  <pre class="code-block"><code class="language-javascript">// 1. Lazy load heavy components
const LazyHeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    &lt;Suspense fallback={&lt;SplashScreen /&gt;}>
      &lt;LazyHeavyComponent /&gt;
    &lt;/Suspense&gt;
  );
}

// 2. Preload critical resources
function App() {
  useEffect(() => {
    // Preload fonts
    Font.loadAsync({
      'inter-bold': require('./assets/fonts/Inter-Bold.ttf'),
    });

    // Preload images
    Image.prefetch('https://example.com/hero-image.jpg');

    // Preload data
    preloadCriticalData();
  }, []);

  return &lt;RootNavigator /&gt;;
}

// 3. Use Hermes for faster startup
// Enable in app.json
{
  "expo": {
    "jsEngine": "hermes"
  }
}

// 4. Optimize Metro bundler
// metro.config.js
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true, // Reduces bundle size
      },
    }),
  },
};

// 5. Minimize initial render
function App() {
  // Don't render heavy components initially
  const [showHeavyComponent, setShowHeavyComponent] = useState(false);

  useEffect(() => {
    // Show heavy component after initial render
    const timer = setTimeout(() => {
      setShowHeavyComponent(true);
    }, 100);

    return () => clearTimeout(timer);
  }, []);

  return (
    &lt;View&gt;
      &lt;LightweightHeader /&gt;
      {showHeavyComponent && &lt;HeavyDashboard /&gt;}
    &lt;/View&gt;
  );
}

// 6. Use FlatList with initialNumToRender={0}
function OptimizedList() {
  return (
    &lt;FlatList
      data={data}
      initialNumToRender={0} // Don't render anything initially
      renderItem={renderItem}
      // Render items progressively
      onEndReached={() => {/* Load more */}}
    /&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PROFILING TOOLS</span>
  <h3 style="margin-top: 0;">Q10: Essential profiling tools for React Native.</h3>

  <p>Use these tools to identify and fix performance issues:</p>

  <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🔧 Essential Profiling Tools:</strong>
    <ul style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Flipper:</strong> Facebook's debugging platform with React DevTools, Network, and Layout inspectors</li>
      <li><strong>React DevTools:</strong> Component tree inspection, performance profiling, and highlighting</li>
      <li><strong>Chrome DevTools:</strong> Network tab, Performance tab, Memory tab for JS profiling</li>
      <li><strong>Systrace:</strong> Android system-level performance tracing</li>
      <li><strong>Xcode Instruments:</strong> iOS performance analysis and memory debugging</li>
      <li><strong>Hermes Debugger:</strong> JavaScript debugging with performance insights</li>
    </ul>
  </div>

  <div class="code-example">Using Performance Monitor</div>
  <pre class="code-block"><code class="language-javascript">// Custom performance hook
function usePerformanceMonitor(componentName) {
  const renderCount = useRef(0);
  const startTime = useRef(performance.now());

  renderCount.current += 1;

  useEffect(() => {
    const renderTime = performance.now() - startTime.current;

    if (__DEV__) {
      console.log(`${componentName} render #${renderCount.current}: ${renderTime.toFixed(2)}ms`);
    }

    // Production monitoring
    if (!__DEV__ && renderTime > 100) {
      analytics.track('SlowRender', {
        component: componentName,
        renderTime,
        renderCount: renderCount.current
      });
    }
  });

  return renderCount.current;
}

// Usage
function ExpensiveComponent() {
  const renderCount = usePerformanceMonitor('ExpensiveComponent');

  return &lt;Text&gt;Renders: {renderCount}&lt;/Text&gt;;
}

// Performance monitoring in production
function PerformanceTracker() {
  const [metrics, setMetrics] = useState({
    fps: 60,
    memoryUsage: 0,
    renderTime: 0,
  });

  useEffect(() => {
    const interval = setInterval(() => {
      // Collect performance metrics
      const newMetrics = {
        fps: calculateFPS(),
        memoryUsage: getMemoryUsage(),
        renderTime: getLastRenderTime(),
      };

      setMetrics(newMetrics);

      // Send to analytics if performance is poor
      if (newMetrics.fps < 30) {
        analytics.track('PoorPerformance', newMetrics);
      }
    }, 5000); // Check every 5 seconds

    return () => clearInterval(interval);
  }, []);

  return null; // Invisible tracker
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">REACT 18 PERFORMANCE</span>
  <h3 style="margin-top: 0;">Q11: React 18 concurrent features and performance improvements.</h3>

  <p>React 18 brings significant performance improvements to React Native:</p>

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
}

// Suspense for data fetching
function DataComponent() {
  const [resource, setResource] = useState(null);

  useEffect(() => {
    setResource(createResource(fetchData()));
  }, []);

  return (
    &lt;Suspense fallback={&lt;Text&gt;Loading...&lt;/Text&gt;}>
      &lt;DataView resource={resource} /&gt;
    &lt;/Suspense&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">NEW ARCHITECTURE PERFORMANCE</span>
  <h3 style="margin-top: 0;">Q12: Performance benefits of React Native's New Architecture.</h3>

  <p>The New Architecture (Fabric + TurboModules) significantly improves performance:</p>

  <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🚀 New Architecture Benefits:</strong>
    <ul style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Fabric:</strong> Async rendering, priority-based updates, reduced bridge communication</li>
      <li><strong>TurboModules:</strong> Synchronous native calls, lazy loading, type safety</li>
      <li><strong>JSI:</strong> Direct JS-native communication without serialization</li>
      <li><strong>Codegen:</strong> Compile-time optimization, better tree-shaking</li>
    </ul>
  </div>

  <div class="code-example">New Architecture Performance Patterns</div>
  <pre class="code-block"><code class="language-javascript">// TurboModule for synchronous native calls
import { NativeModules } from 'react-native';

const { HeavyComputation } = NativeModules;

// Old Bridge (async, expensive)
function OldWay() {
  const [result, setResult] = useState(null);

  const compute = async () => {
    const res = await HeavyComputation.process(data); // Bridge round trip
    setResult(res);
  };

  return &lt;TouchableOpacity onPress={compute} /&gt;;
}

// New Architecture (sync, fast)
function NewWay() {
  const [result, setResult] = useState(null);

  const compute = () => {
    const res = HeavyComputation.process(data); // Direct JSI call
    setResult(res);
  };

  return &lt;TouchableOpacity onPress={compute} /&gt;;
}

// Fabric enables concurrent rendering
function ConcurrentComponent() {
  const [urgent, setUrgent] = useState(false);
  const [background, setBackground] = useState(false);

  // Urgent updates (user interactions) take priority
  const handlePress = () => {
    setUrgent(true);
    // This renders immediately
  };

  // Background updates can be interrupted
  const handleBackgroundUpdate = () => {
    startTransition(() => {
      setBackground(true);
      // This can be interrupted by urgent updates
    });
  };

  return (
    &lt;View&gt;
      &lt;TouchableOpacity onPress={handlePress}>
        &lt;Text&gt;Urgent Update&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
      &lt;TouchableOpacity onPress={handleBackgroundUpdate}>
        &lt;Text&gt;Background Update&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">ANIMATION PERFORMANCE</span>
  <h3 style="margin-top: 0;">Q13: Optimizing animations for 60fps performance.</h3>

  <p>Animations can make or break your app's perceived performance:</p>

  <div class="code-example">Animation Performance Best Practices</div>
  <pre class="code-block"><code class="language-javascript">// 1. Use native driver for off-thread animations
Animated.timing(opacity, {
  toValue: 0,
  duration: 300,
  useNativeDriver: true, // Runs on UI thread
}).start();

// 2. Reanimated for complex animations
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
} from 'react-native-reanimated';

function SmoothAnimation() {
  const offset = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: offset.value }],
  }));

  const handlePress = () => {
    offset.value = withSpring(Math.random() * 200 - 100, {
      damping: 10,
      stiffness: 100,
    });
  };

  return (
    &lt;Animated.View style={animatedStyle}>
      &lt;TouchableOpacity onPress={handlePress}>
        &lt;Text&gt;Animate&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/Animated.View&gt;
  );
}

// 3. Avoid layout animations during scrolling
function ScrollOptimizedList() {
  return (
    &lt;FlatList
      data={data}
      renderItem={({ item }) => (
        &lt;Animated.View
          entering={FadeIn} // Only animate when items appear
          exiting={FadeOut} // Only animate when items disappear
        >
          &lt;Text&gt;{item.title}&lt;/Text&gt;
        &lt;/Animated.View&gt;
      )}
      // Disable layout animations during scroll
      disableIntervalMomentum={true}
    /&gt;
  );
}

// 4. Use transform instead of changing dimensions
// ❌ Bad: Changes layout, causes reflow
Animated.timing(height, {
  toValue: 200,
}).start();

// ✅ Good: Uses transform, no layout change
Animated.timing(scale, {
  toValue: 1.2,
  useNativeDriver: true,
}).start();

// 5. Batch animations together
function BatchedAnimations() {
  const animateAll = () => {
    Animated.parallel([
      Animated.timing(opacity, { toValue: 0, useNativeDriver: true }),
      Animated.spring(scale, { toValue: 1.2, useNativeDriver: true }),
      Animated.timing(translateX, { toValue: 100, useNativeDriver: true }),
    ]).start();
  };

  return &lt;TouchableOpacity onPress={animateAll} /&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">MEMORY PROFILING</span>
  <h3 style="margin-top: 0;">Q14: Memory profiling and leak detection.</h3>

  <p>Memory issues can cause crashes and poor performance:</p>

  <div class="code-example">Memory Profiling Techniques</div>
  <pre class="code-block"><code class="language-javascript">// 1. React DevTools Memory Tab
// - Take heap snapshots
// - Compare snapshots to find leaks
// - Use "Record allocation timeline"

// 2. Chrome DevTools Memory Tab
// - Force garbage collection
// - Take heap snapshots
// - Look for detached DOM nodes (memory leaks)

// 3. Custom memory monitoring
function useMemoryMonitor() {
  const [memoryInfo, setMemoryInfo] = useState(null);

  useEffect(() => {
    const interval = setInterval(async () => {
      if (__DEV__) {
        // Get memory info (limited in RN)
        const memory = {
          used: performance.memory?.usedJSHeapSize || 0,
          total: performance.memory?.totalJSHeapSize || 0,
          limit: performance.memory?.jsHeapSizeLimit || 0,
        };

        setMemoryInfo(memory);

        // Warn if memory usage is high
        if (memory.used > memory.limit * 0.8) {
          console.warn('High memory usage detected');
        }
      }
    }, 5000);

    return () => clearInterval(interval);
  }, []);

  return memoryInfo;
}

// 4. Detecting memory leaks in components
function useMemoryLeakDetector(componentName) {
  const mountedRef = useRef(true);
  const effectsCount = useRef(0);

  useEffect(() => {
    effectsCount.current += 1;

    return () => {
      mountedRef.current = false;
    };
  });

  // Check if component is being recreated too often
  useEffect(() => {
    if (effectsCount.current > 10) {
      console.warn(`${componentName} is being recreated frequently - possible memory leak`);
    }
  }, [effectsCount.current]);

  return mountedRef.current;
}

// 5. Memory-efficient data structures
// Use Maps/Sets for frequent lookups instead of arrays
const userMap = new Map();
const userSet = new Set();

// Instead of:
const users = [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }];
const user = users.find(u => u.id === 1); // O(n)

// Use:
userMap.set(1, { id: 1, name: 'John' });
userMap.set(2, { id: 2, name: 'Jane' });
const user = userMap.get(1); // O(1)

// 6. Object pooling for frequently created objects
class ObjectPool {
  constructor(createFn, size = 10) {
    this.pool = [];
    this.createFn = createFn;
    this.maxSize = size;
  }

  acquire() {
    return this.pool.pop() || this.createFn();
  }

  release(obj) {
    if (this.pool.length < this.maxSize) {
      // Reset object to initial state
      this.pool.push(obj);
    }
  }
}

// Usage for animation objects
const vectorPool = new ObjectPool(() => ({ x: 0, y: 0 }), 100);</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE MONITORING</span>
  <h3 style="margin-top: 0;">Q15: Setting up performance monitoring in production.</h3>

  <p>Monitor your app's performance in the real world to catch issues:</p>

  <div class="code-example">Production Performance Monitoring</div>
  <pre class="code-block"><code class="language-javascript">// 1. Custom performance tracking
function usePerformanceTracker(eventName) {
  const startTime = useRef(performance.now());

  const track = useCallback((additionalData = {}) => {
    const duration = performance.now() - startTime.current;

    analytics.track('PerformanceEvent', {
      eventName,
      duration,
      timestamp: Date.now(),
      platform: Platform.OS,
      ...additionalData
    });
  }, [eventName]);

  return track;
}

// Usage
function LoginScreen() {
  const trackLogin = usePerformanceTracker('user_login');

  const handleLogin = async () => {
    const startTime = performance.now();

    try {
      await authService.login(email, password);
      const duration = performance.now() - startTime;

      trackLogin({
        success: true,
        duration,
        method: 'email'
      });

      navigation.replace('Main');
    } catch (error) {
      trackLogin({
        success: false,
        error: error.message,
        duration: performance.now() - startTime
      });
    }
  };

  return &lt;LoginForm onSubmit={handleLogin} /&gt;;
}

// 2. FPS monitoring
function useFPSMonitor() {
  const [fps, setFps] = useState(60);
  const frameCount = useRef(0);
  const lastTime = useRef(performance.now());

  useFrameCallback(() => {
    frameCount.current += 1;

    const now = performance.now();
    if (now - lastTime.current >= 1000) {
      const currentFps = (frameCount.current * 1000) / (now - lastTime.current);
      setFps(Math.round(currentFps));

      // Alert if FPS drops too low
      if (currentFps < 30) {
        analytics.track('LowFPS', {
          fps: currentFps,
          timestamp: now
        });
      }

      frameCount.current = 0;
      lastTime.current = now;
    }
  });

  return fps;
}

// 3. Component render performance
function useRenderPerformance(componentName) {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(0);

  renderCount.current += 1;

  useEffect(() => {
    const renderTime = performance.now() - lastRenderTime.current;

    if (lastRenderTime.current > 0 && renderTime > 16) { // More than one frame
      console.warn(`${componentName} slow render: ${renderTime.toFixed(2)}ms`);
    }

    lastRenderTime.current = renderTime;
  });

  return renderCount.current;
}

// 4. Network performance monitoring
function useNetworkMonitor() {
  const [metrics, setMetrics] = useState({
    averageResponseTime: 0,
    successRate: 100,
    errorCount: 0,
  });

  const requests = useRef([]);

  const trackRequest = useCallback((url, startTime, success, responseTime) => {
    requests.current.push({
      url,
      startTime,
      success,
      responseTime,
    });

    // Keep only last 50 requests
    if (requests.current.length > 50) {
      requests.current = requests.current.slice(-50);
    }

    // Calculate metrics
    const recentRequests = requests.current.slice(-10);
    const avgResponseTime = recentRequests.reduce((sum, req) =>
      sum + req.responseTime, 0) / recentRequests.length;

    const successCount = recentRequests.filter(req => req.success).length;
    const successRate = (successCount / recentRequests.length) * 100;

    setMetrics({
      averageResponseTime: Math.round(avgResponseTime),
      successRate: Math.round(successRate),
      errorCount: recentRequests.length - successCount,
    });
  }, []);

  return { metrics, trackRequest };
}

// 5. Error boundary with performance tracking
class PerformanceErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Track performance-related errors
    analytics.track('PerformanceError', {
      error: error.message,
      componentStack: errorInfo.componentStack,
      timestamp: Date.now(),
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        &lt;View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          &lt;Text&gt;Something went wrong. Our team has been notified.&lt;/Text&gt;
        &lt;/View&gt;
      );
    }

    return this.props.children;
  }
}</code></pre>
</div>

  <div class="code-example">Performance Basics</div>
  <pre class="code-block"><code class="language-javascript">// ❌ BAD: Heavy computation in render
function SlowComponent({ data }) {
  const processedData = expensiveCalculation(data); // Runs on every render!

  return &lt;Text&gt;{processedData}&lt;/Text&gt;;
}

// ✅ GOOD: Memoize expensive operations
function FastComponent({ data }) {
  const processedData = useMemo(() => expensiveCalculation(data), [data]);

  return &lt;Text&gt;{processedData}&lt;/Text&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">LIST OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q23: Optimizing FlatList for large datasets.</h3>

  <p>FlatList is powerful but needs proper configuration for performance:</p>

  <div class="code-example">FlatList Performance Optimization</div>
  <pre class="code-block"><code class="language-javascript">function OptimizedList({ data }) {
  const renderItem = useCallback(({ item, index }) => (
    &lt;ListItem item={item} index={index} onPress={handlePress} />
  ), [handlePress]);

  const keyExtractor = useCallback((item, index) => {
    return item.id?.toString() || index.toString();
  }, []);

  return (
    &lt;FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={(data, index) => ({
        length: ITEM_HEIGHT,
        offset: ITEM_HEIGHT * index,
        index,
      })}
      initialNumToRender={10}
      maxToRenderPerBatch={10}
      windowSize={10}
      removeClippedSubviews={true}
    /&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">MEMORY LEAKS</span>
  <h3 style="margin-top: 0;">Q24: Common memory leaks and how to prevent them.</h3>

  <p>Memory leaks in React Native can crash your app. Here are the most common causes:</p>

  <div class="code-example">Preventing Memory Leaks</div>
  <pre class="code-block"><code class="language-javascript">// ❌ LEAK: Timer not cleared
function BadTimerComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    // No cleanup!
  }, []);

  return &lt;Text&gt;Count: {count}&lt;/Text&gt;;
}

// ✅ FIXED: Proper cleanup
function GoodTimerComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  return &lt;Text&gt;Count: {count}&lt;/Text&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PROFILING TOOLS</span>
  <h3 style="margin-top: 0;">Q25: Essential profiling tools for React Native.</h3>

  <p>Use these tools to identify and fix performance issues:</p>

  <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🔧 Essential Profiling Tools:</strong>
    <ul style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Flipper:</strong> Facebook's debugging platform</li>
      <li><strong>React DevTools:</strong> Component tree inspection</li>
      <li><strong>Chrome DevTools:</strong> Network and performance profiling</li>
      <li><strong>Systrace:</strong> Android system-level tracing</li>
    </ul>
  </div>
</div>

---

<div class="nav-footer">
  <a href="phase3-navigation-architecture.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 03: Navigation</a>
  <a href="phase5-native-modules-apis.md" style="color: #3b82f6; text-decoration: none; font-weight: 600;">Phase 05: Native Modules ➡️</a>
</div>

</div>
