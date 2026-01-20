# 🎣 React Hooks - ครบถ้วนทุก Hooks

## 📋 สารบัฐ
- [Hooks คืออะไร](#-hooks-คืออะไร)
- [Basic Hooks](#-basic-hooks)
- [Advanced Hooks](#-advanced-hooks)
- [Custom Hooks](#-custom-hooks)
- [Rules of Hooks](#-rules-of-hooks)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🎯 Hooks คืออะไร

Hooks เป็นฟีเจอร์ใน React 16.8 ที่ให้ functional components สามารถใช้ state และ lifecycle features ได้

### ✨ ประโยชน์ของ Hooks
- **Reuse stateful logic**: ใช้ logic ซ้ำได้โดยไม่ต้องเปลี่ยน component hierarchy
- **Simplify components**: ไม่ต้องใช้ classes และ lifecycle methods
- **Better composition**: แบ่ง logic ออกเป็นส่วนๆ ที่ใช้ซ้ำได้
- **Easier testing**: ไม่ต้อง mock lifecycle methods

### 📚 โครงสร้างเอกสาร
- **[BASIC_HOOKS.md](BASIC_HOOKS.md)** - useState, useEffect, useContext
- **[ADVANCED_HOOKS.md](ADVANCED_HOOKS.md)** - useReducer, useCallback, useMemo, useRef
- **[CUSTOM_HOOKS.md](CUSTOM_HOOKS.md)** - Custom hooks และ patterns

## 🎣 Basic Hooks

### 1. **useState**
```typescript
// Basic usage
const [state, setState] = useState(initialValue);

// With TypeScript
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>('');
const [user, setUser] = useState<User | null>(null);

// Lazy initialization
const [data, setData] = useState(() => expensiveCalculation());

// Object state
const [formData, setFormData] = useState({
  name: '',
  email: '',
  age: 0
});
```

### 2. **useEffect**
```typescript
// Basic effect
useEffect(() => {
  // Side effect here
  console.log('Component mounted or updated');
});

// With cleanup
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Timer tick');
  }, 1000);
  
  return () => clearInterval(timer); // Cleanup
}, []);

// With dependencies
useEffect(() => {
  fetchUserData(userId);
}, [userId]); // Run when userId changes
```

### 3. **useContext**
```typescript
// Create context
const ThemeContext = createContext('light');

// Use context
const theme = useContext(ThemeContext);

// With TypeScript
const user = useContext<UserContext>(UserContext);
```

## 🎣 Advanced Hooks

### 1. **useReducer**
```typescript
// Reducer function
function counterReducer(state: State, action: Action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// Use reducer
const [state, dispatch] = useReducer(counterReducer, { count: 0 });
```

### 2. **useCallback**
```typescript
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]); // Dependencies
```

### 3. **useMemo**
```typescript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

### 4. **useRef**
```typescript
// DOM reference
const inputRef = useRef<HTMLInputElement>(null);

// Mutable value
const countRef = useRef(0);
countRef.current = countRef.current + 1;
```

## 🎣 Custom Hooks

### 1. **Basic Custom Hook**
```typescript
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  
  return { count, increment, decrement, reset };
}
```

### 2. **Data Fetching Hook**
```typescript
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      try {
        const response = await fetch(url);
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
}
```

## 📋 Rules of Hooks

### 1. **Only Call Hooks at the Top Level**
```typescript
// ✅ Good
function MyComponent() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    // effect logic
  }, []);
  return <div>{count}</div>;
}

// ❌ Bad
function MyComponent() {
  if (condition) {
    const [count, setCount] = useState(0); // Don't do this!
  }
  return <div></div>;
}
```

### 2. **Only Call Hooks from React Functions**
```typescript
// ✅ Good
function MyComponent() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

// ❌ Bad
function regularFunction() {
  const [count, setCount] = useState(0); // Don't do this!
}
```

## 🎯 Use Cases

### 1. **Form Management**
```typescript
function useForm(initialValues: any) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  
  const setValue = useCallback((name: string, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
  }, []);
  
  const setError = useCallback((name: string, error: string) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  }, []);
  
  const validate = useCallback(() => {
    // validation logic
    return isValid;
  }, [values]);
  
  return { values, errors, setValue, setError, validate };
}
```

### 2. **Local Storage**
```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });
  
  const setValue = useCallback((value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  }, [key]);
  
  return [storedValue, setValue] as const;
}
```

### 3. **Debounce**
```typescript
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(handler);
  }, [value, delay]);
  
  return debouncedValue;
}
```

## ⚠️ Edge Cases

### 1. **Stale Closure**
```typescript
// ❌ Bad - stale closure
function BadCounter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // Always 0
    }, 1000);
    
    return () => clearInterval(timer);
  }, []); // Empty dependency array
  
  return <div>{count}</div>;
}

// ✅ Good - use functional update
function GoodCounter() {
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

### 2. **Infinite Loop**
```typescript
// ❌ Bad - infinite loop
function BadEffect() {
  const [data, setData] = useState({});
  
  useEffect(() => {
    setData({}); // Triggers effect again
  }, [data]); // data in dependencies
  
  return <div></div>;
}

// ✅ Good - proper dependencies
function GoodEffect() {
  const [data, setData] = useState({});
  
  useEffect(() => {
    fetchData().then(setData);
  }, []); // No data in dependencies
  
  return <div></div>;
}
```

### 3. **Memory Leaks**
```typescript
// ❌ Bad - memory leak
function BadAsync() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(result => {
      setData(result); // May update after unmount
    });
  }, []);
  
  return <div></div>;
}

// ✅ Good - cleanup
function GoodAsync() {
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
  
  return <div></div>;
}
```

## 🎯 สรุป

### ✅ สิ่งที่ควรรู้
- Hooks ให้ functional components ใช้ state และ lifecycle ได้
- ต้องทำตาม Rules of Hooks เสมอ
- Custom hooks ใช้แชร์ logic ระหว่าง components
- ระวัง stale closures และ memory leaks

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Basic Hooks** ใน [BASIC_HOOKS.md](BASIC_HOOKS.md)
2. เรียนรู้ **Advanced Hooks** ใน [ADVANCED_HOOKS.md](ADVANCED_HOOKS.md)
3. ศึกษา **Custom Hooks** ใน [CUSTOM_HOOKS.md](CUSTOM_HOOKS.md)

### 💡 แนวทางการฝึก
- สร้าง custom hooks สำหรับ logic ที่ใช้ซ้ำ
- ฝึกใช้ hooks ในสถานการณ์ต่างๆ
- ทดลองสร้าง data fetching hooks
- ฝึกจัดการ edge cases ของ hooks

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2024*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
