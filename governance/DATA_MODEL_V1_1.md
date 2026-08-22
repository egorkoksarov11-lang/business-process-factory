# Data Model v1.1 — модель хранения процессной информации

## 1. Назначение

Версия 1.1 усиливает структуру данных для стадии AS-IS и последующих стадий, не меняя базовую архитектуру конвейера.

Главная цель — чтобы из интервью сохранялась не только последовательность действий, но и:

- бизнес-объекты;
- документы и их состояние;
- сообщения, запросы и уведомления;
- файлы, отчеты, записи и наборы данных;
- системы и каналы;
- передачи ответственности между ролями и процессами;
- ожидания, решения, возвраты и параллельность;
- подтвержденные сроки, частоты и объемы;
- точная трассировка к источнику.

## 2. Backward compatibility

`schema_version: "1.0"` остается допустимой для уже созданных architecture packages.

Активные Agent 01 ветки не должны переписывать свои пакеты только из-за появления v1.1.

Правило миграции:

1. Agent 01 может завершить архитектуру в формате v1.0.
2. После merge архитектуры Agent 02 при старте AS-IS создает v1.1 detailed artifacts для соответствующего процесса.
3. Канонический `process_id`, границы и source refs сохраняются.
4. Нельзя менять подтвержденную архитектуру только ради соответствия новому формату.

## 3. Основная цепочка объектов

`Source → Source Segment → Evidence Fact → Process/L2 → AS-IS Step → Flow/Decision → Handoff → Information Object → Issue → Change → TO-BE Step → Diagram Element`

Каждый слой имеет свою функцию и не должен подменять соседний.

## 4. Source и Source Segment

Raw source хранится неизменно.

Для устойчивых ссылок длинный источник может быть разбит на логические сегменты:

- `segment_id`;
- исходный `source_id`;
- locator: строки, абзац, таймкод или иной устойчивый ориентир;
- короткий anchor/description;
- при необходимости — process refs.

Сегмент не является новой интерпретацией источника. Он только дает устойчивую точку ссылки.

## 5. Evidence Fact

Evidence fact отвечает на вопрос: **что источник подтверждает?**

Обязательная дисциплина:

- `EXPLICIT` — сказано/показано прямо;
- `INFERRED` — логический вывод, явно помеченный;
- `UNKNOWN` — данных недостаточно;
- `CONFLICT` — источники расходятся.

Факт должен ссылаться на `source_id` и по возможности на `segment_id`.

## 6. Process и Subprocess

Process package хранит устойчивую бизнес-границу:

- trigger;
- input;
- business result;
- consumer;
- owner/participants;
- parent/children;
- соседние L1 или L2;
- applicability;
- sources;
- ссылки на подробные AS-IS/TO-BE artifacts.

### 6.1 Cross-level discipline

`previous_processes` и `next_processes` должны по возможности ссылаться на объекты того же архитектурного уровня.

Нельзя использовать обычный `next_process` для смешения:

`L2 → внешний L1` или `L2 → L0`.

Выход из L2 наружу фиксируется через:

- `parent_exit`;
- `handoff`;
- событие/условие на границе parent process.

### 6.2 Applicability

Для элементов, подтвержденных только старой/reference-моделью, хранить применимость отдельно от архитектурной границы:

- `CONFIRMED_CURRENT`;
- `PARTIAL_CURRENT`;
- `UNKNOWN_CURRENT`;
- `REFERENCE_ONLY`;
- `CONDITIONAL`.

`ARCHITECTURE_READY` не означает автоматически `CONFIRMED_CURRENT`.

## 7. AS-IS Step

Шаг отвечает на вопрос: **кто что делает с каким объектом и какой наблюдаемый результат получает?**

Минимальный шаг:

- `step_id`;
- `role_ref`;
- `action`;
- `business_object`;
- `result`;
- incoming/outgoing flow refs;
- information object refs;
- system/channel refs, если значимы;
- timing/wait, если подтверждены;
- evidence refs;
- evidence class.

Шаг не должен превращаться в список кликов.

## 8. Information Object

Information Object — информационный объект, которым оперирует процесс.

Типы:

- `DOCUMENT`;
- `MESSAGE`;
- `REQUEST`;
- `NOTIFICATION`;
- `FILE`;
- `REPORT`;
- `FORM`;
- `RECORD`;
- `DATASET`;
- `REGISTER`;
- `OTHER`.

Примеры:

- заявление сотрудника;
- заявка на подбор;
- акт оказанных услуг;
- письмо в техническую поддержку;
- сообщение главному бухгалтеру о готовности сведений;
- подтверждение сдачи отчетности;
- банковская выписка;
- файл выгрузки;
- отчет для руководителя.

### 8.1 Документная классификация

Для документов дополнительно используется:

- `EXISTING_INTERNAL`;
- `EXTERNAL_STATUTORY`;
- `TRANSACTIONAL`;
- `PROPOSED_TEMPLATE`;
- `PROPOSED_CHECKLIST`;
- `PROPOSED_REGISTER`.

В AS-IS не создавать отсутствующие документы.

### 8.2 Роль объекта в процессе

Один и тот же объект может выступать как:

- `INPUT`;
- `OUTPUT`;
- `SUPPORTING`;
- `APPROVAL_BASIS`;
- `CONFIRMATION`;
- `HANDOFF_PAYLOAD`;
- `ARCHIVE`.

### 8.3 Состояния

Если источник позволяет, фиксировать переход состояния:

`received → checked → prepared → sent → approved/rejected → accepted → archived`.

Не придумывать промежуточные состояния, которых нет в evidence.

## 9. Communication

Сообщение нельзя растворять в поле `channel`.

Если коммуникация влияет на движение процесса, хранить отдельно:

- sender role;
- recipient role(s);
- purpose;
- payload/information object;
- channel;
- related step;
- ожидается ли ответ;
- срок ответа, если подтвержден;
- evidence refs.

Пример:

`Кадровик → Главный бухгалтер → сообщение о готовности сведений → корпоративный канал → требуется отправка в СФР`.

Канал (`email`, `messenger`, `phone`) не является самим сообщением.

## 10. Handoff

Handoff — передача объекта, результата или ответственности между ролями/процессами.

Хранить:

- from role/process;
- to role/process;
- trigger/condition;
- payload refs;
- acceptance/result;
- channel/system при наличии;
- timing/SLA, если известны;
- evidence refs;
- status/ambiguity.

Это особенно важно для межфункциональных процессов HR ↔ IT, HR ↔ Accounting, Accounting ↔ Financial Director и т. п.

## 11. Flow и Decision

Flow хранит маршрутизацию, а не бизнес-содержание шага.

Типы:

- `SEQUENCE`;
- `CONDITIONAL`;
- `RETURN`;
- `PARALLEL_SPLIT`;
- `PARALLEL_JOIN`;
- `HANDOFF`;
- `EXCEPTION`.

Decision хранит:

- вопрос/условие;
- ветви;
- result labels;
- evidence;
- applicability.

Условный reference-derived шаг нельзя превращать в обязательный sequence flow.

## 12. Timings, frequency, volume

Количественные данные фиксируются только если подтверждены источником:

- нормативный срок;
- фактический срок;
- ожидание;
- частота;
- объем;
- количество объектов/систем/юрлиц;
- SLA.

Каждое значение должно иметь source/evidence ref.

`10–15 минут` и `два раза в месяц` — структурируемые факты.

`обычно быстро` без числового значения остается текстовым observation.

## 13. Problems and Change Log

Issue должен ссылаться на конкретные AS-IS steps/flows/handoffs/information objects.

Change Log должен позволять восстановить:

`AS-IS element → Issue → Change → TO-BE element`.

Новый документ/message pattern в TO-BE должен иметь отдельный information object и при необходимости реальный attachment.

## 14. View types

Поддерживаются:

- `ROLE_LANDSCAPE`;
- `DEPARTMENT_LANDSCAPE`;
- `FUNCTION_LANDSCAPE`;
- `END_TO_END`;
- `PROCESS_DETAIL`.

View никогда не владеет копией процесса: он только ссылается на канонические IDs.

## 15. Минимальный пакет Agent 02

Для процесса, который идет в детальный AS-IS, Agent 02 должен создать/обновить минимум:

- `process.yaml`;
- `as-is/model.yaml`;
- `evidence/evidence.yaml` или source-level evidence refs;
- `information/information_objects.yaml`;
- `handoffs/handoffs.yaml`, если есть передачи;
- `analysis/clarifications.yaml`;
- `analysis/issues.yaml`.

Если конкретный тип объекта отсутствует, не создавать пустую фиктивную сущность только ради формальности.

## 16. Главный принцип

Насыщенность данных определяется не количеством полей, а возможностью ответить на вопросы:

1. Что реально происходит?
2. Кто отвечает?
3. С каким объектом работают?
4. Что передается другому участнику?
5. Через какой канал/систему?
6. Какой документ/сообщение подтверждает действие?
7. Где решение, ожидание, возврат или исключение?
8. Чем факт подтвержден?
9. Какая проблема возникает именно здесь?
10. Почему TO-BE изменяет этот элемент именно так?
