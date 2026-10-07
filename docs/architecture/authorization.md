# Fronteira arquitetural de autorização

## Três decisões diferentes

| Dimensão | Pergunta | Responsabilidade |
|---|---|---|
| Authentication | Quem apresenta a identidade e qual evidência é válida? | Adaptador de entrada e serviço/provedor de identidade futuro; mecanismo não escolhido. |
| Authorization | Este sujeito possui Permission, Scope, grant e acesso a este Resource/tenant? | Application, usando catálogo e políticas de autorização aprovados. |
| Business rule | Mesmo autorizado, a operação é válida agora? | Domain e caso de uso, segundo estado, período e condições de negócio. |

Passar uma dimensão não satisfaz as outras. Autenticação técnica não prova papel ou vínculo; papel não prova elegibilidade; Permission não contorna invariantes.

## Ponto de enforcement

Application é a fronteira de autoridade dos casos de uso e deve exigir authorization decision antes de executar a transição. Interface pode autenticar e bloquear cedo por razões operacionais, mas não é a única fonte de enforcement: todos os caminhos que iniciem o caso de uso, inclusive tarefas, devem aplicar as mesmas regras.

Para cada ação, conferir conceitualmente:

`UserAccount + Person + RoleAssignment + Permission + Scope + Resource + TenantContext + condições`

A matriz existente é proposta; conflitos e capacidades ausentes permanecem em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md) e [API authorization](../api/authorization.md).

## Permission check vs resource access check

1. **Permission check:** existe capability para a ação?
2. **Scope check:** o grant cobre o scope requerido segundo abrangência aprovada?
3. **Resource access check:** o recurso e suas referências pertencem ao tenant e fronteira autorizados?
4. **Business validation:** estado e guardas atuais permitem a ação?

Exemplo: `reservation.approve` no papel de syndic não autoriza aprovar uma Reservation de qualquer condomínio. Reservation, CommonArea, Unit e tenant devem ser coerentes e estar dentro do scope do grant.

## Query e dados pessoais

Consultas também exigem tenant, Permission, scope, finalidade e visibilidade por dado/campo. Não retornar coleções cross-tenant com filtro tardio. A ausência de acesso deve evitar revelar existência ou conteúdo de recurso alheio. Dados da portaria e de assembleia têm exposição minimizada segundo PRIVACY rules.

## Serviço versus política

Application pode coordenar uma decisão central de autorização, mas não deve duplicar a taxonomia de roles/permissions em cada controller ou adapter. Domain mantém regras de elegibilidade e validade do negócio; não consulta infraestrutura de identidade/autorização.

`platform.cross_tenant.access` não remove a exigência de permission do recurso, finalidade, alvo explícito e auditoria. Não há bypass implícito de suporte ou papel administrativo global.
