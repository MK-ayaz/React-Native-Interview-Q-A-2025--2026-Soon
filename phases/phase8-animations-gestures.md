# Phase 8: Animations & Gestures 🏃‍♂️

Welcome to **Phase 8**. Senior React Native developers distinguish themselves by creating fluid, "app-like" experiences. This phase covers advanced animations using Reanimated and complex gesture handling.

---

## 📋 Table of Contents
1. [Animated API vs. Reanimated](#1-animated-api-vs-reanimated)
2. [Reanimated 3 Core Concepts](#2-reanimated-3-core-concepts)
3. [Gesture Handling (RNGH)](#3-gesture-handling-rngh)
4. [Shared Element Transitions](#4-shared-element-transitions)
5. [Layout Animations](#5-layout-animations)

---

## 1. Animated API vs. Reanimated
Why do we prefer Reanimated over the built-in `Animated` API?

| Feature | Animated API | Reanimated (v2/v3) |
| :--- | :--- | :--- |
| **Thread** | JS Thread (mostly) | UI Thread (Native) |
| **Performance** | Can drop frames if JS is busy | Stays at 60fps regardless of JS load |
| **Logic** | Limited interpolation | Full C++ worklets (JS-like logic) |
| **Complexity** | Simple fades/slides | Complex physics & gestures |

---

## 2. Reanimated 3 Core Concepts
Reanimated 3 uses **Shared Values** and **Worklets** to run animations directly on the UI thread.

### Example: Simple Animation
```javascript
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

const Box = () => {
  const width = useSharedValue(100);

  const animatedStyle = useAnimatedStyle(() => ({
    width: withSpring(width.value),
  }));

  return <Animated.View style={[styles.box, animatedStyle]} />;
};
```

---

## 3. Gesture Handling (RNGH)
The built-in `PanResponder` is slow because it communicates back and forth with the JS thread. `react-native-gesture-handler` runs natively.

### Swipe-to-Delete Example
```javascript
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

const pan = Gesture.Pan()
  .onUpdate((event) => {
    translateX.value = event.translationX;
  })
  .onEnd(() => {
    if (translateX.value > 100) {
      // Trigger delete
    }
  });

return (
  <GestureDetector gesture={pan}>
    <Animated.View style={animatedStyle} />
  </GestureDetector>
);
```

---

## 4. Shared Element Transitions
Shared element transitions allow a component to "fly" from one screen to another during navigation. Reanimated 3 makes this trivial with the `sharedTransitionTag` prop.

```tsx
<Animated.Image
  source={{ uri: item.image }}
  sharedTransitionTag={`image-${item.id}`}
/>
```

---

## 5. Layout Animations
Want your list items to slide in when added? Use Layout Animations.

```tsx
<Animated.View
  entering={FadeIn.duration(500)}
  exiting={FadeOut}
  layout={Layout.springify()}
>
  <Text>New Item</Text>
</Animated.View>
```

---

## 🏁 Next Steps
Your app is now fast, beautiful, and fluid. But is it reliable? In **Phase 9**, we explore **Testing & Quality Assurance** to ensure your code stands the test of time.

[Back to Main README](../README.md) | [Go to Phase 9: Testing & QA](./phase9-testing-qa.md)
