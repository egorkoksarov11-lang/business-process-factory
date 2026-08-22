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

## Created/modified artifacts

Created:

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

Interview-supported flow includes:

1. Директор по персоналу → КДП: электронное сообщение с данными для приема; source names Telegram / corporate messenger / email.
2. КДП checks intake documents against an existing list.
3. If incomplete, KDP requests missing documents and waits for a complete package; return to completeness check.
4. KDP creates physical person and employee/hire record in `SYS-1C-HR-001` with explicitly listed personal/employment/payment data.
5. Before labor-contract signature: employee is provided local acts and job description for acknowledgement; labor contract is issued in two copies, signed by employee if there are no comments, then by CEO; one copy remains with HR, one is handed to employee.
6. Additional intake artifacts explicitly mentioned: hire order for signature, T-2 formed in 1C, personal-data consent, salary-transfer statement, ambiguous confidentiality/commercial-secret document.
7. SFR: KDP forms report in 1C, tells Chief Accountant it is ready and must be sent; Chief Accountant sends/then reports that it was submitted. Deadline stated by source: no later than the next day after hire date.
8. Military branch: source explicitly describes military-data entry, Form 10, signatures/verification, and information sent by post to the military commissariat within five days when applicable.

Exact current orchestration between 1C core, document-signing block, SFR, MIL and adaptation start is not fabricated where sources do not establish it.

## Blocking UNKNOWN / CONFLICT

1. `SFR`: interview says KDP prepares and Chief Accountant sends/confirms; `SRC-HR-MODEL-001` UNKNOWN lifecycle places the whole block in Chief Accountant lane.
2. `MIL`: exact applicability criterion and identity of `ROLE-MIL-RESP-001` versus `ROLE-HR-KDP-001` remain UNKNOWN; executor of forming/sending commissariat information is not explicit.
3. `HR-HIRE-MIL-001 required_before_parent_close`: UNKNOWN.
4. `HR-HIRE-SFR-001 required_before_parent_close`: UNKNOWN.
5. Timing of complete `HR-HIRE-001` close relative to actual employee start / `HR-ADAPT-001`: UNKNOWN.
6. Boundary role for `Контроль выхода сотрудника и начало адаптации`: KDP in reference model vs Director of HR in interview — CONFLICT.
7. Lifecycle/approval status `SRC-HR-MODEL-001`: UNKNOWN; reference parallel join is kept `REFERENCE_ONLY`.

Detailed questions are in `processes/HR-HIRE-001/analysis/clarifications.yaml`.

## Channel dictionary proposals for Coordinator

`registry/channel_dictionary.yaml` is currently empty. Agent 02 did not add canonical IDs independently. Current evidence requires normalization candidates:

- `CH-EMAIL-001` — Электронная почта — `EMAIL`;
- `CH-TELEGRAM-001` — Telegram — `MESSENGER`;
- `CH-CORP-MESSENGER-001` — Корпоративный мессенджер — `MESSENGER`;
- `CH-IN-PERSON-001` — Лично / очное сообщение — `IN_PERSON`;
- `CH-POSTAL-001` — Почтовое отправление — `POSTAL`.

Until Coordinator canonicalizes them, local AS-IS artifacts keep `channel_ref: null` plus exact `observed_channel_names`; channels are not confused with messages or systems.

No new canonical role/system ID is requested: current roles and `SYS-1C-HR-001` are sufficient for this scope.

## Next action

Coordinator/process owner should obtain answers to blocking clarification items and, if accepted, normalize the channel candidates by additive shared-dictionary update. Agent 02 can then finalize the current AS-IS and re-evaluate `AS_IS_READY`; Agent 03 should not start TO-BE while this status remains `CLARIFICATION_REQUIRED`.
