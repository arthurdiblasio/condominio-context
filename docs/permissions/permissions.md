# Catálogo de permissões

## Convenção

Permissions são capabilities orientadas a ações de negócio, usando nomes `resource.action`. A lista é conceitual e normaliza exemplos que estavam em formato misto (por exemplo `view_unit`/`unit.read`). Não há permissão universal `manage_everything`.

## Condomínio e estrutura

- `condominium.read`, `condominium.update`
- `block.read`, `block.manage`
- `unit.read`, `unit.update`
- `common_area.read`, `common_area.manage`

## Pessoas e vínculos

- `person.read`, `person.update`
- `unit_ownership.read`, `unit_ownership.manage`
- `unit_residency.read`, `unit_residency.manage`
- `unit_tenancy.read`, `unit_tenancy.manage`
- `staff.read`, `staff.manage`

`person.read` não significa leitura irrestrita de todos os campos de toda Person. Finalidade, relação com unidade, tenant e política de dados continuam necessários.

## Reserva e visitas

- `reservation.read`, `reservation.create`, `reservation.update`, `reservation.cancel`, `reservation.approve`
- `event.read`, `event.manage`
- `visit.read`, `visit.register`

## Acesso

- `access_authorization.read`, `access_authorization.create`, `access_authorization.revoke`
- `access_event.read`, `access_event.register`, `access_event.correct`

`access_authorization.*` opera a autorização no sistema; não concede a própria permissão ao visitante. `access_event.register` registra um fato observado, não cria autorização.

## Pacotes

- `package.read`, `package.register`, `package.notify`, `package.confirm`, `package.release`, `package.correct`, `package.history.read`

`package.*` é permissão de ator interno/usuário para operar ou consultar fluxo. `PackageEvent` é o registro do fato; não é permission.

## Assembleias

- `assembly.read`, `assembly.manage`, `assembly.open`, `assembly.close`
- `participant.read`, `participant.manage`, `participant.presence.register`
- `voting_eligibility.read`, `voting_eligibility.determine`
- `proxy.read`, `proxy.register`, `proxy.revoke`
- `vote.participate`, `vote.register`, `vote.correct`, `vote.cancel`, `vote.result.read`
- `quorum.read`, `assembly.result.read`

`vote.register` autoriza a ação de registrar conforme procedimento, mas não determina elegibilidade jurídica. A participação do morador também pode ser representada por `vote.participate`; mapeamento final entre essas capabilities e ações ainda requer aprovação.

## Notificações e modules

- `notification.read`, `notification.create`, `notification.delivery.read`, `notification.channel.configure`
- `module.read`, `module.enable`, `module.disable`
- `feature.read`, `feature.enable`, `feature.disable`

Canal específico não é permission geral. `NotificationDelivery` representa tentativa, canal e resultado.

## Role assignments, auditoria, financeiro e plataforma

- `role_assignment.read`, `role_assignment.grant`, `role_assignment.suspend`, `role_assignment.reactivate`, `role_assignment.revoke`
- `audit_log.read`
- `finance.read`, `finance.income.manage`, `finance.expense.manage`, `finance.supplier.manage`, `finance.report.read`, `finance.document.export`
- `platform.condominium.manage`, `platform.cross_tenant.access`

`platform.cross_tenant.access` deve ser granular por finalidade/escopo aprovado e não substitui permission do recurso, nem concede acesso operacional contínuo.

## Não são permissions

- `Role`, `RoleAssignment` e `Scope` são relações/termos de autorização.
- Elegibilidade, presença, autorização de visitante, estado da Feature e disponibilidade de área são condições de negócio/contexto.
- `ENTRY`, `EXIT`, `PackageEvent` e `NotificationDelivery` são fatos/resultados operacionais.

## Sensibilidade

Operations elevadas/críticas e leitura de campos sensíveis exigem scope mínimo e auditoria conforme políticas em [data-visibility.md](./data-visibility.md) e [../../BUSINESS-RULES.md](../../BUSINESS-RULES.md).
