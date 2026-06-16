# Протокол загрузки контекста

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-004` / `OPT-003`  
Protocol ID: `common/context_loading`  
Источник истины: `Protocols/common/context_loading_protocol.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот протокол задаёт, какие файлы агент загружает при старте и перед выбором следующего действия. Цель — получить минимальный достаточный контекст, а не читать весь репозиторий без причины.

## Preconditions

- Root [README](../../README.md) существует.
- Relevant state существует:
  - [State/service_state.md](../../State/service_state.md) для `Service Mode`;
  - [State/execution_index.md](../../State/execution_index.md) для `Execution Mode`.
- [State/page_registry.jsonl](../../State/page_registry.jsonl) существует.
- [Protocols/catalog.md](../catalog.md) существует.

## Входы

| Вход | Обязательность | Назначение |
|---|---|---|
| `mode` | required or inferred | `service` или `execution` |
| `user_trigger` | required | команда пользователя или причина старта |
| `active_state_path` | required | путь к state текущего режима |
| `active_issue_id` | optional | заполняется после создания issue-модели |
| `active_concept_slug` | optional | только для `Execution Mode` |

## Порядок выполнения

1. **Entry read**: открыть [../../README.md](../../README.md).
2. **Mode state read**:
   - `Service Mode`: открыть [../../State/service_state.md](../../State/service_state.md);
   - `Execution Mode`: открыть [../../State/execution_index.md](../../State/execution_index.md).
3. **Registry check**: открыть [../../State/page_registry.jsonl](../../State/page_registry.jsonl) и проверить, что target Markdown-файлы имеют parent/backlink или entry.
4. **Protocol selection**: открыть [../catalog.md](../catalog.md), выбрать самый локальный protocol по trigger, mode, status, phase и issue type.
5. **Focus packet**: загрузить только:
   - active state;
   - active issue summary, если issue-модель уже создана;
   - выбранный protocol;
   - affected files из write set или requirements;
   - прямые dependencies.
6. **Context lift**: если данных недостаточно, зафиксировать недостающий факт, подняться на один уровень к parent summary, прочитать только нужный файл и вернуться в локальный scope.
7. **Drift check**: перед ответом проверить, что loaded context всё ещё соответствует active state и next-step marker.

## Context budget

| Уровень | Что можно загрузить | Когда допустимо |
|---|---|---|
| `entry` | root README, active state, page registry, catalog | default |
| `focused` | active issue state/reason, selected protocol, affected files | active issue выбран |
| `expanded` | parent summary, dependency summary, linked issue summary | без этого нельзя проверить blocker или requirement |
| `full_scope` | весь service или весь concept scope | финальная проверка или approved refactor |
| `repository_wide` | широкий обход всего репозитория | запрещён по умолчанию; только с reason и validation need |

## Completion signal

Протокол завершён, когда агент может назвать:

- текущий `mode`;
- active state path;
- active issue или reason, почему issue ещё нет;
- выбранный protocol или blocker;
- минимальный focus packet;
- next-step marker.

## Failure behavior

Если relevant state, catalog или registry отсутствуют, агент не продолжает выполнение по памяти. Он создаёт `blocked_on_missing_state_or_catalog` или ограниченный package draft и фиксирует write set для восстановления.

## Связанные файлы

- [Persistence protocol](persistence_protocol.md)
- [Service state](../../State/service_state.md)
- [Execution index](../../State/execution_index.md)
- [Page registry](../../State/page_registry.jsonl)
