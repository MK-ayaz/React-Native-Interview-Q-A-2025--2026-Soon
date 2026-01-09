[⬅️ Back to Home](../README.md)

# 🎯 Phase 05: Native Modules & Device APIs

---

};

  return (
    <View style={containerStyle}>
      <FlatList
        data={filteredUsers}
        renderItem={({ item }) => (
          <TouchableOpacity
            style={{ padding: 12, marginVertical: 4 }}
            onPress={() => handleUserPress(item)} // New function per item!
          >
            <Text style={{ fontSize: 16, fontWeight: 'bold' }}>
              {item.name}
            </Text>
            <Text style={{ fontSize: 14, color: '#666' }}>
              {item.email}
            </Text>
          </TouchableOpacity>
        )}
        keyExtractor={(item) => item.id} // Missing optimization
      />
    </View>
  );
};

// AFTER: Fully optimized component
const UserList = React.memo(({
  users,
  onUserSelect,
  searchQuery,
  theme
}) => {
  console.log('UserList rendering...');

  // Memoize expensive filtering operation
  const filteredUsers = useMemo(() => {
    console.log('Filtering users...');
    return users.filter(user =>
      user.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [users, searchQuery]);

  // Memoize callback to prevent child re-renders
  const handleUserPress = useCallback((user) => {
    console.log('User pressed:', user.name);
    onUserSelect(user);
  }, [onUserSelect]);

  // Memoize style object
  const containerStyle = useMemo(() => ({
    backgroundColor: theme === 'dark' ? '#000' : '#fff',
    padding: 16,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  }), [theme]);

  // Memoize renderItem to prevent recreation
  const renderItem = useCallback(({ item }) => (
    <UserListItem
      user={item}
      onPress={handleUserPress}
      theme={theme}
    />
  ), [handleUserPress, theme]);

  return (
    <View style={containerStyle}>
      <FlatList
        data={filteredUsers}
        renderItem={renderItem}
        keyExtractor={(item) => item.id.toString()}
        getItemLayout={getItemLayout} // Performance optimization
        windowSize={5} // Render 5 screens worth of items
        maxToRenderPerBatch={10} // Batch rendering
        updateCellsBatchingPeriod={50} // Debounce updates
        initialNumToRender={10} // Initial render count
        removeClippedSubviews={true} // Memory optimization
      />
    </View>
  );
});

// Extract item component for better memoization
const UserListItem = React.memo(({ user, onPress, theme }) => {
  const textColor = theme === 'dark' ? '#fff' : '#000';
  const subTextColor = theme === 'dark' ? '#ccc' : '#666';

  return (
    <TouchableOpacity
      style={{ padding: 12, marginVertical: 4 }}
      onPress={() => onPress(user)}
    >
      <Text style={{ fontSize: 16, fontWeight: 'bold', color: textColor }}>
        {user.name}
      </Text>
      <Text style={{ fontSize: 14, color: subTextColor }}>
        {user.email}
      </Text>
    </TouchableOpacity>
  );
});

// Performance helper for FlatList
const getItemLayout = (data, index) => ({
  length: 60, // Item height
  offset: 60 * index, // Item offset
  index,
});

// Custom hook for debounced search
const useDebouncedSearch = (query, delay = 300) => {
  const [debouncedQuery, setDebouncedQuery] = useState(query);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedQuery(query);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [query, delay]);

  return debouncedQuery;
};

// Usage with debounced search
const SearchableUserList = ({ users, onUserSelect, theme }) => {
  const [searchQuery, setSearchQuery] = useState('');
  const debouncedQuery = useDebouncedSearch(searchQuery);

  return (
    <View style={{ flex: 1 }}>
      <TextInput
        placeholder="Search users..."
        value={searchQuery}
        onChangeText={setSearchQuery}
        style={{ padding: 12, borderWidth: 1, margin: 16 }}
      />
      <UserList
        users={users}
        onUserSelect={onUserSelect}
        searchQuery={debouncedQuery} // Use debounced value
        theme={theme}
      />
    </View>
  );
};
```

**Performance Monitoring and Profiling:**
```javascript
// Performance monitoring hook
const usePerformanceMonitor = (componentName) => {
  const renderCountRef = useRef(0);
  const lastRenderTimeRef = useRef(Date.now());

  renderCountRef.current += 1;

  useEffect(() => {
    const now = Date.now();
    const renderTime = now - lastRenderTimeRef.current;
    lastRenderTimeRef.current = now;

    if (__DEV__) {
      console.log(`${componentName} rendered ${renderCountRef.current} times`);
      if (renderTime > 16) { // More than one frame
        console.warn(`${componentName} slow render: ${renderTime}ms`);
      }
    }

    // Report to analytics in production
    if (!__DEV__ && renderTime > 100) {
      analytics.track('slow_render', {
        component: componentName,
        renderTime,
        renderCount: renderCountRef.current,
      });
    }
  });

  return renderCountRef.current;
};

// Usage in component
const OptimizedComponent = () => {
  const renderCount = usePerformanceMonitor('OptimizedComponent');

  // Component logic...
};
```

---

### 4.2 **FlatList Advanced Optimization Techniques**

**Question:** *"Optimize a FlatList with 10,000+ items for smooth scrolling and memory efficiency."*

**Perfect Answer Structure:**
```
FLATLIST OPTIMIZATION STRATEGY:

🎯 RENDERING OPTIMIZATIONS:
- Memoized renderItem function
- Stable keyExtractor
- getItemLayout for performance
- removeClippedSubviews

🎯 MEMORY OPTIMIZATIONS:
- windowSize tuning
- maxToRenderPerBatch
- initialNumToRender
- RecyclerView-like behavior

🎯 SCROLLING OPTIMIZATIONS:
- onEndReachedThreshold
- maintainVisibleContentPosition
- scrollEventThrottle
- pagingEnabled for specific use cases
```

**Complete FlatList Optimization Implementation:**
```javascript
// Advanced FlatList optimization for large datasets
const OptimizedLargeList = ({ data, renderItem, onEndReached }) => {
  const flatListRef = useRef(null);

  // Memoize render function
  const memoizedRenderItem = useCallback(({ item, index }) => {
    // Pass index for alternating styles or other logic
    return renderItem({ item, index });
  }, [renderItem]);

  // Stable key extractor
  const keyExtractor = useCallback((item, index) => {
    return item.id?.toString() || index.toString();
  }, []);

  // Get item layout for better performance
  const getItemLayout = useCallback((data, index) => ({
    length: ITEM_HEIGHT, // Fixed height items
    offset: ITEM_HEIGHT * index,
    index,
  }), []);

  // Optimized scroll handler with throttling
  const onScroll = useCallback(
    throttle((event) => {
      const offsetY = event.nativeEvent.contentOffset.y;
      // Handle scroll-based logic (headers, etc.)
    }, 16), // ~60fps
    []
  );

  // End reached with buffer
  const handleEndReached = useCallback(() => {
    if (onEndReached && !isLoadingMore) {
      onEndReached();
    }
  }, [onEndReached, isLoadingMore]);

  return (
    <FlatList
      ref={flatListRef}
      data={data}
      renderItem={memoizedRenderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}

      // Memory optimizations
      removeClippedSubviews={true} // Unmount off-screen items
      windowSize={5} // Render 5 screens worth
      maxToRenderPerBatch={10} // Batch rendering
      updateCellsBatchingPeriod={50} // Debounce updates

      // Initial render optimization
      initialNumToRender={8} // Start with fewer items
      initialScrollIndex={undefined} // Avoid if not needed

      // Scrolling optimizations
      onScroll={onScroll}
      scrollEventThrottle={16} // ~60fps
      showsVerticalScrollIndicator={false}
      bounces={true}
      overScrollMode="never"

      // Pagination
      onEndReached={handleEndReached}
      onEndReachedThreshold={0.1} // Trigger earlier

      // Performance monitoring
      onScrollBeginDrag={() => {
        // Pause expensive operations during scroll
      }}
      onScrollEndDrag={() => {
        // Resume operations after scroll
      }}

      // Pull to refresh
      refreshing={refreshing}
      onRefresh={onRefresh}

      // Empty state
      ListEmptyComponent={<EmptyState />}
      ListFooterComponent={loading ? <LoadingIndicator /> : null}
    />
  );
};

// Sectioned list optimization for grouped data
const OptimizedSectionList = ({ sections, renderItem, renderSectionHeader }) => {
  const sectionListRef = useRef(null);

  // Memoize section rendering
  const memoizedRenderSectionHeader = useCallback(({ section }) => {
    return renderSectionHeader(section);
  }, [renderSectionHeader]);

  const memoizedRenderItem = useCallback(({ item, section, index }) => {
    return renderItem({ item, section, index });
  }, [renderItem]);

  return (
    <SectionList
      ref={sectionListRef}
      sections={sections}
      renderItem={memoizedRenderItem}
      renderSectionHeader={memoizedRenderSectionHeader}
      keyExtractor={(item, index) => item.id || index.toString()}

      // Performance optimizations
      removeClippedSubviews={true}
      windowSize={3}
      maxToRenderPerBatch={5}
      updateCellsBatchingPeriod={100}

      // Sticky headers
      stickySectionHeadersEnabled={true}

      // Section insets
      SectionSeparatorComponent={() => <View style={{ height: 8 }} />}
      ItemSeparatorComponent={() => <View style={{ height: 1, backgroundColor: '#eee' }} />}
    />
  );
};

// Virtualized list for extremely large datasets
const VirtualizedDataList = ({ data, itemHeight, renderItem }) => {
  const scrollViewRef = useRef(null);
  const [visibleRange, setVisibleRange] = useState({ start: 0, end: 20 });

  // Calculate visible items based on scroll position
  const handleScroll = useCallback((event) => {
    const offsetY = event.nativeEvent.contentOffset.y;
    const start = Math.floor(offsetY / itemHeight);
    const visibleCount = Math.ceil(Dimensions.get('window').height / itemHeight);
    const end = start + visibleCount + 5; // Buffer

    setVisibleRange({ start: Math.max(0, start), end: Math.min(data.length, end) });
  }, [itemHeight, data.length]);

  const visibleData = useMemo(() => {
    return data.slice(visibleRange.start, visibleRange.end);
  }, [data, visibleRange]);

  return (
    <ScrollView
      ref={scrollViewRef}
      onScroll={handleScroll}
      scrollEventThrottle={16}
      contentContainerStyle={{
        height: data.length * itemHeight,
        paddingTop: visibleRange.start * itemHeight
      }}
    >
      {visibleData.map((item, index) => (
        <View key={item.id} style={{ height: itemHeight }}>
          {renderItem({ item, index: visibleRange.start + index })}
        </View>
      ))}
    </ScrollView>
  );
};
```

**Performance Monitoring for Lists:**
```javascript
// List performance monitoring hook
const useListPerformanceMonitor = (listName) => {
  const [metrics, setMetrics] = useState({
    renderTime: 0,
    scrollEvents: 0,
    visibleItems: 0,
  });

  const measureRenderTime = useCallback((startTime) => {
    const renderTime = Date.now() - startTime;
    setMetrics(prev => ({ ...prev, renderTime }));

    if (__DEV__ && renderTime > 16) {
      console.warn(`${listName} slow render: ${renderTime}ms`);
    }
  }, [listName]);

  const trackScrollEvent = useCallback(() => {
    setMetrics(prev => ({ ...prev, scrollEvents: prev.scrollEvents + 1 }));
  }, []);

  const updateVisibleItems = useCallback((count) => {
    setMetrics(prev => ({ ...prev, visibleItems: count }));
  }, []);

  // Report metrics periodically
  useEffect(() => {
    const interval = setInterval(() => {
      if (!__DEV__) {
        analytics.track('list_performance', {
          listName,
          ...metrics,
        });
      }
    }, 30000); // Every 30 seconds

    return () => clearInterval(interval);
  }, [listName, metrics]);

  return {
    metrics,
    measureRenderTime,
    trackScrollEvent,
    updateVisibleItems,
  };
};
```

---

### 4.3 **Image Optimization and Memory Management**

**Question:** *"Implement a comprehensive image loading and caching strategy for optimal memory usage and performance."*

**Perfect Answer Structure:**
```
IMAGE OPTIMIZATION STRATEGY:

🎯 LOADING OPTIMIZATIONS:
- Progressive loading
- Lazy loading for off-screen images
- Placeholder and blur effects
- Network prioritization

🎯 MEMORY MANAGEMENT:
- Image caching strategies
- Memory limits and cleanup
- Resize and compression
- WebP format usage

🎯 PERFORMANCE PATTERNS:
- Preloading critical images
- Background prefetching
- Cache invalidation
- Error handling and fallbacks
```

**Complete Image Optimization Implementation:**
```javascript
// Advanced image component with optimization
const OptimizedImage = React.memo(({
  source,
  style,
  placeholder,
  blurRadius = 2,
  resizeMode = 'cover',
  priority = 'normal',
  ...props
}) => {
  const [imageState, setImageState] = useState({
    loading: true,
    error: false,
    aspectRatio: 1,
  });

  const imageRef = useRef(null);

  // Generate optimized image URL
  const optimizedSource = useMemo(() => {
    if (typeof source === 'number') return source; // Local image

    const uri = source.uri;
    if (!uri) return source;

    // Add optimization parameters
    const url = new URL(uri);
    url.searchParams.set('w', style?.width?.toString() || '300');
    url.searchParams.set('h', style?.height?.toString() || '300');
    url.searchParams.set('fit', 'crop');
    url.searchParams.set('auto', 'compress,format');
    url.searchParams.set('q', '80');

    return { uri: url.toString() };
  }, [source, style]);

  const handleLoadStart = useCallback(() => {
    setImageState(prev => ({ ...prev, loading: true, error: false }));
  }, []);

  const handleLoad = useCallback((event) => {
    const { width, height } = event.nativeEvent.source;
    setImageState({
      loading: false,
      error: false,
      aspectRatio: width / height,
    });
  }, []);

  const handleError = useCallback(() => {
    setImageState(prev => ({ ...prev, loading: false, error: true }));
  }, []);

  // Preload critical images
  useEffect(() => {
    if (priority === 'high') {
      Image.prefetch(typeof optimizedSource === 'object' ? optimizedSource.uri : '');
    }
  }, [optimizedSource, priority]);

  return (
    <View style={[style, { overflow: 'hidden' }]}>
      {/* Placeholder/Loading state */}
      {imageState.loading && placeholder && (
        <View style={[styles.placeholder, style]}>
          {placeholder}
        </View>
      )}

      {/* Main image */}
      <Image
        ref={imageRef}
        source={optimizedSource}
        style={[
          style,
          imageState.loading && { opacity: 0 },
        ]}
        resizeMode={resizeMode}
        blurRadius={imageState.loading ? blurRadius : 0}
        onLoadStart={handleLoadStart}
        onLoad={handleLoad}
        onError={handleError}
        {...props}
      />

      {/* Error state */}
      {imageState.error && (
        <View style={[styles.errorContainer, style]}>
          <Text style={styles.errorText}>Failed to load image</Text>
        </View>
      )}
    </View>
  );
});

// Image prefetching utility
class ImagePrefetcher {
  private static instance: ImagePrefetcher;
  private prefetchQueue: Set<string> = new Set();
  private isPrefetching = false;

  static getInstance(): ImagePrefetcher {
    if (!ImagePrefetcher.instance) {
      ImagePrefetcher.instance = new ImagePrefetcher();
    }
    return ImagePrefetcher.instance;
  }

  async prefetch(urls: string[]): Promise<void> {
    const uniqueUrls = urls.filter(url => !this.prefetchQueue.has(url));

    if (uniqueUrls.length === 0) return;

    uniqueUrls.forEach(url => this.prefetchQueue.add(url));

    if (this.isPrefetching) return;

    this.isPrefetching = true;

    try {
      await Promise.allSettled(
        uniqueUrls.map(url => Image.prefetch(url))
      );
    } finally {
      this.isPrefetching = false;
      uniqueUrls.forEach(url => this.prefetchQueue.delete(url));
    }
  }

  async prefetchPriority(urls: string[]): Promise<void> {
    // Higher priority prefetching
    await Promise.all(
      urls.map(url => Image.prefetch(url))
    );
  }
}

// Image cache management
class ImageCache {
  private static readonly MAX_CACHE_SIZE = 50 * 1024 * 1024; // 50MB
  private cacheSize = 0;
  private cache = new Map<string, { size: number; lastAccessed: number }>();

  add(uri: string, size: number): void {
    const now = Date.now();
    this.cache.set(uri, { size, lastAccessed: now });
    this.cacheSize += size;

    this.enforceCacheLimit();
  }

  get(uri: string): boolean {
    const entry = this.cache.get(uri);
    if (entry) {
      entry.lastAccessed = Date.now();
      return true;
    }
    return false;
  }

  private enforceCacheLimit(): void {
    if (this.cacheSize <= ImageCache.MAX_CACHE_SIZE) return;

    // Remove least recently used items
    const entries = Array.from(this.cache.entries())
      .sort((a, b) => a[1].lastAccessed - b[1].lastAccessed);

    for (const [uri, entry] of entries) {
      if (this.cacheSize <= ImageCache.MAX_CACHE_SIZE) break;

      this.cache.delete(uri);
      this.cacheSize -= entry.size;
    }
  }

  clear(): void {
    this.cache.clear();
    this.cacheSize = 0;
  }
}

// Lazy loading hook
const useLazyImage = (src: string, rootMargin = '50px') => {
  const [isIntersecting, setIsIntersecting] = useState(false);
  const [hasLoaded, setHasLoaded] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsIntersecting(true);
          observer.disconnect();
        }
      },
      { rootMargin }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, [rootMargin]);

  const source = isIntersecting ? src : undefined;

  return { imgRef, source, isIntersecting, hasLoaded, setHasLoaded };
};

// Usage example
const LazyImageGrid = ({ images }) => {
  return (
    <FlatList
      data={images}
      numColumns={3}
      renderItem={({ item }) => <LazyImageItem image={item} />}
      keyExtractor={(item) => item.id}
    />
  );
};

const LazyImageItem = ({ image }) => {
  const { imgRef, source, isIntersecting } = useLazyImage(image.uri);

  return (
    <View ref={imgRef} style={styles.gridItem}>
      {isIntersecting ? (
        <OptimizedImage
          source={{ uri: source }}
          style={styles.gridImage}
          placeholder={<ActivityIndicator />}
        />
      ) : (
        <View style={[styles.gridImage, styles.placeholder]} />
      )}
    </View>
  );
};
```

---

### 4.4 **Memory Leak Prevention and Cleanup Strategies**

**Question:** *"Identify and fix memory leaks in React Native applications with comprehensive cleanup strategies."*

**Perfect Answer Structure:**
```
MEMORY LEAK PREVENTION STRATEGY:

🎯 COMPONENT LIFECYCLE LEAKS:
- Timers and intervals cleanup
- Event listeners removal
- Subscription cleanup
- Async operation cancellation

🎯 STATE MANAGEMENT LEAKS:
- Stale closure issues
- Context subscription cleanup
- Reducer memory management

🎯 NATIVE MODULE LEAKS:
- Bridge communication cleanup
- Native event listeners
- Hardware resource management
```

**Complete Memory Leak Prevention Implementation:**
```javascript
// Memory leak detection hook (development only)
const useMemoryLeakDetector = (componentName: string) => {
  const renderCountRef = useRef(0);
  const mountedRef = useRef(true);

  useEffect(() => {
    renderCountRef.current += 1;

    return () => {
      mountedRef.current = false;
    };
  });

  useEffect(() => {
    if (__DEV__) {
      const timer = setTimeout(() => {
        if (mountedRef.current) {
          console.warn(`${componentName} may have memory leaks - still mounted after 5 minutes`);
        }
      }, 5 * 60 * 1000); // 5 minutes

      return () => clearTimeout(timer);
    }
  }, [componentName]);

  return mountedRef.current;
};

// Safe async operations with cancellation
const useSafeAsync = () => {
  const isMountedRef = useRef(true);
  const abortControllerRef = useRef(new AbortController());

  useEffect(() => {
    return () => {
      isMountedRef.current = false;
      abortControllerRef.current.abort();
    };
  }, []);

  const safeAsync = useCallback(async <T>(
    operation: () => Promise<T>,
    onSuccess?: (result: T) => void,
    onError?: (error: Error) => void
  ): Promise<void> => {
    try {
      abortControllerRef.current = new AbortController();
      const result = await operation();

      if (isMountedRef.current && !abortControllerRef.current.signal.aborted) {
        onSuccess?.(result);
      }
    } catch (error) {
      if (isMountedRef.current && !abortControllerRef.current.signal.aborted && !(error instanceof DOMException && error.name === 'AbortError')) {
        onError?.(error);
      }
    }
  }, []);

  return { safeAsync, isMounted: isMountedRef.current };
};

// BEFORE: Memory leak prone component
const LeakyComponent = () => {
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      const result = await fetch('/api/data');
      setData(await result.json()); // May set state after unmount
    };

    fetchData();

    const interval = setInterval(() => {
      console.log('Polling...'); // Never cleaned up
    }, 1000);

    // Event listener never removed
    const handleResize = () => console.log('Window resized');
    window.addEventListener('resize', handleResize);

    // Subscription never unsubscribed
    const subscription = someService.subscribe(() => {
      setData(prev => ({ ...prev, timestamp: Date.now() }));
    });

    // Timer never cleared
    const timer = setTimeout(() => {
      console.log('Delayed action');
    }, 5000);
  }, []); // Missing cleanup

  return <Text>{data?.message}</Text>;
};

// AFTER: Memory leak free component
const MemorySafeComponent = () => {
  const [data, setData] = useState(null);
  const { safeAsync } = useSafeAsync();
  const isMounted = useMemoryLeakDetector('MemorySafeComponent');

  useEffect(() => {
    // Safe async operation with automatic cancellation
    safeAsync(
      async () => {
        const response = await fetch('/api/data');
        return response.json();
      },
      (result) => setData(result),
      (error) => console.error('Fetch error:', error)
    );

    const interval = setInterval(() => {
      console.log('Polling...');
    }, 1000);

    const handleResize = () => console.log('Window resized');
    window.addEventListener('resize', handleResize);

    const subscription = someService.subscribe((newData) => {
      setData(prev => ({ ...prev, ...newData, timestamp: Date.now() }));
    });

    const timer = setTimeout(() => {
      console.log('Delayed action');
    }, 5000);

    // Comprehensive cleanup
    return () => {
      clearInterval(interval);
      clearTimeout(timer);
      window.removeEventListener('resize', handleResize);
      subscription.unsubscribe();
    };
  }, [safeAsync]);

---
[⬅️ Back to Home](../README.md) | [Next Phase: Phase 6 ➡️](phase6-testing-deployment.md)