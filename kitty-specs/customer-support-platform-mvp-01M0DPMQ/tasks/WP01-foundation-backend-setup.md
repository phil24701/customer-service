---
work_package_id: WP01
title: 'Foundation: Backend Project Setup & Infrastructure'
dependencies: []
requirement_refs:
- C-001
- C-002
- C-003
- C-004
- C-005
- C-006
- C-007
- C-008
- C-009
- C-010
- C-011
- C-012
- C-013
- C-014
- C-015
- C-016
- NFR-001
- NFR-003
- NFR-005
- NFR-006
- NFR-007
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T001
- T002
- T003
- T004
- T005
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/
create_intent:
- backend/package.json
- backend/tsconfig.json
- backend/nest-cli.json
- backend/.eslintrc.js
- backend/.prettierrc
- backend/prisma/schema.prisma
- backend/src/main.ts
- backend/src/app.module.ts
- docker-compose.yml
- docker-compose.prod.yml
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/package.json
- backend/tsconfig.json
- backend/nest-cli.json
- backend/.eslintrc.js
- backend/.prettierrc
- backend/prisma/schema.prisma
- backend/src/main.ts
- backend/src/app.module.ts
- backend/src/config/**
- backend/src/database/**
- backend/src/observability/**
- docker-compose.yml
- docker-compose.prod.yml
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

Before reading any other context, load your assigned agent profile:

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

This will apply the implementer persona with its identity, governance scope, boundaries, and initialization declarations. Do not proceed until the profile is loaded.

---

## Objective

Initialize the NestJS backend project with all foundational infrastructure: TypeScript configuration, Prisma ORM with PostgreSQL schema matching data-model.md, Docker Compose for local development (PostgreSQL, Redis, MinIO), observability stack (Pino, Prometheus, OpenTelemetry), and base NestJS modules.

---

## Context

- **Mission**: customer-support-platform-mvp-01M0DPMQ
- **Phase**: Epic 1 - Authentication & Access foundation
- **Tech Stack**: NestJS, TypeScript 5.x, Node.js 20+ LTS, Prisma ORM, PostgreSQL 15+, Redis 7+, MinIO (S3-compatible)
- **Key Decisions**: 
  - Backend framework: NestJS (DM-01M0DVBY)
  - ORM: Prisma (DM-01M0DVGN)
  - Async: Bull/BullMQ Redis-based (DM-01M0DWBJ)
  - Real-time: Hybrid SSE + REST (DM-01M0DXQS)
  - Search: PostgreSQL full-text (DM-01M0DZMV)
  - Email: Postmark (DM-01M0E1QJ)
  - CI/CD: GitHub Actions (DM-01M0GHZG)
  - Observability: Pino + Prometheus + OpenTelemetry (DM-01M0GJ3P)
  - Storage: S3-compatible MinIO (DM-01M0GKP5)
  - Orchestration: Docker Compose + K8s (DM-01M0GKZ7)
- **Reference Documents**: plan.md (Technical Context, IC-01, IC-13, IC-15), data-model.md (all entities), spec.md (NFR-001, NFR-003, NFR-005, NFR-006, NFR-007, C-001 through C-016)

---

## Detailed Guidance per Subtask

### T001: Initialize NestJS backend project structure with TypeScript, ESLint, Prettier

**Purpose**: Create the backend project scaffold with all tooling configured.

**Steps**:
1. Run `npm init -y` in `/backend` directory
2. Install core dependencies:
   ```bash
   npm install @nestjs/core @nestjs/common @nestjs/platform-express @nestjs/config @nestjs/jwt @nestjs/passport @nestjs/swagger @nestjs/throttler
   npm install prisma @prisma/client bullmq ioredis pino pino-pretty @nestjs/terminus
   npm install class-validator class-transformer zod
   npm install argon2 jsonwebtoken passport passport-jwt
   ```
3. Install dev dependencies:
   ```bash
   npm install -D typescript @types/node @types/jsonwebtoken @types/passport-jwt @types/argon2
   npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin prettier
   npm install -D jest @types/jest ts-jest supertest @nestjs/testing
   npm install -D @nestjs/cli
   ```
4. Create `tsconfig.json` with strict mode, ES2022 target, NodeNext module resolution
5. Create `nest-cli.json` with sourceRoot: "src", compilerOptions: { deleteOutDir: true }
6. Configure ESLint with TypeScript parser, recommended rules, prettier integration
7. Configure Prettier with single quotes, trailing commas, 100 char width
8. Create basic directory structure: `src/{modules,common,config,database,observability,health}`

**Files**: `backend/package.json`, `backend/tsconfig.json`, `backend/nest-cli.json`, `backend/.eslintrc.js`, `backend/.prettierrc`, `backend/src/main.ts`

**Validation**:
- [ ] `npm run build` compiles without errors
- [ ] `npm run lint` passes
- [ ] `npm run format:check` passes
- [ ] NestJS CLI commands work (`nest g module test`)

---

### T002: Configure Prisma ORM with PostgreSQL schema from data-model.md

**Purpose**: Define the complete database schema matching data-model.md exactly.

**Steps**:
1. Run `npx prisma init` to create `prisma/schema.prisma`
2. Define all 20+ entities from data-model.md:
   - Tenant, User, UserRole, Session, Organization, Customer
   - Queue, Category, Status, Priority, Case, Communication, Attachment
   - SLARule, BusinessHours, Macro, AuditLog
   - CaseWatcher, CaseActivity, EmailIngestionLog
3. Use UUID primary keys with `default gen_random_uuid()`
4. Add all indexes exactly as specified in data-model.md
5. Configure enums for UserRole, CommunicationType, CaseSource, AuditEventType
6. Add partial unique indexes (e.g., catch-all queue, shortcut_key)
7. Run `npx prisma generate` to create Prisma Client
8. Create initial migration: `npx prisma migrate dev --name init`

**Files**: `backend/prisma/schema.prisma`, `backend/prisma/migrations/`

**Validation**:
- [ ] `npx prisma validate` passes
- [ ] Migration creates all tables with correct constraints
- [ ] Prisma Client generates with all models
- [ ] `npx prisma db push` works against local PostgreSQL

---

### T003: Set up Docker Compose for local development (PostgreSQL, Redis, MinIO) [P]

**Purpose**: Provide zero-config local development environment.

**Steps**:
1. Create `docker-compose.yml` at repository root with services:
   - `postgres`: postgres:15-alpine, port 5432, volume for data, healthcheck
   - `redis`: redis:7-alpine, port 6379, healthcheck
   - `minio`: minio/minio:latest, ports 9000/9001, volume for data, healthcheck
   - `backend`: build from backend/Dockerfile, depends on postgres/redis/minio
   - `frontend`: build from frontend/Dockerfile (placeholder for now)
2. Configure environment variables for all services
3. Set up MinIO bucket creation on startup (mc alias, mb)
4. Add `docker-compose.prod.yml` for production-like overrides
5. Create `.env.example` with all required variables
6. Test `docker-compose up -d` starts all services healthy

**Files**: `docker-compose.yml`, `docker-compose.prod.yml`, `.env.example`, `backend/Dockerfile`

**Validation**:
- [ ] `docker-compose up -d` starts all 4 services
- [ ] `docker-compose ps` shows all healthy
- [ ] PostgreSQL accessible on localhost:5432
- [ ] Redis accessible on localhost:6379
- [ ] MinIO console on localhost:9001, API on localhost:9000

---

### T004: Configure Pino logging, Prometheus metrics, OpenTelemetry tracing [P]

**Purpose**: Establish observability foundation per DM-01M0GJ3P.

**Steps**:
1. Create `src/observability/logger.ts` - Pino configuration:
   - Pretty print in development, JSON in production
   - Redact sensitive fields (password, token, authorization)
   - Add correlation ID middleware integration
   - Log levels: debug, info, warn, error, fatal
2. Create `src/observability/metrics.ts` - Prometheus metrics:
   - HTTP request duration histogram
   - HTTP request total counter
   - Active connections gauge
   - Business metrics: cases created, SLA breaches, emails processed
3. Create `src/observability/tracing.ts` - OpenTelemetry:
   - NodeSDK with ConsoleSpanExporter (dev) / OTLP (prod)
   - Auto-instrumentation for HTTP, Prisma, Redis, NestJS
   - Custom span attributes for tenant, user, case
4. Create `src/observability/observability.module.ts` exporting all providers
5. Add health check endpoints: `/health/live`, `/health/ready` using @nestjs/terminus

**Files**: `backend/src/observability/logger.ts`, `backend/src/observability/metrics.ts`, `backend/src/observability/tracing.ts`, `backend/src/observability/observability.module.ts`, `backend/src/health/`

**Validation**:
- [ ] Structured JSON logs in production mode
- [ ] Prometheus metrics at `/metrics` endpoint
- [ ] Health checks return 200 with correct status
- [ ] Traces generated for HTTP requests

---

### T005: Create base NestJS modules: AppModule, ConfigModule, DatabaseModule

**Purpose**: Wire together the foundational modules.

**Steps**:
1. Create `src/config/configuration.ts` - centralized config with validation:
   - Use @nestjs/config with Joi validation schema
   - Database URL, JWT secrets, Redis URL, MinIO config, Email config
   - Environment-specific overrides
2. Create `src/database/prisma.service.ts` - PrismaService extending PrismaClient:
   - onModuleInit: connect, onModuleDestroy: disconnect
   - Middleware for query logging (debug), soft delete filtering
3. Create `src/database/database.module.ts` exporting PrismaService
4. Update `src/app.module.ts` to import:
   - ConfigModule.forRoot({ isGlobal: true, load: [configuration] })
   - DatabaseModule
   - ObservabilityModule
   - ThrottlerModule.forRoot({ throttlers: [{ ttl: 60000, limit: 100 }] })
5. Create `src/common/` with:
   - `filters/http-exception.filter.ts` - standardized error responses
   - `interceptors/transform.interceptor.ts` - response wrapping
   - `pipes/validation.pipe.ts` - Zod + class-validator integration
   - `guards/` - placeholder for auth guards
   - `decorators/` - CurrentUser, Roles, Public decorators

**Files**: `backend/src/config/`, `backend/src/database/`, `backend/src/app.module.ts`, `backend/src/common/`

**Validation**:
- [ ] Application starts: `npm run start:dev`
- [ ] Config validation fails fast on missing required vars
- [ ] PrismaService connects to database
- [ ] Global pipes/filters/interceptors active
- [ ] Swagger docs at `/api/docs`

---

## Branch Strategy

- **Planning Branch**: `epic1` (current)
- **Work Package Branch**: `feature/wp01-foundation` (created from `epic1`)
- **Final Merge Target**: `epic1`
- **Execution Worktree**: Will be allocated by `finalize-tasks` based on `lanes.json`
- **Implementation Command**: `spec-kitty agent action implement WP01 --agent claude`

---

## Test Strategy

- Unit tests for configuration validation
- Integration test: PrismaService connects and queries
- Docker Compose integration test in CI

---

## Definition of Done

- [ ] All 5 subtasks complete with validation passing
- [ ] `npm run build` succeeds
- [ ] `npm run test` passes (unit tests)
- [ ] `docker-compose up -d` starts all services healthy
- [ ] Application starts and serves Swagger at `/api/docs`
- [ ] Health checks return 200
- [ ] Prometheus metrics at `/metrics`
- [ ] Prisma schema matches data-model.md exactly
- [ ] Code follows ESLint/Prettier standards

---

## Risks

- Prisma schema must match data-model.md exactly - any deviation breaks downstream WPs
- Docker Compose networking between services must work (service names as hostnames)
- MinIO bucket creation on startup can be flaky - use healthcheck + retry
- OpenTelemetry setup can be verbose - start minimal, expand later

---

## Reviewer Guidance

- Verify Prisma schema against data-model.md entity-by-entity
- Check all indexes, constraints, enums match specification
- Validate Docker Compose service healthchecks are meaningful
- Ensure observability doesn't log secrets (check redacted fields)
- Confirm NestJS module structure follows layered architecture (NFR-001)