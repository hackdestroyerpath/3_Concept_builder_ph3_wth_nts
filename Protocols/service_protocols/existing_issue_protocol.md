# Протокол работы с существующим issue

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-007`  
Protocol ID: `service/existing_issue`  
Источник истины: `Protocols/service_protocols/existing_issue_protocol.md`  
Status: `available`  
Updated: `2026-06-05T09:52:51Z`

## Назначение

Этот протокол выбирает один уже существующий `issue`, проверяет его статус, phase, blockers и собирает минимальный focus packet для дальнейшей работы. Он не создаёт новый `issue`, не пишет requirements и не выполняет solution.

Протокол работает в `Concept Builder Service Mode`. Для будущего `Execution Mode` та же логика будет перенесена в concept-local registry после появления слоя `Concepts/`.

## Когда использовать

Протокол применяется, если пользователь:

- выбрал `2` после [service_start_protocol.md](service_start_protocol.md);
- указал конкретный ID существующего issue, например `EXEC-008` или будущий `SVC-ISS-0001`;
- написал `взять issue`, `открыть issue`, `продолжить issue`, `сфокусироваться на issue`;
- принял предложенный агентом next issue;
- просит продолжить active issue из [service_state.md](../../State/service_state.md), когда active scope уже service-level.

Если пользователь передал новый материал, который не относится к выбранному issue, агент не смешивает его с текущим фокусом. Новый материал направляется в [new_issue_protocol.md](new_issue_protocol.md) или связывается как linked input только после явного решения.

## Обязательные входы

| Файл / вход | Зачем читать |
|---|---|
| [../../State/service_state.md](../../State/service_state.md) | active scope, active issue, phase и next-step marker |
| [../../Issues/issue_registry.md](../../Issues/issue_registry.md) | lifecycle, schema и человекочитаемый snapshot |
| [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) | точная registry entry выбранного issue |
| [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) | blockers, stale dependencies и cycle-status |
| [../../State/page_registry.jsonl](../../State/page_registry.jsonl) | проверка путей, parent links и orphan-risk |
| [../catalog.md](../catalog.md) | выбор следующего протокола по status/phase |
| [../common/context_loading_protocol.md](../common/context_loading_protocol.md) | ограничение focus packet |
| [../common/persistence_protocol.md](../common/persistence_protocol.md) | запись state/registry только через transaction-like guard |

Runtime-файлы `Issues/<issue_id>/state.md`, `reason.md`, `requirements.md`, `solution.md`, `contract.md`, `output/report.md` и `validation_report.md` читаются только если их пути указаны в registry и файл существует. Bootstrap implementation issue могут жить только в registry, без отдельной папки issue; это допустимо, если registry entry содержит sufficient reason и target paths.

## Preconditions

1. [issue_registry.jsonl](../../Issues/issue_registry.jsonl) парсится без ошибок.
2. [dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) парсится без ошибок.
3. Выбранный `issue_id` существует в registry или пользователь получил shortlist вместо выдуманного выбора.
4. Для перевода issue в `active` все blocking dependencies имеют `satisfied_for_draft`, `satisfied`, `not_applicable` или другой явно допустимый статус.
5. Если выбранный issue закрыт, archived, tombstone или deleted, протокол работает только в read-only режиме и не возобновляет выполнение без нового approved linked issue.
6. Если нужна запись, write status остаётся `package_draft_not_committed`, пока нет фактического GitHub commit marker в [persistence_log.jsonl](../../State/persistence_log.jsonl).

## Порядок выполнения

### 1. Resolve issue reference

Агент извлекает issue reference из команды пользователя:

| Ситуация | Действие |
|---|---|
| Указан точный `issue_id` | найти одну registry entry |
| Указан неполный ID или title | найти кандидаты и вернуть shortlist |
| Пользователь выбрал `2` без ID | показать shortlist approved/active/blocked issues с priority и blocker-status |
| В state есть active issue и пользователь сказал `продолжай` | проверить, что active issue существует и не противоречит registry |
| Найдено несколько совпадений | не выбирать за пользователя; вернуть варианты |

Shortlist должен содержать только ID, title, status, phase, dependency readiness и next action. Агент не обязан грузить все phase-файлы ради меню.

### 2. Проверить lifecycle status

| Status / phase | Действие | Следующий protocol ID |
|---|---|---|
| `creating` | показать, что issue ещё не оформлен; вернуться к intake/repair | `service/new_issue` или repair blocker |
| `proposed` | показать reason summary и варианты `утвердить / отклонить / обсудить` | `common/issue_decision_update` planned |
| `needs_discussion` | сфокусировать обсуждение на reason, scope и duplicate-risk | `common/issue_decision_update` planned |
| `approved` без phase | проверить dependencies и выбрать начальную phase | `service/existing_issue`, затем available/planned phase protocol |
| `active: qa` | подготовить продолжение QA | [service/question_answer](question_answer_protocol.md) available |
| `active: requirements` | подготовить requirements work packet | [service/requirements](requirements_protocol.md) available |
| `active: requalification` | проверить simple/complex boundary | [service/complex_issue](complex_issue_protocol.md) available |
| `active: solution` | подготовить solution/contract packet | [service/solution_contract_output](solution_contract_output_protocol.md) available |
| `active: execution` | подготовить execution packet | [service/solution_contract_output](solution_contract_output_protocol.md) available |
| `active: validation` | подготовить validation packet | `common/final_validation` planned |
| `closed` или `rejected` | read-only summary; при cleanup request подготовить archive/tombstone packet | [service/issue_retention](issue_retention_protocol.md) available |
| `archived` | read-only summary и retention routing | [service/issue_retention](issue_retention_protocol.md) available |
| `tombstone`, `deleted` | read-only summary; не активировать без нового linked issue | none |
| `blocked` | показать blocker, owner и точное условие снятия | protocol исходной phase или dependency repair |
| `deferred` | не активировать без revive reason | revive decision / backlog update |

Если status и phase противоречат друг другу, агент не “чинит” молча. Он фиксирует inconsistency, предлагает repair write set и блокирует переход к предметной работе.

### 3. Проверить dependency readiness

Агент читает `dependency_refs` выбранного issue и соответствующие строки [dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).

Проверка считается чистой, если:

- нет cycle marker;
- все blocking edges удовлетворены или явно `not_applicable`;
- нет stale edge на несуществующий issue;
- parent/children/linked issue не требуют обязательного расширения контекста.

Если blocker найден, фокус можно загрузить только для диагностики. Перевод в `active` запрещён. Поле `ready` в JSONL не является dependency resolution без подтверждённого состояния blocking dependency.

### 4. Определить начальную phase

Для `approved` issue без phase агент выбирает начальную phase так:

| Условие | Начальная phase |
|---|---|
| reason weak/missing или есть materially important unknowns | `qa` |
| reason sufficient, но requirements отсутствуют | `requirements` |
| requirements approved, но тип issue не подтверждён | `requalification` |
| approved solution/contract уже есть | `execution` или `validation` по state |

Если нужный phase-протокол ещё `planned`, existing issue protocol не выполняет его вместо отсутствующего файла. QA, requirements, complex decomposition, linked dependency repair, retention и solution/contract/output доступны через [question_answer_protocol.md](question_answer_protocol.md), [requirements_protocol.md](requirements_protocol.md), [complex_issue_protocol.md](complex_issue_protocol.md), [linked_issues_protocol.md](linked_issues_protocol.md), [issue_retention_protocol.md](issue_retention_protocol.md) и [solution_contract_output_protocol.md](solution_contract_output_protocol.md); остальные planned-протоколы возвращают blocker `next_protocol_planned` с owner implementation issue.

### 5. Собрать focus packet

Минимальный focus packet содержит:

| Поле | Значение |
|---|---|
| `issue_id` | выбранный стабильный ID |
| `title` | title из registry |
| `mode` | `Service Mode` |
| `scope_path` | `/` или иной scope из registry |
| `status` / `phase` | reconstructed lifecycle state |
| `dependency_ready` | итог проверки graph |
| `reason_summary` | краткая сводка reason из registry или `reason.md` |
| `loaded_files` | только реально прочитанные файлы |
| `missing_expected_files` | expected paths, которые отсутствуют, с reason |
| `next_protocol` | available или planned protocol ID |
| `next_action` | одно ближайшее действие |

Для bootstrap implementation issue, у которых нет runtime папки, `missing_expected_files` не считается ошибкой, если registry прямо говорит, что full issue folder не создаётся до runtime phase. Пустые runtime-папки не создаются без фактического runtime-перехода.

### 6. Записать изменение focus, если оно требуется

Запись требуется, если агент:

- переводит `approved` issue в `active`;
- меняет phase;
- меняет active issue в [service_state.md](../../State/service_state.md);
- фиксирует blocker или inconsistency;
- обновляет dependency readiness.

Write set обычно включает:

1. [service_state.md](../../State/service_state.md);
2. [issue_registry.jsonl](../../Issues/issue_registry.jsonl);
3. [issue_registry.md](../../Issues/issue_registry.md);
4. [dependency_graph.jsonl](../../Issues/dependency_graph.jsonl), если менялись edge-status;
5. [persistence_log.jsonl](../../State/persistence_log.jsonl).

Если запись невозможна, агент возвращает `blocked_on_persistence` или `package_draft_not_committed` с write set. Он не заявляет, что issue “взят в работу”, пока state/registry не отражают это.

## Формат shortlist ответа

```text
Найденные issue:
| ID | Title | Status | Phase | Dependency | Next action |
|---|---|---|---|---|---|
| SVC-ISS-0001 | ... | approved | null | ready | выбрать / обсудить |

Ответ: выбрать <ID>.
```

## Формат focus packet ответа

```text
Issue выбран: <ID> — <title>.
Status / phase: <status> / <phase>.
Dependency: <ready|blocked|stale|cycle_blocked>.
Загружено: <короткий список source-of-truth файлов>.
Reason summary: <1-2 строки>.
Следующий protocol: <protocol_id> (<available|planned|blocked>).
Next action: <одно действие>.
```

Если следующий protocol planned, ответ обязан сказать это явно и не переходить к phase-work. Planned protocol нельзя исполнять как available.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Issue ID не найден | `issue_not_found` | вернуть shortlist или предложить создать новый issue через [new_issue_protocol.md](new_issue_protocol.md) |
| Найдено несколько кандидатов | `ambiguous_issue_reference` | показать shortlist, не выбирать автоматически |
| Registry parse error | `blocked_on_registry_parse` | не менять focus; создать repair write set |
| Dependency cycle | `blocked_on_dependency_cycle` | не переводить issue в active |
| Blocking dependency unsatisfied | `blocked_on_dependency` | показать edge IDs и condition для снятия |
| Expected runtime file отсутствует | `missing_issue_artifact` | проверить, допустим ли registry-only bootstrap; иначе repair blocker |
| Следующий phase protocol planned | `next_protocol_planned` | остановиться после focus packet и указать owner implementation issue |
| Persistence недоступен | `blocked_on_persistence` | вернуть write set, не заявлять active transition |

## Completion signal

Протокол завершён, когда выполнено одно из условий:

1. выбран ровно один existing issue, собран focus packet, проверены blockers и выбран next protocol;
2. пользователь получил shortlist для точного выбора;
3. выполнение остановлено с честным blocker, write set и next required action.

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
