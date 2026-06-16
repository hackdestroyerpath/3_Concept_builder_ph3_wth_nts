# Индекс execution-объектов

Parent: [README](../README.md)  
Owner issue: `EXEC-004`  
Источник истины: `State/execution_index.md`  
Status: `pass_with_deferred_items`  
Updated: `2026-06-16T16:48:11Z`

## Назначение

Этот файл является верхним индексом `Execution Mode`. Он показывает, какие концепции существуют, какая концепция активна и где искать локальный state конкретной концепции.

## Текущий execution state

| Поле | Значение |
|---|---|
| Mode | `Execution Mode` |
| Active concept | `none` |
| Active concept slug | `none` |
| Active issue | `none` |
| Active phase | `not_started` |
| Project instructions | [../Instructions/execution_mode_project_instructions.md](../Instructions/execution_mode_project_instructions.md) |
| Protocol catalog | [../Protocols/catalog.md](../Protocols/catalog.md) |
| Execution protocols | [../Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| Concepts entry | [../Concepts/README.md](../Concepts/README.md) |
| Concept template | [../Concepts/_template/README.md](../Concepts/_template/README.md) |
| Export protocol | [../Protocols/execution_protocols/concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md) |
| Blocking status | `none` |

Пока конкретные концепции не созданы. Агент не должен создавать папку концепции без явного пользовательского запроса или service/execution issue с владельцем, reason, requirements и критерием закрытия.

Production root: корень репозитория `/`. Execution paths разрешаются от корневых каталогов `Concepts/`, `Protocols/`, `State/` и `Issues/`.

## Реестр концепций

| Concept slug | Название | Status | Active issue | State path |
|---|---|---|---|---|
| `none` | Концепции ещё не созданы | `empty` | `none` | `none` |

## Правило создания новой концепции

Новая концепция создаётся через [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) и [Concepts/_template/README.md](../Concepts/_template/README.md). Минимальный набор внутри `Concepts/<concept_slug>/`:

```text
README.md
State/concept_state.md
State/page_registry.jsonl
Issues/issue_registry.jsonl
Issues/dependency_graph.jsonl
Issues/_archive/README.md
Issues/_tombstones/README.md
Pages/
Output/
Exports/
```

Название концепции может быть русским в заголовках, но slug и пути должны оставаться ASCII-safe.

## Минимальная загрузка при старте `Execution Mode`

1. [../README.md](../README.md) — корневой вход.
2. [execution_index.md](execution_index.md) — этот индекс.
3. [page_registry.jsonl](page_registry.jsonl) — проверка существования root и concept entrypoint pages.
4. [../Concepts/README.md](../Concepts/README.md) — вход слоя концепций.
5. [../Protocols/catalog.md](../Protocols/catalog.md) — выбор protocol по mode/state/phase.
6. [../Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) — правила concept scope и path mapping.
7. [../Protocols/common/context_loading_protocol.md](../Protocols/common/context_loading_protocol.md) — минимальный focus packet.
8. После выбора концепции: локальный `Concepts/<concept_slug>/README.md` и `Concepts/<concept_slug>/State/concept_state.md`.

## Issue registry readiness

Root issue model создана в [../Issues/issue_registry.md](../Issues/issue_registry.md) и [../Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl). Concrete concept получает локальные registry и dependency graph при создании. Root registry не хранит concept-internal задачи без cross-scope reason.

## Export readiness

Concept export доступен через [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md). Closed export требует concept validation и user approval. Если open issue остаются, допустим только `work_in_progress` export с явным списком blockers.

## Связь с service state

Service-level развитие execution-слоя отслеживается в [service_state.md](service_state.md). Если работа над концепцией выявляет дефект core-системы, агент создаёт service-level issue вместо молчаливого изменения root-протоколов из execution-context.

## Next-step marker

Execution next status: `ready_for_concept_selection_or_creation`.

Следующий шаг в `Execution Mode`: получить от пользователя concept slug / title / reason или выбрать существующую концепцию. Следующий шаг в `Service Mode`: открыть новый service issue через lifecycle. Root-flatten repair записан напрямую в `main`, поэтому concrete concept folders не создаются без `concept_slug`.
