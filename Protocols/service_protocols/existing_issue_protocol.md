# Протокол работы с существующим issue

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-007`  
Protocol ID: `service/existing_issue`  
Источник истины: `Protocols/service_protocols/existing_issue_protocol.md`  
Status: `available`  
Updated: `2026-06-18T12:41:00Z`

## Назначение

Этот протокол выбирает ровно один уже существующий service-level `issue`, проверяет registry, state, dependency graph и фокус, затем собирает минимальный focus packet для следующего протокола. Он не создаёт новый `issue`, не пишет requirements, не выполняет solution и не открывает параллельную работу, если существующий active/focused issue уже покрывает запрос пользователя.

Главная защита Stage 04: продолжение существующего issue всегда предпочтительнее создания дубликата. Если registry, `service_state` или focus packet уже указывают на active/approved/blocked issue в том же scope, агент сначала доказывает, что это не та же работа. Без такого доказательства новый issue не создаётся.

## Runtime service issue scaffold

Runtime-папка создаётся только когда lifecycle реально требует файлов phase. Пустые папки ради демонстрации scaffold запрещены.

Рекомендуемая форма runtime issue:

```text
Issues/<issue_id>/
├── state.md
├── reason.md
├── requirements.md
├── solution.md
├── contract.md
└── output/
    ├── report.md
    ├── changed_files.md
    ├── contract_coverage.md
    └── attachments_manifest.jsonl
```

QA artifacts допустимы только когда QA действительно запускалась или protocol явно требует QA trace. Bootstrap implementation issue могут оставаться registry-only: отсутствие папки не является ошибкой, если registry содержит достаточный reason, target paths и next action.

## Когда использовать

| Ситуация | Действие |
|---|---|
| Пользователь указал точный `issue_id` | найти одну registry row и проверить graph/state |
| Пользователь сказал `продолжай` | продолжить active issue из `service_state`, если scope совпадает |
| Пользователь выбрал пункт existing issue после service start | показать shortlist или сфокусировать выбранный issue |
| Registry/state/focus уже указывают на активную работу | продолжить её, не создавать дубликат |
| Новый input похож на active issue | показать duplicate-risk и запросить явное решение merge/split/link |

Новый материал не смешивается с текущим issue автоматически. Он становится linked input только после решения пользователя или отдельного protocol routing.

## Обязательные входы

| Файл / вход | Зачем читать |
|---|---|
| [../../State/service_state.md](../../State/service_state.md) | active scope, active issue, phase, pending user action, return anchor |
| [../../Issues/issue_registry.md](../../Issues/issue_registry.md) | lifecycle summary и человекочитаемый snapshot |
| [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) | точная registry row выбранного issue |
| [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) | blockers, stale dependencies, cycle status |
| [../../State/page_registry.jsonl](../../State/page_registry.jsonl) | path existence, parent links, orphan risk |
| [../catalog.md](../catalog.md) | selected next protocol по status/phase |
| [../common/context_loading_protocol.md](../common/context_loading_protocol.md) | граница focus packet |
| [../common/persistence_protocol.md](../common/persistence_protocol.md) | transaction guard для state/registry updates |

Runtime-файлы `Issues/<issue_id>/state.md`, `reason.md`, `requirements.md`, `solution.md`, `contract.md` и `output/*` читаются только если registry указывает на них и файл существует.

## Preconditions

1. `issue_registry.jsonl` и `dependency_graph.jsonl` парсятся без ошибок.
2. Выбранный `issue_id` существует в registry, либо агент возвращает shortlist вместо выдуманного выбора.
3. Registry row содержит `status`, `phase`, `scope_path`, `target_paths`, `dependency_ready`, `next_action` и reason/source fields достаточные для continuation.
4. Для runtime continuation есть одно из двух оснований:
   - active issue state существует и совпадает с registry;
   - registry-only bootstrap continuation разрешён, потому что registry содержит explicit bootstrap reason и runtime files ещё не требуются.
5. Dependency graph проверен: blocking edges не находятся в `blocked`, `stale`, `cycle_blocked` или `unsatisfied`, если выбранная phase требует execution/validation/closure.
6. Если registry, state и focus packet конфликтуют, агент останавливается с blocker `focus_evidence_conflict` и repair write set.

## Duplicate-prevention gate

Перед созданием нового issue или сменой focus агент обязан выполнить gate:

| Проверка | Pass condition |
|---|---|
| Active state | `service_state.active_issue_id` отсутствует или совпадает с выбранным issue |
| Registry overlap | нет issue со status `creating`, `proposed`, `needs_discussion`, `approved`, `active`, `blocked` или `deferred` с тем же scope и target paths |
| Focus packet | текущий focus не содержит pending user action по тому же issue |
| Dependency graph | нет edge, который показывает, что текущая работа уже depends/informs новый запрос |
| Return anchor | известно, куда вернуться после следующего протокола |

Fail этого gate не означает, что работа невозможна. Он означает, что агент должен продолжить существующий issue, создать linked relation через [linked_issues_protocol.md](linked_issues_protocol.md), либо запросить user decision merge/split/defer.

## Порядок выполнения

### 1. Resolve issue reference

| Ситуация | Действие |
|---|---|
| Точный ID | найти одну registry row |
| Неполный ID/title | вернуть shortlist кандидатов |
| `продолжай` без ID | взять active issue из state, затем сверить registry |
| Несколько совпадений | вернуть shortlist, не выбирать автоматически |
| Active issue отличается от user request | показать conflict и варианты switch/link/defer |

Shortlist содержит только ID, title, status, phase, dependency readiness и next action. Полный контекст phase-файлов не грузится ради меню.

### 2. Проверить lifecycle status

| Status / phase | Действие | Следующий protocol ID |
|---|---|---|
| `creating` | остановиться на intake/repair | `service/new_issue` или repair blocker |
| `proposed` | показать reason summary и decision варианты | planned decision update |
| `needs_discussion` | сфокусировать discussion на reason/scope/duplicate-risk | planned decision update |
| `approved` без phase | проверить dependencies и выбрать phase | `service/existing_issue` routing |
| `active: qa` | подготовить QA continuation | `service/question_answer` |
| `active: requirements` | подготовить requirements packet | `service/requirements` |
| `active: requalification` | проверить simple/complex boundary | `service/complex_issue` |
| `active: solution` | подготовить solution/contract packet | `service/solution_contract_output` |
| `active: execution` | подготовить execution packet | `service/solution_contract_output` |
| `active: validation` | подготовить validation packet | `common/final_validation` |
| `closed` / `rejected` | read-only summary или retention routing | `service/issue_retention` |
| `archived` / `tombstone` / `deleted` | read-only summary; не активировать без нового linked issue | none / retention |
| `blocked` | показать blocker и условие снятия | source phase или dependency repair |
| `deferred` | не активировать без revive reason | revive decision |

Status/phase mismatch не чинится молча. Агент фиксирует inconsistency, показывает repair write set и не переходит к предметной работе.

### 3. Dependency readiness

Агент читает `dependency_refs` выбранного issue и direct graph edges. Проверка чистая, если нет active blocking edges со status `blocked`, `stale`, `cycle_blocked` или `unsatisfied`, нет stale edge на несуществующий issue и parent/child/linked refs не требуют context lift.

`ready` в registry не заменяет graph evidence. Если graph и registry расходятся, действует blocker `graph_registry_mismatch`.

### 4. Return anchor и next protocol

Focus packet обязан содержать:

| Поле | Значение |
|---|---|
| `issue_id` | стабильный ID |
| `title` | title из registry |
| `mode` | `Service Mode` |
| `scope_path` | `/` или иной registry scope |
| `status` / `phase` | reconstructed lifecycle state |
| `dependency_ready` | итог graph check |
| `reason_summary` | краткая сводка reason |
| `loaded_files` | только реально прочитанные файлы |
| `missing_expected_files` | отсутствующие expected paths с reason |
| `return_anchor` | state marker, registry row или user command, куда возвращаться после next protocol |
| `next_protocol` | available/planned/blocked protocol ID |
| `next_action` | одно ближайшее действие |

Если `next_protocol` имеет status `planned`, агент не исполняет его как available и возвращает blocker `next_protocol_planned`.

### 5. Запись focus/state

Запись требуется, если агент переводит issue в `active`, меняет phase, меняет active issue в state, фиксирует blocker/inconsistency или обновляет dependency readiness.

Write set обычно включает `State/service_state.md`, registry JSONL/Markdown, dependency graph при edge changes и `State/persistence_log.jsonl`. Если запись не подтверждена, агент возвращает `blocked_on_persistence` и не заявляет active transition.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Issue ID не найден | `issue_not_found` | вернуть shortlist или route к new issue |
| Найдено несколько candidates | `ambiguous_issue_reference` | показать shortlist |
| Active issue уже покрывает запрос | `duplicate_issue_prevented` | продолжить active issue или запросить merge/split/link decision |
| Registry parse error | `blocked_on_registry_parse` | repair write set |
| Graph/state/registry conflict | `focus_evidence_conflict` | blocker до repair |
| Dependency cycle | `blocked_on_dependency_cycle` | не переводить в active/execution |
| Blocking dependency unsatisfied | `blocked_on_dependency` | показать edge IDs и unblock condition |
| Runtime file отсутствует | `missing_issue_artifact` | разрешить только registry-only bootstrap или repair blocker |
| Следующий phase protocol planned | `next_protocol_planned` | остановиться после focus packet |
| Persistence недоступен | `blocked_on_persistence` | не заявлять active transition |

## Completion signal

Протокол завершён, когда выбран ровно один existing issue, собран focus packet с return anchor, проверены blockers и выбран next protocol; либо пользователь получил shortlist; либо работа остановлена с честным blocker, write set и next required action.

## Связанные файлы

- [Catalog](../catalog.md)
- [Context loading protocol](../common/context_loading_protocol.md)
- [Persistence protocol](../common/persistence_protocol.md)
- [Service start protocol](service_start_protocol.md)
- [New issue protocol](new_issue_protocol.md)
- [Question Answer protocol](question_answer_protocol.md)
- [Requirements protocol](requirements_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Complex Issue protocol](complex_issue_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Issue Retention protocol](issue_retention_protocol.md)
- [Service state](../../State/service_state.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Page registry](../../State/page_registry.jsonl)
