# Протокол Complex Issue

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-010`  
Protocol ID: `service/complex_issue`  
Источник истины: `Protocols/service_protocols/complex_issue_protocol.md`  
Status: `available`  
Updated: `2026-06-05T09:52:44Z`

## Назначение

Этот протокол управляет ситуацией, когда выбранный `issue` нельзя качественно выполнить как один `simple issue`: требуется разложение на дочерние обычные `issue`, отдельные work packages, разные outputs или управляемая последовательность зависимостей.

`Complex issue` не является способом спрятать неопределённость под красивым словом. Он применяется только если decomposition снижает риск, делает работу проверяемой или предотвращает преждевременное выполнение зависимых частей.

Этот протокол не заменяет [requirements_protocol.md](requirements_protocol.md), [solution_contract_output_protocol.md](solution_contract_output_protocol.md) и [linked_issues_protocol.md](linked_issues_protocol.md). Он определяет parent/children-модель, budget декомпозиции и правила возврата результатов child issue в parent.

## Когда использовать

| Состояние | Действие |
|---|---|
| `active: requalification` | проверить, остаётся ли issue `simple`, или требуется перевод в `complex` |
| Approved requirements требуют независимых work packages | подготовить decomposition plan в parent `solution.md` |
| Утверждён parent decomposition plan | создать child issue registry entries и dependency edges |
| Child issue закрыт | поднять результат в parent state и parent contract coverage |
| Parent готовится к validation | проверить закрытие required children и отсутствие dependency blockers |
| Decomposition budget исчерпан | заблокировать дальнейшее разложение и запросить решение пользователя |

Если нужна только зависимость между независимыми issue без parent/child-отношения, используется [linked_issues_protocol.md](linked_issues_protocol.md). Если речь об archive/tombstone cleanup, этот протокол не трогает retention lifecycle.

## Обязательные входы

| Вход | Источник |
|---|---|
| Выбранный parent issue | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) и, если есть, `Issues/<issue_id>/state.md` |
| Requirements | `Issues/<issue_id>/requirements.md`, подготовленный через [requirements_protocol.md](requirements_protocol.md) |
| Reason и input refs | `reason.md`, registry `reason_summary`, `input_ref` |
| Dependency graph | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Existing issue focus | [existing_issue_protocol.md](existing_issue_protocol.md) |
| Solution/contract rules | [solution_contract_output_protocol.md](solution_contract_output_protocol.md) |
| Linked dependency rules | [linked_issues_protocol.md](linked_issues_protocol.md) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Protocol catalog | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |

## Preconditions

1. Issue выбран и имеет `status = active`, либо approved issue переводится в focus через [existing_issue_protocol.md](existing_issue_protocol.md).
2. Requirements уже утверждены, либо пользователь явно просит только preliminary decomposition draft без создания child issue.
3. Причина перевода в `complex` зафиксирована: независимые outputs, разные owners, dependency sequencing, конфликт scope, слишком большой execution package или отдельные validation criteria.
4. Blocking dependencies parent issue проверены по [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
5. Пользователь утвердил смену `type` с `simple` на `complex`, если issue уже был approved как simple.
6. Persistence доступен. Если запись невозможна, child issue не считаются созданными.

## Граница simple / complex

| Признак | Решение |
|---|---|
| Один output, один contract, одно связное изменение | оставить `simple` |
| Несколько независимых outputs или acceptance groups | предложить `complex` |
| Требуются разные sequences: сначала A, потом B | `complex` или linked dependencies, выбрать минимальную модель |
| Часть scope можно выполнить независимо и проверить отдельно | `complex`, если это снижает риск |
| Новая тема не является частью parent goal | создать linked issue через [linked_issues_protocol.md](linked_issues_protocol.md), а не child |
| Decomposition добавляет только папки и статусы без пользы | оставить `simple` |

Тип issue не меняется молча. Если requalification меняет scope, пользователь должен увидеть reason, affected files и последствия для lifecycle.

## Decomposition budget

У каждого `complex issue` есть локальный `decomposition_budget`. Универсального числа уровней для всей системы нет. Budget фиксируется в parent `solution.md` и кратко отражается в registry.

Минимальные поля budget:

| Поле | Значение |
|---|---|
| `budget_reason` | почему decomposition нужна |
| `max_depth` | допустимая глубина children относительно parent |
| `max_child_count` | ожидаемый лимит child issue или reason, почему лимит открытый |
| `child_scope_rule` | какие части scope можно отдавать child issue |
| `dependency_rule` | какие child issue блокируют parent или друг друга |
| `context_budget` | какие parent/child summaries разрешено читать без repository-wide обхода |
| `stop_conditions` | когда decomposition прекращается |
| `escalation_rule` | что делать при исчерпании budget |

Если агент хочет создать новый уровень вложенности сверх budget, он не делает это втихую. Он переводит parent или текущий child в `blocked` / `needs_discussion`, сохраняет reason и предлагает: расширить budget, объединить children, вынести часть scope в non-goal, создать linked issue или вернуться к parent solution.

## Структура decomposition plan в parent `solution.md`

Отдельный `plan.md` не создаётся. Раздел decomposition живёт внутри parent `solution.md`.

```text
## Decomposition plan

Status: draft | review | approved | reopened
Parent issue: <issue_id>
Decomposition reason: <why complex>
Budget:
  max_depth: <number or local rule>
  max_child_count: <number or local rule>
  stop_conditions: <list>

| Child candidate | Purpose | Required output | Blocking parent? | Dependency refs | Acceptance link |
|---|---|---|---|---|---|
| CHILD-001 | ... | ... | yes/no | ... | REQ-... |

## Parent closure rule
- Required children: ...
- Optional / non-goal children: ...
- Parent validation evidence: ...
```

До approval этот раздел является proposal, а не разрешением создавать runtime issue. После approval agent создаёт child issue entries через обычную issue-модель.

## Создание child issue

Каждый child является обычным `issue`, а не отдельным типом `sub-issue`.

Минимальные действия:

1. Сгенерировать стабильный `issue_id` по policy root registry или concept-local registry.
2. Создать registry entry с `parent_id = <parent_issue_id>`, `depth_level = parent.depth_level + 1`, собственным `reason_summary`, `status = proposed` или `approved` по решению пользователя.
3. Обновить parent `children_ids`.
4. Если child блокирует parent closure или другой child, добавить edge в [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) по правилам [linked_issues_protocol.md](linked_issues_protocol.md).
5. Создать runtime files child issue только если lifecycle требует их: `state.md`, `reason.md`, requirements и далее. Пустые папки не создаются.
6. Обновить [../../State/page_registry.jsonl](../../State/page_registry.jsonl), если создан Markdown-файл.
7. Добавить запись в [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl).

Child issue наследует context summary parent, но не копирует весь parent text. Parent хранит только сводку child: purpose, status, dependency readiness, output pointer и next step.

## Dependency readiness для parent / children

| Ситуация | Правило |
|---|---|
| Child нужен для parent closure | parent не переходит в `validation`, пока child не `closed` или явно `deferred/rejected` с approved reason |
| Child A нужен для Child B | edge фиксируется в graph; Child B не идёт в execution до ready edge |
| Child output меняется после использования parent | parent получает `dependency_ready = stale` до перепроверки contract coverage |
| Edge создаёт цикл | edge не становится active blocking; parent/child получает `cycle_blocked` или `needs_discussion` |
| Child оказался самостоятельной темой | child переводится в linked issue или detached issue с reason, parent scope обновляется |

Перед выбором child агент читает только parent summary, child registry entry и прямые dependency edges. Полный контекст parent загружается только при context lift с reason.

## Возврат результата child в parent

После закрытия child issue агент обязан подняться к parent:

1. сохранить child output и validation pointer;
2. обновить parent `state.md`, если он существует;
3. обновить parent registry entry: closed child, remaining children, dependency readiness и parent contract coverage pointer;
4. свернуть детали child до краткой сводки;
5. выбрать следующий child, вернуться к parent solution или запросить user decision;
6. обновить next-step marker.

Parent не закрывается только потому, что последний child закрыт. Parent ещё должен пройти own validation и contract coverage. После закрытия child или parent archive/tombstone routing выполняется через [issue_retention_protocol.md](issue_retention_protocol.md).

## Parent closure gate

`Complex issue` можно закрыть только если:

- все required child issue имеют terminal state `closed`, либо пользователь явно перевёл часть scope в non-goal / deferred / rejected;
- parent contract coverage показывает, какие child outputs покрывают какие requirements;
- dependency graph для parent scope не содержит `blocked`, `stale` или `cycle_blocked`;
- parent validation report сохранён;
- registry, parent state и page registry согласованы.

Если часть child issue отклонена или отложена, это не исчезает из истории. Parent contract coverage должен показать, почему parent всё ещё можно закрыть или почему closure blocked.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Нет approved requirements | `blocked_on_requirements` | вернуться к [requirements_protocol.md](requirements_protocol.md) |
| User не утвердил смену типа | `needs_requalification_approval` | сохранить proposal и ждать решения |
| Budget исчерпан | `blocked_on_decomposition_budget` | запросить расширение budget или изменение scope |
| Child scope дублирует existing issue | `needs_dedup_decision` | использовать [linked_issues_protocol.md](linked_issues_protocol.md) или merge/defer |
| Dependency cycle | `cycle_blocked` | не сохранять active blocking edge, предложить варианты исправления |
| Нельзя записать registry/state | `blocked_on_persistence` | не считать child issue созданными |
| Parent closure без children coverage | `blocked_on_parent_contract_coverage` | вернуть parent к solution/execution/validation |

## Completion signal

Протокол завершён, когда parent issue сохранён как `complex` с approved decomposition plan, child issue entries и dependency edges созданы или работа честно остановлена с blocker. После создания children следующий routing выбирается через [existing_issue_protocol.md](existing_issue_protocol.md): первый child, parent solution review или dependency repair.

## Связанные файлы

- [Catalog](../catalog.md)
- [Catalog JSONL](../catalog.jsonl)
- [Existing issue protocol](existing_issue_protocol.md)
- [Requirements protocol](requirements_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Issue Retention protocol](issue_retention_protocol.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Service state](../../State/service_state.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
