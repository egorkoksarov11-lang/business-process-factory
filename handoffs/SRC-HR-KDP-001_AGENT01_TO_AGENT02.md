# Handoff Agent 01 → Agent 02 — SRC-HR-KDP-001

## Source

- `source_id`: `SRC-HR-KDP-001`
- interviewee role: `ROLE-HR-KDP-001` — Ведущий специалист по кадровому делопроизводству
- scope limitation: это первый источник пилота и рассказ одной роли; он не считается полным описанием всей кадровой службы.

## Canonical L1 processes ready for AS-IS work

| process_id | name | Agent 01 decision |
|---|---|---|
| `HR-HIRE-001` | Оформление приема сотрудника | Единый сквозной L1. Получение сведений, проверка документов, действия в 1С, ознакомление/подписание документов, отчетность о приеме и условная ветка воинского учета не разделены на самостоятельные L1. |
| `HR-TERM-001` | Оформление увольнения сотрудника | Самостоятельный L1. Источник подробно раскрывает прежде всего увольнение по заявлению сотрудника; иные основания UNKNOWN. |
| `HR-MIL-ORG-001` | Постановка организации на воинский учет | Самостоятельный первоначальный L1 с отдельной целью и набором входных документов. Формальный артефакт завершения UNKNOWN. |
| `HR-MIL-CHANGE-001` | Актуализация сведений воинского учета при изменениях данных сотрудника | Самостоятельный событийный L1 для личных/документных/образовательных/должностных изменений. Уведомления, вызванные приемом/увольнением, не дублируются здесь как отдельные L1. |
| `HR-MIL-RECON-001` | Ежегодная сверка воинского учета с военным комиссариатом | Самостоятельный периодический L1: отдельный календарный trigger 1 октября — конец года. |
| `HR-REPORT-REQ-001` | Подготовка кадрового отчета по запросу | Самостоятельный L1: trigger — запрос, result — сформированный и отправленный отчет. Конкретный requester/consumer UNKNOWN. |

## Architecture candidate, not canonicalized

`CAND-HR-MOVE-001` — **Оформление перевода / кадрового перемещения**.

Тема прямо названа одним из основных направлений работы, но источник не раскрывает достаточную кадровую последовательность. Не создавать AS-IS как полноценный процесс до уточнения как минимум trigger, business result, consumer, границ и различия между переводом и иными кадровыми перемещениями.

## Required classification decisions preserved from Agent 01

- Получение сведений для приема — стартовый блок/операция внутри `HR-HIRE-001`, не L1.
- Проверка комплекта документов — контрольный L2-блок внутри `HR-HIRE-001`.
- Ввод и оформление данных в 1С — операции внутри процесса, система не определяет отдельную бизнес-границу.
- Ознакомление с ЛНА, трудовой договор и иные кадровые документы при приеме — L2-блок внутри `HR-HIRE-001`.
- Отчетность о приеме в СФР — L2-блок внутри `HR-HIRE-001`; смена исполнителя на главного бухгалтера не создает новый L1 сама по себе.
- Воинский учет при приеме — условная L2-ветка `HR-HIRE-001`, а не отдельный L1 только для события приема.
- Отчетность об увольнении — L2-блок внутри `HR-TERM-001`.
- Воинский учет при увольнении — условная ветка `HR-TERM-001` до уточнения более самостоятельной границы.
- Постановка организации на воинский учет — отдельный L1.
- Уведомления военкомата об изменениях личных/документных/образовательных/должностных данных — отдельный событийный L1 `HR-MIL-CHANGE-001`.
- Ежегодная сверка — отдельный периодический L1 `HR-MIL-RECON-001`.
- Кадровые отчеты по запросу — отдельный L1 `HR-REPORT-REQ-001`.

## Roles and ambiguity

Normalized roles are in `registry/role_dictionary.yaml`.

Critical ambiguity: `ROLE-MIL-RESP-001` (Ответственный за воинский учет) is confirmed as a function/role, but the source does not establish whether this is a separate employee or a function assigned to `ROLE-HR-KDP-001`. Do not merge these roles without clarification.

The phrase about the employment contract first says it goes to a generic “director”, then explicitly says the General Director signs it. Only `ROLE-CEO-001` was normalized for the signing role; a separate generic “director” role was not invented.

## Systems, channels and external organizations

Confirmed information system:

- `SYS-1C-HR-001` — normalized as `1С: Зарплата и управление персоналом`; exact configuration/version is `UNKNOWN` because the transcript contains a contradictory recognition fragment.

Communication/delivery channels observed but intentionally **not** registered as HR information systems:

- Telegram;
- corporate messenger;
- email;
- in-person/oral communication;
- postal mail.

External organizations, not systems:

- СФР;
- военный комиссариат.

## Source-quality issues that Agent 02 must not silently correct

See `sources/metadata/SRC-HR-KDP-001.yaml`, especially:

1. contradictory 1С product-name fragment;
2. “director” vs explicit General Director in contract-signing passage;
3. internally contradictory phrase “приказ о приеме ... после заявления об увольнении” in the dismissal block;
4. uninterpretable word “кампании” in the initial list of work directions.

## Priority architecture UNKNOWN for AS-IS clarification

- Exact end boundary of `HR-HIRE-001`: whether confirmation of SFR submission is required to close the L1.
- Previous process/source of hiring data before the Director of HR sends them to KDP.
- Identity of the military-accounting responsible role.
- Exact end artifact for organization military-registration and annual reconciliation.
- Boundary of final payroll settlement inside/outside `HR-TERM-001`.
- Owner of the handover-of-duties block during dismissal.
- Requesters/consumers and channel for ad-hoc HR reports.
- Full architecture of transfer/personnel movement before `CAND-HR-MOVE-001` can receive a process_id.

## Views

- `views/VIEW-HR-KDP-ROLE-001.yaml` — role landscape; not a sequential process.
- `views/VIEW-HR-HIRE-E2E-001.yaml` — confirmed hire E2E fragment.
- `views/VIEW-HR-TERM-E2E-001.yaml` — confirmed termination E2E fragment for the described scenario.

## Expected next action

Agent 02 should build factual AS-IS only for the six canonical L1 packages above, preserving `EXPLICIT / INFERRED / UNKNOWN`, and should not build a detailed AS-IS for `CAND-HR-MOVE-001` until its architecture boundary is clarified.
