[⬅️ Back to Home](../README.md)

# 🎯 Phase 01: Core React Native Fundamentals

---


**Senior interviewers LOVE these questions** - they separate juniors from professionals. You must explain how React Native works internally.

### 1.1 **React Native Architecture: Old vs New Architecture**

**Question:** *"Explain React Native's architecture evolution. Why did they move from the Bridge to JSI?"*

**Perfect Answer Structure:**
```
React Native has evolved through two major architectures:

🔴 OLD ARCHITECTURE (Pre-0.60):
- Uses JavaScript Bridge for communication
- Asynchronous JSON message passing
- Threading: JS Thread ↔ Bridge ↔ Native Threads
- Performance bottleneck due to serialization overhead

🟢 NEW ARCHITECTURE (0.60+):
- JavaScript Interface (JSI) replaces Bridge
- Direct C++ object references between JS and Native
- TurboModules for efficient native module calls
- Fabric Renderer for improved rendering performance
```

**Why the change?**
- Bridge serialized everything to JSON → huge overhead
- JSI allows direct object references → zero-copy communication
- 2-10x performance improvement for native module calls

**Follow-up Trap:** *"When would you still use the old Bridge architecture?"*
**Answer:** Only for legacy apps. New apps should use New Architecture. Migration is complex but worth it for performance.

**Real-World Example:**
```javascript
// OLD BRIDGE: Expensive JSON serialization
NativeModules.MyModule.getData().then(result => console.log(result));

// NEW ARCHITECTURE: Direct C++ calls via JSI
const myModule = TurboModuleRegistry.get('MyTurboModule');
const result = myModule.getData(); // Direct native call
```

---

### 1.2 **JavaScript Interface (JSI) Deep Dive**

**Question:** *"How does JSI enable direct communication between JavaScript and native code?"*

**Perfect Answer Structure:**
```
JSI is a lightweight C++ API that:

✅ Creates persistent JavaScript objects on native side
✅ Eliminates JSON serialization overhead
✅ Enables synchronous native method calls
✅ Supports object references across the JS-Native boundary
✅ Powers TurboModules and Fabric Renderer
```

**Technical Implementation:**
```cpp
// JSI HostObject allows JS to hold native C++ objects
class MyTurboModule : public jsi::HostObject {
  jsi::Value get(jsi::Runtime& runtime, const jsi::PropNameID& name) override {
    if (name.utf8(runtime) == "getData") {
      return jsi::Function::createFromHostFunction(/*...*/);
    }
    return jsi::Value::undefined();
  }
};
```

**Follow-up Trap:** *"What's the biggest limitation of JSI?"*
**Answer:** JSI objects can't survive app reloads in development. They need proper cleanup.

---

### 1.3 **TurboModules vs Legacy Native Modules**

**Question:** *"Compare TurboModules with legacy native modules. When should you use each?"*

**Perfect Answer Structure:**
```
🚀 TURBOMODULES (New Architecture):
- Synchronous native calls possible
- JSI-powered, no JSON serialization
- Lazy loading reduces bundle size
- Better performance for frequent native operations

📦 LEGACY NATIVE MODULES (Old Architecture):
- Asynchronous only (Promise-based)
- JSON serialization overhead
- Eager loading increases bundle size
- Better for one-off operations

Migration Strategy:
1. New features → TurboModules
2. Existing modules → Gradually migrate
3. Performance-critical code → Priority migration
```

**Code Comparison:**
```javascript
// LEGACY MODULE: Async + JSON overhead
const result = await NativeModules.ImageProcessor.resizeImage(uri, 300);

// TURBOMODULE: Sync call via JSI
const imageProcessor = TurboModuleRegistry.get('ImageProcessor');
const result = imageProcessor.resizeImageSync(uri, 300); // Synchronous!
```

**Follow-up Trap:** *"Can TurboModules be used in Expo apps?"*
**Answer:** No, Expo uses managed workflow. TurboModules require bare workflow for native code integration.

---

### 1.4 **Fabric Renderer Architecture**

**Question:** *"Explain how Fabric Renderer improves React Native's rendering performance."*

**Perfect Answer Structure:**
```
Fabric is React Native's new rendering system:

🎨 THREE PHASES:
1. Render Phase: React creates virtual DOM
2. Commit Phase: Fabric creates Shadow Tree
3. Mount Phase: Shadow Tree → Native views

✨ KEY IMPROVEMENTS:
- Multithreading: Render on background thread
- Async rendering: No blocking of JS thread
- Better layout: More efficient Flexbox calculations
- Memory efficient: Reduced object allocations
```

**Performance Impact:**
- 50-70% faster initial renders
- Smoother animations (60fps consistent)
- Reduced memory pressure
- Better responsiveness during heavy computations

**Follow-up Trap:** *"Does Fabric change how you write React components?"*
**Answer:** No, it's backward compatible. Your JSX and component logic remain the same.

---

### 1.5 **Metro Bundler Optimization**

**Question:** *"How does Metro bundler optimize React Native apps for production?"*

**Perfect Answer Structure:**
```
Metro handles module resolution and bundling:

🔧 CORE FEATURES:
- Module resolution with platform extensions (.ios.js, .android.js)
- Tree shaking removes unused code
- Code splitting for dynamic imports
- Minification and compression
- Source map generation for debugging

📦 OPTIMIZATION TECHNIQUES:
- Dead code elimination
- Asset optimization (images, fonts)
- Polyfill injection based on platform
- RAM bundle for faster startup
```

**Advanced Configuration:**
```javascript
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
  resolver: {
    alias: {
      '@components': './src/components',
    },
  },
};
```

**Follow-up Trap:** *"What's the difference between Metro and Webpack?"*
**Answer:** Metro is RN-specific, optimized for mobile bundling. Webpack is web-focused. Metro handles platform-specific resolution and RAM bundles.

---

### 1.6 **Hermes Engine vs JSC**

**Question:** *"Compare Hermes and JavaScriptCore engines. When should you choose each?"*

**Perfect Answer Structure:**
```
🟢 HERMES (Recommended for production):
- Facebook's JS engine optimized for RN
- Pre-compiles JS to bytecode → faster startup
- Smaller binary size
- Better memory management
- Ahead-of-time compilation

🔴 JAVASCRIPTCORE (Default on iOS):
- Apple's Nitro engine on iOS
- Just-in-time compilation
- Larger app size
- More memory usage
- Better debugging experience
```

**Performance Metrics:**
```
Startup Time: Hermes 2-3x faster
Memory Usage: Hermes 20-30% less
App Size: Hermes 5-10% smaller
```

**Follow-up Trap:** *"Can you use Hermes in development?"*
**Answer:** Yes, but debugging is harder. Use JSC in dev for better debugging, Hermes in production for performance.

---

### 1.7 **React Native Threading Model**

**Question:** *"Explain React Native's threading architecture and how it affects performance."*

**Perfect Answer Structure:**
```
React Native uses multiple threads:

🧵 MAIN THREAD (UI Thread):
- Handles native UI rendering
- Most expensive operations
- Never block this thread!

🧵 JS THREAD:
- Runs your JavaScript code
- Handles business logic
- Can be blocked by heavy computations

🧵 SHADOW THREAD (Fabric):
- Calculates layouts asynchronously
- Runs Flexbox algorithms
- Prepares native view updates

🧵 NATIVE MODULES THREAD:
- Handles native API calls
- File system, network requests
- Device hardware access
```

**Performance Rules:**
```javascript
// ❌ BLOCKS JS THREAD
const data = heavyComputation(); // 2 seconds
setState({ data });

// ✅ OFFLOADS TO NATIVE THREAD
InteractionManager.runAfterInteractions(() => {
  const data = heavyComputation();
  setState({ data });
});
```

**Follow-up Trap:** *"What happens if you block the main thread?"*

**Answer:** App becomes unresponsive, user sees frozen UI, possible ANR (Android) or watchdog kills (iOS).

---

### 1.8 **Codegen and TypeScript Integration**

**Question:** *"How does React Native's Codegen work with TypeScript for type safety?"*

**Perfect Answer Structure:**
```
Codegen generates type-safe native interfaces:

🔧 PROCESS:
1. TypeScript interfaces → Codegen analysis
2. Generates native type definitions
3. Creates TurboModule/Fabric bindings
4. Ensures JS-Native type safety

📝 TYPE SAFETY LEVELS:
- Flow types (legacy)
- TypeScript strict mode
- Native type validation
- Runtime type checking
```

**Implementation:**
```typescript
// TurboModule interface
export interface Spec extends TurboModule {
  getUser(id: string): Promise<User>;
  saveData(data: UserData): boolean;
}

// Codegen generates:
- Android/Kotlin interfaces
- iOS/Swift protocols
- JSI bindings
- Type validation
```

**Follow-up Trap:** *"Does Codegen replace manual native module creation?"*

**Answer:** No, Codegen assists but you still write native implementations. It generates boilerplate and ensures type safety.

---

### 1.9 **React Native Bridge Communication Patterns**

**Question:** *"Explain the different communication patterns in React Native's Bridge vs JSI."*

**Perfect Answer Structure:**
```
BRIDGE COMMUNICATION (Legacy):
- Asynchronous JSON-RPC calls
- Message queue system
- Batch updates for efficiency
- Error-prone serialization

JSI COMMUNICATION (Modern):
- Synchronous method calls
- Direct object references
- Host objects and functions
- Zero-copy data transfer

COMMUNICATION PATTERNS:
1. Method Calls: JS → Native
2. Callbacks: Native → JS
3. Events: Native → JS (continuous)
4. Promises: Async results
```

**Performance Comparison:**
```javascript
// BRIDGE: Expensive serialization
NativeModules.APIModule.fetchData()
  .then(JSON.parse) // Deserialize
  .then(processData);

// JSI: Direct access
const apiModule = TurboModuleRegistry.get('APIModule');
const data = apiModule.fetchDataSync(); // Direct access
```

---

### 1.10 **React Native's Rendering Pipeline**

**Question:** *"Walk through React Native's complete rendering pipeline from JSX to pixels."*

**Perfect Answer Structure:**
```
COMPLETE RENDERING PIPELINE:

1️⃣ JSX → React Elements
   - JSX compiles to React.createElement calls
   - Creates virtual DOM representation

2️⃣ Reconciliation (React)
   - Diff algorithm compares old/new trees
   - Determines what changed

3️⃣ Commit Phase (Fabric)
   - Creates/updates Shadow Tree
   - Calculates layouts asynchronously

4️⃣ Mount Phase
   - Shadow Tree → Native view commands
   - Updates sent to main thread

5️⃣ Native Rendering
   - Platform-specific view updates
   - GPU acceleration for animations
   - Final pixel rendering
```

**Performance Bottlenecks:**
- Step 2: Expensive for large component trees
- Step 3: Complex layouts block shadow thread
- Step 4: Bridge congestion in old architecture

**Optimization Strategies:**
```javascript
// Memoize expensive computations
const memoizedValue = useMemo(() => expensiveCalc(props), [deps]);

// Prevent unnecessary re-renders
const MyComponent = React.memo(({ data }) => <Text>{data}</Text>);
```

---

### 1.11 **RAM Bundles vs Regular Bundles**

**Question:** *"Explain RAM bundles and when you should use them instead of regular bundles."*

**Perfect Answer Structure:**
```
RAM BUNDLES optimize memory usage:

📦 REGULAR BUNDLE:
- Single large JavaScript file
- Loaded entirely into memory
- Simple but memory-intensive

🧠 RAM BUNDLE:
- Pre-executed JavaScript bytecode
- Modules loaded on-demand
- Significantly reduced memory footprint
- Faster app startup

PERFORMANCE IMPACT:
- 30-50% reduction in memory usage
- Faster cold starts
- Smaller initial memory footprint
```

**When to Use:**
```javascript
// Use RAM bundles for:
- Large apps (>20MB JS)
- Memory-constrained devices
- Apps with many features
- Production deployments

// metro.config.js configuration
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: false,
      },
    }),
  },
};
```

**Follow-up Trap:** *"What's the downside of RAM bundles?"*

**Answer:** More complex bundling process, harder to debug, requires Metro configuration. Use only when memory optimization is critical.

---

### 1.12 **React Native's Module Resolution Strategy**

**Question:** *"How does React Native resolve modules differently from web bundlers?"*

**Perfect Answer Structure:**
```
React Native's module resolution prioritizes platform-specific code:

📱 PLATFORM-SPECIFIC RESOLUTION:
1. index.ios.js / index.android.js
2. index.native.js
3. index.js
4. Fallback to web versions

🔍 HASTE MODULE SYSTEM:
- Global module registry
- @providesModule declarations
- Deterministic module IDs
- Enables fast native loading

📂 FILE EXTENSIONS PRIORITY:
- .native.js (platform-agnostic native)
- .ios.js / .android.js (platform-specific)
- .js (universal fallback)
```

**Resolution Example:**
```javascript
// File structure:
// components/Button.ios.js
// components/Button.android.js
// components/Button.js

import Button from './components/Button';
// iOS → Button.ios.js
// Android → Button.android.js
```

**Advanced Configuration:**
```javascript
// metro.config.js
module.exports = {
  resolver: {
    platforms: ['ios', 'android', 'web'],
    extensions: ['.ios.js', '.android.js', '.js'],
  },
};
```

---

### 1.13 **New Architecture Migration Strategy**

**Question:** *"Design a migration strategy for upgrading a large React Native app to the New Architecture."*

**Perfect Answer Structure:**
```
PHASED MIGRATION APPROACH:

📅 PHASE 1: Preparation (1-2 weeks)
- Audit existing native modules
- Identify performance bottlenecks
- Set up New Architecture in parallel
- Create migration testing strategy

📅 PHASE 2: TurboModules Migration (2-4 weeks)
- Convert high-impact native modules first
- Performance-critical modules priority
- Maintain backward compatibility
- Gradual rollout with feature flags

📅 PHASE 3: Fabric Migration (1-2 weeks)
- Enable Fabric renderer
- Test component compatibility
- Monitor rendering performance
- Rollback plan ready

📅 PHASE 4: Optimization (1-2 weeks)
- Remove legacy Bridge code
- Optimize bundle size
- Performance monitoring
- Production deployment
```

**Risk Mitigation:**
```javascript
// Feature flags for gradual rollout
const useNewArchitecture = __DEV__ ? false : true;

if (useNewArchitecture) {
  // New Architecture code path
  const turboModule = TurboModuleRegistry.get('MyModule');
} else {
  // Legacy Bridge fallback
  const result = await NativeModules.MyModule.getData();
}
```

---

### 1.14 **React Native's Build System (CMake vs Gradle)**

**Question:** *"Explain how React Native integrates with platform-specific build systems."*

**Perfect Answer Structure:**
```
React Native bridges platform build systems:

🤖 ANDROID INTEGRATION:
- Gradle build system
- CMake for C++ native modules
- Auto-linking via react-native.config.js
- Gradle plugin manages dependencies

🍎 IOS INTEGRATION:
- Xcode build system
- CocoaPods for dependencies
- Auto-linking for Swift/Objective-C
- Xcode project generation

🔧 BUILD PROCESS:
1. JavaScript bundling (Metro)
2. Native code compilation
3. Asset processing
4. Platform-specific packaging
5. Code signing and distribution
```

**Configuration Example:**
```javascript
// react-native.config.js
module.exports = {
  dependencies: {
    'react-native-vector-icons': {
      platforms: {
        ios: null, // disable iOS
        android: {
          sourceDir: './android',
        },
      },
    },
  },
};
```

---

### 1.15 **React Native's Hot Reloading vs Live Reloading**

**Question:** *"Compare Hot Reloading, Live Reloading, and Fast Refresh in React Native development."*

**Perfect Answer Structure:**
```
THREE RELOADING MECHANISMS:

🔥 FAST REFRESH (Recommended):
- Preserves component state
- Updates only changed components
- Maintains navigation state
- Works with Hooks and Context

📱 LIVE RELOADING:
- Full app reload on file change
- Preserves app state but slower
- Good for structural changes
- Legacy, being phased out

♨️ HOT RELOADING:
- Injects updated code without restart
- Fastest for UI changes
- May lose state occasionally
- Limited to styling changes
```

**When to Use Each:**
```javascript
// Fast Refresh handles these automatically:
// - Component logic changes
// - Hook updates
// - Style changes
// - Asset imports

// Force full reload for:
- Navigation structure changes
- State management setup
- Native module modifications
```

**Performance Comparison:**
- Fast Refresh: ~100-500ms
- Live Reloading: ~2-5 seconds
- Hot Reloading: ~1-2 seconds

---

*Phase 1 continues with 15+ more questions covering advanced topics like concurrent features, experimental APIs, and production optimization strategies. Each question designed for senior-level interviews with real-world implementation details.*

---

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 2 ➡️](phase2-react-fundamentals.md)