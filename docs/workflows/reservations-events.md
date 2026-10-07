# Áreas comuns, reservas e eventos

## WF-12 — Configurar/bloquear CommonArea

### Objetivo

Disponibilizar, configurar, bloquear por manutenção ou encerrar uso de área comum no tenant.

### Participantes

Iniciador/executor candidato `condominium_admin`, `syndic` ou `property_manager`; `common_area.manage` no scope adequado é obrigatório.

### Contexto/Tenant

CommonArea pertence a um único Condominium; se localizada em Block, o Block pertence ao mesmo tenant.

### Gatilho e pré-condições

Criação da área, alteração de regras ou disponibilidade, manutenção/bloqueio, reabertura ou encerramento.

### Permissões necessárias

`common_area.manage`, leitura `common_area.read`; configurar políticas pode exigir `reservation.approve`/permission administrativa local conforme matriz aprovada.

### Dados/conceitos envolvidos

`Condominium`, `Block` opcional, `CommonArea`, regras locais, reservas associadas, `AuditLog`.

### Fluxo principal

1. Selecionar tenant e área.
2. Registrar identificação e somente os atributos aprovados.
3. Configurar disponibilidade, horários, capacidade, política de aprovação e requisitos de convidados conforme decisão local.
4. Antes de bloqueio/manutenção, identificar reservas afetadas e seguir política de comunicação/cancelamento; não alterar reserva silenciosamente.
5. Bloquear/manter a área por período/contexto informado; após condição de reabertura aprovada, restaurar disponibilidade sem apagar histórico.

### Fluxos alternativos/validações

Area inexistente, tenant inconsistente, conflito com reservas confirmadas, capacidade/política inválida ou falta de permission impedem alteração. Impacto em reservas futuras requer decisão da autoridade e política.

### Transições, eventos, notificações

Estados de disponibilidade propostos (`ACTIVE`, `BLOCKED`, `MAINTENANCE`, `DISABLED`) e impacto sobre reservas estão descritos em [CommonArea lifecycle](../state-machines/identity-and-reservations.md#commonarea). Fatos: area created/configured/blocked/reopened/closed. Notificações para reservas afetadas somente segundo policy/canal; bloqueio não altera reservas por inferência.

### Auditoria e pós-condições

Auditar configuração anterior/nova, ator, tenant, motivo operacional e reservas afetadas. Termina quando configuração é aceita e disponibilidade resulta explícita ou operação é negada.

### OPEN DECISIONS

Capacidade, horários, política auto/manual/rule-based, bloqueio e tratamento de reservas impactadas.

---

## WF-13 — Solicitar, aprovar, cancelar ou concluir Reservation

### Objetivo

Alocar CommonArea a uma Unit/solicitante durante período, respeitando configuração local, disponibilidade e conflitos.

### Participantes

Iniciador: Person com relação à Unit e capability `reservation.create` em contexto. Aprovador: ator com `reservation.approve` se política exigir; executor de cancelamento conforme grants.

### Contexto/Tenant

Condominium da Unit e CommonArea deve ser o mesmo. Scope pode ser Unit/Condominium com cobertura explícita.

### Gatilho e pré-condições

Solicitação de período; CommonArea disponível; Person/Unit válidas; configuração de reserva ativa e módulo/feature habilitado quando exigido.

### Permissões necessárias

`reservation.create/read/update/cancel/approve` conforme ação. Papel `owner/resident/tenant` sozinho não concede capability; vínculo e configuração são condições adicionais.

### Dados/conceitos envolvidos

`Person`, `Unit`, `CommonArea`, `Reservation`, `Event` opcional, convidados, `AccessAuthorization`, configuração local, `Notification`, `AuditLog`.

### Fluxo principal

1. Solicitante escolhe área, período, unidade e fornece detalhes requeridos pela política.
2. Valida-se tenant, vínculo, janela de antecedência, duração, capacidade e disponibilidade conforme configuração.
3. Detecta-se conflito com reservas e bloqueios; política define se uma reserva pendente bloqueia temporariamente.
4. Aplica-se modo aprovado: `AUTO_APPROVED`, `MANUAL_APPROVAL` ou `RULE_BASED`. Os nomes são modos, não comportamento universal.
5. Em modo manual, registra-se pending e direciona-se ao aprovador autorizado; confirmação ocorre somente após aprovação e disponibilidade persistente.
6. Em automático/baseado em regra, avaliam-se condições configuradas; se satisfeitas, confirma-se; caso contrário nega-se ou permanece pendente conforme regra definida.
7. Alterar/cancelar preserva histórico, revalida conflito e aplica prazos/efeitos aprovados.
8. Concluir/expirar quando regras de período assim determinarem; isso não cria AccessEvent nem confirma presença.

### Fluxos alternativos

- Área bloqueada/manutenção: negar ou manter não confirmada; não oferecer como confirmada.
- Duas solicitações concorrentes pelo mesmo slot: resolver conflito sem confirmar ambas quando sobreposição for proibida; critério de precedência permanece aberto.
- Configuração/inadimplência: aplicar somente se regra local/legal aprovada; senão não inferir bloqueio.
- Convidados e acesso: seguem WF-14/WF-15/16; reserva não é autorização de acesso.

### Validações e transições

Lifecycle de referência: [Reservation](../state-machines/identity-and-reservations.md#reservation). Estados em revisão: `DRAFT`, `PENDING`, `CONFIRMED`, `REJECTED`, `CANCELLED`, `COMPLETED`; `EXPIRED` apenas para pedido pendente sob prazo aprovado. `IN_PROGRESS` não é estado base. Só confirmar após validações e approvals exigidas. Mudança de período reexecuta validações. Cancelamento/conclusão/expiração não apaga eventos ou notificação anterior.

### Eventos/fatos e notificações

Reservation requested, pending, approved/confirmed, changed, cancelled, completed/expired; event associado pode ser criado/alterado separadamente. Notification de solicitação, aprovação, recusa, alteração ou cancelamento só conforme política. Falha na entrega não desfaz Reservation.

### Auditoria

Auditar criação, aprovação/recusa, alteração de período/solicitante, cancelamento, exceção, conflito resolvido e ator aprovador.

### Pós-condições/término

Reserva confirmada somente se regra aplicável satisfeita; ou pendente com responsável/condição conhecida; ou encerrada/negada com reason/contexto permitido.

### Erros/negações

Permission/vínculo ausente; scope/tenant errado; área/Unit inexistente; recurso bloqueado; horário/conflito/capacidade/antecedência inválidos; approval ausente; feature desabilitada; regra não definida.

### OPEN DECISIONS

OD-04 e OD-10: conflito concorrente, precedência, duração, antecedência, capacidade, pendências que bloqueiam, inadimplência, cancelamento, cobrança, reembolso e consequência de área bloqueada.

---

## WF-14 — Event e pessoas convidadas

### Objetivo

Associar contexto de evento e participantes convidados à Reservation ou Visit quando aplicável, sem tratar Guest como identidade nem conceder acesso automaticamente.

### Participantes

Organizador com `event.manage` ou permission local; Person convidada. Visitante convidado não executa gestão por mera participação.

### Contexto/Tenant

Event e Reservation/Visit relacionados pertencem ao mesmo Condominium; anfitrião/Unit deve ser validado nesse tenant.

### Gatilho e pré-condições

Reserva confirmada/planejada ou visita/evento que requer contexto; workflow de criação e alterações autorizado.

### Permissões necessárias

`event.read/manage`, `visit.register` e permissions de autorização de acesso se usadas; scope e vínculo aplicáveis.

### Dados/conceitos envolvidos

`Event`, `Reservation`, `Person` como convidada, `Visit`, `AccessAuthorization`, `AuditLog`.

### Fluxo principal

Criar/associar Event; cadastrar Persons convidadas conforme finalidade/campos mínimos; alterar ou cancelar convite/evento de forma rastreável; criar AccessAuthorization separadamente se exigida. Janela de autorização respeita período permitido pelo contexto; Event por si só não libera entrada.

### Alternativas/validações/transições

Duplicidade de pessoa/convite não resolve identidade automaticamente. Alteração/cancelamento do evento exige tratamento explícito de autorizações ainda ativas; não cancelar acesso ou apagar Visit por inferência.

### Eventos, notificações, auditoria, pós-condições

Event created/changed/cancelled, Guest association changed, authorization requested quando ocorrer. Notification aos convidados conforme policy; auditar operações. Termina com evento/associações consistentes ou negação.

### Erros/negações

Tenant incompatível, sem permission, pessoa/área inexistente, período inválido, autorização fora da janela, conflito de estado ou decisão ausente.

### OPEN DECISIONS

Dados e limite de convidados, evento obrigatório para reservar, efeito de cancelamento de Event e validade temporal de autorizações associadas.
