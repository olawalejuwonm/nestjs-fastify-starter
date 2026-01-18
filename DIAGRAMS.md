# Architecture Diagrams

Visual representations of the monorepo architecture and data flow.

## Table of Contents
- [Monorepo Structure](#monorepo-structure)
- [Application Architecture](#application-architecture)
- [Request Flow](#request-flow)
- [Development Workflow](#development-workflow)
- [Deployment Architecture](#deployment-architecture)

---

## Monorepo Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                      MONOREPO ROOT                              │
│                    pnpm-workspace.yaml                          │
│                       turbo.json                                │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    apps/     │     │  packages/   │     │   .github/   │
└──────────────┘     └──────────────┘     └──────────────┘
        │                     │                     │
        │                     │                     │
   ┌────┴────┐           ┌────┴────┐          ┌────┴────┐
   │         │           │         │          │         │
   ▼         ▼           ▼         ▼          ▼         ▼
backend   frontend  shared-   shared-   workflows/ security/
                    types     utils
```

### Apps Directory
```
apps/
├── backend/                    # NestJS + Fastify
│   ├── src/
│   │   ├── modules/           # Feature modules
│   │   │   ├── users/
│   │   │   ├── auth/
│   │   │   └── products/
│   │   ├── common/            # Shared backend code
│   │   │   ├── filters/
│   │   │   ├── guards/
│   │   │   ├── interceptors/
│   │   │   └── pipes/
│   │   ├── config/
│   │   ├── app.module.ts
│   │   └── main.ts
│   └── test/
│
└── frontend/                   # Next.js
    ├── src/
    │   ├── app/               # Next.js 13+ routes
    │   ├── components/        # React components
    │   ├── lib/               # Utilities
    │   └── hooks/             # Custom hooks
    └── public/
```

### Packages Directory
```
packages/
├── shared-types/              # TypeScript types
│   └── src/
│       ├── dtos/
│       ├── entities/
│       └── enums/
│
├── shared-utils/              # Common utilities
│   └── src/
│       ├── validators/
│       ├── formatters/
│       └── helpers/
│
└── shared-config/             # Shared configs
    ├── eslint-config/
    ├── tsconfig/
    └── prettier-config/
```

---

## Application Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                           │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    Next.js Frontend                      │  │
│  │                                                           │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │  │
│  │  │   App    │  │Components│  │   Hooks  │              │  │
│  │  │  Router  │  │   (UI)   │  │ (State)  │              │  │
│  │  └──────────┘  └──────────┘  └──────────┘              │  │
│  │                        │                                 │  │
│  │                        ▼                                 │  │
│  │                  ┌──────────┐                           │  │
│  │                  │   API    │                           │  │
│  │                  │  Client  │                           │  │
│  │                  └──────────┘                           │  │
│  └────────────────────────┬─────────────────────────────────┘  │
└─────────────────────────────┼──────────────────────────────────┘
                              │
                              │ HTTP/HTTPS
                              │ REST API
                              │
┌─────────────────────────────┼──────────────────────────────────┐
│                          API LAYER                              │
│                              │                                  │
│  ┌───────────────────────────▼──────────────────────────────┐  │
│  │               NestJS + Fastify Backend                   │  │
│  │                                                           │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │  │
│  │  │Controllers│→ │ Services │→ │Repository│              │  │
│  │  └──────────┘  └──────────┘  └──────────┘              │  │
│  │       │              │              │                    │  │
│  │       ├─ Guards      ├─ Business   └─ Data Access       │  │
│  │       ├─ Interceptors├─ Logic                           │  │
│  │       └─ Pipes       └─ Validation                      │  │
│  │                                                           │  │
│  └───────────────────────────┬───────────────────────────────┘  │
└─────────────────────────────┼──────────────────────────────────┘
                              │
                    ┌─────────┼─────────┐
                    │         │         │
                    ▼         ▼         ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  PostgreSQL  │  │    Redis     │  │     S3       │        │
│  │  (Primary DB)│  │   (Cache)    │  │   (Files)    │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

### Shared Packages Integration

```
┌─────────────────┐              ┌─────────────────┐
│                 │              │                 │
│   Frontend      │              │    Backend      │
│   (Next.js)     │              │  (NestJS)       │
│                 │              │                 │
└────────┬────────┘              └────────┬────────┘
         │                                │
         │  Import                  Import │
         │  @repo/shared-types             │
         │  @repo/shared-utils             │
         │                                │
         └────────┬───────────────┬────────┘
                  │               │
                  ▼               ▼
         ┌─────────────────────────────┐
         │   Shared Packages           │
         │                             │
         │  ┌────────────────────┐    │
         │  │  shared-types      │    │
         │  │  - User            │    │
         │  │  - CreateUserDto   │    │
         │  │  - ApiResponse     │    │
         │  └────────────────────┘    │
         │                             │
         │  ┌────────────────────┐    │
         │  │  shared-utils      │    │
         │  │  - isValidEmail    │    │
         │  │  - formatDate      │    │
         │  │  - sanitizeInput   │    │
         │  └────────────────────┘    │
         └─────────────────────────────┘
```

---

## Request Flow

### Frontend to Backend Request Flow

```
┌─────────────┐
│   Browser   │
└──────┬──────┘
       │ 1. User Action
       ▼
┌──────────────────────┐
│  Next.js Component   │
│  (Client/Server)     │
└──────┬───────────────┘
       │ 2. API Call
       ▼
┌──────────────────────┐
│    API Client        │
│   (axios/fetch)      │
└──────┬───────────────┘
       │ 3. HTTP Request
       │    (POST /api/users)
       ▼
┌──────────────────────┐
│  Fastify Server      │
│  (Port 3001)         │
└──────┬───────────────┘
       │ 4. Route to Controller
       ▼
┌──────────────────────┐
│  NestJS Middleware   │
│  - CORS              │
│  - Authentication    │
│  - Logging           │
└──────┬───────────────┘
       │ 5. Validation
       ▼
┌──────────────────────┐
│     Controller       │
│  @Post('users')      │
└──────┬───────────────┘
       │ 6. Delegate to Service
       ▼
┌──────────────────────┐
│      Service         │
│  (Business Logic)    │
└──────┬───────────────┘
       │ 7. Data Access
       ▼
┌──────────────────────┐
│    Repository        │
│  (Database Query)    │
└──────┬───────────────┘
       │ 8. Query Database
       ▼
┌──────────────────────┐
│    PostgreSQL        │
└──────┬───────────────┘
       │ 9. Return Data
       ▼
┌──────────────────────┐
│    Repository        │
└──────┬───────────────┘
       │ 10. Transform Data
       ▼
┌──────────────────────┐
│      Service         │
└──────┬───────────────┘
       │ 11. Return DTO
       ▼
┌──────────────────────┐
│     Controller       │
└──────┬───────────────┘
       │ 12. HTTP Response
       ▼
┌──────────────────────┐
│  Fastify Server      │
└──────┬───────────────┘
       │ 13. Send to Client
       ▼
┌──────────────────────┐
│    API Client        │
└──────┬───────────────┘
       │ 14. Update State
       ▼
┌──────────────────────┐
│  Next.js Component   │
└──────┬───────────────┘
       │ 15. Re-render UI
       ▼
┌──────────────┐
│   Browser    │
└──────────────┘
```

### Authentication Flow

```
┌─────────┐
│ User    │
└────┬────┘
     │ 1. Login Request
     ▼
┌──────────────┐
│  Frontend    │
└────┬─────────┘
     │ 2. POST /auth/login
     │    { email, password }
     ▼
┌──────────────────────┐
│  Auth Controller     │
└────┬─────────────────┘
     │ 3. Validate
     ▼
┌──────────────────────┐
│   Auth Service       │
└────┬─────────────────┘
     │ 4. Check DB
     ▼
┌──────────────────────┐
│   Users Repository   │
└────┬─────────────────┘
     │ 5. Return User
     ▼
┌──────────────────────┐
│   Auth Service       │
│ - Verify password    │
│ - Generate JWT       │
└────┬─────────────────┘
     │ 6. Return tokens
     │    { accessToken, refreshToken }
     ▼
┌──────────────────────┐
│  Auth Controller     │
└────┬─────────────────┘
     │ 7. HTTP Response
     ▼
┌──────────────────────┐
│  Frontend            │
│ - Store tokens       │
│ - Redirect to dashboard
└────┬─────────────────┘
     │ 8. Authenticated
     ▼
┌─────────┐
│ User    │
└─────────┘
```

---

## Development Workflow

### Local Development Setup

```
┌──────────────────────────────────────────────────────────┐
│                    Developer Machine                      │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │  1. Clone Repository                               │ │
│  │     git clone <repo-url>                           │ │
│  └────────────────────────────────────────────────────┘ │
│                            ↓                            │
│  ┌────────────────────────────────────────────────────┐ │
│  │  2. Install Dependencies                           │ │
│  │     pnpm install                                   │ │
│  └────────────────────────────────────────────────────┘ │
│                            ↓                            │
│  ┌────────────────────────────────────────────────────┐ │
│  │  3. Start Services                                 │ │
│  │     docker-compose up -d                           │ │
│  │     (PostgreSQL, Redis)                            │ │
│  └────────────────────────────────────────────────────┘ │
│                            ↓                            │
│  ┌────────────────────────────────────────────────────┐ │
│  │  4. Run Development Servers                        │ │
│  │     pnpm dev                                       │ │
│  │                                                     │ │
│  │     ┌─────────────┐      ┌─────────────┐         │ │
│  │     │  Backend    │      │  Frontend   │         │ │
│  │     │  :3001      │      │  :3000      │         │ │
│  │     └─────────────┘      └─────────────┘         │ │
│  └────────────────────────────────────────────────────┘ │
│                            ↓                            │
│  ┌────────────────────────────────────────────────────┐ │
│  │  5. Make Changes                                   │ │
│  │     - Hot reload enabled                           │ │
│  │     - Type checking active                         │ │
│  │     - Linting on save                              │ │
│  └────────────────────────────────────────────────────┘ │
│                            ↓                            │
│  ┌────────────────────────────────────────────────────┐ │
│  │  6. Run Tests                                      │ │
│  │     pnpm test                                      │ │
│  └────────────────────────────────────────────────────┘ │
│                            ↓                            │
│  ┌────────────────────────────────────────────────────┐ │
│  │  7. Commit Changes                                 │ │
│  │     git commit                                     │ │
│  │     - Pre-commit hooks run                         │ │
│  │     - Linting                                      │ │
│  │     - Type checking                                │ │
│  └────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

### Build Pipeline (Turborepo)

```
┌────────────────────────────────────────────────────────┐
│                    Turborepo Build                     │
└────────────────────────────────────────────────────────┘
                           │
                           │ pnpm build
                           ▼
┌────────────────────────────────────────────────────────┐
│              Analyze Dependency Graph                  │
│                                                        │
│     shared-types → shared-utils → backend              │
│                                  → frontend            │
└────────────────────────────────────────────────────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
    │shared-types │  │shared-utils │  │   backend   │
    │             │  │             │  │             │
    │  build      │→ │  build      │→ │  build      │
    │  (cached)   │  │  (cached)   │  │             │
    └─────────────┘  └─────────────┘  └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  frontend   │
                    │             │
                    │  build      │
                    └─────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                    Build Complete                      │
│                                                        │
│  - backend/dist/                                       │
│  - frontend/.next/                                     │
└────────────────────────────────────────────────────────┘
```

---

## Deployment Architecture

### Production Architecture

```
┌────────────────────────────────────────────────────────────┐
│                         Internet                           │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│                    CDN (CloudFlare)                        │
│                   Static Assets, Images                    │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│                  Load Balancer / API Gateway               │
└─────────┬──────────────────────────────────┬───────────────┘
          │                                  │
          │ /                                │ /api/*
          ▼                                  ▼
┌───────────────────────┐        ┌───────────────────────┐
│   Frontend Cluster    │        │   Backend Cluster     │
│                       │        │                       │
│  ┌────────────────┐   │        │  ┌────────────────┐   │
│  │  Next.js       │   │        │  │  NestJS +      │   │
│  │  Instance 1    │   │        │  │  Fastify       │   │
│  │  (Vercel)      │   │        │  │  Instance 1    │   │
│  └────────────────┘   │        │  │  (Container)   │   │
│                       │        │  └────────────────┘   │
│  ┌────────────────┐   │        │                       │
│  │  Next.js       │   │        │  ┌────────────────┐   │
│  │  Instance 2    │   │        │  │  NestJS +      │   │
│  │  (Vercel)      │   │        │  │  Fastify       │   │
│  └────────────────┘   │        │  │  Instance 2    │   │
│                       │        │  │  (Container)   │   │
└───────────────────────┘        │  └────────────────┘   │
                                 │                       │
                                 │  ┌────────────────┐   │
                                 │  │  NestJS +      │   │
                                 │  │  Fastify       │   │
                                 │  │  Instance N    │   │
                                 │  │  (Container)   │   │
                                 │  └────────────────┘   │
                                 └───────┬───────────────┘
                                         │
                        ┌────────────────┼────────────────┐
                        │                │                │
                        ▼                ▼                ▼
            ┌───────────────┐  ┌───────────────┐  ┌───────────────┐
            │  PostgreSQL   │  │     Redis     │  │      S3       │
            │   (Primary)   │  │    (Cache)    │  │   (Storage)   │
            │               │  │               │  │               │
            │  ┌─────────┐  │  │  ┌─────────┐  │  │               │
            │  │Replica 1│  │  │  │Replica 1│  │  │               │
            │  └─────────┘  │  │  └─────────┘  │  │               │
            └───────────────┘  └───────────────┘  └───────────────┘
```

### CI/CD Pipeline

```
┌─────────────┐
│ Git Push    │
└──────┬──────┘
       │
       ▼
┌────────────────────────────────────────┐
│       GitHub Actions Triggered          │
└────────────────────────────────────────┘
       │
       ├─────────────────┬─────────────────┬─────────────────┐
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
┌──────────┐      ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Lint    │      │   Test   │     │  Build   │     │ Security │
│          │      │          │     │          │     │   Scan   │
│ ESLint   │      │  Jest    │     │ Turbo    │     │ CodeQL   │
│ Prettier │      │  E2E     │     │  Build   │     │ Snyk     │
└──────────┘      └──────────┘     └──────────┘     └──────────┘
       │                 │                 │                 │
       └─────────────────┴─────────────────┴─────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │  All Checks Pass │
                    └──────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
          ┌──────────────────┐  ┌──────────────────┐
          │ Deploy Backend   │  │ Deploy Frontend  │
          │ (Docker → K8s)   │  │  (Vercel)        │
          └──────────────────┘  └──────────────────┘
                    │                   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │  Run Smoke Tests │
                    └──────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │  Deployment Done │
                    └──────────────────┘
```

### Container Architecture (Docker)

```
┌───────────────────────────────────────────────────────────┐
│                     Docker Compose                        │
└───────────────────────────────────────────────────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
    │   Backend   │  │  Frontend   │  │   Services  │
    │  Container  │  │  Container  │  │  Container  │
    └─────────────┘  └─────────────┘  └─────────────┘
          │                  │                │
          │                  │                │
          ▼                  ▼                ▼
    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
    │  Node:18    │  │  Node:18    │  │ PostgreSQL  │
    │  Alpine     │  │  Alpine     │  │   :15       │
    │             │  │             │  ├─────────────┤
    │  NestJS +   │  │  Next.js    │  │   Redis     │
    │  Fastify    │  │             │  │    :7       │
    └─────────────┘  └─────────────┘  └─────────────┘
    Port: 3001       Port: 3000       Ports: 5432,
                                            6379
```

---

## Data Flow Patterns

### CRUD Operations

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│          │     │          │     │          │     │          │
│  Client  │────▶│Controller│────▶│ Service  │────▶│Repository│
│          │     │          │     │          │     │          │
└──────────┘     └──────────┘     └──────────┘     └────┬─────┘
     ▲                                                    │
     │                                                    ▼
     │                                             ┌──────────┐
     │                                             │ Database │
     │                                             └────┬─────┘
     │                                                  │
     └──────────────────────────────────────────────────┘
                    Response Flow
```

### Caching Strategy

```
┌─────────┐
│ Request │
└────┬────┘
     │
     ▼
┌─────────────┐    Cache Hit?
│Check Cache  │─────Yes─────▶ Return from Cache
│  (Redis)    │
└─────────────┘
     │ No
     ▼
┌─────────────┐
│Query Database│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Store in    │
│   Cache     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Return Result│
└─────────────┘
```

---

These diagrams provide a visual understanding of how the monorepo is structured, how applications communicate, and how data flows through the system. Use them as reference when implementing the architecture described in ARCHITECTURE.md.
