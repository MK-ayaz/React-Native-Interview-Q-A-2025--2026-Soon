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
    background: linear-gradient(135deg, #1e293b 0%, #334155 100%);
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
<span class="phase-number">Phase 10</span>
  <h1 class="phase-title">Security Best Practices</h1>
  <p style="margin-top: 16px; opacity: 0.9; font-size: 1.1rem;">Authentication, Data Protection, and Secure Development</p>
</div>

---

### 📋 Phase Overview
Security is not a feature; it's a foundation. For Senior developers, protecting user data, securing network communication, and preventing reverse engineering are non-negotiable responsibilities.

---

<div class="qa-card">
<span class="question-tag">SECURE STORAGE</span>

### 10.1 Secure Storage Alternatives
**Question:** *"Why is `AsyncStorage` considered insecure for sensitive data, and what are the alternatives?"*

`AsyncStorage` stores data in plain text on the device's disk. On rooted Android devices or through certain backup exploits on iOS, this data can be easily read by malicious actors.

| Platform | Technology | Recommended Library |
|----------|------------|----------------------|
| **iOS** | Keychain | `react-native-keychain` |
| **Android** | Keystore / EncryptedSharedPreferences | `react-native-keychain` / `react-native-mmkv` |

<div class="senior-insight">

**💡 Senior Insight: Biometric Integration**
For high-security apps (FinTech, Health), don't just store the token in the Keychain. Use `react-native-keychain`'s biometric authentication feature to ensure the user is physically present before the token is released to your JS code. This prevents "at-rest" attacks where someone might try to programmatically extract the key while the device is unlocked.
</div>
</div>

<div class="qa-card">
<span class="question-tag">NETWORK SECURITY</span>

### 10.2 SSL Pinning
**Question:** *"How does SSL Pinning prevent Man-in-the-Middle (MitM) attacks, and what are the risks?"*

SSL Pinning hardcodes the server's public key or certificate into the app. This ensures the app only talks to your server, even if a malicious actor has installed a rogue root certificate on the user's device.

**The Risks:**
- **Certificate Expiry:** If your server certificate expires and you haven't updated the app with the new pin, the app will stop working for all users.
- **Maintenance:** Requires a robust "pin rotation" strategy and often an emergency "kill switch" to disable pinning if things go wrong.

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "Should we pin the Leaf certificate or the Intermediate/Root certificate?"**
**Answer:** Pinning the **Intermediate** certificate is often a better balance. Leaf certificates change frequently (every 90 days with Let's Encrypt), which would require constant app updates. Pinning the Root is too broad. The Intermediate CA is stable but specific enough to prevent generic MitM.
</div>
</div>

<div class="qa-card">
<span class="question-tag">CODE PROTECTION</span>

### 10.3 Reverse Engineering
**Question:** *"How can you protect a React Native app from being reverse-engineered?"*

While no app is 100% unhackable, seniors implement "defense in depth":

1.  **Hermes Engine:** Pre-compiles JS into bytecode, making it harder to read than plain JS (though it can still be disassembled).
2.  **ProGuard/R8 (Android):** Shrinks and obfuscates Java/Kotlin code, renaming classes and methods to meaningless strings.
3.  **Jailbreak/Root Detection:** Using libraries like `react-native-jailbreak-termroot` to disable sensitive features on compromised devices.
4.  **Native Obfuscation:** Using commercial tools like DexGuard or iXGuard for maximum protection of native logic.

<div class="senior-insight">

**💡 Senior Insight: Business Logic Location**
The ultimate security for sensitive logic is to **never put it in the client**. Critical calculations, validation, or secret generation should always happen on the backend. The mobile app should just be a "dumb terminal" for these operations.
</div>
</div>

<div class="qa-card">
<span class="question-tag">DEEP LINKS</span>

### 10.4 Deep Link Security
**Question:** *"What are the security implications of using Custom URL Schemes vs. Universal Links/App Links?"*

**Custom URL Schemes** (e.g., `myapp://`) can be hijacked by any other app on the device. **Universal Links (iOS)** and **App Links (Android)** use a verified association between your website and your app (via `apple-app-site-association` and `assetlinks.json`), making them significantly more secure.

<div class="code-block-header">
<span>DeepLinkHandler.ts</span>
<span>TypeScript</span>
</div>

```typescript
const handleDeepLink = (url: string) => {
  const { path, params } = parse(url);
  // CRITICAL: Always validate sensitive actions
  if (path === '/reset-password' && !params.token) {
    logSecurityEvent('Invalid password reset attempt');
    return;
  }
};
```
</div>

<div class="qa-card">
<span class="question-tag">OBSERVABILITY</span>

### 10.5 Sensitive Data in Logs
**Question:** *"How do you prevent sensitive data from leaking into logs or crash reports?"*

Logs and crash reports (Sentry/Bugsnag) are often forgotten vectors for data leaks. Seniors implement strict sanitization.

**Best Practices:**
- **Intercept Logs:** Use a custom logger that masks sensitive keys (e.g., `password`, `token`, `credit_card`) using regex.
- **Sentry `beforeSend`:** Use the `beforeSend` hook in Sentry to scrub the event object before it's sent to the server.
- **Environment Control:** Ensure `console.log` is completely stripped in production builds using `babel-plugin-transform-remove-console`.

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "What about Redux/Zustand state in crash reports?"**
**Answer:** Many devs send the entire state with a crash report. This is dangerous. You should only send a sanitized version of the state or breadcrumbs that don't contain PII (Personally Identifiable Information).
</div>
</div>

<div class="qa-card">
<span class="question-tag">SECRETS</span>

### 10.6 API Key Obfuscation
**Question:** *"How should you handle API keys in a React Native app?"*

There is no way to hide a secret in a client-side bundle. If a key is in your JS, it can be found.

**The Hierarchy of Security:**
1.  **The Proxy Pattern (Best):** The app never sees the API key. It talks to your own backend, which appends the key and forwards the request.
2.  **Restricted Keys:** If you must use a client-side key (e.g., Google Maps), restrict it by **Bundle ID** or **API Signature** in the provider's console.
3.  **Environment Variables:** Use `react-native-config` to keep keys out of Git, but acknowledge they will still be in the final binary.

<div class="senior-insight">

**💡 Senior Insight: The "Hardcoded Secret" Fallacy**
Never use `dotenv` and assume it's "secure." It's a development tool for convenience, not a security tool. Any string in your `.env` file that ends up in your JS bundle is public knowledge to anyone with a decompiler.
</div>
</div>

<div class="qa-card">
<span class="question-tag">WEBVIEWS</span>

### 10.7 Content Security Policy (CSP)
**Question:** *"What are the risks of using WebViews, and how do you secure them?"*

WebViews are the most vulnerable part of a React Native app because they are susceptible to XSS (Cross-Site Scripting).

**Securing WebViews:**
- **Enable CSP:** Ensure the loaded HTML has a strict Content Security Policy.
- **Disable File Access:** Set `allowFileAccess={false}` and `allowUniversalAccessFromFileURLs={false}`.
- **Origin Validation:** Use the `onShouldStartLoadWithRequest` prop to only allow navigation to trusted domains.
- **Communication Bridge:** Use `window.ReactNativeWebView.postMessage` with strict JSON schema validation on the receiving end.

</div>

<div class="qa-card">
<span class="question-tag">AUTHENTICATION</span>

### 10.8 Session Management & JWT
**Question:** *"How do you securely manage JWTs and handle token expiry?"*

**The Strategy:**
- **Storage:** Store the `accessToken` in memory (state) and the `refreshToken` in the **Keychain**.
- **Silent Refresh:** Use an axios interceptor or RTK Query `baseQuery` to detect a `401 Unauthorized`, trigger a refresh call, and retry the original request.
- **Logout:** Ensure logout clears both the memory and the Keychain, and also invalidates the session on the server side.

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "Why not store the JWT in AsyncStorage?"**
**Answer:** If an attacker gets access to the device (or a backup), they can grab the JWT and impersonate the user until the token expires. Keychain/Keystore provides hardware-backed encryption that makes this significantly harder.
</div>
</div>

<div class="qa-card">
<span class="question-tag">VALIDATION</span>

### 10.9 Input Validation & Sanitization
**Question:** *"Why is client-side validation alone insufficient, and what are the mobile-specific sanitization needs?"*

Client-side validation is for UX; server-side validation is for security. In mobile, "input" isn't just text fields—it's deep links, push notifications, and NFC tags.

**Mobile Sanitization Strategy:**
- **Deep Link Params:** Treat every parameter as untrusted user input. Use a library like `zod` to validate the schema of the incoming URL.
- **Push Notifications:** Never trust the payload of a push notification to perform sensitive actions without re-authenticating the user.
- **XSS in HTML:** If rendering user-generated content in a WebView, use `DOMPurify` (on the server or in a separate JS context) to strip scripts.

<div class="senior-insight">

**💡 Senior Insight: The "Implicit Trust" Trap**
Devs often trust data coming from "system" sources like the Clipboard or the Contacts API. A malicious app could populate the clipboard with a payload that exploits your app's logic when the user "pastes." Always sanitize.
</div>
</div>

<div class="qa-card">
<span class="question-tag">NATIVE BRIDGE</span>

### 10.10 Secure Native Modules
**Question:** *"What are the security risks associated with the React Native Bridge, and how do you mitigate them?"*

The Bridge is the communication layer between JS and Native. If not secured, it can be an entry point for attacks.

**Mitigation Steps:**
- **Argument Validation:** In your `ReactMethod` (Java/Kotlin/Swift), strictly validate the type and range of every argument.
- **Avoid `eval()`:** Never allow the bridge to execute arbitrary strings as code.
- **Limit Surface Area:** Only expose the absolute minimum number of methods to the JS layer.
- **TurboModules:** Moving to the New Architecture (JSI) reduces the "serializable" nature of the bridge, making some types of injection harder, but still requires strict native validation.

</div>

<div class="qa-card">
<span class="question-tag">RUNTIME PROTECTION</span>

### 10.11 Debugger & Dynamic Analysis
**Question:** *"How do you detect if a debugger is attached or if the app is being run in a dynamic analysis tool?"*

Seniors use "Anti-Tampering" techniques to protect intellectual property and user data.

**Detection Techniques:**
- **Is Debugger Attached:** Check the `isDebuggerConnected` flag in Android's `Debug` class or use `sysctl` in iOS to check for the `P_TRACED` flag.
- **Emulator Detection:** Check for specific hardware strings (e.g., "goldfish", "sdk_gphone") that indicate the app is running in an emulator rather than a physical device.
- **Hook Detection:** Detect the presence of hooking frameworks like **Frida** or **Xposed** by checking for specific files in the system or unexpected function redirections.

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "What should the app do if it detects a debugger in production?"**
**Answer:** The app should **gracefully degrade** or **self-terminate**. However, don't just crash with a generic error; clear sensitive data from memory first and then exit.
</div>
</div>

<div class="qa-card">
<span class="question-tag">OAUTH2</span>

### 10.12 OAuth2/OIDC Mobile Best Practices
**Question:** *"What is PKCE, and why is it required for OAuth2 in mobile apps?"*

**PKCE (Proof Key for Code Exchange)** prevents an attacker from intercepting the authorization code and exchanging it for a token.

**Why it's needed:**
Unlike web apps, mobile apps cannot keep a `client_secret` safe. If an attacker intercepts the custom URL scheme callback, they could steal the code. PKCE adds a dynamically generated "verifier" that only the original app knows, making the stolen code useless.

<div class="senior-insight">

**💡 Senior Insight: System Browser vs. In-App WebView**
Always use the **System Browser** (via `ASWebAuthenticationSession` on iOS or `Custom Tabs` on Android) for login, NOT an in-app WebView. The system browser shares cookies with the main browser (allowing SSO) and prevents the app from "seeing" the user's credentials.
</div>
</div>

<div class="qa-card">
<span class="question-tag">PRIVACY</span>

### 10.13 Screenshot & Recording Prevention
**Question:** *"How do you prevent users from taking screenshots or screen recordings in sensitive parts of your app?"*

This is a common requirement for banking and streaming apps (DRM).

**Implementation:**
- **Android:** Set the `FLAG_SECURE` flag on the window. This makes the screen appear black in screenshots, recordings, and the "Recent Apps" switcher.
- **iOS:** There is no direct "prevent screenshot" API. However, you can listen to the `userDidTakeScreenshotNotification` and show a warning, or use a "secure" text field hack (which hides content in recordings).

</div>

<div class="qa-card">
<span class="question-tag">SUPPLY CHAIN</span>

### 10.14 Dependency Security
**Question:** *"How do you manage the security of your `node_modules` and native dependencies?"*

The biggest threat to modern apps is the **Supply Chain Attack**.

**Your Strategy:**
- **Automated Scanning:** Integrate `npm audit` or **Snyk** into your CI/CD pipeline to block builds with known vulnerabilities.
- **Lockfiles:** Always commit `yarn.lock` or `package-lock.json` to ensure every environment uses the exact same versions.
- **Native Audits:** Don't forget `Podfile.lock` and `build.gradle` dependencies. Native vulnerabilities are often more dangerous as they have higher system privileges.

<div class="follow-up-trap">

**⚠️ Follow-up Trap: "What do you do when a critical dependency has a vulnerability but no update is available?"**
**Answer:** 1. Check if you actually use the vulnerable part of the library. 2. Use `patch-package` to apply a manual fix. 3. Fork the repo. 4. If none are possible, replace the library entirely.
</div>
</div>

<div class="qa-card">
<span class="question-tag">BIOMETRICS</span>

### 10.15 Biometric Implementation Pitfalls
**Question:** *"What are the common mistakes when implementing FaceID/TouchID in React Native?"*

**Common Mistakes:**
- **Trusting the Boolean:** Only checking if `isSensorAvailable` returns true and then "unlocking" the UI. An attacker can easily bypass the UI.
- **No Keychain Binding:** The biometric prompt should be a gateway to a **Keychain key**. If the biometrics fail, the key remains encrypted and the token cannot be retrieved.
- **Ignoring "New Biometry":** Not handling the case where a user adds a new fingerprint to the device. In high-security apps, you should invalidate the current session if the biometric database changes.

</div>

<div class="qa-card">
  <span class="question-tag">BASIC SECURITY</span>
  <h3 style="margin-top: 0;">Q16: Fundamental security concepts for React Native apps.</h3>

  <p>Every React Native developer should understand these core security principles:</p>

  <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px; margin: 16px 0;">
    <strong>🔐 Security Fundamentals:</strong>
    <ol style="margin: 8px 0 0 0; padding-left: 20px;">
      <li><strong>Defense in Depth:</strong> Multiple layers of security, no single point of failure</li>
      <li><strong>Least Privilege:</strong> Grant minimal permissions required</li>
      <li><strong>Fail-Safe Defaults:</strong> Default to secure behavior, require explicit opt-in for risky features</li>
      <li><strong>Zero Trust:</strong> Never trust input, always validate and sanitize</li>
      <li><strong>Secure by Design:</strong> Build security into architecture from day one</li>
    </ol>
  </div>

  <div class="code-example">Security-First Development</div>
  <pre class="code-block"><code class="language-javascript">// ❌ INSECURE: Trusting user input directly
function UnsafeComponent({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Direct string interpolation - SQL injection risk!
    fetch(`/api/users/${userId}`) // What if userId is "1; DROP TABLE users;"?
      .then(res => res.json())
      .then(setUser);
  }, [userId]);

  return &lt;Text&gt;{user?.name}&lt;/Text&gt;;
}

// ✅ SECURE: Input validation and sanitization
function SecureComponent({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Validate input
    if (!userId || typeof userId !== 'string' || userId.length > 50) {
      console.error('Invalid userId');
      return;
    }

    // Use parameterized queries or proper encoding
    const encodedId = encodeURIComponent(userId);
    fetch(`/api/users/${encodedId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]);

  return &lt;Text&gt;{user?.name}&lt;/Text&gt;;
}</code></pre>

  <div class="beginner-tip">
    <strong>🟢 Beginner Tip:</strong> Security isn't an afterthought - it's a fundamental part of how you write every line of code. Always ask: "What could go wrong here?" and "How could this be exploited?"
  </div>
</div>

<div class="qa-card">
  <span class="question-tag">AUTHENTICATION BASICS</span>
  <h3 style="margin-top: 0;">Q17: Implementing secure user authentication.</h3>

  <p>Authentication is the foundation of app security. Get it wrong and everything else fails:</p>

  <div class="code-example">Secure Authentication Flow</div>
  <pre class="code-block"><code class="language-javascript">// 1. Secure login implementation
import AsyncStorage from '@react-native-async-storage/async-storage';
import * as Keychain from 'react-native-keychain';

class AuthService {
  async login(email, password) {
    try {
      // Validate input
      if (!this.isValidEmail(email) || !password || password.length < 8) {
        throw new Error('Invalid credentials');
      }

      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        throw new Error('Login failed');
      }

      const { token, refreshToken, user } = await response.json();

      // Store tokens securely
      await Keychain.setGenericPassword('accessToken', token, {
        service: 'com.myapp.tokens',
      });

      await AsyncStorage.setItem('refreshToken', refreshToken);
      await AsyncStorage.setItem('user', JSON.stringify(user));

      return user;
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    }
  }

  async logout() {
    try {
      // Clear all stored data
      await Keychain.resetGenericPassword({ service: 'com.myapp.tokens' });
      await AsyncStorage.multiRemove(['refreshToken', 'user']);

      // Notify server (optional)
      const token = await this.getAccessToken();
      if (token) {
        await fetch('/api/auth/logout', {
          method: 'POST',
          headers: { Authorization: `Bearer ${token}` },
        });
      }
    } catch (error) {
      console.error('Logout error:', error);
    }
  }

  async getAccessToken() {
    try {
      const credentials = await Keychain.getGenericPassword({
        service: 'com.myapp.tokens'
      });
      return credentials ? credentials.password : null;
    } catch {
      return null;
    }
  }

  async refreshAccessToken() {
    try {
      const refreshToken = await AsyncStorage.getItem('refreshToken');
      if (!refreshToken) throw new Error('No refresh token');

      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken }),
      });

      const { token } = await response.json();

      // Update stored token
      await Keychain.setGenericPassword('accessToken', token, {
        service: 'com.myapp.tokens',
      });

      return token;
    } catch (error) {
      // Force logout on refresh failure
      await this.logout();
      throw error;
    }
  }

  isValidEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
}

// Usage in components
function LoginScreen() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);

  const handleLogin = async () => {
    setLoading(true);
    try {
      const user = await authService.login(email, password);
      navigation.replace('Main');
    } catch (error) {
      Alert.alert('Error', error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    &lt;View&gt;
      &lt;TextInput
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      /&gt;
      &lt;TextInput
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      /&gt;
      &lt;TouchableOpacity onPress={handleLogin} disabled={loading}&gt;
        &lt;Text&gt;{loading ? 'Logging in...' : 'Login'}&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">SECURE STORAGE</span>
  <h3 style="margin-top: 0;">Q18: Choosing the right storage mechanism for sensitive data.</h3>

  <p>Not all data should be stored the same way. Choose based on sensitivity and access patterns:</p>

  <table style="width: 100%; border-collapse: collapse; margin: 24px 0;">
    <tr style="background: #f1f5f9;">
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Data Type</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Storage</th>
      <th style="padding: 12px; text-align: left; font-weight: 600; border-bottom: 2px solid #e2e8f0;">Reason</th>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Auth Tokens</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Keychain (iOS) / Keystore (Android)</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Hardware-backed encryption</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">User Preferences</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">AsyncStorage / MMKV</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Fast access, not sensitive</td>
    </tr>
    <tr>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Cached API Data</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Redux Persist / MMKV</td>
      <td style="padding: 12px; border-bottom: 1px solid #e2e8f0;">Structured storage with sync</td>
    </tr>
    <tr>
      <td style="padding: 12px;">Offline Queue</td>
      <td style="padding: 12px;">SQLite / WatermelonDB</td>
      <td style="padding: 12px;">Complex queries, large datasets</td>
    </tr>
  </table>

  <div class="code-example">Secure Storage Implementation</div>
  <pre class="code-block"><code class="language-javascript">// Secure token storage
import * as Keychain from 'react-native-keychain';

class SecureStorage {
  static async storeToken(token) {
    try {
      await Keychain.setGenericPassword('token', token, {
        service: 'com.myapp.auth',
        accessControl: Keychain.ACCESS_CONTROL.BIOMETRY_ANY, // Require biometrics
        accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED, // Only when device unlocked
      });
    } catch (error) {
      console.error('Failed to store token:', error);
      throw error;
    }
  }

  static async getToken() {
    try {
      const credentials = await Keychain.getGenericPassword({
        service: 'com.myapp.auth'
      });
      return credentials ? credentials.password : null;
    } catch {
      return null;
    }
  }

  static async clearToken() {
    try {
      await Keychain.resetGenericPassword({
        service: 'com.myapp.auth'
      });
    } catch (error) {
      console.error('Failed to clear token:', error);
    }
  }
}

// User preferences (less sensitive)
import AsyncStorage from '@react-native-async-storage/async-storage';

class PreferencesStorage {
  static async setTheme(theme) {
    try {
      await AsyncStorage.setItem('theme', theme);
    } catch (error) {
      console.error('Failed to save theme:', error);
    }
  }

  static async getTheme() {
    try {
      return await AsyncStorage.getItem('theme') || 'light';
    } catch {
      return 'light';
    }
  }
}

// Sensitive user data (encrypted)
import EncryptedStorage from 'react-native-encrypted-storage';

class SensitiveDataStorage {
  static async storeUserData(userData) {
    try {
      const jsonValue = JSON.stringify(userData);
      await EncryptedStorage.setItem('userData', jsonValue);
    } catch (error) {
      console.error('Failed to store user data:', error);
    }
  }

  static async getUserData() {
    try {
      const jsonValue = await EncryptedStorage.getItem('userData');
      return jsonValue ? JSON.parse(jsonValue) : null;
    } catch {
      return null;
    }
  }
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">NETWORK SECURITY</span>
  <h3 style="margin-top: 0;">Q19: Securing network communications and API calls.</h3>

  <p>Network security is critical - never send sensitive data over unencrypted connections:</p>

  <div class="code-example">Secure Network Implementation</div>
  <pre class="code-block"><code class="language-javascript">// 1. HTTPS enforcement
import { Alert } from 'react-native';

class ApiService {
  constructor() {
    this.baseURL = __DEV__
      ? 'http://localhost:3000/api'  // Allow HTTP in development
      : 'https://api.myapp.com';     // Force HTTPS in production
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;

    // Enforce HTTPS in production
    if (!__DEV__ && !url.startsWith('https://')) {
      throw new Error('HTTPS required for production');
    }

    const config = {
      headers: {
        'Content-Type': 'application/json',
        'X-App-Version': '1.0.0',
        'X-Platform': Platform.OS,
        ...options.headers,
      },
      ...options,
    };

    // Add auth token if available
    const token = await SecureStorage.getToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    try {
      const response = await fetch(url, config);

      // Handle common HTTP errors
      if (response.status === 401) {
        // Token expired, trigger logout
        await authService.logout();
        navigation.replace('Login');
        throw new Error('Session expired');
      }

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      return await response.json();
    } catch (error) {
      // Don't log sensitive data
      console.error('API Error:', error.message);
      throw error;
    }
  }
}

// 2. Request/response interceptors
class SecureApiClient {
  constructor() {
    this.interceptors = {
      request: [],
      response: [],
    };
  }

  addRequestInterceptor(interceptor) {
    this.interceptors.request.push(interceptor);
  }

  addResponseInterceptor(interceptor) {
    this.interceptors.response.push(interceptor);
  }

  async makeRequest(url, config) {
    let processedConfig = { ...config };

    // Apply request interceptors
    for (const interceptor of this.interceptors.request) {
      processedConfig = await interceptor(processedConfig);
    }

    const response = await fetch(url, processedConfig);

    // Apply response interceptors
    let processedResponse = response;
    for (const interceptor of this.interceptors.response) {
      processedResponse = await interceptor(processedResponse);
    }

    return processedResponse;
  }
}

// Usage
const apiClient = new SecureApiClient();

// Add logging interceptor (development only)
if (__DEV__) {
  apiClient.addRequestInterceptor((config) => {
    console.log('API Request:', config.url);
    return config;
  });
}

// Add auth interceptor
apiClient.addRequestInterceptor(async (config) => {
  const token = await SecureStorage.getToken();
  if (token) {
    config.headers = {
      ...config.headers,
      Authorization: `Bearer ${token}`,
    };
  }
  return config;
});

// Add error handling interceptor
apiClient.addResponseInterceptor(async (response) => {
  if (response.status === 401) {
    await authService.logout();
    throw new Error('Authentication required');
  }
  return response;
});</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">INPUT VALIDATION</span>
  <h3 style="margin-top: 0;">Q20: Validating and sanitizing user inputs.</h3>

  <p>Never trust user input - validate everything on both client and server:</p>

  <div class="code-example">Input Validation & Sanitization</div>
  <pre class="code-block"><code class="language-javascript">// Comprehensive validation utilities
class ValidationService {
  static isValidEmail(email) {
    const emailRegex = /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;
    return emailRegex.test(email) && email.length <= 254;
  }

  static isValidPassword(password) {
    // At least 8 characters, 1 uppercase, 1 lowercase, 1 number
    const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d@$!%*?&]{8,}$/;
    return passwordRegex.test(password);
  }

  static sanitizeString(input) {
    if (typeof input !== 'string') return '';

    // Remove potentially dangerous characters
    return input
      .replace(/[<>]/g, '') // Remove angle brackets
      .replace(/javascript:/gi, '') // Remove javascript: protocol
      .replace(/on\w+=/gi, '') // Remove event handlers
      .trim();
  }

  static validateUsername(username) {
    if (!username || typeof username !== 'string') return false;

    const cleanUsername = this.sanitizeString(username);

    // Username rules: 3-20 chars, alphanumeric + underscore/hyphen
    const usernameRegex = /^[a-zA-Z0-9_-]{3,20}$/;
    return usernameRegex.test(cleanUsername);
  }

  static validateCreditCard(number) {
    // Remove spaces and dashes
    const cleanNumber = number.replace(/[\s-]/g, '');

    // Check if all digits
    if (!/^\d+$/.test(cleanNumber)) return false;

    // Luhn algorithm for credit card validation
    let sum = 0;
    let shouldDouble = false;

    for (let i = cleanNumber.length - 1; i >= 0; i--) {
      let digit = parseInt(cleanNumber.charAt(i), 10);

      if (shouldDouble) {
        digit *= 2;
        if (digit > 9) digit -= 9;
      }

      sum += digit;
      shouldDouble = !shouldDouble;
    }

    return sum % 10 === 0;
  }
}

// Form component with validation
function SecureLoginForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
  });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validateForm = () => {
    const newErrors = {};

    if (!ValidationService.isValidEmail(formData.email)) {
      newErrors.email = 'Please enter a valid email address';
    }

    if (!ValidationService.isValidPassword(formData.password)) {
      newErrors.password = 'Password must be at least 8 characters with uppercase, lowercase, and number';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async () => {
    if (!validateForm()) return;

    setIsSubmitting(true);
    try {
      await authService.login(formData.email, formData.password);
      navigation.replace('Main');
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleInputChange = (field, value) => {
    // Sanitize input immediately
    const sanitizedValue = ValidationService.sanitizeString(value);

    setFormData(prev => ({
      ...prev,
      [field]: sanitizedValue,
    }));

    // Clear field-specific errors on change
    if (errors[field]) {
      setErrors(prev => ({
        ...prev,
        [field]: undefined,
      }));
    }
  };

  return (
    &lt;View&gt;
      &lt;TextInput
        placeholder="Email"
        value={formData.email}
        onChangeText={(value) => handleInputChange('email', value)}
        keyboardType="email-address"
        autoCapitalize="none"
        style={errors.email ? styles.errorInput : styles.input}
      /&gt;
      {errors.email && &lt;Text style={styles.errorText}>{errors.email}&lt;/Text&gt;}

      &lt;TextInput
        placeholder="Password"
        value={formData.password}
        onChangeText={(value) => handleInputChange('password', value)}
        secureTextEntry
        style={errors.password ? styles.errorInput : styles.input}
      /&gt;
      {errors.password && &lt;Text style={styles.errorText}>{errors.password}&lt;/Text&gt;}

      {errors.submit && &lt;Text style={styles.errorText}>{errors.submit}&lt;/Text&gt;}

      &lt;TouchableOpacity
        onPress={handleSubmit}
        disabled={isSubmitting}
        style={styles.button}
      >
        &lt;Text&gt;{isSubmitting ? 'Logging in...' : 'Login'}&lt;/Text&gt;
      &lt;/TouchableOpacity&gt;
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">BIOMETRIC AUTH</span>
  <h3 style="margin-top: 0;">Q21: Implementing biometric authentication (Touch ID / Face ID).</h3>

  <p>Biometric authentication adds a strong layer of security with user-friendly UX:</p>

  <div class="code-example">Biometric Authentication</div>
  <pre class="code-block"><code class="language-javascript">import ReactNativeBiometrics from 'react-native-biometrics';
import * as Keychain from 'react-native-keychain';

class BiometricAuth {
  static async isBiometricAvailable() {
    try {
      const { available, biometryType } = await ReactNativeBiometrics.isSensorAvailable();
      return { available, biometryType };
    } catch {
      return { available: false, biometryType: null };
    }
  }

  static async authenticateWithBiometrics(reason = 'Authenticate to continue') {
    try {
      const { success, error } = await ReactNativeBiometrics.simplePrompt({
        promptMessage: reason,
        cancelButtonText: 'Cancel',
      });

      if (success) {
        // Biometric authentication successful
        // Now we can access the secure token
        const token = await this.getSecureToken();
        return token;
      } else {
        throw new Error(error || 'Biometric authentication failed');
      }
    } catch (error) {
      console.error('Biometric auth error:', error);
      throw error;
    }
  }

  static async enrollBiometricToken(token) {
    try {
      // Store the sensitive token, protected by biometrics
      await Keychain.setGenericPassword('biometricToken', token, {
        service: 'com.myapp.biometric',
        accessControl: Keychain.ACCESS_CONTROL.BIOMETRY_CURRENT_SET,
        accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
        authenticationType: Keychain.AUTHENTICATION_TYPE.BIOMETRICS,
      });
    } catch (error) {
      console.error('Failed to enroll biometric token:', error);
      throw error;
    }
  }

  static async getSecureToken() {
    try {
      const credentials = await Keychain.getGenericPassword({
        service: 'com.myapp.biometric',
        authenticationPrompt: {
          title: 'Authenticate to access your account',
          subtitle: 'Use your fingerprint or face',
        },
      });

      return credentials ? credentials.password : null;
    } catch (error) {
      // User cancelled or authentication failed
      console.error('Failed to get secure token:', error);
      return null;
    }
  }

  static async clearBiometricData() {
    try {
      await Keychain.resetGenericPassword({
        service: 'com.myapp.biometric'
      });
    } catch (error) {
      console.error('Failed to clear biometric data:', error);
    }
  }
}

// Usage in component
function BiometricLoginButton() {
  const [isAvailable, setIsAvailable] = useState(false);
  const [biometryType, setBiometryType] = useState(null);

  useEffect(() => {
    BiometricAuth.isBiometricAvailable().then(({ available, biometryType }) => {
      setIsAvailable(available);
      setBiometryType(biometryType);
    });
  }, []);

  const handleBiometricLogin = async () => {
    try {
      const token = await BiometricAuth.authenticateWithBiometrics(
        `Login with ${biometryType === 'FaceID' ? 'Face ID' : 'Touch ID'}`
      );

      if (token) {
        // Use the token to authenticate with your API
        await apiService.authenticateWithToken(token);
        navigation.replace('Main');
      }
    } catch (error) {
      Alert.alert('Authentication Failed', error.message);
    }
  };

  if (!isAvailable) {
    return null; // Hide button if biometrics not available
  }

  return (
    &lt;TouchableOpacity onPress={handleBiometricLogin} style={styles.biometricButton}>
      &lt;Text>
        Login with {biometryType === 'FaceID' ? 'Face ID' : 'Touch ID'}
      &lt;/Text&gt;
    &lt;/TouchableOpacity&gt;
  );
}

// Secure enrollment flow
function BiometricSetupScreen() {
  const [step, setStep] = useState('check'); // 'check' | 'enroll' | 'complete'

  const handleSetup = async () => {
    try {
      // First check if biometrics are available
      const { available, biometryType } = await BiometricAuth.isBiometricAvailable();

      if (!available) {
        Alert.alert('Not Available', 'Biometric authentication is not available on this device');
        return;
      }

      // Prompt user to authenticate
      await BiometricAuth.authenticateWithBiometrics(
        `Set up ${biometryType === 'FaceID' ? 'Face ID' : 'Touch ID'} for quick login`
      );

      // Generate and store a secure token
      const secureToken = generateSecureToken();
      await BiometricAuth.enrollBiometricToken(secureToken);

      setStep('complete');
      Alert.alert('Success', 'Biometric authentication has been set up!');
    } catch (error) {
      Alert.alert('Setup Failed', error.message);
    }
  };

  return (
    &lt;View style={styles.container}>
      {step === 'check' && (
        &lt;View&gt;
          &lt;Text>Would you like to set up biometric authentication?</Text&gt;
          &lt;TouchableOpacity onPress={handleSetup}>
            &lt;Text>Set Up Biometrics&lt;/Text&gt;
          &lt;/TouchableOpacity&gt;
        &lt;/View&gt;
      )}

      {step === 'complete' && (
        &lt;View&gt;
          &lt;Text>✓ Biometric authentication is now enabled&lt;/Text&gt;
          &lt;TouchableOpacity onPress={() => navigation.goBack()}>
            &lt;Text>Continue&lt;/Text&gt;
          &lt;/TouchableOpacity&gt;
        &lt;/View&gt;
      )}
    &lt;/View&gt;
  );
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">CERTIFICATE PINNING</span>
  <h3 style="margin-top: 0;">Q22: Implementing SSL certificate pinning.</h3>

  <p>Certificate pinning prevents man-in-the-middle attacks by ensuring your app only communicates with your legitimate servers:</p>

  <div class="code-example">SSL Pinning Implementation</div>
  <pre class="code-block"><code class="language-javascript">// Using react-native-ssl-pinning
import { fetch } from 'react-native-ssl-pinning';

class SecureApiService {
  constructor() {
    // Certificate fingerprints (SHA256 hashes)
    this.certificates = {
      'api.myapp.com': [
        'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=', // Primary cert
        'sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=', // Backup cert
      ],
    };
  }

  async secureRequest(endpoint, options = {}) {
    const url = `https://api.myapp.com${endpoint}`;

    try {
      const response = await fetch(url, {
        ...options,
        sslPinning: {
          certs: this.certificates['api.myapp.com'],
        },
        headers: {
          'Content-Type': 'application/json',
          ...options.headers,
        },
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      return await response.json();
    } catch (error) {
      if (error.message.includes('SSL')) {
        // Certificate pinning failed - possible MITM attack
        console.error('SSL Pinning failed - possible security threat');
        throw new Error('Security error: Connection not trusted');
      }
      throw error;
    }
  }
}

// For iOS - App Transport Security settings
// ios/MyApp/Info.plist
/*
&lt;key&gt;NSAppTransportSecurity&lt;/key&gt;
&lt;dict&gt;
  &lt;key&gt;NSAllowsArbitraryLoads&lt;/key&gt;
  &lt;false/&gt;
  &lt;key&gt;NSExceptionDomains&lt;/key&gt;
  &lt;dict&gt;
    &lt;key&gt;api.myapp.com&lt;/key&gt;
    &lt;dict&gt;
      &lt;key&gt;NSIncludesSubdomains&lt;/key&gt;
      &lt;true/&gt;
      &lt;key&gt;NSTemporaryExceptionAllowsInsecureHTTPLoads&lt;/key&gt;
      &lt;false/&gt;
      &lt;key&gt;NSPinnedLeafCertificates&lt;/key&gt;
      &lt;array&gt;
        &lt;string&gt;certificate-hash-here&lt;/string&gt;
      &lt;/array&gt;
    &lt;/dict&gt;
  &lt;/dict&gt;
&lt;/dict&gt;
*/

// For Android - Network Security Config
// android/app/src/main/res/xml/network_security_config.xml
/*
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;network-security-config&gt;
  &lt;domain-config&gt;
    &lt;domain includeSubdomains="true"&gt;api.myapp.com&lt;/domain&gt;
    &lt;pin-set expiration="2025-01-01"&gt;
      &lt;pin digest="SHA-256"&gt;AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==&lt;/pin&gt;
      &lt;pin digest="SHA-256"&gt;BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB==&lt;/pin&gt;
    &lt;/pin-set&gt;
  &lt;/domain-config&gt;
&lt;/network-security-config&gt;
*/

// Certificate rotation strategy
class CertificateManager {
  static async updateCertificates() {
    try {
      // Fetch new certificate fingerprints from a secure source
      const newCerts = await this.fetchCertificateUpdates();

      // Validate the new certificates
      await this.validateCertificates(newCerts);

      // Update stored certificates
      await AsyncStorage.setItem('certificates', JSON.stringify(newCerts));

      console.log('Certificates updated successfully');
    } catch (error) {
      console.error('Certificate update failed:', error);
      // Continue with old certificates if update fails
    }
  }

  static async getCurrentCertificates() {
    try {
      const stored = await AsyncStorage.getItem('certificates');
      return stored ? JSON.parse(stored) : this.defaultCertificates;
    } catch {
      return this.defaultCertificates;
    }
  }
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">APP HARDENING</span>
  <h3 style="margin-top: 0;">Q23: Protecting against reverse engineering and tampering.</h3>

  <p>Make it harder for attackers to analyze and modify your app:</p>

  <div class="code-example">App Hardening Techniques</div>
  <pre class="code-block"><code class="language-javascript">// 1. Code obfuscation (metro config)
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true,
      },
    }),
    minifierConfig: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
};

// 2. Root/jailbreak detection
import JailMonkey from 'jail-monkey';

class SecurityService {
  static async performSecurityChecks() {
    const checks = {
      isJailBroken: JailMonkey.isJailBroken(),
      canMockLocation: JailMonkey.canMockLocation(),
      isOnExternalStorage: JailMonkey.isOnExternalStorage(),
      trustFall: JailMonkey.trustFall(), // Multiple jailbreak checks
    };

    const failedChecks = Object.entries(checks)
      .filter(([, passed]) => passed)
      .map(([check]) => check);

    if (failedChecks.length > 0) {
      console.warn('Security checks failed:', failedChecks);
      // In production, you might want to:
      // - Show a warning
      // - Disable certain features
      // - Report to analytics
      // - Exit the app (extreme)
    }

    return checks;
  }
}

// 3. Anti-debugging measures (use sparingly)
class AntiDebugService {
  static isDebuggerAttached() {
    // This is a simplified check - real implementation would be more complex
    const start = Date.now();
    debugger; // This will be removed in production builds
    const end = Date.now();

    // If a debugger is attached, this will take longer
    return (end - start) > 100;
  }

  static detectEmulator() {
    // Check for common emulator fingerprints
    const emulatorIndicators = [
      'generic',
      'unknown',
      'emulator',
      'sdk',
    ];

    const brand = DeviceInfo.getBrand().toLowerCase();
    const model = DeviceInfo.getModel().toLowerCase();

    return emulatorIndicators.some(indicator =>
      brand.includes(indicator) || model.includes(indicator)
    );
  }
}

// 4. Tamper detection
class IntegrityChecker {
  static async checkAppIntegrity() {
    try {
      // Check file hashes (would need native implementation)
      const isTampered = await NativeModules.IntegrityChecker.checkIntegrity();

      if (isTampered) {
        console.error('App integrity compromised');
        // Handle tampering detection
        await this.handleTamperingDetected();
      }

      return !isTampered;
    } catch (error) {
      console.error('Integrity check failed:', error);
      return false;
    }
  }

  static async handleTamperingDetected() {
    // Log the incident
    await analytics.track('AppTamperingDetected', {
      timestamp: Date.now(),
      deviceInfo: await DeviceInfo.getDeviceInfo(),
    });

    // In high-security apps, you might:
    // - Clear sensitive data
    // - Force logout
    // - Show warning screen
    // - Disable app functionality
  }
}

// 5. Runtime security checks
class RuntimeSecurity {
  static startSecurityMonitoring() {
    // Periodic security checks
    this.securityCheckInterval = setInterval(async () => {
      await SecurityService.performSecurityChecks();
      await IntegrityChecker.checkAppIntegrity();
    }, 30000); // Every 30 seconds

    // Memory monitoring (prevent memory-based attacks)
    if (__DEV__) {
      console.log('Security monitoring started');
    }
  }

  static stopSecurityMonitoring() {
    if (this.securityCheckInterval) {
      clearInterval(this.securityCheckInterval);
    }
  }
}

// Initialize security monitoring
useEffect(() => {
  RuntimeSecurity.startSecurityMonitoring();

  return () => {
    RuntimeSecurity.stopSecurityMonitoring();
  };
}, []);</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">SECURE CODING</span>
  <h3 style="margin-top: 0;">Q24: Secure coding practices and common vulnerabilities.</h3>

  <p>Following secure coding practices prevents the most common security issues:</p>

  <div style="display: grid; grid-template: repeat(2, auto) / repeat(2, 1fr); gap: 16px; margin: 24px 0;">
    <div style="background: #fee2e2; border: 1px solid #fecaca; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #dc2626;">❌ Insecure Practices</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>String concatenation in SQL queries</li>
        <li>Storing sensitive data in AsyncStorage</li>
        <li>Logging sensitive information</li>
        <li>Hardcoded secrets in code</li>
        <li>Trusting client-side validation only</li>
      </ul>
    </div>
    <div style="background: #f0fdf4; border: 1px solid #bbf7d0; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #166534;">✅ Secure Practices</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Use parameterized queries</li>
        <li>Use Keychain for sensitive data</li>
        <li>Sanitize logs and never log tokens</li>
        <li>Use environment variables for secrets</li>
        <li>Always validate on server side</li>
      </ul>
    </div>
    <div style="background: #fef3c7; border: 1px solid #fde68a; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #92400e;">🔍 Common Vulnerabilities</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>SQL Injection</li>
        <li>Cross-Site Scripting (XSS)</li>
        <li>Insecure Direct Object References</li>
        <li>Weak Authentication</li>
        <li>Unencrypted Communications</li>
      </ul>
    </div>
    <div style="background: #eff6ff; border: 1px solid #dbeafe; padding: 16px; border-radius: 8px;">
      <h4 style="margin: 0 0 8px 0; color: #1e40af;">🛡️ Prevention Strategies</h4>
      <ul style="margin: 0; padding-left: 20px; font-size: 0.875rem;">
        <li>Input validation & sanitization</li>
        <li>Content Security Policy</li>
        <li>Proper authentication flows</li>
        <li>HTTPS everywhere</li>
        <li>Regular security audits</li>
      </ul>
    </div>
  </div>

  <div class="code-example">Secure Coding Patterns</div>
  <pre class="code-block"><code class="language-javascript">// 1. Environment variable usage (never hardcode secrets)
const API_KEY = process.env.EXPO_PUBLIC_API_KEY;
const API_URL = process.env.EXPO_PUBLIC_API_URL;

// 2. Safe logging (never log sensitive data)
class SafeLogger {
  static info(message, data = {}) {
    // Remove sensitive fields before logging
    const safeData = { ...data };
    delete safeData.password;
    delete safeData.token;
    delete safeData.creditCard;

    console.log(message, safeData);
  }

  static error(error, context = {}) {
    // Log errors safely
    const safeContext = { ...context };
    // Remove any sensitive data from context

    console.error(error.message, {
      stack: error.stack,
      ...safeContext,
    });
  }
}

// 3. Secure random number generation
class SecurityUtils {
  static generateSecureToken(length = 32) {
    const array = new Uint8Array(length);
    crypto.getRandomValues(array); // Use crypto API, not Math.random
    return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
  }

  static hashPassword(password) {
    // Use a proper password hashing library
    // Never implement your own hashing
    return bcrypt.hash(password, 12);
  }

  static constantTimeCompare(a, b) {
    // Prevent timing attacks
    if (a.length !== b.length) return false;

    let result = 0;
    for (let i = 0; i < a.length; i++) {
      result |= a.charCodeAt(i) ^ b.charCodeAt(i);
    }

    return result === 0;
  }
}

// 4. Rate limiting (prevent brute force attacks)
class RateLimiter {
  constructor(maxAttempts = 5, windowMs = 15 * 60 * 1000) { // 15 minutes
    this.attempts = new Map();
    this.maxAttempts = maxAttempts;
    this.windowMs = windowMs;
  }

  isAllowed(key) {
    const now = Date.now();
    const userAttempts = this.attempts.get(key) || [];

    // Remove old attempts outside the window
    const validAttempts = userAttempts.filter(
      attempt => now - attempt < this.windowMs
    );

    if (validAttempts.length >= this.maxAttempts) {
      return false; // Rate limit exceeded
    }

    validAttempts.push(now);
    this.attempts.set(key, validAttempts);

    return true;
  }

  reset(key) {
    this.attempts.delete(key);
  }
}

// Usage for login attempts
const loginLimiter = new RateLimiter(5, 15 * 60 * 1000); // 5 attempts per 15 minutes

function handleLogin(email, password) {
  if (!loginLimiter.isAllowed(email)) {
    Alert.alert('Too many attempts', 'Please try again later');
    return;
  }

  // Proceed with login
  authService.login(email, password)
    .then(() => {
      loginLimiter.reset(email); // Reset on success
    })
    .catch(() => {
      // Don't reset on failure - user gets limited attempts
    });
}</code></pre>
</div>

<div class="qa-card">
  <span class="question-tag">INCIDENT RESPONSE</span>
  <h3 style="margin-top: 0;">Q25: Handling security incidents and breaches.</h3>

  <p>Have a plan for when security incidents occur:</p>

  <div class="code-example">Incident Response Plan</div>
  <pre class="code-block"><code class="language-javascript">// 1. Security monitoring and alerting
class SecurityMonitor {
  static async reportSecurityIncident(type, details) {
    try {
      // Send to security monitoring service
      await analytics.track('SecurityIncident', {
        type,
        details,
        timestamp: Date.now(),
        deviceInfo: await DeviceInfo.getDeviceInfo(),
        userId: await this.getCurrentUserId(),
      });

      // Also send to dedicated security service
      await this.sendToSecurityService(type, details);
    } catch (error) {
      console.error('Failed to report security incident:', error);
    }
  }

  static async sendToSecurityService(type, details) {
    // Send to your security monitoring service
    // Could be Sentry, DataDog, or custom service
    const securityPayload = {
      incident: type,
      severity: this.getSeverityLevel(type),
      details: this.sanitizeDetails(details),
      timestamp: new Date().toISOString(),
    };

    await fetch('https://security.myapp.com/incidents', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.SECURITY_API_KEY}`,
      },
      body: JSON.stringify(securityPayload),
    });
  }

  static getSeverityLevel(type) {
    const severityMap = {
      'jailbreak_detected': 'high',
      'tampering_detected': 'critical',
      'suspicious_network': 'medium',
      'failed_auth_attempts': 'low',
    };
    return severityMap[type] || 'medium';
  }

  static sanitizeDetails(details) {
    // Remove any sensitive information before sending
    const sanitized = { ...details };
    delete sanitized.password;
    delete sanitized.token;
    delete sanitized.privateKey;
    return sanitized;
  }
}

// 2. Automatic security responses
class SecurityResponse {
  static async handleJailbreakDetected() {
    await SecurityMonitor.reportSecurityIncident('jailbreak_detected', {
      platform: Platform.OS,
      detectionMethod: 'jailbreak_check',
    });

    // Response actions for jailbroken devices
    if (this.shouldRestrictFunctionality()) {
      await this.enableRestrictedMode();
    }
  }

  static async handleSuspiciousActivity(activityType, details) {
    await SecurityMonitor.reportSecurityIncident('suspicious_activity', {
      activityType,
      ...details,
    });

    // Implement progressive security measures
    await this.implementProgressiveSecurity(activityType);
  }

  static async implementProgressiveSecurity(activityType) {
    switch (activityType) {
      case 'multiple_failed_logins':
        // Enable CAPTCHA or additional verification
        await this.enableAdditionalVerification();
        break;

      case 'unusual_location':
        // Require additional authentication
        await this.requireReAuthentication();
        break;

      case 'tampering_attempt':
        // Clear sensitive data and force logout
        await this.performSecurityCleanup();
        break;
    }
  }

  static async performSecurityCleanup() {
    try {
      // Clear all sensitive data
      await AsyncStorage.multiRemove(['userToken', 'refreshToken', 'userData']);
      await Keychain.resetGenericPassword();
      await SecureStorage.clearAll();

      // Force logout
      navigation.reset({
        index: 0,
        routes: [{ name: 'Login' }],
      });

      Alert.alert(
        'Security Notice',
        'For your security, you have been logged out. Please log in again.'
      );
    } catch (error) {
      console.error('Security cleanup failed:', error);
    }
  }
}

// 3. Security audit logging
class SecurityAudit {
  static async logSecurityEvent(event, details) {
    const auditEntry = {
      event,
      details,
      timestamp: new Date().toISOString(),
      userId: await this.getCurrentUserId(),
      deviceId: DeviceInfo.getDeviceId(),
      appVersion: DeviceInfo.getVersion(),
    };

    // Store locally for audit trail
    await this.storeLocalAuditLog(auditEntry);

    // Send to remote logging service (anonymized)
    await this.sendRemoteAuditLog(this.anonymizeAuditEntry(auditEntry));
  }

  static anonymizeAuditEntry(entry) {
    // Remove or hash personally identifiable information
    const anonymized = { ...entry };
    anonymized.userId = this.hashUserId(entry.userId);
    anonymized.deviceId = this.hashDeviceId(entry.deviceId);
    return anonymized;
  }

  static async storeLocalAuditLog(entry) {
    try {
      const existingLogs = await AsyncStorage.getItem('security_audit') || '[]';
      const logs = JSON.parse(existingLogs);

      logs.push(entry);

      // Keep only last 100 entries to prevent storage bloat
      if (logs.length > 100) {
        logs.splice(0, logs.length - 100);
      }

      await AsyncStorage.setItem('security_audit', JSON.stringify(logs));
    } catch (error) {
      console.error('Failed to store audit log:', error);
    }
  }
}

// 4. Emergency security controls
class EmergencySecurity {
  static async activateKillSwitch() {
    // Emergency measure to disable app functionality
    await AsyncStorage.setItem('app_disabled', 'true');

    // Clear sensitive data
    await SecurityResponse.performSecurityCleanup();

    // Show disabled screen
    this.showDisabledScreen();
  }

  static async checkKillSwitch() {
    const isDisabled = await AsyncStorage.getItem('app_disabled');
    if (isDisabled === 'true') {
      this.showDisabledScreen();
      return true;
    }
    return false;
  }

  static showDisabledScreen() {
    // Navigate to a screen that prevents app usage
    navigation.reset({
      index: 0,
      routes: [{
        name: 'Disabled',
        params: {
          reason: 'Security measures activated. Please contact support.'
        }
      }],
    });
  }
}

// Usage throughout the app
// In authentication failures
if (loginAttempts >= MAX_ATTEMPTS) {
  await SecurityMonitor.reportSecurityIncident('multiple_failed_logins', {
    email,
    attempts: loginAttempts,
  });
}

// In root component
useEffect(() => {
  EmergencySecurity.checkKillSwitch();
  SecurityService.performSecurityChecks();
}, []);</code></pre>
</div>

---

<div class="nav-footer">
<a href="phase9-testing-qa.md" style="color: #64748b; text-decoration: none; font-weight: 600;">⬅️ Phase 09</a>
<a href="phase11-cicd-deployment.md" style="color: #1e293b; text-decoration: none; font-weight: 600;">Phase 11: CI/CD ➡️</a>
</div>

</div>
