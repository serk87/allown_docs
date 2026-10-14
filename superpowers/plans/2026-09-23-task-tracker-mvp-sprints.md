# Task Tracker MVP Sprints Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the task tracker from the current prototype/spec into a production-checkable MVP on `plane.skillmad.ru`.

**Architecture:** Work in vertical slices across three independent project repositories: backend, web, and mobile. The app uses a Go API, React web client, PostgreSQL, Redis, an async worker, one shared event model owned by backend contracts, and isolated deployment assets for the host `root@172.16.0.61`.

**Tech Stack:** Go, React, TypeScript, Vite, PostgreSQL, Redis, Asynq, Docker Compose, OpenAPI, WebSocket, Yandex ID OAuth, Yandex Metrica, Yandex AppMetrica later for mobile.

**Spec:** `docs/superpowers/specs/2026-09-23-task-tracker-design.md`

## Global Constraints

- The agreed UI direction keeps the Notion-like workspace structure, but future screens must use Linear as the professional quality reference: dense, calm, precise, compact, and production-grade.
- Do not return to Apple landing-page styling, beige Notion styling, glassmorphism, oversized hero blocks, gradients, decorative cards, or marketing-page composition.
- Every product feature must be API-ready and future AI-agent friendly.
- Do not send concrete user, task, project, description, or comment content to Yandex Metrica, AppMetrica, backend analytics, or frontend analytics.
- Existing containers on `root@172.16.0.61` must not be stopped, restarted, renamed, removed, or modified.
- New app services must use isolated names, networks, volumes, directories, and ports.
- All database schema changes must go through versioned repository migrations; no ad hoc manual SQL is allowed for preview or production schema changes.
- Backend, web, and mobile are separate git repositories with independent `main`, `dev`, feature branches, versions, and changelogs.
- Every accepted task must update `VERSION` and `CHANGELOG.md` in each repository it touches.
- MVP task versions use `0.<sprint>.<task>` per touched repository, for example `S1-T1 -> 0.1.1`.
- During MVP startup, deploy task branches quickly to `plane.skillmad.ru` so the user can check the real product externally.
- A feature branch is merged into `dev` only after the user says the deployed task is accepted.
- `dev` is merged into `main` only after the user explicitly approves the release merge.
- Each implementation task uses a separate feature branch in each repository it touches.

## Review Focus

- Dirty production preview: a task may be deployed before commit during MVP; the deployed build must expose branch, task id, timestamp, and dirty state in a release manifest.
- Existing host collision: deploy scripts must inspect ports, names, networks, volumes, and nginx config before binding anything.
- Event privacy: task changes must write complete internal events while public analytics and external payloads respect scope and privacy rules.
- Realtime consistency: REST mutations, WebSocket messages, webhooks, notifications, and `/events` replay must be emitted from the same event log.
- Design regression: frontend tasks must be checked against the agreed Notion-like structure plus Linear-level visual quality before deployment.

---

## Working Agreement

### Repositories

- `backend/`: Go API, worker, migrations, realtime, queues, integrations, and backend-owned contracts.
- `web/`: React web frontend.
- `mobile/`: Flutter mobile app, planned after the web MVP foundation.

### Branches

- `main`: approved production release line.
- `dev`: integrated MVP line after accepted feature branches.
- `feature/sNN-tMM-short-name`: one task branch in each touched repository, created from that repository's `dev`.

### Task Lifecycle

1. The user says: `делаем задачу SNN-TMM`.
2. Create or switch to `feature/sNN-tMM-short-name` from `dev` in each repository touched by the task.
3. Implement the smallest complete slice for that task.
4. Run the task checks listed in this document.
5. Deploy the task to `plane.skillmad.ru` during MVP startup.
6. The user checks the deployed result.
7. If the user asks for changes, continue on the same feature branch and redeploy.
8. Before final commit, update `VERSION` and `CHANGELOG.md` in each touched repository.
9. When the user says OK, commit the task in each touched repository.
10. Merge each touched repository's feature branch into that repository's `dev`.
11. Create a local version tag on each touched repository's `dev`.
12. Redeploy `dev` to `plane.skillmad.ru` so production reflects the accepted state.
13. Merge `dev` into `main` only after the user explicitly says to release into `main`.

### Commit and Merge Rules

- Commit message format: `feat(SNN-TMM): short result`, `fix(SNN-TMM): short result`, or `docs(SNN-TMM): short result`.
- A task is not merged into `dev` while the user is still reviewing it on `plane.skillmad.ru`.
- If a task requires a database migration, the deploy step must include migration verification and rollback notes.
- Destructive database changes require expand-and-contract migration steps split across accepted tasks.
- If a task touches auth, permissions, tokens, analytics, webhooks, or event payloads, add tests before implementation and run them before deploy.
- A task commit must include the version and changelog changes for that task in each touched repository.

### Deployment Rules

- Target host: `root@172.16.0.61`.
- Ingress nginx host: `root@172.16.0.48`.
- External MVP domain: `plane.skillmad.ru`.
- App directory: `/srv/task-tracker`.
- Existing occupied ports known now: `5050`, `8790`.
- Deployment must not touch existing containers.
- Deployment must not reuse existing container, network, or volume names.
- The first deploy task must inspect nginx, ports, Docker names, Docker networks, and Docker volumes before changing host state.
- The app should expose a release manifest at `/release.json` with task id, component versions, branch, commit if available, dirty flag, build timestamp, and source.

### Definition of Done for Every Task

- Local checks listed in the task pass.
- The UI still follows the agreed Blue Gray Notion-like direction if the task touches frontend.
- API and event payload changes are reflected in contracts or OpenAPI when the task changes public behavior.
- Logs and analytics do not include concrete user, project, task, description, or comment content.
- Database schema changes are delivered through migrations and verified locally before preview deploy.
- `VERSION` and `CHANGELOG.md` match the accepted task id in every touched repository.
- The task is deployed to `plane.skillmad.ru` during MVP startup.
- The user accepts the deployed task.
- The task is committed and merged into `dev` in every touched repository.

---

## Planned File Structure

- Create: `backend/` as an independent git repository for Go API, worker, migrations, realtime, queues, integrations, and backend-owned contracts.
- Create: `web/` as an independent git repository for React, routing, API client, UI state, and Notion-like screens.
- Create: `mobile/` as an independent git repository for Flutter later.
- Create: `backend/infra/docker/` for compose files, app-owned Docker networks, and service definitions.
- Create: `backend/infra/deploy/` for host deploy scripts, release manifest generation, nginx notes, and rollback notes.
- Create: `docs/product/` for product decisions, workflow presets, role matrix, and UX notes.
- Create: `backend/contracts/` for OpenAPI, webhook examples, DTOs, and event payload schemas.
- Modify: `docs/superpowers/specs/2026-09-23-task-tracker-design.md` when product or architecture decisions change.

---

## Sprint 1: Project Control and Production Preview Lane

Sprint goal: turn the current folder into a controlled multi-repository product workspace with branch flow, release metadata, and a safe path to `plane.skillmad.ru`.

### S1-T1: Initialize Git Flow and Project Skeleton

**Branch:** `feature/s1-t1-project-skeleton`

**Files:**
- Create: `.gitignore`
- Create: `README.md`
- Create: `backend/.gitignore`
- Create: `backend/README.md`
- Create: `backend/VERSION`
- Create: `backend/CHANGELOG.md`
- Create: `backend/api/.keep`
- Create: `backend/worker/.keep`
- Create: `backend/contracts/.keep`
- Create: `web/.gitignore`
- Create: `web/README.md`
- Create: `web/VERSION`
- Create: `web/CHANGELOG.md`
- Create: `web/.keep`
- Create: `mobile/.gitignore`
- Create: `mobile/README.md`
- Create: `mobile/VERSION`
- Create: `mobile/CHANGELOG.md`
- Create: `mobile/.keep`
- Create: `backend/infra/docker/.keep`
- Create: `backend/infra/deploy/.keep`
- Create: `docs/product/working-agreement.md`
- Create: `docs/product/versioning.md`

**Interfaces:**
- Produces: three independent local git repositories: `backend/`, `web/`, and `mobile/`.
- Produces: `main`, `dev`, and task branch naming convention for all following work in each repository.
- Produces: project directories used by every implementation task.

- [ ] **Step 1: Initialize repositories**

Run:

```bash
for repo in backend web mobile; do
  git -C "$repo" init
  git -C "$repo" branch -M main
  git -C "$repo" checkout -b dev
  git -C "$repo" checkout -b feature/s1-t1-project-skeleton
done
```

- [ ] **Step 2: Add baseline ignore rules**

Add `.gitignore` entries for local env files, build outputs, node modules, Go binaries, Docker temporary files, and OS metadata.

- [ ] **Step 3: Add working agreement**

Create `docs/product/working-agreement.md` summarizing the branch, deploy, review, and merge rules from this plan.

- [ ] **Step 4: Add versioning and changelog files**

Create `VERSION` and `CHANGELOG.md` in `backend/`, `web/`, and `mobile/` using `0.1.1` for task `S1-T1`. Create `docs/product/versioning.md` to define independent repository versioning.

- [ ] **Step 5: Add project skeleton directories**

Create `.keep` files in each planned directory so the repo records the product structure before app code exists.

- [ ] **Step 6: Run repository check**

Run:

```bash
git -C backend status --short --branch
git -C web status --short --branch
git -C mobile status --short --branch
```

Expected: each repository is on `feature/s1-t1-project-skeleton`, with only that repository's baseline files staged or unstaged.

- [ ] **Step 7: Deploy status**

No application deploy for this task. The external check is the committed workspace shape and docs.

- [ ] **Step 8: User acceptance**

Ask the user to approve the skeleton and working agreement before the first commit.

- [ ] **Step 9: Commit, merge to dev, and tag after approval**

Run:

```bash
for repo in backend web mobile; do
  git -C "$repo" add .
  git -C "$repo" commit -m "docs(S1-T1): initialize project workflow"
  git -C "$repo" branch dev
  git -C "$repo" branch main
  git -C "$repo" tag -a v0.1.1 -m "Task S1-T1"
  git -C "$repo" checkout dev
done
```

This first task bootstraps `main` and `dev` from the accepted initial feature commit because the repositories have no previous commits yet. Starting with S1-T2, feature branches are merged into existing `dev` branches normally.

### S1-T2: Host Inventory and Production Preview Contract

**Branch:** `feature/s1-t2-host-preview-contract`

**Files:**
- Create: `backend/infra/deploy/host-inventory.md`
- Create: `backend/infra/deploy/production-preview.md`
- Create: `backend/infra/deploy/env.example`
- Create: `backend/infra/deploy/release-manifest.schema.json`

**Interfaces:**
- Produces: deploy constraints consumed by all deploy scripts.
- Produces: `/release.json` schema consumed by web and API tasks.

- [ ] **Step 1: Create branch from dev**

Run:

```bash
git -C backend checkout dev
git -C backend checkout -b feature/s1-t2-host-preview-contract
```

- [ ] **Step 2: Inspect host without changes**

Run read-only commands over SSH:

```bash
ssh -o BatchMode=yes root@172.16.0.61 'hostname; ss -ltnp; docker ps --format "{{.Names}}\t{{.Ports}}"'
ssh -o BatchMode=yes root@172.16.0.61 'docker network ls --format "{{.Name}}"'
ssh -o BatchMode=yes root@172.16.0.61 'docker volume ls --format "{{.Name}}"'
ssh -o BatchMode=yes root@172.16.0.48 'hostname; ss -ltnp; nginx -v 2>&1 || true'
ssh -o BatchMode=yes root@172.16.0.48 'nginx -T 2>/tmp/task-tracker-nginx-48-dump.err | grep -n -C 8 "plane.skillmad.ru" || true'
```

- [ ] **Step 3: Document occupied resources**

Write `backend/infra/deploy/host-inventory.md` with observed hostname, occupied ports, existing container names, existing networks, existing volumes, and nginx domain notes for `plane.skillmad.ru`.

- [ ] **Step 4: Define preview environment**

Write `backend/infra/deploy/production-preview.md` with directory `/srv/task-tracker`, domain `plane.skillmad.ru`, selected app ports, release manifest path, rollback command shape, and the rule that existing containers are not modified.

- [ ] **Step 5: Define env example**

Write `backend/infra/deploy/env.example` with non-secret keys:

```env
APP_ENV=production-preview
APP_DOMAIN=plane.skillmad.ru
APP_ROOT=/srv/task-tracker
INGRESS_HOST=172.16.0.48
INGRESS_NGINX_CONFIG=/etc/nginx/sites-enabled/plane.skillmad.ru.conf
COMPOSE_PROJECT_NAME=task_tracker_mvp
WEB_BIND_ADDR=172.16.0.61
WEB_PORT=18080
API_BIND_ADDR=172.16.0.61
API_PORT=18081
POSTGRES_INTERNAL_PORT=5432
REDIS_INTERNAL_PORT=6379
```

- [ ] **Step 6: Define release manifest schema**

Write `backend/infra/deploy/release-manifest.schema.json` requiring `task_id`, `branch`, `commit`, `dirty`, `built_at`, `backend_version`, `web_version`, `mobile_version`, and `source`.

- [ ] **Step 7: Validate docs**

Run:

```bash
rg -n "root@172.16.0.61|plane.skillmad.ru|COMPOSE_PROJECT_NAME|dirty" backend/infra/deploy docs/superpowers
```

- [ ] **Step 8: User acceptance, commit, and merge**

Before user approval, update `backend/VERSION` to `0.1.2` and add a `backend/CHANGELOG.md` entry for `0.1.2`.

After user approval, run:

```bash
git -C backend add infra/deploy VERSION CHANGELOG.md
git -C backend commit -m "docs(S1-T2): define production preview lane"
git -C backend checkout dev
git -C backend merge --no-ff feature/s1-t2-host-preview-contract
git -C backend tag -a v0.1.2 -m "Task S1-T2"
```

### S1-T3: Local App Runtime Skeleton

**Branch:** `feature/s1-t3-local-runtime`

**Files:**
- Create: `backend/infra/docker/docker-compose.local.yml`
- Create: `backend/api/go.mod`
- Create: `backend/api/cmd/api/main.go`
- Create: `backend/api/internal/config/config.go`
- Create: `backend/api/internal/httpapi/router.go`
- Create: `backend/worker/go.mod`
- Create: `backend/worker/cmd/worker/main.go`
- Create: `web/package.json`
- Create: `web/index.html`
- Create: `web/src/main.tsx`
- Create: `web/src/App.tsx`

**Interfaces:**
- Produces: API `GET /healthz`.
- Produces: web app root route that renders the agreed product shell.
- Produces: compose service names `task_tracker_api`, `task_tracker_worker`, `task_tracker_web`, `task_tracker_postgres`, `task_tracker_redis`.

- [ ] **Step 1: Create branch from dev**

Run:

```bash
git -C backend checkout dev
git -C backend checkout -b feature/s1-t3-local-runtime
git -C web checkout dev
git -C web checkout -b feature/s1-t3-local-runtime
```

- [ ] **Step 2: Add Go API health endpoint**

Implement `GET /healthz` returning:

```json
{"status":"ok","service":"api"}
```

- [ ] **Step 3: Add worker process**

Implement a worker entrypoint that starts, logs `worker ready`, and exits cleanly on interrupt.

- [ ] **Step 4: Add React app shell**

Implement the first React screen using the approved Blue Gray Notion-like layout: left nav, page title, database toolbar, task table placeholder, and task drawer placeholder.

- [ ] **Step 5: Add isolated local compose**

Compose must start Postgres, Redis, API, worker, and web without using host production ports.

- [ ] **Step 6: Run local checks**

Run:

```bash
docker compose -f backend/infra/docker/docker-compose.local.yml up --build
curl -fsS http://127.0.0.1:18081/healthz
```

Expected API response:

```json
{"status":"ok","service":"api"}
```

- [ ] **Step 7: Visual check**

Open the web app locally and confirm the first viewport matches the Notion-like Blue Gray direction from the prototype.

- [ ] **Step 8: Deploy preview**

Deploy this skeleton to `plane.skillmad.ru` using isolated names and ports selected in S1-T2.

- [ ] **Step 9: User acceptance, commit, and merge**

Before user approval, update `backend/VERSION` and `web/VERSION` to `0.1.3` and add changelog entries in `backend/CHANGELOG.md` and `web/CHANGELOG.md`.

After user approval on `plane.skillmad.ru`, run:

```bash
git -C backend add .
git -C backend commit -m "feat(S1-T3): add backend runtime skeleton"
git -C backend checkout dev
git -C backend merge --no-ff feature/s1-t3-local-runtime
git -C backend tag -a v0.1.3 -m "Task S1-T3"

git -C web add .
git -C web commit -m "feat(S1-T3): add web runtime skeleton"
git -C web checkout dev
git -C web merge --no-ff feature/s1-t3-local-runtime
git -C web tag -a v0.1.3 -m "Task S1-T3"
```

### S1-T4: Web Page Map and Navigation

**Branch:** `feature/s1-t4-web-page-map`

**Files:**
- Modify: `web/src/App.tsx`
- Modify: `web/src/App.test.tsx`
- Modify: `web/src/styles.css`
- Modify: `web/VERSION`
- Modify: `web/CHANGELOG.md`
- Modify: `web/package.json`
- Modify: `web/package-lock.json`

**Interfaces:**
- Produces: local frontend routes `/projects`, `/projects/:id`, `/tasks/:id`, and `/settings`.
- Produces: navigable placeholder workspace routes `/inbox`, `/roadmap`, `/cycles`, and `/dashboards`.
- Produces: frontend route tests for the core page map.

- [ ] **Step 1: Create branch from dev**

Run:

```bash
git -C web checkout dev
git -C web checkout -b feature/s1-t4-web-page-map
```

- [ ] **Step 2: Write route test**

Add a frontend test that starts at `/projects`, navigates to a project, opens task detail, and opens workspace settings.

- [ ] **Step 3: Add lightweight web routing**

Implement browser-history navigation without adding a routing dependency for the MVP placeholder page map.

- [ ] **Step 4: Add core pages**

Add the projects list, project workspace, task detail, settings, inbox, roadmap, cycles, and dashboards pages while keeping the approved Notion-like Blue Gray density.

- [ ] **Step 5: Run frontend checks**

Run:

```bash
npm test
npm run build
```

- [ ] **Step 6: Visual check**

Open the local preview and verify the new pages do not regress into Apple, beige Notion, glass, hero, gradient, or marketing-page styling.

- [ ] **Step 7: User acceptance, commit, and merge**

Before user approval, update `web/VERSION` to `0.1.4`, update `web/package.json`, `web/package-lock.json`, and add a `web/CHANGELOG.md` entry.

After user approval, run:

```bash
git -C web add .
git -C web commit -m "feat(S1-T4): add web page map"
git -C web checkout dev
git -C web merge --no-ff feature/s1-t4-web-page-map
git -C web tag -a v0.1.4 -m "Task S1-T4"
```

---

## Sprint 2: Real Task Data and Event Foundation

Sprint goal: replace placeholders with real workspace, project, and task data backed by PostgreSQL and an event log.

### S2-T1: Database Migrations and Seed Data

**Branch:** `feature/s2-t1-database-seed`

**Files:**
- Create: `backend/api/internal/db/db.go`
- Create: `backend/api/internal/db/migrations/0001_core.sql`
- Create: `backend/api/internal/db/seed/dev_seed.sql`
- Create: `backend/api/internal/workspaces/repository.go`
- Create: `backend/api/internal/projects/repository.go`
- Create: `backend/api/internal/tasks/repository.go`

**Interfaces:**
- Produces: tables `users`, `workspaces`, `workspace_members`, `teams`, `projects`, `tasks`, and `event_log`.
- Produces: migration tracking table managed by the migration tool.
- Produces: seed workspace, team, project, and tasks matching the prototype.

- [ ] **Step 1: Write migration tests**
- [ ] **Step 2: Add migration runner and migration tracking table**
- [ ] **Step 3: Add core schema migration**
- [ ] **Step 4: Add dev seed data**
- [ ] **Step 5: Wire API database connection**
- [ ] **Step 6: Run migrations in local compose**
- [ ] **Step 7: Confirm seed task count with SQL**
- [ ] **Step 8: Document preview migration verification and rollback notes**
- [ ] **Step 9: Deploy preview and verify seeded data exists**
- [ ] **Step 10: Commit and merge after user acceptance**

### S2-T2: Task Read API and Contracts

**Branch:** `feature/s2-t2-task-read-api`

**Files:**
- Create: `backend/contracts/task.schema.json`
- Create: `backend/contracts/openapi.yaml`
- Create: `backend/api/internal/tasks/handler.go`
- Modify: `backend/api/internal/httpapi/router.go`

**Interfaces:**
- Produces: `GET /api/tasks`.
- Produces: `GET /api/tasks/{id}`.
- Produces: task DTO used by the React API client.

- [ ] **Step 1: Write handler tests for list and detail**
- [ ] **Step 2: Define task DTO schema**
- [ ] **Step 3: Add OpenAPI paths**
- [ ] **Step 4: Implement handlers using repository queries**
- [ ] **Step 5: Run API tests**
- [ ] **Step 6: Deploy preview and verify API responses**
- [x] **Step 7: Commit and merge after user acceptance**

### S2-T3: React Task Table from API

**Branch:** `feature/s2-t3-task-table-api`

**Files:**
- Create: `web/src/api/client.ts`
- Create: `web/src/api/tasks.ts`
- Create: `web/src/features/tasks/TaskTable.tsx`
- Create: `web/src/features/tasks/taskTypes.ts`
- Modify: `web/src/App.tsx`

**Interfaces:**
- Consumes: `GET /api/tasks`.
- Produces: database-style task table with real API data.

- [ ] **Step 1: Add API client with typed task response**
- [ ] **Step 2: Add loading, error, and empty states**
- [ ] **Step 3: Render task key, title, status, priority, assignee, project, and updated time**
- [ ] **Step 4: Keep Blue Gray Notion-like density and table behavior**
- [ ] **Step 5: Run frontend build**
- [ ] **Step 6: Deploy preview and verify real tasks in browser**
- [x] **Step 7: Commit and merge after user acceptance**

### S2-T4: Task Update API with Event Log

**Branch:** `feature/s2-t4-task-update-events`

**Files:**
- Create: `backend/api/internal/events/repository.go`
- Create: `backend/api/internal/events/event.go`
- Modify: `backend/api/internal/tasks/handler.go`
- Modify: `backend/api/internal/tasks/repository.go`
- Modify: `backend/contracts/openapi.yaml`

**Interfaces:**
- Produces: `PATCH /api/tasks/{id}`.
- Produces: internal event type `task.updated`.
- Produces: event rows consumed by realtime, webhooks, notifications, and `/events`.

- [ ] **Step 1: Write API test that updating a task changes the task row and writes one event row**
- [ ] **Step 2: Define event payload version `task.updated.v1`**
- [ ] **Step 3: Implement task update transaction**
- [ ] **Step 4: Ensure event payload does not include private description/comment content**
- [ ] **Step 5: Run API tests**
- [ ] **Step 6: Deploy preview and update a task**
- [x] **Step 7: Commit and merge after user acceptance**

---

## Sprint 3: Realtime Task Experience

Sprint goal: make the product feel alive: drawer detail, realtime updates, autosave cues, and activity based on the event log.

### S3-T1: Task Drawer Backed by API

**Branch:** `feature/s3-t1-task-drawer`

**Files:**
- Create: `web/src/features/tasks/TaskDrawer.tsx`
- Create: `web/src/features/tasks/TaskProperties.tsx`
- Modify: `web/src/features/tasks/TaskTable.tsx`

**Interfaces:**
- Consumes: `GET /api/tasks/{id}`.
- Produces: task detail drawer with title, status, assignee, dates, labels, and activity placeholder.

- [ ] **Step 1: Open drawer from table row**
- [ ] **Step 2: Fetch task detail by id**
- [ ] **Step 3: Render Notion-like properties and compact editor area**
- [ ] **Step 4: Add responsive drawer behavior for mobile**
- [ ] **Step 5: Run frontend build**
- [ ] **Step 6: Deploy preview and verify drawer behavior**
- [x] **Step 7: Commit and merge after user acceptance**

### S3-T2: WebSocket Event Stream

**Branch:** `feature/s3-t2-websocket-events`

**Files:**
- Create: `backend/api/internal/realtime/hub.go`
- Create: `backend/api/internal/realtime/handler.go`
- Modify: `backend/api/internal/events/repository.go`
- Modify: `backend/api/internal/httpapi/router.go`

**Interfaces:**
- Produces: `GET /ws`.
- Produces: realtime message for `task.updated`.

- [ ] **Step 1: Write hub test for scoped broadcast**
- [ ] **Step 2: Add authenticated development WebSocket route**
- [ ] **Step 3: Publish task update events to connected subscribers**
- [ ] **Step 4: Add heartbeat and disconnect handling**
- [ ] **Step 5: Run API tests**
- [ ] **Step 6: Deploy preview and verify updates across two browser tabs**
- [x] **Step 7: Commit and merge after user acceptance**

### S3-T3: Frontend Realtime Sync

**Branch:** `feature/s3-t3-web-realtime-sync`

**Files:**
- Create: `web/src/api/realtime.ts`
- Modify: `web/src/api/tasks.ts`
- Modify: `web/src/features/tasks/TaskTable.tsx`
- Modify: `web/src/features/tasks/TaskDrawer.tsx`

**Interfaces:**
- Consumes: `/ws` task event messages.
- Produces: task table and drawer refresh on realtime events.

- [ ] **Step 1: Add WebSocket client with reconnect backoff**
- [ ] **Step 2: Refresh changed tasks on `task.updated`**
- [ ] **Step 3: Show subtle live status indicator**
- [ ] **Step 4: Run frontend build**
- [ ] **Step 5: Deploy preview and verify two-tab sync**
- [x] **Step 6: Commit and merge after user acceptance**

---

## Sprint 4: Authentication, Roles, and Permissions

Sprint goal: replace dev identity with Yandex ID and enforce workspace/project access before expanding integrations.

### S4-T1: Development Session Boundary

**Branch:** `feature/s4-t1-dev-session-boundary`

**Files:**
- Create: `backend/api/internal/auth/session.go`
- Create: `backend/api/internal/auth/dev_user.go`
- Create: `backend/api/internal/permissions/checker.go`
- Modify: `backend/api/internal/httpapi/router.go`

**Interfaces:**
- Produces: `CurrentUser` and `PermissionChecker` interfaces.
- Produces: `GET /api/auth/session`.

- [ ] **Step 1: Write permission tests for Product Owner, Team Lead, and Developer**
- [ ] **Step 2: Add dev session only behind explicit development flag**
- [ ] **Step 3: Apply permission checks to task list, detail, and update**
- [ ] **Step 4: Run API tests**
- [ ] **Step 5: Deploy preview with dev session enabled only for MVP preview**
- [x] **Step 6: Commit and merge after user acceptance**

### S4-T2: Yandex ID OAuth

**Branch:** `feature/s4-t2-yandex-oauth`

**Files:**
- Create: `backend/api/internal/auth/yandex_oauth.go`
- Create: `backend/api/internal/auth/oauth_state.go`
- Create: `backend/api/internal/db/migrations/0002_auth_sessions.sql`
- Modify: `backend/api/internal/auth/session.go`
- Modify: `backend/api/internal/httpapi/router.go`
- Modify: `backend/contracts/openapi.yaml`
- Modify: `docs/product/auth.md`

**Interfaces:**
- Produces: `GET /auth/yandex/start`.
- Produces: `GET /auth/yandex/callback`.
- Produces: secure HTTP-only session cookie.
- Produces: DB-bound CSRF validation for cookie-authenticated mutations.

- [x] **Step 1: Write OAuth state validation tests**
- [x] **Step 2: Implement Yandex start and callback routes**
- [x] **Step 3: Store or link user by Yandex subject id**
- [x] **Step 4: Add CSRF protection for cookie-authenticated mutation routes**
- [x] **Step 5: Run API tests**
- [x] **Step 6: Deploy preview with Yandex OAuth environment variables**
- [x] **Step 7: Commit and merge after user acceptance**

### S4-T3: API Tokens with Scoped Permissions

**Branch:** `feature/s4-t3-api-tokens`

**Files:**
- Create: `backend/api/internal/tokens/token.go`
- Create: `backend/api/internal/tokens/repository.go`
- Create: `backend/api/internal/tokens/handler.go`
- Modify: `backend/contracts/openapi.yaml`

**Interfaces:**
- Produces: API token creation and revocation endpoints.
- Produces: token scopes by workspace, team, project, and domain.

- [x] **Step 1: Write token hashing and scope tests**
- [x] **Step 2: Add token table migration**
- [x] **Step 3: Implement token creation with one-time plain token return**
- [x] **Step 4: Enforce token scopes on task endpoints**
- [x] **Step 5: Run API tests**
- [x] **Step 6: Deploy preview and verify scoped API access**
- [x] **Step 7: Commit and merge after user acceptance**

---

## Sprint 5: Planning Workflows and Views

Sprint goal: move from simple task tracking into product team workflows: statuses, sprints, releases, filters, and project planning views.

### S5-T1: Workflow Presets and Status Transitions

**Branch:** `feature/s5-t1-workflows`

**Files:**
- Create: `backend/api/internal/workflows/`
- Modify: `backend/api/internal/db/migrations/`
- Modify: `web/src/features/tasks/`

**Interfaces:**
- Produces: workflow statuses and transitions.
- Produces: Software and Kanban workflow presets.

- [ ] **Step 1: Write transition permission tests**
- [ ] **Step 2: Add workflow migrations and seed presets**
- [ ] **Step 3: Add status transition API**
- [ ] **Step 4: Add status controls to table and drawer**
- [ ] **Step 5: Emit workflow transition events**
- [ ] **Step 6: Deploy preview and verify task status flow**
- [x] **Step 7: Commit and merge after user acceptance**

### S5-T2: Sprints and Cycles

**Branch:** `feature/s5-t2-sprints-cycles`

**Files:**
- Create: `backend/api/internal/cycles/`
- Create: `web/src/features/cycles/`
- Modify: `backend/contracts/openapi.yaml`

**Interfaces:**
- Produces: cycle CRUD endpoints.
- Produces: task assignment to cycle.
- Produces: cycle view in frontend.

- [ ] **Step 1: Write cycle date validation tests**
- [ ] **Step 2: Add cycle migrations**
- [ ] **Step 3: Add cycle API handlers**
- [ ] **Step 4: Add cycle field to task drawer**
- [ ] **Step 5: Add cycle board/list view**
- [ ] **Step 6: Deploy preview and verify sprint workflow**
- [x] **Step 7: Commit and merge after user acceptance**

### S5-T3: Milestones, Releases, and Roadmap

**Branch:** `feature/s5-t3-roadmap`

**Files:**
- Create: `backend/api/internal/milestones/`
- Create: `web/src/features/roadmap/`
- Modify: `backend/contracts/openapi.yaml`

**Interfaces:**
- Produces: milestone and release endpoints.
- Produces: roadmap view grouped by milestone/release.

- [x] **Step 1: Write milestone relationship tests**
- [x] **Step 2: Add milestone migrations**
- [x] **Step 3: Add milestone API handlers**
- [x] **Step 4: Add roadmap frontend view**
- [x] **Step 5: Emit milestone change events**
- [x] **Step 6: Deploy preview and verify roadmap**
- [x] **Step 7: Commit and merge after user acceptance**

### S5-T4: Saved Views, Filters, and Search

**Branch:** `feature/s5-t4-views-filter-search`

**Files:**
- Create: `backend/api/internal/savedviews/`
- Create: `backend/api/internal/search/`
- Create: `web/src/features/views/`
- Modify: `web/src/features/tasks/`

**Interfaces:**
- Produces: saved view CRUD endpoints.
- Produces: filter and search query model.
- Produces: list, kanban, calendar, and timeline entry points.

- [x] **Step 1: Write filter query tests**
- [x] **Step 2: Add saved view migrations**
- [x] **Step 3: Add saved view endpoints**
- [x] **Step 4: Add task search endpoint**
- [x] **Step 5: Add frontend view switcher and filter bar**
- [x] **Step 6: Deploy preview and verify saved views**
- [x] **Step 7: Commit and merge after user acceptance**

---

## Sprint 6: Integrations, Events, and Notifications

Sprint goal: make the system useful outside the UI with replayable events, webhooks, import/export, and notifications.

### S6-T1: Events Replay API

**Branch:** `feature/s6-t1-events-replay`

**Files:**
- Create: `backend/api/internal/events/handler.go`
- Modify: `backend/contracts/openapi.yaml`

**Interfaces:**
- Produces: `GET /api/events`.
- Produces: 30-day replay retention behavior.

- [x] **Step 1: Write scope and cursor tests**
- [x] **Step 2: Implement cursor-based event replay**
- [x] **Step 3: Filter events by requester scope**
- [x] **Step 4: Add OpenAPI examples**
- [x] **Step 5: Deploy preview and verify replay**
- [x] **Step 6: Commit and merge after user acceptance**

### S6-T2: Webhooks with Retries

**Branch:** `feature/s6-t2-webhooks`

**Files:**
- Create: `backend/api/internal/webhooks/`
- Create: `backend/worker/internal/webhooks/`
- Modify: `backend/contracts/openapi.yaml`

**Interfaces:**
- Produces: webhook subscription endpoints.
- Produces: webhook delivery worker with retry log.

- [x] **Step 1: Write webhook signing tests**
- [x] **Step 2: Add webhook migrations**
- [x] **Step 3: Add webhook CRUD endpoints**
- [x] **Step 4: Add delivery worker with retry schedule**
- [x] **Step 5: Add delivery log UI entry point**
- [x] **Step 6: Deploy preview and verify webhook delivery**
- [x] **Step 7: Commit and merge after user acceptance**

### S6-T3: Notifications

**Branch:** `feature/s6-t3-notifications`

**Files:**
- Create: `backend/api/internal/notifications/`
- Create: `backend/worker/internal/notifications/`
- Create: `web/src/features/notifications/`

**Interfaces:**
- Produces: in-app notification list.
- Produces: email notification job interface.
- Produces: web push subscription flow.

- [x] **Step 1: Write notification preference tests**
- [x] **Step 2: Add notification migrations**
- [x] **Step 3: Generate in-app notifications from event log**
- [x] **Step 4: Add email job boundary**
- [x] **Step 5: Add web push subscription endpoint**
- [x] **Step 6: Deploy preview and verify in-app notifications**
- [x] **Step 7: Commit and merge after user acceptance**

### S6-T4: CSV and JSON Import/Export

**Branch:** `feature/s6-t4-import-export`

**Files:**
- Create: `backend/api/internal/imports/`
- Create: `backend/api/internal/exports/`
- Create: `backend/worker/internal/imports/`
- Create: `backend/worker/internal/exports/`
- Create: `web/src/features/importExport/`

**Interfaces:**
- Produces: import jobs for CSV and JSON.
- Produces: export jobs for CSV and JSON.

- [x] **Step 1: Write CSV parsing tests with malformed rows**
- [x] **Step 2: Add import/export job tables**
- [x] **Step 3: Add upload and job status endpoints**
- [x] **Step 4: Add worker processing**
- [x] **Step 5: Add import/export UI**
- [x] **Step 6: Deploy preview and verify sample import/export**
- [x] **Step 7: Commit and merge after user acceptance**

---

## Sprint 7: MVP Hardening and Release

Sprint goal: make the MVP stable enough for real team use and a controlled `main` release.

### S7-T1: Analytics and Privacy Guardrails

**Branch:** `feature/s7-t1-analytics-privacy`

**Files:**
- Create: `web/src/analytics/yandexMetrica.ts`
- Create: `backend/api/internal/observability/privacy.go`
- Create: `docs/product/analytics-privacy.md`

**Interfaces:**
- Produces: privacy-safe analytics event map.
- Produces: backend redaction helpers for logs.

- [x] **Step 1: Write redaction tests for task/project/user content**
- [x] **Step 2: Add Yandex Metrica integration using route groups and feature event names**
- [x] **Step 3: Add backend log redaction helper**
- [x] **Step 4: Document allowed and blocked analytics fields**
- [x] **Step 5: Deploy preview and verify network payloads do not contain task titles or project names**
- [x] **Step 6: Commit and merge after user acceptance**

### S7-T2: Internationalization, Themes, and Density

**Branch:** `feature/s7-t2-i18n-theme-density`

**Files:**
- Create: `web/src/i18n/`
- Create: `web/src/theme/`
- Modify: `web/src/App.tsx`

**Interfaces:**
- Produces: Russian and English UI resources.
- Produces: light/dark theme toggle.
- Produces: compact/comfortable density toggle.

- [x] **Step 1: Add translation resource shape**
- [x] **Step 2: Move visible UI copy into ru/en dictionaries**
- [x] **Step 3: Add light and dark theme tokens**
- [x] **Step 4: Add density tokens**
- [x] **Step 5: Run frontend build**
- [x] **Step 6: Deploy preview and verify language/theme/density switches**
- [x] **Step 7: Commit and merge after user acceptance**

### S7-T3: Backup, Restore, and Runbook

**Branch:** `feature/s7-t3-backup-runbook`

**Files:**
- Create: `backend/infra/deploy/backup.sh`
- Create: `backend/infra/deploy/restore.md`
- Create: `backend/infra/deploy/runbook.md`

**Interfaces:**
- Produces: database backup command for the app-owned Postgres service.
- Produces: restore procedure for the app-owned volume.

- [x] **Step 1: Add backup script for app-owned Postgres only**
- [x] **Step 2: Add restore notes with explicit app-owned volume names**
- [x] **Step 3: Add deploy, rollback, logs, and restart runbook**
- [x] **Step 4: Test backup command on preview data**
- [x] **Step 5: Commit and merge after user acceptance**

### S7-T4: Main Release Gate

**Branch:** `feature/s7-t4-main-release-gate`

**Files:**
- Create: `docs/product/mvp-release-checklist.md`
- Modify: `README.md`

**Interfaces:**
- Produces: release checklist for merging `dev` into `main`.

- [x] **Step 1: Add checklist for auth, permissions, event log, realtime, deploy, backup, analytics privacy, and design review**
- [x] **Step 2: Run full local checks**
- [x] **Step 3: Deploy latest `dev` to `plane.skillmad.ru`**
- [x] **Step 4: Ask user for explicit `main` merge approval**
- [x] **Step 5: Merge only after approval**

Run after approval:

```bash
git checkout main
git merge --no-ff dev
git tag mvp-preview-1
```

---

## Sprint 8: Core Task CRUD Completion

Sprint goal: close the most visible production gaps in task CRUD so a real user can create and maintain tasks from the web app instead of only viewing seeded data.

### S8-T1: Task Creation End-to-End

**Branch:** `feature/s8-t1-task-create-e2e`

**Interfaces:**
- Produces: `POST /api/tasks` with generated task keys, role checks, CSRF for cookie sessions, token scope enforcement, event log writes, notification generation, and realtime publication.
- Produces: project workspace task creation dialog in the web app.

- [x] **Step 1: Add backend task creation tests and route**
- [x] **Step 2: Persist created tasks and privacy-safe `task.created.v1` events**
- [x] **Step 3: Add web task creation dialog and API client**
- [x] **Step 4: Update versions, changelogs, OpenAPI, and deploy preview**

### S8-T2: Task Drawer Editing

**Branch:** `feature/s8-t2-task-editing`

**Interfaces:**
- Produces: editable task title, description, and priority through `PATCH /api/tasks/{id}`.
- Produces: privacy-safe `task.updated.v1` events that include changed field names without concrete title or description text values.
- Produces: editable project workspace drawer fields with save feedback.

- [x] **Step 1: Add backend RED tests for editable fields and event privacy**
- [x] **Step 2: Extend task update contract, repository update, and OpenAPI**
- [x] **Step 3: Add web RED test and drawer editing controls**
- [x] **Step 4: Run frontend tests/build and backend API/worker tests**
- [x] **Step 5: Deploy preview and verify production edit flow**
- [ ] **Step 6: Commit and merge after user acceptance**

### S8-T3: API-Backed Project List

**Branch:** `feature/s8-t3-projects-api`

**Interfaces:**
- Produces: `GET /api/projects` with project slug, team, owner initials, task count, and updated timestamp.
- Produces: web project list and sidebar team links loaded from server project data instead of hardcoded rows.

- [x] **Step 1: Add backend RED tests for project list route and authorization**
- [x] **Step 2: Add project DTO repository, handler, router mount, main wiring, and OpenAPI**
- [x] **Step 3: Add web RED test and project API client**
- [x] **Step 4: Replace hardcoded project list/sidebar data with API state**
- [x] **Step 5: Run frontend tests/build and backend API/worker tests**
- [x] **Step 6: Deploy preview and verify `/api/projects` production smoke**
- [ ] **Step 7: Merge after user acceptance**

### S8-T4: Launch Readiness Task Permissions

**Branch:** `feature/s8-t4-release-readiness`

**Interfaces:**
- Fixes: workspace members with developer role can create and edit tasks through the web/API task workflow.
- Preserves: broader workspace-management permissions remain stricter through `CanUpdateTasks`.
- Hardens: embedded migration loading ignores hidden AppleDouble files from macOS release archives.

- [x] **Step 1: Add backend RED tests for developer task creation and task edits**
- [x] **Step 2: Split task-specific create/edit permissions from stricter workspace update permissions**
- [x] **Step 3: Add migration loader regression coverage for hidden macOS metadata files**
- [x] **Step 4: Run backend API tests, worker tests, release gate, web tests, and web build**
- [x] **Step 5: Deploy production preview `S8-T4` on ports API `18096` and web `18097`**
- [x] **Step 6: Verify production smoke for health, release manifest, auth boundary, task create, task read, task title edit, and task status edit**
- [ ] **Step 7: Merge after user acceptance**

### S8-T5: Release-Ready Task Creation and Comments

**Branch:** `feature/s8-t5-task-comments`

**Interfaces:**
- Produces: `GET /api/tasks/{id}/comments` and `POST /api/tasks/{id}/comments` with cookie CSRF, bearer token scope checks, task visibility checks, realtime publication, and privacy-safe `task.comment.created.v1` events without comment body content.
- Produces: task drawer Activity UI backed by the comments API.
- Fixes: workspace header `New task` actions are real global task creation entry points instead of inert buttons outside the project page.

- [x] **Step 1: Add backend RED tests for task comments and route mounting**
- [x] **Step 2: Add task comments migration, repository, handler, router wiring, and OpenAPI**
- [x] **Step 3: Add web RED tests for global task creation and drawer comments**
- [x] **Step 4: Replace inert header task actions with a global create flow and project selection**
- [x] **Step 5: Add task drawer comment loading/posting UI and typed web client**
- [x] **Step 6: Run frontend tests/build and backend API/worker tests**
- [x] **Step 7: Deploy production preview and verify release smoke**
- [ ] **Step 8: Merge after user acceptance**

### S8-T6: Task Soft Delete End-to-End

**Branch:** `feature/s8-t6-task-archive`

**Interfaces:**
- Produces: `DELETE /api/tasks/{id}` that soft-deletes tasks with cookie CSRF, bearer token write scope checks, task visibility checks, privacy-safe `task.deleted.v1` events, realtime publication, and notification fan-out.
- Produces: task lists, search, detail, comments, and project task counts that hide soft-deleted tasks.
- Produces: drawer delete action that refreshes the project table and selects the next visible task.

- [x] **Step 1: Add backend RED tests for task deletion, event privacy, and visibility filtering**
- [x] **Step 2: Add `deleted_at` migration, repository delete flow, handler routing, and OpenAPI**
- [x] **Step 3: Add web RED test and drawer delete action**
- [x] **Step 4: Run frontend tests/build and backend API/worker tests**
- [x] **Step 5: Deploy production preview and verify release smoke**
- [ ] **Step 6: Merge after user acceptance**

### S8-T7: Workflow-Aware Status Controls

**Branch:** `feature/s8-t7-status-transition-ux`

**Interfaces:**
- Produces: task DTO field `availableStatusTransitions` derived from backend workflow transitions.
- Produces: drawer status controls that show the current status plus only backend-allowed transition targets.
- Fixes: impossible status selections such as `planned -> in_review` that returned `400 invalid status transition` during release smoke.

- [x] **Step 1: Add backend RED tests for available task status transitions**
- [x] **Step 2: Add backend transition targets to task reads, search, creation, and OpenAPI task schema**
- [x] **Step 3: Add web RED test for status selector options**
- [x] **Step 4: Consume `availableStatusTransitions` in the drawer status control**
- [x] **Step 5: Run frontend tests/build and backend API/worker tests**
- [x] **Step 6: Deploy production preview and verify release smoke**
- [ ] **Step 7: Merge after user acceptance**

### S8-T8: Task Assignee and Due Date Editing

**Branch:** `feature/s8-t8-task-assignee-due`

**Interfaces:**
- Produces: `PATCH /api/tasks/{id}` support for `assigneeId` and `dueAt`, including nullable clearing, membership validation, privacy-safe update events, and OpenAPI coverage.
- Produces: `GET /api/workspace/members` for assignee selection without exposing member email addresses.
- Produces: drawer Owner and Due date controls backed by API writes.

- [x] **Step 1: Add backend RED tests for assignment, due date, member listing, and validation**
- [x] **Step 2: Implement backend update fields, workspace members endpoint, routing, and OpenAPI**
- [x] **Step 3: Add web RED test for Owner/Due drawer edits**
- [x] **Step 4: Implement typed member client and drawer controls**
- [x] **Step 5: Run frontend tests/build and backend API tests**
- [x] **Step 6: Deploy production preview and verify release smoke**
- [ ] **Step 7: Merge after user acceptance**

---

## First Task to Execute

Start with **S1-T1: Initialize Git Flow and Project Skeleton**.

The first user command can be:

```text
делаем S1-T1
```

After that task is implemented, the user reviews the workspace shape. Once approved, commit it and merge it into `dev`.
