# Execution-протоколы

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-011`  
Protocol ID: `execution/index`  
Источник истины: `Protocols/execution_protocols/README.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот файл является входом в протоколы `Concept Builder / Execution Mode`. Режим работает не с core-системой, а с конкретной концепцией внутри `Concepts/<concept_slug>/`. Он использует общий lifecycle issue, context loading, persistence и validation, но меняет корень scope на папку выбранной концепции.

## Граница режима

| Разрешено в `Execution Mode` | Запрещено без escalation в `Service Mode` |
|---|---|
| создавать и менять файлы внутри `Concepts/<concept_slug>/` | менять root `Instructions/` |
| вести локальные concept issue | менять root `Protocols/` |
| обновлять root [execution_index.md](../../State/execution_index.md) для выбора/статуса концепции | менять root [service_state.md](../../State/service_state.md) как побочный эффект concept work |
| обновлять root [page_registry.jsonl](../../State/page_registry.jsonl), если создаётся или архивируется concept entrypoint | менять service-level [issue_registry.jsonl](../../Issues/issue_registry.jsonl) без service issue |
| создавать export package через [concept_export_protocol.md](concept_export_protocol.md) | закрывать концепцию без closure review и validation |

Если concept-work показывает, что нужен новый core-протокол, изменение project instructions или ремонт service-level registry, агент создаёт service-level issue через [service protocols](../service_protocols/service_start_protocol.md) или просит пользователя переключиться в `Service Mode`.

## Старт `Execution Mode`

1. Открыть root [README](../../README.md).
2. Открыть [State/execution_index.md](../../State/execution_index.md) и проверить active concept.
3. Открыть [Concepts/README.md](../../Concepts/README.md).
4. Если active concept есть, открыть `Concepts/<concept_slug>/README.md`, локальный `State/concept_state.md` и локальный `State/page_registry.jsonl`.
5. Если active concept отсутствует, использовать [Concepts/_template/README.md](../../Concepts/_template/README.md) только как шаблон и не создавать concrete concept без явного пользовательского запроса или approved issue.
6. Открыть [catalog.md](../catalog.md) и выбрать самый локальный protocol.
7. Перед записью применить [persistence_protocol.md](../common/persistence_protocol.md).

Минимальный focus packet в этом режиме: concept README, concept state, local issue registry, selected issue, selected protocol, direct dependencies и affected concept pages. Загружать весь root repository без reason нельзя.

## Создание новой концепции

Новая концепция создаётся только при наличии всех условий:

| Gate | Требование |
|---|---|
| User intent | пользователь явно просит создать концепцию или утверждает issue на создание |
| Slug | `concept_slug` ASCII-safe, не конфликтует с существующими путями |
| Название | человекочитаемое русское или смешанное название концепции |
| Reason | зачем концепция нужна и какой scope она удерживает |
| Boundary | что входит и что не входит в первую версию |
| Write set | перечислены файлы, которые будут созданы |
| Registry update | root execution index, root page registry и local page registry будут обновлены в одной transaction |

Минимальный bootstrap concrete concept:

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

Папки `Pages/`, `Output/` и `Exports/` могут оставаться без Markdown-файлов до появления первого реального artifact. Пустой `README.md` без операционной роли не создаётся.

## Path mapping для issue lifecycle

Service-level protocols остаются канонической моделью lifecycle. В `Execution Mode` они применяются с локальным корнем concept scope:

| Service path | Execution path |
|---|---|
| `State/service_state.md` | `Concepts/<concept_slug>/State/concept_state.md` |
| `Issues/issue_registry.jsonl` | `Concepts/<concept_slug>/Issues/issue_registry.jsonl` |
| `Issues/dependency_graph.jsonl` | `Concepts/<concept_slug>/Issues/dependency_graph.jsonl` |
| `Issues/<issue_id>/` | `Concepts/<concept_slug>/Issues/<issue_id>/` |
| `Issues/_archive/` | `Concepts/<concept_slug>/Issues/_archive/` |
| `Issues/_tombstones/` | `Concepts/<concept_slug>/Issues/_tombstones/` |
| `State/page_registry.jsonl` | `Concepts/<concept_slug>/State/page_registry.jsonl` для concept-internal pages; root registry хранит concept entrypoint |

Протоколы QA, requirements, solution/contract/output, complex issue, linked issues и retention применимы к concept issue только после такой подстановки. Если протокол требует root service artifact, агент должен объяснить, почему нужен context lift, а не молча смешивать scope.

## Concept lifecycle

| Status | Значение |
|---|---|
| `draft` | концепция создана, сеть ещё формируется |
| `active` | есть открытые issue или страницы в работе |
| `ready_for_closure_review` | все required issue закрыты, можно проверять всю concept-сеть |
| `closed` | пользователь утвердил closure после validation |
| `exported` | создан export package |
| `archived` | концепция выведена из active work, история сохранена |

Концепция не считается закрытой только потому, что закрыт один issue. Закрытие концепции требует проверки всей concept-сети и пользовательского approval.

## Closure preconditions

Перед переводом concept state в `closed` агент проверяет:

1. `Concepts/<concept_slug>/README.md` существует и является входом.
2. Локальный page registry не показывает orphan Markdown.
3. Все required concept pages имеют parent/backlink.
4. Все blocking issue закрыты, rejected, deferred с reason, archived или tombstone с понятным решением.
5. Blocking dependencies готовы или явно сняты пользователем.
6. Contract/output coverage для required issue сохранён.
7. Concept-level validation имеет `pass` или пользователь явно принимает `pass_with_notes`.
8. Пользователь утверждает closure.

Если open issue остаются, разрешён только `work_in_progress` export через [concept_export_protocol.md](concept_export_protocol.md); назвать его closed export нельзя.

## Связанные источники истины

| Ресурс | Роль |
|---|---|
| [Concepts/README.md](../../Concepts/README.md) | вход в слой концепций |
| [Concepts/_template/README.md](../../Concepts/_template/README.md) | шаблон новой концепции |
| [concept_export_protocol.md](concept_export_protocol.md) | export и closure package |
| [execution_index.md](../../State/execution_index.md) | root index active concept |
| [catalog.jsonl](../catalog.jsonl) | machine registry protocols |
| [context_loading_protocol.md](../common/context_loading_protocol.md) | focus packet и context lift |
| [persistence_protocol.md](../common/persistence_protocol.md) | transaction-like запись |

## Completion signal

Протокол считается выполненным, если:

- active concept выбран или честно зафиксирован `no_active_concept`;
- для создания concept известен write set или blocker;
- selected local protocol определён;
- root execution index и registry update plan известны;
- агент не начал concept-work вне concept scope.
