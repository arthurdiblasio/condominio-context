# Regras de encomendas e confirmações

## Conceitos

`Package` identifica uma encomenda destinada a pessoa/unidade no condomínio. `PackageEvent` preserva fatos operacionais. `Notification` e `NotificationDelivery` comunicam e registram tentativas, mas não substituem fatos do pacote.

## Regras

- `PACKAGE-01` Registrar `RECEIVED` somente quando a encomenda for fisicamente recebida no contexto do condomínio; guardar pessoa que recebeu, instante e ator que registrou quando conhecidos.
- `PACKAGE-02` O pacote deve ser associado ao condomínio e ao destino/destinatário identificável; resolução incerta não deve atribuir a encomenda silenciosamente a outra unidade.
- `PACKAGE-03` `NOTIFIED` representa comunicação iniciada/registrada; entrega efetiva depende de `NotificationDelivery`.
- `PACKAGE-04` `CONFIRMED` deve referenciar um pacote específico, o ator que confirma e o instante. A confirmação deve ocorrer sob contexto de autorização válido quando esse mecanismo for exigido.
- `PACKAGE-05` `PICKED_UP` é fato de retirada, distinto de confirmação. Registrar retirante e instante quando conhecidos.
- `PACKAGE-06` `CANCELLED` registra cancelamento contextual; não apaga recebimento, notificação, confirmação ou retirada anteriores.
- `PACKAGE-07` Correção de erro acrescenta informação corretiva rastreável; não apaga ou reescreve silenciosamente fato já registrado.
- `PACKAGE-08` Um evento posterior não pode inventar evento anterior ausente. Sequência e estado atual devem refletir fatos realmente conhecidos.

## Ciclo e sequência permitida

`RECEIVED`, `NOTIFIED`, `CONFIRMED`, `PICKED_UP` e `CANCELLED` são eventos históricos, não estados de Package. Como invariante, o ciclo de custódia começa com recebimento físico; `NOTIFIED` pressupõe pacote registrado; `CONFIRMED` e `PICKED_UP` referem-se ao mesmo pacote. Ver [state machine e situação derivada do Package](../state-machines/access-and-packages.md#package-status-derivado).

Caso se exponha uma situação atual, `IN_CUSTODY`, `RELEASED` e `CANCELLED` são projeções candidatas derivadas da sequência factual, não status finais aprovados. `PICKED_UP` continua sendo o evento de retirada, não o status. Se retirada pode ocorrer sem confirmação, se cancelamento é permitido após retirada, a precedência e como resolver destino desconhecido exigem política humana. Prazo de guarda e descarte não estão definidos.

## Confirmação: alternativas não escolhidas

Podem ser considerados link seguro, QR Code, confirmação por conta autorizada ou registro assistido pela portaria. O domínio não escolhe mecanismo ou provedor. Em qualquer alternativa, deve-se decidir como verificar autorização, associar evidência a pacote específico, registrar ator/instante e tratar contestação/correção.

Ver [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
