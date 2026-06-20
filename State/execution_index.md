# Индекс execution-объектов

Parent: [README](../README.md)  
Owner issue: `EXEC-001` … `EXEC-007`  
Источник истины: `State/execution_index.md`  
Status: `stage_05a_branch_result_pending_manual_reviewer`  
Updated: `2026-06-20T17:58:28Z`

## Назначение

Этот файл является верхним индексом `Execution Mode`. Он фиксирует активную концепцию, startup case, readiness и integrity execution-слоя, а также маршрутизирует к локальному state выбранной концепции.

## Текущий execution state

| Поле | Значение |
|---|---|
| Mode | `Execution Mode` |
| Bootstrap contract revision | `stage_05a-r1` |
| Accepted base | `Stage 04 accepted in main`; `main` head `1836d819986c7bf8e73d18ef7ac1e0f489042665` |
| Stage 05A branch status | `stage_05a_branch_result_pending_manual_reviewer` |
| Active concept | `none` |
| Active concept slug | `none` |
| Startup case | `no_active_concept` |
| Active issue | `none` |
| Active phase | `not_started` |
| Readiness status | `ready_for_concept_selection_or_creation` |
| Integrity status | `provisional_branch_validation` |
| Integrity basis | `State/execution_index.md`, `Protocols/execution_protocols/README.md`, `Concepts/README.md`, `Concepts/_template/README.md`, `Protocols/catalog.jsonl`, `State/page_registry.jsonl` |
| Last validation ref | `pending_phase_3_branch_readback_and_manual_reviewer` |
| Blocking status | `none` |
| Project instructions | `Instructions/execution_mode_project_instructions.md` |
| Protocol catalog | `Protocols/catalog.md` |
| Execution protocols | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| Concepts entry | [Concepts/README.md](../Concepts/README.md) |
| Concept template | `Concepts/_template/README.md` |
| Export protocol | `Protocols/execution_protocols/concept_export_protocol.md` |

Конкретные концепции не созданы. Агент не создаёт `Concepts/<concept_slug>/` автоматически: нужны явный пользовательский intent или approved issue, а также slug, title, reason, initial scope и boundary.

## Реестр концепций

| Slug | Title | Lifecycle status | Readiness status | Active issue | State path | Local page registry | Local issue registry | Export status | Last validation ref | Blocking status |
|---|---|---|---|---|---|---|---|---|---|---|
| `none` | Концепции ещё не созданы | `not_applicable` | `ready_for_concept_selection_or_creation` | `none` | `none` | `none` | `none` | `none` | `none` | `none` |

Для будущей реальной строки `slug` и пути остаются ASCII-safe. `Lifecycle status` описывает жизненный цикл концепции, а `Readiness status` — возможность безопасно выполнить следующий шаг; эти поля не взаимозаменяемы.

## Startup cases

### `no_active_concept`

Показать безопасные варианты: перечислить существующие концепции, выбрать существующую или подготовить создание новой. Папка не создаётся до полного creation gate.

### `active_known`

Когда slug и state path согласованы, загрузить concept README, локальные `State/concept_state.md`, `State/page_registry.jsonl`, `Issues/issue_registry.jsonl`, прямые зависимости и восстановить active issue, phase и следующий protocol. Root context сохраняется минимальным.

### `active_unknown`

Восстановить identity по этому индексу и `State/page_registry.jsonl`. До совпадения slug, state path и локального state mutation запрещена. Конфликт возвращается как bounded blocker `active_concept_identity_conflict` с перечислением расходящихся evidence.

## Минимальный committed bootstrap будущей концепции

Первичная transaction создаёт только реально используемые operational files:

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

`Pages/`, `Output/` и `Exports/` являются логическими областями и появляются с первым реальным artifact. Retention entrypoints `Issues/_archive/README.md` и `Issues/_tombstones/README.md` создаются только при фактической retention-потребности или отдельном approved bootstrap. Пустые Markdown placeholders запрещены.

Локальный `State/page_registry.jsonl` — canonical machine-readable structure map. Concept README остаётся human entry map. `State/page_map.md` допустим только как derived artifact для большой сети или export; обязательные `manifest.jsonl`, `structure.md` и `state.json` не вводятся.

## Local issue isolation и escalation

Concept-internal issue и dependency rows остаются внутри `Concepts/<concept_slug>/Issues/`. Root issue registry не зеркалит их. При core-дефекте Execution Mode сохраняет локальный escalation packet, ставит `service_escalation_required`, прекращает затронутую root mutation и запрашивает переход в `Service Mode`. Root service issue и двусторонние cross-scope refs создаются только после такого перехода.

## Next-step marker

Execution next status: `ready_for_concept_selection_or_creation`.

Следующий шаг в `Execution Mode`: выбрать существующую концепцию либо получить slug, title, reason, initial scope и boundary для новой. Stage 05A остаётся branch-scoped и требует manual reviewer decision; merge, main readback и Stage 05B не выполнены.
