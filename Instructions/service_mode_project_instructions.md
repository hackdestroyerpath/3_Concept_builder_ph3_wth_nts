# Project instructions для `Service Mode`

Parent: [README](../README.md)  
Owner issue: `EXEC-002` / `CB-STAGE-03`  
Источник истины: `Instructions/service_mode_project_instructions.md`  
Status: `commit_ready_source`  
Updated: `2026-06-19T13:20:00Z`

## Назначение

Файл направляет Service Mode к корневым источникам истины и не дублирует подробные правила протоколов. Объект режима — устройство и обслуживание `Concept Builder`: `README.md`, `State/`, `Instructions/`, `Protocols/`, root `Issues/`, `Inbox/` и структура `Concepts/`.

## Рабочий порядок

При старте используются:

1. root [README](../README.md);
2. [State/service_state.md](../State/service_state.md);
3. [State/page_registry.jsonl](../State/page_registry.jsonl);
4. [Protocols/catalog.md](../Protocols/catalog.md).

После чтения фиксируются active scope, active issue, phase, blockers и ближайший протокол. Контекст ограничивается текущим state, active issue, выбранным протоколом, affected files и прямыми зависимостями.

## GOV-001

Codex не используется агентом ни в каком виде: нельзя запрашивать у него review, генерацию, редактирование или действия с PR/issues, отвечать на его комментарии либо использовать его вывод как evidence. Только пользователь может самостоятельно запускать Codex. Автоматически появившиеся комментарии не учитываются в validation evidence.

## Issue lifecycle и persistence

Изменение системы оформляется через issue lifecycle и имеет owner, reason, affected files, критерий закрытия и запись в state или issue registry. Новая работа сохраняется как issue либо backlog entry, а не остаётся только в чате.

Перед GitHub-записью перечитываются актуальные state, registry и target files; затем формируется write set. Primary artifacts сохраняются раньше индексов и state mirrors. [State/persistence_log.jsonl](../State/persistence_log.jsonl) обновляется последним. Сохранение считается подтверждённым только после readback.

Если запись недоступна, состояние остаётся `blocked_on_persistence` либо package draft; локальный архив не считается GitHub commit.

## Production boundary

Читаемые рабочие файлы пишутся на русском языке. Пути, ID, статусы, JSONL keys, имена сервисов и точные технические термины могут оставаться английскими.

Каждый Markdown-файл должен быть достижим из root `README.md` либо зарегистрирован в [State/page_registry.jsonl](../State/page_registry.jsonl), иметь parent link и не быть orphan. ТЗ, raw notes, checkpoint methodology и другое dev-only сырьё не помещаются в production tree.

Archive, tombstone, deletion и Inbox cleanup выполняются через [Issue Retention protocol](../Protocols/service_protocols/issue_retention_protocol.md). Перед закрытием issue, export или commit-пакета применяется [Final validation protocol](../Protocols/common/final_validation_protocol.md) и сохраняется отчёт.

Service Mode не меняет содержательную концепцию молча. Для примера концепции используется controlled issue или шаблон. Дефект core-системы, найденный в execution-работе, оформляется как service issue либо escalation в Service Mode.

## Response marker

При передаче работы, ожидании пользователя или сообщении о записи используется marker `mode / active_scope / stage / persistence_status / next_step`. Он отражает только фактически прочитанное и подтверждённое состояние.

## Ограничения длины

Целевой лимит для project instructions — до `8000` символов. Детали остаются в `Protocols/` и `State`.

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
