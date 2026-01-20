# 🚀 คู่มือการ Deploy

## 📋 ภาพรวม
เอกสารนี้อธิบายขั้นตอนการ deploy โปรเจค [Project Name] ไปยังสภาพแวดล้อมต่างๆ รวมถึงการตั้งค่า การเตรียมการ และการแก้ไขปัญหา

## 🌍 สภาพแวดล้อม

### Development Environment
- **URL**: https://dev.example.com
- **Purpose**: การพัฒนาและทดสอบ
- **Database**: Development database
- **Features**: Debug mode, Hot reload, Test data

### Staging Environment
- **URL**: https://staging.example.com
- **Purpose**: ทดสอบก่อน production
- **Database**: Staging database (production-like data)
- **Features**: Production-like settings, Performance monitoring

### Production Environment
- **URL**: https://example.com
- **Purpose**: ใช้งานจริงกับผู้ใช้
- **Database**: Production database
- **Features**: Full monitoring, Security, Backup

## 🏗️ สถาปัตยกรรม Deployment

### Container-based Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   Web Server    │    │   Database      │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │   Nginx     │ │◄──►│ │   Node.js   │ │◄──►│ │ PostgreSQL  │ │
│ │  (SSL/TLS)  │ │    │ │  (React)    │ │    │ │  (Primary)  │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     CDN         │    │     Cache       │    │   Monitoring    │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │ CloudFlare  │ │    │ │    Redis    │ │    │ │ Prometheus  │ │
│ │   (Global)  │ │    │ │ (Sessions)  │ │    │ │ (Metrics)   │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 📦 การเตรียม Build

### 1. Frontend Build
```bash
# Install dependencies
npm ci --only=production

# Run tests
npm test -- --coverage --watchAll=false

# Run linting
npm run lint

# Build for production
npm run build

# Optimize build
npm run optimize
```

### 2. Backend Build
```bash
# Install dependencies
npm ci --only=production

# Run tests
npm test -- --coverage --watchAll=false

# Run type checking
npm run type-check

# Build TypeScript
npm run build

# Create production bundle
npm run bundle
```

### 3. Docker Build
```bash
# Build frontend image
docker build -t app-frontend:latest ./frontend

# Build backend image
docker build -t app-backend:latest ./backend

# Tag images for registry
docker tag app-frontend:latest registry.example.com/app-frontend:latest
docker tag app-backend:latest registry.example.com/app-backend:latest

# Push to registry
docker push registry.example.com/app-frontend:latest
docker push registry.example.com/app-backend:latest
```

## 🔧 Environment Configuration

### 1. Environment Variables
```bash
# .env.production
NODE_ENV=production
PORT=3000

# Database
DATABASE_URL=postgresql://user:password@postgres:5432/app_db
REDIS_URL=redis://redis:6379

# API
API_BASE_URL=https://api.example.com/v1
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# External Services
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=noreply@example.com
SMTP_PASS=smtp-password

# Monitoring
SENTRY_DSN=https://your-sentry-dsn
LOG_LEVEL=info

# CDN
CDN_URL=https://cdn.example.com
```

### 2. Docker Compose Production
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  frontend:
    image: registry.example.com/app-frontend:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl/certs
    depends_on:
      - backend
    restart: unless-stopped

  backend:
    image: registry.example.com/app-backend:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backups:/backups
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl/certs
    depends_on:
      - frontend
      - backend
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

## 🚀 Deployment Process

### 1. Automated Deployment (CI/CD)
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test -- --coverage --watchAll=false
    
    - name: Build application
      run: npm run build
    
    - name: Build Docker images
      run: |
        docker build -t app-frontend:${{ github.sha }} ./frontend
        docker build -t app-backend:${{ github.sha }} ./backend
    
    - name: Push to registry
      run: |
        docker tag app-frontend:${{ github.sha }} registry.example.com/app-frontend:latest
        docker tag app-backend:${{ github.sha }} registry.example.com/app-backend:latest
        docker push registry.example.com/app-frontend:latest
        docker push registry.example.com/app-backend:latest
    
    - name: Deploy to production
      run: |
        ssh deploy@production-server "
          cd /app &&
          docker-compose -f docker-compose.prod.yml pull &&
          docker-compose -f docker-compose.prod.yml up -d &&
          docker system prune -f
        "
    
    - name: Run health checks
      run: |
        curl -f https://api.example.com/health || exit 1
        curl -f https://example.com || exit 1
```

### 2. Manual Deployment
```bash
#!/bin/bash
# deploy.sh

set -e

echo "🚀 Starting deployment..."

# 1. Backup current version
echo "📦 Creating backup..."
docker-compose -f docker-compose.prod.yml exec postgres pg_dump app_db > backups/backup_$(date +%Y%m%d_%H%M%S).sql

# 2. Pull latest images
echo "📥 Pulling latest images..."
docker-compose -f docker-compose.prod.yml pull

# 3. Stop services
echo "⏹️ Stopping services..."
docker-compose -f docker-compose.prod.yml down

# 4. Run database migrations
echo "🗄️ Running migrations..."
docker-compose -f docker-compose.prod.yml run --rm backend npm run migrate

# 5. Start services
echo "▶️ Starting services..."
docker-compose -f docker-compose.prod.yml up -d

# 6. Wait for services to be ready
echo "⏳ Waiting for services..."
sleep 30

# 7. Run health checks
echo "🏥 Running health checks..."
if curl -f http://localhost:3000/health; then
    echo "✅ Deployment successful!"
else
    echo "❌ Deployment failed!"
    exit 1
fi

# 8. Cleanup old images
echo "🧹 Cleaning up..."
docker system prune -f

echo "🎉 Deployment completed!"
```

## 🔍 Health Checks

### 1. Application Health Check
```typescript
// src/routes/health.ts
import { Router } from 'express';
import { db } from '@/database';
import { redis } from '@/cache';

const router = Router();

router.get('/health', async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    services: {
      database: 'unknown',
      redis: 'unknown',
      memory: 'unknown',
    },
  };

  try {
    // Check database
    await db.query('SELECT 1');
    health.services.database = 'healthy';
  } catch (error) {
    health.services.database = 'unhealthy';
    health.status = 'error';
  }

  try {
    // Check Redis
    await redis.ping();
    health.services.redis = 'healthy';
  } catch (error) {
    health.services.redis = 'unhealthy';
    health.status = 'error';
  }

  // Check memory usage
  const memUsage = process.memoryUsage();
  health.services.memory = memUsage.heapUsed < 512 * 1024 * 1024 ? 'healthy' : 'warning';

  const statusCode = health.status === 'ok' ? 200 : 503;
  res.status(statusCode).json(health);
});

export default router;
```

### 2. Docker Health Check
```dockerfile
# Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

### 3. Kubernetes Health Check
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-backend
spec:
  template:
    spec:
      containers:
      - name: backend
        image: app-backend:latest
        ports:
        - containerPort: 3000
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 📊 Monitoring & Logging

### 1. Application Monitoring
```typescript
// src/middleware/monitoring.ts
import prometheus from 'prom-client';

// Create metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

// Middleware
export const monitoringMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const labels = {
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode.toString(),
    };
    
    httpRequestDuration.observe(labels, duration / 1000);
    httpRequestTotal.inc(labels);
  });
  
  next();
};
```

### 2. Structured Logging
```typescript
// src/utils/logger.ts
import winston from 'winston';

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-service' },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

## 🔒 Security Considerations

### 1. SSL/TLS Configuration
```nginx
# nginx/nginx.conf
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/certs/example.com.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    
    location / {
        proxy_pass http://frontend:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /api {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 2. Firewall Rules
```bash
# UFW Configuration
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

## 🔄 Rollback Strategy

### 1. Quick Rollback
```bash
#!/bin/bash
# rollback.sh

set -e

VERSION=$1
if [ -z "$VERSION" ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

echo "🔄 Rolling back to version $VERSION..."

# 1. Pull previous version
docker pull registry.example.com/app-backend:$VERSION
docker pull registry.example.com/app-frontend:$VERSION

# 2. Update docker-compose
sed -i "s/:latest/:$VERSION/g" docker-compose.prod.yml

# 3. Restart services
docker-compose -f docker-compose.prod.yml up -d

# 4. Verify rollback
sleep 30
if curl -f http://localhost:3000/health; then
    echo "✅ Rollback successful!"
else
    echo "❌ Rollback failed!"
    exit 1
fi
```

### 2. Database Rollback
```bash
# Database migration rollback
docker-compose -f docker-compose.prod.yml exec backend npm run migrate:rollback
```

## 📋 Deployment Checklist

### Pre-deployment
- [ ] All tests passing
- [ ] Code coverage meets requirements
- [ ] Security scan completed
- [ ] Performance tests passed
- [ ] Documentation updated
- [ ] Backup current version
- [ ] Notify stakeholders

### During Deployment
- [ ] Zero downtime deployment
- [ ] Health checks passing
- [ ] Monitoring active
- [ ] Error tracking enabled
- [ ] Log collection working

### Post-deployment
- [ ] Verify functionality
- [ ] Check performance metrics
- [ ] Monitor error rates
- [ ] Verify user access
- [ ] Update documentation
- [ ] Notify team of success

## 🚨 Troubleshooting

### Common Issues

#### 1. Container Won't Start
```bash
# Check logs
docker logs container_name

# Check resource usage
docker stats

# Restart container
docker restart container_name
```

#### 2. Database Connection Issues
```bash
# Check database status
docker-compose exec postgres pg_isready

# Check connection string
docker-compose exec backend env | grep DATABASE_URL

# Test connection
docker-compose exec backend npm run db:test
```

#### 3. High Memory Usage
```bash
# Check memory usage
docker stats

# Restart services
docker-compose restart

# Scale horizontally
docker-compose up -d --scale backend=3
```

#### 4. SSL Certificate Issues
```bash
# Check certificate expiry
openssl x509 -in /etc/ssl/certs/example.com.crt -text -noout

# Renew certificate
certbot renew

# Reload nginx
docker-compose exec nginx nginx -s reload
```

## 📞 Emergency Contacts

- **DevOps Team**: devops@example.com
- **Database Admin**: dba@example.com
- **Security Team**: security@example.com
- **On-call Engineer**: +1-555-0123

---
*Deployment Guide v1.0*  
*Last updated: 2024-01-20*  
*Contact: devops@example.com*
