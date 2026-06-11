# Протокол экспорта концепции

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-011`  
Protocol ID: `execution/concept_export`  
Источник истины: `Protocols/execution_protocols/concept_export_protocol.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот протокол управляет export package для конкретной концепции. Он создаёт проверяемую выгрузку concept-сети с сохранением traceability решений, issue и validation evidence.

Экспорт бывает двух типов:

| Export type | Когда допустим | Как называть |
|---|---|---|
| `closed_concept` | concept validation pass, blockers отсутствуют, пользователь утвердил closure | закрытый export концепции |
| `work_in_progress` | остаются open issue, blockers или непокрытые области, но нужен handoff | WIP export; не финальная концепция |

## Когда использовать

- пользователь просит выгрузить концепцию;
- concept state перешёл в `ready_for_closure_review` или `closed`;
- нужен пакет передачи другому агенту;
- нужно зафиксировать промежуточный snapshot с открытыми issue;
- нужно проверить, можно ли вообще называть концепцию закрытой.

Этот протокол не заменяет issue output из service/execution lifecycle. Для результата отдельного issue используется solution/contract/output workflow, а export concept-level собирает сеть целиком.

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
| Output refs | `Concepts/<concept_slug>/Output/` и issue output refs, если есть |
| Protocol route | [README execution-протоколов](README.md) и [catalog.md](../catalog.md) |
| Persistence rules | [persistence_protocol.md](../common/persistence_protocol.md) |

Если конкретная концепция ещё не создана, export невозможен. Агент должен вернуть `blocked_no_concept` и не создавать пустой archive package.

## Preconditions для `closed_concept`

1. `Concepts/<concept_slug>/README.md` существует и является точкой входа.
2. Concept state имеет status `ready_for_closure_review` или `closed`.
3. Local page registry не содержит orphan Markdown.
4. Все required pages связаны parent/backlink.
5. Все blocking concept issue имеют terminal status: `closed`, `rejected`, `deferred`, `archived`, `tombstone` или `deleted` с сохранённым tombstone.
6. Blocking dependency graph пуст для closure scope или все blocking edges имеют `ready` / `satisfied`.
7. Contract/output coverage сохранён для required issue.
8. Concept-level validation result: `pass` или `pass_with_notes` с user acceptance.
9. Пользователь утвердил closure, если export меняет concept status на `closed`.

Если хотя бы один пункт не выполнен, closed export запрещён. Возможен только `work_in_progress`, если пользователь принимает ограничения.

## Preconditions для `work_in_progress`

1. Concept README и concept state существуют.
2. Open issue и blockers перечислены.
3. Export manifest явно содержит `export_type = work_in_progress`.
4. В ответе пользователю сказано, что концепция не закрыта.
5. Known limitations перечислены конкретно, без обобщённых формулировок.

## Export package layout

Источник истины export создаётся внутри concept scope:

```text
Concepts/<concept_slug>/Exports/<export_id>/
├── manifest.md
├── concept_archive.zip
├── page_map.md
├── issue_сводка.md
└── validation_report.md
```

`export_id` формируется стабильно: `export_<YYYYMMDD>_<HHMMSS>_<type>`, где type — `closed` или `wip`.

## `manifest.md` schema

Минимальные разделы manifest:

| Раздел | Содержание |
|---|---|
| Metadata | export ID, timestamp, concept slug, source commit marker или package marker |
| Export type | `closed_concept` или `work_in_progress` |
| Entry point | путь к `README.md` внутри архива |
| Included files | список включённых Markdown/state/output refs |
| Excluded files | development-only, raw, heavy или private files с reason |
| Issue summary | closed / rejected / deferred / archived / tombstone / open |
| Dependency status | blocking, stale, resolved и known cycles |
| Validation result | pass, pass_with_notes, fail или not_available_with_reason |
| Known limitations | обязательно для WIP export |
| Reuse notes | что следующий агент должен открыть первым |

## `concept_archive.zip` минимум

```text
README.md
Pages/
State/page_registry.jsonl или page_map.md
Issues/issue_сводка.md
Output/ или ссылки на output, если они нужны для понимания концепции
validation_report.md
```

Полные рабочие папки issue не включаются по умолчанию. Вместо этого export содержит `issue_сводка.md` со статусами, source refs и ссылками на canonical files в GitHub/source package. Полные issue folders включаются только если без них будущий агент потеряет проверяемый контекст.

## Page map

`page_map.md` содержит:

| Поле | Описание |
|---|---|
| Path | путь внутри export archive |
| Source path | исходный путь в concept scope |
| Parent | родительская страница |
| Backlinks | входящие ссылки или local registry refs |
| Included | да/нет |
| Reason if excluded | почему файл не включён |

Page map нужен даже при наличии `State/page_registry.jsonl`, потому что export может быть compact и не включать все runtime artifacts.

## Issue summary

`issue_сводка.md` содержит по каждому issue:

- issue ID, title, status, phase;
- reason summary;
- dependency refs;
- requirements/contract/output status;
- validation status;
- included artifacts;
- open blockers;
- archive/tombstone refs, если применимо.

Для `work_in_progress` export отдельный раздел `Открытые issue` обязателен.

## Exclusion policy

Нельзя включать в export:

- материалы разработки `Concept Builder`;
- checkpoint methodology и исходное ТЗ;
- raw служебные заметки;
- временные файлы без роли в concept understanding;
- секреты, приватные данные и тяжёлые attachments без явного решения;
- полные issue folders, если достаточно проверяемой сводки.

Исключение файла должно быть отражено в `manifest.md`; иначе export теряет проверяемость и не считается complete.

## Transaction order

1. Перечитать concept README, concept state, local page registry, local issue registry и dependency graph.
2. Определить export type: `closed_concept` или `work_in_progress`.
3. Проверить page/backlink/orphan состояние.
4. Проверить issue/dependency readiness.
5. Проверить production/development boundary для export contents.
6. Собрать `page_map.md`, `issue_сводка.md` и `validation_report.md`.
7. Создать `concept_archive.zip`.
8. Создать `manifest.md` как primary source of truth export.
9. Обновить concept state, local page registry, root [execution_index.md](../../State/execution_index.md) и root [page_registry.jsonl](../../State/page_registry.jsonl), если создаётся export entrypoint или меняется concept status.
10. Добавить запись в persistence log.
11. Ответить пользователю только после сохранения package или честно вернуть blocker.

## Response packet

Короткий ответ после export содержит:

| Поле | Значение |
|---|---|
| Export | `closed_concept` или `work_in_progress` |
| Path | путь к `Concepts/<concept_slug>/Exports/<export_id>/manifest.md` |
| Archive | путь к `concept_archive.zip` |
| Validation | pass / pass_with_notes / fail / not_available |
| Open blockers | список или `none` |
| Next step | принять export, закрыть blockers, продолжить issue или archive concept |

Для `work_in_progress` ответ обязан явно сказать: концепция не закрыта.

## Failure modes

| Failure | Действие |
|---|---|
| Нет active concept | вернуть `blocked_no_concept` |
| Concept README отсутствует | остановиться и предложить bootstrap через [execution index](README.md) |
| Есть orphan pages | не экспортировать closed concept; вернуть repair list |
| Blocking issue открыт | разрешить только WIP export при явном согласии пользователя |
| Dependency graph stale/cycle | остановить closed export и вернуть blocker |
| Нет validation | создать WIP export или дождаться final validation, если нужен closed export |
| Persistence недоступен | вернуть package draft / pending, не заявлять commit |

## Completion signal

Протокол выполнен, если сохранены `manifest.md`, `page_map.md`, `issue_сводка.md`, `validation_report.md`, `concept_archive.zip`, обновлены state/registry/log или честно зафиксирован blocker с next step.
