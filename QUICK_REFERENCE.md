# Quick Reference Guide

Quick reference for common patterns and commands in a Next.js + NestJS/Fastify monorepo.

## Table of Contents
- [Common Commands](#common-commands)
- [Backend Patterns](#backend-patterns)
- [Frontend Patterns](#frontend-patterns)
- [Shared Code](#shared-code)
- [Environment Variables](#environment-variables)
- [Troubleshooting](#troubleshooting)

---

## Common Commands

### Development
```bash
# Start all apps
pnpm dev

# Start backend only
pnpm dev:backend

# Start frontend only
pnpm dev:frontend

# Install dependencies
pnpm install

# Clean all builds
pnpm clean
```

### Building
```bash
# Build all apps
pnpm build

# Build backend only
pnpm build:backend

# Build frontend only
pnpm build:frontend
```

### Testing
```bash
# Run all tests
pnpm test

# Run backend tests
pnpm --filter backend test

# Run frontend tests
pnpm --filter frontend test

# Run with coverage
pnpm test:cov
```

### Code Quality
```bash
# Lint all code
pnpm lint

# Format all code
pnpm format

# Type check
pnpm type-check
```

---

## Backend Patterns

### Creating a Module

```typescript
// users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Export if used by other modules
})
export class UsersModule {}
```

### Creating a Controller

```typescript
// users/users.controller.ts
import { Controller, Get, Post, Body, Param, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiBearerAuth } from '@nestjs/swagger';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';

@ApiTags('users')
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @ApiOperation({ summary: 'Create a new user' })
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Get(':id')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get user by ID' })
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}
```

### Creating a Service

```typescript
// users/users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(createUserDto);
    return await this.usersRepository.save(user);
  }

  async findOne(id: string): Promise<User> {
    const user = await this.usersRepository.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User #${id} not found`);
    }
    return user;
  }
}
```

### Creating a DTO

```typescript
// users/dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, IsNotEmpty } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail()
  @IsNotEmpty()
  email: string;

  @ApiProperty({ example: 'John Doe' })
  @IsString()
  @IsNotEmpty()
  name: string;

  @ApiProperty({ example: 'securePassword123' })
  @IsString()
  @MinLength(8)
  password: string;
}
```

### Creating a Guard

```typescript
// auth/guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }
}
```

### Exception Filter

```typescript
// common/filters/http-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { FastifyReply } from 'fastify';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<FastifyReply>();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const message =
      exception instanceof HttpException
        ? exception.message
        : 'Internal server error';

    response.status(status).send({
      statusCode: status,
      message,
      timestamp: new Date().toISOString(),
    });
  }
}
```

### Interceptor

```typescript
// common/interceptors/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const method = request.method;
    const url = request.url;
    const now = Date.now();

    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        const delay = Date.now() - now;
        this.logger.log(
          `${method} ${url} ${response.statusCode} - ${delay}ms`,
        );
      }),
    );
  }
}
```

---

## Frontend Patterns

### Server Component (Data Fetching)

```typescript
// app/users/page.tsx
import { getUsers } from '@/lib/api/users';

export default async function UsersPage() {
  const users = await getUsers();

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Client Component

```typescript
// components/UserForm.tsx
'use client';

import { useState } from 'react';
import { createUser } from '@/lib/api/users';

export function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await createUser({ name, email });
      // Handle success
    } catch (error) {
      // Handle error
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Create User</button>
    </form>
  );
}
```

### API Client

```typescript
// lib/api/users.ts
import { apiClient, serverApiClient } from '@/lib/api-client';
import type { User, CreateUserDto } from '@repo/shared-types';

// Client-side usage
export async function createUser(data: CreateUserDto): Promise<User> {
  const response = await apiClient.post<User>('/users', data);
  return response.data;
}

// Server-side usage
export async function getUsers(): Promise<User[]> {
  const response = await serverApiClient.get<User[]>('/users');
  return response.data;
}

export async function getUser(id: string): Promise<User> {
  const response = await serverApiClient.get<User>(`/users/${id}`);
  return response.data;
}
```

### Custom Hook

```typescript
// hooks/useUser.ts
'use client';

import { useState, useEffect } from 'react';
import type { User } from '@repo/shared-types';
import { getUser } from '@/lib/api/users';

export function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    getUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);

  return { user, loading, error };
}
```

### Loading State

```typescript
// app/users/loading.tsx
export default function Loading() {
  return <div>Loading users...</div>;
}
```

### Error Boundary

```typescript
// app/users/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

---

## Shared Code

### Shared Types

```typescript
// packages/shared-types/src/user.ts
export interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateUserDto {
  email: string;
  name: string;
  password: string;
}
```

### Shared Utilities

```typescript
// packages/shared-utils/src/validators.ts
export function isValidEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}
```

### Using Shared Code

```typescript
// Backend
import { User } from '@repo/shared-types';
import { isValidEmail } from '@repo/shared-utils';

// Frontend
import type { User, CreateUserDto } from '@repo/shared-types';
import { formatDate } from '@repo/shared-utils';
```

---

## Environment Variables

### Backend (.env)
```env
# Application
NODE_ENV=development
PORT=3001

# Frontend URL
FRONTEND_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=1d

# Redis
REDIS_URL=redis://localhost:6379
```

### Frontend (.env.local)
```env
# Public (accessible in browser)
NEXT_PUBLIC_API_URL=http://localhost:3001

# Server-side only
API_URL=http://localhost:3001
DATABASE_URL=postgresql://...
```

### Accessing Environment Variables

```typescript
// Backend
const port = process.env.PORT || 3001;
const dbUrl = this.configService.get<string>('DATABASE_URL');

// Frontend - Client Component
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Frontend - Server Component
const apiUrl = process.env.API_URL;
```

---

## Troubleshooting

### Port Already in Use
```bash
# Find process using port
lsof -i :3001

# Kill process
kill -9 <PID>
```

### Module Not Found
```bash
# Clean install
rm -rf node_modules
pnpm install

# Clear cache
pnpm store prune
```

### TypeScript Errors
```bash
# Restart TypeScript server in VS Code
Cmd/Ctrl + Shift + P -> "TypeScript: Restart TS Server"

# Clean build
pnpm clean
pnpm build
```

### Docker Issues
```bash
# Rebuild without cache
docker-compose build --no-cache

# Remove all containers and volumes
docker-compose down -v

# View logs
docker-compose logs -f
```

### Database Connection Issues
```bash
# Check if database is running
docker-compose ps

# View database logs
docker-compose logs postgres

# Connect to database
docker-compose exec postgres psql -U user -d dbname
```

### CORS Issues
```typescript
// Ensure CORS is configured in main.ts
app.enableCors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true,
});
```

---

## Quick Tips

### Performance
- Use Server Components by default
- Implement caching for frequently accessed data
- Use database indexes
- Enable compression
- Optimize images

### Security
- Validate all inputs
- Use environment variables for secrets
- Implement rate limiting
- Use HTTPS in production
- Keep dependencies updated

### Testing
- Test business logic thoroughly
- Mock external dependencies
- Use test database for integration tests
- Implement E2E tests for critical flows

### Monitoring
- Set up error tracking (Sentry)
- Monitor application performance
- Track API response times
- Set up health checks
- Monitor database performance

---

For more detailed information, see:
- [ARCHITECTURE.md](./ARCHITECTURE.md) - Architecture and best practices
- [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) - Setup instructions
- [BEST_PRACTICES_CHECKLIST.md](./BEST_PRACTICES_CHECKLIST.md) - Complete checklist
