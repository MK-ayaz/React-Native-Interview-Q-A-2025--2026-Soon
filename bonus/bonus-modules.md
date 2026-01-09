<style>
  .senior-handbook {
    background-color: #ffffff;
    color: #1e293b;
    line-height: 1.8;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    max-width: 1000px;
    margin: 0 auto;
    padding: 40px 20px;
  }

  .phase-header {
    text-align: center;
    padding: 40px 0;
    background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  }

  .phase-header h1 {
    color: #60a5fa;
    font-size: 2.5rem;
    margin-bottom: 12px;
    letter-spacing: -0.05em;
  }

  .phase-header .phase-number {
    display: inline-block;
    background-color: #60a5fa;
    color: #0f172a;
    padding: 6px 14px;
    border-radius: 9999px;
    font-weight: 700;
    font-size: 0.875rem;
    margin-bottom: 16px;
  }

  .module-table {
    width: 100%;
    border-collapse: collapse;
    margin: 40px 0;
    font-size: 0.95rem;
  }

  .module-table th,
  .module-table td {
    padding: 15px;
    border: 1px solid #e2e8f0;
    text-align: left;
    vertical-align: top;
  }

  .module-table th {
    background-color: #f1f5f9;
    font-weight: 700;
    color: #1e293b;
    text-transform: uppercase;
    font-size: 0.85rem;
  }

  .module-table tr:nth-child(even) {
    background-color: #f8fafc;
  }

  .module-table tr:hover {
    background-color: #e2e8f0;
  }

  .category-common {
    background-color: #dcfce7;
    color: #047857;
    font-weight: 600;
    padding: 5px 10px;
    border-radius: 5px;
  }

  .category-important {
    background-color: #fffbeb;
    color: #b45309;
    font-weight: 600;
    padding: 5px 10px;
    border-radius: 5px;
  }

  .category-need-based {
    background-color: #e0f2fe;
    color: #0c4a6e;
    font-weight: 600;
    padding: 5px 10px;
    border-radius: 5px;
  }

  .nav-footer {
    display: flex;
    justify-content: center;
    padding: 30px 0;
    margin-top: 40px;
    border-top: 1px solid #e2e8f0;
  }

  .nav-footer a {
    background-color: #eff6ff;
    color: #2563eb;
    padding: 12px 24px;
    border-radius: 8px;
    text-decoration: none;
    font-weight: 600;
    transition: background-color 0.2s ease, transform 0.2s ease;
  }

  .nav-footer a:hover {
    background-color: #dbeafe;
    transform: translateY(-2px);
  }
</style>

<div class="senior-handbook">

<div class="phase-header">
  <span class="phase-number">BONUS</span>
  <h1 class="phase-title">Essential Development Modules (Prioritized)</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">A categorized and prioritized list of React Native modules most useful during building/development, with brief explanations and use cases.</p>
</div>
---
### 📋 Priority Module List
This section outlines essential React Native modules, categorized and prioritized by their common utility in app development. It focuses on *what* they are and *why* they're important, without deep dives into implementation details.

<table class="module-table">
  <thead>
    <tr>
      <th>Category</th>
      <th>Installation Command</th>
      <th>Description</th>
      <th>Key Use Cases</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><span class="category-common">Most Common</span></td>
      <td><code>npm install @react-navigation/native @react-navigation/native-stack</code><br/>(and other navigators like tabs/drawer)</td>
      <td>The standard, flexible, and extensible navigation solution for React Native apps.</td>
      <td>Authentication flows, tab/drawer navigation, deep linking, screen transitions.</td>
    </tr>
    <tr>
      <td><span class="category-common">Most Common</span></td>
      <td><code>npm install zustand</code> / <code>npm install @reduxjs/toolkit react-redux</code></td>
      <td>Solutions for managing global application state predictably and scalably.</td>
      <td>User authentication, global themes, managing complex forms, server data caching.</td>
    </tr>
    <tr>
      <td><span class="category-common">Most Common</span></td>
      <td><code>npm install axios</code> (Fetch API is built-in)</td>
      <td>Promise-based HTTP clients for making API requests.</td>
      <td>Fetching/sending data to RESTful APIs, handling authentication headers, error handling for network requests.</td>
    </tr>
    <tr>
      <td><span class="category-common">Most Common</span></td>
      <td><code>npm install @react-native-async-storage/async-storage</code> / <code>npm install react-native-mmkv</code></td>
      <td>Provides local, persistent key-value storage for applications.</td>
      <td>Storing user preferences/settings, persisting authentication tokens, offline data caching for non-sensitive info.</td>
    </tr>
    <tr>
      <td><span class="category-common">Most Common</span></td>
      <td><code>npm install @sentry/react-native</code> (Flipper/DevTools often built-in or global)</td>
      <td>Tools for inspecting, debugging, and monitoring React Native applications.</td>
      <td>Network request inspection, UI debugging, real-time crash reporting, performance monitoring.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-reanimated react-native-gesture-handler</code></td>
      <td>Powerful libraries for building smooth, high-performance animations and complex touch gestures.</td>
      <td>Custom swipeable components, fluid screen transitions, interactive elements (draggable, zoomable).</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-fast-image</code></td>
      <td>An enhanced `Image` component focused on performance and advanced caching.</td>
      <td>Displaying large numbers of images in feeds, optimizing image loading in lists, handling authenticated images.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-hook-form</code> / <code>npm install formik</code></td>
      <td>Solutions for building performant, flexible, and extensible forms with validation.</td>
      <td>Building complex forms, user input for authentication/profiles, integrating with UI libraries.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-vector-icons</code></td>
      <td>Customizable icon library with access to thousands of icons from various sets.</td>
      <td>Adding scalable/customizable icons to UI, improving app aesthetics without raster images, cross-platform icon consistency.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install date-fns</code> / <code>npm install moment</code></td>
      <td>Libraries for manipulating, formatting, and calculating dates and times.</td>
      <td>Formatting dates/times for display, performing date calculations (countdowns), handling timezones/localization.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-webview</code></td>
      <td>A `WebView` component for rendering web content in a native view.</td>
      <td>Displaying internal web pages, integrating third-party web services (e.g., payment gateways), rendering rich HTML content.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-camera</code> / <code>npm install react-native-image-picker</code></td>
      <td>Provides access to device camera and photo library for capturing images/videos.</td>
      <td>Taking photos/videos, selecting images from gallery, barcode/QR code scanning.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-keychain</code></td>
      <td>Keychain/Keystore access for storing sensitive data securely.</td>
      <td>Securely storing user credentials, API keys, or other sensitive information.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-fs</code></td>
      <td>Native file system access for reading, writing, and managing files.</td>
      <td>Offline data storage, caching large files, managing user-generated content.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-image-crop-picker</code></td>
      <td>Provides a powerful and highly customizable image picker with cropping functionality.</td>
      <td>User profile picture selection, image editing, uploading images with specific dimensions.</td>
    </tr>
    <tr>
      <td><span class="category-important">Important</span></td>
      <td><code>npm install react-native-pdf</code></td>
      <td>A PDF viewer component for displaying PDF documents within your app.</td>
      <td>Displaying e-books, invoices, reports, or other PDF-based content.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-maps</code></td>
      <td>Customizable Map component for integrating geographical maps into your application.</td>
      <td>Displaying points of interest, building ride-sharing/delivery applications, visualizing data on a map.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install @react-native-firebase/app @react-native-firebase/messaging</code></td>
      <td>Official Firebase module for sending and receiving push notifications.</td>
      <td>Notifying users about new messages/content, sending promotional offers, real-time alerts.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-code-push</code></td>
      <td>Cloud service enabling instant JavaScript and asset updates directly to users' devices, bypassing app stores.</td>
      <td>Deploying urgent bug fixes to production, rapid feature enhancements, A/B testing different feature versions.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install --save-dev @testing-library/react-native detox</code></td>
      <td>Tools for robust unit, integration, and end-to-end testing of React Native applications.</td>
      <td>Unit testing components, integration testing user flows, E2E testing critical app paths for stability.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-device-info</code></td>
      <td>Provides detailed device information like unique ID, system name, version, and app metadata.</td>
      <td>Analytics and user tracking, device-specific feature toggles, reporting device info for bug reports.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-permissions</code></td>
      <td>A unified API for requesting and managing permissions on iOS and Android.</td>
      <td>Requesting camera/location access, managing push notification permissions.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install native-base</code> / <code>npm install @ui-kitten/components @eva-design/eva</code> / <code>npm install react-native-paper</code></td>
      <td>Accessible, utility-first component libraries to build high-quality UIs.</td>
      <td>Rapid prototyping, building consistent UIs, starting new projects without an established design system.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install i18n-js</code></td>
      <td>A simple library to provide i18n translations to your React Native app.</td>
      <td>Supporting multiple languages, adapting UI to different cultures, globalizing applications.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-biometrics</code></td>
      <td>Provides an interface for biometric authentication (Face ID, Touch ID, Android Biometrics).</td>
      <td>Implementing secure login with biometrics, protecting sensitive app sections.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-svg</code></td>
      <td>Render SVG images in React Native, providing support for vector graphics.</td>
      <td>Displaying scalable vector icons, complex custom graphics, dynamic data visualizations.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-calendar-events</code></td>
      <td>Access and manage calendar events on the device.</td>
      <td>Creating/reading calendar events, integrating with personal/work schedules.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-background-fetch</code></td>
      <td>Enables apps to perform background tasks, like data fetching, at opportune moments.</td>
      <td>Periodically syncing data, refreshing content, running small background processes.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-ble-plx</code></td>
      <td>A powerful and flexible library for Bluetooth Low Energy (BLE) communication.</td>
      <td>Interacting with IoT devices, wearables, and other BLE peripherals.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-html-to-pdf</code></td>
      <td>Convert HTML content to a PDF document directly within the app.</td>
      <td>Generating invoices, reports, or shareable documents from dynamic HTML.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install lottie-react-native</code></td>
      <td>Render beautiful Lottie animations in your React Native app.</td>
      <td>Adding high-quality, scalable animations for loading states, onboarding, or micro-interactions.</td>
    </tr>
    <tr>
      <td><span class="category-need-based">Need-Based</span></td>
      <td><code>npm install react-native-haptic-feedback</code></td>
      <td>Trigger native haptic feedback (vibrations) on iOS and Android.</td>
      <td>Providing tactile feedback for user actions (e.g., successful submission, long press).</td>
    </tr>
  </tbody>
</table>

---

<div class="nav-footer">
<a href="../README.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Back to Roadmap</a>
</div>

</div>