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

## Consistência sob concorrência

- Reservation: revalidar disponibilidade no instante de decisão e evitar confirmação dupla se política proíbe conflito; prioridade é OD-04.
- Vote: unicidade/substituição/segredo são OD-01; não sobrescrever votos por concorrência.
- Package: confirmar ocorrência, ator e fato distintos; não colapsar retirada concorrente como uma só; OD-06/18.
- RoleAssignment: não decidir a partir de autoridade/estado obsoleto; vigência e precedência seguem OD-02/14/18.
- AccessAuthorization: validar vigência contra o instante relevante; não restaurar autorização revogada/expirada por efeito de corrida; OD-05.

A implementação deverá definir estratégia de consistência/integridade apropriada depois de decisões e requisitos de escala; este documento não prescreve lock, isolamento SQL, retry técnico ou mecanismo PostgreSQL.

## Unit of Work

**Proposta:** a semântica da fronteira transacional pertence ao caso de uso Application; detalhes de begin/commit/rollback são executados por Infrastructure. Começar com contratos estreitos de execução transacional necessários a casos de uso, não adotar um Unit of Work universal por padrão.

Trade-off: uma abstração de Unit of Work explícita ajuda a agrupar repositories/audit/outbox sob uma transação, mas adiciona indirection e pode vazar session/ORM. Se vários adapters ou necessidade real de atomicidade coordenada surgirem, revisar a decisão mantendo API independente do GORM. Nenhum padrão concreto é escolhido.

## Side effects

Persistir aggregate/fato essencial, audit obrigatório e registro de publicação durável quando exigido pela política deve ter resultado coerente. Side effect externo ocorre após commit e pode falhar independentemente; falha é explícita e não reverte o fato de negócio já confirmado.
