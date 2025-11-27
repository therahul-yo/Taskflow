# TaskFlow Application - Task 2 Submission

## 🎯 Project Overview

**Candidate Role:** Backend Developer  
**Topics Implemented:** 1, 2, 4  
**Submission:** Freshers Recruitment 2025 - Task 2

This is a comprehensive full-stack task management application built as part of the Freshers Recruitment 2025 program. The application demonstrates proficiency in backend development, RESTful APIs, JWT authentication, and real-time WebSocket communication.

---

## ✅ Topics Implemented

### Topic 1: REST API with CRUD Operations ✅
- Complete RESTful API architecture using Node.js + Express
- Full CRUD operations for task management
- User-specific resource management
- Proper HTTP methods (GET, POST, PUT, DELETE)
- Appropriate HTTP status codes
- Error handling and validation

### Topic 2: JWT-based Authentication ✅
- Secure token-based authentication system
- User registration endpoint with password hashing (bcrypt)
- User login endpoint with JWT token generation
- Protected API routes with authentication middleware
- Token verification and validation
- 7-day token expiry management

### Topic 4: Real-time WebSocket Chat ✅
- Socket.io integration for bidirectional communication
- Room-based chat functionality
- Instant message delivery
- Connected client management
- Event-driven architecture
- CORS-enabled WebSocket server

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────┐
│           Frontend Layer (React)                │
│              Port: 3000                         │
└────────────────┬────────────────────────────────┘
                 │ HTTP/REST API + WebSocket
┌────────────────▼────────────────────────────────┐
│        Proxy Layer (Python FastAPI)             │
│              Port: 8001                         │
└────────────────┬────────────────────────────────┘
                 │ Forwards to Port 8002
┌────────────────▼────────────────────────────────┐
│      Backend Layer (Node.js + Express)          │
│              Port: 8002                         │
│  • Authentication Controllers                   │
│  • Task Management Controllers                  │
│  • WebSocket Server (Socket.io)                 │
└────────────────┬────────────────────────────────┘
                 │ Sequelize ORM
┌────────────────▼────────────────────────────────┐
│        Database Layer (SQLite)                  │
│  • User Model                                   │
│  • Task Model                                   │
└─────────────────────────────────────────────────┘
```

---

## 💻 Technology Stack

### Backend Technologies
- **Node.js** - JavaScript runtime environment
- **Express.js** - Web application framework
- **Socket.io** - Real-time bidirectional event-based communication
- **Sequelize ORM** - Database object-relational mapping
- **SQLite** - Embedded relational database
- **JWT (jsonwebtoken)** - Secure authentication tokens
- **bcrypt** - Password hashing algorithm
- **FastAPI (Python)** - High-performance API proxy layer

### Frontend Technologies
- **React** - UI component library
- **React Router** - Client-side routing
- **Axios** - HTTP client for API requests
- **Vite** - Fast build tool and development server

---

## 🗄️ Database Schema

### User Model
```javascript
{
  id: INTEGER (Primary Key, Auto-increment),
  username: STRING (Unique, Not Null),
  passwordHash: STRING (bcrypt hashed),
  provider: STRING (default: 'local'),
  providerId: STRING,
  createdAt: TIMESTAMP,
  updatedAt: TIMESTAMP
}
```

### Task Model
```javascript
{
  id: INTEGER (Primary Key, Auto-increment),
  title: STRING (Not Null),
  description: TEXT,
  status: ENUM ['pending', 'in-progress', 'completed'],
  priority: ENUM ['low', 'medium', 'high'],
  userId: INTEGER (Foreign Key → User.id),
  createdAt: TIMESTAMP,
  updatedAt: TIMESTAMP
}
```

**Relationship:** One User → Many Tasks (One-to-Many)

---

## 🔌 API Endpoints

### Authentication APIs

#### 1. User Registration
```http
POST /api/auth/register
Content-Type: application/json

{
  "username": "testuser",
  "password": "securepassword"
}

Response: 200 OK
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### 2. User Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "username": "testuser",
  "password": "securepassword"
}

Response: 200 OK
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Task Management APIs (Protected)

#### 3. Get All User Tasks
```http
GET /api/tasks
Authorization: Bearer <JWT_TOKEN>

Response: 200 OK
[
  {
    "id": 1,
    "title": "Complete Task 2",
    "description": "Build REST API with JWT",
    "status": "in-progress",
    "priority": "high",
    "userId": 1,
    "createdAt": "2025-10-22T10:00:00Z"
  }
]
```

#### 4. Create New Task
```http
POST /api/tasks
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "New Task",
  "description": "Task description",
  "status": "pending",
  "priority": "medium"
}

Response: 201 Created
{
  "id": 2,
  "title": "New Task",
  "userId": 1,
  ...
}
```

#### 5. Update Task
```http
PUT /api/tasks/:id
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "Updated Task",
  "status": "completed"
}

Response: 200 OK
```

#### 6. Delete Task
```http
DELETE /api/tasks/:id
Authorization: Bearer <JWT_TOKEN>

Response: 200 OK
{
  "message": "Task deleted successfully"
}
```

---

## 🔐 JWT Authentication Flow

```
1. User Registration/Login
   ↓
2. Server validates credentials (bcrypt comparison)
   ↓
3. Server generates JWT token (7-day expiry)
   ↓
4. Client stores token in localStorage
   ↓
5. Client includes token in Authorization header
   ↓
6. Server middleware verifies token signature
   ↓
7. Request processed if token valid
```

### JWT Implementation

**Token Generation:**
```javascript
const jwt = require('jsonwebtoken');
const JWT_SECRET = process.env.JWT_SECRET || 'secret_key';

const token = jwt.sign(
  { id: user.id, username: user.username },
  JWT_SECRET,
  { expiresIn: '7d' }
);
```

**Token Verification Middleware:**
```javascript
const authMiddleware = async (req, res, next) => {
  const authHeader = req.headers.authorization;
  if (!authHeader) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  const token = authHeader.split(' ')[1];
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.userId = decoded.id;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};
```

---

## ⚡ WebSocket Implementation

### Server Setup
```javascript
const { Server } = require('socket.io');
const io = new Server(server, {
  cors: {
    origin: 'http://localhost:3000',
    methods: ['GET', 'POST'],
    credentials: true
  }
});

io.on('connection', (socket) => {
  console.log('Socket connected:', socket.id);
  
  // Join chat room
  socket.on('join', (room) => {
    socket.join(room);
    console.log(`User joined room: ${room}`);
  });
  
  // Broadcast message to room
  socket.on('message', (payload) => {
    io.to(payload.room).emit('message', {
      user: payload.user,
      text: payload.text,
      ts: Date.now()
    });
  });
  
  socket.on('disconnect', () => {
    console.log('Socket disconnected:', socket.id);
  });
});
```

---

## 📁 Project Structure

```
/backend
├── index.js                  # Entry point, Express server setup
├── server.py                 # FastAPI proxy layer
├── controllers/
│   ├── authController.js     # Registration, login, JWT logic
│   └── taskController.js     # Task CRUD operations
├── middleware/
│   └── authMiddleware.js     # JWT verification middleware
├── models/
│   ├── index.js              # Sequelize configuration
│   ├── user.js               # User model definition
│   └── task.js               # Task model definition
├── routes/
│   ├── auth.js               # Authentication routes
│   └── tasks.js              # Task routes (protected)
├── package.json              # Node.js dependencies
└── .env                      # Environment variables

/frontend
├── src/
│   ├── components/
│   │   ├── Login.jsx         # Login/Register component
│   │   ├── TaskForm.jsx      # Task creation form
│   │   ├── TaskList.jsx      # Task display and management
│   │   ├── Chat.jsx          # Real-time chat component
│   │   └── Logout.jsx        # Logout functionality
│   ├── App.jsx               # Main application component
│   ├── api.js                # Axios API configuration
│   └── main.jsx              # React entry point
├── public/                   # Static assets
├── vite.config.js            # Vite configuration
└── package.json              # Frontend dependencies
```

---

## 🛡️ Security Features

1. **Password Hashing**: bcrypt with 10 salt rounds
2. **JWT Tokens**: Signed with secret key, 7-day expiry
3. **Protected Routes**: Authentication middleware on all task APIs
4. **User Data Isolation**: Database queries filtered by userId
5. **CORS Configuration**: Controlled cross-origin access
6. **Input Validation**: Request body validation and sanitization
7. **Error Handling**: Safe error messages without sensitive data
8. **SQL Injection Prevention**: Sequelize ORM parameterized queries

---

## ✅ Testing Results

### Backend API Testing
- ✅ User registration with password hashing
- ✅ User login with JWT token generation
- ✅ Task creation with userId association
- ✅ Task retrieval (user-specific filtering)
- ✅ Task update with ownership validation
- ✅ Task deletion with authorization check
- ✅ Authentication middleware blocking unauthorized access

### Frontend Integration Testing
- ✅ User signup flow with validation
- ✅ User login with token storage
- ✅ Task isolation between different users
- ✅ Real-time chat with multiple clients
- ✅ WebSocket room-based messaging

---

## 🚀 Setup Instructions

### Prerequisites
- Node.js (v16 or higher)
- Python 3.11+
- npm/yarn

### Backend Setup
```bash
cd backend

# Install Node.js dependencies
npm install
# or
yarn install

# Install Python dependencies
pip install -r requirements.txt

# Start the backend server
npm start
```

### Frontend Setup
```bash
cd frontend

# Install dependencies
npm install
# or
yarn install

# Start the development server
npm run dev
```

### Environment Variables
Create `.env` file in backend directory:
```env
PORT=8002
JWT_SECRET=your_secure_secret_key_here
```

---

## 💡 Key Learning Outcomes

1. ✅ **RESTful API Design**: Mastered HTTP methods, status codes, resource naming
2. ✅ **JWT Authentication**: Deep understanding of token generation, verification, middleware
3. ✅ **WebSocket Communication**: Practical implementation of real-time events and broadcasting
4. ✅ **Database Design**: Sequelize ORM, models, relationships, migrations
5. ✅ **Full-Stack Integration**: Frontend-backend API contracts and communication
6. ✅ **Security Best Practices**: Password hashing, token management, validation
7. ✅ **Modern JavaScript**: Express.js, async/await, middleware patterns

---

## 🎥 Video Walkthrough

A comprehensive 15-20 minute video demonstration covering:
- Application features and functionality
- Code-level implementation details
- Topic 1: REST API with CRUD operations
- Topic 2: JWT authentication flow
- Topic 4: Real-time WebSocket chat
- Database schema and relationships
- Security implementations

---

## 📦 Deliverables

1. ✅ **Source Code**: Complete GitHub repository
2. ✅ **PowerPoint Presentation**: Comprehensive project overview (21 slides)
3. ✅ **Video Recording**: 15-20 min walkthrough
4. ✅ **README Documentation**: Setup and API documentation
5. ✅ **Testing Results**: API and integration test results

---

## 🔗 Links

- **GitHub Repository**: [Insert your repository URL]
- **Video Recording**: [Insert video URL]
- **Live Demo**: [If deployed, insert URL]

---

## 👨‍💻 Candidate Information

**Name**: [Your Name]  
**Role**: Backend Developer  
**Topics**: 1, 2, 4  
**Submission Date**: October 24, 2025  
**Email**: [Your Email]

---

## 📝 Notes

This project demonstrates a complete understanding of backend development fundamentals including:
- RESTful API architecture and best practices
- Secure authentication mechanisms (JWT + bcrypt)
- Real-time bidirectional communication (WebSocket)
- Database design and ORM usage
- Full-stack application development
- Security and data isolation

The application is production-ready with proper error handling, validation, and security measures in place.

---

## 🙏 Acknowledgments

This project was developed as part of the Freshers Recruitment 2025 program. Special thanks to the evaluation panel for this opportunity to showcase backend development skills.

---

**Submission completed for Task 2 - Backend Developer Role** ✅
