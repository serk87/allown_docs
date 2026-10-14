# Проверка первого входа — 14 октября 2026

Причина: OAuth issuer добавлял новые identity в DEFAULT_WORKSPACE_ID, а сессии и запрос текущего пользователя требовали членства в пространстве.

Исправлено: новый аккаунт получает сессию без пространства и роли; создание пространства назначает владельца и активирует пространство; при повторном входе выбирается существующее членство пользователя. UI сначала показывает создание пространства и не запрашивает проекты до его создания. Формы переведены на UI kit. Контракт роли допускает пустую строку до создания пространства.

Миграция 0016 разрешает NULL в auth_sessions.workspace_id. Применена штатным механизмом миграций API при RUN_MIGRATIONS=true; dev seed и dev session выключены. Откат старого API потребует завершить сессии без пространства; предпочтительно исправление следующей миграцией.

Проверено:
- Web: 63 теста, сборка TypeScript/Vite.
- PostgreSQL: все тесты auth, в том числе первый и повторный вход Яндекса/VK, сессия и CSRF без пространства, запрет переключения на чужое пространство.
- API выпуска: два разных пользователя, отсутствие членства после входа, создание пространства, роль владельца, создание проекта, изоляция второго пользователя.
- Браузер allown.ru: создание пространства, переход к первому проекту, создание проекта клавишей Enter, пустой список задач.

Релиз: /srv/task-tracker/releases/workspace-onboarding-20261014
Резервная копия перед очисткой: /srv/task-tracker/shared/backups/before-onboarding-reset-20261014.dump

Общий backend-набор имеет 35 прежних падений; каждое имя воспроизведено на версии до исправления. Новых падений нет. Основная причина — старые тесты ожидают демоданные, удалённые из текущего dev seed. Отдельные существующие ошибки: ожидание фиксированного workspace в handler и несколько SQL-команд в prepared statement теста удаления пространства. Полный набор не является зелёным.

- `TestRepositoryCreatesCommentAndWritesPrivacySafeEventIntegration`
- `TestRepositoryHonorsProjectScopeIntegration`
- `TestReplayRepositoryFiltersEventsByTokenProjectScopeIntegration`
- `TestRepositoryCreatesAndListsExportJobsIntegration`
- `TestRepositoryCreatesAndListsImportJobsIntegration`
- `TestRepositoryRejectsReleaseForMilestoneInAnotherProjectIntegration`
- `TestRepositoryCreatesMilestoneAndWritesEventIntegration`
- `TestRepositoryCreatesReleaseAndWritesEventIntegration`
- `TestRepositoryReturnsDefaultPreferencesForWorkspaceMemberIntegration`
- `TestRepositoryUpsertsNotificationPreferencesIntegration`
- `TestRepositoryCreatesInAppNotificationFromTaskEventIntegration`
- `TestRepositoryQueuesEmailWithoutShowingInAppNotificationWhenInAppDisabledIntegration`
- `TestHandlerRejectsWorkspaceMismatch`
- `TestRepositoryCreatesSavedViewAndWritesEventIntegration`
- `TestRepositoryListsSeededSavedViewsIntegration`
- `TestRepositoryReadsSeededTasksIntegration`
- `TestRepositoryReadsTaskDetailByKeyIntegration`
- `TestRepositorySearchesAndFiltersTasksIntegration`
- `TestRepositoryAppliesProjectAccessScopeIntegration`
- `TestRepositoryDoesNotExposeAssigneeOutsideWorkspaceMembershipIntegration`
- `TestRepositoryCreatesTaskAndWritesEventIntegration`
- `TestRepositoryUpdatesTaskAndWritesEventIntegration`
- `TestRepositoryUpdatesTaskAssignmentAndDueDateIntegration`
- `TestRepositoryClearsTaskAssignmentAndDueDateIntegration`
- `TestRepositoryRejectsAssignmentOutsideWorkspaceIntegration`
- `TestRepositoryNoopUpdateDoesNotWriteEventIntegration`
- `TestRepositoryAllowsAnyWorkflowStatusTransitionIntegration`
- `TestRepositoryAssignsTaskToCycleIntegration`
- `TestRepositoryDeletesTaskSoftlyAndWritesEventIntegration`
- `TestRepositoryCreatesTokenAndReturnsPlainValueOnceIntegration`
- `TestRepositoryResolvesBearerTokenSessionIntegration`
- `TestRepositoryRevokesBearerTokenIntegration`
- `TestRepositoryCreatesSubscriptionAndQueuesMatchingEventsIntegration`
- `TestRepositoryListsUpdatesAndRevokesSubscriptionsIntegration`
- `TestRepositoryDeletesWorkspaceCascadeAndMovesOwnerSessionIntegration`
