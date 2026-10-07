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

Não há prioridade universal, fallback obrigatório, número ou intervalo de retry, canal obrigatório, nem estado global da Notification definido. Resultados candidatos de `NotificationDelivery`: `created/attempted`, `submitted`, `delivered`, `read`, `failed`. Eles se aplicam à tentativa e não devem ser promovidos a estado de todas as entregas. Sem evidência de leitura, não declarar lida.

Provedor, templates, custo, retenção, opt-in, opt-out, canal preferencial e fallback permanecem abertos.
