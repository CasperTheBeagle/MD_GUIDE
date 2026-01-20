# 📚 API Documentation

## 📋 ภาพรวม
เอกสารนี้อธิบาย API ของโปรเจค [Project Name] รวมถึง endpoints, request/response formats, authentication, และการใช้งาน

## 🔗 Base URL
```
Development: https://api-dev.example.com/v1
Production:  https://api.example.com/v1
```

## 🔐 Authentication

### JWT Token Authentication
ทุก request ที่ต้องการ authentication ต้องมี JWT token ใน header:

```http
Authorization: Bearer <jwt_token>
```

### การขอ Token
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "user_id",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```

## 📊 Response Format

### Success Response
```json
{
  "success": true,
  "data": {
    // Response data
  },
  "message": "Success message",
  "timestamp": "2024-01-20T10:00:00Z"
}
```

### Error Response
```json
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

### HTTP Status Codes
| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Validation Error |
| 500 | Internal Server Error |

## 👥 Users API

### Get All Users
```http
GET /api/users
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | number | No | Page number (default: 1) |
| limit | number | No | Items per page (default: 20) |
| search | string | No | Search term |
| role | string | No | Filter by role |

**Response:**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "user_id",
        "email": "user@example.com",
        "name": "John Doe",
        "role": "user",
        "isActive": true,
        "createdAt": "2024-01-20T10:00:00Z",
        "updatedAt": "2024-01-20T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 100,
      "totalPages": 5
    }
  }
}
```

### Get User by ID
```http
GET /api/users/{id}
```

**Path Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| id | string | User ID |

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "user_id",
    "email": "user@example.com",
    "name": "John Doe",
    "role": "user",
    "isActive": true,
    "profile": {
      "firstName": "John",
      "lastName": "Doe",
      "avatar": "https://example.com/avatar.jpg",
      "bio": "Software developer"
    },
    "createdAt": "2024-01-20T10:00:00Z",
    "updatedAt": "2024-01-20T10:00:00Z"
  }
}
```

### Create User
```http
POST /api/users
Content-Type: application/json
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "email": "newuser@example.com",
  "name": "New User",
  "password": "password123",
  "role": "user",
  "profile": {
    "firstName": "New",
    "lastName": "User",
    "bio": "New user bio"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "new_user_id",
    "email": "newuser@example.com",
    "name": "New User",
    "role": "user",
    "isActive": true,
    "createdAt": "2024-01-20T10:00:00Z"
  },
  "message": "User created successfully"
}
```

### Update User
```http
PUT /api/users/{id}
Content-Type: application/json
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "name": "Updated Name",
  "role": "admin",
  "profile": {
    "firstName": "Updated",
    "lastName": "Name",
    "bio": "Updated bio"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "user_id",
    "email": "user@example.com",
    "name": "Updated Name",
    "role": "admin",
    "isActive": true,
    "updatedAt": "2024-01-20T11:00:00Z"
  },
  "message": "User updated successfully"
}
```

### Delete User
```http
DELETE /api/users/{id}
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "message": "User deleted successfully"
}
```

## 📝 Posts API

### Get All Posts
```http
GET /api/posts
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | number | No | Page number |
| limit | number | No | Items per page |
| category | string | No | Filter by category |
| author | string | No | Filter by author ID |
| status | string | No | Filter by status (published, draft) |

**Response:**
```json
{
  "success": true,
  "data": {
    "posts": [
      {
        "id": "post_id",
        "title": "Sample Post",
        "content": "Post content...",
        "excerpt": "Post excerpt",
        "category": "Technology",
        "status": "published",
        "author": {
          "id": "author_id",
          "name": "Author Name",
          "avatar": "https://example.com/avatar.jpg"
        },
        "tags": ["react", "javascript"],
        "publishedAt": "2024-01-20T10:00:00Z",
        "createdAt": "2024-01-20T09:00:00Z",
        "updatedAt": "2024-01-20T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 50,
      "totalPages": 3
    }
  }
}
```

### Get Post by ID
```http
GET /api/posts/{id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "post_id",
    "title": "Sample Post",
    "content": "Full post content with markdown...",
    "excerpt": "Post excerpt",
    "category": "Technology",
    "status": "published",
    "author": {
      "id": "author_id",
      "name": "Author Name",
      "email": "author@example.com",
      "avatar": "https://example.com/avatar.jpg",
      "bio": "Author bio"
    },
    "tags": ["react", "javascript"],
    "metadata": {
      "readTime": 5,
      "views": 150,
      "likes": 25
    },
    "publishedAt": "2024-01-20T10:00:00Z",
    "createdAt": "2024-01-20T09:00:00Z",
    "updatedAt": "2024-01-20T10:00:00Z"
  }
}
```

### Create Post
```http
POST /api/posts
Content-Type: application/json
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "title": "New Post Title",
  "content": "Post content in markdown...",
  "excerpt": "Post excerpt",
  "category": "Technology",
  "tags": ["react", "javascript"],
  "status": "published"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "new_post_id",
    "title": "New Post Title",
    "content": "Post content...",
    "excerpt": "Post excerpt",
    "category": "Technology",
    "status": "published",
    "author": {
      "id": "author_id",
      "name": "Author Name"
    },
    "tags": ["react", "javascript"],
    "publishedAt": "2024-01-20T10:00:00Z",
    "createdAt": "2024-01-20T10:00:00Z"
  },
  "message": "Post created successfully"
}
```

## 🔍 Search API

### Global Search
```http
GET /api/search?q={query}
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| q | string | Yes | Search query |
| type | string | No | Search type (posts, users, all) |
| page | number | No | Page number |
| limit | number | No | Items per page |

**Response:**
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "type": "post",
        "id": "post_id",
        "title": "Post Title",
        "excerpt": "Matching excerpt...",
        "url": "/posts/post_id"
      },
      {
        "type": "user",
        "id": "user_id",
        "name": "User Name",
        "bio": "Matching bio...",
        "url": "/users/user_id"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 15,
      "totalPages": 1
    }
  }
}
```

## 📁 File Upload API

### Upload File
```http
POST /api/upload
Content-Type: multipart/form-data
Authorization: Bearer <token>
```

**Request Body:**
```
file: [binary file data]
type: "avatar" | "post-image" | "document"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "url": "https://cdn.example.com/uploads/image.jpg",
    "filename": "image.jpg",
    "size": 1024000,
    "mimeType": "image/jpeg"
  },
  "message": "File uploaded successfully"
}
```

## 📊 Analytics API

### Get Analytics Data
```http
GET /api/analytics
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| startDate | string | No | Start date (YYYY-MM-DD) |
| endDate | string | No | End date (YYYY-MM-DD) |
| metric | string | No | Specific metric (views, likes, comments) |

**Response:**
```json
{
  "success": true,
  "data": {
    "overview": {
      "totalViews": 10000,
      "totalUsers": 500,
      "totalPosts": 100,
      "totalLikes": 2500
    },
    "dailyStats": [
      {
        "date": "2024-01-20",
        "views": 500,
        "users": 25,
        "posts": 5,
        "likes": 125
      }
    ],
    "topPosts": [
      {
        "id": "post_id",
        "title": "Popular Post",
        "views": 1000,
        "likes": 50
      }
    ]
  }
}
```

## ⚠️ Error Codes

### Authentication Errors
| Code | Description |
|------|-------------|
| AUTH_001 | Invalid credentials |
| AUTH_002 | Token expired |
| AUTH_003 | Invalid token |
| AUTH_004 | Insufficient permissions |

### Validation Errors
| Code | Description |
|------|-------------|
| VALID_001 | Required field missing |
| VALID_002 | Invalid email format |
| VALID_003 | Password too short |
| VALID_004 | Invalid file type |

### Resource Errors
| Code | Description |
|------|-------------|
| RES_001 | Resource not found |
| RES_002 | Resource already exists |
| RES_003 | Resource access denied |
| RES_004 | Resource in use |

### System Errors
| Code | Description |
|------|-------------|
| SYS_001 | Internal server error |
| SYS_002 | Database error |
| SYS_003 | External service error |
| SYS_004 | Rate limit exceeded |

## 🔄 Rate Limiting

### Limits
- **Anonymous requests**: 100 requests/hour
- **Authenticated requests**: 1000 requests/hour
- **Upload requests**: 10 requests/hour

### Headers
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642694400
```

### Rate Limit Exceeded Response
```json
{
  "success": false,
  "error": {
    "code": "SYS_004",
    "message": "Rate limit exceeded",
    "details": {
      "limit": 1000,
      "resetTime": "2024-01-20T11:00:00Z"
    }
  }
}
```

## 🧪 Testing API

### Using curl
```bash
# Get users
curl -H "Authorization: Bearer <token>" \
     https://api.example.com/v1/users

# Create post
curl -X POST \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <token>" \
     -d '{"title":"Test Post","content":"Test content"}' \
     https://api.example.com/v1/posts
```

### Using Postman
1. Import Postman collection: `[link-to-collection]`
2. Set environment variables:
   - `base_url`: https://api.example.com/v1
   - `token`: Your JWT token
3. Use pre-request script for auto-refresh token

## 📝 SDK Examples

### JavaScript/TypeScript
```typescript
import { ApiClient } from '@/services/api';

const api = new ApiClient({
  baseURL: 'https://api.example.com/v1',
  token: 'your-jwt-token'
});

// Get users
const users = await api.getUsers();

// Create post
const post = await api.createPost({
  title: 'New Post',
  content: 'Post content'
});
```

### Python
```python
import requests

headers = {
    'Authorization': 'Bearer <token>',
    'Content-Type': 'application/json'
}

# Get users
response = requests.get('https://api.example.com/v1/users', headers=headers)
users = response.json()

# Create post
post_data = {
    'title': 'New Post',
    'content': 'Post content'
}
response = requests.post('https://api.example.com/v1/posts', 
                        json=post_data, headers=headers)
post = response.json()
```

## 📋 Changelog

### v1.2.0 (2024-01-20)
- ✨ Added analytics endpoint
- 🐛 Fixed user pagination bug
- 🔒 Enhanced security validation

### v1.1.0 (2024-01-15)
- ✨ Added file upload endpoint
- ✨ Added global search functionality
- 📝 Improved error messages

### v1.0.0 (2024-01-01)
- 🎉 Initial API release
- ✨ User management endpoints
- ✨ Post management endpoints
- 🔐 JWT authentication

---
*API Documentation v1.2.0*  
*Last updated: 2024-01-20*  
*Contact: api-support@example.com*
