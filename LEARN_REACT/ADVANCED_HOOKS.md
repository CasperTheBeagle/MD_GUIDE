# 🎣 Advanced Hooks - useReducer, useCallback, useMemo, useRef

## 📋 สารบัฐ
- [useReducer](#-usereducer)
- [useCallback](#-usecallback)
- [useMemo](#-usememo)
- [useRef](#-useref)
- [useLayoutEffect](#-uselayouteffect)
- [useImperativeHandle](#-useimperativehandle)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🔄 useReducer

### 📦 พื้นฐาน useReducer
```typescript
// Reducer function
function counterReducer(state: { count: number }, action: { type: string }) {
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
}

// Use reducer
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

### 🎯 useReducer กับ TypeScript
```typescript
// Define types
type CounterAction = 
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' }
  | { type: 'set'; payload: number };

interface CounterState {
  count: number;
  history: number[];
}

// Reducer with types
function counterReducer(state: CounterState, action: CounterAction): CounterState {
  switch (action.type) {
    case 'increment':
      return {
        count: state.count + 1,
        history: [...state.history, state.count]
      };
    case 'decrement':
      return {
        count: state.count - 1,
        history: [...state.history, state.count]
      };
    case 'reset':
      return { count: 0, history: [] };
    case 'set':
      return {
        count: action.payload,
        history: [...state.history, state.count]
      };
    default:
      return state;
  }
}

// Component
function AdvancedCounter() {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    history: []
  });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>Set to 10</button>
      
      <div>
        <h4>History:</h4>
        {state.history.map((value, index) => (
          <span key={index}>{value} </span>
        ))}
      </div>
    </div>
  );
}
```

### 🔄 Complex State with useReducer
```typescript
// Todo app with useReducer
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

type TodoAction =
  | { type: 'add'; payload: string }
  | { type: 'toggle'; payload: string }
  | { type: 'delete'; payload: string }
  | { type: 'clear' }
  | { type: 'filter'; payload: 'all' | 'active' | 'completed' };

interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
}

function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'add':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now().toString(),
            text: action.payload,
            completed: false
          }
        ]
      };
    case 'toggle':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case 'delete':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    case 'clear':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
    case 'filter':
      return {
        ...state,
        filter: action.payload
      };
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all'
  });
  
  const [inputValue, setInputValue] = useState('');
  
  const addTodo = () => {
    if (inputValue.trim()) {
      dispatch({ type: 'add', payload: inputValue });
      setInputValue('');
    }
  };
  
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });
  
  return (
    <div>
      <h1>Todo App</h1>
      
      <div>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add a todo"
        />
        <button onClick={addTodo}>Add</button>
      </div>
      
      <div>
        <button onClick={() => dispatch({ type: 'filter', payload: 'all' })}>
          All ({state.todos.length})
        </button>
        <button onClick={() => dispatch({ type: 'filter', payload: 'active' })}>
          Active ({state.todos.filter(t => !t.completed).length})
        </button>
        <button onClick={() => dispatch({ type: 'filter', payload: 'completed' })}>
          Completed ({state.todos.filter(t => t.completed).length})
        </button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'toggle', payload: todo.id })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'delete', payload: todo.id })}>
              Delete
            </button>
          </li>
        ))}
      </ul>
      
      {state.todos.some(t => t.completed) && (
        <button onClick={() => dispatch({ type: 'clear' })}>
          Clear Completed
        </button>
      )}
    </div>
  );
}
```

## 🔄 useCallback

### 📦 พื้นฐาน useCallback
```typescript
// Basic usage
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// With parameters
const handleClick = useCallback((id: string) => {
  onItemClick(id);
}, [onItemClick]);

// Without dependencies (runs once)
const initialize = useCallback(() => {
  console.log('Initialized');
}, []);
```

### 🎯 useCallback Use Cases
```typescript
// Prevent unnecessary re-renders
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  // Without useCallback - function created on every render
  const handleClick = () => {
    console.log('Button clicked');
  };
  
  // With useCallback - function memoized
  const handleClickMemoized = useCallback(() => {
    console.log('Button clicked');
  }, []); // Empty dependency array
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ExpensiveChildComponent onClick={handleClickMemoized} />
    </div>
  );
}

const ExpensiveChildComponent = React.memo(({ onClick }: { onClick: () => void }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click me</button>;
});
```

### 🔄 useCallback กับ Dependencies
```typescript
function SearchComponent({ data }: { data: Item[] }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState<Item[]>([]);
  
  // Memoized search function
  const search = useCallback((term: string) => {
    const filtered = data.filter(item =>
      item.name.toLowerCase().includes(term.toLowerCase())
    );
    setResults(filtered);
  }, [data]); // Recreated when data changes
  
  // Memoized debounce function
  const debouncedSearch = useCallback(
    debounce((term: string) => {
      search(term);
    }, 300),
    [search]
  );
  
  useEffect(() => {
    debouncedSearch(searchTerm);
  }, [searchTerm, debouncedSearch]);
  
  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Simple debounce implementation
function debounce<T extends (...args: any[]) => any>(
  func: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: NodeJS.Timeout;
  
  return (...args: Parameters<T>) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func(...args), delay);
  };
}
```

## 🔄 useMemo

### 📦 พื้นฐาน useMemo
```typescript
// Basic usage
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

// Complex calculation
const expensiveResult = useMemo(() => {
  console.log('Computing expensive value...');
  return items.reduce((sum, item) => sum + item.value, 0);
}, [items]);

// Memoized object
const memoizedObject = useMemo(() => ({
  name: userData.name,
  email: userData.email,
  age: userData.age
}), [userData.name, userData.email, userData.age]);
```

### 🎯 useMemo Use Cases
```typescript
// Expensive calculations
function ExpensiveCalculation({ numbers }: { numbers: number[] }) {
  const sum = useMemo(() => {
    console.log('Calculating sum...');
    return numbers.reduce((acc, num) => acc + num, 0);
  }, [numbers]);
  
  const average = useMemo(() => {
    console.log('Calculating average...');
    return numbers.length > 0 ? sum / numbers.length : 0;
  }, [numbers, sum]);
  
  return (
    <div>
      <p>Sum: {sum}</p>
      <p>Average: {average}</p>
    </div>
  );
}

// Filtered data
function FilteredList({ items, filter }: { items: Item[]; filter: string }) {
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item =>
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  const sortedItems = useMemo(() => {
    console.log('Sorting items...');
    return [...filteredItems].sort((a, b) => a.name.localeCompare(b.name));
  }, [filteredItems]);
  
  return (
    <ul>
      {sortedItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// Memoized components
function MemoizedComponent({ data }: { data: ComplexData }) {
  const processedData = useMemo(() => {
    return {
      formattedData: formatData(data),
      statistics: calculateStatistics(data),
      chartData: prepareChartData(data)
    };
  }, [data]);
  
  return (
    <div>
      <DataDisplay data={processedData.formattedData} />
      <Statistics stats={processedData.statistics} />
      <Chart data={processedData.chartData} />
    </div>
  );
}
```

## 🔄 useRef

### 📦 พื้นฐาน useRef
```typescript
// DOM reference
const inputRef = useRef<HTMLInputElement>(null);

// Mutable value
const countRef = useRef(0);

// Initialize with value
const initialValueRef = useRef('initial value');

// Usage
function InputFocus() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}
```

### 🎯 useRef Use Cases
```typescript
// Access DOM elements
function ScrollToBottom() {
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const [messages, setMessages] = useState<string[]>([]);
  
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
  
  useEffect(() => {
    scrollToBottom();
  }, [messages]);
  
  const addMessage = () => {
    setMessages(prev => [...prev, `Message ${prev.length + 1}`]);
  };
  
  return (
    <div>
      <div style={{ height: '200px', overflow: 'auto', border: '1px solid #ccc' }}>
        {messages.map((msg, index) => (
          <div key={index}>{msg}</div>
        ))}
        <div ref={messagesEndRef} />
      </div>
      <button onClick={addMessage}>Add Message</button>
    </div>
  );
}

// Store previous values
function PreviousValue({ value }: { value: any }) {
  const prevValueRef = useRef();
  
  useEffect(() => {
    prevValueRef.current = value;
  });
  
  const prevValue = prevValueRef.current;
  
  return (
    <div>
      <p>Current: {value}</p>
      <p>Previous: {prevValue}</p>
    </div>
  );
}

// Mutable counter
function MutableCounter() {
  const countRef = useRef(0);
  const [renderCount, setRenderCount] = useState(0);
  
  const increment = () => {
    countRef.current += 1;
    console.log('Count:', countRef.current);
  };
  
  const forceRender = () => {
    setRenderCount(prev => prev + 1);
  };
  
  return (
    <div>
      <p>Count (ref): {countRef.current}</p>
      <p>Render count: {renderCount}</p>
      <button onClick={increment}>Increment (no re-render)</button>
      <button onClick={forceRender}>Force Render</button>
    </div>
  );
}

// Timer management
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  
  const startTimer = () => {
    if (intervalRef.current) return;
    
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };
  
  const resetTimer = () => {
    stopTimer();
    setSeconds(0);
  };
  
  useEffect(() => {
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);
  
  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <button onClick={resetTimer}>Reset</button>
    </div>
  );
}
```

## 🔄 useLayoutEffect

### 📦 พื้นฐาน useLayoutEffect
```typescript
// useLayoutEffect runs synchronously after DOM update
function MeasureElement() {
  const [height, setHeight] = useState(0);
  const elementRef = useRef<HTMLDivElement>(null);
  
  useLayoutEffect(() => {
    if (elementRef.current) {
      const rect = elementRef.current.getBoundingClientRect();
      setHeight(rect.height);
    }
  }, []);
  
  return (
    <div>
      <div ref={elementRef} style={{ padding: '20px', background: '#f0f0f0' }}>
        This element's height is measured
      </div>
      <p>Height: {height}px</p>
    </div>
  );
}

// Prevent layout shift
function PreventLayoutShift() {
  const [isVisible, setIsVisible] = useState(false);
  const contentRef = useRef<HTMLDivElement>(null);
  const [contentHeight, setContentHeight] = useState(0);
  
  useLayoutEffect(() => {
    if (contentRef.current) {
      const height = contentRef.current.scrollHeight;
      setContentHeight(height);
    }
  }, [isVisible]);
  
  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        Toggle Content
      </button>
      <div
        style={{
          height: isVisible ? contentHeight : 0,
          overflow: 'hidden',
          transition: 'height 0.3s ease'
        }}
      >
        <div ref={contentRef}>
          <h3>Hidden Content</h3>
          <p>This content can be shown or hidden</p>
          <p>Without causing layout shift</p>
        </div>
      </div>
    </div>
  );
}
```

## 🔄 useImperativeHandle

### 📦 พื้นฐาน useImperativeHandle
```typescript
// Forward ref with useImperativeHandle
const CustomInput = React.forwardRef<HTMLInputElement, CustomInputProps>(
  ({ value, onChange, ...props }, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);
    
    useImperativeHandle(ref, () => ({
      focus: () => {
        inputRef.current?.focus();
      },
      clear: () => {
        onChange('');
        inputRef.current?.focus();
      },
      getValue: () => inputRef.current?.value || ''
    }));
    
    return (
      <input
        ref={inputRef}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        {...props}
      />
    );
  }
);

// Usage
function ParentComponent() {
  const inputRef = useRef<{ focus: () => void; clear: () => void; getValue: () => string }>(null);
  const [value, setValue] = useState('');
  
  const handleFocus = () => {
    inputRef.current?.focus();
  };
  
  const handleClear = () => {
    inputRef.current?.clear();
  };
  
  const handleGetValue = () => {
    console.log('Current value:', inputRef.current?.getValue());
  };
  
  return (
    <div>
      <CustomInput ref={inputRef} value={value} onChange={setValue} />
      <button onClick={handleFocus}>Focus</button>
      <button onClick={handleClear}>Clear</button>
      <button onClick={handleGetValue}>Get Value</button>
    </div>
  );
}
```

## 🎯 Use Cases

### 1. **Complex Form with useReducer**
```typescript
interface FormState {
  fields: {
    name: string;
    email: string;
    age: string;
    address: {
      street: string;
      city: string;
      zip: string;
    };
  };
  errors: Record<string, string>;
  isSubmitting: boolean;
}

type FormAction =
  | { type: 'SET_FIELD'; field: string; value: string }
  | { type: 'SET_ERROR'; field: string; error: string }
  | { type: 'CLEAR_ERROR'; field: string }
  | { type: 'SET_SUBMITTING'; payload: boolean }
  | { type: 'RESET' };

function formReducer(state: FormState, action: FormAction): FormState {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        fields: {
          ...state.fields,
          [action.field]: action.value
        }
      };
    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.field]: action.error
        }
      };
    case 'CLEAR_ERROR':
      const newErrors = { ...state.errors };
      delete newErrors[action.field];
      return {
        ...state,
        errors: newErrors
      };
    case 'SET_SUBMITTING':
      return {
        ...state,
        isSubmitting: action.payload
      };
    case 'RESET':
      return {
        fields: {
          name: '',
          email: '',
          age: '',
          address: {
            street: '',
            city: '',
            zip: ''
          }
        },
        errors: {},
        isSubmitting: false
      };
    default:
      return state;
  }
}

function ComplexForm() {
  const [state, dispatch] = useReducer(formReducer, {
    fields: {
      name: '',
      email: '',
      age: '',
      address: {
        street: '',
        city: '',
        zip: ''
      }
    },
    errors: {},
    isSubmitting: false
  });
  
  const validateField = useCallback((field: string, value: string) => {
    let error = '';
    
    switch (field) {
      case 'name':
        if (!value.trim()) error = 'Name is required';
        break;
      case 'email':
        if (!value.trim()) error = 'Email is required';
        else if (!/\S+@\S+\.\S+/.test(value)) error = 'Email is invalid';
        break;
      case 'age':
        if (!value) error = 'Age is required';
        else if (isNaN(Number(value)) || Number(value) < 0) error = 'Age must be a positive number';
        break;
    }
    
    return error;
  }, []);
  
  const setField = useCallback((field: string, value: string) => {
    dispatch({ type: 'SET_FIELD', field, value });
    
    const error = validateField(field, value);
    if (error) {
      dispatch({ type: 'SET_ERROR', field, error });
    } else {
      dispatch({ type: 'CLEAR_ERROR', field });
    }
  }, [validateField]);
  
  const handleSubmit = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Validate all fields
    let isValid = true;
    Object.entries(state.fields).forEach(([field, value]) => {
      if (typeof value === 'string') {
        const error = validateField(field, value);
        if (error) {
          dispatch({ type: 'SET_ERROR', field, error });
          isValid = false;
        }
      }
    });
    
    if (!isValid) return;
    
    dispatch({ type: 'SET_SUBMITTING', payload: true });
    
    try {
      await submitForm(state.fields);
      dispatch({ type: 'RESET' });
    } catch (error) {
      console.error('Submit error:', error);
    } finally {
      dispatch({ type: 'SET_SUBMITTING', payload: false });
    }
  }, [state.fields, validateField]);
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name:</label>
        <input
          value={state.fields.name}
          onChange={(e) => setField('name', e.target.value)}
        />
        {state.errors.name && <span className="error">{state.errors.name}</span>}
      </div>
      
      <div>
        <label>Email:</label>
        <input
          type="email"
          value={state.fields.email}
          onChange={(e) => setField('email', e.target.value)}
        />
        {state.errors.email && <span className="error">{state.errors.email}</span>}
      </div>
      
      <div>
        <label>Age:</label>
        <input
          value={state.fields.age}
          onChange={(e) => setField('age', e.target.value)}
        />
        {state.errors.age && <span className="error">{state.errors.age}</span>}
      </div>
      
      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

### 2. **Performance Optimization with useCallback and useMemo**
```typescript
function PerformanceOptimizedList({ items, onItemClick }: {
  items: Item[];
  onItemClick: (item: Item) => void;
}) {
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState<'name' | 'date'>('name');
  
  // Memoized filtered items
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);
  
  // Memoized sorted items
  const sortedItems = useMemo(() => {
    return [...filteredItems].sort((a, b) => {
      if (sortBy === 'name') {
        return a.name.localeCompare(b.name);
      } else {
        return new Date(a.date).getTime() - new Date(b.date).getTime();
      }
    });
  }, [filteredItems, sortBy]);
  
  // Memoized click handler
  const handleClick = useCallback((item: Item) => {
    onItemClick(item);
  }, [onItemClick]);
  
  // Memoized item component
  const ItemComponent = useCallback(({ item }: { item: Item }) => (
    <div
      className="item"
      onClick={() => handleClick(item)}
      style={{ cursor: 'pointer', padding: '8px', border: '1px solid #ccc' }}
    >
      <h4>{item.name}</h4>
      <p>{item.description}</p>
      <small>{new Date(item.date).toLocaleDateString()}</small>
    </div>
  ), [handleClick]);
  
  return (
    <div>
      <div>
        <input
          type="text"
          placeholder="Search..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value as 'name' | 'date')}>
          <option value="name">Sort by Name</option>
          <option value="date">Sort by Date</option>
        </select>
      </div>
      
      <div>
        {sortedItems.map(item => (
          <ItemComponent key={item.id} item={item} />
        ))}
      </div>
    </div>
  );
}
```

### 3. **Custom Hook with useRef and useLayoutEffect**
```typescript
function useElementSize<T extends HTMLElement>() {
  const elementRef = useRef<T>(null);
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useLayoutEffect(() => {
    const element = elementRef.current;
    if (!element) return;
    
    const updateSize = () => {
      const rect = element.getBoundingClientRect();
      setSize({ width: rect.width, height: rect.height });
    };
    
    updateSize();
    
    const resizeObserver = new ResizeObserver(updateSize);
    resizeObserver.observe(element);
    
    return () => {
      resizeObserver.disconnect();
    };
  }, []);
  
  return { elementRef, size };
}

function ResizableComponent() {
  const { elementRef, size } = useElementSize<HTMLDivElement>();
  
  return (
    <div>
      <div
        ref={elementRef}
        style={{
          width: '100%',
          height: '200px',
          background: '#f0f0f0',
          resize: 'both',
          overflow: 'auto',
          border: '1px solid #ccc'
        }}
      >
        Resize this element
      </div>
      <p>Size: {size.width.toFixed(0)} x {size.height.toFixed(0)}px</p>
    </div>
  );
}
```

## ⚠️ Edge Cases

### 1. **useReducer vs useState**
```typescript
// ❌ Bad - using useState for complex state
function BadComplexState() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // Multiple state updates can be problematic
  const fetchData = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const userData = await fetchUser();
      const postsData = await fetchPosts();
      
      setUser(userData);
      setPosts(postsData); // Multiple re-renders
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  return <div></div>;
}

// ✅ Good - using useReducer for complex state
function GoodComplexState() {
  const [state, dispatch] = useReducer(appReducer, {
    user: null,
    posts: [],
    loading: false,
    error: null
  });
  
  const fetchData = async () => {
    dispatch({ type: 'FETCH_START' });
    
    try {
      const userData = await fetchUser();
      const postsData = await fetchPosts();
      
      dispatch({ type: 'FETCH_SUCCESS', payload: { user: userData, posts: postsData } });
    } catch (err) {
      dispatch({ type: 'FETCH_ERROR', payload: err.message });
    }
  };
  
  return <div></div>;
}
```

### 2. **useCallback dependency issues**
```typescript
// ❌ Bad - missing dependencies
function BadCallback() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log(count); // Stale closure - always 0
  }, []); // Missing count dependency
  
  return <button onClick={handleClick}>Click</button>;
}

// ✅ Good - proper dependencies
function GoodCallback() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    setCount(c => c + 1); // Use functional update
  }, []);
  
  return <button onClick={handleClick}>Click</button>;
}
```

### 3. **useMemo overuse**
```typescript
// ❌ Bad - memoizing simple values
function BadMemo() {
  const [name, setName] = useState('');
  
  const memoizedName = useMemo(() => name, [name]); // Unnecessary
  
  return <div>{memoizedName}</div>;
}

// ✅ Good - memoizing expensive calculations
function GoodMemo({ items }: { items: Item[] }) {
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>Total: {expensiveValue}</div>;
}
```

### 4. **useRef vs useState**
```typescript
// ❌ Bad - using useState for non-rendering values
function BadRefUsage() {
  const [renderCount, setRenderCount] = useState(0);
  
  const handleClick = () => {
    setRenderCount(prev => prev + 1); // Causes re-render
  };
  
  return <button onClick={handleClick}>Click</button>;
}

// ✅ Good - using useRef for non-rendering values
function GoodRefUsage() {
  const renderCount = useRef(0);
  
  const handleClick = () => {
    renderCount.current += 1; // No re-render
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

## 🎯 สรุป

### ✅ สิ่งที่ควรรู้
- **useReducer**: จัดการ state ที่ซับซ้อนด้วย reducer pattern
- **useCallback**: ป้องกันการสร้างฟังก์ชันใหม่ไม่จำเป็น
- **useMemo**: คำนวณค่าที่แพงและ cache ผลลัพธ์
- **useRef**: อ้างอิง DOM elements และเก็บค่า mutable
- **useLayoutEffect**: ทำงานก่อนการ paint สำหรับ DOM measurements
- **useImperativeHandle**: เปิดเผย methods จาก child components

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Custom Hooks** ใน [CUSTOM_HOOKS.md](CUSTOM_HOOKS.md)
2. เรียนรู้ **Performance** ใน [PERFORMANCE.md](PERFORMANCE.md)
3. ทดลองสร้าง custom hooks ที่ซับซ้อน

### 💡 แนวทางการฝึก
- สร้าง complex forms ด้วย useReducer
- ฝึกใช้ useCallback และ useMemo สำหรับ performance
- ทดลอง useRef สำหรับ DOM manipulation
- ฝึกสร้าง imperative handles สำหรับ component APIs

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
