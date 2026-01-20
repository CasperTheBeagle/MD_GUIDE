# 🎨 React Design Patterns - รูปแบบการออกแบบ

## 📋 สารบัฐ
- [Compound Components](#-compound-components)
- [Render Props](#-render-props)
- [Higher-Order Components](#-higher-order-components)
- [Custom Hooks Patterns](#-custom-hooks-patterns)
- [State Management Patterns](#-state-management-patterns)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🎯 Design Patterns คืออะไร

Design patterns คือวิธีการแก้ปัญหาที่พบบ่อยใน React ที่ได้รับการพิสูจน์แล้วว่าใช้ได้ผลดี

### ✨ ประโยชน์ของ Design Patterns
- **Reusable solutions**: ใช้วิธีแก้ปัญหาซ้ำได้
- **Best practices**: ตามมาตรฐานที่ยอมรับ
- **Maintainable code**: โค้ดที่อ่านง่ายและดูแลรักษาง่าย
- **Scalable architecture**: สร้าง application ที่ขยายได้

## 🧩 Compound Components

### 📦 พื้นฐาน Compound Components
```typescript
// Tab component with compound pattern
interface TabContextType {
  activeTab: string;
  setActiveTab: (tab: string) => void;
}

const TabContext = createContext<TabContextType>({
  activeTab: '',
  setActiveTab: () => {}
});

function Tabs({ children, defaultTab }: {
  children: React.ReactNode;
  defaultTab: string;
}) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabContext.Provider>
  );
}

function TabList({ children }: { children: React.ReactNode }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ id, children }: { id: string; children: React.ReactNode }) {
  const { activeTab, setActiveTab } = useContext(TabContext);
  const isActive = activeTab === id;
  
  return (
    <button
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

function TabPanels({ children }: { children: React.ReactNode }) {
  return <div className="tab-panels">{children}</div>;
}

function TabPanel({ id, children }: { id: string; children: React.ReactNode }) {
  const { activeTab } = useContext(TabContext);
  const isActive = activeTab === id;
  
  if (!isActive) return null;
  
  return <div className="tab-panel">{children}</div>;
}

// Usage
function App() {
  return (
    <Tabs defaultTab="profile">
      <TabList>
        <Tab id="profile">Profile</Tab>
        <Tab id="settings">Settings</Tab>
        <Tab id="notifications">Notifications</Tab>
      </TabList>
      <TabPanels>
        <TabPanel id="profile">
          <h2>Profile</h2>
          <p>User profile information</p>
        </TabPanel>
        <TabPanel id="settings">
          <h2>Settings</h2>
          <p>User settings</p>
        </TabPanel>
        <TabPanel id="notifications">
          <h2>Notifications</h2>
          <p>Notification preferences</p>
        </TabPanel>
      </TabPanels>
    </Tabs>
  );
}
```

### 🎯 Advanced Compound Components
```typescript
// Menu component with flexible API
interface MenuContextType {
  isOpen: boolean;
  setIsOpen: (open: boolean) => void;
  toggle: () => void;
}

const MenuContext = createContext<MenuContextType>({
  isOpen: false,
  setIsOpen: () => {},
  toggle: () => {}
});

function Menu({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(false);
  const toggle = () => setIsOpen(!isOpen);
  
  return (
    <MenuContext.Provider value={{ isOpen, setIsOpen, toggle }}>
      <div className="menu">{children}</div>
    </MenuContext.Provider>
  );
}

function MenuButton({ children }: { children: React.ReactNode }) {
  const { toggle } = useContext(MenuContext);
  
  return (
    <button onClick={toggle} className="menu-button">
      {children}
    </button>
  );
}

function MenuItems({ children }: { children: React.ReactNode }) {
  const { isOpen } = useContext(MenuContext);
  
  if (!isOpen) return null;
  
  return <div className="menu-items">{children}</div>;
}

function MenuItem({ children, onClick }: {
  children: React.ReactNode;
  onClick: () => void;
}) {
  const { setIsOpen } = useContext(MenuContext);
  
  const handleClick = () => {
    onClick();
    setIsOpen(false);
  };
  
  return (
    <button onClick={handleClick} className="menu-item">
      {children}
    </button>
  );
}

// Usage
function DropdownMenu() {
  return (
    <Menu>
      <MenuButton>
        Menu ▼
      </MenuButton>
      <MenuItems>
        <MenuItem onClick={() => console.log('Profile')}>
          Profile
        </MenuItem>
        <MenuItem onClick={() => console.log('Settings')}>
          Settings
        </MenuItem>
        <MenuItem onClick={() => console.log('Logout')}>
          Logout
        </MenuItem>
      </MenuItems>
    </Menu>
  );
}
```

## 🎨 Render Props

### 📦 พื้นฐาน Render Props
```typescript
// Mouse tracker with render props
interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  render: (position: MousePosition) => React.ReactNode;
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState<MousePosition>({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return render(position);
}

// Usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          <h1>Mouse Position</h1>
          <p>X: {x}, Y: {y}</p>
        </div>
      )}
    />
  );
}
```

### 🎯 Render Props กับ Children
```typescript
// Data fetcher with children as function
interface DataFetcherProps<T> {
  url: string;
  children: (data: {
    loading: boolean;
    error: string | null;
    data: T | null;
    refetch: () => void;
  }) => React.ReactNode;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [data, setData] = useState<T | null>(null);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error('Failed to fetch data');
      }
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return children({ loading, error, data, refetch: fetchData });
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  return (
    <DataFetcher url={`/api/users/${userId}`}>
      {({ loading, error, data, refetch }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error}</div>;
        if (!data) return <div>No data</div>;
        
        return (
          <div>
            <h2>{data.name}</h2>
            <p>{data.email}</p>
            <button onClick={refetch}>Refresh</button>
          </div>
        );
      }}
    </DataFetcher>
  );
}
```

### 🎯 Advanced Render Props
```typescript
// Form with render props
interface FormField {
  name: string;
  label: string;
  type: string;
  value: string;
  error?: string;
  touched: boolean;
}

interface FormProps<T> {
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit: (values: T) => void;
  children: (props: {
    values: T;
    errors: Partial<Record<keyof T, string>>;
    touched: Partial<Record<keyof T, boolean>>;
    isSubmitting: boolean;
    getFieldProps: (name: keyof T) => {
      value: string;
      onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
      onBlur: () => void;
    };
    handleSubmit: (e: React.FormEvent) => void;
  }) => React.ReactNode;
}

function Form<T extends Record<string, any>>({
  initialValues,
  validate,
  onSubmit,
  children
}: FormProps<T>) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const getFieldProps = (name: keyof T) => ({
    value: values[name] || '',
    onChange: (e: React.ChangeEvent<HTMLInputElement>) => {
      const newValues = { ...values, [name]: e.target.value };
      setValues(newValues);
      
      if (touched[name] && validate) {
        const fieldErrors = validate(newValues);
        setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
      }
    },
    onBlur: () => {
      setTouched(prev => ({ ...prev, [name]: true }));
      
      if (validate) {
        const fieldErrors = validate(values);
        setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
      }
    }
  });
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      setTouched(Object.keys(values).reduce((acc, key) => ({ ...acc, [key]: true }), {}));
      
      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }
    
    setIsSubmitting(true);
    
    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return children({
    values,
    errors,
    touched,
    isSubmitting,
    getFieldProps,
    handleSubmit
  });
}

// Usage
function ContactForm() {
  return (
    <Form
      initialValues={{ name: '', email: '', message: '' }}
      validate={(values) => {
        const errors: Partial<Record<keyof typeof values, string>> = {};
        
        if (!values.name.trim()) errors.name = 'Name is required';
        if (!values.email.trim()) errors.email = 'Email is required';
        if (!values.message.trim()) errors.message = 'Message is required';
        
        return errors;
      }}
      onSubmit={async (values) => {
        await submitForm(values);
        alert('Form submitted!');
      }}
    >
      {({ values, errors, getFieldProps, handleSubmit, isSubmitting }) => (
        <form onSubmit={handleSubmit}>
          <div>
            <label>Name</label>
            <input {...getFieldProps('name')} />
            {errors.name && <span className="error">{errors.name}</span>}
          </div>
          
          <div>
            <label>Email</label>
            <input type="email" {...getFieldProps('email')} />
            {errors.email && <span className="error">{errors.email}</span>}
          </div>
          
          <div>
            <label>Message</label>
            <textarea {...getFieldProps('message')} />
            {errors.message && <span className="error">{errors.message}</span>}
          </div>
          
          <button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Submitting...' : 'Submit'}
          </button>
        </form>
      )}
    </Form>
  );
}
```

## 🎨 Higher-Order Components

### 📦 พื้นฐาน HOC
```typescript
// Loading HOC
interface WithLoadingProps {
  loading: boolean;
}

function withLoading<P extends object>(
  Component: React.ComponentType<P>
) {
  return function WithLoadingComponent(props: P & WithLoadingProps) {
    const { loading, ...restProps } = props;
    
    if (loading) {
      return <div className="loading">Loading...</div>;
    }
    
    return <Component {...(restProps as P)} />;
  };
}

// Usage
const UserProfile = ({ user }: { user: User }) => (
  <div>
    <h2>{user.name}</h2>
    <p>{user.email}</p>
  </div>
);

const UserProfileWithLoading = withLoading(UserProfile);

// Error boundary HOC
interface WithErrorBoundaryProps {
  error?: Error;
  resetError?: () => void;
}

function withErrorBoundary<P extends object>(
  Component: React.ComponentType<P>,
  fallback?: React.ComponentType<{ error: Error; resetError: () => void }>
) {
  return function WithErrorBoundaryComponent(props: P) {
    return (
      <ErrorBoundary
        fallback={fallback}
        onError={(error, resetError) => (
          <Component {...props} error={error} resetError={resetError} />
        )}
      />
    );
  };
}

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
        <div>
          <h2>Something went wrong</h2>
          <details>
            {this.state.error.message}
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
```

### 🎯 Advanced HOCs
```typescript
// Data fetching HOC
interface WithDataProps<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

function withData<T, P extends object>(
  url: string | ((props: P) => string),
  Component: React.ComponentType<P & WithDataProps<T>>
) {
  return function WithDataComponent(props: P) {
    const [data, setData] = useState<T | null>(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState<string | null>(null);
    
    const fetchData = useCallback(async () => {
      setLoading(true);
      setError(null);
      
      try {
        const fetchUrl = typeof url === 'function' ? url(props) : url;
        const response = await fetch(fetchUrl);
        
        if (!response.ok) {
          throw new Error('Failed to fetch data');
        }
        
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    }, [props]);
    
    useEffect(() => {
      fetchData();
    }, [fetchData]);
    
    return (
      <Component
        {...props}
        data={data}
        loading={loading}
        error={error}
        refetch={fetchData}
      />
    );
  };
}

// Authentication HOC
interface WithAuthProps {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

function withAuth<P extends object>(
  Component: React.ComponentType<P & WithAuthProps>
) {
  return function WithAuthComponent(props: P) {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
      const token = localStorage.getItem('token');
      if (token) {
        validateToken(token).then(setUser).finally(() => setLoading(false));
      } else {
        setLoading(false);
      }
    }, []);
    
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
      setUser(user);
    };
    
    const logout = () => {
      localStorage.removeItem('token');
      setUser(null);
    };
    
    if (loading) {
      return <div>Loading...</div>;
    }
    
    return <Component {...props} user={user} login={login} logout={logout} />;
  };
}

// Usage
const ProtectedComponent = withAuth(function Dashboard({ user, logout }: {
  user: User | null;
  logout: () => void;
}) {
  if (!user) {
    return <div>Please login</div>;
  }
  
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
});
```

## 🎨 Custom Hooks Patterns

### 📦 State Management Hook
```typescript
// Generic state management hook
interface StateManager<T> {
  state: T;
  updateState: (updates: Partial<T>) => void;
  resetState: () => void;
}

function useStateManager<T>(initialState: T): StateManager<T> {
  const [state, setState] = useState<T>(initialState);
  
  const updateState = useCallback((updates: Partial<T>) => {
    setState(prev => ({ ...prev, ...updates }));
  }, []);
  
  const resetState = useCallback(() => {
    setState(initialState);
  }, [initialState]);
  
  return { state, updateState, resetState };
}

// Usage
function useApp() {
  const { state, updateState, resetState } = useStateManager({
    user: null,
    theme: 'light',
    notifications: []
  });
  
  return {
    ...state,
    setUser: (user: User) => updateState({ user }),
    setTheme: (theme: string) => updateState({ theme }),
    addNotification: (notification: Notification) => 
      updateState({ notifications: [...state.notifications, notification] }),
    resetApp: resetState
  };
}
```

### 📦 Event Manager Hook
```typescript
function useEventManager<T extends Record<string, (...args: any[]) => void>>(
  handlers: T
) {
  const handlersRef = useRef(handlers);
  
  useEffect(() => {
    handlersRef.current = handlers;
  }, [handlers]);
  
  const trigger = useCallback((event: keyof T, ...args: any[]) => {
    const handler = handlersRef.current[event];
    if (handler) {
      handler(...args);
    }
  }, []);
  
  return { trigger };
}

// Usage
function useTodoEvents() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const { trigger } = useEventManager({
    addTodo: (text: string) => {
      setTodos(prev => [...prev, {
        id: Date.now().toString(),
        text,
        completed: false
      }]);
    },
    toggleTodo: (id: string) => {
      setTodos(prev => prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      ));
    },
    deleteTodo: (id: string) => {
      setTodos(prev => prev.filter(todo => todo.id !== id));
    }
  });
  
  return { todos, trigger };
}
```

## 🎨 State Management Patterns

### 📦 Reducer Pattern
```typescript
// Generic reducer hook
function useReducer<T, A>(
  reducer: (state: T, action: A) => T,
  initialState: T
) {
  const [state, dispatch] = React.useReducer(reducer, initialState);
  
  const actions = useMemo(() => {
    const createActions = (dispatch: React.Dispatch<A>) => ({
      dispatch,
      // Add common actions here
    });
    
    return createActions(dispatch);
  }, [dispatch]);
  
  return { state, actions, dispatch };
}

// Usage with todo app
type TodoAction =
  | { type: 'ADD'; payload: string }
  | { type: 'TOGGLE'; payload: string }
  | { type: 'DELETE'; payload: string };

function todoReducer(state: Todo[], action: TodoAction): Todo[] {
  switch (action.type) {
    case 'ADD':
      return [...state, {
        id: Date.now().toString(),
        text: action.payload,
        completed: false
      }];
    case 'TOGGLE':
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case 'DELETE':
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
}

function useTodos() {
  const { state: todos, dispatch } = useReducer(todoReducer, []);
  
  const actions = useMemo(() => ({
    addTodo: (text: string) => dispatch({ type: 'ADD', payload: text }),
    toggleTodo: (id: string) => dispatch({ type: 'TOGGLE', payload: id }),
    deleteTodo: (id: string) => dispatch({ type: 'DELETE', payload: id })
  }), [dispatch]);
  
  return { todos, actions };
}
```

### 📦 Context + Reducer Pattern
```typescript
// Context with reducer
interface AppState {
  user: User | null;
  theme: 'light' | 'dark';
  loading: boolean;
}

type AppAction =
  | { type: 'SET_USER'; payload: User | null }
  | { type: 'SET_THEME'; payload: 'light' | 'dark' }
  | { type: 'SET_LOADING'; payload: boolean };

function appReducer(state: AppState, action: AppAction): AppState {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    default:
      return state;
  }
}

const AppContext = createContext<{
  state: AppState;
  dispatch: React.Dispatch<AppAction>;
} | null>(null);

function AppProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, {
    user: null,
    theme: 'light',
    loading: false
  });
  
  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
}

function useAppContext() {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useAppContext must be used within AppProvider');
  }
  return context;
}

// Usage
function ThemeToggle() {
  const { state, dispatch } = useAppContext();
  
  const toggleTheme = () => {
    dispatch({
      type: 'SET_THEME',
      payload: state.theme === 'light' ? 'dark' : 'light'
    });
  };
  
  return (
    <button onClick={toggleTheme}>
      Current theme: {state.theme}
    </button>
  );
}
```

## 🎯 Use Cases

### 1. **Compound Components for UI Library**
```typescript
// Accordion component
const AccordionContext = createContext({
  activeItems: new Set<string>(),
  toggle: (id: string) => {}
});

function Accordion({ children, allowMultiple = false }: {
  children: React.ReactNode;
  allowMultiple?: boolean;
}) {
  const [activeItems, setActiveItems] = useState<Set<string>>(new Set());
  
  const toggle = (id: string) => {
    setActiveItems(prev => {
      const newSet = new Set(prev);
      
      if (newSet.has(id)) {
        newSet.delete(id);
      } else if (allowMultiple) {
        newSet.add(id);
      } else {
        return new Set([id]);
      }
      
      return newSet;
    });
  };
  
  return (
    <AccordionContext.Provider value={{ activeItems, toggle }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ id, children }: {
  id: string;
  children: React.ReactNode;
}) {
  const { activeItems, toggle } = useContext(AccordionContext);
  const isActive = activeItems.has(id);
  
  return (
    <div className={`accordion-item ${isActive ? 'active' : ''}`}>
      {children}
    </div>
  );
}

function AccordionHeader({ children }: { children: React.ReactNode }) {
  const { toggle } = useContext(AccordionContext);
  const id = useId();
  
  return (
    <button onClick={() => toggle(id)} className="accordion-header">
      {children}
    </button>
  );
}

function AccordionPanel({ children }: { children: React.ReactNode }) {
  return <div className="accordion-panel">{children}</div>;
}
```

### 2. **Render Props for Data Visualization**
```typescript
function Chart({ data, children }: {
  data: any[];
  children: (props: {
    data: any[];
    width: number;
    height: number;
    scales: any;
  }) => React.ReactNode;
}) {
  const [dimensions, setDimensions] = useState({ width: 400, height: 300 });
  const containerRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const updateDimensions = () => {
      if (containerRef.current) {
        const { width, height } = containerRef.current.getBoundingClientRect();
        setDimensions({ width, height });
      }
    };
    
    updateDimensions();
    window.addEventListener('resize', updateDimensions);
    
    return () => window.removeEventListener('resize', updateDimensions);
  }, []);
  
  const scales = useMemo(() => {
    // Calculate scales based on data and dimensions
    return {
      x: createScale(data, 'x', dimensions.width),
      y: createScale(data, 'y', dimensions.height)
    };
  }, [data, dimensions]);
  
  return (
    <div ref={containerRef} className="chart-container">
      {children({ data, ...dimensions, scales })}
    </div>
  );
}

// Usage
function LineChart({ data }: { data: any[] }) {
  return (
    <Chart data={data}>
      {({ data, width, height, scales }) => (
        <svg width={width} height={height}>
          {/* Render chart using scales */}
        </svg>
      )}
    </Chart>
  );
}
```

## ⚠️ Edge Cases

### 1. **Compound Components Context Issues**
```typescript
// ❌ Bad - context not provided
function BadUsage() {
  return (
    <div>
      <Tab id="profile">Profile</Tab> {/* Error: TabContext not provided */}
    </div>
  );
}

// ✅ Good - proper context structure
function GoodUsage() {
  return (
    <Tabs defaultTab="profile">
      <TabList>
        <Tab id="profile">Profile</Tab>
      </TabList>
      <TabPanels>
        <TabPanel id="profile">Content</TabPanel>
      </TabPanels>
    </Tabs>
  );
}
```

### 2. **Render Props Performance**
```typescript
// ❌ Bad - creating new functions in render
function BadRenderProps() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div onClick={() => console.log(x, y)}> {/* New function every render */}
          Position: {x}, {y}
        </div>
      )}
    />
  );
}

// ✅ Good - memoized render function
function GoodRenderProps() {
  const handleClick = useCallback((x: number, y: number) => {
    console.log(x, y);
  }, []);
  
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div onClick={() => handleClick(x, y)}>
          Position: {x}, {y}
        </div>
      )}
    />
  );
}
```

### 3. **HOC Prop Collision**
```typescript
// ❌ Bad - prop name collision
const BadComponent = withAuth(withLoading(UserProfile));
// Both HOCs might use 'loading' prop

// ✅ Good - unique prop names or rename props
const GoodComponent = withAuth(withLoading(UserProfile, {
  loadingProp: 'dataLoading',
  errorProp: 'dataError'
}));
```

## 🎯 สรุป

### ✨ สิ่งที่ควรรู้
- **Compound Components**: สร้าง flexible API ด้วย context
- **Render Props**: แชร์ logic ผ่าน function props
- **HOCs**: ขยาย functionality ของ components
- **Custom Hooks**: ใช้ซ้ำ logic ระหว่าง components
- **State Patterns**: จัดการ state อย่างมีระเบียบ

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Advanced** ใน [ADVANCED.md](ADVANCED.md)
2. ทดลอบสร้าง UI library ด้วย patterns
3. ฝึกใช้ patterns ใน project จริง

### 💡 แนวทางการฝึก
- สร้าง compound components สำหรับ UI library
- ฝึก render props สำหรับ data visualization
- ทดลอง HOCs สำหรับ cross-cutting concerns
- ฝึก state management patterns ที่ซับซ้อน

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
