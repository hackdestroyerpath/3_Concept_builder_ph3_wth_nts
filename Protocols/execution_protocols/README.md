# Execution-протоколы

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-011`  
Protocol ID: `execution/index`  
Источник истины: `Protocols/execution_protocols/README.md`  
Status: `available`  
Updated: `2026-06-20T20:55:35Z`

## Назначение

Этот файл является входом в `Concept Builder / Execution Mode`. Режим маршрутизирует работу к конкретной концепции внутри `Concepts/<concept_slug>/`, безопасно обрабатывает отсутствие active concept и сохраняет concept-internal state, issue и dependency records локальными.

## Граница режима

| Разрешено в `Execution Mode` | Запрещено без перехода в `Service Mode` |
|---|---|
| создавать и менять files внутри `Concepts/<concept_slug>/` | ремонтировать root `Instructions/` или root `Protocols/` |
| вести local concept issue и dependency edges | менять root [service_state.md](../../State/service_state.md) как побочный эффект concept work |
| обновлять root [execution_index.md](../../State/execution_index.md) для выбора, создания, статуса или archive concept entry | создавать или редактировать root service issue из Execution Mode |
| обновлять root [page_registry.jsonl](../../State/page_registry.jsonl) для фактического concept entrypoint | продолжать затронутую root mutation после `service_escalation_required` |
| создавать export package через [concept_export_protocol.md](concept_export_protocol.md) | закрывать концепцию без closure review и validation |

Production root остаётся `/`. Перед любой записью применяется [persistence_protocol.md](../common/persistence_protocol.md).

## Старт `Execution Mode`

Общая загрузка:

1. открыть root [README](../../README.md);
2. открыть [State/execution_index.md](../../State/execution_index.md) и определить startup case;
3. открыть root page registry, [Concepts/README.md](../../Concepts/README.md) и [catalog.md](../catalog.md);
4. выбрать один из трёх cases ниже;
5. до mutation проверить, что identity, state, readiness, integrity и write scope согласованы.

### `no_active_concept`

Условия: root index показывает `active concept = none` и не существует согласованного active concept state.

Безопасные действия:

- перечислить существующие концепции;
- выбрать существующую концепцию;
- подготовить создание новой концепции.

Папка не создаётся автоматически. Для create нужны user intent или approved issue, `concept_slug`, title, reason, initial scope и boundary.

### `active_known`

Условия: slug, root registry entry, state path и local `Concept slug` согласованы.

Загрузить минимальный focus packet:

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
optional active issue artifacts
selected local protocol
```

Issue/page mutation разрешена только при `Readiness status = ready_for_issue_or_page|active|ready_for_closure_review|validated_for_export` и `Integrity status = verified`. Затем восстановить active issue, active phase, blockers, direct dependencies и следующий protocol.

### `active_unknown`

Условия: root index, root page registry или local state дают неполную либо противоречивую identity.

Recovery:

1. прочитать candidate slug, state path и registry paths из root index;
2. проверить concept entrypoint по root page registry;
3. сопоставить slug с local `State/concept_state.md`;
4. сопоставить local page registry и issue registry paths;
5. не выполнять mutation, пока identity и local state не совпадут.

Если evidence конфликтует, вернуть bounded blocker `active_concept_identity_conflict` с candidate values, checked paths и точным recovery action.

## Создание новой концепции

Новая концепция создаётся только при наличии всех gates:

| Gate | Требование |
|---|---|
| User intent | пользователь явно просит создать концепцию или утверждает issue |
| Slug | ASCII-safe `concept_slug`, не конфликтующий с existing paths |
| Title | человекочитаемое название |
| Reason | зачем концепция нужна |
| Initial scope | что входит в первую версию |
| Boundary | что не входит |
| Write set | перечислены exact operational files |
| Registry update | root index, root page registry и local page registry входят в одну transaction |

## Минимальный bootstrap contract

Bootstrap готовит ровно operational sources:

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

Правила:

- `Pages/`, `Output/` и `Exports/` — logical areas, не обязательные empty directories;
- они появляются с первым реальным artifact;
- retention entrypoints появляются только при retention-потребности или отдельном approved bootstrap;
- empty Markdown placeholders не создаются;
- initial local page registry перечисляет только фактически созданные files;
- local issue registry может быть пустым;
- dependency graph может содержать одну metadata row с `edge_count = 0`.

## Bootstrap persistence и readiness transition

`State/concept_state.md` является authoritative source.

### До persistence/readback

Prewrite state:

```text
Lifecycle status: draft
Readiness status: bootstrap_incomplete
Integrity status: unverified
Last persisted at: null
Next status: needs_bootstrap_persistence
```

Разрешено только complete/recover bootstrap. Issue/page mutation запрещена.

### Проверка и переход

1. Записать пять operational files и required root routing companions.
2. Прочитать все пять paths после последней mutation.
3. Проверить JSONL line-by-line, identity и state revision.
4. Если проверка не прошла, оставить `bootstrap_incomplete` и `unverified|conflict` и выполнить bounded recovery.
5. Если проверка прошла, обновить `State/concept_state.md` до verified state, записать factual timestamp и exact persistence/readback ref.
6. Прочитать state повторно. Только после этого разрешить first issue/page work.

Verified state:

```text
Lifecycle status: draft
Readiness status: ready_for_issue_or_page
Integrity status: verified
Integrity basis: exact five files + state revision + persistence/readback ref
Last persisted at: <factual timestamp>
Next status: needs_first_issue_or_page
```

`ready_for_issue_or_page` требует `Integrity status = verified`. `unverified`, `stale` или `conflict` блокируют issue/page mutation, кроме bounded recovery. Если concept README показывает readiness, значение маркируется как derived mirror local state и обновляется с state одной transaction; default template динамическое readiness-поле в README не создаёт.

## Structure-map contract

`Concepts/<concept_slug>/State/page_registry.jsonl` является canonical machine-readable structure map. Concept README — human entry map. Optional `State/page_map.md` создаётся только для большой сети или export и остаётся derived artifact.

Bootstrap contract не требует mandatory `manifest.jsonl`, `structure.md` или `state.json` и не мигрирует concept state в JSON.

## Concept state contract

Локальный `State/concept_state.md` хранит независимо:

- `Concept slug`;
- `Lifecycle status`;
- `State revision`;
- `Active issue` и `Active phase`;
- `Blocking status`;
- `Readiness status`;
- registry/dependency/export status;
- `Integrity status`, `Integrity basis`, `Last validation ref`, `Last persisted at`;
- `Service escalation status`, `Service escalation ref`, `Service issue ID`;
- `Next status`.

Hash не обязателен. Если он используется, basis и способ вычисления документируются; выдуманный или self-referential hash запрещён.

## Local issue mapping

Service-level lifecycle применяется с concept-local path mapping:

| Service path | Execution path |
|---|---|
| `State/service_state.md` | `Concepts/<concept_slug>/State/concept_state.md` |
| `Issues/issue_registry.jsonl` | `Concepts/<concept_slug>/Issues/issue_registry.jsonl` |
| `Issues/dependency_graph.jsonl` | `Concepts/<concept_slug>/Issues/dependency_graph.jsonl` |
| `Issues/<issue_id>/` | `Concepts/<concept_slug>/Issues/<issue_id>/` |
| `State/page_registry.jsonl` | `Concepts/<concept_slug>/State/page_registry.jsonl` для concept-internal files |

Local ID имеет формат `<concept_slug>-ISS-0001`. Root registry не зеркалит concept-internal rows.

## Service escalation contract

Canonical state anchor:

```text
Concepts/<concept_slug>/State/concept_state.md#pending-service-escalation
```

State fields:

```text
Service escalation status: none | pending_service_mode | service_issue_created | resolved | cancelled
Service escalation ref: State/concept_state.md#pending-service-escalation | none
Service issue ID: <root service issue id> | none
```

Anchor payload:

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

Rules:

1. `source_concept_issue` может быть `none` только если defect найден вне active issue work.
2. При local issue его registry/state хранит тот же `service_escalation_ref` и `return_anchor`.
3. Execution Mode пишет только concept-local state/issue records, устанавливает `service_escalation_required` и прекращает затронутую root mutation.
4. Service Mode создаёт root service issue.
5. После создания root и concept scopes получают bidirectional refs одной controlled transaction.
6. Resolution/cancellation обновляет тот же anchor и его status/timestamps; ad-hoc escalation file не создаётся.

## Concept lifecycle

| Lifecycle status | Значение |
|---|---|
| `draft` | bootstrap создаётся или concept network формируется |
| `active` | есть issue или pages in progress |
| `ready_for_closure_review` | required work завершён, нужна validation |
| `closed` | closure утверждён пользователем |
| `exported` | export package создан |
| `archived` | концепция выведена из active work, история сохранена |

Closure требует проверки local pages, issue registry, dependencies, output coverage, integrity и user approval. Open issue допускают только `work_in_progress` export по отдельному export protocol.

## Связанные источники истины

| Ресурс | Роль |
|---|---|
| [Concepts/README.md](../../Concepts/README.md) | вход в слой концепций |
| [Concepts/_template/README.md](../../Concepts/_template/README.md) | specification bootstrap/state contract |
| [concept_export_protocol.md](concept_export_protocol.md) | export и closure package |
| [execution_index.md](../../State/execution_index.md) | active concept и root routing |
| [service_state.md](../../State/service_state.md) | service-level accepted base и escalation destination |
| root page registry | concept entrypoint registry |
| [persistence_protocol.md](../common/persistence_protocol.md) | transaction-like запись |

## Completion signal

Протокол выполнен, если:

- startup case явно равен `no_active_concept`, `active_known` или `active_unknown`;
- для `active_known` восстановлены focus packet, readiness/integrity, active issue/phase и next protocol;
- для create известен полный gate, exact write set и prewrite state;
- verified readiness установлена только после persistence/readback;
- для recovery или escalation указан bounded blocker/canonical anchor и next action;
- root index/page-registry update plan известен;
- запрещённая root mutation не начата;
- next-step marker восстанавливается без chat memory.
