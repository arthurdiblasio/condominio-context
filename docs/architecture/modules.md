# Módulos e fronteiras

## Estilo proposto

Organizar o backend como monólito modular inicialmente. Um módulo corresponde a responsabilidade de domínio e ciclo de mudança, não a um serviço implantável, schema ou entidade. A separação não antecipa microsserviços.

| Módulo | Responsabilidade principal | Dependências conceituais / limites |
|---|---|---|
| Identity | `Person`, associação conceitual a `UserAccount` e lifecycle correspondente. | Não concede papéis nem implementa autenticação técnica; cardinalidade/autoridade dependem de OD-03. |
| Condominium | `Platform`, tenant `Condominium`, configuração e contexto estrutural do condomínio. | Raiz de tenant; não absorve os dados operacionais de todos os módulos. |
| Units | `Block`, `Unit`, áreas e vínculos tipados de pessoa/unidade, conforme boundaries existentes. | Requer coerência de tenant; identidade referenciada, não copiada como agregado. |
| Permissions / Authorization | Roles, RoleAssignments, Permission, Scope e decisão contextual de acesso. | Integrado por Application; matriz e composição final dependem de OD-02/14/15/16. |
| Reservations | CommonArea, disponibilidade e lifecycle de Reservation/Event conforme política. | Contextos de Person/Unit/tenant referenciados; sem acesso físico implícito. |
| Access | Visit, AccessAuthorization e fatos AccessEvent. | Pessoa, Unit e referências contextuais; não infere ocorrência a partir da autorização. |
| Packages | Package e histórico PackageEvent/custódia. | Notificação desacoplada; não sobrescreve fatos originais. |
| Assemblies | Assembly, AgendaItem, Participant, VotingEligibility, Proxy, Vote e Quorum. | Regras jurídicas e fronteiras de consistência pendentes em OD-01/02; escrita deliberativa condicionada. |
| Notifications | Intenção Notification e ciclos independentes NotificationDelivery. | Adaptadores de canal fora do domínio; eventos de origem são referências, não dependência inversa. |
| Audit | Trilha de ações sensíveis, resultado e correções. | Referencia recurso/tenant sem ser substituto de evento de negócio nem autoridade central sobre todos os agregados. |
| Finance (optional) | Registro e relatórios financeiros conceituais limitados. | Módulo opcional; OD-12. Não incluir em MVP sem decisão de produto. |

`Vehicle` e `Pet` podem ser subdomínios/módulos quando a necessidade justificar; sua existência no catálogo não obriga fronteira executável independente.

## Coordenação sem ciclos

- Application coordena comandos que atravessam módulos, validando todos os contextos antes da transação.
- Um módulo mantém suas invariantes; outro não muta seus objetos internos diretamente.
- Referências a entidades de outro módulo são identificadores/conceitos explícitos, sempre acompanhados pela validação de tenant e finalidade.
- Notificações e auditoria recebem fatos/contexto por uma fronteira explícita; não são chamados por cada entidade de domínio.
- Se uma regra exigir invariantes síncronas entre módulos, documentar a necessidade de consistência antes de separar deployment ou armazenamento.

## Escopo MVP e posterior

O contrato atual classifica funcionalidades como propostas, não roadmap aprovado. Candidato a MVP arquitetural: estrutura de Condominium/Block/Unit, identity e vínculos, role assignments conforme capabilities aprovadas, áreas/reservas, visita/acesso, pacotes, notificações e auditoria essencial.

Assembly e os módulos de identidade/autorização podem ter conceitos presentes desde o início, mas operações ficam limitadas ao que tiver regra e authority aprovadas. Voto/apuração dependem de OD-01. `Vehicle`/`Pet` e Finance podem ficar post-MVP/opcionais; Finance não faz parte do MVP proposto. Nenhuma implantação funcional é autorizada por esta classificação.
