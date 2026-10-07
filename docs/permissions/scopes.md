# Scopes de autorização

## Scope do RoleAssignment e localização do resource

Scope define onde um grant pode ser aplicado. Resource possui sua própria localização, tenant e relações. A decisão exige compatibilidade entre ambos. Um resource não muda de tenant por ser acessado por alguém com escopo platform ou condomínio diferente.

| Scope | Fronteira | Cobertura possível | Herança |
|---|---|---|---|
| `PLATFORM` | Produto/plataforma. | Operações globais expressamente permissionadas; cross-tenant só por grant específico e finalidade. | Não herda permissions operacionais de tenant. |
| `CONDOMINIUM` | Um Condominium. | Condomínio e recursos pertencentes ao tenant conforme definição de abrangência da permission. | Sem herança silenciosa; cobertura descendente é explícita por capability. |
| `BLOCK` | Um Block dentro de um Condominium. | Bloco e recursos nele contidos quando capability diz que se aplica. | Não abrange outros blocos/unidades sem relação explícita. |
| `UNIT` | Uma Unit específica. | Unidade e informação associada autorizada. | Não abrange outras unidades nem todo condomínio. |
| `FEATURE` | Uma feature habilitada em um condomínio. | Ações no contexto da capability específica. | Habilitação não herda nem cria permissions para papéis. |

## Outros scopes

`Assembly`, `AgendaItem`, `AccessPoint` ou `Package` não são adicionados como scopes gerais nesta etapa. Algumas actions têm condições/resource identifiers nesses contextos, mas isso não exige um novo scope de papel. Se surgir uma necessidade de delegação própria de sub-recurso, documentar decisão e evitar scope ad hoc silencioso.

## Regras

- Todo scope block/unit pertence a exatamente um condomínio contextual.
- Scope atribuído deve ser compatível com recurso e permission; caso contrário negar.
- Grant de Condominium não automaticamente delega grant de Block/Unit ou o contrário.
- Uma permission pode declarar operação sobre recursos descendentes do tenant, mas não altera nem amplia o RoleAssignment.
- Scope PLATFORM não é sinônimo de acesso a todas as informações/ações em todos tenants.
- Scope FEATURE depende da habilitação e do condomínio correspondente; não substitui permission funcional.

## Exemplo

Person A possui `RoleAssignment(syndic, Condominium A)`. Se a capability e cobertura aprovada incluírem dados condominiais descendentes, poderá agir em Unit 101/202 e Assembly X que pertençam a A. Não poderá agir em Unit 301 de Condominium B sem outro assignment/grant compatível. A identidade comum não altera essa fronteira.

## Decisão aberta

Definir o modelo de abrangência de capabilities para recursos descendentes, suporte de grants multi-scope e permissões de plataforma que necessitam acesso cross-tenant.
