# Catálogo de recursos da API

Classificação proposta, sujeita a revisão do primeiro release. **Core** não significa CRUD: somente operações necessárias aos workflows são listadas. `INTERNAL` não é recurso público direto, embora consultas/ações que o envolvam possam existir por outro recurso.

| Conceito/recurso | Classe proposta | Exposição conceitual | Limites / CRUD |
|---|---|---|---|
| `Platform` | ADMINISTRATIVE | Contexto de operações globais autorizadas. | Não expor entidade global mutável; somente operações administrativas justificadas. |
| `Condominium` | CORE + ADMINISTRATIVE | Consultar/configurar tenant; comando administrativo para criar. | Administração SaaS separada da operação do condomínio. |
| `Block` | CORE | Consultar/criar/alterar/encerrar estruturalmente. | Sem transferência entre tenants. |
| `Unit` | CORE | Consultar/criar/alterar/encerrar estruturalmente; relacionar com pessoas por vínculos tipados. | Não embutir status de proprietário/residente/locatário. |
| `Person` | CORE | Criar, consultar, atualizar e relacionar dentro de finalidade e tenant autorizados. | Não usar como sinônimo de conta ou role; identidade/deduplicação permanece OD-03/07/09. |
| `UserAccount` | ADMINISTRATIVE | Operações conceituais de convite, ativação, suspensão, reativação e desativação. | Sem login/authentication API nesta etapa; capabilities `user_account.*` ausentes, ver OD-03. |
| `Role` | INTERNAL / somente leitura administrativa se aprovado | Catálogo consultável/selecionável ao gerir atribuições. | Não implica permissão, nem CRUD de definição de role nesta superfície base. |
| `RoleAssignment` | ADMINISTRATIVE | Consultar histórico e emitir comandos de grant, suspend, reactivate, revoke. | Sem PATCH/CRUD genérico; `Permission` e `Scope` permanecem distintos. |
| `Permission` | INTERNAL / catálogo de referência | Metadado conceitual necessário para administração autorizada de grants. | Não conceder/alterar permission diretamente através de recurso genérico. |
| `Scope` | INTERNAL / valor de contexto | Condição avaliada e informada em operações autorizadas. | Não é objeto com CRUD público. |
| `UnitOwnership`, `UnitResidency`, `UnitTenancy` | CORE | Comandos tipados para criar, encerrar e corrigir; consulta do atual e histórico. | Sem exclusão comum; cada tipo tem authority distinta. |
| `Vehicle`, `Pet` | POST-MVP / configurável | Operações/cadastro conforme política local. | Não essenciais ao core inicial proposto; sem regras universais inventadas. |
| `CommonArea` | CORE | Consultar disponibilidade/configuração; comandos de criar, configurar, bloquear, manutenção, reabrir/desabilitar. | Disponibilidade não cancela reserva por inferência. |
| `Reservation` | CORE | Consultar disponibilidade/reservas; comandos de solicitar, aprovar, rejeitar, alterar, cancelar e concluir quando autorizado. | Sem CRUD genérico de estado. |
| `Event` | CORE / configurável | Consultar/criar/alterar/cancelar eventos e pessoas convidadas quando aplicável. | Evento não autoriza acesso por si só. |
| `Visit` | CORE | Consultar/registrar/alterar/cancelar contexto de visita. | Presença física não se inventa por CRUD. |
| `AccessAuthorization` | CORE | Criar/consultar/revogar/cancelar segundo política. | Não comprova entrada. |
| `AccessEvent` | INTERNAL operacional; leitura controlada e comandos para registrar/corrigir | Registro de fatos ENTRY/EXIT/DENIED e consulta restrita. | Sem create/update/delete genérico; gerado por comando observado. |
| `Package` | CORE | Registrar recebimento, consultar situação/histórico, confirmar e registrar retirada/cancelamento por comandos. | Estado corrente pode ser derivado e não está aprovado. |
| `PackageEvent` | INTERNAL histórico | Consulta histórica; fatos gerados por comandos Package. | Sem criação, edição ou deleção genérica. |
| `Notification` | CORE operacional | Criar quando regra permite; consultar intent/destinatário conforme visibilidade; cancelar quando política permitir. | Não é canal/provedor ou estado de entrega. |
| `NotificationDelivery` | INTERNAL, leitura operacional restrita | Consultar tentativas/resultados; iniciar reenvio apenas se permitido como nova tentativa. | Nunca expor comando genérico para provedor ou editar resultado. |
| `Assembly` | POST-MVP / CORE de domínio com regra legal | Criar, agendar, convocar, abrir, fechar, cancelar, consultar conforme processo aprovado. | Superfície de escrita depende de OD-01 e validação jurídica. |
| `Participant` | POST-MVP / CORE de domínio | Registrar/consultar participant e fato de presença; corrigir por operação auditada. | Não confundir presença com elegibilidade ou representação. |
| `AgendaItem` | POST-MVP / CORE de domínio | Criar/alterar/abrir votação/encerrar/cancelar pauta. | Comandos distintos para discussão, votação e encerramento. |
| `VotingEligibility` | INTERNAL / decisão operacional sensível | Determinar, contestar/reavaliar e consultar somente sob grants e regra aplicável. | Não é Permission nem CRUD de elegibilidade. |
| `Proxy` | POST-MVP / CORE de assembleia | Registrar, validar/decidir, revogar, consultar e registrar uso. | Não é RoleAssignment; depende de OD-01/02. |
| `Vote` | INTERNAL / ato de negócio sensível | Comando de registrar; consultar apenas conforme sigilo e permissão. Corrigir/invalidar por workflow separado. | Sem CRUD genérico ou alteração silenciosa. |
| `Quorum` / resultado | INTERNAL / consulta derivada | Consultar/apurar/publicar apenas sob regra aprovada. | Não há operação de cálculo/validação jurídica fechada. |
| `Module`, `Feature` | INTERNAL / catálogo administrativo | Consulta de capabilities globais ao ator autorizado. | Sem CRUD público geral do catálogo no tenant API. |
| `CondominiumModule`, `CondominiumFeature` | ADMINISTRATIVE | Consultar e habilitar/desabilitar no tenant explícito. | Ação administrativa auditada; habilitação não concede grants nem elimina histórico. |
| `AuditLog` | INTERNAL, somente leitura restrita | Consultas específicas por finalidade/tenant/scope. | Sem edição ou deleção por API operacional comum. |
| `Income`, `Expense`, `Supplier`, `FinancialDocument`, `FinancialReport` | OPTIONAL MODULE | Operações mínimas para registro, consulta, fornecedor e relatório sob módulo/grants. | Não constituem ERP nem contabilidade oficial. |

## Recursos não publicados diretamente

As entidades abaixo não devem ser expostas como coleção livre para CRUD: `PackageEvent`, `AccessEvent`, `NotificationDelivery`, `AuditLog`, `Vote`, `VotingEligibility`, `Permission`, `Scope`, `Role`, `Quorum`. Operações correspondentes devem passar pelo comando de negócio ou consulta autorizada que preserve as invariantes.

## Escopo do release — proposta

- **MVP candidato:** `Condominium`, `Block`, `Unit`, `Person`, vínculos tipados, `RoleAssignment`, `CommonArea`, `Reservation`, `Visit`, `AccessAuthorization`, fatos de acesso, `Package`, `Notification` e auditoria necessária.
- **Administrativo separado:** criação/configuração de condomínio, conta, roles/assignments e habilitação de módulo/feature.
- **Post-MVP candidato:** `Vehicle`, `Pet`, eventos e operações avançadas de assembleia/proxy/voto; status jurídico/legal continua bloqueador para uso deliberativo.
- **Módulo opcional:** financeiro básico e relatórios internos limitados, não ERP.
- **Future / decisão pendente:** APIs de login/autenticação, catálogo global mutável de Permission/Role, publicação de resultados jurídicos, gestão de retenção e integrações/provedores.

Essa classificação é proposta para revisão, não compromisso de roadmap nem substituto de decisão legal.
