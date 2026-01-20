# 🧩 React Components & Props - คอมโพเนนต์และพร็อพส์

## 📋 สารบัฐ
- [Component Types](#-component-types)
- [Props คืออะไร](#-props-คืออะไร)
- [Component Composition](#-component-composition)
- [Children Props](#-children-props)
- [Conditional Rendering](#-conditional-rendering)
- [Use Cases](#-use-cases)
- [Edge Cases](#-edge-cases)

## 🏗️ Component Types

### 1. **Functional Components** (แนะนำ)
```typescript
// Simple Functional Component
function Welcome({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

// Arrow Function Component
const Welcome = ({ name }: { name: string }) => {
  return <h1>Hello, {name}!</h1>;
};

// With TypeScript interface
interface WelcomeProps {
  name: string;
  age?: number; // Optional prop
}

const Welcome: React.FC<WelcomeProps> = ({ name, age }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>Age: {age}</p>}
    </div>
  );
};
```

### 2. **Class Components** (Legacy)
```typescript
// Class Component
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// With Props Interface
interface WelcomeProps {
  name: string;
}

class Welcome extends React.Component<WelcomeProps> {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// With State
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
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

### 3. **Pure Components**
```typescript
// Functional Component with React.memo
const Welcome = React.memo(({ name }: { name: string }) => {
  console.log('Welcome rendered');
  return <h1>Hello, {name}!</h1>;
});

// Class Pure Component
class Welcome extends React.PureComponent {
  render() {
    console.log('Welcome rendered');
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

## 📦 Props คืออะไร

Props เป็นข้อมูลที่ส่งจาก parent component ไปยัง child component

### 🔄 การส่ง Props
```typescript
// Parent Component
function App() {
  const user = {
    name: 'John Doe',
    age: 30,
    email: 'john@example.com'
  };

  return (
    <div>
      <UserProfile 
        user={user}
        onUpdate={handleUpdate}
        isAdmin={true}
      />
    </div>
  );
}

// Child Component
interface UserProfileProps {
  user: {
    name: string;
    age: number;
    email: string;
  };
  onUpdate: () => void;
  isAdmin: boolean;
}

function UserProfile({ user, onUpdate, isAdmin }: UserProfileProps) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Age: {user.age}</p>
      <p>Email: {user.email}</p>
      {isAdmin && <button onClick={onUpdate}>Edit</button>}
    </div>
  );
}
```

### 🏷️ PropTypes (JavaScript)
```typescript
import PropTypes from 'prop-types';

function Welcome({ name, age, isAdmin }) {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>Age: {age}</p>}
      {isAdmin && <p>Admin User</p>}
    </div>
  );
}

Welcome.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isAdmin: PropTypes.bool
};

Welcome.defaultProps = {
  age: 0,
  isAdmin: false
};
```

### 📝 Default Props
```typescript
// TypeScript Default Props
interface WelcomeProps {
  name: string;
  age?: number;
  isAdmin?: boolean;
}

const Welcome: React.FC<WelcomeProps> = ({ 
  name, 
  age = 0, 
  isAdmin = false 
}) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Age: {age}</p>
      {isAdmin && <p>Admin User</p>}
    </div>
  );
};

// Class Component Default Props
class Welcome extends React.Component<WelcomeProps> {
  static defaultProps: Partial<WelcomeProps> = {
    age: 0,
    isAdmin: false
  };

  render() {
    const { name, age, isAdmin } = this.props;
    return (
      <div>
        <h1>Hello, {name}!</h1>
        <p>Age: {age}</p>
        {isAdmin && <p>Admin User</p>}
      </div>
    );
  }
}
```

## 🔄 Component Composition

### 1. **Containment**
```typescript
// Container Component
function Card({ children }: { children: React.ReactNode }) {
  return (
    <div className="card">
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// Usage
function App() {
  return (
    <Card>
      <h2>Card Title</h2>
      <p>Card content goes here</p>
      <button>Action</button>
    </Card>
  );
}
```

### 2. **Specialization**
```typescript
// Generic Component
function Dialog({ title, children }: { 
  title: string; 
  children: React.ReactNode 
}) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <div className="dialog-content">
        {children}
      </div>
    </div>
  );
}

// Specialized Components
function WelcomeDialog() {
  return (
    <Dialog title="Welcome">
      <p>Thank you for visiting our application!</p>
      <button>Get Started</button>
    </Dialog>
  );
}

function ErrorDialog({ error }: { error: string }) {
  return (
    <Dialog title="Error">
      <p>{error}</p>
      <button>Close</button>
    </Dialog>
  );
}
```

### 3. **Composition with Props**
```typescript
// Base Component
function Button({ 
  variant = 'primary', 
  size = 'medium', 
  children, 
  ...props 
}: {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  children: React.ReactNode;
  [key: string]: any;
}) {
  const className = `btn btn-${variant} btn-${size}`;
  
  return (
    <button className={className} {...props}>
      {children}
    </button>
  );
}

// Specialized Components
function PrimaryButton({ children, ...props }) {
  return <Button variant="primary" {...props}>{children}</Button>;
}

function LargeButton({ children, ...props }) {
  return <Button size="large" {...props}>{children}</Button>;
}

// Usage
function App() {
  return (
    <div>
      <PrimaryButton>Click me</PrimaryButton>
      <LargeButton variant="secondary">Big Button</LargeButton>
    </div>
  );
}
```

## 👶 Children Props

### 1. **Basic Children**
```typescript
function Wrapper({ children }: { children: React.ReactNode }) {
  return (
    <div className="wrapper">
      {children}
    </div>
  );
}

// Usage
function App() {
  return (
    <Wrapper>
      <h1>Title</h1>
      <p>Content</p>
    </Wrapper>
  );
}
```

### 2. **Children as Function**
```typescript
interface DataProviderProps {
  data: any[];
  children: (data: any[]) => React.ReactNode;
}

function DataProvider({ data, children }: DataProviderProps) {
  return (
    <div className="data-provider">
      {children(data)}
    </div>
  );
}

// Usage
function App() {
  const users = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' }
  ];

  return (
    <DataProvider data={users}>
      {(data) => (
        <ul>
          {data.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      )}
    </DataProvider>
  );
}
```

### 3. **Multiple Children**
```typescript
interface LayoutProps {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  content: React.ReactNode;
  footer: React.ReactNode;
}

function Layout({ header, sidebar, content, footer }: LayoutProps) {
  return (
    <div className="layout">
      <header>{header}</header>
      <div className="main">
        <aside>{sidebar}</aside>
        <main>{content}</main>
      </div>
      <footer>{footer}</footer>
    </div>
  );
}

// Usage
function App() {
  return (
    <Layout
      header={<h1>My App</h1>}
      sidebar={<nav>Navigation</nav>}
      content={<div>Main Content</div>}
      footer={<p>&copy; 2024</p>}
    />
  );
}
```

### 4. **Children Validation**
```typescript
import React from 'react';

function Card({ children }: { children: React.ReactNode }) {
  // Validate children
  const childCount = React.Children.count(children);
  const childArray = React.Children.toArray(children);

  return (
    <div className="card">
      <div className="card-header">
        Card ({childCount} items)
      </div>
      <div className="card-content">
        {childArray.map((child, index) => (
          <div key={index} className="card-item">
            {child}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 🔄 Conditional Rendering

### 1. **If Statement**
```typescript
function UserGreeting({ isLoggedIn, user }: { 
  isLoggedIn: boolean; 
  user: User | null 
}) {
  if (isLoggedIn && user) {
    return <h1>Welcome back, {user.name}!</h1>;
  } else {
    return <h1>Please sign in.</h1>;
  }
}
```

### 2. **Ternary Operator**
```typescript
function Message({ count }: { count: number }) {
  return (
    <div>
      {count > 0 ? (
        <p>You have {count} messages</p>
      ) : (
        <p>No messages</p>
      )}
    </div>
  );
}
```

### 3. **Logical AND**
```typescript
function Notifications({ notifications }: { notifications: Notification[] }) {
  return (
    <div>
      {notifications.length > 0 && (
        <div className="notifications">
          {notifications.map(notif => (
            <div key={notif.id}>{notif.message}</div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### 4. **Switch Statement**
```typescript
function StatusBadge({ status }: { status: string }) {
  const getBadge = () => {
    switch (status) {
      case 'active':
        return <span className="badge badge-success">Active</span>;
      case 'inactive':
        return <span className="badge badge-secondary">Inactive</span>;
      case 'pending':
        return <span className="badge badge-warning">Pending</span>;
      default:
        return <span className="badge badge-default">Unknown</span>;
    }
  };

  return <div>{getBadge()}</div>;
}
```

### 5. **Component-based Conditional**
```typescript
function LoadingSpinner() {
  return <div className="spinner">Loading...</div>;
}

function ErrorMessage({ error }: { error: string }) {
  return <div className="error">Error: {error}</div>;
}

function DataDisplay({ data, loading, error }: {
  data: any[] | null;
  loading: boolean;
  error: string | null;
}) {
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data) return <div>No data</div>;

  return (
    <ul>
      {data.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

## 🎯 Use Cases

### 1. **Reusable UI Components**
```typescript
// Button Component
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

function Button({ 
  variant = 'primary', 
  size = 'medium', 
  disabled = false, 
  loading = false,
  onClick,
  children 
}: ButtonProps) {
  const className = `btn btn-${variant} btn-${size}`;
  
  return (
    <button 
      className={className}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
}

// Usage
function App() {
  return (
    <div>
      <Button onClick={() => console.log('Clicked')}>
        Click me
      </Button>
      <Button variant="secondary" size="large">
        Large Button
      </Button>
      <Button disabled loading>
        Disabled
      </Button>
    </div>
  );
}
```

### 2. **Form Components**
```typescript
// Input Component
interface InputProps {
  label: string;
  type?: string;
  value: string;
  onChange: (value: string) => void;
  error?: string;
  required?: boolean;
}

function Input({ 
  label, 
  type = 'text', 
  value, 
  onChange, 
  error, 
  required = false 
}: InputProps) {
  return (
    <div className="form-group">
      <label>
        {label}
        {required && <span className="required">*</span>}
      </label>
      <input
        type={type}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className={error ? 'error' : ''}
      />
      {error && <span className="error-message">{error}</span>}
    </div>
  );
}

// Usage
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});

  return (
    <form>
      <Input
        label="Email"
        type="email"
        value={email}
        onChange={setEmail}
        error={errors.email}
        required
      />
      <Input
        label="Password"
        type="password"
        value={password}
        onChange={setPassword}
        error={errors.password}
        required
      />
    </form>
  );
}
```

### 3. **List Components**
```typescript
// List Item Component
interface ListItemProps {
  item: {
    id: string;
    title: string;
    description?: string;
    completed?: boolean;
  };
  onToggle?: (id: string) => void;
  onDelete?: (id: string) => void;
}

function ListItem({ item, onToggle, onDelete }: ListItemProps) {
  return (
    <div className={`list-item ${item.completed ? 'completed' : ''}`}>
      <div className="item-content">
        <h3>{item.title}</h3>
        {item.description && <p>{item.description}</p>}
      </div>
      <div className="item-actions">
        {onToggle && (
          <button onClick={() => onToggle(item.id)}>
            {item.completed ? 'Undo' : 'Complete'}
          </button>
        )}
        {onDelete && (
          <button onClick={() => onDelete(item.id)}>
            Delete
          </button>
        )}
      </div>
    </div>
  );
}

// List Component
function List({ items, onToggle, onDelete }: {
  items: any[];
  onToggle?: (id: string) => void;
  onDelete?: (id: string) => void;
}) {
  if (items.length === 0) {
    return <div className="empty-state">No items found</div>;
  }

  return (
    <div className="list">
      {items.map(item => (
        <ListItem
          key={item.id}
          item={item}
          onToggle={onToggle}
          onDelete={onDelete}
        />
      ))}
    </div>
  );
}
```

## ⚠️ Edge Cases

### 1. **Props Validation**
```typescript
// ❌ ไม่ดี - ไม่ตรวจสอบ props
function UserCard({ user }: { user: User }) {
  return <h1>{user.name}</h1>; // Error ถ้า user เป็น null
}

// ✅ ดี - ตรวจสอบ props
function UserCard({ user }: { user: User | null }) {
  if (!user) {
    return <div>User not found</div>;
  }
  return <h1>{user.name}</h1>;
}

// ✅ ดีกว่า - ใช้ TypeScript
interface UserCardProps {
  user: User;
  fallback?: React.ReactNode;
}

function UserCard({ user, fallback = <div>User not found</div> }: UserCardProps) {
  if (!user) {
    return fallback;
  }
  return <h1>{user.name}</h1>;
}
```

### 2. **Children Type Checking**
```typescript
// ❌ ไม่ดี - ไม่ตรวจสอบ children
function Card({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}

// ✅ ดี - ตรวจสอบ children
function Card({ children }: { children: React.ReactNode }) {
  if (!children) {
    return <div className="empty-card">Empty card</div>;
  }

  const childArray = React.Children.toArray(children);
  if (childArray.length === 0) {
    return <div className="empty-card">Empty card</div>;
  }

  return <div className="card">{children}</div>;
}
```

### 3. **Prop Drilling Prevention**
```typescript
// ❌ ไม่ดี - Prop drilling
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <Page theme={theme} setTheme={setTheme} />
  );
}

function Page({ theme, setTheme }) {
  return (
    <Layout theme={theme} setTheme={setTheme} />
  );
}

function Layout({ theme, setTheme }) {
  return (
    <Content theme={theme} setTheme={setTheme} />
  );
}

function Content({ theme, setTheme }) {
  return (
    <Button onClick={() => setTheme('dark')}>
      Toggle Theme
    </Button>
  );
}

// ✅ ดี - ใช้ Context
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Page />
    </ThemeContext.Provider>
  );
}

function Button() {
  const { setTheme } = useContext(ThemeContext);
  return (
    <button onClick={() => setTheme('dark')}>
      Toggle Theme
    </button>
  );
}
```

### 4. **Component Re-rendering**
```typescript
// ❌ ไม่ดี - Re-render ไม่จำเป็น
function List({ items, onItemClick }: {
  items: Item[];
  onItemClick: (item: Item) => void;
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// ✅ ดี - ใช้ React.memo
const ListItem = React.memo(({ item, onClick }: {
  item: Item;
  onClick: (item: Item) => void;
}) => {
  return (
    <li onClick={() => onClick(item)}>
      {item.name}
    </li>
  );
});

function List({ items, onItemClick }: {
  items: Item[];
  onItemClick: (item: Item) => void;
}) {
  return (
    <ul>
      {items.map(item => (
        <ListItem
          key={item.id}
          item={item}
          onClick={onItemClick}
        />
      ))}
    </ul>
  );
}
```

### 5. **Props Mutation**
```typescript
// ❌ ไม่ดี - แก้ไข props โดยตรง
function UserList({ users }: { users: User[] }) {
  // ไม่ควรแก้ไข props โดยตรง
  users.push({ id: 'new', name: 'New User' });
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// ✅ ดี - สร้าง copy ใหม่
function UserList({ users }: { users: User[] }) {
  const usersWithNew = [...users, { id: 'new', name: 'New User' }];
  
  return (
    <ul>
      {usersWithNew.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## 🎯 สรุป

### ✅ สิ่งที่ควรรู้
- Functional components คือวิธีที่แนะนำ
- Props ใช้ส่งข้อมูลจาก parent ไป child
- Component composition ทำให้ code ใช้ซ้ำได้
- Children props ให้ความยืดหยุ่นสูง
- Conditional rendering ควบคุมการแสดงผล

### 🚀 ขั้นตอนถัดไป
1. ศึกษา **State & Lifecycle** ใน [STATE.md](STATE.md)
2. เรียนรู้ **Hooks** ใน [HOOKS.md](HOOKS.md)
3. ทดลองสร้าง reusable components

### 💡 แนวทางการฝึก
- สร้าง UI components ที่ใช้ซ้ำได้
- ฝึกการส่ง props และจัดการ children
- ทดลอง conditional rendering ในสถานการณ์ต่างๆ
- ฝึก component composition patterns

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2024*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: react@example.com*
