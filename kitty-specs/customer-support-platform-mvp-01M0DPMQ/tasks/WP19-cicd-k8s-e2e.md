---
work_package_id: WP19
title: CI/CD, Kubernetes & E2E Validation
dependencies:
- WP01
- WP02
- WP03
- WP04
- WP05
- WP06
- WP07
- WP08
- WP09
- WP10
- WP11
- WP12
- WP13
- WP14
- WP15
- WP16
- WP17
- WP18
requirement_refs:
- C-011
- C-012
- NFR-006
- NFR-008
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: feature/wp19-cicd-k8s-e2e
subtasks:
- T069
- T070
- T071
- T072
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: .github/workflows/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- .github/workflows/
- k8s/
- docker-compose.prod.yml
- backend/Dockerfile
- frontend/Dockerfile
- frontend/tests/e2e/
role: implementer
tags: []
---

## Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Configure GitHub Actions CI/CD pipeline, create Kubernetes manifests (base + dev/prod overlays), write E2E tests for critical flows, and verify seed data works for portfolio demo.

---

## Context

- **Dependencies**: All prior WPs
- **Reference Documents**: spec.md (NFR-006, NFR-008, C-011, C-012), plan.md (IC-15), DM-01M0GHZG (GitHub Actions), DM-01M0GKZ7 (Docker Compose + K8s)

---

## Detailed Guidance per Subtask

### T069: Configure GitHub Actions CI/CD: lint, typecheck, test, build, deploy

**Purpose**: Automated pipeline per C-012, DM-01M0GHZG.

**Steps**:
1. Create `.github/workflows/ci.yml`:
   - Trigger: push to epic1, PR to epic1
   - Jobs:
     - `lint`: backend + frontend lint
     - `typecheck`: backend + frontend tsc --noEmit
     - `test-backend`: unit + integration (Testcontainers)
     - `test-frontend`: unit + component (Vitest)
     - `build`: Docker images for backend + frontend
     - `deploy-dev`: deploy to dev K8s (on epic1 push)
2. Create `.github/workflows/cd.yml`:
   - Manual trigger or tag push
   - Deploy to prod K8s
3. Secrets: DOCKER_REGISTRY, KUBECONFIG_DEV, KUBECONFIG_PROD, POSTMARK_SECRET, JWT_SECRET, etc.
4. Cache: npm, Docker layers

**Files**: `.github/workflows/ci.yml`, `.github/workflows/cd.yml`

**Validation**:
- [ ] CI runs on push/PR
- [ ] All lint/typecheck/test jobs pass
- [ ] Docker images built and pushed
- [ ] Dev deployment works
- [ ] Secrets configured

---

### T070: Create Kubernetes manifests (base + dev/prod overlays with kustomize) [P]

**Purpose**: K8s deployment per DM-01M0GKZ7.

**Steps**:
1. Create `k8s/base/`:
   - `namespace.yaml`
   - `backend/deployment.yaml`, `service.yaml`, `configmap.yaml`, `secret.yaml`
   - `frontend/deployment.yaml`, `service.yaml`, `configmap.yaml`
   - `postgres/statefulset.yaml`, `service.yaml`, `pvc.yaml`
   - `redis/deployment.yaml`, `service.yaml`
   - `minio/statefulset.yaml`, `service.yaml`, `pvc.yaml`
   - `ingress.yaml` (nginx)
   - `kustomization.yaml`
2. Create `k8s/overlays/dev/kustomization.yaml`:
   - Replicas: 1
   - Resources: minimal
   - Image tags: latest
   - Ingress: dev.domain.com
3. Create `k8s/overlays/prod/kustomization.yaml`:
   - Replicas: 3 (backend), 2 (frontend)
   - Resources: production limits
   - Image tags: semver
   - Ingress: prod.domain.com
   - HPA for backend
4. Test: `kubectl apply -k k8s/overlays/dev`

**Files**: `k8s/base/`, `k8s/overlays/dev/`, `k8s/overlays/prod/`

**Validation**:
- [ ] Base manifests complete
- [ ] Dev overlay deploys successfully
- [ ] Prod overlay deploys successfully
- [ ] Ingress works
- [ ] HPA configured for prod

---

### T071: Write E2E tests for critical user flows (Playwright) [P]

**Purpose**: E2E validation per NFR-010.

**Steps**:
1. Create `frontend/tests/e2e/` with Playwright:
   - `auth.spec.ts`: login, logout, token refresh, role access
   - `customer-flow.spec.ts`: create case, view case, reply, attachment
   - `agent-flow.spec.ts`: queue view, case edit, macro, collision, bulk
   - `supervisor-flow.spec.ts`: dashboard, drill-down
   - `admin-flow.spec.ts`: config CRUD, audit log
   - `email-flow.spec.ts`: inbound creates case, reply attaches
2. Test data: use seeded demo data
3. CI integration: run on PR, headless Chromium
4. Screenshots on failure

**Files**: `frontend/tests/e2e/`, `frontend/playwright.config.ts`

**Validation**:
- [ ] All critical flows covered
- [ ] Tests run in CI
- [ ] Flaky tests minimized
- [ ] Screenshots on failure

---

### T072: Verify seed data works end-to-end for portfolio demo [P]

**Purpose**: Demo readiness per NFR-008.

**Steps**:
1. Deploy to dev environment via CI/CD
2. Run seed: `npx prisma db seed`
3. Verify demo accounts work:
   - admin@demo.com / password123 (admin)
   - agent1@demo.com / password123 (agent)
   - supervisor@demo.com / password123 (supervisor)
   - customer1@demo.com / password123 (customer)
4. Verify demo cases exist in all statuses
5. Test email-to-case: send email to Postmark inbound address
6. Document demo script for portfolio presentation
7. Performance check: <200ms p95 API, <100ms p95 SSE

**Files**: Documentation in `DEMO.md` or README

**Validation**:
- [ ] All demo accounts login
- [ ] Cases in all statuses visible
- [ ] Email-to-case works
- [ ] Real-time updates work
- [ ] Performance targets met
- [ ] Demo script documented

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp19-cicd-k8s-e2e` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP19 --agent claude`

---

## Test Strategy

- CI pipeline validation
- K8s manifest validation (kubeval, kustomize build)
- E2E test execution in CI
- Demo verification checklist

---

## Definition of Done

- [ ] GitHub Actions CI/CD pipeline complete
- [ ] K8s manifests (base + dev/prod overlays)
- [ ] E2E tests for all critical flows
- [ ] Seed data verified for demo
- [ ] Performance targets met
- [ ] Documentation for demo

---

## Risks

- GitHub Actions secrets management
- K8s manifest correctness
- E2E flakiness
- Demo data realism vs privacy
- Multi-service deployment ordering

---

## Reviewer Guidance

- Verify CI runs all quality gates
- Check K8s manifests with kubeval
- Confirm E2E tests cover critical paths
- Test demo end-to-end
- Verify secrets not in repo