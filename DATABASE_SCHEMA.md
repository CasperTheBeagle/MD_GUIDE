# 🗄️ คู่มือโครงสร้างฐานข้อมูล

## 🎯 วัตถุประสงค์
เอกสารนี้อธิบายโครงสร้างฐานข้อมูลของ [Project Name] สำหรับ React applications

## 📋 สารบัญ
- [ภาพรวมฐานข้อมูล](#-ภาพรวมฐานข้อมูล)
- [โครงสร้างตาราง](#-โครงสร้างตาราง)
- [ความสัมพันธ์ระหว่างตาราง](#-ความสัมพันธ์ระหว่างตาราง)
- [ดัชนีและประสิทธิภาพ](#-ดัชนีและประสิทธิภาพ)
- [การย้ายข้อมูล](#-การย้ายข้อมูล)
- [การสำรองข้อมูล](#-การสำรองข้อมูล)

## 🏗️ ภาพรวมฐานข้อมูล

### 1. ข้อมูลทั่วไป
- **ประเภทฐานข้อมูล**: PostgreSQL 14+
- **ชื่อฐานข้อมูล**: project_name_db
- **ชุดอักขระ**: UTF-8
- **Collation**: en_US.UTF-8
- **Timezone**: Asia/Bangkok

### 2. หลักการออกแบบ
- **Normalization**: ปฏิบัติตามรูปแบบปกติที่ 3 (3NF)
- **Primary Keys**: ใช้ UUID สำหรับตารางหลัก
- **Foreign Keys**: ใช้ UUID สำหรับการอ้างอิง
- **Timestamps**: ใช้ UTC สำหรับ timestamps
- **Soft Deletes**: ใช้ deleted_at สำหรับการลบแบบ soft

### 3. การตั้งชื่อ
- **ตาราง**: ใช้ snake_case พหูพจน์ (users, posts, comments)
- **คอลัมน์**: ใช้ snake_case (user_id, created_at, updated_at)
- **ดัชนี**: ใช้ idx_ตาราง_คอลัมน์ (idx_users_email)
- **คีย์นอก**: ใช้ fk_ตาราง_คอลัมน์ (fk_posts_user_id)

## 📊 โครงสร้างตาราง

### 1. ตารางผู้ใช้ (users)
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    avatar_url VARCHAR(500),
    date_of_birth DATE,
    gender VARCHAR(10) CHECK (gender IN ('male', 'female', 'other')),
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'inactive', 'suspended', 'deleted')),
    email_verified BOOLEAN DEFAULT FALSE,
    phone_verified BOOLEAN DEFAULT FALSE,
    last_login_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

-- ดัชนี
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### 2. ตารางโปรไฟล์ (user_profiles)
```sql
CREATE TABLE user_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    bio TEXT,
    website VARCHAR(500),
    location VARCHAR(200),
    language VARCHAR(10) DEFAULT 'en',
    timezone VARCHAR(50) DEFAULT 'UTC',
    currency VARCHAR(3) DEFAULT 'USD',
    notification_email BOOLEAN DEFAULT TRUE,
    notification_push BOOLEAN DEFAULT TRUE,
    notification_sms BOOLEAN DEFAULT FALSE,
    privacy_profile BOOLEAN DEFAULT TRUE,
    privacy_email BOOLEAN DEFAULT FALSE,
    privacy_phone BOOLEAN DEFAULT FALSE,
    preferences JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ดัชนี
CREATE INDEX idx_user_profiles_user_id ON user_profiles(user_id);
CREATE INDEX idx_user_profiles_language ON user_profiles(language);
```

### 3. ตารางบทบาท (roles)
```sql
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) UNIQUE NOT NULL,
    display_name VARCHAR(100) NOT NULL,
    description TEXT,
    permissions JSONB DEFAULT '[]',
    is_system BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ข้อมูลเริ่มต้น
INSERT INTO roles (name, display_name, description, permissions, is_system) VALUES
('admin', 'Administrator', 'Full system access', '["*"]', TRUE),
('manager', 'Manager', 'Can manage users and content', '["read:*", "write:*", "delete:*"]', TRUE),
('user', 'User', 'Regular user access', '["read:own", "write:own"]', TRUE),
('guest', 'Guest', 'Limited access', '["read:public"]', TRUE);
```

### 4. ตารางบทบาทผู้ใช้ (user_roles)
```sql
CREATE TABLE user_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    assigned_by UUID REFERENCES users(id),
    assigned_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP WITH TIME ZONE,
    UNIQUE(user_id, role_id)
);

-- ดัชนี
CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_user_roles_role_id ON user_roles(role_id);
CREATE INDEX idx_user_roles_expires_at ON user_roles(expires_at);
```

### 5. ตารางโพสต์ (posts)
```sql
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    content TEXT NOT NULL,
    excerpt TEXT,
    featured_image VARCHAR(500),
    author_id UUID NOT NULL REFERENCES users(id) ON DELETE SET NULL,
    status VARCHAR(20) DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'archived', 'deleted')),
    visibility VARCHAR(20) DEFAULT 'public' CHECK (visibility IN ('public', 'private', 'unlisted')),
    comment_enabled BOOLEAN DEFAULT TRUE,
    like_count INTEGER DEFAULT 0,
    view_count INTEGER DEFAULT 0,
    share_count INTEGER DEFAULT 0,
    published_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

-- ดัชนี
CREATE INDEX idx_posts_author_id ON posts(author_id);
CREATE INDEX idx_posts_status ON posts(status);
CREATE INDEX idx_posts_visibility ON posts(visibility);
CREATE INDEX idx_posts_published_at ON posts(published_at);
CREATE INDEX idx_posts_created_at ON posts(created_at);
CREATE INDEX idx_posts_slug ON posts(slug);

-- Full-text search
CREATE INDEX idx_posts_search ON posts USING GIN(to_tsvector('english', title || ' ' || content));
```

### 6. ตารางหมวดหมู่ (categories)
```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    parent_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    icon VARCHAR(50),
    color VARCHAR(7),
    sort_order INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ดัชนี
CREATE INDEX idx_categories_parent_id ON categories(parent_id);
CREATE INDEX idx_categories_slug ON categories(slug);
CREATE INDEX idx_categories_is_active ON categories(is_active);
```

### 7. ตารางแท็ก (tags)
```sql
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    color VARCHAR(7),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ดัชนี
CREATE INDEX idx_tags_slug ON tags(slug);
```

### 8. ตารางโพสต์หมวดหมู่ (post_categories)
```sql
CREATE TABLE post_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    category_id UUID NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, category_id)
);

-- ดัชนี
CREATE INDEX idx_post_categories_post_id ON post_categories(post_id);
CREATE INDEX idx_post_categories_category_id ON post_categories(category_id);
```

### 9. ตารางโพสต์แท็ก (post_tags)
```sql
CREATE TABLE post_tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    tag_id UUID NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, tag_id)
);

-- ดัชนี
CREATE INDEX idx_post_tags_post_id ON post_tags(post_id);
CREATE INDEX idx_post_tags_tag_id ON post_tags(tag_id);
```

### 10. ตารางความคิดเห็น (comments)
```sql
CREATE TABLE comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    author_id UUID REFERENCES users(id) ON DELETE SET NULL,
    parent_id UUID REFERENCES comments(id) ON DELETE CASCADE,
    author_name VARCHAR(100),
    author_email VARCHAR(255),
    author_ip INET,
    content TEXT NOT NULL,
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'rejected', 'spam')),
    like_count INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

-- ดัชนี
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_author_id ON comments(author_id);
CREATE INDEX idx_comments_parent_id ON comments(parent_id);
CREATE INDEX idx_comments_status ON comments(status);
CREATE INDEX idx_comments_created_at ON comments(created_at);
```

### 11. ตารางไลค์ (likes)
```sql
CREATE TABLE likes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    target_type VARCHAR(50) NOT NULL CHECK (target_type IN ('post', 'comment')),
    target_id UUID NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, target_type, target_id)
);

-- ดัชนี
CREATE INDEX idx_likes_user_id ON likes(user_id);
CREATE INDEX idx_likes_target ON likes(target_type, target_id);
CREATE INDEX idx_likes_created_at ON likes(created_at);
```

### 12. ตารางการติดตาม (follows)
```sql
CREATE TABLE follows (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    follower_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    following_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(follower_id, following_id),
    CHECK(follower_id != following_id)
);

-- ดัชนี
CREATE INDEX idx_follows_follower_id ON follows(follower_id);
CREATE INDEX idx_follows_following_id ON follows(following_id);
CREATE INDEX idx_follows_created_at ON follows(created_at);
```

### 13. ตารางไฟล์ (files)
```sql
CREATE TABLE files (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    filename VARCHAR(255) NOT NULL,
    original_name VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    size_bytes BIGINT NOT NULL,
    path VARCHAR(500) NOT NULL,
    url VARCHAR(500) NOT NULL,
    uploaded_by UUID NOT NULL REFERENCES users(id) ON DELETE SET NULL,
    is_public BOOLEAN DEFAULT FALSE,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ดัชนี
CREATE INDEX idx_files_uploaded_by ON files(uploaded_by);
CREATE INDEX idx_files_mime_type ON files(mime_type);
CREATE INDEX idx_files_is_public ON files(is_public);
CREATE INDEX idx_files_created_at ON files(created_at);
```

### 14. ตารางเซสชัน (sessions)
```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL UNIQUE,
    ip_address INET,
    user_agent TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_accessed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ดัชนี
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_token_hash ON sessions(token_hash);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);
CREATE INDEX idx_sessions_is_active ON sessions(is_active);
```

### 15. ตารางบันทึกกิจกรรม (activity_logs)
```sql
CREATE TABLE activity_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    details JSONB DEFAULT '{}',
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ดัชนี
CREATE INDEX idx_activity_logs_user_id ON activity_logs(user_id);
CREATE INDEX idx_activity_logs_action ON activity_logs(action);
CREATE INDEX idx_activity_logs_resource ON activity_logs(resource_type, resource_id);
CREATE INDEX idx_activity_logs_created_at ON activity_logs(created_at);

-- Partition by month for better performance
CREATE TABLE activity_logs_y2024m01 PARTITION OF activity_logs
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

## 🔗 ความสัมพันธ์ระหว่างตาราง

### 1. แผนภาพความสัมพันธ์
```
users (1) -----> (1) user_profiles
users (1) -----> (N) user_roles
roles (1) -----> (N) user_roles
users (1) -----> (N) posts
users (1) -----> (N) comments
posts (1) -----> (N) comments
posts (N) <----> (N) categories
posts (N) <----> (N) tags
users (1) -----> (N) likes
users (1) -----> (N) follows
users (1) -----> (N) files
users (1) -----> (N) sessions
users (1) -----> (N) activity_logs
```

### 2. ความสัมพันธ์แบบ One-to-Many
- **users → posts**: ผู้ใช้สามารถมีหลายโพสต์
- **users → comments**: ผู้ใช้สามารถมีหลายความคิดเห็น
- **posts → comments**: โพสต์สามารถมีหลายความคิดเห็น
- **categories → posts**: หมวดหมู่สามารถมีหลายโพสต์

### 3. ความสัมพันธ์แบบ Many-to-Many
- **posts ↔ categories**: โพสต์สามารถอยู่ในหลายหมวดหมู่
- **posts ↔ tags**: โพสต์สามารถมีหลายแท็ก
- **users ↔ roles**: ผู้ใช้สามารถมีหลายบทบาท

### 4. ความสัมพันธ์แบบ Self-referencing
- **comments → comments**: ความคิดเห็นสามารถตอบกลับความคิดเห็นอื่น
- **categories → categories**: หมวดหมู่สามารถมีหมวดหมู่ย่อย
- **users → users**: ผู้ใช้สามารถติดตามผู้ใช้อื่น

## ⚡ ดัชนีและประสิทธิภาพ

### 1. ดัชนีหลัก
```sql
-- Primary keys (UUID)
-- สร้างอัตโนมัติเมื่อสร้างตาราง

-- Unique constraints
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_username ON users(username);
CREATE UNIQUE INDEX idx_posts_slug ON posts(slug);
CREATE UNIQUE INDEX idx_categories_slug ON categories(slug);
CREATE UNIQUE INDEX idx_tags_slug ON tags(slug);
```

### 2. ดัชนีต่างๆ
```sql
-- Foreign key indexes
CREATE INDEX idx_posts_author_id ON posts(author_id);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_author_id ON comments(author_id);
CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_user_roles_role_id ON user_roles(role_id);

-- Composite indexes
CREATE INDEX idx_posts_status_published_at ON posts(status, published_at);
CREATE INDEX idx_comments_post_id_status ON comments(post_id, status);
CREATE INDEX idx_activity_logs_user_id_created_at ON activity_logs(user_id, created_at);

-- Partial indexes
CREATE INDEX idx_active_users ON users(id) WHERE status = 'active';
CREATE INDEX idx_published_posts ON posts(id) WHERE status = 'published';
CREATE INDEX idx_approved_comments ON comments(id) WHERE status = 'approved';
```

### 3. Full-text search indexes
```sql
-- Posts search
CREATE INDEX idx_posts_search ON posts USING GIN(to_tsvector('english', title || ' ' || content));

-- Users search
CREATE INDEX idx_users_search ON users USING GIN(to_tsvector('english', first_name || ' ' || last_name || ' ' || username));
```

### 4. JSONB indexes
```sql
-- User preferences
CREATE INDEX idx_user_profiles_preferences ON user_profiles USING GIN(preferences);

-- Activity logs details
CREATE INDEX idx_activity_logs_details ON activity_logs USING GIN(details);
```

## 🔄 การย้ายข้อมูล

### 1. Migration ตัวอย่าง
```sql
-- Migration: 001_create_users_table.sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_status ON users(status);
```

### 2. Trigger สำหรับ updated_at
```sql
-- Function to update updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Apply to all tables with updated_at
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_posts_updated_at BEFORE UPDATE ON posts
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 3. Function สำหรับ Soft Delete
```sql
-- Function to soft delete
CREATE OR REPLACE FUNCTION soft_delete(table_name TEXT, record_id UUID)
RETURNS BOOLEAN AS $$
BEGIN
    EXECUTE format('UPDATE %I SET deleted_at = CURRENT_TIMESTAMP WHERE id = $1', table_name)
    USING record_id;
    RETURN FOUND;
END;
$$ LANGUAGE plpgsql;
```

## 💾 การสำรองข้อมูล

### 1. Backup Script
```bash
#!/bin/bash
# backup_database.sh

DB_NAME="project_name_db"
DB_USER="postgres"
BACKUP_DIR="/var/backups/database"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_backup_$DATE.sql"

# Create backup
pg_dump -U $DB_USER -h localhost $DB_NAME > $BACKUP_FILE

# Compress
gzip $BACKUP_FILE

# Remove old backups (keep 30 days)
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_FILE.gz"
```

### 2. Restore Script
```bash
#!/bin/bash
# restore_database.sh

BACKUP_FILE=$1
DB_NAME="project_name_db"
DB_USER="postgres"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

# Restore database
gunzip -c $BACKUP_FILE | psql -U $DB_USER -h localhost $DB_NAME

echo "Restore completed from: $BACKUP_FILE"
```

### 3. Monitoring Query
```sql
-- Database size
SELECT pg_size_pretty(pg_database_size('project_name_db'));

-- Table sizes
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

## 📊 การตรวจสอบประสิทธิภาพ

### 1. Slow Query Log
```sql
-- Enable slow query log
ALTER SYSTEM SET log_min_duration_statement = 1000; -- 1 second
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();
```

### 2. Performance Monitoring
```sql
-- Active connections
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

-- Long running queries
SELECT 
    pid,
    now() - pg_stat_activity.query_start AS duration,
    query,
    state
FROM pg_stat_activity 
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes'
ORDER BY duration DESC;

-- Table statistics
SELECT 
    schemaname,
    tablename,
    n_tup_ins,
    n_tup_upd,
    n_tup_del,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables;
```

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2024*  
*เวอร์ชัน: 1.0*  
*ติดต่อ: database@example.com*
