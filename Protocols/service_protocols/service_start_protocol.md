# Протокол старта Service Mode

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-006`  
Protocol ID: `service/service_start`  
Источник истины: `Protocols/service_protocols/service_start_protocol.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот протокол описывает старт `Concept Builder Service Mode`: чтение корневой карты, восстановление service-состояния, проверку issue-модели и выбор одной из двух первичных веток — создать новый `issue` или взять существующий. Он не заменяет [common/context_loading](../common/context_loading_protocol.md), а задаёт service-level оболочку поверх него.

Цель старта — вернуть пользователю короткий статус и безопасную навигацию, а не менять файлы без выбранного protocol, issue и write set.

## Когда использовать

Протокол применяется при следующих trigger:

| Trigger | Условие | Действие |
|---|---|---|
| `пинг` | новый чат или неизвестный service scope | загрузить минимальный service context |
| `старт` | пользователь просит старт `Service Mode` | загрузить минимальный service context |
| `1` | нет активного меню, но пользователь запускает service-проект короткой командой | трактовать как service start, затем показать меню |
| `продолжай` | active state указывает на service-level next step | восстановить context и выбрать следующий локальный протокол |

Если `1` отправлено сразу после уже показанного service-меню, оно означает выбор ветки “создать новый issue” и передаётся в [new_issue_protocol.md](new_issue_protocol.md).

## Обязательные входы

| Файл | Зачем читать |
|---|---|
| [../../README.md](../../README.md) | корневая карта и разрешённые entry points |
| [../../State/service_state.md](../../State/service_state.md) | active scope, active issue, phase и next-step marker |
| [../../State/page_registry.jsonl](../../State/page_registry.jsonl) | проверка существования страниц, parent и orphan-risk |
| [../catalog.md](../catalog.md) | выбор ближайшего протокола |
| [../common/context_loading_protocol.md](../common/context_loading_protocol.md) | правила минимального focus packet |
| [../common/persistence_protocol.md](../common/persistence_protocol.md) | правило записи state и честного `pending/blocked` статуса |
| [../../Issues/issue_registry.md](../../Issues/issue_registry.md) | lifecycle issue и человекочитаемый snapshot |
| [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) | машинный registry для active/approved/blocked issue |
| [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) | readiness и blockers перед выбором issue |

[../../Inbox/README.md](../../Inbox/README.md) читается только если пользователь выбирает создание нового `issue` или уже передал input/attachments вместе с командой.

## Preconditions

Перед выводом service-статуса агент проверяет:

1. root [README](../../README.md), service state, page registry и catalog доступны;
2. [Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) парсится как JSONL;
3. [Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) парсится как JSONL и не содержит известного cycle blocker;
4. active issue из [../../State/service_state.md](../../State/service_state.md) существует в registry или явно отсутствует с reason;
5. planned-протокол не выполняется как будто он available;
6. write status не выдаётся за GitHub commit, если [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) не содержит подтверждённый `committed=true` marker.

## Порядок выполнения

1. **Entry read**: открыть [../../README.md](../../README.md) и найти service branch.
2. **State read**: открыть [../../State/service_state.md](../../State/service_state.md), выписать `Mode`, `Active scope`, `Active issue`, `Active phase`, `Write status`, `Blocking status` и next-step marker.
3. **Registry sanity check**: открыть [../../State/page_registry.jsonl](../../State/page_registry.jsonl), [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) и [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
4. **Protocol routing**: открыть [../catalog.md](../catalog.md) и выбрать ближайший `available` protocol:
   - создание нового issue: [new_issue_protocol.md](new_issue_protocol.md);
   - выбор существующего issue: [existing_issue_protocol.md](existing_issue_protocol.md);
   - продолжение QA phase: [question_answer_protocol.md](question_answer_protocol.md);
   - продолжение requirements phase: [requirements_protocol.md](requirements_protocol.md);
   - продолжение solution/execution phase: [solution_contract_output_protocol.md](solution_contract_output_protocol.md);
   - продолжение retention/archive/tombstone cleanup: [issue_retention_protocol.md](issue_retention_protocol.md);
   - продолжение implementation step: использовать next-step marker в [../../State/service_state.md](../../State/service_state.md);
   - phase `qa`, `requirements`, `solution` или `execution`: открыть [question_answer_protocol.md](question_answer_protocol.md), [requirements_protocol.md](requirements_protocol.md) или [solution_contract_output_protocol.md](solution_contract_output_protocol.md), если active issue уже сфокусирован.
5. **Focus packet**: удерживать только root service packet; не загружать все issue-папки, all concepts или development materials.
6. **User response**: вернуть короткий loaded-status, текущий next-step и доступные действия.
7. **Persistence decision**: если старт только читает состояние, запись не нужна. Если во время старта обнаружен broken state и агент создаёт repair issue/backlog, применяется [../common/persistence_protocol.md](../common/persistence_protocol.md).

## Формат ответа при чистом старте

```text
Service Mode загружен.
Состояние: <status из State/service_state.md>.
Active issue: <id или none>.
Write status: <committed / package_draft_not_committed / blocked>.
Доступные действия:
1 — создать новый issue
2 — взять в работу существующий issue
```

Ветка `2` доступна через [existing_issue_protocol.md](existing_issue_protocol.md). Если после фокусировки нужна QA, requirements, requalification, solution, execution, dependency repair или retention cleanup phase, агент использует [question_answer_protocol.md](question_answer_protocol.md), [requirements_protocol.md](requirements_protocol.md), [complex_issue_protocol.md](complex_issue_protocol.md), [linked_issues_protocol.md](linked_issues_protocol.md), [issue_retention_protocol.md](issue_retention_protocol.md) или [solution_contract_output_protocol.md](solution_contract_output_protocol.md). Если следующий phase-протокол всё ещё planned, агент обязан остановиться на focus packet и честно показать blocker.

## Формат ответа при `продолжай`

```text
Service context восстановлен.
Следующий ограниченный участок: <next-step marker>.
Будет применён protocol: <protocol_id или blocker>.
```

Если next-step требует planned-протокол, агент не выполняет предметную работу. QA, requirements, solution/contract/output, complex issue, linked issues и retention уже доступны; execution-layer должен ждать своего implementation issue.

## Error handling

| Ситуация | Статус | Действие |
|---|---|---|
| Нет root README/state/catalog | `blocked_on_missing_state_or_catalog` | остановить выполнение, создать repair write set |
| Registry JSONL не парсится | `blocked_on_registry_parse` | не выбирать issue; предложить repair issue |
| Dependency graph содержит cycle | `blocked_on_dependency_cycle` | не переводить issue в `active` |
| User command неоднозначна | `needs_user_choice` | запросить только выбор branch или entrypoint |
| GitHub write недоступен | `package_draft_not_committed` или `blocked_on_persistence` | не заявлять commit; вернуть пакет/write set |

## Completion signal

Протокол считается завершённым, когда известны:

- `mode = Service Mode`;
- active state path: [../../State/service_state.md](../../State/service_state.md);
- active issue или reason его отсутствия;
- выбранный next protocol или честный blocker;
- минимальный focus packet;
- краткий user-facing статус и первичная навигация.

## Связанные файлы

- [Catalog](../catalog.md)
- [Context loading protocol](../common/context_loading_protocol.md)
- [Persistence protocol](../common/persistence_protocol.md)
- [Question Answer protocol](question_answer_protocol.md)
- [Requirements protocol](requirements_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Complex Issue protocol](complex_issue_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Issue Retention protocol](issue_retention_protocol.md)
- [Service state](../../State/service_state.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Inbox rules](../../Inbox/README.md)
