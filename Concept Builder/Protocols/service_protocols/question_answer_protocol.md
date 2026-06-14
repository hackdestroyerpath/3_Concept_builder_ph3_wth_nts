# Протокол Question Answer

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-008`  
Protocol ID: `service/question_answer`  
Источник истины: `Protocols/service_protocols/question_answer_protocol.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

`Question Answer` — локальный протокол уточнения выбранного `issue` перед созданием или обновлением [requirements](requirements_protocol.md). Он нужен не для формального опроса пользователя, а для закрытия materially important unknowns: фактов, без которых требования, scope, contract или выполнение будут строиться на недостоверных допущениях.

Протокол применяется в `Service Mode`. После появления execution-слоя та же механика может использоваться для concept-local issue, но источник истины и registry будут локальными для концепции.

## Когда использовать

Запускай этот протокол, если выбранное issue находится в `active: qa` или если [existing issue protocol](existing_issue_protocol.md) определил, что без уточнения нельзя перейти к requirements.

| Сигнал | Действие |
|---|---|
| Нельзя сформулировать проверяемое требование | задать точечный вопрос |
| Есть несколько реалистичных вариантов, меняющих scope | спросить выбор или приоритет |
| В input, reason или registry есть противоречие | вынести противоречие пользователю |
| Reason достаточен для issue, но слаб для будущего contract | уточнить критерий результата |
| Пользовательское утверждение нужно превратить в requirement | уточнить формулировку или acceptance note |

QA пропускается, если issue очевидно, reason достаточен, scope узкий и requirements можно безопасно составить из уже сохранённых источников. Пропуск QA не отменяет [requirements_protocol.md](requirements_protocol.md): требования всё равно сохраняются и выносятся на утверждение. Это правило действует и для простых задач.

## Обязательные входы

| Вход | Источник |
|---|---|
| Выбранный issue | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) и, если есть, `Issues/<issue_id>/state.md` |
| Reason | registry `reason_summary` или `Issues/<issue_id>/reason.md` |
| Input refs | `input_ref` из registry и [../../Inbox/README.md](../../Inbox/README.md), если input находится в Inbox |
| Dependency status | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Protocol catalog | [../catalog.md](../catalog.md) |

Если issue является bootstrap registry-only entry и runtime-папки нет, QA не создаёт папку без операционной роли автоматически. Runtime QA-файлы создаются только когда issue реально проходит phase `qa` и это не противоречит текущему implementation scope.

## Preconditions

1. Issue выбран через [existing_issue_protocol.md](existing_issue_protocol.md) или создан через [new_issue_protocol.md](new_issue_protocol.md).
2. Registry и dependency graph парсятся без ошибок.
3. `dependency_ready` не равен `blocked`, `stale` или `cycle_blocked`, если blocker влияет на формирование требований.
4. Есть write scope для QA-файлов или честный `package_draft_not_committed` / `blocked_on_persistence` status.
5. Вопрос связан с будущим requirement, а не с побочным обсуждением вне scope.

## Материально важные неизвестные

Перед вопросами агент составляет короткий список unknowns.

| Поле | Значение |
|---|---|
| `unknown_id` | стабильный ID внутри issue, например `UNK-001` |
| `source` | откуда возникла неизвестность: reason, input, dependency, user assertion |
| `impact` | на что влияет: scope, requirement, contract, execution, validation |
| `blocking` | `true`, если без ответа нельзя составить requirements без догадок |
| `question_id` | связанный вопрос или `null`, если unknown отложен |
| `resolution` | `answered`, `deferred`, `accepted_risk`, `blocked` |

Если unknown не влияет на требования, его не нужно превращать в вопрос.

## Типы вопросов

| Type | Когда использовать | Формат ответа |
|---|---|---|
| `boolean` | нужно подтвердить одно условие | `да` / `нет` |
| `ternary` | допустимо отсутствие предпочтения | `да` / `нет` / `не имеет значения` |
| `choice` | нужно выбрать один или несколько вариантов | варианты + `другое` |
| `open` | без свободного ответа нельзя сформулировать requirement | короткий свободный ответ |
| `clarification` | пользователь не понял вопрос или указал непонятную часть | переформулированный вопрос |

Приоритет у вопросов, на которые можно ответить коротко. Развёрнутый вопрос допустим только если короткая форма исказит смысл.

## Runtime layout QA

Источник истины QA для runtime issue:

```text
Issues/<issue_id>/qa/
├── questions.jsonl
├── answers.jsonl
└── qa_summary.md
```

`qa_summary.md` является нормализованным именем файла в этой рабочей сети. Если при импорте найден старый вариант `qa_сводка.md`, агент создаёт compatibility note и не создаёт два равноправных источника истины.

## `questions.jsonl`

Каждая строка — один JSON object.

| Field | Значение |
|---|---|
| `question_id` | стабильный ID вопроса, например `Q-001` |
| `parent_question_id` | ID родительского вопроса или `null` |
| `unknown_ids` | unknowns, которые закрывает вопрос |
| `type` | `boolean`, `ternary`, `choice`, `open`, `clarification` |
| `question` | текст вопроса пользователю |
| `why` | зачем вопрос нужен для requirements |
| `allowed_answers` | варианты ответа или `null` |
| `suggested_answer` | предлагаемый ответ агента или `null` |
| `status` | `draft`, `asked`, `answered`, `rephrased`, `skipped` |
| `derived_requirement_ids` | requirement IDs, появившиеся из ответа |
| `created_at` / `updated_at` | timestamps |

## `answers.jsonl`

| Field | Значение |
|---|---|
| `answer_id` | стабильный ID ответа |
| `question_id` | связанный вопрос |
| `answer_text` | ответ пользователя |
| `normalized_answer` | короткая нормализация, если применимо |
| `agent_interpretation` | как агент понял ответ |
| `needs_confirmation` | `true`, если интерпретация неочевидна |
| `derived_requirement_ids` | requirement IDs, которые надо перенести в requirements |
| `created_at` | timestamp |

## `qa_summary.md`

Минимальная структура:

```text
# QA summary — <issue_id>

Parent: ../state.md
Status: draft | complete | blocked
Updated: <timestamp>

## Материально важные выводы
- ...

## Открытые unknowns
- ...

## Требования-кандидаты
- REQ-...: ...

## Переход к requirements
- ready | blocked
```

Сводка содержит только выводы, нужные для [requirements_protocol.md](requirements_protocol.md). Полный trace остаётся в JSONL. Нельзя переносить в summary всё подряд, иначе summary станет избыточным дубликатом полного trace.

## Порядок выполнения

1. **Focus read**: прочитать выбранное issue, reason, input refs и dependency status.
2. **Unknown scan**: выписать materially important unknowns и отделить их от несущественных деталей.
3. **Skip decision**: если unknowns нет, записать `qa_skipped_with_reason` в issue state или registry note и перейти к [requirements_protocol.md](requirements_protocol.md).
4. **Question drafting**: сформулировать минимальный набор вопросов. Одна пачка должна быть малой: обычно 1–5 вопросов.
5. **Persistence before asking**: сохранить `questions.jsonl` и, если нужно, `qa_summary.md` через [persistence protocol](../common/persistence_protocol.md).
6. **Ask user**: показать только вопросы, варианты ответа и зачем они нужны.
7. **Answer processing**: после ответа сохранить `answers.jsonl`, обновить статусы вопросов и `qa_summary.md`.
8. **Branch/rephrase**: если ответ породил уточнение или пользователь не понял вопрос, создать новый вопрос с `parent_question_id`.
9. **Completion check**: убедиться, что все blocking unknowns закрыты, отложены с reason или превращены в blocker.
10. **Transition**: перевести issue phase к `requirements` и открыть [requirements_protocol.md](requirements_protocol.md).

## Ветвление и непонятный вопрос

Если пользователь отвечает `не понял`, `что это значит` или указывает непонятную часть, агент:

1. не считает исходный вопрос answered;
2. создаёт вопрос типа `clarification`;
3. связывает его через `parent_question_id`;
4. упрощает формулировку без потери смысла;
5. сохраняет обе записи, чтобы trace оставался проверяемым и последовательным.

## Переход к requirements

QA завершён, когда выполнено одно из условий:

- все blocking unknowns имеют ответ и interpretation не требует подтверждения;
- unknowns явно отложены как не влияющие на первый draft requirements;
- найден blocker, без которого requirements будут недостоверны.

Если blocker есть, issue получает status `blocked` или phase остаётся `qa` с `blocking_reason`. Если blocker нет, следующий protocol — [requirements_protocol.md](requirements_protocol.md).

## Формат ответа пользователю при вопросах

```text
Issue: <issue_id> — <title>.
Phase: qa.
Сохранено: Issues/<issue_id>/qa/questions.jsonl.
Нужно уточнить:
1. <вопрос> — варианты: ...; зачем: ...

Ответь коротко по номерам.
```

Если persistence не выполнен, агент не пишет “сохранено”. Он возвращает `blocked_on_persistence` или `package_draft_not_committed` с write set.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Registry parse error | `blocked_on_registry_parse` | остановить QA, создать repair write set |
| Dependency blocker влияет на requirements | `blocked_on_dependency` | показать dependency edge и required artifact |
| Нет reason/input для вопроса | `blocked_on_missing_source` | вернуться к reason/intake repair |
| User answer ambiguous | `needs_confirmation` | сохранить interpretation и запросить подтверждение |
| QA-файлы нельзя записать | `blocked_on_persistence` | не задавать вопросы как сохранённый protocol step |
| Вопрос не влияет на requirements | `question_rejected_as_noise` | не задавать; записать skip reason при необходимости |

## Completion signal

Протокол завершён, когда QA trace сохранён или явно пропущен с reason, blocking unknowns закрыты/отложены/заблокированы, а issue готов перейти к [requirements_protocol.md](requirements_protocol.md) или получил честный blocker.

## Связанные файлы

- [Catalog](../catalog.md)
- [Requirements protocol](requirements_protocol.md)
- [Existing issue protocol](existing_issue_protocol.md)
- [New issue protocol](new_issue_protocol.md)
- [Service state](../../State/service_state.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Inbox rules](../../Inbox/README.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
