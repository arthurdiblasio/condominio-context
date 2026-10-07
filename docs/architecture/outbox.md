# Avaliação conceitual de Transactional Outbox

## Necessidade

Quando um caso de uso precisa persistir um fato e garantir que uma reação assíncrona futura seja tentada, gravar apenas o estado e publicar externamente depois pode perder o evento se o processo falhar entre commit e publicação. Publicar antes do commit pode anunciar fato que não foi persistido.

Um Transactional Outbox é um padrão candidato para registrar, na mesma unidade atômica da persistência de negócio, a intenção durável de encaminhar eventos selecionados. Um publicador/worker futuro processaria esses registros fora da transação principal.

## Benefícios e custos

**Benefícios**

- alinha persistência do fato e intenção de publicação;
- permite retomada após indisponibilidade externa;
- mantém providers/brokers fora do domínio;
- facilita retries observáveis sem fingir que entrega ocorreu.

**Custos**

- armazenamento e ciclo de vida adicional;
- processamento de duplicatas, ordenação, retenção, falhas e operação da fila;
- necessidade de idempotência nos consumers e monitoramento de atrasos;
- uma garantia de publicação/entrega ainda não significa entrega final ao destinatário.

## Workflows candidatos

- `PackageReceived` que cria Notification e requer tentativa de entrega durável;
- `ReservationConfirmed` e mudanças notificáveis;
- mudanças sensíveis de RoleAssignment cuja comunicação for exigida por política;
- expiração/lembretes e outros jobs somente quando aprovados.

Não é requisito para queries, alterações puramente internas sem reação externa ou eventos cujo processamento eventual não seja exigido.

## Proposta de escopo

**Recomendação para MVP, condicional:** se NotificationDelivery assíncrona confiável fizer parte do primeiro release, adotar o princípio de outbox para o conjunto limitado de efeitos externos que precisa sobreviver a falhas entre commit e publicação. Não adotar um event bus universal nem armazenar indiscriminadamente todos os eventos do domínio em outbox.

Se não houver requisito de entrega durável no MVP, manter a decisão adiada e preservar separação de interfaces para introduzi-la sem alterar Domain. Produto deve definir quais notificações são essenciais; arquitetura deve então escolher mecanismo e semântica operacional.

## Garantias e limites

- Outbox preserva a intenção de publicação junto ao fato; não garante entrega única por rede.
- Consumers devem tolerar repetição sem duplicar efeito de negócio; a semântica depende de OD-08/18.
- Resultado de envio precisa de evidência e fica em NotificationDelivery.
- Erro do provider não reverte Package, Reservation, RoleAssignment ou outro fato persistido.
- Audit obrigatório continua distinto da mensagem outbox.

Nenhum schema, tabela, fila, broker ou formato de mensagem é definido.
