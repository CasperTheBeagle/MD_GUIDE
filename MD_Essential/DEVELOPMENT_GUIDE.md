# 📖 คู่มือการพัฒนา

## 🚀 การเริ่มต้น

### Prerequisites
- Node.js 18+ 
- npm 8+ หรือ yarn 1.22+
- Git 2.30+
- VS Code (แนะนำ)

### การติดตั้ง
```bash
# 1. Clone repository
git clone [repository-url]
cd [project-name]

# 2. Install dependencies
npm install

# 3. Setup environment
cp .env.example .env
# แก้ไข .env ตามความเหมาะสม

# 4. ติดตั้ง pre-commit hooks
npm run prepare

# 5. เริ่ม development server
npm run dev
```

## 🛠️ Development Workflow

### 1. สร้าง Branch
```bash
# สร้าง branch สำหรับ feature ใหม่
git checkout -b feature/user-profile

# สร้าง branch สำหรับแก้ bug
git checkout -b bugfix/login-issue
```

### 2. การพัฒนา
```bash
# เริ่ม development server
npm run dev

# รัน tests ใน background
npm run test:watch

# ตรวจสอบ linting
npm run lint

# จัดรูปแบบโค้ด
npm run format
```

### 3. การทดสอบ
```bash
# รัน tests ทั้งหมด
npm test

# รัน tests พร้อม coverage
npm run test:coverage

# รัน E2E tests
npm run test:e2e
```

### 4. การ Commit
```bash
# ตรวจสอบ linting และ tests
npm run pre-commit

# Commit โค้ด
git add .
git commit -m "feat(auth): add user login functionality"

# Push ไปยัง remote
git push origin feature/user-profile
```

### 5. การ Pull Request
1. สร้าง Pull Request บน GitHub/GitLab
2. รอ code review จากทีม
3. แก้ไขตาม feedback
4. Merge หลังได้การอนุมัติ

## 📁 โครงสร้างโปรเจค

```
project-root/
├── public/                 # Static assets
├── src/
│   ├── components/        # Reusable components
│   │   ├── ui/           # UI components
│   │   ├── forms/        # Form components
│   │   └── layout/       # Layout components
│   ├── pages/            # Page components
│   ├── hooks/            # Custom hooks
│   ├── utils/            # Utility functions
│   ├── types/            # TypeScript types
│   ├── services/         # API services
│   ├── store/            # State management
│   ├── styles/           # Global styles
│   └── assets/           # Images, icons
├── tests/                # Test files
├── docs/                 # Documentation
├── .env.example          # Environment template
├── package.json          # Dependencies
├── tsconfig.json         # TypeScript config
├── vite.config.ts        # Vite config
└── README.md             # Project info
```

## 🎯 การพัฒนา Components

### 1. สร้าง Component
```typescript
// src/components/ui/Button.tsx
import React from 'react';
import { cn } from '@/utils/cn';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
}

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  children,
  onClick,
}) => {
  return (
    <button
      className={cn(
        'btn',
        `btn-${variant}`,
        `btn-${size}`
      )}
      onClick={onClick}
    >
      {children}
    </button>
  );
};
```

### 2. การทดสอบ Component
```typescript
// tests/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/ui/Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

## 🔧 การจัดการ State

### 1. Local State
```typescript
const [count, setCount] = useState(0);
const [user, setUser] = useState<User | null>(null);
```

### 2. Global State (Zustand)
```typescript
// src/store/userStore.ts
import { create } from 'zustand';
import { User } from '@/types/user';

interface UserStore {
  user: User | null;
  setUser: (user: User) => void;
  clearUser: () => void;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null }),
}));
```

### 3. Server State (React Query)
```typescript
// src/hooks/useUsers.ts
import { useQuery, useMutation } from '@tanstack/react-query';
import { userService } from '@/services/userService';

export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: userService.getUsers,
  });
};

export const useCreateUser = () => {
  return useMutation({
    mutationFn: userService.createUser,
    onSuccess: () => {
      // Invalidate cache
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

## 🌐 การทำงานกับ APIs

### 1. API Service
```typescript
// src/services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  timeout: 10000,
});

// Request interceptor
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

### 2. การใช้งาน
```typescript
// src/services/userService.ts
import api from './api';
import { User } from '@/types/user';

export const userService = {
  getUsers: (): Promise<User[]> => 
    api.get('/users').then(res => res.data),
    
  getUserById: (id: string): Promise<User> => 
    api.get(`/users/${id}`).then(res => res.data),
    
  createUser: (user: Omit<User, 'id'>): Promise<User> => 
    api.post('/users', user).then(res => res.data),
};
```

## 🎨 การจัดการ Styles

### 1. Tailwind CSS
```typescript
// ใช้ utility classes
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow-md">
  <h2 className="text-lg font-semibold text-gray-900">Title</h2>
  <button className="px-4 py-2 text-white bg-blue-500 rounded hover:bg-blue-600">
    Action
  </button>
</div>
```

### 2. CSS Modules
```typescript
// src/components/Card.module.css
.card {
  @apply bg-white rounded-lg shadow-md p-4;
}

.title {
  @apply text-lg font-semibold text-gray-900;
}

// src/components/Card.tsx
import styles from './Card.module.css';

export const Card = ({ title, children }) => (
  <div className={styles.card}>
    <h2 className={styles.title}>{title}</h2>
    {children}
  </div>
);
```

## 🐛 Debugging

### 1. การใช้ DevTools
- React DevTools - ตรวจสอบ components และ state
- Redux DevTools - ตรวจสอบ state changes
- Network Tab - ตรวจสอบ API calls

### 2. Console Logging
```typescript
// ✅ ดี - มี context
console.log('User data:', userData);
console.error('API error:', error);

// ❌ ไม่ดี - ไม่มี context
console.log(data);
console.log(error);
```

### 3. Error Boundaries
```typescript
// src/components/ErrorBoundary.tsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

## 📋 การตรวจสอบคุณภาพ

### Pre-commit Checklist
- [ ] Code ผ่าน linting
- [ ] Tests ผ่านทั้งหมด
- [ ] ไม่มี console.log ที่ไม่จำเป็น
- [ ] มี TypeScript types ครบถ้วน
- [ ] Components มี tests

### Performance Checklist
- [ ] ไม่มี memory leaks
- [ ] ใช้ React.memo สำหรับ expensive components
- [ ] ใช้ useMemo/useCallback สำหรับ expensive computations
- [ ] ไม่มี unnecessary re-renders

## 🚀 การ Deploy

### 1. Build Process
```bash
# Build for production
npm run build

# Preview build
npm run preview
```

### 2. Environment Variables
```bash
# .env.production
REACT_APP_API_URL=https://api.production.com
REACT_APP_ENV=production
```

### 3. Deployment Steps
1. รัน build process
2. อัปโหลดไฟล์ build
3. ตั้งค่า environment variables
4. รัน health checks
5. แจ้งทีมว่า deploy เสร็จสิ้น

---
*อัปเดตล่าสุด: [วันที่]*
*เจ้าของเอกสาร: [ชื่อ]*
