# 🏗️ React Architecture - สถาปัตยกรรมและ Diagrams

## 📋 สารบัฐ
- [Component Architecture](#-component-architecture)
- [State Management Architecture](#-state-management-architecture)
- [Data Flow Patterns](#-data-flow-patterns)
- [Performance Architecture](#-performance-architecture)
- [Scalable Architecture](#-scalable-architecture)
- [Common Diagrams](#-common-diagrams)
- [Best Practices](#-best-practices)

## 🎯 React Architecture คืออะไร

React Architecture คือการออกแบบโครงสร้างของ React application ให้มีประสิทธิภาพ บำรุงรักษาง่าย และขยายได้

### ✨ ประโยชน์ของ Good Architecture
- **Maintainability**: โค้ดง่ายต่อการดูแลรักษา
- **Scalability**: ขยาย application ได้ง่าย
- **Performance**: ทำงานได้เร็วและมีประสิทธิภาพ
- **Team Collaboration**: ทีมงานทำงานร่วมกันได้ง่าย
- **Testing**: ทดสอบได้ง่ายและครอบคลุม

## 🏢 สถาปัตยกรรมระบบ React โดยรวม

### แผนภาพสถาปัตยกรรม React
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Side   │    │   Server Side   │    │   Data Layer    │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │   React     │ │◄──►│ │  Node.js    │ │◄──►│ │ PostgreSQL  │ │
│ │  Frontend   │ │    │ │   Backend   │ │    │ │  Database   │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │   Redux     │ │    │ │    API      │ │    │ │    Redis    │ │
│ │   Store     │ │    │ │  Gateway    │ │    │ │   Cache     │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### แผนภาพ Data Flow ใน React
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    User     │    │   React     │    │   Redux     │    │    API      │
│  Interaction│───▶│  Component  │───▶│   Store     │───▶│   Service   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                       │                    │                    │
                       ▼                    ▼                    ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│     UI       │◄───│   State     │◄───│   Action    │◄───│   Response  │
│   Update     │    │   Update    │    │  Dispatch   │    │     Data     │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### แผนภาพ Component Hierarchy ใน React
```
App
├── Router
├── ThemeProvider
├── AuthProvider
├── Layout
│   ├── Header
│   │   ├── Logo
│   │   ├── Navigation
│   │   │   ├── NavItem
│   │   │   └── NavItem
│   │   └── UserMenu
│   │       ├── Avatar
│   │       └── Dropdown
│   ├── Sidebar
│   │   ├── MenuItems
│   │   └── UserInfo
│   ├── Main
│   │   ├── Dashboard
│   │   │   ├── StatsCards
│   │   │   ├── Charts
│   │   │   └── ActivityFeed
│   │   ├── Content
│   │   │   ├── ArticleList
│   │   │   │   └── ArticleCard
│   │   │   └── ArticleDetail
│   │   └── Settings
│   │       ├── ProfileForm
│   │       └── Preferences
│   └── Footer
└── Modals
    ├── LoginModal
    ├── ConfirmModal
    └── ErrorModal
```

### แผนภาพ State Management ใน React
```
┌─────────────────────────────────────────────────────────────┐
│                    Redux Store                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │    User     │  │    Posts    │  │   Settings  │        │
│  │   Slice     │  │   Slice     │  │   Slice     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    React Components                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │ useSelector  │  │ useDispatch │  │   connect   │        │
│  │   Hooks     │  │    Hook     │  │   HOC       │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### แผนภาพ API Architecture สำหรับ React
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   API       │    │  Business   │    │   Data      │
│             │    │  Gateway    │    │   Logic     │    │   Layer     │
│ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │
│ │ React   │ │◄──►│ │ Routes  │ │◄──►│ │Services │ │◄──►│ │ Models  │ │
│ │ App     │ │    │ │ Auth    │ │    │ │ Auth    │ │    │ │ User    │ │
│ └─────────┘ │    │ │ Users   │ │    │ │ Users   │ │    │ │ Post    │ │
│             │    │ │ Posts   │ │    │ │ Posts   │ │    │ └─────────┘ │
│ ┌─────────┐ │    │ │ Settings│ │    │ └─────────┘ │    │             │
│ │ Redux   │ │    │ └─────────┘ │    │             │    │ ┌─────────┐ │
│ │ Store   │ │    │             │    │ ┌─────────┐ │    │ │ Database│ │
│ └─────────┘ │    │ ┌─────────┐ │    │ │ Utils   │ │    │ │ PostgreSQL│ │
│             │    │ │ Middleware│ │    │ │ Helpers │ │    │ │ Redis   │ │
└─────────────┘    │ │ Auth    │ │    │ │ Validators│ │    │ │ Cache   │ │
                   │ │ Logger  │ │    │ └─────────┘ │    │ └─────────┘ │
                   │ │ Error   │ │    │             │    └─────────────┘
                   │ └─────────┘ │    └─────────────┘
                   └─────────────┘
```

## 🧩 Component Architecture

### 📦 พื้นฐาน Component Hierarchy
```
App
├── Header
│   ├── Logo
│   ├── Navigation
│   └── UserMenu
├── Main
│   ├── Dashboard
│   │   ├── StatsCard
│   │   ├── ChartContainer
│   │   └── ActivityFeed
│   ├── Content
│   │   ├── ArticleList
│   │   │   ├── ArticleCard
│   │   │   └── ArticleCard
│   │   └── Sidebar
│   │       ├── Categories
│   │       └── Tags
│   └── Footer
└── Modals
    ├── LoginModal
    └── SettingsModal
```

### 🎯 Component Types Architecture
```typescript
// 1. Presentational Components (UI Components)
interface ButtonProps {
  children: React.ReactNode;
  variant: 'primary' | 'secondary';
  onClick: () => void;
}

const Button: React.FC<ButtonProps> = ({ children, variant, onClick }) => (
  <button className={`btn btn-${variant}`} onClick={onClick}>
    {children}
  </button>
);

// 2. Container Components (Logic Components)
interface UserListContainerProps {
  userId: string;
}

const UserListContainer: React.FC<UserListContainerProps> = ({ userId }) => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    fetchUsers(userId).then(setUsers);
  }, [userId]);
  
  if (loading) return <LoadingSpinner />;
  
  return <UserList users={users} />;
};

// 3. Layout Components
interface PageLayoutProps {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  content: React.ReactNode;
  footer: React.ReactNode;
}

const PageLayout: React.FC<PageLayoutProps> = ({
  header,
  sidebar,
  content,
  footer
}) => (
  <div className="page-layout">
    <header className="header">{header}</header>
    <div className="main-content">
      <aside className="sidebar">{sidebar}</aside>
      <main className="content">{content}</main>
    </div>
    <footer className="footer">{footer}</footer>
  </div>
);
```

### 🎯 Smart vs Dumb Components
```typescript
// Dumb Component (Presentational)
interface UserCardProps {
  user: {
    id: string;
    name: string;
    email: string;
    avatar: string;
  };
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
}

const UserCard: React.FC<UserCardProps> = ({ user, onEdit, onDelete }) => (
  <div className="user-card">
    <img src={user.avatar} alt={user.name} />
    <h3>{user.name}</h3>
    <p>{user.email}</p>
    <div className="actions">
      <button onClick={() => onEdit?.(user.id)}>Edit</button>
      <button onClick={() => onDelete?.(user.id)}>Delete</button>
    </div>
  </div>
);

// Smart Component (Container)
const UserList: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    userService.getUsers()
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);
  
  const handleEdit = (id: string) => {
    // Edit logic
  };
  
  const handleDelete = (id: string) => {
    // Delete logic
  };
  
  return (
    <div className="user-list">
      {loading ? (
        <LoadingSpinner />
      ) : (
        users.map(user => (
          <UserCard
            key={user.id}
            user={user}
            onEdit={handleEdit}
            onDelete={handleDelete}
          />
        ))
      )}
    </div>
  );
};
```

## 🔄 State Management Architecture

### 📦 Local State Architecture
```typescript
// Component-level state
const Counter: React.FC = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
};

// Complex local state with useReducer
interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  loading: boolean;
}

type TodoAction =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: string }
  | { type: 'SET_FILTER'; payload: 'all' | 'active' | 'completed' }
  | { type: 'SET_LOADING'; payload: boolean };

const todoReducer = (state: TodoState, action: TodoAction): TodoState => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [...state.todos, {
          id: Date.now().toString(),
          text: action.payload,
          completed: false
        }]
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    default:
      return state;
  }
};

const TodoApp: React.FC = () => {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all',
    loading: false
  });
  
  return (
    <TodoContext.Provider value={{ state, dispatch }}>
      <TodoUI />
    </TodoContext.Provider>
  );
};
```

### 📦 Global State Architecture
```typescript
// Context API Architecture
interface AppContextType {
  user: User | null;
  theme: 'light' | 'dark';
  language: 'en' | 'th';
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  setTheme: (theme: 'light' | 'dark') => void;
  setLanguage: (language: 'en' | 'th') => void;
}

const AppContext = createContext<AppContextType | null>(null);

export const AppProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  const [language, setLanguage] = useState<'en' | 'th'>('th');
  
  const login = async (email: string, password: string) => {
    const userData = await authService.login(email, password);
    setUser(userData);
  };
  
  const logout = () => {
    setUser(null);
    authService.logout();
  };
  
  const value = {
    user,
    theme,
    language,
    login,
    logout,
    setTheme,
    setLanguage
  };
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
};

// Multiple Contexts Architecture
const AuthContext = createContext<AuthContextType | null>(null);
const ThemeContext = createContext<ThemeContextType | null>(null);
const LanguageContext = createContext<LanguageContextType | null>(null);

const App: React.FC = () => (
  <LanguageProvider>
    <ThemeProvider>
      <AuthProvider>
        <Router>
          <Routes>
            <Route path="/" element={<HomePage />} />
            <Route path="/dashboard" element={<DashboardPage />} />
          </Routes>
        </Router>
      </AuthProvider>
    </ThemeProvider>
  </LanguageProvider>
);
```

### 📦 State Management Library Architecture
```typescript
// Redux Toolkit Architecture
import { configureStore, createSlice, PayloadAction } from '@reduxjs/toolkit';

// User Slice
const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null as User | null,
    loading: false,
    error: null as string | null
  },
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.data = action.payload;
      state.loading = false;
      state.error = null;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
      state.loading = false;
    }
  }
});

// Store Configuration
const store = configureStore({
  reducer: {
    user: userSlice.reducer,
    posts: postsSlice.reducer,
    comments: commentsSlice.reducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: false
    })
});

// React-Redux Integration
const App: React.FC = () => (
  <Provider store={store}>
    <Router>
      <Routes>
        <Route path="/" element={<HomePage />} />
      </Routes>
    </Router>
  </Provider>
);

// Zustand Architecture
import { create } from 'zustand';

interface UserStore {
  user: User | null;
  loading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const useUserStore = create<UserStore>((set, get) => ({
  user: null,
  loading: false,
  login: async (email: string, password: string) => {
    set({ loading: true });
    try {
      const user = await authService.login(email, password);
      set({ user, loading: false });
    } catch (error) {
      set({ loading: false });
      throw error;
    }
  },
  logout: () => {
    set({ user: null });
    authService.logout();
  }
}));
```

## 🌊 Data Flow Patterns

### 📦 Unidirectional Data Flow
```
User Action → Dispatch Action → Reducer → State Update → Component Re-render → UI Update
```

```typescript
// Example: Todo Application Flow
// 1. User Action
const handleAddTodo = (text: string) => {
  // 2. Dispatch Action
  dispatch({ type: 'ADD_TODO', payload: text });
};

// 3. Reducer
const todoReducer = (state: TodoState, action: TodoAction) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [...state.todos, {
          id: Date.now().toString(),
          text: action.payload,
          completed: false
        }]
      };
    default:
      return state;
  }
};

// 4. State Update & Component Re-render
const TodoList: React.FC = () => {
  const { state } = useTodoContext();
  
  return (
    <ul>
      {state.todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
};
```

### 📦 Parent-Child Data Flow
```typescript
// Parent Component
const ParentComponent: React.FC = () => {
  const [data, setData] = useState<string[]>([]);
  
  const handleAddItem = (item: string) => {
    setData(prev => [...prev, item]);
  };
  
  return (
    <div>
      <ChildComponent data={data} onAddItem={handleAddItem} />
    </div>
  );
};

// Child Component
interface ChildComponentProps {
  data: string[];
  onAddItem: (item: string) => void;
}

const ChildComponent: React.FC<ChildComponentProps> = ({ data, onAddItem }) => {
  const [inputValue, setInputValue] = useState('');
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (inputValue.trim()) {
      onAddItem(inputValue);
      setInputValue('');
    }
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Add item"
        />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {data.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
};
```

### 📦 Sibling Communication
```typescript
// Using Context for Sibling Communication
const SiblingContext = createContext<{
  sharedData: string;
  updateSharedData: (data: string) => void;
}>({
  sharedData: '',
  updateSharedData: () => {}
});

const ParentComponent: React.FC = () => {
  const [sharedData, setSharedData] = useState('');
  
  const updateSharedData = (data: string) => {
    setSharedData(data);
  };
  
  return (
    <SiblingContext.Provider value={{ sharedData, updateSharedData }}>
      <Sibling1 />
      <Sibling2 />
    </SiblingContext.Provider>
  );
};

const Sibling1: React.FC = () => {
  const { sharedData, updateSharedData } = useContext(SiblingContext);
  
  return (
    <div>
      <p>Shared Data: {sharedData}</p>
      <button onClick={() => updateSharedData('From Sibling 1')}>
        Update from Sibling 1
      </button>
    </div>
  );
};

const Sibling2: React.FC = () => {
  const { sharedData, updateSharedData } = useContext(SiblingContext);
  
  return (
    <div>
      <p>Shared Data: {sharedData}</p>
      <button onClick={() => updateSharedData('From Sibling 2')}>
        Update from Sibling 2
      </button>
    </div>
  );
};
```

## ⚡ Performance Architecture

### 📦 Component Optimization Architecture
```typescript
// React.memo Architecture
const ExpensiveComponent = React.memo(({ data, onUpdate }: {
  data: Item[];
  onUpdate: (id: string) => void;
}) => {
  return (
    <div>
      {data.map(item => (
        <Item key={item.id} item={item} onUpdate={onUpdate} />
      ))}
    </div>
  );
}, (prevProps, nextProps) => {
  // Custom comparison
  return prevProps.data.length === nextProps.data.length &&
         prevProps.data.every((item, index) => 
           item.id === nextProps.data[index].id
         );
});

// useMemo Architecture
const DataProcessor: React.FC<{ items: Item[] }> = ({ items }) => {
  const processedData = useMemo(() => {
    console.log('Processing data...');
    return items
      .filter(item => item.active)
      .map(item => ({
        ...item,
        processed: true,
        value: item.value * 2
      }))
      .sort((a, b) => a.value - b.value);
  }, [items]);
  
  return <DataList data={processedData} />;
};

// useCallback Architecture
const EventHandler: React.FC<{ onSave: (data: any) => void }> = ({ onSave }) => {
  const [formData, setFormData] = useState({});
  
  const handleSubmit = useCallback((e: React.FormEvent) => {
    e.preventDefault();
    onSave(formData);
  }, [formData, onSave]);
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
};
```

### 📦 Virtualization Architecture
```typescript
// Virtual List Architecture
interface VirtualListProps<T> {
  items: T[];
  itemHeight: number;
  containerHeight: number;
  renderItem: (item: T, index: number) => React.ReactNode;
}

const VirtualList = <T,>({ items, itemHeight, containerHeight, renderItem }: VirtualListProps<T>) => {
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
            {renderItem(item, startIndex + index)}
          </div>
        ))}
      </div>
    </div>
  );
};
```

### 📦 Code Splitting Architecture
```typescript
// Route-based Code Splitting
const HomePage = lazy(() => import('./pages/HomePage'));
const DashboardPage = lazy(() => import('./pages/DashboardPage'));
const SettingsPage = lazy(() => import('./pages/SettingsPage'));

const App: React.FC = () => (
  <Router>
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/dashboard" element={<DashboardPage />} />
        <Route path="/settings" element={<SettingsPage />} />
      </Routes>
    </Suspense>
  </Router>
);

// Component-based Code Splitting
const HeavyChart = lazy(() => import('./components/HeavyChart'));

const Dashboard: React.FC = () => {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>
      
      {showChart && (
        <Suspense fallback={<ChartLoader />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
};
```

## 🏗️ Scalable Architecture

### 📦 Feature-based Architecture
```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── ProtectedRoute.tsx
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   └── usePermissions.ts
│   │   ├── services/
│   │   │   └── authService.ts
│   │   ├── types/
│   │   │   └── auth.ts
│   │   └── index.ts
│   ├── users/
│   │   ├── components/
│   │   │   ├── UserList.tsx
│   │   │   ├── UserCard.tsx
│   │   │   └── UserForm.tsx
│   │   ├── hooks/
│   │   │   ├── useUsers.ts
│   │   │   └── useUser.ts
│   │   ├── services/
│   │   │   └── userService.ts
│   │   ├── types/
│   │   │   └── user.ts
│   │   └── index.ts
│   └── posts/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       ├── types/
│       └── index.ts
├── shared/
│   ├── components/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Modal.tsx
│   ├── hooks/
│   │   ├── useApi.ts
│   │   └── useLocalStorage.ts
│   ├── utils/
│   │   ├── api.ts
│   │   ├── validation.ts
│   │   └── helpers.ts
│   └── types/
│       └── common.ts
├── pages/
│   ├── HomePage.tsx
│   ├── DashboardPage.tsx
│   └── SettingsPage.tsx
├── App.tsx
└── index.tsx
```

### 📦 Micro-frontend Architecture
```typescript
// Shell Application
const ShellApp: React.FC = () => {
  return (
    <div>
      <Header />
      <main>
        <Routes>
          <Route path="/auth/*" element={<AuthModule />} />
          <Route path="/users/*" element={<UsersModule />} />
          <Route path="/posts/*" element={<PostsModule />} />
        </Routes>
      </main>
      <Footer />
    </div>
  );
};

// Module Federation Configuration
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        auth: 'auth@http://localhost:3001/remoteEntry.js',
        users: 'users@http://localhost:3002/remoteEntry.js',
        posts: 'posts@http://localhost:3003/remoteEntry.js'
      }
    })
  ]
};
```

### 📦 Clean Architecture
```typescript
// Domain Layer
interface User {
  id: string;
  name: string;
  email: string;
}

interface UserRepository {
  findById(id: string): Promise<User>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}

// Application Layer
interface UserService {
  getUser(id: string): Promise<User>;
  createUser(userData: Omit<User, 'id'>): Promise<User>;
  updateUser(id: string, userData: Partial<User>): Promise<User>;
  deleteUser(id: string): Promise<void>;
}

class UserServiceImpl implements UserService {
  constructor(private userRepository: UserRepository) {}
  
  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error('User not found');
    }
    return user;
  }
  
  async createUser(userData: Omit<User, 'id'>): Promise<User> {
    // Business logic validation
    if (!userData.email.includes('@')) {
      throw new Error('Invalid email');
    }
    
    return this.userRepository.save({
      ...userData,
      id: generateId()
    });
  }
}

// Infrastructure Layer
class HttpUserRepository implements UserRepository {
  async findById(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }
  
  async save(user: User): Promise<User> {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(user)
    });
    return response.json();
  }
  
  async delete(id: string): Promise<void> {
    await fetch(`/api/users/${id}`, { method: 'DELETE' });
  }
}

// Presentation Layer
const UserComponent: React.FC = () => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  
  const userService = useMemo(() => 
    new UserServiceImpl(new HttpUserRepository()), []
  );
  
  useEffect(() => {
    const loadUser = async () => {
      setLoading(true);
      try {
        const userData = await userService.getUser('123');
        setUser(userData);
      } catch (error) {
        console.error('Failed to load user:', error);
      } finally {
        setLoading(false);
      }
    };
    
    loadUser();
  }, [userService]);
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

## 📊 Common Diagrams

### 🎯 Component Lifecycle Diagram
```
Mounting Phase:
constructor() → getDerivedStateFromProps() → render() → componentDidMount()

Updating Phase:
getDerivedStateFromProps() → shouldComponentUpdate() → render() → 
getSnapshotBeforeUpdate() → componentDidUpdate()

Unmounting Phase:
componentWillUnmount()
```

### 🎯 Data Flow Diagram
```
User Interaction
       ↓
Event Handler
       ↓
Action Dispatch
       ↓
Reducer/State Update
       ↓
Component Re-render
       ↓
UI Update
```

### 🎯 Component Tree Diagram
```
App
├── Router
├── AuthProvider
├── ThemeProvider
├── Layout
│   ├── Header
│   │   ├── Logo
│   │   ├── Navigation
│   │   └── UserMenu
│   ├── Sidebar
│   │   ├── MenuItems
│   │   └── UserInfo
│   ├── Main
│   │   ├── Dashboard
│   │   │   ├── StatsCards
│   │   │   ├── Charts
│   │   │   └── ActivityFeed
│   │   └── Content
│   │       ├── ArticleList
│   │       └── ArticleDetail
│   └── Footer
└── Modals
    ├── LoginModal
    └── SettingsModal
```

### 🎯 State Management Flow Diagram
```
Context Provider
       ↓
Consumer Hook
       ↓
State Access
       ↓
State Update
       ↓
Re-render Consumers
```

## ✅ Best Practices

### 🎯 Component Architecture Best Practices
```typescript
// 1. Single Responsibility Principle
const UserProfile: React.FC<{ user: User }> = ({ user }) => (
  <div>
    <Avatar src={user.avatar} />
    <UserInfo name={user.name} email={user.email} />
    <UserStats posts={user.posts.length} followers={user.followers.length} />
  </div>
);

// 2. Composition over Inheritance
const Card: React.FC<{ children: React.ReactNode; className?: string }> = ({
  children,
  className
}) => (
  <div className={`card ${className || ''}`}>
    {children}
  </div>
);

const CardHeader: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <div className="card-header">{children}</div>
);

const CardBody: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <div className="card-body">{children}</div>
);

// Usage
<Card>
  <CardHeader>
    <h2>User Profile</h2>
  </CardHeader>
  <CardBody>
    <UserProfile user={user} />
  </CardBody>
</Card>

// 3. Props Interface Design
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false,
  onClick,
  type = 'button'
}) => {
  const baseClasses = 'btn';
  const variantClasses = `btn-${variant}`;
  const sizeClasses = `btn-${size}`;
  const disabledClasses = disabled ? 'btn-disabled' : '';
  const loadingClasses = loading ? 'btn-loading' : '';
  
  const className = `${baseClasses} ${variantClasses} ${sizeClasses} ${disabledClasses} ${loadingClasses}`;
  
  return (
    <button
      type={type}
      className={className}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {loading ? <Spinner /> : children}
    </button>
  );
};
```

### 🎯 State Management Best Practices
```typescript
// 1. Keep State Close to Where It's Used
const TodoItem: React.FC<{ todo: Todo; onUpdate: (todo: Todo) => void }> = ({
  todo,
  onUpdate
}) => {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);
  
  const handleSave = () => {
    onUpdate({ ...todo, text: editText });
    setIsEditing(false);
  };
  
  return (
    <div>
      {isEditing ? (
        <input
          value={editText}
          onChange={(e) => setEditText(e.target.value)}
          onBlur={handleSave}
          onKeyPress={(e) => e.key === 'Enter' && handleSave()}
        />
      ) : (
        <span onClick={() => setIsEditing(true)}>{todo.text}</span>
      )}
    </div>
  );
};

// 2. Use Derived State Instead of Redundant State
const TodoList: React.FC<{ todos: Todo[] }> = ({ todos }) => {
  const completedCount = todos.filter(todo => todo.completed).length;
  const activeCount = todos.length - completedCount;
  
  return (
    <div>
      <p>Active: {activeCount}</p>
      <p>Completed: {completedCount}</p>
      <ul>
        {todos.map(todo => (
          <TodoItem key={todo.id} todo={todo} onUpdate={() => {}} />
        ))}
      </ul>
    </div>
  );
};

// 3. Normalize State Shape
interface NormalizedState {
  entities: { [key: string]: Entity };
  ids: string[];
  loading: boolean;
  error: string | null;
}

const useNormalizedState = <T extends { id: string }>() => {
  const [state, setState] = useState<NormalizedState>({
    entities: {},
    ids: [],
    loading: false,
    error: null
  });
  
  const addEntity = (entity: T) => {
    setState(prev => ({
      ...prev,
      entities: { ...prev.entities, [entity.id]: entity },
      ids: [...prev.ids, entity.id]
    }));
  };
  
  const updateEntity = (id: string, updates: Partial<T>) => {
    setState(prev => ({
      ...prev,
      entities: {
        ...prev.entities,
        [id]: { ...prev.entities[id], ...updates }
      }
    }));
  };
  
  const removeEntity = (id: string) => {
    setState(prev => {
      const { [id]: removed, ...entities } = prev.entities;
      return {
        ...prev,
        entities,
        ids: prev.ids.filter(entityId => entityId !== id)
      };
    });
  };
  
  return {
    ...state,
    addEntity,
    updateEntity,
    removeEntity
  };
};
```

### 🎯 Performance Best Practices
```typescript
// 1. Memoization Strategy
const ExpensiveComponent: React.FC<{ data: Item[]; onSelect: (item: Item) => void }> = ({
  data,
  onSelect
}) => {
  // Memoize expensive calculations
  const processedData = useMemo(() => {
    return data
      .filter(item => item.active)
      .map(item => ({ ...item, value: item.value * 2 }))
      .sort((a, b) => a.value - b.value);
  }, [data]);
  
  // Memoize event handlers
  const handleSelect = useCallback((item: Item) => {
    onSelect(item);
  }, [onSelect]);
  
  return (
    <div>
      {processedData.map(item => (
        <ItemCard
          key={item.id}
          item={item}
          onSelect={handleSelect}
        />
      ))}
    </div>
  );
};

// 2. Virtualization for Large Lists
const VirtualizedList: React.FC<{ items: Item[] }> = ({ items }) => {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={80}
      itemData={items}
    >
      {({ index, style, data }) => (
        <div style={style}>
          <ItemCard item={data[index]} />
        </div>
      )}
    </FixedSizeList>
  );
};

// 3. Code Splitting Strategy
const LazyComponent = lazy(() => 
  import('./HeavyComponent').then(module => ({
    default: module.HeavyComponent
  }))
);

const App: React.FC = () => {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Router>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/heavy" element={<LazyComponent />} />
        </Routes>
      </Router>
    </Suspense>
  );
};
```

## 🎯 สรุป

### ✨ สิ่งที่ควบคุม
- **Component Architecture**: การออกแบบโครงสร้าง components
- **State Management**: การจัดการ state อย่างมีประสิทธิภาพ
- **Data Flow**: การไหลของข้อมูลใน application
- **Performance**: การปรับปรุงประสิทธิภาพ
- **Scalability**: การออกแบบที่ขยายได้

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Design Patterns** ใน [PATTERNS.md](PATTERNS.md)
2. ทดลอบสร้าง scalable architecture
3. ฝึก performance optimization techniques

### 💡 แนวทางการฝึก
- วาด component diagrams ก่อนเขียนโค้ด
- ฝึกออกแบบ state management architecture
- ทดลอบ different architectural patterns
- วัดผล performance ของแต่ละ pattern

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
