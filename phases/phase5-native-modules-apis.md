# Phase 5: Native Modules & Device APIs 📱

Welcome to **Phase 5**. A true Senior React Native Engineer knows when the JavaScript world ends and the Native world begins. This phase focuses on bridging that gap, managing device-level permissions, and implementing complex hardware integrations.

---

## 📋 Table of Contents
1. [Custom Native Modules](#1-custom-native-modules)
2. [TurboModules & JSI](#2-turbomodules--jsi)
3. [Permissions Management](#3-permissions-management)
4. [Push Notifications](#4-push-notifications)
5. [Device APIs (Camera & Storage)](#5-device-apis-camera--storage)

---

## 1. Custom Native Modules
When a feature isn't available in the React Native ecosystem, you must write it yourself in Java/Kotlin (Android) or Objective-C/Swift (iOS).

### The Bridge Pattern (Old)
1. **Define Interface:** Create a class extending `ReactContextBaseJavaModule`.
2. **Export Methods:** Use the `@ReactMethod` annotation.
3. **Register Module:** Add it to the `ReactPackage`.

> [!NOTE]
> **Key Limitation:** Communication via the Bridge is asynchronous and requires JSON serialization, which can be a bottleneck for high-frequency data (like sensors).

---

## 2. TurboModules & JSI
The New Architecture replaces the Bridge with **JSI (JavaScript Interface)**, allowing JS to hold a reference to C++ host objects and call methods synchronously.

| Feature | Bridge | TurboModules (JSI) |
| :--- | :--- | :--- |
| **Invocation** | Asynchronous | Synchronous/Asynchronous |
| **Type Safety** | Loose (JSON) | Strong (Codegen) |
| **Initialization** | Eager (at startup) | Lazy (on demand) |
| **Performance** | Slower (overhead) | Near-native speed |

---

## 3. Permissions Management
Handling permissions across two platforms requires a centralized system to avoid "permission hell."

### Robust Permission Hook
```javascript
const usePermissions = (type) => {
  const [status, setStatus] = useState('pending');

  const request = async () => {
    const result = await check(type);
    if (result === RESULTS.DENIED) {
      const newStatus = await request(type);
      setStatus(newStatus);
    } else {
      setStatus(result);
    }
  };

  return { status, request };
};
```

---

## 4. Push Notifications
Push notifications are critical for user retention. A senior implementation handles three states: **Foreground, Background, and Quit.**

### Lifecycle Management
```javascript
class NotificationManager {
  static async initialize() {
    // 1. Request Permissions
    // 2. Get Device Token (FCM/APNs)
    // 3. Set up listeners for received/opened events
  }

  static handleDeepLink(notification) {
    const { data } = notification;
    if (data.route) {
      NavigationService.navigate(data.route, data.params);
    }
  }
}
```

---

## 5. Device APIs (Camera & Storage)
Integrating with hardware requires careful handling of lifecycle events and memory.

> [!WARNING]
> **Warning:** When using the Camera or Large File Storage, always release resources or close streams in the cleanup phase to prevent native-side memory leaks.

### Common Senior Interview Questions:
- *"How do you handle 'Don't ask again' permission states on Android?"*
- *"Explain the difference between a Push Token and a Device ID."*
- *"How would you stream high-frequency accelerometer data to the JS thread?"*

---

## 🏁 Next Steps
Native power comes with complexity. In **Phase 6**, we return to the JS layer to master **State Management** at scale using Redux, Zustand, and RTK Query.

[Back to Main README](../README.md) | [Go to Phase 6: State Management](./phase6-state-management.md)
