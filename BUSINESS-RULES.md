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

O catálogo completo de estados, estados iniciais/terminais, guardas, atores, permissions, scopes, efeitos, notificações, auditoria, repetições, concorrência e correção está em [docs/state-machines/](./docs/state-machines/README.md). Esse catálogo distingue estados do domínio de fatos e explicita quando um estado é apenas candidato/condicional a decisão humana.

Princípios invariáveis:

- `ENTRY`, `EXIT`, `DENIED`, `RECEIVED`, `NOTIFIED`, `CONFIRMED`, `PICKED_UP`, presença e voto são fatos/eventos, não estados por si sós.
- Uma `AccessAuthorization` ativa não prova entrada; `NotificationDelivery(SUBMITTED)` não prova entrega.
- Um estado incompatível, recurso expirado ou assignment suspenso/revogado/expirado não autoriza nova transição.
- Toda operação respeita Permission, RoleAssignment, Scope, tenant e pré-condições do workflow; a existência de um estado não concede authority.
- Correção preserva o fato original, a autoria e o contexto; terminais não são reabertos sem procedimento aprovado.
- Semântica de repetição/concorrência continua sujeita a OD-18 e às decisões específicas do domínio.

Os documentos em `docs/state-machines/` substituem os vocabulários legados abaixo como catálogo de revisão; políticas dependentes de decisão permanecem abertas.

## OPEN DECISIONS classificadas

| Prioridade | Decisão humana | Impacto |
|---|---|---|
| CRITICAL | Regras legais de assembleia: elegibilidade por pauta, quórum, peso, procuração, convocação e validade de resultado. | Pode invalidar votação, apuração e registros do condomínio. |
| CRITICAL | Regras de concessão/revogação de papéis, delegação de autoridade e matriz de permissões. | Determina acesso a dados e operações sensíveis entre escopos e tenants. |
| HIGH | OD-03: associação Person-UserAccount, cardinalidade, convite/expiração, ativação, suspensão/desativação, reativação e capabilities de ciclo de vida. | Afeta identidade, acesso, recuperação e continuidade de relações. |
| HIGH | OD-04: política local de reservas, rejeição/expiração/conclusão, reabertura, bloqueio de área, concorrência, cancelamento e eventual cobrança. | Determina consistência de agenda e expectativas de usuários. |
| HIGH | OD-05: ciclo de visitas/autorização, validade, uso, cancelamento/revogação, encerramento/no-show, exceções e correção de fatos de acesso. | Afeta controle de entrada, segurança e auditoria. |
| HIGH | OD-06: confirmação/retirada de pacote, situação derivada, authority, contestação, correção, sequência e concorrência. | Afeta custódia e responsabilização por encomendas. |
| HIGH | Finalidade, categorias de dados pessoais, visibilidade, retenção e anonimização validadas com responsáveis de privacidade/jurídicos. | Afeta conformidade e exposição de dados entre pessoas do condomínio. |
| MEDIUM | OD-08: configuração de canais, estados de tentativa, expiração/cancelamento, opt-in/opt-out, retry, fallback e semântica de entrega/leitura. | Afeta notificações e expectativas sem determinar canal técnico. |
| MEDIUM | Política de veículos, estacionamento, duplicidade, placas temporárias e vínculo de visitante. | Afeta portaria e associação de veículos a unidades/pessoas. |
| MEDIUM | Política de pets: responsável(is), encerramento, dados e regras locais de convivência. | Afeta cadastro e administração das informações de animais. |
| MEDIUM | OD-11/17: quem habilita/desabilita módulos/features, dependências, grants e efeito sobre operações em curso. | Afeta disponibilidade funcional; dados históricos devem permanecer preservados. |
| MEDIUM | OD-09/18: efetividade temporal de vínculos, repetição, duplicidade de fatos e resolução de operações concorrentes. | Afeta histórico de relações, contagem de fatos e consistência entre workflows. |
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
