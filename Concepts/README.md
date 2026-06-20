# Слой концепций

Parent: [Concept Builder README](../README.md)  
Owner issue: `EXEC-001` … `EXEC-007`  
Источник истины: `Concepts/README.md`  
Status: `available-empty-layer`  
Updated: `2026-06-20T17:58:28Z`

## Назначение

`Concepts/` хранит реальные концепции, развиваемые в `Concept Builder / Execution Mode`. Этот файл является human entry map слоя, но не является концепцией сам по себе и не создаёт runtime scope без утверждённого intent.

## Текущий статус

| Поле | Значение |
|---|---|
| Concepts created | `0` |
| Active concept | `none` |
| Startup case | `no_active_concept` |
| Bootstrap contract revision | `stage_05a-r1` |
| Stage 05A status | `stage_05a_branch_result_pending_manual_reviewer` |
| Template | [Concepts/_template/README.md](_template/README.md) |
| Execution protocols | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| Root index | [State/execution_index.md](../State/execution_index.md) |

Конкретные концепции не созданы. `Concepts/<concept_slug>/` появляется только после явного пользовательского запроса или approved issue и полного creation gate.

## Реестр концепций

| Slug | Title | Lifecycle | Readiness | Active issue | State path | Local page registry | Local issue registry | Export status | Last validation ref | Blocking status |
|---|---|---|---|---|---|---|---|---|---|---|
| `none` | Концепции ещё не созданы | `not_applicable` | `ready_for_concept_selection_or_creation` | `none` | `none` | `none` | `none` | `none` | `none` | `none` |

Root source of truth для active concept и полного registry contract: [State/execution_index.md](../State/execution_index.md).

## Creation gate

Перед созданием новой концепции агент обязан:

1. открыть [execution protocols](../Protocols/execution_protocols/README.md);
2. проверить root index и [State/page_registry.jsonl](../State/page_registry.jsonl);
3. использовать [шаблон](_template/README.md) только как specification;
4. получить `concept_slug`, title, reason, initial scope и boundary;
5. подтвердить отсутствие path conflict;
6. перечислить exact write set;
7. сохранить operational files и registry updates одной transaction.

## Минимальный committed bootstrap

Первая transaction создаёт только пять реально используемых files:

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

`Pages/`, `Output/` и `Exports/` — логические области. Они появляются с первым реальным artifact, а не как заранее созданные пустые directories. `Issues/_archive/README.md` и `Issues/_tombstones/README.md` создаются только при фактической retention-потребности или отдельном approved bootstrap. Пустые Markdown placeholders запрещены.

Initial local issue registry может быть пустым. Initial dependency graph может содержать одну metadata row с нулём edges. Concept issue не создаётся для заполнения структуры.

## Structure map

`Concepts/<concept_slug>/State/page_registry.jsonl` — canonical machine-readable structure map и перечисляет только реально существующие files. Concept README содержит human entry tables/links. Optional `State/page_map.md` допустим как derived artifact для большой сети или export.

Обязательные `manifest.jsonl`, `structure.md` и `state.json` не входят в bootstrap contract.

## State, issues и dependencies

`State/concept_state.md` разделяет lifecycle, readiness и integrity. Local issue ID имеет формат `<concept_slug>-ISS-0001`. Concept-internal issue rows и dependency edges остаются локальными; root registry не зеркалит их.

При core-дефекте Execution Mode сохраняет local service-escalation packet, устанавливает `service_escalation_required`, останавливает затронутую root mutation и просит переход в Service Mode. Root issue и bidirectional refs создаются после переключения режима.

## Scope boundaries

| Слой | Что хранит |
|---|---|
| Root `README.md` | вход в систему и route к этому слою |
| Root execution index | active concept, registry contract и startup case |
| `Concepts/<concept_slug>/README.md` | human entry конкретной концепции |
| `Concepts/<concept_slug>/State/` | local state и canonical local page registry |
| `Concepts/<concept_slug>/Issues/` | local issue registry, dependency graph и реальные issue artifacts |
| `Concepts/<concept_slug>/Pages/` | реальные содержательные pages после их появления |
| `Concepts/<concept_slug>/Output/` | реальные concept-level results |
| `Concepts/<concept_slug>/Exports/` | реальные export packages |

## Lifecycle и readiness

Lifecycle statuses: `draft`, `active`, `ready_for_closure_review`, `closed`, `exported`, `archived`.

Readiness statuses: `bootstrap_incomplete`, `ready_for_issue_or_page`, `active`, `ready_for_closure_review`, `validated_for_export`, `blocked`.

Одинаково названный lifecycle/readiness status не делает поля взаимозаменяемыми: lifecycle описывает положение концепции, readiness — допустимость следующего действия.

## Export

Экспорт выполняется только через [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md). Stage 05A не меняет export protocol, naming или blocker matrix. При open issue допустим только `work_in_progress`; closed export требует validation и user approval.

## Связанные файлы

- [Root README](../README.md)
- [Execution index](../State/execution_index.md)
- [Page registry](../State/page_registry.jsonl)
- [Execution protocols](../Protocols/execution_protocols/README.md)
- [Concept export protocol](../Protocols/execution_protocols/concept_export_protocol.md)
- [Concept template](_template/README.md)
