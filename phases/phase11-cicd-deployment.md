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
    background: linear-gradient(135deg, #475569 0%, #1e293b 100%);
    padding: 40px;
    border-radius: 24px;
    color: white;
    margin-bottom: 40px;
    text-align: center;
    box-shadow: 0 10px 25px -5px rgba(30, 41, 59, 0.2);
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
  table {
    width: 100%;
    border-collapse: collapse;
    margin: 24px 0;
  }
  th {
    background: #f1f5f9;
    text-align: left;
    padding: 12px;
    font-weight: 600;
    border-bottom: 2px solid #e2e8f0;
  }
  td {
    padding: 12px;
    border-bottom: 1px solid #e2e8f0;
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
<span class="phase-number">Phase 11</span>
  <h1 class="phase-title">CI/CD & Deployment</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Automated Pipelines, App Store Deployment, and Release Management</p>
</div>

---

### 📋 Phase Overview
A Senior Engineer doesn't just "ship code"; they build **Reliable Delivery Systems**. This phase covers the automation of builds, certificate management, Over-the-Air (OTA) updates, and phased release strategies.

---

<div class="qa-card">
<span class="question-tag">PIPELINE</span>

### 11.1 The Gold Standard Pipeline
**Question:** *"What does a 'Gold Standard' CI/CD pipeline look like for a React Native project?"*

A mature pipeline is divided into stages to provide fast feedback and ensure binary stability.

1.  **Commit Stage:** Linting (ESLint), Type checking (TSC), and Unit tests (Jest).
2.  **Verification Stage:** Component tests (RNTL) and E2E tests (Detox) on specific PRs or nightly builds.
3.  **Build Stage:** Generating `.aab` (Android) and `.ipa` (iOS) using **Fastlane**.
4.  **Distribution Stage:** Uploading to TestFlight (iOS) and Firebase App Distribution (Android) for QA.

<div class="senior-insight">

**💡 Senior Insight: Fastlane Match**
Never manage certificates manually. Use **Fastlane Match** to store encrypted certificates in a private Git repo. This ensures the entire team (and the CI server) uses the same signing identity, avoiding the "it doesn't build on my machine" nightmare during release week.
</div>
</div>

<div class="qa-card">
<span class="question-tag">OTA UPDATES</span>

### 11.2 Over-the-Air (OTA) Updates
**Question:** *"When should you use OTA updates (CodePush/Expo Updates), and what are the critical limitations?"*

OTA updates allow you to fix JS bugs or update assets without waiting for App Store review.

**The Golden Rule:** OTA updates can ONLY change the JavaScript bundle and assets. If you add a new native library or change `Info.plist/AndroidManifest.xml`, an OTA update will **crash the app** for users on the old native version.

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "How do you prevent an OTA update from crashing an older version of the app?"**
**Answer:** Use **Target Binary Versioning**. Ensure the OTA update is only delivered to the specific native build version it was tested against. In CodePush, this is handled via the `--targetBinaryVersion` flag.
</div>
</div>

<div class="qa-card">
<span class="question-tag">RELEASES</span>

### 11.3 Phased Release Strategy
**Question:** *"What is a 'Phased Release' and why is it essential for mobile?"*

Unlike web, you cannot "roll back" a native app once it's downloaded. A **Phased Release** (available in both App Store and Play Store) allows you to release to a small percentage of users (e.g., 1%, 5%, 20%) over 7 days.

**Senior Approach:** Monitor Sentry and Crashlytics during the 1% phase. If the crash rate spikes, **Pause** the release immediately, fix the bug, and submit a new version. This limits the "blast radius" of a bad release.

</div>

<div class="qa-card">
<span class="question-tag">ENVIRONMENTS</span>

### 11.4 Environment Management
**Question:** *"How do you manage multiple environments (Staging, Production) in React Native?"*

Seniors avoid hardcoded URLs. They use tools like `react-native-config` combined with **Build Flavors (Android)** and **Schemes (iOS)**.

<div class="code-block-header">
<span>Fastfile</span>
<span>Ruby</span>
</div>

```ruby
lane :beta do
  match(type: "appstore")
  gym(scheme: "MyApp-Staging")
  pilot(skip_waiting_for_build_processing: true)
end
```

<div class="senior-insight">

**💡 Senior Insight: Native Configurations**
Don't just use JS-level environment variables. For things like API keys for native SDKs (Firebase, Maps), you must configure them in the native files (`AndroidManifest.xml`, `Info.plist`) using build-time variables provided by the build system.
</div>
</div>

<div class="qa-card">
<span class="question-tag">REVIEW</span>

### 11.5 App Store Review Rejections
**Question:** *"What are the most common reasons for App Store rejections in React Native, and how do you avoid them?"*

**Common Reasons:**
- **Incomplete Information:** Not providing a demo account for reviewers.
- **Performance:** App crashes on launch (often due to missing native dependencies in the release build).
- **Privacy:** Requesting permissions (Camera, Location) without a clear usage description in `Info.plist`.
- **UI/UX:** Using "Web-like" interactions that don't follow the Human Interface Guidelines (HIG).

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "Apple rejected my app because it uses CodePush. Is CodePush illegal?"**
**Answer:** No. Apple's guidelines (Section 2.4.5) allow downloading code that is run by the system's built-in interpreter (JSCore/Hermes), provided it doesn't significantly change the app's primary purpose.
</div>
</div>

<div class="qa-card">
<span class="question-tag">SCREENSHOTS</span>

### 11.6 Automating Screenshots
**Question:** *"How do you handle generating screenshots for 20+ languages and multiple device sizes?"*

Manual screenshots are a waste of senior engineering time. Use **Fastlane Frameit** and **Snapshot**.

**The Workflow:**
1.  Write UI tests (using `XCUITest` or `UiAutomator`) that navigate to key screens.
2.  Take screenshots at specific points in the test.
3.  Use `frameit` to put those screenshots into device frames and add translated marketing text.
4.  Upload them automatically to App Store Connect and Google Play Console.

</div>

<div class="qa-card">
<span class="question-tag">VERSIONING</span>

### 11.7 Mobile Semantic Versioning
**Question:** *"How does versioning in mobile differ from standard SemVer?"*

Mobile has two version numbers:
- **Version Name (Marketing Version):** e.g., `1.2.0`. Shown to users.
- **Build Number (Version Code):** e.g., `42`. An integer that must always increase.

**Senior Tip:** Use a script to automatically increment the build number based on the number of commits or the CI build ID. This prevents "Duplicate Version" errors during upload.

</div>

<div class="qa-card">
<span class="question-tag">MONITORING</span>

### 11.8 Sentry vs. Crashlytics
**Question:** *"Which monitoring tool would you choose for a large-scale React Native app, and why?"*

**Sentry** is usually preferred for React Native because it has superior support for **Source Maps**. It can map a crash in the obfuscated JS bundle back to the exact line in your TypeScript source.

**Crashlytics** is excellent for native-level crashes and is free, but its JS-layer reporting is less detailed than Sentry's.

<div class="senior-insight">

**💡 Senior Insight: Breadcrumbs**
Don't just look at the stack trace. Use **Breadcrumbs** (logs of user actions leading up to the crash) to reproduce "impossible" bugs. In Sentry, you can automatically capture navigation events, button clicks, and API calls.
</div>
</div>

<div class="qa-card">
<span class="question-tag">FLAGS</span>

### 11.9 Feature Flags & Remote Config
**Question:** *"How do you implement feature flags to decouple 'Deployment' from 'Release'?"*

A feature is **Deployed** when the code is in the binary, but **Released** when the flag is turned on.

**Implementation:**
- Use **Firebase Remote Config** or **LaunchDarkly**.
- Fetch the flags at app startup and store them in a local cache.
- Use a hook like `useFeatureFlag('new_checkout_flow')` to conditionally render UI.

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "What happens if the app is offline and can't fetch the flags?"**
**Answer:** Always provide **Default Values** in your code. The app should function perfectly with a "safe" set of default flags if the network is unavailable.
</div>
</div>

<div class="qa-card">
<span class="question-tag">DEPENDENCIES</span>

### 11.10 Modern Dependency Management
**Question:** *"Why might you choose Yarn Berry (v3+) or pnpm over standard npm for a React Native monorepo?"*

- **pnpm:** Uses a content-addressable store to save disk space and is significantly faster at installing dependencies.
- **Yarn Berry:** Provides **Zero Installs** (storing dependencies in Git) and excellent support for workspace protocols in monorepos.

**Crucial Note:** standard React Native has historically struggled with "Plug'n'Play" (PnP). Most teams use the `node-modules` linker even when using modern Yarn or pnpm.

</div>

<div class="qa-card">
<span class="question-tag">PERFORMANCE</span>

### 11.11 Build Performance Optimization
**Question:** *"How do you reduce build times in a CI environment (e.g., GitHub Actions, Bitrise)?"*

1.  **Caching:** Cache `node_modules`, `~/.gradle`, and `~/Library/Developer/Xcode/DerivedData`.
2.  **Incremental Builds:** Use **ccache** for C++ compilation (Hermes/Native Modules).
3.  **No-op Tasks:** Skip generating source maps or stripping symbols for internal QA builds.
4.  **Parallelization:** Run Android and iOS builds in parallel on separate runners.

</div>

<div class="qa-card">
<span class="question-tag">SECRETS</span>

### 11.12 Secret Management in CI
**Question:** *"How do you securely handle signing keys and API secrets in a public CI environment?"*

- **Never** commit secrets to Git.
- Use **CI Secrets** (GitHub Secrets, Bitrise Env Vars).
- For signing keys, use a base64 encoded string of the `.jks` or `.p12` file stored as a secret, and decode it during the build process.
- **Fastlane Match** (as mentioned in 11.1) is the preferred way for iOS certificates.

</div>

<div class="qa-card">
<span class="question-tag">SHRINKING</span>

### 11.13 App Bundles & Code Shrinking
**Question:** *"What are App Bundles (.aab) and how do they differ from APKs?"*

An **APK** contains all assets and binaries for all device types. An **App Bundle** is a publishing format. Google Play uses it to generate **Split APKs** tailored to the specific user's device (correct screen density, CPU architecture, and language), reducing the download size by up to 50%.

**Code Shrinking:** Use **ProGuard** or **R8** on Android to remove unused code from your native dependencies, further reducing size.

</div>

<div class="qa-card">
<span class="question-tag">ASSETS</span>

### 11.14 Handling Large Assets
**Question:** *"How do you manage an app that needs 500MB+ of assets (videos, 3D models) given store size limits?"*

- **On-Demand Resources (iOS)** and **Play Asset Delivery (Android):** Allow you to download assets only when needed.
- **Remote Assets:** Store assets on a CDN (S3/CloudFront) and download them to the app's document directory on first launch.

<div class="senior-insight">

**💡 Senior Insight: The "Initial Experience"**
Never make the user wait for a 500MB download before they can see the home screen. Download the "core" experience in the binary and fetch the rest in the background.
</div>
</div>

<div class="qa-card">
<span class="question-tag">QUALITY</span>

### 11.15 Code Quality Gates
**Question:** *"What metrics should block a PR from being merged in a senior-led team?"*

1.  **TypeScript Errors:** 0 allowed.
2.  **Linting Errors:** 0 allowed.
3.  **Test Coverage:** Minimum 80% on business logic (utils/services).
4.  **Snapshots:** Any change in UI snapshots must be manually reviewed.
5.  **Bundle Size:** Any PR that increases the binary size by more than 1MB should require justification.

</div>

<div class="qa-card">
  <span class="question-tag">CI BASICS</span>
  <h3 style="margin-top: 0;">Q16: Setting up continuous integration for React Native.</h3>

  <p><strong>Continuous Integration (CI)</strong> automates testing and building your app. Here's how to set it up:</p>

  <div class="code-example">GitHub Actions CI Setup</div>
  <pre class="code-block"><code class="language-yaml"># .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run type checking
      run: npm run type-check

    - name: Run unit tests
      run: npm run test -- --coverage --watchAll=false

    - name: Run integration tests
      run: npm run test:e2e

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  build:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-

    - name: Build Android
      run: cd android && ./gradlew assembleRelease --no-daemon

    - name: Build iOS
      run: |
        cd ios
        pod install
        xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -sdk iphoneos -archivePath $PWD/MyApp.xcarchive archive

    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: app-builds
        path: |
          android/app/build/outputs/apk/release/app-release.apk
          ios/MyApp.xcarchive</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Start with a simple CI pipeline that just runs tests on pull requests. As your app grows, add more sophisticated checks like bundle size monitoring and performance regression tests.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">FASTLANE SETUP</span>
  <h3 style="margin-top: 0;">Q27: Automating app store deployments with Fastlane.</h3>

  <p><strong>Fastlane</strong> automates the tedious parts of mobile deployment:</p>

  <div class="code-example">Fastlane Configuration</div>
  <pre class="code-block"><code class="language-ruby"># Gemfile
source "https://rubygems.org"
gem "fastlane"

# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :beta do
    increment_build_number(xcodeproj: "MyApp.xcodeproj")
    build_app(
      scheme: "MyApp",
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          "com.myapp.bundle" => "MyApp AppStore"
        }
      }
    )
    upload_to_testflight
  end

  desc "Deploy a new version to the App Store"
  lane :release do
    # Ensure we're on the right branch
    ensure_git_branch(branch: 'main')

    # Ensure we have a clean git status
    ensure_git_status_clean

    # Get the latest version from package.json
    package = load_json(json_path: "./package.json")
    version = package["version"]

    # Set iOS version
    increment_version_number(
      version_number: version,
      xcodeproj: "MyApp.xcodeproj"
    )

    # Build and upload
    build_app(
      scheme: "MyApp",
      export_method: "app-store"
    )

    upload_to_app_store(
      force: true,
      reject_if_possible: true,
      skip_metadata: false,
      skip_screenshots: false,
      skip_binary_upload: false
    )

    # Create git tag
    add_git_tag(tag: "ios/v#{version}")
    push_git_tags
  end
end

platform :android do
  desc "Deploy a new version to the Google Play Store"
  lane :release do
    # Ensure we're on the right branch
    ensure_git_branch(branch: 'main')

    package = load_json(json_path: "./package.json")
    version = package["version"]

    # Set Android version
    increment_version_code(
      gradle_file_path: "android/app/build.gradle"
    )

    increment_version_name(
      gradle_file_path: "android/app/build.gradle",
      version_name: version
    )

    # Build AAB (Android App Bundle)
    gradle(
      task: "bundle",
      build_type: "Release",
      project_dir: "android"
    )

    # Upload to Google Play
    upload_to_play_store(
      track: 'production',
      aab: 'android/app/build/outputs/bundle/release/app-release.aab'
    )

    # Create git tag
    add_git_tag(tag: "android/v#{version}")
    push_git_tags
  end
end</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">CODEPUSH OTA</span>
  <h3 style="margin-top: 0;">Q28: Implementing over-the-air updates with CodePush.</h3>

  <p><strong>CodePush</strong> enables instant app updates without app store releases:</p>

  <div class="code-example">CodePush Integration</div>
  <pre class="code-block"><code class="language-javascript">// Installation
// npm install react-native-code-push
// npx react-native link react-native-code-push (or autolinking)

// App.js
import codePush from "react-native-code-push";

const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_RESUME,
  installMode: codePush.InstallMode.IMMEDIATE,
  mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
};

function App() {
  const [syncStatus, setSyncStatus] = useState(null);

  useEffect(() => {
    // Listen to sync status updates
    const syncListener = (status) => {
      setSyncStatus(status);
      console.log('CodePush sync status:', status);
    };

    codePush.getUpdateMetadata().then((update) => {
      if (update) {
        console.log('Current update:', update.label);
      }
    });

    return () => {
      // Cleanup if needed
    };
  }, []);

  const handleManualSync = () => {
    codePush.sync(
      {
        installMode: codePush.InstallMode.ON_NEXT_RESUME,
        mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
      },
      (status) => {
        console.log('Manual sync status:', status);
      },
      (progress) => {
        console.log('Download progress:', progress);
      }
    );
  };

  return (
    &lt;View style={styles.container}>
      &lt;Text>App Version: {DeviceInfo.getVersion()}</Text>
      &lt;Text>CodePush Status: {syncStatus}</Text>

      &lt;TouchableOpacity onPress={handleManualSync}>
        &lt;Text>Check for Updates&lt;/Text>
      &lt;/TouchableOpacity>
    &lt;/View>
  );
}

export default codePush(codePushOptions)(App);

// Deployment script
// npx appcenter codepush release-react -a MyApp/Android -d Production
// npx appcenter codepush release-react -a MyApp/iOS -d Production</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> CodePush is perfect for bug fixes and small feature updates. Never use it for breaking changes that require native code updates - those still need app store releases.
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">ENVIRONMENT MANAGEMENT</span>
  <h3 style="margin-top: 0;">Q29: Managing different environments (dev, staging, production).</h3>

  <p>Proper environment management prevents configuration mistakes:</p>

  <div class="code-example">Environment Configuration</div>
  <pre class="code-block"><code class="language-javascript">// config/index.js
const environments = {
  development: {
    API_URL: 'http://localhost:3000/api',
    SENTRY_DSN: null,
    CODEPUSH_KEY: null,
    ANALYTICS_KEY: null,
  },
  staging: {
    API_URL: 'https://api-staging.myapp.com',
    SENTRY_DSN: 'https://staging-sentry-dsn@sentry.io/project',
    CODEPUSH_KEY: 'staging-codepush-key',
    ANALYTICS_KEY: 'staging-analytics-key',
  },
  production: {
    API_URL: 'https://api.myapp.com',
    SENTRY_DSN: 'https://prod-sentry-dsn@sentry.io/project',
    CODEPUSH_KEY: 'prod-codepush-key',
    ANALYTICS_KEY: 'prod-analytics-key',
  },
};

// Determine current environment
const getCurrentEnvironment = () => {
  // Check for __DEV__ (React Native development mode)
  if (__DEV__) return 'development';

  // Check app version for staging (common pattern)
  const version = DeviceInfo.getVersion();
  if (version.includes('-staging') || version.includes('-beta')) {
    return 'staging';
  }

  return 'production';
};

export const config = {
  ...environments[getCurrentEnvironment()],
  environment: getCurrentEnvironment(),
};

// Usage in app
import { config } from './config';

console.log('Current environment:', config.environment);
console.log('API URL:', config.API_URL);

// Environment-specific components
function DebugMenu() {
  if (config.environment !== 'development') {
    return null;
  }

  return (
    &lt;View style={styles.debugMenu}>
      &lt;Text>Debug Tools&lt;/Text&gt;
      &lt;TouchableOpacity onPress={() => console.log('Debug action')}>
        &lt;Text>Debug Action&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">APP STORE SUBMISSION</span>
  <h3 style="margin-top: 0;">Q30: Preparing and submitting apps to app stores.</h3>

  <p>App store submission is a complex process with many requirements:</p>

  <div class="code-example">App Store Preparation</div>
  <pre class="code-block"><code class="language-javascript">// app.json / app.config.js (Expo)
{
  "expo": {
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.mycompany.myapp",
      "buildNumber": "1.0.0",
      "icon": "./assets/icon-ios.png",
      "splash": {
        "image": "./assets/splash-ios.png"
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.mycompany.myapp",
      "versionCode": 1,
      "icon": "./assets/icon-android.png",
      "splash": {
        "image": "./assets/splash-android.png"
      },
      "permissions": [
        "android.permission.INTERNET",
        "android.permission.SYSTEM_ALERT_WINDOW"
      ]
    },
    "plugins": [
      [
        "expo-build-properties",
        {
          "android": {
            "compileSdkVersion": 34,
            "targetSdkVersion": 34,
            "buildToolsVersion": "34.0.0"
          },
          "ios": {
            "deploymentTarget": "13.4"
          }
        }
      ]
    ]
  }
}

// iOS App Store Connect requirements
// 1. App Icons (various sizes)
// 2. Screenshots (5-10 per device type)
// 3. App Description and Keywords
// 4. Privacy Policy URL
// 5. Support URL
// 6. App Category and Subcategory
// 7. Content Rights
// 8. Age Rating

// Android Play Store requirements
// 1. App Icons (various densities)
// 2. Feature Graphic (1024x500)
// 3. Screenshots (2-8 per device type)
// 4. Short and Full Description
// 5. App Category
// 6. Content Rating
// 7. Privacy Policy
// 8. Target Audience

// Automated screenshot generation
// fastlane/Fastfile
desc "Generate and upload screenshots"
lane :screenshots do
  # For iOS
  capture_ios_screenshots

  # For Android
  capture_android_screenshots

  # Upload to stores
  upload_to_app_store(
    skip_binary_upload: true,
    skip_metadata: false,
    skip_screenshots: false
  )
end</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">DEPLOYMENT STRATEGIES</span>
  <h3 style="margin-top: 0;">Q16: Choosing between app stores vs direct distribution.</h3>

  <p>Different deployment strategies for different use cases:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Method</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Pros</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Cons</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Best For</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">App Store/Play Store</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Discovery, trust, payments</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">30% fee, review delays</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Consumer apps</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Enterprise Distribution</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">No fees, instant updates</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Limited discovery, complex setup</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Internal tools</td>
    </tr>
    <tr>
      <td style="padding: 12px;">Direct APK/IPA</td>
      <td style="padding: 12px;">No fees, full control</td>
      <td style="padding: 12px;">Security risks, complex distribution</td>
      <td style="padding: 12px;">Beta testing, custom distribution</td>
    </tr>
  </table>
</div>

<div class="qa-card">
  <span class="question-tag">ROLLBACK STRATEGIES</span>
  <h3 style="margin-top: 0;">Q17: Planning for deployment failures and rollbacks.</h3>

  <p>Always have a rollback plan for when deployments go wrong:</p>

  <div class="code-example">Rollback Strategies</div>
  <pre class="code-block"><code class="yaml"># Rollback workflow
name: Rollback Deployment

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to rollback to'
        required: true
      reason:
        description: 'Reason for rollback'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest

    steps:
    - name: Rollback iOS
      run: |
        # Rollback TestFlight
        # Rollback CodePush

    - name: Rollback Android
      run: |
        # Rollback Play Store to previous version
        # Rollback CodePush</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">MONITORING & ALERTS</span>
  <h3 style="margin-top: 0;">Q18: Setting up deployment monitoring and alerting.</h3>

  <p>Monitor deployments and get alerts when things go wrong:</p>

  <div class="code-example">Deployment Monitoring</div>
  <pre class="code-block"><code class="yaml"># Deployment monitoring with DataDog
monitors:
  - name: "Deployment Success Rate"
    type: "metric alert"
    query: "avg(last_1h):avg:deployment.success_rate{environment:production} < 95"
    message: "Deployment success rate dropped below 95%"</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">FEATURE FLAGS</span>
  <h3 style="margin-top: 0;">Q19: Implementing feature flags for safe deployments.</h3>

  <p>Feature flags allow you to deploy code without exposing features to users:</p>

  <div class="code-example">Feature Flag Implementation</div>
  <pre class="code-block"><code class="javascript">// Feature flag service
class FeatureFlags {
  constructor() {
    this.flags = {};
    this.loadFlags();
  }

  async loadFlags() {
    try {
      const response = await fetch('/api/feature-flags');
      this.flags = await response.json();
    } catch (error) {
      this.flags = this.defaultFlags;
    }
  }

  isEnabled(flagName) {
    return this.flags[flagName] || false;
  }
}

function NewFeature({ userId }) {
  const isEnabled = FeatureFlags.isEnabled('new-dashboard');

  if (!isEnabled) {
    return &lt;OldDashboard /&gt;;
  }

  return &lt;NewDashboard /&gt;;
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">PERFORMANCE BUDGETS</span>
  <h3 style="margin-top: 0;">Q20: Setting and enforcing performance budgets.</h3>

  <p>Performance budgets prevent deployments that degrade user experience:</p>

  <div class="code-example">Performance Budgets</div>
  <pre class="code-block"><code class="yaml"># budget.json (Lighthouse)
{
  "budgets": [
    {
      "timings": [
        {
          "metric": "interactive",
          "budget": 3000
        }
      ]
    }
  ]
}</code></pre>
</div>

---

<div class="nav-footer">
<a href="phase10-security-best-practices.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 10</a>
<a href="phase12-portfolio-defense.md" style="color: #1e293b; text-decoration: none; font-weight: 600;">Phase 12: Defense ➡️</a>
</div>

</div>
