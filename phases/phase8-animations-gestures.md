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
    background: linear-gradient(135deg, #ef4444 0%, #b91c1c 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(239, 68, 68, 0.2);
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
    background: #fef2f2;
    color: #b91c1c;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #fff1f2;
    border-left: 4px solid #ef4444;
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
  <span class="phase-number">Phase 08</span>
  <h1 class="phase-title">Animations & Gesture Handling</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Reanimated, RNGH, Physics, and Interactive Experiences</p>
</div>

---

### 📋 Phase Overview
Master animations and gestures to create native-feeling interactions. Learn Reanimated for 60fps animations, gesture recognition, physics-based motion, and accessibility considerations for smooth, responsive UIs.

---

<div class="qa-card">
<span class="question-tag">ARCHITECTURE</span>
<h3>8.1 Reanimated vs. Animated API</h3>

**Question:** *"Why is the legacy Animated API considered insufficient for modern, complex React Native gestures?"*

<p>The legacy <code>Animated</code> API is bridge-dependent. Every frame update or gesture event must be serialized as JSON, sent across the bridge to the JS thread, processed, and sent back to the native thread. If the JS thread is busy (e.g., processing a large API response or rendering a complex list), the animation will drop frames, leading to a "janky" experience.</p>

<p><strong>Reanimated 3 advantages:</strong></p>
<ul>
<li><strong>Worklets:</strong> JavaScript functions marked with <code>'worklet';</code> are compiled into bytecode and run on a secondary JS VM on the UI thread.</li>
<li><strong>Synchronous Execution:</strong> Logic runs directly on the UI thread, allowing for zero-latency response to touch events.</li>
<li><strong>Shared Values:</strong> Thread-safe pointers to memory that both threads can see, but only the UI thread updates at 60/120fps.</li>
</ul>

<div class="senior-insight">
<strong>💡 Senior Insight: The "One-Way Street"</strong><br/>
The legacy API's <code>useNativeDriver: true</code> only works for non-layout properties (opacity, transform). Reanimated allows you to animate *anything* (width, flex, colors) on the UI thread because it operates below the React reconciliation layer.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "If Reanimated is so good, why not use it for everything?"<br/>
<strong>Answer:</strong> Complexity and Bundle Size. For simple one-off fades, the built-in API is lighter. Reanimated also introduces a slight overhead in initialization and requires C++ TurboModule support.
</div>
</div>

<div class="qa-card">
<span class="question-tag">CORE CONCEPTS</span>
<h3>8.2 Shared Values & useAnimatedStyle</h3>

**Question:** *"Explain how useSharedValue and useAnimatedStyle work together to create an animation."*

<p><code>useSharedValue</code> is the "source of truth" that lives on the UI thread. Unlike React state, updating a shared value does not trigger a re-render of the component. Instead, it triggers a "style update" on the UI thread.</p>

<div class="code-block-header">Reanimated 3 Basic Implementation</div>

```typescript
const offset = useSharedValue(0);

const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: offset.value }],
}));

// Triggered from JS or UI thread
const startAnim = () => {
  offset.value = withSpring(100);
};
```

<div class="senior-insight">
<strong>💡 Senior Insight: Worklet Capturing</strong><br/>
Variables used inside <code>useAnimatedStyle</code> are "captured" into the worklet. If you use a standard React state variable inside it, the worklet will only ever see the value it had when the worklet was first created, unless you include it in the dependency array.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "What happens if you try to read <code>offset.value</code> in a standard <code>useEffect</code>?"<br/>
<strong>Answer:</strong> You will get the value, but it might be slightly out of sync with what's on the screen because the JS thread reads it asynchronously from the UI thread.
</div>
</div>

<div class="qa-card">
<span class="question-tag">GESTURES</span>
<h3>8.3 RNGH: PanResponder vs. GestureDetector</h3>

**Question:** *"Why should you use react-native-gesture-handler (RNGH) instead of the built-in PanResponder?"*

<p><strong>PanResponder</strong> is a JS-level implementation. It intercepts touch events at the top of the view hierarchy and sends them across the bridge. <strong>RNGH</strong> hooks into the native platform's gesture recognition system (iOS <code>UIGestureRecognizer</code>, Android <code>GestureDetector</code>).</p>

<p><strong>Key Differences:</strong></p>
<ul>
<li><strong>Deterministic:</strong> RNGH works even if the JS thread is 100% blocked.</li>
<li><strong>Composition:</strong> RNGH v2 <code>GestureDetector</code> allows for elegant composition like <code>Gesture.Exclusive(doubleTap, singleTap)</code>.</li>
<li><strong>Native Logic:</strong> It can decide whether to fail or activate based on native view scroll state (e.g., swiping in a ScrollView).</li>
</ul>

<div class="code-block-header">RNGH v2 GestureDetector</div>

```typescript
const pan = Gesture.Pan()
  .onUpdate((event) => {
    translationX.value = event.translationX;
  })
  .onEnd(() => {
    translationX.value = withSpring(0);
  });

return <GestureDetector gesture={pan}><Animated.View /></GestureDetector>;
```

<div class="senior-insight">
<strong>💡 Senior Insight: The "Race Condition"</strong><br/>
Standard React Native touch events (TouchableOpacity) often fight with ScrollViews. RNGH solves this by allowing you to define `simultaneousHandlers` or `waitFor`, giving you fine-grained control over which gesture "wins."
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "How do you handle gestures in a list where the user might want to scroll or swipe an item?"<br/>
<strong>Answer:</strong> Use <code>Gesture.Pan().activeOffsetX([-10, 10])</code>. This ensures the gesture only activates after a 10px horizontal movement, preventing it from intercepting vertical scrolls.
</div>
</div>

<div class="qa-card">
<span class="question-tag">PHYSICS</span>
<h3>8.4 Spring vs. Timing Animations</h3>

**Question:** *"When is it better to use withSpring instead of withTiming, and how do you tune them?"*

<p><strong>withTiming</strong> is duration-based. It follows a curve (easing) over a fixed time. <strong>withSpring</strong> is physics-based. It doesn't have a fixed duration; it calculates the time based on velocity and physical constants.</p>

<p><strong>Tuning Parameters:</strong></p>
<ul>
<li><strong>Damping:</strong> How quickly the spring stops (friction).</li>
<li><strong>Stiffness:</strong> How "tight" the spring is.</li>
<li><strong>Mass:</strong> The weight of the object (inertia).</li>
</ul>

<div class="senior-insight">
<strong>💡 Senior Insight: Response to Velocity</strong><br/>
Springs are essential for gestures. If a user swipes fast and releases, <code>withSpring</code> can take that initial <strong>velocity</strong> into account to continue the movement naturally. <code>withTiming</code> would just start a pre-defined animation from the current point, feeling "robotic."
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "How do you make a spring animation stop immediately without bouncing?"<br/>
<strong>Answer:</strong> Set <code>overshootClamping: true</code> or increase the <code>damping</code> to the point of critical damping.
</div>
</div>

<div class="qa-card">
<span class="question-tag">LAYOUT</span>
<h3>8.5 Reanimated Layout Animations</h3>

**Question:** *"How do you implement entry, exit, and layout transitions without manual state management?"*

<p>Reanimated 3 handles "Layout Transitions" by hooking into the native layout engine. When a component's position changes (e.g., an item is removed from a list), Reanimated intercepts the layout change and animates it.</p>

<div class="code-block-header">Layout Animation Example</div>

```typescript
<Animated.View 
  entering={FadeIn.duration(500)} 
  exiting={SlideOutLeft}
  layout={Layout.springify()} 
/>
```

<div class="senior-insight">
<strong>💡 Senior Insight: Custom Layout Transitions</strong><br/>
You can create custom layout animations using <code>LayoutAnimationConfig</code>. This is powerful for "Magic Move" effects where an element moves across the screen when its parent's layout changes.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "Why do Layout Animations sometimes flicker on Android?"<br/>
<strong>Answer:</strong> Android's <code>overflow: 'hidden'</code> and elevation can interfere with how the native view is clipped during a layout transition. Setting <code>extraNodeProps</code> or using a wrapper View usually fixes it.
</div>
</div>

<div class="qa-card">
<span class="question-tag">PERFORMANCE</span>
<h3>8.6 The "InteractionManager" Pattern</h3>

**Question:** *"How do you ensure heavy JS work doesn't interrupt a smooth navigation animation?"*

<p>Use <code>InteractionManager.runAfterInteractions(() => { ... })</code>. This is a built-in RN utility that keeps track of active animations. It defers the execution of the callback until all current animations have finished.</p>

<div class="senior-insight">
<strong>💡 Senior Insight: The Double-Edged Sword</strong><br/>
While <code>runAfterInteractions</code> prevents jank, it can make the app feel slow if your animations take too long. A senior dev might prioritize "Optimistic UI" updates or use <code>DeviceEventEmitter</code> to trigger data loading slightly before the animation ends.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "What happens if an animation never finishes?"<br/>
<strong>Answer:</strong> The callback will be stuck forever. Always provide a timeout or use <code>Promise.race</code> to ensure your app remains functional.
</div>
</div>

<div class="qa-card">
<span class="question-tag">INTERPOLATION</span>
<h3>8.7 Extrapolating Shared Values</h3>

**Question:** *"What is interpolation, and how do you use extrapolation to prevent layout breaking?"*

<p>Interpolation maps an input value (like scroll position) to an output value (like opacity). <strong>Extrapolation</strong> defines what happens when the input goes outside the defined range.</p>

<div class="code-block-header">Interpolation with Clamp</div>

```typescript
const opacity = interpolate(
  scrollY.value,
  [0, 100], // Input range
  [1, 0],   // Output range
  Extrapolation.CLAMP // Prevents opacity < 0 or > 1
);
```

<div class="senior-insight">
<strong>💡 Senior Insight: Color Interpolation</strong><br/>
Reanimated's <code>interpolateColor</code> is highly optimized. It handles hex, RGB, and HSL conversions on the UI thread, allowing for smooth background color transitions that would be incredibly expensive if handled via React state.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "What is the difference between CLAMP and IDENTITY extrapolation?"<br/>
<strong>Answer:</strong> <code>CLAMP</code> will stop at the edge values (0 or 1), while <code>IDENTITY</code> will continue the linear mapping indefinitely (e.g., if input is 200, output becomes -1).
</div>
</div>

<div class="qa-card">
<span class="question-tag">SHARED ELEMENTS</span>
<h3>8.8 Shared Element Transitions (SET)</h3>

**Question:** *"What is a Shared Element Transition, and what are its limitations in Reanimated 3?"*

<p>SET allows a component to appear as if it's moving from one screen to another. Reanimated 3 uses <code>sharedTransitionTag</code> to identify matching components across different navigation screens.</p>

<p><strong>Limitations:</strong></p>
<ul>
<li><strong>Fabric Support:</strong> Fabric (New Arch) support for SET is still maturing.</li>
<li><strong>Image Loading:</strong> If the destination image hasn't loaded yet, the transition might "pop" or flicker.</li>
<li><strong>Nested Views:</strong> Complex nested structures can confuse the layout measurement engine.</li>
</ul>

<div class="senior-insight">
<strong>💡 Senior Insight: The "Placeholder" Trick</strong><br/>
To ensure a smooth SET, always render a low-resolution placeholder or a colored box with the same dimensions on the destination screen to prevent layout shifts during the transition.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "Can you use Shared Element Transitions with React Navigation's native stack?"<br/>
<strong>Answer:</strong> Yes, but it requires specific configuration. Reanimated's SET works by intercepting the screen transition, which can sometimes conflict with native OS transitions (like iOS swipe-back).
</div>
</div>

<div class="qa-card">
<span class="question-tag">SENSORS</span>
<h3>8.9 Using Device Sensors with Animations</h3>

**Question:** *"How do you use useAnimatedSensor to create a 'parallax' effect?"*

<p><code>useAnimatedSensor</code> provides access to the Gyroscope, Accelerometer, or Magnetometer directly on the UI thread. This is critical for performance because sensor data can fire at 100Hz+.</p>

<div class="code-block-header">Parallax with Gyroscope</div>

```typescript
const sensor = useAnimatedSensor(SensorType.GYROSCOPE);

const style = useAnimatedStyle(() => {
  const { x, y } = sensor.sensor.value;
  return {
    transform: [
      { translateX: x * 10 },
      { translateY: y * 10 },
    ],
  };
});
```

<div class="senior-insight">
<strong>💡 Senior Insight: Sensor Fusion</strong><br/>
For professional parallax, don't rely on raw gyroscope data alone as it drifts. Senior devs use <code>SensorType.ROTATION</code> (which uses sensor fusion on the native side) to get a stable, absolute orientation of the device.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "Does <code>useAnimatedSensor</code> drain the battery?"<br/>
<strong>Answer:</strong> Yes, if not managed. You should pass an <code>interval</code> option (e.g., 16ms for 60fps) and ensure the sensor is only active when the component is focused.
</div>
</div>

<div class="qa-card">
<span class="question-tag">GESTURE COMPOSITION</span>
<h3>8.10 Simultaneous Gesture Handling</h3>

**Question:** *"How do you allow a user to pinch-to-zoom and pan an image at the same time?"*

<p>Using RNGH v2, you compose gestures using <code>Gesture.Simultaneous()</code>. Without this, one gesture would "cancel" the other as soon as it's recognized.</p>

<div class="code-block-header">Simultaneous Pan & Pinch</div>

```typescript
const pan = Gesture.Pan();
const pinch = Gesture.Pinch();

const composed = Gesture.Simultaneous(pan, pinch);

return <GestureDetector gesture={composed}>...</GestureDetector>;
```

<div class="senior-insight">
<strong>💡 Senior Insight: State Management</strong><br/>
When combining gestures, you often need to store the "last known" scale and translation in shared values to prevent the image from "jumping" back to the start when the user begins a second interaction.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "How do you prevent a pinch gesture from triggering a swipe-back navigation on iOS?"<br/>
<strong>Answer:</strong> Use <code>Gesture.Pan().activeOffsetX([-10, 10])</code> or wrap the component in a <code>NativeViewGestureHandler</code> to prioritize the internal gestures over the navigation controller.
</div>
</div>

<div class="qa-card">
<span class="question-tag">C++ WORKLETS</span>
<h3>8.11 The "runOnJS" Function</h3>

**Question:** *"Why can't you call a normal JS function directly from a worklet, and how do you use runOnJS?"*

<p>Worklets run in a separate JavaScript runtime (the UI thread's VM). They don't have access to the JS thread's closure, variables, or functions. <code>runOnJS</code> acts as a bridge to send a message back to the JS thread.</p>

<div class="senior-insight">
<strong>💡 Senior Insight: Thread Hopping</strong><br/>
Overusing <code>runOnJS</code> can defeat the purpose of Reanimated. If you call <code>runOnJS</code> every frame (e.g., to update React state during an animation), you're back to bridge-limited performance. Only use it for "terminal" events like <code>onEnd</code> or <code>onFinalize</code>.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "Can you pass complex objects (like a full React component) to <code>runOnJS</code>?"<br/>
<strong>Answer:</strong> No. Only serializable data or functions defined on the JS thread can be passed. Complex React objects cannot be 'teleported' across threads like that.
</div>
</div>

<div class="qa-card">
<span class="question-tag">LAYOUT MEASUREMENT</span>
<h3>8.12 measure() and getRelativeCoords()</h3>

**Question:** *"How do you get the position and size of a component on the UI thread?"*

<p>The <code>measure(animatedRef)</code> function is a synchronous call on the UI thread. It returns <code>x</code>, <code>y</code>, <code>width</code>, <code>height</code>, <code>pageX</code>, and <code>pageY</code>.</p>

<div class="code-block-header">Measuring UI Elements</div>

```typescript
const aRef = useAnimatedRef<View>();

const onTouch = Gesture.Tap().onEnd(() => {
  const layout = measure(aRef);
  console.log('Width is:', layout.width);
});
```

<div class="senior-insight">
<strong>💡 Senior Insight: Coordinate Conversion</strong><br/>
Senior devs use <code>getRelativeCoords</code> when they need to know where a touch event happened *relative* to a specific component. This is vital for "Touch to Ripple" effects or custom sliders where the local X/Y is more important than the screen-wide PageX/Y.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "Why does <code>measure()</code> sometimes return null?"<br/>
<strong>Answer:</strong> It returns null if the component hasn't been mounted yet or if the native view hasn't performed its first layout pass.
</div>
</div>

<div class="qa-card">
<span class="question-tag">SVG ANIMATIONS</span>
<h3>8.13 Path Interpolation & Skia</h3>

**Question:** *"How do you animate an SVG path (e.g., morphing shapes)?"*

<p>While <code>react-native-svg</code> works, <strong>React Native Skia</strong> is the modern standard for complex path animations. Skia runs entirely on the UI thread and uses the same rendering engine as Flutter.</p>

<div class="senior-insight">
<strong>💡 Senior Insight: Skia vs. SVG</strong><br/>
For simple icons, use SVG. For dynamic charts, morphing shapes, or complex shaders (like glassmorphism), use Skia. Skia's <code>Path</code> object allows for synchronous <code>interpolatePaths</code>, which is much smoother than JS-based string manipulation.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "Can you animate a path between two strings that have a different number of points?"<br/>
<strong>Answer:</strong> Standard interpolation requires the same number of points. To morph between different shapes, you need to use a library like <code>flubber</code> (on the JS thread) or pre-process the paths to have matching segments.
</div>
</div>

<div class="qa-card">
<span class="question-tag">SCROLL EVENTS</span>
<h3>8.14 useAnimatedScrollHandler</h3>

**Question:** *"How do you create a 'Sticky Header' effect that responds to scroll position?"*

<p>You bind a shared value to the <code>onScroll</code> event. This value can then drive the <code>translateY</code> of a header component.</p>

<div class="code-block-header">Scroll-Driven Header</div>

```typescript
const scrollY = useSharedValue(0);

const scrollHandler = useAnimatedScrollHandler((event) => {
  scrollY.value = event.contentOffset.y;
});

const headerStyle = useAnimatedStyle(() => ({
  transform: [{ translateY: interpolate(scrollY.value, [0, 100], [0, -50], Extrapolation.CLAMP) }],
}));
```

<div class="senior-insight">
<strong>💡 Senior Insight: Fling Velocity</strong><br/>
A senior implementation doesn't just look at the scroll position; it looks at <code>velocity.y</code>. If the user "flings" the list, you can use that velocity to hide the header faster or trigger a "back-to-top" button with a spring-loaded entrance.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "How do you prevent the scroll event from clogging the bridge?"<br/>
<strong>Answer:</strong> By using <code>useAnimatedScrollHandler</code>, the event is handled entirely on the UI thread. No data is sent to the JS thread unless you explicitly call <code>runOnJS</code>.
</div>
</div>

<div class="qa-card">
<span class="question-tag">DEBUGGING</span>
<h3>8.15 Debugging Reanimated</h3>

**Question:** *"How do you debug values that live on the UI thread?"*

<p>Standard <code>console.log</code> works but can be misleading. A better way is <code>useAnimatedReaction</code>.</p>

<div class="code-block-header">Animated Reaction Debugging</div>

```typescript
useAnimatedReaction(
  () => offset.value,
  (current, previous) => {
    if (current !== previous) {
      console.log("Value changed to:", current);
    }
  }
);
```

<div class="senior-insight">
<strong>💡 Senior Insight: Flipper & Layout Inspector</strong><br/>
Use the Flipper "Layout" plugin to see if views are actually moving on the native side. Sometimes an animation "runs" but the view is clipped or hidden by a parent, making it look like it's failing.
</div>

<div class="follow-up-trap">
<strong>⚠️ Follow-up Trap:</strong> "Can you use breakpoints inside a worklet?"<br/>
<strong>Answer:</strong> Generally no, as they run in a separate C++-based JS VM (Hermes/JSC) on the UI thread. Logging and the Layout Inspector are your primary tools.
</div>
</div>

<div class="qa-card">
  <span class="question-tag">SEQUENCING</span>
  <h3>8.16 withSequence and withDelay</h3>
  
  **Question:** *"How do you create a 'staggered' animation effect for a list of items?"*
  
  <p>Staggering is achieved by applying a <code>withDelay</code> based on the item's index. <code>withSequence</code> is used for multi-stage animations on a single item (e.g., pop up, then shake, then fade).</p>

  <div class="code-block-header">Staggered Entry</div>

  ```typescript
  const style = useAnimatedStyle(() => ({
    opacity: withDelay(index * 100, withTiming(1)),
    transform: [{ scale: withDelay(index * 100, withSpring(1)) }],
  }));
  ```

  <div class="follow-up-trap">
    <strong>⚠️ Follow-up Trap:</strong> "What is the risk of using too many delayed animations in a long list?"<br/>
    <strong>Answer:</strong> Memory usage. Each animation creates a worklet and a timer on the UI thread. For lists with 100+ items, use <code>FlashList</code>'s <code>onLoad</code> or similar to only animate visible items.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">HAPTICS</span>
  <h3>8.17 Triggering Haptics with Gestures</h3>
  
  **Question:** *"When and how should you trigger haptic feedback during an animation?"*
  
  <p>Haptics should be used to confirm "Success," "Warning," or "Impact." In Reanimated, you must trigger them via <code>runOnJS</code>.</p>

  <div class="senior-insight">
    <strong>💡 Senior Insight: Haptic Overkill</strong><br/>
    Never trigger haptics on every frame of a gesture (e.g., during <code>onUpdate</code>). Only trigger them when a state changes (e.g., <code>onActive</code>) or when a threshold is crossed (e.g., a "swipe-to-delete" distance is met).
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">CANVAS</span>
  <h3>8.18 Skia Canvas vs. Native Views</h3>
  
  **Question:** *"Why is Skia often faster than using 100 Animated.Views?"*
  
  <p>Every <code>Animated.View</code> is a real native view (iOS <code>UIView</code>, Android <code>View</code>). Each has its own layout, layer, and memory overhead. <strong>Skia</strong> uses a single native view (a Canvas) and draws everything inside it manually using GPU-accelerated commands. It's essentially like a game engine for your UI.</p>

  <div class="follow-up-trap">
    <strong>⚠️ Follow-up Trap:</strong> "Can you use standard React components (like <code><Text></code>) inside a Skia Canvas?"<br/>
    <strong>Answer:</strong> No. You must use Skia's own components (<code><Text /></code>, <code><Rect /></code>, <code><Image /></code>) which don't support standard CSS styling.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">GESTURE STATES</span>
  <h3>8.19 Handling "Canceled" Gestures</h3>
  
  **Question:** *"Why is it important to handle the 'onFinalize' or 'onCancel' state of a gesture?"*
  
  <p>Gestures can be interrupted by the system (e.g., an incoming call) or by other gestures (e.g., a parent ScrollView taking over). If you only handle <code>onEnd</code>, your UI might stay in its "active" state (e.g., scaled up or semi-transparent) forever.</p>

  <div class="senior-insight">
    <strong>💡 Senior Insight: The onFinalize pattern</strong><br/>
    RNGH v2's <code>onFinalize</code> runs whether the gesture succeeded or was canceled. It's the best place to put "cleanup" logic (like resetting a shared value to its original state).
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ACCESSIBILITY</span>
  <h3>8.20 Reduced Motion Support</h3>
  
  **Question:** *"How do you respect a user's 'Reduce Motion' system setting?"*
  
  <p>Forcing animations on users with vestibular disorders can cause physical discomfort. Use <code>AccessibilityInfo.isReduceMotionEnabled()</code> to check the setting.</p>

  <div class="code-block-header">Accessibility-Aware Animation</div>

  ```typescript
  const config = isReduceMotion ? { duration: 0 } : { damping: 10 };
  offset.value = withSpring(100, config);
  ```

  <div class="senior-insight">
    <strong>💡 Senior Insight: Platform Inconsistency</strong><br/>
    On iOS, "Reduce Motion" also affects system transitions. On Android, it's often ignored by third-party apps. A senior dev will implement a custom "App Settings" toggle that overrides the system setting if needed.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">MEMOIZATION</span>
  <h3>8.21 useAnimatedRef vs. useRef</h3>
  
  **Question:** *"When must you use useAnimatedRef instead of a standard React useRef?"*
  
  <p><code>useAnimatedRef</code> is a specialized hook that returns a ref object that can be shared between the JS and UI threads. It's required for functions like <code>measure()</code>, <code>scrollTo()</code>, and <code>dispatchCommand()</code> which need to interact with the native view directly from a worklet.</p>

  <div class="follow-up-trap">
    <strong>⚠️ Follow-up Trap:</strong> "Can you use <code>useAnimatedRef</code> to change a View's background color directly?"<br/>
    <strong>Answer:</strong> No. <code>useAnimatedRef</code> is for measurement and commands. To change styles, you should use <code>useAnimatedStyle</code>.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ANIMATION BASICS</span>
  <h3 style="margin-top: 0;">Q22: Basic animations with React Native's Animated API.</h3>

  <p>The Animated API is React Native's built-in animation system:</p>

  <div class="code-example">Basic Animated API</div>
  <pre class="code-block"><code class="language-javascript">import { Animated, TouchableOpacity } from 'react-native';

function FadeInView({ children }) {
  const fadeAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 1000,
      useNativeDriver: true, // Hardware acceleration
    }).start();
  }, [fadeAnim]);

  return (
    &lt;Animated.View style={{ opacity: fadeAnim }}>
      {children}
    &lt;/Animated.View&gt;
  );
}

// Sequence of animations
function SequenceAnimation() {
  const scaleAnim = useRef(new Animated.Value(1)).current;
  const rotateAnim = useRef(new Animated.Value(0)).current;

  const animate = () => {
    Animated.sequence([
      Animated.spring(scaleAnim, {
        toValue: 1.5,
        useNativeDriver: true,
      }),
      Animated.timing(rotateAnim, {
        toValue: 1,
        duration: 500,
        useNativeDriver: true,
      }),
    ]).start(() => {
      // Reset animations
      scaleAnim.setValue(1);
      rotateAnim.setValue(0);
    });
  };

  const rotate = rotateAnim.interpolate({
    inputRange: [0, 1],
    outputRange: ['0deg', '360deg'],
  });

  return (
    &lt;TouchableOpacity onPress={animate}>
      &lt;Animated.View style={{
        transform: [
          { scale: scaleAnim },
          { rotate },
        ],
      }}>
        &lt;Text>Tap to Animate&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/TouchableOpacity&gt;
  );
}

// Parallel animations
function ParallelAnimation() {
  const fadeAnim = useRef(new Animated.Value(0)).current;
  const slideAnim = useRef(new Animated.Value(-100)).current;

  useEffect(() => {
    Animated.parallel([
      Animated.timing(fadeAnim, {
        toValue: 1,
        duration: 1000,
        useNativeDriver: true,
      }),
      Animated.spring(slideAnim, {
        toValue: 0,
        useNativeDriver: true,
      }),
    ]).start();
  }, []);

  return (
    &lt;Animated.View style={{
      opacity: fadeAnim,
      transform: [{ translateX: slideAnim }],
    }}>
      &lt;Text>Fade and Slide In&lt;/Text&gt;
    &lt;/Animated.View&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always use <code>useNativeDriver: true</code> for better performance. This runs animations on the native thread instead of the JS thread, preventing frame drops during heavy JS work.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">GESTURE BASICS</span>
  <h3 style="margin-top: 0;">Q23: Basic gesture handling with PanResponder.</h3>

  <p>PanResponder is React Native's built-in gesture recognition system:</p>

  <div class="code-example">PanResponder Basics</div>
  <pre class="code-block"><code class="language-javascript">import { PanResponder, Animated } from 'react-native';

function DraggableBox() {
  const pan = useRef(new Animated.ValueXY()).current;

  const panResponder = useRef(
    PanResponder.create({
      onStartShouldSetPanResponder: () => true,
      onPanResponderGrant: () => {
        pan.setOffset({
          x: pan.x._value,
          y: pan.y._value,
        });
      },
      onPanResponderMove: Animated.event(
        [null, { dx: pan.x, dy: pan.y }],
        { useNativeDriver: false }
      ),
      onPanResponderRelease: () => {
        pan.flattenOffset();
        Animated.spring(pan, {
          toValue: { x: 0, y: 0 },
          useNativeDriver: false,
        }).start();
      },
    })
  ).current;

  return (
    &lt;Animated.View
      {...panResponder.panHandlers}
      style={{
        transform: [{ translateX: pan.x }, { translateY: pan.y }],
      }}
    >
      &lt;View style={styles.box}>
        &lt;Text>Drag me!&lt;/Text&gt;
      &lt;/View&gt;
    &lt;/Animated.View&gt;
  );
}

// Swipe gesture
function SwipeableCard() {
  const translateX = useRef(new Animated.Value(0)).current;

  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: (evt, gestureState) => {
        return Math.abs(gestureState.dx) > 20;
      },
      onPanResponderMove: (evt, gestureState) => {
        translateX.setValue(gestureState.dx);
      },
      onPanResponderRelease: (evt, gestureState) => {
        if (Math.abs(gestureState.dx) > 100) {
          // Swipe threshold met
          Animated.timing(translateX, {
            toValue: gestureState.dx > 0 ? 300 : -300,
            duration: 200,
            useNativeDriver: true,
          }).start(() => {
            // Handle swipe action (like delete)
            onSwipeComplete();
          });
        } else {
          // Return to original position
          Animated.spring(translateX, {
            toValue: 0,
            useNativeDriver: true,
          }).start();
        }
      },
    })
  ).current;

  return (
    &lt;Animated.View
      {...panResponder.panHandlers}
      style={{
        transform: [{ translateX }],
      }}
    >
      &lt;View style={styles.card}>
        &lt;Text>Swipe me left or right&lt;/Text&gt;
      &lt;/View&gt;
    &lt;/Animated.View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">REANIMATED BASICS</span>
  <h3 style="margin-top: 0;">Q24: Introduction to React Native Reanimated.</h3>

  <p>Reanimated is a powerful animation library that runs animations on the UI thread:</p>

  <div class="code-example">Reanimated Fundamentals</div>
  <pre class="code-block"><code class="language-javascript">import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
} from 'react-native-reanimated';

function ReanimatedBox() {
  const offset = useSharedValue(0);
  const scale = useSharedValue(1);

  const animatedStyles = useAnimatedStyle(() => ({
    transform: [
      { translateX: offset.value },
      { scale: scale.value },
    ],
  }));

  const handlePress = () => {
    offset.value = withSpring(Math.random() * 200 - 100);
    scale.value = withTiming(scale.value > 1 ? 1 : 1.2);
  };

  return (
    &lt;Animated.View style={[styles.box, animatedStyles]}>
      &lt;TouchableOpacity onPress={handlePress}>
        &lt;Text>Tap to animate&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/Animated.View&gt;
  );
}

// Gesture with Reanimated
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

function PanGestureBox() {
  const offset = useSharedValue({ x: 0, y: 0 });
  const start = useSharedValue({ x: 0, y: 0 });

  const animatedStyles = useAnimatedStyle(() => ({
    transform: [
      { translateX: offset.value.x },
      { translateY: offset.value.y },
    ],
  }));

  const gesture = Gesture.Pan()
    .onStart(() => {
      start.value = { x: offset.value.x, y: offset.value.y };
    })
    .onUpdate((e) => {
      offset.value = {
        x: start.value.x + e.translationX,
        y: start.value.y + e.translationY,
      };
    })
    .onEnd(() => {
      offset.value = withSpring({ x: 0, y: 0 });
    });

  return (
    &lt;GestureDetector gesture={gesture}>
      &lt;Animated.View style={[styles.box, animatedStyles]}>
        &lt;Text>Drag me around&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/GestureDetector&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Reanimated animations continue running even when the JS thread is blocked, making them perfect for smooth interactions. Shared values can be safely accessed from both JS and UI threads.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">SPRING PHYSICS</span>
  <h3 style="margin-top: 0;">Q25: Creating natural-feeling animations with spring physics.</h3>

  <p>Spring animations feel more natural than linear timing animations:</p>

  <div class="code-example">Spring Animations</div>
  <pre class="code-block"><code class="language-javascript">// Reanimated springs
import {
  withSpring,
  withTiming,
  ReduceMotion,
} from 'react-native-reanimated';

function SpringButton() {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePress = () => {
    scale.value = withSpring(1.2, {
      damping: 10,      // How quickly the spring slows down
      stiffness: 100,   // How stiff the spring is
      mass: 1,         // Mass of the object
      overshootClamping: false, // Allow overshoot
      restDisplacementThreshold: 0.01, // When to stop
      restSpeedThreshold: 2,           // Speed threshold
    });
  };

  const handlePressOut = () => {
    scale.value = withSpring(1, {
      damping: 15,
      stiffness: 150,
    });
  };

  return (
    &lt;Pressable onPressIn={handlePress} onPressOut={handlePressOut}>
      &lt;Animated.View style={[styles.button, animatedStyle]}>
        &lt;Text>Spring Button&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/Pressable&gt;
  );
}

// Sequence of springs
function BounceAnimation() {
  const translateY = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: translateY.value }],
  }));

  const startBounce = () => {
    translateY.value = withSequence(
      withSpring(-50, { damping: 12 }),
      withSpring(0, { damping: 12 }),
      withSpring(-25, { damping: 8 }),
      withSpring(0, { damping: 8 }),
    );
  };

  return (
    &lt;TouchableOpacity onPress={startBounce}>
      &lt;Animated.View style={[styles.box, animatedStyle]}>
        &lt;Text>Bounce me!&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/TouchableOpacity&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">GESTURE RECOGNITION</span>
  <h3 style="margin-top: 0;">Q26: Advanced gesture recognition with RNGH.</h3>

  <p>React Native Gesture Handler provides reliable gesture recognition:</p>

  <div class="code-example">Advanced Gestures</div>
  <pre class="code-block"><code class="language-javascript">import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

function PinchToZoom() {
  const scale = useSharedValue(1);
  const savedScale = useSharedValue(1);

  const pinchGesture = Gesture.Pinch()
    .onUpdate((e) => {
      scale.value = savedScale.value * e.scale;
    })
    .onEnd(() => {
      savedScale.value = scale.value;
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    &lt;GestureDetector gesture={pinchGesture}>
      &lt;Animated.View style={[styles.image, animatedStyle]}>
        &lt;Text>Pinch to zoom&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/GestureDetector&gt;
  );
}

// Simultaneous gestures
function MultiGestureHandler() {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);
  const scale = useSharedValue(1);

  const panGesture = Gesture.Pan()
    .onUpdate((e) => {
      translateX.value = e.translationX;
      translateY.value = e.translationY;
    })
    .onEnd(() => {
      translateX.value = withSpring(0);
      translateY.value = withSpring(0);
    });

  const pinchGesture = Gesture.Pinch()
    .onUpdate((e) => {
      scale.value = e.scale;
    });

  const composedGesture = Gesture.Simultaneous(panGesture, pinchGesture);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { translateY: translateY.value },
      { scale: scale.value },
    ],
  }));

  return (
    &lt;GestureDetector gesture={composedGesture}>
      &lt;Animated.View style={[styles.box, animatedStyle]}>
        &lt;Text>Pan and pinch simultaneously&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/GestureDetector&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">SCROLL ANIMATIONS</span>
  <h3 style="margin-top: 0;">Q27: Scroll-based animations and parallax effects.</h3>

  <p>Scroll position can drive smooth animations:</p>

  <div class="code-example">Scroll Animations</div>
  <pre class="code-block"><code class="language-javascript">import { useAnimatedScrollHandler, useAnimatedRef } from 'react-native-reanimated';

function ParallaxScroll() {
  const scrollY = useSharedValue(0);
  const scrollHandler = useAnimatedScrollHandler({
    onScroll: (event) => {
      scrollY.value = event.contentOffset.y;
    },
  });

  const headerStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: scrollY.value * 0.5 }], // Parallax effect
    opacity: interpolate(scrollY.value, [0, 200], [1, 0], Extrapolate.CLAMP),
  }));

  const titleStyle = useAnimatedStyle(() => ({
    transform: [
      { scale: interpolate(scrollY.value, [0, 100], [1, 0.8], Extrapolate.CLAMP) },
      { translateY: scrollY.value * 0.2 },
    ],
  }));

  return (
    &lt;View style={styles.container}>
      &lt;Animated.ScrollView
        onScroll={scrollHandler}
        scrollEventThrottle={16}
      >
        &lt;Animated.View style={[styles.header, headerStyle]}>
          &lt;Animated.Text style={[styles.title, titleStyle]}>
            Parallax Header
          &lt;/Animated.Text&gt;
        &lt;/Animated.View&gt;

        &lt;View style={styles.content}>
          {Array.from({ length: 20 }, (_, i) => (
            &lt;Text key={i} style={styles.item}>Item {i + 1}&lt;/Text&gt;
          ))}
        &lt;/View&gt;
      &lt;/Animated.ScrollView&gt;
    &lt;/View&gt;
  );
}

// Sticky header with scroll
function StickyHeaderScroll() {
  const scrollY = useSharedValue(0);
  const headerHeight = 100;

  const scrollHandler = useAnimatedScrollHandler({
    onScroll: (event) => {
      scrollY.value = event.contentOffset.y;
    },
  });

  const headerStyle = useAnimatedStyle(() => ({
    transform: [{
      translateY: interpolate(
        scrollY.value,
        [0, headerHeight],
        [0, -headerHeight],
        Extrapolate.CLAMP
      ),
    }],
  }));

  return (
    &lt;View style={styles.container}>
      &lt;Animated.ScrollView
        onScroll={scrollHandler}
        scrollEventThrottle={16}
        stickyHeaderIndices={[0]}
      >
        &lt;Animated.View style={[styles.stickyHeader, headerStyle]}>
          &lt;Text style={styles.headerText}>Sticky Header&lt;/Text&gt;
        &lt;/Animated.View&gt;

        &lt;View style={styles.content}>
          &lt;Text>Scroll content here...&lt;/Text&gt;
        &lt;/View&gt;
      &lt;/Animated.ScrollView&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">ACCESSIBILITY</span>
  <h3 style="margin-top: 0;">Q28: Making animations accessible and respecting user preferences.</h3>

  <p>Animations should respect accessibility settings:</p>

  <div class="code-example">Accessible Animations</div>
  <pre class="code-block"><code class="language-javascript">import { AccessibilityInfo, useAccessibilityInfo } from 'react-native';

function AccessibleAnimation() {
  const { reduceMotionEnabled } = useAccessibilityInfo();
  const [animationEnabled, setAnimationEnabled] = useState(true);

  useEffect(() => {
    // Check if user prefers reduced motion
    AccessibilityInfo.isReduceMotionEnabled().then(setAnimationEnabled);
  }, []);

  const handlePress = () => {
    if (!animationEnabled) {
      // Skip animation, just update state
      setScale(1.2);
      return;
    }

    // Normal animation
    scale.value = withSpring(1.2);
  };

  return (
    &lt;TouchableOpacity onPress={handlePress}>
      &lt;Animated.View style={animatedStyle}>
        &lt;Text>Accessible Button&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/TouchableOpacity&gt;
  );
}

// Respecting prefers-reduced-motion in Reanimated
import { ReduceMotion } from 'react-native-reanimated';

function ReducedMotionAnimation() {
  const scale = useSharedValue(1);

  const handlePress = () => {
    scale.value = withSpring(1.2, {
      reduceMotion: ReduceMotion.System, // Respects system setting
    });
  };

  return (
    &lt;Pressable onPress={handlePress}>
      &lt;Animated.View style={animatedStyle}>
        &lt;Text>Reduced Motion Aware&lt;/Text&gt;
      &lt;/Animated.View&gt;
    &lt;/Pressable&gt;
  );
}

// Alternative: Provide static versions
function AdaptiveButton({ children, onPress }) {
  const { reduceMotionEnabled } = useAccessibilityInfo();

  if (reduceMotionEnabled) {
    // No animation version
    return (
      &lt;TouchableOpacity
        style={[styles.button, { transform: [{ scale: isPressed ? 1.1 : 1 }] }]}
        onPress={onPress}
        onPressIn={() => setIsPressed(true)}
        onPressOut={() => setIsPressed(false)}
      >
        {children}
      &lt;/TouchableOpacity&gt;
    );
  }

  // Animated version
  return (
    &lt;Pressable onPress={onPress}>
      &lt;Animated.View style={animatedStyle}>
        {children}
      &lt;/Animated.View&gt;
    &lt;/Pressable&gt;
  );
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Always check for <code>prefers-reduced-motion</code> settings. Some users experience motion sickness or have vestibular disorders that make animations problematic. Use <code>AccessibilityInfo.isReduceMotionEnabled()</code> or Reanimated's <code>ReduceMotion</code> enum.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE OPTIMIZATION</span>
  <h3 style="margin-top: 0;">Q29: Performance considerations for animations and gestures.</h3>

  <p>Animations can hurt performance if not implemented correctly:</p>

  <div class="code-example">Animation Performance Tips</div>
  <pre class="code-block"><code class="language-javascript">// 1. Use worklets for complex calculations
const heavyCalculation = (value) => {
  'worklet'; // Run on UI thread
  let result = value;
  for (let i = 0; i < 1000; i++) {
    result = Math.sin(result) * Math.cos(result);
  }
  return result;
};

function OptimizedAnimation() {
  const animatedValue = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{
      rotate: `${heavyCalculation(animatedValue.value)}deg`
    }],
  }));

  return &lt;Animated.View style={animatedStyle} /&gt;;
}

// 2. Avoid unnecessary re-renders
function BadAnimatedList({ items }) {
  return items.map((item, index) => (
    &lt;AnimatedItem key={item.id} item={item} index={index} />
  ));
}

function GoodAnimatedList({ items }) {
  const animatedItems = useMemo(() =>
    items.map((item, index) => ({
      ...item,
      animatedStyle: { opacity: 1, transform: [{ translateY: index * 10 }] }
    })), [items]
  );

  return animatedItems.map((item) => (
    &lt;AnimatedItem key={item.id} item={item} />
  ));
}

// 3. Use shared values efficiently
function EfficientSharedValues() {
  // Bad: Creating new objects every render
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: sharedValue.value }],
  }));

  // Good: Reuse objects
  const animatedStyle = useAnimatedStyle(() => {
    transform.value = [{ translateX: sharedValue.value }];
    return { transform: transform.value };
  });

  return &lt;Animated.View style={animatedStyle} /&gt;;
}

// 4. Batch updates
function BatchUpdates() {
  const handleMultipleAnimations = () => {
    'worklet';
    // Run all updates simultaneously
    scale.value = withSpring(1.2);
    opacity.value = withTiming(0.8);
    translateX.value = withSpring(50);
  };

  return &lt;TouchableOpacity onPress={handleMultipleAnimations} /&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">DEBUGGING ANIMATIONS</span>
  <h3 style="margin-top: 0;">Q30: Debugging animations and gesture issues.</h3>

  <p>Debugging animations requires special techniques:</p>

  <div class="code-example">Animation Debugging</div>
  <pre class="code-block"><code class="language-javascript">// 1. Log shared values (development only)
function DebuggableAnimation() {
  const scale = useSharedValue(1);

  // Add debugging in development
  if (__DEV__) {
    useAnimatedReaction(() => scale.value, (value) => {
      console.log('Scale changed to:', value);
    });
  }

  return &lt;Animated.View style={animatedStyle} /&gt;;
}

// 2. Visual debugging overlay
function AnimationDebugger() {
  const [debugMode, setDebugMode] = useState(__DEV__);

  if (!debugMode) return null;

  return (
    &lt;View style={styles.debugOverlay}>
      &lt;Text>Animation Debug Info&lt;/Text&gt;
      &lt;Text>Shared value: {sharedValue.value}&lt;/Text&gt;
      &lt;TouchableOpacity onPress={() => setDebugMode(false)}>
        &lt;Text>Hide Debug&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}

// 3. Test gestures in isolation
function GestureTester() {
  const gesture = Gesture.Pan()
    .onStart(() => console.log('Gesture started'))
    .onUpdate((e) => console.log('Gesture update:', e))
    .onEnd(() => console.log('Gesture ended'));

  return (
    &lt;GestureDetector gesture={gesture}>
      &lt;View style={styles.testArea}>
        &lt;Text>Test gestures here - check console&lt;/Text&gt;
      &lt;/View&gt;
    &lt;/GestureDetector&gt;
  );
}

// 4. Performance monitoring
function AnimationPerformanceMonitor() {
  const frameCount = useRef(0);
  const lastTime = useRef(Date.now());

  useFrameCallback(() => {
    frameCount.current += 1;

    const now = Date.now();
    if (now - lastTime.current >= 1000) {
      const fps = (frameCount.current * 1000) / (now - lastTime.current);
      console.log(`FPS: ${fps.toFixed(1)}`);
      frameCount.current = 0;
      lastTime.current = now;
    }
  });

  return null; // Invisible monitor
}

// 5. Common debugging commands
// In React Native Debugger:
// - Show Perf Monitor: Cmd+D -> Show Perf Monitor
// - Debug JS Remotely: Cmd+D -> Debug JS Remotely
// - Show Inspector: Cmd+I

// Flipper plugins for Reanimated:
// - Reanimated Debugger
// - Gesture Handler Debugger</code></pre>
</div>

---

<div class="nav-footer">
  <a href="phase7-styling-design.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 07: Styling</a>
  <a href="phase9-testing-qa.md" style="color: #64748b; text-decoration: none; font-weight: 600;">Phase 09: Testing ➡️</a>
</div>

</div>