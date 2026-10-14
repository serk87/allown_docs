> Историческое соглашение от 23 сентября 2026 года. С 14 октября 2026 текущие правила работы с репозиториями, UI и выпуска находятся в [repository-workflow.md](repository-workflow.md). Ниже сохранено прежнее соглашение для истории; оно не задаёт текущий процесс.

# Working Agreement

Date: 2026-09-23

This document defines how we move through MVP tasks.

## Roles

- User: product owner and final acceptance gate.
- Codex: implementation partner for tasks from the sprint plan.

## Repositories

The product is split into three independent repositories:

- `backend/`: Go API, worker, migrations, realtime, queues, integrations, and backend-owned contracts.
- `web/`: React web frontend.
- `mobile/`: Flutter mobile app, planned after the web MVP foundation.

`docs/`, `infra/`, and `prototype/` are coordination materials in the current workspace until a separate product-docs repository is approved.

## Branches

- `main`: approved production release line.
- `dev`: integrated line for accepted MVP work.
- `feature/sNN-tMM-short-name`: one branch per task in every repository touched by that task, created from that repository's `dev`.

Example:

```text
feature/s1-t1-project-skeleton
```

## Task Lifecycle

1. The user says which task to start, for example `делаем S1-T1`.
2. Codex creates or switches to the task feature branch in each touched repository.
3. Codex implements the task.
4. Codex runs the checks listed for the task.
5. During MVP startup, Codex deploys the task to `plane.skillmad.ru` when the task has a runnable app surface.
6. The user reviews the result.
7. If changes are needed, Codex continues on the same feature branch and redeploys.
8. Before final commit, Codex updates each touched repository's `VERSION` and `CHANGELOG.md`.
9. When the user says OK, Codex commits the task in each touched repository.
10. Codex merges each touched repository's feature branch into that repository's `dev`.
11. Codex creates a local version tag on each touched repository's `dev`.
12. Codex redeploys `dev` to `plane.skillmad.ru`.
13. Codex merges `dev` into `main` only after explicit user approval.

## Commit Rules

Commit messages use the task id:

```text
docs(S1-T1): initialize project workflow
feat(S1-T3): add local runtime skeleton
fix(S2-T4): prevent private event payload leak
```

A task is not committed or merged while the user is still reviewing it.

## Versioning Rules

Current version sources:

```text
backend/VERSION
web/VERSION
mobile/VERSION
```

Changelogs:

```text
backend/CHANGELOG.md
web/CHANGELOG.md
mobile/CHANGELOG.md
```

MVP version format:

```text
0.<sprint>.<task>
```

Examples:

```text
S1-T1 -> 0.1.1
S2-T4 -> 0.2.4
```

Every accepted task must update `VERSION` and `CHANGELOG.md` in each repository it touches.

After a task is accepted, committed, and merged into `dev`, create a local annotated tag in each touched repository:

```bash
git tag -a v0.sprint.task -m "Task S<sprint>-T<task>"
```

Do not push tags until a remote repository and push policy are explicitly approved.

## Deployment Rules

Target host:

```text
root@172.16.0.61
```

External MVP domain:

```text
plane.skillmad.ru
```

Deployment constraints:

- Do not stop, restart, rename, remove, or modify existing containers.
- Do not reuse existing container names, networks, or volumes.
- Do not bind to occupied ports.
- Use `/srv/task-tracker` for app-owned deployment files.
- Inspect host ports, Docker resources, and nginx before the first real deploy.
- Apply database schema changes only through versioned repository migrations.
- Do not run ad hoc manual SQL against preview or production schema.
- Every preview build should expose release metadata at `/release.json` once the app runtime exists.

## Database Rules

All schema changes must be delivered through migrations stored in the repository.

Migration requirements:

- Each migration has a stable version and is tracked by the migration tool.
- Application code and the migration it needs are reviewed together.
- Local migration checks run before preview deploy.
- Preview deploys with schema changes include verification and rollback notes.
- Destructive changes use expand-and-contract: add new schema, backfill, switch code, remove old schema in a later accepted task.

## Design Rules

The agreed UI direction keeps the Notion-like workspace structure, but future screens should be judged against Linear-level product quality: dense, calm, professional, precise spacing, compact data surfaces, and fewer decorative blocks. Keep the Blue Gray base only where it supports that cleaner Linear-inspired product feel.

Use Manrope as the primary interface font across the web app. Keep provider-branded controls on their official typography and spacing, including the Yandex ID button with `YS Text`.

Do not return to:

- Apple landing-page styling
- Beige Notion base colors
- Glassmorphism
- Oversized hero sections
- Decorative gradient cards
- Marketing-page composition
- Loose prototype spacing that makes the UI feel generated instead of production-grade

## Privacy Rules

Do not send concrete user, project, task, description, or comment content to analytics providers.

Yandex Metrica and future AppMetrica payloads may include event names, route groups, feature flags, timings, counts, and anonymized identifiers only.
