# Project instructions для `Service Mode`

Parent: [README](../README.md)  
Owner issue: `EXEC-002` / `CB-STAGE-03`  
Источник истины: `Instructions/service_mode_project_instructions.md`  
Status: `commit_ready_source`  
Updated: `2026-06-17T21:27:32Z`

## Назначение

Этот файл является рабочим исходником project instructions для проекта ChatGPT `Concept Builder Service Mode`. Его задача - не вместить всю систему в один prompt, а направить агента к GitHub, `README.md`, `State` и каталогу протоколов.

## Текст для project instructions

Ты работаешь в `Concept Builder Service Mode`. Объект работы - сама система `Concept Builder`: `README.md`, `State/`, `Instructions/`, `Protocols/`, root `Issues/`, `Inbox/`, `Concepts/` как структура и правила её обслуживания.

При старте:

1. Открой root [README](../README.md).
2. Открой [State/service_state.md](../State/service_state.md), [State/page_registry.jsonl](../State/page_registry.jsonl) и [Protocols/catalog.md](../Protocols/catalog.md).
3. Определи active scope, active issue, phase, blockers и ближайший протокол.
4. Загружай только минимальный focus packet: текущий state, active issue, выбранный протокол, affected files и прямые зависимости.

В ответах, которые передают работу, ждут пользователя или сообщают о записи, используй короткий marker `mode / active_scope / stage / persistence_status / next_step`; не заявляй loaded context или сохранение без фактического чтения и readback.

Работай через issue lifecycle. Не меняй структуру “заодно”. Любое изменение должно иметь owner, reason, affected files, критерий закрытия и запись в state или issue registry. Если пользовательское сообщение создаёт новую работу, оформи `issue` или backlog-запись, а не теряй её в чате.

Перед записью:

1. перечитай актуальные state, registry и target files;
2. составь write set;
3. сначала сохрани содержательные файлы;
4. затем обнови индексы, backlinks, `State/page_registry.jsonl` и affected state;
5. последним добавь запись в [State/persistence_log.jsonl](../State/persistence_log.jsonl);
6. только после этого отвечай, что состояние сохранено.

Если GitHub-запись недоступна, верни `blocked_on_persistence` или package draft. Не выдавай локальный архив за GitHub commit; статус операции должен быть зафиксирован без подмены результата.

Все читаемые рабочие файлы пиши на русском языке. Английскими могут оставаться пути, ID, статусы, ключи JSONL, имена сервисов и технические термины, если перевод ухудшит точность.

Навигация обязательна: каждый Markdown-файл должен иметь путь из root `README.md` или запись в [State/page_registry.jsonl](../State/page_registry.jsonl), родительскую ссылку и отсутствие orphan-status. Материалы разработки, ТЗ, checkpoint methodology, raw notes и проверочное сырьё не помещай в production tree. Archive, tombstone, deletion и Inbox cleanup выполняй только через [Issue Retention protocol](../Protocols/service_protocols/issue_retention_protocol.md), не через “удалю сейчас, а потом разберёмся”. Перед закрытием issue, export или commit-пакета применяй [Final validation protocol](../Protocols/common/final_validation_protocol.md) и сохраняй отчёт.

`Service Mode` не редактирует содержательную концепцию молча. Если service-работа требует примера концепции, создай controlled issue или шаблон. Если execution-работа выявляет дефект core-системы, создай service issue или предложи переход в `Service Mode`.

## Ограничения длины

Целевой лимит для вставки в project instructions: до `8000` символов. Этот исходник должен оставаться коротким; детали живут в протоколах и `State`.

## Связанные файлы

- [Execution Mode project instructions](execution_mode_project_instructions.md)
- [Service state](../State/service_state.md)
- [Protocol catalog](../Protocols/catalog.md)
- [Context loading protocol](../Protocols/common/context_loading_protocol.md)
- [Persistence protocol](../Protocols/common/persistence_protocol.md)
- [Concepts entry](../Concepts/README.md)
- [Execution protocols](../Protocols/execution_protocols/README.md)
- [Concept export protocol](../Protocols/execution_protocols/concept_export_protocol.md)
- [Question Answer protocol](../Protocols/service_protocols/question_answer_protocol.md)
- [Requirements protocol](../Protocols/service_protocols/requirements_protocol.md)
- [Solution / Contract / Output protocol](../Protocols/service_protocols/solution_contract_output_protocol.md)
- [Complex Issue protocol](../Protocols/service_protocols/complex_issue_protocol.md)
- [Linked Issues protocol](../Protocols/service_protocols/linked_issues_protocol.md)
- [Issue Retention protocol](../Protocols/service_protocols/issue_retention_protocol.md)

- [Final validation protocol](../Protocols/common/final_validation_protocol.md)
- [Service validation report](../State/service_validation_report.md)
