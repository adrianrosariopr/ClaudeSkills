<overview>
Node.js backend patterns covering Express, Fastify, Prisma ORM, TypeScript, and async programming. Focus on modern practices for building scalable APIs.
</overview>

<project_structure>
**Recommended Structure:**

```
src/
├── controllers/          # Route handlers
├── services/            # Business logic
├── repositories/        # Data access
├── models/              # Type definitions
├── middleware/          # Express middleware
├── routes/              # Route definitions
├── utils/               # Helpers
├── config/              # Configuration
├── jobs/                # Background jobs
└── index.ts             # Entry point

prisma/
├── schema.prisma        # Database schema
└── migrations/

tests/
├── unit/
├── integration/
└── fixtures/
```
</project_structure>

<express_patterns>
**Express Setup:**

```typescript
import express, { Express, Request, Response, NextFunction } from 'express';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';

const app: Express = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json());
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));

// Routes
app.use('/api/users', userRouter);
app.use('/api/posts', postRouter);

// Error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});

export default app;
```

**Route Organization:**

```typescript
// routes/users.ts
import { Router } from 'express';
import { UserController } from '../controllers/UserController';
import { authenticate } from '../middleware/auth';
import { validate } from '../middleware/validate';
import { createUserSchema, updateUserSchema } from '../schemas/user';

const router = Router();
const controller = new UserController();

router.get('/', controller.list);
router.get('/:id', controller.show);
router.post('/', validate(createUserSchema), controller.create);
router.put('/:id', authenticate, validate(updateUserSchema), controller.update);
router.delete('/:id', authenticate, controller.delete);

export default router;
```
</express_patterns>

<fastify_patterns>
**Fastify Setup (faster alternative):**

```typescript
import Fastify from 'fastify';
import cors from '@fastify/cors';
import helmet from '@fastify/helmet';

const fastify = Fastify({
  logger: true,
});

await fastify.register(cors);
await fastify.register(helmet);

// Routes with schema validation
fastify.post('/users', {
  schema: {
    body: {
      type: 'object',
      required: ['email', 'password'],
      properties: {
        email: { type: 'string', format: 'email' },
        password: { type: 'string', minLength: 8 },
      },
    },
    response: {
      201: {
        type: 'object',
        properties: {
          id: { type: 'string' },
          email: { type: 'string' },
        },
      },
    },
  },
  handler: async (request, reply) => {
    const user = await userService.create(request.body);
    return reply.code(201).send(user);
  },
});
```
</fastify_patterns>

<prisma_orm>
**Prisma ORM:**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  password  String
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())

  @@index([authorId])
}
```

**Prisma Client Usage:**

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Create
const user = await prisma.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
    password: hashedPassword,
  },
});

// Find with relations (eager loading)
const posts = await prisma.post.findMany({
  where: { published: true },
  include: {
    author: { select: { id: true, name: true } },
  },
  orderBy: { createdAt: 'desc' },
  take: 20,
});

// Transaction
const [order, payment] = await prisma.$transaction([
  prisma.order.create({ data: orderData }),
  prisma.payment.create({ data: paymentData }),
]);

// Interactive transaction
await prisma.$transaction(async (tx) => {
  const user = await tx.user.findUnique({ where: { id: userId } });
  if (user.balance < amount) throw new Error('Insufficient funds');
  await tx.user.update({
    where: { id: userId },
    data: { balance: { decrement: amount } },
  });
});
```
</prisma_orm>

<typescript_patterns>
**Type Definitions:**

```typescript
// Types from Prisma
import { User, Post, Prisma } from '@prisma/client';

// Input types
type CreateUserInput = Prisma.UserCreateInput;
type UpdateUserInput = Prisma.UserUpdateInput;

// Custom types
interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    perPage: number;
    totalPages: number;
  };
}

// Response types
type UserResponse = Pick<User, 'id' | 'email' | 'name'>;

// Service interface
interface UserService {
  findById(id: string): Promise<User | null>;
  create(data: CreateUserInput): Promise<User>;
  update(id: string, data: UpdateUserInput): Promise<User>;
  delete(id: string): Promise<void>;
}
```

**Zod Validation:**

```typescript
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().optional(),
});

const updateUserSchema = createUserSchema.partial();

type CreateUserInput = z.infer<typeof createUserSchema>;

// Middleware
const validate = (schema: z.ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      req.body = schema.parse(req.body);
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        res.status(422).json({ errors: error.errors });
      } else {
        next(error);
      }
    }
  };
};
```
</typescript_patterns>

<async_patterns>
**Async Best Practices:**

```typescript
// Async controller wrapper
const asyncHandler = (fn: Function) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage
router.get('/users', asyncHandler(async (req, res) => {
  const users = await userService.findAll();
  res.json(users);
}));

// Parallel operations
const [users, posts, comments] = await Promise.all([
  userService.findAll(),
  postService.findAll(),
  commentService.findAll(),
]);

// Error handling
class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public code?: string,
  ) {
    super(message);
  }
}

// Retry with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000,
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise((r) => setTimeout(r, delay * Math.pow(2, i)));
    }
  }
  throw new Error('Max retries exceeded');
}
```
</async_patterns>

<authentication>
**JWT Authentication:**

```typescript
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET!;

// Generate tokens
function generateTokens(userId: string) {
  const accessToken = jwt.sign({ sub: userId }, JWT_SECRET, {
    expiresIn: '15m',
  });

  const refreshToken = jwt.sign({ sub: userId, type: 'refresh' }, JWT_SECRET, {
    expiresIn: '7d',
  });

  return { accessToken, refreshToken };
}

// Auth middleware
function authenticate(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const payload = jwt.verify(token, JWT_SECRET) as { sub: string };
    req.userId = payload.sub;
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Passport.js alternative for OAuth
import passport from 'passport';
import { Strategy as GoogleStrategy } from 'passport-google-oauth20';

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID!,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
  callbackURL: '/auth/google/callback',
}, async (accessToken, refreshToken, profile, done) => {
  const user = await userService.findOrCreateFromGoogle(profile);
  done(null, user);
}));
```
</authentication>

<queues>
**Background Jobs (BullMQ):**

```typescript
import { Queue, Worker, Job } from 'bullmq';
import Redis from 'ioredis';

const connection = new Redis(process.env.REDIS_URL);

// Define queue
const emailQueue = new Queue('email', { connection });

// Add job
await emailQueue.add('welcome', {
  userId: user.id,
  email: user.email,
}, {
  delay: 5000, // 5 second delay
  attempts: 3,
  backoff: { type: 'exponential', delay: 1000 },
});

// Process jobs
const worker = new Worker('email', async (job: Job) => {
  switch (job.name) {
    case 'welcome':
      await sendWelcomeEmail(job.data.email);
      break;
    case 'reminder':
      await sendReminderEmail(job.data.email);
      break;
  }
}, { connection });

worker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
  console.error(`Job ${job?.id} failed:`, err);
});
```
</queues>

<testing>
**Testing with Jest:**

```typescript
import request from 'supertest';
import { prisma } from '../src/prisma';
import app from '../src/app';

describe('POST /api/users', () => {
  beforeEach(async () => {
    await prisma.user.deleteMany();
  });

  it('creates a user with valid data', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'test@example.com',
        password: 'password123',
      });

    expect(response.status).toBe(201);
    expect(response.body).toHaveProperty('id');
    expect(response.body.email).toBe('test@example.com');
  });

  it('returns 422 for invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'invalid-email',
        password: 'password123',
      });

    expect(response.status).toBe(422);
  });
});

// Mocking
jest.mock('../src/services/email', () => ({
  sendEmail: jest.fn().mockResolvedValue(undefined),
}));
```
</testing>

<quick_reference>
**Common Commands:**

```bash
# Prisma
npx prisma migrate dev
npx prisma migrate deploy
npx prisma db seed
npx prisma studio

# Development
npm run dev          # nodemon/tsx
npm run build        # tsc
npm run start        # production

# Testing
npm test
npm run test:watch
npm run test:coverage
```
</quick_reference>
