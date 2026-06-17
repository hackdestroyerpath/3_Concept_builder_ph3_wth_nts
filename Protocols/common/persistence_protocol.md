# Протокол сохранения состояния

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-004`  
Protocol ID: `common/persistence_transaction`  
Источник истины: `Protocols/common/persistence_protocol.md`  
Status: `available`  
Updated: `2026-06-17T17:23:49Z`

## Назначение

Этот протокол задаёт transaction-like порядок записи рабочих файлов. GitHub не обязан давать атомарность на пачку файлов, поэтому агент обязан делать запись проверяемой, ограниченной, классифицированной и восстановимой.

Донорские репозитории могут давать только локальные правила write-safety. Архитектура лидера сохраняется: сначала читается актуальный контекст, затем проверяется конфликт, затем пишутся primary artifacts, затем связанные indexes/state/registry/backlinks, затем выполняется light validation, и только последней записью фиксируется marker сохранения.

## Preconditions

- Active scope определён через [context_loading_protocol.md](context_loading_protocol.md).
- Target files входят в approved scope текущего issue/protocol.
- Write set составлен до изменения файлов.
- Для GitHub-записи известны repository, branch и допустимый write scope. Если они неизвестны, операция остаётся `package_draft_not_committed` или `blocked_on_persistence`.
- Для `update`, `delete`, `move` или `archive` известны свежие `pre_sha` или актуальное содержимое target file.
- Агент не пишет development-only материалы в production tree и не создаёт scripts/runtime concept folders без явного approved issue/contract.

## Классификация target files

Перед записью каждый target path получает классификацию. Классификация входит в write package и проверяется до GitHub mutation.

| Class | Meaning | Action |
|---|---|---|
| `production` | Runtime repository files: `README.md`, `State/`, `Instructions/`, `Protocols/`, `Issues/`, `Inbox/`, `Concepts/`, templates, validation | Allowed only inside current approved scope |
| `development` | Handoff, prompts, checkpoint archives, audit/source notes, temporary reports | Do not write into production tree |
| `conditional` | Scripts, generated archives, heavy attachments, optional machine companions | Requires explicit issue/contract/benefit reason |
| `debris` | Obsolete duplicate paths or repair leftovers | Remove/migrate only with evidence and validation |

Если классификация неоднозначна, запись target path не выполняется до решения scope. Безопасная часть write set может быть записана отдельно; небезопасная часть остаётся в blocker/remaining write set.

## Write package

Перед записью агент фиксирует пакет. Это не отчёт после факта, а входной контракт записи.

```text
mode:
active_object:
active_issue:
reason:
operation: create | update | delete | move | archive
classification:
target_paths:
pre_sha:
expected_post_state:
related_registry_paths:
related_state_paths:
related_event_paths:
validation_plan:
rollback_plan:
write_mode: branch_pr | direct_main | package_draft
branch:
readback_refs:
```

Минимальные требования к полям:

| Поле | Требование |
|---|---|
| `mode` | `Service Mode`, `Execution Mode` или другой явно поддержанный режим. |
| `active_object` | Root service, concept slug, issue или иной объект, которому принадлежит запись. |
| `active_issue` | Issue/backlog ID или `none` с reason, если запись выполняется как structural repair. |
| `reason` | Почему изменение нужно сейчас и почему выбранные paths входят в scope. |
| `operation` | Одна из разрешённых операций: `create`, `update`, `delete`, `move`, `archive`. |
| `classification` | Одна классификация на каждый target path. |
| `target_paths` | Root-relative paths, которые изменяются. |
| `pre_sha` | SHA/content baseline для существующих файлов или `absent` для новых файлов. |
| `expected_post_state` | Проверяемые признаки результата после записи. |
| `related_registry_paths` | Registry/backlink/catalog paths, которые должны измениться или явно остаться без изменений. |
| `related_state_paths` | State files, которые должны измениться или получить no-change reason. |
| `related_event_paths` | Issue/event/log paths для traceability, если применимо. |
| `validation_plan` | Конкретные JSONL/link/readback/style/boundary checks. |
| `rollback_plan` | Безопасный patch-forward/rollback/blocker plan до записи. |
| `write_mode` | `branch_pr`, `direct_main` или `package_draft`. |
| `branch` | Active branch/ref, где выполняется запись или `none` для package draft. |
| `readback_refs` | Заполняется после записи ссылками на GitHub file readback, commit/PR metadata, compare output или validation refs. |

## Порядок записи

1. **Preflight read**: перечитать latest [service_state.md](../../State/service_state.md) или relevant concept state, [page_registry.jsonl](../../State/page_registry.jsonl), registry и target files.
2. **Conflict check**: если target изменился после загрузки, reload и безопасный merge; при конфликте — abort с blocker.
3. **Classify/write package**: классифицировать target files и зафиксировать write package до mutation.
4. **Apply content first**: сохранить primary artifacts: instructions, protocol, issue file, requirements, solution, output, page или export.
5. **Update indexes after content**: обновить state, issue registry, page registry, backlinks, dependency graph и parent summaries после content writes.
6. **Light validation**: проверить ссылки изменённых Markdown-файлов, наличие parent link, JSONL parse, orphan-risk, production/development boundary и соответствие write package.
7. **Commit marker last**: последней записью добавить JSONL-строку в [persistence_log.jsonl](../../State/persistence_log.jsonl), если repository write доступен и marker входит в approved scope.
8. **Response after persistence**: ответить пользователю только тем статусом, который уже отражён в сохранённых файлах или честно указан как pending/blocker/package draft.

Эта последовательность не заменяется donor protocol wholesale. Donor rules могут только усиливать отдельные проверки внутри этого порядка.

## Write modes

| `write_mode` | Семантика | Обязательная честность статуса |
|---|---|---|
| `branch_pr` | Запись идёт в task branch; validation и readback выполняются на этой ветке; PR/merge проверяются отдельно. | Нельзя заявлять main verification до merge и readback из `main`. |
| `direct_main` | Прямая запись в `main`, разрешённая только явным контрактом repair/direct-main. | Отчёт обязан назвать direct-main и дать readback из `main`. |
| `package_draft` | GitHub commit не выполнен; агент возвращает proposed files/write set/package. | `committed=false`; нельзя заявлять GitHub persistence. |

`branch_pr` может иметь статус `committed_on_task_branch` или `partial` до merge. `direct_main` может иметь final persistence status только после readback target paths из `main`. `package_draft` является честным fallback, а не скрытым commit.

## Readback evidence и статусные слова

Слова `synced`, `committed`, `passed`, `ready`, `closed`, `final`, `OK` или любые эквивалентные final-status labels допустимы только при наличии evidence.

Valid evidence:

- GitHub file readback target paths на активной ветке;
- commit metadata или PR metadata;
- compare output для branch/base или pre/post refs;
- validation report references, которые сами указывают checked paths, readback refs и failed checks.

Branch work не проверяется через `main` до merge. Если работа идёт в branch, статус формулируется как branch-scoped. Если работа идёт direct-main, отчёт прямо говорит `direct_main` и показывает main readback. Если readback отсутствует или расходится с expectation, статус остаётся `written_unverified`, `partial`, `blocked_on_persistence` или `package_draft_not_committed`.

## Формат persistence log entry

Минимальные поля новой строки [persistence_log.jsonl](../../State/persistence_log.jsonl):

```json
{"transaction_id":"...","timestamp":"...","mode":"Service Mode","issue_id":"...","scope":"...","repository":"...","branch":"...","write_mode":"branch_pr|direct_main|package_draft","committed":false,"status":"package_draft_not_committed","write_set":["..."],"changed_files":["..."],"readback_refs":["..."],"validation_refs":["..."],"next_state":"...","blocking_status":"none|...","notes":"..."}
```

`committed=false` означает, что создан package draft или локальный checkpoint, но GitHub commit не подтверждён. `committed=true` допустим только после фактической успешной записи в GitHub и проверки commit marker/readback. Исторические строки не переписываются ради новой схемы; новая схема применяется к новым операциям. Destructive log rewrite запрещён без отдельного repair reason и validation.

## Failure behavior

| Сбой | Действие |
|---|---|
| Нет write-инструмента GitHub | вернуть `package_draft_not_committed`, приложить архив или write set, не заявлять commit |
| Нет прав или branch protected | вернуть `blocked_on_persistence`, сохранить recovery plan, не заявлять сохранение |
| Stale `sha` | перечитать target, получить свежий `sha`, перепланировать bounded write; старый запрос не повторять вслепую |
| `409` conflict | остановить blind overwrite, reload target branch/ref, сравнить expected/current, retry только если конфликт понятен |
| Unexpected target difference | зафиксировать mismatch, проверить branch/ref и scope, затем patch-forward, rollback или blocker |
| Partial write | перечитать changed paths, перечислить written/pending paths, продолжить связку вперёд или вернуть blocker с remaining write set |
| Unsafe rollback | не откатывать, если rollback уничтожит verified work, затронет чужой scope или потребует force/default-branch rewrite |
| Validation failed after write | не закрывать issue; сохранить validation result/recovery plan и выбрать patch-forward, safe rollback или blocker |
| Conflict target file | reload, merge только когда конфликт понятен, иначе abort с blocker |
| Запись оборвалась до commit marker | записать failed/partial transaction, если возможно; иначе сообщить uncertain write set и readback status |

Запрещены blind overwrite, merge без понимания различия, rollback поверх verified user work и final-status claim без readback. Если безопасна только часть write set, сначала записывается безопасная часть, а оставшаяся часть получает `remaining_write_set` и next recovery step.

## Package draft behavior

Если commit нельзя безопасно выполнить:

1. вернуть `committed=false`;
2. предоставить proposed files или write set;
3. записать blocker/package draft status в state/log, если это безопасно и доступно;
4. не заявлять GitHub persistence, synced/ready/final status или main verification;
5. указать recovery plan: required branch, target paths, pre_sha/readback needs, validation plan.

Package draft может быть достаточным только когда задача явно просит package-only result или GitHub write недоступен/небезопасен. Для GitHub-oriented задачи package draft не маскирует незавершённую persistence.

## Completion signal

Операция завершена, когда выполнено одно из двух:

1. `committed=true`: target files записаны, readback evidence совпадает с expected_post_state, related state/registry/backlinks/log согласованы, commit marker записан последним или documented no-marker reason входит в validation refs;
2. `committed=false`: package draft или blocker честно отражён в state/log или return package, пользователь получил proposed files/write set, а ответ не заявляет GitHub persistence.

Для branch work completion scope остаётся branch-scoped до merge/readback из `main`. Для direct-main completion scope должен быть подтверждён readback из `main`.

## Связанные файлы

- [Context loading protocol](context_loading_protocol.md)
- [Service state](../../State/service_state.md)
- [Persistence log](../../State/persistence_log.jsonl)
- [Page registry](../../State/page_registry.jsonl)
