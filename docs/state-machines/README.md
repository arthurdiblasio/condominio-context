# State machines e ciclos de vida

## Escopo e autoridade

Este catálogo organiza os ciclos de vida sugeridos pelos workflows. É documentação de domínio: não define armazenamento, execução automática, relógios técnicos, mecanismos de concorrência, filas ou integração com provedores.

Os estados marcados como **base conceitual** decorrem de invariantes já documentadas. Os marcados como **condicionais** só se aplicam se a política local, de produto ou jurídica correspondente for aprovada. Uma transição não listada não é implicitamente permitida. Este catálogo não aprova a matriz proposta de papéis e permissões.

## Documentos

- [Identidade, vínculos, áreas e reservas](./identity-and-reservations.md)
- [Visitas, acesso e pacotes](./access-and-packages.md)
- [Notificações, assembleias e votação](./communications-and-assemblies.md)
- [Módulos e features](./modules.md)

## EVENTS ARE NOT STATES

Estado descreve uma condição de ciclo de vida. Evento registra algo que ocorreu. A observação de um fato pode afetar uma condição derivada, mas não a substitui nem prova outros fatos.

| Evento/fato | O que não significa por si só |
|---|---|
| `AccessEvent(ENTRY)` | Não ativa nem consome automaticamente `AccessAuthorization`, nem prova `EXIT`. |
| `AccessEvent(EXIT)` | Não cria uma `ENTRY` ausente nem encerra, por si só, uma `Visit` segundo regra universal. |
| `AccessEvent(DENIED)` | Não é estado de autorização nem altera a autorização para ativa. |
| `PackageEvent(RECEIVED)` | É recebimento físico, não notificação, confirmação ou retirada. |
| `PackageEvent(NOTIFIED)` | Não comprova que a mensagem foi entregue. |
| `PackageEvent(CONFIRMED)` | Não comprova retirada física. |
| `PackageEvent(PICKED_UP)` | É fato de retirada; um eventual status atual do pacote é derivado por regra separada. |
| `Participant` presente | Não implica elegibilidade, representação válida ou voto. |
| `VotingEligibility(ELIGIBLE)` | Não é `Permission` e não registra voto. |
| `Vote` registrado | É o fato de voto; não significa resultado apurado/publicado. |
| Convocação, abertura, encerramento ou publicação | São fatos/ações; só são estados quando o lifecycle específico os define como tal. |

## Catálogo resumido

| Conceito | Estados/condição base ou proposta | Inicial | Terminais | Dependências / decisões |
|---|---|---|---|---|
| `UserAccount` | `INVITED`, `ACTIVE`, `SUSPENDED`, `DISABLED`; `EXPIRED` não adotado | `INVITED` quando criado por convite | `DISABLED`; convite expirado encerra convite, não necessariamente conta | OD-03; não há permissions `user_account.*` aprovadas |
| `RoleAssignment` | `ACTIVE`, `SUSPENDED`, `REVOKED`, `EXPIRED`; `PENDING` condicional | `ACTIVE` quando grant aprovado e efetivo; `PENDING` só quando configurado | `REVOKED`, `EXPIRED` | OD-02/03/14/15/16; permissões específicas de assignment |
| Vínculos de unidade | `FUTURE`, `ACTIVE`, `ENDED` como condição temporal | Definido por período e política | `ENDED` | OD-09; histórico não se apaga |
| `CommonArea` | `ACTIVE`, `BLOCKED`, `MAINTENANCE`, `DISABLED` como proposta operacional | Definido no cadastro conforme decisão local | `DISABLED` para uso corrente; reativação depende de autorização | OD-04/11 |
| `Reservation` | `DRAFT`, `PENDING`, `CONFIRMED`, `REJECTED`, `CANCELLED`, `COMPLETED`; `EXPIRED` condicional | `DRAFT` ou `PENDING` conforme fluxo | `REJECTED`, `CANCELLED`, `COMPLETED`, `EXPIRED` | OD-04/18; não há estado `IN_PROGRESS` |
| `AccessAuthorization` | `DRAFT`, `ACTIVE`, `EXPIRED`, `REVOKED`, `CANCELLED` | `DRAFT` se requer revisão; caso contrário `ACTIVE` após requisitos | `EXPIRED`, `REVOKED`, `CANCELLED` | OD-05; não há `USED` automático |
| `AccessEvent` | Sem state machine; fato registrado e correção referenciada | Fato observado | Não aplicável | `access_event.register/correct`; OD-05/07/18 |
| `Visit` | `PLANNED`, `CANCELLED`; `COMPLETED`/`NO_SHOW` condicionais | `PLANNED` | `CANCELLED`, `COMPLETED`, `NO_SHOW` quando habilitados | OD-05/07; chegada/permanência derivam de fatos de acesso |
| `Package` | Situação derivada candidata: `IN_CUSTODY`, `RELEASED`, `CANCELLED` | Após `RECEIVED`, se a projeção for adotada | `RELEASED`, `CANCELLED` condicionais | OD-06/07/18; `PICKED_UP` é evento, não status |
| `PackageEvent` | Tipos: `RECEIVED`, `NOTIFIED`, `CONFIRMED`, `PICKED_UP`, `CANCELLED`; correção vinculada | `RECEIVED` é primeiro fato de custódia | Não é máquina de estados | OD-06/18; eventos originais preservados |
| `Notification` | `CREATED`, `CANCELLED`; resultado global derivado, não estado assumido | `CREATED` | `CANCELLED` impede tentativas futuras conforme policy | OD-08 |
| `NotificationDelivery` | `PENDING`, `SUBMITTED`, `DELIVERED`, `FAILED`, `CANCELLED`, `EXPIRED`; `READ` condicional | `PENDING` | `READ` quando evidenciado, `FAILED`, `CANCELLED`, `EXPIRED` por tentativa | OD-08; retry cria outra tentativa |
| `Assembly` | `DRAFT`, `SCHEDULED`, `OPEN`, `CLOSED`, `CANCELLED` | `DRAFT` se houver elaboração; senão `SCHEDULED` | `CLOSED`, `CANCELLED` | OD-01/07; `VOTING` pertence à pauta |
| `AgendaItem` | `DRAFT`, `OPEN`, `VOTING`, `CLOSED`, `CANCELLED` | `DRAFT` ou `OPEN` conforme o processo aprovado | `CLOSED`, `CANCELLED` | OD-01; deliberação/resultado não é estado |
| `Participant` | Relação `REGISTERED`; presença/ausência são fatos ou classificação final condicional | `REGISTERED` | `ABSENT` somente após critério aprovado | OD-01/07; `REPRESENTED` não é estado |
| `VotingEligibility` | Condição/decisão `ELIGIBLE`, `INELIGIBLE`; `CONTESTED` condicional | Não aplicável | Não aplicável | OD-01; não é Permission |
| `Quorum` | Avaliação/resultado segundo regra, não máquina de estados | Não aplicável | Não aplicável | OD-01; regra, apuração e authority ainda abertas |
| `Proxy` | `PENDING_VALIDATION` condicional, `ACTIVE`, `REJECTED` condicional, `REVOKED`, `EXPIRED` | Ativo após validação exigida | `REJECTED`, `REVOKED`, `EXPIRED` | OD-01/02; `USED` é fato, não estado |
| `Vote` | Fato `RECORDED`; `INVALIDATED` somente por correção autorizada | Ao registrar o voto | Não reabrir sem procedimento aprovado | OD-01/07/18 |
| `CondominiumModule` / `CondominiumFeature` | `ENABLED`, `DISABLED` | Configuração inicial aprovada | Nenhum terminal irreversível definido | OD-11/17; não apaga dados |

Estados candidatos não são necessariamente persistidos: prazos e situações derivadas podem ser calculados conforme política de domínio, sem introduzir mecanismo técnico.

## Guardas transversais

Toda transição que modifica dados ou estado deve:

1. selecionar explicitamente um tenant quando o recurso for tenant-scoped;
2. confirmar que pessoa, recurso, referências e escopo pertencem ao mesmo condomínio;
3. exigir `UserAccount` ativa quando a ação for autenticada;
4. exigir `RoleAssignment` vigente, `Permission` e `Scope` compatíveis com a ação e recurso;
5. verificar estado de workflow, período, dependências, vínculo e condições de negócio aplicáveis;
6. não tratar papel, presença, autorização, elegibilidade ou habilitação de feature como substitutos uns dos outros;
7. auditar transição sensível e preservar o tenant e a referência histórica.

Permissões abaixo são as capabilities conceituais já catalogadas em [PERMISSIONS.md](../../PERMISSIONS.md). Onde não existe capability, isso é uma lacuna a resolver, não autorização presumida.

## Transições proibidas ou condicionadas

| Tentativa | Tratamento |
|---|---|
| Conta `SUSPENDED` ou `DISABLED` praticar nova ação autenticada | Proibida; preservar pessoa, vínculos, assignments e autoria histórica. |
| `RoleAssignment(REVOKED/EXPIRED)` voltar a `ACTIVE` | Proibida no mesmo ciclo; criar novo grant ou aplicar reativação apenas a `SUSPENDED` sob authority aprovada. |
| Qualquer vínculo de pessoa ou assignment atravessar tenant por alteração de unidade/contexto | Proibida; encerramento/criação explícitos no tenant correto, se autorizados. |
| Reserva `CONFIRMED` sem aprovação obrigatória, sem disponibilidade ou em conflito vedado | Proibida. |
| `REJECTED`, `CANCELLED`, `COMPLETED` ou `EXPIRED` voltar a `CONFIRMED` | Proibida sem fluxo de reabertura aprovado; não reabrir silenciosamente. |
| `IN_PROGRESS` usado como estado persistente de Reservation sem regra de início/encerramento | Não adotado; condição temporal/operacional não foi definida. |
| Área `BLOCKED`, `MAINTENANCE` ou `DISABLED` receber confirmação conflitante | Proibida enquanto indisponível conforme política. Reservas preexistentes requerem decisão explícita, sem cancelamento silencioso. |
| Autorização expirada/revogada/cancelada ser reativada | Proibida; requer nova autorização ou reemissão explicitamente aprovada. |
| `AccessEvent(EXIT)` sem `ENTRY` ser corrigido inventando `ENTRY` | Proibida; registrar divergência e correção vinculada se cabível. |
| `ENTRY`, `EXIT` ou `DENIED` ser estado de `AccessAuthorization` | Proibida; são tipos de `AccessEvent`. |
| Visita ser marcada `ARRIVED`/`INSIDE` sem observação correspondente, ou inferida só da autorização | Proibida; usar fatos de acesso, sem inventar presença. |
| `PackageEvent(NOTIFIED/CONFIRMED/PICKED_UP)` preceder recebimento sem política/fato justificante | Proibida como regra-base; exceções e sequências especiais requerem decisão. |
| Evento original de pacote ou acesso ser substituído/apagado para aparentar outro histórico | Proibida; correção referenciada e auditada. |
| Voto ser aceito fora de `AgendaItem(VOTING)` ou após seu encerramento | Proibida, salvo processo formal de reabertura aprovado. |
| Voto ser duplicado/substituído sem regra por pauta e trilha | Proibida enquanto OD-01 estiver aberta. |
| `REPRESENTED` ser inferido de presença, ou `VotingEligibility` ser inferida de Permission | Proibida. |
| `Proxy(USED)` tornar-se estado consumido universal ou grant administrativo | Proibida; uso é fato delimitado, não RoleAssignment. |
| Entrega `SUBMITTED` ser marcada `DELIVERED` sem evidência | Proibida; “enviado/submetido” e “entregue” são resultados distintos. |
| Falha de uma `NotificationDelivery` sobrescrever outra tentativa ou o fato de negócio | Proibida. |
| Desabilitar módulo/feature apagar histórico ou grants | Proibida. |
| Mudar Module/Feature em um condomínio afetar outro | Proibida. |

Reabertura ou correção em qualquer outro caso depende do documento específico e das decisões listadas na seção **OPEN DECISIONS**.

## Repetição de operação e idempotência de negócio

As classificações são **orientação de domínio para revisão**, não mecanismo técnico nem política final. `IDEMPOTENT` significa que a repetição mantém o mesmo resultado de negócio sem um segundo fato; `NO-OP` que o estado já pretendido permanece sem novo fato; `REJECT` que a operação não deve prosseguir; `NEW_EVENT` que houve uma ocorrência distinta comprovável. Sem prova de que a repetição representa a mesma ocorrência, não a colapsar automaticamente.

| Operação repetida | Classificação proposta | Regra/limite |
|---|---|---|
| Confirmar o mesmo pacote duas vezes | `IDEMPOTENT` se for inequivocamente o mesmo ato; caso contrário `REJECT`/revisão | Não criar duas confirmações nem contar repetição como novo consentimento. OD-06/18. |
| Registrar a mesma presença para a mesma pessoa/assembleia | `NO-OP` quando for a mesma observação; nova evidência de presença é `NEW_EVENT` apenas se prevista | Não duplicar quórum; não fundir observações distintas por semelhança. OD-01/18. |
| Registrar voto duas vezes para pessoa/pauta | `REJECT` até procedimento aprovado | Não escolher silenciosamente qual voto vale nem sobrepor o primeiro. OD-01/18. |
| Cancelar reserva já cancelada | `NO-OP` para o estado; não criar um segundo cancelamento | Tentativa e auditoria podem ser registradas conforme policy. OD-04/18. |
| Revogar autorização já revogada | `NO-OP` para o estado; não criar nova transição | Não reativar nem estender validade. OD-05/18. |
| Registrar novamente o mesmo recebimento/retirada | `REJECT`/revisão sem prova de evento distinto | `NEW_EVENT` somente para encomenda ou ocorrência física distinta comprovada. OD-06/18. |
| Reenviar uma entrega de notificação | `NEW_EVENT` como nova `NotificationDelivery` | Manter a tentativa anterior; não criar automaticamente nova intenção lógica. OD-08/18. |
| Repetir grant/alteração de papel | `NO-OP` somente se mesmo alvo, conteúdo e período; conflito de mudança requer `REJECT`/revisão | Não criar assignments cumulativos nem sobrescrever. OD-02/18. |
| Repetir ENTRY/EXIT observado | `NEW_EVENT` somente se observação física distinta; ambiguidade requer revisão | Não deduplicar exclusivamente por proximidade temporal. OD-05/18. |

## Concorrência de negócio

Não se prescrevem locks, transações ou mecanismos de sincronização.

| Conflito concorrente | Regra de negócio |
|---|---|
| Duas reservas incompatíveis para a mesma área/período | Não confirmar ambas se a política proíbe sobreposição; critério de precedência/resultado é OD-04. |
| Dois votos para a mesma pessoa/pauta | Preservar cada tentativa; não aceitar/substituir silenciosamente. A regra de voto único/substituição e desempate é OD-01. |
| Duas retiradas do mesmo pacote | Não registrar dois fatos como retirada única. Confirmar entrega física e ator; conflito exige revisão, sem apagar a primeira evidência. OD-06/18. |
| Alterações concorrentes de RoleAssignment | Não permitir que uma atualização silencie a outra; validar autoridade e estado vigente no momento efetivo. Conflitos de grants seguem OD-02/14/18. |
| Revogação concorrente com ação já iniciada | O ponto efetivo da revogação e tratamento de ação em andamento dependem de política; nenhuma revogação altera ações passadas. OD-02/03/05. |
| Bloqueio/manutenção de área com reserva confirmada | Preservar a reserva e comunicar/decidir sua continuidade conforme OD-04; não cancelar nem manter como válida por efeito implícito. |
| Confirmação/retirada concorrente com correção de PackageEvent | Preservar fatos, suspender conclusão se houver inconsistência e revisar sob autoridade de correção. OD-06/18. |

## Correção histórica, terminais e reabertura

- Uma correção é vinculada ao registro/fato original e identifica autor, instante, escopo/tenant e motivo quando exigido.
- O original permanece preservado; correção não equivale a exclusão, substituição silenciosa ou autorização retroativa.
- O estado corrente pode ser recalculado somente segundo regra de domínio aprovada; notificações e workflows dependentes não são reexecutados por inferência.
- `REVOKED`, `EXPIRED`, `CANCELLED`, `REJECTED`, `COMPLETED` e `CLOSED` são terminais no ciclo descrito, a menos que o documento específico marque expressamente uma reabertura aprovada.
- `SUSPENDED` de `UserAccount`/`RoleAssignment` é reversível somente pela permission e autoridade aprovadas; não restaura outros grants.
- Vínculos de unidade terminam por vigência/encerramento; correções de período preservam versão factual anterior.
- Estados derivados ou factuais (presença, elegibilidade e resultado de entrega) não são reabertos como se fossem estados persistidos.
- Correção posterior ao término deve preservar a finalização original e registrar decisão nova, não ressuscitar automaticamente a entidade.

### Aplicação por conceito

| Conceito | Correção admissível | O que preservar / Permission e scope |
|---|---|---|
| `AccessEvent` | Registrar ocorrência corretiva ligada ao fato observado | Preservar ENTRY/EXIT/DENIED original, ator, instante, tenant e divergência; `access_event.correct` no scope original. |
| `PackageEvent` | Acrescentar correção referenciada ao evento | Preservar recebimento/confirmação/retirada/cancelamento original, ator/registrador, instante e evidência; `package.correct` no tenant do Package. |
| `Vote` | Invalidar/corrigir somente sob procedimento aprovado | Preservar o voto original e sigilo aplicável; `vote.correct`/`vote.cancel` no contexto Assembly/AgendaItem. Reapuração é etapa distinta. |
| `Participant` | Corrigir registro de inscrição/presença com referência à observação original | Preservar presença/ausência e ator/instante anteriores; `participant.manage` ou capability específica a definir, no contexto da Assembly. |
| `Reservation` | Alterar/corrigir dados por workflow rastreável; revalidar conflito e approval | Preservar solicitante/período/decisões anteriores; `reservation.update`/`reservation.cancel`, tenant da Unit/CommonArea; não converter status terminal em ativo por correção simples. |
| `RoleAssignment` | Corrigir grant/scope/vigência por decisão auditada; substituição de scope pode requerer revogação e novo grant | Preservar concessor/recebedor, Role, Scope, tenant, estado e período anteriores; `role_assignment.*` compatível com authority. Não reativar grant revogado/expirado. |

## OPEN DECISIONS

Decisões existentes relacionadas ao lifecycle: OD-01 a OD-12, OD-14 a OD-18. O catálogo detalha impactos nos documentos temáticos; particularmente:

- **CRITICAL:** OD-01 (assembleias, elegibilidade, voto, procuração, fechamento e correções); OD-02 (autoridade sobre atribuições).
- **HIGH:** OD-03 (conta/convite/suspensão), OD-04 (reserva, estados, área bloqueada e concorrência), OD-05 (visita/acesso/eventos), OD-06 (custódia e estado derivado do pacote).
- **MEDIUM:** OD-08 (estados e tentativas de notificação), OD-09 (vigência/sobreposição de vínculos), OD-11 (habilitação e operações em curso), OD-17 (grants por feature), OD-18 (repetições e concorrência).

Não há decisão puramente técnica a registrar como decisão de domínio.
