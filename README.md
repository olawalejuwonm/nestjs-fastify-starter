# NestJS + Fastify Starter (Monorepo Ready)

<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

## Description

A production-ready starter repository for building full-stack applications with [NestJS](https://github.com/nestjs/nest) and Fastify. This repository includes comprehensive documentation for transforming it into a monorepo structure with Next.js frontend and NestJS/Fastify backend.

## 📚 Documentation

This repository includes extensive documentation to help you build production-ready applications:

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Comprehensive architecture analysis and best practices for monorepo projects
- **[MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md)** - Step-by-step guide to transform this into a full monorepo
- **[BEST_PRACTICES_CHECKLIST.md](./BEST_PRACTICES_CHECKLIST.md)** - Complete checklist of best practices for Next.js + NestJS projects

## 🚀 Current Setup

This starter currently includes:
- NestJS 10.3.2 with TypeScript
- Express HTTP adapter (can be migrated to Fastify)
- Jest for testing
- ESLint + Prettier for code quality
- Basic project structure

## 🎯 Recommended Architecture

For production applications, we recommend:
- **Monorepo Structure** - Using PNPM workspaces + Turborepo
- **Fastify** - High-performance HTTP adapter (2-3x faster than Express)
- **Next.js Frontend** - Modern React framework with App Router
- **Shared Packages** - Type-safe shared code between frontend and backend

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed architecture documentation.

## Installation

```bash
$ npm install
```

## Running the app

```bash
# development
$ npm run start

# watch mode
$ npm run start:dev

# production mode
$ npm run start:prod
```

## Test

```bash
# unit tests
$ npm run test

# e2e tests
$ npm run test:e2e

# test coverage
$ npm run test:cov
```

## 📖 Getting Started

### Quick Start (Current Setup)

```bash
# Install dependencies
npm install

# Development mode
npm run start:dev

# Production build
npm run build
npm run start:prod
```

### Migrate to Monorepo

Follow the [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) for step-by-step instructions to:
1. Set up monorepo structure
2. Add Fastify adapter
3. Create Next.js frontend
4. Add shared packages
5. Configure build system (Turborepo)
6. Set up CI/CD

## 🏗️ Architecture Highlights

### Why Monorepo?
- **Type Safety** - Shared types between frontend and backend
- **Code Reuse** - Common utilities and configurations
- **Faster Development** - Synchronized changes across applications
- **Better DX** - Single repository, easier dependency management

### Why Fastify?
- **Performance** - 2-3x faster than Express
- **Schema Validation** - Built-in JSON schema validation
- **TypeScript** - Better TypeScript integration
- **Modern** - Active development and rich ecosystem

### Why Next.js?
- **Performance** - Built-in optimizations
- **SEO** - Server-side rendering and static generation
- **Developer Experience** - File-based routing, hot reload
- **Production Ready** - Used by major companies

## 📋 Best Practices

Key best practices covered in our documentation:

### Backend (NestJS + Fastify)
- ✅ Modular architecture with feature-based modules
- ✅ Configuration management with validation
- ✅ Swagger/OpenAPI documentation
- ✅ Global exception handling
- ✅ Request validation with class-validator
- ✅ Security best practices (Helmet, CORS, rate limiting)

### Frontend (Next.js)
- ✅ App Router with Server Components
- ✅ Type-safe API client
- ✅ Optimized images and code splitting
- ✅ SEO optimization
- ✅ Error boundaries and loading states

### Shared Code
- ✅ Shared TypeScript types
- ✅ Common utilities and validators
- ✅ Shared configuration files
- ✅ Consistent linting and formatting

See [BEST_PRACTICES_CHECKLIST.md](./BEST_PRACTICES_CHECKLIST.md) for complete checklist.

## 🧪 Testing

```bash
# Unit tests
npm run test

# E2E tests
npm run test:e2e

# Test coverage
npm run test:cov
```

## 🔍 Code Quality

```bash
# Linting
npm run lint

# Format code
npm run format
```

## 📦 Project Structure (After Migration)

```
├── apps/
│   ├── backend/          # NestJS + Fastify backend
│   └── frontend/         # Next.js frontend
├── packages/
│   ├── shared-types/     # Shared TypeScript types
│   ├── shared-utils/     # Common utilities
│   └── shared-config/    # Shared configurations
├── docs/                 # Documentation
├── scripts/              # Build and utility scripts
└── .github/
    └── workflows/        # CI/CD pipelines
```

## 🚢 Deployment

### Backend Deployment Options
- **Docker** - Containerized deployment
- **Cloud Run** - Serverless containers
- **AWS ECS/EKS** - Container orchestration
- **Kubernetes** - Self-hosted clusters

### Frontend Deployment Options
- **Vercel** - Optimal for Next.js
- **Netlify** - Alternative serverless platform
- **CloudFlare Pages** - Edge deployment
- **Docker** - Self-hosted option

## 🔐 Security

Security considerations covered in documentation:
- Input validation and sanitization
- SQL injection prevention
- XSS protection
- CSRF tokens
- Rate limiting
- Helmet security headers
- Proper CORS configuration
- Secrets management

## 📈 Performance

Performance optimizations included:
- Fastify's high performance
- Caching strategies
- Database query optimization
- Connection pooling
- Next.js automatic optimizations
- Image optimization
- Code splitting

## 🤝 Contributing

Contributions are welcome! Please read our contributing guidelines before submitting PRs.

## 📄 License

This project is [MIT licensed](LICENSE).

## 🙏 Acknowledgments

- [NestJS](https://nestjs.com/) - A progressive Node.js framework
- [Fastify](https://www.fastify.io/) - Fast and low overhead web framework
- [Next.js](https://nextjs.org/) - The React Framework for Production
- [Turborepo](https://turbo.build/) - High-performance build system

## 📞 Support

For questions and support:
- Review the [ARCHITECTURE.md](./ARCHITECTURE.md) documentation
- Check the [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) for setup help
- Open an issue for bugs or feature requests

---

**Ready to build something amazing? Start by reading our [Architecture Guide](./ARCHITECTURE.md)!** 🎉
