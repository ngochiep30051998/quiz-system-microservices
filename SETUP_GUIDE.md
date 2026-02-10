# 🚀 Complete Setup & Development Guide

This guide walks you through setting up the entire Quiz System locally and understanding the codebase.

---

## 📄 Prerequisites

### Required Software
- **Node.js** 18.0.0 or higher
- **npm** 9.0.0 or higher (or yarn/pnpm)
- **Docker** & **Docker Compose**
- **Git**
- **VS Code** (recommended) with extensions:
  - TypeScript Vue Plugin
  - ES7+ React/Redux/React-Native snippets
  - Prettier
  - ESLint

### System Requirements
- **RAM**: 8GB minimum
- **Disk Space**: 10GB minimum
- **OS**: Windows 10+, macOS 10.15+, or Ubuntu 20.04+

---

## ⚙️ Step 1: Clone & Initial Setup

### 1.1 Clone Repository

```bash
git clone https://github.com/yourusername/quiz-system-microservices.git
cd quiz-system-microservices
```

### 1.2 Install Dependencies

```bash
# Using npm (recommended)
npm install

# Or using yarn
yarn install

# Or using pnpm
pnpm install
```

### 1.3 Setup Environment Variables

```bash
# Copy example to actual .env
cp .env.example .env

# Edit with your configuration (if needed)
vim .env
```

---

## 💾 Step 2: Start Infrastructure

### 2.1 Start Docker Services

```bash
# Start all services (PostgreSQL, MongoDB, Kafka, Redis)
docker-compose up -d

# Wait for services to be healthy (~30 seconds)
docker-compose ps

# View logs
docker-compose logs -f
```

### 2.2 Verify Services

```bash
# Check PostgreSQL
psql -h localhost -U quiz_user -d quiz_db

# Check MongoDB
mongosh mongodb://mongo_user:mongo_password@localhost:27017

# Check Kafka
kafka-topics.sh --list --bootstrap-server localhost:9092

# Check Redis
redis-cli ping
```

### 2.3 Access Kafka UI (Optional)

Open browser: `http://localhost:8080`

---

## 💻 Step 3: Setup Backend Services

### 3.1 Install Backend Dependencies

```bash
cd packages/backend
npm install
```

### 3.2 Generate Services (if not already created)

```bash
# Create each service using NestJS CLI
nest new api-gateway
nest new user-service
nest new question-service
nest new exam-service
nest new quiz-service
nest new scoring-service

# Install shared dependencies for all services
npm install --save @nestjs/config @nestjs/jwt @nestjs/passport @nestjs/microservices @nestjs/typeorm typeorm pg mongodb mongoose @nestjs/mongoose class-validator class-transformer bcrypt uuid dotenv
```

### 3.3 Database Setup

```bash
# Run migrations (from each service directory)
cd user-service
npm run typeorm:migration:run

cd ../question-service
# MongoDB setup happens automatically on connection
```

---

## 🚀 Step 4: Start Backend Services

Open **6 separate terminal windows** and run each:

### Terminal 1: API Gateway

```bash
cd packages/backend/api-gateway
npm run start:dev
# Output: API Gateway running on port 3000
```

### Terminal 2: User Service

```bash
cd packages/backend/user-service
npm run start:dev
# Output: User Service listening on port 3001
```

### Terminal 3: Question Service

```bash
cd packages/backend/question-service
npm run start:dev
# Output: Question Service listening on port 3002
```

### Terminal 4: Exam Service

```bash
cd packages/backend/exam-service
npm run start:dev
# Output: Exam Service listening on port 3003
```

### Terminal 5: Quiz Service

```bash
cd packages/backend/quiz-service
npm run start:dev
# Output: Quiz Service listening on port 3004
```

### Terminal 6: Scoring Service

```bash
cd packages/backend/scoring-service
npm run start:dev
# Output: Scoring Service listening on port 3005
```

---

## ⚡️ Step 5: Setup Frontend

### 5.1 Install Frontend Dependencies

```bash
cd packages/frontend/admin-dashboard
npm install

# In another terminal
cd packages/frontend/student-portal
npm install
```

### 5.2 Start Frontend Applications

**Terminal 7: Admin Dashboard**

```bash
cd packages/frontend/admin-dashboard
npm run dev
# Output: Local: http://localhost:5173
```

**Terminal 8: Student Portal**

```bash
cd packages/frontend/student-portal
npm run dev
# Output: Local: http://localhost:5174
```

---

## 📋 Step 6: Verify Everything Works

### 6.1 Backend Health Check

```bash
# Check API Gateway
curl http://localhost:3000/health

# Expected response:
# {"status":"ok"}
```

### 6.2 Frontend Access

- **Admin Dashboard**: http://localhost:5173
- **Student Portal**: http://localhost:5174

### 6.3 Test API Endpoints

```bash
# Register new user
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email":"admin@example.com",
    "password":"Password123",
    "full_name":"Admin User"
  }'

# Login
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email":"admin@example.com",
    "password":"Password123"
  }'

# Create question
curl -X POST http://localhost:3000/questions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "text":"What is 2+2?",
    "type":"single_choice",
    "options":[
      {"text":"3","is_correct":false},
      {"text":"4","is_correct":true}
    ],
    "points":1
  }'
```

---

## 👨‍💻 Step 7: Development Workflow

### 7.1 Code Organization

```
Backend Structure:
service/
  ├── src/
  │   ├── <feature>/
  │   │   ├── dto/
  │   │   ├── entities/
  │   │   ├── services/
  │   │   ├── controllers/
  │   │   └── <feature>.module.ts
  │   ├── app.module.ts
  │   └── main.ts
  ├── test/
  ├── package.json
  └── tsconfig.json
```

### 7.2 Creating New Features

```bash
# Example: Add new feature to user service
cd packages/backend/user-service

# Generate new module
nest generate module roles
nest generate service roles
nest generate controller roles

# Create entities and DTOs
touch src/roles/entities/role.entity.ts
touch src/roles/dto/create-role.dto.ts
```

### 7.3 Running Tests

```bash
# Unit tests
cd packages/backend/user-service
npm run test

# Test coverage
npm run test:cov

# E2E tests
npm run test:e2e
```

### 7.4 Code Quality

```bash
# Lint
npm run lint

# Format code
npm run format

# Fix linting issues
npm run lint:fix
```

---

## 📄 Step 8: Frontend Development

### 8.1 Component Structure

```
frontend/
  src/
  ├── components/
  │   ├── common/      (Header, Sidebar, Footer)
  │   ├── layout/      (Main layout)
  │   ├── admin/       (Admin-specific)
  │   ├── student/     (Student-specific)
  │   └── forms/       (Shared forms)
  ├── pages/
  ├── store/         (Zustand stores)
  ├── hooks/         (Custom hooks)
  ├── utils/         (API client, helpers)
  ├── types/         (TypeScript interfaces)
  ├── styles/
  └── App.tsx
```

### 8.2 Adding New Pages

```bash
cd packages/frontend/admin-dashboard

# Create page component
mkdir src/pages/new-feature
touch src/pages/new-feature/index.tsx
touch src/pages/new-feature/useNewFeature.ts

# Update router
# Add route in src/App.tsx
```

### 8.3 Creating API Client

```typescript
// src/utils/api.ts
import axios from 'axios';
import { useAuthStore } from '../store/auth.store';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

// Add token to requests
apiClient.interceptors.request.use((config) => {
  const { token } = useAuthStore.getState();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default apiClient;
```

---

## 📁 Step 9: Database Management

### 9.1 PostgreSQL

```bash
# Connect to PostgreSQL
psql -h localhost -U quiz_user -d quiz_db

# Common commands
\dt              # List tables
\d <table_name>  # Describe table
\l              # List databases
\q              # Quit
```

### 9.2 MongoDB

```bash
# Connect to MongoDB
mongosh mongodb://mongo_user:mongo_password@localhost:27017/quiz_db

# Common commands
db.collections()           # List collections
db.<collection>.find()     # View documents
db.<collection>.drop()     # Delete collection
exit                       # Quit
```

### 9.3 Run Migrations

```bash
# TypeORM migrations
cd packages/backend/user-service
npm run typeorm:migration:generate -- -n CreateUsersTable
npm run typeorm:migration:run

# Revert migration
npm run typeorm:migration:revert
```

---

## 🛠️ Troubleshooting

### Problem: Port Already in Use

```bash
# Find process using port
lsof -i :3000

# Kill process
kill -9 <PID>
```

### Problem: Database Connection Failed

```bash
# Check Docker containers
docker ps

# Check container logs
docker logs quiz_postgres
docker logs quiz_mongodb

# Restart services
docker-compose restart
```

### Problem: Dependencies Not Installing

```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules
rm -rf node_modules package-lock.json

# Reinstall
npm install
```

### Problem: Hot Reload Not Working

```bash
# Make sure you're using start:dev
npm run start:dev

# Not start:prod
```

---

## 📄 Useful Commands Reference

```bash
# View all running services
docker-compose ps

# Stop all services
docker-compose down

# Remove volumes (delete data)
docker-compose down -v

# Rebuild images
docker-compose build --no-cache

# View logs
docker-compose logs -f <service_name>

# Run tests for all services
npm run test:all

# Build for production
npm run build

# Lint entire project
npm run lint:all
```

---

## 🚀 Next Steps

1. **Understand Architecture**: Read `docs/ARCHITECTURE.md`
2. **API Documentation**: Check `docs/API.md`
3. **Development**: Start with `docs/DEVELOPMENT.md`
4. **Deployment**: See `docs/DEPLOYMENT.md`
5. **Contributing**: Review `CONTRIBUTING.md`

---

## 📞 Support

For issues:
1. Check existing issues in GitHub
2. Create detailed issue with:
   - Error message
   - Steps to reproduce
   - Environment info (Node version, OS, etc.)
   - Logs

---

**Last Updated**: February 2026
