# 🔒 แนวทานความปลอดภัย

## 🎯 วัตถุประสงค์
เอกสารนี้ให้คำแนะนำและแนวทานในการรักษาความปลอดภัยสำหรับ React applications เพื่อป้องกันการโจมตี ปกป้องข้อมูลผู้ใช้ และให้ความเชื่อมั่นแก่ผู้ใช้งาน

## 🛡️ Security Principles

### 1. Core Principles
- **Least Privilege**: ให้สิทธิ์เท่าที่จำเป็นเท่านั้น
- **Defense in Depth**: มีหลายชั้นของการป้องกัน
- **Fail Securely**: ค่าเริ่มต้นควรปลอดภัย
- **Keep it Simple**: ความซับซ้อนเป็นศัตรูของความปลอดภัย

### 2. Threat Model
| Threat | Description | Impact | Mitigation |
|--------|-------------|--------|------------|
| **XSS** | Cross-Site Scripting | High | Input validation, CSP |
| **CSRF** | Cross-Site Request Forgery | Medium | CSRF tokens |
| **Injection** | SQL/NoSQL Injection | High | Parameterized queries |
| **Data Breach** | Unauthorized data access | High | Encryption, access control |
| **DoS** | Denial of Service | Medium | Rate limiting, caching |

## 🔐 Authentication & Authorization

### 1. JWT Implementation
```typescript
// ✅ ดี - Secure JWT implementation
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';

class AuthService {
  private readonly jwtSecret = process.env.JWT_SECRET!;
  private readonly jwtExpiresIn = '7d';

  async hashPassword(password: string): Promise<string> {
    const saltRounds = 12;
    return bcrypt.hash(password, saltRounds);
  }

  async verifyPassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }

  generateToken(payload: any): string {
    return jwt.sign(payload, this.jwtSecret, {
      expiresIn: this.jwtExpiresIn,
      issuer: 'your-app',
      audience: 'your-users',
    });
  }

  verifyToken(token: string): any {
    try {
      return jwt.verify(token, this.jwtSecret);
    } catch (error) {
      throw new Error('Invalid token');
    }
  }

  refreshToken(oldToken: string): string {
    const decoded = jwt.verify(oldToken, this.jwtSecret, { ignoreExpiration: true });
    delete decoded.exp;
    delete decoded.iat;
    return this.generateToken(decoded);
  }
}
```

### 2. Role-Based Access Control (RBAC)
```typescript
// ✅ ดี - RBAC implementation
export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user',
  GUEST = 'guest',
}

export enum Permission {
  // User permissions
  READ_OWN_PROFILE = 'read:own_profile',
  UPDATE_OWN_PROFILE = 'update:own_profile',
  
  // Admin permissions
  READ_ALL_USERS = 'read:all_users',
  UPDATE_ALL_USERS = 'update:all_users',
  DELETE_USERS = 'delete:users',
  
  // Manager permissions
  READ_TEAM_USERS = 'read:team_users',
  UPDATE_TEAM_USERS = 'update:team_users',
  
  // System permissions
  ACCESS_ADMIN_PANEL = 'access:admin_panel',
  VIEW_ANALYTICS = 'view:analytics',
  MANAGE_SYSTEM = 'manage:system',
}

export const rolePermissions: Record<UserRole, Permission[]> = {
  [UserRole.GUEST]: [],
  [UserRole.USER]: [
    Permission.READ_OWN_PROFILE,
    Permission.UPDATE_OWN_PROFILE,
  ],
  [UserRole.MANAGER]: [
    Permission.READ_OWN_PROFILE,
    Permission.UPDATE_OWN_PROFILE,
    Permission.READ_TEAM_USERS,
    Permission.UPDATE_TEAM_USERS,
    Permission.VIEW_ANALYTICS,
  ],
  [UserRole.ADMIN]: Object.values(Permission),
};

export class AuthorizationService {
  static hasPermission(userRole: UserRole, permission: Permission): boolean {
    const permissions = rolePermissions[userRole] || [];
    return permissions.includes(permission);
  }

  static canAccessResource(userRole: UserRole, resource: string, action: string): boolean {
    const permission = `${action}:${resource}` as Permission;
    return this.hasPermission(userRole, permission);
  }
}

// ✅ ดี - Authorization middleware
export const requirePermission = (permission: Permission) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;
    
    if (!user) {
      return res.status(401).json({
        success: false,
        error: { code: 'UNAUTHORIZED', message: 'Authentication required' }
      });
    }

    if (!AuthorizationService.hasPermission(user.role, permission)) {
      return res.status(403).json({
        success: false,
        error: { code: 'FORBIDDEN', message: 'Insufficient permissions' }
      });
    }

    next();
  };
};

// การใช้งาน
router.get('/admin/users', 
  authenticateToken, 
  requirePermission(Permission.READ_ALL_USERS), 
  getUsersController
);
```

### 3. Session Management
```typescript
// ✅ ดี - Secure session management
import { createClient } from 'redis';

class SessionService {
  private redis = createClient({
    url: process.env.REDIS_URL!,
  });

  constructor() {
    this.redis.connect();
  }

  async createSession(userId: string, userAgent: string): Promise<string> {
    const sessionId = crypto.randomUUID();
    const sessionData = {
      userId,
      userAgent,
      createdAt: new Date().toISOString(),
      lastAccessed: new Date().toISOString(),
    };

    // Store session with 7-day expiration
    await this.redis.setEx(`session:${sessionId}`, 7 * 24 * 60 * 60, JSON.stringify(sessionData));
    
    return sessionId;
  }

  async getSession(sessionId: string): Promise<any> {
    const sessionData = await this.redis.get(`session:${sessionId}`);
    
    if (!sessionData) {
      throw new Error('Session not found');
    }

    const session = JSON.parse(sessionData);
    
    // Update last accessed time
    session.lastAccessed = new Date().toISOString();
    await this.redis.setEx(`session:${sessionId}`, 7 * 24 * 60 * 60, JSON.stringify(session));
    
    return session;
  }

  async destroySession(sessionId: string): Promise<void> {
    await this.redis.del(`session:${sessionId}`);
  }

  async destroyAllUserSessions(userId: string): Promise<void> {
    const keys = await this.redis.keys('session:*');
    const pipeline = this.redis.multi();
    
    for (const key of keys) {
      const sessionData = await this.redis.get(key);
      if (sessionData) {
        const session = JSON.parse(sessionData);
        if (session.userId === userId) {
          pipeline.del(key);
        }
      }
    }
    
    await pipeline.exec();
  }
}
```

## 🛡️ Input Validation & Sanitization

### 1. Client-Side Validation
```typescript
// ✅ ดี - Form validation with Zod
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, 'Password must contain uppercase, lowercase, and number'),
  firstName: z.string().min(1, 'First name is required').max(50, 'First name too long'),
  lastName: z.string().min(1, 'Last name is required').max(50, 'Last name too long'),
  age: z.number().min(18, 'Must be at least 18').max(120, 'Invalid age'),
});

type UserInput = z.infer<typeof userSchema>;

const UserForm = () => {
  const [errors, setErrors] = useState<Record<string, string>>({});

  const validateForm = (data: unknown) => {
    try {
      return userSchema.parse(data);
    } catch (error) {
      if (error instanceof z.ZodError) {
        const fieldErrors: Record<string, string> = {};
        error.errors.forEach(err => {
          fieldErrors[err.path.join('.')] = err.message;
        });
        setErrors(fieldErrors);
      }
      return null;
    }
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const formData = new FormData(e.target as HTMLFormElement);
    const data = Object.fromEntries(formData.entries());
    
    const validatedData = validateForm(data);
    if (validatedData) {
      // Submit data
      submitUser(validatedData);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields with error handling */}
    </form>
  );
};
```

### 2. Server-Side Validation
```typescript
// ✅ ดี - Server-side validation middleware
import { Request, Response, NextFunction } from 'express';
import { z } from 'zod';

export const validateBody = (schema: z.ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      schema.parse(req.body);
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        return res.status(400).json({
          success: false,
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid input data',
            details: error.errors.map(err => ({
              field: err.path.join('.'),
              message: err.message,
            })),
          },
        });
      }
      next(error);
    }
  };
};

// การใช้งาน
const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  firstName: z.string().min(1),
  lastName: z.string().min(1),
});

router.post('/users', validateBody(createUserSchema), createUserController);
```

### 3. XSS Prevention
```typescript
// ✅ ดี - XSS prevention utilities
import DOMPurify from 'dompurify';

export const sanitizeHtml = (html: string): string => {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['class'],
  });
};

export const escapeHtml = (text: string): string => {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
};

// React component ที่ปลอดภัย
const SafeHtmlRenderer = ({ html }) => {
  const sanitizedHtml = useMemo(() => sanitizeHtml(html), [html]);
  
  return (
    <div 
      dangerouslySetInnerHTML={{ __html: sanitizedHtml }}
    />
  );
};

// ✅ ดี - Content Security Policy
const cspMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const csp = [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' https://cdn.trusted.com",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "font-src 'self' https://fonts.gstatic.com",
    "img-src 'self' data: https:",
    "connect-src 'self' https://api.example.com",
    "frame-ancestors 'none'",
    "base-uri 'self'",
    "form-action 'self'",
  ].join('; ');

  res.setHeader('Content-Security-Policy', csp);
  next();
};
```

## 🔒 Data Protection

### 1. Encryption
```typescript
// ✅ ดี - Encryption utilities
import crypto from 'crypto';

class EncryptionService {
  private readonly algorithm = 'aes-256-gcm';
  private readonly keyLength = 32;
  private readonly ivLength = 16;
  private readonly tagLength = 16;

  private getKey(): Buffer {
    const key = process.env.ENCRYPTION_KEY;
    if (!key || key.length !== this.keyLength * 2) {
      throw new Error('Invalid encryption key');
    }
    return Buffer.from(key, 'hex');
  }

  encrypt(text: string): { encrypted: string; iv: string; tag: string } {
    const key = this.getKey();
    const iv = crypto.randomBytes(this.ivLength);
    
    const cipher = crypto.createCipher(this.algorithm, key);
    cipher.setAAD(Buffer.from('additional-data'));
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const tag = cipher.getAuthTag();
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: tag.toString('hex'),
    };
  }

  decrypt(encrypted: string, iv: string, tag: string): string {
    const key = this.getKey();
    const ivBuffer = Buffer.from(iv, 'hex');
    const tagBuffer = Buffer.from(tag, 'hex');
    
    const decipher = crypto.createDecipher(this.algorithm, key);
    decipher.setAAD(Buffer.from('additional-data'));
    decipher.setAuthTag(tagBuffer);
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}

// การใช้งาน
const encryptionService = new EncryptionService();

// เข้ารหัสข้อมูลที่ละเอียดอ่อน
const encryptedData = encryptionService.encrypt('sensitive-user-data');

// ถอดรหัสข้อมูล
const decryptedData = encryptionService.decrypt(
  encryptedData.encrypted,
  encryptedData.iv,
  encryptedData.tag
);
```

### 2. Secure Storage
```typescript
// ✅ ดี - Secure localStorage alternative
class SecureStorage {
  private encryptionService = new EncryptionService();

  setItem(key: string, value: any): void {
    const encrypted = this.encryptionService.encrypt(JSON.stringify(value));
    localStorage.setItem(key, JSON.stringify(encrypted));
  }

  getItem<T>(key: string): T | null {
    const encryptedStr = localStorage.getItem(key);
    if (!encryptedStr) return null;

    try {
      const encrypted = JSON.parse(encryptedStr);
      const decrypted = this.encryptionService.decrypt(
        encrypted.encrypted,
        encrypted.iv,
        encrypted.tag
      );
      return JSON.parse(decrypted);
    } catch {
      return null;
    }
  }

  removeItem(key: string): void {
    localStorage.removeItem(key);
  }

  clear(): void {
    localStorage.clear();
  }
}

// การใช้งาน
const secureStorage = new SecureStorage();

// เก็บข้อมูลที่ละเอียดอ่อน
secureStorage.setItem('user-token', authToken);
secureStorage.setItem('user-preferences', userPreferences);

// ดึงข้อมูล
const token = secureStorage.getItem<string>('user-token');
const preferences = secureStorage.getItem<UserPreferences>('user-preferences');
```

### 3. Data Masking
```typescript
// ✅ ดี - Data masking for logs
export const maskSensitiveData = (data: any): any => {
  const sensitiveFields = ['password', 'token', 'ssn', 'creditCard', 'email'];
  
  if (typeof data !== 'object' || data === null) {
    return data;
  }

  const masked = { ...data };

  for (const field of sensitiveFields) {
    if (field in masked) {
      if (field === 'email') {
        masked[field] = maskEmail(masked[field]);
      } else {
        masked[field] = '***MASKED***';
      }
    }
  }

  return masked;
};

export const maskEmail = (email: string): string => {
  const [username, domain] = email.split('@');
  if (username.length <= 2) {
    return `${username[0]}***@${domain}`;
  }
  return `${username.slice(0, 2)}***@${domain}`;
};

// การใช้งานกับ logger
const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format(maskSensitiveData)(),
    winston.format.json()
  ),
});

// จะไม่ log ข้อมูลที่ละเอียดอ่อน
logger.info('User login attempt', {
  email: 'user@example.com',
  password: 'secret123', // จะถูก mask
  ip: '192.168.1.1',
});
```

## 🌐 Network Security

### 1. HTTPS Configuration
```typescript
// ✅ ดี - HTTPS enforcement middleware
const enforceHttps = (req: Request, res: Response, next: NextFunction) => {
  if (!req.secure && req.get('x-forwarded-proto') !== 'https') {
    const httpsUrl = `https://${req.get('host')}${req.url}`;
    return res.redirect(301, httpsUrl);
  }
  next();
};

// ✅ ดี - Security headers middleware
const securityHeaders = (req: Request, res: Response, next: NextFunction) => {
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // Prevent MIME type sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Enable XSS protection
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // Force HTTPS
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  
  // Referrer policy
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // Permissions policy
  res.setHeader('Permissions-Policy', 
    'geolocation=(), microphone=(), camera=(), payment=(), usb=()'
  );
  
  next();
};
```

### 2. CSRF Protection
```typescript
// ✅ ดี - CSRF token implementation
import csrf from 'csurf';

const csrfProtection = csrf({
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
  },
});

// Get CSRF token
router.get('/api/csrf-token', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Protected routes
router.post('/api/sensitive-action', 
  csrfProtection, 
  authenticateToken, 
  sensitiveActionController
);

// Frontend implementation
const useCSRFToken = () => {
  const [csrfToken, setCsrfToken] = useState<string>('');

  useEffect(() => {
    const fetchToken = async () => {
      try {
        const response = await fetch('/api/csrf-token');
        const { csrfToken } = await response.json();
        setCsrfToken(csrfToken);
      } catch (error) {
        console.error('Failed to fetch CSRF token:', error);
      }
    };

    fetchToken();
  }, []);

  return csrfToken;
};

// การใช้งานใน form
const SecureForm = () => {
  const csrfToken = useCSRFToken();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const formData = new FormData(e.target as HTMLFormElement);
    formData.append('csrfToken', csrfToken);

    const response = await fetch('/api/sensitive-action', {
      method: 'POST',
      body: formData,
      headers: {
        'X-CSRF-Token': csrfToken,
      },
    });

    // Handle response
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="hidden" name="csrfToken" value={csrfToken} />
      {/* Form fields */}
    </form>
  );
};
```

### 3. Rate Limiting
```typescript
// ✅ ดี - Rate limiting implementation
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

// General rate limiting
const generalLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args: string[]) => redisClient.sendCommand(args),
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: {
    success: false,
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Too many requests, please try again later.',
    },
  },
  standardHeaders: true,
  legacyHeaders: false,
});

// Strict rate limiting for sensitive endpoints
const strictLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args: string[]) => redisClient.sendCommand(args),
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Limit each IP to 5 requests per windowMs
  message: {
    success: false,
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Too many attempts, please try again later.',
    },
  },
});

// Apply to routes
app.use('/api/', generalLimiter);
app.post('/api/auth/login', strictLimiter);
app.post('/api/auth/register', strictLimiter);
app.post('/api/users/forgot-password', strictLimiter);
```

## 🔍 Security Monitoring

### 1. Security Event Logging
```typescript
// ✅ ดี - Security event logger
class SecurityLogger {
  private logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
      winston.format.timestamp(),
      winston.format.json()
    ),
    transports: [
      new winston.transports.File({ filename: 'logs/security.log' }),
    ],
  });

  logLoginAttempt(email: string, ip: string, success: boolean, userAgent?: string) {
    this.logger.info('Login attempt', {
      event: 'LOGIN_ATTEMPT',
      email: maskEmail(email),
      ip,
      success,
      userAgent,
      timestamp: new Date().toISOString(),
    });
  }

  logPasswordChange(userId: string, ip: string) {
    this.logger.warn('Password changed', {
      event: 'PASSWORD_CHANGE',
      userId,
      ip,
      timestamp: new Date().toISOString(),
    });
  }

  logSuspiciousActivity(description: string, ip: string, userId?: string) {
    this.logger.error('Suspicious activity detected', {
      event: 'SUSPICIOUS_ACTIVITY',
      description,
      ip,
      userId,
      timestamp: new Date().toISOString(),
    });
  }

  logDataAccess(userId: string, resource: string, action: string, ip: string) {
    this.logger.info('Data access', {
      event: 'DATA_ACCESS',
      userId,
      resource,
      action,
      ip,
      timestamp: new Date().toISOString(),
    });
  }
}

// การใช้งาน
const securityLogger = new SecurityLogger();

// In authentication middleware
if (loginSuccess) {
  securityLogger.logLoginAttempt(email, ip, true, userAgent);
} else {
  securityLogger.logLoginAttempt(email, ip, false, userAgent);
}
```

### 2. Intrusion Detection
```typescript
// ✅ ดี - Basic intrusion detection
class IntrusionDetector {
  private failedAttempts = new Map<string, number>();
  private readonly maxAttempts = 5;
  private readonly lockoutTime = 15 * 60 * 1000; // 15 minutes

  recordFailedAttempt(ip: string): boolean {
    const attempts = this.failedAttempts.get(ip) || 0;
    this.failedAttempts.set(ip, attempts + 1);

    if (attempts + 1 >= this.maxAttempts) {
      this.lockoutIP(ip);
      return true; // Should lockout
    }

    return false;
  }

  private lockoutIP(ip: string): void {
    // Log the lockout
    securityLogger.logSuspiciousActivity(`IP locked out due to too many failed attempts`, ip);
    
    // Set timeout to clear lockout
    setTimeout(() => {
      this.failedAttempts.delete(ip);
    }, this.lockoutTime);
  }

  isIPLocked(ip: string): boolean {
    return this.failedAttempts.get(ip) >= this.maxAttempts;
  }

  clearFailedAttempts(ip: string): void {
    this.failedAttempts.delete(ip);
  }
}

// การใช้งานใน auth middleware
const intrusionDetector = new IntrusionDetector();

const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const ip = req.ip;
  
  if (intrusionDetector.isIPLocked(ip)) {
    return res.status(429).json({
      success: false,
      error: {
        code: 'IP_LOCKED',
        message: 'Too many failed attempts. Please try again later.',
      },
    });
  }

  // Continue with authentication...
};
```

## 📋 Security Checklist

### Authentication & Authorization
- [ ] Password hashing with bcrypt/scrypt/Argon2
- [ ] JWT tokens with proper expiration
- [ ] Role-based access control implemented
- [ ] Session management with Redis
- [ ] Multi-factor authentication for sensitive accounts

### Input Validation
- [ ] Server-side validation for all inputs
- [ ] XSS prevention with output encoding
- [ ] SQL injection prevention with parameterized queries
- [ ] File upload validation and scanning
- [ ] CSRF protection for state-changing operations

### Data Protection
- [ ] Encryption for sensitive data at rest
- [ ] HTTPS for all communications
- [ ] Secure storage of secrets and keys
- [ ] Data masking in logs
- [ ] Regular data backup and recovery testing

### Network Security
- [ ] Security headers configured
- [ ] Rate limiting implemented
- [ ] IP whitelisting for admin areas
- [ ] DDoS protection
- [ ] Regular security scanning

### Monitoring & Auditing
- [ ] Security event logging
- [ ] Intrusion detection system
- [ ] Regular security audits
- [ ] Vulnerability scanning
- [ ] Security incident response plan

---
*Security Guidelines v1.0*  
*Last updated: 2024-01-20*  
*Contact: security-team@example.com*
