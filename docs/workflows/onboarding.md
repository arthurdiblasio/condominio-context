# Onboarding de condomínio

## WF-01 — Criar e preparar tenant

### Objetivo

Incorporar um Condominium à Platform, estabelecer contexto isolado e prepará-lo para administração sem conceder acesso por mera criação.

### Participantes

- Iniciador candidato: ator de operação SaaS ou representante autorizado do condomínio.
- Executor: `platform_admin` somente com permission de plataforma apropriada; autoridade final de iniciar/aprovar o onboarding é `OPEN DECISION`.
- Primeiro administrador: Person designada com `RoleAssignment` tenant-scoped; escolha, verificação e aceite são `OPEN DECISION`.

### Contexto/Tenant

Antes da existência do Condominium, contexto Platform; após criação, operações de configuração e cadastro ficam limitadas ao novo Condominium. Nenhum dado de outro tenant deve ser copiado ou exposto implicitamente.

### Gatilho e pré-condições

- Solicitação de inclusão de condomínio.
- Dados mínimos de identificação e contato a definir pelo produto, sem coletar dados não necessários.
- Autorização do solicitante para pedir a inclusão.
- Nenhum tenant alvo existente pode ser alterado por resolução ambígua da solicitação.

### Permissões necessárias

- `platform.condominium.manage` no `PLATFORM` para criação/administração global se atribuído ao executor.
- Após o tenant existir: `condominium.update`, `module.enable`, `feature.enable`, `role_assignment.grant` ou outras permissions do catálogo somente quando necessárias e concedidas no scope correspondente.
- A matriz atual é proposta; o primeiro grant administrativo não está aprovado.

### Dados/conceitos envolvidos

`Platform`, `Condominium`, `Person`, `UserAccount`, `RoleAssignment`, `Module`, `Feature`, `CondominiumModule`, `CondominiumFeature`, `Block`, `Unit`, `CommonArea`, `AuditLog`.

### Fluxo principal

1. Ator autorizado solicita a criação e informa os dados mínimos aprovados.
2. Valida-se permissão na Platform e a não ambiguidade do alvo; dados de tenants existentes não são reutilizados sem base autorizada.
3. Cria-se o `Condominium` em estado operacional inicial definido pelo produto. Não se presume que criação o torne operacional.
4. Define-se configuração inicial do tenant; módulos/features somente são habilitados por capability e autoridade concedidas.
5. Registra-se estrutura básica conforme necessário (ver WF-02/WF-03).
6. Identifica-se Person designada como administrador inicial; associa UserAccount somente pelo workflow de identidade.
7. Cria-se `RoleAssignment` limitado ao novo tenant, conforme regra aprovada para primeiro administrador.
8. Revê-se se estrutura e grants necessários estão consistentes antes de declarar onboarding operacional.

### Fluxos alternativos

- Onboarding incompleto: preservar o tenant em estado não operacional/incompleto se esse estado for aprovado; impedir uso operacional até completar condições obrigatórias.
- Condomínio existente ou solicitação duplicada: não criar/alterar sem resolução explícita de identidade do tenant.
- Módulo não habilitado: continuar apenas com capacidades disponíveis; não provisionar grants relacionados.
- Primeiro administrador não verificável: interromper concessão de acesso e encaminhar para decisão/validação humana.

### Validações e transições

- Criador tem permission `platform.condominium.manage` e scope `PLATFORM`.
- Novos registros pertencem ao tenant recém-criado.
- `RoleAssignment` inicial aponta para Person, role e scope coerentes, sem autoridade superior à do concedente.
- Habilitação de feature/module não concede permission a usuários.
- Estado “operacional” só pode ser declarado após critérios objetivos definidos por decisão humana.

### Eventos/fatos registrados

Solicitação de onboarding, criação de Condominium, alterações de configuração, habilitações, cadastro de estrutura e grant inicial, quando ocorrerem. São fatos separados e com ator/instante/contexto conhecidos; o `AuditLog` registra ações sensíveis correspondentes.

### Notificações

Notificação de convite/ativação do administrador pode ocorrer se canal e política estiverem habilitados. Falha de entrega não autoriza acesso nem invalida a criação do tenant.

### Auditoria

Auditar solicitante, decisão de criação, Condominium alvo, configuração, módulos/features habilitados, criação e concessão do primeiro papel, resultado e contexto Platform/tenant.

### Pós-condições e término

Tenant identificado e isolado; estado operacional/incompleto explícito segundo critérios ainda a aprovar; nenhum administrador recebe acesso sem conta/assignment/permission aplicáveis.

### Erros/negações

Sem permission Platform; dados mínimos ausentes; solicitação duplicada/ambígua; recurso inexistente; tenant incorreto; grant acima da autoridade; feature não habilitada; identidade do primeiro administrador não verificada. Negar ou interromper sem revelar dados de outro tenant.

### OPEN DECISIONS

Autoridade de criação, dados mínimos, estado e critérios de prontidão, validação do condomínio, seleção do primeiro administrador, assignment bootstrap, módulos default e política de convite/ativação.
