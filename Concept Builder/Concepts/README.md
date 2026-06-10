# Слой концепций

Parent: [Concept Builder README](../README.md)  
Owner issue: `EXEC-011`  
Источник истины: `Concepts/README.md`  
Status: `available-empty-layer`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

`Concepts/` хранит конкретные концепции, которые развиваются в `Concept Builder / Execution Mode`. Этот файл является entry point слоя концепций. Он не является концепцией сам по себе и не используется для несогласованных идей вне утверждённого scope.

## Текущий статус

| Поле | Значение |
|---|---|
| Concepts created | `0` |
| Active concept | `none` |
| Template | [Concepts/_template/README.md](_template/README.md) |
| Execution protocols | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| Export protocol | [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md) |
| Root index | [State/execution_index.md](../State/execution_index.md) |

Конкретные концепции пока не созданы. Папка `Concepts/<concept_slug>/` создаётся только после явного запроса пользователя или approved execution issue.

## Реестр концепций

| Concept slug | Название | Status | Active issue | State path |
|---|---|---|---|---|
| `none` | Концепции ещё не созданы | `empty` | `none` | `none` |

Root source of truth для списка концепций: [State/execution_index.md](../State/execution_index.md). Этот README показывает только человекочитаемый вход слоя.

## Создание концепции

Перед созданием новой концепции агент обязан:

1. открыть [execution protocols](../Protocols/execution_protocols/README.md);
2. проверить [State/execution_index.md](../State/execution_index.md);
3. проверить [State/page_registry.jsonl](../State/page_registry.jsonl);
4. использовать [шаблон](_template/README.md) как спецификацию, но не как готовую концепцию;
5. получить `concept_slug`, title, reason, initial scope и boundary;
6. перечислить write set;
7. сохранить файлы, registry и persistence log в одной transaction.

Примечание: ссылка на шаблон выше должна вести в `_template/README.md`; Markdown target должен оставаться точным и проверяемым. Canonical ссылка: [Concepts/_template/README.md](_template/README.md).

## Минимальная структура concrete concept

```text
Concepts/<concept_slug>/
├── README.md
├── State/
│   ├── concept_state.md
│   └── page_registry.jsonl
├── Issues/
│   ├── issue_registry.jsonl
│   ├── dependency_graph.jsonl
│   ├── _archive/README.md
│   └── _tombstones/README.md
├── Pages/
├── Output/
└── Exports/
```

Каждый concrete Markdown-файл концепции должен иметь parent link и входящую ссылку из concept README, local page registry, локального issue или export package.

## Scope boundaries

| Слой | Что хранит |
|---|---|
| Root `README.md` | вход в систему и ссылка на этот слой |
| [State/execution_index.md](../State/execution_index.md) | список концепций, active concept и top-level routing |
| `Concepts/<concept_slug>/README.md` | вход конкретной концепции |
| `Concepts/<concept_slug>/State/` | local state, local page registry и focus markers |
| `Concepts/<concept_slug>/Issues/` | локальные issue концепции |
| `Concepts/<concept_slug>/Pages/` | содержательные страницы концепции |
| `Concepts/<concept_slug>/Output/` | итоговые результаты concept-level работы |
| `Concepts/<concept_slug>/Exports/` | export packages |

`Execution Mode` не должен менять root service protocols ради одной концепции. Если концепция выявила дефект core-системы, создаётся service issue.

## Lifecycle статусы концепции

| Status | Значение |
|---|---|
| `draft` | concept folder создана, сеть ещё формируется |
| `active` | есть open issue или pages in progress |
| `ready_for_closure_review` | required work завершён, нужна validation |
| `closed` | closure утверждён пользователем |
| `exported` | export package создан |
| `archived` | концепция выведена из active work, история сохранена |

## Export

Экспорт выполняется только через [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md). Если у концепции есть open issue, допустим только `work_in_progress` export. Closed export требует validation, absence of blockers и user approval.

## Связанные файлы

- [Root README](../README.md)
- [Execution index](../State/execution_index.md)
- [Page registry](../State/page_registry.jsonl)
- [Execution protocols](../Protocols/execution_protocols/README.md)
- [Concept export protocol](../Protocols/execution_protocols/concept_export_protocol.md)
- [Concept template](_template/README.md)

Canonical template link: [Concepts/_template/README.md](_template/README.md).
