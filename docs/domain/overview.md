# Visão geral do domínio

Este documento resume o domínio. As regras verificáveis ficam consolidadas em [BUSINESS-RULES.md](../../BUSINESS-RULES.md) e detalhadas em [docs/business-rules/](../business-rules/).

## Contexto

A plataforma SaaS atende vários condomínios isolados. Cada operação tenant-scoped deve usar o contexto de condomínio correto; `Person` pode participar de vários sem misturar dados entre eles.

## Conceitos centrais

- Estrutura: `Platform`, `Condominium`, `Block`, `Unit`, `CommonArea`
- Identidade e autorização: `Person`, `UserAccount`, `Role`, `RoleAssignment`, `Permission`, `Scope`
- Vínculos com unidade: `UnitOwnership`, `UnitResidency`, `UnitTenancy`
- Operações: `Vehicle`, `Pet`, `Reservation`, `Event`, `Visit`, `AccessAuthorization`, `AccessEvent`, `Package`, `PackageEvent`
- Comunicação: `Notification`, `NotificationDelivery`
- Assembleia: `Assembly`, `Participant`, `AgendaItem`, `VotingEligibility`, `Proxy`, `Vote`, `Quorum`
- Capacidades: `Module`, `Feature`, `CondominiumModule`, `CondominiumFeature`
- Governança: `AuditLog`

## Relações principais

- `Platform` agrega `Condominium`; condomínio contextualiza blocos, unidades, áreas comuns e dados de operação.
- Pessoa e conta de acesso são distintas; conta não implica vínculo com unidade.
- Vínculos de propriedade, residência e locação são separados.
- Autorização de acesso não comprova entrada; notificação não comprova entrega.
- Presença, elegibilidade, representação, voto e resultado são conceitos distintos.

Ver [domain-model.md](./domain-model.md), [relationships.md](./relationships.md) e [responsibility-matrix.md](./responsibility-matrix.md).
