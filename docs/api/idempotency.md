# Idempotência, concorrência, audit e efeitos

Semantics abaixo são conceituais e vêm das [state machines](../state-machines/README.md), workflows e OD-18. Não definem header/token técnico, locking, transação, fila, retry automático ou mecanismo de execução.

## Repetição de comandos

| Command | Classificação conceitual | Resultado de negócio |
|---|---|---|
| Confirmar Package novamente, comprovadamente o mesmo ato | `IDEMPOTENT`; caso ambíguo `REJECT`/review | Não gerar CONFIRMED duplicado. |
| Cancelar Reservation já cancelada | `NO-OP` | Manter terminal, sem novo cancelamento factual; tentativa sensível pode ser auditada. |
| Revogar AccessAuthorization já revogada | `NO-OP` | Não reativar, estender ou gerar segunda transição. |
| Registrar presença da mesma pessoa na mesma observação | `NO-OP` se comprovadamente mesma observação | Não contar presença/quorum em dobro. |
| Registrar voto mais de uma vez para mesma pessoa/pauta | `REJECT` enquanto substituição não for aprovada | Não sobrescrever nem escolher voto silenciosamente. |
| Registrar ENTRY/EXIT novamente | `NEW_EVENT` somente se observação física distinta | Proximidade temporal não prova duplicata; ambiguity requires review. |
| Registrar recebimento/retirada de Package novamente | `REJECT` ou revisão sem fato físico distinto; `NEW_EVENT` para ocorrência distinta comprovada | Preservar todas as evidências. |
| Reenviar NotificationDelivery | `NEW_EVENT` | Criar nova tentativa conceitual, manter status anterior; não duplicar Notification intention automaticamente. |
| Repetir grant idêntico | `NO-OP` somente se pessoa, Role, Scope, tenant, período e efeito forem idênticos; caso contrário `REJECT`/review | Não acumular assignments nem alterar concessor/histórico silenciosamente. |
| Repetir a mesma alteração de enablement | `NO-OP` no estado corrente | Não criar mudança duplicada; audit conforme policy. |

Classificações dependem de distinguir a mesma operação do mesmo evento físico. Onde identidade do ato ou deduplicação não estão definidas, não consolidar fatos automaticamente.

## Identidade de operação, replay e falha incerta

Idempotência semântica requer reconhecer **a mesma intenção de comando**, não apenas comandos com payloads iguais. Uma identidade estável de operação deve acompanhar tentativas/retries e sua conclusão observável. A identidade e seu transporte/armazenamento são contratos técnicos futuros; não se define header, token, coluna ou protocolo nesta documentação.

- Repetição após confirmação conhecida retorna/representa o resultado original ou `NO-OP` conforme a semântica abaixo; não cria nova ocorrência nem audit de sucesso duplicado.
- Em timeout ou conexão perdida após o pedido, o resultado é **incerto**, não necessariamente falha. Resolver pela identidade/histórico antes de tentar aplicar novamente.
- Uma operação diferente com mesmo conteúdo pode ser uma nova intenção legítima; payload/fingerprint não é, por si só, identidade.
- Replay recebido por consumer/outbox deve manter a identidade original; nova tentativa deliberada de NotificationDelivery tem identidade nova e referência à intenção.
- Eventos físicos `ENTRY`, `EXIT`, `RECEIVED` e `PICKED_UP` não são deduplicados por pessoa, unidade, conteúdo ou proximidade temporal. Só a mesma mensagem/observação comprovadamente repetida pode ser replay.
- Se identidade/resultado não puderem ser estabelecidos, não inventar sucesso ou duplicata: conflito/revisão manual conforme regra aplicável.

| Operação | Mesma identidade após timeout/replay | Intenção nova/concorrência |
|---|---|---|
| Reservation approval/cancel | Retornar resultado já commitado/NO-OP sem nova transição; se estado mudou sob outro comando, revalidar e retornar conflito. | Approval revalida disponibilidade/estado atual; competição proibida não confirma ambas. Cancel concorrente revalida status. Prioridade/regras permanecem OD-04/18. |
| Package confirm/release | Não criar segundo CONFIRMED/PICKED_UP para o mesmo ato; identificar resultado anterior. | Só registrar nova ocorrência física comprovada; retirada simultânea/ambígua entra em revisão, sem colapsar fatos. OD-06/18. |
| AccessEvent | Replay da mesma observação não gera evento/audit de sucesso duplicado. | Nova observação é `NEW_EVENT`, mesmo para pessoa e ponto iguais; não deduplicar por proximidade. OD-05/18. |
| RoleAssignment grant/revoke | Mesmo comando não cria outro ciclo nem troca silenciosamente concessor/authority. | Grants idênticos podem ser NO-OP somente se igualdade semântica for verificável; operações concorrentes revalidam authority/estado. OD-02/14/18. |
| Vote | Replay devolve resultado do mesmo voto sem registrar outro. | Segundo voto para mesma pessoa/pauta é rejeitado até policy de substituição aprovada; concorrência nunca escolhe silenciosamente. OD-01/18. |
| Assembly close | Replay não cria novo fechamento. | Vote/close precisam de ordenação consistente: nenhum voto é aceito como posterior ao fechamento efetivo. Reabertura segue OD-01. |
| Feature enable/disable | Repetição do mesmo alvo/estado é NO-OP, sem segunda mudança factual. | Toggle concorrente revalida estado/configuração; operação dependente não usa uma configuração stale. Efeito em operação em andamento é OD-11. |
| Notification creation/delivery | Replay de criação não duplica a mesma intenção/tentativa. | Retry deliberado gera nova NotificationDelivery, preserva anterior e referencia intenção; política de retries/cancelamento é OD-08. |

Concorrência tem um ponto de decisão linearizável por operação, mas nenhum mecanismo técnico (lock, versionamento, constraint, transação SQL) é escolhido aqui. Casos sem regra de precedência/conflict não devem executar de modo permissivo.

## Concorrência de negócio

| Commands concorrentes | Regra conceitual |
|---|---|
| Solicitações/aprovações de Reservation para mesma CommonArea/período | Revalidar disponibilidade na decisão; não confirmar ambas se conflito proibido. Prioridade/empate é OD-04. |
| Votos para mesmo eleitor/pauta | Não contar duplicado nem substituir. Unicidade, janela, sigilo e regra de conflito são OD-01. |
| Duas retiradas de um Package | Não apresentar duas retiradas como uma só; confirmar fatos e atores, preservar ambas as evidências e encaminhar conflito (OD-06/18). |
| Grant/suspend/revoke concorrentes | Não aplicar decisão baseada em authority/states stale por convenção; efetividade temporal e precedência são OD-02/14/18. |
| `AccessAuthorization` usada enquanto revogada/expirada | Avaliar vigência e instante de uso conforme OD-05; nunca fazer a autorização retornar a active automaticamente. |
| Module/Feature disable durante operação ativa | Não apagar/forçar estado final da operação; efeito depende de OD-11. Bloquear novas operações que exigem capability conforme policy aprovada. |
| Correção concorrente com ação sobre Package/AccessEvent/Vote | Preservar o original e registrar ambos os contextos; suspender conclusão conflitante e solicitar revisão quando necessário. |

## Audit requirement por command

Para command que altera ou produz fato sensível, preservar conceitualmente: actor/subject, tenant/context, resource, action, instante, estado anterior e resultante quando houver, resultado da decisão, reason quando exigido, referências e correlação com o fato corrigido. Falha de autorização sensível pode ser auditada conforme policy; consultas ao AuditLog são restritas e também sujeitas a audit.

Correções de `AccessEvent`, `PackageEvent`, `Vote`, `Participant`, `Reservation` e `RoleAssignment` mantêm referência ao original, autoria, tenant, motivo exigido e resultado. Não suportar update/delete genérico para estes históricos.

## Eventos de domínio conceituais

Commands bem-sucedidos podem produzir facts/events como:

- `CondominiumCreated`, `CondominiumConfigured`;
- `BlockCreated/Changed`, `UnitCreated/Changed`;
- `PersonCreated/Updated/Associated`, `UnitRelationshipStarted/Ended/Corrected`;
- `UserAccountInvited/Activated/Suspended/Reactivated/Disabled` (actor/capability pendentes);
- `RoleGranted/Suspended/Reactivated/Revoked/Expired`;
- `CommonAreaBlocked/Reopened/Disabled`;
- `ReservationRequested/Confirmed/Rejected/Cancelled/Completed`;
- `VisitPlanned/Cancelled/Completed`; `AccessAuthorizationCreated/Activated/Revoked/Expired/Cancelled`;
- `AccessEventRecorded/Corrected` com tipo `ENTRY`, `EXIT` ou `DENIED`;
- `PackageReceived/Notified/Confirmed/Released/Cancelled/Corrected`;
- `NotificationCreated`; `NotificationDeliveryAttempted/Submitted/Delivered/Failed/Read`;
- `AssemblyScheduled/Opened/Closed/Cancelled`; `AgendaItemVotingOpened/Closed`;
- `ParticipantRegistered/PresenceRecorded`; `VotingEligibilityDetermined`; `ProxySubmitted/Validated/Used/Revoked/Expired`;
- `VoteRecorded/Invalidated`; `QuorumCalculated`, `AssemblyResultPublished` se processo aprovado;
- `CondominiumModuleEnabled/Disabled`, `CondominiumFeatureEnabled/Disabled`;
- Financial entry/supplier/report facts, somente se optional module habilitado.

Nomes são exemplos semânticos, não esquema, garantia de emissão, event stream, broker ou resposta da API. Negação não deve produzir evento de sucesso. Correção produz referência/fato corretivo, preservando o original.

## Side effects de Notification

| Fact/command fonte | Notification possível | Condição |
|---|---|---|
| Package received/confirmed/released | destinatário ou portaria | finalidade, destinatário, permission/configuração e OD-08. |
| Reservation requested/decision/change/cancel | solicitante/afetados | policy local; entrega não altera estado da reserva. |
| Visit/AccessAuthorization create/revoke | anfitrião/pessoa autorizada | finalidade/canal/visibilidade e OD-05/08. |
| Role grant/suspend/revoke | pessoa afetada/admin | somente se política disser; notificação não ativa/revoga grant. |
| Assembly convocation/change/close/result | participantes | convocação/publicação legalmente configuradas; delivery não define validade. |
| Module/Feature change | responsáveis do tenant | configuração local e OD-11. |

Notification/Delivery não é parte obrigatória do sucesso do comando fonte salvo regra explícita futura; falha não reverte o fato. Nenhum canal/provedor específico é assumido.
