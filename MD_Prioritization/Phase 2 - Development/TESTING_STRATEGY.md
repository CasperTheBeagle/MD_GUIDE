# 🧪 กลยุทธ์การทดสอบ

## 🎯 วัตถุประสงค์
- รับประกันคุณภาพของซอฟต์แวร์
- ลด bugs และปัญหาใน production
- เพิ่มความมั่นใจในการเปลี่ยนแปลงโค้ด
- ทำให้การบำรุงรักษาง่ายขึ้น

## 📊 ประเภทการทดสอบ

### 1. Unit Tests
- **วัตถุประสงค์**: ทดสอบฟังก์ชัน/คอมโพเนนต์แยกกัน
- **Coverage เป้าหมาย**: 80%+
- **Tools**: Jest, React Testing Library
- **ระยะเวลา**: น้อยกว่า 5 นาที

### 2. Integration Tests
- **วัตถุประสงค์**: ทดสอบการทำงานร่วมกันของ components
- **Coverage เป้าหมาย**: 60%+
- **Tools**: Jest, Supertest
- **ระยะเวลา**: 5-15 นาที

### 3. End-to-End Tests
- **วัตถุประสงค์**: ทดสอบ user flows ทั้งหมด
- **Coverage เป้าหมาย**: Critical paths 100%
- **Tools**: Cypress, Playwright
- **ระยะเวลา**: 15-30 นาที

### 4. Performance Tests
- **วัตถุประสงค์**: ทดสอบประสิทธิภาพและ load
- **Metrics**: Response time, throughput, memory usage
- **Tools**: Lighthouse, Artillery
- **ระยะเวลา**: 30-60 นาที

## 🏗️ โครงสร้างการทดสอบ

```
tests/
├── unit/                 # Unit tests
│   ├── components/      # Component tests
│   ├── hooks/           # Hook tests
│   ├── utils/           # Utility function tests
│   └── services/        # Service tests
├── integration/         # Integration tests
│   ├── api/            # API integration tests
│   └── components/     # Component integration tests
├── e2e/                # End-to-end tests
│   ├── auth/           # Authentication flows
│   ├── dashboard/      # Dashboard flows
│   └── user-flows/     # User journey tests
├── performance/        # Performance tests
│   ├── load/           # Load testing
│   └── lighthouse/    # Lighthouse audits
├── fixtures/           # Test data
│   ├── users.json      # User test data
│   └── products.json   # Product test data
└── helpers/            # Test utilities
    ├── setup.ts        # Test setup
    ├── mocks.ts        # Mock functions
    └── utils.ts        # Test utilities
```

## 📝 Unit Testing

### 1. Component Testing
```typescript
// tests/unit/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/ui/Button';

describe('Button Component', () => {
  const defaultProps = {
    children: 'Click me',
    onClick: jest.fn(),
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders button with correct text', () => {
    render(<Button {...defaultProps} />);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    render(<Button {...defaultProps} />);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(defaultProps.onClick).toHaveBeenCalledTimes(1);
  });

  it('applies correct variant classes', () => {
    render(<Button {...defaultProps} variant="secondary" />);
    const button = screen.getByText('Click me');
    expect(button).toHaveClass('btn-secondary');
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button {...defaultProps} disabled />);
    const button = screen.getByText('Click me');
    expect(button).toBeDisabled();
  });

  it('shows loading state', () => {
    render(<Button {...defaultProps} loading />);
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });
});
```

### 2. Hook Testing
```typescript
// tests/unit/hooks/useAuth.test.ts
import { renderHook, act } from '@testing-library/react';
import { useAuth } from '@/hooks/useAuth';
import { authService } from '@/services/authService';

jest.mock('@/services/authService');

describe('useAuth Hook', () => {
  const mockAuthService = authService as jest.Mocked<typeof authService>;

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('initializes with correct default state', () => {
    const { result } = renderHook(() => useAuth());
    
    expect(result.current.user).toBeNull();
    expect(result.current.loading).toBe(false);
    expect(result.current.error).toBeNull();
  });

  it('logs in user successfully', async () => {
    const mockUser = { id: '1', name: 'John Doe', email: 'john@example.com' };
    mockAuthService.login.mockResolvedValue(mockUser);

    const { result } = renderHook(() => useAuth());

    await act(async () => {
      await result.current.login('john@example.com', 'password');
    });

    expect(result.current.user).toEqual(mockUser);
    expect(result.current.loading).toBe(false);
    expect(result.current.error).toBeNull();
  });

  it('handles login error', async () => {
    mockAuthService.login.mockRejectedValue(new Error('Invalid credentials'));

    const { result } = renderHook(() => useAuth());

    await act(async () => {
      await result.current.login('john@example.com', 'wrong-password');
    });

    expect(result.current.user).toBeNull();
    expect(result.current.loading).toBe(false);
    expect(result.current.error).toBe('Invalid credentials');
  });
});
```

### 3. Service Testing
```typescript
// tests/unit/services/userService.test.ts
import { userService } from '@/services/userService';
import { api } from '@/services/api';

jest.mock('@/services/api');

const mockApi = api as jest.Mocked<typeof api>;

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getUsers', () => {
    it('fetches users successfully', async () => {
      const mockUsers = [
        { id: '1', name: 'John', email: 'john@example.com' },
        { id: '2', name: 'Jane', email: 'jane@example.com' },
      ];
      
      mockApi.get.mockResolvedValue({ data: mockUsers });

      const result = await userService.getUsers();

      expect(mockApi.get).toHaveBeenCalledWith('/users');
      expect(result).toEqual(mockUsers);
    });

    it('handles API error', async () => {
      mockApi.get.mockRejectedValue(new Error('Network error'));

      await expect(userService.getUsers()).rejects.toThrow('Network error');
    });
  });

  describe('createUser', () => {
    it('creates user successfully', async () => {
      const newUser = { name: 'John', email: 'john@example.com' };
      const createdUser = { id: '1', ...newUser };
      
      mockApi.post.mockResolvedValue({ data: createdUser });

      const result = await userService.createUser(newUser);

      expect(mockApi.post).toHaveBeenCalledWith('/users', newUser);
      expect(result).toEqual(createdUser);
    });
  });
});
```

## 🔗 Integration Testing

### 1. API Integration
```typescript
// tests/integration/api/auth.test.ts
import request from 'supertest';
import { app } from '@/app';
import { db } from '@/database';

describe('Auth API Integration', () => {
  beforeAll(async () => {
    await db.connect();
  });

  afterAll(async () => {
    await db.disconnect();
  });

  beforeEach(async () => {
    await db.clear();
  });

  describe('POST /api/auth/login', () => {
    it('logs in user with valid credentials', async () => {
      // Create test user
      await db.collection('users').insertOne({
        email: 'test@example.com',
        passwordHash: 'hashedpassword',
        name: 'Test User',
      });

      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password',
        });

      expect(response.status).toBe(200);
      expect(response.body.success).toBe(true);
      expect(response.body.data.token).toBeDefined();
    });

    it('rejects invalid credentials', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword',
        });

      expect(response.status).toBe(401);
      expect(response.body.success).toBe(false);
    });
  });
});
```

### 2. Component Integration
```typescript
// tests/integration/components/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { UserProfile } from '@/components/UserProfile';
import { userService } from '@/services/userService';

jest.mock('@/services/userService');

const mockUserService = userService as jest.Mocked<typeof userService>;

const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});

describe('UserProfile Integration', () => {
  let queryClient: QueryClient;

  beforeEach(() => {
    queryClient = createTestQueryClient();
    jest.clearAllMocks();
  });

  const renderWithQueryClient = (component: React.ReactElement) => {
    return render(
      <QueryClientProvider client={queryClient}>
        {component}
      </QueryClientProvider>
    );
  };

  it('displays user data after loading', async () => {
    const mockUser = {
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
    };

    mockUserService.getUserById.mockResolvedValue(mockUser);

    renderWithQueryClient(<UserProfile userId="1" />);

    expect(screen.getByText('Loading...')).toBeInTheDocument();

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('john@example.com')).toBeInTheDocument();
    });
  });

  it('displays error message when API fails', async () => {
    mockUserService.getUserById.mockRejectedValue(new Error('User not found'));

    renderWithQueryClient(<UserProfile userId="999" />);

    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
});
```

## 🌐 End-to-End Testing

### 1. Authentication Flow
```typescript
// tests/e2e/auth/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('should login successfully with valid credentials', async ({ page }) => {
    // Fill login form
    await page.fill('[data-testid=email-input]', 'test@example.com');
    await page.fill('[data-testid=password-input]', 'password123');
    await page.click('[data-testid=login-button]');

    // Should redirect to dashboard
    await expect(page).toHaveURL('/dashboard');
    
    // Should show user menu
    await expect(page.locator('[data-testid=user-menu]')).toBeVisible();
  });

  test('should show error with invalid credentials', async ({ page }) => {
    await page.fill('[data-testid=email-input]', 'test@example.com');
    await page.fill('[data-testid=password-input]', 'wrongpassword');
    await page.click('[data-testid=login-button]');

    // Should stay on login page
    await expect(page).toHaveURL('/login');
    
    // Should show error message
    await expect(page.locator('[data-testid=error-message]')).toBeVisible();
    await expect(page.locator('[data-testid=error-message]')).toContainText('Invalid credentials');
  });

  test('should logout successfully', async ({ page }) => {
    // Login first
    await page.fill('[data-testid=email-input]', 'test@example.com');
    await page.fill('[data-testid=password-input]', 'password123');
    await page.click('[data-testid=login-button]');
    await page.waitForURL('/dashboard');

    // Logout
    await page.click('[data-testid=user-menu]');
    await page.click('[data-testid=logout-button]');

    // Should redirect to login
    await expect(page).toHaveURL('/login');
  });
});
```

### 2. User Journey
```typescript
// tests/e2e/user-journeys/create-post.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Create Post Journey', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[data-testid=email-input]', 'author@example.com');
    await page.fill('[data-testid=password-input]', 'password123');
    await page.click('[data-testid=login-button]');
    await page.waitForURL('/dashboard');
  });

  test('should create and view new post', async ({ page }) => {
    // Navigate to create post
    await page.click('[data-testid=create-post-button]');
    await expect(page).toHaveURL('/posts/create');

    // Fill post form
    await page.fill('[data-testid=post-title]', 'My Test Post');
    await page.fill('[data-testid=post-content]', 'This is the content of my test post.');
    await page.selectOption('[data-testid=post-category]', 'Technology');
    
    // Submit form
    await page.click('[data-testid=submit-button]');

    // Should redirect to post detail
    await expect(page.locator('h1')).toContainText('My Test Post');
    await expect(page.locator('.post-content')).toContainText('This is the content of my test post');
    await expect(page.locator('.post-category')).toContainText('Technology');

    // Verify post appears in dashboard
    await page.goto('/dashboard');
    await expect(page.locator('[data-testid=post-list]')).toContainText('My Test Post');
  });

  test('should show validation errors for empty fields', async ({ page }) => {
    await page.goto('/posts/create');
    
    // Submit empty form
    await page.click('[data-testid=submit-button]');

    // Should show validation errors
    await expect(page.locator('[data-testid=title-error]')).toBeVisible();
    await expect(page.locator('[data-testid=content-error]')).toBeVisible();
  });
});
```

## ⚡ Performance Testing

### 1. Lighthouse Audit
```typescript
// tests/performance/lighthouse/dashboard.spec.ts
import { test } from '@playwright/test';
import { playAudit } from 'playwright-lighthouse';

test.describe('Performance Tests', () => {
  test('should meet performance thresholds', async ({ page }) => {
    await page.goto('/dashboard');
    
    await playAudit({
      thresholds: {
        performance: 80,
        accessibility: 90,
        'best-practices': 80,
        seo: 80,
      },
      port: 9222,
    });
  });

  test('should load quickly', async ({ page }) => {
    const startTime = Date.now();
    await page.goto('/dashboard');
    await page.waitForLoadState('networkidle');
    const loadTime = Date.now() - startTime;
    
    // Should load in less than 3 seconds
    expect(loadTime).toBeLessThan(3000);
  });
});
```

### 2. Load Testing
```typescript
// tests/performance/load/api-load.test.js
import { check, sleep } from 'k6';
import http from 'k6/http';

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up to 100 users
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 0 },   // Ramp down
  ],
};

export default function () {
  let response = http.get('https://api.example.com/users');
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });
  
  sleep(1);
}
```

## 📊 Test Coverage

### 1. Coverage Configuration
```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{ts,tsx}',
    '!src/**/index.ts',
  ],
};
```

### 2. Coverage Scripts
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --coverage --ci --watchAll=false",
    "test:e2e": "playwright test",
    "test:e2e:headed": "playwright test --headed",
    "test:performance": "playwright test tests/performance"
  }
}
```

## 🔄 CI/CD Integration

### 1. GitHub Actions
```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linting
      run: npm run lint
    
    - name: Run type checking
      run: npm run type-check
    
    - name: Run unit tests
      run: npm run test:ci
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
    
    - name: Run E2E tests
      run: npm run test:e2e
    
    - name: Run performance tests
      run: npm run test:performance
```

## 📋 Best Practices

### 1. Test Organization
- **Arrange-Act-Assert**: จัดระเบียบ tests ให้ชัดเจน
- **Descriptive names**: ใช้ชื่อที่บอกว่าทดสอบอะไร
- **One assertion per test**: ทดสอบสิ่งเดียวในแต่ละ test
- **Test data isolation**: ใช้ test data แยกกัน

### 2. Mocking Strategy
- **Mock external dependencies**: API, database, third-party services
- **Use consistent mocks**: สร้าง mock utilities ที่ใช้ซ้ำได้
- **Avoid over-mocking**: ทดสอบ logic จริง ไม่ใช่ mocks

### 3. Test Data Management
- **Factory pattern**: สร้าง test data อย่างเป็นระบบ
- **Fixtures**: ใช้ fixtures สำหรับข้อมูลที่ใช้บ่อย
- **Cleanup**: ล้างข้อมูลหลังการทดสอบ

### 4. Performance Considerations
- **Parallel execution**: รัน tests แบบขนาน
- **Selective testing**: รันเฉพาะ tests ที่เกี่ยวข้อง
- **Test caching**: ใช้ cache สำหรับ dependencies

---
*เอกสารนี้จัดทำโดย QA Team*  
*อัปเดตล่าสุด: [วันที่]*
