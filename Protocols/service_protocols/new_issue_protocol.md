# Протокол создания нового issue

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-006` / `USER-004`  
Protocol ID: `service/new_issue`  
Источник истины: `Protocols/service_protocols/new_issue_protocol.md`  
Status: `available`  
Updated: `2026-06-04T23:49:04Z`

## Назначение

Этот протокол переводит вход пользователя в управляемые `issue`: сохраняет input в [Inbox](../../Inbox/README.md), выделяет entrypoint и attachments, создаёт draft/proposed registry entries, создаёт `state.md` и `reason.md` для предложенных issue и возвращает пользователю только то, что уже отражено в файлах.

Главное ограничение: агент не анализирует input как источник истины, пока input не сохранён или пока не зафиксирован честный `blocked_on_persistence`. Решение без сохранённого input не считается проверяемым.

## Когда использовать

Протокол применяется в `Service Mode`, если пользователь:

- выбрал `1` после [service_start_protocol.md](service_start_protocol.md);
- написал `создать новый issue`, `создать issue`, `новый issue` или эквивалентную команду;
- передал текст, файлы или attachments как материал для нового service-level issue;
- на этапе утверждения существующего proposed registry явно добавил новую отдельную задачу.

Execution-level issue внутри `Concepts/` будут использовать те же базовые правила traceability, но локальная concept-раскладка создаётся позже в execution-слое. До появления execution-протоколов этот файл является service-level источником истины.

## Обязательные входы

| Вход | Источник |
|---|---|
| Команда пользователя и текст input | сообщение пользователя |
| Attachments | файлы пользователя, если есть |
| Текущий service state | [../../State/service_state.md](../../State/service_state.md) |
| Правила Inbox | [../../Inbox/README.md](../../Inbox/README.md) |
| Issue lifecycle и schema | [../../Issues/issue_registry.md](../../Issues/issue_registry.md) |
| Machine registry | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) |
| Dependency graph | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |

## Preconditions

1. [service_start_protocol.md](service_start_protocol.md) или эквивалентный context load подтвердил `Service Mode`.
2. [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) и [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) читаются без parse error.
3. Для записи известен write scope. Если фактический GitHub commit невозможен, результат остаётся package draft или `blocked_on_persistence`.
4. Новый issue не дублирует очевидный active/proposed/approved issue. Проверка дубля делается по title, reason_summary, input_ref и linked refs.
5. Reason quality не ниже `sufficient` для перехода в `proposed`.

## Разбор input

| Ситуация | Entry point | Attachments | Действие |
|---|---|---|---|
| Пользователь явно указал главный файл | указанный файл | остальные приложенные файлы | сохранить связь в `Inbox/<input_id>/manifest.md` |
| Пользователь дал команду и текст | текст после команды | приложенные файлы | сохранить текст как `entrypoint.md` |
| Пользователь дал только файлы | неизвестно | все файлы pending | если связь неочевидна, сохранить pending manifest и запросить entrypoint |
| Пользователь дал несколько независимых задач | общий input или несколько entrypoints | файлы по связи | создать несколько candidate issue, но не смешивать reasons |

Если разделить entrypoint и attachments невозможно, агент сохраняет raw input как pending inbox packet, фиксирует `intake_needs_entrypoint` и задаёт один короткий вопрос. Proposed issue при этом не создаётся.

## Runtime layout Inbox

Создаваемые во время реальной работы input-папки имеют вид:

```text
Inbox/<input_id>/
├── manifest.md
├── entrypoint.md
├── raw_user_message.md
├── attachments_manifest.jsonl
└── attachments/
```

Для бинарных или внешних файлов `manifest.md` хранит только проверяемое описание, filename, source, received timestamp, content hash если он доступен, и связь с `issue_id`. Агент не должен выдавать содержимое нераспознанного attachment как прочитанный факт.

## ID allocation

| Объект | Формат | Источник следующего номера |
|---|---|---|
| Input packet | `INBOX-YYYYMMDD-NNNN` | максимальный существующий input id за дату в `Inbox/` |
| Service issue | `SVC-ISS-0001` | максимальный `SVC-ISS-*` в [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl), включая closed/rejected/tombstone |
| Execution issue | `<concept_slug>-ISS-0001` | локальный registry концепции, после реализации execution layer |

ID не переиспользуется после reject, archive, tombstone или deletion.

## Reason quality gate

Перед созданием `proposed` issue агент проверяет, что reason отвечает на пять критериев из [issue registry](../../Issues/issue_registry.md):

| Критерий | Минимальная проверка |
|---|---|
| `Source` | понятно, из какого input или решения возникла задача |
| `Problem / need` | названа проблема, риск, неопределённость или изменение |
| `Value` | понятно, что улучшится после закрытия |
| `Scope boundary` | указано, что входит и что не входит |
| `Non-duplicate` | проверено, что это не дубль существующего issue |

Если reason `missing` или `weak`, агент не создаёт `proposed`. Допустимые действия:

1. задать 1–3 точечных вопроса только по недостающему критерию;
2. предложить `agent_inferred_reason` на утверждение, если он надёжно следует из input;
3. оставить draft entry в статусе `creating`, если input уже сохранён, но reason ещё не готов.

## Создание proposed issue

Для каждого issue, прошедшего reason gate, агент создаёт или обновляет:

```text
Issues/<issue_id>/
├── state.md
└── reason.md
```

Минимальный `state.md` содержит:

| Поле | Значение |
|---|---|
| `issue_id` | стабильный ID |
| `status` | `proposed` |
| `phase` | `null` |
| `scope_type` | `service` |
| `scope_path` | `/` |
| `input_ref` | `Inbox/<input_id>/manifest.md` |
| `reason_path` | `Issues/<issue_id>/reason.md` |
| `next_step_marker` | ожидание решения пользователя: approve/reject/discuss/add |

Минимальный `reason.md` содержит source, problem/need, value, scope boundary, non-duplicate check и краткий reason summary. Текст reason в ответе пользователю должен совпадать по смыслу с этим файлом.

## Последовательность записи

1. Сформировать write set: `Inbox`, candidate issue folders, registry, graph, page registry, persistence log.
2. Сохранить `Inbox/<input_id>/` и manifest.
3. Создать draft registry entries со статусом `creating`, если issue ещё анализируется.
4. Провести reason quality gate и duplicate check.
5. Для каждого достаточного issue создать `Issues/<issue_id>/state.md` и `Issues/<issue_id>/reason.md`.
6. Обновить [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl), [../../Issues/issue_registry.md](../../Issues/issue_registry.md) и, если появились связи, [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
7. Обновить [../../State/page_registry.jsonl](../../State/page_registry.jsonl) для новых Markdown-файлов.
8. Последним шагом добавить запись в [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl).
9. Только после этого вернуть пользователю краткую таблицу proposed issue.

## Команды решения по proposed registry

После показа предложенного реестра пользователь может ответить:

| Команда | Эффект |
|---|---|
| `утвердить все` | все `proposed` issue переходят в `approved` |
| `утвердить SVC-ISS-0001` | выбранные issue переходят в `approved` |
| `отклонить SVC-ISS-0001: <reason>` | выбранные issue переходят в `rejected` с reason |
| `обсудить SVC-ISS-0001` | выбранные issue переходят в `needs_discussion` |
| `добавить issue: <title/reason>` | создаётся новая отдельная candidate-запись после обработки текущих решений |

## Ветка добавления issue во время утверждения

Это закрывает пользовательское issue `USER-004`.

Если в одном ответе пользователь одновременно утверждает/отклоняет текущие proposed issue и просит добавить новое, агент действует в таком порядке:

1. применить решения к уже предложенным issue;
2. сохранить обновлённый registry/state;
3. отдельно обработать добавленное issue через этот же protocol;
4. проверить reason quality;
5. сохранить новый input_ref или linked reason;
6. добавить новый issue только как `creating` или `proposed`, не переключая active focus;
7. вернуть одну сводку: какие решения применены и какое новое issue добавлено или почему оно ждёт reason.

Агент не имеет права переходить к глубокой работе по новому issue, пока не завершён этап утверждения текущего набора. Фокус работы ограничен текущим утверждённым набором.

## Формат ответа после успешного создания proposed registry

```text
Input сохранён: Inbox/<input_id>/manifest.md.
Созданы proposed issue:
| ID | Title | Reason summary | Next action |
|---|---|---|---|
| SVC-ISS-0001 | ... | ... | утвердить / отклонить / обсудить |

Доступные ответы:
- утвердить все
- утвердить <ID>
- отклонить <ID>: <reason>
- обсудить <ID>
- добавить issue: <title + reason>
```

Если запись не подтверждена, этот формат запрещён. В ответе должен быть `blocked_on_persistence` или `package_draft_not_committed` с write set.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Нет entrypoint | `intake_needs_entrypoint` | сохранить pending inbox packet, спросить один уточняющий вопрос |
| Reason слабый | `needs_user_reason` | не создавать `proposed`; запросить недостающий reason или предложить inferred reason |
| Duplicate найден | `duplicate_candidate` | предложить linked/update existing issue вместо нового |
| Нельзя записать Inbox | `blocked_on_persistence` | не анализировать input как сохранённый source of truth |
| Registry parse error | `blocked_on_registry_parse` | не создавать issue; подготовить repair step |
| Dependency conflict | `blocked_on_dependency_graph` | не переводить в approved/active до устранения |

## Completion signal

Протокол завершён, когда выполнено одно из условий:

1. input сохранён в [../../Inbox/README.md](../../Inbox/README.md)-совместимом layout, registry и proposed issue artifacts сохранены, persistence marker записан;
2. создание остановлено с честным blocker, write set и next required user answer/action.

## Связанные файлы

- [Service start protocol](service_start_protocol.md)
- [Existing issue protocol](existing_issue_protocol.md)
- [Question Answer protocol](question_answer_protocol.md)
- [Requirements protocol](requirements_protocol.md)
- [Inbox rules](../../Inbox/README.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
