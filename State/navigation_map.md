# Карта навигации

Parent: [README](../README.md)  
Источник истины для связей: [page_registry.jsonl](page_registry.jsonl)  
Machine companion: [file_manifest.jsonl](file_manifest.jsonl)  
Status: `active`  
Updated: `2026-06-16T20:44:45Z`

## Назначение

Этот файл — человекочитаемый companion для быстрого входа в production tree `Concept Builder`. Он не заменяет [page_registry.jsonl](page_registry.jsonl): source of truth для `links`, `backlinks`, `entrypoint` и `orphan` остаётся там. [file_manifest.jsonl](file_manifest.jsonl) даёт компактный machine overview production-файлов, но не решает reachability сам по себе.

Stage 01 сохраняет архитектурный стержень лидера: production root остаётся `/`, `State/page_registry.jsonl` остаётся главным registry связей, а Repo 1 и Repo 2 используются только как доноры точечных идей.

## Root route

1. [README](../README.md) — корневой вход и выбор режима.
2. [service_state.md](service_state.md) — верхнее состояние `Service Mode`.
3. [execution_index.md](execution_index.md) — верхнее состояние `Execution Mode`.
4. [page_registry.jsonl](page_registry.jsonl) — проверка parent, links, backlinks и orphan-status.
5. [file_manifest.jsonl](file_manifest.jsonl) — compact machine companion для production-файлов.
6. [Protocols/catalog.md](../Protocols/catalog.md) — выбор ближайшего локального protocol.

## Service route

1. [README](../README.md)
2. [service_state.md](service_state.md)
3. [../Instructions/service_mode_project_instructions.md](../Instructions/service_mode_project_instructions.md)
4. [../Protocols/catalog.md](../Protocols/catalog.md)
5. [../Protocols/service_protocols/service_start_protocol.md](../Protocols/service_protocols/service_start_protocol.md)
6. По trigger: `new_issue`, `existing_issue`, QA, requirements, solution/contract/output, complex/linked issue или retention protocol.
7. Для issue state: [../Issues/issue_registry.md](../Issues/issue_registry.md), [../Issues/issue_registry.jsonl](../Issues/issue_registry.jsonl), [../Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl).

## Execution route

1. [README](../README.md)
2. [execution_index.md](execution_index.md)
3. [../Instructions/execution_mode_project_instructions.md](../Instructions/execution_mode_project_instructions.md)
4. [../Concepts/README.md](../Concepts/README.md)
5. [../Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md)
6. [../Concepts/_template/README.md](../Concepts/_template/README.md) используется только после явного `concept_slug`, title, reason и initial scope.
7. [../Protocols/execution_protocols/concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md) применяется только для export-запроса или closure/export routing.

## Persistence / validation route

1. Перед production write открыть [../Protocols/common/persistence_protocol.md](../Protocols/common/persistence_protocol.md).
2. Зафиксировать write set и branch/write scope.
3. Сначала сохранить primary artifacts.
4. Затем обновить indexes: [page_registry.jsonl](page_registry.jsonl), affected state, issue registry и dependency graph, если они действительно затронуты.
5. Последним добавить запись в [persistence_log.jsonl](persistence_log.jsonl).
6. Перед финальным статусом применить [../Protocols/common/final_validation_protocol.md](../Protocols/common/final_validation_protocol.md) и обновить [service_validation_report.md](service_validation_report.md), если scope — root service.

## Проверка сирот

1. Распарсить [page_registry.jsonl](page_registry.jsonl) построчно как JSONL.
2. Распарсить [file_manifest.jsonl](file_manifest.jsonl) построчно как JSONL.
3. Для каждого Markdown-файла проверить parent link в header.
4. Сравнить `links` и `backlinks` в page registry с фактическими relative links.
5. Проверить, что `orphan=false` имеет маршрут из root route, service route или execution route.
6. Если файл есть в manifest, но отсутствует в page registry, считать это navigation mismatch до объяснения.
7. Если файл есть в page registry, но отсутствует в manifest, обновить manifest или записать reason в validation report.

## Что не является production route

Следующие материалы не являются рабочими route в production tree и не должны попадать сюда как target paths Stage 01:

- dev-only archives;
- handoff packages;
- checkpoint drafts;
- prompt packages;
- audit source notes;
- Phase 2 / P2R5 repair-history materials;
- wholesale donor repo layouts;
- service scripts без отдельного approved issue;
- runtime concept folders без `concept_slug`, title, reason и initial scope.

## Stage 01 boundary

Stage 01 добавляет только navigation companions и точечную style-gate правку. Он не меняет persistence/state/export/issue lifecycle широко, не создаёт service scripts и не создаёт real concept runtime folders.
