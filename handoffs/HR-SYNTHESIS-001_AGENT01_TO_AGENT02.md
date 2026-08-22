# Handoff Agent 01 → Agent 02 — HR synthesis 001

## Scope after final architecture cleanup

Agent 01 завершил architecture cleanup по четырем независимым источникам:

- `SRC-HR-KDP-001` — интервью ведущего специалиста по кадровому делопроизводству;
- `SRC-HR-DIR-001` — интервью директора по персоналу;
- `SRC-HR-MODEL-001` — существующая пользовательская схема «Прием на работу»;
- `SRC-HR-MODEL-002` — существующая пользовательская схема «Заявка на подбор».

Lifecycle/approval status reference-моделей остается `UNKNOWN`. Они не считаются автоматически действующим AS-IS или TO-BE.

Element-level mappings сохранены без потери `CONFLICT / UNKNOWN`:

- `mappings/SRC-HR-MODEL-001.yaml`;
- `mappings/SRC-HR-MODEL-002.yaml`.

Детальный AS-IS, BPMN, TO-BE и оптимизация не создавались.

## Правило связей уровней

После cleanup применяется единое правило:

- `previous_processes / next_processes` у L1 содержат только L1↔L1 связи;
- `previous_processes / next_processes` у L2 содержат только sibling L2 одного `parent_process`;
- L1→L2 entry и L2→внешний L1 exit фиксируются через `parent_entry`, `parent_exit` или `boundary_handoff`;
- cross-level переходы в обычных `previous_processes / next_processes` не использовать.

Критично: удален прямой `HR-HIRE-KDP-001 → HR-ADAPT-001`. Запуск адаптации хранится на boundary родителя `HR-HIRE-001`.

## Канонические L1

1. `HR-STAFF-REQ-001` — Формирование и согласование заявки на подбор.
2. `HR-RECRUIT-001` — Подбор сотрудника на вакантную позицию.
3. `HR-HIRE-001` — Кадровое оформление приема сотрудника.
4. `HR-ADAPT-001` — Сопровождение адаптации и испытательного срока.
5. `HR-TERM-001` — Прекращение трудовых отношений.
6. `HR-MIL-ORG-001` — Постановка организации на воинский учет.
7. `HR-MIL-CHANGE-001` — Актуализация сведений воинского учета.
8. `HR-MIL-RECON-001` — Ежегодная сверка воинского учета.
9. `HR-REPORT-REQ-001` — Подготовка кадрового отчета по запросу.

## Recruitment L2 и conditional/reference-derived элементы

Под `HR-RECRUIT-001` формализованы:

- `HR-RECRUIT-SOURCE-001` — поиск/привлечение кандидатов;
- `HR-RECRUIT-SELECT-001` — отбор/собеседования/выбор;
- `HR-RECRUIT-SEC-001` — проверка СБ;
- `HR-RECRUIT-OFFER-001` — формирование/согласование/принятие оффера.

Канонический внутренний путь:

`SOURCE → SELECT → OFFER`

Conditional reference-derived вариант:

`SOURCE → SELECT → SEC → OFFER`

`HR-RECRUIT-SEC-001` имеет:

- `status: ARCHITECTURE_READY` только потому, что taxonomy Agent 01 использует этот статус для готовой архитектурной границы;
- `architecture_applicability.mode: CONDITIONAL_REFERENCE_DERIVED`;
- `as_is_applicability: UNKNOWN`;
- `source_confidence: REFERENCE_ONLY`;
- `source_lifecycle_status: UNKNOWN`.

**Agent 02 не должен включать SEC как обязательный AS-IS участок без подтверждения фактической применимости.**

Условное согласование оффера Генеральным директором также не является безусловным правилом. Оно хранится внутри `HR-RECRUIT-OFFER-001` как `conditional_reference_rule` с `applicability: UNKNOWN`.

## Boundary приема и адаптации

`HR-HIRE-001` больше не моделируется как простая последовательность `HR-HIRE → HR-ADAPT`.

### Trigger HR-ADAPT-001

Отдельное бизнес-событие:

`Фактический выход сотрудника и начало периода адаптации/испытательного срока`.

Это подтверждено `SRC-HR-DIR-001`.

### L2 внутри HR-HIRE-001

- `HR-HIRE-KDP-001` — core кадровое оформление;
- `HR-HIRE-MIL-001` — условная ветка воинского учета;
- `HR-HIRE-SFR-001` — сведения в СФР.

`SRC-HR-MODEL-001` показывает после core оформления параллельные ветки:

- начало адаптации;
- воинский учет;
- СФР.

Но lifecycle reference-модели UNKNOWN.

Поэтому:

- `HR-HIRE-KDP-001` обязателен для core result;
- applicability `HR-HIRE-MIL-001` — conditional;
- должен ли `HR-HIRE-MIL-001` завершиться до parent close — `UNKNOWN`;
- должен ли `HR-HIRE-SFR-001` завершиться до parent close — `UNKNOWN`;
- должен ли `HR-HIRE-001` полностью закрыться до старта `HR-ADAPT-001` — `UNKNOWN`.

Handoff в адаптацию хранится как boundary event на `HR-HIRE-001`, а не как `next_process` у L2.

## Source conflicts, которые нельзя разрешать молча

### Адаптация

- `SRC-HR-DIR-001`: полноценный контроль адаптации/испытательного срока — Директор по персоналу.
- `SRC-HR-MODEL-001`: `Контроль выхода сотрудника и начало адаптации` расположен в lane Кадровика.

Текущая архитектура трактует MODEL-001#7 как boundary/handoff с role conflict. Она не доказывает участие КДП во всем `HR-ADAPT-001`.

### СФР

- `SRC-HR-KDP-001`: КДП формирует сведения, Главный бухгалтер отправляет/подтверждает сдачу.
- `SRC-HR-MODEL-001`: подготовка и отправка целиком в lane Главного бухгалтера.

Owner `HR-HIRE-SFR-001` остается `UNKNOWN`.

## Views

### `VIEW-HR-FUNCTION-001`

Новый `FUNCTION_LANDSCAPE` HR/кадровой функции.

Это не официальное название подразделения: `official_department_name: UNKNOWN`.

Landscape показывает:

- все подтвержденные канонические L1;
- L1-группы: staffing/entry, employment relations, military accounting, HR reporting;
- участие `ROLE-HR-DIRECTOR-001`;
- участие `ROLE-HR-KDP-001`;
- architecture candidates отдельно от canonical `process_refs`.

### Role Landscapes

- `VIEW-HR-DIRECTOR-ROLE-001`;
- `VIEW-HR-KDP-ROLE-001`.

### End-to-End

- `VIEW-HR-HIRE-E2E-001` — conceptual multi-process E2E после согласованной заявки; не читать как строгую последовательность полного закрытия приема перед адаптацией.
- `VIEW-HR-MODEL-ENTRY-E2E-001` — source-oriented mapping существующей схемы «Прием на работу», lifecycle UNKNOWN.

## Architecture candidates

В `VIEW-HR-FUNCTION-001` и registry отдельно сохранены:

- `CAND-HR-MOVE-001`;
- `CAND-HR-WORKFORCE-001`;
- `CAND-HR-LEAVE-SCHEDULE-001`;
- `CAND-HR-LNA-001`;
- `CAND-HR-CONFLICT-001`;
- `CAND-HR-INSPECTION-001`;
- `CAND-HR-FOT-001`;
- `CAND-HR-CORP-EVENT-001`;
- `CAND-HR-TIME-LEAVE-001`.

Не превращать их в AS-IS scope без отдельного boundary confirmation.

## Priority UNKNOWN / CONFLICT for Agent 02

1. Фактическая применимость проверки СБ.
2. Фактическая применимость условного согласования оффера Генеральным директором.
3. Точный состав handoff `HR-RECRUIT-001 → HR-HIRE-001`.
4. Timing relation: старт `HR-ADAPT-001` относительно закрытия `HR-HIRE-001`.
5. Обязано ли завершение `HR-HIRE-MIL-001` предшествовать закрытию parent.
6. Обязано ли завершение `HR-HIRE-SFR-001` предшествовать закрытию parent.
7. CONFLICT по owner/распределению ответственности СФР.
8. CONFLICT по роли КДП vs Директора в boundary начала адаптации.
9. Identity `ROLE-MIL-RESP-001`.
10. Формальные результаты воинских процессов и иных ранее сохраненных candidates.

## Instruction to Agent 02

`ARCHITECTURE_READY` означает, что архитектурный объект/граница готовы к проверке следующей стадией. Для reference-derived L2 это **не** доказательство текущего AS-IS.

При построении AS-IS обязательно различать:

- interview-supported canonical blocks;
- conditional/reference-derived blocks;
- `UNKNOWN`;
- `CONFLICT`.

Не превращать reference-модель в фактический поток без подтверждения.
