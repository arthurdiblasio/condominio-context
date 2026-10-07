# Canais de notificação

## Canais conceituais

- Push
- WhatsApp
- E-mail
- In-app

## Regras

- `Notification` contém destinatário, finalidade/tipo e conteúdo lógico; pode originar múltiplas `NotificationDelivery`s;
- canal e resultado/status pertencem a cada `NotificationDelivery`, não à intenção lógica da mensagem;
- o WhatsApp deve ser documentado como canal possível, não como implementação obrigatória.

Ver [state machines de Notification e NotificationDelivery](../state-machines/communications-and-assemblies.md#notification) para separar intenção, tentativa, submissão, entrega e leitura.

## Decisão pendente

Provedor, templates e política de opt-in continuam abertos.
