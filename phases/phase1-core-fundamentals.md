[⬅️ Back to Home](../README.md)

# 🎯 Phase 01: Core React Native Fundamentals

<div align="center">
  <img src="https://img.shields.io/badge/Level-Senior-red?style=for-the-badge" alt="Senior Level" />
  <img src="https://img.shields.io/badge/Status-Complete-green?style=for-the-badge" alt="Status Complete" />
</div>

---

## 📋 Table of Contents
1. [Architecture: Old vs New](#11-react-native-architecture-old-vs-new-architecture)
2. [JSI Deep Dive](#12-javascript-interface-jsi-deep-dive)
3. [TurboModules vs Legacy](#13-turbomodules-vs-legacy-native-modules)
4. [Fabric Renderer](#14-fabric-renderer-architecture)
5. [Metro Bundler Optimization](#15-metro-bundler-optimization)
6. [Hermes Engine vs JSC](#16-hermes-engine-vs-jsc)
7. [Threading Model](#17-react-native-threading-model)
8. [Codegen & TypeScript](#18-codegen-and-typescript-integration)
9. [Bridge Communication](#19-react-native-bridge-communication-patterns)
10. [Rendering Pipeline](#110-react-native-s-rendering-pipeline)
11. [RAM Bundles](#111-ram-bundles-vs-regular-bundles)
12. [Module Resolution](#112-react-native-s-module-resolution-strategy)
13. [Migration Strategy](#113-new-architecture-migration-strategy)
14. [Build Systems](#114-react-native-s-build-system-cmake-vs-gradle)
15. [Reloading Mechanisms](#115-react-native-s-hot-reloading-vs-live-reloading)

---

> [!IMPORTANT]
> **Senior interviewers LOVE these questions** - they separate juniors from professionals. You must be able to explain how React Native works internally, not just how to use it.

---

### 1.1 React Native Architecture: Old vs New Architecture

**Question:** *"Explain React Native's architecture evolution. Why did they move from the Bridge to JSI?"*

#### 💡 The Core Concept
React Native has evolved from a message-passing architecture to a direct-reference architecture.

| Feature | Old Architecture (Bridge) | New Architecture (JSI) |
| :--- | :--- | :--- |
| **Communication** | Asynchronous JSON Messages | Direct C++ Object References |
| **Serialization** | Required (Expensive) | Zero-copy (Direct Access) |
| **Threading** | JS Thread ↔ Bridge ↔ Native | Shared Memory Access |
| **Performance** | Bottlenecked by JSON overhead | 2-10x faster native calls |

#### 🚀 Senior Insight: Why the change?
The Bridge was the single biggest performance bottleneck. Every interaction (scroll, touch, data fetch) had to be serialized into a JSON string, sent over the bridge, and deserialized on the other side. JSI (JavaScript Interface) allows JS to hold a reference to a C++ object, making native calls synchronous and incredibly fast.

#### 🪤 Follow-up Trap
> *"When would you still use the old Bridge architecture?"*
>
> **Answer:** You shouldn't for new projects. However, legacy applications might stay on the Bridge due to the complexity of migrating custom native modules that aren't yet compatible with TurboModules or Fabric.

#### 💻 Real-World Comparison
```javascript
// OLD BRIDGE: Expensive JSON serialization
NativeModules.MyModule.getData().then(result => console.log(result));

// NEW ARCHITECTURE: Direct C++ calls via JSI
const myModule = TurboModuleRegistry.get('MyTurboModule');
const result = myModule.getData(); // Direct synchronous native call
```

---

### 1.2 JavaScript Interface (JSI) Deep Dive

**Question:** *"How does JSI enable direct communication between JavaScript and native code?"*

#### 🛠️ Technical Implementation
JSI is a lightweight C++ API that allows the JavaScript engine to talk directly to the native host environment.

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

#### ✅ Key Advantages
- **Synchronous Execution**: You can call native functions and get the result immediately, just like a standard JS function.
- **Shared Memory**: JS and Native share the same memory space for specific objects.
- **Engine Agnostic**: JSI isn't tied to a specific JS engine (like Hermes or V8).

---

### 1.3 TurboModules vs Legacy Native Modules

**Question:** *"Compare TurboModules with legacy native modules. When should you use each?"*

#### 🚀 TurboModules (New)
- **Lazy Loading**: Native modules are only loaded when first used, improving app startup time.
- **Type Safety**: Strongly typed interfaces via Codegen.
- **Performance**: Direct JSI calls instead of Bridge messages.

#### 📦 Legacy Modules (Old)
- **Eager Loading**: All modules are loaded at startup, increasing memory pressure and startup time.
- **Promise-based**: Always asynchronous communication.

#### 💻 Code Comparison
```javascript
// LEGACY MODULE: Async + JSON overhead
const result = await NativeModules.ImageProcessor.resizeImage(uri, 300);

// TURBOMODULE: Sync call via JSI
const imageProcessor = TurboModuleRegistry.get('ImageProcessor');
const result = imageProcessor.resizeImageSync(uri, 300); // Synchronous!
```

---

### 1.4 Fabric Renderer Architecture

**Question:** *"Explain how Fabric Renderer improves React Native's rendering performance."*

#### 🎨 The Three Phases of Fabric
1. **Render Phase**: React creates the virtual DOM.
2. **Commit Phase**: Fabric creates the Shadow Tree (layout calculation).
3. **Mount Phase**: Shadow Tree is converted into platform-specific native views.

#### ✨ Key Improvements
- **Multithreading**: Layout calculations can happen on background threads without blocking the JS thread.
- **Synchronous Updates**: Essential for things like `TextInput` where the UI must stay in sync with the user's typing immediately.
- **Improved Priority**: High-priority updates (like animations) can interrupt lower-priority work.

---

### 1.5 Metro Bundler Optimization

**Question:** *"How does Metro bundler optimize React Native apps for production?"*

#### 🔧 Core Optimization Techniques
- **Tree Shaking**: Removing unused code from the final bundle.
- **Inline Requires**: Delaying the execution of modules until they are needed.
- **RAM Bundles**: Splitting the bundle into multiple files that are loaded on demand.

```javascript
// metro.config.js
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true, // Crucial for performance!
      },
    }),
  },
};
```

---

### 1.6 Hermes Engine vs JSC

**Question:** *"Compare Hermes and JavaScriptCore engines. When should you choose each?"*

#### 🟢 Hermes (Facebook's Optimized Engine)
- **Bytecode Pre-compilation**: Compiles JS to bytecode during the build process, not at runtime.
- **Memory Efficient**: Optimized for mobile devices with limited RAM.
- **Faster TTI**: Significantly reduces "Time to Interactive."

#### 🔴 JavaScriptCore (iOS Default)
- **JIT Compilation**: Compiles code as it runs, which can lead to spikes in CPU usage.
- **Legacy Support**: Better compatibility with some very old JS features, but generally less efficient for RN.

---

### 1.7 React Native Threading Model

**Question:** *"Explain React Native's threading architecture and how it affects performance."*

#### 🧵 The Four Main Threads
1. **Main Thread (UI)**: Handles native UI rendering and user interactions.
2. **JS Thread**: Where your business logic and React components live.
3. **Shadow Thread**: Handles layout calculations (Yoga engine).
4. **Native Modules Thread**: Used by native modules for background tasks (e.g., File I/O).

#### ⚠️ Critical Rule
**NEVER block the Main Thread.** If the main thread is blocked, the UI freezes, leading to a poor user experience.

---

### 1.8 Codegen and TypeScript Integration

**Question:** *"How does React Native's Codegen work with TypeScript for type safety?"*

#### 🔧 The Workflow
1. You define a TypeScript interface for your native module.
2. Codegen parses the interface and generates C++ / Java / Objective-C glue code.
3. This ensures that the data passed between JS and Native is always valid and type-safe.

---

### 1.9 React Native Bridge Communication Patterns

**Question:** *"Explain the different communication patterns in React Native's Bridge vs JSI."*

#### 🔄 Communication Flow
- **Bridge**: Asynchronous, serializable, batched.
- **JSI**: Synchronous, direct references, zero-copy.

---

### 1.10 React Native's Rendering Pipeline

**Question:** *"Walk through React Native's complete rendering pipeline from JSX to pixels."*

#### 🏗️ The Pipeline
1. **JSX** → 2. **React Elements** → 3. **Shadow Tree (Yoga)** → 4. **Native Views** → 5. **Pixels (GPU)**

---

### 1.11 RAM Bundles vs Regular Bundles

**Question:** *"Explain RAM bundles and when you should use them instead of regular bundles."*

#### 🧠 RAM (Random Access Modules)
- Loads only the necessary modules at startup.
- Great for large apps to reduce initial memory usage.
- **Downside**: Slightly slower module access after startup.

---

### 1.12 React Native's Module Resolution Strategy

**Question:** *"How does React Native resolve modules differently from web bundlers?"*

#### 📱 Platform-Specific Extensions
React Native looks for files in this order:
1. `Component.ios.js` or `Component.android.js`
2. `Component.native.js`
3. `Component.js`

---

### 1.13 New Architecture Migration Strategy

**Question:** *"Design a migration strategy for upgrading a large React Native app to the New Architecture."*

#### 📅 Phased Approach
1. **Audit**: Identify all custom native modules.
2. **Preparation**: Upgrade to the latest RN version (0.70+ recommended).
3. **TurboModules**: Migrate core native modules first.
4. **Fabric**: Enable the new renderer and test UI consistency.

---

### 1.14 React Native's Build System (CMake vs Gradle)

**Question:** *"Explain how React Native integrates with platform-specific build systems."*

- **Android**: Uses Gradle and CMake for C++ code.
- **iOS**: Uses Xcode and CocoaPods.

---

### 1.15 React Native's Hot Reloading vs Live Reloading

**Question:** *"Compare Hot Reloading, Live Reloading, and Fast Refresh in React Native development."*

- **Fast Refresh**: The modern standard. It's smart enough to preserve state when possible and perform a full reload when necessary.

---

### 🚀 Next Steps
*Phase 1 has covered the engine room of React Native. In Phase 2, we move to the UI layer and React Hooks mastery.*

---

[⬅️ Back to Home](../README.md) | [Next Phase: Phase 02 ➡️](phase2-react-fundamentals.md)
