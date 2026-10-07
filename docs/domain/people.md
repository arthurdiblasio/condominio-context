# Pessoas, contas e vínculos

## Conceitos

- `Person`: pessoa do domínio, pode existir sem conta e participar de vários condomínios.
- `User` / `UserAccount`: conta de acesso associada a Person; cardinalidade e processo de associação estão abertos.
- `Role`: definição de papel.
- `RoleAssignment`: atribuição de papel à pessoa em escopo contextual.
- `Permission`: autorização de ação; não é vínculo com unidade nem papel.

## Vínculos com unidade

- `UnitOwnership`: propriedade/titularidade.
- `UnitResidency`: residência.
- `UnitTenancy`: locação/ocupação.

Essas relações são distintas e podem coexistir quando os fatos assim indicarem; uma não implica a outra nem implica conta ou papel.

## Papéis de operação

`Syndic`, `PropertyManager`, `Doorman`, `Employee` e `Staff` descrevem funções/papéis em contexto. `Visitor` e `Guest` descrevem uma Person em visita/evento, não identidades independentes. Uma pessoa pode ter múltiplos vínculos e papéis em diferentes condomínios sem misturar seus dados tenant-scoped.

Ver as regras detalhadas em [identity.md](../business-rules/identity.md), [units.md](../business-rules/units.md) e [permissions.md](../business-rules/permissions.md).
