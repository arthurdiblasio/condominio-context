# Application: casos de uso, commands e queries

## Casos de uso

Cada caso de uso representa uma intenção de negócio do [API Contract](../../API-CONTRACT.md) ou uma etapa de workflow. O caso de uso orquestra, valida contexto/authorization, chama o domínio, exige persistência/auditoria conforme o caso e coordena fatos/efeitos.

## Fronteira obrigatória para operações com efeito

Toda operação que possa alterar estado, registrar fato, produzir side effect ou iniciar uma mudança de autoridade deve passar por esta mesma fronteira de Application, independentemente da origem:

- request/API;
- handler de evento, consumer ou retomada/retry;
- ação agendada/expiração;
- administração ou suporte assistido;
- processo de plataforma;
- operação interna iniciada por outro módulo.

Adapters, job handlers e consumers não chamam repositories nem mutam agregados diretamente. Eles entregam um comando a um use case e fornecem um contexto de actor/origem explícito. Não existe “caminho interno confiável” que ignore tenant, business guards, authorization ou audit.

Antes de qualquer efeito, o use case:

1. resolve `Actor` e seu tipo sem confundir com subject/Person afetada;
2. valida `Candidate Tenant` e cria contexto validado para uma operação tenant-scoped (ou classifica a operação explicitamente Platform-scoped);
3. carrega recurso sob esse contexto e valida pertencimento/referências;
4. exige authority apropriada ao tipo de actor e ação, Permission/Scope quando aplicáveis e regras de domínio;
5. confirma efeito/fato com AuditLog obrigatório na mesma unidade atômica;
6. só após resultado confirmado disponibiliza efeitos externos; se necessário, grava sua intenção durável atomicamente.

Capability não aprovada/ausente, actor ambíguo, tenant ausente ou policy de decisão pendente bloqueia a operação antes do efeito. A arquitetura não inventa uma permissão substituta.

### Actor e subject

| Actor | Identidade/autoria | Requisito de autoridade |
|---|---|---|
| `HUMAN ACTOR` | Usuário humano autenticado; registrar `UserAccount` e `Person` quando associação conhecida, além da ação e contexto. | Validar a conta e grants atuais no tenant/resource/scope. |
| `SYSTEM ACTOR` | Processo de produto como expiração agendada; identificar processo, regra/gatilho, origem e tenant. Não é uma pessoa. | Somente transição automática explicitamente prevista por policy e limitada ao efeito já autorizado; não é bypass nem grant universal. Authority semântica e revisão seguem decisão pendente quando regra/autoridade não existe. |
| `SERVICE ACTOR` | Serviço/integração identificado; não se passa por humano nem por Person afetada. | Capability delegada, tenant, purpose e ações explicitamente limitados; evidência técnica não substitui autoridade de negócio. |
| Assisted operation | Registrar operador humano real, pessoa representada/alvo, motivo/finalidade e vínculo de representação aplicável. | O suporte não herda authority da pessoa representada. Delegação/emergência depende de OD-16. |
| Affected Person / subject | Pessoa ou conta cujos direitos/dados/recursos são alvo. | Não é automaticamente o actor; registrar ambos distintamente quando diferentes. |
| Assembly `Proxy` | Representação restrita ao contexto e pauta conforme OD-01. | Nunca é representação genérica de suporte, serviço ou ator de domínio. |

Jobs/eventos carregam actor/source, tenant, purpose, origin fact, operation identity e correlation. No consumo, valida-se envelope e recurso novamente. Se o trabalho executa uma nova decisão de negócio (por exemplo, aprovar, retirar, conceder ou votar), exige autorização corrente de um actor autorizado; não pode usar autoria do evento original. Se apenas cumpre consequência previamente autorizada (por exemplo, enviar Notification já criada ou aplicar expiração determinística definida), valida a intenção, escopo, estado e validade atuais e usa actor de serviço/sistema restrito. A fronteira entre consequência e nova decisão deve ser registrada para cada job; ambiguidade bloqueia automação.

Retry não reduz guardas. Uma retomada após crash volta ao mesmo use case e à mesma identidade de operação; não duplica fato nem assume que commit falhou/sucedeu sem resolver o resultado.

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

Contratos de repositório, transação, leitura, tempo, ID, auditoria e publicação existem conceitualmente somente para atender casos de uso concretos. Cada porta tenant-scoped exige contexto tenant validado ou garantia equivalente; não pode permitir omissão. Infrastructure fornece implementações. Não criar abstrações universais ou `GenericRepository<T>` por conveniência.

## Erros

Application diferencia falta de contexto/autoridade, regra inválida, conflito, decisão pendente e falha técnica de dependência. Preserva causa/contexto interno para diagnóstico autorizado; Interface mapeia para categorias externas minimizadas conforme [modelo de erros](../api/errors.md).
