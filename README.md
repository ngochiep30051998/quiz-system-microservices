# 🎯 Quiz System - NestJS Microservices

A production-ready quiz/exam system built with **NestJS Microservices**, **PostgreSQL + MongoDB**, **Kafka**, **RBAC**, and **React Admin Dashboard**.

## 📋 Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Services Overview](#services-overview)
- [Database Schema](#database-schema)
- [API Documentation](#api-documentation)
- [Development](#development)
- [Deployment](#deployment)

---

## ✨ Features

### Backend
- ✅ **Microservices Architecture** - 6 independent services with Kafka event streaming
- ✅ **RBAC Authentication** - JWT + Role-Based Access Control (Admin, Teacher, Student)
- ✅ **Hybrid Database** - PostgreSQL for structured data, MongoDB for flexible schemas
- ✅ **Async Processing** - Kafka for event-driven communication
- ✅ **API Gateway** - Centralized routing, auth, rate limiting
- ✅ **Auto Scoring** - Real-time score calculation on quiz submission

### Frontend
- ✅ **Admin Dashboard** - Manage questions, question banks, exams, users
- ✅ **Student Portal** - Take exams with timer, auto-save, instant results
- ✅ **Responsive UI** - Built with React + Ant Design + TypeScript
- ✅ **Real-time Updates** - WebSocket integration for live score updates

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| **Backend Framework** | NestJS 10+, TypeScript |
| **Databases** | PostgreSQL 15, MongoDB 7 |
| **Message Queue** | Apache Kafka 7.5+ |
| **ORM** | TypeORM, Mongoose |
| **Authentication** | JWT, Passport |
| **Frontend** | React 18+, Ant Design 5+, TypeScript, Vite |
| **Container** | Docker, Docker Compose |
| **CI/CD** | GitHub Actions |

---

## 📁 Project Structure

```
quiz-system-microservices/
├── packages/
│   ├── backend/
│   │   ├── api-gateway/          # Port 3000 - Routes & auth
│   │   ├── user-service/         # Port 3001 - User management
│   │   ├── question-service/     # Port 3002 - Question bank
│   │   ├── exam-service/         # Port 3003 - Exam management
│   │   ├── quiz-service/         # Port 3004 - Quiz sessions
│   │   ├── scoring-service/      # Port 3005 - Score calculation
│   │   └── shared/               # Common libs, types, constants
│   │
│   └── frontend/
│       ├── admin-dashboard/      # Admin panel (React + Vite)
│       └── student-portal/       # Student app (React + Vite)
│
├── docker-compose.yml            # Multi-container setup
├── .env.example                  # Environment variables template
├── .github/
│   └── workflows/                # CI/CD pipelines
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites

- **Node.js** 18+ & npm/pnpm/yarn
- **Docker** & Docker Compose
- **Git**

### Installation

1. **Clone Repository**
   ```bash
   git clone https://github.com/yourusername/quiz-system-microservices.git
   cd quiz-system-microservices
   ```

2. **Setup Environment Variables**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

3. **Start Databases & Kafka**
   ```bash
   docker-compose up -d
   # Wait ~30 seconds for services to start
   ```

4. **Install Dependencies**
   ```bash
   # Install all packages
   npm install
   # or
   pnpm install
   ```

5. **Start Services**
   ```bash
   # Terminal 1: API Gateway
   cd packages/backend/api-gateway
   npm run start:dev

   # Terminal 2: User Service
   cd packages/backend/user-service
   npm run start:dev

   # Terminal 3: Question Service
   cd packages/backend/question-service
   npm run start:dev

   # Terminal 4: Exam Service
   cd packages/backend/exam-service
   npm run start:dev

   # Terminal 5: Quiz Service
   cd packages/backend/quiz-service
   npm run start:dev

   # Terminal 6: Scoring Service
   cd packages/backend/scoring-service
   npm run start:dev
   ```

6. **Start Frontend (New Terminal)**
   ```bash
   # Admin Dashboard
   cd packages/frontend/admin-dashboard
   npm run dev

   # Student Portal (another terminal)
   cd packages/frontend/student-portal
   npm run dev
   ```

7. **Access Applications**
   - **Admin Dashboard**: http://localhost:5173 (default admin credentials provided)
   - **Student Portal**: http://localhost:5174
   - **API Gateway**: http://localhost:3000
   - **Kafka UI**: http://localhost:8080 (optional)

---

## ⚙️ Services Overview

### 1. API Gateway (Port: 3000)
**Responsibilities:**
- Route requests to appropriate microservices
- JWT token validation
- Rate limiting
- Error handling & CORS management
- Request/Response logging

**Key Endpoints:**
```
POST   /auth/register
POST   /auth/login
GET    /auth/me
POST   /auth/refresh
```

### 2. User Service (Port: 3001)
**Database:** PostgreSQL
**Responsibilities:**
- User registration & authentication
- Role & permission management
- User profile management
- JWT token generation

**Key Endpoints:**
```
POST   /users/register
POST   /users/login
GET    /users/:id
PUT    /users/:id
GET    /users (admin only)
PUT    /users/:id/role (admin only)
```

### 3. Question Service (Port: 3002)
**Database:** MongoDB
**Responsibilities:**
- Create/update/delete questions
- Manage question categories & difficulty levels
- Question bank creation
- Question validation

**Key Endpoints:**
```
POST   /questions (admin/teacher)
GET    /questions
GET    /questions/:id
PUT    /questions/:id (admin/teacher)
DELETE /questions/:id (admin/teacher)
POST   /question-banks (admin/teacher)
```

### 4. Exam Service (Port: 3003)
**Database:** PostgreSQL (metadata) + MongoDB (questions)
**Responsibilities:**
- Exam creation & management
- Question bank assignment to exams
- Exam scheduling
- Exam configuration

**Key Endpoints:**
```
POST   /exams (admin/teacher)
GET    /exams
GET    /exams/:id
PUT    /exams/:id (admin/teacher)
DELETE /exams/:id (admin/teacher)
POST   /exams/:id/questions
```

### 5. Quiz Service (Port: 3004)
**Database:** MongoDB
**Responsibilities:**
- Quiz session management
- Answer submission handling
- Answer validation
- Quiz state management

**Key Endpoints:**
```
POST   /quiz-sessions (start exam)
GET    /quiz-sessions/:id
POST   /quiz-sessions/:id/submit-answer
POST   /quiz-sessions/:id/submit (complete exam)
GET    /quiz-sessions/:id/result
```

### 6. Scoring Service (Port: 3005)
**Database:** PostgreSQL
**Responsibilities:**
- Listen to quiz.submitted events
- Calculate scores
- Store results
- Generate ranking
- Emit score.calculated events

**Kafka Events:**
- **Consumes**: `quiz.submitted`
- **Produces**: `score.calculated`, `score.updated`

---

## 💾 Database Schema

### PostgreSQL Tables

```sql
-- Users & RBAC
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  password_hash VARCHAR NOT NULL,
  full_name VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE roles (
  id SERIAL PRIMARY KEY,
  name VARCHAR UNIQUE,
  permissions JSONB
);

CREATE TABLE user_roles (
  user_id UUID REFERENCES users(id),
  role_id INT REFERENCES roles(id),
  PRIMARY KEY (user_id, role_id)
);

-- Exams
CREATE TABLE exams (
  id UUID PRIMARY KEY,
  title VARCHAR NOT NULL,
  description TEXT,
  duration_minutes INT,
  passing_score DECIMAL,
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Scores
CREATE TABLE scores (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  exam_id UUID REFERENCES exams(id),
  total_score DECIMAL,
  percentage DECIMAL,
  status VARCHAR,
  submitted_at TIMESTAMP
);
```

### MongoDB Collections

```javascript
// Questions
db.questions.insertOne({
  _id: ObjectId(),
  text: "Question text",
  type: "multiple_choice",
  options: [
    { text: "Option A", is_correct: true },
    { text: "Option B", is_correct: false }
  ],
  category: "Math",
  difficulty: "medium",
  points: 1,
  created_at: ISODate()
});

// Quiz Submissions
db.quiz_submissions.insertOne({
  _id: ObjectId(),
  user_id: UUID,
  exam_id: UUID,
  answers: [
    {
      question_id: ObjectId(),
      selected_option: 0,
      is_correct: true,
      points_earned: 1
    }
  ],
  total_score: 10,
  started_at: ISODate(),
  submitted_at: ISODate()
});
```

---

## 📚 API Documentation

Full API documentation available in `/docs/API.md`

### Authentication

All protected endpoints require JWT token in Authorization header:

```bash
Authorization: Bearer <token>
```

### Example Request

```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password"}'
```

---

## 👨‍💻 Development

### Adding a New Service

1. Create new service directory
   ```bash
   cd packages/backend
   nest new my-service
   ```

2. Configure Kafka consumer/producer
3. Add TypeORM or Mongoose models
4. Register routes in API Gateway

### Running Tests

```bash
# Unit tests
npm run test

# Integration tests
npm run test:e2e

# Test coverage
npm run test:cov
```

### Code Quality

```bash
# Lint code
npm run lint

# Format code
npm run format
```

---

## 🐳 Docker Deployment

### Build Images

```bash
docker-compose build
```

### Deploy to Production

```bash
docker-compose -f docker-compose.prod.yml up -d
```

---

## 📊 RBAC Roles & Permissions

| Role | Permissions |
|------|-------------|
| **Admin** | Manage users, create/edit questions & exams, view all scores |
| **Teacher** | Create/edit questions & exams, view student scores |
| **Student** | Take exams, view own scores |

---

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

---

## 📝 License

MIT License - see LICENSE file for details

---

## 📞 Support

For issues and questions:
- Open an [Issue](https://github.com/yourusername/quiz-system-microservices/issues)
- Contact: support@quizsystem.com

---

## 🙏 Acknowledgments

- NestJS community
- Ant Design team
- Apache Kafka
- PostgreSQL & MongoDB communities

---

**Last Updated:** February 2026 | **Version:** 1.0.0
