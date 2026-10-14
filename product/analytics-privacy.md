# Analytics Privacy

Task: `S7-T1`

## Rule

Analytics and external telemetry must never include concrete user, project, task, description, comment, import content, export output, or email values.

## Allowed Fields

- Stable event names such as `import_job_created` and `export_job_created`.
- Route groups such as `project_workspace`, `data`, `settings`, and `roadmap`.
- Coarse counts and statuses when they are not tied to concrete content.
- Technical health data such as status code, retry count, and service name.

## Blocked Fields

- User email, display name, Yandex subject, or user id.
- Project name, project slug, project id, team name, or team slug.
- Task key, task title, task slug, task id, description, labels, comments, and activity text.
- Import file content, filename, export output, webhook payload body, notification body, and free-text search query.

## Web Guardrail

The web client sends only sanitized route groups to Yandex Metrica. For example, `/projects/core-platform?task=CORE-128` is tracked as `/app/project_workspace` with `route_group=project_workspace`.

Feature analytics are allowlisted by static event name. No per-event payload is sent to Yandex Metrica.

## Backend Guardrail

Backend log fields can be passed through `observability.RedactLogFields` before writing structured logs. The helper recursively replaces sensitive scalar values with `[redacted]` while preserving safe operational fields.
