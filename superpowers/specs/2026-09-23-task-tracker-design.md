# Task Tracker Service Design

Date: 2026-09-23
Status: Draft for review

## Goal

Build a modern task tracker for product and development teams. The product should feel like a clean Notion-like workspace, but operate like a serious task system: fast, realtime, API-first, integration-friendly, and efficient enough to run initially on one VPS.

Primary roles:

- Product Owner
- Team Lead
- Developer

The first production target is a web app. Mobile is planned later with Flutter, using the same backend API and realtime contracts.

## Product Shape

The core hierarchy is:

- Workspace
- Team
- Project
- Task

`Task` is the universal work item. It supports task types now and hierarchy later. Each task receives a readable key such as `CORE-128`, based on the team or project prefix.

The app must support classic task tracking, but the core differentiator is that it is optimized for API integrations and future AI agents. External tools should be able to create, read, update, subscribe to, and replay task activity without scraping UI state.

## V1 Scope

V1 includes:

- Workspaces, teams, projects, tasks
- Product Owner, Team Lead, Developer roles
- Custom workflows and workflow presets
- Kanban/list/calendar/timeline views
- Sprints/cycles
- Milestones/releases
- Roadmap view
- Dependency graph and impact analysis
- Dashboards and project health hints
- Manual date planning warnings, without automatic rescheduling
- Recurring tasks
- Task and project templates
- Saved views, filters, and search
- Command palette
- Task details as drawer and full-page view
- Block-style task description
- Comments with threads
- Soft realtime editing states and autosave indicators
- In-app, email, and web push notifications
- Open REST API with OpenAPI documentation
- WebSocket realtime updates
- Webhooks with versioned payloads and retry logging
- `/events` replay endpoint with 30-day retention
- API tokens scoped by workspace/team/project/domain
- CSV and JSON import/export
- Russian and English UI
- Light and dark themes
- Compact and comfortable density modes

Not included in V1:

- Built-in AI assistant
- Billing
- Native mobile application
- Automatic rescheduling
- Deep document editor parity with Notion

The system must still be designed so these can be added later.

## Recommended Stack

Backend:

- Go
- PostgreSQL as source of truth
- Redis for cache, pub/sub support, and queue backend
- Asynq for background jobs
- Chi or Echo for HTTP routing
- Gorilla/websocket or nhooyr websocket for realtime
- OpenAPI generated from source or maintained spec

Frontend:

- React
- TypeScript
- Vite
- TanStack Query for server state
- Zustand for local UI state
- React Router
- CSS modules or plain scoped CSS first; design system can be extracted later

Mobile later:

- Flutter
- Reuse REST, WebSocket, and push notification contracts

Deployment:

- One VPS initially
- Docker Compose for app-owned services
- PostgreSQL, Redis, API, worker, web frontend
- Existing containers on `root@172.16.0.61` must not be stopped, restarted, renamed, or modified until a deploy plan explicitly says so and the user approves it.
- New services should use isolated project names, directories, networks, volumes, and ports.

## Repository Layout

Use three independent project repositories inside one local coordination workspace:

```text
task_tracker/
  backend/      # separate git repository
    api/
    worker/
    contracts/
  web/          # separate git repository
  mobile/       # separate git repository
  docs/         # coordination docs for now
  infra/        # coordination/deploy docs for now
  prototype/
```

Each project has its own git history, `main`, `dev`, feature branches, version file, and changelog.

Backend owns OpenAPI and generated/public contracts. Web and mobile consume backend contracts instead of sharing a fourth package repository during MVP.

## Backend Boundaries

The backend owns:

- Authentication sessions
- Workspace membership and roles
- Authoritative task/project/team data
- Workflow transitions
- Event log
- Webhook delivery
- Notification scheduling
- API token permissions
- Import/export jobs

Suggested Go packages:

```text
cmd/api
cmd/worker
internal/auth
internal/config
internal/db
internal/events
internal/httpapi
internal/notifications
internal/permissions
internal/projects
internal/realtime
internal/tasks
internal/webhooks
internal/workspaces
```

## Data Model

Core tables:

- users
- workspaces
- workspace_members
- teams
- projects
- project_members
- workflows
- workflow_statuses
- workflow_transitions
- tasks
- task_relations
- task_comments
- task_comment_threads
- task_descriptions
- cycles
- milestones
- saved_views
- notifications
- api_tokens
- webhooks
- webhook_deliveries
- event_log
- imports
- exports

Schema management rules:

- All database schema changes must be represented as versioned migrations in the repository.
- Do not change production or preview database schema with ad hoc manual SQL.
- Each schema migration must be reviewed together with the application code that depends on it.
- Migrations must be safe to run repeatedly through the migration tool: already-applied versions must be tracked by a schema migrations table.
- Destructive changes require an explicit expand-and-contract plan: add new schema first, backfill, switch application code, then remove old schema in a later accepted task.
- Deploy steps that include migrations must document verification and rollback notes before touching preview or production data.

Important event principles:

- Every meaningful mutation writes an `event_log` row in the same transaction.
- Realtime, webhooks, notifications, and `/events` replay are consumers of the same event stream.
- Public event payloads must be versioned separately from internal storage.
- Events must avoid leaking private project data outside the requester scope.

## API Design

REST endpoints are the primary public interface.

Initial endpoint groups:

- `/auth/yandex/start`
- `/auth/yandex/callback`
- `/auth/session`
- `/workspaces`
- `/teams`
- `/projects`
- `/tasks`
- `/tasks/{id}/comments`
- `/tasks/{id}/relations`
- `/cycles`
- `/milestones`
- `/saved-views`
- `/events`
- `/webhooks`
- `/api-tokens`
- `/imports`
- `/exports`

Realtime:

- `/ws`
- Authenticated by session or scoped token
- Subscriptions are scoped to workspace, team, project, and saved view
- Initial events: task created, task updated, task deleted, comment added, relation changed, project health changed, notification created

OpenAPI:

- Generate or maintain `backend/contracts/openapi.yaml`
- Public payloads must include `api_version`, `event_version`, `event_id`, `occurred_at`, `actor`, `entity`, and `payload`

## Auth

V1 human login:

- Yandex ID OAuth

Auth requirements:

- Secure HTTP-only session cookie for the web app
- CSRF protection for cookie-authenticated mutating requests
- API tokens for external integrations
- Scoped token permissions by workspace/team/project/domain

No password login in V1.

## Notifications

Channels:

- In-app
- Email
- Web push

Notification generation is asynchronous. User/project data must not be sent to product analytics logs such as Yandex Metrica or AppMetrica.

## Observability

Operational logging:

- Structured backend logs
- Request IDs
- Error IDs shown to users
- Worker job logs
- Webhook delivery logs

Product analytics:

- Yandex Metrica for web product analytics
- Yandex AppMetrica later for mobile

Privacy rule:

- Do not send concrete user, task, project, description, or comment content to analytics providers.
- Use event names, route groups, feature flags, timing, counts, and anonymized IDs only.

## Frontend Design Direction

Use the agreed workspace direction, updated after the S6-T2 design review: Notion-like information structure, but with Linear as the quality bar for visual execution.

Design reference:

- The structural reference is Notion Projects / Notion workspace behavior: left workspace navigation, page icon/title, database-style task views, quiet property tags, command palette, and page peek/drawer.
- The current professional quality reference is Linear: dense task surfaces, restrained typography, careful spacing, compact controls, serious production feel, and no decorative filler.
- The product must not use the earlier Apple product-page direction.
- The product must not use beige/sand/tan Notion colors as the base palette; the user rejected that look.
- The current palette may remain Blue Gray, but it must feel closer to a polished Linear-like product than a loose prototype.

Core layout:

- Left navigation
- Page title and page icon
- Database-style tables
- Linked views
- Quiet tags and properties
- Drawer/page task detail
- Command palette

Current agreed palette:

- White main canvas
- Blue-gray sidebar: `#f4f7fb`
- Secondary sidebar surface: `#e8eef7`
- Main text: `#26313d`
- Soft page block surface: `#f8fbff`
- Selected navigation: `rgba(47, 111, 235, .1)`
- Primary action blue: `#2f6feb`
- Subtle low-contrast borders
- Red/amber/green/purple status tags

Interaction and density:

- The UI must remain dense enough for daily team operations.
- Prioritize database ergonomics over decorative cards.
- Keep controls quiet and compact: view tabs, filters, sort, properties, avatars, tags, comments, and side peek.
- New frontend work should be reviewed against Linear-level alignment, text scale, empty states, hover states, and table density before deployment.
- Mobile should collapse the full sidebar into workspace header plus horizontally scrollable primary pages.
- Avoid marketing-page composition, oversized hero sections, glassmorphism, product rails, gradient decoration, and decorative cards.

## First Implementation Slice

The first real slice should deliver a runnable local product foundation:

- Monorepo structure
- Go API with health endpoint
- PostgreSQL and Redis in isolated compose services
- Database migrations
- Seed workspace/team/project/tasks
- REST endpoints for reading and updating tasks
- Event log on mutations
- WebSocket broadcast for task changes
- React app using the current Notion-like UI direction
- API client and typed DTOs
- Task drawer backed by real API data
- Docker Compose for local development

This slice should not implement Yandex OAuth yet. Use a local dev user/session stub behind a clear development flag, then replace it with Yandex ID in the next slice.

## Host Deployment Constraints

Target host:

- `root@172.16.0.61`

Ingress nginx host:

- `root@172.16.0.48`

External MVP domain:

- `plane.skillmad.ru`

Known existing ports:

- `5050`
- `8790`

Rules:

- Do not stop, restart, rename, remove, or modify existing containers.
- Do not reuse existing container names, networks, or volumes.
- Do not bind to occupied ports.
- Use a separate directory such as `/srv/task-tracker`.
- Use `plane.skillmad.ru` as the external production-preview domain during MVP startup.
- Treat `172.16.0.61` as the app host and `172.16.0.48` as the nginx ingress host.
- Before first deploy, inspect available ports and write the selected ports into an env file.
- Apply database schema changes only through repository migrations.
- Deploy only after the local slice is verified.

## Risks

- Scope is large for a first pass. Work must be split into vertical slices.
- Realtime, event log, webhooks, and notifications must share one event model to avoid inconsistency.
- OAuth and API tokens must not be rushed without permission tests.
- Analytics privacy rules must be enforced from the beginning.
- The existing host has running containers; deployment must stay isolated.

## Acceptance For Slice 1

Slice 1 is complete when:

- `docker compose up` starts Postgres, Redis, API, worker, and web locally.
- Database schema is created by migrations, not manual SQL.
- The web app shows real task data from the API.
- Updating a task writes to Postgres.
- Updating a task writes an event row.
- WebSocket subscribers receive the update.
- The task drawer loads task detail from the API.
- Backend and frontend tests/checks run successfully.
- No host containers are touched.
