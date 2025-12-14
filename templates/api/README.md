# API Template - REST APIs & Backend Services

## 🔌 Ejemplo de uso

```typescript
// PROYECTO: API REST para sistema de reservas con autenticación JWT
// Stack: Express + TypeScript + PostgreSQL + Redis + Docker
```

## 📦 Stack Generado

- **Framework**: Express / NestJS / Fastify
- **Lenguaje**: TypeScript 5.x
- **Database**: PostgreSQL + Prisma
- **Cache**: Redis
- **Auth**: JWT + bcrypt
- **Validation**: Zod
- **Documentation**: Swagger/OpenAPI
- **Testing**: Jest + Supertest
- **Deployment**: Docker + Kubernetes

## 📁 Estructura

```
api-project/
├── src/
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── user.controller.ts
│   │   └── resource.controller.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   └── user.service.ts
│   ├── repositories/
│   │   └── user.repository.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   ├── validation.middleware.ts
│   │   └── errorHandler.middleware.ts
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   └── user.routes.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   └── helpers.ts
│   ├── types/
│   ├── config/
│   │   ├── database.ts
│   │   └── redis.ts
│   └── index.ts
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/
│   └── swagger.yaml
├── Dockerfile
├── docker-compose.yml
└── package.json
```

## 🎯 Características Incluidas

### Core Features
- RESTful API design
- JWT authentication
- Role-based access control (RBAC)
- Request validation
- Error handling
- Rate limiting
- CORS configuration
- Request logging
- Health checks
- API versioning

### Security
- Helmet.js headers
- Input sanitization
- SQL injection prevention
- XSS protection
- CSRF tokens
- Password hashing (bcrypt)
- Secure session management

### Performance
- Response caching (Redis)
- Database query optimization
- Connection pooling
- Compression (gzip/brotli)
- Pagination
- Lazy loading

## 📝 Ejemplo: Express + TypeScript API

### Main Application
```typescript
// src/index.ts
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import compression from 'compression';
import { env } from './config/env';
import { logger } from './utils/logger';
import { errorHandler } from './middleware/errorHandler';
import routes from './routes';

const app = express();

// Middleware
app.use(helmet());
app.use(cors({ origin: env.CORS_ORIGIN }));
app.use(compression());
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Logging
app.use((req, res, next) => {
  logger.info(`${req.method} ${req.path}`);
  next();
});

// Routes
app.use('/api/v1', routes);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Error handling
app.use(errorHandler);

// Start server
const PORT = env.PORT || 3000;
app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`);
});

export default app;
```

### Controller Example
```typescript
// src/controllers/user.controller.ts
import { Request, Response, NextFunction } from 'express';
import { UserService } from '@/services/user.service';
import { CreateUserDto, UpdateUserDto } from '@/types/dto';
import { asyncHandler } from '@/utils/asyncHandler';

export class UserController {
  constructor(private readonly userService: UserService) {}

  getAll = asyncHandler(async (req: Request, res: Response) => {
    const { page = 1, limit = 10 } = req.query;
    const users = await this.userService.findAll({
      page: Number(page),
      limit: Number(limit),
    });

    res.json({
      success: true,
      data: users.items,
      pagination: {
        page: users.page,
        limit: users.limit,
        total: users.total,
      },
    });
  });

  getById = asyncHandler(async (req: Request, res: Response) => {
    const { id } = req.params;
    const user = await this.userService.findById(id);

    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found',
      });
    }

    res.json({ success: true, data: user });
  });

  create = asyncHandler(async (req: Request, res: Response) => {
    const userData: CreateUserDto = req.body;
    const user = await this.userService.create(userData);

    res.status(201).json({
      success: true,
      data: user,
      message: 'User created successfully',
    });
  });

  update = asyncHandler(async (req: Request, res: Response) => {
    const { id } = req.params;
    const userData: UpdateUserDto = req.body;
    const user = await this.userService.update(id, userData);

    res.json({
      success: true,
      data: user,
      message: 'User updated successfully',
    });
  });

  delete = asyncHandler(async (req: Request, res: Response) => {
    const { id } = req.params;
    await this.userService.delete(id);

    res.json({
      success: true,
      message: 'User deleted successfully',
    });
  });
}
```

### Service Layer
```typescript
// src/services/user.service.ts
import { UserRepository } from '@/repositories/user.repository';
import { CreateUserDto, UpdateUserDto } from '@/types/dto';
import { AppError } from '@/utils/errors';
import { hashPassword } from '@/utils/crypto';
import { logger } from '@/utils/logger';

export class UserService {
  constructor(private readonly repository: UserRepository) {}

  async findAll(options: { page: number; limit: number }) {
    return this.repository.findAll(options);
  }

  async findById(id: string) {
    const user = await this.repository.findById(id);
    if (!user) {
      throw new AppError(404, 'User not found');
    }
    return user;
  }

  async findByEmail(email: string) {
    return this.repository.findByEmail(email);
  }

  async create(data: CreateUserDto) {
    // Validate email uniqueness
    const existingUser = await this.findByEmail(data.email);
    if (existingUser) {
      throw new AppError(409, 'Email already in use');
    }

    // Hash password
    const hashedPassword = await hashPassword(data.password);

    // Create user
    const user = await this.repository.create({
      ...data,
      password: hashedPassword,
    });

    logger.info(`User created: ${user.id}`);
    return user;
  }

  async update(id: string, data: UpdateUserDto) {
    const user = await this.findById(id);

    if (data.password) {
      data.password = await hashPassword(data.password);
    }

    return this.repository.update(id, data);
  }

  async delete(id: string) {
    await this.findById(id);
    return this.repository.delete(id);
  }
}
```

### Authentication Middleware
```typescript
// src/middleware/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { env } from '@/config/env';
import { AppError } from '@/utils/errors';

interface JwtPayload {
  userId: string;
  email: string;
  role: string;
}

export const authenticate = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  try {
    const token = req.headers.authorization?.replace('Bearer ', '');

    if (!token) {
      throw new AppError(401, 'Authentication required');
    }

    const decoded = jwt.verify(token, env.JWT_SECRET) as JwtPayload;
    req.user = decoded;

    next();
  } catch (error) {
    next(new AppError(401, 'Invalid or expired token'));
  }
};

export const authorize = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return next(new AppError(403, 'Insufficient permissions'));
    }
    next();
  };
};
```

### Request Validation
```typescript
// src/middleware/validation.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { z } from 'zod';
import { AppError } from '@/utils/errors';

export const validate = (schema: z.ZodSchema) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        const errors = error.errors.map((err) => ({
          path: err.path.join('.'),
          message: err.message,
        }));
        next(new AppError(400, 'Validation failed', errors));
      } else {
        next(error);
      }
    }
  };
};

// Example schema
export const createUserSchema = z.object({
  body: z.object({
    email: z.string().email(),
    password: z.string().min(8),
    name: z.string().min(2),
    role: z.enum(['user', 'admin']).optional(),
  }),
});
```

### Prisma Schema
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
  password  String
  name      String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
}

enum Role {
  USER
  ADMIN
}
```

## 📦 Dependencies

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "typescript": "^5.0.0",
    "@prisma/client": "^5.5.0",
    "prisma": "^5.5.0",
    "redis": "^4.6.0",
    "jsonwebtoken": "^9.0.0",
    "bcrypt": "^5.1.0",
    "zod": "^3.22.0",
    "helmet": "^7.1.0",
    "cors": "^2.8.5",
    "compression": "^1.7.4",
    "winston": "^3.11.0",
    "dotenv": "^16.3.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.0",
    "@types/node": "^20.8.0",
    "@types/bcrypt": "^5.0.0",
    "@types/jsonwebtoken": "^9.0.0",
    "jest": "^29.7.0",
    "supertest": "^6.3.0",
    "ts-node": "^10.9.0",
    "nodemon": "^3.0.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0"
  }
}
```

## 🚀 Scripts

```json
{
  "scripts": {
    "dev": "nodemon --exec ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write src/**/*.ts",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:studio": "prisma studio"
  }
}
```

## 🧪 Tests

```typescript
// tests/integration/user.test.ts
import request from 'supertest';
import app from '@/index';
import { prisma } from '@/config/database';

describe('User API', () => {
  beforeEach(async () => {
    await prisma.user.deleteMany();
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  describe('POST /api/v1/users', () => {
    it('should create a new user', async () => {
      const userData = {
        email: 'test@test.com',
        password: 'password123',
        name: 'Test User',
      };

      const response = await request(app)
        .post('/api/v1/users')
        .send(userData)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data.email).toBe(userData.email);
      expect(response.body.data.password).toBeUndefined();
    });

    it('should return 400 for invalid data', async () => {
      const response = await request(app)
        .post('/api/v1/users')
        .send({ email: 'invalid' })
        .expect(400);

      expect(response.body.success).toBe(false);
    });
  });
});
```

## 🐳 Docker

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
COPY prisma ./prisma/
RUN npm ci
RUN npx prisma generate
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
COPY package*.json ./
EXPOSE 3000
CMD ["sh", "-c", "npx prisma migrate deploy && node dist/index.js"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  api:
    build: .
    restart: unless-stopped
    ports:
      - "3000:3000"
    env_file: .env
    depends_on:
      - postgres
      - redis
    networks:
      - api-network

  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: apidb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - api-network

  redis:
    image: redis:7-alpine
    networks:
      - api-network

volumes:
  postgres_data:

networks:
  api-network:
```

## 📚 API Documentation (Swagger)

```typescript
// src/config/swagger.ts
import swaggerJsdoc from 'swagger-jsdoc';

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'API Documentation',
      version: '1.0.0',
      description: 'REST API documentation',
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server',
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
    },
  },
  apis: ['./src/routes/*.ts'],
};

export const swaggerSpec = swaggerJsdoc(options);
```

## ✅ Checklist de Funcionalidades

- [x] RESTful endpoints
- [x] JWT authentication
- [x] RBAC (Role-based access)
- [x] Input validation (Zod)
- [x] Error handling centralizado
- [x] Request logging
- [x] Rate limiting
- [x] CORS configurado
- [x] Helmet security headers
- [x] Database migrations
- [x] Redis caching
- [x] API documentation (Swagger)
- [x] Unit tests
- [x] Integration tests
- [x] Docker containerized
- [x] CI/CD pipeline
