# Migration Guide: Monorepo with Next.js + NestJS/Fastify

This guide provides step-by-step instructions for transforming the current NestJS starter into a production-ready monorepo with Next.js frontend and NestJS/Fastify backend.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Phase 1: Monorepo Setup](#phase-1-monorepo-setup)
3. [Phase 2: Fastify Integration](#phase-2-fastify-integration)
4. [Phase 3: Next.js Frontend](#phase-3-nextjs-frontend)
5. [Phase 4: Shared Packages](#phase-4-shared-packages)
6. [Phase 5: Build System](#phase-5-build-system)
7. [Phase 6: CI/CD Setup](#phase-6-cicd-setup)
8. [Verification Steps](#verification-steps)

---

## Prerequisites

### Required Tools
- Node.js 18+ (LTS recommended)
- PNPM 8+ (recommended) or Yarn 3+
- Git
- Docker (optional, for containerization)

### Install PNPM
```bash
npm install -g pnpm
```

---

## Phase 1: Monorepo Setup

### Step 1.1: Create Workspace Structure

```bash
# Create directories
mkdir -p apps/backend
mkdir -p apps/frontend
mkdir -p packages/shared-types/src
mkdir -p packages/shared-utils/src
mkdir -p packages/shared-config
mkdir -p docker/backend
mkdir -p docker/frontend
mkdir -p docs
mkdir -p scripts
mkdir -p .github/workflows
```

### Step 1.2: Move Backend Code

```bash
# Move existing code to backend app
mv src apps/backend/
mv test apps/backend/
mv tsconfig.json apps/backend/
mv tsconfig.build.json apps/backend/
mv nest-cli.json apps/backend/
```

### Step 1.3: Create Workspace Configuration

**Root package.json:**
```json
{
  "name": "nestjs-fastify-monorepo",
  "version": "1.0.0",
  "private": true,
  "description": "Monorepo with Next.js frontend and NestJS/Fastify backend",
  "scripts": {
    "dev": "turbo run dev",
    "dev:backend": "turbo run dev --filter=backend",
    "dev:frontend": "turbo run dev --filter=frontend",
    "build": "turbo run build",
    "build:backend": "turbo run build --filter=backend",
    "build:frontend": "turbo run build --filter=frontend",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,js,jsx,json,md}\"",
    "type-check": "turbo run type-check",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "turbo": "^1.11.0",
    "prettier": "^3.2.5",
    "typescript": "^5.3.3"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  },
  "packageManager": "pnpm@8.15.0"
}
```

**pnpm-workspace.yaml:**
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

**turbo.json:**
```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "build/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "outputs": []
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "type-check": {
      "outputs": []
    },
    "clean": {
      "cache": false
    }
  }
}
```

### Step 1.4: Create Backend Package Configuration

**apps/backend/package.json:**
```json
{
  "name": "backend",
  "version": "1.0.0",
  "private": true,
  "description": "NestJS backend with Fastify",
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/jest/bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "type-check": "tsc --noEmit",
    "clean": "rm -rf dist coverage"
  },
  "dependencies": {
    "@nestjs/common": "^10.3.2",
    "@nestjs/core": "^10.3.2",
    "@nestjs/platform-fastify": "^10.3.2",
    "@nestjs/config": "^3.1.1",
    "@nestjs/swagger": "^7.2.0",
    "@fastify/helmet": "^11.1.1",
    "@fastify/cors": "^9.0.1",
    "@fastify/compress": "^7.0.0",
    "class-validator": "^0.14.1",
    "class-transformer": "^0.5.1",
    "reflect-metadata": "^0.2.1",
    "rxjs": "^7.8.1"
  },
  "devDependencies": {
    "@nestjs/cli": "^10.3.1",
    "@nestjs/schematics": "^10.1.0",
    "@nestjs/testing": "^10.3.2",
    "@swc/cli": "^0.3.9",
    "@swc/core": "^1.4.0",
    "@types/jest": "^29.5.12",
    "@types/node": "^20.11.16",
    "@types/supertest": "^6.0.2",
    "@typescript-eslint/eslint-plugin": "^6.21.0",
    "@typescript-eslint/parser": "^6.21.0",
    "eslint": "^8.56.0",
    "eslint-config-prettier": "^9.1.0",
    "eslint-plugin-prettier": "^5.1.3",
    "jest": "^29.7.0",
    "prettier": "^3.2.5",
    "source-map-support": "^0.5.21",
    "supertest": "^6.3.4",
    "ts-jest": "^29.1.2",
    "ts-loader": "^9.5.1",
    "ts-node": "^10.9.2",
    "tsconfig-paths": "^4.2.0",
    "typescript": "^5.3.3"
  }
}
```

### Step 1.5: Update Root .gitignore

Add to the existing .gitignore:
```gitignore
# Monorepo
.turbo
.pnpm-store

# Build outputs
**/dist
**/.next
**/build
**/coverage

# Environment files
.env
.env.local
.env.*.local

# Logs
logs
*.log
npm-debug.log*
pnpm-debug.log*
lerna-debug.log*

# OS
.DS_Store
Thumbs.db
```

---

## Phase 2: Fastify Integration

### Step 2.1: Install Fastify Dependencies

```bash
cd apps/backend
pnpm add @nestjs/platform-fastify @fastify/helmet @fastify/cors @fastify/compress
```

### Step 2.2: Update main.ts

**apps/backend/src/main.ts:**
```typescript
import { NestFactory } from '@nestjs/core';
import {
  FastifyAdapter,
  NestFastifyApplication,
} from '@nestjs/platform-fastify';
import { ValidationPipe } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  // Create Fastify adapter with options
  const fastifyAdapter = new FastifyAdapter({
    logger: process.env.NODE_ENV === 'development',
    trustProxy: true,
  });

  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    fastifyAdapter,
  );

  // Global prefix
  app.setGlobalPrefix('api');

  // CORS configuration
  app.enableCors({
    origin: process.env.FRONTEND_URL || 'http://localhost:3000',
    credentials: true,
  });

  // Global validation pipe
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  // Helmet for security headers
  await app.register(require('@fastify/helmet'), {
    contentSecurityPolicy: {
      directives: {
        defaultSrc: [`'self'`],
        styleSrc: [`'self'`, `'unsafe-inline'`],
        imgSrc: [`'self'`, 'data:', 'validator.swagger.io'],
        scriptSrc: [`'self'`, `https: 'unsafe-inline'`],
      },
    },
  });

  // Compression
  await app.register(require('@fastify/compress'), {
    encodings: ['gzip', 'deflate'],
  });

  // Swagger documentation
  if (process.env.NODE_ENV !== 'production') {
    const config = new DocumentBuilder()
      .setTitle('API Documentation')
      .setDescription('NestJS + Fastify API')
      .setVersion('1.0')
      .addBearerAuth()
      .build();
    
    const document = SwaggerModule.createDocument(app, config);
    SwaggerModule.setup('api/docs', app, document);
  }

  const port = process.env.PORT || 3001;
  await app.listen(port, '0.0.0.0');
  
  console.log(`🚀 Application is running on: http://localhost:${port}`);
  console.log(`📚 API Documentation: http://localhost:${port}/api/docs`);
}

bootstrap();
```

### Step 2.3: Create Configuration Module

**apps/backend/src/config/configuration.ts:**
```typescript
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3001,
  nodeEnv: process.env.NODE_ENV || 'development',
  frontendUrl: process.env.FRONTEND_URL || 'http://localhost:3000',
  database: {
    url: process.env.DATABASE_URL,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1d',
  },
});
```

**apps/backend/src/app.module.ts:**
```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import configuration from './config/configuration';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [configuration],
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### Step 2.4: Update E2E Tests for Fastify

**apps/backend/test/app.e2e-spec.ts:**
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
import { AppModule } from './../src/app.module';

describe('AppController (e2e)', () => {
  let app: NestFastifyApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication<NestFastifyApplication>(
      new FastifyAdapter(),
    );
    await app.init();
    await app.getHttpAdapter().getInstance().ready();
  });

  afterAll(async () => {
    await app.close();
  });

  it('/api (GET)', () => {
    return app
      .inject({
        method: 'GET',
        url: '/api',
      })
      .then((result) => {
        expect(result.statusCode).toEqual(200);
        expect(result.payload).toEqual('Hello World!');
      });
  });
});
```

### Step 2.5: Create Environment File Template

**apps/backend/.env.example:**
```env
# Application
NODE_ENV=development
PORT=3001

# Frontend
FRONTEND_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# JWT
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRES_IN=1d

# Redis (optional)
REDIS_URL=redis://localhost:6379
```

---

## Phase 3: Next.js Frontend

### Step 3.1: Create Next.js Application

```bash
cd apps
npx create-next-app@latest frontend --typescript --tailwind --app --src-dir --import-alias "@/*"
```

Or manually:

**apps/frontend/package.json:**
```json
{
  "name": "frontend",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit",
    "clean": "rm -rf .next"
  },
  "dependencies": {
    "next": "^14.1.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.6.5"
  },
  "devDependencies": {
    "@types/node": "^20.11.16",
    "@types/react": "^18.2.48",
    "@types/react-dom": "^18.2.18",
    "autoprefixer": "^10.4.17",
    "eslint": "^8.56.0",
    "eslint-config-next": "^14.1.0",
    "postcss": "^8.4.33",
    "tailwindcss": "^3.4.1",
    "typescript": "^5.3.3"
  }
}
```

### Step 3.2: Create Basic Next.js Structure

**apps/frontend/src/app/layout.tsx:**
```typescript
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Next.js + NestJS App',
  description: 'Full-stack application with Next.js and NestJS',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

**apps/frontend/src/app/page.tsx:**
```typescript
import { getServerSideAPI } from '@/lib/api-client';

export default async function Home() {
  const message = await getServerSideAPI();

  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <h1 className="text-4xl font-bold mb-4">Welcome to Next.js + NestJS</h1>
      <p className="text-lg">Message from backend: {message}</p>
    </main>
  );
}
```

### Step 3.3: Create API Client

**apps/frontend/src/lib/api-client.ts:**
```typescript
import axios from 'axios';

const API_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3001';

// Client-side API client
export const apiClient = axios.create({
  baseURL: `${API_URL}/api`,
  withCredentials: true,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Server-side API client
export const serverApiClient = axios.create({
  baseURL: process.env.API_URL || 'http://localhost:3001/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

// Example API calls
export async function getServerSideAPI() {
  try {
    const response = await serverApiClient.get('/');
    return response.data;
  } catch (error) {
    console.error('API Error:', error);
    return 'Error fetching data';
  }
}
```

### Step 3.4: Configure Next.js

**apps/frontend/next.config.js:**
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  transpilePackages: ['@repo/shared-types', '@repo/shared-utils'],
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: `${process.env.API_URL || 'http://localhost:3001'}/api/:path*`,
      },
    ];
  },
};

module.exports = nextConfig;
```

**apps/frontend/.env.local:**
```env
NEXT_PUBLIC_API_URL=http://localhost:3001
API_URL=http://localhost:3001
```

### Step 3.5: Configure TypeScript

**apps/frontend/tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ES2021",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"],
      "@repo/shared-types": ["../../packages/shared-types/src"],
      "@repo/shared-utils": ["../../packages/shared-utils/src"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

---

## Phase 4: Shared Packages

### Step 4.1: Create Shared Types Package

**packages/shared-types/package.json:**
```json
{
  "name": "@repo/shared-types",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "type-check": "tsc --noEmit"
  },
  "devDependencies": {
    "typescript": "^5.3.3"
  }
}
```

**packages/shared-types/tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "ESNext",
    "lib": ["ES2021"],
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

**packages/shared-types/src/index.ts:**
```typescript
export * from './user';
export * from './common';
```

**packages/shared-types/src/user.ts:**
```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateUserDto {
  email: string;
  password: string;
  name: string;
}

export interface UpdateUserDto {
  email?: string;
  name?: string;
}

export interface UserResponseDto {
  id: string;
  email: string;
  name: string;
  createdAt: string;
}
```

**packages/shared-types/src/common.ts:**
```typescript
export interface PaginationParams {
  page: number;
  limit: number;
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}
```

### Step 4.2: Create Shared Utils Package

**packages/shared-utils/package.json:**
```json
{
  "name": "@repo/shared-utils",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "type-check": "tsc --noEmit",
    "test": "jest"
  },
  "devDependencies": {
    "@types/jest": "^29.5.12",
    "jest": "^29.7.0",
    "ts-jest": "^29.1.2",
    "typescript": "^5.3.3"
  }
}
```

**packages/shared-utils/src/index.ts:**
```typescript
export * from './validators';
export * from './formatters';
```

**packages/shared-utils/src/validators.ts:**
```typescript
export function isValidEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

export function isValidPassword(password: string): boolean {
  // At least 8 characters, one uppercase, one lowercase, one number
  const regex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d@$!%*?&]{8,}$/;
  return regex.test(password);
}
```

**packages/shared-utils/src/formatters.ts:**
```typescript
export function formatDate(date: Date | string): string {
  const d = typeof date === 'string' ? new Date(date) : date;
  return d.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  });
}

export function truncateString(str: string, maxLength: number): string {
  if (str.length <= maxLength) return str;
  return str.substring(0, maxLength - 3) + '...';
}
```

---

## Phase 5: Build System

### Step 5.1: Install Dependencies

```bash
# From root
pnpm install
```

### Step 5.2: Test the Setup

```bash
# Backend
cd apps/backend
pnpm install
pnpm run dev

# Frontend (in another terminal)
cd apps/frontend
pnpm install
pnpm run dev
```

### Step 5.3: Test with Turbo

```bash
# From root
pnpm dev  # Runs both apps
pnpm build  # Builds both apps
pnpm test  # Runs all tests
```

---

## Phase 6: CI/CD Setup

### Step 6.1: Create GitHub Actions Workflow

**.github/workflows/ci.yml:**
```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Lint
        run: pnpm lint
      
      - name: Type check
        run: pnpm type-check
      
      - name: Test
        run: pnpm test
      
      - name: Build
        run: pnpm build
```

---

## Verification Steps

### 1. Backend Verification

```bash
cd apps/backend

# Install and run
pnpm install
pnpm run dev

# Test endpoints
curl http://localhost:3001/api
curl http://localhost:3001/api/docs

# Run tests
pnpm test
pnpm test:e2e
```

### 2. Frontend Verification

```bash
cd apps/frontend

# Install and run
pnpm install
pnpm run dev

# Visit
open http://localhost:3000
```

### 3. Shared Packages Verification

```bash
# From root
cd packages/shared-types
pnpm type-check

cd ../shared-utils
pnpm type-check
pnpm test
```

### 4. Full Build Verification

```bash
# From root
pnpm build
```

---

## Troubleshooting

### Common Issues

**1. Port conflicts:**
- Backend: Change PORT in .env
- Frontend: Run with `PORT=3001 pnpm dev`

**2. Module resolution errors:**
- Run `pnpm install` from root
- Check tsconfig paths
- Restart TypeScript server in IDE

**3. CORS errors:**
- Check FRONTEND_URL in backend .env
- Verify CORS configuration in main.ts

**4. Build failures:**
- Clear caches: `pnpm clean`
- Delete node_modules: `rm -rf node_modules`
- Reinstall: `pnpm install`

---

## Next Steps

After successful migration:

1. **Add Authentication**: Implement JWT-based auth
2. **Database Integration**: Add TypeORM or Prisma
3. **API Routes**: Create feature modules (users, products, etc.)
4. **UI Components**: Build component library
5. **Testing**: Add comprehensive test coverage
6. **Documentation**: Update API docs and README
7. **Deployment**: Set up production deployment

---

## Additional Resources

- [NestJS Documentation](https://docs.nestjs.com/)
- [Fastify Documentation](https://www.fastify.io/)
- [Next.js Documentation](https://nextjs.org/docs)
- [Turborepo Documentation](https://turbo.build/repo/docs)
- [PNPM Workspaces](https://pnpm.io/workspaces)
