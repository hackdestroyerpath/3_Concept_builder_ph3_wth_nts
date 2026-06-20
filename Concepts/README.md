# Слой концепций

Parent: [Concept Builder README](../README.md)  
Owner issue: `EXEC-001` … `EXEC-007`  
Источник истины: `Concepts/README.md`  
Status: `available-empty-layer`  
Updated: `2026-06-20T19:24:43Z`

## Назначение

`Concepts/` хранит реальные концепции, развиваемые в `Concept Builder / Execution Mode`. Этот файл является human entry map слоя, но не является concept state и не создаёт runtime scope без утверждённого intent.

## Текущий статус

| Поле | Значение |
|---|---|
| Concepts created | `0` |
| Active concept | `none` |
| Startup case | `no_active_concept` |
| Bootstrap contract revision | `execution-bootstrap-v1` |
| Layer status | `available-empty-layer` |
| Template | [Concepts/_template/README.md](_template/README.md) |
| Execution protocols | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| Root index | [State/execution_index.md](../State/execution_index.md) |

Конкретные концепции не созданы. `Concepts/<concept_slug>/` появляется только после user intent или approved issue и полного creation gate.

## Реестр концепций

| Slug | Title | Lifecycle | Readiness mirror | Integrity mirror | Active issue | State path | Local page registry | Local issue registry | Export status | Last validation ref | Blocking status |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `none` | Концепции ещё не созданы | `not_applicable` | `ready_for_concept_selection_or_creation` | `verified` | `none` | `none` | `none` | `none` | `none` | `none` | `none` |

Root source of truth для active concept и registry contract: root execution index. Для concrete concept authoritative readiness/integrity хранит `State/concept_state.md`; таблица выше является derived mirror и обновляется с local state одной transaction.

## Creation gate

Перед созданием новой концепции агент обязан:

1. открыть [execution protocols](../Protocols/execution_protocols/README.md);
2. проверить root index и [State/page_registry.jsonl](../State/page_registry.jsonl);
3. использовать [шаблон](_template/README.md) только как specification;
4. получить `concept_slug`, title, reason, initial scope и boundary;
5. подтвердить отсутствие path conflict;
6. перечислить exact write set;
7. выполнить bootstrap persistence/readback transition.

## Минимальный bootstrap contract

Bootstrap готовит пять operational files:

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

`Pages/`, `Output/` и `Exports/` — logical areas. Они появляются с первым реальным artifact, а не как empty directories. Retention entrypoints создаются только при retention-потребности или отдельном approved bootstrap. Empty Markdown placeholders запрещены.

Initial local issue registry может быть пустым. Initial dependency graph может содержать одну metadata row с нулём edges. Concept issue не создаётся ради заполнения структуры.

## Structure map

`Concepts/<concept_slug>/State/page_registry.jsonl` — canonical machine-readable structure map и перечисляет только реально существующие files. Concept README содержит human entry links. Optional `State/page_map.md` допустим как derived artifact для большой сети или export.

Mandatory `manifest.jsonl`, `structure.md` и `state.json` не входят в bootstrap contract.

## Readiness transition

`State/concept_state.md` является authoritative source.

До persistence/readback:

```text
Lifecycle status: draft
Readiness status: bootstrap_incomplete
Integrity status: unverified
Last persisted at: null
Next status: needs_bootstrap_persistence
```

Разрешено только complete/recover bootstrap.

После существования всех пяти files, JSONL parse, identity check, successful persistence и readback:

```text
Lifecycle status: draft
Readiness status: ready_for_issue_or_page
Integrity status: verified
Integrity basis: exact five files + state revision + persistence/readback ref
Last persisted at: <factual timestamp>
Next status: needs_first_issue_or_page
```

`unverified`, `stale` или `conflict` блокируют issue/page mutation, кроме bounded recovery. Default concept README не хранит dynamic readiness metadata; если human mirror добавляется позже, он явно помечается derived и обновляется с state одной transaction.

## State, issues и dependencies

Local issue ID имеет формат `<concept_slug>-ISS-0001`. Concept-internal issue rows и dependency edges остаются local; root registry не зеркалит их.

## Service escalation

Canonical anchor:

```text
Concepts/<concept_slug>/State/concept_state.md#pending-service-escalation
```

Local state хранит `service_escalation_status`, `service_escalation_ref`, `service_issue_id`, `created_at` и `updated_at`. Anchor содержит source concept/issue, defect summary, affected root paths, evidence, safe workaround, requested service action и return anchor.

При local issue его registry/state хранит тот же `service_escalation_ref` и `return_anchor`. Execution Mode записывает только concept-local records и прекращает затронутую root mutation. Service Mode создаёт root issue; затем root и concept scopes получают bidirectional refs одной transaction. Resolution/cancellation обновляет тот же anchor.

## Scope boundaries

| Слой | Что хранит |
|---|---|
| Root `README.md` | вход в систему и route к этому слою |
| Root execution index | active concept, registry contract и startup case |
| `Concepts/<concept_slug>/README.md` | human entry конкретной концепции |
| `Concepts/<concept_slug>/State/` | authoritative local state и canonical local page registry |
| `Concepts/<concept_slug>/Issues/` | local issue registry, dependency graph и реальные issue artifacts |
| `Concepts/<concept_slug>/Pages/` | реальные содержательные pages после их появления |
| `Concepts/<concept_slug>/Output/` | реальные concept-level results |
| `Concepts/<concept_slug>/Exports/` | реальные export packages |

## Lifecycle и readiness

Lifecycle statuses: `draft`, `active`, `ready_for_closure_review`, `closed`, `exported`, `archived`.

Readiness statuses: `bootstrap_incomplete`, `ready_for_issue_or_page`, `active`, `ready_for_closure_review`, `validated_for_export`, `blocked`.

Lifecycle описывает положение concept, readiness — допустимость следующего действия, integrity — доверие к state/registry evidence.

## Export

Экспорт выполняется только через [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md). При open issue допустим только `work_in_progress`; closed export требует validation, verified integrity и user approval.

## Связанные файлы

- [Root README](../README.md)
- root execution index
- [Page registry](../State/page_registry.jsonl)
- [Execution protocols](../Protocols/execution_protocols/README.md)
- [Concept export protocol](../Protocols/execution_protocols/concept_export_protocol.md)
- [Concept template](_template/README.md)
