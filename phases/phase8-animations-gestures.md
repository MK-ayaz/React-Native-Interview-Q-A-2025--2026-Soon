# Phase 08: Animations & Gesture Handling

> **Master animations and gestures to create native-feeling interactions. Learn Reanimated for 60fps animations, gesture recognition, physics-based motion, and accessibility considerations for smooth, responsive UIs.**

---

### Q1: [ARCHITECTURE] Reanimated vs. Animated API
*"Why is the legacy Animated API considered insufficient for modern, complex React Native gestures?"*

The legacy `Animated` API is bridge-dependent. Every frame update or gesture event must be serialized as JSON, sent across the bridge to the JS thread, processed, and sent back to the native thread. If the JS thread is busy (e.g., processing a large API response or rendering a complex list), the animation will drop frames, leading to a "janky" experience.

**Reanimated 3 advantages:**
- **Worklets:** JavaScript functions marked with `'worklet';` are compiled into bytecode and run on a secondary JS VM on the UI thread.
- **Synchronous Execution:** Logic runs directly on the UI thread, allowing for zero-latency response to touch events.
- **Shared Values:** Thread-safe pointers to memory that both threads can see, but only the UI thread updates at 60/120fps.

> [!TIP]
> **Senior Insight: The "One-Way Street"**
> The legacy API's `useNativeDriver: true` only works for non-layout properties (opacity, transform). Reanimated allows you to animate *anything* (width, flex, colors) on the UI thread because it operates below the React reconciliation layer.

> [!WARNING]
> **Follow-up Trap:** "If Reanimated is so good, why not use it for everything?"
> **Answer:** Complexity and Bundle Size. For simple one-off fades, the built-in API is lighter. Reanimated also introduces a slight overhead in initialization and requires C++ TurboModule support.

---

### Q2: [CORE CONCEPTS] Shared Values & useAnimatedStyle
*"Explain how useSharedValue and useAnimatedStyle work together to create an animation."*

`useSharedValue` is the "source of truth" that lives on the UI thread. Unlike React state, updating a shared value does not trigger a re-render of the component. Instead, it triggers a "style update" on the UI thread.

```typescript
// Reanimated 3 Basic Implementation
const offset = useSharedValue(0);

const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: offset.value }],
}));

// Triggered from JS or UI thread
const startAnim = () => {
  offset.value = withSpring(100);
};
```

> [!TIP]
> **Senior Insight: Worklet Capturing**
> Variables used inside `useAnimatedStyle` are "captured" into the worklet. If you use a standard React state variable inside it, the worklet will only ever see the value it had when the worklet was first created, unless you include it in the dependency array.

> [!WARNING]
> **Follow-up Trap:** "What happens if you try to read `offset.value` in a standard `useEffect`?"
> **Answer:** You will get the value, but it might be slightly out of sync with what's on the screen because the JS thread reads it asynchronously from the UI thread.

---

### Q3: [GESTURES] RNGH: PanResponder vs. GestureDetector
*"Why should you use react-native-gesture-handler (RNGH) instead of the built-in PanResponder?"*

**PanResponder** is a JS-level implementation. It intercepts touch events at the top of the view hierarchy and sends them across the bridge. **RNGH** hooks into the native platform's gesture recognition system (iOS `UIGestureRecognizer`, Android `GestureDetector`).

**Key Differences:**
- **Deterministic:** RNGH works even if the JS thread is 100% blocked.
- **Composition:** RNGH v2 `GestureDetector` allows for elegant composition like `Gesture.Exclusive(doubleTap, singleTap)`.
- **Native Logic:** It can decide whether to fail or activate based on native view scroll state (e.g., swiping in a ScrollView).

```typescript
// RNGH v2 GestureDetector
const pan = Gesture.Pan()
  .onUpdate((event) => {
    translationX.value = event.translationX;
  })
  .onEnd(() => {
    translationX.value = withSpring(0);
  });

return <GestureDetector gesture={pan}><Animated.View /></GestureDetector>;
```

> [!TIP]
> **Senior Insight: The "Race Condition"**
> Standard React Native touch events (TouchableOpacity) often fight with ScrollViews. RNGH solves this by allowing you to define `simultaneousHandlers` or `waitFor`, giving you fine-grained control over which gesture "wins."

> [!WARNING]
> **Follow-up Trap:** "How do you handle gestures in a list where the user might want to scroll or swipe an item?"
> **Answer:** Use `Gesture.Pan().activeOffsetX([-10, 10])`. This ensures the gesture only activates after a 10px horizontal movement, preventing it from intercepting vertical scrolls.

---


### Q4: [PHYSICS] Spring vs. Timing Animations
*"When is it better to use withSpring instead of withTiming, and how do you tune them?"*

**withTiming** is duration-based. It follows a curve (easing) over a fixed time. **withSpring** is physics-based. It doesn't have a fixed duration; it calculates the time based on velocity and physical constants.

**Tuning Parameters:**
- **Damping:** How quickly the spring stops (friction).
- **Stiffness:** How "tight" the spring is.
- **Mass:** The weight of the object (inertia).

> [!TIP]
> **Senior Insight: Response to Velocity**
> Springs are essential for gestures. If a user swipes fast and releases, `withSpring` can take that initial **velocity** into account to continue the movement naturally. `withTiming` would just start a pre-defined animation from the current point, feeling "robotic."

> [!WARNING]
> **Follow-up Trap:** "How do you make a spring animation stop immediately without bouncing?"
> **Answer:** Set `overshootClamping: true` or increase the `damping` to the point of critical damping.

---

### Q5: [LAYOUT] Reanimated Layout Animations
*"How do you implement entry, exit, and layout transitions without manual state management?"*

Reanimated 3 handles "Layout Transitions" by hooking into the native layout engine. When a component's position changes (e.g., an item is removed from a list), Reanimated intercepts the layout change and animates it.

```typescript
// Layout Animation Example
<Animated.View 
  entering={FadeIn.duration(500)} 
  exiting={SlideOutLeft}
  layout={Layout.springify()} 
/>
```

> [!TIP]
> **Senior Insight: Custom Layout Transitions**
> You can create custom layout animations using `LayoutAnimationConfig`. This is powerful for "Magic Move" effects where an element moves across the screen when its parent's layout changes.

> [!WARNING]
> **Follow-up Trap:** "Why do Layout Animations sometimes flicker on Android?"
> **Answer:** Android's `overflow: 'hidden'` and elevation can interfere with how the native view is clipped during a layout transition. Setting `extraNodeProps` or using a wrapper View usually fixes it.

---

### Q6: [PERFORMANCE] The "InteractionManager" Pattern
*"How do you ensure heavy JS work doesn't interrupt a smooth navigation animation?"*

Use `InteractionManager.runAfterInteractions(() => { ... })`. This is a built-in RN utility that keeps track of active animations. It defers the execution of the callback until all current animations have finished.

> [!TIP]
> **Senior Insight: The Double-Edged Sword**
> While `runAfterInteractions` prevents jank, it can make the app feel slow if your animations take too long. A senior dev might prioritize "Optimistic UI" updates or use `DeviceEventEmitter` to trigger data loading slightly before the animation ends.

> [!WARNING]
> **Follow-up Trap:** "What happens if an animation never finishes?"
> **Answer:** The callback will be stuck forever. Always provide a timeout or use `Promise.race` to ensure your app remains functional.

---


### Q7: [INTERPOLATION] Extrapolating Shared Values
*"What is interpolation, and how do you use extrapolation to prevent layout breaking?"*

Interpolation maps an input value (like scroll position) to an output value (like opacity). **Extrapolation** defines what happens when the input goes outside the defined range.

```typescript
// Interpolation with Clamp
const opacity = interpolate(
  scrollY.value,
  [0, 100], // Input range
  [1, 0],   // Output range
  Extrapolation.CLAMP // Prevents opacity < 0 or > 1
);
```

> [!TIP]
> **Senior Insight: Color Interpolation**
> Reanimated's `interpolateColor` is highly optimized. It handles hex, RGB, and HSL conversions on the UI thread, allowing for smooth background color transitions that would be incredibly expensive if handled via React state.

> [!WARNING]
> **Follow-up Trap:** "What is the difference between CLAMP and IDENTITY extrapolation?"
> **Answer:** `CLAMP` will stop at the edge values (0 or 1), while `IDENTITY` will continue the linear mapping indefinitely (e.g., if input is 200, output becomes -1).

---

### Q8: [SHARED ELEMENTS] Shared Element Transitions (SET)
*"What is a Shared Element Transition, and what are its limitations in Reanimated 3?"*

SET allows a component to appear as if it's moving from one screen to another. Reanimated 3 uses `sharedTransitionTag` to identify matching components across different navigation screens.

**Limitations:**
- **Fabric Support:** Fabric (New Arch) support for SET is still maturing.
- **Image Loading:** If the destination image hasn't loaded yet, the transition might "pop" or flicker.
- **Nested Views:** Complex nested structures can confuse the layout measurement engine.

> [!TIP]
> **Senior Insight: The "Placeholder" Trick**
> To ensure a smooth SET, always render a low-resolution placeholder or a colored box with the same dimensions on the destination screen to prevent layout shifts during the transition.

> [!WARNING]
> **Follow-up Trap:** "Can you use Shared Element Transitions with React Navigation's native stack?"
> **Answer:** Yes, but it requires specific configuration. Reanimated's SET works by intercepting the screen transition, which can sometimes conflict with native OS transitions (like iOS swipe-back).

---

### Q9: [SENSORS] Using Device Sensors with Animations
*"How do you use useAnimatedSensor to create a 'parallax' effect?"*

`useAnimatedSensor` provides access to the Gyroscope, Accelerometer, or Magnetometer directly on the UI thread. This is critical for performance because sensor data can fire at 100Hz+.

```typescript
// Parallax with Gyroscope
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

> [!TIP]
> **Senior Insight: Sensor Fusion**
> For professional parallax, don't rely on raw gyroscope data alone as it drifts. Senior devs use `SensorType.ROTATION` (which uses sensor fusion on the native side) to get a stable, absolute orientation of the device.

> [!WARNING]
> **Follow-up Trap:** "Does `useAnimatedSensor` drain the battery?"
> **Answer:** Yes, if not managed. You should pass an `interval` option (e.g., 16ms for 60fps) and ensure the sensor is only active when the component is focused.

---


### Q10: [GESTURE COMPOSITION] Simultaneous Gesture Handling
*"How do you allow a user to pinch-to-zoom and pan an image at the same time?"*

Using RNGH v2, you compose gestures using `Gesture.Simultaneous()`. Without this, one gesture would "cancel" the other as soon as it's recognized.

```typescript
// Simultaneous Pan & Pinch
const pan = Gesture.Pan();
const pinch = Gesture.Pinch();

const composed = Gesture.Simultaneous(pan, pinch);

return <GestureDetector gesture={composed}>...</GestureDetector>;
```

> [!TIP]
> **Senior Insight: State Management**
> When combining gestures, you often need to store the "last known" scale and translation in shared values to prevent the image from "jumping" back to the start when the user begins a second interaction.

> [!WARNING]
> **Follow-up Trap:** "How do you prevent a pinch gesture from triggering a swipe-back navigation on iOS?"
> **Answer:** Use `Gesture.Pan().activeOffsetX([-10, 10])` or wrap the component in a `NativeViewGestureHandler` to prioritize the internal gestures over the navigation controller.

---

### Q11: [C++ WORKLETS] The "runOnJS" Function
*"Why can't you call a normal JS function directly from a worklet, and how do you use runOnJS?"*

Worklets run in a separate JavaScript runtime (the UI thread's VM). They don't have access to the JS thread's closure, variables, or functions. `runOnJS` acts as a bridge to send a message back to the JS thread.

> [!TIP]
> **Senior Insight: Thread Hopping**
> Overusing `runOnJS` can defeat the purpose of Reanimated. If you call `runOnJS` every frame (e.g., to update React state during an animation), you're back to bridge-limited performance. Only use it for "terminal" events like `onEnd` or `onFinalize`.

> [!WARNING]
> **Follow-up Trap:** "Can you pass complex objects (like a full React component) to `runOnJS`?"
> **Answer:** No. Only serializable data or functions defined on the JS thread can be passed. Complex React objects cannot be 'teleported' across threads like that.

---

### Q12: [LAYOUT MEASUREMENT] measure() and getRelativeCoords()
*"How do you get the position and size of a component on the UI thread?"*

The `measure(animatedRef)` function is a synchronous call on the UI thread. It returns `x`, `y`, `width`, `height`, `pageX`, and `pageY`.

```typescript
// Measuring UI Elements
const aRef = useAnimatedRef<View>();

const onTouch = Gesture.Tap().onEnd(() => {
  const layout = measure(aRef);
  console.log('Width is:', layout.width);
});
```

> [!TIP]
> **Senior Insight: Coordinate Conversion**
> Senior devs use `getRelativeCoords` when they need to know where a touch event happened *relative* to a specific component. This is vital for "Touch to Ripple" effects or custom sliders where the local X/Y is more important than the screen-wide PageX/Y.

> [!WARNING]
> **Follow-up Trap:** "Why does `measure()` sometimes return null?"
> **Answer:** It returns null if the component hasn't been mounted yet or if the native view hasn't performed its first layout pass.

---

### Q13: [SVG ANIMATIONS] Path Interpolation & Skia
*"How do you animate an SVG path (e.g., morphing shapes)?"*

While `react-native-svg` works, **React Native Skia** is the modern standard for complex path animations. Skia runs entirely on the UI thread and uses the same rendering engine as Flutter.

> [!TIP]
> **Senior Insight: Skia vs. SVG**
> For simple icons, use SVG. For dynamic charts, morphing shapes, or complex shaders (like glassmorphism), use Skia. Skia's `Path` object allows for synchronous `interpolatePaths`, which is much smoother than JS-based string manipulation.

> [!WARNING]
> **Follow-up Trap:** "Can you animate a path between two strings that have a different number of points?"
> **Answer:** Standard interpolation requires the same number of points. To morph between different shapes, you need to use a library like `flubber` (on the JS thread) or pre-process the paths to have matching segments.

---

### Q14: [SCROLL EVENTS] useAnimatedScrollHandler
*"How do you create a 'Sticky Header' effect that responds to scroll position?"*

You bind a shared value to the `onScroll` event. This value can then drive the `translateY` of a header component.

```typescript
// Scroll-Driven Header
const scrollY = useSharedValue(0);

const scrollHandler = useAnimatedScrollHandler((event) => {
  scrollY.value = event.contentOffset.y;
});

const headerStyle = useAnimatedStyle(() => ({
  transform: [{ translateY: interpolate(scrollY.value, [0, 100], [0, -50], Extrapolation.CLAMP) }],
}));
```

> [!TIP]
> **Senior Insight: Fling Velocity**
> A senior implementation doesn't just look at the scroll position; it looks at `velocity.y`. If the user "flings" the list, you can use that velocity to hide the header faster or trigger a "back-to-top" button with a spring-loaded entrance.

> [!WARNING]
> **Follow-up Trap:** "How do you prevent the scroll event from clogging the bridge?"
> **Answer:** By using `useAnimatedScrollHandler`, the event is handled entirely on the UI thread. No data is sent to the JS thread unless you explicitly call `runOnJS`.

---

### Q15: [DEBUGGING] Debugging Reanimated
*"How do you debug values that live on the UI thread?"*

Standard `console.log` works but can be misleading. A better way is `useAnimatedReaction`.

```typescript
// Animated Reaction Debugging
useAnimatedReaction(
  () => offset.value,
  (current, previous) => {
    if (current !== previous) {
      console.log("Value changed to:", current);
    }
  }
);
```

> [!TIP]
> **Senior Insight: Flipper & Layout Inspector**
> Use the Flipper "Layout" plugin to see if views are actually moving on the native side. Sometimes an animation "runs" but the view is clipped or hidden by a parent, making it look like it's failing.

> [!WARNING]
> **Follow-up Trap:** "Can you use breakpoints inside a worklet?"
> **Answer:** Generally no, as they run in a separate C++-based JS VM (Hermes/JSC) on the UI thread. Logging and the Layout Inspector are your primary tools.

---


### Q16: [SEQUENCING] withSequence and withDelay

*"How do you create a 'staggered' animation effect for a list of items?"*

Staggering is achieved by applying a `withDelay` based on the item's index. `withSequence` is used for multi-stage animations on a single item (e.g., pop up, then shake, then fade).

```typescript
const style = useAnimatedStyle(() => ({
  opacity: withDelay(index * 100, withTiming(1)),
  transform: [{ scale: withDelay(index * 100, withSpring(1)) }],
}));
```

> [!WARNING]
> **Follow-up Trap: "What is the risk of using too many delayed animations in a long list?"**
> **Answer:** Memory usage. Each animation creates a worklet and a timer on the UI thread. For lists with 100+ items, use `FlashList`'s `onLoad` or similar to only animate visible items.

---

### Q17: [HAPTICS] Triggering Haptics with Gestures

*"When and how should you trigger haptic feedback during an animation?"*

Haptics should be used to confirm "Success," "Warning," or "Impact." In Reanimated, you must trigger them via `runOnJS`.

> [!TIP]
> **Senior Insight: Haptic Overkill**
> Never trigger haptics on every frame of a gesture (e.g., during `onUpdate`). Only trigger them when a state changes (e.g., `onActive`) or when a threshold is crossed (e.g., a "swipe-to-delete" distance is met).

---

### Q18: [CANVAS] Skia Canvas vs. Native Views

*"Why is Skia often faster than using 100 Animated.Views?"*

Every `Animated.View` is a real native view (iOS `UIView`, Android `View`). Each has its own layout, layer, and memory overhead. **Skia** uses a single native view (a Canvas) and draws everything inside it manually using GPU-accelerated commands. It's essentially like a game engine for your UI.

> [!WARNING]
> **Follow-up Trap: "Can you use standard React components (like `<Text>`) inside a Skia Canvas?"**
> **Answer:** No. You must use Skia's own components (`<Text />`, `<Rect />`, `<Image />`) which don't support standard CSS styling.

---

### Q19: [GESTURE STATES] Handling "Canceled" Gestures

*"Why is it important to handle the 'onFinalize' or 'onCancel' state of a gesture?"*

Gestures can be interrupted by the system (e.g., an incoming call) or by other gestures (e.g., a parent ScrollView taking over). If you only handle `onEnd`, your UI might stay in its "active" state (e.g., scaled up or semi-transparent) forever.

> [!TIP]
> **Senior Insight: The onFinalize pattern**
> RNGH v2's `onFinalize` runs whether the gesture succeeded or was canceled. It's the best place to put "cleanup" logic (like resetting a shared value to its original state).

---

### Q20: [ACCESSIBILITY] Reduced Motion Support

*"How do you respect a user's 'Reduce Motion' system setting?"*

Forcing animations on users with vestibular disorders can cause physical discomfort. Use `AccessibilityInfo.isReduceMotionEnabled()` to check the setting.

```typescript
const config = isReduceMotion ? { duration: 0 } : { damping: 10 };
offset.value = withSpring(100, config);
```

> [!TIP]
> **Senior Insight: Platform Inconsistency**
> On iOS, "Reduce Motion" also affects system transitions. On Android, it's often ignored by third-party apps. A senior dev will implement a custom "App Settings" toggle that overrides the system setting if needed.

---

### Q21: [MEMOIZATION] useAnimatedRef vs. useRef

*"When must you use useAnimatedRef instead of a standard React useRef?"*

`useAnimatedRef` is a specialized hook that returns a ref object that can be shared between the JS and UI threads. It's required for functions like `measure()`, `scrollTo()`, and `dispatchCommand()` which need to interact with the native view directly from a worklet.

> [!WARNING]
> **Follow-up Trap: "Can you use `useAnimatedRef` to change a View's background color directly?"**
> **Answer:** No. `useAnimatedRef` is for measurement and commands. To change styles, you should use `useAnimatedStyle`.

---


### Q22: [ANIMATION BASICS] Basic animations with React Native's Animated API

The Animated API is React Native's built-in animation system.

```javascript
import { Animated, TouchableOpacity } from 'react-native';

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
    <Animated.View style={{ opacity: fadeAnim }}>
      {children}
    </Animated.View>
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
    <TouchableOpacity onPress={animate}>
      <Animated.View style={{
        transform: [
          { scale: scaleAnim },
          { rotate },
        ],
      }}>
        <Text>Tap to Animate</Text>
      </Animated.View>
    </TouchableOpacity>
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
    <Animated.View style={{
      opacity: fadeAnim,
      transform: [{ translateX: slideAnim }],
    }}>
      <Text>Fade and Slide In</Text>
    </Animated.View>
  );
}
```

> [!TIP]
> **Beginner Tip: Native Driver**
> Always use `useNativeDriver: true` for better performance. This runs animations on the native thread instead of the JS thread, preventing frame drops during heavy JS work.

---

### Q23: [GESTURE BASICS] Basic gesture handling with PanResponder

PanResponder is React Native's built-in gesture recognition system.

```javascript
import { PanResponder, Animated } from 'react-native';

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
    <Animated.View
      {...panResponder.panHandlers}
      style={{
        transform: [{ translateX: pan.x }, { translateY: pan.y }],
      }}
    >
      <View style={styles.box}>
        <Text>Drag me!</Text>
      </View>
    </Animated.View>
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
    <Animated.View
      {...panResponder.panHandlers}
      style={{
        transform: [{ translateX }],
      }}
    >
      <View style={styles.card}>
        <Text>Swipe me left or right</Text>
      </View>
    </Animated.View>
  );
}
```

---

### Q24: [REANIMATED BASICS] Introduction to React Native Reanimated

Reanimated is a powerful animation library that runs animations on the UI thread.

```javascript
import Animated, {
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
    <Animated.View style={[styles.box, animatedStyles]}>
      <TouchableOpacity onPress={handlePress}>
        <Text>Tap to animate</Text>
      </TouchableOpacity>
    </Animated.View>
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
    <GestureDetector gesture={gesture}>
      <Animated.View style={[styles.box, animatedStyles]}>
        <Text>Drag me around</Text>
      </Animated.View>
    </GestureDetector>
  );
}
```

> [!TIP]
> **Beginner Tip: Threading**
> Reanimated animations continue running even when the JS thread is blocked, making them perfect for smooth interactions. Shared values can be safely accessed from both JS and UI threads.

---

### Q25: [SPRING PHYSICS] Creating natural-feeling animations with spring physics

Spring animations feel more natural than linear timing animations.

```javascript
// Reanimated springs
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
    <Pressable onPressIn={handlePress} onPressOut={handlePressOut}>
      <Animated.View style={[styles.button, animatedStyle]}>
        <Text>Spring Button</Text>
      </Animated.View>
    </Pressable>
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
    <TouchableOpacity onPress={startBounce}>
      <Animated.View style={[styles.box, animatedStyle]}>
        <Text>Bounce me!</Text>
      </Animated.View>
    </TouchableOpacity>
  );
}
```

---

### Q26: [GESTURE RECOGNITION] Advanced gesture recognition with RNGH

React Native Gesture Handler provides reliable gesture recognition.

```javascript
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
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
    <GestureDetector gesture={pinchGesture}>
      <Animated.View style={[styles.image, animatedStyle]}>
        <Text>Pinch to zoom</Text>
      </Animated.View>
    </GestureDetector>
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
    <GestureDetector gesture={composedGesture}>
      <Animated.View style={[styles.box, animatedStyle]}>
        <Text>Pan and pinch simultaneously</Text>
      </Animated.View>
    </GestureDetector>
  );
}
```

---


### Q27: [SCROLL] Scroll-Driven Animations & Parallax
*"How do you create a 'Sticky Header' or 'Parallax' effect that responds to scroll position?"*

You bind a shared value to the `onScroll` event using `useAnimatedScrollHandler`. This value can then drive the transformations of other components via `useAnimatedStyle` and `interpolate`.

```typescript
// Scroll-Driven Parallax Header
const scrollY = useSharedValue(0);

const scrollHandler = useAnimatedScrollHandler({
  onScroll: (event) => {
    scrollY.value = event.contentOffset.y;
  },
});

const headerStyle = useAnimatedStyle(() => ({
  transform: [
    { translateY: scrollY.value * 0.5 }, // Parallax effect
  ],
  opacity: interpolate(
    scrollY.value,
    [0, 200],
    [1, 0],
    Extrapolation.CLAMP
  ),
}));
```

> [!TIP]
> **Senior Insight: Fling Velocity & Snap Points**
> A senior implementation doesn't just look at the scroll position; it looks at `velocity.y`. If the user "flings" the list, you can use that velocity to hide the header faster. Additionally, use `snapToOffsets` on the ScrollView to ensure the header always stops at a clean state.

> [!WARNING]
> **Follow-up Trap:** "How do you prevent the scroll event from clogging the bridge?"
> **Answer:** By using `useAnimatedScrollHandler`, the event is handled entirely on the UI thread. No data is sent to the JS thread unless you explicitly call `runOnJS`.

---

### Q28: [ACCESSIBILITY] Reduced Motion & User Preferences
*"How do you respect a user's 'Reduce Motion' system setting in your animations?"*

Forcing animations on users with vestibular disorders can cause physical discomfort. You should check the system setting and either disable animations or use a non-moving transition (like a simple fade).

```typescript
import { AccessibilityInfo } from 'react-native';

// Reanimated 3 approach
const config = {
  duration: 500,
  reduceMotion: ReduceMotion.System, // Automatically respects system setting
};

offset.value = withSpring(100, config);
```

> [!TIP]
> **Senior Insight: Adaptive UI**
> Instead of just turning animations off, consider "Adaptive UI." If motion is reduced, replace a slide-in animation with a subtle fade-in. This preserves the visual hierarchy and feedback without triggering motion sensitivity.

---

### Q29: [PERFORMANCE] Optimizing Complex Gesture interactions
*"What are the primary performance bottlenecks when handling multiple simultaneous gestures?"*

The main bottlenecks are **JS thread blocking** and **over-communication between threads**. 

**Optimization Strategies:**
1. **Worklets:** Ensure all gesture logic stays inside `'worklet';` functions.
2. **Avoid runOnJS:** Minimize calls back to the JS thread during active gestures.
3. **Layout Isolation:** Use `overflow: 'hidden'` and `pointerEvents` correctly to prevent the native layer from over-calculating layouts.
4. **GPU Acceleration:** Only animate properties that are GPU-accelerated (transform, opacity).

> [!WARNING]
> **Follow-up Trap:** "Why does adding a `console.log` inside `onUpdate` make my animation janky?"
> **Answer:** Even if Reanimated handles the log, it still introduces synchronous overhead on the UI thread. In some environments, it might even trigger a bridge transfer to show the log in the JS console, causing frame drops.

---

### Q30: [DEBUGGING] Advanced Animation Debugging
*"How do you debug an animation that looks correct in code but 'flickers' or 'pops' on the device?"*

1. **useAnimatedReaction:** Use this to log values as they change on the UI thread without stopping the animation.
2. **Layout Inspector:** Use Flipper or React DevTools to check if the native view's properties (like `zIndex` or `elevation`) are changing unexpectedly.
3. **Slow Animations:** Wrap your animations in a global multiplier (e.g., `duration * 10`) to see exactly where the "pop" occurs.
4. **Thread Monitoring:** Check if the UI thread is actually hitting 60fps or if the native layout pass is taking too long.

> [!TIP]
> **Senior Insight: The "Z-Index" Ghost**
> Flickering on Android is often caused by `elevation`. When two views animate near each other, Android's depth sorter might flip their render order mid-animation. Explicitly managing `zIndex` and `elevation` during the animation lifecycle is a common fix.

---

[⬅️ Phase 07: Styling](phase7-styling-design.md) | [Phase 09: Testing ➡️](phase9-testing-qa.md)