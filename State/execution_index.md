# Индекс execution-объектов

Parent: [README](../README.md)  
Owner issue: `EXEC-001` … `EXEC-007`  
Источник истины: `State/execution_index.md`  
Status: `active`  
Updated: `2026-06-20T19:24:43Z`

## Назначение

Этот файл является верхним индексом `Execution Mode`. Он фиксирует active concept, startup case, readiness и integrity execution-слоя и маршрутизирует к локальному state выбранной концепции.

## Текущий execution state

| Поле | Значение |
|---|---|
| Mode | `Execution Mode` |
| Bootstrap contract revision | `execution-bootstrap-v1` |
| Service base | `State/service_state.md / Stage 04 accepted` |
| Active concept | `none` |
| Active concept slug | `none` |
| Startup case | `no_active_concept` |
| Active issue | `none` |
| Active phase | `not_started` |
| Readiness status | `ready_for_concept_selection_or_creation` |
| Integrity status | `verified` |
| Integrity basis | `State/execution_index.md; Protocols/execution_protocols/README.md; Concepts/README.md; Concepts/_template/README.md; Protocols/catalog.jsonl; State/page_registry.jsonl; contract=execution-bootstrap-v1` |
| Last validation ref | `State/persistence_log.jsonl#tx-cb-execution-bootstrap-v1-20260620` |
| Blocking status | `none` |
| Project instructions | `Instructions/execution_mode_project_instructions.md` |
| Execution protocols | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| Concepts entry | [Concepts/README.md](../Concepts/README.md) |
| Concept template | `Concepts/_template/README.md` |
| Export protocol | `Protocols/execution_protocols/concept_export_protocol.md` |

Конкретные концепции не созданы. Агент не создаёт `Concepts/<concept_slug>/` автоматически: нужны явный пользовательский intent или approved issue, а также slug, title, reason, initial scope и boundary.

## Реестр концепций

| Slug | Title | Lifecycle status | Readiness status | Integrity status | Active issue | State path | Local page registry | Local issue registry | Export status | Last validation ref | Blocking status |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `none` | Концепции ещё не созданы | `not_applicable` | `ready_for_concept_selection_or_creation` | `verified` | `none` | `none` | `none` | `none` | `none` | `none` | `none` |

Для будущей реальной строки lifecycle, readiness и integrity являются разными полями. Значения readiness/integrity в root registry — derived mirror локального `State/concept_state.md` и обновляются с ним одной transaction.

## Startup cases

### `no_active_concept`

Показать безопасные варианты: перечислить существующие концепции, выбрать существующую или подготовить создание новой. Папка не создаётся до полного creation gate.

### `active_known`

Когда slug и state path согласованы, загрузить concept README, локальные `State/concept_state.md`, `State/page_registry.jsonl`, `Issues/issue_registry.jsonl`, прямые зависимости и восстановить active issue, phase и следующий protocol. Root context сохраняется минимальным.

### `active_unknown`

Восстановить identity по этому индексу и `State/page_registry.jsonl`. До совпадения slug, state path и локального state mutation запрещена. Конфликт возвращается как bounded blocker `active_concept_identity_conflict` с перечислением расходящихся evidence.

## Минимальный bootstrap будущей концепции

Первичная operation готовит только operational files:

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

`Pages/`, `Output/` и `Exports/` появляются с первым реальным artifact. Retention entrypoints создаются только при фактической retention-потребности или отдельном approved bootstrap. Пустые Markdown placeholders запрещены.

Локальный `State/page_registry.jsonl` — canonical machine-readable structure map. Concept README остаётся human entry map. `State/page_map.md` допустим только как derived artifact для большой сети или export; обязательные `manifest.jsonl`, `structure.md` и `state.json` не вводятся.

## Readiness и integrity transition

До persistence/readback пяти operational files локальный `State/concept_state.md` обязан показывать:

```text
Lifecycle status: draft
Readiness status: bootstrap_incomplete
Integrity status: unverified
Last persisted at: null
Next status: needs_bootstrap_persistence
```

В этом состоянии разрешены только completion/recovery bootstrap. Issue/page mutation запрещена.

`ready_for_issue_or_page` допустим только после существования всех пяти files, parse JSONL, совпадения identity, успешной persistence и readback. Тогда локальный state получает `Integrity status: verified`, factual `Last persisted at`, exact integrity basis с state revision и persistence/readback ref и `Next status: needs_first_issue_or_page`.

`unverified`, `stale` или `conflict` блокируют issue/page mutation, кроме bounded recovery. Локальный `State/concept_state.md` является authoritative source; root registry хранит только derived mirror.

## Local issue isolation и service escalation

Concept-internal issue и dependency rows остаются внутри `Concepts/<concept_slug>/Issues/`. Root issue registry не зеркалит их.

Канонический anchor escalation:

```text
Concepts/<concept_slug>/State/concept_state.md#pending-service-escalation
```

Локальный state хранит `service_escalation_status`, `service_escalation_ref`, `service_issue_id`, `created_at` и `updated_at`, а anchor содержит:

```text
source_concept
source_concept_issue
defect_summary
affected_root_paths
evidence_or_reproduction
safe_local_workaround
requested_service_action
return_anchor
created_at
updated_at
service_escalation_status
service_issue_id
```

`source_concept_issue` может быть `none` только вне active issue work. Если local issue существует, его registry/state хранит тот же `service_escalation_ref` и `return_anchor`. Execution Mode записывает только concept-local state/issue records и прекращает затронутую root mutation. Service Mode создаёт root service issue; затем root и concept scopes получают bidirectional refs одной controlled transaction. Resolution/cancellation обновляет тот же anchor, а не создаёт ad-hoc file.

## Next-step marker

Execution next status: `ready_for_concept_selection_or_creation`.

Следующий runtime-шаг: выбрать существующую концепцию либо получить slug, title, reason, initial scope и boundary для новой.
