# 🚀 Quick Start - Get Running in 5 Minutes

## Prerequisites
- Node.js 18+
- Docker & Docker Compose
- Git

## 🎯 TL;DR

```bash
# 1. Clone
git clone https://github.com/yourusername/quiz-system-microservices.git
cd quiz-system-microservices

# 2. Install & Setup
npm install
cp .env.example .env

# 3. Start Infrastructure
docker-compose up -d

# 4. Wait 30 seconds, then open 8 terminals and run:
# Terminal 1
cd packages/backend/api-gateway && npm run start:dev
# Terminal 2
cd packages/backend/user-service && npm run start:dev
# Terminal 3
cd packages/backend/question-service && npm run start:dev
# Terminal 4
cd packages/backend/exam-service && npm run start:dev
# Terminal 5
cd packages/backend/quiz-service && npm run start:dev
# Terminal 6
cd packages/backend/scoring-service && npm run start:dev
# Terminal 7
cd packages/frontend/admin-dashboard && npm run dev
# Terminal 8
cd packages/frontend/student-portal && npm run dev

# 5. Access
Admin Dashboard: http://localhost:5173
Student Portal: http://localhost:5174
API: http://localhost:3000
Kafka UI: http://localhost:8080
```

## 📄 Full Setup Instructions

See **SETUP_GUIDE.md** for detailed step-by-step instructions.

## 📚 Documentation

| Document | Purpose |
|----------|----------|
| **README.md** | Project overview & features |
| **SETUP_GUIDE.md** | Complete setup instructions |
| **docs/SOURCE_CODE_STRUCTURE.md** | Code templates & architecture |
| **docker-compose.yml** | Infrastructure definition |
| **.env.example** | Environment variables |

## 🚗 Troubleshooting

### Services won't start
```bash
# Check Docker services
docker-compose ps

# View logs
docker-compose logs -f kafka
```

### Port already in use
```bash
# Find process
lsof -i :3000
# Kill it
kill -9 <PID>
```

### Database connection error
```bash
# Restart Docker
docker-compose restart
```

## 📄 Project Structure

```
quiz-system-microservices/
├── packages/backend/          # 6 NestJS microservices
│   ├── api-gateway/
│   ├── user-service/
│   ├── question-service/
│   ├── exam-service/
│   ├── quiz-service/
│   ├── scoring-service/
│   └── shared/
├── packages/frontend/         # React applications
│   ├── admin-dashboard/
│   └── student-portal/
├── docs/                      # Documentation
├── docker-compose.yml
├── .env.example
├── README.md
├── SETUP_GUIDE.md
└── QUICK_START.md
```

## 🌟 Key URLs

| Service | URL |
|---------|-----|
| Admin Dashboard | http://localhost:5173 |
| Student Portal | http://localhost:5174 |
| API Gateway | http://localhost:3000 |
| Kafka UI | http://localhost:8080 |
| PostgreSQL | localhost:5432 |
| MongoDB | localhost:27017 |
| Redis | localhost:6379 |

## 📧 API Examples

```bash
# Register
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@test.com","password":"Test123","full_name":"Test User"}'

# Login
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@test.com","password":"Test123"}'

# Get Current User
curl http://localhost:3000/auth/me \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## 💻 Development Commands

```bash
# Lint
npm run lint

# Format
npm run format

# Test
npm run test

# Build
npm run build

# Docker
docker-compose up      # Start
docker-compose down    # Stop
docker-compose down -v # Stop & delete data
```

## 🛠️ System Requirements

- RAM: 8GB minimum
- Disk Space: 10GB minimum
- CPU: Dual core minimum

## 📚 Architecture Overview

```
┌─────────────┐
│  React Frontend   │
│  (Admin/Student)  │
└─────────────┘
         ┃
┌─────────────┐
│   API Gateway    │ (3000)
└─────────────┘
    │    │     │      │        │
    │    │     │      │        │
    │    │     │      │        │
USER QUESTION EXAM QUIZ SCORING
(3001) (3002) (3003)(3004)(3005)
    │    │     │      │        │
    └──────────────┘
         ┃
┌─────────────┐
│  Kafka (Event Bus)│
└─────────────┘
         ┃
┌─────────────┐
│PostgreSQL|MongoDB │
│  Redis   |Kafka   │
└─────────────┘
```

## 📝 Environment Variables

See `.env.example` for all available variables.

Key variables:
- `NODE_ENV` - development/production
- `DB_HOST`, `DB_USER`, `DB_PASSWORD` - PostgreSQL
- `MONGO_URI` - MongoDB connection
- `KAFKA_BROKERS` - Kafka brokers
- `JWT_SECRET` - JWT token secret

## 🚀 Next Steps

1. **Setup**: Follow SETUP_GUIDE.md
2. **Code**: Check docs/SOURCE_CODE_STRUCTURE.md
3. **Develop**: Start building features
4. **Deploy**: See docs/DEPLOYMENT.md

## 📞 Support

Having issues?
1. Check SETUP_GUIDE.md troubleshooting section
2. Review Docker logs: `docker-compose logs’
3. Create GitHub issue with details

---

**Ready to get started?** Follow the TL;DR above! 🎉
