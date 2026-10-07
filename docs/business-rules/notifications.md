# Regras de notificações

## Conceitos

- `Notification`: conteúdo/intenção lógica e destinatário.
- `NotificationDelivery`: uma tentativa e seu resultado por canal.
- Canal não é provedor. WhatsApp, push, e-mail e in-app são possibilidades, não dependências.

## Regras

- `NOTIFICATION-01` Uma notificação identifica destinatário, finalidade/tipo e conteúdo autorizado.
- `NOTIFICATION-02` Cada tentativa pertence a uma notificação e registra canal, momento e resultado conhecido.
- `NOTIFICATION-03` Uma notificação pode ter zero, uma ou várias entregas por canais diferentes conforme política aprovada.
- `NOTIFICATION-04` Nova tentativa ou canal alternativo gera registro separado e não apaga falhas anteriores.
- `NOTIFICATION-05` `submitted`, `delivered` ou `read` só pode ser afirmado se o canal oferecer evidência suficiente; tentativa de envio não equivale a entrega ou leitura.
- `NOTIFICATION-06` Falha em canal não altera automaticamente estado do evento de negócio que motivou a notificação.
- `NOTIFICATION-07` Mensagem e destinatário devem respeitar finalidade e visibilidade do tenant e não expor dados pessoais desnecessários.
- `NOTIFICATION-08` Opt-in/opt-out aplicável deve ser respeitado quando estabelecido pela política do canal/uso; exceções e base legal precisam de validação humana.

## Prioridade, fallback e ciclos

Ver [state machines de Notification e NotificationDelivery](../state-machines/communications-and-assemblies.md#notification). A Notification representa intenção; resultado agregado de entregas não é estado canônico no modelo-base. Cada tentativa inicia `PENDING`; `SUBMITTED` não equivale a `DELIVERED`, e leitura só pode ser declarada com evidência. Retry cria uma nova tentativa, sem status `RETRYING` nem sobrescrita da falha anterior.

Não há prioridade universal, fallback obrigatório, número ou intervalo de retry, canal obrigatório nem validade de tentativa definidos. `CANCELLED` e `EXPIRED` dependem de política. Provedor, templates, custo, retenção, opt-in, opt-out, canal preferencial e fallback permanecem abertos.

Provedor, templates, custo, retenção, opt-in, opt-out, canal preferencial e fallback permanecem abertos.
