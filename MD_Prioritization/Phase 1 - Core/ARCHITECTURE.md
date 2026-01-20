# 🏗️ สถาปัตยกรรมระบบ [Project Name]

## 📋 ภาพรวม
เอกสารนี้อธิบายสถาปัตยกรรมของระบบ [Project Name] รวมถึงโครงสร้าง คอมโพเนนต์ และการเชื่อมต่อระหว่างส่วนต่างๆ

## 🎯 วัตถุประสงค์
- จัดเตรียมภาพรวมของสถาปัตยกรรมระบบ
- แนะนำโครงสร้างและ patterns ที่ใช้
- แนะนำเทคโนโลยีและ tools
- แนะนำ best practices สำหรับการพัฒนา

## 🏢 สถาปัตยกรรมระบบโดยรวม

### แผนภาพสถาปัตยกรรม
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

### แผนภาพ Data Flow
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

### แผนภาพ Component Hierarchy
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

### แผนภาพ State Management
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

### แผนภาพ API Architecture
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

## 🎨 Frontend Architecture

### 1. โครงสร้างโฟลเดอร์
```
src/
├── components/           # Reusable components
│   ├── ui/              # Basic UI components
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── index.ts
│   ├── forms/           # Form components
│   ├── layout/          # Layout components
│   └── features/        # Feature-specific components
├── pages/               # Page components
│   ├── Home/
│   ├── Dashboard/
│   └── Profile/
├── hooks/               # Custom hooks
│   ├── useAuth.ts
│   ├── useApi.ts
│   └── useLocalStorage.ts
├── store/               # State management
│   ├── slices/
│   ├── middleware/
│   └── index.ts
├── services/            # API services
│   ├── api.ts
│   ├── auth.ts
│   └── users.ts
├── utils/               # Utility functions
│   ├── helpers.ts
│   ├── constants.ts
│   └── validators.ts
├── types/               # TypeScript types
│   ├── api.ts
│   ├── user.ts
│   └── common.ts
├── styles/              # Global styles
│   ├── globals.css
│   ├── variables.css
│   └── components.css
└── assets/              # Static assets
    ├── images/
    ├── icons/
    └── fonts/
```

### 2. Component Architecture
```typescript
// Component Structure Example
interface ComponentProps {
  // Props definitions
}

export const Component: React.FC<ComponentProps> = ({ 
  prop1, 
  prop2 
}) => {
  // 1. Hooks
  const [state, setState] = useState();
  const { data, loading } = useCustomHook();
  
  // 2. Event handlers
  const handleClick = useCallback(() => {
    // Handle logic
  }, []);
  
  // 3. Effects
  useEffect(() => {
    // Side effects
  }, []);
  
  // 4. Conditional renders
  if (loading) return <LoadingSpinner />;
  
  // 5. Main render
  return (
    <div className="component">
      {/* JSX content */}
    </div>
  );
};
```

### 3. State Management
```typescript
// Redux Toolkit Example
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async () => {
    const response = await usersService.getUsers();
    return response.data;
  }
);

// Slice
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    loading: false,
    error: null,
  },
  reducers: {
    // Sync reducers
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});
```

## 🔧 Backend Architecture

### 1. โครงสร้างโฟลเดอร์
```
src/
├── controllers/         # Route controllers
│   ├── authController.ts
│   ├── userController.ts
│   └── index.ts
├── services/           # Business logic
│   ├── authService.ts
│   ├── userService.ts
│   └── index.ts
├── models/             # Data models
│   ├── User.ts
│   ├── Product.ts
│   └── index.ts
├── repositories/       # Data access
│   ├── userRepository.ts
│   ├── productRepository.ts
│   └── index.ts
├── middleware/         # Custom middleware
│   ├── auth.ts
│   ├── validation.ts
│   └── errorHandler.ts
├── routes/             # API routes
│   ├── auth.ts
│   ├── users.ts
│   └── index.ts
├── utils/              # Utility functions
│   ├── logger.ts
│   ├── helpers.ts
│   └── constants.ts
├── config/             # Configuration
│   ├── database.ts
│   ├── redis.ts
│   └── app.ts
└── types/              # TypeScript types
    ├── express.d.ts
    ├── user.ts
    └── common.ts
```

### 2. API Design
```typescript
// RESTful API Structure
GET    /api/users           # Get all users
GET    /api/users/:id       # Get user by ID
POST   /api/users           # Create new user
PUT    /api/users/:id       # Update user
DELETE /api/users/:id       # Delete user

// Response Format
{
  "success": true,
  "data": {
    // Response data
  },
  "message": "Success message",
  "timestamp": "2024-01-20T10:00:00Z"
}

// Error Response Format
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Error description",
    "details": {}
  },
  "timestamp": "2024-01-20T10:00:00Z"
}
```

### 3. Service Layer Pattern
```typescript
// Service Example
export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  async createUser(userData: CreateUserDto): Promise<User> {
    // 1. Validate input
    const validatedData = await this.validateUserData(userData);
    
    // 2. Check if user exists
    const existingUser = await this.userRepository.findByEmail(
      validatedData.email
    );
    if (existingUser) {
      throw new ConflictError('User already exists');
    }
    
    // 3. Create user
    const user = await this.userRepository.create(validatedData);
    
    // 4. Send welcome email
    await this.emailService.sendWelcomeEmail(user.email);
    
    // 5. Return user without password
    const { password, ...userWithoutPassword } = user;
    return userWithoutPassword;
  }

  private async validateUserData(userData: CreateUserDto) {
    // Validation logic
    return userData;
  }
}
```

## 🗄️ Database Architecture

### 1. Database Schema
```sql
-- Users Table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  role VARCHAR(50) DEFAULT 'user',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products Table
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  price DECIMAL(10,2) NOT NULL,
  category_id UUID REFERENCES categories(id),
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2. Data Access Layer
```typescript
// Repository Pattern
export interface IUserRepository {
  create(userData: CreateUserDto): Promise<User>;
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  update(id: string, userData: UpdateUserDto): Promise<User>;
  delete(id: string): Promise<void>;
  findAll(filters: UserFilters): Promise<User[]>;
}

export class UserRepository implements IUserRepository {
  constructor(private db: Database) {}

  async create(userData: CreateUserDto): Promise<User> {
    const query = `
      INSERT INTO users (email, password_hash, first_name, last_name)
      VALUES ($1, $2, $3, $4)
      RETURNING *
    `;
    
    const values = [
      userData.email,
      userData.passwordHash,
      userData.firstName,
      userData.lastName
    ];
    
    const result = await this.db.query(query, values);
    return this.mapToUser(result.rows[0]);
  }

  private mapToUser(row: any): User {
    return {
      id: row.id,
      email: row.email,
      firstName: row.first_name,
      lastName: row.last_name,
      role: row.role,
      isActive: row.is_active,
      createdAt: row.created_at,
      updatedAt: row.updated_at,
    };
  }
}
```

## 🔐 Security Architecture

### 1. Authentication Flow
```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Client  │    │ Backend │    │ Database│    │  Redis  │
│         │    │         │    │         │    │         │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
     │               │               │               │
     │ 1. Login      │               │               │
     ├──────────────►│               │               │
     │               │ 2. Validate   │               │
     │               ├──────────────►│               │
     │               │               │ 3. User Data  │
     │               │◄──────────────│               │
     │               │ 4. Generate   │               │
     │               │    JWT        │               │
     │               │ 5. Store      │               │
     │               ├──────────────►│               │
     │ 6. JWT Token  │               │               │
     │◄──────────────│               │               │
```

### 2. Authorization
```typescript
// Role-based Access Control (RBAC)
export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user',
}

export const permissions = {
  [UserRole.ADMIN]: [
    'users:read',
    'users:write',
    'users:delete',
    'system:config',
  ],
  [UserRole.MANAGER]: [
    'users:read',
    'users:write',
    'reports:read',
  ],
  [UserRole.USER]: [
    'profile:read',
    'profile:write',
  ],
};

// Middleware for authorization
export const requirePermission = (permission: string) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;
    const userPermissions = permissions[user.role] || [];
    
    if (!userPermissions.includes(permission)) {
      return res.status(403).json({
        success: false,
        error: {
          code: 'INSUFFICIENT_PERMISSIONS',
          message: 'You do not have permission to perform this action',
        },
      });
    }
    
    next();
  };
};
```

## 🚀 Deployment Architecture

### 1. Container Architecture
```dockerfile
# Frontend Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```dockerfile
# Backend Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

### 2. Infrastructure
```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend
  
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
  
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=app_db
      - POSTGRES_USER=app_user
      - POSTGRES_PASSWORD=app_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## 📊 Monitoring & Logging

### 1. Application Monitoring
```typescript
// Metrics Collection
import { createPrometheusMetrics } from './monitoring';

export const metrics = createPrometheusMetrics({
  httpRequestDuration: 'histogram',
  httpRequestTotal: 'counter',
  activeConnections: 'gauge',
  databaseConnections: 'gauge',
});

// Middleware for metrics
export const metricsMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    metrics.httpRequestDuration.observe(duration, {
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode,
    });
    
    metrics.httpRequestTotal.inc({
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode,
    });
  });
  
  next();
};
```

### 2. Logging Strategy
```typescript
// Structured Logging
import winston from 'winston';

export const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-service' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

## 🔄 Design Patterns

### 1. Repository Pattern
- แยก business logic จาก data access
- ทำให้ง่ายต่อการทดสอบ
- รองรับการเปลี่ยน database ได้ง่าย

### 2. Service Layer Pattern
- จัดการ business logic
- ทำ transaction management
- จัดการการเรียกใช้ repositories

### 3. Factory Pattern
- สร้าง objects ตามเงื่อนไข
- ลดการพึ่งพา concrete classes
- ทำให้ง่ายต่อการขยาย

### 4. Observer Pattern
- จัดการ events และ notifications
- ลoose coupling ระหว่าง components
- รองรับ real-time updates

## 📈 Performance Considerations

### 1. Caching Strategy
- **Redis**: สำหรับ session data และ frequently accessed data
- **Browser Cache**: สำหรับ static assets
- **CDN**: สำหรับ global content delivery

### 2. Database Optimization
- **Indexing**: สร้าง indexes สำหรับ queries ที่ใช้บ่อย
- **Connection Pooling**: จัดการ database connections อย่างมีประสิทธิภาพ
- **Query Optimization**: ใช้ EXPLAIN ANALYZE สำหรับ query tuning

### 3. Frontend Optimization
- **Code Splitting**: แบ่งโค้ดตาม routes
- **Lazy Loading**: โหลด components เมื่อต้องการ
- **Bundle Optimization**: ลดขนาด bundle ด้วย tree shaking

---
*เอกสารนี้จัดทำโดยทีมสถาปัตยกรรม*  
*อัปเดตล่าสุด: [วันที่]*
