# Visitas e controle de acesso

## WF-15 — Planejar e conduzir Visit

### Objetivo

Conectar pessoa visitante, visita planejada, autorização (se exigida) e ocorrências físicas, sem confundir os conceitos.

### Participantes

Anfitrião (Person vinculada a Unit), Person visitante, `doorman` ou funcionário autorizado para registrar fatos e ator com authority para emitir authorization.

### Contexto/Tenant

Um Condominium e Unit anfitriã no mesmo tenant. Se Event/Reservation estiver associado, deve pertencer ao mesmo contexto.

### Gatilho e pré-condições

Visita planejada ou chegada identificada. Exigir autorização somente segundo regra local; policy de identificação e dados mínimos deve estar aprovada.

### Permissões necessárias

`visit.register`, `visit.read`, `access_authorization.read`, `access_event.register` conforme tarefas e grants; anfitrião só pode usar `access_authorization.create` se grant e política permitirem.

### Dados/conceitos envolvidos

`Person`, `Visit`, `Unit`, anfitrião, `Event`/`Reservation` opcionais, `AccessAuthorization`, `AccessEvent`, `AuditLog`.

### Fluxo principal

1. Planeja-se/associa-se Visit a pessoa e anfitrião/contexto.
2. Se policy exigir, cria-se e valida-se AccessAuthorization no WF-16.
3. Na chegada, funcionário avalia identidade/contexto e autorização se exigida, sem inferir identidade de conta.
4. Se entrada observada/permitida, registra-se `AccessEvent ENTRY`.
5. Durante permanência, Visit permanece contexto; autorização não representa presença contínua nem cria saída.
6. Ao sair, registra-se `AccessEvent EXIT` se observado.
7. Visit termina segundo procedimento local; o histórico de eventos permanece.

### Fluxos alternativos

- Sem autorização válida quando exigida: tentativa registrada `DENIED`; não criar ENTRY.
- Autorização existe, mas ninguém chega: não criar ENTRY; autorização pode expirar.
- ENTRY sem EXIT registrado: deixar a ausência explícita; não presumir saída.
- EXIT sem ENTRY no histórico: registrar como divergência operacional, sem criar/reescrever ENTRY.
- Visitante não identificável ou autorização pendente: aplicar procedimento local; não converter pending em active automaticamente.

### Validações e transições

`Visit` não é estado de autorização; autorização não é fato de acesso. O uso deve estar dentro do tempo/escopo autorizado. Lifecycle proposto de Visit e de AccessAuthorization está em [state machines de visita e acesso](../state-machines/access-and-packages.md). Access fact é ENTRY/EXIT/DENIED; chegada/permanência não são inferidas da autorização.

### Eventos/fatos e notificações

Visit planned/registered/changed/cancelled; AccessEvent ENTRY/EXIT/DENIED quando observado. Notificação a anfitrião ou unidade é configurável, não pré-condição universal.

### Auditoria

Auditar criação/alteração da visita, autorização consultada/alterada, eventos registrados/corrigidos, ator, momento, tenant, local/contexto e resultado; limitar dados pessoais.

### Pós-condições/término

Visita encerrada conforme procedimento local ou permanece com status de permanência não resolvido se saída não observada. Nenhum evento físico é inventado.

### Erros/negações

Scope/tenant incorreto; permission ausente; Visit/Unit inexistente; authorization inexistente/inválida/expirada/revogada quando exigida; identidade inconclusiva; estado operacional divergente.

### OPEN DECISIONS

OD-05/OD-07: quando autorização é exigida, dados de identificação, exceções, duração, saída não registrada e retenção.

---

## WF-16 — Criar, ativar, usar, revogar ou expirar AccessAuthorization

### Objetivo

Expressar permissão temporal e contextual para acesso; não registrar a passagem física.

### Participantes

Solicitante com grant e regra local, pessoa autorizada como alvo, approver se exigido, `doorman` como verificador.

### Contexto/Tenant

Condominium identificável; Unit responsável quando aplicável; Event/Reservation/Visit opcional dentro do mesmo tenant.

### Gatilho e pré-condições

Pedido de acesso temporário; pessoa/critério de identificação aprovado, escopo, período e finalidade disponíveis.

### Permissões necessárias

`access_authorization.create/read/revoke` e permission de aprovação se definida; scope Unit/Block/Condominium compatível. A permission do ator não transfere permission à pessoa autorizada.

### Dados/conceitos envolvidos

`Person`, `Unit`, `Visit`, `Event`, `Reservation`, `AccessAuthorization`, `AccessEvent`, `AuditLog`.

### Fluxo principal

1. Solicitante define pessoa-alvo, responsável, escopo, período e associação opcional.
2. Valida-se que referências pertencem ao mesmo tenant e Event/Reservation não amplia a janela permitida.
3. Aplica-se aprovação/configuração local; estado active só após condições satisfeitas.
4. Na tentativa de uso, funcionário consulta autorização vigente; resultado físico é separado no WF-17.
5. Após término do período, autorização expira para novas entradas; autoridade pode revogar/cancelar conforme regras.

### Fluxos alternativos e validações

Autorização pode existir sem Visit ou sem ENTRY. Período inválido, referência de outro tenant, revogação ou expiração negam novo uso. Entrada observada fora de autorização não reescreve autorização; registra ocorrência aplicável.

### Transições

Vocabulário de referência: `DRAFT → ACTIVE → EXPIRED`, com saídas `REVOKED`/`CANCELLED` conforme política. `DRAFT` só se houver validação; autorização sem revisão pode tornar-se `ACTIVE` após as guardas satisfeitas. Entrada/saída não transforma automaticamente autorização em `USED`/consumida. Revogação/expiração não remove AccessEvents anteriores.

### Eventos, notificações, auditoria e pós-condições

Authorization created/activated/revoked/expired/cancelled; notificar anfitrião/autorizado somente conforme policy/canal. Auditar ator, motivo/contexto, tenant, período e resultado. Termina com autorização efetiva/negada/cancelada e história preservada.

### Erros/negações

Falta de permission/approval; scope/tenant errado; pessoa ou referência inválida; período ausente/fora da janela; estado incompatível; feature desabilitada se exigida.

### OPEN DECISIONS

OD-05: emissor/aprovador, janela inclusiva, uso único/repetido, exceções, dados mínimos e relação obrigatória/opcional com Visit/Event/Reservation.

---

## WF-17 — Registrar ENTRY, EXIT, DENIED ou corrigir AccessEvent

### Objetivo

Preservar fatos observados de acesso e divergências de portaria.

### Participantes

`doorman`/`employee` autorizado registra; administrador operacional com `access_event.correct` pode propor/efetuar correção conforme policy.

### Contexto/Tenant

Condominium/ponto operacional e pessoa/autorização/contexto conhecidos; nenhuma associação entre tenants deve ser inferida.

### Gatilho e pré-condições

Observação real de tentativa/entrada/saída ou descoberta de erro no histórico.

### Permissões necessárias

`access_event.register/read`; correção requer `access_event.correct`, scope e authority distinta/definida.

### Dados/conceitos envolvidos

`AccessEvent`, `Person` se identificada, `Visit`, `AccessAuthorization`, `Unit`, source/actor, `AuditLog`.

### Fluxo principal

1. Funcionário avalia autorização quando policy exige.
2. Registra evento correspondente ao fato: ENTRY, EXIT ou DENIED; coleta só contexto permitido.
3. ENTRY exige condições de autorização quando requeridas; DENIED não altera authorization para ativa.
4. Divergência entre entrada/saída é registrada como ocorrência, sem fabricar evento faltante.
5. Correção referencia evento original, motivo e autor e cria trilha corretiva.

### Fluxos alternativos/validações

Sem authorization: DENIED se tentativa rejeitada, ou registrar exceção operacional autorizada sem declarar que havia grant; exceções precisam de regra. EXIT pode ocorrer sem ENTRY prévio. Duplicidade possível não deve apagar primeira observação; aplicação de deduplicação depende de decisão.

### Transições/eventos/notificações

AccessEvent é append-only conceitual; correção é novo fato/annotation rastreável, não transição de authorization. Notificar responsável/gestor somente conforme ocorrência e policy.

### Auditoria/pós-condições

Auditar cada evento e correção, identificador de ator, tenant, instante e resultado. Termina com fato registrado ou ocorrência encaminhada sem alteração falsa.

### Erros/negações

Sem permissão; tenant/ponto incorreto; recurso inexistente; timestamp/contexto inconsistente; tentativa de editar/apagar histórico; estado ou exceção não aprovada.

### OPEN DECISIONS

OD-05/OD-07: fonte e confiança do evento, correção, duplicate events, tratamento de exceção e retenção.
