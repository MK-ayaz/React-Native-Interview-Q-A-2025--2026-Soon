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
    background: linear-gradient(135deg, #0f172a 0%, #334155 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(15, 23, 42, 0.2);
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
    background: #f1f5f9;
    color: #475569;
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    display: inline-block;
    margin-bottom: 16px;
  }
  .senior-insight {
    background: #f0f9ff;
    border-left: 4px solid #0ea5e9;
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
  <span class="phase-number">Phase 05</span>
  <h1 class="phase-title">Native Modules & APIs</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Bridging the gap between JavaScript and the underlying OS.</p>
</div>

---

### 📋 Phase Overview
A true Senior React Native Engineer knows when the JavaScript world ends and the Native world begins. This phase focuses on TurboModules, C++ integration via JSI, and high-performance hardware interaction.

---

<div class="qa-card">
  <span class="question-tag">NEW ARCHITECTURE</span>
  <h3>5.1 TurboModules vs. Legacy Bridge</h3>
  
  **Question:** *"Why were TurboModules introduced, and how do they solve the startup performance issues of the legacy Bridge?"*

  #### 🚀 Lazy Loading & Initialization
  In the legacy architecture, all native modules are initialized during app startup, even if they aren't used. TurboModules are **lazy-loaded**, meaning they are only initialized the first time they are called from JS, significantly reducing TTI.

  <div class="senior-insight">
    <strong>Senior Insight: The JSI Connection</strong><br/>
    TurboModules use JSI to hold a direct reference to native functions, eliminating the overhead of JSON serialization and the asynchronous nature of the old Bridge.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">C++ INTEROP</span>
  <h3>5.2 JSI & Synchronous Execution</h3>

  **Question:** *"Explain how JSI allows for synchronous communication between JS and C++. What are the risks?"*

  #### ⚡ Direct Function Invocation
  JSI (JavaScript Interface) allows JS to directly call C++ functions without passing through a "bridge." This is essential for high-frequency data (like camera frames or sensors).

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What happens if you run a blocking network call synchronously via JSI?"</em><br/>
    <strong>Answer:</strong> It will freeze the JS thread entirely, making the app unresponsive until the network call returns. JSI sync calls should only be used for fast computations.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">CODEGEN</span>
  <h3>5.3 The Role of Codegen</h3>

  **Question:** *"Why is Codegen mandatory in the New Architecture, and what does it actually generate?"*

  #### 🛡️ Type-Safe Contracts
  Codegen takes your TypeScript/Flow spec and generates C++ "glue code." This ensures that both JS and Native sides adhere to the same contract, preventing runtime crashes due to mismatched data types.

  <div class="code-block-header">
    <span>NativeMyModule.ts</span>
    <span>TypeScript</span>
  </div>
  ```typescript
  export interface Spec extends TurboModule {
    getDeviceSecret(): Promise<string>;
    setVolume(value: number): void;
  }
  ```

  <div class="senior-insight">
    <strong>Senior Insight: C++ Mapping</strong><br/>
    Codegen maps JS types to C++ types (e.g., <code>number</code> to <code>double</code>, <code>string</code> to <code>jsi::String</code>). Understanding this mapping is key to debugging native crashes.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">BACKGROUND</span>
  <h3>5.4 Background Tasks: Android vs. iOS</h3>

  **Question:** *"How do you implement a background sync task that works reliably on both Android and iOS?"*

  #### 🌑 Platform Differences
  - **Android**: Use **Headless JS** or **WorkManager**. Android allows for long-running background services.
  - **iOS**: Highly restricted. Use **Background Tasks** (BGTaskScheduler). You typically get a maximum of 30 seconds of execution time.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How do you handle tasks if the app is killed?"</em><br/>
    <strong>Answer:</strong> On Android, Headless JS can still run. On iOS, you are limited to system-triggered background fetches.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">PERMISSIONS</span>
  <h3>5.5 Granular Media Permissions</h3>

  **Question:** *"How do you handle Android 13's 'Read Images' vs. 'Read Video' permissions compared to iOS's 'Limited' access?"*

  #### 🛡️ Permission Architecture
  Modern OSs require granular checks. You can't just ask for "Storage" anymore. You must check for specific media types and handle "Partial" access (iOS) where the user only allows access to specific photos.

  <div class="senior-insight">
    <strong>Senior Insight: Re-checking Permissions</strong><br/>
    Always re-check permissions when the app moves from <code>background</code> to <code>active</code>, as the user might have revoked them in the system settings while the app was suspended.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">NATIVE UI</span>
  <h3>5.6 Custom Native UI Components</h3>

  **Question:** *"Walk through the creation of a custom Native UI component. How do JS and Native communicate props?"*

  #### 🎨 View Managers
  - **Android**: Extend `SimpleViewManager` and use `@ReactProp` annotations.
  - **iOS**: Extend `RCTViewManager` and use `RCT_EXPORT_VIEW_PROPERTY`.
  Communication is handled via a command system (Native -> JS) and prop updates (JS -> Native).

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How do you send events from the Native View back to JS?"</em><br/>
    <strong>Answer:</strong> Use <code>RCTDirectEventBlock</code> (iOS) or <code>RCTEventEmitter</code> (Android) to bubble up events like <code>onPress</code>.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">SECURITY</span>
  <h3>5.7 Biometrics & Keychain Security</h3>

  **Question:** *"How do you securely store a JWT after FaceID authentication?"*

  #### 🔑 Secure Hardware
  Never store sensitive data in `AsyncStorage`. Use the **iOS Keychain** and **Android Keystore**. Biometric libraries should return a success callback, after which you retrieve the encrypted key from secure storage.

  <div class="senior-insight">
    <strong>Senior Insight: Invalidation</strong><br/>
    Configure your Keystore to invalidate the key if a new biometric (fingerprint/face) is added to the device for maximum security.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">NETWORKING</span>
  <h3>5.8 SSL Pinning & MITM Protection</h3>

  **Question:** *"What is SSL Pinning, and how is it implemented in a React Native app?"*

  #### 🛡️ Traffic Security
  SSL Pinning ensures the app only trusts a specific public key or certificate. It must be implemented at the **Native Networking Layer** (using OkHttp on Android or TrustKit on iOS), as the JS `fetch` polyfill uses these native libraries under the hood.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What happens when the certificate expires?"</em><br/>
    <strong>Answer:</strong> The app will stop working. You must have a strategy for "Pin Rotation" or use a library that supports dynamic updates.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">FILES</span>
  <h3>5.9 Large File Handling (Streaming)</h3>

  **Question:** *"How do you upload a 100MB video file without crashing the app due to memory limits?"*

  #### 📤 Chunking & Streaming
  Avoid reading the entire file into JS memory as a Base64 string. Use a native module that supports **multipart uploads** or **chunking** (like `react-native-fs`). Pass only the file path to the native side.

  <div class="senior-insight">
    <strong>Senior Insight: Background Uploads</strong><br/>
    Use <code>react-native-background-upload</code> to ensure the upload continues even if the user closes the app or the OS suspends the process.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">THREADS</span>
  <h3>5.10 Native Threading Models</h3>

  **Question:** *"Which thread does your Native Module code run on by default? How do you update the UI from it?"*

  #### 🧵 The Native Module Thread
  By default, native methods run on a dedicated **Native Module Thread**. If you want to show an Alert, Toast, or update a Native View, you must explicitly dispatch to the **Main UI Thread**.

  <div class="code-block-header">
    <span>Android (Java)</span>
  </div>
  ```java
  activity.runOnUiThread(new Runnable() {
      public void run() {
          // UI Updates here
      }
  });
  ```

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What happens if you block the Main UI Thread?"</em><br/>
    <strong>Answer:</strong> The app freezes and eventually triggers an ANR (Application Not Responding) dialog.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">LIFECYCLE</span>
  <h3>5.11 AppState & Native Bridge</h3>

  **Question:** *"How does the AppState API interact with native lifecycle events?"*

  #### 🔄 State Synchronization
  The native side (AppDelegate/MainActivity) emits events when the app moves between `active`, `background`, and `inactive`. The JS side listens to these via `AppState`. This is critical for pausing timers, stopping camera sessions, or clearing sensitive data from memory.

  <div class="senior-insight">
    <strong>Senior Insight: iOS 'Inactive' State</strong><br/>
    On iOS, the <code>inactive</code> state occurs when the Control Center is pulled down or a phone call comes in. Handle this state to blur sensitive UI (like bank balances).
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">LOCATION</span>
  <h3>5.12 Geolocation: Battery vs. Accuracy</h3>

  **Question:** *"How do you implement background location tracking while minimizing battery drain?"*

  #### 📍 Balanced Accuracy
  - **Android**: Use the **Fused Location Provider**.
  - **iOS**: Use **Significant Location Changes** or **Region Monitoring (Geofencing)** instead of standard GPS updates.
  Only request high accuracy when the app is active or for critical tasks.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What permission is required for background location on Android 10+?"</em><br/>
    <strong>Answer:</strong> <code>ACCESS_BACKGROUND_LOCATION</code>, which must be requested separately and explained to the user via a clear disclosure.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">HARDWARE</span>
  <h3>5.13 Camera Resource Management</h3>

  **Question:** *"What are the most common causes of native memory leaks when using the camera?"*

  #### 📸 Session Cleanup
  Camera sessions are heavy native objects. Leaks occur when the JS component unmounts but the native camera session isn't explicitly stopped and released. Always use the `useEffect` cleanup return to call native `stop()` methods.

  <div class="senior-insight">
    <strong>Senior Insight: Multiple Cameras</strong><br/>
    Modern phones have 3+ cameras. Ensure your native logic handles session switching and resolution constraints correctly to avoid crashes on lower-end devices.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">NOTIFICATIONS</span>
  <h3>5.14 Push Notification Lifecycle</h3>

  **Question:** *"How do you handle a notification that arrives while the app is completely killed (Quit State)?"*

  #### 📬 Initial Notification
  When the app is killed, the notification is handled by the OS. When the user taps it, the app is launched. You must use `Messaging.getInitialNotification()` (FCM) or check the launch options in `AppDelegate` to retrieve the payload and navigate accordingly.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What are 'Silent Notifications'?"</em><br/>
    <strong>Answer:</strong> Notifications with no alert/sound that trigger a background task to sync data (e.g., content-available: 1 on iOS).
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">IDENTIFIERS</span>
  <h3>5.15 Push Token vs. Device ID</h3>

  **Question:** *"Why should you never use a hardware Device ID for user tracking? What is the alternative?"*

  #### 🆔 Privacy-Safe IDs
  Hardware IDs (IMEI/Serial) are restricted for privacy. If the user reinstalls the app, the ID might change or be blocked.
  **Alternative**: Use **IDFV** (Identifier for Vendor) on iOS or **Android ID**. For push, always use the **FCM/APNs Token**, but be prepared for it to refresh.

  <div class="senior-insight">
    <strong>Senior Insight: Token Rotation</strong><br/>
    Always implement <code>onTokenRefresh</code> listeners. If the server has an old token, push notifications will fail silently.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">INTEROP</span>
  <h3>5.16 External App Communication (Linking)</h3>

  **Question:** *"How do you handle 'Deep Linking' vs. 'Universal Links' (iOS) / 'App Links' (Android)?"*

  #### 🔗 Linking Strategies
  - **Deep Links**: Custom schemes (<code>myapp://</code>). Easy to setup but prone to conflicts.
  - **Universal/App Links**: Standard <code>https://</code> links. Require a server-side file (<code>apple-app-site-association</code> or <code>assetlinks.json</code>) to verify ownership. More secure and better UX.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How do you handle the link if the app is already open?"</em><br/>
    <strong>Answer:</strong> Use the <code>Linking.addEventListener('url', callback)</code> to catch incoming links while the app is in the background.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">SENSORS</span>
  <h3>5.17 High-Frequency JSI Streaming</h3>

  **Question:** *"How would you build a high-performance AR or fitness tracking feature that requires 60Hz sensor data?"*

  #### ⚡ JSI Streaming
  The old bridge would choke on 60 messages per second. A senior approach uses a **JSI-based Native Module** that writes sensor data directly to a **SharedValue** (Reanimated) or calls a JS callback directly from C++ to minimize overhead.

  <div class="senior-insight">
    <strong>Senior Insight: Frame Sync</strong><br/>
    Use <code>requestAnimationFrame</code> or Reanimated's <code>useFrameCallback</code> to sync your JS logic with the device's screen refresh rate.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">NETWORKING</span>
  <h3>5.18 Native Networking Internals</h3>

  **Question:** *"How does React Native's 'fetch' work under the hood? Why does it matter for debugging?"*

  #### 🌐 The Polyfill
  React Native polyfills the web `fetch` API. On the native side, it uses **OkHttp** (Android) and **NSURLSession** (iOS). This means native interceptors, proxy settings, and system-level VPNs automatically apply to your JS network calls.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Can you use Axios?"</em><br/>
    <strong>Answer:</strong> Yes, Axios also uses the same underlying native networking polyfills in a React Native environment.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">MEMORY</span>
  <h3>5.19 C++ JSI Object Lifecycle</h3>

  **Question:** *"How do you prevent memory leaks when passing C++ objects to JavaScript via JSI?"*

  #### 🧠 Host Objects
  JSI uses **HostObjects**. You must ensure that the C++ object's lifecycle is tied to the JS object's garbage collection. If the C++ object is deleted while JS still holds a reference, the app will crash with a segmentation fault.

  <div class="senior-insight">
    <strong>Senior Insight: Smart Pointers</strong><br/>
    Always use <code>std::shared_ptr</code> or <code>std::unique_ptr</code> in your C++ code to manage memory safely within the JSI environment.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">CODEGEN</span>
  <h3>5.20 TurboModule Codegen Types</h3>

  **Question:** *"What are the limitations of types you can pass through Codegen in a TurboModule spec?"*

  #### 🛑 Type Constraints
  Codegen doesn't support all TypeScript types. You are limited to:
  - Primitives (string, number, boolean)
  - Objects (as interfaces)
  - Arrays (of primitives or objects)
  - Promises and Callbacks
  Generic types or complex Union types are generally **not supported** and will fail the build.

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"How do you pass a complex Union?"</em><br/>
    <strong>Answer:</strong> Pass it as a string and parse/validate it on the native side, or use a numeric enum.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">HARDWARE</span>
  <h3>5.21 Low-Level Device Info Internals</h3>

  **Question:** *"How does react-native-device-info fetch 'Unique IDs' without violating OS privacy rules?"*

  #### 🔍 Privacy Compliance
  It uses **IDFV** on iOS (which is unique per developer) and **ANDROID_ID** on Android. It avoids hardware-bound IDs like IMEI or MAC addresses, which are now blocked by Apple and Google.

  <div class="senior-insight">
    <strong>Senior Insight: Privacy Manifests</strong><br/>
    In recent iOS versions, you must declare *why* you are accessing these IDs in a **Privacy Manifest** file, or your app will be rejected from the App Store.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ARCHITECTURE</span>
  <h3>5.22 The Bridge vs JSI: Memory-Level Deep Dive</h3>

  **Question:** *"Explain the fundamental architectural shift from the JSON Bridge to JSI. What happens at the memory level during communication?"*

  #### 🔄 Bridge: Asynchronous JSON Serialization
  The legacy **Bridge** relied on asynchronous JSON serialization where every JS-Native communication required:
  1. Data serialization to JSON string
  2. Queueing in a single-channel pipe
  3. Deserialization on the receiving side
  4. Callback execution

  This caused massive bottlenecks for high-frequency events like scrolling or animations.

  #### ⚡ JSI: Direct Memory References
  **JSI (JavaScript Interface)** is a lightweight C++ API that allows the JavaScript engine (Hermes/V8) to hold a **direct reference** to C++ Host Objects. No serialization overhead - JS can call native methods **synchronously**.

  <div class="code-block-header">
    <span>JSI HostObject Example</span>
    <span>C++</span>
  </div>
  ```cpp
  class MyTurboModule : public jsi::HostObject {
    jsi::Value get(jsi::Runtime& runtime, const jsi::PropNameID& name) override {
      if (name.utf8(runtime) == "getData") {
        return jsi::Function::createFromHostFunction(runtime,
          [](jsi::Runtime& rt, const jsi::Value& thisVal,
             const jsi::Value* args, size_t count) -> jsi::Value {
            // Direct native execution - no bridge!
            return jsi::String::createFromUtf8(rt, "native data");
          });
      }
      return jsi::Value::undefined();
    }
  };
  ```

  <div class="senior-insight">
    <strong>Senior Insight: Zero-Copy Data Transfer</strong><br/>
    With JSI, we achieve **zero-copy data transfer**. Instead of copying large arrays into JSON strings, the C++ layer provides direct memory pointers that the JS engine can read. This is why libraries like `react-native-skia` can render complex graphics at 60fps.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">RENDERING</span>
  <h3>5.23 Fabric Renderer: Threading & Priority</h3>

  **Question:** *"How does Fabric Renderer improve React Native's rendering performance and what are its three phases?"*

  #### 🎨 Fabric's Three-Phase Architecture
  Fabric is the new rendering system that brings thread safety and priority-based rendering:

  1. **Render Phase**: React creates the virtual DOM in JavaScript thread
  2. **Commit Phase**: Fabric creates the Shadow Tree (layout calculations) in C++ on a background thread
  3. **Mount Phase**: Shadow Tree converts to platform-specific native views on the Main Thread

  #### 🧵 Thread Separation Benefits
  - **Layout calculations** happen off the main thread (no UI jank)
  - **Priority-based updates** ensure critical UI updates (like TextInput) take precedence
  - **Concurrent rendering** allows multiple updates to be processed simultaneously

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"Can Fabric render everything asynchronously?"</em><br/>
    <strong>Answer:</strong> No! TextInput requires **Synchronous Interruptions** - the UI must update immediately as the user types, or it feels laggy. Fabric allows these targeted sync updates while keeping most rendering async.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ENGINE</span>
  <h3>5.24 Hermes vs JSC: TTI & Memory Trade-offs</h3>

  **Question:** *"Compare Hermes and JSC in terms of Time To Interactive and heap memory usage. When should you choose each?"*

  #### ⚡ Hermes: Ahead-of-Time Compilation
  - **Compilation**: AOT bytecode pre-compilation reduces startup time
  - **Memory**: Significantly lower heap memory usage
  - **Performance**: Faster TTI, optimized for mobile constraints
  - **Best for**: Most production React Native apps

  #### 🏃‍♂️ JSC: Just-in-Time Compilation
  - **Compilation**: JIT compilation can lead to CPU spikes
  - **Memory**: Higher memory footprint
  - **Performance**: May be faster for long-running math tasks
  - **Best for**: Apps with heavy computational requirements

  <div class="senior-insight">
    <strong>Senior Insight: The Memory-Time Trade-off</strong><br/>
    Hermes trades slightly slower execution speed for dramatically better startup performance and memory efficiency. For 99% of apps, this trade-off is worthwhile - users care more about fast app launch than micro-optimizations in number crunching.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">THREADING</span>
  <h3>5.25 React Native Threading Architecture</h3>

  **Question:** *"Explain React Native's threading model and how the four main threads interact during a typical user interaction."*

  #### 🧵 The Four Main Threads

  | Thread | Responsibility | Language |
  |--------|---------------|----------|
  | **Main Thread** | Native UI rendering, user interactions | Platform (Java/Obj-C) |
  | **JS Thread** | Business logic, React components | JavaScript |
  | **Shadow Thread** | Layout calculations (Yoga) | C++ |
  | **Native Modules** | Background tasks (File I/O, network) | Platform/C++ |

  #### 🔄 Interaction Flow Example
  1. User taps button → **Main Thread** receives touch event
  2. Event sent to **JS Thread** → React updates state
  3. **Shadow Thread** calculates new layout with Yoga
  4. **Main Thread** applies changes to native views

  <div class="follow-up-trap">
    <strong>Follow-up Trap:</strong> <em>"What happens if the JS Thread is blocked?"</em><br/>
    <strong>Answer:</strong> The entire app becomes unresponsive! No animations, no touch responses, no state updates. This is why heavy computations must be moved to native threads or web workers.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">BUNDLING</span>
  <h3>5.26 Metro Bundler: Tree Shaking & RAM Bundles</h3>

  **Question:** *"How does Metro bundler optimize React Native apps for production, and when should you use RAM bundles?"*

  #### 📦 Metro Optimizations
  - **Tree Shaking**: Removes unused code automatically
  - **Inline Requires**: Loads modules only when needed
  - **Minification**: Reduces bundle size
  - **Source Maps**: Maintains debuggability

  #### 🧠 RAM Bundles
  RAM (Random Access Modules) bundles load only essential modules at startup, lazy-loading others. Benefits:
  - **Faster startup** (smaller initial bundle)
  - **Lower memory pressure** at launch
  - **Progressive loading** of features

  <div class="code-block-header">
    <span>metro.config.js</span>
    <span>JavaScript</span>
  </div>
  ```javascript
  module.exports = {
    transformer: {
      getTransformOptions: async () => ({
        transform: {
          experimentalImportSupport: false,
          inlineRequires: true, // Critical for performance
        },
      }),
    },
  };
  ```

  <div class="senior-insight">
    <strong>Senior Insight: RAM Bundle Trade-offs</strong><br/>
    RAM bundles reduce startup time but increase complexity in code splitting. Use them for apps >20MB where the initial load time is critical. For smaller apps, regular bundles are simpler and often faster overall.
  </div>
</div>

<div class="nav-footer">
  <a href="phase4-performance-memory.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 04: Performance</a>
  <a href="phase6-state-management.md" style="color: #3b82f6; text-decoration: none; font-weight: 600;">Phase 06: State Management ➡️</a>
</div>

</div>
