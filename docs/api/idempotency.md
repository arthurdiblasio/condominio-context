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
