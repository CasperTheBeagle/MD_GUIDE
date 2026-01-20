# 🔄 React State & Lifecycle - สถานะและวงจรชีวิต

## 📋 สารบัฐ
- [State คืออะไร](#-state-คืออะไร)
- [setState และการอัปเดต](#-setstate-และการอัปเดต)
- [Lifecycle Methods](#-lifecycle-methods)
- [State Lifting](#-state-lifting)
- [Controlled vs Uncontrolled Components](#-controlled-vs-uncontrolled-components)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🎯 State คืออะไร

State เป็นข้อมูลภายใน component ที่สามารถเปลี่ยนแปลงได้และทำให้ component re-render

### 📦 พื้นฐาน State
```typescript
// Functional Component with useState
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

// Class Component with State
class Counter extends React.Component {
  state = {
    count: 0
  };

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>
          Increment
        </button>
      </div>
    );
  }
}
```

### 🔄 Multiple State
```typescript
function UserProfile() {
  const [name, setName] = useState('John');
  const [age, setAge] = useState(30);
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Status: {isLoggedIn ? 'Logged in' : 'Logged out'}</p>
    </div>
  );
}

// Class Component with Multiple State
class UserProfile extends React.Component {
  state = {
    name: 'John',
    age: 30,
    isLoggedIn: false
  };

  updateName = (newName: string) => {
    this.setState({ name: newName });
  };

  render() {
    const { name, age, isLoggedIn } = this.state;
    return (
      <div>
        <h2>{name}</h2>
        <p>Age: {age}</p>
        <p>Status: {isLoggedIn ? 'Logged in' : 'Logged out'}</p>
      </div>
    );
  }
}
```

### 📊 Complex State
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

interface AppState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

function App() {
  const [state, setState] = useState<AppState>({
    user: null,
    loading: false,
    error: null
  });

  const fetchUser = async (id: string) => {
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const response = await fetch(`/api/users/${id}`);
      const user = await response.json();
      setState({ user, loading: false, error: null });
    } catch (error) {
      setState({ 
        user: null, 
        loading: false, 
        error: error.message 
      });
    }
  };

  return (
    <div>
      {state.loading && <p>Loading...</p>}
      {state.error && <p>Error: {state.error}</p>}
      {state.user && (
        <div>
          <h2>{state.user.name}</h2>
          <p>{state.user.email}</p>
        </div>
      )}
    </div>
  );
}
```

## 🔄 setState และการอัปเดต

### 1. **Functional setState**
```typescript
function Counter() {
  const [count, setCount] = useState(0);

  // Direct update
  const increment = () => {
    setCount(count + 1);
  };

  // Functional update (recommended for multiple updates)
  const incrementTwice = () => {
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
  };

  // Object update
  const [user, setUser] = useState({ name: 'John', age: 30 });
  
  const updateName = (newName: string) => {
    setUser(prevUser => ({ ...prevUser, name: newName }));
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={incrementTwice}>Increment Twice</button>
    </div>
  );
}
```

### 2. **Class Component setState**
```typescript
class Counter extends React.Component {
  state = {
    count: 0,
    user: { name: 'John', age: 30 }
  };

  // Direct update
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  // Functional update (recommended)
  incrementTwice = () => {
    this.setState(prevState => ({ count: prevState.count + 1 }));
    this.setState(prevState => ({ count: prevState.count + 1 }));
  };

  // Object update
  updateName = (newName: string) => {
    this.setState(prevState => ({
      user: { ...prevState.user, name: newName }
    }));
  };

  // Multiple updates
  updateMultiple = () => {
    this.setState(prevState => ({
      count: prevState.count + 1,
      user: { ...prevState.user, age: prevState.user.age + 1 }
    }));
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <p>User: {this.state.user.name}</p>
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.incrementTwice}>Increment Twice</button>
      </div>
    );
  }
}
```

### 3. **Batching Updates**
```typescript
function Counter() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');

  const updateBoth = () => {
    // React 18+ จะ batch updates อัตโนมัติ
    setCount(count + 1);
    setName('Jane');
    
    // หรือใช้ unstable_batchedUpdates (React 17-)
    import { unstable_batchedUpdates } from 'react-dom';
    unstable_batchedUpdates(() => {
      setCount(count + 1);
      setName('Jane');
    });
  };

  return (
    <div>
      <p>Count: {count}, Name: {name}</p>
      <button onClick={updateBoth}>Update Both</button>
    </div>
  );
}
```

## 🔄 Lifecycle Methods

### 1. **Class Component Lifecycle**
```typescript
class UserProfile extends React.Component {
  state = {
    user: null,
    loading: false,
    error: null
  };

  // Mounting
  componentDidMount() {
    console.log('Component mounted');
    this.fetchUser();
  }

  // Updating
  componentDidUpdate(prevProps: any) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }

  // Unmounting
  componentWillUnmount() {
    console.log('Component will unmount');
    // Cleanup
  }

  // Error handling
  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Error caught:', error, errorInfo);
  }

  fetchUser = async () => {
    this.setState({ loading: true });
    
    try {
      const response = await fetch(`/api/users/${this.props.userId}`);
      const user = await response.json();
      this.setState({ user, loading: false });
    } catch (error) {
      this.setState({ error: error.message, loading: false });
    }
  };

  render() {
    const { user, loading, error } = this.state;
    
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    if (!user) return <div>No user found</div>;

    return (
      <div>
        <h2>{user.name}</h2>
        <p>{user.email}</p>
      </div>
    );
  }
}
```

### 2. **Functional Component Lifecycle (Hooks)**
```typescript
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // componentDidMount + componentDidUpdate
  useEffect(() => {
    fetchUser();
  }, [userId]); // Dependency array

  // componentWillUnmount
  useEffect(() => {
    return () => {
      console.log('Cleanup');
      // Cleanup code here
    };
  }, []);

  // Only on mount (empty dependency array)
  useEffect(() => {
    console.log('Component mounted');
  }, []);

  // Error handling with Error Boundary
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

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### 3. **Advanced Lifecycle Patterns**
```typescript
// Custom hook for lifecycle
function useLifecycle({
  onMount,
  onUnmount,
  onUpdate
}: {
  onMount?: () => void;
  onUnmount?: () => void;
  onUpdate?: (prevProps: any) => void;
}) {
  const prevProps = useRef();

  useEffect(() => {
    if (onMount) onMount();
    return () => {
      if (onUnmount) onUnmount();
    };
  }, []);

  useEffect(() => {
    if (onUpdate && prevProps.current) {
      onUpdate(prevProps.current);
    }
    prevProps.current = { ...prevProps.current };
  });
}

// Usage
function MyComponent(props: any) {
  useLifecycle({
    onMount: () => console.log('Mounted'),
    onUnmount: () => console.log('Unmounted'),
    onUpdate: (prevProps) => console.log('Updated', prevProps, props)
  });

  return <div>My Component</div>;
}
```

## 🔄 State Lifting

State lifting คือการย้าย state ขึ้นไปไว้ที่ parent component เพื่อแชร์ระหว่าง child components

### 1. **Basic State Lifting**
```typescript
// Parent Component
function TemperatureCalculator() {
  const [celsius, setCelsius] = useState('');
  const [fahrenheit, setFahrenheit] = useState('');

  const handleCelsiusChange = (value: string) => {
    setCelsius(value);
    const f = value ? (parseFloat(value) * 9/5) + 32 : '';
    setFahrenheit(f.toString());
  };

  const handleFahrenheitChange = (value: string) => {
    setFahrenheit(value);
    const c = value ? (parseFloat(value) - 32) * 5/9 : '';
    setCelsius(c.toString());
  };

  return (
    <div>
      <TemperatureInput
        scale="c"
        value={celsius}
        onChange={handleCelsiusChange}
      />
      <TemperatureInput
        scale="f"
        value={fahrenheit}
        onChange={handleFahrenheitChange}
      />
    </div>
  );
}

// Child Component
interface TemperatureInputProps {
  scale: 'c' | 'f';
  value: string;
  onChange: (value: string) => void;
}

function TemperatureInput({ scale, value, onChange }: TemperatureInputProps) {
  const scaleName = scale === 'c' ? 'Celsius' : 'Fahrenheit';
  
  return (
    <fieldset>
      <legend>Enter temperature in {scaleName}:</legend>
      <input
        value={value}
        onChange={(e) => onChange(e.target.value)}
      />
    </fieldset>
  );
}
```

### 2. **Complex State Lifting**
```typescript
// Parent Component
function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all');

  const addTodo = (text: string) => {
    const newTodo: Todo = {
      id: Date.now().toString(),
      text,
      completed: false
    };
    setTodos([...todos, newTodo]);
  };

  const toggleTodo = (id: string) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const deleteTodo = (id: string) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <div>
      <TodoForm onAdd={addTodo} />
      <TodoFilter filter={filter} onChange={setFilter} />
      <TodoList
        todos={filteredTodos}
        onToggle={toggleTodo}
        onDelete={deleteTodo}
      />
    </div>
  );
}

// Child Components
function TodoForm({ onAdd }: { onAdd: (text: string) => void }) {
  const [text, setText] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (text.trim()) {
      onAdd(text);
      setText('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Add a todo"
      />
      <button type="submit">Add</button>
    </form>
  );
}
```

## 🎛️ Controlled vs Uncontrolled Components

### 1. **Controlled Components**
```typescript
function ControlledForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    age: ''
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Form data:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        name="age"
        value={formData.age}
        onChange={handleChange}
        placeholder="Age"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 2. **Uncontrolled Components**
```typescript
function UncontrolledForm() {
  const nameRef = useRef<HTMLInputElement>(null);
  const emailRef = useRef<HTMLInputElement>(null);
  const ageRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    const formData = {
      name: nameRef.current?.value || '',
      email: emailRef.current?.value || '',
      age: ageRef.current?.value || ''
    };
    
    console.log('Form data:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={nameRef}
        placeholder="Name"
        defaultValue=""
      />
      <input
        ref={emailRef}
        placeholder="Email"
        defaultValue=""
      />
      <input
        ref={ageRef}
        placeholder="Age"
        defaultValue=""
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 3. **Mixed Approach**
```typescript
function MixedForm() {
  const [name, setName] = useState('');
  const emailRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    const formData = {
      name,
      email: emailRef.current?.value || ''
    };
    
    console.log('Form data:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Controlled */}
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      
      {/* Uncontrolled */}
      <input
        ref={emailRef}
        placeholder="Email"
        defaultValue=""
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

## 🎯 Use Cases

### 1. **Form State Management**
```typescript
function ComplexForm() {
  const [formData, setFormData] = useState({
    personal: {
      firstName: '',
      lastName: '',
      email: ''
    },
    address: {
      street: '',
      city: '',
      zipCode: ''
    },
    preferences: {
      newsletter: false,
      notifications: true
    }
  });

  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const updateNestedField = (category: string, field: string, value: any) => {
    setFormData(prev => ({
      ...prev,
      [category]: {
        ...prev[category as keyof typeof prev],
        [field]: value
      }
    }));
  };

  const validateForm = () => {
    const newErrors: Record<string, string> = {};
    
    if (!formData.personal.firstName) {
      newErrors.firstName = 'First name is required';
    }
    
    if (!formData.personal.email) {
      newErrors.email = 'Email is required';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!validateForm()) return;
    
    setIsSubmitting(true);
    
    try {
      await submitForm(formData);
      console.log('Form submitted successfully');
    } catch (error) {
      console.error('Submission error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.personal.firstName}
        onChange={(e) => updateNestedField('personal', 'firstName', e.target.value)}
        placeholder="First Name"
      />
      {errors.firstName && <span className="error">{errors.firstName}</span>}
      
      <input
        value={formData.personal.email}
        onChange={(e) => updateNestedField('personal', 'email', e.target.value)}
        placeholder="Email"
      />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

### 2. **Real-time Data Sync**
```typescript
function RealtimeCounter() {
  const [count, setCount] = useState(0);
  const [isConnected, setIsConnected] = useState(false);
  const wsRef = useRef<WebSocket | null>(null);

  useEffect(() => {
    // Connect to WebSocket
    wsRef.current = new WebSocket('ws://localhost:8080');
    
    wsRef.current.onopen = () => {
      setIsConnected(true);
      console.log('Connected to server');
    };
    
    wsRef.current.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.type === 'count_update') {
        setCount(data.count);
      }
    };
    
    wsRef.current.onclose = () => {
      setIsConnected(false);
      console.log('Disconnected from server');
    };
    
    return () => {
      wsRef.current?.close();
    };
  }, []);

  const increment = () => {
    if (wsRef.current && isConnected) {
      wsRef.current.send(JSON.stringify({ type: 'increment' }));
    }
  };

  return (
    <div>
      <p>Count: {count}</p>
      <p>Status: {isConnected ? 'Connected' : 'Disconnected'}</p>
      <button onClick={increment} disabled={!isConnected}>
        Increment
      </button>
    </div>
  );
}
```

### 3. **Local Storage Sync**
```typescript
function PersistentForm() {
  const [formData, setFormData] = useState(() => {
    // Load from localStorage on mount
    const saved = localStorage.getItem('formData');
    return saved ? JSON.parse(saved) : { name: '', email: '' };
  });

  // Save to localStorage when formData changes
  useEffect(() => {
    localStorage.setItem('formData', JSON.stringify(formData));
  }, [formData]);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const clearForm = () => {
    setFormData({ name: '', email: '' });
    localStorage.removeItem('formData');
  };

  return (
    <div>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <button onClick={clearForm}>Clear</button>
    </div>
  );
}
```

## ⚠️ Edge Cases

### 1. **State Initialization**
```typescript
// ❌ ไม่ดี - State initialization ที่ซับซ้อนใน render
function ExpensiveComponent() {
  const [data, setData] = useState(
    // นี่จะทำงานทุกครั้งที่ render
    expensiveCalculation(props.initialValue)
  );
  
  return <div>{data}</div>;
}

// ✅ ดี - ใช้ lazy initialization
function ExpensiveComponent() {
  const [data, setData] = useState(() => 
    // นี่จะทำงานแค่ครั้งแรก
    expensiveCalculation(props.initialValue)
  );
  
  return <div>{data}</div>;
}
```

### 2. **State Updates in Loops**
```typescript
// ❌ ไม่ดี - อาจทำให้ infinite loop
function BadLoop() {
  const [count, setCount] = useState(0);
  
  // นี่จะทำงานทุกครั้งที่ render
  while (count < 10) {
    setCount(count + 1);
  }
  
  return <div>{count}</div>;
}

// ✅ ดี - ใช้ useEffect
function GoodLoop() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    if (count < 10) {
      setCount(count + 1);
    }
  }, [count]);
  
  return <div>{count}</div>;
}
```

### 3. **Stale Closure**
```typescript
// ❌ ไม่ดี - Stale closure
function StaleClosure() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      // count จะเป็นค่าเริ่มต้นตลอดเวลา
      console.log(count);
    }, 1000);
    
    return () => clearInterval(timer);
  }, []); // Empty dependency array
  
  return <div>{count}</div>;
}

// ✅ ดี - ใช้ functional update
function NoStaleClosure() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      // ใช้ functional update เพื่อเข้าถึงค่าล่าสุด
      setCount(prevCount => {
        console.log(prevCount);
        return prevCount + 1;
      });
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  return <div>{count}</div>;
}
```

### 4. **Memory Leaks**
```typescript
// ❌ ไม่ดี - Memory leak
function MemoryLeak() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    let isMounted = true;
    
    fetchData().then(result => {
      // อาจ set state หลังจาก component unmount
      if (isMounted) {
        setData(result);
      }
    });
    
    // ไม่มี cleanup
  }, []);
  
  return <div>{data}</div>;
}

// ✅ ดี - มี cleanup
function NoMemoryLeak() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    let isMounted = true;
    
    fetchData().then(result => {
      if (isMounted) {
        setData(result);
      }
    });
    
    return () => {
      isMounted = false;
    };
  }, []);
  
  return <div>{data}</div>;
}
```

### 5. **Async State Updates**
```typescript
// ❌ ไม่ดี - Async state อาจเก่า
function AsyncState() {
  const [data, setData] = useState([]);
  
  const handleClick = async () => {
    const result = await fetchData();
    // data อาจเปลี่ยนไปตอนที่รอ API
    setData([...data, result]);
  };
  
  return <button onClick={handleClick}>Add Data</button>;
}

// ✅ ดี - ใช้ functional update
function AsyncStateCorrect() {
  const [data, setData] = useState([]);
  
  const handleClick = async () => {
    const result = await fetchData();
    // ใช้ functional update เพื่อให้แน่ใจว่าได้ค่าล่าสุด
    setData(prevData => [...prevData, result]);
  };
  
  return <button onClick={handleClick}>Add Data</button>;
}
```

## 🎯 สรุป

### ✅ สิ่งที่ควรรู้
- State เป็นข้อมูลภายใน component ที่เปลี่ยนแปลงได้
- setState ควรใช้ functional update สำหรับความปลอดภัย
- Lifecycle methods ควบคุมการทำงานในแต่ละขั้นตอน
- State lifting ใช้แชร์ข้อมูลระหว่าง components
- Controlled components ควบคุม form ด้วย state

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Hooks** ใน [HOOKS.md](HOOKS.md)
2. เรียนรู้ **Context API** ใน [CONTEXT.md](CONTEXT.md)
3. ทดลองสร้าง forms และ state management

### 💡 แนวทางการฝึก
- สร้าง forms ที่ซับซ้อน
- ฝึก state lifting ระหว่าง components
- ทดลอง controlled/uncontrolled components
- ฝึกจัดการ async state updates

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
