# Execution-протоколы

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-001` … `EXEC-007`  
Protocol ID: `execution/index`  
Источник истины: `Protocols/execution_protocols/README.md`  
Status: `available`  
Updated: `2026-06-20T17:58:28Z`

## Назначение

Этот файл является входом в `Concept Builder / Execution Mode`. Режим маршрутизирует работу к конкретной концепции внутри `Concepts/<concept_slug>/`, но безопасно обрабатывает и состояние, когда концепция ещё не выбрана. Он использует общий issue lifecycle, persistence и validation, сохраняя concept-internal state и issue records локальными.

## Граница режима

| Разрешено в `Execution Mode` | Запрещено без escalation в `Service Mode` |
|---|---|
| создавать и менять файлы внутри `Concepts/<concept_slug>/` | ремонтировать root `Instructions/` или root `Protocols/` |
| вести локальные concept issue и dependency edges | менять root [service_state.md](../../State/service_state.md) как побочный эффект concept work |
| обновлять root [execution_index.md](../../State/execution_index.md) для выбора, создания, статуса или архивации concept entry | менять service-level issue registry без созданного service issue |
| обновлять root [page_registry.jsonl](../../State/page_registry.jsonl) для фактического concept entrypoint | продолжать затронутую root mutation после `service_escalation_required` |
| создавать export package через [concept_export_protocol.md](concept_export_protocol.md) | закрывать концепцию без closure review и validation |

Production root остаётся `/`. Перед любой записью применяется [persistence_protocol.md](../common/persistence_protocol.md).

## Старт `Execution Mode`

Общая загрузка:

1. открыть root [README](../../README.md);
2. открыть [State/execution_index.md](../../State/execution_index.md) и определить startup case;
3. открыть root [page_registry.jsonl](../../State/page_registry.jsonl), [Concepts/README.md](../../Concepts/README.md) и [catalog.md](../catalog.md);
4. выбрать один из трёх cases ниже;
5. до mutation проверить, что identity, state и write scope согласованы.

### `no_active_concept`

Условия: root index показывает `active concept = none` и не существует согласованного active concept state.

Безопасные действия:

- перечислить существующие концепции;
- выбрать существующую концепцию;
- подготовить создание новой концепции.

Папка не создаётся автоматически. Для create нужны user intent или approved issue, `concept_slug`, title, reason, initial scope и boundary. До полного gate ответ фиксирует `no_active_concept`, доступные варианты и next-step marker.

### `active_known`

Условия: slug, root registry entry, state path и локальный `Concept slug` согласованы.

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

Затем восстановить active issue, active phase, blockers, direct dependencies и следующий protocol. Root context не расширяется без конкретного reason.

### `active_unknown`

Условия: root index, root page registry или локальный state дают неполную либо противоречивую identity.

Recovery:

1. прочитать candidate slug, state path и registry paths из root index;
2. проверить существование concept entrypoint по root page registry;
3. сопоставить slug с локальным `State/concept_state.md`;
4. сопоставить local page registry и issue registry paths;
5. не выполнять mutation, пока identity и local state не совпадут.

Если evidence конфликтует, вернуть bounded blocker `active_concept_identity_conflict` с candidate values, checked paths и точным recovery action. Не выбирать концепцию по догадке.

## Создание новой концепции

Новая концепция создаётся только при наличии всех gates:

| Gate | Требование |
|---|---|
| User intent | пользователь явно просит создать концепцию или утверждает issue |
| Slug | ASCII-safe `concept_slug`, не конфликтующий с существующими paths |
| Title | человекочитаемое название |
| Reason | зачем концепция нужна |
| Initial scope | что входит в первую версию |
| Boundary | что не входит |
| Write set | перечислены exact operational files |
| Registry update | root index, root page registry и local page registry входят в одну transaction |

## Минимальный committed bootstrap

Первичная transaction создаёт ровно operational sources, которые сразу имеют функцию:

```text
Concepts/<concept_slug>/README.md
Concepts/<concept_slug>/State/concept_state.md
Concepts/<concept_slug>/State/page_registry.jsonl
Concepts/<concept_slug>/Issues/issue_registry.jsonl
Concepts/<concept_slug>/Issues/dependency_graph.jsonl
```

Правила:

- `Pages/`, `Output/` и `Exports/` — логические области, не обязательные пустые директории;
- они появляются с первым реальным artifact;
- `Issues/_archive/README.md` и `Issues/_tombstones/README.md` появляются только при retention-потребности или отдельном approved bootstrap;
- пустые Markdown placeholders не создаются;
- initial local page registry перечисляет только фактически созданные files;
- local issue registry может быть пустым, concept issue не изобретается;
- dependency graph может содержать одну metadata row с `edge_count = 0`.

## Structure-map contract

`Concepts/<concept_slug>/State/page_registry.jsonl` является canonical machine-readable structure map. Concept README — human entry map. Optional `State/page_map.md` создаётся только для действительно большой сети или export и остаётся derived artifact.

Stage 05A не вводит обязательные `manifest.jsonl`, `structure.md` или `state.json` и не мигрирует concept state в JSON.

## Concept state readiness contract

Локальный `State/concept_state.md` хранит независимо:

- `Concept slug`;
- `Lifecycle status`;
- `State revision`;
- `Active issue` и `Active phase`;
- `Blocking status`;
- `Readiness status`;
- `Page registry status`;
- `Issue registry status`;
- `Dependency readiness`;
- `Export status`;
- `Integrity status` и `Integrity basis`;
- `Last validation ref`;
- `Last persisted at`;
- `Next-step marker`.

`readiness_status`:

| Status | Семантика |
|---|---|
| `bootstrap_incomplete` | один из пяти operational files отсутствует либо registry не разбирается |
| `ready_for_issue_or_page` | bootstrap complete, registries parse, blocking status отсутствует |
| `active` | есть active issue или содержательная page work |
| `ready_for_closure_review` | required work завершён и готов к concept-level validation |
| `validated_for_export` | concept-level validation ref зафиксирован; export mechanics остаются отдельным protocol scope |
| `blocked` | следующий шаг остановлен конкретным blocker |

`integrity_status`:

| Status | Семантика |
|---|---|
| `unverified` | basis ещё не проверен |
| `verified` | перечисленный `integrity_basis` проверен на указанной revision/ref |
| `stale` | один из basis files изменился после validation |
| `conflict` | state, registry или identity противоречат друг другу |

Хеш не обязателен. Если он используется, basis и способ вычисления должны быть документированы; выдуманный или self-referential hash запрещён. Lifecycle и readiness не смешиваются.

## Local issue mapping

Service-level lifecycle применяется с concept-local path mapping:

| Service path | Execution path |
|---|---|
| `State/service_state.md` | `Concepts/<concept_slug>/State/concept_state.md` |
| `Issues/issue_registry.jsonl` | `Concepts/<concept_slug>/Issues/issue_registry.jsonl` |
| `Issues/dependency_graph.jsonl` | `Concepts/<concept_slug>/Issues/dependency_graph.jsonl` |
| `Issues/<issue_id>/` | `Concepts/<concept_slug>/Issues/<issue_id>/` |
| `State/page_registry.jsonl` | `Concepts/<concept_slug>/State/page_registry.jsonl` для concept-internal files |

Local ID имеет формат `<concept_slug>-ISS-0001`. Root registry не зеркалит concept-internal rows. Cross-scope service issue появляется только для фактического core defect; после его создания local issue/state получают обратную ссылку, а service issue — return anchor к concept context.

## Service escalation packet

При core-дефекте Execution Mode сохраняет локальный packet:

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

Execution Mode:

1. записывает packet в локальный concept scope;
2. устанавливает `Blocking status = service_escalation_required`;
3. прекращает затронутую root mutation;
4. предлагает перейти в `Service Mode`.

Service Mode затем создаёт root service issue и записывает bidirectional cross-scope refs. Execution Mode не выполняет root repair вместо этого перехода.

## Concept lifecycle

| Lifecycle status | Значение |
|---|---|
| `draft` | bootstrap создан, сеть ещё формируется |
| `active` | есть issue или pages in progress |
| `ready_for_closure_review` | required work завершён, нужна validation |
| `closed` | closure утверждён пользователем |
| `exported` | export package создан |
| `archived` | концепция выведена из active work, история сохранена |

Closure требует проверки local pages, issue registry, dependencies, output coverage, integrity и user approval. Открытые issue допускают только `work_in_progress` export по отдельному export protocol.

## Связанные источники истины

| Ресурс | Роль |
|---|---|
| [Concepts/README.md](../../Concepts/README.md) | вход в слой концепций |
| [Concepts/_template/README.md](../../Concepts/_template/README.md) | спецификация bootstrap/state contract |
| [concept_export_protocol.md](concept_export_protocol.md) | export и closure package |
| [execution_index.md](../../State/execution_index.md) | active concept и root routing |
| [service_state.md](../../State/service_state.md) | service-level accepted base и escalation destination |
| [page_registry.jsonl](../../State/page_registry.jsonl) | root concept entrypoint registry |
| [persistence_protocol.md](../common/persistence_protocol.md) | transaction-like запись |

## Completion signal

Протокол выполнен, если:

- startup case явно равен `no_active_concept`, `active_known` или `active_unknown`;
- для `active_known` восстановлены focus packet, active issue/phase и next protocol;
- для create известен полный gate и exact write set;
- для recovery или escalation указан bounded blocker/packet и next action;
- root index/page-registry update plan известен;
- запрещённая root mutation не начата;
- next-step marker можно восстановить без chat memory.
