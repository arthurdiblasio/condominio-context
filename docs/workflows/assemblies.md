# Assembleias, participação, procuração e votação

## WF-21 — Criar, convocar, conduzir e encerrar Assembly

### Objetivo

Registrar reunião, pauta, participação, presença e deliberações sem confundir convite, elegibilidade e voto.

### Participantes

Organizador/administrador com permission aplicável; Persons convocadas/participantes; representante sob Proxy; atores que registram presença e conduzem apuração.

### Contexto/Tenant

Assembly, AgendaItem, Participant, Proxy, Quorum e Vote pertencem ao mesmo Condominium. Pessoas podem participar sem que seus dados em outros tenants sejam consultados.

### Gatilho e pré-condições

Necessidade de convocar deliberação; critérios de convocação, agenda, período e procedimento identificados. Condições jurídicas não aprovadas impedem declarar validade legal do resultado.

### Permissões necessárias

`assembly.manage/read/open/close`, `participant.manage/read/presence.register`, `voting_eligibility.determine/read`, `proxy.register/revoke`, `vote.register/correct/cancel`, `quorum.read`, `assembly.result.read`, conforme cada ação. A autorização funcional não determina direito legal de votar.

### Dados/conceitos envolvidos

`Assembly`, `AgendaItem`, `Participant`, presença, `VotingEligibility`, `Proxy`, `Vote`, `Quorum`, resultado/ata, `Notification`, `AuditLog`.

### Fluxo principal

1. Ator autorizado cria Assembly no tenant, com data/local/dados exigidos por política.
2. Cria AgendaItems e associa materiais/dados permitidos; alterações posteriores ficam rastreáveis.
3. Convoca pessoas/unidades segundo procedimento aprovado; NotificationDelivery é registrada separadamente, falha não significa convocação legalmente inválida ou válida por si só.
4. Registra participantes/inscritos sem inferir presença.
5. Na assembleia, registra presença como fato distinto de convocação e elegibilidade.
6. Registra e valida Proxy no WF-22.
7. Para cada AgendaItem, determina VotingEligibility segundo regra identificável e autoridade aprovada.
8. Abre-se agenda/votação somente após conditions do procedimento aprovado.
9. Votantes elegíveis ou representantes reconhecidos registram votos; `Vote` se vincula à pauta, ator/votante e modo aplicável.
10. Fecha-se votação/assembleia segundo procedimento; apura-se Quorum e resultado apenas conforme regra aprovada.
11. Publica-se resultado/ata e histórico com acesso restrito; correções são rastreáveis.

### Fluxos alternativos

- Sem quorum: registrar condição e aplicar consequência definida; não apresentar resultado como válido sem regra.
- Pauta sem regra de elegibilidade: não habilitar voto/resultado definitivo; solicitar decisão humana.
- Proxy duplicado/conflitante: suspender decisão daquele representante/pauta e encaminhar revisão; não selecionar automaticamente.
- Voto secreto: não expor vínculo voto-votante além do permitido pelo procedimento aprovado.
- Cancelamento/adiamento: preservar presença, comunicações e fatos; notificar conforme policy.

### Validações e transições

Assembly candidate states `planned/open/closed/cancelled`; Vote candidate `pending/open/closed/cancelled`. Apenas abertura aprovada aceita voto; após fechar, novos votos não são aceitos sem processo de reabertura definido. Presença não implica elegibilidade; elegibilidade não implica voto; procuração não transfere elegibilidade automaticamente.

### Eventos/fatos e notificações

Assembly created/changed/convoked/opened/closed/cancelled; AgendaItem created/changed; Participant registered/present; Proxy submitted/validated/revoked/used; VotingEligibility determined; Vote registered/corrected/cancelled; quorum/result calculated. Notifications de convocação, alteração, encerramento e publicação conforme policy.

### Auditoria

Auditar criação e alteração de agenda, convocação relevante, presença/correção, decisão de elegibilidade, Proxy, abertura/fechamento, cada voto e correção/cancelamento, apuração e publicação. Registrar ator, tenant, pauta, período e regra identificada sem comprometer sigilo de voto além da política.

### Pós-condições/término

Assembly encerrada/cancelada com fatos e decisões preservados; resultado é apresentado como definitivo somente se regra aprovada foi aplicada. Em caso contrário, resultado fica pendente/não validado conforme estado definido.

### Erros/negações

Sem permission; tenant errado; agenda/assembly inexistente; período fechado; participante sem contexto; elegibilidade/proxy ambíguos; quorum/regra legal ausente; tentativa de voto duplicado ou fora de janela; módulo desabilitado.

### OPEN DECISIONS

OD-01: convocação, presença/quorum, elegibilidade por pauta, peso, inadimplência, Proxy, sigilo, opções, janela, correção, apuração e validade legal.

---

## WF-22 — Criar, validar, usar, expirar ou revogar Proxy

### Objetivo

Registrar representação delimitada em assembleia ou pauta, separada de delegação administrativa.

### Participantes

Outorgante, representante, validador autorizado conforme governance e participantes da Assembly.

### Contexto/Tenant

Condominium e Assembly/AgendaItem explicitamente relacionados. Proxy de um evento não é transferido para outra assembleia/tenant.

### Gatilho e pré-condições

Solicitação de representação; outorgante/representante identificados; período e escopo declarados; validade jurídica/política verificável.

### Permissões necessárias

`proxy.register/read/revoke`, `assembly.read`, scope de tenant e assembly; actor e permission não substituem a validade do instrumento.

### Dados/conceitos envolvidos

`Person` outorgante/representante, `Proxy`, Assembly, AgendaItem, validade, `VotingEligibility`, `AuditLog`.

### Fluxo principal

1. Outorgante ou ator autorizado solicita Proxy para escopo/período declarado.
2. Valida-se integridade do tenant, pessoas, Assembly/pauta, documento/forma exigida e limite de representação conforme regra aprovada.
3. Registra-se Proxy como pendente/validado segundo state vocabulary aprovado; aprovação cria direito apenas dentro daquele escopo.
4. No uso, verifica-se vigência, revogação, conflitos e regra de elegibilidade do representado.
5. Registra-se o representante presente/votante e referência ao Proxy sem confundir com RoleAssignment.
6. Expira-se ou revoga-se conforme validade/autoridade; fatos de uso permanecem.

### Fluxos alternativos e validações

Procuração duplicada, conflito entre representantes, escopo omitido, data expirada ou validade não determinável impede uso até revisão. Nova Assembly não herda Proxy. Revogação após voto não apaga voto sem procedimento específico.

### Eventos, notificações, auditoria e pós-condições

Proxy submitted/validated/rejected/used/revoked/expired; notificações a partes conforme policy. Auditar decisão, documento/contexto de validação e uso respeitando privacidade. Termina ativo dentro do escopo, rejeitado, expirado ou revogado.

### Erros/negações

Permission ausente; tenant/Assembly incompatível; validade/escopo insuficientes; duplicidade/conflito; regra legal não definida; tentativa de usar como grant administrativo.

### OPEN DECISIONS

OD-01 e OD-02: quem pode outorgar/validar, forma aceita, alcance por item/assembleia, limites, conflito, revogação e efeitos no voto.
