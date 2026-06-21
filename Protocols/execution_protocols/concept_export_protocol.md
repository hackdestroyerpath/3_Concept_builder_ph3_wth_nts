# Протокол экспорта концепции

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-011`  
Protocol ID: `execution/concept_export`  
Источник истины: `Protocols/execution_protocols/concept_export_protocol.md`  
Status: `available`  
Updated: `2026-06-21T03:03:27Z`

## Назначение

Этот протокол управляет проверкой возможности экспорта и созданием export package для выбранной концепции. Он сохраняет проверяемую concept-сеть, traceability решений, issue, dependency evidence и validation evidence, не подменяя runtime-состояние концепции отдельной donor-архитектурой.

Протокол поддерживает три операции:

| Operation | Mutating | Результат |
|---|---:|---|
| `export_precheck` | нет | машиночитаемый dry-run: разрешённый тип результата, blockers, limitations и next step |
| `closed_concept` | да | финальный package только после прохождения всех closure/export gates |
| `work_in_progress` | да | явно нефинальный WIP package с конкретными limitations и open items |

`export_precheck` всегда выполняется до создания package. Он не создаёт export directory, archive, registry row, persistence row и не изменяет concept/root state.

## Когда использовать

- пользователь просит проверить возможность экспорта;
- пользователь просит выгрузить концепцию;
- concept state перешёл в `ready_for_closure_review` или `closed`;
- нужен проверяемый package для передачи другому агенту;
- нужен промежуточный snapshot с открытыми issue;
- нужно определить, допустим ли `closed_concept` либо только `work_in_progress`;
- нужно диагностировать export blocker без mutation.

Этот протокол не заменяет issue output из service/execution lifecycle. Для результата отдельного issue используется solution/contract/output workflow, а concept-level export собирает сеть целиком.

## Обязательные входы

| Вход | Источник |
|---|---|
| Root execution index | [../../State/execution_index.md](../../State/execution_index.md) |
| Concept layer entry | [../../Concepts/README.md](../../Concepts/README.md) |
| Selected concept README | `Concepts/<concept_slug>/README.md` |
| Concept state | `Concepts/<concept_slug>/State/concept_state.md` |
| Local page registry | `Concepts/<concept_slug>/State/page_registry.jsonl` или сохранённый `page_map.md` |
| Local issue registry | `Concepts/<concept_slug>/Issues/issue_registry.jsonl` |
| Local dependency graph | `Concepts/<concept_slug>/Issues/dependency_graph.jsonl` |
| Output refs | `Concepts/<concept_slug>/Output/` и issue output refs, если они нужны для понимания концепции |
| Export request | requested operation/type, user acceptance для WIP limitations, если требуется |
| Protocol route | [README execution-протоколов](README.md) и [catalog.md](../catalog.md) |
| Persistence rules | [persistence_protocol.md](../common/persistence_protocol.md) |

Если выбранная концепция отсутствует, package создавать нельзя. `export_precheck` возвращает `allowed_result = blocked`, blocker `blocked_no_concept` и точный next step.

## `export_precheck`

### Read-only contract

`export_precheck` читает только выбранный concept scope и необходимые root routing companions. До выдачи результата запрещено:

- создавать `Concepts/<concept_slug>/Exports/`;
- резервировать export ID или path;
- собирать ZIP;
- обновлять state, registry, execution index или persistence log;
- исправлять найденные дефекты внутри того же precheck.

Precheck фиксирует фактическую `state_revision`, integrity/readiness evidence, source-link состояние, issue/dependency состояние и возможный package route. Сгенерированное имя в precheck является candidate и не считается сохранённым export до persistence/readback.

### Минимальная схема результата

```yaml
operation: export_precheck
concept_slug: string|null
requested_export_type: closed_concept|work_in_progress
concept_state_readable: true|false
state_revision: integer|null
integrity_status: verified|unverified|stale|conflict|unknown
readiness_status: string|null
required_output_status: ready|missing|not_applicable|unknown
page_registry_parse: pass|fail|not_available
orphan_files: []
broken_source_links: []
open_blocking_issues: []
open_nonblocking_issues: []
blocking_dependencies: []
validation_status: pass|pass_with_deferred_items|blocked|not_available
language_gate: pass|fail|not_checked
candidate_export_id: string|null
candidate_archive_name: string|null
candidate_export_path: string|null
package_local_open_status: pass|fail|not_run
blockers: []
limitations: []
allowed_result: closed_concept|work_in_progress|blocked
next_step: string
```

Массив `blocking_dependencies` содержит для каждого direct blocking edge observed status, normalized readiness и evidence. Только normalized `ready` допускает closure/export. Legacy `status = satisfied` без required artifact/state evidence не считается `ready`.

### Выбор `allowed_result`

1. `closed_concept` разрешён только при отсутствии blockers и при canonical validation `pass` либо `pass_with_deferred_items`, где deferred items неблокирующие, имеют owner/reason/next action и приняты пользователем для closure.
2. `work_in_progress` разрешён только если concept state читаем, `integrity_status = verified`, package может быть собран без entrypoint/root-escape нарушения, а все переносимые defects перечислены как exact limitations.
3. `blocked` возвращается, когда mutation небезопасна, identity/integrity не подтверждены, действует hard local-open blocker (missing/unopenable entrypoint или root escape), path конфликтует или persistence недоступен.
4. Запрос `closed_concept` не понижается молча до WIP. Precheck может предложить WIP как next step, но создание требует явного выбора/acceptance.

### Перевод donor blockers в leader semantics

- `invalid state hash` означает `integrity_status != verified`, stale/conflicting revision либо отсутствие достаточного integrity basis; обязательный `state_hash` не вводится;
- `manifest/structure mismatch` означает расхождение local page registry, derived `page_map.md`, package manifest и фактически включённых файлов; обязательные donor `state.json`, `manifest.jsonl` и `structure.md` не вводятся.

## Blocker и limitation matrix

| Условие | `closed_concept` | `work_in_progress` | Код / обязательное evidence |
|---|---|---|---|
| Нет active/selected concept | blocked | blocked | `blocked_no_concept`; `concept_slug = null` |
| Нет concept README или state | blocked | blocked | `blocked_missing_concept_entrypoint_or_state` |
| Concept state не читается | blocked | blocked | `blocked_unreadable_concept_state` |
| `integrity_status` не `verified` | blocked | blocked | `blocked_integrity_not_verified`; observed status и basis |
| Revision stale/conflict или basis отсутствует | blocked | blocked | `blocked_state_revision_or_integrity_basis` |
| Readiness ниже export/closure gate | blocked | `draft_with_notice` только при verified integrity | `blocked_readiness_for_closed`; observed readiness |
| Required output отсутствует | blocked | `draft_with_notice`, если package всё ещё понимаем | `blocked_required_output_missing`; exact output refs |
| Local page registry не парсится | blocked | blocked, пока нельзя доказать complete included set | `blocked_page_registry_parse` |
| Есть orphan или broken source link | blocked | `draft_with_notice`, кроме missing root README/root escape | `blocked_source_graph_for_closed`; exact paths |
| Есть open blocking issue | blocked | `draft_with_notice` | `blocked_open_issue`; issue IDs/status |
| Direct blocking dependency не normalized `ready` | blocked | `draft_with_notice` | `blocked_dependency_not_ready`; raw `satisfied` недостаточно |
| Concept validation отсутствует | blocked | `draft_with_notice` | `blocked_validation_not_available` |
| Validation не `pass`/`pass_with_deferred_items` | blocked | `draft_with_notice` только для documented non-structural defects | `blocked_validation_status` |
| Language gate = `fail` | blocked | `draft_with_notice` | `blocked_language_gate`; affected files |
| Manifest/page-map/content расходятся | blocked | `draft_with_notice`, если entrypoint/root safety сохранены | `blocked_package_content_mismatch` |
| Root README отсутствует/не открывается | blocked | blocked | `blocked_package_entrypoint` |
| Относительный path выходит из package root | blocked | blocked | `blocked_package_root_escape` |
| Package local-open fail по другим target | blocked | `draft_with_notice` с exact broken targets | `blocked_package_local_open_for_closed` |
| Export ID/path уже занят другим content/state | blocked | blocked | `blocked_export_id_collision` |
| Persistence недоступен | precheck может завершиться, export blocked | precheck может завершиться, export blocked | `blocked_on_persistence` |

Closed export невозможен, пока остаётся хотя бы один closed-export blocker. Blocker нельзя переименовать в limitation ради продолжения операции.

## Политика `draft_with_notice`

WIP export с дефектами допустим только когда одновременно выполняются условия:

1. concept identity и state revision подтверждены;
2. `integrity_status = verified`;
3. root README существует и открывается;
4. ни один included path не выходит из package root;
5. manifest перечисляет каждый defect как конкретную limitation;
6. open blockers/issues/dependencies сохранены в `issue_сводка.md` и validation report;
7. пользователь явно принял перечисленные limitations;
8. manifest и ответ маркируют package как `work_in_progress`, `draft_with_notice`, `not_final`.

WIP нельзя представлять как закрытую или финальную концепцию. User acceptance относится к конкретному списку limitations; изменение списка требует нового acceptance.

## Preconditions для `closed_concept`

1. `Concepts/<concept_slug>/README.md` существует и является entrypoint.
2. Concept state имеет status `ready_for_closure_review` или `closed` и `integrity_status = verified`.
3. `state_revision` задан и соответствует readback scope.
4. Local page registry парсится и не содержит orphan Markdown.
5. Все required pages связаны parent/backlink; source links существуют.
6. Все blocking concept issue имеют terminal status: `closed`, `rejected`, `deferred`, `archived`, `tombstone` или `deleted` с сохранённым tombstone; deferred blockers не допускаются.
7. Каждый direct blocking dependency normalized как `ready` с evidence.
8. Contract/output coverage сохранён для required issue.
9. Concept-level validation имеет canonical status `pass` или `pass_with_deferred_items`; deferred items неблокирующие и приняты для closure.
10. Language gate = `pass`.
11. Package manifest, page map и included contents согласованы.
12. Package-root local-open = `pass`.
13. Candidate export path свободен либо доказан exact idempotent retry.
14. Пользователь утвердил closure, если export меняет concept status на `closed`.

Если хотя бы один пункт не выполнен, closed export запрещён. Возможен только явно выбранный WIP route при соблюдении `draft_with_notice` или `blocked`.

## Preconditions для `work_in_progress`

1. Concept README и concept state существуют и читаются.
2. `integrity_status = verified`; state revision известна.
3. Root package README существует и не создаёт root escape.
4. Open issue, dependencies, source/package defects и limitations перечислены точно.
5. Manifest содержит `export_type = work_in_progress`, `finality = not_final` и policy `clean_draft` либо `draft_with_notice`.
6. Пользователь принял limitations для `draft_with_notice`.
7. В ответе явно сказано, что концепция не закрыта.

## Export package layout

Источник истины export создаётся внутри concept scope:

```text
Concepts/<concept_slug>/Exports/<export_id>/
├── manifest.md
├── <export_id>.zip
├── page_map.md
├── issue_сводка.md
└── validation_report.md
```

## Детерминированное имя и collision policy

```text
export_id = <concept_slug>__<closed|wip>__r<state_revision>__<YYYYMMDDTHHMMSSZ>
archive_name = <export_id>.zip
export_path = Concepts/<concept_slug>/Exports/<export_id>/
```

Правила:

- timestamp берётся из фактического UTC времени начала persistence transaction;
- тип `closed` соответствует `closed_concept`, тип `wip` соответствует `work_in_progress`;
- random suffix и predicted commit SHA запрещены;
- `state_revision` берётся из precheck/readback, а не вычисляется из package;
- существующий path не перезаписывается молча;
- существующий path с другим revision/content блокирует operation как `blocked_export_id_collision`;
- exact idempotent retry разрешён только когда readback доказывает одинаковые export type, revision, included-file identities, manifest fields и archive identity;
- retry не создаёт новую registry/persistence запись, пока не установлено, что предыдущая transaction не завершилась.

## `manifest.md` schema

| Раздел | Содержание |
|---|---|
| Metadata | export ID, UTC timestamp, concept slug, state revision, source commit/package marker |
| Export type | `closed_concept` или `work_in_progress` |
| Finality | `final` только для closed; `not_final` для WIP |
| WIP policy | `clean_draft` или `draft_with_notice` |
| Entry point | root-relative path к `README.md` внутри archive |
| Included files | полный список included Markdown/state/output refs и identity evidence |
| Excluded files | development-only, raw, heavy, private и context-loss decisions с reason |
| Issue summary | closed / rejected / deferred / archived / tombstone / open |
| Dependency status | normalized readiness, blockers, stale/cycles и evidence |
| Validation result | `pass`, `pass_with_deferred_items` или `blocked` |
| Local-open result | status, checked files, broken targets, root-escape result |
| Known limitations | обязательно для WIP; exact paths/issue/dependency IDs |
| Reuse notes | что следующий агент должен открыть первым |

## Archive minimum

```text
README.md
Pages/
State/page_registry.jsonl или page_map.md
Issues/issue_сводка.md
Output/ или необходимые output refs
validation_report.md
```

Полные рабочие папки issue не включаются по умолчанию. Вместо них export содержит `issue_сводка.md` со статусами, source refs и ссылками/идентификаторами canonical files. Полные issue folders/artifacts допустимы только если:

```text
context_loss_risk = true
inclusion_reason = конкретное evidence-based обоснование
```

Каждое исключение отражается в included/excluded tables manifest. Export не должен превращаться в копию repository tree.

## Page map

`page_map.md` содержит:

| Поле | Описание |
|---|---|
| Path | path внутри export archive |
| Source path | исходный path в concept scope |
| Parent | родительская страница |
| Backlinks | входящие links или local registry refs |
| Included | да/нет |
| Identity evidence | фактический content marker/readback basis |
| Reason if excluded | почему файл не включён |

Page map нужен даже при наличии local page registry, потому что export может быть compact и не включать все runtime artifacts.

## Issue summary

`issue_сводка.md` содержит по каждому issue:

- issue ID, title, status и phase;
- reason summary;
- dependency refs и normalized readiness;
- requirements/contract/output status;
- validation status;
- included artifacts;
- open blockers;
- archive/tombstone refs, если применимо.

Для WIP export отдельные разделы `Открытые issue`, `Blocking dependencies` и `Known limitations` обязательны.

## Exclusion policy

Нельзя включать в export:

- материалы разработки `Concept Builder`;
- checkpoint methodology и исходное ТЗ;
- raw служебные заметки;
- временные файлы без роли в concept understanding;
- секреты, приватные данные и тяжёлые attachments без явного решения;
- полные issue folders без доказанного context-loss risk;
- export packages другого revision как вложенные archives.

Исключение файла отражается в `manifest.md`; иначе export теряет проверяемость и не считается complete.

## Package-root local-open verification

Проверка выполняется после assembly, но до финализации manifest/state/persistence. Допускается распаковка во временную untracked directory вне production tree либо эквивалентная in-memory simulation.

Обязательные проверки:

1. package root содержит declared root `README.md` с точным регистром имени;
2. каждый included file из manifest/page map существует с exact case;
3. relative Markdown file links вне code fences разрешаются внутри package root;
4. external URLs, `mailto:` и pure anchors исключаются из file-target resolution;
5. нормализованный relative path не выходит из package root;
6. excluded source file не упоминается как included package target;
7. checked files и каждый exact broken target записаны в validation report;
8. manifest, page map и фактический included set совпадают.

Минимальный результат:

```yaml
package_local_open_status: pass|fail
entrypoint: README.md
entrypoint_opened: true|false
checked_files: []
broken_targets: []
case_mismatches: []
root_escape_targets: []
manifest_content_mismatches: []
```

Failure policy:

- любой fail блокирует `closed_concept`;
- WIP может продолжиться как `draft_with_notice` только для exact broken targets/case/content mismatch с user acceptance;
- missing/unopenable root README и любой root escape блокируют создание package любого типа.

## Export state update contract

Только после успешной persistence package и readback concept state обновляется минимум полями:

```text
Export status
Last export ID
Last export type
Last export report
Last exported at
Last export package
Open issues snapshot
Last validation ref
Next status
```

Правила:

- `Export status` различает `closed_export_persisted`, `wip_export_persisted` и pending/blocked states;
- state не ссылается на candidate path до фактического readback;
- root execution index меняется только когда concept status/export status действительно изменился;
- local/root registries перечисляют только реальные persisted files;
- повторный idempotent readback не создаёт ложный новый export event.

## Synthetic smoke strategy

Smoke для этого протокола является только non-production validation strategy. Запрещено создавать production `Concepts/smoke`, tracked fixture, runtime concept или commit-ready temporary directory.

Допустимые execution surfaces:

- in-memory map файлов;
- временная untracked directory вне production tree;
- локальная simulation package root, удаляемая после проверки.

Минимальные сценарии:

| Сценарий | Вход | Ожидаемый результат |
|---|---|---|
| Closed happy path | verified integrity, ready dependencies, canonical validation, valid links | precheck `allowed_result = closed_concept`; local-open `pass` |
| Blocking issue/dependency | open blocker или normalized readiness не `ready` | closed blocked; WIP route описан как non-final с exact limitations |
| Broken relative package link | included Markdown с отсутствующим target | local-open `fail`; closed blocked; WIP только по `draft_with_notice`, если нет entrypoint/root escape нарушения |
| Naming/collision/idempotency | одинаковый revision/timestamp candidate и существующий path | different content blocked; exact identity permits idempotent retry |

Evidence сохраняется в validation/persistence/PR notes. Файлы smoke-сценария не добавляются в repository registries.

## Transaction order

1. Перечитать selected concept README/state, local page registry, local issue registry и dependency graph.
2. Выполнить read-only `export_precheck` и сохранить/показать его результат без mutation.
3. Определить requested export type; не понижать closed до WIP молча.
4. Зафиксировать factual state revision, UTC timestamp, deterministic candidate ID/path.
5. Проверить source page/backlink/orphan/link состояние.
6. Проверить issue/output/dependency readiness; только normalized `ready` допускает closed route.
7. Проверить production/development boundary и compact issue-summary policy.
8. Собрать `page_map.md`, `issue_сводка.md`, `validation_report.md`, package contents и archive candidate во временной области.
9. Выполнить package-root local-open verification.
10. Повторно сверить blocker/limitation matrix и user acceptance для `draft_with_notice`.
11. Проверить collision/idempotency по фактическому export path.
12. Persist package и `manifest.md`; выполнить readback.
13. Только после readback обновить concept state export fields.
14. Обновить local page registry, root [execution_index.md](../../State/execution_index.md) и root [page registry](../../State/page_registry.jsonl) только при фактическом status/file/link delta.
15. Добавить factual persistence entry без predicted self-identity.
16. Ответить пользователю после readback либо честно вернуть blocker.

## Response packet

| Поле | Значение |
|---|---|
| Operation | `export_precheck`, `closed_concept` или `work_in_progress` |
| Allowed result | `closed_concept`, `work_in_progress` или `blocked` |
| Path | persisted manifest path либо `none` для precheck/blocker |
| Archive | persisted archive path либо `none` |
| Validation | `pass`, `pass_with_deferred_items`, `blocked` или `not_available` |
| Local-open | `pass`, `fail` или `not_run` |
| Open blockers | точный список или `none` |
| Limitations | точный список или `none` |
| Next step | принять export, принять WIP limitations, закрыть blockers или повторить precheck |

Для `work_in_progress` ответ обязан явно сказать: концепция не закрыта, package не финальный.

## Failure modes

| Failure | Действие |
|---|---|
| Нет selected concept | вернуть `blocked_no_concept`; не создавать package |
| Concept README/state отсутствует | вернуть `blocked_missing_concept_entrypoint_or_state`; route через [execution index](README.md) |
| Integrity не verified или revision конфликтует | вернуть integrity/revision blocker; mutation запрещена |
| Registry не парсится | остановить package creation до repair |
| Есть orphan/broken source links | closed blocked; WIP только по matrix и acceptance |
| Blocking issue/dependency открыт | closed blocked; WIP non-final route может быть предложен |
| Validation отсутствует/blocked | closed blocked; WIP route только по matrix |
| Package root README отсутствует или path выходит из root | блокировать оба export types |
| Local-open fail | closed blocked; WIP только `draft_with_notice`, кроме hard local-open blockers |
| Export path collision | `blocked_export_id_collision`, кроме доказанного exact idempotent retry |
| Persistence недоступен | вернуть `blocked_on_persistence`; draft не считать сохранённым export |

## Completion signal

Протокол выполнен, если произошло одно из событий:

- read-only `export_precheck` вернул полную schema, blockers/limitations, allowed result и next step без mutation;
- persisted `closed_concept` package прошёл readback, local-open, state/registry/persistence updates;
- persisted `work_in_progress` package прошёл допустимые gates, содержит exact limitations и явно помечен non-final;
- честно зафиксирован blocker до package creation или persistence.
