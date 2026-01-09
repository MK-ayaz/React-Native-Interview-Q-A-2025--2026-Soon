[⬅️ Back to Home](../README.md)

# 🎯 Phase 10: Security, Privacy & Best Practices

---

primary: '#0A84FF',
    background: {
      primary: '#000000',
      secondary: '#1C1C1E',
      tertiary: '#2C2C2E',
    },
    text: {
      primary: '#FFFFFF',
      secondary: '#EBEBF5',
      tertiary: '#8E8E93',
      inverse: '#000000',
    },
    border: {
      light: '#38383A',
      medium: '#48484A',
      dark: '#8E8E93',
    },
  },
};

// context/ThemeContext.tsx
interface ThemeContextType {
  theme: Theme;
  isDark: boolean;
  toggleTheme: () => void;
  setTheme: (theme: 'light' | 'dark') => void;
}

const ThemeContext = React.createContext<ThemeContextType | undefined>(undefined);

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

interface ThemeProviderProps {
  children: React.ReactNode;
  initialTheme?: 'light' | 'dark';
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({
  children,
  initialTheme = 'light'
}) => {
  const [themeName, setThemeName] = useState<'light' | 'dark'>(initialTheme);

  const theme = useMemo(() => {
    return themeName === 'dark' ? darkTheme : lightTheme;
  }, [themeName]);

  const toggleTheme = useCallback(() => {
    setThemeName(prev => prev === 'light' ? 'dark' : 'light');
  }, []);

  const setTheme = useCallback((newTheme: 'light' | 'dark') => {
    setThemeName(newTheme);
  }, []);

  const contextValue = useMemo(() => ({
    theme,
    isDark: themeName === 'dark',
    toggleTheme,
    setTheme,
  }), [theme, themeName, toggleTheme, setTheme]);

  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  );
};

// utils/styled.ts
type StyleFunction<T> = (theme: Theme, props?: T) => any;

export const createStyles = <T = {}>(styleFunction: StyleFunction<T>) => {
  return (props?: T) => {
    const { theme } = useTheme();
    return styleFunction(theme, props);
  };
};

export const styled = <P extends {}>(
  Component: React.ComponentType<P>
) => {
  return React.forwardRef<any, P>((props, ref) => {
    const { theme } = useTheme();
    return <Component ref={ref} theme={theme} {...props} />;
  });
};

// components/Button.tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
  onPress: () => void;
  style?: any;
}

const ButtonComponent: React.FC<ButtonProps & { theme: Theme }> = ({
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  children,
  onPress,
  theme,
  style,
  ...props
}) => {
  const buttonStyle = useMemo(() => {
    const baseStyle = {
      borderRadius: theme.borderRadius.md,
      alignItems: 'center',
      justifyContent: 'center',
      ...theme.shadows.sm,
    };

    const sizeStyles = {
      sm: {
        paddingHorizontal: theme.spacing.sm,
        paddingVertical: theme.spacing.xs,
        minHeight: 36,
      },
      md: {
        paddingHorizontal: theme.spacing.md,
        paddingVertical: theme.spacing.sm,
        minHeight: 44,
      },
      lg: {
        paddingHorizontal: theme.spacing.lg,
        paddingVertical: theme.spacing.md,
        minHeight: 52,
      },
    };

    const variantStyles = {
      primary: {
        backgroundColor: theme.colors.primary,
      },
      secondary: {
        backgroundColor: theme.colors.secondary,
      },
      outline: {
        backgroundColor: 'transparent',
        borderWidth: 1,
        borderColor: theme.colors.primary,
      },
      ghost: {
        backgroundColor: 'transparent',
      },
    };

    return [
      baseStyle,
      sizeStyles[size],
      variantStyles[variant],
      disabled && { opacity: 0.5 },
      style,
    ];
  }, [variant, size, disabled, theme, style]);

  const textStyle = useMemo(() => {
    const sizeStyles = {
      sm: { fontSize: theme.typography.fontSize.sm },
      md: { fontSize: theme.typography.fontSize.md },
      lg: { fontSize: theme.typography.fontSize.lg },
    };

    const variantStyles = {
      primary: { color: '#FFFFFF' },
      secondary: { color: '#FFFFFF' },
      outline: { color: theme.colors.primary },
      ghost: { color: theme.colors.primary },
    };

    return [
      sizeStyles[size],
      variantStyles[variant],
      { fontWeight: theme.typography.fontWeight.medium },
    ];
  }, [variant, size, theme]);

  const handlePress = useCallback(() => {
    if (!disabled && !loading) {
      onPress();
    }
  }, [disabled, loading, onPress]);

  return (
    <TouchableOpacity
      style={buttonStyle}
      onPress={handlePress}
      disabled={disabled || loading}
      {...props}
    >
      {loading ? (
        <ActivityIndicator color={textStyle.color} />
      ) : (
        <Text style={textStyle}>{children}</Text>
      )}
    </TouchableOpacity>
  );
};

export const Button = styled(ButtonComponent);

// components/Card.tsx
interface CardProps {
  variant?: 'elevated' | 'outlined' | 'filled';
  padding?: keyof Theme['spacing'];
  children: React.ReactNode;
  style?: any;
}

const CardComponent: React.FC<CardProps & { theme: Theme }> = ({
  variant = 'elevated',
  padding = 'md',
  children,
  theme,
  style,
  ...props
}) => {
  const cardStyle = useMemo(() => {
    const baseStyle = {
      borderRadius: theme.borderRadius.lg,
      padding: theme.spacing[padding],
    };

    const variantStyles = {
      elevated: {
        backgroundColor: theme.colors.background.primary,
        ...theme.shadows.md,
      },
      outlined: {
        backgroundColor: theme.colors.background.primary,
        borderWidth: 1,
        borderColor: theme.colors.border.light,
      },
      filled: {
        backgroundColor: theme.colors.background.secondary,
      },
    };

    return [baseStyle, variantStyles[variant], style];
  }, [variant, padding, theme, style]);

  return (
    <View style={cardStyle} {...props}>
      {children}
    </View>
  );
};

export const Card = styled(CardComponent);

// hooks/useComponentStyles.ts
export const useComponentStyles = <T extends Record<string, any>>(
  styleConfig: (theme: Theme) => T
): T => {
  const { theme } = useTheme();
  return useMemo(() => styleConfig(theme), [theme]);
};

// Example usage with hook-based styling
const useButtonStyles = (variant: ButtonProps['variant'] = 'primary', size: ButtonProps['size'] = 'md') => {
  return useComponentStyles(theme => ({
    container: {
      borderRadius: theme.borderRadius.md,
      paddingHorizontal: theme.spacing[size === 'sm' ? 'sm' : size === 'lg' ? 'lg' : 'md'],
      paddingVertical: theme.spacing.xs,
      minHeight: size === 'sm' ? 36 : size === 'lg' ? 52 : 44,
      backgroundColor: variant === 'primary' ? theme.colors.primary :
                      variant === 'secondary' ? theme.colors.secondary :
                      'transparent',
      borderWidth: variant === 'outline' ? 1 : 0,
      borderColor: theme.colors.primary,
      ...theme.shadows.sm,
    },
    text: {
      color: variant === 'primary' || variant === 'secondary' ? '#FFFFFF' : theme.colors.primary,
      fontSize: theme.typography.fontSize[size === 'sm' ? 'sm' : size === 'lg' ? 'xl' : 'md'],
      fontWeight: theme.typography.fontWeight.medium,
      textAlign: 'center',
    },
  }));
};

// Component variants using render props
interface ButtonVariantsProps {
  primary: React.ComponentProps<typeof Button>;
  secondary: React.ComponentProps<typeof Button>;
  outline: React.ComponentProps<typeof Button>;
}

const ButtonVariants: React.FC<ButtonVariantsProps> = ({
  primary,
  secondary,
  outline,
}) => (
  <View>
    <Button {...primary} />
    <Button variant="secondary" {...secondary} />
    <Button variant="outline" {...outline} />
  </View>
);

// Theme-aware component composition
const ThemedScreen = () => {
  const { isDark, toggleTheme } = useTheme();

  return (
    <View style={{ flex: 1, backgroundColor: 'background.primary' }}>
      <Card variant="elevated" padding="lg">
        <Text style={{ color: 'text.primary' }}>
          Welcome to the app!
        </Text>
        <Button onPress={toggleTheme}>
          Switch to {isDark ? 'Light' : 'Dark'} Theme
        </Button>
      </Card>
    </View>
  );
};
```

---

### 7.3 **Custom Component Patterns and Composition**

**Question:** *"Implement advanced component composition patterns including compound components, render props, and higher-order components for a flexible UI library."*

**Perfect Answer Structure:**
```
COMPONENT COMPOSITION PATTERNS:

🎯 COMPOUND COMPONENTS:
- Related components that work together
- Shared state through context
- Flexible API design
- Type-safe composition

🎯 RENDER PROPS PATTERN:
- Component logic reuse
- Cross-cutting concerns
- Conditional rendering logic
- Performance optimizations

🎯 HIGHER-ORDER COMPONENTS:
- Component enhancement
- Cross-cutting logic injection
- Decorator pattern implementation
- Reusability maximization
```

**Complete Component Composition Implementation:**
```typescript
// Compound Components Pattern
interface TabsContextType {
  activeTab: string;
  setActiveTab: (id: string) => void;
}

const TabsContext = React.createContext<TabsContextType | undefined>(undefined);

const useTabs = () => {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('Tabs compound components must be used within Tabs');
  }
  return context;
};

interface TabsProps {
  defaultActiveTab?: string;
  activeTab?: string;
  onChange?: (tabId: string) => void;
  children: React.ReactNode;
}

const Tabs: React.FC<TabsProps> & {
  Tab: typeof Tab;
  TabPanel: typeof TabPanel;
  TabList: typeof TabList;
} = ({ defaultActiveTab, activeTab: controlledActiveTab, onChange, children }) => {
  const [internalActiveTab, setInternalActiveTab] = useState(defaultActiveTab || '');
  const activeTab = controlledActiveTab ?? internalActiveTab;

  const setActiveTab = useCallback((tabId: string) => {
    if (controlledActiveTab === undefined) {
      setInternalActiveTab(tabId);
    }
    onChange?.(tabId);
  }, [controlledActiveTab, onChange]);

  const contextValue = useMemo(() => ({
    activeTab,
    setActiveTab,
  }), [activeTab, setActiveTab]);

  return (
    <TabsContext.Provider value={contextValue}>
      {children}
    </TabsContext.Provider>
  );
};

interface TabProps {
  id: string;
  children: React.ReactNode;
  disabled?: boolean;
}

const Tab: React.FC<TabProps> = ({ id, children, disabled }) => {
  const { activeTab, setActiveTab } = useTabs();
  const isActive = activeTab === id;

  const handlePress = useCallback(() => {
    if (!disabled) {
      setActiveTab(id);
    }
  }, [id, disabled, setActiveTab]);

  return (
    <TouchableOpacity
      onPress={handlePress}
      disabled={disabled}
      style={{
        padding: 12,
        borderBottomWidth: 2,
        borderBottomColor: isActive ? '#007AFF' : 'transparent',
        opacity: disabled ? 0.5 : 1,
      }}
    >
      <Text style={{
        color: isActive ? '#007AFF' : '#666',
        fontWeight: isActive ? 'bold' : 'normal',
      }}>
        {children}
      </Text>
    </TouchableOpacity>
  );
};

interface TabPanelProps {
  tabId: string;
  children: React.ReactNode;
}

const TabPanel: React.FC<TabPanelProps> = ({ tabId, children }) => {
  const { activeTab } = useTabs();

  if (activeTab !== tabId) {
    return null;
  }

  return <View>{children}</View>;
};

interface TabListProps {
  children: React.ReactNode;
}

const TabList: React.FC<TabListProps> = ({ children }) => (
  <View style={{ flexDirection: 'row', borderBottomWidth: 1, borderBottomColor: '#eee' }}>
    {children}
  </View>
);

// Attach compound components
Tabs.Tab = Tab;
Tabs.TabPanel = TabPanel;
Tabs.TabList = TabList;

// Render Props Pattern
interface DataFetcherProps<T> {
  url: string;
  children: (data: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
  initialData?: T;
}

function DataFetcher<T = any>({
  url,
  children,
  initialData = null
}: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(initialData);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  const renderProps = useMemo(() => ({
    data,
    loading,
    error,
    refetch: fetchData,
  }), [data, loading, error, fetchData]);

  return <>{children(renderProps)}</>;
}

// Higher-Order Components
function withLoadingIndicator<P extends object>(
  Component: React.ComponentType<P>
) {
  return React.forwardRef<any, P & { loading?: boolean }>((props, ref) => {
    const { loading, ...restProps } = props;

    if (loading) {
      return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          <ActivityIndicator size="large" color="#007AFF" />
        </View>
      );
    }

    return <Component ref={ref} {...(restProps as P)} />;
  });
}

function withErrorBoundary<P extends object>(
  Component: React.ComponentType<P>,
  fallback?: React.ComponentType<{ error: Error; retry: () => void }>
) {
  return class extends React.Component<P, { hasError: boolean; error: Error | null }> {
    constructor(props: P) {
      super(props);
      this.state = { hasError: false, error: null };
    }

    static getDerivedStateFromError(error: Error) {
      return { hasError: true, error };
    }

    componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
      console.error('Error caught by boundary:', error, errorInfo);
    }

    retry = () => {
      this.setState({ hasError: false, error: null });
    };

    render() {
      if (this.state.hasError) {
        if (fallback) {
          const FallbackComponent = fallback;
          return <FallbackComponent error={this.state.error!} retry={this.retry} />;
        }

        return (
          <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 }}>
            <Text style={{ fontSize: 18, marginBottom: 10 }}>Something went wrong</Text>
            <Text style={{ color: '#666', textAlign: 'center', marginBottom: 20 }}>
              {this.state.error?.message}
            </Text>
            <Button title="Try Again" onPress={this.retry} />
          </View>
        );
      }

      return <Component {...this.props} />;
    }
  };
}

function withTheme<P extends object>(
  Component: React.ComponentType<P>
) {
  return React.forwardRef<any, P>((props, ref) => {
    const { theme } = useTheme();
    return <Component ref={ref} theme={theme} {...props} />;
  });
}

function withDataSubscription<P extends object>(
  Component: React.ComponentType<P & { data: any; loading: boolean; error: Error | null }>,
  dataSelector: (state: any) => any
) {
  return (props: P) => {
    const data = useSelector(dataSelector);
    const dispatch = useDispatch();

    useEffect(() => {
      dispatch(fetchDataIfNeeded());
    }, [dispatch]);

    const { data: selectedData, loading, error } = data;

    return (
      <Component
        {...props}
        data={selectedData}
        loading={loading}
        error={error}
      />
    );
  };
}

// Hook-based composition utilities
function useControllableState<T>(
  value?: T,
  defaultValue?: T,
  onChange?: (value: T) => void
): [T, (value: T) => void] {
  const [internalValue, setInternalValue] = useState(defaultValue);
  const isControlled = value !== undefined;
  const currentValue = isControlled ? value : internalValue;

  const setValue = useCallback((newValue: T) => {
    if (!isControlled) {
      setInternalValue(newValue);
    }
    onChange?.(newValue);
  }, [isControlled, onChange]);

  return [currentValue, setValue];
}

function useCompoundComponent<T>(
  defaultValue?: T
): [T, (value: T) => void, (component: React.ReactElement) => React.ReactElement] {
  const [value, setValue] = useState(defaultValue);
  const [components, setComponents] = useState<React.ReactElement[]>([]);

  const registerComponent = useCallback((component: React.ReactElement) => {
    setComponents(prev => [...prev, component]);
    return component;
  }, []);

  return [value, setValue, registerComponent];
}

// Advanced composition with slots
interface SlottableComponentProps {
  children?: React.ReactNode;
  asChild?: boolean;
}

function Slot({ children, ...props }: SlottableComponentProps & any) {
  if (React.isValidElement(children)) {
    return React.cloneElement(children, { ...props, ...children.props });
  }
  return null;
}

interface SelectProps {
  value?: string;
  onValueChange?: (value: string) => void;
  children: React.ReactNode;
}

const Select: React.FC<SelectProps> & {
  Trigger: typeof SelectTrigger;
  Content: typeof SelectContent;
  Item: typeof SelectItem;
} = ({ value, onValueChange, children }) => {
  const [internalValue, setInternalValue] = useControllableState(value, '', onValueChange);
  const [isOpen, setIsOpen] = useState(false);

  const contextValue = useMemo(() => ({
    value: internalValue,
    onValueChange: setInternalValue,
    isOpen,
    setIsOpen,
  }), [internalValue, setInternalValue, isOpen]);

  return (
    <SelectContext.Provider value={contextValue}>
      {children}
    </SelectContext.Provider>
  );
};

const SelectTrigger = React.forwardRef<View, SlottableComponentProps>((props, ref) => {
  const { isOpen, setIsOpen } = useSelect();

  return (
    <Slot
      ref={ref}
      onPress={() => setIsOpen(!isOpen)}
      {...props}
    >
      <View style={{ borderWidth: 1, padding: 12 }}>
        <Text>Select an option...</Text>
      </View>
    </Slot>
  );
});

const SelectContent = ({ children }: { children: React.ReactNode }) => {
  const { isOpen } = useSelect();

  if (!isOpen) return null;

  return (
    <View style={{
      position: 'absolute',
      top: '100%',
      left: 0,
      right: 0,
      backgroundColor: 'white',
      borderWidth: 1,
      zIndex: 1000,
    }}>
      {children}
    </View>
  );
};

const SelectItem = ({ value, children }: { value: string; children: React.ReactNode }) => {
  const { onValueChange, setIsOpen } = useSelect();

  const handlePress = useCallback(() => {
    onValueChange(value);
    setIsOpen(false);
  }, [value, onValueChange, setIsOpen]);

  return (
    <TouchableOpacity onPress={handlePress} style={{ padding: 12 }}>
      <Text>{children}</Text>
    </TouchableOpacity>
  );
};

// Attach compound components
Select.Trigger = SelectTrigger;
Select.Content = SelectContent;
Select.Item = SelectItem;

// Usage examples
const ExampleUsage = () => {
  return (
    // Compound Components
    <Tabs defaultActiveTab="tab1">
      <Tabs.TabList>
        <Tabs.Tab id="tab1">Tab 1</Tabs.Tab>
        <Tabs.Tab id="tab2">Tab 2</Tabs.Tab>
      </Tabs.TabList>

      <Tabs.TabPanel tabId="tab1">
        <Text>Content 1</Text>
      </Tabs.TabPanel>

      <Tabs.TabPanel tabId="tab2">
        <Text>Content 2</Text>
      </Tabs.TabPanel>
    </Tabs>

    // Render Props
    <DataFetcher url="/api/users">
      {({ data, loading, error, refetch }) => {
        if (loading) return <ActivityIndicator />;
        if (error) return <Text>Error: {error.message}</Text>;

        return (
          <FlatList
            data={data}
            renderItem={({ item }) => <Text>{item.name}</Text>}
            onRefresh={refetch}
            refreshing={loading}
          />
        );
      }}
    </DataFetcher>

    // HOC Usage
    <EnhancedComponent loading={false} />

    // Slot-based Composition
    <Select>
      <Select.Trigger asChild>
        <Button>Choose option</Button>
      </Select.Trigger>

      <Select.Content>
        <Select.Item value="option1">Option 1</Select.Item>
        <Select.Item value="option2">Option 2</Select.Item>
      </Select.Content>
    </Select>
  );
};
```

---

*Phase 7 continues with advanced styling techniques, accessibility implementation, and performance optimization for complex component trees. Each question includes production-ready patterns with TypeScript support and cross-platform compatibility.*

---

## 🎯 Phase 8 — State Management Patterns

**State management questions test your ability to choose and implement the right solution** - expect deep comparisons between Context API, Redux, Zustand, and MobX for different use cases.

### 8.1 **Context API vs Redux vs Zustand Decision Framework**

**Question:** *"Design a state management strategy comparing Context API, Redux, and Zustand for different application scales and complexity levels."*

**Perfect Answer Structure:**
```
STATE MANAGEMENT DECISION MATRIX:

🎯 CONTEXT API FOR:
- Small to medium apps (<10 components sharing state)
- UI-only state (themes, modals, form state)
- Simple state transitions
- Avoiding prop drilling
- Learning curve: Low

🎯 REDUX FOR:
- Large, complex apps (>20 components sharing state)
- Complex state interactions and side effects
- Time-travel debugging requirements
- Middleware-heavy applications
- Learning curve: High

🎯 ZUSTAND FOR:
- Medium to large apps
- Simpler API than Redux
- Less boilerplate than Redux
- TypeScript-friendly
- Learning curve: Medium
```

**Complete State Management Comparison:**
```javascript
// Context API Implementation
const AppContext = React.createContext();

const appReducer = (state, action) => {
  switch (action.type) {
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_USER':
      return { ...state, user: action.payload, loading: false };
    case 'SET_ERROR':
      return { ...state, error: action.payload, loading: false };
    case 'LOGOUT':
      return { ...state, user: null, loading: false, error: null };
    default:
      return state;
  }
};

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 11 ➡️](phase11-advanced-patterns.md)