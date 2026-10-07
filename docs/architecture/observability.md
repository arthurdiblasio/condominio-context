# Observabilidade, privacidade e cache

## Observabilidade técnica

- **Logs técnicos:** falhas, disponibilidade e contexto operacional; evitar tokens, credenciais e dados pessoais desnecessários.
- **Metrics:** contadores, duração, falhas e atraso de processamento; agregar por dimensão sem expor pessoas ou misturar tenants inadvertidamente.
- **Tracing:** acompanhar uma operação entre camadas/serviços; identificador de trace/correlation não é identidade, tenant proof nem autorização.
- **Correlation ID:** correlaciona entrada, use case, persistência e efeitos; deve ser propagado a handlers quando aplicável e não conter dados pessoais.

Não foi escolhido fornecedor nem padrão de instrumentação (por exemplo, OpenTelemetry ou Datadog).

## AuditLog é distinto

`AuditLog` é trilha de negócio governada por AUDIT-01..10, sujeita a permission, scope, finalidade, retenção e correções rastreáveis. Logs técnicos podem ser efêmeros/incompletos e não provam por si quem concedeu papel, registrou entrada ou alterou voto. Audit não pode ser reduzido a log de request nem substituir AccessEvent/PackageEvent/Domain Event.

## Segurança e dados sensíveis

- Isolar contexto de tenant nas consultas, logs, métricas, traces, cache e efeitos externos.
- Minimizar valores pessoais; usar identificadores/referências e redigir dados sensíveis onde detalhes não forem necessários.
- Dados de acesso, votação, vínculos, notificações, evidências e documentos exigem controle e retenção segundo privacy policies pendentes.
- Erros para clientes podem ser minimizados para evitar enumeração cross-tenant; diagnóstico detalhado somente a atores autorizados.
- Falha de observabilidade não deve criar sucesso falso nem dispensar audit obrigatório.

## Cache

Cache é otimização posterior, não fonte de verdade nem mecanismo de authorization. Classificação inicial:

| Dados/decisão | Classificação | Condição |
|---|---|---|
| Conteúdo estático global, como catálogo de capabilities públicas | `CACHEABLE` | Sem dados tenant/pessoais e com invalidação explícita. |
| Configuration de módulo/feature e metadados de condomínio | `CONDITIONAL` | Chave tenant-aware, curta validade/invalidação e revalidação para commands. |
| Permission/RoleAssignment/decisão de autorização | `NON-CACHEABLE` para decisão de mutação; `CONDITIONAL` somente para leitura com versão/validade e invalidação segura | Mudança de grant deve produzir revogação efetiva; cache não amplia authority. |
| Dados pessoais, packages, AccessEvents, NotificationDelivery, AuditLog | `NON-CACHEABLE` por padrão | Qualquer cache requer minimização, finalidade, isolamento e política explícita. |
| Disponibilidade de área | `NON-CACHEABLE` para decisão de confirmação; `CONDITIONAL` para consulta indicativa | Revalidar atomicamente/consistência necessária ao aprovar; resultado exibido não reserva a área. |
| VotingEligibility, Vote, quorum e resultados | `NON-CACHEABLE` para decisão sensível | OD-01 e privacidade determinam visibilidade; não servir dado obsoleto como base de voto/apuração. |

Não adicionar Redis ou outro cache distribuído por antecipação.
