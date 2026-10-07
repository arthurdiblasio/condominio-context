# Architecture Remediation

## 1. Executive summary

Esta remediação documental responde aos achados da [revisão crítica](./ARCHITECTURE-CRITICAL-REVIEW.md) sem criar código, API executável, banco, migrations, infraestrutura ou escolher mecanismo técnico ainda aberto.

**Status final:** `ARCHITECTURE_REMEDIATION_REQUIRES_REVIEW`

As garantias conceituais foram fortalecidas para persistência tenant-scoped fail-closed, fronteira obrigatória de autorização/Application, attribution de actor, atomicidade entre fato de sucesso e AuditLog obrigatório, matriz transacional, retry/replay e modelo temporal. Isso reduz ambiguidades documentais; não prova segurança de uma implementação futura. Identidade, permissões, políticas de negócio, privacidade e opções técnicas continuam exigindo revisão/decisão.

## 2. Overall assessment

**Assessment:** `REQUIRES HUMAN REVIEW`

As mudanças atendem ao objetivo de remediação estrutural e estabelecem requisitos não opcionais para implementação. Não é apropriado declarar `ARCHITECTURE_REMEDIATION_COMPLETE`: não há implementação para testar, a autoridade de várias ações está aberta, os mecanismos de isolamento/atomicidade/replay não foram escolhidos e perguntas de produto/jurídicas ainda bloqueiam workflows. O status anterior `ARCHITECTURE_REQUIRES_REVISION` permanece histórico na revisão crítica; este documento registra o resultado posterior, sem apagar o diagnóstico.

## 3. Scope and non-goals

**Incluído:** regras arquiteturais conceituais em `docs/architecture`, alinhamento do contrato conceitual, correção de um link interno, relatório e índices.

**Não incluído:** código, stack nova, APIs/endpoints/DTOs, SQL/schema, migrations, ORM, transações concretas, RLS, filas, broker, provedores, formato de tokens/IDs, padrão de autenticação ou decisão jurídica/de produto.

## 4. Sources and precedence

Fontes consultadas: `DOMAIN.md`, `BUSINESS-RULES.md`, `PERMISSIONS.md`, `WORKFLOWS.md`, `API-CONTRACT.md`, `OPEN-DECISIONS.md`, state machines de identidade/reservas, acesso/pacotes, notificações/assembleias e módulos, regras de auditoria, idempotência/API, e documentos de arquitetura citados na [revisão crítica](./ARCHITECTURE-CRITICAL-REVIEW.md).

Regras de negócio e decisões abertas continuam autoritativas para o domínio. Propostas arquiteturais não aprovam grants nem substituem OD-01..18. Em caso de lacuna, a operação não ganha default permissivo.

## 5. Tenant isolation and data scope

`PLATFORM-SCOPED` e `TENANT-SCOPED` são classes explícitas. Todo recurso deve ser classificado; falta de classificação não permite consulta global. `Person` pode relacionar-se a múltiplos condomínios, mas isso não combina dados operacionais nem grants entre tenants.

Para cada leitura/escrita tenant-scoped, TenantContext validado é obrigatório no contrato ou deve existir garantia equivalente explícita, verificável e impossível de omitir acidentalmente. Lookup somente por ID, filtros opcionais, fallback global, referência/join sem checagem de ownership e filtro depois de paginação/agregação são proibidos. Candidate tenant do cliente/job não é confiável até validação.

Essa exigência se aplica a API, Application, Domain quando necessário para invariantes, ports/repos, Infrastructure, audit, events/outbox, notifications, jobs, cache, search, reports, export, observability e consumers. Cross-tenant é uma operação excepcional com alvo, purpose, authority e audit explícitos, nunca uma consulta unscoped.

Negativa por mismatch não revela existência/dados do outro tenant. Classificação interna `TENANT_MISMATCH` pode ser preservada para diagnóstico autorizado; resposta externa indistinguível de recurso não encontrado/não visível é a recomendação conservadora, não uma decisão de protocolo/HTTP.

## 6. Authorization and use-case boundary

Application é a fronteira única de autoridade para toda operação com efeito. Request, job, agendamento, consumer, operação assistida, administração, processo de plataforma e chamada interna passam pelo use case; adapters não chamam repositories nem alteram agregado diretamente.

Separar Authentication, Authorization, resource access e business validation. A autorização contextual verifica actor/subject, grants, Permission, Scope, tenant validado, Resource, purpose, estado de conta e condições de negócio aplicáveis. Queries restritas também exigem tenant/scope/finalidade/visibilidade. Capability ausente, ambígua ou ainda não aprovada bloqueia a operação.

`platform.cross_tenant.access` não elimina necessidade de target explícito, autorização do recurso, purpose ou auditoria. Feature habilitada, papel, vínculo de unidade, autenticação e elegibilidade não são substitutos entre si.

## 7. Actor, subject and identity

Distinções obrigatórias:

- `HUMAN ACTOR`: operador humano autenticado; attribution conserva UserAccount e Person associada quando conhecida.
- `SYSTEM ACTOR`: processo interno identificado, limitado a transições automáticas previstas; não é pessoa nem bypass.
- `SERVICE ACTOR`: integração com identidade própria e capability/purpose/tenant limitados; não se faz passar por humano.
- Subject/affected Person: alvo da decisão, distinto do ator quando apropriado.
- Assisted operation: conserva operador, pessoa representada/alvo, fundamento de representação e purpose.
- Assembly `Proxy`: representação restrita ao escopo assemblear e nunca autoridade de suporte genérica.

`Person`/`UserAccount` cardinalidade, associação, lifecycle e capabilities permanecem OD-03. Concessão/delegação de autoridade, incluindo actors não humanos e suporte emergencial, permanece OD-02/16. A arquitetura não inventa modelo de conta ou role.

## 8. Jobs, events and assisted operations

Jobs/handlers carregam contexto mínimo verificável: actor/source, candidate tenant e classificação, resource/origin fact, purpose, tipo de operação, operation identity e correlation. Envelope e correlation não são credentials. Consumer valida origem, tenant, resource, estado, vigência e authority no momento do efeito.

Cada trabalho é classificado como:

1. **Nova decisão de negócio:** reautorizar a authority atual e revalidar todas as guardas; autoria de evento prévio não concede poder.
2. **Consequência já autorizada:** executar apenas a intenção persistida ainda válida e não cancelada, com actor de sistema/serviço restrito.

Ambiguidade impede execução automática. Retry retorna ao use case com a mesma operation identity; não há herança de tenant/request nem tratamento privilegiado de `system`. Integração assistida não usa a Person afetada como autora fictícia.

## 9. Transactional consistency and audit

**Invariante:** fato/estado de negócio confirmado e AuditLog obrigatório confirmam ou falham juntos. É inválido retornar sucesso sem audit mandatado ou registrar audit de sucesso sem fato correspondente. Falha ao gravar audit impede confirmar o fato. Audit de tentativa negada, se exigido, registra negação e não evento de sucesso.

Quando publicação durável for requisito, fato, AuditLog obrigatório e intenção de publicação/outbox pertencem à mesma unidade atômica. Outbox não é universal e não garante entrega “exactly once”. Provider/rede nunca participa da transação do fato; falha posterior de entrega não reverte negócio.

Campos mínimos conceituais de audit, sujeitos a minimização: actor e tipo; subject/affected target quando distinto; ação/resultado; resource e tenant ou classificação Platform; scope/purpose; instante; estado anterior/novo quando houver; reason quando exigido; referências a origem/correção; operation/correlation identity. Não copiar payload pessoal completo por padrão. AUDIT-01..10, OD-07 e política de retenção permanecem aplicáveis.

## 10. Transaction matrix

| Operação | Fato/estado na fronteira | Audit | Event/outbox |
|---|---|---|---|
| Reservation approval/cancel | Decisão e estado aceitos; approval revalida availability/conflict. | Fato + audit de sucesso atomicamente. | Event correspondente ao fato aceito; Notification/outbox só conforme política de entrega durável. |
| Package confirm/release | `CONFIRMED` e `PICKED_UP` são ocorrências distintas; projeção apenas se aprovada. | PackageEvent + audit atomicamente; correção mantém original. | Event específico; Notification/outbox condicional. |
| AccessEvent | ENTRY/EXIT/DENIED observados, com contexto conhecido; não cria autorização retroativa. | Fato + audit quando obrigatório atomicamente. | Evento após aceitação; outbox somente para reação durável aprovada. |
| RoleAssignment grant/revoke | Transição sob autoridade atual; preserva autoria/vigência/histórico. | Estado/fato + audit atomicamente. | Event da transição; Notification/outbox condicional. |
| Vote | Voto em pauta aberta, elegibilidade/uniqueness de acordo com policy aprovada. | Voto + audit atomicamente, sem expor conteúdo indevido. | `VoteRecorded/Invalidated`; apuração/publicação separadas. |
| Assembly close | Fechamento de Assembly/AgendaItem segundo workflow; não presume resultado publicado. | Fechamento + audit atomicamente. | Event de fechamento; comunicação condicional e assíncrona. |
| Feature enable/disable | Configuração do tenant efetiva; não resolve operação em curso por inferência. | Alteração + audit atomicamente. | Event de mudança; consequência depende de OD-11. |
| Notification creation | Intenção lógica, nunca prova de envio/entrega. | Audit conforme classe/policy; atômico se obrigatório. | NotificationCreated; Delivery/attempt separado; outbox se durabilidade exigida. |

## 11. Concurrency invariants

Antes de cada command crítico deve haver um ponto consistente de decisão e revalidação de estado/authority. Requisitos:

- Reservation: revalidar área/período e não aceitar confirmações incompatíveis quando policy proíbe sobreposição; prioridade/empate é OD-04.
- Package: não apresentar retirada duplicada do mesmo ato; preservar observações físicas distintas; conflito/contestação é OD-06/18.
- Access: registrar observações físicas reais; não deduplicar por proximidade nem fabricar EXIT; validade de authorization avaliada no instante definido em OD-05.
- RoleAssignment: validar authority/concessor e estado vigentes; não usar grants revogados/expirados/stale; conflito/precedência é OD-02/14/18.
- Vote/close: resultado deve respeitar janela efetiva da pauta; não contar/substituir voto por corrida; unicidade, segredo, fechamento/reabertura dependem OD-01.
- Feature disable vs command: usar configuração válida no ponto de decisão; operação em andamento depende OD-11.
- Correção concorrente: manter original e ambas as evidências; encaminhar ambiguidade sem sobrescrita.

Nenhum lock, isolation level, constraint, optimistic version ou mecanismo de concorrência é escolhido.

## 12. Idempotency and replay

Semântica depende da identidade da intenção, não de payload semelhante. A operation identity deve sobreviver a timeout/restart/retry, mas seu formato/transport/storage não foi escolhido. Resultado de commit incerto é investigado pela identidade/histórico antes de reaplicar; sem identidade confiável, não especular.

Replay da mesma operação não cria fato/audit/Notification attempt duplicado. A intenção nova pode ter payload igual. `ENTRY`, `EXIT`, `RECEIVED` e `PICKED_UP` são eventos físicos e não se deduplicam por conteúdo ou tempo. Reenvio deliberado é nova `NotificationDelivery`, preservando a anterior. Detalhes por command constam em [idempotency.md](./docs/api/idempotency.md), e regras de produto permanecem OD-18/01/04/05/06/08/11.

## 13. Temporal model

`INSTANT`, `LOCAL DATE`, `LOCAL TIME`, `DATE-TIME WITH TIMEZONE`, `DURATION`, `INTERVAL` e `VALIDITY WINDOW` são tipos conceituais distintos. Local rule usa timezone do Condominium, nunca timezone implícito do host/cliente. Limites inclusivos/exclusivos são definidos por workflow; nenhum default universal foi inventado. Um command usa um instante de referência consistente e Domain não consulta diretamente relógio de sistema.

Documento: [temporal-model.md](./docs/architecture/temporal-model.md). Ainda faltam decisões de domínio sobre calendários/janelas e decisões técnicas sobre representação da zona, fonte do clock, precisão e ambiguidade de horário.

## 14. API and workflow consistency

`API-CONTRACT.md` e `docs/api/*` permanecem conceituais; nenhum endpoint/payload/HTTP status foi criado. Categorias de erro preservam distinction `UNAUTHENTICATED`, `ACCOUNT_INACTIVE`, `FORBIDDEN`, `SCOPE_MISMATCH`, `TENANT_MISMATCH`, `NOT_FOUND`, `AUDIT_REQUIRED`, `DECISION_REQUIRED`; não mapeiam a HTTP.

WORKFLOWS e state machines continuam fontes de regras e transições. Commands sem capability/authority aprovada não devem ser implementados. A matriz transacional cobre operações solicitadas no Prompt 10, mas a matriz de rastreabilidade de todos os WF, suas classes de risco aprovadas, dados/retenção e erro por operação ainda precisa ser conferida ao definir release.

## 15. Critical review finding disposition

| Finding | Disposição nesta remediação | Remanescente |
|---|---|---|
| AR-01 Tenant isolation | Requisito fail-closed e tenant obrigatório formalizados em contexto, ports e fronteiras. | Implementação/mecanismo e testes negativos ainda não existem. |
| AR-02 Authorization bypass | Application boundary obrigatória para requests/jobs/assistência documentada. | Capabilities/authority permanecem OD-02/03/14/15/16. |
| AR-03 Audit atomicity | Fato + AuditLog obrigatório descritos como indivisíveis. | Seleção/validação do mecanismo e cobertura concreta dos commands. |
| AR-04 Concurrency | Invariantes e corridas principais especificadas. | Políticas OD e técnica de ordenação/consistência pendentes. |
| AR-05 Replay | Identity da operação e semântica por comando descritas conceitualmente. | Formato, transporte, persistência e recuperação técnica pendentes. |
| AR-06 Capability gaps | Regra fail-closed reafirmada. | Decisões humanas para cada operação a expor. |
| AR-07 Jobs/actors | Actor, envelope e revalidação registrados. | Authority system/service e classificação workflow a workflow. |
| AR-08 Identity | Conceitos separados e limites reafirmados. | OD-03 impede fechar associação/cardinalidade/autenticação. |
| AR-09 Domain/Application | Uso de case orquestra; regra/invariante segue Domain. | Fronteiras por aggregate/módulo precisam de validação em cada workflow. |
| AR-10 API errors | Taxonomia vigente preservada e erro externo minimizado. | Mapeamento/UX conceitual deve ser validado antes de contrato estável. |
| AR-11 Time | Tipos temporais e zone context documentados. | Fonte de timezone/clock, bordas de validade e casos DST dependem de decisões. |
| AR-12 Privacy/retention | Minimização e proteção cross-tenant reafirmadas. | Retenção, anonimização, backups e finalidade dependem OD-07/validadores. |
| AR-13 Audit fields | Campos mínimos conceituais descritos neste relatório. | Matriz de campos/reason/risco por operação e privacy ainda requer validação. |
| AR-14 Workflow coverage | Fronteira global e subset transacional mapeados. | Matriz completa WF→authority/transaction/audit/replay/erro ainda depende release. |

## 16. Human decisions and blockers before implementation

Decisões que continuam críticas:

- **OD-01:** validade jurídica, elegibilidade/peso/quórum/proxy, voto, encerramento e publicação de assembleia.
- **OD-02:** quem concede/suspende/revoga grants, self-grant e authority/delegação.
- **OD-03:** identidade e cardinalidade Person–UserAccount, lifecycle e primeira autoridade de onboarding.
- **OD-04:** disponibilidade, aprovação, conflito, prioridade e cancelamento de reserva.
- **OD-05:** autorização/visita/acesso, validade, divergência e correção de AccessEvent.
- **OD-06:** custódia, confirmação, retirada, representação, disputa e fato concorrente de package.
- **OD-07:** finalidades, campos visíveis, retenção, descarte, evidências e logs.
- **OD-08:** canais, retry/fallback, evidência de entrega, cancelamento e término de Notification.
- **OD-09:** autoridade/prova/sobreposição e validade de vínculos pessoais/unidade.
- **OD-11/17:** authority/defaults/dependências para enablement e grants relacionados.
- **OD-14/15/16:** precedência de grants, alcance de Scope, delegação e emergência.
- **OD-18:** conflito/repetição nas operações críticas.

**Perguntas humanas antes dos commands correspondentes:**

1. Quem tem autoridade, em cada tenant e Scope, para conceder ou revogar cada papel, e quais permissões são delegáveis?
2. Qual é a associação/cardinalidade entre Person e UserAccount e quem pode criar, vincular, suspender e recuperar uma conta?
3. Quais regras de reserva/acesso/pacote podem decidir estado e instante-limite, incluindo concorrência e contestação?
4. Quais decisões de assembleia requerem validação jurídica e qual é a autoridade para elegibilidade, voto, procuração e publicação?
5. Quais dados pessoais/audit são visíveis e por quanto tempo, com quais finalidades e exceções?
6. Quais notificações precisam de promessa de entrega durável e qual comportamento é aceito quando provider falha?
7. Qual authority limitada é permitida a system/service actors e que jobs são consequência autorizada vs nova decisão?
8. Quais recursos/commands entram no primeiro release e quais podem permanecer indisponíveis até decisão?

Nenhuma resposta é inferida nesta remediação.

## 17. ADR review and validation

ARCH-07 (tenant persistence), ARCH-08 (consistency/audit/replay), ARCH-09 (time), ARCH-10 (actor/jobs) e ARCH-11 (audit atomicity) foram detalhados em requisitos, não promovidos a ADR aceito. ARCH-01..06 também permanecem propostas. A stack Go/Gin/GORM/PostgreSQL continua entrada informada, não validação completa de arquitetura.

**Documentos alterados:** `docs/architecture/tenant-context.md`, `infrastructure.md`, `layers.md`, `overview.md`, `application.md`, `authorization.md`, `transactions.md`, `events.md`, `outbox.md`, `observability.md`, `decisions.md`; adicionado `temporal-model.md`; alinhados `docs/api/idempotency.md`, `API-CONTRACT.md`, `PERMISSIONS.md`, `README.md`, `docs/README.md`.

**Validação:** documental, sem testes de runtime aplicáveis. Antes de encerrar a revisão, verificar links internos/anchors, fences, Mermaid, formatação (`git diff --check`), status/índices e confirmar que nenhuma implementação proibida foi criada. A ausência de runtime significa que controles de isolamento/atomicidade só serão demonstrados em implementação futura por testes específicos.

### Status de encerramento

`ARCHITECTURE_REMEDIATION_REQUIRES_REVIEW`

Há invariantes conceituais agora explícitas, mas ainda existe risco material de implementação acidental se capabilities abertas forem tratadas como grants, se classes de actor não forem autorizadas por regra aprovada, ou se contratos fail-closed não forem testados no código futuro. Não iniciar workflows autenticados/sensíveis até fechar decisions/blockers aplicáveis.
