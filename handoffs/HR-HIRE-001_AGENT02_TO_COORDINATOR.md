# Handoff Agent 02 → Coordinator — HR-HIRE-001 AS-IS 001

## Scope

Agent 02 восстановил первый detailed AS-IS только для:

- `HR-HIRE-001` — Кадровое оформление приема сотрудника;
- `HR-HIRE-KDP-001`;
- `HR-HIRE-MIL-001`;
- `HR-HIRE-SFR-001`.

Подбор, адаптация, увольнение и другие HR L1 не детализировались. `HR-ADAPT-001` используется только как внешний parent boundary target.

## Status

`CLARIFICATION_REQUIRED`.

Причина: core flow подтвержден интервью, но остаются блокирующие UNKNOWN/CONFLICT по SFR role boundary, воинской applicability/role, completion boundary parent и Hire→Adaptation boundary.

## Bounded normalization cleanup after Coordinator review

После review выполнен только Schema v1.1 / traceability / orchestration cleanup без нового synthesis и без добавления бизнес-логики:

- `processes/HR-HIRE-001/process.yaml` переведен на `schema_version: "1.1"`;
- нестандартная секция `as_is_artifacts` удалена;
- канонические ссылки на detailed artifacts находятся только в `model_refs` (`as_is_model`, `information_objects`, `handoff_register`, `clarification_register`, `issues`, `change_log`, `to_be_model`);
- parent `as-is/model.yaml` содержит полную traceability к `Q-HR-HIRE-001-001` ... `Q-HR-HIRE-001-010`;
- partial order формализован в `orchestration_constraints.confirmed_order` и `orchestration_constraints.order_unknown`;
- sequence flows оставлены только там, где относительный порядок подтвержден; искусственный единый порядок между 1С, документным блоком, приказом, Т-2, согласиями/заявлениями, confidentiality/commercial-secret document, MIL, SFR и Adaptation не создавался;
- `SRC-HR-MODEL-001` остается `REFERENCE_ONLY`, lifecycle/approval status `UNKNOWN`; его parallel split/join не используется как current AS-IS;
- traceability в `clarifications.yaml` и `issues.yaml` очищена от ссылок на снятые искусственные flow IDs.

Никакой новой бизнес-логики в cleanup не добавлялось.

## Created/modified artifacts

Created in the original AS-IS reconstruction:

- `processes/HR-HIRE-001/evidence/evidence.yaml`
- `processes/HR-HIRE-001/as-is/model.yaml`
- `processes/HR-HIRE-001/information/information_objects.yaml`
- `processes/HR-HIRE-001/handoffs/handoffs.yaml`
- `processes/HR-HIRE-001/analysis/clarifications.yaml`
- `processes/HR-HIRE-001/analysis/issues.yaml`
- `handoffs/HR-HIRE-001_AGENT02_TO_COORDINATOR.md`

Modified:

- `processes/HR-HIRE-001/process.yaml`

Shared registries/dictionaries не изменялись.

## Confirmed current AS-IS core

Interview-supported facts include:

1. Директор по персоналу → КДП: электронное сообщение с данными для приема; source names Telegram / corporate messenger / email.
2. КДП checks intake documents against an existing list.
3. If incomplete, KDP requests missing documents and waits for a complete package; return to completeness check.
4. KDP creates physical person and employee/hire record in `SYS-1C-HR-001` with explicitly listed personal/employment/payment data.
5. Local acts and job description are provided before labor-contract signature; labor contract is issued in two copies, signed by employee if there are no comments, then by CEO; one copy remains with HR, one is handed to employee.
6. Additional intake artifacts explicitly mentioned: hire order for signature, T-2 formed in 1C, personal-data consent, salary-transfer statement, ambiguous confidentiality/commercial-secret document. Their relative order is not invented.
7. SFR: after employee creation is completed in 1C, KDP forms report in 1C, tells Chief Accountant it is ready and must be sent; Chief Accountant sends and then reports that it was submitted. Deadline stated by source: no later than the next day after hire date.
8. Military branch: source explicitly describes military-data entry, Form 10, signatures/verification, and information sent by post to the military commissariat within five days when applicable. Only explicit internal ordering is retained; the relation between Form 10 and commissariat-information subblocks remains `ORDER_UNKNOWN`.

## Partial-order representation

Current AS-IS does not assert a single end-to-end sequence across all hire artifacts. Confirmed ordering constraints include:

- document completeness check → physical-person/employee creation in 1C;
- local acts + job description → before employee signature of the labor contract;
- employee labor-contract signature → CEO signature;
- employee/hire creation completed in 1C → SFR data formation;
- SFR internal order: KDP forms → informs Chief Accountant → Chief Accountant sends → informs KDP;
- Form 10 exists before operations performed on that card; mutual order of employee signature vs responsible-person check/signature is `ORDER_UNKNOWN`;
- commissariat information formation → postal sending.

All other relative positions are stored as `ORDER_UNKNOWN` / `UNKNOWN` rather than artificial sequence or parallel flows.

## Blocking UNKNOWN / CONFLICT

1. `SFR`: interview says KDP prepares and Chief Accountant sends/confirms; `SRC-HR-MODEL-001` UNKNOWN lifecycle places the whole block in Chief Accountant lane.
2. `MIL`: exact applicability criterion and identity of `ROLE-MIL-RESP-001` versus `ROLE-HR-KDP-001` remain UNKNOWN; executor of forming/sending commissariat information is not explicit.
3. `HR-HIRE-MIL-001 required_before_parent_close`: UNKNOWN.
4. `HR-HIRE-SFR-001 required_before_parent_close`: UNKNOWN.
5. Timing of complete `HR-HIRE-001` close relative to actual employee start / `HR-ADAPT-001`: `UNKNOWN_MAY_OVERLAP`.
6. Boundary role for `Контроль выхода сотрудника и начало адаптации`: KDP in reference model vs Director of HR in interview — CONFLICT.
7. Lifecycle/approval status `SRC-HR-MODEL-001`: UNKNOWN; reference parallel join is kept `REFERENCE_ONLY`.
8. Branch for employee comments to the labor contract: UNKNOWN and remains non-blocking clarification `Q-HR-HIRE-001-010`.

Detailed questions are in `processes/HR-HIRE-001/analysis/clarifications.yaml`; all Q001–Q010 are reachable from parent AS-IS `open_questions`.

## Channel dictionary proposals for Coordinator

`registry/channel_dictionary.yaml` remains unchanged. Agent 02 did not add canonical IDs independently. Current evidence retains local `channel_ref: null` plus `observed_channel_names` for:

- `Telegram`;
- `Корпоративный мессенджер`;
- `Электронная почта`;
- `Лично`;
- `Почтовое отправление`.

Candidate canonical IDs remain proposals only:

- `CH-EMAIL-001` — Электронная почта — `EMAIL`;
- `CH-TELEGRAM-001` — Telegram — `MESSENGER`;
- `CH-CORP-MESSENGER-001` — Корпоративный мессенджер — `MESSENGER`;
- `CH-IN-PERSON-001` — Лично / очное сообщение — `IN_PERSON`;
- `CH-POSTAL-001` — Почтовое отправление — `POSTAL`.

No new canonical role/system ID is requested: current roles and `SYS-1C-HR-001` are sufficient for this scope.

## Next action

PR #6 remains Draft for repeat Coordinator review. No TO-BE, BPMN, Visio, document design or merge should start from this cleanup. Blocking clarifications remain unchanged in meaning; after answers, Agent 02 can re-evaluate `AS_IS_READY`.
