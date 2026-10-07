# Catálogo de papéis

Roles são vocabulário configurável, não permissões incorporadas. Capacidades listadas são típicas e condicionais; nenhuma ação é concedida sem `RoleAssignment`, permission, scope, tenant e condições aplicáveis.

| Role | Propósito e natureza | Escopo natural | Capacidades típicas | Limitações e delegação |
|---|---|---|---|---|
| `platform_admin` | Administração da plataforma; administrativo. | `PLATFORM`. | `module.read`, `module.enable/disable` no escopo autorizado e `platform.condominium.manage`; suporte cross-tenant somente com `platform.cross_tenant.access` explicitamente concedida e finalidade autorizada. | Não é admin operacional de todos os condomínios; não vê dados tenant por padrão. Gestão de seus próprios grants exige governança dedicada. |
| `condominium_admin` | Administração da operação do tenant; administrativo. | `CONDOMINIUM`. | `condominium.read/update`, estrutura, `role_assignment.read`, grants operacionais delegados, configuração module/feature se autorizada. | Não atravessa tenants nem eleva privilégio automaticamente. Assembleia, finanças, dados sensíveis e concessão de papéis privilegiados são condicionais. |
| `property_manager` | Administração operacional por mandato em condomínios específicos. | `CONDOMINIUM`, opcionalmente block/unit. | Operar estrutura, pessoas, reservas, comunicação e fluxos delegados. | Cada condomínio requer assignment próprio; não há acesso a condomínio não atribuído. Não concede grants sem permission delegada. |
| `syndic` | Gestão representativa do condomínio; administrativo/governança. | `CONDOMINIUM`. | `condominium.read/update` conforme policy; `assembly.manage`, reservation approval, staff/relationship management e finance permissions se concedidas. | Ser síndico não implica assembleia válida, acesso integral a finanças/dados, permissão para grants ou capacidade legal irrestrita. |
| `assistant_syndic` | Apoio delegado ao syndic; administrativo/operacional. | Condominium e scopes específicos delegados. | Leitura e execução de tarefas delegadas, como auxiliar workflow, comunicar ou revisar cadastros. | Não herda grants do syndic; capacidades são explicitamente configuradas e podem ser revogadas/temporárias. Concessão de papéis não é padrão. |
| `doorman` | Operação de portaria; operacional. | `CONDOMINIUM`, ponto de operação quando definido. | `visit.register`, `access_authorization.read`, `access_event.register`, `package.register/read`, notificação/retirada apenas conforme grants. | Sem alteração de titularidade/residência, grants/permissões, financeiro ou configuração administrativa por padrão. Campos exibidos limitados à necessidade operacional. |
| `employee` | Categoria funcional de trabalho; operacional. | Condominium, block, unit ou feature conforme função. | Capacidades específicas atribuídas para atividades do funcionário. | Não implica doorman, admin nem acesso a todo o staff. Não pode conceder papéis sem grants expressos. |
| `owner` | Perfil de acesso ligado potencialmente à propriedade; residencial. | `UNIT` ligado a `UnitOwnership` e scope aplicável. | Ler informação autorizada da unidade, gerir ação própria quando configurado e participar de assembleia se autorizado/elegível. | Papel não comprova titularidade; não implica residência, administração, voto legal, financeiro ou permission para alterar titularidade. |
| `resident` | Perfil de acesso residencial; residencial. | `UNIT` ligado a residência reconhecida e scope aplicável. | Ler dados próprios permitidos, reservation.create, package.read e autorização/visita quando configurado. | Não altera outros moradores, proprietários, permissões ou configuração por padrão; não é automaticamente elegível para voto. |
| `tenant` | Perfil de acesso de locação/ocupação; residencial. | `UNIT` ligada a `UnitTenancy` e scope aplicável. | Capacidades residenciais configuradas, como reservation.create e package.read quando permitidas. | Papel não comprova locação; não implica titularidade, administração, acesso financeiro nem voto legal. |

## Conceder, revogar e receber papéis

- Um papel pode coexistir com outros na mesma Person; permissões são avaliadas por grants efetivamente aplicáveis.
- Ser proprietário/residente/locatário não concede automaticamente papel admin nem autoridade para conceder.
- `platform_admin` e `condominium_admin` não podem conceder qualquer papel somente em razão do nome: precisam de `role_assignment.grant` no scope pertinente e limites de autoridade aprovados.
- Quem pode conceder/revogar cada Role, autoatribuir-se, delegar ou administrar roles privilegiados continua aberto.

### Avaliação por papel

| Role | Pode coexistir com outros roles na mesma Person? | Pode conceder/revogar roles por padrão? |
|---|---|---|
| `platform_admin` | Sim, se houver assignments independentes; isso não soma acesso a tenants automaticamente. | Não automaticamente. Somente `role_assignment.*` e escopo explicitamente delegados; administração de platform roles é decisão crítica. |
| `condominium_admin` | Sim, com assignments e escopos independentes. | Não automaticamente. Pode receber grant administrativo local se aprovado; não pode conceder além de sua autoridade. |
| `property_manager` | Sim; assignments próprios para cada condomínio. | Não por padrão; delegação administrativa específica pode ser configurada e revogada. |
| `syndic` | Sim; função de síndico não substitui outros vínculos/roles. | Não por padrão; eventual gestão de roles depende de grant, convenção e limites aprovados. |
| `assistant_syndic` | Sim; apoio não significa herança do syndic. | Não por padrão; somente delegação explícita e delimitada se aprovada. |
| `doorman` | Sim, se pessoa possuir outras funções separadamente autorizadas. | Não por padrão. |
| `employee` | Sim; cada função operacional requer scope/grants pertinentes. | Não por padrão; função de staff não é autoridade administrativa. |
| `owner` | Sim; role não cria/prova UnitOwnership nem exclui residência/tenant roles. | Não por padrão. |
| `resident` | Sim; role não cria UnitResidency nem exclui owner/tenant roles. | Não por padrão. |
| `tenant` | Sim; role não cria/prova UnitTenancy nem exclui residence role. | Não por padrão. |

“Pode coexistir” significa apenas que o modelo não proíbe atribuições múltiplas; não significa combinação permitida em toda circunstância. Conflitos de interesse ou incompatibilidade entre roles continuam sujeitos a política.

## Valores da matriz

`ALLOW` aqui significa capability diretamente esperada em um perfil, sujeita ainda a identidade, scope, tenant e negócio; `CONDITIONAL` depende de vínculo, regra ou elegibilidade; `CONFIGURABLE` deve ser concedida explicitamente por política local; `DENY` significa ausência intencional de capacidade padrão, não proibição de delegação futura.
