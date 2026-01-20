# 🌱 React Basics - พื้นฐาน React

## 📋 สารบัฐ
- [React คืออะไร](#-react-คืออะไร)
- [การติดตั้ง](#-การติดตั้ง)
- [JSX คืออะไร](#-jsx-คืออะไร)
- [Virtual DOM](#-virtual-dom)
- [การสร้าง Component แรก](#-การสร้าง-component-แรก)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🎯 React คืออะไร

React เป็น JavaScript library สำหรับสร้าง User Interface ที่พัฒนาโดย Facebook มีคุณสมบัติหลักคือ:

### ✨ คุณสมบัติหลัก
- **Component-based**: แบ่ง UI เป็นส่วนๆ ที่ใช้ซ้ำได้
- **Declarative**: บอกว่าต้องการให้ UI เป็นอย่างไร ไม่ใช่วิธีทำ
- **Virtual DOM**: อัปเดต UI อย่างมีประสิทธิภาพ
- **Learn Once, Write Anywhere**: ใช้ได้ทั้ง web, mobile, desktop

### 🔄 วิธีทำงาน
```typescript
// 1. สร้าง Component
function Welcome({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

// 2. ใช้ Component
function App() {
  return <Welcome name="World" />;
}

// 3. Render หน้าจอ
ReactDOM.render(<App />, document.getElementById('root'));
```

## 🛠️ การติดตั้ง

### 1. ใช้ Create React App (เหมาะสำหรับมือใหม่)
```bash
npx create-react-app my-app
cd my-app
npm start
```

### 2. ใช้ Vite (เร็วกว่า)
```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

### 3. ใช้ Next.js (Full-stack)
```bash
npx create-next-app@latest my-app
cd my-app
npm run dev
```

### 4. ติดตั้งด้วย TypeScript
```bash
npx create-react-app my-app --template typescript
# หรือ
npm create vite@latest my-app -- --template react-ts
```

## 🎨 JSX คืออะไร

JSX เป็น syntax extension สำหรับ JavaScript ที่ให้เขียน HTML ใน JavaScript ได้

### 📝 พื้นฐาน JSX
```typescript
// JSX ปกติ
const element = <h1>Hello, World!</h1>;

// ใช้ตัวแปร
const name = "John";
const element = <h1>Hello, {name}!</h1>;

// ใช้ expression
const element = <h1>The sum is {2 + 2}</h1>;

// ใช้ function
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = { firstName: 'John', lastName: 'Doe' };
const element = <h1>Hello, {formatName(user)}!</h1>;
```

### 🏷️ JSX Attributes
```typescript
// String attributes
const element = <div className="container">Hello</div>;

// JavaScript expression
const element = <img src={user.avatarUrl} alt={user.name} />;

// Boolean attributes
const element = <button disabled={isDisabled}>Click</button>;

// Object spread
const props = { className: "btn", disabled: false };
const element = <button {...props}>Click</button>;
```

### ⚠️ JSX Rules
```typescript
// ✅ ดี - ต้องมี root element เดียว
const element = (
  <div>
    <h1>Title</h1>
    <p>Content</p>
  </div>
);

// ✅ ดี - ใช้ Fragment แทน div
const element = (
  <>
    <h1>Title</h1>
    <p>Content</p>
  </>
);

// ❌ ไม่ดี - มีหลาย root elements
const element = (
  <h1>Title</h1>
  <p>Content</p>
);

// ✅ ดี - ใช้ camelCase
const element = <div className="container" onClick={handleClick}>Hello</div>;

// ❌ ไม่ดี - ใช้ class แทน className
const element = <div class="container">Hello</div>;
```

## 🌳 Virtual DOM

Virtual DOM เป็นการ copy ของ Real DOM ที่อยู่ใน memory ใช้สำหรับเปรียบเทียบการเปลี่ยนแปลง

### 🔄 การทำงานของ Virtual DOM
```typescript
// 1. Initial Render
// Virtual DOM ถูกสร้างขึ้น
const virtualDOM = {
  type: 'div',
  props: { className: 'container' },
  children: [
    { type: 'h1', props: {}, children: ['Hello'] },
    { type: 'p', props: {}, children: ['World'] }
  ]
};

// 2. State Change
// Component state เปลี่ยนแปลง
setState({ message: 'New Message' });

// 3. Reconciliation
// React เปรียบเทียบ Virtual DOM เก่ากับใหม่
const changes = diff(oldVirtualDOM, newVirtualDOM);

// 4. Minimal Updates
// อัปเดตเฉพาะส่วนที่เปลี่ยนแปลงใน Real DOM
updateRealDOM(changes);
```

### ⚡ ประโยชน์ของ Virtual DOM
- **Performance**: อัปเดตเฉพาะส่วนที่จำเป็น
- **Batching**: รวมการอัปเดตหลายๆ ครั้ง
- **Declarative**: ไม่ต้องจัดการ DOM โดยตรง

## 🏗️ การสร้าง Component แรก

### 📦 Functional Component
```typescript
// Component แรก
function Welcome({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

// ใช้งาน
function App() {
  return (
    <div>
      <Welcome name="Alice" />
      <Welcome name="Bob" />
    </div>
  );
}
```

### 🎯 Component ที่ซับซ้อนขึ้น
```typescript
interface User {
  name: string;
  age: number;
  isOnline: boolean;
}

interface UserProfileProps {
  user: User;
  onMessage: () => void;
}

function UserProfile({ user, onMessage }: UserProfileProps) {
  const statusColor = user.isOnline ? 'green' : 'red';
  const statusText = user.isOnline ? 'Online' : 'Offline';

  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>Age: {user.age}</p>
      <p>
        Status: 
        <span style={{ color: statusColor }}>
          {statusText}
        </span>
      </p>
      {user.isOnline && (
        <button onClick={onMessage}>
          Send Message
        </button>
      )}
    </div>
  );
}
```

### 🔄 Component ที่มีการคำนวณ
```typescript
interface CalculatorProps {
  a: number;
  b: number;
  operation: 'add' | 'subtract' | 'multiply' | 'divide';
}

function Calculator({ a, b, operation }: CalculatorProps) {
  const calculate = () => {
    switch (operation) {
      case 'add':
        return a + b;
      case 'subtract':
        return a - b;
      case 'multiply':
        return a * b;
      case 'divide':
        return b !== 0 ? a / b : 'Cannot divide by zero';
      default:
        return 'Invalid operation';
    }
  };

  const result = calculate();

  return (
    <div className="calculator">
      <p>{a} {operation} {b} = {result}</p>
    </div>
  );
}
```

## 🎯 Use Cases

### 1. **Display Component** - แสดงข้อมูล
```typescript
// Use Case: แสดงข้อมูลผู้ใช้
function UserCard({ user }: { user: User }) {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}
```

### 2. **Form Component** - ฟอร์ม
```typescript
// Use Case: ฟอร์มล็อกอิน
function LoginForm({ onLogin }: { onLogin: (email: string, password: string) => void }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onLogin(email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

### 3. **List Component** - รายการ
```typescript
// Use Case: แสดงรายการสินค้า
function ProductList({ products }: { products: Product[] }) {
  return (
    <div className="product-list">
      {products.map(product => (
        <div key={product.id} className="product-item">
          <h3>{product.name}</h3>
          <p>Price: ${product.price}</p>
        </div>
      ))}
    </div>
  );
}
```

### 4. **Layout Component** - โครงสร้าง
```typescript
// Use Case: Layout หลัก
function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="layout">
      <header>
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
      </header>
      <main>{children}</main>
      <footer>
        <p>&copy; 2024 My App</p>
      </footer>
    </div>
  );
}
```

## ⚠️ Edge Cases

### 1. **Props เป็น null/undefined**
```typescript
// ❌ ไม่ดี - อาจเกิด error
function UserCard({ user }: { user: User }) {
  return <h1>{user.name}</h1>; // Error ถ้า user เป็น null
}

// ✅ ดี - ตรวจสอบก่อน
function UserCard({ user }: { user: User | null }) {
  if (!user) {
    return <div>Loading...</div>;
  }
  return <h1>{user.name}</h1>;
}

// ✅ ดีกว่า - ใช้ default props
function UserCard({ user }: { user: User }) {
  return <h1>{user?.name || 'Unknown'}</h1>;
}
```

### 2. **Array ว่าง**
```typescript
// ❌ ไม่ดี - อาจเกิด error
function TodoList({ todos }: { todos: Todo[] }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}

// ✅ ดี - ตรวจสอบ array ว่าง
function TodoList({ todos }: { todos: Todo[] }) {
  if (todos.length === 0) {
    return <p>No todos yet!</p>;
  }

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

### 3. **การจัดการ Error**
```typescript
// ❌ ไม่ดี - ไม่จัดการ error
function Image({ src, alt }: { src: string; alt: string }) {
  return <img src={src} alt={alt} />;
}

// ✅ ดี - จัดการ error
function Image({ src, alt }: { src: string; alt: string }) {
  const [hasError, setHasError] = useState(false);

  if (hasError) {
    return <div className="image-error">Failed to load image</div>;
  }

  return (
    <img
      src={src}
      alt={alt}
      onError={() => setHasError(true)}
    />
  );
}
```

### 4. **การจัดการ Loading**
```typescript
// ✅ ดี - จัดการ loading state
function DataComponent({ data, loading, error }: {
  data: Data[] | null;
  loading: boolean;
  error: string | null;
}) {
  if (loading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error: {error}</div>;
  }

  if (!data || data.length === 0) {
    return <div>No data available</div>;
  }

  return (
    <ul>
      {data.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### 5. **การจัดการ Large Lists**
```typescript
// ❌ ไม่ดี - render ทั้งหมดพร้อมกัน
function LongList({ items }: { items: Item[] }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// ✅ ดี - ใช้ virtualization หรือ pagination
function LongList({ items }: { items: Item[] }) {
  const [visibleItems, setVisibleItems] = useState(items.slice(0, 50));

  const loadMore = () => {
    setVisibleItems(prev => [
      ...prev,
      ...items.slice(prev.length, prev.length + 50)
    ]);
  };

  return (
    <div>
      <ul>
        {visibleItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
      {visibleItems.length < items.length && (
        <button onClick={loadMore}>Load More</button>
      )}
    </div>
  );
}
```

## 🎯 สรุป

### ✅ สิ่งที่ควรรู้
- React เป็น library สำหรับสร้าง UI
- JSX คือ syntax สำหรับเขียน HTML ใน JavaScript
- Virtual DOM ช่วยให้การอัปเดตมีประสิทธิภาพ
- Component เป็นหน่วยพื้นฐานของ React

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **Components & Props** ใน [COMPONENTS.md](COMPONENTS.md)
2. เรียนรู้ **State & Lifecycle** ใน [STATE.md](STATE.md)
3. ทดลองสร้าง project แรกด้วย Create React App

### 💡 แนวทานการฝึก
- สร้าง component ง่ายๆ เช่น การ์ดผู้ใช้
- ลองใช้ JSX กับข้อมูลต่างๆ
- ทดลองจัดการ edge cases ที่พบบ่อย
- ฝึกสร้าง layout และ form ง่ายๆ

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2024*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
