# Arquitetura conceitual do backend

## Objetivo e status

Esta documentação propõe a arquitetura do futuro `condominio-api`, derivada do domínio, das regras, permissões, workflows, state machines e do [API Contract](../../API-CONTRACT.md). Não descreve uma aplicação existente e não autoriza implementação.

**Status:** `ARCHITECTURE_REMEDIATION_REQUIRES_REVIEW`

O objetivo é manter regras e identidade de domínio independentes de Gin, GORM, PostgreSQL, HTTP e provedores externos. A stack informada para a implementação futura é Go, Gin, GORM e PostgreSQL; a forma de uso e os detalhes arquiteturais ainda precisam de validação humana.

Fontes de verdade: [DOMAIN.md](../../DOMAIN.md), [BUSINESS-RULES.md](../../BUSINESS-RULES.md), [PERMISSIONS.md](../../PERMISSIONS.md), [WORKFLOWS.md](../../WORKFLOWS.md), [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md), [modelo estrutural](../domain/domain-model.md), [regras detalhadas](../business-rules/), [autorização](../permissions/), [workflows](../workflows/), [state machines](../state-machines/README.md) e [API Contract](../../API-CONTRACT.md).

## Proposta

- Começar como **monólito modular**, com módulos de domínio e casos de uso separados por responsabilidade.
- Aplicar dependência em direção a Application e Domain; Infrastructure implementa portas requeridas pelos casos de uso.
- Tratar o tenant e a decisão de autorização como contexto obrigatório e verificável para cada operação tenant-scoped.
- Fazer do TenantContext validado requisito do contrato de persistência tenant-scoped; ausência/mismatch falha fechada.
- Conduzir toda operação com efeito por Application authorization boundary, seja humana, administrativa, assistida ou automática.
- Usar comandos semânticos por workflow e consultas de leitura sem efeitos de negócio; não assumir CQRS completo.
- Delimitar transações pela consistência necessária a cada caso de uso, não por endpoint ou por conjunto de tabelas.
- Confirmar fato/estado sensível e AuditLog obrigatório na mesma unidade de consistência.
- Manter integrações, relógio, IDs, armazenamento, persistência e mensageria atrás de fronteiras substituíveis.
- Adiar microsserviços, broker, cache distribuído e mecanismo de busca especializado até existir necessidade demonstrada.

Estas são propostas arquiteturais sujeitas à revisão; não alteram as regras de produto nem resolvem decisões abertas de domínio.

## Stack de referência

| Tecnologia informada | Fronteira conceitual | Limite |
|---|---|---|
| Go | Linguagem da aplicação futura. | Tipos e pacotes não devem forçar detalhes de persistência ao Domain. |
| Gin | Adaptador de entrada HTTP da camada Interface/API. | Não contém autorização completa, orquestração nem regras de negócio. |
| GORM | Adaptador de persistência em Infrastructure. | Modelos GORM não são automaticamente entidades de domínio. |
| PostgreSQL | Persistência relacional em Infrastructure. | Nenhum schema, migration, lock ou política RLS é especificado aqui. |

O uso da stack não define transportes futuros para eventos, storage, autenticação, cache ou tarefas agendadas.

## Fronteiras de confiança

1. A interface recebe dados externos não confiáveis.
2. Authentication identifica UserAccount; associação a Person é uma relação de domínio separada e pode depender de OD-03.
3. Tenant solicitado é candidate; Application valida contra identity/grant e só então cria contexto validado.
4. Authorization e access-to-resource são gates obrigatórios antes do efeito.
5. Domain aplica invariantes e transições de negócio, sem confiar em autorização implícita.
6. Infrastructure exige tenant em cada operação scoped, persiste fato e audit obrigatório atomicamente e não decide policy.
7. Jobs/handlers usam actor/processo explícito, purpose e tenant; nunca recebem bypass por serem automáticos.
8. Integrações externas só recebem dados autorizados e mínimos para sua finalidade.

## Documentos

- [Camadas e dependências](./layers.md)
- [Grafo de dependências](./dependencies.md)
- [Módulos e fronteiras](./modules.md)
- [Casos de uso e consultas](./application.md)
- [Domínio independente de ORM](./domain.md)
- [Autorização](./authorization.md)
- [TenantContext](./tenant-context.md)
- [Transações, concorrência e Unit of Work](./transactions.md)
- [Eventos, auditoria e efeitos](./events.md)
- [Outbox conceitual](./outbox.md)
- [Infraestrutura e serviços externos](./infrastructure.md)
- [Observabilidade e cache](./observability.md)
- [Modelo temporal](./temporal-model.md)
- [Estrutura conceitual do projeto](./project-structure.md)
- [Revisão crítica da arquitetura](../../ARCHITECTURE-CRITICAL-REVIEW.md)
- [Remediação arquitetural](../../ARCHITECTURE-REMEDIATION.md)

## Estado das decisões

As decisões de produto/domínio permanecem em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md). Este conjunto de documentos não acrescenta nele decisões técnicas. Propostas e questões arquiteturais/tecnológicas estão resumidas em [decisions.md](./decisions.md).

## Fora de escopo

Não foram criados código Go, handlers, middleware, models GORM, repositories concretos, migrations, SQL, Docker, infraestrutura executável, filas, Redis, broker ou providers. Também não foram definidos endpoints, autenticação, status HTTP ou esquemas de API.
