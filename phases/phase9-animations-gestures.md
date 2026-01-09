[⬅️ Back to Home](../README.md)

# 🎯 Phase 09: Animations & Gestures

---

runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}

    steps:
    - name: Download Android build
      uses: actions/download-artifact@v3
      with:
        name: android-release
        path: android/app/build/outputs/apk/release/

    - name: Deploy to Google Play Internal
      uses: r0adkll/upload-google-play@v1
      with:
        serviceAccountJsonPlainText: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT }}
        packageName: com.myapp
        releaseFiles: android/app/build/outputs/apk/release/app-release.apk
        track: internal
        inAppUpdatePriority: 3
        userFraction: 0.1

    - name: Deploy to Firebase App Distribution
      uses: wzieba/Firebase-Distribution-Github-Action@v1
      with:
        appId: ${{ secrets.FIREBASE_APP_ID }}
        serviceCredentialsFileContent: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
        groups: beta-testers
        file: android/app/build/outputs/apk/release/app-release.apk

  deploy-ios:
    runs-on: macos-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}

    steps:
    - name: Download iOS build
      uses: actions/download-artifact@v3
      with:
        name: ios-release
        path: ios/build/

    - name: Deploy to TestFlight
      run: |
        xcrun altool --upload-app --type ios --file "ios/build/MyApp.ipa" --username "${{ secrets.APPLE_ID_USERNAME }}" --password "${{ secrets.APPLE_ID_PASSWORD }}"
      env:
        FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD: ${{ secrets.APPLE_APPLICATION_SPECIFIC_PASSWORD }}

    - name: Create GitHub release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: v${{ github.run_number }}
        release_name: Release v${{ github.run_number }}
        body: |
          ## Changes
          - Automated release from CI/CD pipeline
          - Includes latest features and bug fixes
        draft: false
        prerelease: false

# Fastlane configuration for advanced iOS deployment
# fastlane/Fastfile
platform :ios do
  desc "Deploy to TestFlight"
  lane :beta do
    # Increment build number
    increment_build_number(
      build_number: latest_testflight_build_number + 1,
      xcodeproj: "MyApp.xcodeproj"
    )

    # Build and upload to TestFlight
    build_app(
      scheme: "MyApp",
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          "com.myapp" => "MyApp AppStore"
        }
      }
    )

    upload_to_testflight(
      skip_waiting_for_build_processing: true,
      distribute_external: true,
      groups: ["Beta Testers"],
      changelog: "New beta release with latest features"
    )

    # Notify team
    slack(
      message: "New beta version uploaded to TestFlight!",
      channel: "#releases",
      success: true
    )
  end

  desc "Deploy to App Store"
  lane :release do
    # Ensure clean git status
    ensure_git_status_clean

    # Get latest version from package.json
    package = load_json(json_path: "../package.json")
    version = package["version"]

    # Create release branch
    sh "git checkout -b release/v#{version}"

    # Update version numbers
    increment_version_number(
      version_number: version,
      xcodeproj: "MyApp.xcodeproj"
    )

    # Build for App Store
    build_app(
      scheme: "MyApp",
      export_method: "app-store"
    )

    # Upload to App Store Connect
    deliver(
      submit_for_review: false,
      automatic_release: true,
      force: true,
      skip_metadata: false,
      skip_screenshots: false,
      submission_information: {
        add_id_info_limits_tracking: true,
        add_id_info_serves_ads: false,
        add_id_info_tracks_action: true,
        add_id_info_tracks_install: true,
        add_id_info_uses_idfa: false,
        content_rights_has_rights: true,
        content_rights_contains_third_party_content: true,
        export_compliance_platform: 'ios',
        export_compliance_compliance_required: false,
        export_compliance_encryption_updated: false,
        export_compliance_app_type: nil,
        export_compliance_uses_encryption: false,
        export_compliance_is_exempt: false,
        export_compliance_contains_third_party_cryptography: false,
        export_compliance_contains_proprietary_cryptography: false,
        export_compliance_available_on_french_store: true
      }
    )

    # Create git tag
    add_git_tag(tag: "v#{version}")
    push_git_tags

    # Create GitHub release
    github_release = set_github_release(
      repository_name: "myorg/myapp",
      api_token: ENV["GITHUB_TOKEN"],
      name: "Release v#{version}",
      tag_name: "v#{version}",
      description: "Production release v#{version}",
      commitish: "main"
    )

    # Notify team and stakeholders
    slack(
      message: "🚀 New version #{version} released to App Store!",
      channel: "#releases",
      success: true,
      payload: {
        "App Store URL" => "https://apps.apple.com/app/myapp",
        "Release Notes" => "See changelog for details"
      }
    )
  end
end

platform :android do
  desc "Deploy to Google Play"
  lane :beta do
    # Build Android bundle
    gradle(
      task: "bundle",
      build_type: "Release",
      project_dir: "android/"
    )

    # Upload to Google Play Beta
    upload_to_play_store(
      track: 'beta',
      rollout: '0.1', # 10% rollout
      json_key_data: ENV['GOOGLE_PLAY_JSON_KEY'],
      package_name: 'com.myapp',
      aab: 'android/app/build/outputs/bundle/release/app-release.aab'
    )
  end

  desc "Promote beta to production"
  lane :promote do
    # Promote from beta to production
    upload_to_play_store(
      track: 'production',
      track_promote_to: 'production',
      rollout: '0.05', # Start with 5% rollout
      json_key_data: ENV['GOOGLE_PLAY_JSON_KEY'],
      package_name: 'com.myapp'
    )
  end
end

# Bundle analysis script
# scripts/analyze-bundle.js
const { execSync } = require('child_process');
const fs = require('fs');

function analyzeBundle() {
  console.log('Analyzing bundle size...');

  try {
    // Generate bundle analysis
    execSync('npx react-native-bundle-visualizer --platform ios --dev false', {
      stdio: 'inherit'
    });

    // Read bundle stats
    const stats = JSON.parse(fs.readFileSync('bundle-stats.json', 'utf8'));

    const totalSize = stats.assets.reduce((sum, asset) => sum + asset.size, 0);
    const jsSize = stats.assets
      .filter(asset => asset.name.endsWith('.js'))
      .reduce((sum, asset) => sum + asset.size, 0);

    console.log(`Total bundle size: ${(totalSize / 1024 / 1024).toFixed(2)} MB`);
    console.log(`JS bundle size: ${(jsSize / 1024 / 1024).toFixed(2)} MB`);

    // Check against limits
    const MAX_BUNDLE_SIZE = 15 * 1024 * 1024; // 15MB
    if (totalSize > MAX_BUNDLE_SIZE) {
      console.error('❌ Bundle size exceeds limit!');
      process.exit(1);
    }

    console.log('✅ Bundle size within limits');

  } catch (error) {
    console.error('Bundle analysis failed:', error);
    process.exit(1);
  }
}

if (require.main === module) {
  analyzeBundle();
}

// Environment configuration
// config/environments.js
const environments = {
  development: {
    API_BASE_URL: 'http://localhost:3000/api',
    ENABLE_ANALYTICS: false,
    LOG_LEVEL: 'debug',
    BUNDLE_ID: 'com.myapp.dev',
  },
  staging: {
    API_BASE_URL: 'https://api-staging.myapp.com',
    ENABLE_ANALYTICS: true,
    LOG_LEVEL: 'info',
    BUNDLE_ID: 'com.myapp.staging',
  },
  production: {
    API_BASE_URL: 'https://api.myapp.com',
    ENABLE_ANALYTICS: true,
    LOG_LEVEL: 'warn',
    BUNDLE_ID: 'com.myapp',
  },
};

function getCurrentEnvironment() {
  // Check for environment variable
  const env = process.env.NODE_ENV || 'development';

  // Override for specific builds
  if (__DEV__) return 'development';

  return env;
}

export const config = environments[getCurrentEnvironment()];
export const isProduction = getCurrentEnvironment() === 'production';
export const isDevelopment = getCurrentEnvironment() === 'development';
```

---

*Phase 6 continues with advanced deployment strategies, rollback procedures, A/B testing frameworks, and production monitoring systems. Each question includes enterprise-grade implementation with security, scalability, and maintainability considerations.*

---

## 🎯 Phase 7 — Styling & UI Components System

**UI development questions test your design system expertise** - senior roles expect comprehensive knowledge of styling, responsive design, and component architecture.

### 7.1 **Advanced Flexbox Layouts and Responsive Design**

**Question:** *"Design a responsive card grid layout that adapts to different screen sizes and orientations using Flexbox and responsive utilities."*

**Perfect Answer Structure:**
```
RESPONSIVE DESIGN ARCHITECTURE:

🎯 FLEXBOX MASTER PATTERNS:
- Container-based layouts
- Dynamic flex ratios
- Cross-platform responsive units
- Orientation-aware layouts

🎯 RESPONSIVE DESIGN SYSTEM:
- Breakpoint management
- Fluid typography
- Adaptive spacing
- Device-specific optimizations

🎯 COMPONENT RESPONSIVENESS:
- Self-adjusting components
- Content-aware layouts
- Touch-friendly interactions
- Accessibility considerations
```

**Complete Responsive Design Implementation:**
```javascript
// utils/responsive.js
import { Dimensions, PixelRatio, Platform } from 'react-native';

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

// Breakpoint system (similar to CSS breakpoints)
export const breakpoints = {
  xs: 320,
  sm: 480,
  md: 768,
  lg: 1024,
  xl: 1280,
};

export const isOrientationLandscape = () => {
  const { width, height } = Dimensions.get('window');
  return width > height;
};

// Responsive size utilities
export const responsiveSize = (size) => {
  const scale = Math.min(SCREEN_WIDTH / 375, SCREEN_HEIGHT / 667); // Based on iPhone 6/7/8
  const newSize = size * scale;
  return Math.round(PixelRatio.roundToNearestPixel(newSize));
};

export const responsiveWidth = (percentage) => {
  return (SCREEN_WIDTH * percentage) / 100;
};

export const responsiveHeight = (percentage) => {
  return (SCREEN_HEIGHT * percentage) / 100;
};

// Typography scale
export const typography = {
  h1: responsiveSize(32),
  h2: responsiveSize(28),
  h3: responsiveSize(24),
  h4: responsiveSize(20),
  h5: responsiveSize(18),
  body: responsiveSize(16),
  caption: responsiveSize(14),
  small: responsiveSize(12),
};

// Spacing scale
export const spacing = {
  xs: responsiveSize(4),
  sm: responsiveSize(8),
  md: responsiveSize(16),
  lg: responsiveSize(24),
  xl: responsiveSize(32),
  xxl: responsiveSize(48),
};

// Border radius scale
export const borderRadius = {
  sm: responsiveSize(4),
  md: responsiveSize(8),
  lg: responsiveSize(12),
  xl: responsiveSize(16),
  round: responsiveSize(999),
};

// Shadow system (platform-aware)
export const shadows = {
  sm: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 1 },
      shadowOpacity: 0.1,
      shadowRadius: 2,
    },
    android: {
      elevation: 2,
    },
  }),
  md: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.15,
      shadowRadius: 4,
    },
    android: {
      elevation: 4,
    },
  }),
  lg: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 4 },
      shadowOpacity: 0.2,
      shadowRadius: 8,
    },
    android: {
      elevation: 8,
    },
  }),
};

// Responsive hook
export const useResponsive = () => {
  const [dimensions, setDimensions] = useState(Dimensions.get('window'));

  useEffect(() => {
    const subscription = Dimensions.addEventListener('change', ({ window }) => {
      setDimensions(window);
    });

    return () => subscription?.remove();
  }, []);

  const isSmall = dimensions.width < breakpoints.sm;
  const isMedium = dimensions.width >= breakpoints.sm && dimensions.width < breakpoints.md;
  const isLarge = dimensions.width >= breakpoints.md;

  return {
    width: dimensions.width,
    height: dimensions.height,
    isSmall,
    isMedium,
    isLarge,
    isLandscape: dimensions.width > dimensions.height,
    responsiveSize,
    responsiveWidth: (percentage) => (dimensions.width * percentage) / 100,
    responsiveHeight: (percentage) => (dimensions.height * percentage) / 100,
  };
};

// Responsive container component
const ResponsiveContainer = ({ children, style, ...props }) => {
  const { isSmall, isMedium, isLarge } = useResponsive();

  const containerStyle = useMemo(() => [
    {
      paddingHorizontal: isSmall ? spacing.sm : isMedium ? spacing.md : spacing.lg,
      paddingVertical: isSmall ? spacing.sm : spacing.md,
    },
    style,
  ], [isSmall, isMedium, isLarge, style]);

  return (
    <View style={containerStyle} {...props}>
      {children}
    </View>
  );
};

// Advanced card grid layout
const ResponsiveCardGrid = ({
  data,
  numColumns = { xs: 1, sm: 2, md: 3, lg: 4 },
  cardHeight = 200,
  gap = spacing.md,
  ...props
}) => {
  const { isSmall, isMedium, isLarge } = useResponsive();

  const columns = useMemo(() => {
    if (isSmall) return numColumns.xs || 1;
    if (isMedium) return numColumns.sm || 2;
    if (isLarge) return numColumns.md || 3;
    return numColumns.lg || 4;
  }, [isSmall, isMedium, isLarge, numColumns]);

  const cardWidth = useMemo(() => {
    const totalGap = gap * (columns - 1);
    const availableWidth = responsiveWidth(100) - (totalGap / columns);
    return (availableWidth / columns) - gap;
  }, [columns, gap]);

  const renderItem = useCallback(({ item, index }) => (
    <View
      style={{
        width: cardWidth,
        height: cardHeight,
        marginRight: (index + 1) % columns !== 0 ? gap : 0,
        marginBottom: gap,
      }}
    >
      <Card item={item} />
    </View>
  ), [cardWidth, cardHeight, columns, gap]);

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
      numColumns={columns}
      contentContainerStyle={{
        padding: gap,
      }}
      showsVerticalScrollIndicator={false}
      {...props}
    />
  );
};

// Complex responsive form layout
const ResponsiveForm = ({ children, orientation = 'vertical' }) => {
  const { isLandscape, isSmall } = useResponsive();

  const formStyle = useMemo(() => ({
    flexDirection: (isLandscape && !isSmall && orientation === 'adaptive')
      ? 'row'
      : 'column',
    alignItems: 'stretch',
  }), [isLandscape, isSmall, orientation]);

  const fieldStyle = useMemo(() => ({
    marginBottom: formStyle.flexDirection === 'column' ? spacing.md : 0,
    marginRight: formStyle.flexDirection === 'row' ? spacing.md : 0,
    flex: formStyle.flexDirection === 'row' ? 1 : undefined,
  }), [formStyle.flexDirection]);

  return (
    <View style={formStyle}>
      {React.Children.map(children, (child, index) => (
        <View key={index} style={fieldStyle}>
          {child}
        </View>
      ))}
    </View>
  );
};

// Adaptive navigation bar
const AdaptiveNavBar = ({ title, leftComponent, rightComponent }) => {
  const { isSmall, isLandscape } = useResponsive();

  const navStyle = useMemo(() => ({
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingHorizontal: isSmall ? spacing.sm : spacing.md,
    paddingVertical: spacing.sm,
    minHeight: responsiveSize(56),
    backgroundColor: '#fff',
    ...shadows.sm,
  }), [isSmall]);

  const titleStyle = useMemo(() => ({
    fontSize: isSmall ? typography.h5 : typography.h4,
    fontWeight: 'bold',
    flex: 1,
    textAlign: isLandscape ? 'center' : 'left',
    marginHorizontal: spacing.sm,
  }), [isSmall, isLandscape]);

  return (
    <View style={navStyle}>
      <View style={{ minWidth: responsiveSize(40) }}>
        {leftComponent}
      </View>

      <Text style={titleStyle} numberOfLines={1}>
        {title}
      </Text>

      <View style={{ minWidth: responsiveSize(40), alignItems: 'flex-end' }}>
        {rightComponent}
      </View>
    </View>
  );
};

// Fluid typography hook
export const useFluidTypography = (minSize, maxSize, minWidth = 320, maxWidth = 1280) => {
  const { width } = useResponsive();

  const fontSize = useMemo(() => {
    if (width <= minWidth) return minSize;
    if (width >= maxWidth) return maxSize;

    const slope = (maxSize - minSize) / (maxWidth - minWidth);
    const size = minSize + slope * (width - minWidth);

    return Math.round(size);
  }, [width, minSize, maxSize, minWidth, maxWidth]);

  return fontSize;
};

// Usage examples
const ExampleScreen = () => {
  const fluidTitleSize = useFluidTypography(24, 48);

  return (
    <ResponsiveContainer>
      <AdaptiveNavBar
        title="Dashboard"
        leftComponent={<MenuButton />}
        rightComponent={<ProfileButton />}
      />

      <Text style={{ fontSize: fluidTitleSize, marginBottom: spacing.lg }}>
        Welcome Back!
      </Text>

      <ResponsiveCardGrid
        data={dashboardCards}
        numColumns={{ xs: 1, sm: 2, md: 3 }}
        gap={spacing.md}
      />

      <ResponsiveForm orientation="adaptive">
        <TextInput placeholder="First Name" />
        <TextInput placeholder="Last Name" />
        <TextInput placeholder="Email" style={{ flex: 2 }} />
      </ResponsiveForm>
    </ResponsiveContainer>
  );
};
```

---

### 7.2 **Design System and Component Library Architecture**

**Question:** *"Architect a scalable design system with theming, component variants, and TypeScript support for a large React Native application."*

**Perfect Answer Structure:**
```
DESIGN SYSTEM ARCHITECTURE:

🎯 THEME MANAGEMENT:
- Centralized theme configuration
- Runtime theme switching
- Platform-specific adaptations
- Accessibility-compliant colors

🎯 COMPONENT VARIANTS:
- Size-based variants (sm, md, lg)
- State-based variants (default, pressed, disabled)
- Context-aware styling
- Composition over configuration

🎯 TYPE-SAFE STYLING:
- TypeScript interfaces for themes
- Styled component utilities
- Style composition helpers
- IntelliSense support
```

**Complete Design System Implementation:**
```typescript
// types/theme.ts
export interface ColorPalette {
  primary: string;
  secondary: string;
  success: string;
  warning: string;
  error: string;
  info: string;
  background: {
    primary: string;
    secondary: string;
    tertiary: string;
  };
  text: {
    primary: string;
    secondary: string;
    tertiary: string;
    inverse: string;
  };
  border: {
    light: string;
    medium: string;
    dark: string;
  };
}

export interface TypographyScale {
  fontSize: {
    xs: number;
    sm: number;
    md: number;
    lg: number;
    xl: number;
    xxl: number;
  };
  fontWeight: {
    light: '300';
    regular: '400';
    medium: '500';
    semibold: '600';
    bold: '700';
  };
  lineHeight: {
    tight: 1.25;
    normal: 1.5;
    relaxed: 1.75;
  };
}

export interface SpacingScale {
  xs: number;
  sm: number;
  md: number;
  lg: number;
  xl: number;
  xxl: number;
}

export interface BorderRadiusScale {
  none: number;
  sm: number;
  md: number;
  lg: number;
  xl: number;
  full: number;
}

export interface ShadowScale {
  none: {};
  sm: any;
  md: any;
  lg: any;
  xl: any;
}

export interface Theme {
  colors: ColorPalette;
  typography: TypographyScale;
  spacing: SpacingScale;
  borderRadius: BorderRadiusScale;
  shadows: ShadowScale;
  breakpoints: {
    xs: number;
    sm: number;
    md: number;
    lg: number;
    xl: number;
  };
}

// themes/light.ts
export const lightTheme: Theme = {
  colors: {
    primary: '#007AFF',
    secondary: '#5856D6',
    success: '#34C759',
    warning: '#FF9500',
    error: '#FF3B30',
    info: '#5AC8FA',
    background: {
      primary: '#FFFFFF',
      secondary: '#F2F2F7',
      tertiary: '#FFFFFF',
    },
    text: {
      primary: '#000000',
      secondary: '#3C3C43',
      tertiary: '#8E8E93',
      inverse: '#FFFFFF',
    },
    border: {
      light: '#C6C6C8',
      medium: '#D1D1D6',
      dark: '#8E8E93',
    },
  },
  typography: {
    fontSize: {
      xs: 12,
      sm: 14,
      md: 16,
      lg: 18,
      xl: 20,
      xxl: 24,
    },
    fontWeight: {
      light: '300',
      regular: '400',
      medium: '500',
      semibold: '600',
      bold: '700',
    },
    lineHeight: {
      tight: 1.25,
      normal: 1.5,
      relaxed: 1.75,
    },
  },
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
    xxl: 48,
  },
  borderRadius: {
    none: 0,
    sm: 4,
    md: 8,
    lg: 12,
    xl: 16,
    full: 999,
  },
  shadows: {
    none: {},
    sm: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 1 },
      shadowOpacity: 0.1,
      shadowRadius: 2,
      elevation: 2,
    },
    md: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.15,
      shadowRadius: 4,
      elevation: 4,
    },
    lg: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 4 },
      shadowOpacity: 0.2,
      shadowRadius: 8,
      elevation: 8,
    },
    xl: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 8 },
      shadowOpacity: 0.25,
      shadowRadius: 16,
      elevation: 16,
    },
  },
  breakpoints: {
    xs: 320,
    sm: 480,
    md: 768,
    lg: 1024,
    xl: 1280,
  },
};

// themes/dark.ts
export const darkTheme: Theme = {
  ...lightTheme,
  colors: {
    ...lightTheme.colors,

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 10 ➡️](phase10-security-best-practices.md)