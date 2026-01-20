# 🏗️ คู่ม่าโครงสร้างพื้นฐาน

## 🎯 วัตถุประสงค์
เอกสารนี้อธิบายโครงสร้างพื้นฐานของ [Project Name] สำหรับ React applications

## 📋 สารบัญ
- [ภาพรวมสถาปัตยกรรม](#-ภาพรวมสถาปัตยกรรม)
- [โครงสร้างเครือข่าย](#-โครงสร้างเครือข่าย)
- [เซิร์ฟเวอร์และคอมพิวเตอร์](#-เซิร์ฟเวอร์และคอมพิวเตอร์)
- [ฐานข้อมูล](#-ฐานข้อมูล)
- [การจัดเก็บข้อมูล](#-การจัดเก็บข้อมูล)
- [การรักษาความปลอดภัย](#-การรักษาความปลอดภัย)
- [การติดตามและตรวจสอบ](#-การติดตามและตรวจสอบ)

## 🏛️ ภาพรวมสถาปัตยกรรม

### 1. สถาปัตยกรรมระบบ
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   Web Servers   │    │   Database      │
│   (Nginx/HAProxy)│────│   (Node.js)     │────│   (PostgreSQL)  │
│   Port: 80/443  │    │   Port: 3000    │    │   Port: 5432    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   CDN           │    │   Cache         │    │   Storage       │
│   (CloudFlare)  │    │   (Redis)       │    │   (S3)          │
│   Port: 80/443  │    │   Port: 6379    │    │   Port: 443     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 2. สภาพแวดล้อม
- **Production**: สภาพแวดล้อมจริงสำหรับผู้ใช้งาน
- **Staging**: สภาพแวดล้อมทดสอบก่อนเปิดใช้งานจริง
- **Development**: สภาพแวดล้อมพัฒนาสำหรับทีมพัฒนา
- **Testing**: สภาพแวดล้อมทดสอบอัตโนมัติ

### 3. หลักการออกแบบ
- **High Availability**: ระบบพร้อมใช้งานสูง
- **Scalability**: สามารถขยายได้
- **Security**: ความปลอดภัยสูง
- **Performance**: ประสิทธิภาพสูง
- **Monitoring**: ติดตามและตรวจสอบได้

## 🌐 โครงสร้างเครือข่าย

### 1. Network Topology
```
Internet
    │
    ▼
┌─────────────────┐
│   Firewall      │
│   (iptables)    │
│   Port: 80/443  │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│   Load Balancer │
│   (Nginx)       │
│   Port: 80/443  │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│   Web Servers   │
│   (Node.js)     │
│   Port: 3000    │
└─────────────────┘
```

### 2. Load Balancer Configuration
```nginx
# /etc/nginx/nginx.conf
upstream backend {
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
    server 10.0.1.12:3000;
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3. Firewall Rules
```bash
#!/bin/bash
# firewall.sh

# Clear existing rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow database connections from web servers
iptables -A INPUT -s 10.0.1.0/24 -p tcp --dport 5432 -j ACCEPT

# Allow Redis connections from web servers
iptables -A INPUT -s 10.0.1.0/24 -p tcp --dport 6379 -j ACCEPT

# Save rules
iptables-save > /etc/iptables/rules.v4
```

## 🖥️ เซิร์ฟเวอร์และคอมพิวเตอร์

### 1. Server Specifications

**Web Servers (3 servers)**
- **CPU**: 4 vCPU
- **Memory**: 8GB RAM
- **Storage**: 100GB SSD
- **OS**: Ubuntu 22.04 LTS
- **Network**: 1Gbps

**Database Server (1 server)**
- **CPU**: 8 vCPU
- **Memory**: 16GB RAM
- **Storage**: 500GB SSD
- **OS**: Ubuntu 22.04 LTS
- **Network**: 1Gbps

**Cache Server (1 server)**
- **CPU**: 2 vCPU
- **Memory**: 8GB RAM
- **Storage**: 50GB SSD
- **OS**: Ubuntu 22.04 LTS
- **Network**: 1Gbps

### 2. Server Configuration
```bash
#!/bin/bash
# server_setup.sh

# Update system
apt update && apt upgrade -y

# Install required packages
apt install -y nginx nodejs npm postgresql redis-server

# Configure Nginx
systemctl enable nginx
systemctl start nginx

# Configure Node.js
npm install -g pm2
pm2 startup

# Configure PostgreSQL
systemctl enable postgresql
systemctl start postgresql

# Configure Redis
systemctl enable redis-server
systemctl start redis-server

# Configure firewall
ufw enable
ufw allow ssh
ufw allow 80
ufw allow 443
```

### 3. Application Deployment
```bash
#!/bin/bash
# deploy.sh

# Pull latest code
git pull origin main

# Install dependencies
npm install

# Build application
npm run build

# Restart application
pm2 restart app

# Check status
pm2 status
```

### 4. Process Management
```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'app',
    script: 'dist/server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: '/var/log/app/error.log',
    out_file: '/var/log/app/out.log',
    log_file: '/var/log/app/combined.log',
    time: true
  }]
};
```

## 🗄️ ฐานข้อมูล

### 1. Database Configuration
```sql
-- postgresql.conf
listen_addresses = 'localhost'
port = 5432
max_connections = 200
shared_buffers = 256MB
effective_cache_size = 4GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
```

### 2. Replication Setup
```bash
#!/bin/bash
# replication_setup.sh

# Master configuration
sudo -u postgres psql -c "CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'password';"
sudo -u postgres psql -c "CREATE PUBLICATION mypub FOR ALL TABLES;"

# Slave configuration
sudo -u postgres psql -c "CREATE SUBSCRIPTION mysub CONNECTION 'host=master_ip user=replicator password=password dbname=project_name_db' PUBLICATION mypub;"
```

### 3. Backup Strategy
```bash
#!/bin/bash
# backup_database.sh

DB_NAME="project_name_db"
BACKUP_DIR="/var/backups/database"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup
pg_dump -h localhost -U postgres $DB_NAME | gzip > $BACKUP_DIR/$DB_NAME_$DATE.sql.gz

# Upload to S3
aws s3 cp $BACKUP_DIR/$DB_NAME_$DATE.sql.gz s3://backups/database/

# Clean old backups
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete
```

## 💾 การจัดเก็บข้อมูล

### 1. S3 Configuration
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

### 2. File Upload Configuration
```javascript
// upload.js
const AWS = require('aws-sdk');
const s3 = new AWS.S3({
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  region: 'us-east-1'
});

const uploadFile = async (file) => {
  const params = {
    Bucket: 'example-bucket',
    Key: `uploads/${Date.now()}-${file.originalname}`,
    Body: file.buffer,
    ContentType: file.mimetype
  };

  return s3.upload(params).promise();
};
```

### 3. CDN Configuration
```javascript
// cdn.js
const Cloudflare = require('cloudflare');
const cf = new Cloudflare({
  email: process.env.CLOUDFLARE_EMAIL,
  key: process.env.CLOUDFLARE_KEY
});

const purgeCache = async (urls) => {
  return cf.zones.purge_cache('zone_id', {
    files: urls
  });
};
```

## 🔒 การรักษาความปลอดภัย

### 1. SSL/TLS Configuration
```nginx
# ssl.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

### 2. Security Headers
```nginx
# security.conf
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
```

### 3. Rate Limiting
```nginx
# rate_limit.conf
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
    }
    
    location /api/auth/login {
        limit_req zone=login burst=5 nodelay;
        proxy_pass http://backend;
    }
}
```

### 4. Intrusion Detection
```bash
#!/bin/bash
# intrusion_detection.sh

# Monitor failed login attempts
FAILED_LOGINS=$(journalctl -u ssh | grep "Failed password" | wc -l)

if [ $FAILED_LOGINS -gt 10 ]; then
    echo "High number of failed login attempts detected: $FAILED_LOGINS"
    # Send alert
    curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"High number of failed login attempts detected"}' \
        https://hooks.slack.com/services/xxx/yyy/zzz
fi
```

## 📊 การติดตามและตรวจสอบ

### 1. Monitoring Setup
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nodejs'
    static_configs:
      - targets: ['localhost:3000']
    
  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9113']
    
  - job_name: 'postgres'
    static_configs:
      - targets: ['localhost:9187']
    
  - job_name: 'redis'
    static_configs:
      - targets: ['localhost:9121']
```

### 2. Grafana Dashboard
```json
{
  "dashboard": {
    "title": "Application Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      }
    ]
  }
}
```

### 3. Alerting Rules
```yaml
# alerts.yml
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} errors per second"
      
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
          description: "95th percentile response time is {{ $value }} seconds"
```

### 4. Log Management
```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/*.log
    fields:
      service: nginx
    fields_under_root: true

  - type: log
    enabled: true
    paths:
      - /var/log/app/*.log
    fields:
      service: app
    fields_under_root: true

output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "logs-%{+yyyy.MM.dd}"
```

## 🚀 Deployment Pipeline

### 1. CI/CD Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
      - run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to server
        uses: appleboy/ssh-action@v0.1.4
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /var/www/app
            git pull origin main
            npm install
            npm run build
            pm2 restart app
```

### 2. Docker Configuration
```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

### 3. Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
    depends_on:
      - db
      - redis

  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=dbname
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

## 📈 Performance Optimization

### 1. Caching Strategy
```javascript
// cache.js
const redis = require('redis');
const client = redis.createClient({
  host: 'localhost',
  port: 6379
});

const cache = {
  get: async (key) => {
    const value = await client.get(key);
    return value ? JSON.parse(value) : null;
  },
  
  set: async (key, value, ttl = 3600) => {
    await client.setex(key, ttl, JSON.stringify(value));
  },
  
  del: async (key) => {
    await client.del(key);
  }
};
```

### 2. Database Optimization
```sql
-- Create indexes
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
CREATE INDEX CONCURRENTLY idx_posts_author_id ON posts(author_id);

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Update statistics
ANALYZE users;
ANALYZE posts;
```

### 3. Application Optimization
```javascript
// performance.js
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  require('./server.js');
}
```

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2026*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: infrastructure@example.com*
