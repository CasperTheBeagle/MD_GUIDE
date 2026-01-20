# 🎣 Custom Hooks - สร้าง Hooks ของคุณเอง

## 📋 สารบัฐ
- [Custom Hooks คืออะไร](#-custom-hooks-คืออะไร)
- [การสร้าง Custom Hooks](#-การสร้าง-custom-hooks)
- [Common Patterns](#-common-patterns)
- [Advanced Patterns](#-advanced-patterns)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🎯 Custom Hooks คืออะไร

Custom hooks เป็นฟังก์ชัน JavaScript ที่ชื่อขึ้นต้นด้วย "use" และสามารถเรียกใช้ hooks อื่นๆ ภายในได้

### ✨ ประโยชน์ของ Custom Hooks
- **Reuse stateful logic**: ใช้ logic ซ้ำได้โดยไม่ต้องเปลี่ยน component hierarchy
- **Separation of concerns**: แยก logic ออกจาก UI components
- **Testability**: ทดสอบ logic ได้ง่ายขึ้น
- **Composition**: ผสม hooks ต่างๆ เพื่อสร้าง functionality ใหม่

### 📋 กฎพื้นฐาน
- ชื่อต้องขึ้นต้นด้วย "use"
- สามารถเรียกใช้ hooks อื่นๆ ภายในได้
- ไม่ควรเรียกภายใน loops หรือ conditions
- ควร return consistent API

## 🏗️ การสร้าง Custom Hooks

### 📦 พื้นฐาน Custom Hook
```typescript
// Basic custom hook
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(c => c - 1);
  }, []);
  
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  return { count, increment, decrement, reset };
}

// Usage
function CounterComponent() {
  const { count, increment, decrement, reset } = useCounter(10);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### 🎯 Custom Hook กับ TypeScript
```typescript
// Define interface for hook return
interface UseCounterReturn {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  setCount: (value: number) => void;
}

// Custom hook with TypeScript
function useCounter(initialValue = 0): UseCounterReturn {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(c => c - 1);
  }, []);
  
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  return { count, increment, decrement, reset, setCount };
}
```

### 🔄 Custom Hook กับ Parameters
```typescript
// Hook with configuration options
interface UseToggleOptions {
  initial?: boolean;
  onToggle?: (value: boolean) => void;
}

function useToggle(options: UseToggleOptions = {}) {
  const { initial = false, onToggle } = options;
  const [value, setValue] = useState(initial);
  
  const toggle = useCallback(() => {
    const newValue = !value;
    setValue(newValue);
    onToggle?.(newValue);
  }, [value, onToggle]);
  
  const setTrue = useCallback(() => {
    setValue(true);
    onToggle?.(true);
  }, [onToggle]);
  
  const setFalse = useCallback(() => {
    setValue(false);
    onToggle?.(false);
  }, [onToggle]);
  
  return [value, toggle, setTrue, setFalse] as const;
}

// Usage
function ToggleComponent() {
  const [isOn, toggle, setTrue, setFalse] = useToggle({
    initial: false,
    onToggle: (value) => console.log('Toggled:', value)
  });
  
  return (
    <div>
      <p>Status: {isOn ? 'ON' : 'OFF'}</p>
      <button onClick={toggle}>Toggle</button>
      <button onClick={setTrue}>Set ON</button>
      <button onClick={setFalse}>Set OFF</button>
    </div>
  );
}
```

## 🎨 Common Patterns

### 1. **Data Fetching Hook**
```typescript
interface UseFetchState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

interface UseFetchReturn<T> extends UseFetchState<T> {
  refetch: () => Promise<void>;
  setData: (data: T) => void;
}

function useFetch<T>(url: string, options?: RequestInit): UseFetchReturn<T> {
  const [state, setState] = useState<UseFetchState<T>>({
    data: null,
    loading: true,
    error: null
  });
  
  const fetchData = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const response = await fetch(url, options);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({
        data: null,
        loading: false,
        error: error instanceof Error ? error.message : 'Unknown error'
      });
    }
  }, [url, options]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  const setData = useCallback((data: T) => {
    setState(prev => ({ ...prev, data, loading: false, error: null }));
  }, []);
  
  return { ...state, refetch: fetchData, setData };
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error, refetch } = useFetch<User>(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={refetch}>Refresh</button>
    </div>
  );
}
```

### 2. **Local Storage Hook**
```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  // Get from local storage then parse stored json or return initialValue
  const readValue = (): T => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  };
  
  const [storedValue, setStoredValue] = useState<T>(readValue);
  
  // Return a wrapped version of useState's setter function that ...
  // ... persists the new value to localStorage.
  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      // Allow value to be a function so we have the same API as useState
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      
      // Save state
      setStoredValue(valueToStore);
      
      // Save to local storage
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.warn(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);
  
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === key) {
        setStoredValue(e.newValue ? JSON.parse(e.newValue) : initialValue);
      }
    };
    
    window.addEventListener('storage', handleStorageChange);
    
    return () => {
      window.removeEventListener('storage', handleStorageChange);
    };
  }, [key, initialValue]);
  
  return [storedValue, setValue] as const;
}

// Usage
function SettingsComponent() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'en');
  
  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value as 'light' | 'dark')}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      
      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Español</option>
        <option value="fr">Français</option>
      </select>
    </div>
  );
}
```

### 3. **Debounce Hook**
```typescript
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 300);
  
  const { data: results, loading } = useFetch(
    debouncedSearchTerm ? `/api/search?q=${debouncedSearchTerm}` : null
  );
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      {loading && <div>Searching...</div>}
      {results && (
        <ul>
          {results.map((item: any) => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### 4. **Window Size Hook**
```typescript
interface WindowSize {
  width: number;
  height: number;
}

function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>({
    width: 0,
    height: 0
  });
  
  useEffect(() => {
    // Handler to call on window resize
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }
    
    // Add event listener
    window.addEventListener('resize', handleResize);
    
    // Call handler right away so state gets updated with initial window size
    handleResize();
    
    // Remove event listener on cleanup
    return () => window.removeEventListener('resize', handleResize);
  }, []); // Empty array ensures that effect is only run on mount and unmount
  
  return windowSize;
}

// Usage
function ResponsiveComponent() {
  const { width, height } = useWindowSize();
  
  const isMobile = width < 768;
  const isTablet = width >= 768 && width < 1024;
  const isDesktop = width >= 1024;
  
  return (
    <div>
      <p>Window size: {width} x {height}</p>
      <p>Device: {isMobile ? 'Mobile' : isTablet ? 'Tablet' : 'Desktop'}</p>
    </div>
  );
}
```

### 5. **Media Query Hook**
```typescript
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);
  
  useEffect(() => {
    const media = window.matchMedia(query);
    
    // Update the state with the current match
    if (media.matches !== matches) {
      setMatches(media.matches);
    }
    
    // Add listener
    const listener = () => setMatches(media.matches);
    media.addEventListener('change', listener);
    
    // Cleanup
    return () => media.removeEventListener('change', listener);
  }, [matches, query]);
  
  return matches;
}

// Usage
function MediaQueryComponent() {
  const isSmallScreen = useMediaQuery('(max-width: 768px)');
  const isDarkMode = useMediaQuery('(prefers-color-scheme: dark)');
  
  return (
    <div>
      <p>Small screen: {isSmallScreen ? 'Yes' : 'No'}</p>
      <p>Dark mode: {isDarkMode ? 'Yes' : 'No'}</p>
    </div>
  );
}
```

## 🎨 Advanced Patterns

### 1. **WebSocket Hook**
```typescript
interface UseWebSocketOptions {
  onOpen?: (event: Event) => void;
  onMessage?: (event: MessageEvent) => void;
  onError?: (event: Event) => void;
  onClose?: (event: CloseEvent) => void;
  shouldReconnect?: boolean;
  reconnectInterval?: number;
}

interface UseWebSocketReturn {
  sendMessage: (message: string) => void;
  lastMessage: MessageEvent | null;
  readyState: number;
  reconnect: () => void;
}

function useWebSocket(url: string, options: UseWebSocketOptions = {}): UseWebSocketReturn {
  const {
    onOpen,
    onMessage,
    onError,
    onClose,
    shouldReconnect = true,
    reconnectInterval = 3000
  } = options;
  
  const [lastMessage, setLastMessage] = useState<MessageEvent | null>(null);
  const [readyState, setReadyState] = useState<number>(WebSocket.CONNECTING);
  const webSocketRef = useRef<WebSocket | null>(null);
  const reconnectTimeoutRef = useRef<NodeJS.Timeout>();
  
  const connect = useCallback(() => {
    try {
      const ws = new WebSocket(url);
      webSocketRef.current = ws;
      
      ws.onopen = (event) => {
        setReadyState(WebSocket.OPEN);
        onOpen?.(event);
      };
      
      ws.onmessage = (event) => {
        setLastMessage(event);
        onMessage?.(event);
      };
      
      ws.onerror = (event) => {
        setReadyState(WebSocket.CLOSED);
        onError?.(event);
      };
      
      ws.onclose = (event) => {
        setReadyState(WebSocket.CLOSED);
        onClose?.(event);
        
        if (shouldReconnect) {
          reconnectTimeoutRef.current = setTimeout(connect, reconnectInterval);
        }
      };
    } catch (error) {
      console.error('WebSocket connection error:', error);
    }
  }, [url, onOpen, onMessage, onError, onClose, shouldReconnect, reconnectInterval]);
  
  const sendMessage = useCallback((message: string) => {
    if (webSocketRef.current?.readyState === WebSocket.OPEN) {
      webSocketRef.current.send(message);
    }
  }, []);
  
  const reconnect = useCallback(() => {
    if (webSocketRef.current) {
      webSocketRef.current.close();
    }
    connect();
  }, [connect]);
  
  useEffect(() => {
    connect();
    
    return () => {
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
      }
      if (webSocketRef.current) {
        webSocketRef.current.close();
      }
    };
  }, [connect]);
  
  return { sendMessage, lastMessage, readyState, reconnect };
}

// Usage
function ChatComponent() {
  const [messages, setMessages] = useState<string[]>([]);
  const [input, setInput] = useState('');
  
  const { sendMessage, lastMessage, readyState } = useWebSocket('ws://localhost:8080', {
    onMessage: (event) => {
      setMessages(prev => [...prev, event.data]);
    }
  });
  
  const handleSend = () => {
    if (input.trim()) {
      sendMessage(input);
      setInput('');
    }
  };
  
  return (
    <div>
      <div>
        {messages.map((msg, index) => (
          <div key={index}>{msg}</div>
        ))}
      </div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && handleSend()}
      />
      <button onClick={handleSend} disabled={readyState !== WebSocket.OPEN}>
        Send
      </button>
    </div>
  );
}
```

### 2. **Geolocation Hook**
```typescript
interface UseGeolocationOptions {
  enableHighAccuracy?: boolean;
  timeout?: number;
  maximumAge?: number;
}

interface UseGeolocationReturn {
  location: GeolocationPosition | null;
  error: GeolocationPositionError | null;
  loading: boolean;
  requestLocation: () => void;
}

function useGeolocation(options: UseGeolocationOptions = {}): UseGeolocationReturn {
  const [location, setLocation] = useState<GeolocationPosition | null>(null);
  const [error, setError] = useState<GeolocationPositionError | null>(null);
  const [loading, setLoading] = useState(false);
  
  const requestLocation = useCallback(() => {
    if (!navigator.geolocation) {
      setError(new GeolocationPositionError(1, 'Geolocation is not supported'));
      return;
    }
    
    setLoading(true);
    setError(null);
    
    navigator.geolocation.getCurrentPosition(
      (position) => {
        setLocation(position);
        setLoading(false);
      },
      (error) => {
        setError(error);
        setLoading(false);
      },
      options
    );
  }, [options]);
  
  useEffect(() => {
    requestLocation();
  }, [requestLocation]);
  
  return { location, error, loading, requestLocation };
}

// Usage
function LocationComponent() {
  const { location, error, loading, requestLocation } = useGeolocation({
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 0
  });
  
  if (loading) return <div>Getting location...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!location) return <div>No location data</div>;
  
  return (
    <div>
      <p>Latitude: {location.coords.latitude}</p>
      <p>Longitude: {location.coords.longitude}</p>
      <p>Accuracy: {location.coords.accuracy}m</p>
      <button onClick={requestLocation}>Update Location</button>
    </div>
  );
}
```

### 3. **Animation Hook**
```typescript
interface UseAnimationOptions {
  duration?: number;
  easing?: string;
  delay?: number;
}

function useAnimation(
  trigger: boolean,
  options: UseAnimationOptions = {}
): [React.CSSProperties, (style: React.CSSProperties) => void] {
  const { duration = 300, easing = 'ease-in-out', delay = 0 } = options;
  const [style, setStyle] = useState<React.CSSProperties>({});
  const [isAnimating, setIsAnimating] = useState(false);
  
  const animate = useCallback((newStyle: React.CSSProperties) => {
    setStyle(newStyle);
    setIsAnimating(true);
    
    setTimeout(() => {
      setIsAnimating(false);
    }, duration + delay);
  }, [duration, delay]);
  
  useEffect(() => {
    if (trigger) {
      animate({
        transition: `all ${duration}ms ${easing} ${delay}ms`,
        ...style
      });
    }
  }, [trigger, animate, duration, easing, delay, style]);
  
  return [
    {
      transition: `all ${duration}ms ${easing} ${delay}ms`,
      ...style
    },
    animate
  ];
}

// Usage
function AnimatedComponent() {
  const [isVisible, setIsVisible] = useState(false);
  const [style, animate] = useAnimation(isVisible, {
    duration: 500,
    easing: 'ease-out'
  });
  
  useEffect(() => {
    if (isVisible) {
      animate({
        opacity: 1,
        transform: 'translateY(0)'
      });
    } else {
      animate({
        opacity: 0,
        transform: 'translateY(-20px)'
      });
    }
  }, [isVisible, animate]);
  
  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        Toggle
      </button>
      <div style={style}>
        Animated Content
      </div>
    </div>
  );
}
```

### 4. **Form Hook**
```typescript
interface UseFormOptions<T> {
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit?: (values: T) => Promise<void> | void;
}

interface UseFormReturn<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  isValid: boolean;
  setFieldValue: (field: keyof T, value: T[keyof T]) => void;
  setFieldError: (field: keyof T, error: string) => void;
  setFieldTouched: (field: keyof T, touched: boolean) => void;
  handleSubmit: (e: React.FormEvent) => Promise<void>;
  resetForm: () => void;
}

function useForm<T extends Record<string, any>>(options: UseFormOptions<T>): UseFormReturn<T> {
  const { initialValues, validate, onSubmit } = options;
  
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const setFieldValue = useCallback((field: keyof T, value: T[keyof T]) => {
    setValues(prev => ({ ...prev, [field]: value }));
    
    if (touched[field]) {
      const newErrors = validate?.({ ...values, [field]: value }) || {};
      setErrors(newErrors);
    }
  }, [values, touched, validate]);
  
  const setFieldError = useCallback((field: keyof T, error: string) => {
    setErrors(prev => ({ ...prev, [field]: error }));
  }, []);
  
  const setFieldTouched = useCallback((field: keyof T, touchedValue: boolean) => {
    setTouched(prev => ({ ...prev, [field]: touchedValue }));
    
    if (touchedValue && validate) {
      const fieldErrors = validate(values);
      setErrors(prev => ({ ...prev, [field]: fieldErrors[field] || '' }));
    }
  }, [values, validate]);
  
  const isValid = useMemo(() => {
    if (!validate) return true;
    const validationErrors = validate(values);
    return Object.keys(validationErrors).length === 0;
  }, [values, validate]);
  
  const handleSubmit = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!onSubmit) return;
    
    // Validate all fields
    const validationErrors = validate?.(values) || {};
    setErrors(validationErrors);
    setTouched(Object.keys(values).reduce((acc, key) => ({ ...acc, [key]: true }), {}));
    
    if (Object.keys(validationErrors).length > 0) {
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      await onSubmit(values);
    } catch (error) {
      console.error('Submit error:', error);
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validate, onSubmit]);
  
  const resetForm = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    isValid,
    setFieldValue,
    setFieldError,
    setFieldTouched,
    handleSubmit,
    resetForm
  };
}

// Usage
function ContactForm() {
  const {
    values,
    errors,
    touched,
    isSubmitting,
    isValid,
    setFieldValue,
    setFieldTouched,
    handleSubmit,
    resetForm
  } = useForm({
    initialValues: {
      name: '',
      email: '',
      message: ''
    },
    validate: (values) => {
      const errors: Partial<Record<keyof typeof values, string>> = {};
      
      if (!values.name.trim()) {
        errors.name = 'Name is required';
      }
      
      if (!values.email.trim()) {
        errors.email = 'Email is required';
      } else if (!/\S+@\S+\.\S+/.test(values.email)) {
        errors.email = 'Email is invalid';
      }
      
      if (!values.message.trim()) {
        errors.message = 'Message is required';
      }
      
      return errors;
    },
    onSubmit: async (values) => {
      await submitContactForm(values);
      alert('Form submitted successfully!');
      resetForm();
    }
  });
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name</label>
        <input
          value={values.name}
          onChange={(e) => setFieldValue('name', e.target.value)}
          onBlur={() => setFieldTouched('name', true)}
        />
        {touched.name && errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <label>Email</label>
        <input
          type="email"
          value={values.email}
          onChange={(e) => setFieldValue('email', e.target.value)}
          onBlur={() => setFieldTouched('email', true)}
        />
        {touched.email && errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <label>Message</label>
        <textarea
          value={values.message}
          onChange={(e) => setFieldValue('message', e.target.value)}
          onBlur={() => setFieldTouched('message', true)}
        />
        {touched.message && errors.message && <span className="error">{errors.message}</span>}
      </div>
      
      <button type="submit" disabled={!isValid || isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## 🎯 Use Cases

### 1. **Authentication Hook**
```typescript
interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  loading: boolean;
}

interface UseAuthReturn extends AuthState {
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  register: (userData: RegisterData) => Promise<void>;
}

function useAuth(): UseAuthReturn {
  const [state, setState] = useState<AuthState>({
    user: null,
    token: null,
    isAuthenticated: false,
    loading: true
  });
  
  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      validateToken(token);
    } else {
      setState(prev => ({ ...prev, loading: false }));
    }
  }, []);
  
  const validateToken = async (token: string) => {
    try {
      const response = await fetch('/api/auth/validate', {
        headers: { Authorization: `Bearer ${token}` }
      });
      
      if (response.ok) {
        const user = await response.json();
        setState({
          user,
          token,
          isAuthenticated: true,
          loading: false
        });
      } else {
        localStorage.removeItem('token');
        setState(prev => ({ ...prev, loading: false }));
      }
    } catch (error) {
      localStorage.removeItem('token');
      setState(prev => ({ ...prev, loading: false }));
    }
  };
  
  const login = async (email: string, password: string) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    
    if (!response.ok) {
      throw new Error('Login failed');
    }
    
    const { user, token } = await response.json();
    localStorage.setItem('token', token);
    
    setState({
      user,
      token,
      isAuthenticated: true,
      loading: false
    });
  };
  
  const logout = () => {
    localStorage.removeItem('token');
    setState({
      user: null,
      token: null,
      isAuthenticated: false,
      loading: false
    });
  };
  
  const register = async (userData: RegisterData) => {
    const response = await fetch('/api/auth/register', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error('Registration failed');
    }
    
    await login(userData.email, userData.password);
  };
  
  return { ...state, login, logout, register };
}
```

### 2. **Infinite Scroll Hook**
```typescript
interface UseInfiniteScrollOptions {
  hasNextPage: boolean;
  fetchNextPage: () => Promise<void>;
  threshold?: number;
}

function useInfiniteScroll(options: UseInfiniteScrollOptions) {
  const { hasNextPage, fetchNextPage, threshold = 100 } = options;
  const [isFetching, setIsFetching] = useState(false);
  
  const handleScroll = useCallback(async () => {
    if (isFetching || !hasNextPage) return;
    
    const scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
    const scrollHeight = document.documentElement.scrollHeight || document.body.scrollHeight;
    const clientHeight = document.documentElement.clientHeight || window.innerHeight;
    
    if (scrollTop + clientHeight >= scrollHeight - threshold) {
      setIsFetching(true);
      await fetchNextPage();
      setIsFetching(false);
    }
  }, [isFetching, hasNextPage, fetchNextPage, threshold]);
  
  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);
  
  return { isFetching };
}
```

## ⚠️ Edge Cases

### 1. **Hook Dependencies**
```typescript
// ❌ Bad - missing dependencies
function useBadHook(value: string) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData(value).then(setData);
  }, []); // Missing value dependency
  
  return data;
}

// ✅ Good - proper dependencies
function useGoodHook(value: string) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData(value).then(setData);
  }, [value]); // Proper dependency
  
  return data;
}
```

### 2. **Conditional Hook Calls**
```typescript
// ❌ Bad - conditional hook call
function useBadConditionalHook(condition: boolean) {
  if (condition) {
    useState(0); // Don't do this!
  }
  
  useEffect(() => {
    // effect logic
  }, []);
}

// ✅ Good - always call hooks
function useGoodConditionalHook(condition: boolean) {
  const [value, setValue] = useState(0);
  
  useEffect(() => {
    if (condition) {
      // conditional logic inside effect
      setValue(1);
    }
  }, [condition]);
}
```

### 3. **Performance Issues**
```typescript
// ❌ Bad - creating new objects on every render
function useBadPerformance() {
  const [data, setData] = useState(null);
  
  return {
    data,
    setData,
    // New object created on every render
    actions: {
      refresh: () => setData(null),
      clear: () => setData(null)
    }
  };
}

// ✅ Good - memoized return value
function useGoodPerformance() {
  const [data, setData] = useState(null);
  
  const actions = useMemo(() => ({
    refresh: () => setData(null),
    clear: () => setData(null)
  }), []);
  
  return { data, setData, actions };
}
```

## 🎯 สรุป

### ✅ สิ่งที่ควรรู้
- Custom hooks คือฟังก์ชันที่ชื่อขึ้นต้นด้วย "use"
- สามารถใช้ hooks อื่นๆ ภายในได้
- ช่วยให้ logic ใช้ซ้ำได้และทดสอบง่าย
- ควรมี API ที่ consistent และ predictable

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Performance** ใน [PERFORMANCE.md](PERFORMANCE.md)
2. เรียนรู้ **Patterns** ใน [PATTERNS.md](PATTERNS.md)
3. ทดลอบสร้าง custom hooks ของคุณเอง

### 💡 แนวทางการฝึก
- สร้าง custom hooks สำหรับ logic ที่ใช้ซ้ำบ่อย
- ฝึกจัดการ dependencies อย่างถูกต้อง
- ทดสอบ custom hooks แยกจาก components
- ฝึกสร้าง hooks ที่มีประสิทธิภาพสูง

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2024*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
