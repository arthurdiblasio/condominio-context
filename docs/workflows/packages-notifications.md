# Pacotes e notificações

## WF-18 — Receber e registrar Package

### Objetivo

Registrar custódia de encomenda na portaria e associá-la ao destino sem perder histórico.

### Participantes

Iniciador/registrador: `doorman` ou `employee` com grant `package.register`; destinatário e `Person`/Unit responsáveis.

### Contexto/Tenant

Condominium e Unit de destino explicitamente identificados e coerentes.

### Gatilho e pré-condições

Encomenda fisicamente recebida. Destinatário/unidade identificável conforme política; se incerto, não atribuir a uma unidade por suposição.

### Permissões necessárias

`package.register`; leitura posterior `package.read`; scope no tenant e operação de portaria. Ator autenticado e assignment válidos conforme PERMISSIONS.

### Dados/conceitos envolvidos

`Package`, `PackageEvent(RECEIVED)`, `Person`, `Unit`, funcionário recebedor/registrador, origem/descrição conforme minimização, `Notification`, `AuditLog`.

### Fluxo principal

1. Funcionário identifica Condominium, entrega e unidade/destinatário.
2. Registra recebimento físico como PackageEvent `RECEIVED`, separando quem recebeu de quem cadastrou quando forem pessoas distintas.
3. Registra instante/contexto e atributos de pacote aprovados.
4. Verifica se notificação é devida e cria Notification independente (WF-20); falha de entrega não reverte RECEIVED.
5. Pacote passa à próxima atividade operacional segundo regra vigente, mantendo todos os eventos anteriores.

### Fluxos alternativos e validações

Destino desconhecido ou ambíguo: manter sem atribuição final/encaminhar conforme policy; não notificar pessoa errada. Pacote recusado, danificado, sem destinatário ou duplicado exige procedimento local; não criar estados/eventos não aprovados.

### Transições, eventos e notificações

`RECEIVED` é fato, não state transition ou status consolidado. Notification é gatilhada por policy após registro; delivery status não muda PackageEvent.

### Auditoria

Auditar recebedor, registrador, timestamp, tenant, destino, alterações/correções e resultado. Proteger dados de destinatário/mercadoria conforme privacy.

### Pós-condições/término

Recebimento físico representado por evento rastreável e associado com destino conhecido ou pendência explícita.

### Erros/negações

Permission ausente; Unit de outro tenant; pacote não recebido fisicamente; destino ambíguo; campo excessivo/sem finalidade; evento duplicado ou estado incompatível.

### OPEN DECISIONS

OD-06/OD-07: campos, sem destinatário, pacote recusado/danificado, identificação duplicada, duração de custódia e retenção.

---

## WF-19 — Confirmar, contestar, corrigir e retirar Package

### Objetivo

Registrar separadamente confirmação de recebimento informada e retirada física da encomenda.

### Participantes

Destinatário Person ou representante permitido para confirmar/retirar; funcionário da portaria registra/observa retirada; contestante e responsável pela revisão conforme política.

### Contexto/Tenant

Package e destinatário no mesmo Condominium; conta e vínculo do agente são verificados quando ação autenticada. Link/QR/evidência são mecanismos possíveis, não definidos.

### Gatilho e pré-condições

Package registrado; mecanismo de confirmação ou retirada definido/validado; ator com permission e autoridade aplicáveis.

### Permissões necessárias

`package.read`, `package.confirm`, `package.release`, `package.correct`, `package.history.read` conforme ação. Destinatário necessita condição de relação/autorização e permission correspondente; matriz atual não resolve grant final por papel.

### Dados/conceitos envolvidos

`Package`, `PackageEvent(CONFIRMED/PICKED_UP/CANCELLED)`, `Person`, `UserAccount`, autorização de ação, evidência disponível, `AuditLog`, contestação.

### Fluxo principal

1. Ator seleciona um pacote específico no tenant autorizado.
2. Confirmação: valida-se autoridade do ator e associação ao pacote; registra-se `CONFIRMED`, ator, instante e evidência disponível.
3. Retirada: funcionário/observador registra `PICKED_UP`, retirante e instante quando conhecidos; confirmações não substituem retirada.
4. Contestação: abre-se revisão ligada aos fatos contestados, sem apagar nem reescrever eventos.
5. Correção: ator autorizado registra correção referenciando evento original e motivo; histórico original permanece visível conforme permissions.

### Fluxos alternativos e validações

- Confirmação pela portaria pode significar “entreguei ao destinatário” e deve ser distinguida de confirmação feita pelo destinatário; política semântica não está decidida.
- Link/QR inválido/expirado, ator sem permission ou token/contexto não validado: negar sem produzir CONFIRMED.
- Confirmação duplicada/retirada duplicada: não criar múltiplos fatos como se fossem eventos independentes sem validação; comportamento idempotente/rejeição requer decisão.
- Retirada sem confirmação pode ou não ser permitida segundo policy; não presumir dependência.

### Transições/eventos/notificações

CONFIRMED, PICKED_UP, CANCELLED são fatos PackageEvent distintos. Notificação à portaria/destinatário sobre confirmação/retirada é opcional e independente.

### Auditoria

Registrar ator real da confirmação/retirada, quem lançou o evento, timestamp, pacote, unit/tenant, evidência/referência aprovada, contestação e correção.

### Pós-condições/término

Fato confirmado/retirada registrado, contestação em acompanhamento ou ação negada. Estado atual derivado somente após regra definida; termina sem destruir a trilha.

### Erros/negações

Pacote inexistente ou de outro tenant; account inativa; destinatário/representante não autorizado; permission ausente; mecanismo inválido; estado incompatível; tentativa de reescrever evento.

### OPEN DECISIONS

OD-06: quem confirma/retira, papel da portaria, representantes, mecanismo, evidência, contestação, correção, confirmação duplicada e sequência obrigatória.

---

## WF-20 — Gerar Notification e acompanhar NotificationDelivery

### Objetivo

Comunicar evento de negócio e registrar separadamente cada tentativa/canal/resultado.

### Participantes

Gatilho: fato de negócio ou ação de ator com `notification.create`; Person destinatária; capacity/canal de entrega conforme configuração. Nenhum provider é participante de domínio.

### Contexto/Tenant

Notification conserva tenant e finalidade do evento; destinatário pertence ao tenant/contexto permitido. Não buscar fallback de contatos cross-tenant.

### Gatilho e pré-condições

Evento elegível à comunicação (ex.: package received, reservation decision, access authorization); finalidade/conteúdo permitidos e canal configurado.

### Permissões necessárias

`notification.create`, `notification.delivery.read` e `notification.channel.configure` quando a ação envolver configuração; permission e scope tenant-scoped. Processo automatizado continua sujeito a finalidade e configuração do tenant.

### Dados/conceitos envolvidos

Evento fonte, `Notification`, destinatário, tipo/conteúdo, `NotificationDelivery`, canais (Push, WhatsApp, email, in-app), preferences/opt-in, `AuditLog`.

### Fluxo principal

1. Evento de domínio elegível ocorre e fornece referência/origem.
2. Valida-se destinatário, finalidade, conteúdo mínimo, tenant, preference/opt-in aplicável e canais habilitados.
3. Cria-se Notification lógica sem afirmar entrega.
4. Para cada canal/tentativa selecionada pela política, registra-se NotificationDelivery.
5. Cada resultado é registrado apenas conforme evidência: attempted/submitted/delivered/read/failed.
6. Retry ou fallback, se configurado, cria tentativa adicional separada; falhas anteriores permanecem.
7. Fluxo de comunicação encerra quando policy declara tentativas terminadas; o workflow de negócio fonte continua independente.

### Fluxos alternativos e validações

Destinatário inválido, opt-out aplicável, conteúdo indevido, canal desabilitado ou tenant incompatível bloqueiam aquela entrega; o comportamento do evento fonte não é revertido. Se todos os canais falham, registra-se falha de delivery e preserva-se evento fonte. Opt-out pode ser limitado por finalidade/obrigação, a decidir.

### Transições/eventos e notificações

Notification é intenção; NotificationDelivery é tentativa. Gatilho deve sempre ser um evento/ação identificável. Delivery success/failure não é novo evento de acesso, confirmação ou aprovação.

### Auditoria

Auditar criação sensível, destinatário/tenant conforme minimização, canal, timestamps, resultado, retry/fallback e alterações de preferência/configuração sem armazenar segredo ou conteúdo excessivo.

### Pós-condições/término

Intenção e tentativas registradas com resultados conhecidos; nenhuma tentativa reportada como lida/entregue sem prova de canal.

### Erros/negações

Sem permission/finalidade; tenant incorreto; destinatário desconhecido; opt-in ausente quando requerido; canal indisponível/desabilitado; conteúdo ou estado inválido.

### OPEN DECISIONS

OD-08: prioridade, canais, opt-in/out, fallback, retry, sucesso/leitura, política de encerramento e retenção.
