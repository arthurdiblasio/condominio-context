# Transações, consistência e concorrência

## Fronteira

A fronteira de consistência é determinada pelo caso de uso e pelas invariantes do domínio. Application define quais fatos precisam ser aceitos ou recusados juntos; Infrastructure executa a transação com o mecanismo de persistência futuro.

O objetivo não é manter toda a operação de negócio em uma transação longa. Chamadas de rede, notificações, e-mail, Push, storage externo e espera por usuário não pertencem à transação síncrona do agregado.

## Casos que exigem consistência coordenada

| Caso de uso | Alterações/fatos que precisam ser coerentes | Auditoria/efeitos |
|---|---|---|
| Grant/suspend/revoke RoleAssignment | Estado efetivo do grant e seu histórico/transição sem sobrescrever fatos concorrentes. | Audit obrigatório com ator, tenant, scope, resultado e autoridade; notificação é posterior/opcional. |
| Aprovar Reservation | Estado de aprovação e validação de disponibilidade/conflicto para recurso/período segundo regra local. | Audit e fato de confirmação coerentes; Notification não bloqueia a aprovação. |
| Confirmar ou retirar Package | PackageEvent específico, autoria/tenant e projeção de situação corrente, caso adotada. | Audit/correção ligados ao fato; não enviar mensagem dentro da transação. |
| Registrar Vote | Fato Vote, pauta aberta, eligibility contextual e regra de unicidade aplicável. | Audit e voto coerentes; detalhes dependem de OD-01 e não definem segredo/ponderação. |
| Encerrar Assembly | Estado de Assembly/AgendaItem permitido e fatos de fechamento correspondentes. | Audit; apuração/publicação só separadamente conforme regra aprovada. |
| Registrar AccessEvent | Fato observado e metadados de origem/autoria/tenant conhecidos. | Audit conforme exigência; nenhuma autorização é criada retroativamente. |
| Habilitar/desabilitar Feature | Habilitação efetiva e auditabilidade da alteração no tenant correto. | Não apagar histórico, grants nem resolver operações ativas por inferência. |

Tabela define requisitos conceituais, não afirma que cada item é atualmente permitido. Decisões de negócio abertas continuam bloqueando transições correspondentes.

## Matriz de fronteira transacional por operação

“Fato principal” é uma mudança de estado ou ocorrência de domínio. Audit de sucesso obrigatório e o fato correspondente são indivisíveis: se um deles falha, nenhum pode ser confirmado. Audit de tentativa negada, quando exigido, registra o resultado negado e nunca descreve sucesso. Domain Event não implica persistência/publicação universal. Outbox só participa da unidade atômica quando publicação durável for requisito explícito; provider/canal nunca é chamado dentro da transação.

| Operação | Fato principal e validação | AuditLog obrigatório | Domain Event | Outbox/efeito assíncrono |
|---|---|---|---|---|
| Reservation approval/cancel | Decisão/transição aceita e disponibilidade/conflict revalidada no instante da aprovação. Cancelamento não apaga a confirmação histórica. | Fato + audit de sucesso atomicamente; tentativa negada separada conforme policy. | Confirmed/Cancelled somente para fato aceito. | Notification opcional sob OD-08; intenção/outbox atomicamente se entrega durável exigida; provider async. |
| Package confirm/release | `PackageEvent(CONFIRMED)` ou `PICKED_UP`, ator, tenant; situação derivada apenas se adotada. Confirmação não substitui retirada. | Evento + audit atomicamente; correção mantém referência ao original. | Evento específico do fato aceito. | Notification opcional; outbox atomicamente se entrega durável exigida; provider async. |
| Access event | `ENTRY`, `EXIT` ou `DENIED` realmente observado com instante, tenant, origem e actor conhecidos. | Observação + audit atomicamente quando obrigatório; tentativa negada não é sucesso. | AccessEventRecorded/Corrected após validação. | Somente reação externa durável aprovada; nunca cria Authorization retroativa. |
| RoleAssignment grant/revoke/suspend/reactivate | Estado/vigência alterados sob authority corrente; preservar autoria e histórico. | Mudança + audit obrigatórios atomicamente. | Evento correspondente à transição aceita. | Notification condicional; outbox atomicamente somente se entrega durável requerida. |
| Vote | Vote aceito em pauta aberta com eligibility contextual e unicidade conforme OD-01. | Vote + audit obrigatório atomicamente, respeitando sigilo/visibilidade aprovados. | VoteRecorded/Invalidated para fato/correção aceita. | Apuração/publicação não é automática; downstream somente após regra aprovada. |
| Assembly close | Transição de Assembly/AgendaItem e fatos de fechamento permitidos; não inclui apuração/publicação automaticamente. | Fechamento + audit atomicamente. | AssemblyClosed/AgendaItemClosed quando aceitos. | Notificação condicional; apuração/publicação são processos separados sob OD-01. |
| Feature enable/disable | Estado de CondominiumFeature/Module alterado no tenant alvo; operações em andamento não se resolvem por inferência. | Mudança + audit atomicamente. | Evento de habilitação/desabilitação aceito. | Notificação condicional; efeitos sobre operações em curso dependem de OD-11. |
| Notification creation | Intenção lógica, destinatário/finalidade/canais permitidos; criação não prova tentativa/entrega. | Intenção + audit atomicamente se audit for obrigatório para essa classe. | NotificationCreated se aceita. | Delivery é tentativa distinta; outbox atômica se tentativa durável for requisito; provider async. |

Fato de sucesso sem seu AuditLog obrigatório, ou AuditLog que afirme sucesso sem o fato, é estado arquiteturalmente inválido. Se a escrita do audit falhar, o fato dependente não pode ser confirmado. Uma falha/recusa não pode ser convertida em evento de sucesso por handler, retry ou resposta externa.

## Consistência sob concorrência

- Reservation: revalidar disponibilidade no instante de decisão e evitar confirmação dupla se política proíbe conflito; prioridade é OD-04.
- Vote: unicidade/substituição/segredo são OD-01; não sobrescrever votos por concorrência.
- Package: confirmar ocorrência, ator e fato distintos; não colapsar retirada concorrente como uma só; OD-06/18.
- RoleAssignment: não decidir a partir de autoridade/estado obsoleto; vigência e precedência seguem OD-02/14/18.
- AccessAuthorization: validar vigência contra o instante relevante; não restaurar autorização revogada/expirada por efeito de corrida; OD-05.
- AccessEvent/PackageEvent: preservar observações distintas; apenas replay comprovado do mesmo input/operation é repetição, nunca proximidade temporal.
- Feature disable: ordenar/revalidar a mudança frente a commands dependentes; não declarar operação concorrente válida com base em configuração já inválida.

A implementação deverá definir estratégia de consistência/integridade apropriada depois de decisões e requisitos de escala; este documento não prescreve lock, isolamento SQL, retry técnico ou mecanismo PostgreSQL.

Toda operação distingue validação/recusa, sucesso commitado e resultado de commit incerto (por exemplo, conexão perdida após o envio). Resultado incerto não autoriza mutação repetida às cegas: resolver pela identidade da operação/histórico e retornar resultado anterior ou conflito. Sem identidade confiável, exigir revisão e não reportar sucesso especulativo.

## Unit of Work

**Proposta:** a semântica da fronteira transacional pertence ao caso de uso Application; detalhes de begin/commit/rollback são executados por Infrastructure. Começar com contratos estreitos de execução transacional necessários a casos de uso, não adotar um Unit of Work universal por padrão.

Trade-off: uma abstração de Unit of Work explícita ajuda a agrupar repositories/audit/outbox sob uma transação, mas adiciona indirection e pode vazar session/ORM. Se vários adapters ou necessidade real de atomicidade coordenada surgirem, revisar a decisão mantendo API independente do GORM. Nenhum padrão concreto é escolhido.

## Side effects

Persistir aggregate/fato essencial, audit obrigatório e registro de publicação durável quando exigido pela política deve ter resultado coerente. Side effect externo ocorre após commit e pode falhar independentemente; falha é explícita e não reverte o fato de negócio já confirmado.
