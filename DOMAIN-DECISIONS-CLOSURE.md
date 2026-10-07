# Domain Decisions Closure

## 1. Executive Summary

Esta revisão classifica decisões de produto, domínio e política existentes antes de persistência/commands. Não escolhe comportamento ausente, não presume legislação nem converte recomendações em regra aprovada.

**Status final:** `DOMAIN_DECISIONS_REQUIRES_HUMAN_INPUT`

Há invariantes já documentadas que podem ser tratadas como decididas — por exemplo, `Person` é diferente de `UserAccount`; propriedade, residência e locação são relações separadas; autorização não prova entrada; `CONFIRMED` não prova retirada; `Notification` não prova entrega; e feature habilitada não concede Permission. Essas decisões parciais não encerram as ODs completas. Nenhuma das ODs pedidas está integralmente resolvida; várias dependem de política do condomínio, governança do produto ou validação jurídica.

Este relatório é classificação e rastreabilidade, não roadmap. As classificações MVP são candidatas derivadas do contrato conceitual de recursos, não compromisso de release. Capacidades marcadas como bloqueadas não devem ser expostas como commands até que as decisões necessárias sejam tomadas.

## 2. Decision Classification

Cada linha de OD recebe exatamente uma classificação, referente à natureza predominante do assunto que continua aberto:

| Categoria | Uso nesta revisão |
|---|---|
| `DECIDED` | A documentação contém decisão suficiente e final para a questão examinada; não usar apenas porque há uma regra parcial. |
| `CONFIGURABLE_BY_CONDOMINIUM` | O comportamento varia por condomínio dentro de invariantes globais; configuração, autoridade e limites ainda precisam ser explicitados. |
| `LEGISLATION_OR_POLICY_DEPENDENT` | A escolha depende de lei, convenção, regimento, instrumento formal ou política competente; não generalizar no produto. |
| `REQUIRES_HUMAN_DECISION` | Falta escolha de produto/domínio/governança para determinar o comportamento. |
| `MVP_OUT_OF_SCOPE` | A capacidade foi explicitamente excluída do MVP candidato; a exclusão não resolve regras futuras. |

“Bloqueia implementação?” refere-se ao command/capacidade afetada, não a toda a documentação ou ao trabalho em outros domínios. `YES` bloqueia a implementação funcional do escopo; `CONDITIONAL` permite apenas um recorte que respeite as invariantes já decididas; `NO` significa que o tema não precisa ser implementado no MVP candidato, não que suas regras estejam resolvidas.

## 3. Decision Matrix

| ID | Tema | Decisão / questão remanescente | Classificação | Impacto | Bloqueia implementação? |
|---|---|---|---|---|---|
| OD-01 | Assembleias e validade de votação | Elegibilidade, peso, quórum, procuração, representação, unicidade, segredo, fechamento/reabertura, invalidação, apuração e validade/publicação ainda não estão determinados; as regras pertinentes exigem validação competente. | `LEGISLATION_OR_POLICY_DEPENDENT` | Pode alterar quem delibera e se o resultado é válido. | YES para voto, apuração e publicação; criação/agenda pode ser condicional sem alegar validade legal. |
| OD-02 | Governança de RoleAssignment | Autoridade por operação/papel/scope, self-grant, elevação, grants privilegiados e delegação não foram atribuídos. | `REQUIRES_HUMAN_DECISION` | Pode ampliar privilégios entre pessoas e tenants. | YES para grant/suspend/reactivate/revoke funcionais. |
| OD-03 | Person e UserAccount | Cardinalidade, associação, convite, ativação, suspensão, recuperação, capabilities e primeiro administrador estão abertos. | `REQUIRES_HUMAN_DECISION` | Afeta subject, autenticação, autoria e continuidade de vínculos. | YES para operações de conta/autenticadas; criação de Person sem conta pode ser condicional. |
| OD-04 | Reservas e áreas comuns | A documentação prevê política local `AUTO_APPROVED`, `MANUAL_APPROVAL` ou `RULE_BASED`; valores e critérios de conflito, prioridade, fila, limites e ciclo precisam de definição por condomínio/produto. | `CONFIGURABLE_BY_CONDOMINIUM` | Afeta disponibilidade, alocação e promessas de uso. | YES para approval/complete com regra incompleta; create somente após configuração e capability aprovadas. |
| OD-05 | Acesso e visitas | Emissão/aprovação, exigência de autorização, validade/recorrência, exceções, encerramento de Visit e correção de AccessEvent requerem política aplicável a cada condomínio. | `CONFIGURABLE_BY_CONDOMINIUM` | Afeta segurança operacional, acesso físico e histórico. | YES para emissão/revogação/correção sem authority; registro factual pode ser condicional. |
| OD-06 | Custódia de Package | Autoridade de confirmar/retirar, representante, evidência, contestação, recusado/danificado, destino desconhecido, sequência e projeção corrente não estão definidos. | `CONFIGURABLE_BY_CONDOMINIUM` | Afeta cadeia de custódia e resolução de contestação. | YES para confirmação, liberação e projeção; recebimento factual em recorte restrito é condicional. |
| OD-07 | Privacidade e retenção | Finalidade, categorias, visibilidade detalhada, retenção, descarte, anonimização, evidência e logs dependem de política e revisão de privacidade/jurídica; prazo não foi fixado. | `LEGISLATION_OR_POLICY_DEPENDENT` | Afeta coleta, exposição, conservação e direitos de titulares. | YES para exposição/retention/export de dados afetados sem política; coleta mínima pode ser condicional. |
| OD-08 | Notification e Delivery | Canais habilitados, preferência, prioridade, fallback/retry, confirmação, cancelamento, expiração e expectativa de entrega são definidos por política de comunicação; provider é fora do domínio. | `CONFIGURABLE_BY_CONDOMINIUM` | Afeta comunicação, preferências e confiabilidade percebida. | YES para promessa/canal/fallback; criação de intenção sem entrega garantida é condicional. |
| OD-09 | Relações Person/Unit | Autoridade/prova, copropriedade, múltiplas relações, sobreposições, períodos futuros, dependentes e direitos dependem de instrumentos e política local/legal. | `LEGISLATION_OR_POLICY_DEPENDENT` | Afeta titularidade, visibilidade, responsabilidades e elegibilidade. | YES para efetivar/corrigir vínculos contestáveis; conceitos distintos e histórico podem ser modelados. |
| OD-11 | Module/Feature por Condominium | Habilitação é por tenant e não concede acesso; defaults, autoridade, dependências, conflitos, operações em curso e acesso histórico devem ser definidos no contexto de cada condomínio/produto. | `CONFIGURABLE_BY_CONDOMINIUM` | Afeta disponibilidade funcional e continuidade de operações/dados. | YES para habilitar/desabilitar sem authority/default; estrutura conceitual pode ser modelada. |
| OD-14 | Conflito entre grants | Precedência, `ALLOW`/`DENY`/`EXPLICIT_DENY`, papel múltiplo e regra de decisão final não foram aprovados; recomendação em PERMISSIONS não é decisão. | `REQUIRES_HUMAN_DECISION` | Afeta decisão de autorização e risco de privilégio implícito. | YES para autorização quando grants conflitarem. |
| OD-15 | Abrangência de Scope | Herança/abrangência de recursos descendentes e necessidade de scopes adicionais precisam de decisão; não há herança automática aprovada. | `REQUIRES_HUMAN_DECISION` | Afeta alcance efetivo de cada grant. | YES para commands cujo recurso não esteja coberto explicitamente. |
| OD-16 | Delegação e acesso emergencial | Permissions delegáveis, limites, duração, transitividade, break-glass, suporte e revisão posterior não têm autoridade definida. | `REQUIRES_HUMAN_DECISION` | Afeta elevação/exceção de acesso e accountability. | YES para delegação/break-glass; não criar mecanismo ou bypass. |
| OD-17 | Permission por Feature | Se habilitação muda alguma capacidade de leitura e quais grants permitem cada Feature ainda não foi determinado. | `REQUIRES_HUMAN_DECISION` | Evita que disponibilidade se transforme em acesso. | YES para autorização de uso de Feature com grants não definidos. |
| OD-18 | Repetição e concorrência | Semânticas já propostas para alguns commands não fecham identificação de mesma intenção, conflito/replay por operação e evidência operacional. | `REQUIRES_HUMAN_DECISION` | Pode duplicar fatos ou descartar evento físico distinto. | YES para commands críticos após timeout/concorrência; regras de repetição já definidas podem ser preservadas. |

**IDs:** todos os IDs solicitados (OD-01..09, OD-11, OD-14..18) existem em `OPEN-DECISIONS.md`; nenhum foi inventado ou renumerado. OD-10, OD-12 e OD-13 também existem, mas não fazem parte da matriz solicitada e não são fechados por esta revisão. Não há ID ausente no conjunto solicitado.

## 4. Assembly Decisions

### O que já está documentado

- `Assembly`, `AgendaItem`, `Participant`, presença, `VotingEligibility`, `Proxy`, `Vote`, `Quorum` e resultado são conceitos distintos.
- Elegibilidade é contextual a uma pauta; presença não implica elegibilidade, e elegibilidade não implica voto.
- `Vote` é fato registrado; correção/invalidação precisa preservar histórico e auditoria.
- `Proxy` é restrita ao escopo/período reconhecido e não equivale a RoleAssignment nem delegação administrativa.
- Fechamento não implica apuração, validade jurídica ou publicação; `VOTING` pertence a `AgendaItem`.
- Sem regra aprovada, resultado não é apresentado como deliberação validada.

Referências: ASSEMBLY-01..08 em [assemblies.md](./docs/business-rules/assemblies.md), [workflow de assemblies](./docs/workflows/assemblies.md) e [state machines de comunicação/assembleia](./docs/state-machines/communications-and-assemblies.md).

### O que continua sem decisão

| Tema | Situação |
|---|---|
| Elegibilidade, peso do voto e efeitos de inadimplência | `REQUIRES_HUMAN_DECISION` com validação jurídica/política; nenhum critério foi inferido. |
| Quórum e regra de apuração | `REQUIRES_HUMAN_DECISION`; falta fórmula, denominador, momento e autoridade. |
| Procuração/representação | `LEGISLATION_OR_POLICY_DEPENDENT`; forma, limites, conflitos, escopo por pauta/assembleia, validação e revogação. |
| Unicidade por unidade vs participante | `REQUIRES_HUMAN_DECISION`; documentos não escolhem nem autorizam um voto por unidade ou por pessoa. |
| Voto secreto/aberto e associação voto-votante | `LEGISLATION_OR_POLICY_DEPENDENT`; não definir visibilidade técnica como regra. |
| Convocação, abertura, encerramento e reabertura | `LEGISLATION_OR_POLICY_DEPENDENT`; procedimentos, autoridades e efeitos requerem instrumento/política válida. |
| Correção/invalidação/repetição/substituição de voto | `REQUIRES_HUMAN_DECISION` e validação legal; operação não deve ser habilitada por conveniência. |
| Publicação de resultado/ata | `LEGISLATION_OR_POLICY_DEPENDENT`; depende de apuração e autoridade reconhecidas. |

**Conclusão OD-01:** podem ser modeladas separadamente as relações/fatos estruturais já registrados. Commands de voto, apuração, invalidação e publicação permanecem bloqueados; criar ou agendar assembleia não pode afirmar validade deliberativa.

## 5. Authorization / Grants Decisions

### Partes já decididas conceitualmente

`Role` nomeia função; `RoleAssignment` atribui Role a Person em tenant/scope/vigência; `Permission` é capacidade; `Scope` delimita grant; Resource conserva tenant/contexto. Nenhum papel pelo nome é grant automático. Não há herança automática de tenant/scope. Autoconcessão privilegiada e autoridade superior sem delegação válida não são permitidas pelo modelo-base.

Isso não identifica a pessoa/órgão que possui authority nem resolve conflitos. A recomendação `deny by default`, `EXPLICIT_DENY overrides ALLOW` e ausência de `ROLE_PRIORITY` em [PERMISSIONS.md](./PERMISSIONS.md) permanece recomendação não aprovada.

### Questões necessárias por operação

| Operation | Decidido | Ainda precisa decidir |
|---|---|---|
| `role_assignment.grant` | Precisa de permission/scope aplicável; concessor não pode ultrapassar sua authority sem delegação explícita. | Quem concede cada papel em Platform/Condominium/Block/Unit/Feature; se condomínio_admin, syndic ou assistant_syndic podem conceder quais papéis; self-grant; grant a Person sem UserAccount. |
| `role_assignment.revoke` | Revogação encerra grant futuro e preserva ações/histórico anteriores. | Authority para revogar cada papel, auto-revogação e eventual revisão/apelação. |
| `role_assignment.suspend` | Suspensão é distinta de revogação e pode ser revertida; não suspende Person/UserAccount globalmente. | Quem pode suspender, por quais razões/escopo, duração e revisão. |
| `role_assignment.reactivate` (`restore`) | State machine nomeia `reactivate`; somente grant `SUSPENDED` pode voltar a `ACTIVE` no ciclo normal. Reativação não restaura outros grants expirados/revogados. | Autoridade, validações, limites e necessidade de revisão. `role_assignment.restore` não aparece no catálogo: não inventar capability alternativa. |
| Alterar Role/Scope | Não há transferência implícita de tenant nem alteração silenciosa de autoridade. | Se mudança é alteração, encerramento+novo grant, workflow de aprovação e permissions próprias. |

`condominium_admin`, `syndic`, `assistant_syndic`, `property_manager`, `platform_admin` e suporte futuro não possuem authority inferida pelo nome. `platform_admin` não tem acesso operacional tenant-scoped automaticamente. Assembly `Proxy` não é delegação de grants nem representação genérica de suporte.

**Conclusão OD-02/14/15/16:** bloqueiam operações de grants em execução até autoridade, resolução de conflito, cobertura de scope e delegação/emergência serem aprovadas. Ausência de regra não permite escolher papel de maior prioridade nem ampliar alcance.

## 6. Identity Decisions

### Decidido

- `Person` pode existir sem conta.
- `Person`, `UserAccount`, `Role`, `RoleAssignment`, `Permission`, `UnitOwnership`, `UnitResidency` e `UnitTenancy` são conceitos distintos.
- Uma Person pode ter vínculos e papéis em mais de um condomínio; isso não concede acesso entre tenants.
- Ativar/desativar conta não cria/remove Person, vínculo de unidade ou RoleAssignment automaticamente.
- Conta desativada não inicia nova ação autenticada segundo a regra documentada; histórico não é transferido ao alterar associação conta-pessoa.

### Não decidido (OD-03)

- cardinalidade Person–UserAccount, inclusive: **uma Person pode possuir mais de uma UserAccount?**
- associação inicial e correção/reassociação;
- convite, primeiro acesso, ativação direta, aceitação, validade, expiração e reemissão;
- quem pode criar/convidar/ativar/suspender/reativar/desativar/recuperar;
- lifecycle, motivos e reversibilidade;
- capabilities `user_account.*` (não existem no catálogo atual);
- primeira autoridade/admin no onboarding;
- associação e requisitos por tenant quando uma mesma conta participa de vários condomínios.

**Conclusão:** identidade conceitual pode ser modelada; lifecycle/commands de conta e operações humanas autenticadas não podem ser fechadas antes da resposta de governança e produto.

## 7. Reservation Decisions

### Invariantes documentadas

- Reservation referencia Unit e CommonArea do mesmo Condominium.
- Confirmação valida disponibilidade, bloqueio/manutenção e aprovações aplicáveis.
- Se a política proíbe sobreposição, duas reservas incompatíveis não podem ser confirmadas.
- `PENDING` não bloqueia universalmente o slot; isso depende de regra local.
- Event/Reservation não concedem acesso automaticamente.
- Alteração/cancelamento preserva histórico; bloqueio/desativação não cancela reserva por inferência.

### Universal invariants vs condomínio

| Universal já documentado | Configurável por condomínio/área conforme regra aprovada |
|---|---|
| Tenant e referências coerentes; não aceitar estado confirmado sem condições necessárias. | Modo `AUTO_APPROVED`, `MANUAL_APPROVAL` ou `RULE_BASED`; disponibilidade e períodos operáveis. |
| Quando sobreposição proibida, não confirmar reservas incompatíveis. | Se pendência bloqueia provisoriamente, limites de reserva, capacidade, antecedência, duração, recorrência e regras de feriados. |
| Conflito concorrente não produz duas confirmações proibidas pela policy. | Prioridade/fila/empate, cancelamento/no-show, conclusão/expiração e consequências de bloqueio/manutenção. |
| Não inventar inadimplência ou regra legal como guarda sem policy validada. | Cobrança/reembolso e efeitos de alteração depois de confirmação, se o produto/local aprovar. |

Configuração local não pode enfraquecer isolamento, autorização, audit ou invariantes gerais.

### Commands bloqueados

- `reservation.create`: **CONDITIONAL** para recorte que só aceite configuração/policy conhecida, permission `reservation.create`, tenant e vínculo validados; campos, limites e pending semantics ainda bloqueiam cobertura completa.
- `reservation.approve`: **NO** para fluxo completo enquanto criteria, authority, conflito/concorrência e permission de rejeição/aprovação não estiverem definidos.
- `reservation.cancel`: **CONDITIONAL/NO**; preservação é conhecida, mas authority, prazo, efeito financeiro e capability de cancelamento podem estar incompletos.
- `reservation.complete`: **NO** até definir instante/critério de conclusão; não derivar presença ou AccessEvent do fim do período.

OD-04 é principalmente configurável por Condominium, com decisões de produto necessárias para os limites e campos de configuração antes dos commands.

## 8. Access Decisions

### Distinções já decididas

`Person` é a pessoa; `Visit` é contexto/intenção; `AccessAuthorization` é permissão temporal/contextual; `AccessEvent` é fato observado `ENTRY`, `EXIT` ou `DENIED`. Authorization pode existir sem Visit e sem ENTRY. ENTRY não é inferida da autorização; EXIT pode existir sem ENTRY anterior registrado; DENIED não é ENTRY. Visitor/Guest são descrições contextuais, não identidades duplicadas. Referências opcionais a Event/Reservation/Visit não ampliam tenant nem período.

### Procedimentos ainda dependentes

- quando autorização é necessária, uso único/repetido, janela e tratamento de acesso recorrente;
- quem pode pedir, emitir, aprovar, revogar e cancelar autorização;
- identificação mínima, exceções e procedimento local para visitante, funcionário/prestador;
- conclusão/no-show da Visit, saída não observada e resolução de divergência;
- authority e processo de correção de AccessEvent, evidência e revisão;
- tratamento local de evento ambíguo/duplicado sem inventar observação física.

**Conclusão OD-05:** os fatos e distinções são decididos; a política de acesso, authority e correção são `CONFIGURABLE_BY_CONDOMINIUM`/decisão humana dentro dos limites. Registrar fato observado pode ser considerado somente em recorte onde source, actor, tenant, permission e campos mínimos tenham sido aprovados.

## 9. Package Decisions

### Decidido

- `RECEIVED` é registrado apenas após recebimento físico; guarda quem recebeu, quando e quem registrou quando conhecidos.
- destino incerto não é atribuído silenciosamente a unidade/pessoa.
- `NOTIFIED` é distinto de entrega de Notification.
- `CONFIRMED` e `PICKED_UP` são fatos distintos; confirmação não prova retirada.
- correção mantém o evento original; evento posterior não inventa fato anterior.

### Ainda precisa de escolha local/humana

Quem pode confirmar/retirar; validade de representante; se confirmação precede retirada; evidência aceitável; contestação; pacote recusado/danificado/não retirado; destinatário desconhecido; regra para duplicidade física/replay; ordem de confirmação/retirada/cancelamento; situação corrente derivada de PackageEvent; retenção/descarte.

**Conclusão OD-06:** semântica do histórico está decidida parcialmente; comando de confirmação/liberação e projeção de status ficam bloqueados até authority e sequência local. Recebimento físico pode ser um recorte condicional, sem inventar destinatário ou status.

## 10. Privacy / Retention Decisions

| Dimensão | O que já está decidido | O que não está decidido |
|---|---|---|
| Finalidade | Coletar/manter o mínimo para finalidade identificada; consultas respeitam purpose/permission/scope. | Finalidades e base por workflow/dado/ator, quando consentimento/base legal aplica. |
| Dados mínimos | Minimização; portaria só vê o necessário; não expor dados de moradores por compartilhar tenant. | Campos obrigatórios/opcionais em Person, Visit, Package, notification, audit, voto e evidências. |
| Visibilidade | Restrita por permission/scope; acesso de plataforma não é implícito; relatório deve minimizar quando possível. | Visibilidade de cada campo por papel, portaria/admin, unidade, assembleia e suporte. |
| Retenção | Histórico não é apagado automaticamente por encerrar conta/vínculo ou desabilitar feature; retenção não é indefinida por padrão. | Prazos, descarte, anonimização, exceções legais, backups, export e incidentes. |
| Audit/logs/evidências | AuditLog, eventos operacionais e logs técnicos são distintos; correções têm referência; audit read é restrito. | Campos/payloads finais, retention, redaction, export, acesso a detalhes pessoais e eliminação/anonimização. |

OD-07 é `LEGISLATION_OR_POLICY_DEPENDENT`. Nenhum prazo é proposto. Exposição, export e retenção de dados pessoais/evidências permanecem bloqueadas para uso real até validação adequada.

## 11. Notification Decisions

### Conceitos já decididos

- `Notification` é intenção lógica, pode ter zero ou várias `NotificationDelivery`s.
- Cada entrega/tentativa é registrada separadamente por canal; nova tentativa não apaga falha anterior.
- `SUBMITTED` não implica `DELIVERED`; `DELIVERED` não implica `READ`; cada status requer evidência.
- Falha de entrega não altera o fato de negócio que originou a comunicação.
- Canal é abstrato e não depende de provedor.

### Configurável / em aberto

Channels enabled, preferências/opt-in aplicável, prioridade, fallback, retries, expiração/cancelamento, definição de encerramento de Notification, canais obrigatórios/opcionais, conteúdo e retenção. Provider/templates/custos permanecem implementação/decisão técnica ou comercial, não entidade de domínio.

**Conclusão OD-08:** disponibilidade/canais e preferências são `CONFIGURABLE_BY_CONDOMINIUM` dentro de capacidade de produto e regras de privacidade. Sem promessa de comunicação aprovada, criar intenção é condicional; não afirmar delivery.

## 12. Person / Unit Relationship Decisions

### Decidido

- `UnitOwnership`, `UnitResidency` e `UnitTenancy` são vínculos distintos; nenhum é Role nem concede conta/admin por si.
- Unidade pertence a exatamente um Condominium; referências de pessoa/unidade/tenant devem ser coerentes.
- copropriedade/múltiplos vínculos não podem ser descartados por pressuposto.
- encerrar vínculo não apaga histórico; propriedade não implica residência, residência não implica propriedade ou locação, locação não implica residência automaticamente.

### Não decidido

Prova e authority para criar/encerrar/corrigir; percentuais e copropriedade; múltiplos proprietários/residentes; dependentes; sobreposição; período futuro e efetividade; responsável financeiro e direitos; relação e permission necessárias a cada capacidade; regras legais de titularidade/ocupação.

OD-09 é `LEGISLATION_OR_POLICY_DEPENDENT`. Não transformar vínculo em Role. Pode-se modelar tipos e invariantes; commands que declarem titularidade/ocupação efetiva dependem de evidência, autoridade e regras aprovadas.

## 13. Module / Feature Decisions

### Decidido

- `Module` agrupa capacidade; `Feature` é capacidade específica; `CondominiumModule`/`CondominiumFeature` exprimem habilitação tenant-scoped.
- Habilitação A não habilita B; habilitar não concede Role/Permission/Scope.
- desabilitar não apaga dados/histórico e não invalida operations em curso por inferência.
- não assumir Module como agregado/entidade de negócio independente em cada caso.

### Configuração/decisão pendente

Default de tenant novo, autoridade para enable/disable, catálogo de dependências/incompatibilidades, efeitos em operação ativa, acesso a histórico após disable, reativação, permission por Feature e qualquer efeito em read. OD-11 é `CONFIGURABLE_BY_CONDOMINIUM` em disponibilidade; OD-17 é `REQUIRES_HUMAN_DECISION` sobre grants.

**Bloqueio:** modelagem conceitual da habilitação pode avançar; command de enable/disable ou gate funcional não pode assumir default/authority/grant por produto.

## 14. Repetition / Idempotency Decisions

### Semântica já documentada

- Mesma operação comprovada é distinta de nova operação com payload igual.
- `NO-OP`/idempotência só se a identidade do ato for estabelecida; caso contrário não consolidar fatos por similaridade.
- `ENTRY`/`EXIT` e package event físico distinto não são deduplicados por proximidade temporal.
- repetir voto não substitui voto silenciosamente enquanto regra de substituição não for aprovada.
- retry de NotificationDelivery é nova tentativa e mantém anteriores.
- conflito de reserva depende OD-04; grants, acesso, pacote e presença/voto também guardam OD-18 e suas ODs específicas.
- timeout/commit desconhecido não significa falha nem sucesso; não repetir às cegas.

### Em aberto

Como identificar mesma intenção por operação, respostas a timeout/replay, resultado de concorrência, correção vs novo evento e período de deduplicação/retenção. Nenhuma idempotency key, mecanismo de lock ou storage foi decidido.

OD-18 é `REQUIRES_HUMAN_DECISION` para conflitos e casos ambíguos, apesar dos princípios semânticos já documentados. Identidade técnica e recuperação devem ser decididas antes de publicar commands externos.

## 15. MVP Classification

Classificação de capacidades com base em [API resources](./docs/api/resources.md), `WORKFLOWS.md` e decisões abertas. Cada capacidade recebe uma categoria: `MVP`, `POST-MVP` ou `CONDITIONAL`. **Não é roadmap ou compromisso de release.**

| Capability | Classificação documental candidata | Condição/limite |
|---|---|---|
| Condominium | MVP | Onboarding e autoridade para tenant/primeiro admin precisam ser definidos. |
| Block | MVP | Identificação, unicidade, dependência de Unit e política de encerramento ainda requerem definição local. |
| Unit | MVP | Relação com Block e variações físicas dependem OD-13. |
| Person | MVP | Sem criação automática de conta; create capability/deduplicação/privacy ainda precisam de decisão. |
| UserAccount | CONDITIONAL | OD-03 e capabilities `user_account.*` ausentes bloqueiam lifecycle. |
| UnitOwnership | MVP | Authority/prova/copropriedade e regras OD-09 ainda bloqueiam a efetivação do vínculo. |
| UnitResidency | MVP | Authority, elegibilidade, múltiplos residentes/sobreposição OD-09 ainda bloqueiam a efetivação do vínculo. |
| UnitTenancy | MVP | Prova, período, rights e sobreposição OD-09 ainda bloqueiam a efetivação do vínculo. |
| Vehicle | POST-MVP | Marcado assim no catálogo de recursos; sem regras universais. |
| Pet | POST-MVP | Marcado assim no catálogo de recursos; política local ainda necessária. |
| Role / RoleAssignment | CONDITIONAL | OD-02/03/14/15/16 bloqueiam grant e enforcement completos. |
| CommonArea | MVP | Capacity/schedule/policy locais são necessários às reservas. |
| Reservation | MVP | OD-04 bloqueia política, conflicts, authority e transitions completas. |
| Event | CONDITIONAL | Catálogo sugere core/configurável; evento não concede access. |
| Guests | CONDITIONAL | São Persons contextuais; privacy, finalidade e association/access gates por definir. |
| Visit | MVP | Lifecycle final e autorização necessária dependem OD-05. |
| AccessAuthorization | MVP | Emitter/approval, validity, scope e recurrence OD-05. |
| AccessEvent | MVP | Observed facts conceitualmente definidos; fonte, permissions, correction/privacy devem estar aprovadas. |
| Package | MVP | Recebimento físico definido; confirmação/liberação/custody OD-06. |
| Notification | MVP | Intenção e independente de business facts; canal/expectativa de entrega OD-08. |
| NotificationDelivery | CONDITIONAL | Evidência, retry/cancelamento/retention OD-08; provider fora de escopo documental. |
| Assembly | POST-MVP | API catalog classifica escrita como post-MVP/OD-01; pode-se modelar estrutura sem validar deliberação. |
| Participant / AgendaItem | POST-MVP | Presença/convocação e lifecycle dependem OD-01. |
| VotingEligibility / Vote / Proxy / Quorum | CONDITIONAL | OD-01/02 e validação jurídica; não habilitar votação válida nem apuração/publicação definitiva. |
| Module / Feature | CONDITIONAL | Modelo conceitual existe; defaults, authority OD-11 e grants OD-17. |
| Audit | MVP | AUDIT-01..10 existem; privacy/retention e fields/operations ainda precisam revisão. |
| Finance | POST-MVP | OPTIONAL MODULE; OD-12; não ERP nem contabilidade oficial; scope e validação contábil/legal pendentes. |

As categorias refletem candidatos documentados e não significam decisão aprovada. Operações devem ser recortadas conforme a matriz de bloqueio abaixo.

## 16. Implementation Blockers

| Capability | Decisões necessárias | Pode implementar agora? | Bloqueador |
|---|---|---|---|
| Criar Condominium | Autoridade de onboarding/primeiro admin; identidade do ator. | CONDITIONAL | Só foundation e flow administrativo explicitamente limitado; grant inicial sem OD-02/03 não. |
| Criar Block | Authority, critério de identificação e política de encerramento/associação. | CONDITIONAL | Não resolver uniqueness nem transferir unidades por suposição. |
| Criar Unit | OD-13, identificador e regra de Block por tipo; authority. | CONDITIONAL | Apenas tipos/configuração aprovados; estacionamento/vagas não inferidos. |
| Person creation | Capability distinta `person.create` (não catalogada), finalidade, dados mínimos, matching/deduplicação e visibilidade. | CONDITIONAL | Person sem conta é decidido; persistência de PII operacional aguarda OD-07 e autoridade. |
| UserAccount invitation/activation | OD-03, cardinalidade, associação, authority, lifecycle, capabilities. | NO | OD-03; `user_account.*` ausentes. |
| UnitOwnership/Residency/Tenancy | OD-09: prova, autoridade, período/sobreposição, múltiplos vínculos/direitos. | CONDITIONAL | Tipos e invariantes podem ser modelados; efetivação/correção de vínculo não. |
| Role grant | OD-02/14/15/16, OD-03 para subject; scope/permission sem gaps. | NO | Authority, conflitos, delegação e capability final não decididos. |
| Role suspend/reactivate/revoke | Authority/efeitos/revisão/permissions por operação. | NO | State semantics parciais não resolvem ator autorizado. |
| Reservation create | Configuração OD-04, permission/actor/vínculo, horário/limites. | CONDITIONAL | Apenas em configuração explicitamente definida; ausência de policy não pode defaultar. |
| Reservation approve/reject | OD-04, authority, conflicts/prioridade, capabilities de reject e concurrency. | NO | Decision + authorization pendentes. |
| Reservation cancel/complete | Authority/capability, janela/financeiro e critério de conclusão. | NO | Preservação histórica está decidida; efeito de negócio permanece aberto. |
| AccessAuthorization | OD-05: who creates/approves/revokes, window/reuse/exceptions. | NO | Não usar permission de host/visitor por inferência. |
| AccessEvent register | Autor/source/grant, fields minimizados/privacy e regras de validação/correction. | CONDITIONAL | Registro estritamente factual pode ser possível com authority e contexto aprovados; caso contrário não. |
| Package receipt | Recebimento físico e actor/registrar definidos; destino, fields, authority e retention. | CONDITIONAL | Só se pacote foi fisicamente recebido e destino não for inferido; workflow completo aguarda OD-06/07. |
| Package confirm/release | OD-06: actor, representative, evidence, order, disputes, replay. | NO | CONFIRMED não pode ser tratado como PICKED_UP. |
| Notification intention | Purpose, recipient/visibility, `notification.create`, política para a intenção. | CONDITIONAL | Pode registrar intenção sem afirmar envio; audience/requiredness e privacy devem estar aprovadas. |
| Notification delivery/retry | OD-08: evidence, channel preference, fallback/retry/cancel/expiry. | CONDITIONAL | Registro de tentativa sem promessa pode ser avaliado após policy/canal; não afirmar delivered/read sem evidência. |
| Assembly create/schedule | OD-01/02/07 authority, convocação e dados. | CONDITIONAL | Estrutura rascunho pode existir sem alegar validade legal; convocação como válida não. |
| Presence/Proxy/VotingEligibility | OD-01/02; autoridade, evidência, elegibilidade e scope legal. | NO | Não inferir presença, representação ou direito a voto. |
| Vote / quorum / result publication | OD-01 integral e validação jurídica. | NO | Crítico; não registrar/apurar/publicar como válido. |
| Module/Feature enablement | OD-11 defaults/authority/dependencies/effects; OD-17 permissions. | NO | Modelo conceitual pode avançar; command funcional não tem authority/default/grants definidos. Feature enabled não concede permission. |
| Audit | AUDIT-01..10; operação/actor/contexto e política de visibilidade/retention OD-07. | CONDITIONAL | Requisito de audit sensível decidido; cobertura/campos/retention específicos ainda precisam revisão. |
| Finance | OD-12: capabilities, approvals, fields, documents, legal/accounting scope. | NO | Módulo opcional; não representar como contabilidade oficial. |

## 17. Human Decisions Required

1. Uma `Person` pode possuir mais de uma `UserAccount`?
2. Quem pode criar e associar uma `UserAccount` a uma `Person`?
3. Quem tem autoridade para conceder o papel `doorman` em um Condominium?
4. Um `syndic` pode conceder o papel `syndic` a outra pessoa?
5. Quais operações de RoleAssignment pode executar um `condominium_admin`, e em quais scopes?
6. Quem pode suspender e reativar um RoleAssignment, e qual é o processo de revisão?
7. `assistant_syndic` pode conceder alguma autoridade a terceiros? Se sim, quais permissions e limites?
8. `platform_admin` pode acessar dados operacionais de um Condominium em suporte? Qual finalidade e aprovação são exigidas?
9. A política de reserva é escolhida por Condominium, por CommonArea ou por ambos?
10. Uma reserva pode ser confirmada automaticamente, ou sempre precisa de aprovação? Em quais condições?
11. Uma reserva `PENDING` bloqueia temporariamente o horário?
12. Como resolver duas solicitações concorrentes para o mesmo horário quando ambas não podem ser confirmadas?
13. Quais são os limites de antecedência, duração, recorrência e cancelamento por tipo de área?
14. Quais fatos determinam que Reservation foi concluída ou que uma pessoa não compareceu?
15. Em quais situações o visitante precisa de AccessAuthorization?
16. Quem pode emitir/aprovar/revogar autorização de acesso, inclusive para acesso recorrente?
17. Qual é a janela de validade e a autorização pode ser usada mais de uma vez?
18. Quem pode corrigir `AccessEvent` e qual revisão/evidência essa correção exige?
19. Quem pode confirmar que recebeu um Package: destinatário, portaria ou ambos?
20. Retirada física pode ser registrada sem confirmação prévia?
21. Quais representantes podem retirar Package e como sua autoridade é verificada?
22. Qual procedimento se aplica a pacote danificado, recusado, sem destino identificado ou contestado?
23. Quais dados de pessoa, visitante, acesso, package, voto e audit cada papel pode consultar?
24. Quais categorias de dados têm prazo de retenção, descarte ou anonimização, após validação de privacidade/jurídica?
25. Para quais eventos a comunicação é obrigatória e para quais é apenas opcional?
26. Quais canais e preferências o Condominium pode configurar, e como tratar opt-out/fallback?
27. `Notification` tem um critério de encerramento agregado ou permanece como intenção com tentativas individuais?
28. Quem pode habilitar/desabilitar Module/Feature; quais são os defaults de um tenant novo?
29. O que ocorre com cada operação já iniciada quando uma feature é desabilitada?
30. Habilitar uma Feature concede algum read permission, ou cada acesso exige grant separado?
31. Em caso de grants conflitantes, qual precedência se aplica e existe `EXPLICIT_DENY`?
32. Assignment `CONDOMINIUM` cobre quais recursos descendentes, se algum? São necessários scopes Assembly/AccessPoint?
33. Há delegação de authority ou break-glass? Se sim, quem concede, com quais limites e revisão posterior?
34. Para cada command crítico, como distinguir retry da mesma operação de um novo pedido com os mesmos dados?
35. Quais regras legais e da convenção/regimento regem elegibilidade, peso, quórum, procurações, voto secreto, correção, reabertura e publicação da assembleia?
36. Quais capacidades da tabela MVP candidate realmente compõem o primeiro release?

As respostas precisam ser fornecidas pelo dono de produto/governança e, onde indicado, submetidas aos responsáveis jurídicos/de privacidade. Este relatório não as responde.

## 18. Traceability

| Decision | Business Rule | Permission | Workflow | State Machine | API Capability | Architecture Impact |
|---|---|---|---|---|---|---|
| OD-01 Assembleia | [ASSEMBLY-01..08](./docs/business-rules/assemblies.md) | `assembly.*`, `participant.*`, `voting_eligibility.*`, `proxy.*`, `vote.*`, `quorum.*` em [PERMISSIONS.md](./PERMISSIONS.md) | [WF-21/22](./docs/workflows/assemblies.md) | [Assembly/AgendaItem/Participant/Vote/Proxy](./docs/state-machines/communications-and-assemblies.md) | [Assembly, eligibility, Vote, result](./docs/api/resources.md) | Audit, atomic vote/close, ordering/replay e temporal model; commands deliberativos bloqueados. |
| OD-02 Grants | [RoleAssignment rules](./docs/permissions/role-assignment.md) | `role_assignment.grant/suspend/reactivate/revoke` | [WF-09](./docs/workflows/roles.md) | [RoleAssignment](./docs/state-machines/identity-and-reservations.md) | RoleAssignment commands em [API resources](./docs/api/resources.md) | Authorization boundary, actor attribution, audit atomicity e concorrência. |
| OD-03 Identidade | [IDENTITY-01..07](./docs/business-rules/identity.md) | `user_account.*` ainda ausente; `person.*` incompleto | [WF-04/05](./docs/workflows/identity-and-units.md) | [UserAccount](./docs/state-machines/identity-and-reservations.md) | Person/UserAccount em [API resources](./docs/api/resources.md) | Authentication subject, actor, account state, tenant association e audit. |
| OD-04 Reservas | [RESERVATION-01..08](./docs/business-rules/reservations.md) | `reservation.create/approve/cancel/update`; gaps permanecem | [WF-12/13](./docs/workflows/reservations-events.md) | [CommonArea/Reservation](./docs/state-machines/identity-and-reservations.md) | Reservation commands/queries em [API resources](./docs/api/resources.md) | Config por tenant, temporal model, consistency de slot e replay. |
| OD-05 Acesso | [ACCESS-01..09](./docs/business-rules/access.md) | `visit.*`, `access_authorization.*`, `access_event.*` | [WF-15..17](./docs/workflows/access.md) | [Visit/Authorization/Event](./docs/state-machines/access-and-packages.md) | Visita, autorização e fato em [API resources](./docs/api/resources.md) | Actor/source, tenant, instante, audit e correção referenciada. |
| OD-06 Packages | [PACKAGE-01..08](./docs/business-rules/packages.md) | `package.register/confirm/release/correct` | [WF-18/19](./docs/workflows/packages-notifications.md) | [PackageEvent/projeção](./docs/state-machines/access-and-packages.md) | Package commands em [API resources](./docs/api/resources.md) | Custody facts, concorrência, evidência, replay e retention. |
| OD-07 Privacy | [PRIVACY-01..10](./docs/business-rules/privacy.md) | [visibility/scopes](./docs/permissions/data-visibility.md) | Cross-cutting WF-04/15/18/21/25 | Lifecycles preservam histórico; retention não fixada | Query/export de campos em [API queries](./docs/api/queries.md) | Minimização em tenant, audit, logs, cache, reports e backup/retention. |
| OD-08 Notification | [NOTIFICATION-01..08](./docs/business-rules/notifications.md) | `notification.*`, channel config | [WF-20](./docs/workflows/packages-notifications.md) | [Notification/Delivery](./docs/state-machines/communications-and-assemblies.md) | Notification create/read/retry em [API resources](./docs/api/resources.md) | Delivery assíncrona, evidence, retry, outbox condicional e privacy. |
| OD-09 Unit links | [UNIT-01..03](./docs/business-rules/units.md) | `unit_ownership/residency/tenancy.*` | [WF-06..08](./docs/workflows/identity-and-units.md) | [Unit relationships](./docs/state-machines/identity-and-reservations.md) | Vínculos tipados em [API resources](./docs/api/resources.md) | Tenant ownership, temporal intervals, audit e authorization context. |
| OD-11/17 Features | [MODULE-01..07](./docs/business-rules/modules.md) | `module.*`, `feature.*`; grants por feature abertos | [WF-23](./docs/workflows/modules-finance.md) | [CondominiumModule/Feature](./docs/state-machines/modules.md) | Enable/disable em [API resources](./docs/api/resources.md) | Feature availability ≠ permission; tenant config, concurrent toggle, audit. |
| OD-14/15/16 | Authorization model and scopes in [authorization-model.md](./docs/permissions/authorization-model.md) | Role permissions, Scope and delegation | [WF-09](./docs/workflows/roles.md) | [RoleAssignment](./docs/state-machines/identity-and-reservations.md) | Auth mapping in [API authorization](./docs/api/authorization.md) | Authorization boundary, fail-closed resource access, job/support actor. |
| OD-18 | Business invariants in [BUSINESS-RULES.md](./BUSINESS-RULES.md) and domain-specific rules above | Permission of each command remains independent from replay | All operations, notably WF-13/17/19/20/21 | Events vs states and per-operation lifecycles | [Idempotency/concurrency](./docs/api/idempotency.md) | Audit atomicity, transaction/replay identity, concurrency and uncertain commits. |

## 19. Documents Changed

- Criado `DOMAIN-DECISIONS-CLOSURE.md`.
- Atualizados `OPEN-DECISIONS.md`, `README.md` e `docs/README.md` para apontar para este fechamento e indicar que as ODs continuam abertas.
- `BUSINESS-RULES.md`, `PERMISSIONS.md`, `WORKFLOWS.md` e state machines não foram modificados: a análise não recebeu decisão humana nova e não encontrou base para alterar regras, authorities ou transições já documentadas.
- Os documentos de arquitetura e API também não foram alterados nesta etapa.

## 20. Validation

Validação concluída: `git diff --check` passou; links/âncoras Markdown relativos e fences nos arquivos alterados foram verificados; a matriz contém exatamente os 15 IDs solicitados, uma classificação por OD, categorias válidas de MVP e statuses YES/NO/CONDITIONAL; o relatório contém 21 seções e 36 perguntas numeradas. Não há testes de runtime aplicáveis a esta alteração documental; nenhum artefato de aplicação foi criado.

## 21. Final Status

`DOMAIN_DECISIONS_REQUIRES_HUMAN_INPUT`

Os conceitos e invariantes já documentados foram distinguidos das escolhas ainda abertas. As decisões humanas da seção 17 são necessárias antes de fechar os commands e a persistência afetados. Não há decisão de produto, permission, regra jurídica ou tecnologia tomada implicitamente neste relatório.
