# Шаблон концепции

Parent: [Слой концепций](../README.md)  
Owner issue: `EXEC-001` … `EXEC-007`  
Источник истины: `Concepts/_template/README.md`  
Status: `template`  
Updated: `2026-06-20T17:58:28Z`

## Назначение

Этот файл задаёт минимальный bootstrap и state contract будущей concrete concept. Он не является рабочей концепцией и не получает issue, pages, output или export как живой runtime object.

## Как использовать

1. Открыть [execution protocols](../../Protocols/execution_protocols/README.md).
2. Получить user intent или approved issue, `concept_slug`, title, reason, initial scope и boundary.
3. Проверить [State/execution_index.md](../../State/execution_index.md) и root [page_registry.jsonl](../../State/page_registry.jsonl).
4. Создать только пять operational files из bootstrap set ниже.
5. Заполнить local state и local registries фактическими данными.
6. Обновить root index/root registry и применить [persistence protocol](../../Protocols/common/persistence_protocol.md) одной transaction.

## Минимальные metadata concept README

```text
# <Название концепции>

Parent: [Concepts](../README.md)
Concept slug: <concept_slug>
Owner mode: `Execution Mode`
Источник истины: `Concepts/<concept_slug>/README.md`
Lifecycle status: `draft`
Readiness status: `ready_for_issue_or_page`
Updated: <timestamp>

## Назначение
<Зачем существует концепция.>

## Scope
### Входит
- ...

### Не входит
- ...

## Operational model
<Как концепция работает как система.>

## Pages
<Только фактически созданные pages.>

## Issues
Local registry: `Issues/issue_registry.jsonl`.

## Output
<Ссылки появляются после реального output.>

## Exports
<Ссылки появляются после реального export package.>

## Next-step marker
Next status: `needs_first_issue_or_page`.
```

## Минимальный committed bootstrap

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

`Pages/`, `Output/` и `Exports/` не создаются как empty placeholders. `Issues/_archive/README.md` и `Issues/_tombstones/README.md` создаются только при фактической retention-потребности или отдельном approved bootstrap.

## `State/concept_state.md` schema

```text
# State концепции — <concept_slug>

Parent: [Concept README](../README.md)
Источник истины: `Concepts/<concept_slug>/State/concept_state.md`
Updated: <timestamp>

## State contract

| Field | Value |
|---|---|
| Concept slug | <concept_slug> |
| Lifecycle status | draft |
| State revision | 1 |
| Active issue | none |
| Active phase | not_started |
| Blocking status | none |
| Readiness status | ready_for_issue_or_page |
| Page registry status | parseable_current |
| Issue registry status | parseable_empty |
| Dependency readiness | ready_no_edges |
| Export status | none |
| Integrity status | unverified |
| Integrity basis | README.md; State/concept_state.md@revision=1; State/page_registry.jsonl; Issues/issue_registry.jsonl; Issues/dependency_graph.jsonl |
| Last validation ref | none |
| Last persisted at | <timestamp> |

## Context budget

| Level | Files |
|---|---|
| entry | README, concept_state, local page registry, local issue registry |
| focused | active issue, selected protocol, affected pages |
| expanded | linked issue and direct dependency summaries |

## Next-step marker

Next status: `needs_first_issue_or_page`.
```

Lifecycle status и readiness status хранят разные факты. `Integrity basis` перечисляет реально проверенные files/revisions; hash не требуется и не выдумывается.

## Readiness semantics

| Status | Условие |
|---|---|
| `bootstrap_incomplete` | один из operational files отсутствует или registry не parseable |
| `ready_for_issue_or_page` | bootstrap complete, registries parse, blocker отсутствует |
| `active` | active issue или page work выполняется |
| `ready_for_closure_review` | required work завершён и готов к validation |
| `validated_for_export` | concept-level validation ref зафиксирован; export остаётся отдельным protocol scope |
| `blocked` | следующий шаг остановлен конкретным blocker |

## Integrity semantics

| Status | Условие |
|---|---|
| `unverified` | basis ещё не проверен |
| `verified` | basis проверен на указанной revision/ref |
| `stale` | basis изменился после validation |
| `conflict` | identity, state или registry расходятся |

## Initial local page registry

`Concepts/<concept_slug>/State/page_registry.jsonl` — canonical machine structure map. Он перечисляет только пять реально созданных bootstrap files и позднее добавляет реальные artifacts. Concept README остаётся human entry map. Optional `State/page_map.md` — только derived artifact для большой сети или export.

Минимальная строка для concept entrypoint:

```json
{"path":"README.md","kind":"markdown","parent":null,"owner":"concept","source_of_truth":true,"entrypoint":true,"links":[],"backlinks":[],"orphan":false,"status":"draft"}
```

Mandatory `manifest.jsonl`, `structure.md` и `state.json` не создаются.

## Initial local issue registry

`Concepts/<concept_slug>/Issues/issue_registry.jsonl` использует schema root [issue registry](../../Issues/issue_registry.md), но `scope_type = execution`, `scope_path = Concepts/<concept_slug>/`, а ID имеет формат `<concept_slug>-ISS-0001`.

Initial registry может быть пустым. Не создавать issue только ради заполнения bootstrap.

## Initial dependency graph

`Concepts/<concept_slug>/Issues/dependency_graph.jsonl` хранит concept-local edges и может начинаться с одной metadata row:

```json
{"graph_version":1,"scope_type":"execution","scope_path":"Concepts/<concept_slug>/","edge_count":0,"cycle_status":"clear","updated_at":"<timestamp>"}
```

Root dependency graph не зеркалит concept-internal edges. Cross-scope edge появляется только после созданного service issue и явного reason.

## Service escalation packet

При core defect сохранить локально:

```text
source_concept
source_concept_issue
defect_summary
affected_root_paths
evidence_or_reproduction
safe_local_workaround
requested_service_action
return_anchor
```

Затем установить `Blocking status = service_escalation_required`, остановить затронутую root mutation и запросить переход в Service Mode. Service Mode создаёт root issue; local state/issue и service issue получают bidirectional refs.

## Issue workflow внутри концепции

1. Зафиксировать reason.
2. Провести QA, если unknowns влияют на requirements.
3. Сохранить requirements.
4. Выполнить requalification.
5. Сохранить solution и contract.
6. Выполнить approved work и output.
7. Провести validation.
8. Закрыть issue или вернуть его в нужную phase.

## Closure и export

Closure требует проверки local pages, local issues, dependencies, output coverage, integrity и user approval. Export выполняется через [concept_export_protocol.md](../../Protocols/execution_protocols/concept_export_protocol.md). При open issue разрешён только `work_in_progress`.

## Guardrails

- Не копировать этот шаблон как готовую концепцию без reason и scope.
- Не создавать concrete concept без user intent или approved issue.
- Не создавать empty directories/Markdown placeholders.
- Не смешивать lifecycle, readiness и integrity.
- Не зеркалить local issue rows в root registry.
- Не выполнять root repair из Execution Mode после `service_escalation_required`.
- Не добавлять files, которых нет в local page registry.
