# Architecture Critical Review

## 1. Executive Summary

Esta revisão adversarial cruza o modelo de domínio, regras, permissões, workflows, state machines, API Contract e os documentos de arquitetura. A avaliação é documental: não há aplicação para testar e nenhum comportamento de runtime é afirmado como vulnerabilidade comprovada.

A direção geral — monólito modular, domínio independente do ORM, autorização contextual, tenant explícito, comandos semânticos e ausência de tecnologias distribuídas prematuras — é pragmática. Entretanto, a documentação ainda descreve controles críticos como princípios, sem especificar invariantes de implementação suficientemente obrigatórias. O maior risco é um desenvolvedor interpretar TenantContext como filtro opcional, carregar recurso pelo ID antes de verificar sua titularidade ou criar um caminho alternativo que não passe pela autorização central. Um segundo grupo de riscos está na consistência entre fato de negócio, AuditLog e outbox, e na falta de critérios fechados para corridas, replay e idempotência.

Não recomendo iniciar operações autenticadas/sensíveis do MVP antes de fechar o contrato de isolamento por tenant, o enforcement único da autorização, a atomicidade exigida para auditoria/fatos e as invariantes de concorrência. Isso não impede trabalho documental ou scaffolding neutro, mas bloqueia a implementação funcional desses caminhos.

**Assessment:** `NEEDS REVISION`  
**Status final:** `ARCHITECTURE_REQUIRES_REVISION`

## 2. Overall Assessment

### NEEDS REVISION

Os princípios estão majoritariamente coerentes e evitam microservices, Event Sourcing, CQRS completo e cache/broker prematuros. A proposta ainda não está pronta para ser tratada como blueprint de implementação porque:

- os repositórios devem receber tenant, mas falta tornar impossível ou detectável a omissão no contrato de acesso a dados;
- o caminho de autorização é descrito, mas o limite técnico obrigatório para todos os comandos e jobs precisa de regras testáveis;
- atomicidade de AuditLog, fato principal e intenção de publicação é condicional/ambígua;
- cenários críticos de concorrência não têm invariantes nem resultados suficientemente identificados;
- classificação de risco `NORMAL`/`ELEVATED`/`CRITICAL` solicitada não está definida;
- idempotência identifica resultados candidatos, mas não define identidade da intenção nem cobertura de todos os commands;
- algumas decisões de produto (grants, conta, assembleia, custódia) são bloqueios específicos para seus workflows e não devem ser tratados como requisitos resolvidos pela arquitetura.

As lacunas representam exposição potencial se uma futura implementação interpretar os princípios permissivamente; não são prova de falha em código inexistente.

## 3. Critical Findings

| ID | Área | Problema | Gravidade | Impacto | Recomendação |
|---|---|---|---|---|---|
| AR-01 | Tenant isolation | Tenant é obrigatório conceitualmente, mas filtros de persistência continuam descritos como responsabilidade da Application; não há contrato anti-omissão ou defesa em profundidade decidida. | CRITICAL | IDOR/tenant escape, leitura ou alteração de dados de outro condomínio por ID válido. | Tornar todo acesso operacional tenant-scoped por construção/contrato; proibir caminhos unscoped por padrão; validar ownership de referências e criar testes negativos cross-tenant. Avaliar controles adicionais na persistência sem decidir RLS/schema nesta revisão. |
| AR-02 | Autorização | O “ponto de enforcement” está definido conceitualmente, mas não há regra explícita de que todo command, incluindo jobs e operações assistidas, só possa ser alcançado por um use case autorizado nem de como capability ausente bloqueia exposição. | CRITICAL | Bypass de autorização, privilege escalation ou confused deputy. | Exigir uma única fronteira de execução de commands; classificar operações e manter gaps como não disponíveis até decisão de authority/capability. |
| AR-03 | Audit e transação | AUDIT é obrigatório em ações sensíveis, mas a arquitetura só exige co-commit “quando a regra exige inseparabilidade”; isso deixa a garantia de ação-sucedida-sem-AuditLog ambígua. | HIGH | Perda de evidência após crash/falha e impossibilidade de reconstruir ação privilegiada. | Definir como invariant arquitetural que AuditLog obrigatório e mudança/fato sensível confirmada são atômicos, ou especificar mecanismo durável equivalente antes da implementação. |
| AR-04 | Concorrência | Transações dizem quais casos coordenar, mas não fecham invariantes para corrida entre confirmação/liberação, aprovação/alteração, voto/fechamento ou habilitação/desabilitação. | HIGH | Dupla reserva/retirada/votação, autorização baseada em estado obsoleto ou transição proibida. | Para cada command crítico, definir estado prévio, pós-condição, condição de conflito e resultado de retry. Deixar lock/constraint/versionamento técnico para ADR posterior. |
| AR-05 | Idempotência e replay | `IDEMPOTENT`, `NO-OP`, `REJECT`, `NEW_EVENT` não definem como identificar a mesma intenção após timeout/restart; há comandos importantes sem semântica específica. | HIGH | Duplicar fatos, tentativas de Notification, auditoria ou efeitos externos, ou apagar uma ocorrência legítima ao deduplicar. | Definir identidade de operação/evento e semântica por command antes de contratos executáveis; não deduplicar fatos físicos por semelhança temporal. |
| AR-06 | Decisões de autorização | A matriz e várias capabilities ainda são propostas/gaps, inclusive UserAccount, rejeição/cancelamento, Proxy, correção, release e apuração. | HIGH | Implementar defaults como se fossem grants aprovados ou bloquear operações essenciais no meio do desenvolvimento. | Resolver governança/capabilities para cada operação que será exposta; manter outras fora da superfície executável. |
| AR-07 | Jobs e eventos assíncronos | Actor, finalidade e tenant são citados, mas não há modelo operacional completo para principal de processo, revogação, retries, retomada após restart e autorização no momento da execução. | HIGH | Job executar com tenant errado, authority obsoleta ou privilégio implícito de `system`. | Definir envelope/contexto confiável por trabalho e regra de revalidação na execução; nunca herdar contexto de request nem usar bypass global. |
| AR-08 | Identidade e account | A autorização depende de `UserAccount + Person`, mas cardinalidade, associação e lifecycle seguem OD-03; a autenticação é igualmente não decidida. | HIGH | Principal ambíguo, autoria incorreta ou direitos associados à pessoa errada. | Bloqueia operações autenticadas até identidade/associação e atribuição de autoria estarem decididas; não precisa bloquear documentação ou subsistemas sem login. |
| AR-09 | Application / Domain | Application coordena autorização, carregamento, aggregates, transação, repositories, auditoria, eventos e integrações sem critério explícito para evitar “God Layer”; inversamente, “solicitar decisões do domínio” pode deixar regras em services anêmicos. | MEDIUM | Regra duplicada em use cases ou lógica de negócio sem dono/teste no Domain. | Separar orquestração de invariantes por critérios: Application coordena; agregado/policy protege decisão de domínio reutilizável e transição. Revisar cada caso, sem criar abstrações genéricas. |
| AR-10 | API errors | A lista do prompt contém `UNAUTHORIZED`, enquanto o catálogo vigente separa `UNAUTHENTICATED` e `FORBIDDEN`; há risco de consumidores e camadas tratarem autenticação e autorização como a mesma falha. | MEDIUM | Semântica inconsistente, exposição indevida ou retries/UX incorretos. | Tratar `docs/api/errors.md` como fonte atual, manter distinção conceitual e documentar a correspondência de erros internos/externos sem definir HTTP. |
| AR-11 | Tempo | Clock está previsto, mas não há modelo de distinção entre instante UTC, data civil, horário local/timezone, período inclusivo e expiração. | HIGH | Reservas, autorizações, assembleias e expirações divergem por fuso, DST ou bordas de período. | Definir vocabulário temporal e política de timezone antes dos workflows com agendamento/validade; manter a escolha de biblioteca aberta. |
| AR-12 | Privacidade / operações | Minimização é registrada, mas retenção, exclusão/anonimização, backups, log técnico, dados de notificação e documentos continuam sem ciclo de vida operacional. | MEDIUM | Exposição excessiva e incapacidade de responder a política de retenção aprovada. | Definir classificação/finalidade e requisitos de ciclo de vida com responsáveis competentes; não inventar prazo legal nem bloquear código que não processe essas classes. |
| AR-13 | Audit fields | A regra define actor, ação, recurso, instante, tenant, resultado/contexto; motivo, estado anterior/posterior e correlação são condicionais, não há contrato mínimo por classe de operação. | MEDIUM | Auditoria incompleta para grant, correção, pacote, acesso e assembleia. | Criar catálogo mínimo de campos/eventos por operação sensível e política de minimização; não registrar payload completo por padrão. |
| AR-14 | Cobertura de workflows | Use cases estão mapeados por grupos WF, mas faltam vínculos rastreáveis por workflow a transação, audit, idempotência, erro e efeitos; WF-25/26 e transições internas de WF-21/22 têm cobertura especialmente agregada. | MEDIUM | Operação pode ficar sem responsável arquitetural ou ser implementada com semântica divergente. | Completar a matriz como checklist de implementação para os workflows que entrarem no MVP, referenciando API traceability e state machines sem duplicar regras. |

## 4. Tenant Isolation Findings

### O que está bem definido

- Um único tenant explícito por operação tenant-scoped.
- Tenant solicitado é candidato não confiável até resolução em Application.
- Tenant do recurso e das referências deve ser comparado com o contexto.
- `Person` pode participar de vários condomínios sem fundir relações.
- Cross-tenant exige authority, purpose e auditoria explícitos.
- Cache, jobs e chamadas externas não devem compartilhar contexto entre tenants.

Ver [TENANCY rules](docs/business-rules/tenancy.md), [TenantContext](docs/architecture/tenant-context.md), [authorization](docs/architecture/authorization.md) e [infrastructure](docs/architecture/infrastructure.md).

### Problema principal: controle lógico sem contrato fail-closed na persistência

`TenantContext` diz que Application inclui tenant em toda leitura/escrita, e Infrastructure diz que repository “não deve aceitar tenant omitido”. São princípios, não definição de API de persistência, restrição por default, ou regra para operação global. Se alguém criar uma consulta por ID sem filtro ou esquecer uma referência associada, o desenho não mostra uma barreira que rejeite a operação.

**Recomendação:** todo port para dado operacional tenant-scoped deve exigir contexto/alvo no contrato e declarar explicitamente operações globais/cross-tenant excepcionais. O resultado carregado precisa ser revalidado contra tenant e referências relacionadas antes de autorização/mutação. Nenhum ID externo é autorização. Avaliar depois uma camada de defesa adicional de persistência; não escolher RLS, schema-per-tenant ou outra estratégia sem análise técnica.

### Cenários adversariais

| Cenário | Risco atual | Controle recomendado antes do código |
|---|---|---|
| Usuário de A fornece ID válido de B | HIGH/CRITICAL: comparação está em princípio, não há contrato de query por tenant fechado. | Carregamento scoped por tenant mais verificação explícita de ownership; resposta não revela existência de B. |
| Usuário com grants em A e B alterna contexto | HIGH: contexto precisa ser por operação, nunca “tenant atual” global do processo/request reutilizado. | Resolver e autorizar cada chamada; não cachear tenant em principal mutável ou estado compartilhado. |
| Mesma Person em A e B | MEDIUM/HIGH: modelagem separa tenants, mas query global por Person pode combinar vínculos. | Query sempre ancora relações no tenant selecionado; operações globais de identidade são exceção explícita e minimizada. |
| Unit de outro condomínio referenciada por Reservation/Package | HIGH: tenant do recurso pai não basta se referências forem inconsistentes. | Validar ownership tenant de todas as referências no caso de uso e preservar integridade na persistência. |
| Recurso sem tenant explícito | HIGH: não há regra arquitetural detalhada para classificar objetos globais, derivados ou tenant-scoped. | Catálogo de tenancy por recurso antes de persistência; ausência de tenant só para global documentado. |
| Repository sem filtro | CRITICAL: documento pede tenant, mas construção não é demonstrada/impossível de omitir. | Proibir port genérico/unscoped para dados operacionais e adicionar testes de contrato negativos. |
| Job sem TenantContext | HIGH: referência diz não herdar contexto, mas identidade de processo e autorização continuam abertas. | Contexto versionado por mensagem/tarefa; revalidar tenant, purpose, authority e estado no momento de executar. |
| Evento assíncrono sem tenant/correlation | HIGH: handlers devem preservar contexto, sem formato obrigatório ou campo mínimo fechado. | Envelope de publicação com origem, tenant, correlation e finalidade mínimos; validar consumer antes do efeito. |
| Cache compartilhado | MEDIUM: classificação é conservadora; invalidação/tenant-key não especificados. | Sem cache de dados tenant/sensíveis no MVP; qualquer exceção precisa de chave tenant-aware, teste e invalidação segura. |
| Busca/relatório global | HIGH: queries conceituais exigem filtro; relatórios, exports e operações de plataforma podem abrir agregação. | Tenant filter antes de paginação/agregação/export; cross-tenant requer permissão específica e propósito. |
| Identificador enviado diretamente pela API | HIGH: recurso não deve ser autorizado por ID; API minimiza erro cross-tenant, mas implementação pode fazer lookup global. | Não usar lookup global seguido de autorização tardia; escopo de busca restringe o objeto e a resposta minimiza divergência. |

### Quem seleciona o tenant

O cliente pode selecionar ou indicar um candidato, mas não pode tornar essa seleção confiável. Application valida que a identidade e grant cobrem aquele tenant. Decidir como o tenant é transportado é técnico e permanece aberto; nenhuma fonte de contexto pode, sozinha, autorizar o tenant.

## 5. Authorization Findings

### Modelo

`Subject + RoleAssignment + Permission + Scope + Resource + Context` é uma base apropriada, mas não suficiente como checklist sem: estado da conta, finalidade, feature habilitada, contexto temporal, consistência do tenant e regra de negócio. Esses itens aparecem em outros trechos e devem ser tratados como parte obrigatória da decisão, não gates opcionais.

Authentication responde quem apresentou a identidade; Authorization responde se a ação/resource/context está permitido; Domain valida se a ação é legítima naquele estado. Interface pode rejeitar cedo, mas nunca deve ser o único enforcement. Domain pode ser chamado sem autenticação para testes e regras puras; o limite de segurança é que comandos só são expostos pelos casos de uso que sempre executam autorização (ou por uma autoridade de processo explicitamente definida).

### Risco por operação

| Operação | Risco | Lacuna/condição |
|---|---|---|
| `role_assignment.grant` / `revoke` | CRITICAL | OD-02/14/15/16: autoridade do concessor, delegação, autoelevação, conflitos e efetividade. Revisar grant concorrente com revogação e revalidar authority no commit. |
| `audit_log.read` | HIGH | AUDIT-10 exige purpose/scope; risco de platform admin ou filtro amplo acessar dados de todos os tenants. Paginação/export precisam do mesmo enforcement de query. |
| `access_event.correct` | HIGH | Deve referenciar fato original e tenant/scope, restringir authority revisora, reter original e auditar motivo/resultado. Não é simples update de evento. |
| `package.release` | HIGH | Validar destinatário/representante e autoridade conforme OD-06; não confundir `package.confirm` com `package.release`; operação repetida/concorrente precisa preservar evidência. |
| `reservation.approve` | HIGH | Permission não substitui policy de reserva, disponibilidade, tenant, autoridade de approve/reject e concorrência (OD-04). |
| `participant.presence.register` | HIGH | Presença factual distinta de convidado, proxy e eligibility; ator/ponto/assembleia corretos e duplicidade conforme OD-01/18. |
| `vote.register` | CRITICAL | OD-01 bloqueia regra legal, elegibilidade, unicidade, segredo, proxy e janela; não tornar command disponível como consequência de existir `vote.register`. |

A arquitetura não define classes `NORMAL`, `ELEVATED`, `CRITICAL` nem controles graduados (step-up, dupla validação ou revisão). Não criar tais controles por suposição: primeiro classificar operações com produto/segurança e resolver a governança aplicável.

### Feature, papel e relacionamento

Feature habilitada não é permission; `Role` não é `RoleAssignment`; relacionamento UnitOwnership/Residency/Tenancy não é acesso. Essas distinções estão consistentes em API e domínio, mas devem ser verificadas em cada command e query, principalmente funções de portaria e leitura de pessoa.

## 6. Domain / Infrastructure Findings

- **GORM contamination:** arquitetura afirma separação de persistence model, mapper e Domain; risco atual baixo no documento. O futuro perigo são tags, hooks, soft-delete, zero values, preload/lazy-loading, tipos de erro e sessão/transação ORM introduzidos no agregado. Recomendação: mapper explícito apenas onde há divergência; teste do Domain sem DB; nenhuma regra em hook.
- **Transactions leakage:** contratos de Application não devem receber `*gorm.DB`, transaction handle ou tipo PostgreSQL. Infrastructure executa a transação conforme caso de uso.
- **Domain anêmico:** nomes de aggregates estão listados, mas não há regra textual obrigando transições/invariantes a serem protegidas pelos agregados/policies. A lista de casos de uso pode crescer para serviços que carregam toda regra. Registrar critério de colocação da lógica antes de implementação.
- **Repositories:** evitar GenericRepository é correto. Ainda não existe catálogo de query/use-case e ownership de port explícito para cada módulo; interface grande é risco, não defeito presente. Ports de consulta complexa podem ser específicos de Application em vez de forçar Domain a receber consultas arbitrárias.
- **IDs e timestamps:** persistence fields, versão/concurrency token e timestamps podem contaminar conceitos se adotados como requisitos de domínio sem decisão. IDs não são capability nem segredo.
- **Soft delete:** o modelo exige histórico preservado, não um mecanismo universal de soft-delete. Não substituir evento corretivo/auditoria por flag ORM ou apagar fisicamente sem política de retenção aprovada.
- **Business errors:** Infrastructure deve classificar falhas técnicas conhecidas sem expor SQL/driver nem reclassificar como regra de negócio; camada API minimiza cross-tenant.

Ver [domain independence](docs/architecture/domain.md), [layers](docs/architecture/layers.md) e [infrastructure](docs/architecture/infrastructure.md).

## 7. Transaction & Concurrency Findings

### Cenários exigidos

| Corrida | Invariante a preservar | Camada principal | Transação? / decisão pendente |
|---|---|---|---|
| Duas pessoas reservam mesma área/período | Se a policy proibir sobreposição, no máximo as confirmações compatíveis previstas pela policy. | Domain aplica regra; Application coordena decisão; persistência garante não aceitar decisão concorrente obsoleta. | Sim para decisão + fato + Audit obrigatório. OD-04 define conflito/prioridade; técnica de concorrência fica para ADR de implementação. |
| Dois funcionários liberam o mesmo pacote | Nenhuma retirada duplicada do mesmo ato; fatos reais distintos não podem ser fundidos. | Pacote/Domain e caso de uso; confirmação concorrente revalida estado factual. | Sim para a decisão que classifica a ocorrência e fato/audit. OD-06/18 definem semântica e evidência. |
| Duas entradas simultâneas | Registrar cada observação física legítima; não deduplicar duas pessoas/atos distintos nem fabricar saída. | Access Domain/use case; fonte operacional fornece identidade de observação quando disponível. | Persistir fato/audit coerentes; uma “unicidade por pessoa/tempo” não está aprovada. OD-05/18. |
| Dois grants simultâneos | Nenhum ator concede acima de authority; transições e conflitos/duplicatas preservados. | Authorization/Application revalida concessor, alvo, scope e estado. | Sim para grant + audit; OD-02/14/15/16/18. |
| Dois votos simultâneos | Voto só em pauta aberta e eligibility reconhecida; regra de unicidade/substituição não duplicada. | Assemblies Domain/use case com estado atual da pauta e decisão de eligibility. | Sim para voto, guardas e audit; exclusividade/voto secreto dependem de OD-01/18. |
| Fechar assembleia enquanto alguém vota | Voto aceito ou fechamento deve obedecer a um ponto de ordenação consistente; nenhum voto após encerramento efetivo. | Um boundary de consistência comum para Assembly/AgendaItem e Vote. | Sim para guarda e escrita; OD-01 define fechamento/reabertura; definir ordem/conflict result antes de código. |
| Alterar Reservation enquanto outra requisição aprova | Approve valida versão/período/área atuais, não os dados lidos antes de uma alteração concorrente. | Reservation Domain/Application mais consistência no armazenamento. | Sim; mudança/cancelamento/approve revalidam; OD-04/18 definem conflitos e resultado. |
| Habilitar/desabilitar Feature enquanto command dependente ocorre | Command deve ser aceito conforme política em vigor num ponto consistente; não começar operação incompatível após disable efetivo. | Application avalia availability; persistence coordena a decisão concorrente. | Sim para mudança de toggle e operação protegida segundo regra futura; OD-11/17 decide operações já iniciadas. |

### Falha do processo e atomicidade

- **Antes do commit:** command não pode comunicar sucesso; qualquer tentativa externa antecipada pode produzir mensagem para fato não ocorrido.
- **Após commit, antes de evento/notificação:** sem outbox, reação assíncrona pode se perder; outbox resolve publicação durável selecionada, não entrega final.
- **Após evento publicado, antes de marcar processado:** pode haver replay; consumers e NotificationDelivery precisam tolerar repetição.
- **Após fato, antes de AuditLog:** proibido para ações cuja auditoria é obrigatória; declarar atomicidade ou garantia equivalente antes do primeiro command sensível.
- **Retry após timeout do cliente:** cliente pode não saber se o commit ocorreu; sem identidade estável da operação, repetir command pode duplicar fato. Definir estratégia sem pressupor idempotency key já escolhida.

### Fronteira e Unit of Work

Conceito de transação por caso de uso é coerente; Unit of Work universal não é necessário por moda. Contudo, ARCH-06 não fecha como um caso coordena repository + Audit + outbox sem sessão GORM no Application. Antes do código, descrever garantia observável (quais registros são atômicos); depois selecionar abstração mínima e mecanismo.

## 8. Events & Outbox Findings

### Vocabulário e ownership

- Command nasce na entrada/ator e pode ser rejeitado.
- Domain Event nasce da transição aceita; Domain não conhece broker nem Notification provider.
- Operational Event nasce de observação, como ENTRY ou RECEIVED; eventos do mundo físico não são automaticamente idempotentes por chave lógica.
- AuditLog nasce da execução/decisão, inclusive tentativas negadas quando exigido; não prova que o fato de negócio ocorreu.
- Notification nasce de uma policy de comunicação; NotificationDelivery representa tentativa e resultado, não o evento de origem.
- Outbox, se aprovada, é mecanismo durável de publicação em Infrastructure/coordenação Application, não entidade do domínio nem sinônimo de Domain Event.

O documento [events.md](docs/architecture/events.md) separa bem esses conceitos. A lacuna é o ciclo de persistência/consumo: quais facts são persistidos, quais são publicados, quais são síncronos, campos mínimos de tenant/correlation/origem, retenção, falha e reprocessamento.

### Avaliação da Outbox

Outbox é justificável onde há requisito de não perder a intenção de comunicação/reação depois do commit. Candidatos são PackageReceived e ReservationConfirmed quando notificação confiável for MVP, e grants somente se policy exigir comunicação. Não usar outbox universal para cada evento, reads, audit duplicativo ou fato sem consumer.

Uma tentativa direta pós-commit não garante durabilidade. Enviar antes do commit pode criar falso positivo. Outbox oferece publicação retomável, geralmente com possibilidade de duplicidade; não promete entrega “exactly once” ao canal. Notificação entregue depende de evidência do provider e permanece NotificationDelivery.

Antes de incluir no MVP, produto deve decidir se a notificação é essencial e qual consequência de indisponibilidade é aceitável. Se for essencial, persistir intenção/publicação junto ao fato e definir consumer idempotente, tenant/origem/correlation, backoff/retentativa observáveis e dead-letter/manual review em decisão técnica posterior. Credenciais de provider ficam exclusivamente em Infrastructure/secrets management futuro.

## 9. Audit & Observability Findings

### AuditLog

As regras permitem responder a actor, ação, instante, tenant/scope, resource e resultado. Cobertura nominal inclui grants, correções, pacote, acesso, assembleia, habilitação e consulta de audit. Ainda precisam ser padronizados por risco: estado anterior/novo, motivo, finalidade, request/correlation, autoridade efetivamente verificada e fonte de ator (humano/processo).

Operações de leitura de AuditLog devem ser filtradas antes de paginação/export, exigir purpose e própria auditoria, sem acesso implícito de `platform_admin`. Log técnico não deve conter payload completo nem ser usado como AuditLog.

### Logs, traces e correlação

Correlation ID está previsto até handlers, porém não há regra de formato/fonte/propagação em retry, outbox, job e provider callback. Propagar correlação sem usar como auth/tenant selector; em retry manter ligação à operação de origem e emitir correlação da tentativa.

Redigir CPF, telefone, endereço, token/credential, payload de voto e documentos por padrão. Tenant ID é dado contextual; registrar apenas quando necessário em log com acesso controlado, evitando labels de alta cardinalidade em métricas. Definir retenção e redaction de logs separadamente da política de AuditLog.

Nenhuma solução de logging/tracing foi selecionada.

## 10. API Contract Consistency

| Tema | Consistência | Lacuna |
|---|---|---|
| Commands/Queries | API command/query-first consistente com Application; sem CRUD genérico e sem CQRS completo. | Há mapping por operação e gaps de permission; categorias e cobertura transaction/idempotency não estão completas para todos os comandos. |
| Authorization | Ambas exigem subject, assignment, Permission, Scope, Resource, contexto e condições. | Identidade UserAccount/Person e permissões são decisões/gaps; deployment não pode expor comando de capability não aprovada. |
| Tenant | Contrato exige tenant único e minimiza vazamento; arquitetura faz tenant candidate → validação. | Falta port de repositório tenant-mandatory e tratamento explícito de recurso global versus tenant-scoped. |
| Errors | Taxonomia API é rica e deliberadamente independente de HTTP. | “UNAUTHORIZED” do checklist do anexo diverge; catálogo atual distingue UNAUTHENTICATED e FORBIDDEN e inclui `ACCOUNT_INACTIVE`, `SCOPE_MISMATCH`, `AUDIT_REQUIRED`, `DECISION_REQUIRED` etc. Consolidar correspondência antes da API executável. |
| Resources/transitions | Internal/fact resources não são CRUD; state machines protegem eventos e fatos. | API lista correção/cancelamento/retry em algumas áreas sem capability fechada; devem ficar indisponíveis, não implicitamente autorizadas. |
| Side effects | Notification independente e falha não reverte negócio; compatível. | Reliability/async/outbox e comportamento de timeout ainda não decididos. |
| Versioning/pagination | Contrato prevê compatibilidade, ordenação e paginação sem escolher mecanismo. | São decisões necessárias antes de consumidores públicos estáveis, mas não bloqueiam modelagem interna. |

Não foram encontrados endpoints concretos ou escolhas HTTP incompatíveis no desenho conceitual. A revisão recomenda considerar [API Contract](API-CONTRACT.md) como fonte única para categorias de erro e mapeamento de permission.

## 11. Workflow Consistency

`docs/architecture/application.md` mapeia WF-01..WF-26 por grupos, portanto nenhum workflow está totalmente sem um contexto conceitual. Isso é uma cobertura de catálogo, não prova de implementação completa. A matriz em [docs/api/traceability.md](docs/api/traceability.md) fornece Permission, scope, fato, regra e side effect; WORKFLOWS contém guardas, decisões, auditoria e resultados.

Lacunas de arquitetura que devem ser fechadas por workflow antes de colocá-lo em execução:

- **WF-01, WF-04/05, WF-09:** bootstrap, capabilities de conta, autoridade/concorrência de grants e identidade de actor.
- **WF-06–08:** datas, sobreposição, correção e sua consistência/audit.
- **WF-12/13:** bloqueio de área vs reservas, concorrência de approve/alteração/cancelamento e decisão de conflito.
- **WF-14–17:** capability de convite/cancelamento, aprovação de Authorization, prova/fonte do evento e correção.
- **WF-18/19:** identificação de ocorrência, liberação concorrente, retries e resultados de comunicação.
- **WF-20:** retries e status de Delivery após falha/restart; capability para cancelar/retry.
- **WF-21/22:** elegibilidade, presence/vote/proxy, fechamento concurrente, autoridade de cálculo/publicação; OD-01 bloqueia ações deliberativas.
- **WF-23:** corrida de toggle com uso ativo e fronteira entre catálogo global e habilitação do tenant.
- **WF-24:** fora de MVP candidato; módulo opcional e OD-12.
- **WF-25/26:** queries/export de audit e operações cross-tenant precisam de purpose explícito, minimização, trilha própria e não revelação.

Para cada workflow do primeiro release, produzir/validar uma linha de entrega com use case, Permission, tenant/scope, transição, regra, fronteira transacional, audit, erro, idempotência, efeitos/retries e testes negativos. Evitar duplicar o conteúdo normativo já mantido em workflow/API.

## 12. State Machine Consistency

O princípio **“Events are not states”** está preservado nos documentos de Domain, state machines e arquitetura:

- `AccessEvent(ENTRY/EXIT/DENIED)` e `PackageEvent(RECEIVED/NOTIFIED/CONFIRMED/PICKED_UP)` são fatos;
- `NotificationDelivery` é tentativa/resultado separado da intenção Notification;
- `VoteRecorded` não é apuração/publicação; presença não é eligibility;
- estados/projeções derivados de Package não foram declarados como estado canônico;
- correção deve referenciar o original, sem apagar ocorrência.

Risco residual: [transactions.md](docs/architecture/transactions.md) cita situação/projeção de Package e “estado efetivo” de Assignment sem dizer se projeções serão reconstruíveis e não autoritativas. Manter fato como fonte de verdade e marcar projeção explicitamente; evitar persistir estado derivado como segunda authority sem regra de reconciliação.

Eventos não podem ser “corrigidos” por update/delete ORM nem estado terminal reaberto por simples PATCH; comandos corretivos precisam originar fato/audit e respeitar transições.

## 13. Security Findings

### Riscos arquiteturais

- **IDOR/tenant escape:** principal risco AR-01; IDs nunca substituem authorization nem ownership check.
- **Privilege escalation:** grant/revoke exige comparar authority do concessor em cada Scope e revalidar concorrência; autoelevação/delegação pendentes.
- **Confused deputy:** serviços de plataforma, jobs e handlers não devem agir em nome de tenant sem resource, purpose e grant/process authority explicitados.
- **Context forgery:** tenant, Person, role e correlation vindos de request são entrada não confiável até serem associados à identidade autenticada e grants atuais.
- **Replay/duplicate:** retry de comando e reentrega de evento exigem identity/dedup com semântica específica; fato físico pode ser ocorrência nova.
- **Background jobs:** não herdar sessão/principal de request nem executar `system` como superusuário implícito.
- **Audit integrity:** evitar que o mesmo caminho mutável altere a fonte do AuditLog ou apague fato original; mecanismos concretos de imutabilidade e administração ainda abertos.
- **Secrets/provider credentials:** permanecer em Infrastructure; nunca em domínio, evento público, logs, API error ou Notification payload auditado sem necessidade.
- **Enumeration:** negar com resposta minimizada para recurso cross-tenant; não distinguir recurso inexistente de invisível quando isso revelar informação.

Não se escolhem autenticação, key management ou mecanismos de segurança técnicos nesta revisão. Testes de segurança futuros devem incluir autorização horizontal/vertical, troca de tenant, replay, revogação concorrente, acesso por job e minimização de erros.

## 14. LGPD / Privacy Architecture Findings

### Requisitos arquiteturais evidentes (sem estabelecer obrigações legais)

- Tenant isolation e acesso por purpose/least privilege devem ser aplicados em APIs, relatórios, notificações, logs, jobs, storage e cache.
- Dados necessários devem ser minimizados por caso de uso e audiência; portaria não precisa de projeção completa de Person.
- Dados de voto, acesso, visita, vínculo e documentos precisam de regras explícitas de visibilidade.
- Correções e retenção devem preservar evidência necessária sem transformar histórico em retenção indefinida.
- Logs técnicos, mensagens, traces, backups e arquivos externos são superfícies próprias de exposição e precisam de acesso/redaction/retention.
- Exclusão, anonimização e correção devem respeitar referências e obrigações de negócio definidas, sem inferência de destruição universal.

### Decisões humanas/jurídicas a manter abertas

Finalidade/base apropriada, categorias sensíveis, visibilidade por papel, direitos dos titulares, prazo de retenção, eliminação/anonimização, backups, documentação/evidências e notificações precisam de revisão competente (OD-07). Esta revisão não fixa prazo legal, base jurídica ou obrigação de consentimento.

## 15. Anti-Pattern Assessment

| Anti-pattern | Risco atual | Evidência | Gravidade | Recomendação |
|---|---|---|---|---|
| Big Ball of Mud | Baixo no desenho; risco de crescer se camadas forem apenas diretórios. | Há módulos e dependências desenhados, mas sem contrato de import/ownership executável. | MEDIUM | Manter boundaries verificáveis e evitar import cruzado arbitrário. |
| Anemic Domain Model | Médio. | Application “solicita decisões do domínio”; regras e invariantes ainda estão distribuídas nos workflows e não há critérios de comportamento agregado. | MEDIUM | Domain/policies protegem invariantes; Application não replica as regras. |
| God Service | Médio/alto. | Application assume autorização, repositories, transactions, audit, events, outbox, integrações. | HIGH | Dividir casos de uso por workflow e coordenação transversal por ports pequenas; não centralizar tudo em um serviço genérico. |
| God Repository | Baixo/médio. | Repositories específicos são recomendados; leituras e muitas referências podem inflar interfaces. | MEDIUM | Interfaces estreitas por necessidade e query-specific; sem repository transversal que conheça todo domínio. |
| Generic Repository | Baixo pelo princípio declarado, risco de conveniência de ORM. | Documento explicitamente rejeita `GenericRepository<T>`. | LOW | Preservar posição e não expor CRUD genérico no Domain/Application. |
| Service Layer Explosion | Médio. | Cada command/query mapeia um use case; pode virar wrapper sem regra/valor. | MEDIUM | Criar caso de uso por intenção/limite transacional, não por entidade/CRUD automaticamente. |
| ORM Domain Model | Baixo no documento, alto se tags/hooks forem introduzidos em Domain. | GORM explicitamente isolado, mas sem enforcement/teste futuro. | MEDIUM | DTO/persistence model separado quando necessário; testes Domain sem ORM. |
| Distributed Monolith | Baixo atualmente. | Proposta de monólito modular, sem serviços/deploy separados. | LOW | Manter módulos no mesmo boundary de deployment até requisito real. |
| Premature Microservices | Baixo. | Explicitamente postergados. | LOW | Não separar deploy ou banco por módulo agora. |
| Premature CQRS | Baixo. | Commands/Queries só separados semanticamente; CQRS completo rejeitado. | LOW | Manter leituras no mesmo sistema até requisito mensurável. |
| Event Soup | Médio. | Catálogo amplo de nomes de eventos sem envelope, consumers e lifecycle definido. | MEDIUM | Publicar somente facts com consumers/requisito; distinguir event interno, operacional, audit e notification. |
| Shared Database Coupling | Baixo no monólito, médio futuro. | PostgreSQL/GORM único candidato e módulos não devem importar tabelas alheias; sem ownership de dados formal. | MEDIUM | Monólito pode compartilhar DB, mas módulos não devem escrever diretamente em tabelas uns dos outros; ownership conceitual por módulo. |
| Authorization Bypass | Alto. | Enforcement na Application é princípio, mas caminhos alternativos/automação ainda sem principal definido. | HIGH | Todos os command entry points passam pelo mesmo authorization/use case boundary. |
| Tenant Leakage | Alto. | Tenant filtro obrigatório por convenção, isolamento de persistência ainda não escolhido. | CRITICAL | AR-01: contrato fail-closed, ownership checks e testes; defesa adicional a avaliar. |

## 16. Decisions Required Before Coding

### BLOCKER

Blockers são limitados a código que efetivamente exerça as operações correspondentes; não impedem protótipos documentais ou infraestrutura neutra.

1. **Contrato de tenant-scoped access:** classificar recursos globais vs tenant-scoped, exigir tenant em cada acesso operacional e decidir tratamento de lookup por ID, referencias e relatórios; testar ausência/mismatch. Não é necessário escolher RLS para fechar este contrato.
2. **Fronteira de autorização:** garantir que todo command/query sensível, incluindo job e operação assistida, passa por authorization checks; definir capabilities/grants aprovados para operações a expor. Resolver OD-02/14/15/16 conforme escopo.
3. **Principal e actor:** para qualquer operação autenticada, decidir associação UserAccount–Person, estado, attribution e tratamento de processo/ator operacional (OD-03). Provedor/método de autenticação pode continuar separado.
4. **Auditoria obrigatória e atomicidade:** listar operações auditáveis no release e garantir que não exista sucesso sem o registro obrigatório; definir minimum fields por operação sensível.
5. **Invariantes de concorrência/replay dos commands do MVP:** aprovar semântica para Reservation, Package, RoleAssignment, AccessAuthorization e qualquer rotina habilitada; impedir voto/apuração enquanto OD-01 não estiver resolvida.
6. **Semântica temporal usada pelo MVP:** definir time zone de calendário do Condominium, representar instante vs data/hora local e fronteiras inclusivas/exclusivas de validade/agendamento usadas por reservas/acesso. Não requer escolha de biblioteca.

### IMPORTANT

1. Escolher formato de IDs antes de contratos públicos/persistência estável; não os usar como proteção.
2. Definir error mapping conceitual estável, incluindo autenticação, autorização, tenant invisível, conflitos e falha técnica; manter fora de HTTP status.
3. Decidir confiabilidade exigida das NotificationDelivery do MVP e, então, adotar ou postergar Outbox para efeitos selecionados.
4. Definir redaction, logging access, correlation across job/event/provider e retenção técnica.
5. Aprovar política de privacy/retention por categoria antes de expor dados pessoais, logs de portaria, votos, arquivos ou exportações.
6. Definir check de transação de AuditLog/Domain fact e como efeitos falhos são relatados ao cliente após commit.
7. Para uma operação que deve ser retomada após timeout, escolher semântica da identidade/replay antes de publicar seu contrato a clientes.

### LATER

1. Redis/cache distribuído, sujeito a requisito demonstrado.
2. Elasticsearch/OpenSearch; consultas atuais não demonstram necessidade além de PostgreSQL.
3. Broker específico, scheduler e topologia de worker; primeiro definir requisitos de entrega, volume e operação.
4. Deployment por serviço, multi-region e particionamento, sem evidência atual que justifique.
5. Otimizações de Unit of Work ou projeções separadas/CQRS completo.
6. Formato físico de storage e estratégia detalhada de backups, antes de implementar upload/documentos; requisitos de privacidade/ciclo de vida devem ser definidos antes de receber esses dados.

## 17. ADR Review

| Decisão | Está suficientemente definida? | Justificativa / consequência | Alternativas consideradas | Dependências e revisão |
|---|---|---|---|---|
| ARCH-01 — Monólito modular | Parcialmente. Define estilo, não regra verificável de modularidade ou critérios de extração. | Simplicidade operacional, transação local e boundaries conceituais; consequência é disciplina contra acesso cross-module ao storage. | Monólito não modular; microsserviços. | Não exige decisão de produto; acrescentar checks/ownership ao iniciar o repositório de aplicação. Manter proposta. |
| ARCH-02 — Commands/Queries sem CQRS completo | Suficiente como direção inicial, não como medição de escalabilidade. | Evita CQRS por moda e mantém leitura/escrita simples. | CRUD indiferenciado; CQRS completo/read store. | Reabrir se workloads divergirem por evidência. Manter proposta. |
| ARCH-03 — Application orquestra, Domain decide | Parcialmente; pode tornar Application um God Layer ou Domain anêmico. | Centraliza autorização e sequência sem levar regras ao handler. | Domain Service genérico; lógica em controllers; use cases finos. | Revisar boundaries e ownership de autorização/transação/eventos com critérios explícitos. |
| ARCH-04 — Audit origina-se dos use cases | Parcialmente; localização está clara, atomicidade e campos mínimos não. | Evita contorno via outro entry point; impacto de ausência de Audit pode ser crítico. | Audit em middleware; log best-effort separado; transação atômica. | Resolver AR-03/13 antes de commands sensíveis. Revisar e fortalecer. |
| ARCH-05 — Outbox condicional | Parcialmente; gatilho é “publicação durável necessária”, mas não qual comunicação é necessária no MVP. | Evita perda entre commit e publish sem impor broker. | Síncrono pós-commit; retry in-memory; outbox limitada; event bus universal. | Produto define promessa de entrega (OD-08); em seguida fechar escolha técnica. Manter condicional. |
| ARCH-06 — Ports específicas; sem GenericRepository/UoW universal | Parcialmente; ownership e garantia multi-port ainda não demonstrados. | Evita acoplamento ORM e abstração sem demanda. | Repositories de Domain; ports de Application; adapter com transaction runner. | Selecionar ao mapear casos de uso e atomicidade audit/outbox. Não cristalizar padrão antes. |
| TECH-01 — Go/Gin/GORM/PostgreSQL | Não é ADR completo. Stack foi fornecida como entrada e não tem justificativa, avaliação de alternativas ou consequência no registro. | Evitar reabrir uma decisão externa ao review sem pedido do usuário; ainda há implicações de framework/ORM/DB a verificar. | Não comparadas nesta documentação. | Registrar como “stack informada, aceite de arquitetura pendente”; confirmar em `condominio-api` antes de implementar. |

### Novas decisões arquiteturais necessárias

Recomendo criar propostas (não decisões aprovadas) antes do código para:

- **ARCH-07 — Tenant-scoped persistence contract and defense in depth:** mandatory tenant access, global exceptions, reference ownership, test obligations; escolher defesa concreta após avaliação técnica.
- **ARCH-08 — Command consistency, audit atomicity and replay semantics:** em quais comandos fato + audit + publication intent precisam ser atômicos e como retry/recovery é observável.
- **ARCH-09 — Temporal model for condominium operations:** instante, data/hora civil, timezone e períodos; decisões de negócio continuam em OD-04/05/01.

## 18. Recommended Changes

### MUST CHANGE

- Completar `TenantContext`/Infrastructure com um requisito testável: acessos tenant-scoped não podem omitir o tenant e nenhum resource ID, filtro ou correlation ID confere acesso.
- Declarar que todo command (HTTP, tarefa, handler ou operação assistida) atravessa a fronteira de authorization da Application e recebe identity/actor/purpose confiáveis; capability não aprovada bloqueia exposição.
- Especificar quais mutations sensíveis não podem confirmar sem AuditLog e se fato + audit + outbox intent devem compartilhar atomicidade. Não deixar o sucesso/auditoria dependente de callback best-effort.
- Definir invariantes de concorrência/replay por operation antes de implementar os comandos com risco alto, especialmente reservation approval, package release, grant/revoke, access authorization, voto/fechamento e module disable.
- Manter operações deliberativas e capabilities em aberto indisponíveis até resolver OD-01/02/03/05/06/11/14/15/16/17/18 aplicáveis.
- Definir modelo temporal e actor de jobs para os workflows incluídos no primeiro release.

### SHOULD CHANGE

- Completar a matriz de arquitetura dos workflows MVP com transaction boundary, audit, errors, idempotency/replay, side effects e process-failure outcome, referenciando documentos existentes.
- Estabelecer classificação de risco `NORMAL`/`ELEVATED`/`CRITICAL` e exigir revisão de autoridade para operações privilegiadas; controles extras dependem de decisão.
- Reescrever ARCH-04 para atomicidade de audit e campos mínimos; ARCH-06 para contrato transacional sem GORM leak; TECH-01 como stack fornecida, não escolha comparada.
- Propagar correlation de forma explícita por command/outbox/job/entrega, com redaction e sem dados pessoais/segredos.
- Estabelecer time/data/timezone e semântica de IDs antes de publicar operações temporais ou contrato público.
- Manter fonte factual vs projeções explicitamente diferenciada e reconstruível/reconciliável se projeção persistida for necessária.

### OPTIONAL

- Avaliar RLS como defesa adicional junto de enforcement na Application; não como substituto de autorização.
- Considerar testes automatizados de boundaries/imports e testes de contrato de repository quando o código existir.
- Reavaliar search, cache, scheduler, storage e extração de serviços somente com métricas/requisitos concretos.
- Criar um mapa de propriedade de dados por módulo quando a equipe/tamanho do repositório de aplicação justificar.

Não alterei regras de domínio nem fechei decisões de produto ou jurídicas.

## 19. Architecture After Review

Conceitualmente, manteria:

1. **Monólito modular** com Go e stack informada, sem microservices antecipados.
2. **Interface fina** como adaptador de transporte; autenticação técnica termina em principal verificado, sem decidir regra de negócio.
3. **Application como fronteira de command/query**: resolve um TenantContext por chamada, identifica o ator e finalidade, valida permission/scope/resource ownership, aplica authorization, escolhe transação e coordena audit/fatos/effects.
4. **Domain sem infraestrutura**: protege invariantes, regras temporais e state transitions; events são fatos após aceitação; corrections são fatos referenciados. Nenhum GORM tag/hook/provedor no modelo.
5. **Tenant-scoped persistence contract fail-closed**: queries e mutations operacionais requerem tenant em sua fronteira; referências relacionadas são coerentes; caminhos globais/cross-tenant são enumerados e autorizados explicitamente. Uma proteção de persistência adicional é avaliada separadamente.
6. **Transação mínima por caso de uso**: state/fact + Audit obrigatório e, quando a entrega durável foi exigida, intenção outbox juntos; sem chamadas de provider dentro do commit.
7. **Outbox seletiva** somente para effects assíncronos com garantia requerida; handlers preservam tenant/origem/correlation, toleram replay e registram tentativas sem prometer entrega única.
8. **Read paths** são tenant/purpose/Permission/scope filtrados antes de paginação, agregação, cache ou export; nenhuma query/ID global por conveniência.
9. **Jobs** usam identidade de processo e autoridade/purpose explicitadas, revalidam contexto e state no momento da execução; não herdam grant/request.
10. **Persistência pragmática**: GORM models separados quando necessário, repositories/queries específicos, sem GenericRepository e sem Unit of Work universal até haver demanda.
11. **No MVP**: sem cache sensível, search externo, broker específico, CQRS completo ou split de serviços. Paginação, errors, clock e IDs fechados na medida necessária a contratos e operações expostas.

Esse estado alvo continua sujeito à revisão humana e não escolhe RLS, mecanismo de auth, provider, lock, schema, broker, biblioteca, forma de API ou regra jurídica.

## 20. Final Status

**Assessment:** `NEEDS REVISION`

**Final status:** `ARCHITECTURE_REQUIRES_REVISION`

Principais razões: enforcement de tenant não fail-closed no contrato de persistência, atomicidade de audit/fatos não fechada, concorrência/replay insuficientemente especificados e decisões de identidade/authorization ainda bloqueando operações autenticadas críticas. A documentação atual oferece uma base arquitetural boa, mas deve incorporar as ações `MUST CHANGE` e obter decisões humanas aplicáveis antes de iniciar os fluxos funcionais correspondentes.
