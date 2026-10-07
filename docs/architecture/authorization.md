# Fronteira arquitetural de autorização

## Três decisões diferentes

| Dimensão | Pergunta | Responsabilidade |
|---|---|---|
| Authentication | Quem apresenta a identidade e qual evidência é válida? | Adaptador de entrada e serviço/provedor de identidade futuro; mecanismo não escolhido. |
| Authorization | Este sujeito possui Permission, Scope, grant e acesso a este Resource/tenant? | Application, usando catálogo e políticas de autorização aprovados. |
| Business rule | Mesmo autorizado, a operação é válida agora? | Domain e caso de uso, segundo estado, período e condições de negócio. |

Passar uma dimensão não satisfaz as outras. Autenticação técnica não prova papel ou vínculo; papel não prova elegibilidade; Permission não contorna invariantes.

## Ponto de enforcement

Application é a fronteira de autoridade dos casos de uso e deve exigir authorization decision antes de executar qualquer efeito. Interface pode autenticar e bloquear cedo por razões operacionais, mas não é a única fonte de enforcement: requests, jobs, ações agendadas, event handlers/consumers, processos de serviço, operações administrativas e suporte assistido passam pela mesma fronteira. Não há caminho de escrita que acesse Domain/repository ignorando o use case.

Para cada ação tenant-scoped, conferir conceitualmente:

`Actor + Subject + RoleAssignment? + Permission? + Scope? + Resource + Validated TenantContext + Purpose + condições`

Interrogações indicam que um actor de serviço/sistema pode ter authority limitada sem UserAccount/Person humana; a forma concreta de concessão continua decisão técnica/de governança, não implica acesso. Para ação humana, associação UserAccount-Person, RoleAssignment, Permission, Scope, conflito e authority seguem a matriz e as decisões abertas. Operação global exige classificação Platform-scoped e authority correspondente, nunca ausência de tenant por conveniência.

A matriz existente é proposta; conflito, capabilities ausentes e authority de actors não humanos permanecem em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md), [PERMISSIONS.md](../../PERMISSIONS.md) e [API authorization](../api/authorization.md). Uma lacuna significa que a operação não pode ser exposta/executada, não que o sistema cria um grant padrão.

## Modelo de actor para automação e assistência

- **Human actor:** conservar identidade autenticada, associação a Person quando resolvida e grants correntes.
- **System actor:** processo interno nomeado que executa apenas uma regra automática explicitamente definida; o rótulo `system` nunca contorna authorization.
- **Service actor:** integração com identidade própria e capability delegada, delimitada por ação, tenant e purpose; não reutiliza credenciais ou Person de solicitante.
- **Assisted action:** auditar o operador que iniciou, o sujeito/alvo afetado, a representação e o purpose separadamente. Não atribuir a ação à pessoa afetada como se ela a tivesse executado.
- **Assembly Proxy:** vale somente para sua representação de assembleia e escopo validado; não autoriza suporte nem jobs.

### Trabalho assíncrono

O producer persiste contexto mínimo e confiável: tenant/classificação Platform, actor/origin, purpose, resource, tipo de operação e correlação. O consumer trata campos como dados sujeitos a validação, não como credencial. Na execução:

1. verificar integridade/origem do envelope por mecanismo ainda não escolhido;
2. revalidar tenant/resource/state/validity e permissões atuais para uma nova decisão;
3. para consequência previamente autorizada, limitar a execução exatamente à intenção persistida e revalidar que ela continua válida/não cancelada;
4. negar se a authority foi revogada quando a ação ainda é uma nova decisão; não usar permissão histórica/actor original para ampliar efeitos;
5. registrar actor de serviço/sistema e resultado em audit conforme requisito; retries usam a mesma identidade de operação e são idempotentes;
6. não executar quando purpose, tenant ou actor estejam ausentes/ambíguos.

Cada job deve ser classificado como **consequência autorizada** ou **nova decisão de negócio**. A classificação depende do domínio/policy e não é definida universalmente aqui.

## Permission check vs resource access check

1. **Permission check:** existe capability para a ação?
2. **Scope check:** o grant cobre o scope requerido segundo abrangência aprovada?
3. **Resource access check:** o recurso e suas referências pertencem ao tenant e fronteira autorizados?
4. **Business validation:** estado e guardas atuais permitem a ação?

Exemplo: `reservation.approve` no papel de syndic não autoriza aprovar uma Reservation de qualquer condomínio. Reservation, CommonArea, Unit e tenant devem ser coerentes e estar dentro do scope do grant.

## Query e dados pessoais

Consultas também exigem tenant, Permission, scope, finalidade e visibilidade por dado/campo. Não retornar coleções cross-tenant com filtro tardio. Ausência de acesso deve evitar revelar existência ou conteúdo de recurso alheio. `TENANT_MISMATCH` pode ser usado internamente em diagnóstico auditado; a resposta externa deve ser indistinguível de não encontrado/não visível por padrão. Expor motivo diferente requer decisão de segurança/privacy; não define HTTP status. Dados da portaria e de assembleia têm exposição minimizada segundo PRIVACY rules.

## Serviço versus política

Application pode coordenar uma decisão central de autorização, mas não deve duplicar a taxonomia de roles/permissions em cada controller ou adapter. Domain mantém regras de elegibilidade e validade do negócio; não consulta infraestrutura de identidade/autorização.

`platform.cross_tenant.access` não remove a exigência de permission do recurso, finalidade, alvo explícito e auditoria. Não há bypass implícito de suporte ou papel administrativo global.
