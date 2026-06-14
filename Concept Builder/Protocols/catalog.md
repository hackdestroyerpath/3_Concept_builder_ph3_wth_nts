# Каталог протоколов

Parent: [README](../README.md)  
Owner issue: `EXEC-003`  
Источник истины: `Protocols/catalog.md`  
Machine companion: [catalog.jsonl](catalog.jsonl)  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот каталог является источником истины для выбора протокола. Агент не выбирает протокол по памяти, по названию папки или по предположению. Сначала определяется `mode`, `state`, `phase`, active `issue` и trigger, затем загружается самый локальный подходящий протокол.

## Правило выбора

1. Прочитать root [README](../README.md) и релевантный state:
   - для `Service Mode`: [State/service_state.md](../State/service_state.md);
   - для `Execution Mode`: [State/execution_index.md](../State/execution_index.md).
2. Проверить [State/page_registry.jsonl](../State/page_registry.jsonl), чтобы не создать Markdown-сироту.
3. Если действие касается issue, проверить соответствующий registry и dependency graph: root [Issues/issue_registry.jsonl](../Issues/issue_registry.jsonl) для service scope или local concept registry для execution scope.
4. Найти trigger в таблице выбора ниже.
5. Если протокол имеет статус `available`, открыть только его файл и прямые зависимости.
6. Если статус `deferred` или `planned`, не имитировать выполнение. Создать/обновить issue/backlog и перейти к допустимому implementation step.
7. Если подходят несколько протоколов, выбрать самый локальный. Подъём на уровень выше разрешён только через context lift и с reason.

## Доступные протоколы

| Protocol ID | Файл | Mode | Когда использовать | Completion signal |
|---|---|---|---|---|
| `common/context_loading` | [common/context_loading_protocol.md](common/context_loading_protocol.md) | `common` | старт чата, восстановление scope, focus packet | выбран минимальный context packet |
| `common/persistence_transaction` | [common/persistence_protocol.md](common/persistence_protocol.md) | `common` | любая production-запись или package draft | commit marker или честный pending/blocked status |
| `common/final_validation` | [common/final_validation_protocol.md](common/final_validation_protocol.md) | `common` | закрытие issue, pre-commit/export validation, contract coverage | validation report сохранён |
| `service/service_start` | [service_protocols/service_start_protocol.md](service_protocols/service_start_protocol.md) | `service` | старт Service Mode и первичная навигация | выбран next protocol или blocker |
| `service/new_issue` | [service_protocols/new_issue_protocol.md](service_protocols/new_issue_protocol.md) | `service` | создание service-level issue из input/attachments | input и registry artifacts сохранены или blocker |
| `service/existing_issue` | [service_protocols/existing_issue_protocol.md](service_protocols/existing_issue_protocol.md) | `service` | выбор или продолжение существующего issue | focus packet и blockers известны |
| `service/question_answer` | [service_protocols/question_answer_protocol.md](service_protocols/question_answer_protocol.md) | `service/execution` | закрытие materially important unknowns | QA trace или skip/blocker marker |
| `service/requirements` | [service_protocols/requirements_protocol.md](service_protocols/requirements_protocol.md) | `service/execution` | создание, изменение или approval requirements | requirements review/approved или blocker |
| `service/solution_contract_output` | [service_protocols/solution_contract_output_protocol.md](service_protocols/solution_contract_output_protocol.md) | `service/execution` | solution/contract/output workflow | solution/contract или output package сохранён |
| `service/complex_issue` | [service_protocols/complex_issue_protocol.md](service_protocols/complex_issue_protocol.md) | `service/execution` | decomposition или requalification в complex | child issue entries или blocker |
| `service/linked_issues` | [service_protocols/linked_issues_protocol.md](service_protocols/linked_issues_protocol.md) | `service/execution` | dependency edges, stale/cycle handling | edge сохранён или отклонён с reason |
| `service/issue_retention` | [service_protocols/issue_retention_protocol.md](service_protocols/issue_retention_protocol.md) | `service/execution` | archive/tombstone/deletion/Inbox cleanup | retention decision сохранён |
| `execution/index` | [execution_protocols/README.md](execution_protocols/README.md) | `execution` | старт Execution Mode, выбор/создание concept scope | active concept выбран или blocker |
| `execution/concept_export` | [execution_protocols/concept_export_protocol.md](execution_protocols/concept_export_protocol.md) | `execution` | closed/WIP export концепции | export package или blocker |

## Selection matrix

| Trigger / состояние | Mode | Protocol ID | Статус | Обязательные входы | Выходы | Следующий state / phase |
|---|---|---|---|---|---|---|
| Старт service-чата: `пинг`, `старт`, `1` без активного меню | `service` | `service/service_start` | `available` | `README.md`, `State/service_state.md`, catalog, issue registry | loaded-status, primary navigation | выбрать new/existing issue branch |
| Старт execution-чата или выбор концепции | `execution` | `execution/index` | `available` | `README.md`, execution index, Concepts entry/template | focus packet, selected concept или blocker | selected concept или concept creation review |
| Восстановление контекста или focus packet | `service/execution` | `common/context_loading` | `available` | entry point, relevant State, catalog | loaded-status, focus packet | active scope определён |
| Любая production-запись | `common` | `common/persistence_transaction` | `available` | latest state, target files, write set | affected files, registry/state updates, persistence marker | committed или pending/blocked |
| Создание нового service-level `issue` | `service` | `service/new_issue` | `available` | user input, attachments, Inbox rules | Inbox input, registry entry, issue artifacts | proposed issue или blocker |
| Пользователь выбирает existing issue | `service` | `service/existing_issue` | `available` | issue registry, dependency graph, optional issue artifacts | shortlist или focus packet | active/focused issue или blocker |
| Недостаточно контекста для requirements | `service/execution` | `service/question_answer` | `available` | issue, reason, input, requirements draft | QA trace или skip/blocker marker | requirements phase или blocker |
| Нужно подготовить requirements | `service/execution` | `service/requirements` | `available` | issue, reason, input, QA, assertions | requirements.md, approval log | requirements review/approved или blocker |
| Issue готов к requalification | `service/execution` | `service/complex_issue` | `available` | approved requirements, issue state | simple/complex decision | solution или decomposition review |
| Issue требует child work packages | `service/execution` | `service/complex_issue` | `available` | parent issue, requirements, dependency graph | decomposition plan, child registry entries | first child focus или blocker |
| Issue готов к solution/contract | `service/execution` | `service/solution_contract_output` | `available` | approved requirements, dependency status | solution.md, contract.md, review packet | review или execution-ready state |
| Solution и contract утверждены | `service/execution` | `service/solution_contract_output` | `available` | approved solution/contract, affected files | output/report.md, changed_files.md, contract_coverage.md | validation phase или blocker |
| Есть blocking/artifact dependency | `service/execution` | `service/linked_issues` | `available` | registry, dependency graph, linked states | edge readiness/stale/cycle markers | dependencies clean/blocked/stale/cycle_blocked |
| Issue закрыт, отклонён, архивируется или требует Inbox cleanup | `service/execution` | `service/issue_retention` | `available` | registry, graph, archive/tombstone entry points, Inbox manifest | archive/tombstone/retention decision | retained/archived/tombstone/delete allowed/blocker |
| Концепция готовится к export | `execution` | `execution/concept_export` | `available` | concept state, pages, issues, validation, export request | export manifest, page map, issue summary, archive | closed_concept/work_in_progress/blocker |
| Issue готов к закрытию или пакет готовится к commit | `service/execution` | `common/final_validation` | `available` | state, requirements/contract/output if any, page registry, issue registry | validation report, issue coverage, contract coverage | pass/pass_with_deferred_items/blocked |
| Нужно оценить служебный скрипт | `service` | `service/script_evaluation` | `deferred` | structural backlog, candidate use cases | decision record or rejection note | new approved issue или оставить deferred |

## Связанные реестры issue и state

| Ресурс | Файл | Когда читать |
|---|---|---|
| Человекочитаемый реестр issue | [Issues/issue_registry.md](../Issues/issue_registry.md) | выбор или изменение service issue, проверка lifecycle/status |
| Машинный registry | [Issues/issue_registry.jsonl](../Issues/issue_registry.jsonl) | фильтрация, dependency refs, next action |
| Dependency graph | [Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl) | перед переводом issue в `active`, проверка blockers/cycles |
| Concepts entry | [Concepts/README.md](../Concepts/README.md) | старт или создание execution scope |
| Concept template | [Concepts/_template/README.md](../Concepts/_template/README.md) | создание новой концепции |
| Inbox rules | [Inbox/README.md](../Inbox/README.md) | перед сохранением input/attachments, созданием issue или cleanup |
| Issue archive | [Issues/_archive/README.md](../Issues/_archive/README.md) | перед archive decision и проверкой закрытых issue |
| Issue tombstones | [Issues/_tombstones/README.md](../Issues/_tombstones/README.md) | перед tombstone cleanup и удалением неисторических файлов |
| Финальный отчёт | [State/service_validation_report.md](../State/service_validation_report.md) | перед commit/export или закрытием service-layer изменений |

## Deferred protocol candidates

Эти candidates не являются broken links и не входят в обязательный production manifest. Они требуют отдельного approved issue перед созданием:

| Protocol ID | Reason | Source issue |
|---|---|---|
| `service/script_evaluation` | служебные скрипты требуют отдельной оценки пользы, стоимости поддержки и GitHub policy | `USER-001` |
| `common/issue_decision_update` | может быть выделен позже, если decision-flow станет слишком тяжёлым внутри issue protocols | future service issue |
| `common/response_packet` | может быть выделен позже, если формат ответов начнёт дрейфовать | future service issue |

## Связь с project instructions

Project instructions не содержат подробную routing matrix. Они направляют агента сюда:

- [Service Mode instructions](../Instructions/service_mode_project_instructions.md)
- [Execution Mode instructions](../Instructions/execution_mode_project_instructions.md)

## Обновление каталога

При создании нового протокола агент обязан:

1. создать Markdown-файл протокола с parent link на этот каталог;
2. добавить запись в [catalog.jsonl](catalog.jsonl);
3. заменить planned path в этой странице на clickable link;
4. обновить [State/page_registry.jsonl](../State/page_registry.jsonl);
5. добавить persistence entry в [State/persistence_log.jsonl](../State/persistence_log.jsonl);
6. перед закрытием выполнить [final_validation_protocol.md](common/final_validation_protocol.md).
