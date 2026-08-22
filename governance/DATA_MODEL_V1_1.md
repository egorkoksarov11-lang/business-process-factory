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

`schema_version: "1.0"` остается допустимой для уже созданных архитектурных артефактов в `main`, включая process packages, views и source/evidence artifacts. Появление шаблона v1.1 само по себе не требует переписывать существующие v1.0 файлы.

Активные Agent 01 ветки не должны переписывать свои пакеты или связанные архитектурные артефакты только из-за появления v1.1.

Правило миграции:

1. Agent 01 может завершить архитектуру в формате v1.0.
2. После merge архитектуры Agent 02 при старте AS-IS создает v1.1 detailed artifacts для соответствующего процесса.
3. Канонический `process_id`, границы, views, mappings и source refs сохраняются.
4. Существующие v1.0 evidence/view artifacts могут использоваться как вход без формальной миграции; при естественном обновлении файла допускается перевести его на v1.1.
5. Нельзя менять подтвержденную архитектуру только ради соответствия новому формату.
6. Если Agent 02 естественным образом обновляет `process.yaml`, чтобы подключить detailed AS-IS artifacts, он переводит этот package на `schema_version: "1.1"` и использует только канонический `model_refs`. Собственные альтернативные секции ссылок (`as_is_artifacts` и аналоги) не создаются.

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

### 7.1 Partial order / неизвестный взаимный порядок

Наличие нескольких подтвержденных шагов не означает, что источник подтверждает их полную линейную последовательность.

Если известна только часть порядка:

- подтвержденные зависимости хранятся в `orchestration_constraints.confirmed_order`;
- существенные пары/блоки с неизвестным взаимным порядком хранятся в `orchestration_constraints.order_unknown` с `relation: ORDER_UNKNOWN`;
- искусственный `SEQUENCE` между такими шагами не создается;
- отсутствие sequence flow в этой ситуации не является дефектом модели.

`confirmed_order` должен иметь evidence. `order_unknown` должен явно объяснять, какой порядок источник не устанавливает.

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

### 8.4 Разделение AS-IS и TO-BE

Information objects и communications подчиняются тому же правилу неизменности истории, что и шаги процесса.

Каждая запись должна явно относиться к `model_layer: AS_IS` или `model_layer: TO_BE`.

Правила:

- AS-IS object/communication после фиксации не переписывается TO-BE дизайном;
- если TO-BE использует объект без изменений, допускается ссылаться на AS-IS ID;
- если объект или коммуникация меняется, TO-BE получает отдельный ID с `derived_from_information_ref` / `derived_from_communication_ref`;
- новый или измененный TO-BE объект должен ссылаться на `introduced_by_change_refs`;
- proposed object не должен маскироваться под подтвержденный AS-IS даже если название совпадает.

Это позволяет восстановить не только изменение шага, но и изменение документа, сообщения, payload, состояния или канала.

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

### 9.1 Channel reference contract

Канонические ссылки на канал всегда используют IDs из `registry/channel_dictionary.yaml` вида `CH-*`.

- `communication.channel_ref` — **одиночное** поле: `CH-*` либо `null`;
- `information_object.channel_refs` и `handoff.channel_refs` — массивы canonical `CH-*` IDs;
- enum `EMAIL`, `PHONE`, `MESSENGER`, `TICKET_PORTAL` является `channel_type`, а не `channel_ref`;
- свободное название из источника не записывается как canonical ref.

Если канал еще не нормализован Coordinator:

- canonical ref остается `null` / `[]` в зависимости от поля;
- исходное wording хранится в `observed_channel_names`;
- Agent 02 передает dictionary proposal и не создает новый `CH-*` самостоятельно.

System и channel классифицируются отдельно. Интерфейс/«личный кабинет» не должен автоматически становиться каналом, пока не установлен его статус относительно системы.

### 9.2 Acknowledgement semantics

Acknowledgement — отдельная communication/information object, если источник подтверждает факт сообщения/подтверждения приема.

`UNKNOWN` acknowledgement означает неизвестность, а не отсутствие. Нельзя использовать неизвестный acknowledgement как доказанный gateway condition либо обходить основную процессную ветку так, будто сообщения точно нет.

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

Handoff не создается только ради структурного перехода между двумя L2 одного parent process. Должна существовать реальная передача объекта, результата или ответственности.

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

Decision не должен логически перескакивать через подтвержденные шаги передачи/ожидания. Если routing criterion неизвестен, он остается `UNKNOWN`, а не достраивается правилом «по смыслу».

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

Новый или измененный документ/message pattern в TO-BE должен иметь отдельный TO-BE information object, связь с исходным AS-IS object при наличии и при необходимости реальный attachment.

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
