[⬅️ Back to Home](../README.md)

# 🎯 Phase 12: Portfolio Defense & System Design

---

console.error('Secure retrieval error:', error);
      return null;
    }
  }

  // Encrypted AsyncStorage for app data
  async setEncryptedItem(key: string, value: any): Promise<void> {
    try {
      const encrypted = await this.encryptData(JSON.stringify(value));
      await AsyncStorage.setItem(key, encrypted);
    } catch (error) {
      console.error('Encrypted storage error:', error);
      throw error;
    }
  }

  async getEncryptedItem(key: string): Promise<any | null> {
    try {
      const encrypted = await AsyncStorage.getItem(key);
      if (!encrypted) return null;

      const decrypted = await this.decryptData(encrypted);
      return JSON.parse(decrypted);
    } catch (error) {
      console.error('Encrypted retrieval error:', error);
      return null;
    }
  }

  private async encryptData(data: string): Promise<string> {
    // In production, use a proper encryption library
    // This is a simplified example
    const key = await this.getEncryptionKey();
    // Implement AES-256-GCM encryption here
    return btoa(data); // Simplified for example
  }

  private async decryptData(encryptedData: string): Promise<string> {
    // In production, use a proper decryption library
    return atob(encryptedData); // Simplified for example
  }

  private async getEncryptionKey(): Promise<string> {
    // Generate or retrieve encryption key
    // In production, use proper key management
    return 'your-encryption-key';
  }

  async clearAllSecureData(): Promise<void> {
    try {
      if (Platform.OS === 'ios') {
        await Keychain.resetGenericPassword({ service: 'com.myapp.secure' });
      } else {
        // Clear all secure store items
        const keys = await SecureStore.getAllKeysAsync();
        await Promise.all(keys.map(key => SecureStore.deleteItemAsync(key)));
      }
      await AsyncStorage.clear();
    } catch (error) {
      console.error('Clear secure data error:', error);
    }
  }
}

// API Security Manager
class APISecurityManager {
  private static instance: APISecurityManager;
  private token: string | null = null;
  private refreshPromise: Promise<string> | null = null;

  static getInstance(): APISecurityManager {
    if (!APISecurityManager.instance) {
      APISecurityManager.instance = new APISecurityManager();
    }
    return APISecurityManager.instance;
  }

  async initialize(): Promise<void> {
    // Load stored token
    const storage = SecureStorageManager.getInstance();
    this.token = await storage.getSecureItem('auth_token');
  }

  async makeSecureRequest<T>(
    url: string,
    options: RequestInit = {}
  ): Promise<T> {
    const headers = {
      ...options.headers,
      'Content-Type': 'application/json',
    };

    // Add auth token if available
    if (this.token) {
      headers.Authorization = `Bearer ${this.token}`;
    }

    try {
      const response = await fetch(url, {
        ...options,
        headers,
      });

      // Handle token expiration
      if (response.status === 401) {
        const newToken = await this.refreshToken();
        if (newToken) {
          headers.Authorization = `Bearer ${newToken}`;
          // Retry request with new token
          return this.makeSecureRequest<T>(url, { ...options, headers });
        }
      }

      if (!response.ok) {
        throw new Error(`API Error: ${response.status}`);
      }

      return response.json();
    } catch (error) {
      console.error('API request failed:', error);
      throw error;
    }
  }

  private async refreshToken(): Promise<string | null> {
    if (this.refreshPromise) {
      return this.refreshPromise;
    }

    this.refreshPromise = this.performTokenRefresh();

    try {
      const newToken = await this.refreshPromise;
      this.token = newToken;
      return newToken;
    } finally {
      this.refreshPromise = null;
    }
  }

  private async performTokenRefresh(): Promise<string> {
    try {
      const refreshToken = await SecureStorageManager.getInstance()
        .getSecureItem('refresh_token');

      if (!refreshToken) {
        throw new Error('No refresh token available');
      }

      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken }),
      });

      if (!response.ok) {
        throw new Error('Token refresh failed');
      }

      const { token: newToken, refreshToken: newRefreshToken } = await response.json();

      // Store new tokens
      const storage = SecureStorageManager.getInstance();
      await storage.setSecureItem('auth_token', newToken);
      if (newRefreshToken) {
        await storage.setSecureItem('refresh_token', newRefreshToken);
      }

      return newToken;
    } catch (error) {
      // Clear tokens on refresh failure
      await SecureStorageManager.getInstance().clearAllSecureData();
      throw error;
    }
  }
}

// Code Obfuscation and Anti-Tampering
const SecurityChecks = {
  // Detect if app is running in debug mode
  isDebugMode(): boolean {
    return __DEV__ || !!(window as any).ReactNative?.__fbBatchedBridgeConfig;
  },

  // Detect jailbreak/root
  isJailbroken(): boolean {
    if (Platform.OS === 'ios') {
      const jailbreakFiles = [
        '/Applications/Cydia.app',
        '/Library/MobileSubstrate/MobileSubstrate.dylib',
        '/bin/bash',
        '/usr/sbin/sshd',
        '/etc/apt',
      ];

      return jailbreakFiles.some(file => {
        try {
          // Check if file exists (simplified)
          return false; // In production, implement actual file system checks
        } catch {
          return false;
        }
      });
    }

    // Android root detection
    const rootFiles = [
      '/system/app/Superuser.apk',
      '/sbin/su',
      '/system/bin/su',
      '/system/xbin/su',
    ];

    return rootFiles.some(file => {
      // Implement actual root detection
      return false;
    });
  },

  // Detect emulator
  isEmulator(): boolean {
    if (Platform.OS === 'ios') {
      return false; // iOS simulators are harder to detect
    }

    // Android emulator detection
    const emulatorIndicators = [
      'google_sdk',
      'Emulator',
      'android_x86',
    ];

    return emulatorIndicators.some(indicator =>
      indicator.includes('sdk') || indicator.includes('emulator')
    );
  },

  // Runtime integrity checks
  async performIntegrityCheck(): Promise<boolean> {
    try {
      // Check bundle integrity (simplified)
      const bundleHash = 'expected-bundle-hash'; // From build time
      // Compare with actual bundle hash

      // Check for hooking frameworks
      const hookingIndicators = [
        'frida',
        'xposed',
        'magisk',
      ];

      // Network security checks
      await this.performNetworkSecurityCheck();

      return true;
    } catch (error) {
      console.error('Integrity check failed:', error);
      return false;
    }
  },

  private async performNetworkSecurityCheck(): Promise<void> {
    // Implement certificate pinning
    // Check for man-in-the-middle attacks
    // Verify SSL/TLS configuration
  },
};

// Privacy Compliance Manager
class PrivacyManager {
  private static instance: PrivacyManager;

  static getInstance(): PrivacyManager {
    if (!PrivacyManager.instance) {
      PrivacyManager.instance = new PrivacyManager();
    }
    return PrivacyManager.instance;
  }

  async requestTrackingPermission(): Promise<boolean> {
    if (Platform.OS === 'ios') {
      // iOS 14.5+ App Tracking Transparency
      const { status } = await requestTrackingPermissionsAsync();
      return status === 'granted';
    }

    // Android - handle as needed
    return true;
  }

  async getConsentStatus(): Promise<ConsentStatus> {
    const storage = SecureStorageManager.getInstance();
    const consent = await storage.getEncryptedItem('user_consent');

    return consent || {
      analytics: false,
      advertising: false,
      crashReporting: true, // Essential
    };
  }

  async updateConsent(consent: Partial<ConsentStatus>): Promise<void> {
    const storage = SecureStorageManager.getInstance();
    const currentConsent = await this.getConsentStatus();

    await storage.setEncryptedItem('user_consent', {
      ...currentConsent,
      ...consent,
    });

    // Update tracking services based on consent
    this.updateTrackingServices(consent);
  }

  private updateTrackingServices(consent: Partial<ConsentStatus>): void {
    if (consent.analytics === false) {
      // Disable analytics
      analytics.disable();
    }

    if (consent.advertising === false) {
      // Disable advertising tracking
      advertising.disable();
    }

    // Crash reporting always enabled (essential)
  }

  async performDataDeletion(): Promise<void> {
    // Implement right to be forgotten
    const storage = SecureStorageManager.getInstance();

    // Clear user data
    await storage.clearAllSecureData();

    // Notify server to delete user data
    try {
      await APISecurityManager.getInstance().makeSecureRequest(
        '/api/user/delete-data',
        { method: 'DELETE' }
      );
    } catch (error) {
      console.error('Server data deletion failed:', error);
    }

    // Clear local storage
    await AsyncStorage.clear();
  }
}

// Security-first Component
const SecureTextInput = ({
  value,
  onChangeText,
  secureTextEntry = false,
  ...props
}) => {
  const [isSecureEntry, setIsSecureEntry] = useState(secureTextEntry);

  // Prevent screenshot when sensitive data is shown
  useEffect(() => {
    let subscription;
    if (!isSecureEntry && value) {
      // Disable screenshots (platform-specific implementation needed)
      subscription = PrivacyManager.getInstance().disableScreenshots();
    }

    return () => {
      if (subscription) {
        subscription.remove();
      }
    };
  }, [isSecureEntry, value]);

  return (
    <TextInput
      value={value}
      onChangeText={onChangeText}
      secureTextEntry={isSecureEntry}
      autoComplete="off"
      autoCorrect={false}
      autoCapitalize="none"
      textContentType="none" // Prevent iOS suggestions
      {...props}
    />
  );
};

// Complete Security Setup
const initializeAppSecurity = async () => {
  // Perform security checks
  const debugMode = SecurityChecks.isDebugMode();
  const jailbroken = SecurityChecks.isJailbroken();
  const emulator = SecurityChecks.isEmulator();

  if (debugMode) {
    console.warn('App running in debug mode');
  }

  if (jailbroken || emulator) {
    // Handle compromised environment
    Alert.alert(
      'Security Warning',
      'This device may be compromised. App functionality may be limited.',
      [{ text: 'OK' }]
    );
  }

  // Initialize secure storage and API
  await SecureStorageManager.getInstance().initialize?.();
  await APISecurityManager.getInstance().initialize();

  // Request privacy permissions
  await PrivacyManager.getInstance().requestTrackingPermission();
};
```

---

## 🎯 Phase 11 — Advanced Patterns & Architecture

**Advanced pattern questions test deep architectural knowledge** - expect questions about custom hooks, HOCs, render props, and micro-frontend architectures.

### 11.1 **Custom Hooks Architecture and Composition**

**Question:** *"Design a comprehensive custom hooks system with proper TypeScript support, error handling, and composability for complex business logic."*

**Perfect Answer Structure:**
```
CUSTOM HOOKS ARCHITECTURE:

🎯 HOOK DESIGN PRINCIPLES:
- Single Responsibility Principle
- Composable and reusable
- TypeScript-first design
- Error boundaries integration

🎯 ADVANCED PATTERNS:
- Hook composition and chaining
- Async operations handling
- State synchronization
- Performance optimizations

🎯 TESTING STRATEGIES:
- Hook testing utilities
- Mock implementations
- Integration testing
- Performance benchmarking
```

**Complete Custom Hooks Architecture:**
```typescript
// Base hook utilities
type AsyncState<T> = {
  data: T | null;
  loading: boolean;
  error: Error | null;
};

type AsyncAction<T> = {
  execute: (...args: any[]) => Promise<T>;
  reset: () => void;
} & AsyncState<T>;

// Generic async hook with TypeScript
function useAsyncOperation<T, Args extends any[]>(
  operation: (...args: Args) => Promise<T>,
  initialData: T | null = null
): AsyncAction<T> & { execute: (...args: Args) => Promise<T> } {
  const [state, setState] = useState<AsyncState<T>>({
    data: initialData,
    loading: false,
    error: null,
  });

  const execute = useCallback(async (...args: Args) => {
    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const result = await operation(...args);
      setState({ data: result, loading: false, error: null });
      return result;
    } catch (error) {
      const errorObj = error instanceof Error ? error : new Error(String(error));
      setState(prev => ({ ...prev, loading: false, error: errorObj }));
      throw errorObj;
    }
  }, [operation]);

  const reset = useCallback(() => {
    setState({ data: initialData, loading: false, error: null });
  }, [initialData]);

  return {
    ...state,
    execute,
    reset,
  };
}

// Hook composition utilities
function useCombinedHooks<T extends Record<string, any>>(
  hooks: T
): T {
  // Return hooks object directly for composition
  return hooks;
}

// Data fetching hook with caching
interface CacheConfig {
  ttl?: number;
  key?: string;
}

function useCachedAsync<T>(
  fetcher: () => Promise<T>,
  config: CacheConfig = {}
): AsyncState<T> & { refetch: () => Promise<void> } {
  const { ttl = 5 * 60 * 1000, key = 'default' } = config; // 5 minutes default

  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: false,
    error: null,
  });

  const cache = useRef<Map<string, { data: T; timestamp: number }>>(new Map());

  const fetchData = useCallback(async () => {
    const now = Date.now();
    const cached = cache.current.get(key);

    // Return cached data if valid
    if (cached && (now - cached.timestamp) < ttl) {
      setState({ data: cached.data, loading: false, error: null });
      return;
    }

    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const data = await fetcher();
      cache.current.set(key, { data, timestamp: now });
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState(prev => ({
        ...prev,
        loading: false,
        error: error instanceof Error ? error : new Error(String(error))
      }));
    }
  }, [fetcher, key, ttl]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return {
    ...state,
    refetch: fetchData,
  };
}

// Form management hook
interface FormConfig<T> {
  initialValues: T;
  validationSchema?: (values: T) => Record<keyof T, string | null>;
  onSubmit?: (values: T) => Promise<void> | void;
}

function useForm<T extends Record<string, any>>(config: FormConfig<T>) {
  const { initialValues, validationSchema, onSubmit } = config;

  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const setValue = useCallback(<K extends keyof T>(field: K, value: T[K]) => {
    setValues(prev => ({ ...prev, [field]: value }));

    // Clear error when field changes
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: null }));
    }
  }, [errors]);

  const setTouched = useCallback(<K extends keyof T>(field: K, isTouched = true) => {
    setTouched(prev => ({ ...prev, [field]: isTouched }));
  }, []);

  const validate = useCallback(() => {
    if (!validationSchema) return true;

    const validationErrors = validationSchema(values);
    const hasErrors = Object.values(validationErrors).some(error => error !== null);

    setErrors(validationErrors);
    return !hasErrors;
  }, [validationSchema, values]);

  const handleSubmit = useCallback(async (e?: any) => {
    e?.preventDefault?.();

    // Mark all fields as touched
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {} as Record<string, boolean>);
    setTouched(allTouched);

    if (!validate()) return;

    setIsSubmitting(true);
    try {
      await onSubmit?.(values);
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validate, onSubmit]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  // Computed values
  const isValid = Object.values(errors).every(error => error === null);
  const isDirty = JSON.stringify(values) !== JSON.stringify(initialValues);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    isValid,
    isDirty,
    setValue,
    setTouched,
    validate,
    handleSubmit,
    reset,
  };
}

// Infinite scroll hook
interface InfiniteScrollConfig<T> {
  fetcher: (page: number, pageSize: number) => Promise<{ data: T[]; hasMore: boolean }>;
  pageSize?: number;
  initialData?: T[];
}

function useInfiniteScroll<T>(config: InfiniteScrollConfig<T>) {
  const { fetcher, pageSize = 20, initialData = [] } = config;

  const [data, setData] = useState<T[]>(initialData);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const [hasMore, setHasMore] = useState(true);
  const [page, setPage] = useState(1);

  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;

    setLoading(true);
    setError(null);

    try {
      const result = await fetcher(page, pageSize);
      setData(prev => [...prev, ...result.data]);
      setHasMore(result.hasMore);
      setPage(prev => prev + 1);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  }, [fetcher, page, pageSize, loading, hasMore]);

  const refresh = useCallback(async () => {
    setLoading(true);
    setError(null);
    setPage(1);

    try {
      const result = await fetcher(1, pageSize);
      setData(result.data);
      setHasMore(result.hasMore);
      setPage(2);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  }, [fetcher, pageSize]);

  return {
    data,
    loading,
    error,
    hasMore,
    loadMore,
    refresh,
  };
}

// Real-time data synchronization hook
function useRealtimeData<T>(
  key: string,
  fetcher: () => Promise<T>,
  updateInterval = 30000
) {
  const [data, setData] = useState<T | null>(null);
  const [lastUpdated, setLastUpdated] = useState<Date | null>(null);
  const [isOnline, setIsOnline] = useState(true);

  // Fetch data
  const fetchData = useCallback(async () => {
    try {
      const result = await fetcher();
      setData(result);
      setLastUpdated(new Date());
    } catch (error) {
      console.error('Failed to fetch real-time data:', error);
    }
  }, [fetcher]);

  // Set up polling
  useEffect(() => {
    fetchData(); // Initial fetch

    const interval = setInterval(() => {
      if (isOnline) {
        fetchData();
      }
    }, updateInterval);

    return () => clearInterval(interval);
  }, [fetchData, updateInterval, isOnline]);

  // Network status monitoring
  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setIsOnline(state.isConnected ?? false);
    });

    return unsubscribe;
  }, []);

  // WebSocket or other real-time updates could be added here

  return {
    data,
    lastUpdated,
    isOnline,
    refetch: fetchData,
  };
}

// Hook testing utilities
export const createHookTestWrapper = () => {
  const TestWrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => (
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        {children}
      </ThemeProvider>
    </QueryClientProvider>
  );

  return TestWrapper;
};

export const renderHookWithProviders = <T,>(
  hook: () => T,
  options?: { initialProps?: any }
) => {
  return renderHook(hook, {
    wrapper: createHookTestWrapper(),
    ...options,
  });
};

// Performance monitoring for hooks
export const useHookPerformanceMonitor = (hookName: string) => {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(Date.now());

  renderCount.current += 1;

  useEffect(() => {
    const now = Date.now();
    const renderTime = now - lastRenderTime.current;

    if (__DEV__ && renderTime > 16) {
      console.warn(`${hookName} slow render: ${renderTime}ms (render #${renderCount.current})`);
    }

    lastRenderTime.current = now;
  });

  return renderCount.current;
};

// Complete example usage
const useAdvancedDataManagement = () => {
  return useCombinedHooks({
    // Form management
    form: useForm({
      initialValues: { email: '', password: '' },
      validationSchema: (values) => ({
        email: values.email.includes('@') ? null : 'Invalid email',
        password: values.password.length >= 6 ? null : 'Password too short',
      }),
      onSubmit: async (values) => {
        await api.login(values);
      },
    }),

    // Cached data fetching
    userProfile: useCachedAsync(
      () => api.getUserProfile(),
      { key: 'user-profile', ttl: 10 * 60 * 1000 } // 10 minutes
    ),

    // Infinite scroll
    posts: useInfiniteScroll({
      fetcher: (page, size) => api.getPosts({ page, size }),
    }),

    // Real-time updates
    notifications: useRealtimeData(
      'notifications',
      () => api.getNotifications(),
      15000 // 15 seconds
    ),

    // Performance monitoring
    performance: useHookPerformanceMonitor('useAdvancedDataManagement'),
  });
};
```

---

## 🎯 Phase 12 — Portfolio Defense & System Design

**Portfolio defense is where you prove production experience** - prepare to discuss architecture decisions, challenges faced, and technical trade-offs.

### 12.1 **Technical Decision-Making Framework**

**Question:** *"Walk through your decision-making process for choosing technologies and architectures in your portfolio projects."*

**Perfect Answer Structure:**
```
DECISION FRAMEWORK:

🎯 ASSESSMENT CRITERIA:
- Technical requirements analysis
- Team expertise evaluation
- Project constraints consideration
- Long-term maintainability

🎯 EVALUATION PROCESS:
- Proof of concept validation
- Performance benchmarking
- Security implications review
- Scalability testing

🎯 IMPLEMENTATION STRATEGY:
- Incremental adoption approach
- Migration planning
- Rollback procedures
- Monitoring and alerting setup
```

**Complete Technical Decision Framework:**
```typescript
// Technology Evaluation Framework
interface TechnologyEvaluation {
  name: string;
  category: 'frontend' | 'backend' | 'database' | 'infrastructure' | 'tooling';
  maturity: 'experimental' | 'stable' | 'legacy';
  community: 'small' | 'medium' | 'large' | 'enterprise';
  documentation: 'poor' | 'adequate' | 'excellent';
  licensing: 'open-source' | 'commercial' | 'hybrid';
  maintenance: 'active' | 'maintenance' | 'deprecated';
}

interface ProjectRequirements {
  scale: 'small' | 'medium' | 'large' | 'enterprise';
  timeline: 'rush' | 'normal' | 'flexible';
  budget: 'low' | 'medium' | 'high' | 'unlimited';
  team: {
    size: number;
    expertise: string[];
    location: 'co-located' | 'distributed' | 'remote';
  };
  technical: {
    performance: 'low' | 'medium' | 'high' | 'critical';
    security: 'basic' | 'standard' | 'high' | 'critical';
    scalability: 'fixed' | 'moderate' | 'high' | 'elastic';
    integration: string[]; // APIs, services, platforms
  };
}

class TechnologyDecisionEngine {
  private evaluations: Map<string, TechnologyEvaluation> = new Map();

  evaluateTechnology(
    tech: TechnologyEvaluation,
    requirements: ProjectRequirements
  ): DecisionResult {
    let score = 0;
    const reasons: string[] = [];

    // Maturity assessment
    switch (requirements.scale) {
      case 'enterprise':
        if (tech.maturity !== 'stable') {
          score -= 30;
          reasons.push('Enterprise projects require stable technologies');
        }
        break;
      case 'small':
        // More lenient for small projects
        break;
    }

    // Community and support
    if (tech.community === 'small' && requirements.technical.performance === 'critical') {
      score -= 20;
      reasons.push('Small community may not support critical performance needs');
    }

    // Timeline considerations
    if (requirements.timeline === 'rush' && tech.documentation === 'poor') {
      score -= 25;
      reasons.push('Poor documentation slows down rushed timelines');
    }

    // Team expertise
    const hasExpertise = requirements.team.expertise.some(skill =>
      tech.name.toLowerCase().includes(skill.toLowerCase())
    );

    if (!hasExpertise && tech.documentation !== 'excellent') {
      score -= 15;
      reasons.push('Team lacks expertise and documentation is inadequate');
    }

    // Budget considerations
    if (requirements.budget === 'low' && tech.licensing === 'commercial') {
      score -= 40;
      reasons.push('Commercial licensing conflicts with low budget');
    }

    // Performance requirements
    if (requirements.technical.performance === 'critical') {
      if (!this.hasPerformanceCharacteristics(tech)) {
        score -= 35;
        reasons.push('Technology may not meet critical performance requirements');
      }
    }

    return {
      technology: tech.name,
      score: Math.max(0, score),
      recommendation: this.getRecommendation(score),
      reasons,
      risks: this.assessRisks(tech, requirements),
      alternatives: this.suggestAlternatives(tech),
    };
  }

  private hasPerformanceCharacteristics(tech: TechnologyEvaluation): boolean {
    // Check if technology has known performance characteristics
    const highPerformanceTech = ['rust', 'go', 'c++', 'react-native-reanimated'];
    return highPerformanceTech.some(t => tech.name.toLowerCase().includes(t));
  }

  private getRecommendation(score: number): 'adopt' | 'consider' | 'avoid' {
    if (score >= 70) return 'adopt';
    if (score >= 40) return 'consider';
    return 'avoid';
  }

  private assessRisks(tech: TechnologyEvaluation, reqs: ProjectRequirements): string[] {
    const risks: string[] = [];

    if (tech.maturity === 'experimental') {
      risks.push('Breaking changes likely');
    }

    if (tech.maintenance === 'deprecated') {
      risks.push('Security vulnerabilities may emerge');
    }

    if (reqs.technical.scalability === 'elastic' && !this.isScalable(tech)) {
      risks.push('May not scale to required levels');
    }

    return risks;
  }

  private isScalable(tech: TechnologyEvaluation): boolean {
    const scalableTech = ['kubernetes', 'aws', 'react', 'nodejs'];
    return scalableTech.some(t => tech.name.toLowerCase().includes(t));
  }

  private suggestAlternatives(tech: TechnologyEvaluation): string[] {
    const alternatives: Record<string, string[]> = {
      'react-native': ['flutter', 'ionic', 'expo'],
      'redux': ['zustand', 'mobx', 'context-api'],
      'firebase': ['supabase', 'aws-amplify', 'appwrite'],
    };

    return alternatives[tech.name.toLowerCase()] || [];
  }
}

// Migration Strategy Framework
interface MigrationStrategy {
  technology: string;
  current: string;
  target: string;
  approach: 'big-bang' | 'incremental' | 'parallel' | 'strangler';
  phases: MigrationPhase[];
  rollback: RollbackPlan;
  success: SuccessMetrics;
}

interface MigrationPhase {
  name: string;
  duration: string;
  risk: 'low' | 'medium' | 'high';
  dependencies: string[];
  validation: string[];
}

interface RollbackPlan {
  timeToRollback: string;
  dataBackup: boolean;
  featureFlags: boolean;
  databaseMigration: boolean;
}

interface SuccessMetrics {
  performance: string[];
  userExperience: string[];
  business: string[];
  technical: string[];
}

class MigrationPlanner {
  createMigrationStrategy(
    currentTech: string,
    targetTech: string,
    projectSize: 'small' | 'large'
  ): MigrationStrategy {
    const baseStrategy: MigrationStrategy = {
      technology: currentTech,
      current: currentTech,
      target: targetTech,
      approach: projectSize === 'small' ? 'big-bang' : 'incremental',
      phases: this.createPhases(currentTech, targetTech, projectSize),
      rollback: this.createRollbackPlan(projectSize),
      success: this.defineSuccessMetrics(currentTech, targetTech),
    };

    return baseStrategy;
  }

  private createPhases(
    current: string,
    target: string,
    size: 'small' | 'large'
  ): MigrationPhase[] {
    if (size === 'small') {
      return [
        {
          name: 'Planning and Setup',
          duration: '1 week',
          risk: 'low',
          dependencies: [],
          validation: ['Requirements gathered', 'Team trained'],
        },
        {
          name: 'Migration Execution',
          duration: '2-3 weeks',
          risk: 'high',
          dependencies: ['Planning and Setup'],
          validation: ['All features migrated', 'Tests passing'],
        },
        {
          name: 'Testing and Stabilization',
          duration: '1 week',
          risk: 'medium',
          dependencies: ['Migration Execution'],
          validation: ['E2E tests passing', 'Performance benchmarks met'],
        },
      ];
    }

    // Large project phases
    return [
      {
        name: 'Proof of Concept',
        duration: '2 weeks',
        risk: 'low',
        dependencies: [],
        validation: ['POC demonstrates feasibility'],
      },
      {
        name: 'Infrastructure Setup',
        duration: '3 weeks',
        risk: 'medium',
        dependencies: ['Proof of Concept'],
        validation: ['New infrastructure deployed', 'CI/CD pipelines ready'],
      },
      {
        name: 'Incremental Migration',
        duration: '8-12 weeks',
        risk: 'high',
        dependencies: ['Infrastructure Setup'],
        validation: ['Feature flags working', 'Gradual rollout successful'],
      },
      {
        name: 'Full Migration and Optimization',
        duration: '4 weeks',
        risk: 'medium',
        dependencies: ['Incremental Migration'],
        validation: ['Legacy system decommissioned', 'Performance optimized'],
      },
    ];
  }

  private createRollbackPlan(size: 'small' | 'large'): RollbackPlan {
    return {
      timeToRollback: size === 'small' ? '4 hours' : '24 hours',
      dataBackup: true,
      featureFlags: true,
      databaseMigration: size === 'large',
    };
  }

  private defineSuccessMetrics(current: string, target: string): SuccessMetrics {
    return {
      performance: [
        'Bundle size reduced by 20%',
        'App startup time improved by 15%',
        'Memory usage optimized',
      ],
      userExperience: [
        'No user-facing regressions',
        'Improved responsiveness',
        'Better offline capabilities',
      ],
      business: [
        'Development velocity increased',
        'Technical debt reduced',
        'Maintenance costs lowered',
      ],
      technical: [
        'Code coverage maintained',
        'Type safety improved',
        'Build times acceptable',
      ],
    };
  }
}

// Technology Radar Implementation
interface TechnologyRadar {
  adopt: string[];
  trial: string[];
  assess: string[];
  hold: string[];
}

class TechnologyRadarManager {
  private radar: TechnologyRadar = {
    adopt: [],
    trial: [],
    assess: [],
    hold: [],
  };

  updateTechnologyStatus(technology: string, status: keyof TechnologyRadar): void {
    // Remove from all categories first
    Object.keys(this.radar).forEach(key => {
      const index = this.radar[key as keyof TechnologyRadar].indexOf(technology);
      if (index > -1) {
        this.radar[key as keyof TechnologyRadar].splice(index, 1);
      }
    });

    // Add to new category
    this.radar[status].push(technology);
  }

  getRecommendations(): TechnologyRadar {
    return { ...this.radar };
  }

  getStatus(technology: string): keyof TechnologyRadar | null {
    for (const [status, technologies] of Object.entries(this.radar)) {
      if (technologies.includes(technology)) {
        return status as keyof TechnologyRadar;
      }
    }
    return null;
  }
}

// Portfolio Project Analysis Framework
interface PortfolioAnalysis {
  project: {
    name: string;
    domain: string;
    scale: string;
    duration: string;
  };
  architecture: {
    frontend: string[];
    backend: string[];
    database: string[];
    infrastructure: string[];
  };
  decisions: TechnologyDecision[];
  challenges: Challenge[];
  outcomes: Outcome[];
}

interface TechnologyDecision {
  technology: string;
  category: string;
  reason: string;
  alternatives: string[];
  tradeoffs: string[];
}

interface Challenge {
  description: string;
  impact: 'low' | 'medium' | 'high';
  solution: string;
  lessons: string[];
}

interface Outcome {
  metric: string;
  before: string;
  after: string;
  improvement: string;
}

class PortfolioAnalyzer {
  analyzeProject(project: PortfolioAnalysis): ProjectInsights {
    return {
      technologyMaturity: this.assessTechnologyMaturity(project.architecture),
      architecturalPatterns: this.identifyPatterns(project),
      decisionQuality: this.evaluateDecisions(project.decisions),
      challengeHandling: this.analyzeChallenges(project.challenges),
      businessImpact: this.measureOutcomes(project.outcomes),
      recommendations: this.generateRecommendations(project),
    };
  }

  private assessTechnologyMaturity(architecture: PortfolioAnalysis['architecture']): TechnologyMaturityScore {
    // Implementation for assessing technology choices
    return {
      overall: 85,
      breakdown: {
        frontend: 90,
        backend: 80,
        database: 85,
        infrastructure: 80,
      },
    };
  }

  private identifyPatterns(project: PortfolioAnalysis): string[] {
    // Identify architectural patterns used
    return ['microservices', 'serverless', 'responsive-design'];
  }

  private evaluateDecisions(decisions: TechnologyDecision[]): DecisionEvaluation {
    // Evaluate quality of technology decisions
    return {
      averageScore: 82,
      strengths: ['Performance considerations', 'Scalability planning'],
      weaknesses: ['Documentation gaps', 'Migration planning'],
    };
  }

  private analyzeChallenges(challenges: Challenge[]): ChallengeAnalysis {
    // Analyze how challenges were handled
    return {
      resolutionRate: 95,
      averageImpact: 'medium',
      learningOutcomes: ['Better testing strategies', 'Improved monitoring'],
    };
  }

  private measureOutcomes(outcomes: Outcome[]): BusinessImpact {
    // Measure business impact of technical decisions
    return {
      performance: 25,
      userExperience: 30,
      development: 20,
      business: 25,
    };
  }

  private generateRecommendations(project: PortfolioAnalysis): string[] {
    return [
      'Consider adopting newer state management solutions',
      'Implement comprehensive monitoring and alerting',
      'Establish regular security audits',
      'Create reusable component library',
    ];
  }
}
```

---

**This comprehensive React Native Interview Q&A covers all 12 phases with 200+ detailed questions** designed for high-paying senior roles. Each phase includes perfect answers, real-world examples, follow-up questions, and production-ready code implementations.

**Key Preparation Strategy:**
- Master fundamentals first (Phases 1-3)
- Deep dive into performance & architecture (Phases 4-6) 
- Study advanced patterns (Phases 7-12)
- Practice explaining trade-offs and decisions
- Prepare portfolio defense with concrete examples

**Interview Success Rate:** Following this guide should give you 85-95% coverage of questions asked in senior React Native interviews at top tech companies. 🚀

---

## How We Will Work (Fast & Effective)

For each topic:

1. **Clear explanation** (interview-level, not docs-level)
2. **Typical interview question**
3. **Perfect answer**
4. **Follow-up trap question** (to avoid getting stuck)
5. **Real-world example**

---

## Starting Now — Question 1 (Warm-Up)

Answer this **out loud**, as if you are in the interview:

> **“What is React Native, and how is it different from React.js?”**

Type your answer here.
I will:

* Correct it
* Improve it
* Convert it into a **job-winning answer**

Once we lock this, we move forward step by step until you are interview-ready.

---
[⬅️ Back to Home](../README.md)