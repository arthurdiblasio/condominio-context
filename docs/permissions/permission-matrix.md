# Matriz de capacidades por papel

## Como ler

Esta é uma matriz conceitual inicial, não grant efetivo. Toda célula `ALLOW` ainda exige conta ativa (em ação autenticada), RoleAssignment, Scope/resource compatíveis, tenant correto e condições de negócio. `CONDITIONAL` depende de vínculo, estado ou elegibilidade. `CONFIGURABLE` requer grant explícito por governança/localidade. `DENY` é ausência de capacidade padrão; exceção exige decisão e grant explícitos.

Abreviações: `PA` platform_admin; `CA` condominium_admin; `PM` property_manager; `SY` syndic; `AS` assistant_syndic; `DO` doorman; `EM` employee; `OW` owner; `RE` resident; `TE` tenant.

## Matriz Role × Permission × Scope

Na matriz de capacidades abaixo, cada permission usa o scope correspondente nesta tabela. O status por papel e o scope formam a combinação Role × Permission × Scope; ambos ainda se sujeitam a resource, tenant e condições. `CONDOMINIUM(descendants explicit)` só alcança recursos descendentes se a permission definir cobertura.

| Permissions / capacidade | Scope conceitual | Condição de resource |
|---|---|---|
| `platform.condominium.manage` | `PLATFORM` | Só o ciclo/configuração de tenants pela plataforma. |
| `platform.cross_tenant.access` | `PLATFORM` + target tenant | Acesso excepcional e permission explícita sobre cada tipo de dado/finalidade. |
| `condominium.read/update` | `CONDOMINIUM` | O condomínio alvo, sem acesso lateral. |
| `unit.read`, `unit_ownership.*`, `unit_residency.*`, `unit_tenancy.*` | `UNIT` ou `CONDOMINIUM(descendants explicit)` | Unidade e vínculo no tenant correspondente. |
| `reservation.*`, `event.*`, `visit.*` | `UNIT`, `BLOCK` ou `CONDOMINIUM(descendants explicit)` | Recurso pertence ao escopo; papel residencial requer vínculo quando aplicável. |
| `access_authorization.*`, `access_event.*` | `CONDOMINIUM`, `BLOCK` ou `UNIT` | Autorização/ocorrência no local e tenant designados. |
| `package.*` | `UNIT` ou `CONDOMINIUM(descendants explicit)` | Pacote do destinatário/unidade; leitura limitada à finalidade. |
| `assembly.*`, `participant.*`, `voting_eligibility.*`, `proxy.*`, `vote.*`, `quorum.*` | `CONDOMINIUM` e contexto `Assembly/AgendaItem` do resource | Contexto de assembleia/pauta é condição de resource, não herança automática do assignment. |
| `notification.*` | `CONDOMINIUM` ou `FEATURE` | Destinatário e finalidade no tenant; cada entrega mantém canal próprio. |
| `module.*`, `feature.*` | `PLATFORM` para catálogo; `CONDOMINIUM`/`FEATURE` para habilitação local | Habilitação por tenant não concede permission aos usuários. |
| `finance.*` | `FEATURE` + `CONDOMINIUM` | Módulo habilitado, recurso financeiro do tenant e permission específica. |
| `role_assignment.*` | Scope do assignment alvo, limitado ao scope do concessor | Concessor só pode atuar dentro de authority/delegation validada. |
| `audit_log.read` | Mesmo tenant/scope do registro ou `PLATFORM` por grant específico | Não implica leitura de todos os tenants nem todos os campos sensíveis. |

| Permission / capacidade | PA | CA | PM | SY | AS | DO | EM | OW | RE | TE |
|---|---|---|---|---|---|---|---|---|---|---|
| `platform.condominium.manage` (PLATFORM) | ALLOW | DENY | DENY | DENY | DENY | DENY | DENY | DENY | DENY | DENY |
| `condominium.read` | CONFIGURABLE | ALLOW | CONDITIONAL | ALLOW | CONFIGURABLE | CONDITIONAL | CONFIGURABLE | DENY | DENY | DENY |
| `condominium.update` | CONFIGURABLE | ALLOW | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `unit.read` | CONFIGURABLE | ALLOW | CONDITIONAL | CONDITIONAL | CONFIGURABLE | CONDITIONAL | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `unit_ownership.manage` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `unit_residency.manage` / `unit_tenancy.manage` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `reservation.read` | DENY | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONFIGURABLE | DENY | DENY | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `reservation.create` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `reservation.approve` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY |
| `visit.register` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONDITIONAL | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `access_authorization.read` | DENY | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONFIGURABLE | CONDITIONAL | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `access_authorization.create` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `access_authorization.revoke` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `access_event.register` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONDITIONAL | CONDITIONAL | DENY | DENY | DENY |
| `access_event.correct` | DENY | DENY | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `package.register` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONDITIONAL | CONDITIONAL | DENY | DENY | DENY |
| `package.read` | DENY | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `package.confirm` / `package.release` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONDITIONAL | CONDITIONAL | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `package.correct` | DENY | DENY | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `assembly.manage` / `assembly.open` / `assembly.close` | DENY | CONFIGURABLE | DENY | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY |
| `participant.presence.register` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY |
| `voting_eligibility.determine` | DENY | CONFIGURABLE | DENY | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY |
| `proxy.register` | DENY | CONFIGURABLE | DENY | CONFIGURABLE | CONFIGURABLE | DENY | DENY | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `vote.participate` / `vote.register` | DENY | DENY | DENY | DENY | DENY | DENY | DENY | CONDITIONAL | CONDITIONAL | CONDITIONAL |
| `vote.correct` / `vote.cancel` | DENY | DENY | DENY | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `module.enable/disable`, `feature.enable/disable` | CONFIGURABLE | CONFIGURABLE | DENY | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `finance.read`, `finance.*.manage` | DENY | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY |
| `audit_log.read` | CONFIGURABLE | CONFIGURABLE | DENY | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY |
| `role_assignment.grant/revoke` | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY |
| `platform.cross_tenant.access` | CONFIGURABLE | DENY | DENY | DENY | DENY | DENY | DENY | DENY | DENY | DENY |

## Condições importantes

- `unit.read` para OW/RE/TE requer vínculo válido com a unidade e limite de campo/finalidade; não concede leitura de todas as units.
- PM precisa de assignment separado por cada condomínio em que atua.
- Doorman/employee só podem registrar fatos no tenant/ponto operacional e dados estritamente necessários.
- Voto depende de elegibilidade por AgendaItem; authorization matrix não decide direito legal.
- `finance.*` exige Feature/Módulo habilitado, scope financeiro apropriado e grant específico.
- `role_assignment.grant/revoke` é `CONFIGURABLE` somente para explicitar que governança precisa atribuir capability; não quer dizer que todo papel listado recebe o grant.
- Platform support e auditoria não conferem acesso universal a dados residenciais.
- `CONDOMINIUM(descendants explicit)` descreve cobertura potencial da permission, não herança automática de papéis ou RoleAssignments.

## Operações sensíveis

| Nível | Exemplos | Controle conceitual |
|---|---|---|
| `NORMAL` | Leitura de dados não sensíveis no scope; criar reserva própria; registrar pacote na tarefa de portaria. | Grant restrito a finalidade, vínculo e tenant. |
| `ELEVATED` | Alterar vínculo residencial/propriedade; criar/revogar autorização; liberar pacote; alterar configuração; consultar informação pessoal não pública. | Permission distinta, scope mínimo, condição de negócio e AuditLog. |
| `CRITICAL` | Grant/revoke de papel privilegiado; leitura de audit log/dados sensíveis; corrigir/cancelar voto ou resultado; acesso cross-tenant. | Governança explícita, separação de escopo, auditoria detalhada; eventual aprovação dupla é decisão aberta. |

## Limites

A matriz representa uma baseline proposta para revisão, não distribuição aprovada de grants. Não resolve explicit deny, conflitos entre papéis, limites de delegação, campos individuais nem requisitos jurídicos. Negação em uma célula não pode ser sobreposta automaticamente por outro role até uma política de conflitos ser aprovada.
