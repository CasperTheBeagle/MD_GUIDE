# ⚡ React Performance - การปรับปรุงประสิทธิภาพ

## 📋 สารบัฐ
- [React.memo](#-reactmemo)
- [useMemo](#-usememo)
- [useCallback](#-usecallback)
- [Code Splitting](#-code-splitting)
- [Lazy Loading](#-lazy-loading)
- [Virtualization](#-virtualization)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🎯 React Performance คืออะไร

Performance optimization ใน React คือการทำให้ application ทำงานเร็วขึ้นและใช้ทรัพยากรน้อยลง

### ✨ ประโยชน์ของ Performance Optimization
- **Faster rendering**: ลดเวลาที่ใช้ในการ render
- **Better user experience**: ทำให้ app รู้สึก responsive
- **Lower resource usage**: ลดการใช้ CPU และ memory
- **Improved SEO**: หน้าเว็บโหลดเร็วขึ้น

## 🔄 React.memo

### 📦 พื้นฐาน React.memo
```typescript
// Without React.memo - re-renders every time
function ExpensiveComponent({ data }: { data: Item[] }) {
  console.log('ExpensiveComponent rendered');
  return (
    <ul>
      {data.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// With React.memo - only re-renders when props change
const MemoizedComponent = React.memo(function ExpensiveComponent({ data }: { data: Item[] }) {
  console.log('MemoizedComponent rendered');
  return (
    <ul>
      {data.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});
```

### 🎯 React.memo กับ Custom Comparison
```typescript
// Custom comparison function
const UserCard = React.memo(function UserCard({ user }: { user: User }) {
  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}, (prevProps, nextProps) => {
  // Return true if props are equal (don't re-render)
  return prevProps.user.id === nextProps.user.id &&
         prevProps.user.name === nextProps.user.name &&
         prevProps.user.email === nextProps.user.email;
});

// Usage
function App() {
  const [users, setUsers] = useState<User[]>([]);
  
  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

### ⚠️ React.memo Edge Cases
```typescript
// ❌ Bad - passing new objects as props
function BadExample() {
  const [user, setUser] = useState({ name: 'John', age: 30 });
  
  return (
    <MemoizedComponent 
      user={user}
      style={{ color: 'blue' }} // New object every render
      onClick={() => console.log('clicked')} // New function every render
    />
  );
}

// ✅ Good - stable props
function GoodExample() {
  const [user, setUser] = useState({ name: 'John', age: 30 });
  
  const style = useMemo(() => ({ color: 'blue' }), []);
  const handleClick = useCallback(() => console.log('clicked'), []);
  
  return (
    <MemoizedComponent 
      user={user}
      style={style}
      onClick={handleClick}
    />
  );
}
```

## 🔄 useMemo

### 📦 พื้นฐาน useMemo
```typescript
// Expensive calculation without memoization
function ExpensiveCalculation({ numbers }: { numbers: number[] }) {
  const sum = numbers.reduce((acc, num) => acc + num, 0); // Runs every render
  
  return <div>Sum: {sum}</div>;
}

// With useMemo
function MemoizedCalculation({ numbers }: { numbers: number[] }) {
  const sum = useMemo(() => {
    console.log('Calculating sum...');
    return numbers.reduce((acc, num) => acc + num, 0);
  }, [numbers]); // Only recalculates when numbers change
  
  return <div>Sum: {sum}</div>;
}
```

### 🎯 useMemo Use Cases
```typescript
// Filtering and sorting data
function FilteredList({ items, filter, sortBy }: {
  items: Item[];
  filter: string;
  sortBy: 'name' | 'date';
}) {
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item =>
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  const sortedItems = useMemo(() => {
    console.log('Sorting items...');
    return [...filteredItems].sort((a, b) => {
      if (sortBy === 'name') {
        return a.name.localeCompare(b.name);
      } else {
        return new Date(a.date).getTime() - new Date(b.date).getTime();
      }
    });
  }, [filteredItems, sortBy]);
  
  return (
    <ul>
      {sortedItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// Memoizing complex objects
function UserProfile({ user }: { user: User }) {
  const userContext = useMemo(() => ({
    id: user.id,
    name: user.name,
    permissions: user.permissions,
    isAdmin: user.role === 'admin'
  }), [user.id, user.name, user.permissions, user.role]);
  
  return <UserDashboard context={userContext} />;
}
```

### ⚠️ useMemo Edge Cases
```typescript
// ❌ Bad - memoizing simple values
function BadMemo() {
  const [count, setCount] = useState(0);
  
  const doubled = useMemo(() => count * 2, [count]); // Unnecessary
  
  return <div>{doubled}</div>;
}

// ✅ Good - memoizing expensive operations
function GoodMemo({ items }: { items: Item[] }) {
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>Total: {expensiveValue}</div>;
}
```

## 🔄 useCallback

### 📦 พื้นฐาน useCallback
```typescript
// Without useCallback - new function every render
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    console.log('Button clicked');
  }; // New function every render
  
  return <ChildComponent onClick={handleClick} />;
}

// With useCallback
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('Button clicked');
  }, []); // Same function reference
  
  return <ChildComponent onClick={handleClick} />;
}
```

### 🎯 useCallback Use Cases
```typescript
// Event handlers
function TodoList({ todos, onToggle, onDelete }: {
  todos: Todo[];
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
}) {
  const handleToggle = useCallback((id: string) => {
    onToggle(id);
  }, [onToggle]);
  
  const handleDelete = useCallback((id: string) => {
    onDelete(id);
  }, [onDelete]);
  
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
}

// API calls
function DataComponent({ userId }: { userId: string }) {
  const [data, setData] = useState(null);
  
  const fetchData = useCallback(async () => {
    const response = await fetch(`/api/users/${userId}`);
    const result = await response.json();
    setData(result);
  }, [userId]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return <div>{JSON.stringify(data)}</div>;
}
```

### ⚠️ useCallback Edge Cases
```typescript
// ❌ Bad - missing dependencies
function BadCallback() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log(count); // Stale closure
  }, []); // Missing count dependency
  
  return <button onClick={handleClick}>Click</button>;
}

// ✅ Good - proper dependencies
function GoodCallback() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    setCount(c => c + 1); // Functional update
  }, []);
  
  return <button onClick={handleClick}>Click</button>;
}
```

## 🔄 Code Splitting

### 📦 พื้นฐาน Code Splitting
```typescript
// Dynamic import with React.lazy
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  const [showComponent, setShowComponent] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowComponent(true)}>
        Load Component
      </button>
      
      {showComponent && (
        <React.Suspense fallback={<div>Loading...</div>}>
          <LazyComponent />
        </React.Suspense>
      )}
    </div>
  );
}
```

### 🎯 Route-based Code Splitting
```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = React.lazy(() => import('./pages/Home'));
const About = React.lazy(() => import('./pages/About'));
const Contact = React.lazy(() => import('./pages/Contact'));

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={
          <React.Suspense fallback={<div>Loading...</div>}>
            <Home />
          </React.Suspense>
        } />
        <Route path="/about" element={
          <React.Suspense fallback={<div>Loading...</div>}>
            <About />
          </React.Suspense>
        } />
        <Route path="/contact" element={
          <React.Suspense fallback={<div>Loading...</div>}>
            <Contact />
          </React.Suspense>
        } />
      </Routes>
    </BrowserRouter>
  );
}
```

### 🎯 Component-based Code Splitting
```typescript
// Heavy component that loads on demand
const HeavyChart = React.lazy(() => import('./HeavyChart'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      <button onClick={() => setShowChart(true)}>
        Show Analytics
      </button>
      
      {showChart && (
        <React.Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </React.Suspense>
      )}
    </div>
  );
}
```

## 🔄 Lazy Loading

### 📦 พื้นฐาน Lazy Loading
```typescript
// Image lazy loading
function LazyImage({ src, alt, placeholder }: {
  src: string;
  alt: string;
  placeholder?: string;
}) {
  const [imageSrc, setImageSrc] = useState(placeholder || '');
  const [imageRef, inView] = useInView({
    triggerOnce: true,
    threshold: 0.1
  });
  
  useEffect(() => {
    if (inView && src) {
      setImageSrc(src);
    }
  }, [inView, src]);
  
  return (
    <img
      ref={imageRef}
      src={imageSrc}
      alt={alt}
      style={{ opacity: imageSrc === src ? 1 : 0.5 }}
    />
  );
}

// Component lazy loading with intersection observer
function useInView(options?: IntersectionObserverInit) {
  const [ref, setRef] = useState<HTMLElement | null>(null);
  const [inView, setInView] = useState(false);
  
  useEffect(() => {
    if (!ref) return;
    
    const observer = new IntersectionObserver(([entry]) => {
      setInView(entry.isIntersecting);
    }, options);
    
    observer.observe(ref);
    
    return () => {
      observer.unobserve(ref);
    };
  }, [ref, options]);
  
  return { ref: setRef, inView };
}
```

### 🎯 Advanced Lazy Loading
```typescript
// Progressive loading
function ProgressiveImage({ src, placeholder, alt }: {
  src: string;
  placeholder: string;
  alt: string;
}) {
  const [imgSrc, setImgSrc] = useState(placeholder);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const img = new Image();
    img.src = src;
    
    img.onload = () => {
      setImgSrc(src);
      setLoading(false);
    };
    
    img.onerror = () => {
      setLoading(false);
    };
  }, [src]);
  
  return (
    <div className="progressive-image">
      <img
        src={imgSrc}
        alt={alt}
        style={{ 
          filter: loading ? 'blur(5px)' : 'none',
          transition: 'filter 0.3s'
        }}
      />
      {loading && <div className="loading-spinner" />}
    </div>
  );
}
```

## 🔄 Virtualization

### 📦 พื้นฐาน Virtualization
```typescript
// Simple virtual list
function VirtualList({ items, itemHeight, containerHeight }: {
  items: any[];
  itemHeight: number;
  containerHeight: number;
}) {
  const [scrollTop, setScrollTop] = useState(0);
  
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(
    startIndex + Math.ceil(containerHeight / itemHeight) + 1,
    items.length
  );
  
  const visibleItems = items.slice(startIndex, endIndex);
  
  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.currentTarget.scrollTop)}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        {visibleItems.map((item, index) => (
          <div
            key={startIndex + index}
            style={{
              position: 'absolute',
              top: (startIndex + index) * itemHeight,
              height: itemHeight,
              width: '100%'
            }}
          >
            {item.content}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 🎯 Advanced Virtualization
```typescript
// Grid virtualization
function VirtualGrid({ items, itemWidth, itemHeight, containerWidth, containerHeight }: {
  items: any[];
  itemWidth: number;
  itemHeight: number;
  containerWidth: number;
  containerHeight: number;
}) {
  const [scrollTop, setScrollTop] = useState(0);
  const [scrollLeft, setScrollLeft] = useState(0);
  
  const columnsCount = Math.floor(containerWidth / itemWidth);
  const rowsCount = Math.ceil(items.length / columnsCount);
  
  const startRow = Math.floor(scrollTop / itemHeight);
  const endRow = Math.min(
    startRow + Math.ceil(containerHeight / itemHeight) + 1,
    rowsCount
  );
  
  const startCol = Math.floor(scrollLeft / itemWidth);
  const endCol = Math.min(
    startCol + Math.ceil(containerWidth / itemWidth) + 1,
    columnsCount
  );
  
  const visibleItems = [];
  
  for (let row = startRow; row < endRow; row++) {
    for (let col = startCol; col < endCol; col++) {
      const index = row * columnsCount + col;
      if (index < items.length) {
        visibleItems.push({
          item: items[index],
          row,
          col,
          index
        });
      }
    }
  }
  
  return (
    <div
      style={{
        width: containerWidth,
        height: containerHeight,
        overflow: 'auto'
      }}
      onScroll={(e) => {
        setScrollTop(e.currentTarget.scrollTop);
        setScrollLeft(e.currentTarget.scrollLeft);
      }}
    >
      <div
        style={{
          height: rowsCount * itemHeight,
          width: columnsCount * itemWidth,
          position: 'relative'
        }}
      >
        {visibleItems.map(({ item, row, col, index }) => (
          <div
            key={index}
            style={{
              position: 'absolute',
              top: row * itemHeight,
              left: col * itemWidth,
              width: itemWidth,
              height: itemHeight
            }}
          >
            {item.content}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 🎯 Use Cases

### 1. **Optimized List Component**
```typescript
const ListItem = React.memo(function ListItem({ item, onSelect }: {
  item: Item;
  onSelect: (item: Item) => void;
}) {
  return (
    <div
      className="list-item"
      onClick={() => onSelect(item)}
    >
      <h3>{item.title}</h3>
      <p>{item.description}</p>
    </div>
  );
});

function OptimizedList({ items, onSelect }: {
  items: Item[];
  onSelect: (item: Item) => void;
}) {
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState<'title' | 'date'>('title');
  
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.title.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);
  
  const sortedItems = useMemo(() => {
    return [...filteredItems].sort((a, b) => {
      if (sortBy === 'title') {
        return a.title.localeCompare(b.title);
      } else {
        return new Date(a.date).getTime() - new Date(b.date).getTime();
      }
    });
  }, [filteredItems, sortBy]);
  
  const handleSelect = useCallback((item: Item) => {
    onSelect(item);
  }, [onSelect]);
  
  return (
    <div>
      <input
        type="text"
        placeholder="Search..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
      />
      
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value as 'title' | 'date')}>
        <option value="title">Sort by Title</option>
        <option value="date">Sort by Date</option>
      </select>
      
      <VirtualList
        items={sortedItems}
        itemHeight={80}
        containerHeight={400}
      >
        {(item) => (
          <ListItem
            key={item.id}
            item={item}
            onSelect={handleSelect}
          />
        )}
      </VirtualList>
    </div>
  );
}
```

### 2. **Performance Monitoring**
```typescript
function usePerformanceMonitor(componentName: string) {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(Date.now());
  
  useEffect(() => {
    renderCount.current += 1;
    const now = Date.now();
    const timeSinceLastRender = now - lastRenderTime.current;
    
    console.log(`${componentName} render #${renderCount.current}, time since last: ${timeSinceLastRender}ms`);
    
    lastRenderTime.current = now;
  });
  
  return renderCount.current;
}

function MonitoredComponent({ data }: { data: any[] }) {
  const renderCount = usePerformanceMonitor('MonitoredComponent');
  
  return (
    <div>
      <p>Render count: {renderCount}</p>
      <p>Data length: {data.length}</p>
    </div>
  );
}
```

## ⚠️ Edge Cases

### 1. **Over-optimization**
```typescript
// ❌ Bad - unnecessary memoization
function BadOptimization() {
  const [count, setCount] = useState(0);
  
  const doubled = useMemo(() => count * 2, [count]); // Simple calculation
  const handleClick = useCallback(() => setCount(c => c + 1), []); // Simple function
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled: {doubled}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

// ✅ Good - appropriate optimization
function GoodOptimization({ items }: { items: Item[] }) {
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>Total value: {expensiveValue}</div>;
}
```

### 2. **Stale Props in Memoized Components**
```typescript
// ❌ Bad - stale props
const BadMemoizedComponent = React.memo(function BadComponent({ data, onUpdate }: {
  data: any[];
  onUpdate: (data: any[]) => void;
}) {
  return (
    <div>
      <button onClick={() => onUpdate(data)}>
        Update with current data
      </button>
    </div>
  );
});

// ✅ Good - functional updates
const GoodMemoizedComponent = React.memo(function GoodComponent({ data, onUpdate }: {
  data: any[];
  onUpdate: (getData: () => any[]) => void;
}) {
  return (
    <div>
      <button onClick={() => onUpdate(() => data)}>
        Update with current data
      </button>
    </div>
  );
});
```

## 🎯 สรุป

### ✅ สิ่งที่ควบคุมประสิทธิภาพ
- **React.memo**: ป้องกันการ re-render ที่ไม่จำเป็น
- **useMemo**: cache ผลลัพธ์การคำนวณที่แพง
- **useCallback**: cache ฟังก์ชันที่ใช้ซ้ำ
- **Code Splitting**: แบ่งโค้ดเพื่อลดขนาด bundle
- **Virtualization**: แสดงข้อมูลจำนวนมากอย่างมีประสิทธิภาพ

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Patterns** ใน [PATTERNS.md](PATTERNS.md)
2. เรียนรู้ **Advanced** ใน [ADVANCED.md](ADVANCED.md)
3. ทดสอบ performance ของ application คุณ

### 💡 แนวทางการฝึก
- ใช้ React DevTools Profiler วิเคราะห์ performance
- ฝึกใช้ virtualization สำหรับข้อมูลจำนวนมาก
- ทดลอง code splitting กับ components ต่างๆ
- วัดผล performance ก่อนและหลัง optimization

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
