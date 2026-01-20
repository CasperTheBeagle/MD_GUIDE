# 📏 มาตรฐานการเขียนโค้ด

## 🎯 วัตถุประสงค์
- รักษาความสม่ำเสมอของโค้ดทั่วทั้งโปรเจค
- ทำให้การ review โค้ดง่ายขึ้น
- ลดการเกิด bugs จากความผิดพลาดที่ซ้ำซ้อน
- เพิ่มความสามารถในการบำรุงรักษา

## 🔧 การตั้งค่า Tools

### ESLint Configuration
```json
{
  "extends": [
    "@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "react/prop-types": "off"
  }
}
```

### Prettier Configuration
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

## 📝 การตั้งชื่อ (Naming Conventions)

### Files & Directories
- **Components**: `PascalCase.tsx` (เช่น `UserProfile.tsx`)
- **Utilities**: `camelCase.ts` (เช่น `formatDate.ts`)
- **Constants**: `UPPER_SNAKE_CASE.ts` (เช่น `API_ENDPOINTS.ts`)
- **Directories**: `kebab-case` (เช่น `user-profile/`)

### Variables & Functions
```typescript
// ✅ ดี
const userName = 'john';
const isLoggedIn = true;
const getUserData = () => {};

// ❌ ไม่ดี
const user_name = 'john';
const flag = true;
const getData = () => {};
```

### Components
```typescript
// ✅ ดี
const UserProfileCard: React.FC<UserProfileProps> = ({ user }) => {
  return <div>{user.name}</div>;
};

// ❌ ไม่ดี
const card = ({ user }) => {
  return <div>{user.name}</div>;
};
```

### Constants
```typescript
// ✅ ดี
const API_BASE_URL = 'https://api.example.com';
const MAX_FILE_SIZE = 5 * 1024 * 1024;

// ❌ ไม่ดี
const apiUrl = 'https://api.example.com';
const maxSize = 5 * 1024 * 1024;
```

## 🏗️ โครงสร้างโค้ด

### Component Structure
```typescript
import React, { useState, useEffect } from 'react';
import { Button } from '@/components/ui/Button';
import { User } from '@/types/user';

interface UserProfileProps {
  userId: string;
  onUpdate?: (user: User) => void;
}

export const UserProfile: React.FC<UserProfileProps> = ({ 
  userId, 
  onUpdate 
}) => {
  // 1. States
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // 2. Effects
  useEffect(() => {
    fetchUser();
  }, [userId]);

  // 3. Functions
  const fetchUser = async () => {
    try {
      setLoading(true);
      // fetch logic
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  // 4. Handlers
  const handleUpdate = (updatedUser: User) => {
    setUser(updatedUser);
    onUpdate?.(updatedUser);
  };

  // 5. Conditional renders
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  // 6. Main render
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <Button onClick={() => handleUpdate(user)}>
        Update Profile
      </Button>
    </div>
  );
};
```

## 📋 การจัดรูปแบบ (Formatting)

### Indentation & Spacing
- ใช้ 2 spaces สำหรับ indentation
- ไม่ใช้ tabs
- มี space หลัง comma และ colon
- ไม่มี space หลัง (

```typescript
// ✅ ดี
const getUser = (id: string): Promise<User> => {
  return api.get(`/users/${id}`);
};

// ❌ ไม่ดี
const getUser = (id:string):Promise<User>=>{
  return api.get(`/users/${id}`);
};
```

### Line Length
- จำกัดความยาวไม่เกิน 80 characters
- แบ่งบรรทัดถ้าเกิน limit

```typescript
// ✅ ดี
const longFunctionName = (
  param1: string,
  param2: number,
  param3: boolean
): ReturnType => {
  // function body
};

// ❌ ไม่ดี
const longFunctionName = (param1: string, param2: number, param3: boolean): ReturnType => {
  // function body
};
```

## 🎨 Best Practices

### 1. การใช้ TypeScript
```typescript
// ✅ ดี - มี type definitions
interface UserData {
  id: string;
  name: string;
  email: string;
}

const getUser = async (id: string): Promise<UserData> => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

// ❌ ไม่ดี - ไม่มี types
const getUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};
```

### 2. การจัดการ Error
```typescript
// ✅ ดี - มี error handling
const fetchData = async () => {
  try {
    const response = await api.get('/data');
    return response.data;
  } catch (error) {
    console.error('Failed to fetch data:', error);
    throw new Error('Data fetch failed');
  }
};

// ❌ ไม่ดี - ไม่มี error handling
const fetchData = async () => {
  const response = await api.get('/data');
  return response.data;
};
```

### 3. การใช้ Constants
```typescript
// ✅ ดี - ใช้ constants
const API_ENDPOINTS = {
  USERS: '/api/users',
  POSTS: '/api/posts',
} as const;

const fetchUsers = () => api.get(API_ENDPOINTS.USERS);

// ❌ ไม่ดี - hardcode strings
const fetchUsers = () => api.get('/api/users');
```

### 4. การจัดการ State
```typescript
// ✅ ดี - ใช้ proper state management
const [users, setUsers] = useState<User[]>([]);
const [loading, setLoading] = useState(false);

// ❌ ไม่ดี - ใช้ any
const [users, setUsers] = useState<any[]>([]);
```

## 🔄 Git Conventions

### Branch Naming
- `feature/feature-name` - สำหรับฟีเจอร์ใหม่
- `bugfix/bug-description` - สำหรับแก้บัก
- `hotfix/urgent-fix` - สำหรับแก้ไขด่วน
- `refactor/code-improvement` - สำหรับ refactor

### Commit Messages
```
type(scope): description

feat(auth): add user login functionality
fix(api): resolve timeout issue
docs(readme): update installation guide
```

## 📝 Code Review Guidelines

### สิ่งที่ต้องตรวจสอบ
- [ ] Code ทำงานตามที่คาดหวัง
- [ ] มี error handling ครบถ้วน
- [ ] มี tests ครบถ้วน
- [ ] ไม่มี console.log ที่ไม่จำเป็น
- [ ] มี TypeScript types ครบถ้วน
- [ ] ทำตาม coding standards

### การให้ feedback
- เน้นที่โค้ด ไม่ใช่คน
- ให้เหตุผลประกอบ
- เสนอทางแก้ไข
- ยอมรับความคิดเห็นของผู้อื่น

---
*อัปเดตล่าสุด: [วันที่]*
*เจ้าของเอกสาร: [ชื่อ]*
