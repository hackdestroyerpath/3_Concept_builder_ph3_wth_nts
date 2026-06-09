# Execution index

Parent: [Concept Builder README](../README.md)  
Owner issue: `EXEC-004` / `EXEC-011`  
Источник истины: `State/execution_index.md`  
Status: `draft-ready`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот файл является root index для `Concept Builder / Execution Mode`. Он показывает, есть ли активная концепция, где искать template, какой protocol использовать для export и когда нельзя продолжать без service route.

## Текущее состояние

| Поле | Значение |
|---|---|
| Active concept | `none` |
| Runtime concept folders | `none_created` |
| Concepts entry | [Concepts/README.md](../Concepts/README.md) |
| Concept template | [Concepts/_template/README.md](../Concepts/_template/README.md) |
| Execution protocols | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| Concept export protocol | [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md) |
| Blocking status | `none_until_concept_creation_request` |
| Next status | `ready_for_concept_selection_or_creation_request` |

## Минимальный старт Execution Mode

1. Открыть [README.md](../README.md).
2. Открыть этот файл.
3. Открыть [Concepts/README.md](../Concepts/README.md).
4. Открыть [execution protocols](../Protocols/execution_protocols/README.md).
5. Если нужна новая концепция, использовать [Concepts/_template/README.md](../Concepts/_template/README.md).
6. Если нужен export, использовать [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md).
7. Перед записью проверить [page_registry.jsonl](page_registry.jsonl) и [persistence_protocol.md](../Protocols/common/persistence_protocol.md).

## Creation gate

Новая концепция создаётся только если известны:

| Поле | Требование |
|---|---|
| `concept_slug` | уникальный slug по правилам [Concepts/README.md](../Concepts/README.md) |
| `goal` | практическая цель concept scope |
| `reason_to_persist` | почему нужен файловый concept, а не обычный ответ |
| `source_input` | user input, issue или документ-источник |
| `write_scope` | какие файлы будут созданы |
| `next_action` | минимальный следующий шаг |

Если данных не хватает, Execution Mode возвращает blocker. Это фиксирует границу между обсуждением и созданием файлов.

## Concept registry

Пока реальные концепции не созданы.

| Concept slug | Status | Goal | Path | Last update |
|---|---|---|---|---|
| `none` | `not_created` | нет активной концепции | `n/a` | `2026-06-05T11:45:45Z` |

После создания первой концепции эта таблица обновляется только для discoverable concepts. Локальные черновики, не предназначенные для root navigation, фиксируются в local concept state.

## Execution boundary

| Запрос | Где выполнять |
|---|---|
| развить конкретную концепцию | `Concepts/<concept_slug>/` |
| создать новую концепцию | `Concepts/` + template |
| экспортировать концепцию | `Protocols/execution_protocols/concept_export_protocol.md` |
| изменить service protocol или root State | `Service Mode` |
| изменить issue lifecycle | `Service Mode` |
| изменить project instructions | `Service Mode` |

## Export readiness

Export route готов на уровне protocol layer. Реальный export возможен только после появления `Concepts/<concept_slug>/` и local state. Незакрытая концепция экспортируется как `work_in_progress`, а не как якобы завершённый результат.

## Next-step marker

Next status: `ready_for_concept_selection_or_creation_request`.

Если работа продолжается как service implementation, следующий service шаг: собрать final package или выполнить ручной GitHub commit после проверки.
