# Application: casos de uso, commands e queries

## Casos de uso

Cada caso de uso representa uma intenção de negócio do [API Contract](../../API-CONTRACT.md) ou uma etapa de workflow. O caso de uso orquestra, aplica autorização/contexto, chama o domínio, exige persistência/auditoria conforme o caso e coordena fatos/efeitos.

Exemplos conceituais: `CreateCondominium`, `CreatePerson`, `GrantRole`, `CreateReservation`, `ApproveReservation`, `RegisterPackage`, `ConfirmPackage`, `ReleasePackage`, `CreateAccessAuthorization`, `RecordAccessEvent`, `CreateAssembly`, `RegisterPresence`, `RecordVote`, `CloseAssembly`. Nomes são rótulos de documentação, não classes, structs ou assinatura de código.

## Mapeamento dos workflows

| Workflows | Grupos de casos de uso conceituais |
|---|---|
| WF-01 | Solicitar/criar/configurar tenant e concluir etapas de onboarding autorizadas. |
| WF-02–03 | Criar, alterar ou encerrar Block e Unit. |
| WF-04–05 | Criar/associar Person e executar lifecycle de UserAccount após OD-03/capabilities aprovadas. |
| WF-06–08 | Iniciar, encerrar ou corrigir UnitOwnership, UnitResidency e UnitTenancy. |
| WF-09 | Grant, suspend, reactivate, revoke e expiração de RoleAssignment. |
| WF-10–11 | Gerir Vehicle e Pet conforme política local e escopo da entrega. |
| WF-12 | Configurar, bloquear, colocar em manutenção, reabrir ou desabilitar CommonArea. |
| WF-13 | Solicitar, aprovar, rejeitar, alterar, cancelar e concluir Reservation conforme regra. |
| WF-14 | Criar/alterar/cancelar Event e registrar convidados sem conceder acesso implicitamente. |
| WF-15–17 | Planejar/cancelar Visit, gerir AccessAuthorization e registrar/corrigir AccessEvent. |
| WF-18–19 | Registrar recebimento, notificação, confirmação, retirada, cancelamento/correção de Package. |
| WF-20 | Criar Notification e acompanhar cada NotificationDelivery, sem provider específico. |
| WF-21–22 | Gerir Assembly, AgendaItem, Participant, presença, eligibility, Proxy e Vote dentro das regras aprovadas. |
| WF-23 | Habilitar/desabilitar Module ou Feature em catálogo/tenant e verificar authority. |
| WF-24 | Operações financeiras opcionais, somente se o módulo for aprovado e habilitado. |
| WF-25–26 | Consultar AuditLog, solicitar/aplicar correções e tratar operação no tenant errado. |

A lista deriva de [WORKFLOWS.md](../../WORKFLOWS.md); ausência de capability ou regra é lacuna, não autorização.

## Command vs Query

- **Command:** expressa intenção que pode alterar estado, registrar fato, aplicar regra e produzir eventos. Application não aceita alteração genérica de estado que contorne state machine.
- **Query:** consulta uma representação autorizada e filtrada. Não aprova, cancela, reserva disponibilidade, marca presença, envia voto nem altera lifecycle.
- **CQRS completo:** não é necessário na proposta inicial. Commands e Queries são responsabilidades distintas, mas podem compartilhar processo, modelo de domínio e persistência enquanto consistentes.

## Portas de aplicação

Contratos de repositório, transação, leitura, tempo, ID, auditoria e publicação existem conceitualmente somente para atender casos de uso concretos. Infrastructure fornece implementações. Não criar abstrações universais ou `GenericRepository<T>` por conveniência.

## Erros

Application diferencia falta de contexto/autoridade, regra inválida, conflito, decisão pendente e falha técnica de dependência. Preserva causa/contexto interno para diagnóstico autorizado; Interface mapeia para categorias externas minimizadas conforme [modelo de erros](../api/errors.md).
