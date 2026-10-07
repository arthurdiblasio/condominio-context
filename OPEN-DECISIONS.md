# OPEN-DECISIONS.md

## Objetivo

Registrar somente decisões que requerem autoridade de produto, política local, governança ou validação jurídica. Não são especificações técnicas nem recomendações jurídicas.

O [API Contract conceitual](./API-CONTRACT.md) referencia as decisões abaixo e destaca lacunas de authority/capabilities por operação. Não adiciona endpoints, protocolos ou decisões técnicas a este registro.

A classificação e a análise de bloqueios das ODs solicitadas estão em [DOMAIN-DECISIONS-CLOSURE.md](./DOMAIN-DECISIONS-CLOSURE.md). Esse relatório não fecha decisões sem autoridade humana, não altera a prioridade registrada e não substitui as definições abaixo.

## CRITICAL

### OD-01 — Regras de assembleia e validade de votação

Definir convocação, critérios de elegibilidade por pauta, peso, quórum, efeitos de inadimplência quando aplicável, procuração/representação, voto secreto ou aberto, abertura/fechamento de assembleia e pauta, correção, repetição/substituição de voto, reabertura e validade do resultado; submeter as regras pertinentes à validação jurídica. Distinguir inscrição, presença/ausência, representação, elegibilidade, registro do voto, apuração e publicação; definir autoridade e momento para cada transição.

**Impacto:** determina quem pode deliberar e se uma deliberação pode ser considerada válida.

### OD-02 — Governança de papéis e permissões

Definir quem pode conceder/suspender/revogar `RoleAssignment` em cada escopo, quem pode delegar, quais permissões pertencem a cada papel e se há herança, precedência ou negação. Confirmar restrição contra concessão de autoridade superior à do concedente, autoatribuição e exceções delegadas.

**Impacto:** define acesso a dados e operações sensíveis entre plataforma, condomínios, blocos, unidades e features.

## HIGH

### OD-03 — Person e UserAccount

Decidir cardinalidade e associação entre `Person` e `UserAccount`, autoridade para criação/convite, ativação, associação, bloqueio, suspensão, desativação, recuperação, reativação e efeito sobre vínculos e atribuições. Definir validade/expiração de convite, transições irreversíveis e quais capabilities permitem cada transição de conta; o catálogo atual ainda não contém `user_account.*`. Decidir autoridade para primeiro administrador no onboarding. Não definir método técnico de autenticação nesta decisão de domínio.

**Impacto:** afeta identidade, acesso, histórico e continuidade das relações de domínio.

### OD-04 — Política de reservas e áreas comuns

Definir por condomínio/área a política de aprovação (`AUTO_APPROVED`, `MANUAL_APPROVAL` ou `RULE_BASED`), disponibilidade, duração, antecedência, conflitos, concorrência/prioridade entre solicitações simultâneas, capacidade, bloqueios/manutenção, cancelamento e efeitos de cobrança/reembolso. Especificar quando uma solicitação fica pendente, rejeitada ou expira; critério de conclusão, alterações após confirmação, destino de reservas afetadas por bloqueio/desativação e condições para reabrir um estado terminal.

**Impacto:** determina alocação de recursos compartilhados e estados/transições de reservas.

### OD-05 — Visitas e controle de acesso

Definir quem pode criar/revogar autorizações, quais visitas exigem autorização, regras de validade/uso repetido, identificação, exceções e procedimento de divergência entre entrada/saída observada e registros. Distinguir cancelamento de autorização de revogação, definir quando convite/visita pode ser marcado concluído ou no-show, janela de expiração e autoridade/procedimento para corrigir eventos de acesso ou reabrir visitas/autorização.

**Impacto:** afeta controle de entrada, segurança operacional e trilha de auditoria.

### OD-06 — Pacotes: confirmação, retirada e contestação

Definir quem pode confirmar ou retirar, quando representante pode agir, se confirmação é obrigatória antes da retirada, mecanismo de autorização, evidências aceitáveis, contestação, correção, tratamento de destino desconhecido e resposta a confirmação/retirada repetida ou concorrente. Definir a regra para derivar a situação corrente do pacote a partir de `PackageEvent`, a ordem permitida de confirmação/retirada/cancelamento e se cancelamento após retirada é possível.

**Impacto:** define responsabilidade por custódia e resolução de disputas operacionais.

### OD-07 — Privacidade, dados e retenção

Definir finalidades, categorias de dados pessoais/sensíveis, visibilidade entre moradores, acesso da portaria/administração, retenção, descarte/anonimização, consentimento quando aplicável e tratamento de evidências/logs, com validação apropriada de privacidade e jurídica.

**Impacto:** afeta exposição de dados e obrigações de conformidade. Não fixar prazos legais sem validação.

### OD-14 — Conflito entre múltiplos grants e roles

Escolher precedência entre grants concorrentes, inclusive se haverá `EXPLICIT_DENY`, como negar por padrão e se qualquer exceção de prioridade de papel existirá. Modelo recomendado em [PERMISSIONS.md](./PERMISSIONS.md): negar em ausência/ambiguidade, explicit deny overrides allow e evitar role priority; ainda não é decisão aprovada.

**Impacto:** pode ampliar ou bloquear privilégios quando uma pessoa possui papéis múltiplos ou grants sobrepostos.

### OD-15 — Escopo e abrangência de recursos

Definir se um assignment em Condominium pode cobrir recursos descendentes e quais permissions podem declarar essa abrangência; confirmar se novos scopes, como assembleia ou ponto de acesso, são necessários.

**Impacto:** determina limite efetivo de grants sem permitir herança silenciosa de papel ou acesso.

### OD-16 — Delegação de autoridade e acesso emergencial

Definir permissions delegáveis, limites e vigência da delegação, transitividade, revisão/auditoria e se é necessário procedimento break-glass. Não criar papel ou exceção de emergência até essa decisão.

**Impacto:** evita escalada de privilégios e define como tratar acesso excepcional em situação urgente.

## MEDIUM

### OD-08 — Canais e política de notificação

Definir canais habilitados, prioridades, opt-in/opt-out e exceções, fallback, retry, significado de entrega/leitura, conteúdo e retenção. Definir quais resultados de `NotificationDelivery` são distinguíveis, expiração/cancelamento de uma tentativa e critério para considerar encerrada a intenção agregada `Notification`; retry deve continuar sendo nova tentativa, sem apagar anterior. Decidir provedor WhatsApp e templates quando o produto decidir implementar esse canal.

**Impacto:** afeta expectativa de comunicação e tratamento de falhas; não altera o fato de negócio notificado.

### OD-09 — Vínculos residenciais e titularidade

Definir prova e autoridade para criar/encerrar `UnitOwnership`, `UnitResidency` e `UnitTenancy`, copropriedade, múltiplos responsáveis, períodos futuros/sobrepostos, dependentes e direitos associados. Confirmar se vínculos com início futuro são aceitos e quando passam a efetivos; definir correção de datas e sobreposição. Regras legais devem ser validadas.

**Impacto:** influencia visibilidade, responsabilidades e eventual elegibilidade; os vínculos devem permanecer conceitualmente separados.

### OD-10 — Veículos, estacionamento e pets

Definir vínculo de veículos temporários/de visitantes, tratamento de duplicidade/placas, estacionamento/vaga e política local de cadastro/responsáveis por pets. Não estabelecer limites universais de veículos ou animais.

**Impacto:** afeta cadastro operacional, portaria e regras de convivência.

### OD-11 — Habilitação de módulos e features

Definir quem habilita/desabilita `CondominiumModule` e `CondominiumFeature`, defaults, dependências, incompatibilidades, efeito sobre operações em andamento, critério para habilitação inicial/reativação e acesso aos dados após desativação. Preservação automática do histórico é princípio; não apagar dados por desabilitar módulo.

**Impacto:** define disponibilidade funcional por tenant e transição segura entre configurações.

### OD-12 — Módulo financeiro e prestação de contas

Definir escopo de receitas/despesas, fornecedores, documentos, aprovações, relatórios e permissões. Avaliar obrigações contábeis/jurídicas antes de definir comportamento oficial. O módulo permanece opcional.

**Impacto:** evita que ferramenta de prestação de contas seja confundida com ERP ou contabilidade oficial.

### OD-17 — Grants por feature habilitada

Definir quais permissions específicas são necessárias para utilizar cada feature habilitada, inclusive se habilitação expõe alguma capacidade de leitura ou se todo uso exige grant individual.

**Impacto:** evita que habilitar uma feature conceda acesso indevido a todos os usuários do condomínio.

### OD-18 — Repetição de operações e concorrência operacional

Definir, por operação, quando uma repetição deve ser reconhecida como repetição sem novo fato (`IDEMPOTENT`/`NO-OP`), rejeitada (`REJECT`), corrigida ou aceita como evento distinto (`NEW_EVENT`); especialmente recebimento/confirmação/retirada de pacote, presença/voto, acesso observado, cancelamento/revogação e grants concorrentes. Definir resultados de conflito operacional sem escolher mecanismos de sincronização. Para reservas, a regra de prioridade/conflito pertence à OD-04; esta decisão cobre repetição de comandos e duplicação de fatos.

**Impacto:** evita duplicidade de custódia, dupla contagem de presença/voto, conflito de autoridade e confirmação concorrente de recurso. Não escolhe mecanismo técnico de idempotência ou sincronização.

## LOW

### OD-13 — Variação da estrutura física

Confirmar se todo tipo de unidade requer bloco, como representar estacionamentos/vagas e se há áreas comuns vinculadas a bloco ou diretamente ao condomínio.

**Impacto:** afeta variações do modelo estrutural, sem alterar isolamento de tenant.

## Decisões técnicas fora deste documento

Provedores de identidade e mensagens, protocolos, armazenamento, tokens, QR Code, controle físico, notificações push e infraestrutura são decisões técnicas futuras. Não constituem regras de negócio e não são escolhidos aqui.

## PERMISSIONS / AUTHORIZATION

Decisões desta etapa, priorizadas e sem duplicar os itens gerais acima:

### CRITICAL

- **OD-02 — Concessão e revogação:** quem pode administrar cada RoleAssignment, autoatribuição, elevação de autoridade, papéis privilegiados e limites por scope.
- **OD-14 — Conflito entre roles/permissões:** `ALLOW`, `DENY`, `EXPLICIT_DENY`, precedência, negação por padrão e proibição ou exceção de `ROLE_PRIORITY`.

### HIGH

- **OD-15 — Abrangência de scopes:** cobrir recursos descendentes, herança explicitamente declarada e eventual necessidade de scopes adicionais.
- **OD-16 — Delegação e break-glass:** quem delega/recebe, permissions delegáveis, vigência, limites, emergência e auditoria.
- **OD-07 — Visibilidade e dados pessoais:** campos por papel/finalidade, acesso de portaria/admin e restrições específicas de dados sensíveis.
- **OD-03 — Subject autenticado:** associação Person-UserAccount, cardinalidade, estados da conta e requisito de conta para cada tipo de ação.

### MEDIUM

- **OD-17 — Feature versus grant:** permissões específicas por módulo/feature e efeito da habilitação sobre o acesso sem concessão implícita a usuários.
- **OD-10 — Acesso por papel residencial/operacional:** quais capacidades padrão são vinculadas a owner/resident/tenant/employee e que customização local é permitida sem confundir papel com vínculo.
- **OD-18 — Repetição/concorrência:** semântica de repetição em operações críticas e regra de resolução para concorrência além do conflito de reservas.

As marcações de prioridade indicam impacto potencial e devem ser confirmadas por governança de produto; a matriz de papéis/permissões é proposta, não grant efetivo. Ver [PERMISSIONS.md](./PERMISSIONS.md) e [docs/permissions/](./docs/permissions/).

## Documentos relacionados

- [BUSINESS-RULES.md](./BUSINESS-RULES.md)
- [DOMAIN.md](./DOMAIN.md)
- [PERMISSIONS.md](./PERMISSIONS.md)
- [WORKFLOWS.md](./WORKFLOWS.md)
- [API-CONTRACT.md](./API-CONTRACT.md)
- [docs/api/authorization.md](./docs/api/authorization.md)
- [docs/domain/domain-model.md](./docs/domain/domain-model.md)
