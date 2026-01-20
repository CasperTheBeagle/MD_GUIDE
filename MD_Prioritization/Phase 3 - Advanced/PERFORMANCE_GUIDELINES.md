# ⚡ แนวทานการปรับปรุงประสิทธิภาพ

## 🎯 วัตถุประสงค์
เอกสารนี้ให้คำแนะนำและแนวทานในการปรับปรุงประสิทธิภาพของ React applications เพื่อให้แอปพลิเคชันทำงานได้เร็ว ตอบสนองได้ดี และให้ประสบการณ์ผู้ใช้ที่ดีที่สุด

## 📊 Performance Metrics

### 1. Core Web Vitals
| Metric | Target | Description |
|--------|--------|-------------|
| **LCP** (Largest Contentful Paint) | < 2.5s | การโหลดเนื้อหาที่ใหญ่ที่สุด |
| **FID** (First Input Delay) | < 100ms | เวลาตอบสนองครั้งแรก |
| **CLS** (Cumulative Layout Shift) | < 0.1 | การเปลี่ยนแปลง layout สะสม |

### 2. Custom Metrics
| Metric | Target | Description |
|--------|--------|-------------|
| **TTI** (Time to Interactive) | < 3.8s | เวลาที่แอปพลิเคชันพร้อมใช้งาน |
| **FCP** (First Contentful Paint) | < 1.8s | เวลาที่เนื้อหาแรกปรากฏ |
| **Bundle Size** | < 1MB (gzipped) | ขนาดของ bundle |
| **Memory Usage** | < 50MB | การใช้หน่วยความจำ |

## 🚀 Loading Performance

### 1. Code Splitting
```typescript
// ✅ ดี - Route-based code splitting
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

const App = () => (
  <Suspense fallback={<PageSkeleton />}>
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/profile" element={<Profile />} />
    </Routes>
  </Suspense>
);

// ✅ ดี - Component-based code splitting
const HeavyChart = lazy(() => import('./components/HeavyChart'));

const Dashboard = () => (
  <div>
    <h1>Dashboard</h1>
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart data={chartData} />
    </Suspense>
  </div>
);
```

### 2. Bundle Optimization
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { splitVendorChunkPlugin } from 'vite';

export default defineConfig({
  plugins: [react(), splitVendorChunkPlugin()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
          ui: ['@mui/material', '@emotion/react'],
          charts: ['chart.js', 'react-chartjs-2'],
        },
      },
    },
    chunkSizeWarningLimit: 1000,
  },
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom'],
  },
});
```

### 3. Asset Optimization
```typescript
// ✅ ดี - Lazy loading images
const LazyImage = ({ src, alt, className }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsInView(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <div ref={imgRef} className={className}>
      {isInView && (
        <img
          src={src}
          alt={alt}
          onLoad={() => setIsLoaded(true)}
          style={{ opacity: isLoaded ? 1 : 0 }}
        />
      )}
      {!isLoaded && <ImageSkeleton />}
    </div>
  );
};

// ✅ ดี - Progressive image loading
const ProgressiveImage = ({ lowQualitySrc, highQualitySrc, alt }) => {
  const [imageSrc, setImageSrc] = useState(lowQualitySrc);

  useEffect(() => {
    const img = new Image();
    img.src = highQualitySrc;
    img.onload = () => setImageSrc(highQualitySrc);
  }, [highQualitySrc]);

  return <img src={imageSrc} alt={alt} />;
};
```

## 🔄 Runtime Performance

### 1. React Optimization
```typescript
// ✅ ดี - React.memo สำหรับ pure components
const UserCard = React.memo(({ user, onSelect }) => {
  return (
    <div className="user-card" onClick={() => onSelect(user)}>
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
});

// ✅ ดี - useMemo สำหรับ expensive calculations
const ExpensiveList = ({ items, filter }) => {
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  const sortedItems = useMemo(() => {
    console.log('Sorting items...');
    return filteredItems.sort((a, b) => a.name.localeCompare(b.name));
  }, [filteredItems]);

  return (
    <ul>
      {sortedItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
};

// ✅ ดี - useCallback สำหรับ function references
const ParentComponent = ({ items }) => {
  const [selectedItem, setSelectedItem] = useState(null);

  const handleSelect = useCallback((item) => {
    setSelectedItem(item);
  }, []);

  return (
    <div>
      {items.map(item => (
        <UserCard 
          key={item.id} 
          user={item} 
          onSelect={handleSelect} 
        />
      ))}
    </div>
  );
};
```

### 2. State Management Optimization
```typescript
// ✅ ดี - Context splitting
const UserContext = createContext();
const ThemeContext = createContext();

const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
};

const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

// ✅ ดี - State selector สำหรับ large state
const useUserSelector = (selector) => {
  const { user } = useContext(UserContext);
  return useMemo(() => selector(user), [user, selector]);
};

// การใช้งาน
const UserName = () => {
  const userName = useUserSelector(user => user?.name);
  return <span>{userName}</span>;
};
```

### 3. Virtual Scrolling
```typescript
// ✅ ดี - Virtual list สำหรับ large datasets
import { FixedSizeList as List } from 'react-window';

const VirtualizedList = ({ items }) => {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ListItem item={items[index]} />
    </div>
  );

  return (
    <List
      height={600}
      itemCount={items.length}
      itemSize={80}
      width="100%"
    >
      {Row}
    </List>
  );
};

// ✅ ดี - Infinite scroll สำหรับ pagination
const InfiniteScrollList = () => {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const [page, setPage] = useState(1);

  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    
    setLoading(true);
    try {
      const newItems = await fetchItems(page);
      setItems(prev => [...prev, ...newItems]);
      setPage(prev => prev + 1);
      setHasMore(newItems.length > 0);
    } finally {
      setLoading(false);
    }
  }, [page, loading, hasMore]);

  return (
    <InfiniteScroll
      dataLength={items.length}
      next={loadMore}
      hasMore={hasMore}
      loader={<LoadingSpinner />}
    >
      {items.map(item => (
        <ListItem key={item.id} item={item} />
      ))}
    </InfiniteScroll>
  );
};
```

## 🎨 Rendering Performance

### 1. CSS Optimization
```css
/* ✅ ดี - ใช้ CSS transforms แทน top/left */
.animated-element {
  transform: translateX(100px);
  will-change: transform;
}

/* ✅ ดี - ใช้ opacity สำหรับ fade effects */
.fade-element {
  opacity: 0;
  transition: opacity 0.3s ease;
}

.fade-element.visible {
  opacity: 1;
}

/* ❌ ไม่ดี - ใช้ top/left ที่ trigger reflow */
.bad-animation {
  top: 100px;
  left: 100px;
}

/* ✅ ดี - ใช้ contain สำหรับ isolation */
.isolated-component {
  contain: strict;
}

/* ✅ ดี - ใช้ content-visibility สำหรับ off-screen content */
.off-screen-content {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}
```

### 2. Layout Optimization
```typescript
// ✅ ดี - ใช้ CSS Grid และ Flexbox อย่างมีประสิทธิภาพ
const OptimizedLayout = () => (
  <div className="grid-container">
    <header className="header">Header</header>
    <aside className="sidebar">Sidebar</aside>
    <main className="main">Main Content</main>
    <footer className="footer">Footer</footer>
  </div>
);

/* CSS */
.grid-container {
  display: grid;
  grid-template-areas: 
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

### 3. Animation Performance
```typescript
// ✅ ดี - ใช้ Framer Motion สำหรับ animations
import { motion, AnimatePresence } from 'framer-motion';

const AnimatedModal = ({ isOpen, onClose, children }) => (
  <AnimatePresence>
    {isOpen && (
      <motion.div
        initial={{ opacity: 0, scale: 0.8 }}
        animate={{ opacity: 1, scale: 1 }}
        exit={{ opacity: 0, scale: 0.8 }}
        transition={{ duration: 0.2 }}
        className="modal-overlay"
        onClick={onClose}
      >
        <motion.div
          initial={{ y: 50 }}
          animate={{ y: 0 }}
          exit={{ y: 50 }}
          transition={{ type: "spring", stiffness: 300 }}
          className="modal-content"
          onClick={(e) => e.stopPropagation()}
        >
          {children}
        </motion.div>
      </motion.div>
    )}
  </AnimatePresence>
);

// ✅ ดี - ใช้ CSS transforms สำหรับ smooth animations
const AnimatedCard = ({ isHovered, children }) => (
  <motion.div
    whileHover={{ scale: 1.05 }}
    whileTap={{ scale: 0.95 }}
    transition={{ type: "spring", stiffness: 400, damping: 17 }}
    className="card"
  >
    {children}
  </motion.div>
);
```

## 📱 Memory Management

### 1. Memory Leak Prevention
```typescript
// ✅ ดี - Cleanup subscriptions และ timers
const useDataSubscription = (id) => {
  const [data, setData] = useState(null);

  useEffect(() => {
    const subscription = dataService.subscribe(id, (newData) => {
      setData(newData);
    });

    // Cleanup subscription
    return () => subscription.unsubscribe();
  }, [id]);

  return data;
};

// ✅ ดี - Cleanup event listeners
const useWindowResize = (callback) => {
  useEffect(() => {
    const handleResize = () => callback(window.innerWidth, window.innerHeight);
    
    window.addEventListener('resize', handleResize);
    
    // Cleanup event listener
    return () => window.removeEventListener('resize', handleResize);
  }, [callback]);
};

// ✅ ดี - Cleanup intervals
const useTimer = (interval, callback) => {
  useEffect(() => {
    const timerId = setInterval(callback, interval);
    
    // Cleanup interval
    return () => clearInterval(timerId);
  }, [interval, callback]);
};
```

### 2. Object Pooling
```typescript
// ✅ ดี - Object pooling สำหรับ frequently created objects
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    
    // Pre-allocate objects
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }

  acquire() {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    return this.createFn();
  }

  release(obj) {
    this.resetFn(obj);
    this.pool.push(obj);
  }
}

// การใช้งาน
const particlePool = new ObjectPool(
  () => ({ x: 0, y: 0, vx: 0, vy: 0, life: 1 }),
  (particle) => {
    particle.x = 0;
    particle.y = 0;
    particle.vx = 0;
    particle.vy = 0;
    particle.life = 1;
  }
);

const ParticleSystem = () => {
  const [particles, setParticles] = useState([]);

  const addParticle = useCallback(() => {
    const particle = particlePool.acquire();
    particle.x = Math.random() * 100;
    particle.y = Math.random() * 100;
    
    setParticles(prev => [...prev, particle]);
  }, []);

  const removeParticle = useCallback((index) => {
    setParticles(prev => {
      const newParticles = [...prev];
      const removed = newParticles.splice(index, 1)[0];
      particlePool.release(removed);
      return newParticles;
    });
  }, []);

  return (
    <div>
      <button onClick={addParticle}>Add Particle</button>
      {particles.map((particle, index) => (
        <div
          key={index}
          style={{
            position: 'absolute',
            left: particle.x,
            top: particle.y,
            width: 4,
            height: 4,
            backgroundColor: 'red',
          }}
        />
      ))}
    </div>
  );
};
```

## 🌐 Network Performance

### 1. API Optimization
```typescript
// ✅ ดี - Request batching
const useBatchedRequests = () => {
  const [queue, setQueue] = useState([]);
  const [results, setResults] = useState({});

  useEffect(() => {
    if (queue.length === 0) return;

    const timer = setTimeout(() => {
      const requests = [...queue];
      setQueue([]);
      
      // Batch requests
      Promise.all(
        requests.map(req => 
          fetch(req.url).then(res => res.json())
        )
      ).then(batchResults => {
        const newResults = {};
        requests.forEach((req, index) => {
          newResults[req.id] = batchResults[index];
        });
        setResults(prev => ({ ...prev, ...newResults }));
      });
    }, 100);

    return () => clearTimeout(timer);
  }, [queue]);

  const addRequest = useCallback((url, id) => {
    setQueue(prev => [...prev, { url, id }]);
  }, []);

  return { addRequest, results };
};

// ✅ ดี - Request deduplication
const useDedupedRequest = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    let cancelled = false;
    
    const fetchData = async () => {
      if (loading) return; // Prevent duplicate requests
      
      setLoading(true);
      try {
        const response = await fetch(url);
        const result = await response.json();
        
        if (!cancelled) {
          setData(result);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [url, loading]);

  return { data, loading };
};
```

### 2. Caching Strategy
```typescript
// ✅ ดี - Service Worker สำหรับ caching
// public/sw.js
const CACHE_NAME = 'app-v1';
const urlsToCache = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
  '/api/users',
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        // Cache hit - return response
        if (response) {
          return response;
        }

        return fetch(event.request).then(
          (response) => {
            // Check if valid response
            if (!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }

            // Clone response
            const responseToCache = response.clone();

            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, responseToCache);
              });

            return response;
          }
        );
      })
  );
});

// ✅ ดี - Memory cache สำหรับ API responses
const useAPICache = () => {
  const cache = useRef(new Map());

  const get = useCallback((key) => {
    const item = cache.current.get(key);
    if (!item) return null;
    
    // Check if expired (5 minutes)
    if (Date.now() - item.timestamp > 5 * 60 * 1000) {
      cache.current.delete(key);
      return null;
    }
    
    return item.data;
  }, []);

  const set = useCallback((key, data) => {
    cache.current.set(key, {
      data,
      timestamp: Date.now(),
    });
  }, []);

  const clear = useCallback(() => {
    cache.current.clear();
  }, []);

  return { get, set, clear };
};
```

## 📊 Performance Monitoring

### 1. Real User Monitoring (RUM)
```typescript
// ✅ ดี - Performance monitoring
const usePerformanceMonitoring = () => {
  useEffect(() => {
    // Monitor Core Web Vitals
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        switch (entry.entryType) {
          case 'largest-contentful-paint':
            analytics.track('LCP', { value: entry.startTime });
            break;
          case 'first-input':
            analytics.track('FID', { value: entry.processingStart - entry.startTime });
            break;
          case 'layout-shift':
            if (!entry.hadRecentInput) {
              analytics.track('CLS', { value: entry.value });
            }
            break;
        }
      });
    });

    observer.observe({ entryTypes: ['largest-contentful-paint', 'first-input', 'layout-shift'] });

    return () => observer.disconnect();
  }, []);
};

// ✅ ดี - Custom performance metrics
const useCustomMetrics = () => {
  const measureComponentRender = useCallback((componentName) => {
    const startTime = performance.now();
    
    return () => {
      const endTime = performance.now();
      const renderTime = endTime - startTime;
      
      analytics.track('component_render_time', {
        component: componentName,
        duration: renderTime,
      });
      
      if (renderTime > 16) { // More than one frame
        console.warn(`Slow render detected: ${componentName} took ${renderTime}ms`);
      }
    };
  }, []);

  return { measureComponentRender };
};

// การใช้งาน
const ExpensiveComponent = ({ data }) => {
  const { measureComponentRender } = useCustomMetrics();
  
  useEffect(() => {
    return measureComponentRender('ExpensiveComponent');
  }, [measureComponentRender]);

  return (
    <div>
      {/* Complex rendering logic */}
    </div>
  );
};
```

### 2. Performance Budgets
```javascript
// package.json
{
  "scripts": {
    "build:analyze": "npm run build && npx bundle-analyzer build/static/js/*.js",
    "perf:audit": "npm run build && npx lighthouse http://localhost:3000 --output=json --output-path=./lighthouse-report.json",
    "perf:bundle-size": "npm run build && bundlesize"
  },
  "bundlesize": [
    {
      "path": "build/static/js/*.js",
      "maxSize": "500kb"
    },
    {
      "path": "build/static/css/*.css",
      "maxSize": "50kb"
    }
  ]
}
```

## 🔧 Performance Testing

### 1. Load Testing
```javascript
// k6 load test script
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up to 100 users
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 200 }, // Ramp up to 200 users
    { duration: '5m', target: 200 }, // Stay at 200 users
    { duration: '2m', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.1'],    // Error rate under 10%
  },
};

export default function () {
  let response = http.get('https://example.com/api/users');
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);
}
```

### 2. Performance Profiling
```typescript
// ✅ ดี - React Profiler integration
import { Profiler } from 'react';

const onRenderCallback = (id, phase, actualDuration, baseDuration, startTime, commitTime) => {
  analytics.track('react_profile', {
    component: id,
    phase: phase, // 'mount' or 'update'
    actualDuration,
    baseDuration,
    startTime,
    commitTime,
  });
};

const ProfiledComponent = ({ children }) => (
  <Profiler id="App" onRender={onRenderCallback}>
    {children}
  </Profiler>
);

// ✅ ดี - Custom performance hooks
const useRenderPerformance = (componentName) => {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(Date.now());

  useEffect(() => {
    renderCount.current += 1;
    const now = Date.now();
    const timeSinceLastRender = now - lastRenderTime.current;
    
    if (renderCount.current > 1 && timeSinceLastRender < 16) {
      console.warn(`${componentName} re-rendered quickly (${timeSinceLastRender}ms)`);
    }
    
    lastRenderTime.current = now;
  });

  return renderCount.current;
};
```

## 📋 Performance Checklist

### Pre-Deployment
- [ ] Bundle size analysis completed
- [ ] Code splitting implemented
- [ ] Images optimized
- [ ] Core Web Vitals meet targets
- [ ] Memory leaks tested
- [ ] Load testing completed

### Runtime Monitoring
- [ ] Performance metrics tracked
- [ ] Error monitoring active
- [ ] Real user monitoring enabled
- [ ] Performance budgets set
- [ ] Alert thresholds configured

### Optimization Techniques
- [ ] React.memo used appropriately
- [ ] useMemo/useCallback implemented
- [ ] Virtual scrolling for large lists
- [ ] Image lazy loading
- [ ] API response caching
- [ ] Service worker implemented

---
*Performance Guidelines v1.0*  
*Last updated: 2024-01-20*  
*Contact: performance-team@example.com*
