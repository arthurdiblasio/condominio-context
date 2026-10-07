# Autorização e contexto tenant no contrato

## Decisão conceitual de autorização

Cada command/query é avaliado para:

`UserAccount ativa + Person + RoleAssignment + Permission + Scope + Resource + Contexto + Condições do domínio`

Ausência, incompatibilidade ou ambiguidade não concede acesso. A matriz Role × Permission × Scope é uma baseline proposta em [PERMISSIONS.md](../../PERMISSIONS.md), não um grant efetivo.

## Contexto de tenant

- Toda operação tenant-scoped identifica um único Condominium/contexto alvo.
- O contexto não é inferido apenas da pessoa, Unit, Block ou Role homônimo.
- Unit, Block, CommonArea, relações tipadas e resource operacional devem pertencer ao mesmo Condominium.
- Uma ação autorizada no Condominium A não se aplica ao recurso correspondente de B.
- Para suporte cross-tenant, `platform.cross_tenant.access` não substitui permission do resource, finalidade explícita, alvo e auditoria. Não há fallback global.
- A estrutura técnica que transportará o tenant é decisão posterior. O requisito é tornar contexto e recurso suficientemente explícitos para verificar igualdade/coerência.

## Scopes

Scopes aceitos no modelo atual: `PLATFORM`, `CONDOMINIUM`, `BLOCK`, `UNIT`, `FEATURE`. O scope limita o grant e não muda a localização/tenant do resource. Cobertura descendente só existe quando capability declara explicitamente; não presumir herança. Assembly/AgendaItem podem contextualizar o resource sem se tornar novo scope.

## Per-operation authorization mapping

| Área | Permissions conceituais catalogadas | Scope/contexto | Lacunas importantes |
|---|---|---|---|
| Tenant/platform | `platform.condominium.manage`, `platform.cross_tenant.access`, `condominium.read/update` | `PLATFORM` ou `CONDOMINIUM` | Bootstrap/primeiro admin e grants cross-tenant OD-03/14/15/16. |
| Estrutura | `block.manage`, `unit.read/update`, `common_area.read/manage` | Condominium/Block/Unit com coverage declarada | Fechamento estrutural precisa obedecer relações/atividade histórica. |
| Person/links | `person.read/update`, `unit_ownership.*`, `unit_residency.*`, `unit_tenancy.*` | Tenant e Unit | `person.create`, associação/deduplicação e escopo de fields não estão definidos; OD-03/07/09. |
| Account | Nenhuma `user_account.*` | Conta global/local a decidir; comandos de estado também requerem tenant purpose quando aplicável | OD-03 bloqueia permission mapping final; não reutilizar `person.update`. |
| RoleAssignment | `role_assignment.read/grant/suspend/reactivate/revoke` | Scope alvo dentro da autoridade do concessor | Governança, conflitos e delegação OD-02/14/15/16. |
| Reservation/Event | `reservation.read/create/update/cancel/approve`, `event.read/manage` | Unit/Block/Condominium conforme grant | `reservation.reject` não é capability distinta; reuso de `approve` para rejeitar carece de decisão. |
| Visit/access | `visit.read/register`, `access_authorization.read/create/revoke`, `access_event.read/register/correct` | Condominium/Block/Unit + período/recurso | Approve/cancel AccessAuthorization, gerenciar/concluir Visit sem capability exata; OD-05. |
| Package | `package.read/register/notify/confirm/release/correct/history.read` | Unit/Condominium no tenant correto | Cancelar/contestar, correção e atores finais OD-06. |
| Notification | `notification.read/create/delivery.read/channel.configure` | Tenant/finalidade/destinatário | Cancelar/retry e autorizar processo automático dependem de OD-08. |
| Assembly/vote | `assembly.read/manage/open/close`, `participant.*`, `voting_eligibility.*`, `proxy.*`, `vote.*`, `quorum.read`, `assembly.result.read` | Condominium + Assembly/AgendaItem como contexto do recurso | Apuração/publicação, cancelamento de pauta, contestação e validade legal OD-01/02. |
| Modules/features | `module.read/enable/disable`, `feature.read/enable/disable` | `PLATFORM` no catálogo; Condominium/Feature para habilitação | OD-11/17, dados históricos e operações ativas. |
| Finance | `finance.read`, `finance.income.manage`, `finance.expense.manage`, `finance.supplier.manage`, `finance.report.read`, `finance.document.export` | Feature + Condominium | Optional module; limites contábeis OD-12. |
| Audit | `audit_log.read` e capability específica da ação corretiva | Tenant/resource/scope originais | Ler audit log é também auditável; acesso e retenção OD-07. |

## Negação e minimização

Diferenciar internamente identidade ausente/inativa, grant ausente, Permission ausente, Scope incompatível, tenant mismatch, recurso fora do scope e regra/estado de negócio inválidos. A resposta conceitual ao cliente não revela existência, dados, campos ou estado de recurso cross-tenant; o nível de detalhe segue privacy e [error model](./errors.md).

Permission de operação não é condição de domínio substituta: possuir `vote.participate` não dá `VotingEligibility`; ser resident/owner/tenant não substitui relação validada nem grant; Module habilitado não concede Permission.

## Auditoria

Command sensível identifica actor/subject, tenant, recurso, ação, estado anterior/resultante quando houver, instante, resultado e reason quando exigido. Leitura de dados sensíveis/audit log tem seu próprio grant e finalidade.
