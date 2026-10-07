# Eventos, auditoria e efeitos

## Vocabulário

| Conceito | Significado | Exemplo / limite |
|---|---|---|
| Command | Intenção de tentar uma operação. | `package.receive`; pode ser recusada e não gerar fato de sucesso. |
| Domain Event | Fato de domínio já ocorrido como resultado de regra aceita. | `ReservationConfirmed`, `RoleGranted`. Não é comando nem resposta HTTP. |
| Operational Event | Observação factual de processo. | `AccessEvent(ENTRY)`, `PackageEvent(RECEIVED)`; não é estado derivado. |
| Audit Event / AuditLog | Evidência de quem solicitou/executou ação, quando, onde e resultado. | Pode auditar tentativa negada; não inventa sucesso de domínio. |
| Notification | Intenção de comunicar. | Entrega por canal possui seu próprio lifecycle; não é evento de domínio nem prova de leitura. |

Domain events nascem quando a operação de domínio é aceita e representam algo que ocorreu. Application reúne os fatos resultantes, coordena persistência e efeitos. Um nome de evento no catálogo não determina esquema, broker, event sourcing ou emissão garantida.

## Reações a eventos

Handlers pertencem conceitualmente à Application/integrações, fora do aggregate que originou o fato:

- `PackageReceived` pode solicitar criação de Notification conforme policy.
- `ReservationConfirmed` pode iniciar intenção de Notification.
- `RoleGranted` exige Audit; comunicação à pessoa é condicional.
- `AccessEventRecorded` pode acionar auditoria/rotina operacional aprovada.
- `VoteRecorded` e `AssemblyClosed` não produzem apuração/publicação automaticamente enquanto OD-01 estiver aberta.

Handlers devem ser idempotentes para efeitos repetíveis, manter tenant/origem/finalidade e não executar transições que não estejam aprovadas.

## Síncrono vs candidato assíncrono

| Efeito | Classificação conceitual | Regra |
|---|---|---|
| Validar guardas e atualizar o estado/fato principal | `SYNCHRONOUS` | Comando só confirma sucesso após decisão e persistência necessárias. |
| Registrar AuditLog obrigatório | `SYNCHRONOUS` com ação sensível | Não deixar para callback HTTP ou tarefa best-effort. |
| Registrar informação corretiva junto à referência original | `SYNCHRONOUS` | A correção não apaga o original. |
| Criar intenção Notification necessária ao fato | `SYNCHRONOUS` ou persistida atomicamente como trabalho pendente | Não afirmar entrega. |
| Enviar e-mail, WhatsApp ou Push | `ASYNC-CANDIDATE` | Falha não reverte o fato fonte; provider permanece abstrato. |
| Atualizar uma projeção derivada não crítica | `ASYNC-CANDIDATE` | Só se fonte factual preservada e consistência/latência aceitáveis forem aprovadas. |
| Publicar resultado de assembleia | Não habilitado por arquitetura | Depende de autoridade, regra legal e OD-01. |

Não é necessário que todo evento use fila ou processamento assíncrono.

## Auditoria como responsabilidade transversal

Use cases devem gerar os dados de auditoria exigidos por AUDIT-01..10. O registro obrigatório é coordenado em Application e deve pertencer à mesma decisão/commit quando a regra exige que a ação e audit sejam inseparáveis. Múltiplas entradas (HTTP, job, operação assistida) passam pelo mesmo caso de uso ou produzem auditoria equivalente, sem controller como única origem.

Logs técnicos, tracing e eventos de negócio não substituem AuditLog. Operação negada pode exigir audit conforme policy, mas nunca gera evento de sucesso. Leitura de auditoria é autorizada e também pode ser auditada.

## Falhas e repetição

Erro de handler/provider não substitui nem apaga a tentativa anterior; NotificationDelivery registra tentativa/resultados distintos. Handler pode repetir trabalho somente com identidade de operação apropriada e idempotência aprovada. Sem prova de que dois fatos físicos representam o mesmo ato, não deduplicá-los automaticamente (OD-18).
