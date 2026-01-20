# 🎣 Basic Hooks - useState, useEffect, useContext

## 📋 สารบัฐ
- [useState](#-usestate)
- [useEffect](#-useeffect)
- [useContext](#-usecontext)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🔄 useState

### 📦 พื้นฐาน useState
```typescript
// Basic syntax
const [state, setState] = useState(initialValue);

// Number
const [count, setCount] = useState(0);

// String
const [name, setName] = useState('John');

// Boolean
const [isVisible, setIsVisible] = useState(true);

// Object
const [user, setUser] = useState({ name: 'John', age: 30 });

// Array
const [items, setItems] = useState([1, 2, 3]);

// Null/Undefined
const [data, setData] = useState(null);
const [loading, setLoading] = useState(undefined);
```

### 🎯 การอัปเดต State
```typescript
// Direct update
const increment = () => {
  setCount(count + 1);
};

// Functional update (recommended)
const increment = () => {
  setCount(prevCount => prevCount + 1);
};

// Object update
const updateUser = (newName: string) => {
  setUser(prevUser => ({ ...prevUser, name: newName }));
};

// Array update
const addItem = (newItem: string) => {
  setItems(prevItems => [...prevItems, newItem]);
};

// Remove item
const removeItem = (index: number) => {
  setItems(prevItems => prevItems.filter((_, i) => i !== index));
};
```

### 🏷️ TypeScript กับ useState
```typescript
// Explicit type
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>('');
const [user, setUser] = useState<User | null>(null);

// Interface
interface User {
  id: string;
  name: string;
  email: string;
}

const [user, setUser] = useState<User>({
  id: '1',
  name: 'John',
  email: 'john@example.com'
});

// Union type
const [status, setStatus] = useState<'loading' | 'success' | 'error'>('loading');
```

### 🔄 Lazy Initialization
```typescript
// ❌ Bad - runs on every render
const [data, setData] = useState(expensiveCalculation());

// ✅ Good - runs only once
const [data, setData] = useState(() => expensiveCalculation());

// With parameters
const [data, setData] = useState(() => calculateInitialValue(props.id));
```

### 📊 Complex State Examples
```typescript
// Form state
function useForm(initialValues: any) {
  const [values, setValues] = useState(initialValues);
  
  const setValue = (name: string, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
  };
  
  const setValues = (newValues: any) => {
    setValues(prev => ({ ...prev, ...newValues }));
  };
  
  const reset = () => {
    setValues(initialValues);
  };
  
  return { values, setValue, setValues, reset };
}

// Usage
function ContactForm() {
  const { values, setValue } = useForm({
    name: '',
    email: '',
    message: ''
  });
  
  return (
    <form>
      <input
        value={values.name}
        onChange={(e) => setValue('name', e.target.value)}
        placeholder="Name"
      />
      <input
        value={values.email}
        onChange={(e) => setValue('email', e.target.value)}
        placeholder="Email"
      />
      <textarea
        value={values.message}
        onChange={(e) => setValue('message', e.target.value)}
        placeholder="Message"
      />
    </form>
  );
}
```

## 🔄 useEffect

### 📦 พื้นฐาน useEffect
```typescript
// Run after every render
useEffect(() => {
  console.log('Component rendered');
});

// Run only once (mount)
useEffect(() => {
  console.log('Component mounted');
}, []);

// Run when dependencies change
useEffect(() => {
  console.log('Count changed:', count);
}, [count]);

// With cleanup
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Timer tick');
  }, 1000);
  
  return () => {
    console.log('Cleanup timer');
    clearInterval(timer);
  };
}, []);
```

### 🎯 การใช้ useEffect ทั่วไป
```typescript
// Data fetching
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;
  
  return <div>{user.name}</div>;
}

// DOM manipulation
function Tooltip({ text, children }: { text: string; children: React.ReactNode }) {
  const [isVisible, setIsVisible] = useState(false);
  const tooltipRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const tooltip = tooltipRef.current;
    if (!tooltip) return;
    
    const updatePosition = () => {
      const rect = tooltip.getBoundingClientRect();
      // Update tooltip position based on viewport
    };
    
    updatePosition();
    window.addEventListener('resize', updatePosition);
    
    return () => {
      window.removeEventListener('resize', updatePosition);
    };
  }, [isVisible]);
  
  return (
    <div className="tooltip-container">
      {children}
      {isVisible && (
        <div ref={tooltipRef} className="tooltip">
          {text}
        </div>
      )}
    </div>
  );
}
```

### 🔄 Multiple Effects
```typescript
function Component() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  // Effect for count
  useEffect(() => {
    console.log('Count changed:', count);
    document.title = `Count: ${count}`;
  }, [count]);
  
  // Effect for name
  useEffect(() => {
    console.log('Name changed:', name);
    localStorage.setItem('name', name);
  }, [name]);
  
  // Effect for cleanup
  useEffect(() => {
    const handleKeyPress = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        setCount(0);
        setName('');
      }
    };
    
    window.addEventListener('keydown', handleKeyPress);
    
    return () => {
      window.removeEventListener('keydown', handleKeyPress);
    };
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Name: {name}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter name"
      />
    </div>
  );
}
```

### ⚠️ useEffect Edge Cases
```typescript
// ❌ Bad - infinite loop
function BadEffect() {
  const [data, setData] = useState({});
  
  useEffect(() => {
    setData({}); // Triggers effect again
  }, [data]);
  
  return <div></div>;
}

// ✅ Good - proper dependencies
function GoodEffect() {
  const [data, setData] = useState({});
  const [trigger, setTrigger] = useState(0);
  
  useEffect(() => {
    fetchData().then(setData);
  }, [trigger]); // Use trigger instead of data
  
  return <div></div>;
}

// ❌ Bad - stale closure
function BadClosure() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // Always 0
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  return <div>{count}</div>;
}

// ✅ Good - functional update
function GoodClosure() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1); // Functional update
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  return <div>{count}</div>;
}
```

## 🔄 useContext

### 📦 พื้นฐาน useContext
```typescript
// Create context
const ThemeContext = createContext('light');

// Provider component
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

// Consumer component
function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
    Toolbar
  </div>;
}
```

### 🎯 Context กับ TypeScript
```typescript
// Define context type
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// Create context with default value
const ThemeContext = createContext<ThemeContextType>({
  theme: 'light',
  toggleTheme: () => {}
});

// Provider component
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const value = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer component
function ThemeButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        background: theme === 'dark' ? '#fff' : '#333',
        color: theme === 'dark' ? '#333' : '#fff'
      }}
    >
      Toggle Theme
    </button>
  );
}
```

### 🔄 Multiple Contexts
```typescript
// Auth context
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType>({
  user: null,
  login: async () => {},
  logout: () => {}
});

// Theme context
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType>({
  theme: 'light',
  toggleTheme: () => {}
});

// App with multiple providers
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <Router>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/profile" element={<Profile />} />
          </Routes>
        </Router>
      </ThemeProvider>
    </AuthProvider>
  );
}

// Component using multiple contexts
function Header() {
  const { user, logout } = useContext(AuthContext);
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <header>
      <h1>My App</h1>
      {user ? (
        <div>
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <Link to="/login">Login</Link>
      )}
      <button onClick={toggleTheme}>
        Current theme: {theme}
      </button>
    </header>
  );
}
```

### 🎯 Context Use Cases
```typescript
// User context
const UserContext = createContext<User | null>(null);

function UserProfile() {
  const user = useContext(UserContext);
  
  if (!user) {
    return <div>Please login</div>;
  }
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Language context
const LanguageContext = createContext('en');

function Greeting() {
  const language = useContext(LanguageContext);
  
  const greetings = {
    en: 'Hello',
    es: 'Hola',
    fr: 'Bonjour',
    th: 'สวัสดี'
  };
  
  return <h1>{greetings[language] || 'Hello'}</h1>;
}

// Settings context
interface SettingsContextType {
  notifications: boolean;
  darkMode: boolean;
  language: string;
  updateSettings: (settings: Partial<SettingsContextType>) => void;
}

const SettingsContext = createContext<SettingsContextType>({
  notifications: true,
  darkMode: false,
  language: 'en',
  updateSettings: () => {}
});

function Notifications() {
  const { notifications, updateSettings } = useContext(SettingsContext);
  
  return (
    <div>
      <label>
        <input
          type="checkbox"
          checked={notifications}
          onChange={(e) => updateSettings({ notifications: e.target.checked })}
        />
        Enable notifications
      </label>
    </div>
  );
}
```

## 🎯 Use Cases

### 1. **Form with useState and useEffect**
```typescript
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitStatus, setSubmitStatus] = useState<'idle' | 'success' | 'error'>('idle');
  
  // Auto-save to localStorage
  useEffect(() => {
    localStorage.setItem('formData', JSON.stringify(formData));
  }, [formData]);
  
  // Load from localStorage on mount
  useEffect(() => {
    const saved = localStorage.getItem('formData');
    if (saved) {
      setFormData(JSON.parse(saved));
    }
  }, []);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    try {
      await submitForm(formData);
      setSubmitStatus('success');
      setFormData({ name: '', email: '', message: '' });
    } catch (error) {
      setSubmitStatus('error');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
        placeholder="Name"
        required
      />
      <input
        value={formData.email}
        onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
        placeholder="Email"
        type="email"
        required
      />
      <textarea
        value={formData.message}
        onChange={(e) => setFormData(prev => ({ ...prev, message: e.target.value }))}
        placeholder="Message"
        required
      />
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Sending...' : 'Send'}
      </button>
      {submitStatus === 'success' && <p>Message sent!</p>}
      {submitStatus === 'error' && <p>Error sending message</p>}
    </form>
  );
}
```

### 2. **Real-time data with useEffect**
```typescript
function RealtimeData() {
  const [data, setData] = useState([]);
  const [connectionStatus, setConnectionStatus] = useState<'connecting' | 'connected' | 'disconnected'>('connecting');
  const wsRef = useRef<WebSocket | null>(null);
  
  useEffect(() => {
    // Connect to WebSocket
    wsRef.current = new WebSocket('ws://localhost:8080');
    
    wsRef.current.onopen = () => {
      setConnectionStatus('connected');
      console.log('Connected to WebSocket');
    };
    
    wsRef.current.onmessage = (event) => {
      const newData = JSON.parse(event.data);
      setData(prevData => [...prevData, newData]);
    };
    
    wsRef.current.onclose = () => {
      setConnectionStatus('disconnected');
      console.log('Disconnected from WebSocket');
    };
    
    wsRef.current.onerror = (error) => {
      console.error('WebSocket error:', error);
      setConnectionStatus('disconnected');
    };
    
    // Cleanup
    return () => {
      if (wsRef.current) {
        wsRef.current.close();
      }
    };
  }, []);
  
  const reconnect = () => {
    setConnectionStatus('connecting');
    // Trigger reconnection by creating new connection
    if (wsRef.current) {
      wsRef.current.close();
    }
  };
  
  return (
    <div>
      <h2>Real-time Data</h2>
      <p>Status: {connectionStatus}</p>
      {connectionStatus === 'disconnected' && (
        <button onClick={reconnect}>Reconnect</button>
      )}
      <ul>
        {data.map((item, index) => (
          <li key={index}>{JSON.stringify(item)}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 3. **Theme switcher with useContext**
```typescript
const ThemeContext = createContext({
  theme: 'light' as 'light' | 'dark',
  toggleTheme: () => {}
});

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  useEffect(() => {
    document.body.className = theme;
  }, [theme]);
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemeSwitcher() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        background: theme === 'dark' ? '#fff' : '#333',
        color: theme === 'dark' ? '#333' : '#fff',
        border: 'none',
        padding: '8px 16px',
        borderRadius: '4px',
        cursor: 'pointer'
      }}
    >
      Switch to {theme === 'light' ? 'dark' : 'light'} theme
    </button>
  );
}

function ThemedComponent() {
  const { theme } = useContext(ThemeContext);
  
  return (
    <div
      style={{
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#333',
        padding: '20px',
        borderRadius: '8px',
        margin: '10px 0'
      }}
    >
      <h3>Themed Component</h3>
      <p>Current theme: {theme}</p>
    </div>
  );
}
```

## ⚠️ Edge Cases

### 1. **useState with objects**
```typescript
// ❌ Bad - mutating state directly
function BadStateUpdate() {
  const [user, setUser] = useState({ name: 'John', age: 30 });
  
  const updateAge = () => {
    user.age = user.age + 1; // Direct mutation
    setUser(user); // Won't trigger re-render
  };
  
  return <button onClick={updateAge}>Update Age</button>;
}

// ✅ Good - creating new object
function GoodStateUpdate() {
  const [user, setUser] = useState({ name: 'John', age: 30 });
  
  const updateAge = () => {
    setUser(prevUser => ({ ...prevUser, age: prevUser.age + 1 }));
  };
  
  return <button onClick={updateAge}>Update Age</button>;
}
```

### 2. **useEffect dependency arrays**
```typescript
// ❌ Bad - missing dependencies
function BadEffect() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + step); // Uses step but not in dependencies
    }, 1000);
    
    return () => clearInterval(timer);
  }, [count]); // Missing step dependency
  
  return <div></div>;
}

// ✅ Good - all dependencies included
function GoodEffect() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + step); // Functional update
    }, 1000);
    
    return () => clearInterval(timer);
  }, [step]);
  
  return <div></div>;
}
```

### 3. **Context performance**
```typescript
// ❌ Bad - context value changes too often
function BadProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  // Object created on every render
  const value = {
    user,
    setUser,
    theme,
    setTheme
  };
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ Good - memoized context value
function GoodProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  // Memoized value
  const value = useMemo(() => ({
    user,
    setUser,
    theme,
    setTheme
  }), [user, theme]);
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}
```

### 4. **Context with default values**
```typescript
// ❌ Bad - undefined context
function BadConsumer() {
  const value = useContext(MyContext);
  return <div>{value.someProperty}</div>; // Error if value is undefined
}

// ✅ Good - handle undefined context
function GoodConsumer() {
  const value = useContext(MyContext);
  
  if (!value) {
    throw new Error('Component must be used within MyContext.Provider');
  }
  
  return <div>{value.someProperty}</div>;
}

// ✅ Better - provide default value
const MyContext = createContext({
  someProperty: 'default',
  someMethod: () => {}
});
```

## 🎯 สรุป

### ✅ สิ่งที่ควรรู้
- **useState**: จัดการ state ใน functional components
- **useEffect**: จัดการ side effects และ lifecycle
- **useContext**: แชร์ข้อมูลระหว่าง components โดยไม่ต้อง prop drilling
- ต้องระวัง stale closures และ dependency arrays
- Context values ควรถูก memoize สำหรับ performance

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Advanced Hooks** ใน [ADVANCED_HOOKS.md](ADVANCED_HOOKS.md)
2. เรียนรู้ **Custom Hooks** ใน [CUSTOM_HOOKS.md](CUSTOM_HOOKS.md)
3. ทดลองสร้าง custom hooks ของคุณเอง

### 💡 แนวทางการฝึก
- สร้าง forms ที่ซับซ้อนด้วย useState และ useEffect
- ฝึกใช้ context สำหรับ theme, auth, หรือ settings
- ทดลองสร้าง data fetching hooks
- ฝึกจัดการ edge cases ของ basic hooks

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
