# Handoff Agent 02 → Coordinator — ACC-PRIMARY-001 bounded cleanup

## Status

`CLARIFICATION_REQUIRED`

PR #5 остается Draft. Новый synthesis интервью, расширение scope, TO-BE и моделирование не выполнялись.

## Что исправлено после review

1. Routing contradiction устранен.
   - Противоречивый `D001`, который после `A002` визуально перескакивал сразу к `A005-FIN` / `A005-RESP`, удален.
   - Канонический фактический маршрут теперь непротиворечив: `A002 → A003 → A004 → A005-FIN / A005-RESP → A006` при положительном решении.
   - Критерий выбора между `ROLE-FIN-DIRECTOR-001` и `ROLE-CONTRACT-OBLIGATION-RESP-001` остается `UNKNOWN`; новое routing rule не вводилось.

2. `HO-ACC-PRIMARY-005` удален.
   - Он дублировал уже смоделированный возврат решения через `HO-ACC-PRIMARY-003` / `HO-ACC-PRIMARY-004` и не представлял отдельный explicit business handoff.
   - Переход `APPROVE → POST` остается внутренней последовательностью parent process после положительного решения.
   - `SYS-1C-ACCOUNTING-001` остается системой выполнения `A006`, а не участником handoff.

3. Ссылки синхронизированы.
   - Удалены ссылки на `HO-ACC-PRIMARY-005` из clarification/issues/evidence.
   - `information/information_objects.yaml` и `process.yaml` проверены: ссылок на удаленный handoff или `D001` в них не было, поэтому бизнес-содержание этих файлов не изменялось.

## Что не изменялось

Бизнес-содержание detailed AS-IS не менялось: сохранены `CHECK → APPROVE → POST`, роли Главного бухгалтера, Финансового директора и функционального согласующего, отсутствие `ROLE-CEO-001`, `NOT_CONFIRMED` позиция про инициатора договора, а также `UNKNOWN` для routing, channel, reject/return и no-response/SLA/escalation.

Shared registry/dictionaries не изменялись. `CAND-ACC-BANK-001` не затрагивался. Новые процессы не создавались.
