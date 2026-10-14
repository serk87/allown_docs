# Versioning

Date: 2026-09-23

This product uses task-based versioning during the MVP phase, with independent versions for backend, web, and mobile.

## Source of Truth

The current backend version is stored in:

```text
backend/VERSION
```

The backend changelog is stored in:

```text
backend/CHANGELOG.md
```

The current web version is stored in:

```text
web/VERSION
```

The web changelog is stored in:

```text
web/CHANGELOG.md
```

The current mobile version is stored in:

```text
mobile/VERSION
```

The mobile changelog is stored in:

```text
mobile/CHANGELOG.md
```

## MVP Version Format

Use:

```text
0.<sprint>.<task>
```

Examples:

```text
S1-T1 -> 0.1.1
S1-T3 -> 0.1.3
S2-T4 -> 0.2.4
S7-T4 -> 0.7.4
```

Major version `0` means the product is still in MVP/beta development.

## Task Rule

Every accepted task must update both files in each repository it touches:

- `VERSION`
- `CHANGELOG.md`

The version and changelog entry are part of the task branch in that repository and must be reviewed before commit.

If the user asks for changes before accepting a task, keep the same task version in the touched repositories and update the same changelog entries before commit.

If a fix is needed after a task has already been accepted and merged, create a new task and give it its own version.

## Changelog Entry Format

Each version entry uses:

```markdown
## [0.sprint.task] - YYYY-MM-DD

### Added
### Changed
### Fixed
### Removed
### Security
```

Only include sections that have content.

Each bullet should describe a user-visible, API-visible, deployment-visible, or engineering-process-visible change.

## Git Tags

After a task is accepted, committed, and merged into `dev`, create a local annotated tag in each touched repository:

```bash
git tag -a v0.sprint.task -m "Task S<sprint>-T<task>"
```

Do not push tags until a remote repository and push policy are explicitly approved.

## Release Manifest

Once the app runtime exists, `/release.json` must include component versions from touched repositories.

Required fields:

- `backend_version`
- `web_version`
- `mobile_version`
- `task_id`
- `branch`
- `commit`
- `dirty`
- `built_at`
- `source`
