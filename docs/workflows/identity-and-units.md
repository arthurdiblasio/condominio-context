# Identidade, unidade e relações

## WF-02 — Cadastrar ou alterar Block

### Objetivo

Manter estrutura física de blocos em um condomínio.

### Participantes

Iniciador: `condominium_admin` ou `property_manager` candidato; executor somente com permission no tenant. Responsáveis finais são configuráveis.

### Contexto/Tenant

Um `Condominium` identificado. Block não pode ser movido/associado implicitamente a outro tenant.

### Gatilho e pré-condições

Necessidade de criar, atualizar ou encerrar um bloco; Condominium existe e está apto a ser administrado.

### Permissões necessárias

`block.manage` no scope `CONDOMINIUM`; leitura para revisão exige `block.read`. Catálogo não concede essas permissions automaticamente.

### Dados/conceitos envolvidos

`Condominium`, `Block`, `Unit`, `CommonArea`, `AuditLog`.

### Fluxo principal

1. Ator seleciona explicitamente o Condominium.
2. Informa identificação e dados definidos como necessários.
3. Valida-se unicidade conforme critério de identificação aprovado no tenant.
4. Cria-se ou atualiza-se Block no mesmo tenant.
5. Para encerramento/desativação, verifica-se referência a Units/CommonAreas; vínculos não são movidos ou apagados automaticamente.
6. As unidades continuam associadas ao bloco/histórico até que regra explícita determine mudança.

### Fluxos alternativos, validações e transições

Block pode estar em uso. Reatribuição de unidades, exclusão física, desativação, reativação e tratamento de áreas dependem de política estrutural. Uma Block de outro tenant não pode ser associada.

### Eventos/fatos e notificações

Fato de cadastro/alteração/encerramento de bloco; notificação somente se configuração local definir destinatários e finalidade.

### Auditoria

Registrar ator, Condominium, recurso, campos/conceito alterado, resultado e contexto. Correção preserva trilha.

### Pós-condições/término

Estrutura consistente dentro do mesmo tenant; cadastro encerra ao persistir mudança conceitual aceita ou negar operação.

### Erros/negações

Permission ausente; tenant incompatível; duplicidade; bloco inexistente; dependências não resolvidas; estado incompatível.

### OPEN DECISIONS

Identificador único, nomenclatura, possibilidade de unidade sem bloco, desativação e política de reatribuição de unidades/áreas.

---

## WF-03 — Cadastrar, configurar ou encerrar Unit

### Objetivo

Representar unidade pertencente a um único Condominium e, quando aplicável, um Block.

### Participantes

Iniciador/executor candidato: `condominium_admin`, `property_manager` ou `syndic` com `unit.update` e scope adequado.

### Contexto/Tenant

Condominium explicitamente selecionado; Block opcional somente quando o modelo do tipo de propriedade permitir.

### Gatilho e pré-condições

Criação, ajuste estrutural, associação a bloco, registro de estacionamento/vaga se aprovado ou encerramento.

### Permissões necessárias

`unit.update` e, para a associação correspondente, `block.manage`; cada scope deve cobrir recurso alvo. Códigos do catálogo não são grants automáticos.

### Dados/conceitos envolvidos

`Condominium`, `Block`, `Unit`, `CommonArea`/estacionamento se aplicável, vínculos de pessoa e `AuditLog`.

### Fluxo principal

1. Seleciona-se Condominium e Block, se aplicável.
2. Valida-se que Block pertence ao Condominium e que identificador da Unit não conflita conforme política aprovada.
3. Registra-se tipo/configuração estrutural sem inferir propriedade, residência ou locação.
4. Associações de pessoas seguem WF-06/07/08; conta de usuário segue WF-05.
5. Encerramento/desativação verifica reservas, acessos e vínculos ativos; não apaga nem migra história automaticamente.

### Fluxos alternativos, validações e transições

Unit sem Block pode ser aceita apenas para tipos permitidos. Estacionamento/vaga não ganha modelo, exclusividade ou direitos por suposição. Desativação com atividades/vínculos em andamento exige resolução conforme regra aprovada; operação cross-tenant é negada.

### Eventos/fatos e notificações

Criação, alteração, associação/desassociação de Block e encerramento da Unit; notificações apenas se explicitamente configuradas.

### Auditoria

Auditar criação, alteração estrutural, associação de bloco e encerramento, com tenant, ator e resultado.

### Pós-condições/término

Unit pertence a exatamente um Condominium; Block, se definido, pertence ao mesmo Condominium.

### Erros/negações

Falta de permission; condomínio/bloco incompatíveis; identificador duplicado segundo regra; estado que impede alteração; vínculos ativos sem tratamento.

### OPEN DECISIONS

Bloco obrigatório por tipo; identificadores, estacionamentos/vagas, exclusividade de recursos e efeitos de encerramento.

---

## WF-04 — Criar/atualizar Person e associação tenant-scoped

### Objetivo

Representar pessoa e relações em condomínio sem criar automaticamente UserAccount.

### Participantes

Iniciador pode ser pessoa identificada, administrador ou agente com autorização; executor de cadastro depende de `person.update`. Fluxo de autopreenchimento/auto-registro não está aprovado.

### Contexto/Tenant

Person pode participar de diversos condomínios. Cada associação, dado operacional e vínculo unitário preserva o tenant de origem.

### Gatilho e pré-condições

Necessidade operacional legítima de representar pessoa para vínculo, acesso, pacote, visita ou participação. Dados limitados à finalidade.

### Permissões necessárias

`person.update` e scope/permission de associação aplicável; o catálogo não lista uma permission separada `person.create`, portanto a necessidade de distingui-la deve ser aprovada antes de implementar. Leitura segue `data-visibility.md`.

### Dados/conceitos envolvidos

`Person`, `UserAccount` opcional, `UnitOwnership`, `UnitResidency`, `UnitTenancy`, `RoleAssignment`, `AuditLog`.

### Fluxo principal

1. Identifica-se finalidade, Condominium e campos necessários.
2. Procura-se identidade existente no escopo permitido para evitar duplicidade sem fazer busca global indevida.
3. Cria-se Person ou atualiza-se dado autorizado; associação ao tenant é explicitamente contextual.
4. Vínculo com unidade é feito por workflow tipado separado.
5. Se a pessoa precisar de acesso, encaminha-se a WF-05; não se infere conta.

### Fluxos alternativos e validações

Possível duplicidade ou identidade não verificável interrompe a fusão automática; conflito de tenant nega acesso; correção de dados pessoais preserva atribuição/auditoria conforme regras de privacidade.

### Eventos/fatos e notificações

Person criada/atualizada ou associada a contexto local. Notificação somente se a finalidade e canal estiverem aprovados.

### Auditoria

Auditar mudanças relevantes, ator, finalidade/contexto, Condominium e campos categóricos alterados sem expor mais dados pessoais que o necessário.

### Pós-condições/término

Person representada e associada somente aos contextos autorizados; vínculos e conta permanecem independentes.

### Erros/negações

Sem finalidade/permission; duplicidade não resolvida; scope insuficiente; tenant incorreto; campo sem autorização; regra de privacidade não satisfeita.

### OPEN DECISIONS

Identificação/deduplicação, autorregistro, dados mínimos, permissões de leitura por campo, associação global versus local e correção/anonimização.

---

## WF-05 — Criar/associar/ativar/suspender/desativar UserAccount

### Objetivo

Disponibilizar acesso a uma Person sem substituir a identidade, vínculos ou grants.

### Participantes

Person convidada e ator convidador com permission de administração de convite/conta. Papel que concede convite ou ativa conta não está fechado.

### Contexto/Tenant

Conta pode ser global, mas cada acesso operacional continua tenant-scoped por RoleAssignment. Não há transferência automática entre tenants.

### Gatilho e pré-condições

Person existente ou identificada conforme política, finalidade de acesso e necessidade de autenticação.

### Permissões necessárias

O catálogo atual não define capabilities específicas `user_account.*`; `person.update` não é automaticamente equivalente. Permissão necessária e atores elegíveis são uma lacuna a decidir antes de implementação.

### Dados/conceitos envolvidos

`Person`, `UserAccount`, `RoleAssignment`, status da conta, `AuditLog`, Notification opcional.

### Fluxo principal

1. Ator com autoridade solicita associação/convite.
2. Verifica-se a Person-alvo e ausência de conflito/duplicidade segundo política.
3. Registra-se convite sem ativar acesso automaticamente.
4. Associação é confirmada pelo processo de identidade aprovado.
5. Conta torna-se active somente após requisitos definidos; acesso a cada tenant continua dependendo dos grants respectivos.

### Fluxos alternativos e ciclo

Lifecycle de referência: [UserAccount](../state-machines/identity-and-reservations.md#useraccount). Estados candidatos: `INVITED`, `ACTIVE`, `SUSPENDED`, `DISABLED`; expiração encerra convite se aprovada, não é estado universal de conta. Não há capabilities `user_account.*` no catálogo; ator e authority ficam em OD-03. Suspensão/desativação bloqueia novas ações autenticadas conforme efetividade definida, mas preserva Person, vínculos, RoleAssignments históricos e autoria.

### Validações

Não associar conta a pessoa incorreta; não ativar sem requisitos; não conceder grants por convite; não permitir que conta suspensa aja.

### Eventos/fatos e notificações

Convite, associação, ativação, suspensão, reativação e desativação são fatos separados. Mensagem de convite/estado pode ser enviada; falha não ativa nem desativa conta.

### Auditoria

Auditar ator convidador, pessoa-alvo, mudanças de associação/estado, momento e resultado. Não registrar segredos de autenticação.

### Pós-condições/término

Conta ativa associada à Person, ou fluxo encerrado sem acesso; grants tenant-scoped não são alterados por mera desativação.

### Erros/negações

Person não verificável; duplicidade; ator sem authority; tenant/contexto impróprio; convite rejeitado/expirado; requisitos de ativação ausentes.

### OPEN DECISIONS

OD-03 em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md): cardinalidade pessoa-conta, criação, convite, associação, ativação, suspensão/desativação, recuperação e efeitos sobre grants.

---

## WF-06/07/08 — UnitOwnership, UnitResidency e UnitTenancy

### Objetivo

Criar, ajustar, iniciar e encerrar três relações distintas entre Person e Unit, preservando validade temporal e história.

### Participantes

Person relacionada e administrador/representante autorizado para manutenção. O titular da authority e verificação documental requerem política validada.

### Contexto/Tenant

Unit e relação pertencem ao mesmo Condominium; pessoa pode ter outros vínculos em outros tenants sem contaminação de dados.

### Gatilho e pré-condições

Compra/alteração de titularidade; início/encerramento de residência; locação/ocupação; correção. Cada tipo usa o workflow próprio.

### Permissões necessárias

`unit_ownership.manage`, `unit_residency.manage` ou `unit_tenancy.manage` com scope Unit ou Condominium cobrindo explicitamente a unidade. Pessoas relacionadas não recebem automaticamente poder de manter o próprio vínculo.

### Dados/conceitos envolvidos

`Person`, `Unit`, um tipo de vínculo, período, evidência/documentação conforme policy, `RoleAssignment` eventualmente separado, `AuditLog`.

### Fluxo principal

1. Seleciona-se tipo de vínculo, pessoa, unidade e Condominium explicitamente.
2. Verifica-se tenant coerente, autoridade do mantenedor e campos/documentos definidos como exigidos.
3. Cria-se/agenda-se início do vínculo sem alterar os outros tipos de vínculo.
4. Sobreposição ou copropriedade é permitida somente se política aplicável aceitar; não se presume exclusividade.
5. Alteração/encerramento registra vigência final e motivo/contexto permitido, preservando relação histórica.
6. Ajustes em RoleAssignment/UserAccount ocorrem separadamente, nunca como efeito implícito do vínculo.

### Fluxos alternativos e validações

Vínculo futuro pode ser aceito apenas se política definir; conflito de datas não resolvido interrompe/encaminha revisão; encerramento não apaga reservas, pacotes, auditoria nem acessos anteriores. Múltiplas unidades/residências não são proibidas pelo modelo sem decisão.

### Eventos/fatos e notificações

Vínculo criado, iniciado, alterado, encerrado ou corrigido. Notificações a envolvidos dependem de finalidade e política.

### Auditoria

Auditar tipo, Unit, Person, Condominium, vigência afetada, ator, resultado e referência corretiva quando houver; proteger documentos pessoais.

### Pós-condições/término

Relação correta e temporalmente identificável; nenhuma identidade, papel ou outra relação alterada automaticamente.

### Erros/negações

Permission ausente; Unit/person de tenants incompatíveis; vigência inválida; conflito segundo regra local; autoridade/documentação insuficiente; tentativa de eliminar história.

### OPEN DECISIONS

OD-09: autoridade e comprovação, copropriedade, dependentes, sobreposição, vigência futura, residência múltipla e direitos legais.

---

## WF-10 — Cadastrar/alterar/desativar Vehicle

### Objetivo

Manter veículo associado a Person e, quando aplicável, Unit/Condominium sem confundir veículo de visitante com residente.

### Participantes

Person responsável, `resident`/`owner`/`tenant` candidatos; `doorman` ou administração somente com grants específicos.

### Contexto/Tenant

Veículo cadastrado para operação de condomínio pertence ao contexto desse Condominium; Vehicle global/múltiplo-tenant não é presumido.

### Gatilho e pré-condições

Cadastro/alteração/encerramento; pessoa e unidade responsáveis identificadas conforme política.

### Permissões necessárias

`vehicle.manage` no scope da Unit/Condominium; leitura `vehicle.read`. Ator e autogerenciamento dependem de grants e vínculo.

### Dados/conceitos envolvidos

`Vehicle`, `Person`, `Unit`, `Visit`/`AccessAuthorization` para temporário, histórico, `AuditLog`.

### Fluxo principal

Validar tenant e responsável; registrar/alterar dados conceituais (placa, marca, modelo, cor, tipo/status conforme campos aprovados); encerrar vínculo preservando histórico. Veículo visitante é relacionado a visita/autorização conforme contexto e não cria residency/ownership.

### Alternativas/validações/transições

Placa duplicada, temporária, inválida, mudança de responsável e estacionamento/vaga devem seguir policy local. Não assumir exclusividade de veículo entre condomínios ou unidades.

### Eventos, notificações, auditoria, pós-condições

Registrar Vehicle created/updated/deactivated e mudança de relacionamento; notificar somente se configurado. Auditoria inclui ator, tenant, recurso e alteração; termina com vínculo atualizado preservando histórico.

### Erros/negações

Sem permission/vínculo; responsável/unit de outro tenant; dado inválido conforme regra aprovada; duplicidade não resolvida; estado incompatível.

### OPEN DECISIONS

OD-10: veículo temporário, placa/duplicidade, campos mínimos, estacionamento/vaga e retenção.

---

## WF-11 — Cadastrar/atualizar/encerrar Pet

### Objetivo

Representar Pet e pessoas responsáveis em contexto de unidade sem impor limite/política universal.

### Participantes

Responsável(is) da Person; administração somente por permission atribuída.

### Contexto/Tenant

Pet e vínculo pertencem ao Condominium/Unit identificado; associação não atravessa tenant automaticamente.

### Gatilho e pré-condições

Necessidade de cadastro, atualizar responsável ou encerrar associação.

### Permissões necessárias

`pet.manage`/`pet.read` e scope Unit/Condominium conforme grant; vínculo à unidade é condição quando definido pela política.

### Dados/conceitos envolvidos

`Pet`, `Person`, `Unit`, histórico, `AuditLog`.

### Fluxo principal

Validar contexto e permission; registrar Pet e uma ou mais pessoas responsáveis somente se configurado; atualizar ou encerrar associação preservando fatos anteriores.

### Fluxos alternativos/validações

Não impor quantidade, espécie, raça, campos obrigatórios ou proibição de circulação sem regra aprovada. Transferência de responsável/saída deve acrescentar histórico, não remover eventos.

### Eventos, notificações, auditoria, pós-condições

Fatos de cadastro/alteração/encerramento; notificação depende de policy. Auditoria registra alterações e ator. Conclui com vínculo consistente ou negação explícita.

### Erros/negações

Tenant ou unit incompatível; permission ausente; dado inválido segundo policy; conflito de responsável não resolvido.

### OPEN DECISIONS

OD-10: cadastro exigido, múltiplos responsáveis, dados, encerramento, visibilidade e políticas locais de convivência.
