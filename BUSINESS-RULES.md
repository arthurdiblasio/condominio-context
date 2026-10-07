# BUSINESS-RULES.md

## Objetivo

Índice consolidado das regras de negócio do Condomínio. As regras são conceituais, verificáveis e independentes de tecnologia. Regras condicionadas a escolha de produto, política local ou legislação não são tratadas como padrão universal.

## Classificação

- `INVARIANT`: condição que deve permanecer verdadeira.
- `VALIDATION`: condição para aceitar ou rejeitar uma operação.
- `AUTHORIZATION`: quem pode agir, limitado por papel, permissão e escopo.
- `LIFECYCLE`: fatos, estados e transições de um conceito.
- `CONSISTENCY`: coerência entre conceitos relacionados.
- `TENANCY`: separação entre plataforma e condomínios.
- `PRIVACY`: finalidade, minimização, visibilidade e retenção.
- `OPERATIONAL`: procedimento sujeito à configuração operacional.
- `CONFIGURATION`: regra configurável sem generalização universal.

As regras definitivas estão detalhadas em [docs/business-rules/](./docs/business-rules/). `OPEN DECISION` significa que o domínio não tem dados suficientes para escolher entre alternativas.

## Regras globais

1. `TENANCY`: todo registro ou relação operacional deve preservar seu condomínio de contexto; ações cross-tenant exigem autorização explícita de plataforma e finalidade definida.
2. `INVARIANT`: uma relação de pessoa, unidade, papel ou evento em um condomínio não concede acesso ou vínculo automático em outro.
3. `INVARIANT`: `Person`, `UserAccount`, `Role`, `RoleAssignment` e `Permission` são conceitos distintos.
4. `INVARIANT`: propriedade, residência, locação, conta de acesso e papel administrativo não são equivalentes.
5. `AUTHORIZATION`: ações restritas exigem `Permission` concedida pelo papel em `RoleAssignment` válido no escopo e condomínio da ação.
6. `INVARIANT`: autorização não prova ocorrência física; intenção de comunicação não prova entrega; estado atual não substitui histórico factual.
7. `PRIVACY`: acesso a dados pessoais deve ser necessário para a finalidade e limitado ao escopo autorizado; prazos de retenção dependem de decisão validada.

## Regras por domínio

| Domínio | Regras detalhadas |
|---|---|
| Multi-tenancy | [tenancy.md](./docs/business-rules/tenancy.md) |
| Identidade e conta | [identity.md](./docs/business-rules/identity.md) |
| Unidades, veículos e pets | [units.md](./docs/business-rules/units.md) |
| Papéis, permissões e escopos | [permissions.md](./docs/business-rules/permissions.md) |
| Visitas e controle de acesso | [access.md](./docs/business-rules/access.md) |
| Áreas comuns e reservas | [reservations.md](./docs/business-rules/reservations.md) |
| Pacotes e confirmação | [packages.md](./docs/business-rules/packages.md) |
| Notificações | [notifications.md](./docs/business-rules/notifications.md) |
| Assembleias e votação | [assemblies.md](./docs/business-rules/assemblies.md) |
| Módulos e features | [modules.md](./docs/business-rules/modules.md) |
| Auditoria | [audit.md](./docs/business-rules/audit.md) |
| Privacidade | [privacy.md](./docs/business-rules/privacy.md) |

## Regras globais, configuráveis, legais e operacionais

### Globais

- Isolamento tenant-scoped e proibição de concessão implícita cross-tenant.
- Separação semântica entre identidade, conta, vínculo, papel, permissão, autorização e fato operacional.
- Encerramento de uma relação não apaga o fato histórico de que ela existiu.
- Registros de evento não devem ser apresentados como prova de evento diferente.

### Configuráveis por condomínio

Sujeitas a decisão e governança explícitas: política de reservas (incluindo `AUTO_APPROVED`, `MANUAL_APPROVAL` ou `RULE_BASED`), horários, antecedência, duração, capacidade, cancelamento, regras de área, procedimentos de portaria e canais de comunicação habilitados. Configuração local não pode enfraquecer as invariantes globais nem criar acesso a outro tenant.

### Dependentes de legislação ou instrumentos do condomínio

Elegibilidade, peso e quórum de voto; procurações; convocação e validade de deliberações; requisitos de documentos; privacidade e retenção legalmente exigidas; regras de titularidade e ocupação. Exigem validação competente antes de virarem regra, não aconselhamento jurídico.

### Operacionais

Conferência de encomenda, registro de entrada/saída, correção de ocorrência, aprovação local e tratamento de falha de comunicação. Procedimentos podem variar, mas devem preservar autorização, tenant, rastreabilidade e privacidade.

## Invariantes

Estas invariantes são aplicáveis sem decidir políticas locais:

1. `TENANCY`: toda operação tenant-scoped identifica um único condomínio-alvo.
2. `TENANCY`: dados operacionais de condomínios distintos não são combinados em uma mesma operação sem autorização cross-tenant explícita.
3. `TENANCY`: `RoleAssignment` de um condomínio não concede papel em outro.
4. `TENANCY`: todo vínculo com unidade referencia unidade e condomínio coerentes.
5. `INVARIANT`: uma `Person` pode existir sem `UserAccount`.
6. `INVARIANT`: identidade de `Person` não é criada ou removida implicitamente ao ativar/desativar uma conta.
7. `INVARIANT`: `Role` não concede acesso sem `RoleAssignment` válido e `Permission` correspondente.
8. `AUTHORIZATION`: concessor não pode conceder autoridade superior à que possui, salvo delegação explícita e válida.
9. `INVARIANT`: `UnitOwnership` não implica `UnitResidency`, `UnitTenancy` ou papel.
10. `INVARIANT`: `UnitResidency` não implica propriedade nem locação.
11. `INVARIANT`: encerrar vínculo não apaga seu histórico.
12. `INVARIANT`: encerramento de vínculo de unidade não apaga histórico de acesso, reserva, pacote ou assembleia.
13. `INVARIANT`: toda unidade pertence a exatamente um condomínio.
14. `INVARIANT`: toda `AccessAuthorization` tem período e escopo identificáveis; depois do fim do período não autoriza novo acesso.
15. `INVARIANT`: `AccessAuthorization` não é evidência de `ENTRY`.
16. `INVARIANT`: tentativa `DENIED` pode existir sem autorização válida e não deve ser registrada como entrada.
17. `INVARIANT`: `EXIT` não deve ser fabricado como se uma `ENTRY` tivesse sido registrada; divergência é ocorrência operacional.
18. `INVARIANT`: cada `PackageEvent` permanece no histórico mesmo quando uma informação atual é corrigida.
19. `INVARIANT`: `CONFIRMED` e `PICKED_UP` são fatos distintos; confirmação não prova necessariamente retirada física.
20. `INVARIANT`: `Notification` não prova entrega; cada tentativa/result `NotificationDelivery` pertence a uma notificação e canal.
21. `INVARIANT`: presença em assembleia não implica elegibilidade para votar.
22. `INVARIANT`: elegibilidade não implica que voto foi realizado; `Vote` exige pauta e contexto de elegibilidade permitido.
23. `INVARIANT`: uma `Proxy` não se aplica fora do escopo e período a que foi validada.
24. `INVARIANT`: desabilitar módulo/feature não apaga automaticamente dados ou histórico existentes.
25. `INVARIANT`: dados pessoais não ficam visíveis a todos os moradores por mera participação no mesmo condomínio.
26. `VALIDATION`: quando a política de uma área proíbe conflito de horários, duas reservas confirmadas incompatíveis não podem ocupar o mesmo recurso e período.
27. `VALIDATION`: a confirmação de reserva requer disponibilidade e todas as aprovações que a política local exigir.
28. `INVARIANT`: uma tentativa de operação não pode alterar recurso de outro condomínio por resolução ambígua de contexto.

## State machines e estados

Os documentos de domínio contêm vocabulários propostos; somente invariantes factuais acima são definitivas. Transições que dependem de política estão explicitamente abertas.

| Conceito | Ciclo/estados candidatos | Regra verificável e limite |
|---|---|---|
| UserAccount | invited, active, suspended, deactivated (proposta) | A conta só pode ser usada quando ativa. A associação Person-conta, quem ativa/suspende e efeitos sobre vínculos/papéis requerem decisão humana. |
| RoleAssignment | pending, active, suspended, revoked, expired (proposta) | Deve indicar pessoa, papel, escopo e período/validade aplicável. Revogação/suspensão não apaga ações históricas; concessor deve ter autoridade. Estados e suspensão ainda requerem aprovação. |
| Reservation | draft, pending, confirmed, cancelled, completed, expired (vocabulário existente) | Não confirmar se indisponível ou se aprovação requerida falta. Transições, cancelamento após confirmação, repetição e efeitos financeiros dependem de política. |
| AccessAuthorization | draft, active, expired, revoked, cancelled (proposta revisada) | Autorização ativa só permite tentativa dentro do período, escopo e condições aprovados. `ENTRY`, `EXIT`, `DENIED` são `AccessEvent`, não estados de autorização. |
| Package | estado atual derivado de eventos (estado fechado não definido) | `RECEIVED` deve identificar ocorrência de recebimento; mudanças não apagam `PackageEvent`. Sequência de NOTIFIED/CONFIRMED/PICKED_UP e qual evento define situação atual precisam de decisão. |
| NotificationDelivery | created/attempted, submitted, delivered, read, failed (resultados possíveis) | Só registrar resultados sustentados pela evidência do canal; retry é nova tentativa e não apaga falha prévia. Vocabulário e política de retry/fallback são abertos. |
| Assembly | planned, open, closed, cancelled (candidatos) | Votação não pode ser apresentada como aberta antes da condição de abertura definida; fechamento preserva registros. Convocação, quórum e efeitos do fechamento dependem de validação. |
| Vote | pending, open, closed, cancelled (vocabulário existente) | Registrar voto apenas no período de votação definido e para elegibilidade aprovada na pauta. Correção, anulação, reabertura, sigilo e contagem precisam de decisão. |

Uma transição não listada acima não é implicitamente permitida. A definição de transições completas, atores e efeitos é necessária antes de implementar cada workflow.

### Transições conceituais para revisão humana

Os ciclos a seguir formalizam guardas e limites já identificáveis, mas estados indicados como candidatos e política local/jurídica ainda requerem aprovação. As ações listadas só podem ser executadas por papel/permissão no escopo correto.

| Conceito | Transição permitida conceitualmente | Guarda / efeito | Transição proibida sem regra explícita |
|---|---|---|---|
| UserAccount | invited → active | associação válida com Person e requisitos de ativação aprovados; habilita acesso autenticado. | Conta sem Person associada tornar-se ativa. |
| UserAccount | active → suspended / deactivated | autoridade definida pela política; impede novas ações autenticadas, preservando Person, vínculos e trilha. | Conta suspensa/desativada executar ação autenticada; apagar vínculos por desativação. |
| UserAccount | suspended → active | somente após remoção válida da suspensão e autorização competente. | Reativar automaticamente ou restaurar papéis expirados/revogados. |
| RoleAssignment | pending → active | concedente tem autoridade e o papel/escopo/validade são válidos; concede somente permissões aplicáveis. | Ativar escopo inválido ou autoridade superior à do concedente sem delegação. |
| RoleAssignment | active → suspended / revoked / expired | suspensão/revogação por autoridade; expiração ao fim da vigência aprovada; preserva histórico. | Usar atribuição suspensa, revogada ou expirada; apagar trilha da mudança. |
| Reservation | draft → pending | validação estrutural; fica aguardando regras/aprovação requeridas. | Tratar pending como confirmação ou bloqueio definitivo sem política. |
| Reservation | draft/pending → confirmed | disponibilidade validada e aprovações/configurações obrigatórias satisfeitas; confirma período para o recurso. | Confirmar conflito proibido ou aprovação obrigatória ausente. |
| Reservation | active state → cancelled / completed / expired | cancelamento autorizado; completion/expiration conforme período e política; preserva histórico. | Reabrir, concluir antes do período ou ressuscitar cancelled/expired sem regra explícita. |
| AccessAuthorization | draft → active | pessoa/critério, escopo, período e requisitos aprovados; habilita tentativa, não registra entrada. | Ativar sem período ou escopo identificável. |
| AccessAuthorization | active → expired / revoked / cancelled | validade termina ou autoridade cancela/revoga; bloqueia novas utilizações conforme alcance aprovado. | Tratar uma ENTRY como transição automática para estado `used` universal. |
| Package | registrar RECEIVED | registro de recebimento físico com ator/instante quando conhecidos. | Registrar recebimento sem fato correspondente ou apagar ocorrência depois. |
| Package | acrescentar NOTIFIED / CONFIRMED / PICKED_UP / CANCELLED | cada evento referencia o pacote e registra ator/instante conforme conhecido. NOTIFIED registra comunicação, não entrega. | Ordem entre confirmação, retirada e cancelamento, além dos fatos mínimos acima, não está aprovada; não impor sequência como regra universal. Corrigir contradições por trilha auditável. |
| NotificationDelivery | created/attempted → submitted / delivered / read / failed | registrar apenas resultado evidenciado pelo canal. | Marcar entregue/lida sem evidência; apagar tentativa falha ao repetir. |
| NotificationDelivery | failed → nova tentativa | retry cria outro registro vinculado à mesma Notification. | Sobrescrever falha anterior ou assumir fallback automático. |
| Assembly | planned → open → closed | abertura/fechamento conforme convocação e procedimento aprovados; fechamento preserva pauta e registros. | Abrir sem condições definidas ou reabrir após closed sem política. |
| Assembly | planned/open → cancelled | cancelamento por autoridade/procedimento definido; preserva fatos anteriores. | Cancelar e apagar presença/votos/fatos já registrados. |
| Vote | pending → open → closed | período e procedimento da pauta aprovados; abrir requer pauta habilitada. Fechar impede novos votos naquele ciclo. | Registrar voto fora de open ou após closed. |
| Vote | pending/open → cancelled | somente segundo autoridade e procedimento definidos; mantém auditoria. | Reabrir, corrigir ou anular voto sem procedimento validado e trilha. |

`PackageEvent` e `AccessEvent` são fatos, não estados. A sequência indicada não fecha política jurídica ou operacional de votação, custódia, autorização ou entrega.

## OPEN DECISIONS classificadas

| Prioridade | Decisão humana | Impacto |
|---|---|---|
| CRITICAL | Regras legais de assembleia: elegibilidade por pauta, quórum, peso, procuração, convocação e validade de resultado. | Pode invalidar votação, apuração e registros do condomínio. |
| CRITICAL | Regras de concessão/revogação de papéis, delegação de autoridade e matriz de permissões. | Determina acesso a dados e operações sensíveis entre escopos e tenants. |
| HIGH | Política de associação Person-UserAccount, cardinalidade, convite, ativação, suspensão, desativação e efeitos sobre vínculos. | Afeta identidade, acesso, recuperação e continuidade de relações. |
| HIGH | Política local de reservas: aprovação, disponibilidade, duração, conflito, cancelamento, bloqueio e eventual cobrança. | Determina consistência de agenda e expectativas de usuários. |
| HIGH | Regras operacionais de visita e autorização: quem pode emitir/revogar, verificação, exceções e fatos de acesso. | Afeta controle de entrada, segurança e auditoria. |
| HIGH | Confirmação/retirada de pacote, autoridade de representantes, contestação, correção e evidência. | Afeta custódia e responsabilização por encomendas. |
| HIGH | Finalidade, categorias de dados pessoais, visibilidade, retenção e anonimização validadas com responsáveis de privacidade/jurídicos. | Afeta conformidade e exposição de dados entre pessoas do condomínio. |
| MEDIUM | Configuração de canais, opt-in/opt-out, retry, fallback, prioridade e semântica de entrega/leitura. | Afeta notificações e expectativas sem determinar canal técnico. |
| MEDIUM | Política de veículos, estacionamento, duplicidade, placas temporárias e vínculo de visitante. | Afeta portaria e associação de veículos a unidades/pessoas. |
| MEDIUM | Política de pets: responsável(is), encerramento, dados e regras locais de convivência. | Afeta cadastro e administração das informações de animais. |
| MEDIUM | Quem pode habilitar/desabilitar módulos/features, dependências e procedimento de desativação. | Afeta disponibilidade funcional; dados históricos devem permanecer preservados. |
| LOW | Se bloco é obrigatório para todos os tipos de unidade e quais formas de estacionamento pertencem ao condomínio/unidade. | Afeta variações da estrutura física, sem alterar isolamento ou identidade. |

## Não implementar ainda

- Regras jurídicas de voto, quórum, inadimplência ou representação sem validação.
- Cardinalidade de conta-pessoa e política de autenticação não decididas.
- Aprovação, capacidade, duração, precedência e cancelamento de reserva sem configuração aprovada.
- Políticas definitivas de visitantes, veículos, pets, evidências e retenção.
- Provedor/canal específico, QR/token, hardware, mecanismo de retry ou integração física.
- ERP, contabilidade oficial, cobrança SaaS ou plano comercial.
- Remoção de histórico ao desativar módulo, conta ou vínculo.

## Documentos detalhados

- [OPEN-DECISIONS.md](./OPEN-DECISIONS.md)
- [DOMAIN.md](./DOMAIN.md)
- [PERMISSIONS.md](./PERMISSIONS.md)
- [WORKFLOWS.md](./WORKFLOWS.md)

## Status

`BUSINESS_RULES_REQUIRES_HUMAN_REVIEW`
