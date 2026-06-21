# Шаблон концепции

Parent: [Слой концепций](../README.md)  
Owner issue: `EXEC-011`  
Источник истины: `Concepts/_template/README.md`  
Status: `template`  
Updated: `2026-06-20T23:57:36Z`

## Назначение

Этот файл задаёт минимальный bootstrap и state contract будущей concrete concept. Он не является рабочей концепцией и не получает issue, pages, output или export как live runtime object.

## Как использовать

1. Открыть [execution protocols](../../Protocols/execution_protocols/README.md).
2. Получить user intent или approved issue, `concept_slug`, title, reason, initial scope и boundary.
3. Проверить [State/execution_index.md](../../State/execution_index.md) и root [page_registry.jsonl](../../State/page_registry.jsonl).
4. Подготовить пять operational files из bootstrap set ниже.
5. Заполнить local state в prewrite состоянии `bootstrap_incomplete + unverified`.
6. Записать files/root companions через [persistence protocol](../../Protocols/common/persistence_protocol.md), выполнить readback и JSONL/identity checks.
7. Только после успешной проверки обновить concept state до verified readiness, применить [persistence protocol](../../Protocols/common/persistence_protocol.md) и прочитать state повторно.

## Минимальные metadata concept README

Default concept README намеренно не хранит dynamic readiness/integrity metadata. Authoritative значения находятся в `State/concept_state.md`.

```text
# <Название концепции>

Parent: [Concepts](../README.md)
Concept slug: <concept_slug>
Owner mode: `Execution Mode`
Источник истины: `Concepts/<concept_slug>/README.md`
State source: `State/concept_state.md`
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
```

Если human-facing readiness mirror позже добавляется в README, он маркируется как derived from `State/concept_state.md` и обновляется с state одной transaction.

## Минимальный bootstrap contract

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

`Pages/`, `Output/` и `Exports/` не создаются как empty placeholders. Retention entrypoints создаются только при фактической retention-потребности или отдельном approved bootstrap.

## `State/concept_state.md` prewrite schema

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
| Blocking status | bootstrap_persistence_required |
| Readiness status | bootstrap_incomplete |
| Page registry status | pending_persistence |
| Issue registry status | pending_persistence |
| Dependency readiness | pending_persistence |
| Export status | none |
| Integrity status | unverified |
| Integrity basis | pending bootstrap readback |
| Last validation ref | none |
| Last persisted at | null |
| Service escalation status | none |
| Service escalation ref | none |
| Service issue ID | none |

## Next-step marker

Next status: needs_bootstrap_persistence.

<a id="pending-service-escalation"></a>
## Pending service escalation

Status: none
```

В prewrite состоянии разрешены только bootstrap completion/recovery.

## Post-persistence verified state

После существования всех пяти files, line-by-line JSONL parse, identity agreement, successful persistence и readback обновить state:

```text
| Lifecycle status | draft |
| Blocking status | none |
| Readiness status | ready_for_issue_or_page |
| Page registry status | parseable_current |
| Issue registry status | parseable_current |
| Dependency readiness | ready_no_blocking_edges |
| Integrity status | verified |
| Integrity basis | README.md; State/concept_state.md@revision=<n>; State/page_registry.jsonl; Issues/issue_registry.jsonl; Issues/dependency_graph.jsonl; persistence/readback=<ref> |
| Last validation ref | <persistence/readback ref> |
| Last persisted at | <factual timestamp> |

Next status: needs_first_issue_or_page.
```

После update state читается повторно. `ready_for_issue_or_page` допустим только при `Integrity status = verified`. `unverified`, `stale` или `conflict` блокируют issue/page mutation, кроме bounded recovery.

## Readiness semantics

| Status | Условие |
|---|---|
| `bootstrap_incomplete` | operational files ещё не persisted/read back либо registry/identity check не прошёл |
| `ready_for_issue_or_page` | five-file bootstrap persisted, JSONL/identity/readback pass и integrity verified |
| `active` | active issue или page work выполняется при verified integrity |
| `ready_for_closure_review` | required work завершён и готов к validation |
| `validated_for_export` | concept-level validation ref зафиксирован |
| `blocked` | следующий шаг остановлен конкретным blocker |

## Integrity semantics

| Status | Условие |
|---|---|
| `unverified` | bootstrap/readback basis ещё не проверен |
| `verified` | exact basis проверен на указанной state revision/ref |
| `stale` | basis изменился после validation |
| `conflict` | identity, state или registry расходятся |

Hash не обязателен. Если он используется, basis и способ вычисления документируются; fake/self-referential hash запрещён.

## Initial local page registry

`Concepts/<concept_slug>/State/page_registry.jsonl` — canonical machine structure map. Он перечисляет только пять реально созданных bootstrap files и позднее добавляет реальные artifacts. Concept README остаётся human entry map. Optional `State/page_map.md` — derived artifact для большой сети или export.

Минимальная строка concept entrypoint:

```json
{"path":"README.md","kind":"markdown","parent":null,"owner":"concept","source_of_truth":true,"entrypoint":true,"links":[],"backlinks":[],"orphan":false,"status":"draft"}
```

Mandatory `manifest.jsonl`, `structure.md` и `state.json` не создаются.

## Initial local issue registry

`Concepts/<concept_slug>/Issues/issue_registry.jsonl` использует schema root [issue registry](../../Issues/issue_registry.md), но `scope_type = execution`, `scope_path = Concepts/<concept_slug>/`, а ID имеет формат `<concept_slug>-ISS-0001`.

Initial registry может быть пустым. Не создавать issue только ради заполнения bootstrap.

## Initial dependency graph

`Concepts/<concept_slug>/Issues/dependency_graph.jsonl` при отсутствии edges содержит ровно одну metadata row:

```json
{
  "record_type": "metadata",
  "graph_id": "concept_<concept_slug>_dependency_graph",
  "scope_type": "execution",
  "scope_path": "Concepts/<concept_slug>/",
  "source_registry": "Issues/issue_registry.jsonl",
  "node_count": 0,
  "edge_count": 0,
  "cycle_check": "pass",
  "cycle_nodes": [],
  "updated_at": "<timestamp>"
}
```

Rules:

- `cycle_status` не используется;
- при отсутствии edges metadata row является единственной initial row;
- будущие edges используют `record_type=edge` и canonical Linked Issues relation/readiness vocabulary;
- при изменении issues или edges обновляются `node_count`, `edge_count`, `cycle_check`, `cycle_nodes` и `updated_at`;
- local graph остаётся concept-local;
- root graph не зеркалит concept-internal edges без реальной cross-scope причины.

## Service escalation state contract

Canonical anchor:

```text
Concepts/<concept_slug>/State/concept_state.md#pending-service-escalation
```

State fields:

```text
Service escalation status: none | pending_service_mode | service_issue_created | resolved | cancelled
Service escalation ref: State/concept_state.md#pending-service-escalation | none
Service issue ID: <root service issue id> | none
```

Generated `State/concept_state.md` keeps the same anchor:

```text
<a id="pending-service-escalation"></a>
## Pending service escalation

source_concept: <concept_slug>
source_concept_issue: <local issue id | none>
defect_summary: <summary>
affected_root_paths: <paths>
evidence_or_reproduction: <evidence>
safe_local_workaround: <workaround | none>
requested_service_action: <action>
return_anchor: <concept/local issue anchor>
created_at: <timestamp>
updated_at: <timestamp>
service_escalation_status: pending_service_mode
service_issue_id: none
```

Rules:

- `source_concept_issue` может быть `none` только если defect найден вне active issue work;
- при local issue его registry/state хранит тот же `service_escalation_ref` и `return_anchor`;
- Execution Mode пишет только concept-local state/issue records, устанавливает `service_escalation_required` и прекращает затронутую root mutation;
- Service Mode создаёт root service issue;
- после создания root и concept scopes получают bidirectional refs одной controlled transaction;
- resolution/cancellation обновляет этот же anchor, status и timestamps; ad-hoc escalation file не создаётся.

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
- Не выставлять ready state до verified persistence/readback.
- Не зеркалить local issue rows в root registry.
- Не выполнять root repair из Execution Mode после `service_escalation_required`.
- Не добавлять files, которых нет в local page registry.
