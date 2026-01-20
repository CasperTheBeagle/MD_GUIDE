# 🌟 แนวทางปฏิบัติที่ดีที่สุด

## 🎯 วัตถุประสงค์
เอกสารนี้รวบรวมแนวทางปฏิบัติที่ดีที่สุดสำหรับการพัฒนา React applications เพื่อให้แน่ใจว่าโค้ดมีคุณภาพสูง บำรุงรักษาง่าย และทำงานได้ดี

## 🏗️ สถาปัตยกรรมโค้ด

### 1. การจัดระเบียบโค้ด
```typescript
// ✅ ดี - จัดระเบียบตามความรับผิดชอบ
src/
├── components/
│   ├── ui/              # Reusable UI components
│   ├── features/        # Feature-specific components
│   └── layout/          # Layout components
├── hooks/               # Custom hooks
├── services/            # API services
├── utils/               # Utility functions
├── types/               # TypeScript types
└── constants/           # Application constants

// ❌ ไม่ดี - จัดระเบียบตาม file type
src/
├── components/
├── hooks/
├── services/
├── utils/
└── types/
```

### 2. Component Design Patterns
```typescript
// ✅ ดี - Single Responsibility Principle
const UserAvatar = ({ user, size, onClick }) => (
  <img 
    src={user.avatar} 
    alt={user.name}
    className={`avatar avatar-${size}`}
    onClick={onClick}
  />
);

const UserProfile = ({ user }) => (
  <div className="user-profile">
    <UserAvatar user={user} size="large" />
    <h2>{user.name}</h2>
    <p>{user.bio}</p>
  </div>
);

// ❌ ไม่ดี - Component ทำหลายอย่าง
const UserComponent = ({ user, showAvatar, showProfile, onAvatarClick }) => (
  <div>
    {showAvatar && (
      <img src={user.avatar} onClick={onAvatarClick} />
    )}
    {showProfile && (
      <div>
        <h2>{user.name}</h2>
        <p>{user.bio}</p>
      </div>
    )}
  </div>
);
```

### 3. Custom Hooks
```typescript
// ✅ ดี - Reusable logic ใน hook
const useApi = (url, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url, options);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
};

// การใช้งาน
const UserList = () => {
  const { data: users, loading, error } = useApi('/api/users');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};
```

## 🎨 การจัดการ State

### 1. Local State
```typescript
// ✅ ดี - ใช้ useState สำหรับ local state
const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

// ✅ ดี - ใช้ useReducer สำหรับ complex state
const counterReducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
};

const AdvancedCounter = () => {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
};
```

### 2. Global State
```typescript
// ✅ ดี - ใช้ Context API สำหรับ global state
const AppContext = createContext();

const AppProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = {
    user,
    setUser,
    theme,
    setTheme,
  };
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
};

// Custom hook สำหรับใช้ context
const useApp = () => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp must be used within AppProvider');
  }
  return context;
};

// การใช้งาน
const UserProfile = () => {
  const { user, theme } = useApp();
  
  return (
    <div className={`profile profile-${theme}`}>
      <h1>{user?.name}</h1>
    </div>
  );
};
```

### 3. Server State
```typescript
// ✅ ดี - ใช้ React Query สำหรับ server state
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const response = await fetch('/api/users');
      return response.json();
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

const useCreateUser = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (userData) => {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      });
      return response.json();
    },
    onSuccess: () => {
      // Invalidate cache และ refetch
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

## 🔄 การจัดการ Side Effects

### 1. useEffect Best Practices
```typescript
// ✅ ดี - มี cleanup function
const Timer = () => {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    // Cleanup
    return () => clearInterval(interval);
  }, []); // Empty dependency array
  
  return <div>Seconds: {seconds}</div>;
};

// ✅ ดี - มี dependencies ที่ถูกต้อง
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const fetchUser = async () => {
      const response = await fetch(`/api/users/${userId}`);
      const userData = await response.json();
      setUser(userData);
    };
    
    fetchUser();
  }, [userId]); // userId เป็น dependency
  
  return user ? <div>{user.name}</div> : <div>Loading...</div>;
};

// ❌ ไม่ดี - ไม่มี cleanup หรือ dependencies ผิด
const BadEffect = () => {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/data').then(setData); // ไม่มี cleanup
  }); // ไม่มี dependencies จะทำงานทุกครั้งที่ re-render
  
  return <div>{data}</div>;
};
```

### 2. Custom Hooks for Side Effects
```typescript
// ✅ ดี - แยก side effects ไว้ใน custom hook
const useLocalStorage = (key, initialValue) => {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
};

// การใช้งาน
const App = () => {
  const [name, setName] = useLocalStorage('name', 'Guest');
  
  return (
    <input 
      value={name}
      onChange={(e) => setName(e.target.value)}
      placeholder="Enter your name"
    />
  );
};
```

## 🎯 Performance Optimization

### 1. React.memo
```typescript
// ✅ ดี - ใช้ React.memo สำหรับ expensive components
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  console.log('ExpensiveComponent rendered');
  
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      processed: expensiveCalculation(item)
    }));
  }, [data]);
  
  return (
    <div>
      {processedData.map(item => (
        <div key={item.id}>{item.processed}</div>
      ))}
    </div>
  );
});

// ❌ ไม่ดี - Component ที่แพงแต่ไม่มี memoization
const BadExpensiveComponent = ({ data, onUpdate }) => {
  console.log('BadExpensiveComponent rendered');
  
  const processedData = data.map(item => ({
    ...item,
    processed: expensiveCalculation(item)
  }));
  
  return (
    <div>
      {processedData.map(item => (
        <div key={item.id}>{item.processed}</div>
      ))}
    </div>
  );
};
```

### 2. useMemo และ useCallback
```typescript
// ✅ ดี - ใช้ useMemo สำหรับ expensive calculations
const FilteredList = ({ items, filter }) => {
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
};

// ✅ ดี - ใช้ useCallback สำหรับ functions
const ButtonList = ({ items, onItemClick }) => {
  const handleClick = useCallback((item) => {
    onItemClick(item);
  }, [onItemClick]);
  
  return (
    <div>
      {items.map(item => (
        <button 
          key={item.id}
          onClick={() => handleClick(item)}
        >
          {item.name}
        </button>
      ))}
    </div>
  );
};

// ❌ ไม่ดี - สร้าง functions ใหม่ทุกครั้ง
const BadButtonList = ({ items, onItemClick }) => {
  return (
    <div>
      {items.map(item => (
        <button 
          key={item.id}
          onClick={() => onItemClick(item)} // สร้าง function ใหม่ทุกครั้ง
        >
          {item.name}
        </button>
      ))}
    </div>
  );
};
```

### 3. Code Splitting
```typescript
// ✅ ดี - ใช้ lazy loading สำหรับ large components
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

const App = () => (
  <div>
    <header>Header</header>
    <main>
      <React.Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </React.Suspense>
    </main>
  </div>
);

// ✅ ดี - ใช้ lazy loading สำหรับ routes
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

const App = () => (
  <BrowserRouter>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Suspense>
  </BrowserRouter>
);
```

## 🔒 การจัดการ Error

### 1. Error Boundaries
```typescript
// ✅ ดี - ใช้ Error Boundaries สำหรับจัดการ errors
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // ส่ง error ไปยัง monitoring service
    reportError(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong.</h2>
          <details>
            {this.state.error && this.state.error.toString()}
          </details>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// การใช้งาน
const App = () => (
  <ErrorBoundary>
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Router>
  </ErrorBoundary>
);
```

### 2. Async Error Handling
```typescript
// ✅ ดี - จัดการ async errors อย่างเป็นระบบ
const useAsyncOperation = () => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const execute = async (operation) => {
    try {
      setLoading(true);
      setError(null);
      const result = await operation();
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  return { execute, loading, error };
};

// การใช้งาน
const DataFetcher = () => {
  const { execute, loading, error } = useAsyncOperation();
  const [data, setData] = useState(null);
  
  const fetchData = async () => {
    try {
      const result = await execute(() => fetch('/api/data').then(r => r.json()));
      setData(result);
    } catch (err) {
      // Error ถูกจัดการใน hook แล้ว
    }
  };
  
  return (
    <div>
      <button onClick={fetchData} disabled={loading}>
        {loading ? 'Loading...' : 'Fetch Data'}
      </button>
      {error && <div className="error">Error: {error}</div>}
      {data && <div>Data loaded: {data.length} items</div>}
    </div>
  );
};
```

## 🧪 Testing Best Practices

### 1. Component Testing
```typescript
// ✅ ดี - ทดสอบ behavior ไม่ใช่ implementation
import { render, screen, fireEvent } from '@testing-library/react';
import { Counter } from './Counter';

describe('Counter', () => {
  it('increments count when increment button is clicked', () => {
    render(<Counter />);
    
    const button = screen.getByRole('button', { name: /increment/i });
    const count = screen.getByText(/count: 0/i);
    
    fireEvent.click(button);
    
    expect(count).toHaveTextContent('count: 1');
  });
  
  it('does not increment when disabled', () => {
    render(<Counter disabled />);
    
    const button = screen.getByRole('button', { name: /increment/i });
    
    expect(button).toBeDisabled();
  });
});

// ❌ ไม่ดี - ทดสอบ implementation details
import Counter from './Counter';

describe('Counter', () => {
  it('calls setState with correct value', () => {
    const mockSetState = jest.fn();
    jest.spyOn(React, 'useState').mockImplementation(() => [0, mockSetState]);
    
    render(<Counter />);
    
    fireEvent.click(screen.getByRole('button'));
    
    expect(mockSetState).toHaveBeenCalledWith(1); // ทดสอบ implementation
  });
});
```

### 2. Integration Testing
```typescript
// ✅ ดี - ทดสอบการทำงานร่วมกัน
import { render, screen, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { UserList } from './UserList';
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' },
      ])
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('UserList Integration', () => {
  it('displays users from API', async () => {
    const queryClient = new QueryClient();
    
    render(
      <QueryClientProvider client={queryClient}>
        <UserList />
      </QueryClientProvider>
    );
    
    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
      expect(screen.getByText('Jane')).toBeInTheDocument();
    });
  });
});
```

## 🎨 Styling Best Practices

### 1. CSS-in-JS
```typescript
// ✅ ดี - ใช้ styled-components หรือ emotion
import styled from '@emotion/styled';

const Button = styled.button`
  background-color: ${props => props.variant === 'primary' ? '#007bff' : '#6c757d'};
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    opacity: 0.8;
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

// การใช้งาน
const App = () => (
  <div>
    <Button variant="primary">Primary Button</Button>
    <Button>Secondary Button</Button>
  </div>
);
```

### 2. Utility-First CSS
```typescript
// ✅ ดี - ใช้ Tailwind CSS
const Card = ({ title, children, className = '' }) => (
  <div className={`bg-white rounded-lg shadow-md p-6 ${className}`}>
    <h2 className="text-xl font-semibold mb-4">{title}</h2>
    <div className="text-gray-700">{children}</div>
  </div>
);

// การใช้งาน
const App = () => (
  <div className="max-w-md mx-auto p-4">
    <Card title="Welcome" className="border-2 border-blue-500">
      <p>This is a card component.</p>
    </Card>
  </div>
);
```

## 📝 Documentation Best Practices

### 1. Component Documentation
```typescript
/**
 * Button component with different variants and sizes
 * 
 * @example
 * ```tsx
 * <Button variant="primary" size="large" onClick={handleClick}>
 *   Click me
 * </Button>
 * ```
 */
interface ButtonProps {
  /** Button variant */
  variant?: 'primary' | 'secondary' | 'danger';
  /** Button size */
  size?: 'small' | 'medium' | 'large';
  /** Button content */
  children: React.ReactNode;
  /** Click handler */
  onClick?: () => void;
  /** Disable button */
  disabled?: boolean;
}

/**
 * Button component
 */
export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'medium',
  children,
  onClick,
  disabled = false,
}) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

### 2. Code Comments
```typescript
// ✅ ดี - คอมเมนต์ที่อธิบาย "why" ไม่ใช่ "what"
const calculateDiscount = (price, discountPercentage) => {
  // คำนวณส่วนลดตามนโยบายราคาที่ลดลงตามปริมาณ
  const baseDiscount = price * (discountPercentage / 100);
  
  // เพิ่มส่วนลดพิเศษสำหรับลูกค้า VIP
  const vipBonus = isVipCustomer() ? baseDiscount * 0.1 : 0;
  
  return baseDiscount + vipBonus;
};

// ❌ ไม่ดี - คอมเมนต์ที่บอก "what"
const calculateDiscount = (price, discountPercentage) => {
  // คูณราคากับเปอร์เซ็นต์ส่วนลด
  const baseDiscount = price * (discountPercentage / 100);
  
  // คูณส่วนลดพื้นฐานกับ 0.1 สำหรับ VIP
  const vipBonus = isVipCustomer() ? baseDiscount * 0.1 : 0;
  
  return baseDiscount + vipBonus;
};
```

## 🔄 Git Best Practices

### 1. Commit Messages
```
✅ ดี
feat(auth): add JWT authentication
fix(api): resolve timeout issue
docs(readme): update installation guide
style(components): fix button alignment
refactor(utils): simplify date formatting
test(user): add integration tests
chore(deps): update react to v18

❌ ไม่ดี
fixed bug
update
wip
stuff
```

### 2. Branch Strategy
```bash
# Feature branches
git checkout -b feature/user-authentication
git checkout -b feature/payment-integration

# Bugfix branches
git checkout -b bugfix/login-validation-error
git checkout -b bugfix/memory-leak-in-dashboard

# Release branches
git checkout -b release/v1.2.0

# Hotfix branches
git checkout -b hotfix/critical-security-patch
```

## 📊 Monitoring & Analytics

### 1. Performance Monitoring
```typescript
// ✅ ดี - ติดตาม performance metrics
const usePerformanceTracking = () => {
  useEffect(() => {
    // ติดตาม page load time
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        if (entry.entryType === 'navigation') {
          analytics.track('page_load_time', {
            duration: entry.duration,
            page: window.location.pathname,
          });
        }
      });
    });
    
    observer.observe({ entryTypes: ['navigation'] });
    
    return () => observer.disconnect();
  }, []);
};
```

### 2. Error Tracking
```typescript
// ✅ ดี - ติดตาม errors
const useErrorTracking = () => {
  useEffect(() => {
    const handleError = (event) => {
      analytics.track('javascript_error', {
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        stack: event.error?.stack,
      });
    };
    
    window.addEventListener('error', handleError);
    
    return () => window.removeEventListener('error', handleError);
  }, []);
};
```

---
*Best Practices Guide v1.0*  
*Last updated: 2024-01-20*  
*Contact: tech-lead@example.com*
