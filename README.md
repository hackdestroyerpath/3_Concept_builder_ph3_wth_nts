# Concept Builder

Статус: `pass_with_deferred_items`  
Источник истины этого входа: `README.md`  
Последнее обновление: `2026-06-16T16:48:11Z`  
Реестр страниц: [State/page_registry.jsonl](State/page_registry.jsonl)  
Финальная проверка: [State/service_validation_report.md](State/service_validation_report.md)

## Назначение

`Concept Builder` — рабочая сеть файлов для совместного построения концепций через ChatGPT и GitHub. Система разделяет два режима:

1. `Concept Builder Service Mode` — обслуживание самой структуры: `State`, project instructions, протоколы, issue-модель, навигация, retention и проверки.
2. `Concept Builder / Execution Mode` — развитие конкретных концепций внутри [Concepts/](Concepts/README.md).

Production root: корень репозитория `/`; бывшая wrapper-папка `Concept Builder/` вынесена в корень и не является production boundary.

Этот файл является главным входом в репозиторий. Он не дублирует всю систему: он ведёт агента к ближайшему источнику истины и помогает выбрать минимальный рабочий контекст.

## Старт агента

1. Открой этот `README.md` как корневую карту.
2. Определи режим работы:
   - для обслуживания системы открой [State/service_state.md](State/service_state.md) и [Protocols/service_protocols/service_start_protocol.md](Protocols/service_protocols/service_start_protocol.md);
   - для выбора или создания концепции открой [State/execution_index.md](State/execution_index.md), [Concepts/README.md](Concepts/README.md) и [Protocols/execution_protocols/README.md](Protocols/execution_protocols/README.md).
3. Проверь страницу и владельца через [State/page_registry.jsonl](State/page_registry.jsonl).
4. Открой [Protocols/catalog.md](Protocols/catalog.md) и выбери самый локальный протокол.
5. Перед любой записью примени [Protocols/common/persistence_protocol.md](Protocols/common/persistence_protocol.md) и проверь [State/persistence_log.jsonl](State/persistence_log.jsonl).
6. Если пользователь создаёт новый service-level `issue`, используй [Protocols/service_protocols/new_issue_protocol.md](Protocols/service_protocols/new_issue_protocol.md) и [Inbox/README.md](Inbox/README.md).
7. Если пользователь выбирает существующий service-level `issue`, используй [Protocols/service_protocols/existing_issue_protocol.md](Protocols/service_protocols/existing_issue_protocol.md).
8. Если phase требует QA, используй [Protocols/service_protocols/question_answer_protocol.md](Protocols/service_protocols/question_answer_protocol.md).
9. Если phase требует требований, используй [Protocols/service_protocols/requirements_protocol.md](Protocols/service_protocols/requirements_protocol.md).
10. Если phase требует solution, contract или output, используй [Protocols/service_protocols/solution_contract_output_protocol.md](Protocols/service_protocols/solution_contract_output_protocol.md).
11. Если issue требует decomposition, используй [Protocols/service_protocols/complex_issue_protocol.md](Protocols/service_protocols/complex_issue_protocol.md).
12. Если нужно создать, проверить или починить dependency edge между issue, используй [Protocols/service_protocols/linked_issues_protocol.md](Protocols/service_protocols/linked_issues_protocol.md).
13. Если нужен archive, tombstone, deletion или cleanup issue/Inbox, используй [Protocols/service_protocols/issue_retention_protocol.md](Protocols/service_protocols/issue_retention_protocol.md).
14. Если нужно создать concept scope или выбрать execution routing, используй [Protocols/execution_protocols/README.md](Protocols/execution_protocols/README.md) и [Concepts/_template/README.md](Concepts/_template/README.md).
15. Если нужен export концепции, используй [Protocols/execution_protocols/concept_export_protocol.md](Protocols/execution_protocols/concept_export_protocol.md).
16. Если нужно закрыть issue, проверить пакет перед commit/export или провести repair-check, используй [Protocols/common/final_validation_protocol.md](Protocols/common/final_validation_protocol.md).
17. Если появляется структурное решение или незакрытый вопрос, проверь [State/structural_backlog.jsonl](State/structural_backlog.jsonl).
18. Загружай только текущий режим, текущий scope, активный `issue` и ближайшие зависимости.

## Текущая entry map

| Область | Путь | Роль | Источник истины |
|---|---|---|---|
| Корневой вход | `README.md` | навигация верхнего уровня | этот файл |
| Service state | [State/service_state.md](State/service_state.md) | состояние обслуживания системы | `State/service_state.md` |
| Execution index | [State/execution_index.md](State/execution_index.md) | индекс концепций и execution-объектов | `State/execution_index.md` |
| Реестр страниц | [State/page_registry.jsonl](State/page_registry.jsonl) | машинная карта страниц, родителей и backlinks | `State/page_registry.jsonl` |
| Журнал сохранения | [State/persistence_log.jsonl](State/persistence_log.jsonl) | записи transaction-like сохранения | `State/persistence_log.jsonl` |
| Structural backlog | [State/structural_backlog.jsonl](State/structural_backlog.jsonl) | структурные решения, отложенные вопросы и guards | `State/structural_backlog.jsonl` |
| Финальный отчёт | [State/service_validation_report.md](State/service_validation_report.md) | проверка service layer, contract coverage и commit readiness | `State/service_validation_report.md` |
| Service project instructions | [Instructions/service_mode_project_instructions.md](Instructions/service_mode_project_instructions.md) | исходник project instructions для `Service Mode` | `Instructions/` |
| Execution project instructions | [Instructions/execution_mode_project_instructions.md](Instructions/execution_mode_project_instructions.md) | исходник project instructions для `Execution Mode` | `Instructions/` |
| Protocol catalog | [Protocols/catalog.md](Protocols/catalog.md) | выбор протокола по mode/state/phase/issue | `Protocols/catalog.md` |
| Protocol registry | [Protocols/catalog.jsonl](Protocols/catalog.jsonl) | машиночитаемый companion каталога | `Protocols/catalog.jsonl` |
| Context loading | [Protocols/common/context_loading_protocol.md](Protocols/common/context_loading_protocol.md) | минимальная загрузка контекста и context lift | `Protocols/common/` |
| Persistence protocol | [Protocols/common/persistence_protocol.md](Protocols/common/persistence_protocol.md) | transaction-like сохранение и honest pending status | `Protocols/common/` |
| Final validation protocol | [Protocols/common/final_validation_protocol.md](Protocols/common/final_validation_protocol.md) | pre-commit/export validation и contract coverage | `Protocols/common/` |
| Service start protocol | [Protocols/service_protocols/service_start_protocol.md](Protocols/service_protocols/service_start_protocol.md) | старт `Service Mode` и первичная навигация | `Protocols/service_protocols/` |
| New issue protocol | [Protocols/service_protocols/new_issue_protocol.md](Protocols/service_protocols/new_issue_protocol.md) | создание service-level issue из input/attachments | `Protocols/service_protocols/` |
| Existing issue protocol | [Protocols/service_protocols/existing_issue_protocol.md](Protocols/service_protocols/existing_issue_protocol.md) | выбор существующего issue, focus packet и blockers | `Protocols/service_protocols/` |
| Question Answer protocol | [Protocols/service_protocols/question_answer_protocol.md](Protocols/service_protocols/question_answer_protocol.md) | QA trace, skip criteria и materially important unknowns | `Protocols/service_protocols/` |
| Requirements protocol | [Protocols/service_protocols/requirements_protocol.md](Protocols/service_protocols/requirements_protocol.md) | `requirements.md`, review и approval workflow | `Protocols/service_protocols/` |
| Solution / Contract / Output protocol | [Protocols/service_protocols/solution_contract_output_protocol.md](Protocols/service_protocols/solution_contract_output_protocol.md) | `solution.md`, `contract.md`, execution и `output/` | `Protocols/service_protocols/` |
| Complex Issue protocol | [Protocols/service_protocols/complex_issue_protocol.md](Protocols/service_protocols/complex_issue_protocol.md) | parent/children decomposition и decomposition budget | `Protocols/service_protocols/` |
| Linked Issues protocol | [Protocols/service_protocols/linked_issues_protocol.md](Protocols/service_protocols/linked_issues_protocol.md) | dependency edges, readiness, stale/cycle handling | `Protocols/service_protocols/` |
| Issue Retention protocol | [Protocols/service_protocols/issue_retention_protocol.md](Protocols/service_protocols/issue_retention_protocol.md) | archive, tombstone, deletion и Inbox cleanup lifecycle | `Protocols/service_protocols/` |
| Execution protocols | [Protocols/execution_protocols/README.md](Protocols/execution_protocols/README.md) | старт `Execution Mode`, concept scope и path mapping | `Protocols/execution_protocols/` |
| Concept export protocol | [Protocols/execution_protocols/concept_export_protocol.md](Protocols/execution_protocols/concept_export_protocol.md) | closed/WIP export package концепции | `Protocols/execution_protocols/` |
| Issue registry | [Issues/issue_registry.md](Issues/issue_registry.md) | человекочитаемый реестр issue, статусы и lifecycle | `Issues/issue_registry.md` |
| Issue registry JSONL | [Issues/issue_registry.jsonl](Issues/issue_registry.jsonl) | машиночитаемый реестр issue | `Issues/issue_registry.jsonl` |
| Dependency graph | [Issues/dependency_graph.jsonl](Issues/dependency_graph.jsonl) | граф blocking/non-blocking зависимостей issue | `Issues/dependency_graph.jsonl` |
| Issue archive | [Issues/_archive/README.md](Issues/_archive/README.md) | вход в архив закрытых/отклонённых issue | `Issues/_archive/` |
| Issue tombstones | [Issues/_tombstones/README.md](Issues/_tombstones/README.md) | вход в tombstone-сводки после cleanup | `Issues/_tombstones/` |
| Inbox rules | [Inbox/README.md](Inbox/README.md) | правила хранения input и attachments | `Inbox/README.md` |
| Concepts layer | [Concepts/README.md](Concepts/README.md) | вход в слой концепций | `Concepts/README.md` |
| Concept template | [Concepts/_template/README.md](Concepts/_template/README.md) | шаблон новой концепции | `Concepts/_template/README.md` |

## Production-области

| Область | Назначение | Статус наполнения |
|---|---|---|
| `State/` | состояние, реестры, журнал сохранения, backlog и финальный validation report | готово к runtime use |
| `Instructions/` | исходники project instructions для двух режимов | готово к runtime use |
| `Protocols/` | каталог и протоколы выбора действий | common, service, execution и final validation доступны |
| `Issues/` | service-level issue, dependency graph, archive и tombstones | bootstrap issue закрыты или явно deferred |
| `Inbox/` | входящие материалы пользователя и attachments | rules entrypoint создан; cleanup только через retention protocol |
| `Concepts/` | конкретные концепции, локальный state, pages, output и exports | layer entry и template созданы; concrete concepts ещё не созданы |

## Правила навигации

- Каждый Markdown-файл должен иметь запись в [State/page_registry.jsonl](State/page_registry.jsonl) или входящий путь из этой карты.
- Каждый дочерний Markdown-файл должен иметь ссылку на родительский уровень.
- Новый рабочий файл создаётся только если у него есть owner, назначение, источник истины и lifecycle.
- Материалы разработки, технические задания, checkpoint-архивы, исходные notes и проверочное сырьё не входят в production-слой.
- При создании, переименовании, архивировании или удалении страницы обновляются parent link, backlink, [State/page_registry.jsonl](State/page_registry.jsonl) и соответствующая проверка.

## Источники истины верхнего уровня

| Данные | Основной источник |
|---|---|
| Навигация репозитория | `README.md` и [State/page_registry.jsonl](State/page_registry.jsonl) |
| Состояние обслуживания | [State/service_state.md](State/service_state.md) |
| Индекс концепций | [State/execution_index.md](State/execution_index.md) |
| Project instructions | [Instructions/service_mode_project_instructions.md](Instructions/service_mode_project_instructions.md) и [Instructions/execution_mode_project_instructions.md](Instructions/execution_mode_project_instructions.md) |
| Протоколы | [Protocols/catalog.md](Protocols/catalog.md) и [Protocols/catalog.jsonl](Protocols/catalog.jsonl) |
| Issue lifecycle | [Issues/issue_registry.md](Issues/issue_registry.md) |
| Dependency graph | [Issues/dependency_graph.jsonl](Issues/dependency_graph.jsonl) |
| Concept layer | [Concepts/README.md](Concepts/README.md) |
| Concept template | [Concepts/_template/README.md](Concepts/_template/README.md) |
| Concept export | [Protocols/execution_protocols/concept_export_protocol.md](Protocols/execution_protocols/concept_export_protocol.md) |
| Финальная проверка | [Protocols/common/final_validation_protocol.md](Protocols/common/final_validation_protocol.md) и [State/service_validation_report.md](State/service_validation_report.md) |

## Текущий статус сборки

`Service Mode` и базовый `Execution Mode` находятся в состоянии `pass_with_deferred_items` после direct-main root-flatten repair. Production root: корень репозитория `/`; бывшая wrapper-папка `Concept Builder/` удалена из tracked production files. Фактическая GitHub-запись выполнена напрямую в `main` для `hackdestroyerpath/3_Concept_builder_ph3_wth_nts`; transaction `tx-cb-root-flatten-20260616`. Конкретные concept folders не созданы, потому что пользователь не дал `concept_slug` и initial scope. `USER-001` остаётся deferred/non-blocking и не создаёт service scripts без отдельного approved issue.
