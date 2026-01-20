# 🚀 React Advanced - หัวข้อขั้นสูง

## 📋 สารบัฐ
- [Error Boundaries](#-error-boundaries)
- [Suspense](#-suspense)
- [Concurrent Features](#-concurrent-features)
- [Server Components](#-server-components)
- [Testing Strategies](#-testing-strategies)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🎯 Advanced React คืออะไร

Advanced React features คือเทคนิคและ concepts ขั้นสูงที่ช่วยให้ application ทำงานได้ดีขึ้นและมีประสิทธิภาพสูง

### ✨ ประโยชน์ของ Advanced Features
- **Better error handling**: จัดการ errors อย่างมีประสิทธิภาพ
- **Improved UX**: ทำให้ app รู้สึก smooth และ responsive
- **Performance**: ปรับปรุง performance ของ application
- **Scalability**: สร้าง applications ที่ขยายได้

## 🚨 Error Boundaries

### 📦 พื้นฐาน Error Boundaries
```typescript
// Basic error boundary
class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ComponentType<{ error: Error; resetError: () => void }> },
  { hasError: boolean; error: Error | null }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    
    // Log to error reporting service
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError && this.state.error) {
      const FallbackComponent = this.props.fallback;
      
      if (FallbackComponent) {
        return (
          <FallbackComponent
            error={this.state.error}
            resetError={() => this.setState({ hasError: false, error: null })}
          />
        );
      }
      
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <details>
            {this.state.error.message}
          </details>
          <button 
            onClick={() => this.setState({ hasError: false, error: null })}
          >
            Try again
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary>
      <Header />
      <ErrorBoundary fallback={ErrorFallback}>
        <MainContent />
      </ErrorBoundary>
      <Footer />
    </ErrorBoundary>
  );
}

function ErrorFallback({ error, resetError }: {
  error: Error;
  resetError: () => void;
}) {
  return (
    <div className="error-fallback">
      <h2>Oops! Something went wrong</h2>
      <p>We're sorry for the inconvenience.</p>
      <details>
        <summary>Error details</summary>
        <pre>{error.stack}</pre>
      </details>
      <button onClick={resetError}>Try again</button>
    </div>
  );
}
```

### 🎯 Advanced Error Boundaries
```typescript
// Error boundary with error reporting
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
  errorInfo: React.ErrorInfo | null;
  retryCount: number;
}

class AdvancedErrorBoundary extends React.Component<
  {
    children: React.ReactNode;
    fallback?: React.ComponentType<{
      error: Error;
      resetError: () => void;
      retryCount: number;
    }>;
    onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
    maxRetries?: number;
  },
  ErrorBoundaryState
> {
  constructor(props: any) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      retryCount: 0
    };
  }
  
  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    this.setState({ errorInfo });
    
    // Report error
    this.props.onError?.(error, errorInfo);
    
    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.error('Error caught by boundary:', error, errorInfo);
    }
  }
  
  resetError = () => {
    const { maxRetries = 3 } = this.props;
    const { retryCount } = this.state;
    
    if (retryCount < maxRetries) {
      this.setState(prevState => ({
        hasError: false,
        error: null,
        errorInfo: null,
        retryCount: prevState.retryCount + 1
      }));
    }
  };
  
  render() {
    const { hasError, error, retryCount } = this.state;
    const { fallback: FallbackComponent, children } = this.props;
    
    if (hasError && error) {
      if (FallbackComponent) {
        return (
          <FallbackComponent
            error={error}
            resetError={this.resetError}
            retryCount={retryCount}
          />
        );
      }
      
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>Attempt {retryCount + 1}</p>
          <button onClick={this.resetError}>
            {retryCount < (this.props.maxRetries || 3) ? 'Try again' : 'Reload page'}
          </button>
        </div>
      );
    }
    
    return children;
  }
}

// Functional error boundary hook
function useErrorBoundary() {
  const [error, setError] = useState<Error | null>(null);
  
  const resetError = useCallback(() => {
    setError(null);
  }, []);
  
  const captureError = useCallback((error: Error) => {
    setError(error);
  }, []);
  
  if (error) {
    throw error;
  }
  
  return { captureError, resetError };
}
```

## 🔄 Suspense

### 📦 พื้นฐาน Suspense
```typescript
// Data fetching with Suspense
interface Resource<T> {
  read(): T;
}

function createResource<T>(fetcher: () => Promise<T>): Resource<T> {
  let status: 'pending' | 'success' | 'error' = 'pending';
  let result: T;
  let error: Error;
  
  const promise = fetcher()
    .then(data => {
      status = 'success';
      result = data;
    })
    .catch(err => {
      status = 'error';
      error = err;
    });
  
  return {
    read(): T {
      switch (status) {
        case 'pending':
          throw promise;
        case 'success':
          return result;
        case 'error':
          throw error;
        default:
          throw promise;
      }
    }
  };
}

// Usage
const userResource = createResource(() => fetchUser());

function UserProfile() {
  const user = userResource.read(); // Suspends if loading
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

function App() {
  return (
    <Suspense fallback={<div>Loading user profile...</div>}>
      <UserProfile />
    </Suspense>
  );
}
```

### 🎯 Advanced Suspense Patterns
```typescript
// Multiple suspense boundaries
function App() {
  return (
    <div>
      <Suspense fallback={<HeaderSkeleton />}>
        <Header />
      </Suspense>
      
      <Suspense fallback={<MainSkeleton />}>
        <MainContent />
      </Suspense>
      
      <Suspense fallback={<FooterSkeleton />}>
        <Footer />
      </Suspense>
    </div>
  );
}

// Lazy loading with Suspense
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function LazyWrapper() {
  return (
    <Suspense fallback={<div>Loading component...</div>}>
      <LazyComponent />
    </Suspense>
  );
}

// Image loading with Suspense
function ImageResource(src: string) {
  let status: 'pending' | 'success' | 'error' = 'pending';
  let result: HTMLImageElement;
  let error: Error;
  
  const img = new Image();
  img.src = src;
  
  const promise = new Promise<HTMLImageElement>((resolve, reject) => {
    img.onload = () => {
      status = 'success';
      result = img;
      resolve(img);
    };
    
    img.onerror = () => {
      status = 'error';
      error = new Error(`Failed to load image: ${src}`);
      reject(error);
    };
  });
  
  return {
    read(): HTMLImageElement {
      switch (status) {
        case 'pending':
          throw promise;
        case 'success':
          return result;
        case 'error':
          throw error;
        default:
          throw promise;
      }
    }
  };
}

function SuspenseImage({ src, alt }: { src: string; alt: string }) {
  const imageResource = useMemo(() => ImageResource(src), [src]);
  const img = imageResource.read();
  
  return <img src={img.src} alt={alt} />;
}
```

## 🔄 Concurrent Features

### 📦 useTransition Hook
```typescript
// Basic useTransition
function SearchableList({ items }: { items: Item[] }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [isPending, startTransition] = useTransition();
  
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);
  
  const handleSearch = (value: string) => {
    // Urgent update
    setSearchTerm(value);
    
    // Transition update (can be interrupted)
    startTransition(() => {
      // Expensive filtering happens here
      // React can interrupt this if user types again
    });
  };
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      
      {isPending && <div>Searching...</div>}
      
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 🎯 Advanced useDeferredValue
```typescript
// useDeferredValue for search
function AdvancedSearch({ items }: { items: Item[] }) {
  const [searchTerm, setSearchTerm] = useState('');
  const deferredSearchTerm = useDeferredValue(searchTerm);
  
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(deferredSearchTerm.toLowerCase())
    );
  }, [items, deferredSearchTerm]);
  
  const isStale = searchTerm !== deferredSearchTerm;
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        {filteredItems.map(item => (
          <div key={item.id} style={{ opacity: isStale ? 0.7 : 1 }}>
            {item.name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 🖥️ Server Components

### 📦 พื้นฐาน Server Components
```typescript
// Server component (runs on server)
async function UserProfileServer({ userId }: { userId: string }) {
  // Direct data fetching - no loading states needed
  const user = await fetchUser(userId);
  const posts = await fetchUserPosts(userId);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
      
      {/* Client component for interactivity */}
      <PostListClient posts={posts} />
    </div>
  );
}

// Client component (runs on client)
'use client';

function PostListClient({ posts }: { posts: Post[] }) {
  const [likedPosts, setLikedPosts] = useState<Set<string>>(new Set());
  
  const toggleLike = (postId: string) => {
    setLikedPosts(prev => {
      const newSet = new Set(prev);
      if (newSet.has(postId)) {
        newSet.delete(postId);
      } else {
        newSet.add(postId);
      }
      return newSet;
    });
  };
  
  return (
    <div>
      {posts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.content}</p>
          <button 
            onClick={() => toggleLike(post.id)}
            style={{ color: likedPosts.has(post.id) ? 'red' : 'black' }}
          >
            {likedPosts.has(post.id) ? '❤️' : '🤍'} Like
          </button>
        </div>
      ))}
    </div>
  );
}
```

### 🎯 Server Components Patterns
```typescript
// Data fetching pattern
async function ProductPage({ productId }: { productId: string }) {
  // Parallel data fetching
  const [product, reviews, recommendations] = await Promise.all([
    fetchProduct(productId),
    fetchProductReviews(productId),
    fetchRecommendations(productId)
  ]);
  
  return (
    <div>
      <ProductDetails product={product} />
      <ProductReviews reviews={reviews} />
      <ProductRecommendations recommendations={recommendations} />
    </div>
  );
}

// Streaming pattern
async function StreamingPage() {
  const stream = await fetchDataStream();
  
  return (
    <div>
      <h1>Streaming Data</h1>
      <DataStreamClient stream={stream} />
    </div>
  );
}

'use client';

function DataStreamClient({ stream }: { stream: ReadableStream }) {
  const [data, setData] = useState<any[]>([]);
  
  useEffect(() => {
    const reader = stream.getReader();
    
    const readStream = async () => {
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        
        setData(prev => [...prev, value]);
      }
    };
    
    readStream();
  }, [stream]);
  
  return (
    <div>
      {data.map((item, index) => (
        <div key={index}>{JSON.stringify(item)}</div>
      ))}
    </div>
  );
}
```

## 🧪 Testing Strategies

### 📦 Unit Testing with Jest
```typescript
// Component testing
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import '@testing-library/jest-dom';
import { Counter } from './Counter';

describe('Counter Component', () => {
  test('renders initial count', () => {
    render(<Counter initialCount={0} />);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
  
  test('increments count when button is clicked', () => {
    render(<Counter initialCount={0} />);
    
    const incrementButton = screen.getByText('Increment');
    fireEvent.click(incrementButton);
    
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });
  
  test('calls onCountChange when count changes', () => {
    const mockOnCountChange = jest.fn();
    render(<Counter initialCount={0} onCountChange={mockOnCountChange} />);
    
    fireEvent.click(screen.getByText('Increment'));
    
    expect(mockOnCountChange).toHaveBeenCalledWith(1);
  });
});

// Hook testing
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from './useCounter';

describe('useCounter Hook', () => {
  test('returns initial count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    expect(result.current.count).toBe(5);
  });
  
  test('increments count', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  test('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

### 🎯 Integration Testing
```typescript
// Integration testing
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TodoApp } from './TodoApp';
import { rest } from 'msw';
import { setupServer } from 'msw/node';

// Mock API server
const server = setupServer(
  rest.get('/api/todos', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: '1', text: 'Test todo 1', completed: false },
        { id: '2', text: 'Test todo 2', completed: true }
      ])
    );
  }),
  rest.post('/api/todos', (req, res, ctx) => {
    return res(
      ctx.status(201),
      ctx.json({ id: '3', text: 'New todo', completed: false })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('TodoApp Integration', () => {
  test('loads and displays todos', async () => {
    render(<TodoApp />);
    
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    
    await waitFor(() => {
      expect(screen.getByText('Test todo 1')).toBeInTheDocument();
      expect(screen.getByText('Test todo 2')).toBeInTheDocument();
    });
  });
  
  test('adds new todo', async () => {
    const user = userEvent.setup();
    render(<TodoApp />);
    
    const input = screen.getByPlaceholderText('Add a todo');
    const addButton = screen.getByText('Add');
    
    await user.type(input, 'New todo');
    await user.click(addButton);
    
    await waitFor(() => {
      expect(screen.getByText('New todo')).toBeInTheDocument();
    });
  });
});
```

### 🎯 E2E Testing with Playwright
```typescript
// E2E testing
import { test, expect } from '@playwright/test';

test.describe('Todo App E2E', () => {
  test('user can add and complete todos', async ({ page }) => {
    await page.goto('/');
    
    // Add todo
    await page.fill('[data-testid="todo-input"]', 'Learn Playwright');
    await page.click('[data-testid="add-button"]');
    
    // Verify todo is added
    await expect(page.locator('[data-testid="todo-item"]')).toContainText('Learn Playwright');
    
    // Complete todo
    await page.click('[data-testid="todo-checkbox"]');
    
    // Verify todo is completed
    await expect(page.locator('[data-testid="todo-item"]')).toHaveClass('completed');
  });
  
  test('user can filter todos', async ({ page }) => {
    await page.goto('/');
    
    // Add multiple todos
    await page.fill('[data-testid="todo-input"]', 'Active todo');
    await page.click('[data-testid="add-button"]');
    
    await page.fill('[data-testid="todo-input"]', 'Completed todo');
    await page.click('[data-testid="add-button"]');
    
    // Complete second todo
    await page.locator('[data-testid="todo-checkbox"]').last().click();
    
    // Filter active todos
    await page.click('[data-testid="filter-active"]');
    
    // Verify only active todo is shown
    await expect(page.locator('[data-testid="todo-item"]')).toHaveCount(1);
    await expect(page.locator('[data-testid="todo-item"]')).toContainText('Active todo');
  });
});
```

### 🎯 Performance Testing
```typescript
// Performance testing
import { render, screen } from '@testing-library/react';
import { PerformanceProfiler } from './PerformanceProfiler';

test('component renders within performance budget', async () => {
  const onRender = jest.fn();
  
  render(
    <PerformanceProfiler onRender={onRender}>
      <ExpensiveComponent data={largeDataSet} />
    </PerformanceProfiler>
  );
  
  // Wait for component to render
  await screen.findByTestId('expensive-component');
  
  // Check render time
  const renderCalls = onRender.mock.calls;
  const lastRender = renderCalls[renderCalls.length - 1][0];
  
  expect(lastRender.duration).toBeLessThan(100); // 100ms budget
});

// Memory leak testing
test('component does not leak memory', () => {
  const { unmount } = render(<ComponentWithIntervals />);
  
  // Component should clean up intervals on unmount
  unmount();
  
  // Verify no active intervals
  expect(clearInterval).toHaveBeenCalled();
});
```

## 🎯 Use Cases

### 1. **Error Boundary for Data Fetching**
```typescript
function DataFetchingWithErrorBoundary() {
  return (
    <ErrorBoundary
      fallback={({ error, resetError }) => (
        <div className="error-fallback">
          <h2>Failed to load data</h2>
          <p>{error.message}</p>
          <button onClick={resetError}>Retry</button>
        </div>
      )}
      onError={(error, errorInfo) => {
        // Send to error reporting service
        reportError(error, errorInfo);
      }}
    >
      <Suspense fallback={<DataLoadingSkeleton />}>
        <UserData />
      </Suspense>
    </ErrorBoundary>
  );
}
```

### 2. **Concurrent Search with Transitions**
```typescript
function ConcurrentSearch() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  const deferredQuery = useDeferredValue(query);
  
  const results = useMemo(() => {
    return searchItems(deferredQuery);
  }, [deferredQuery]);
  
  const handleSearch = (value: string) => {
    // Immediate UI update
    setQuery(value);
    
    // Defer expensive search
    startTransition(() => {
      // Search happens here
    });
  };
  
  return (
    <div>
      <input
        value={query}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      
      {isPending && <div className="search-indicator">Searching...</div>}
      
      <div style={{ opacity: query !== deferredQuery ? 0.7 : 1 }}>
        {results.map(item => (
          <div key={item.id}>{item.name}</div>
        ))}
      </div>
    </div>
  );
}
```

## ⚠️ Edge Cases

### 1. **Error Boundary Limitations**
```typescript
// ❌ Error boundaries don't catch errors in event handlers
function BadErrorBoundary() {
  const handleClick = () => {
    throw new Error('Event handler error'); // Won't be caught
  };
  
  return <button onClick={handleClick}>Click me</button>;
}

// ✅ Use try-catch for event handlers
function GoodErrorBoundary() {
  const handleClick = () => {
    try {
      riskyOperation();
    } catch (error) {
      console.error('Handled error:', error);
    }
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

### 2. **Suspense with Server Components**
```typescript
// ❌ Mixing server and client components incorrectly
function BadMix() {
  const [state, setState] = useState(); // Client hook in server component
  
  return <div>{state}</div>;
}

// ✅ Proper separation
function ServerComponent() {
  const data = await fetchData(); // Server-side data fetching
  
  return (
    <div>
      <h1>Server Data</h1>
      <ClientComponent initialData={data} />
    </div>
  );
}

'use client';

function ClientComponent({ initialData }) {
  const [state, setState] = useState(initialData);
  
  return <div>{state}</div>;
}
```

## 🎯 สรุป

### ✨ สิ่งที่ควบคุม
- **Error Boundaries**: จัดการ errors อย่าง graceful
- **Suspense**: จัดการ loading states อย่างมีประสิทธิภาพ
- **Concurrent Features**: ทำให้ UI รู้สึก responsive
- **Server Components**: ปรับปรุง performance ด้วย server-side rendering
- **Testing**: ทดสอบ applications อย่างครอบคลุม

### 🚀 ขั้นตอนถัดไป
1. ทดลอบสร้าง production applications
2. ศึกษา deployment strategies
3. เรียนรู้ monitoring และ debugging

### 💡 แนวทางการฝึก
- สร้าง error boundaries สำหรับ production
- ฝึกใช้ concurrent features สำหรับ UX
- ทดลอบ server components เมื่อ available
- เขียน comprehensive tests สำหรับ applications

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
