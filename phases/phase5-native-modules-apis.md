# Phase 05: Native Modules & APIs
**Bridging the gap between JavaScript and the underlying OS.**

---

### 📋 Phase Overview
A true Senior React Native Engineer knows when the JavaScript world ends and the Native world begins. This phase focuses on TurboModules, C++ integration via JSI, and high-performance hardware interaction.

---

### Q1: [NEW ARCHITECTURE] TurboModules vs. Legacy Bridge
**Question:** Why were TurboModules introduced, and how do they solve the startup performance issues of the legacy Bridge?

#### 🚀 Lazy Loading & Initialization
In the legacy architecture, all native modules are initialized during app startup, even if they aren't used. TurboModules are **lazy-loaded**, meaning they are only initialized the first time they are called from JS, significantly reducing TTI.

> [!TIP]
> **Senior Insight: The JSI Connection**
> TurboModules use JSI to hold a direct reference to native functions, eliminating the overhead of JSON serialization and the asynchronous nature of the old Bridge.

---

### Q2: [C++ INTEROP] JSI & Synchronous Execution
**Question:** Explain how JSI allows for synchronous communication between JS and C++. What are the risks?

#### ⚡ Direct Function Invocation
JSI (JavaScript Interface) allows JS to directly call C++ functions without passing through a "bridge." This is essential for high-frequency data (like camera frames or sensors).

> [!WARNING]
> **Follow-up Trap: "What happens if you run a blocking network call synchronously via JSI?"**
> **Answer:** It will freeze the JS thread entirely, making the app unresponsive until the network call returns. JSI sync calls should only be used for fast computations.

---

### Q3: [CODEGEN] The Role of Codegen
**Question:** Why is Codegen mandatory in the New Architecture, and what does it actually generate?

#### 🛡️ Type-Safe Contracts
Codegen takes your TypeScript/Flow spec and generates C++ "glue code." This ensures that both JS and Native sides adhere to the same contract, preventing runtime crashes due to mismatched data types.

```typescript
// NativeMyModule.ts
export interface Spec extends TurboModule {
  getDeviceSecret(): Promise<string>;
  setVolume(value: number): void;
}
```

> [!TIP]
> **Senior Insight: C++ Mapping**
> Codegen maps JS types to C++ types (e.g., `number` to `double`, `string` to `jsi::String`). Understanding this mapping is key to debugging native crashes.

---

### Q4: [BACKGROUND] Background Tasks: Android vs. iOS
**Question:** How do you implement a background sync task that works reliably on both Android and iOS?

#### 🌑 Platform Differences
- **Android**: Use **Headless JS** or **WorkManager**. Android allows for long-running background services.
- **iOS**: Highly restricted. Use **Background Tasks** (BGTaskScheduler). You typically get a maximum of 30 seconds of execution time.

> [!WARNING]
> **Follow-up Trap: "How do you handle tasks if the app is killed?"**
> **Answer:** On Android, Headless JS can still run. On iOS, you are limited to system-triggered background fetches.

---

### Q5: [PERMISSIONS] Granular Media Permissions
**Question:** How do you handle Android 13's 'Read Images' vs. 'Read Video' permissions compared to iOS's 'Limited' access?

#### 🛡️ Permission Architecture
Modern OSs require granular checks. You can't just ask for "Storage" anymore. You must check for specific media types and handle "Partial" access (iOS) where the user only allows access to specific photos.

> [!TIP]
> **Senior Insight: Re-checking Permissions**
> Always re-check permissions when the app moves from `background` to `active`, as the user might have revoked them in the system settings while the app was suspended.

---

### Q6: [NATIVE UI] Custom Native UI Components
**Question:** Walk through the creation of a custom Native UI component. How do JS and Native communicate props?

#### 🎨 View Managers
- **Android**: Extend `SimpleViewManager` and use `@ReactProp` annotations.
- **iOS**: Extend `RCTViewManager` and use `RCT_EXPORT_VIEW_PROPERTY`.
Communication is handled via a command system (Native -> JS) and prop updates (JS -> Native).

> [!WARNING]
> **Follow-up Trap: "How do you send events from the Native View back to JS?"**
> **Answer:** Use `RCTDirectEventBlock` (iOS) or `RCTEventEmitter` (Android) to bubble up events like `onPress`.

---

### Q7: [SECURITY] Biometrics & Keychain Security
**Question:** How do you securely store a JWT after FaceID authentication?

#### 🔑 Secure Hardware
Never store sensitive data in `AsyncStorage`. Use the **iOS Keychain** and **Android Keystore**. Biometric libraries should return a success callback, after which you retrieve the encrypted key from secure storage.

> [!TIP]
> **Senior Insight: Invalidation**
> Configure your Keystore to invalidate the key if a new biometric (fingerprint/face) is added to the device for maximum security.

---

### Q8: [NETWORKING] SSL Pinning & MITM Protection
**Question:** What is SSL Pinning, and how is it implemented in a React Native app?

#### 🛡️ Traffic Security
SSL Pinning ensures the app only trusts a specific public key or certificate. It must be implemented at the **Native Networking Layer** (using OkHttp on Android or TrustKit on iOS), as the JS `fetch` polyfill uses these native libraries under the hood.

> [!WARNING]
> **Follow-up Trap: "What happens when the certificate expires?"**
> **Answer:** The app will stop working. You must have a strategy for "Pin Rotation" or use a library that supports dynamic updates.

---

### Q9: [FILES] Large File Handling (Streaming)
**Question:** How do you upload a 100MB video file without crashing the app due to memory limits?

#### 📤 Chunking & Streaming
Avoid reading the entire file into JS memory as a Base64 string. Use a native module that supports **multipart uploads** or **chunking** (like `react-native-fs`). Pass only the file path to the native side.

> [!TIP]
> **Senior Insight: Background Uploads**
> Use `react-native-background-upload` to ensure the upload continues even if the user closes the app or the OS suspends the process.

---

### Q10: [THREADS] Native Threading Models
**Question:** Which thread does your Native Module code run on by default? How do you update the UI from it?

#### 🧵 The Native Module Thread
By default, native methods run on a dedicated **Native Module Thread**. If you want to show an Alert, Toast, or update a Native View, you must explicitly dispatch to the **Main UI Thread**.

```java
// Android (Java)
activity.runOnUiThread(new Runnable() {
    public void run() {
        // UI Updates here
    }
});
```

> [!WARNING]
> **Follow-up Trap: "What happens if you block the Main UI Thread?"**
> **Answer:** The app freezes and eventually triggers an ANR (Application Not Responding) dialog.

---

### Q11: [LIFECYCLE] AppState & Native Bridge
**Question:** How does the AppState API interact with native lifecycle events?

#### 🔄 State Synchronization
The native side (AppDelegate/MainActivity) emits events when the app moves between `active`, `background`, and `inactive`. The JS side listens to these via `AppState`. This is critical for pausing timers, stopping camera sessions, or clearing sensitive data from memory.

> [!TIP]
> **Senior Insight: iOS 'Inactive' State**
> On iOS, the `inactive` state occurs when the Control Center is pulled down or a phone call comes in. Handle this state to blur sensitive UI (like bank balances).

---

### Q12: [LOCATION] Geolocation: Battery vs. Accuracy
**Question:** How do you implement background location tracking while minimizing battery drain?

#### 📍 Balanced Accuracy
- **Android**: Use the **Fused Location Provider**.
- **iOS**: Use **Significant Location Changes** or **Region Monitoring (Geofencing)** instead of standard GPS updates.
Only request high accuracy when the app is active or for critical tasks.

> [!WARNING]
> **Follow-up Trap: "What permission is required for background location on Android 10+?"**
> **Answer:** `ACCESS_BACKGROUND_LOCATION`, which must be requested separately and explained to the user via a clear disclosure.

---

### Q13: [HARDWARE] Camera Resource Management
**Question:** What are the most common causes of native memory leaks when using the camera?

#### 📸 Session Cleanup
Camera sessions are heavy native objects. Leaks occur when the JS component unmounts but the native camera session isn't explicitly stopped and released. Always use the `useEffect` cleanup return to call native `stop()` methods.

> [!TIP]
> **Senior Insight: Multiple Cameras**
> Modern phones have 3+ cameras. Ensure your native logic handles session switching and resolution constraints correctly to avoid crashes on lower-end devices.

---

### Q14: [NOTIFICATIONS] Push Notification Lifecycle
**Question:** How do you handle a notification that arrives while the app is completely killed (Quit State)?

#### 📬 Initial Notification
When the app is killed, the notification is handled by the OS. When the user taps it, the app is launched. You must use `Messaging.getInitialNotification()` (FCM) or check the launch options in `AppDelegate` to retrieve the payload and navigate accordingly.

> [!WARNING]
> **Follow-up Trap: "What are 'Silent Notifications'?"**
> **Answer:** Notifications with no alert/sound that trigger a background task to sync data (e.g., content-available: 1 on iOS).

---

### Q15: [IDENTIFIERS] Push Token vs. Device ID
**Question:** Why should you never use a hardware Device ID for user tracking? What is the alternative?

#### 🆔 Privacy-Safe IDs
Hardware IDs (IMEI/Serial) are restricted for privacy. If the user reinstalls the app, the ID might change or be blocked.
**Alternative**: Use **IDFV** (Identifier for Vendor) on iOS or **Android ID**. For push, always use the **FCM/APNs Token**, but be prepared for it to refresh.

> [!TIP]
> **Senior Insight: Token Rotation**
> Always implement `onTokenRefresh` listeners. If the server has an old token, push notifications will fail silently.

---

### Q16: [INTEROP] External App Communication (Linking)
**Question:** How do you handle 'Deep Linking' vs. 'Universal Links' (iOS) / 'App Links' (Android)?

#### 🔗 Linking Strategies
- **Deep Links**: Custom schemes (`myapp://`). Easy to setup but prone to conflicts.
- **Universal/App Links**: Standard `https://` links. Require a server-side file (`apple-app-site-association` or `assetlinks.json`) to verify ownership. More secure and better UX.

> [!WARNING]
> **Follow-up Trap: "How do you handle the link if the app is already open?"**
> **Answer:** Use the `Linking.addEventListener('url', callback)` to catch incoming links while the app is in the background.

---

### Q17: [SENSORS] High-Frequency JSI Streaming
**Question:** How would you build a high-performance AR or fitness tracking feature that requires 60Hz sensor data?

#### ⚡ JSI Streaming
The old bridge would choke on 60 messages per second. A senior approach uses a **JSI-based Native Module** that writes sensor data directly to a **SharedValue** (Reanimated) or calls a JS callback directly from C++ to minimize overhead.

> [!TIP]
> **Senior Insight: Frame Sync**
> Use `requestAnimationFrame` or Reanimated's `useFrameCallback` to sync your JS logic with the device's screen refresh rate.

---

### Q18: [NETWORKING] Native Networking Internals
**Question:** How does React Native's 'fetch' work under the hood? Why does it matter for debugging?

#### 🌐 The Polyfill
React Native polyfills the web `fetch` API. On the native side, it uses **OkHttp** (Android) and **NSURLSession** (iOS). This means native interceptors, proxy settings, and system-level VPNs automatically apply to your JS network calls.

> [!WARNING]
> **Follow-up Trap: "Can you use Axios?"**
> **Answer:** Yes, Axios also uses the same underlying native networking polyfills in a React Native environment.

---

### Q19: [MEMORY] C++ JSI Object Lifecycle
**Question:** How do you prevent memory leaks when passing C++ objects to JavaScript via JSI?

#### 🧠 Host Objects
JSI uses **HostObjects**. You must ensure that the C++ object's lifecycle is tied to the JS object's garbage collection. If the C++ object is deleted while JS still holds a reference, the app will crash with a segmentation fault.

> [!TIP]
> **Senior Insight: Smart Pointers**
> Always use `std::shared_ptr` or `std::unique_ptr` in your C++ code to manage memory safely within the JSI environment.

---

### Q20: [CODEGEN] TurboModule Codegen Types
**Question:** What are the limitations of types you can pass through Codegen in a TurboModule spec?

#### 🛑 Type Constraints
Codegen doesn't support all TypeScript types. You are limited to:
- Primitives (string, number, boolean)
- Objects (as interfaces)
- Arrays (of primitives or objects)
- Promises and Callbacks
Generic types or complex Union types are generally **not supported** and will fail the build.

> [!WARNING]
> **Follow-up Trap: "How do you pass a complex Union?"**
> **Answer:** Pass it as a string and parse/validate it on the native side, or use a numeric enum.

---

### Q21: [HARDWARE] Low-Level Device Info Internals
**Question:** How does react-native-device-info fetch 'Unique IDs' without violating OS privacy rules?

#### 🔍 Privacy Compliance
It uses **IDFV** on iOS (which is unique per developer) and **ANDROID_ID** on Android. It avoids hardware-bound IDs like IMEI or MAC addresses, which are now blocked by Apple and Google.

> [!TIP]
> **Senior Insight: Privacy Manifests**
> In recent iOS versions, you must declare *why* you are accessing these IDs in a **Privacy Manifest** file, or your app will be rejected from the App Store.

---

### Q22: [ARCHITECTURE] The Bridge vs JSI: Memory-Level Deep Dive
**Question:** Explain the fundamental architectural shift from the JSON Bridge to JSI. What happens at the memory level during communication?

#### 🔄 Bridge: Asynchronous JSON Serialization
The legacy **Bridge** relied on asynchronous JSON serialization where every JS-Native communication required:
1. Data serialization to JSON string
2. Queueing in a single-channel pipe
3. Deserialization on the receiving side
4. Callback execution

#### ⚡ JSI: Direct Memory References
**JSI (JavaScript Interface)** is a lightweight C++ API that allows the JavaScript engine (Hermes/V8) to hold a **direct reference** to C++ Host Objects. No serialization overhead - JS can call native methods **synchronously**.

```cpp
// JSI HostObject Example (C++)
class MyTurboModule : public jsi::HostObject {
  jsi::Value get(jsi::Runtime& runtime, const jsi::PropNameID& name) override {
    if (name.utf8(runtime) == "getData") {
      // Direct JS invocation
    }
  }
}
```

---

[⬅️ Phase 04: Performance](phase4-performance-memory.md) | [Phase 06: State Management ➡️](phase6-state-management.md)
