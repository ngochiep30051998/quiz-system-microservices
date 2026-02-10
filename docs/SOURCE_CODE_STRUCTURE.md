# 📊 Source Code Structure & Templates

This document provides complete source code templates for all microservices and frontend applications.

---

## 📁 Backend Services Structure

### Project Layout

```
packages/backend/
├── api-gateway/
│   ├── src/
│   │   ├── app.controller.ts
│   │   ├── app.module.ts
│   │   ├── config/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── main.ts
│   ├── package.json
│   ├── tsconfig.json
│   └── .env
│
├── user-service/
│   ├── src/
│   │   ├── users/
│   │   │   ├── dto/
│   │   │   ├── entities/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── guards/
│   │   │   └── users.module.ts
│   │   ├── auth/
│   │   ├── roles/
│   │   ├── config/
│   │   ├── app.module.ts
│   │   └── main.ts
│   ├── package.json
│   ├── tsconfig.json
│   └── .env
│
├── question-service/
├── exam-service/
├── quiz-service/
├── scoring-service/
└── shared/
    ├── src/
    │   ├── decorators/
    │   ├── filters/
    │   ├── guards/
    │   ├── interceptors/
    │   ├── pipes/
    │   ├── interfaces/
    │   ├── constants/
    │   └── types/
    └── package.json
```

---

## 💻 Backend Code Templates

### 1. API Gateway - Main Configuration

**File: `packages/backend/api-gateway/src/main.ts`**

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';
import { HttpExceptionFilter } from './filters/http-exception.filter';
import { LoggingInterceptor } from './interceptors/logging.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Global pipes
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  // Global filters
  app.useGlobalFilters(new HttpExceptionFilter());

  // Global interceptors
  app.useGlobalInterceptors(new LoggingInterceptor());

  // CORS
  app.enableCors({
    origin: process.env.CORS_ORIGIN?.split(',') || '*',
    credentials: true,
  });

  const port = process.env.GATEWAY_PORT || 3000;
  await app.listen(port);
  console.log(`API Gateway running on port ${port}`);
}

bootstrap();
```

### 2. User Service - Authentication Module

**File: `packages/backend/user-service/src/auth/auth.controller.ts`**

```typescript
import { Controller, Post, Body, UseGuards, Get, Req } from '@nestjs/common';
import { AuthService } from './auth.service';
import { RegisterDto, LoginDto } from './dto';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { Request } from 'express';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('register')
  async register(@Body() registerDto: RegisterDto) {
    return this.authService.register(registerDto);
  }

  @Post('login')
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }

  @UseGuards(JwtAuthGuard)
  @Get('me')
  async getCurrentUser(@Req() req: Request) {
    return (req as any).user;
  }

  @Post('refresh')
  async refreshToken(@Body('refresh_token') refreshToken: string) {
    return this.authService.refreshToken(refreshToken);
  }
}
```

**File: `packages/backend/user-service/src/auth/auth.service.ts`**

```typescript
import { Injectable, UnauthorizedException, BadRequestException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import { RegisterDto, LoginDto } from './dto';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async register(registerDto: RegisterDto) {
    const user = await this.usersService.findByEmail(registerDto.email);
    if (user) {
      throw new BadRequestException('User already exists');
    }

    const hashedPassword = await bcrypt.hash(registerDto.password, 10);
    const newUser = await this.usersService.create({
      ...registerDto,
      password: hashedPassword,
    });

    const { password, ...result } = newUser;
    return {
      user: result,
      access_token: this.generateAccessToken(result),
      refresh_token: this.generateRefreshToken(result),
    };
  }

  async login(loginDto: LoginDto) {
    const user = await this.usersService.findByEmail(loginDto.email);
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(
      loginDto.password,
      user.password,
    );
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const { password, ...result } = user;
    return {
      user: result,
      access_token: this.generateAccessToken(result),
      refresh_token: this.generateRefreshToken(result),
    };
  }

  private generateAccessToken(user: any) {
    return this.jwtService.sign(
      {
        sub: user.id,
        email: user.email,
        role: user.role,
      },
      {
        secret: process.env.JWT_SECRET,
        expiresIn: process.env.JWT_EXPIRATION,
      },
    );
  }

  private generateRefreshToken(user: any) {
    return this.jwtService.sign(
      { sub: user.id },
      {
        secret: process.env.JWT_REFRESH_SECRET,
        expiresIn: process.env.JWT_REFRESH_EXPIRATION,
      },
    );
  }

  async refreshToken(refreshToken: string) {
    try {
      const payload = this.jwtService.verify(refreshToken, {
        secret: process.env.JWT_REFRESH_SECRET,
      });
      const user = await this.usersService.findById(payload.sub);
      return {
        access_token: this.generateAccessToken(user),
      };
    } catch (error) {
      throw new UnauthorizedException('Invalid refresh token');
    }
  }
}
```

### 3. Question Service - MongoDB Integration

**File: `packages/backend/question-service/src/questions/schemas/question.schema.ts`**

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document, Types } from 'mongoose';

export type QuestionDocument = Question & Document;

@Schema({ timestamps: true })
export class Question {
  @Prop({ required: true })
  text: string;

  @Prop({
    enum: ['single_choice', 'multiple_choice', 'true_false'],
    required: true,
  })
  type: string;

  @Prop([
    {
      text: String,
      is_correct: Boolean,
    },
  ])
  options: Array<{ text: string; is_correct: boolean }>;

  @Prop()
  category: string;

  @Prop({ enum: ['easy', 'medium', 'hard'], default: 'medium' })
  difficulty: string;

  @Prop({ default: 1 })
  points: number;

  @Prop()
  explanation: string;

  @Prop({ type: Types.ObjectId })
  created_by: Types.ObjectId;

  @Prop({ type: Types.ObjectId, ref: 'QuestionBank' })
  question_bank_id: Types.ObjectId;
}

export const QuestionSchema = SchemaFactory.createForClass(Question);
```

**File: `packages/backend/question-service/src/questions/questions.service.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { Question, QuestionDocument } from './schemas/question.schema';
import { CreateQuestionDto, UpdateQuestionDto } from './dto';

@Injectable()
export class QuestionsService {
  constructor(
    @InjectModel(Question.name) private questionModel: Model<QuestionDocument>,
  ) {}

  async create(
    createQuestionDto: CreateQuestionDto,
    userId: string,
  ): Promise<Question> {
    const createdQuestion = new this.questionModel({
      ...createQuestionDto,
      created_by: userId,
    });
    return createdQuestion.save();
  }

  async findAll(filters?: any): Promise<Question[]> {
    const query: any = {};
    if (filters?.category) query.category = filters.category;
    if (filters?.difficulty) query.difficulty = filters.difficulty;
    return this.questionModel.find(query).exec();
  }

  async findById(id: string): Promise<Question> {
    return this.questionModel.findById(id).exec();
  }

  async update(
    id: string,
    updateQuestionDto: UpdateQuestionDto,
  ): Promise<Question> {
    return this.questionModel
      .findByIdAndUpdate(id, updateQuestionDto, { new: true })
      .exec();
  }

  async delete(id: string): Promise<void> {
    await this.questionModel.findByIdAndDelete(id).exec();
  }

  async findByIds(ids: string[]): Promise<Question[]> {
    return this.questionModel.find({ _id: { $in: ids } }).exec();
  }
}
```

### 4. Quiz Service - Kafka Integration

**File: `packages/backend/quiz-service/src/quiz/quiz.controller.ts`**

```typescript
import {
  Controller,
  Post,
  Get,
  Param,
  Body,
  UseGuards,
  BadRequestException,
} from '@nestjs/common';
import { QuizService } from './quiz.service';
import { JwtAuthGuard } from '../../../shared/src/guards/jwt-auth.guard';
import { GetUser } from '../../../shared/src/decorators/get-user.decorator';
import { CreateQuizSessionDto, SubmitAnswerDto } from './dto';

@Controller('quiz')
@UseGuards(JwtAuthGuard)
export class QuizController {
  constructor(private quizService: QuizService) {}

  @Post('sessions')
  async startQuiz(
    @Body() createQuizSessionDto: CreateQuizSessionDto,
    @GetUser() user: any,
  ) {
    return this.quizService.createSession(
      createQuizSessionDto.exam_id,
      user.id,
    );
  }

  @Get('sessions/:id')
  async getSession(@Param('id') sessionId: string) {
    return this.quizService.getSession(sessionId);
  }

  @Post('sessions/:id/submit-answer')
  async submitAnswer(
    @Param('id') sessionId: string,
    @Body() submitAnswerDto: SubmitAnswerDto,
  ) {
    return this.quizService.submitAnswer(sessionId, submitAnswerDto);
  }

  @Post('sessions/:id/submit')
  async submitQuiz(@Param('id') sessionId: string) {
    return this.quizService.submitQuiz(sessionId);
  }

  @Get('sessions/:id/result')
  async getResult(@Param('id') sessionId: string) {
    return this.quizService.getResult(sessionId);
  }
}
```

**File: `packages/backend/quiz-service/src/quiz/quiz.service.ts`**

```typescript
import { Injectable, BadRequestException } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { ClientKafka } from '@nestjs/microservices';
import { QuizSubmission, QuizSubmissionDocument } from './schemas';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class QuizService {
  constructor(
    @InjectModel(QuizSubmission.name)
    private quizSubmissionModel: Model<QuizSubmissionDocument>,
    private kafkaClient: ClientKafka,
  ) {}

  async createSession(examId: string, userId: string) {
    const sessionId = uuidv4();
    const session = new this.quizSubmissionModel({
      _id: sessionId,
      exam_id: examId,
      user_id: userId,
      answers: [],
      started_at: new Date(),
      status: 'in_progress',
    });
    await session.save();

    // Emit Kafka event
    this.kafkaClient.emit('quiz.started', {
      session_id: sessionId,
      user_id: userId,
      exam_id: examId,
      started_at: new Date(),
    });

    return { session_id: sessionId };
  }

  async submitAnswer(
    sessionId: string,
    { question_id, selected_option }: any,
  ) {
    const session = await this.quizSubmissionModel.findById(sessionId);
    if (!session) {
      throw new BadRequestException('Session not found');
    }

    session.answers.push({
      question_id,
      selected_option,
      submitted_at: new Date(),
    });

    return session.save();
  }

  async submitQuiz(sessionId: string) {
    const session = await this.quizSubmissionModel.findById(sessionId);
    if (!session) {
      throw new BadRequestException('Session not found');
    }

    session.status = 'submitted';
    session.submitted_at = new Date();
    await session.save();

    // Emit Kafka event for scoring
    this.kafkaClient.emit('quiz.submitted', {
      session_id: sessionId,
      user_id: session.user_id,
      exam_id: session.exam_id,
      answers: session.answers,
      submitted_at: new Date(),
    });

    return { message: 'Quiz submitted successfully' };
  }

  async getSession(sessionId: string) {
    return this.quizSubmissionModel.findById(sessionId);
  }

  async getResult(sessionId: string) {
    return this.quizSubmissionModel.findById(sessionId);
  }
}
```

### 5. Scoring Service - Kafka Consumer

**File: `packages/backend/scoring-service/src/scoring/scoring.service.ts`**

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { EventPattern, Payload, Ctx, KafkaContext } from '@nestjs/microservices';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Score } from './entities/score.entity';

@Injectable()
export class ScoringService {
  private readonly logger = new Logger(ScoringService.name);

  constructor(
    @InjectRepository(Score)
    private scoreRepository: Repository<Score>,
  ) {}

  @EventPattern('quiz.submitted')
  async handleQuizSubmitted(
    @Payload() data: any,
    @Ctx() context: KafkaContext,
  ) {
    try {
      const { session_id, user_id, exam_id, answers } = data;

      // Calculate score
      const score = this.calculateScore(answers);

      // Save to database
      const scoreRecord = this.scoreRepository.create({
        user_id,
        exam_id,
        quiz_session_id: session_id,
        total_score: score.totalScore,
        percentage: score.percentage,
        status: score.percentage >= 70 ? 'passed' : 'failed',
        submitted_at: new Date(),
      });

      await this.scoreRepository.save(scoreRecord);

      this.logger.log(
        `Score calculated for user ${user_id}: ${score.percentage}%`,
      );

      // Emit score calculated event
      // this.kafkaClient.emit('score.calculated', { ... });
    } catch (error) {
      this.logger.error(`Error calculating score: ${error.message}`);
    }
  }

  private calculateScore(
    answers: Array<{
      question_id: string;
      selected_option: number;
      is_correct?: boolean;
      points_earned?: number;
    }>,
  ) {
    let totalScore = 0;
    let maxScore = answers.length; // Assuming 1 point per question

    answers.forEach((answer) => {
      if (answer.is_correct) {
        totalScore += answer.points_earned || 1;
      }
    });

    return {
      totalScore,
      percentage: (totalScore / maxScore) * 100,
    };
  }
}
```

---

## ⚡️ Frontend Code Templates

### 1. React Setup - Admin Dashboard

**File: `packages/frontend/admin-dashboard/src/App.tsx`**

```typescript
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { ConfigProvider, theme } from 'antd';
import { useAuthStore } from './store/auth.store';
import Layout from './components/layout';
import Dashboard from './pages/dashboard';
import QuestionsPage from './pages/questions';
import ExamsPage from './pages/exams';
import LoginPage from './pages/login';
import ProtectedRoute from './components/ProtectedRoute';

function App() {
  const { token: authToken } = useAuthStore();

  return (
    <ConfigProvider
      theme={{
        algorithm: theme.defaultAlgorithm,
        token: {
          colorPrimary: '#0284c7',
        },
      }}
    >
      <Router>
        {!authToken ? (
          <Routes>
            <Route path="/login" element={<LoginPage />} />
            <Route path="*" element={<LoginPage />} />
          </Routes>
        ) : (
          <Layout>
            <Routes>
              <Route path="/" element={<Dashboard />} />
              <Route path="/questions" element={<QuestionsPage />} />
              <Route path="/exams" element={<ExamsPage />} />
            </Routes>
          </Layout>
        )}
      </Router>
    </ConfigProvider>
  );
}

export default App;
```

**File: `packages/frontend/admin-dashboard/src/store/auth.store.ts`**

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  role: string;
}

interface AuthStore {
  token: string | null;
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>(
  persist(
    (set) => ({
      token: null,
      user: null,
      login: async (email, password) => {
        // API call to login
        const response = await fetch('http://localhost:3000/auth/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email, password }),
        });
        const data = await response.json();
        set({ token: data.access_token, user: data.user });
      },
      logout: () => set({ token: null, user: null }),
    }),
    { name: 'auth-store' },
  ),
);
```

---

## 📻 How to Generate Full Services

### Using NestJS CLI

```bash
# Generate User Service
cd packages/backend
nest new user-service
cd user-service

# Generate CRUD modules
nest generate module users
nest generate controller users
nest generate service users

# Add TypeORM
npm install @nestjs/typeorm typeorm pg
```

### Configuration Files

Each service needs:
- `app.module.ts` - Main module
- `main.ts` - Entry point
- `package.json` - Dependencies
- `tsconfig.json` - TypeScript config
- `Dockerfile` - Docker configuration

---

## 🚀 Deployment

See `docs/DEPLOYMENT.md` for production setup instructions.

---

**Note**: Use these templates as a base and customize according to your needs.
