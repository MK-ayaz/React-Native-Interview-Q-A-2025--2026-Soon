[⬅️ Back to Home](../README.md)

# 🎯 Phase 11: Advanced Patterns & Architecture

---

const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, {
    user: null,
    loading: true,
    error: null,
  });

  const login = async (credentials) => {
    dispatch({ type: 'SET_LOADING', payload: true });
    try {
      const user = await api.login(credentials);
      dispatch({ type: 'SET_USER', payload: user });
    } catch (error) {
      dispatch({ type: 'SET_ERROR', payload: error.message });
    }
  };

  const logout = () => {
    dispatch({ type: 'LOGOUT' });
  };

  return (
    <AppContext.Provider value={{ ...state, login, logout }}>
      {children}
    </AppContext.Provider>
  );
};

// Redux Implementation
// store.js
const userSlice = createSlice({
  name: 'user',
  initialState: {
    currentUser: null,
    loading: false,
    error: null,
  },
  reducers: {
    loginStart: (state) => {
      state.loading = true;
      state.error = null;
    },
    loginSuccess: (state, action) => {
      state.currentUser = action.payload;
      state.loading = false;
    },
    loginFailure: (state, action) => {
      state.error = action.payload;
      state.loading = false;
    },
    logout: (state) => {
      state.currentUser = null;
      state.error = null;
    },
  },
});

export const { loginStart, loginSuccess, loginFailure, logout } = userSlice.actions;

// userThunks.js
export const loginUser = (credentials) => async (dispatch) => {
  dispatch(loginStart());
  try {
    const user = await api.login(credentials);
    dispatch(loginSuccess(user));
  } catch (error) {
    dispatch(loginFailure(error.message));
  }
};

// selectors.js
export const selectUser = (state) => state.user.currentUser;
export const selectUserLoading = (state) => state.user.loading;
export const selectUserError = (state) => state.user.error;

// Zustand Implementation
import create from 'zustand';
import { devtools, persist } from 'zustand/middleware';

const useUserStore = create(
  devtools(
    persist(
      (set, get) => ({
        user: null,
        loading: false,
        error: null,

        login: async (credentials) => {
          set({ loading: true, error: null });
          try {
            const user = await api.login(credentials);
            set({ user, loading: false });
          } catch (error) {
            set({ error: error.message, loading: false });
          }
        },

        logout: () => {
          set({ user: null, error: null });
        },

        updateProfile: (updates) => {
          const { user } = get();
          if (user) {
            set({ user: { ...user, ...updates } });
          }
        },

        clearError: () => set({ error: null }),
      }),
      {
        name: 'user-storage',
        partialize: (state) => ({ user: state.user }),
      }
    ),
    { name: 'user-store' }
  )
);

// Performance Comparison Components
const ContextUserProfile = () => {
  const { user, loading } = useContext(AppContext);

  if (loading) return <ActivityIndicator />;
  if (!user) return <Text>Not logged in</Text>;

  return <Text>Welcome {user.name}!</Text>;
};

const ReduxUserProfile = () => {
  const user = useSelector(selectUser);
  const loading = useSelector(selectUserLoading);

  if (loading) return <ActivityIndicator />;
  if (!user) return <Text>Not logged in</Text>;

  return <Text>Welcome {user.name}!</Text>;
};

const ZustandUserProfile = () => {
  const { user, loading } = useUserStore();

  if (loading) return <ActivityIndicator />;
  if (!user) return <Text>Not logged in</Text>;

  return <Text>Welcome {user.name}!</Text>;
};

// Complex State Management Example
// Shopping Cart with multiple state management solutions

// Context + useReducer for Cart
const CartContext = React.createContext();

const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
      };

    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload),
      };

    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        ),
      };

    case 'CLEAR_CART':
      return { ...state, items: [] };

    default:
      return state;
  }
};

const CartProvider = ({ children }) => {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  const addItem = (item) => dispatch({ type: 'ADD_ITEM', payload: item });
  const removeItem = (id) => dispatch({ type: 'REMOVE_ITEM', payload: id });
  const updateQuantity = (id, quantity) => dispatch({
    type: 'UPDATE_QUANTITY',
    payload: { id, quantity }
  });
  const clearCart = () => dispatch({ type: 'CLEAR_CART' });

  const total = state.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);

  return (
    <CartContext.Provider value={{
      items: state.items,
      total,
      addItem,
      removeItem,
      updateQuantity,
      clearCart,
    }}>
      {children}
    </CartContext.Provider>
  );
};

// Redux Cart Slice
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addItem: (state, action) => {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        existingItem.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
    },
    removeItem: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
    },
    updateQuantity: (state, action) => {
      const item = state.items.find(item => item.id === action.payload.id);
      if (item) {
        item.quantity = action.payload.quantity;
      }
    },
    clearCart: (state) => {
      state.items = [];
    },
  },
});

export const { addItem, removeItem, updateQuantity, clearCart } = cartSlice.actions;

export const selectCartItems = (state) => state.cart.items;
export const selectCartTotal = (state) =>
  state.cart.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);

// Zustand Cart Store
const useCartStore = create((set, get) => ({
  items: [],

  addItem: (item) => set((state) => {
    const existingItem = state.items.find(i => i.id === item.id);
    if (existingItem) {
      return {
        items: state.items.map(i =>
          i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
        ),
      };
    }
    return { items: [...state.items, { ...item, quantity: 1 }] };
  }),

  removeItem: (id) => set((state) => ({
    items: state.items.filter(item => item.id !== id),
  })),

  updateQuantity: (id, quantity) => set((state) => ({
    items: state.items.map(item =>
      item.id === id ? { ...item, quantity } : item
    ),
  })),

  clearCart: () => set({ items: [] }),

  get total() {
    return get().items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  },
}));

// Performance and Bundle Size Comparison
const BundleSizeComparison = {
  ContextAPI: '~2KB (React built-in)',
  Redux: '~8KB (with RTK)',
  Zustand: '~3KB (core) + middlewares',
  MobX: '~20KB (runtime + decorators)',
};

// Performance Characteristics
const PerformanceComparison = {
  ContextAPI: {
    'Simple State': 'Excellent',
    'Complex State': 'Poor (re-renders)',
    'Async Actions': 'Manual implementation',
    'DevTools': 'None built-in',
  },
  Redux: {
    'Simple State': 'Good (boilerplate)',
    'Complex State': 'Excellent',
    'Async Actions': 'Thunks/Sagas',
    'DevTools': 'Excellent',
  },
  Zustand: {
    'Simple State': 'Excellent',
    'Complex State': 'Good',
    'Async Actions': 'Built-in support',
    'DevTools': 'Good (middleware)',
  },
};

// Migration Strategies
const MigrationStrategies = {
  // Context → Redux
  contextToRedux: `
  1. Create Redux slices for each context
  2. Move reducers to slices
  3. Replace useContext with useSelector/useDispatch
  4. Update component props
  5. Test thoroughly (state shape changes)
  `,

  // Redux → Zustand
  reduxToZustand: `
  1. Create Zustand stores for each slice
  2. Move state and actions to stores
  3. Replace useSelector with store hooks
  4. Remove Redux boilerplate
  5. Update TypeScript types
  `,

  // Context → Zustand
  contextToZustand: `
  1. Create Zustand store with context state
  2. Move context logic to store
  3. Replace useContext with useStore
  4. Remove context provider
  5. Add persistence if needed
  `,
};
```

---

## 🎯 Phase 9 — Animations & Gestures

**Animation and gesture questions demonstrate advanced UX skills** - expect questions about Reanimated 3, PanResponder, and Lottie integration.

### 9.1 **React Native Reanimated 3 Advanced Patterns**

**Question:** *"Implement complex animations with shared values, gesture-based interactions, and performance optimizations using React Native Reanimated 3."*

**Perfect Answer Structure:**
```
REANIMATED 3 ARCHITECTURE:

🎯 SHARED VALUES & DERIVED VALUES:
- Reactive state management
- Automatic dependency tracking
- Memory-efficient updates
- Cross-thread communication

🎯 GESTURE-BASED ANIMATIONS:
- Pan, pinch, rotation gestures
- Gesture composition
- Simultaneous gesture handling
- Platform-specific optimizations

🎯 PERFORMANCE OPTIMIZATIONS:
- Worklet functions
- UI thread execution
- Reduced bridge communication
- 60fps animations
```

**Complete Reanimated 3 Implementation:**
```javascript
import Animated, {
  useSharedValue,
  useDerivedValue,
  useAnimatedStyle,
  useAnimatedGestureHandler,
  runOnUI,
  runOnJS,
  withTiming,
  withSpring,
  withSequence,
  withDelay,
  Easing,
  interpolate,
  Extrapolate,
} from 'react-native-reanimated';
import { PanGestureHandler, PinchGestureHandler } from 'react-native-gesture-handler';

// Advanced Shared Values and Derived Values
const useAdvancedAnimation = () => {
  const progress = useSharedValue(0);
  const scale = useSharedValue(1);
  const rotation = useSharedValue(0);

  // Derived values for complex calculations
  const animatedProgress = useDerivedValue(() => {
    return progress.value * 100;
  });

  const interpolatedColor = useDerivedValue(() => {
    return interpolate(
      progress.value,
      [0, 0.5, 1],
      ['#FF0000', '#FFFF00', '#00FF00'],
      Extrapolate.CLAMP
    );
  });

  const combinedTransform = useDerivedValue(() => {
    const translateX = interpolate(
      progress.value,
      [0, 1],
      [0, 200],
      Extrapolate.CLAMP
    );

    return {
      transform: [
        { translateX },
        { scale: scale.value },
        { rotate: `${rotation.value}deg` },
      ],
    };
  });

  const startAnimation = () => {
    progress.value = withTiming(1, { duration: 1000 });
    scale.value = withSpring(1.5, { damping: 10 });
    rotation.value = withTiming(360, { duration: 2000 });
  };

  const resetAnimation = () => {
    progress.value = withTiming(0);
    scale.value = withTiming(1);
    rotation.value = withTiming(0);
  };

  return {
    progress,
    scale,
    rotation,
    animatedProgress,
    interpolatedColor,
    combinedTransform,
    startAnimation,
    resetAnimation,
  };
};

// Complex Gesture-Based Animations
const useGestureAnimations = () => {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);
  const scale = useSharedValue(1);
  const rotation = useSharedValue(0);

  const panGestureHandler = useAnimatedGestureHandler({
    onStart: (_, context) => {
      context.startX = translateX.value;
      context.startY = translateY.value;
    },
    onActive: (event, context) => {
      translateX.value = context.startX + event.translationX;
      translateY.value = context.startY + event.translationY;

      // Add resistance at edges
      if (Math.abs(translateX.value) > 100) {
        translateX.value = context.startX + event.translationX * 0.5;
      }
    },
    onEnd: (event) => {
      // Snap back with spring animation
      if (Math.abs(translateX.value) < 50) {
        translateX.value = withSpring(0);
        translateY.value = withSpring(0);
      } else {
        // Animate to edge
        const direction = translateX.value > 0 ? 1 : -1;
        translateX.value = withSpring(direction * 150);
        translateY.value = withSpring(0);
      }
    },
  });

  const pinchGestureHandler = useAnimatedGestureHandler({
    onStart: (_, context) => {
      context.startScale = scale.value;
    },
    onActive: (event, context) => {
      scale.value = context.startScale * event.scale;

      // Limit scale bounds
      scale.value = Math.max(0.5, Math.min(3, scale.value));
    },
    onEnd: () => {
      // Snap to nearest scale
      const snapPoints = [0.5, 1, 1.5, 2, 3];
      const closest = snapPoints.reduce((prev, curr) =>
        Math.abs(curr - scale.value) < Math.abs(prev - scale.value) ? curr : prev
      );
      scale.value = withSpring(closest);
    },
  });

  const rotationGestureHandler = useAnimatedGestureHandler({
    onStart: (_, context) => {
      context.startRotation = rotation.value;
    },
    onActive: (event, context) => {
      rotation.value = context.startRotation + event.rotation;
    },
    onEnd: (event) => {
      // Snap to nearest 90 degrees
      const normalizedRotation = rotation.value % 360;
      const snapAngles = [0, 90, 180, 270];
      const closest = snapAngles.reduce((prev, curr) =>
        Math.abs(curr - normalizedRotation) < Math.abs(prev - normalizedRotation) ? curr : prev
      );
      rotation.value = withSpring(closest);
    },
  });

  return {
    translateX,
    translateY,
    scale,
    rotation,
    panGestureHandler,
    pinchGestureHandler,
    rotationGestureHandler,
  };
};

// Worklet Functions for Performance
const heavyCalculation = (input) => {
  'worklet';
  let result = input;
  for (let i = 0; i < 1000000; i++) {
    result += Math.sin(i) * Math.cos(i);
  }
  return result;
};

const useWorkletAnimation = () => {
  const animatedValue = useSharedValue(0);

  const runHeavyCalculation = () => {
    runOnUI(() => {
      'worklet';
      const result = heavyCalculation(animatedValue.value);
      console.log('Heavy calculation result:', result);
    })();
  };

  const updateFromJS = (value) => {
    // Run on UI thread
    runOnUI(() => {
      'worklet';
      animatedValue.value = value;
    })();
  };

  const notifyJS = () => {
    // Communicate back to JS thread
    runOnJS(() => {
      console.log('Animation completed on JS thread');
    })();
  };

  return {
    animatedValue,
    runHeavyCalculation,
    updateFromJS,
    notifyJS,
  };
};

// Complex Animation Sequences
const useSequenceAnimations = () => {
  const opacity = useSharedValue(0);
  const translateY = useSharedValue(50);
  const scale = useSharedValue(0.8);

  const startEntranceAnimation = () => {
    // Staggered entrance animation
    opacity.value = withDelay(0, withTiming(1, { duration: 300 }));
    translateY.value = withDelay(100, withSpring(0, { damping: 12 }));
    scale.value = withDelay(200, withSpring(1, { damping: 12 }));
  };

  const startExitAnimation = () => {
    // Reverse exit animation
    scale.value = withTiming(0.8, { duration: 200 });
    translateY.value = withTiming(50, { duration: 200 });
    opacity.value = withDelay(100, withTiming(0, { duration: 200 }));
  };

  const pulseAnimation = () => {
    scale.value = withSequence(
      withTiming(1.2, { duration: 150 }),
      withTiming(1, { duration: 150 }),
      withTiming(1.1, { duration: 150 }),
      withTiming(1, { duration: 150 })
    );
  };

  return {
    opacity,
    translateY,
    scale,
    startEntranceAnimation,
    startExitAnimation,
    pulseAnimation,
  };
};

// Lottie Animation Integration with Reanimated
const useLottieWithReanimated = (lottieRef) => {
  const progress = useSharedValue(0);
  const isPlaying = useSharedValue(false);

  const playAnimation = () => {
    isPlaying.value = true;
    progress.value = withTiming(1, { duration: 2000 }, (finished) => {
      if (finished) {
        isPlaying.value = false;
        progress.value = 0;
      }
    });
  };

  const pauseAnimation = () => {
    isPlaying.value = false;
    progress.value = progress.value; // Keep current progress
  };

  const resetAnimation = () => {
    progress.value = 0;
    isPlaying.value = false;
  };

  // Sync with Lottie
  useEffect(() => {
    if (lottieRef.current) {
      lottieRef.current.play();
    }
  }, []);

  return {
    progress,
    isPlaying,
    playAnimation,
    pauseAnimation,
    resetAnimation,
  };
};

// Performance Monitoring for Animations
const useAnimationPerformanceMonitor = () => {
  const frameCount = useSharedValue(0);
  const lastFrameTime = useSharedValue(0);
  const fps = useSharedValue(60);

  const updateFrame = () => {
    'worklet';
    const now = performance.now();
    frameCount.value += 1;

    if (lastFrameTime.value !== 0) {
      const deltaTime = now - lastFrameTime.value;
      fps.value = 1000 / deltaTime;
    }

    lastFrameTime.value = now;

    // Log performance issues
    if (fps.value < 50) {
      console.warn('Low FPS detected:', fps.value);
    }
  };

  return {
    fps,
    updateFrame,
  };
};

// Complete Animated Component Example
const AdvancedAnimatedCard = ({ children, onPress }) => {
  const {
    translateX,
    translateY,
    scale,
    panGestureHandler,
  } = useGestureAnimations();

  const {
    opacity,
    startEntranceAnimation,
    pulseAnimation,
  } = useSequenceAnimations();

  const { updateFrame } = useAnimationPerformanceMonitor();

  const animatedStyle = useAnimatedStyle(() => {
    updateFrame(); // Monitor performance

    return {
      opacity: opacity.value,
      transform: [
        { translateX: translateX.value },
        { translateY: translateY.value },
        { scale: scale.value },
      ],
    };
  });

  useEffect(() => {
    startEntranceAnimation();
  }, []);

  const handlePress = () => {
    pulseAnimation();
    onPress?.();
  };

  return (
    <PanGestureHandler onGestureEvent={panGestureHandler}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <TouchableWithoutFeedback onPress={handlePress}>
          {children}
        </TouchableWithoutFeedback>
      </Animated.View>
    </PanGestureHandler>
  );
};

// Layout Animations with Reanimated
const useLayoutAnimations = () => {
  const height = useSharedValue(100);
  const width = useSharedValue(200);

  const expandCard = () => {
    height.value = withSpring(200);
    width.value = withSpring(300);
  };

  const collapseCard = () => {
    height.value = withSpring(100);
    width.value = withSpring(200);
  };

  const animatedLayoutStyle = useAnimatedStyle(() => ({
    height: height.value,
    width: width.value,
  }));

  return {
    animatedLayoutStyle,
    expandCard,
    collapseCard,
  };
};

const LayoutAnimatedCard = () => {
  const { animatedLayoutStyle, expandCard, collapseCard } = useLayoutAnimations();

  return (
    <Animated.View style={[styles.layoutCard, animatedLayoutStyle]}>
      <Button title="Expand" onPress={expandCard} />
      <Button title="Collapse" onPress={collapseCard} />
    </Animated.View>
  );
};
```

---

## 🎯 Phase 10 — Security, Privacy & Best Practices

**Security questions show production readiness** - senior roles expect knowledge of secure storage, code obfuscation, and data privacy compliance.

### 10.1 **Secure Storage and Data Protection**

**Question:** *"Implement comprehensive security measures including encrypted storage, secure API communication, and protection against common vulnerabilities."*

**Perfect Answer Structure:**
```
SECURITY ARCHITECTURE:

🎯 SECURE STORAGE:
- Encrypted data persistence
- Key management strategies
- Biometric authentication
- Secure enclave usage

🎯 API SECURITY:
- Certificate pinning
- Token management
- Request/response encryption
- Man-in-the-middle protection

🎯 CODE PROTECTION:
- Code obfuscation
- Anti-tampering measures
- Runtime security checks
- Debug detection
```

**Complete Security Implementation:**
```javascript
// Secure Storage Manager
import * as SecureStore from 'expo-secure-store';
import * as Keychain from 'react-native-keychain';
import AsyncStorage from '@react-native-async-storage/async-storage';

class SecureStorageManager {
  private static instance: SecureStorageManager;

  static getInstance(): SecureStorageManager {
    if (!SecureStorageManager.instance) {
      SecureStorageManager.instance = new SecureStorageManager();
    }
    return SecureStorageManager.instance;
  }

  // High-security storage (encrypted, biometric)
  async setSecureItem(key: string, value: string): Promise<void> {
    try {
      if (Platform.OS === 'ios') {
        await Keychain.setGenericPassword(key, value, {
          service: 'com.myapp.secure',
          accessControl: Keychain.ACCESS_CONTROL.BIOMETRY_CURRENT_SET,
          accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
        });
      } else {
        await SecureStore.setItemAsync(key, value, {
          keychainService: 'com.myapp.secure',
          requireAuthentication: true,
        });
      }
    } catch (error) {
      console.error('Secure storage error:', error);
      throw new Error('Failed to store sensitive data');
    }
  }

  async getSecureItem(key: string): Promise<string | null> {
    try {
      if (Platform.OS === 'ios') {
        const credentials = await Keychain.getGenericPassword({
          service: 'com.myapp.secure',
        });
        return credentials ? credentials.password : null;
      } else {
        return await SecureStore.getItemAsync(key, {
          keychainService: 'com.myapp.secure',
        });
      }
    } catch (error) {

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 12 ➡️](phase12-portfolio-defense.md)