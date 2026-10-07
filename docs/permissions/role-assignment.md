# Regras de RoleAssignment

## Conteúdo conceitual mínimo

Um `RoleAssignment` precisa expressar:

- `Person` que recebe o papel;
- `Role`;
- `Scope` e recurso/contexto alvo;
- condomínio quando tenant-scoped;
- início/fim ou regra de vigência;
- estado e autoridade/concessor quando definidos pela política.

Não define formato de persistência.

## Regras

- RoleAssignment liga papel à Person; associação a UserAccount é condição separada para agir de forma autenticada.
- Assignment tenant-scoped referencia um único condomínio coerente com block/unit/feature do scope.
- Conceder, alterar scope, suspender, reativar ou revogar exige permission/grant explícito do ator no mesmo limite de autoridade.
- Um ator não pode conceder permission ou scope superior à sua autoridade, salvo delegação explícita, válida, delimitada e auditada.
- Autoconcessão de privilégio não é permitida pelo modelo-base; exceções de governança permanecem abertas.
- Assignment não se transfere automaticamente com mudança de unidade, contratação, eleição, associação de conta ou presença em outro tenant.
- Fim de vigência impede novas decisões autorizadas baseadas no assignment a partir do instante efetivo definido pela política.
- Revogação e suspensão impedem uso futuro do grant no escopo afetado; não apagam ações históricas nem reescrevem autoria.
- Suspender um assignment não suspende a Person ou a UserAccount globalmente.
- Ativar/reativar não restaura outros grants expirados ou revogados sem decisão explícita.
- Toda mutação crítica do assignment deve ser auditada; consulta de assignment também exige scope.

## Delegação versus Proxy

Delegação de autoridade permite a um concedente autorizado transferir um subconjunto explícito de permissions para outro RoleAssignment, limitada por escopo e vigência. `Proxy` representa uma pessoa em assembleia/ato jurídico delimitado; não habilita acesso ao produto, não cria RoleAssignment e não delega gestão administrativa.

## Ciclo candidato

Ver [state machine de RoleAssignment](../state-machines/identity-and-reservations.md#roleassignment). `ACTIVE`, `SUSPENDED`, `REVOKED` e `EXPIRED` têm efeitos distintos; `PENDING` só se uma regra aprovada exigir grant futuro ou validação antes da efetividade. `SUSPENDED` pode voltar a `ACTIVE` com authority apropriada; `REVOKED` e `EXPIRED` são terminais no ciclo normal. Vigência, pausa e efetividade temporal ainda dependem de governança.

## Decisões críticas

Quem pode conceder/revogar cada papel; quem administra `platform_admin`; delegação transitiva; autoatribuição; duração e término; suspensão e recurso; grants a pessoas sem conta; grants de autoridade superior. Ver seção correspondente em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
