# Consultas, filtros, ordenação e paginação

Consultas são conceituais; não definem query string, projeção técnica, URL ou forma da resposta. Cada resultado é filtrado por tenant, Scope, Permission, finalidade e visibilidade de campos. Identificadores ou filtros não substituem autorização.

## Consultas principais

| Consulta | Conceitos / permission | Filtros de domínio candidatos | Ordenação / paginação |
|---|---|---|---|
| Condomínios administráveis | Condominium; `condominium.read` ou grant Platform apropriado | contexto/status administrativo aprovados | Paginável se muitos; tenant/platform conforme finalidade. |
| Blocks e Units por condomínio | Block/Unit; `block.read`, `unit.read` | condomínio, block, estado estrutural | Ordenação determinística por referência estável; coleções pagináveis. |
| Persons relacionadas a unidade | Person + vínculo tipado; `person.read` + `unit_ownership.read`, `unit_residency.read` ou `unit_tenancy.read` conforme finalidade | unit, relação, vigência, pessoa; campos minimizados | Paginável; não pesquisar globalmente sem purpose/grant. |
| Vínculos de uma Person/Unit | UnitOwnership/Residency/Tenancy; permissions correspondentes `.read` | pessoa, unidade, tipo, período atual/histórico | Ordenar por período/identificador determinístico; histórico paginável. |
| Assignments e histórico | RoleAssignment; `role_assignment.read` | pessoa, Role, scope, estado, validade, actor se permitido | Paginável; não mostrar roles/grants fora do scope. |
| CommonArea e disponibilidade | CommonArea/Reservation; `common_area.read`, `reservation.read` | condomínio, datas/período, status/availability | Ordenação estável por recurso e período; disponibilidade consulta não reserva. |
| Reservations por período | Reservation; `reservation.read` | período, unit, block, person, area, status, requester | Ordenar deterministicamente por período/recurso; paginável. |
| Events/visits e convidados | Event/Visit/Person; `event.read`, `visit.read`, `person.read` | data, unit, person, area, estado conforme permitido | Paginar listas extensas; fields de identidade mínimos. |
| Authorizations ativas/por contexto | AccessAuthorization; `access_authorization.read` | pessoa, Unit, período, Visit/Event/Reservation, status | Ordenar por validade e recurso; paginar quando necessário. |
| AccessEvents por período | AccessEvent; `access_event.read` | ENTRY/EXIT/DENIED, intervalo temporal, Unit/Person, actor, authorization, ponto se definido | Ordenar por instante + critério estável de desempate; paginável. Não deduzir eventos faltantes. |
| Packages aguardando retirada e histórico | Package; `package.read`, `package.history.read` | Unit, destinatário, situação derivada, evento e período | Ordenação temporal estável; paginar; só exibir projection de status se aprovada. |
| Notification/Delivery | Notification; `notification.read`, `notification.delivery.read` | destinatário, origem, canal, resultado, período | Ordenação determinística por criação/tentativa; paginar; sem dados cross-tenant. |
| Assemblies, agendas e Participants | Assembly/AgendaItem/Participant; `assembly.read`, `participant.read` | período, estado, agenda, pessoa, presença conforme privacy | Paginável; estado e presença distintos. |
| Eligibility, Votes, quorum/result | VotingEligibility/Vote/Quorum; `voting_eligibility.read`, `vote.result.read`, `quorum.read`, `assembly.result.read` | Assembly, AgendaItem, status/resultado, filtros minimizados permitidos | Restrições especiais de sigilo e tenant; paginação sem quebrar regra de anonimização/visibilidade. |
| Finance entries/suppliers/reports | Income/Expense/Supplier/Report; `finance.read`, `finance.report.read` | categoria, período, tipo, fornecedor, status segundo política | Paginável; saída/export exige `finance.document.export`. Optional module. |
| AuditLog | AuditLog; `audit_log.read` | período, resource, actor, action, resultado, tenant/scope | Ordenação determinística por timestamp e chave; paginável; própria leitura sujeita a audit/purpose. |

## Filtros

Filtros de domínio que podem ser úteis: `status`, intervalo de datas, `unit`, `block`, `person`, `common area`, `actor`, tipo de fato/evento, categoria/ação e vínculo. Disponibilidade só pode ser consultada para CommonArea e período identificados. Filtro omitido não autoriza ampliação de tenant/scope; valores inválidos/incompatíveis geram erro conceitual, não fallback global.

## Paginação

Coleções potencialmente grandes — people, units, packages, AccessEvents, notifications/deliveries, audit logs, assemblies, financial entries e históricos — exigem paginação conceitual e limites de exposição. Não se escolhe offset, cursor, keyset, tamanho máximo ou token de continuação.

## Ordenação

Packages, AccessEvents, NotificationDeliveries, AuditLog e Reservations precisam de ordenação determinística e estável para que páginas não omitam nem repitam itens na mesma visão. Critério principal deve refletir tempo/período do domínio e ter desempate estável; estratégia técnica, direção padrão e timezone permanecem abertas.

Consultas são somente leitura: não executam approve/cancel/release/vote, não reservam disponibilidade e não alteram lifecycle por leitura.
