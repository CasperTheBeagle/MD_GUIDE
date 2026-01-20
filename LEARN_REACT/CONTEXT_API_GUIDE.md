# 🎯 React Context API คู่มือสอนใช้งาน

## 📋 สารบัญ
- [Context API คืออะไร](#-context-api-คืออะไร)
- [Use Cases หลัก](#-use-cases-หลัก)
- [Edge Cases และวิธีจัดการ](#-edge-cases-และวิธีจัดการ)
- [ตัวอย่างการใช้งาน](#-ตัวอย่างการใช้งาน)
- [Performance Optimization](#-performance-optimization)
- [Best Practices](#-best-practices)
- [Common Pitfalls](#-common-pitfalls)

## 🎯 Context API คืออะไร

Context API เป็นวิธีการส่งผ่านข้อมูลผ่าน component tree โดยไม่ต้องส่ง props ผ่านทุก level ทำให้สามารถแชร์ข้อมูลระหว่าง components ที่อยู่ห่างกันได้

### ปัญหาที่ Context API แก้ไข
```typescript
// ❌ ไม่ดี - Prop Drilling
function App() {
  const user = { name: "John", role: "admin" };
  return <Header user={user} />;
}

function Header({ user }) {
  return <Navigation user={user} />;
}

function Navigation({ user }) {
  return <UserProfile user={user} />;
}

function UserProfile({ user }) {
  return <span>{user.name}</span>;
}

// ✅ ดี - ใช้ Context API
const UserContext = createContext();

function App() {
  const user = { name: "John", role: "admin" };
  return (
    <UserContext.Provider value={user}>
      <Header />
    </UserContext.Provider>
  );
}

function Header() {
  return <Navigation />;
}

function Navigation() {
  return <UserProfile />;
}

function UserProfile() {
  const user = useContext(UserContext);
  return <span>{user.name}</span>;
}
```

## 🚀 Use Cases หลัก

### 1. การจัดการ Theme (Dark/Light Mode)
```typescript
// ThemeContext.tsx
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

// การใช้งาน
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

function Header() {
  const { theme, toggleTheme } = useTheme();
  return (
    <header className={theme}>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}
```

### 2. การจัดการ Authentication
```typescript
// AuthContext.tsx
interface User {
  id: string;
  name: string;
  email: string;
  role: string;
}

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
  error: string | null;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const login = async (email: string, password: string) => {
    setIsLoading(true);
    setError(null);
    
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });
      
      if (!response.ok) {
        throw new Error('Login failed');
      }
      
      const userData = await response.json();
      setUser(userData);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('token');
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading, error }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

### 3. การจัดการ Shopping Cart
```typescript
// CartContext.tsx
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartContextType {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  total: number;
}

const CartContext = createContext<CartContextType | undefined>(undefined);

export const CartProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [items, setItems] = useState<CartItem[]>([]);

  const addItem = (item: Omit<CartItem, 'quantity'>) => {
    setItems(prev => {
      const existing = prev.find(i => i.id === item.id);
      if (existing) {
        return prev.map(i => 
          i.id === item.id 
            ? { ...i, quantity: i.quantity + 1 }
            : i
        );
      }
      return [...prev, { ...item, quantity: 1 }];
    });
  };

  const removeItem = (id: string) => {
    setItems(prev => prev.filter(item => item.id !== id));
  };

  const updateQuantity = (id: string, quantity: number) => {
    if (quantity <= 0) {
      removeItem(id);
      return;
    }
    setItems(prev => 
      prev.map(item => 
        item.id === id 
          ? { ...item, quantity }
          : item
      )
    );
  };

  const clearCart = () => {
    setItems([]);
  };

  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);

  return (
    <CartContext.Provider value={{ 
      items, 
      addItem, 
      removeItem, 
      updateQuantity, 
      clearCart, 
      total 
    }}>
      {children}
    </CartContext.Provider>
  );
};

export const useCart = () => {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
};
```

### 4. การจัดการ Notifications
```typescript
// NotificationContext.tsx
interface Notification {
  id: string;
  type: 'success' | 'error' | 'warning' | 'info';
  message: string;
  duration?: number;
}

interface NotificationContextType {
  notifications: Notification[];
  addNotification: (notification: Omit<Notification, 'id'>) => void;
  removeNotification: (id: string) => void;
  clearNotifications: () => void;
}

const NotificationContext = createContext<NotificationContextType | undefined>(undefined);

export const NotificationProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  const addNotification = (notification: Omit<Notification, 'id'>) => {
    const id = Date.now().toString();
    const newNotification = { ...notification, id };
    
    setNotifications(prev => [...prev, newNotification]);
    
    // Auto remove after duration
    const duration = notification.duration || 5000;
    setTimeout(() => {
      removeNotification(id);
    }, duration);
  };

  const removeNotification = (id: string) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  };

  const clearNotifications = () => {
    setNotifications([]);
  };

  return (
    <NotificationContext.Provider value={{ 
      notifications, 
      addNotification, 
      removeNotification, 
      clearNotifications 
    }}>
      {children}
    </NotificationContext.Provider>
  );
};

export const useNotification = () => {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotification must be used within NotificationProvider');
  }
  return context;
};
```

## ⚠️ Edge Cases และวิธีจัดการ

### 1. Context Value เปลี่ยนแปลงบ่อยเกินไป
```typescript
// ❌ ไม่ดี - ทำให้ re-render บ่อย
function App() {
  const [count, setCount] = useState(0);
  
  return (
    <CountContext.Provider value={{ count, setCount }}>
      <Counter />
      <OtherComponent />
    </CountContext.Provider>
  );
}

// ✅ ดี - แยก value ที่เปลี่ยนบ่อย
function App() {
  const [count, setCount] = useState(0);
  
  // แยก setter ออกมา
  const contextValue = useMemo(() => ({ count }), [count]);
  const setters = useMemo(() => ({ setCount }), [setCount]);
  
  return (
    <CountContext.Provider value={contextValue}>
      <CountSettersContext.Provider value={setters}>
        <Counter />
        <OtherComponent />
      </CountSettersContext.Provider>
    </CountContext.Provider>
  );
}
```

### 2. การจัดการ Complex State
```typescript
// ❌ ไม่ดี - State ซับซ้อนใน context เดียว
const AppContext = createContext({
  user: null,
  theme: 'light',
  cart: [],
  notifications: [],
  // ... อีกมากมาย
});

// ✅ ดี - แยก context ตามความเป็นจริง
const UserContext = createContext();
const ThemeContext = createContext();
const CartContext = createContext();
const NotificationContext = createContext();

function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <CartProvider>
          <NotificationProvider>
            <Main />
          </NotificationProvider>
        </CartProvider>
      </ThemeProvider>
    </UserProvider>
  );
}
```

### 3. การจัดการ Async Operations
```typescript
// ✅ ดี - จัดการ async ใน context
const DataContext = createContext();

export const DataProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch('/api/data');
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, []);

  return (
    <DataContext.Provider value={{ data, loading, error, refetch: fetchData }}>
      {children}
    </DataContext.Provider>
  );
};
```

### 4. การจัดการ Default Values
```typescript
// ❌ ไม่ดี - ใช้ undefined และต้อง check ทุกที่
const ThemeContext = createContext(undefined);

function Component() {
  const theme = useContext(ThemeContext);
  if (!theme) {
    throw new Error('Component must be used within ThemeProvider');
  }
  return <div>{theme}</div>;
}

// ✅ ดี - ใช้ default value ที่มีความหมาย
const defaultTheme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
  },
  spacing: {
    small: '8px',
    medium: '16px',
    large: '24px',
  }
};

const ThemeContext = createContext(defaultTheme);

function Component() {
  const theme = useContext(ThemeContext);
  return <div style={{ color: theme.colors.primary }}>Hello</div>;
}
```

### 5. การจัดการ Context ใน SSR
```typescript
// ✅ ดี - จัดการ SSR อย่างถูกต้อง
const ThemeContext = createContext('light');

function App() {
  const [theme, setTheme] = useState('light');
  
  // ตรวจสอบว่าอยู่บน client หรือไม่
  useEffect(() => {
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
      setTheme(savedTheme);
    }
  }, []);

  return (
    <ThemeContext.Provider value={theme}>
      <Main />
    </ThemeContext.Provider>
  );
}
```

## 🎨 ตัวอย่างการใช้งาน

### 1. Multi-Context Application
```typescript
// contexts/index.ts
export { ThemeProvider, useTheme } from './ThemeContext';
export { AuthProvider, useAuth } from './AuthContext';
export { CartProvider, useCart } from './CartContext';
export { NotificationProvider, useNotification } from './NotificationContext';

// App.tsx
import { ThemeProvider, AuthProvider, CartProvider, NotificationProvider } from './contexts';

function App() {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>
          <NotificationProvider>
            <Router>
              <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/login" element={<Login />} />
                <Route path="/cart" element={<Cart />} />
              </Routes>
            </Router>
          </NotificationProvider>
        </CartProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}
```

### 2. Context Composition
```typescript
// สร้าง HOC สำหรับรวม providers
const withProviders = (providers: React.FC[]) => 
  (Component: React.FC) => 
    (props: any) => 
      providers.reduceRight(
        (acc, Provider) => 
          <Provider>{acc}</Provider>,
        <Component {...props} />
      );

// การใช้งาน
const providers = [
  ThemeProvider,
  AuthProvider,
  CartProvider,
  NotificationProvider
];

const AppWithProviders = withProviders(providers)(App);
```

### 3. Context Selector Pattern
```typescript
// สร้าง custom hook สำหรับเลือกเฉพาะส่วนที่ต้องการ
const useUser = () => {
  const { user, login, logout } = useAuth();
  return { user, login, logout };
};

const useCartItems = () => {
  const { items, addItem, removeItem } = useCart();
  return { items, addItem, removeItem };
};

const useCartTotal = () => {
  const { total } = useCart();
  return total;
};
```

## ⚡ Performance Optimization

### 1. ใช้ useMemo เพื่อหลีกเลี่ยง re-render
```typescript
// ✅ ดี - ใช้ useMemo
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);

  const value = useMemo(() => ({
    theme,
    toggleTheme
  }), [theme, toggleTheme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

### 2. แยก Context ที่มีการอัปเดตบ่อย
```typescript
// ✅ ดี - แยก context
const StaticDataContext = createContext(staticData);
const DynamicDataContext = createContext(dynamicData);

function App() {
  return (
    <StaticDataProvider value={staticData}>
      <DynamicDataProvider>
        <Main />
      </DynamicDataProvider>
    </StaticDataProvider>
  );
}
```

### 3. ใช้ React.memo กับ components ที่ใช้ context
```typescript
// ✅ ดี - ใช้ React.memo
const ExpensiveComponent = React.memo(({ data }) => {
  console.log('ExpensiveComponent rendered');
  return <div>{data}</div>;
});

function Parent() {
  const { data } = useDataContext();
  return <ExpensiveComponent data={data} />;
}
```

## 📋 Best Practices

### 1. ✅ ใช้ Context อย่างเหมาะสม
- **Theme**: สี, fonts, spacing
- **Authentication**: ข้อมูลผู้ใช้, สถานะการล็อกอิน
- **Shopping Cart**: รายการสินค้า, ราคารวม
- **Notifications**: ข้อความแจ้งเตือน
- **Language/Localization**: ภาษา, ข้อความแปล

### 2. ❌ หลีกเลี่ยงการใช้ Context
- **Local component state**: ข้อมูลที่ใช้เฉพาะ component และ children
- **Form data**: ข้อมูลฟอร์มที่ไม่ต้องแชร์ข้าม components
- **Server state**: ข้อมูลจาก server (ใช้ React Query, SWR)
- **High-frequency updates**: ข้อมูลที่อัปเดตบ่อย (ใช้ state management library)

### 3. 📝 ตั้งชื่อ Context อย่างชัดเจน
```typescript
// ✅ ดี - ชื่อชัดเจน
const UserContext = createContext();
const ThemeContext = createContext();
const CartContext = createContext();

// ❌ ไม่ดี - ชื่อไม่ชัดเจน
const Context1 = createContext();
const DataContext = createContext();
const AppContext = createContext();
```

### 4. 🎯 สร้าง Custom Hooks
```typescript
// ✅ ดี - สร้าง custom hooks
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

## 🚨 Common Pitfalls

### 1. ใช้ Context มากเกินไป
```typescript
// ❌ ไม่ดี - ใช้ context สำหรับทุกอย่าง
const FormContext = createContext({
  name: '',
  email: '',
  password: '',
  setName: () => {},
  setEmail: () => {},
  setPassword: () => {}
});

// ✅ ดี - ใช้ local state สำหรับฟอร์ม
function ContactForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  // ...
}
```

### 2. ไม่ใช้ default values
```typescript
// ❌ ไม่ดี - ไม่มี default value
const ThemeContext = createContext();

// ✅ ดี - มี default value
const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {}
});
```

### 3. ไม่ handle edge cases
```typescript
// ❌ ไม่ดี - ไม่ตรวจสอบ context
function Component() {
  const theme = useContext(ThemeContext);
  return <div>{theme}</div>; // อาจเป็น undefined
}

// ✅ ดี - ตรวจสอบ context
function Component() {
  const theme = useContext(ThemeContext);
  if (!theme) {
    throw new Error('Component must be used within ThemeProvider');
  }
  return <div>{theme}</div>;
}
```

### 4. อัปเดต context value ทุกครั้ง
```typescript
// ❌ ไม่ดี - สร้าง object ใหม่ทุกครั้ง
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// ✅ ดี - ใช้ useMemo
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
