# Allown — документация

- [Правила работы с репозиториями](product/repository-workflow.md)
- [Требования MVP](superpowers/specs/2026-09-23-task-tracker-design.md)
- [План MVP](superpowers/plans/2026-09-23-task-tracker-mvp-sprints.md)
- [Авторизация](product/auth.md)
- [Приватность аналитики](product/analytics-privacy.md)

## Репозитории

- [Backend](https://github.com/serk87/allown_backend)
- [Web](https://github.com/serk87/allown_web)
- [Лендинг](https://github.com/serk87/allown_landing) — отдельный репозиторий
- [UI kit](https://github.com/serk87/allown_ui)

Для совместной разработки клонируйте их в соседние каталоги `backend`, `web`, `ui-kit`, `landing`, а этот репозиторий — в `docs`. Сохраняйте локальные имена: web использует `file:../ui-kit`, production Dockerfile собирается из общего родительского каталога.

Первичная загрузка сохраняет рабочий снимок backend/web в ветке `import/current-workspace`, а исторические ветки и теги — без перезаписи. Это резервирование текущей разработки, а не подтверждение нового production-релиза. В backend известны прежние падения интеграционных тестов; они описаны в [отчёте](product/onboarding-verification-2026-10-14.md).
