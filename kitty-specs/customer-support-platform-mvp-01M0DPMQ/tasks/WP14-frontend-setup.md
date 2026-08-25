---
work_package_id: WP14
title: Frontend Project Setup & Architecture
dependencies: []
requirement_refs:
- C-001
- NFR-010
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T057
- T058
- T059
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: frontend/
create_intent:
- frontend/package.json
- frontend/tsconfig.json
- frontend/vite.config.ts
- frontend/.eslintrc.js
- frontend/.prettierrc
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- frontend/package.json
- frontend/tsconfig.json
- frontend/vite.config.ts
- frontend/src/**
- frontend/.eslintrc.js
- frontend/.prettierrc
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Initialize React + TypeScript + Vite frontend with React Query, React Router, React Hook Form, Zod schemas, and atomic design component structure.

---

## Context

- **Dependencies**: None (can run parallel with WP01)
- **Reference Documents**: spec.md (C-001, NFR-010), plan.md (Technical Context, IC-15), research.md (React + TypeScript Frontend Architecture), contracts/openapi.yaml (for Zod schema generation)

---

## Detailed Guidance per Subtask

### T057: Initialize React + TypeScript + Vite frontend project

**Purpose**: Project scaffold per C-001.

**Steps**:
1. Run `npm create vite@latest frontend -- --template react-ts`
2. Install dependencies:
   ```bash
   npm install react-router-dom @tanstack/react-query @tanstack/react-query-devtools
   npm install react-hook-form @hookform/resolvers zod
   npm install axios date-fns clsx tailwind-merge
   npm install lucide-react @headlessui/react
   ```
3. Dev dependencies:
   ```bash
   npm install -D typescript eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
   npm install -D prettier eslint-config-prettier eslint-plugin-react-hooks
   npm install -D tailwindcss postcss autoprefixer
   npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
   npm install -D playwright @playwright/test
   ```
4. Configure TypeScript strict mode
5. Configure ESLint + Prettier
6. Configure Tailwind CSS
7. Create `vite.config.ts` with:
   - Proxy to backend (`/api` → `http://localhost:3000`)
   - Path aliases (`@/` → `src/`)
   - Build output to `../backend/public` for embedded serving

**Files**: `frontend/package.json`, `frontend/tsconfig.json`, `frontend/vite.config.ts`, `frontend/.eslintrc.js`, `frontend/.prettierrc`, `frontend/tailwind.config.js`, `frontend/postcss.config.js`

**Validation**:
- [ ] `npm run dev` starts dev server
- [ ] `npm run build` produces production build
- [ ] `npm run lint` passes
- [ ] `npm run test` runs Vitest
- [ ] Proxy works to backend

---

### T058: Configure React Query, React Router, React Hook Form, Zod schemas [P]

**Purpose**: Core libraries setup.

**Steps**:
1. Create `src/providers/QueryProvider.tsx` - React Query client with:
   - Default staleTime: 30s
   - Retry: 3
   - Error boundary integration
2. Create `src/router/AppRouter.tsx` - React Router v6:
   - Public routes (login, register)
   - Protected routes (require auth)
   - Role-based routes (customer, agent, supervisor, admin)
   - Lazy-loaded page chunks
3. Create `src/lib/zod.ts` - shared Zod schemas:
   - Generate from OpenAPI or manually mirror backend DTOs
   - Export: LoginSchema, CreateCaseSchema, UpdateCaseSchema, etc.
4. Create `src/lib/form.ts` - React Hook Form helpers with Zod resolver

**Files**: `frontend/src/providers/`, `frontend/src/router/`, `frontend/src/lib/`

**Validation**:
- [ ] React Query provider wraps app
- [ ] Router handles public/protected/role routes
- [ ] Zod schemas match backend DTOs
- [ ] Form helpers work with validation

---

### T059: Set up atomic design component structure [P]

**Purpose**: Component architecture per research.md.

**Steps**:
1. Create directory structure:
   ```
   src/components/
   ├── atoms/      # Button, Input, Select, Badge, Avatar, Icon
   ├── molecules/  # FormField, Card, Table, Modal, Dropdown, Toast
   ├── organisms/  # Header, Sidebar, CaseList, CaseEditor, Dashboard
   ├── pages/      # LoginPage, DashboardPage, CaseDetailPage, SettingsPage
   ```
2. Create base atoms: Button, Input, Textarea, Select, Checkbox, Radio, Badge, Avatar, Spinner
3. Create layout components: Header (user menu, notifications), Sidebar (navigation per role)
4. Create `src/styles/` with Tailwind utilities, CSS variables for theming
5. Storybook setup (optional): `npx storybook@latest init`

**Files**: `frontend/src/components/`, `frontend/src/styles/`

**Validation**:
- [ ] Atomic structure in place
- [ ] Base atoms reusable
- [ ] Layout components responsive
- [ ] Storybook runs (if configured)

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp14-frontend-setup` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP14 --agent claude`

---

## Test Strategy

- Unit tests for atoms with Vitest + Testing Library
- Component tests for molecules/organisms
- E2E tests with Playwright (WP19)

---

## Definition of Done

- [ ] React + TypeScript + Vite initialized
- [ ] React Query, Router, Hook Form, Zod configured
- [ ] Atomic component structure
- [ ] Tailwind CSS working
- [ ] Dev server + build + lint + test all work
- [ ] Proxy to backend functional

---

## Risks

- Shared Zod schemas with backend - keep in sync
- Vite proxy config for API calls
- TypeScript strict mode - fix all errors
- Role-based routing complexity

---

## Reviewer Guidance

- Verify Zod schemas match backend DTOs
- Check role-based route protection
- Confirm atomic design structure followed
- Test responsive breakpoints