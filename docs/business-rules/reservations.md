# Regras de áreas comuns, eventos e reservas

## Contexto

`Reservation` associa solicitante (Person e unidade relacionada), `CommonArea` e período pretendido. `Event`, `Visit` e convidados são relações opcionais segundo o uso e a configuração do condomínio.

## Regras

- `RESERVATION-01` Uma reserva pertence ao condomínio da unidade e área comum; essas referências devem pertencer ao mesmo tenant.
- `RESERVATION-02` O solicitante deve possuir autorização válida para criar ou gerir a reserva no escopo aplicável.
- `RESERVATION-03` Antes de confirmar, validar se a área e o período são utilizáveis e se as aprovações configuradas foram satisfeitas.
- `RESERVATION-04` Quando a política proibir sobreposição, não confirmar reservas incompatíveis no mesmo recurso/período.
- `RESERVATION-05` Reserva pendente não representa área confirmadamente ocupada, salvo regra local explícita sobre bloqueio provisório.
- `RESERVATION-06` Cancelamento, alteração de período e mudança de solicitante devem preservar histórico e gerar auditoria quando forem sensíveis.
- `RESERVATION-07` `Event` relacionado não autoriza acesso por si só; autorizações devem respeitar seu próprio escopo e validade.
- `RESERVATION-08` Desabilitar reservas ou bloquear área não apaga reservas nem ocorrências históricas.

## Configuração local

Cada condomínio/área pode adotar uma política aprovada como `AUTO_APPROVED`, `MANUAL_APPROVAL` ou `RULE_BASED`. Pode também configurar disponibilidade, manutenção/bloqueios, duração, antecedência, capacidade, convidados, cobrança e cancelamento. A política deve indicar qual condição leva ao estado confirmado; nenhum dos três modos é universal.

## Estados

Ver [state machine de Reservation e CommonArea](../state-machines/identity-and-reservations.md#reservation). Vocabulário em revisão: `DRAFT`, `PENDING`, `CONFIRMED`, `REJECTED`, `CANCELLED`, `COMPLETED`; `EXPIRED` somente para pedido pendente se houver prazo aprovado. `IN_PROGRESS` não é adotado como estado distinto no modelo-base. Não confirmar se período inválido, conflito proibido, área indisponível ou aprovação obrigatória ausente. Estados terminais não reabrem sem processo aprovado; bloqueio/desativação não cancela reserva silenciosamente.

Ver [WORKFLOWS.md](../../WORKFLOWS.md) e [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
