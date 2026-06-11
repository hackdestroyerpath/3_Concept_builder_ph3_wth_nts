# Сервисное состояние

Parent: [README](../README.md)  
Owner issue: `EXEC-004`  
Источник истины: `State/service_state.md`  
Status: `commit_ready_package`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот файл хранит верхнее состояние `Concept Builder Service Mode`. Его читают при старте обслуживания системы, перед изменением структуры репозитория и перед переходом к service-level issue.

## Текущий service scope

| Поле | Значение |
|---|---|
| Mode | `Service Mode` |
| Active scope | `root_service` |
| Active issue | `none` |
| Active phase | `none` |
| Repository | `hackdestroyerpath/3_Concept_builder_ph3_wth_nts` |
| Default branch | `main` |
| Write status | `committed_via_github_connector` |
| GitHub metadata | `verified` |
| Validation status | `pass_with_deferred_items` |
| Validation report | [service_validation_report.md](service_validation_report.md) |
| Blocking status | `none` |

## Минимальная загрузка при старте `Service Mode`

1. [../README.md](../README.md) — корневой вход и карта маршрутов.
2. [service_state.md](service_state.md) — текущий service-scope, validation status и next-step marker.
3. [page_registry.jsonl](page_registry.jsonl) — проверка страниц, parent, backlinks и orphan-status.
4. [../Protocols/catalog.md](../Protocols/catalog.md) — выбор локального протокола.
5. [../Protocols/service_protocols/service_start_protocol.md](../Protocols/service_protocols/service_start_protocol.md) — старт `Service Mode`.
6. [../Protocols/common/context_loading_protocol.md](../Protocols/common/context_loading_protocol.md) — focus packet.
7. [../Protocols/common/persistence_protocol.md](../Protocols/common/persistence_protocol.md) — запись и честный статус.
8. [../Protocols/common/final_validation_protocol.md](../Protocols/common/final_validation_protocol.md) — финальная проверка.
9. [../Issues/issue_registry.md](../Issues/issue_registry.md) — человекочитаемый issue registry.
10. [../Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl) — dependency graph.

## Context budget

| Уровень | Что читать |
|---|---|
| `entry` | root README, service state, page registry, protocol catalog |
| `focused` | active issue, selected protocol, affected files |
| `expanded` | direct dependency summaries and parent summary |
| `full_scope` | только для final validation или approved refactor |
| `repository_wide` | запрещён без explicit reason |

## Routing

| Trigger | Protocol |
|---|---|
| старт / пинг | [service_start_protocol.md](../Protocols/service_protocols/service_start_protocol.md) |
| новый issue | [new_issue_protocol.md](../Protocols/service_protocols/new_issue_protocol.md) |
| существующий issue | [existing_issue_protocol.md](../Protocols/service_protocols/existing_issue_protocol.md) |
| QA | [question_answer_protocol.md](../Protocols/service_protocols/question_answer_protocol.md) |
| requirements | [requirements_protocol.md](../Protocols/service_protocols/requirements_protocol.md) |
| solution/output | [solution_contract_output_protocol.md](../Protocols/service_protocols/solution_contract_output_protocol.md) |
| complex/decomposition | [complex_issue_protocol.md](../Protocols/service_protocols/complex_issue_protocol.md) |
| linked dependencies | [linked_issues_protocol.md](../Protocols/service_protocols/linked_issues_protocol.md) |
| retention/archive/tombstone | [issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md) |

## Deferred items

| ID | Status | Next action |
|---|---|---|
| `USER-001` | `deferred_non_blocking` | Оценивать служебные скрипты только через отдельный approved issue при реальной необходимости. |

## Production boundary

Production tree ограничен папкой `Concept Builder/` и содержит только рабочие источники: `README.md`, `State/`, `Instructions/`, `Protocols/`, `Issues/`, `Inbox/`, `Concepts/`. Development handoff, checkpoint reports, audit сырьё и локальные архивы не являются production files.

## Next-step marker

Next status: `ready_for_github_validation_or_runtime_concept_request`.

Следующий service шаг: принимать новые service issue, runtime concept request или повторную финальную проверку после апгрейда compact files.
