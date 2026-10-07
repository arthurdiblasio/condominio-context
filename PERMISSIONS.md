# PERMISSIONS.md

## Objetivo e status

Índice e visão consolidada do modelo conceitual de autorização. Define capacidades e limites de domínio sem prescrever autenticação, armazenamento, middleware, API ou mecanismo técnico.

Status: `PERMISSIONS_REQUIRES_HUMAN_REVIEW`

## Modelo fundamental

Uma decisão conceitual de autorização responde:

> Subject X pode executar a ação Y sobre o recurso Z neste contexto?

```text
Subject
+ RoleAssignment
+ Permission
+ Scope
+ Resource
+ Context
= Authorization Decision
```

O sujeito autenticado é `UserAccount` associado a `Person`. `Person` sem conta não executa automaticamente ações autenticadas. Papel não concede ação por si só; permissão não concede escopo por si só.

## Ordem de avaliação conceitual

1. Identificar `UserAccount` e `Person` associados; verificar se a conta está ativa segundo política aprovada.
2. Resolver explicitamente o condomínio/contexto da operação quando o recurso for tenant-scoped.
3. Identificar recurso e seu tenant; negar incompatibilidade ou ambiguidade de contexto.
4. Encontrar `RoleAssignment` válido para a pessoa, escopo, vigência e contexto exatos.
5. Verificar se o papel concede a `Permission` específica para a ação.
6. Verificar se o `Scope` cobre o recurso por regra de abrangência explícita; não presumir herança.
7. Verificar se a feature/módulo está habilitada quando exigida.
8. Verificar condições de negócio, vínculo com unidade, elegibilidade, período e restrições configuráveis/legais.
9. Aplicar decisão de conflito conforme política aprovada. Até lá, conflito não resulta em concessão implícita.
10. Autorizar somente se todos os requisitos aplicáveis forem satisfeitos; ações elevadas/críticas devem ser auditadas.

Este é um modelo conceitual, não um algoritmo, ordem de execução técnica ou decisão final sobre conflitos.

O [API Contract conceitual](./API-CONTRACT.md) aplica estes mesmos gates a commands e queries; seus permission mappings continuam sujeitos às decisões e não constituem grants efetivos.

## Menor privilégio e invariantes

- Identidade no sistema não concede acesso por si só.
- Ter um papel não concede automaticamente todas as operações do domínio.
- `owner`, `resident` e `tenant` não são papéis administrativos por implicação.
- Cada concessão é contextual a pessoa, papel, permission, scope, recurso e tenant.
- Um papel em Condominium A não concede acesso a Condominium B.
- `platform_admin` não recebe automaticamente permissões operacionais em cada tenant.
- A Feature habilitada permite disponibilidade da capacidade, não concede permission.
- Vínculos de pessoa e unidade não são criados por grants de papel.
- Ausência de grant aplicável ou contexto ambíguo deve resultar em negação segundo o modelo recomendado; confirmar política de conflito permanece decisão humana.

## Scopes

| Scope | Significado | Recursos abrangíveis | Herança |
|---|---|---|---|
| `PLATFORM` | Governança do produto/plataforma SaaS. | Recursos globais da plataforma; acesso a tenant somente por capacidade cross-tenant explicitamente concedida e justificada. | Nenhuma concessão operacional de condomínio é automática. |
| `CONDOMINIUM` | Um condomínio específico. | O próprio condomínio e recursos pertencentes ao tenant quando a permission declarar que sua abrangência inclui recursos descendentes. | Não automática; cobertura descendente deve estar definida por permission. |
| `BLOCK` | Um bloco específico de um condomínio. | Bloco e recursos nele contidos quando a permission permitir. | Não alcança outros blocos nem recursos sem relação declarada. |
| `UNIT` | Uma unidade específica e vínculos de unidade pertinentes. | Unidade e dados associados explicitamente cobertos pela permission. | Não alcança outras unidades nem todo o condomínio. |
| `FEATURE` | Uma capacidade/módulo habilitado para um condomínio. | A operação específica daquela Feature e seus recursos autorizados. | Habilitação não herda permissions; o grant e o tenant continuam obrigatórios. |

O recurso conserva seu próprio contexto de condomínio. Scope do papel é o limite do grant; escopo do recurso é onde o objeto realmente pertence. O grant só alcança recurso se o scope for compatível e a permission declarar cobertura. Herança automática de scope não existe no modelo-base. Outros scopes (por exemplo Assembly, AgendaItem, access point) não são adicionados nesta etapa; se uma permissão exigir tal granularidade, submeter a decisão de domínio.

## Taxonomia de permissões

Nomes abaixo são capacidades conceituais, em taxonomia orientada à operação, não lista de grants default.

### Condomínio e estrutura

- `condominium.read`, `condominium.update`
- `block.read`, `block.manage`
- `unit.read`, `unit.update`
- `common_area.read`, `common_area.manage`

### Pessoas e vínculos

- `person.read`, `person.update`
- `unit_ownership.read`, `unit_ownership.manage`
- `unit_residency.read`, `unit_residency.manage`
- `unit_tenancy.read`, `unit_tenancy.manage`
- `staff.read`, `staff.manage`

### Veículos e pets

- `vehicle.read`, `vehicle.manage`
- `pet.read`, `pet.manage`

### Reservas e eventos

- `reservation.read`, `reservation.create`, `reservation.update`, `reservation.cancel`, `reservation.approve`
- `event.read`, `event.manage`
- `visit.read`, `visit.register`

### Acesso

- `access_authorization.read`, `access_authorization.create`, `access_authorization.revoke`
- `access_event.read`, `access_event.register`, `access_event.correct`

### Pacotes

- `package.read`, `package.register`, `package.notify`, `package.confirm`, `package.release`, `package.correct`, `package.history.read`

### Assembleias

- `assembly.read`, `assembly.manage`, `assembly.open`, `assembly.close`
- `participant.read`, `participant.manage`, `participant.presence.register`
- `voting_eligibility.read`, `voting_eligibility.determine`
- `proxy.read`, `proxy.register`, `proxy.revoke`
- `vote.participate`, `vote.register`, `vote.correct`, `vote.cancel`, `vote.result.read`
- `quorum.read`, `assembly.result.read`

### Notificações

- `notification.read`, `notification.create`, `notification.delivery.read`
- `notification.channel.configure`

Não usar permissão chamada `send_whatsapp` como capacidade central: envio é uma entrega por canal e deve obedecer `notification.create`/canal habilitado e regra aplicável.

### Papéis, módulos, financeiro e auditoria

- `role_assignment.read`, `role_assignment.grant`, `role_assignment.suspend`, `role_assignment.reactivate`, `role_assignment.revoke`
- `module.read`, `module.enable`, `module.disable`
- `feature.read`, `feature.enable`, `feature.disable`
- `finance.read`, `finance.income.manage`, `finance.expense.manage`, `finance.supplier.manage`, `finance.report.read`, `finance.document.export`
- `audit_log.read`
- `platform.condominium.manage` e `platform.cross_tenant.access` para capacidades de plataforma delimitadas; não representam acesso irrestrito.

Permissões de correção, voto, dados pessoais, papel e auditoria exigem classificação de sensibilidade e podem exigir procedimento adicional aprovado.

## Roles e capacidade típica

| Role | Natureza / escopo natural | Capacidades típicas (sempre sujeitas a grants) | Limitações / concessão |
|---|---|---|---|
| `platform_admin` | Administrativo da plataforma; `PLATFORM`. | Gerir catálogo global, ciclo comercial/plataforma e suporte global explicitamente autorizado. | Não recebe gestão operacional tenant-scoped, dados de residentes ou auditoria completa por padrão. Não concede grants fora de authority/delegation aprovadas. |
| `condominium_admin` | Administrativo tenant; `CONDOMINIUM`. | Estrutura, configurações locais, dados de pessoas/vínculos e gestão de atribuições conforme delegação. | Sem acesso a outros condomínios; financeiro, assembleia, dados sensíveis e grants privilegiados não são automáticos. |
| `property_manager` | Operacional/administrativo, contextual por condomínio. | Operar gestão delegada em cada condomínio com RoleAssignment individual. | Atuar em A/B/C não concede D; não recebe acesso global por nome do papel. |
| `syndic` | Governança administrativa do condomínio; `CONDOMINIUM`. | Gestão condominial e capacidades delegadas de reserva, assembleia, comunicação e contas. | Autoridade jurídica, financeira, acesso a dados e grants são delimitados; não é acesso ilimitado nem platform admin. |
| `assistant_syndic` | Suporte administrativo; escopo delegado no tenant. | Capacidades explicitamente delegadas, como leitura e apoio a workflows. | Não herda exatamente as permissions de syndic. Grant, revogação, delegabilidade e período precisam de política. |
| `doorman` | Operacional de portaria; `CONDOMINIUM` e/ou recursos atribuídos. | `visit.register`, `access_authorization.read`, `access_event.register`, `package.register`, `package.read`, e `package.release` somente se concedido. | Sem alteração de proprietário/vínculos, grants, permissões, financeiro ou configuração por padrão; dados limitados à operação. |
| `employee` | Operacional; escopo depende da função. | Capacidades operacionais explicitamente atribuídas. | Não é sinônimo de doorman nem recebe toda permissão de staff. |
| `owner` | Residencial/propriedade como classificação de acesso; `UNIT` contextual. | Leitura de informações permitidas de unidades às quais a pessoa está relacionada; participação em workflows quando elegível/permitido. | Papel não prova `UnitOwnership`, não implica residência, administração, finance.manage ou voto elegível. |
| `resident` | Residencial; `UNIT` contextual. | Leitura permitida de sua unidade, reserva, pacote e criação de autorização se configuração permitir. | Não pode alterar outros residentes, titularidade, papéis ou configurações por padrão. |
| `tenant` | Residencial/ocupação; `UNIT` contextual. | Capacidades relativas às unidades com `UnitTenancy` e grants/configuração aplicáveis. | Não é proprietário nem admin; direitos de reserva, pacote e assembleia dependem de política/elegibilidade. |

Todos os papéis podem coexistir em uma pessoa. A lista não define poderes por si só. Condições típicas estão detalhadas em [permission-matrix.md](./docs/permissions/permission-matrix.md) e [roles.md](./docs/permissions/roles.md).

## Grants, revogação e delegação

- `RoleAssignment` identifica pessoa, role, scope, tenant quando aplicável, vigência/condição de validade e estado.
- Conceder, alterar escopo, suspender, reativar ou revogar requer permission de grant no mesmo escopo ou uma autoridade explicitamente delegada.
- O concessor não concede autoridade maior que a própria, salvo delegação explícita, limitada e válida.
- A pessoa não deve se conceder privilégio por si mesma; qualquer exceção requer decisão de governança explícita e auditoria.
- Revogação, suspensão ou expiração impede novas autorizações com base nessa atribuição a partir do momento efetivo definido pela política; não apaga ações históricas.
- Atribuição expirada/revogada não deve ser reativada silenciosamente; nova atribuição ou reativação autorizada deve ficar auditada.
- Delegação operacional deve ser específica em scope, permissions e vigência e revogável.
- `Proxy` é representação no contexto de assembleia/ato autorizado, não `RoleAssignment`; não concede login nem poderes administrativos gerais.
- Quem pode conceder papel administrativo, administrar `platform_admin`, alterar scope, fazer autoatribuição, suspender e reativar permanece `OPEN DECISION` crítica.

## Conflitos entre múltiplos papéis

Modelo conceitual recomendado para revisão: avaliar grants aplicáveis em conjunto, mas autorizar somente quando os requisitos explícitos de todas as dimensões passam; não eleger silenciosamente um papel de maior prioridade. Como regra provisória de segurança, conflito ou ambiguidade não deve gerar allow implícito. Não está escolhida uma política final para `ALLOW`, `DENY`, `EXPLICIT_DENY` ou prioridade de papel.

Recomendação para decisão humana: política de *deny by default* e *explicit deny overrides allow*, sem `ROLE_PRIORITY`. É recomendação, não regra aprovada. Até aprovação, não interpretar ausência de grant como concessão nem resolver conflito de forma mais permissiva.

## Matriz e operações sensíveis

- [Permission matrix](./docs/permissions/permission-matrix.md): `ALLOW`, `DENY`, `CONDITIONAL`, `CONFIGURABLE`.
- [Responsabilidade e capacidade por papel](./docs/permissions/roles.md).
- [Visibilidade de dados pessoais](./docs/permissions/data-visibility.md).

Classificação:

- `NORMAL`: leitura/ação operacional rotineira restrita por scope e finalidade.
- `ELEVATED`: alteração de vínculos, acesso, autorizações, configuração ou operação com impacto em outras pessoas.
- `CRITICAL`: grants, dados de auditoria/sensíveis, alteração de voto/resultado e operação cross-tenant.

## Break-glass

Acesso excepcional de emergência não é concedido por este modelo nem necessário como regra comum já definida. Sua existência, atores, gatilhos, dados expostos, duração, revisão posterior e auditoria são `OPEN DECISION` futura de prioridade HIGH. Não criar papel universal “break-glass” ou exceção permanente.

## Decisões em aberto

Ver [OPEN-DECISIONS.md — PERMISSIONS / AUTHORIZATION](./OPEN-DECISIONS.md#permissions--authorization). Decisões técnicas de identidade e enforcement não fazem parte deste documento.

Lacunas de permissão por operação, inclusive `user_account.*`, rejeição/cancelamento onde não há capability específica e apuração/publicação de resultado, estão identificadas no [contrato conceitual](./docs/api/authorization.md); não devem ser preenchidas reutilizando permissões sem decisão.

## Documentos

- [authorization-model.md](./docs/permissions/authorization-model.md)
- [roles.md](./docs/permissions/roles.md)
- [permissions.md](./docs/permissions/permissions.md)
- [scopes.md](./docs/permissions/scopes.md)
- [role-assignment.md](./docs/permissions/role-assignment.md)
- [data-visibility.md](./docs/permissions/data-visibility.md)
- [permission-matrix.md](./docs/permissions/permission-matrix.md)
