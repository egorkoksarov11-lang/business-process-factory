# Handoff Agent 01 → Agent 02 — HR synthesis 001

## Scope

Agent 01 завершил bounded architecture synthesis по четырем отдельным источникам:

- `SRC-HR-KDP-001` — интервью ведущего специалиста по кадровому делопроизводству;
- `SRC-HR-DIR-001` — интервью директора по персоналу;
- `SRC-HR-MODEL-001` — существующая пользовательская схема «Прием на работу»;
- `SRC-HR-MODEL-002` — существующая пользовательская схема «Заявка на подбор».

**Важно:** lifecycle/approval status обеих существующих схем — `UNKNOWN`. Не объявлять их автоматически действующим AS-IS, TO-BE или утвержденным регламентом. Использовать их как самостоятельные источники доказательств и проверять расхождения с интервью.

Детальный AS-IS Agent 01 не создавал. `as_is.steps` во всех новых пакетах пуст.

Source-specific evidence:

- `sources/evidence/SRC-HR-KDP-001_architecture.yaml`;
- `sources/evidence/SRC-HR-DIR-001_architecture.yaml`;
- `sources/evidence/SRC-HR-MODEL-001_architecture.yaml`;
- `sources/evidence/SRC-HR-MODEL-002_architecture.yaml`.

Element-level mappings существующих моделей:

- `mappings/SRC-HR-MODEL-001.yaml`;
- `mappings/SRC-HR-MODEL-002.yaml`.

## Каноническая L1-цепочка входа сотрудника

После synthesis архитектура имеет четыре последовательные управляемые границы:

`HR-STAFF-REQ-001 → HR-RECRUIT-001 → HR-HIRE-001 → HR-ADAPT-001`

### HR-STAFF-REQ-001 — Формирование и согласование заявки на подбор

Новый L1 канонизирован по `SRC-HR-MODEL-002` и collapsed step `MODEL-001#1`.

- trigger: возникла потребность в сотруднике;
- positive result: согласованная заявка направлена HR;
- alternate result: подбор сотрудника не согласован;
- consumer при positive result: `HR-RECRUIT-001`.

Элементы `1.1–1.7` остаются действиями/решениями этого L1. Не создавать семь микропроцессов.

Документы схемы зарегистрированы только как `EXISTING_INTERNAL` references внутри package:

- `DOC-HR-STAFF-LOAD-001` — Таблица оценки загрузки;
- `DOC-HR-STAFF-AUTHORITY-001` — Приказ / распоряжение / распределение функций / должностная инструкция;
- `DOC-HR-STAFF-REQUEST-001` — Заявка на подбор.

Шаблоны Agent 01 не создавал.

**Critical UNKNOWN:** схема может не быть текущим AS-IS. Проверить фактическую обязательность согласования Генеральным директором и upstream-логику до детализации.

### HR-RECRUIT-001 — Подбор сотрудника на вакантную позицию

Trigger пересмотрен: `HR получена согласованная заявка на подбор`.

Business result уточнен по `SRC-HR-MODEL-001`: кандидат выбран **и согласился на оффер**, после чего возможен handoff в `HR-HIRE-001`.

Формализованы бизнес-значимые L2:

- `HR-RECRUIT-SOURCE-001` — поиск и привлечение кандидатов;
- `HR-RECRUIT-SELECT-001` — отбор, собеседования и выбор;
- `HR-RECRUIT-SEC-001` — проверка Службой безопасности;
- `HR-RECRUIT-OFFER-001` — формирование, согласование и принятие оффера.

Проверка СБ и условное согласование оффера Генеральным директором подтверждены только reference-моделью `SRC-HR-MODEL-001`. Не превращать их в подтвержденный AS-IS без проверки статуса/фактической практики.

Циклы возврата к sourcing при несогласовании Генеральным директором или отказе кандидата сохранены в mapping и должны быть проверены Agent 02.

### HR-HIRE-001 — Кадровое оформление приема сотрудника

Trigger: кандидат согласился на оффер и сведения/условия переданы в КДП.

Формализованы L2:

- `HR-HIRE-KDP-001` — оформление кадровых документов и данных приема;
- `HR-HIRE-MIL-001` — условная ветка воинского учета при приеме;
- `HR-HIRE-SFR-001` — подготовка и отправка сведений о приеме в СФР.

Проверка документов, ввод в 1С, ознакомление с ЛНА, отдельные кадровые документы и подписи остаются операциями/контролями внутри `HR-HIRE-KDP-001`; отдельные process_id для них не создавались.

### HR-ADAPT-001 — Сопровождение адаптации и испытательного срока

Остается самостоятельным L1. Однако появился важный source conflict:

- `SRC-HR-DIR-001`: полноценный контроль адаптации/испытательного срока — функция Директора по персоналу;
- `SRC-HR-MODEL-001#7`: `Контроль выхода сотрудника и начало адаптации` расположен в lane Кадровика.

Agent 01 **не выбрал одну роль молча**. `owner_role` остается `UNKNOWN`; `ROLE-HR-KDP-001` включена только как participant-candidate на основании source boundary element.

Проверить у Agent 02: MODEL-001#7 — только контроль фактического выхода/handoff или участие Кадровика в полном процессе адаптации.

## Source conflict — СФР

`HR-HIRE-SFR-001` намеренно имеет `owner_role: UNKNOWN`.

- `SRC-HR-KDP-001`: Кадровик формирует сведения в 1С → сообщает Главному бухгалтеру → Главный бухгалтер отправляет/подтверждает сдачу.
- `SRC-HR-MODEL-001#9`: collapsed subprocess `Подготовка и отправка отчета в СФР` целиком расположен в lane Главного бухгалтера.

Не исправлять и не выбирать распределение до AS-IS clarification.

## Existing model «Прием на работу» сохранена как view, а не новый mega-process

View: `views/VIEW-HR-MODEL-ENTRY-E2E-001.yaml`.

Source-oriented E2E mapping:

`HR-STAFF-REQ-001 → HR-RECRUIT-001 → HR-HIRE-001 → boundary HR-ADAPT-001`

Он сохраняет путь source-модели от возникновения потребности до завершения кадрового оформления и начала адаптации. Это **не** означает, что вся исходная схема стала одним L1.

Расширенный view до завершения адаптации остается отдельно:

`views/VIEW-HR-HIRE-E2E-001.yaml` — `HR-RECRUIT-001 → HR-HIRE-001 → HR-ADAPT-001`, entry после согласованной заявки.

## Role Landscapes

### VIEW-HR-KDP-ROLE-001

L1 process refs сохраняются:

- `HR-HIRE-001`;
- `HR-TERM-001`;
- `HR-MIL-ORG-001`;
- `HR-MIL-CHANGE-001`;
- `HR-MIL-RECON-001`;
- `HR-REPORT-REQ-001`.

`HR-ADAPT-001` не добавлен как полноценное участие до разрешения conflict; MODEL-001#7 зафиксирован в notes/mapping как boundary element.

### VIEW-HR-DIRECTOR-ROLE-001

- `HR-RECRUIT-001`;
- `HR-HIRE-001`;
- `HR-ADAPT-001`;
- `HR-TERM-001`.

`HR-STAFF-REQ-001` не добавлен как участие Директора по персоналу: SRC-HR-MODEL-002 показывает Руководителя подразделения + Генерального директора; HR является consumer. Generic lane `HR` из MODEL-001 не приписан Директору автоматически.

## Другие канонические L1 без изменения границы

- `HR-TERM-001` — Прекращение трудовых отношений с сотрудником;
- `HR-MIL-ORG-001` — Постановка организации на воинский учет;
- `HR-MIL-CHANGE-001` — Актуализация сведений воинского учета при изменениях;
- `HR-MIL-RECON-001` — Ежегодная сверка воинского учета;
- `HR-REPORT-REQ-001` — Подготовка кадрового отчета по запросу.

## Architecture candidates

Остаются кандидатами, не AS-IS scope:

- `CAND-HR-MOVE-001` — перевод / кадровое перемещение;
- `CAND-HR-WORKFORCE-001` — более широкий контур управления изменением численности; отдельный case-level request теперь покрывает `HR-STAFF-REQ-001`;
- `CAND-HR-LEAVE-SCHEDULE-001`;
- `CAND-HR-LNA-001`;
- `CAND-HR-CONFLICT-001`;
- `CAND-HR-INSPECTION-001`;
- `CAND-HR-FOT-001`;
- `CAND-HR-CORP-EVENT-001`;
- `CAND-HR-TIME-LEAVE-001`.

## Priority UNKNOWN / clarification for Agent 02

1. Lifecycle/approval status `SRC-HR-MODEL-001/002`.
2. Фактическая обязательность approval Генеральным директором в `HR-STAFF-REQ-001`.
3. Соответствие generic lane `HR` роли Директора по персоналу.
4. Фактическая обязательность `HR-RECRUIT-SEC-001` и условного CEO approval оффера.
5. Точный handoff `HR-RECRUIT-001 → HR-HIRE-001` после принятого оффера.
6. Conflict роли по MODEL-001#7 / `HR-ADAPT-001`.
7. Conflict распределения ответственности в `HR-HIRE-SFR-001`.
8. Identity `ROLE-MIL-RESP-001`.
9. Остальные UNKNOWN `HR-TERM-001`, воинских L1 и ad-hoc reporting из предыдущего synthesis.

## Ready architecture scope for Agent 02

L1:

- `HR-STAFF-REQ-001`;
- `HR-RECRUIT-001`;
- `HR-HIRE-001`;
- `HR-ADAPT-001`;
- `HR-TERM-001`;
- `HR-MIL-ORG-001`;
- `HR-MIL-CHANGE-001`;
- `HR-MIL-RECON-001`;
- `HR-REPORT-REQ-001`.

L2 architecture packages:

- `HR-RECRUIT-SOURCE-001`;
- `HR-RECRUIT-SELECT-001`;
- `HR-RECRUIT-SEC-001`;
- `HR-RECRUIT-OFFER-001`;
- `HR-HIRE-KDP-001`;
- `HR-HIRE-MIL-001`;
- `HR-HIRE-SFR-001`.

`ARCHITECTURE_READY` означает готовую границу для следующей стадии, **не** подтверждение того, что reference-модель является действующим AS-IS. Agent 02 обязан сохранять `EXPLICIT / INFERRED / UNKNOWN / CONFLICT` и не достраивать процессы из общей HR-практики.
