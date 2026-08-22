# Handoff Agent 02 → Coordinator — IT-SUPPORT-001 Schema v1.1 cleanup

## Scope

В PR #7 выполнен только bounded Schema v1.1 cleanup уже принятой содержательной реконструкции AS-IS `IT-SUPPORT-001`.

Новый analysis/synthesis не выполнялся. Архитектура L1/L2, TO-BE, automation, BPMN/Visio и shared registries/dictionaries не изменялись.

## Process package

`processes/IT-SUPPORT-001/process.yaml` стандартизирован на `schema_version: "1.1"`.

Detailed artifacts теперь подключаются только через канонический `model_refs`:

- `as_is_model` → `processes/IT-SUPPORT-001/as-is/model.yaml`
- `information_objects` → `processes/IT-SUPPORT-001/information/information_objects.yaml`
- `handoff_register` → `processes/IT-SUPPORT-001/handoffs/handoffs.yaml`
- `clarification_register` → `processes/IT-SUPPORT-001/analysis/clarifications.yaml`
- `issues` → `processes/IT-SUPPORT-001/analysis/issues.yaml`
- `change_log` → `null`
- `to_be_model` → `null`

Собственный альтернативный контракт ссылок внутри `as_is` удален.

## Channel cleanup

Enum-like значения `EMAIL`, `PHONE`, `PERSONAL_CABINET` удалены из `channel_refs` во всех detailed AS-IS artifacts и communications.

Пока canonical channel IDs отсутствуют, используется:

- `channel_refs: []`;
- `observed_channel_names` с исходными наблюдаемыми названиями каналов.

Agent 02 не создавал `CH-*` IDs и не менял `registry/channel_dictionary.yaml`.

### Channel candidates для решения Coordinator

- `Электронная почта` → candidate channel type `EMAIL`.
- `Телефон` → candidate channel type `PHONE`.
- `Личный кабинет` → вероятный `TICKET_PORTAL` / portal interface; требуется Coordinator canonicalization до назначения channel ID.

`PERSONAL_CABINET` не предлагается как самостоятельный canonical ID/type.

## Acknowledgement semantics

Business evidence сохранен без расширения:

- email acknowledgement → `CONFIRMED_CURRENT`;
- acknowledgement по телефону → `UNKNOWN_CURRENT`;
- acknowledgement через личный кабинет → `UNKNOWN_CURRENT`.

Основной переход `INTAKE → RESOLVE` теперь моделируется независимо от acknowledgement. Email acknowledgement сохранен как отдельная подтвержденная коммуникация и не является gateway, блокирующим взятие ticket в работу.

Новые сообщения для телефона или личного кабинета не создавались.

## Business AS-IS unchanged

Содержательная логика не пересматривалась:

`User request → HelpDesk ticket → ticket number → support takes ticket → processing → temporary OR permanent solution → conditional finalize after temporary solution`.

Сохранено:

- HelpDesk = system `SYS-IT-HELPDESK-001`;
- ticket = `RECORD` information object;
- request = отдельный information object;
- message ≠ channel;
- 30 минут не трактуются как подтвержденный SLA;
- 10–15 минут сохранены как observation с `measurement_scope: UNKNOWN`;
- closure после временного решения = UNKNOWN;
- support → IT lead handoff = `UNKNOWN_CURRENT`;
- messenger bot = `IN_DEVELOPMENT_NOT_AS_IS`;
- repeated/duplicate requests = issues, не процессы;
- process status = `CLARIFICATION_REQUIRED`.

## Ref integrity check

После cleanup проверены связи:

- steps ↔ flows;
- communications ↔ information objects;
- handoffs ↔ communications;
- clarifications ↔ steps / flows / handoffs / information objects;
- issues ↔ AS-IS elements.

Удаленный acknowledgement-routing flow больше не используется как предположение об отсутствии acknowledgement по non-email каналам. Основной lifecycle и условный `IT-SUPPORT-FINALIZE-001` сохранены.

## Open Coordinator decision

Требуется только отдельное решение Coordinator по canonical channel normalization и, при необходимости, созданию `CH-*` IDs в shared channel dictionary.

До такого решения Agent 02 оставляет все channel refs пустыми и хранит только observed channel names.

## PR state

PR #7 должен оставаться Draft для повторного Coordinator review. Merge не выполнять.

Final Schema v1.1 cleanup completed: communication channel field normalized from channel_refs to canonical channel_ref. Business AS-IS unchanged.
