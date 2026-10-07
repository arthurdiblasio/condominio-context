# Regras de papéis, atribuições e permissões

## Modelo

`Role` define um papel; `RoleAssignment` atribui o papel a uma `Person` num `Scope`; `Permission` expressa ação autorizável. Uma conta autenticada pode atuar em nome da pessoa associada, mas não é dona do papel.

## Regras

- `PERMISSION-01` Ação restrita requer permissão para a ação e recurso no escopo atual.
- `PERMISSION-02` A `RoleAssignment` deve identificar pessoa, papel e escopo válido; atribuição tenant-scoped inclui o condomínio correspondente.
- `PERMISSION-03` A mesma pessoa pode receber papéis diferentes em condomínios distintos; não há herança lateral entre tenants.
- `PERMISSION-04` Atribuir papel não cria vínculo com unidade nem altera `UnitOwnership`, `UnitResidency` ou `UnitTenancy`.
- `PERMISSION-05` Concessor não pode conceder autoridade superior à que possui, salvo delegação explícita, válida e dentro do seu escopo.
- `PERMISSION-06` Suspender ou revogar atribuição impede seu uso futuro no escopo afetado sem apagar auditoria nem invalidar automaticamente ações passadas.
- `PERMISSION-07` Permissão sobre recurso não implica acesso a dados além dos necessários à ação e escopo.
- `PERMISSION-08` Permissão concedida em escopo mais amplo não é presumida em escopo menor/maior; relação de herança deve estar declarada e aprovada.
- `PERMISSION-09` Conflito entre permissões, múltiplos papéis, negação explícita e precedência não são resolvidos por suposição.

## Autoridades e ciclos

Podem conceder, suspender ou revogar somente pessoas com permissão explícita para administrar atribuições no escopo em questão. Os papéis candidatos e exemplos da matriz não são grants automáticos. Quem pode administrar atribuição de `platform_admin`, papéis de tenant e autoatribuição precisa de aprovação de governança.

Vigência pode limitar atribuição; não são definidos duração padrão, estados (`pending`, `active`, `suspended`, `revoked`, `expired`) ou renovação até que haja decisão. Mudanças devem ser auditadas.

## Escopos

`platform`, `condominium`, `block`, `unit`, `feature`. O escopo escolhido deve conter o recurso alvo. O limite entre papel vinculado à pessoa e autenticação por UserAccount deve ser preservado; uma operação autenticada precisa de associação válida com Person.

Ver [PERMISSIONS.md](../../PERMISSIONS.md) e [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
