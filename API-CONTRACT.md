# API Contract — contrato conceitual

## Objetivo e status

Este documento descreve a superfície conceitual de operações do produto Condomínio. Não é uma API implementada, uma especificação OpenAPI ou um compromisso sobre URLs, verbos/protocolos, payloads, status HTTP, autenticação ou tecnologia.

**Status:** `API_CONTRACT_REQUIRES_HUMAN_REVIEW`

O contrato deriva de [DOMAIN.md](./DOMAIN.md), [BUSINESS-RULES.md](./BUSINESS-RULES.md), [PERMISSIONS.md](./PERMISSIONS.md), [WORKFLOWS.md](./WORKFLOWS.md), [OPEN-DECISIONS.md](./OPEN-DECISIONS.md) e [state machines](./docs/state-machines/README.md). É conceitual e não autoriza implementação antes da revisão humana.

A arquitetura conceitual do backend que deverá preservar este contrato está em [docs/architecture/overview.md](./docs/architecture/overview.md), também sujeita a revisão humana e sem especificar endpoints ou implementação.

## Princípios

- Distinguir **Resource**, **Query**, **Command** e **Workflow Operation**. A existência de um recurso não implica CRUD completo.
- Alterações com consequência de negócio usam comandos semânticos; não permitir alteração genérica de estado que contorne guardas, permissions ou audit.
- Tenant e scope do recurso são explícitos e coerentes para toda operação condominial. A forma técnica de indicar contexto permanece aberta.
- Operação humana autenticada requer `UserAccount` ativa, associação a `Person` conforme OD-03, `RoleAssignment`, `Permission`, `Scope`, tenant e pré-condições do workflow; a matriz é proposta. Operação automática/de serviço deve ter actor identificado e authority limitada, nunca bypass implícito. Toda operação tenant-scoped usa contexto validado.
- Eventos de domínio/operacionais são fatos produzidos pelo comando, não endpoints genéricos para inserir/editar eventos.
- Resposta de API não é Domain Event. A comunicação/entrega de Notification é efeito independente e não reverte o resultado de negócio.
- Não apagar histórico por atualização, cancelamento, revogação, desativação, correção ou desabilitação de módulo.

## Documentos

- [Visão, contexto e audiências](./docs/api/overview.md)
- [Catálogo de recursos e escopo de release](./docs/api/resources.md)
- [Comandos e operações de workflow](./docs/api/commands.md)
- [Consultas, filtros, ordenação e paginação](./docs/api/queries.md)
- [Autorização, tenant e escopo](./docs/api/authorization.md)
- [Categorias de erro](./docs/api/errors.md)
- [Idempotência, concorrência, auditoria, eventos e notificações](./docs/api/idempotency.md)
- [Matriz de rastreabilidade WF → operação](./docs/api/traceability.md)
- [Versionamento conceitual](./docs/api/versioning.md)

O [modelo temporal conceitual](./docs/architecture/temporal-model.md) define distinções de instante, calendário local, timezone e validade sem fixar formato técnico ou regra legal.

## Limites do primeiro release

As classificações MVP, post-MVP, optional e future nos documentos são **propostas para revisão de produto**, não decisão aprovada. Assembleias, contas, gestão de papéis, plataforma e finanças têm decisões pendentes relevantes; seus comandos não devem ser implementados como regra consolidada só por estarem catalogados.

## OPEN DECISIONS relevantes

O contrato depende das decisões existentes, principalmente OD-01/02/03/04/05/06/07/08/09/11/12/14/15/16/17/18. Gaps de capability estão identificados por operação; não foram substituídos por permissões inventadas. Não foram acrescentadas decisões puramente técnicas ao [OPEN-DECISIONS.md](./OPEN-DECISIONS.md).
