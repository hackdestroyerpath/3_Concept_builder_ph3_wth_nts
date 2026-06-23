# Concept Builder

Статус: `repair_candidate_pending_manual_reviewer`  
Источник истины этого входа: `README.md`  
Последнее обновление: `2026-06-22T21:56:58+02:00`  
Реестр страниц: [State/page_registry.jsonl](State/page_registry.jsonl)  
Карта навигации: [State/navigation_map.md](State/navigation_map.md)  
Финальная проверка: [State/service_validation_report.md](State/service_validation_report.md)

## Назначение

`Concept Builder` — рабочая сеть файлов для совместного построения концепций через ChatGPT и GitHub. Система разделяет два режима:

1. `Concept Builder Service Mode` обслуживает root `State`, project instructions, protocols, issue-модель, navigation, retention и validation.
2. `Concept Builder / Execution Mode` развивает конкретные концепции внутри [Concepts/](Concepts/README.md).

Production root — корень репозитория `/`. Материалы задачи, checkpoint archives, prompts, handoff и иное development-only сырьё не входят в production boundary.

## Текущая repository truth

```yaml
repository: hackdestroyerpath/3_Concept_builder_ph3_wth_nts
verified_base_main: 194970c7c5ac37f2dbbfcd51256caaa46f67f8f9
latest_accepted_implementation: Stage 06 merged in PR #21
stage_07_status: repair_candidate_pending_manual_reviewer
base_residual_ids: [RES-CLOSURE-001, RES-LANG-001]
candidate_residual_ids: []
closure_candidate: true
closure_allowed: false
```

Implementation through merged PR `#21` присутствует в `main`. Коммит `194970c7c5ac37f2dbbfcd51256caaa46f67f8f9` изменил только [State/service_validation_report.md](State/service_validation_report.md) и добавил неподтверждённое cross-file утверждение о финальном закрытии. Эта формулировка superseded данным bounded repair candidate.

`candidate_residual_ids=[]` и `closure_candidate=true` относятся только к полностью записанному и перечитанному head ветки `agent/cbu-current-main-truth-language-repair-20260622`. Они не означают merge или final closure. `closure_allowed=false` сохраняется до manual reviewer acceptance, squash merge, fresh `main` readback и отдельного final closure transition.

## Старт агента

1. Открой этот `README.md` как root entry map.
2. Определи режим:
   - Service Mode: [State/service_state.md](State/service_state.md) и [service start protocol](Protocols/service_protocols/service_start_protocol.md);
   - Execution Mode: [State/execution_index.md](State/execution_index.md), [Concepts/README.md](Concepts/README.md) и [execution protocols](Protocols/execution_protocols/README.md).
3. Проверь target и owner через [State/page_registry.jsonl](State/page_registry.jsonl).
4. Выбери самый локальный available route через [Protocols/catalog.md](Protocols/catalog.md) и [Protocols/catalog.jsonl](Protocols/catalog.jsonl).
5. Перед production write примени [persistence protocol](Protocols/common/persistence_protocol.md) и проверь [State/persistence_log.jsonl](State/persistence_log.jsonl).
6. Перед closure, commit package или export примени [final validation protocol](Protocols/common/final_validation_protocol.md).

## Service route

- новый service issue: [new issue protocol](Protocols/service_protocols/new_issue_protocol.md) и [Inbox rules](Inbox/README.md);
- существующий issue: [existing issue protocol](Protocols/service_protocols/existing_issue_protocol.md);
- materially important unknowns: [Question Answer protocol](Protocols/service_protocols/question_answer_protocol.md);
- requirements: [requirements protocol](Protocols/service_protocols/requirements_protocol.md);
- solution, contract и output: [solution/contract/output protocol](Protocols/service_protocols/solution_contract_output_protocol.md);
- decomposition: [complex issue protocol](Protocols/service_protocols/complex_issue_protocol.md);
- dependency edges: [linked issues protocol](Protocols/service_protocols/linked_issues_protocol.md);
- archive, tombstone и cleanup: [issue retention protocol](Protocols/service_protocols/issue_retention_protocol.md).

Root issue sources: [human registry](Issues/issue_registry.md), [machine registry](Issues/issue_registry.jsonl) и [dependency graph](Issues/dependency_graph.jsonl). Archive и tombstone entrypoints: [Issues/_archive/README.md](Issues/_archive/README.md), [Issues/_tombstones/README.md](Issues/_tombstones/README.md).

## Execution route

- active concept и readiness: [State/execution_index.md](State/execution_index.md);
- concept layer: [Concepts/README.md](Concepts/README.md);
- bootstrap specification: [Concepts/_template/README.md](Concepts/_template/README.md);
- export precheck/WIP/closed package: [concept export protocol](Protocols/execution_protocols/concept_export_protocol.md).

Concrete concept folder не создаётся без explicit intent, `concept_slug`, title, reason, initial scope и boundary.

## Production-области

| Область | Назначение | Текущий статус |
|---|---|---|
| `State/` | state, registries, persistence, backlog, validation | repair candidate pending manual reviewer |
| `Instructions/` | compact loaders для двух режимов | available |
| `Protocols/` | common/service/execution routing | available |
| `Issues/` | root issue model, dependency graph, retention | available; `USER-001` deferred/nonblocking |
| `Inbox/` | input и attachment traceability | available-empty-entrypoint |
| `Concepts/` | concept-local work и exports | available-empty-layer |

## Navigation и integrity

- Каждый production Markdown path должен быть достижим из root/service/execution route либо зарегистрирован в [State/page_registry.jsonl](State/page_registry.jsonl).
- Каждый дочерний Markdown-файл имеет parent route.
- Actual exact-case links, registry `links` и reciprocal `backlinks` проверяются одной bounded transaction.
- Новый operational file требует owner, purpose, source-of-truth role и lifecycle.
- Нельзя выдавать `passed`, `ready`, `committed`, `closed` или `final` без factual persistence/readback evidence.

## Current next step

Следующий gate: manual review текущего seven-file PR candidate. Merge, fresh `main` readback, donor deletion readiness и eventual positive final-closure transition находятся вне этой executor-задачи.
