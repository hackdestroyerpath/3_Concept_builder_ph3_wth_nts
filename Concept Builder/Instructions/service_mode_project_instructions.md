# Project instructions для `Service Mode`

Parent: [README](../README.md)
Status: `commit_ready_source`
Updated: `2026-06-05T11:45:45Z`

## Назначение

Инструкции для обслуживания структуры `Concept Builder`: State, Protocols, Issues, Inbox, Concepts entry, validation и persistence.

## Правила работы

1. Начинай с [../README.md](../README.md), [../State/service_state.md](../State/service_state.md), [../Protocols/catalog.md](../Protocols/catalog.md).
2. Выбирай ближайший protocol по текущему trigger, mode, issue status и phase.
3. Любой новый файл должен иметь owner, parent link, source of truth и registry entry.
4. Перед записью применяй [persistence_protocol.md](../Protocols/common/persistence_protocol.md).
5. Перед закрытием issue применяй [final_validation_protocol.md](../Protocols/common/final_validation_protocol.md).
6. Не смешивай service-layer и concept-layer без explicit cross-scope reason.
7. GitHub commit/push не считается выполненным без фактического commit marker.

## Связанные файлы

- [Execution instructions](execution_mode_project_instructions.md)
- [Service state](../State/service_state.md)
- [Protocol catalog](../Protocols/catalog.md)
- [Issue registry](../Issues/issue_registry.md)
- [Page registry](../State/page_registry.jsonl)
