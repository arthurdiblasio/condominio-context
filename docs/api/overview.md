# Visão do contrato conceitual da API

## Escopo

O contrato expõe operações do domínio para Admin Web, Mobile Resident, Mobile Doorman e Mobile Syndic. Essas audiências são limites lógicos de uso, não APIs físicas ou permissões próprias. A mesma operação pode servir mais de um cliente; acesso depende de identidade, contexto, grant e finalidade.

Não se definem caminhos/URLs, HTTP verbs/status, nomes de campos, formato de representação, transporte ou mecanismo de autenticação nesta etapa.

## Vocabulário do contrato

| Termo | Significado no contrato |
|---|---|
| **Resource** | Conceito sobre o qual uma pessoa autorizada pode consultar ou executar operações semânticas. Não implica CRUD completo. |
| **Query** | Leitura de uma representação ou coleção, filtrada pelo tenant, scope, finalidade e permissions aplicáveis. |
| **Command** | Solicitação de uma ação de negócio com ator, alvo, contexto, guardas e resultado identificáveis. |
| **Workflow Operation** | Comando ou sequência conceitual que realiza uma etapa de workflow. Seu processamento respeita state machine e não se reduz a atualizar um campo. |
| **Domain Event / Operational Event** | Fato produzido quando a regra do comando é satisfeita. Não é response nem comando de criação arbitrária de evento. |
| **Side effect** | Notificação, auditoria ou efeito decorrente, sujeito a regra; não torna a operação de domínio bem-sucedida ou falha por si só. |

Exemplos são nomes conceituais (`reservation.approve`, `package.receive`), não endpoints, nem uma decisão de naming final.

## Contexto de recursos

- **Platform** contém catálogo/capacidades globais e contexto de administração da plataforma.
- **Condominium** delimita o tenant dos recursos operacionais.
- **Block** e **Unit** pertencem a um único Condominium; **CommonArea** também tem contexto condominial.
- Cada operação identifica o Condominium-alvo quando o recurso é tenant-scoped e confirma a coerência do alvo, referências e scope.
- Não se decide se o contexto será expresso na hierarquia de recursos, em metadado de requisição ou em outro mecanismo. A escolha técnica deve tornar o tenant inequívoco.

## Fronteiras lógicas

| Fronteira | Responsabilidade conceitual | Exemplos de audiência |
|---|---|---|
| Platform Administration | Criar e configurar tenant e catálogo global; habilitações locais apenas por autoridade explicitamente concedida. | Operação SaaS/admin autorizado. |
| Condominium Administration | Manter estrutura, cadastros, vínculos, áreas e atribuições dentro do tenant. | Admin Web, syndic ou property manager com grants. |
| Resident | Consultar e agir sobre próprios dados/unidade conforme vínculo e grants. | Mobile Resident. |
| Portaria / Operations | Registrar eventos observados, visitas e custódia com dados mínimos. | Mobile Doorman/employee com grants. |
| Assembly | Operar agenda, participação, elegibilidade, procuração e voto conforme regras humanas/jurídicas. | Syndic/admin/participantes autorizados. |
| Finance | Gerir informações financeiras apenas com módulo e permissions habilitados. | Usuários financeiros autorizados. |

São separações de responsabilidade e dados, não cinco APIs físicas. `platform_admin` não recebe acesso operacional automático a tenants.

## Recursos e operações

O contrato é command/query-first:

- Query de disponibilidade não reserva recurso; criar reserva não equivale a confirmar.
- `AccessAuthorization` é autorização prevista; comando de registrar ENTRY/EXIT/DENIED relata fato observado separado.
- `PackageEvent` e `AccessEvent` são gerados por operações de negócio/fato; não são livremente editáveis.
- `Vote` é ato sensível condicionado por elegibilidade e janela da pauta.
- `NotificationDelivery` é resultado de tentativa por canal, não comando para provedor específico.
- `AuditLog` é consultável sob escopo; não é editável pela API operacional comum.

Ver [resources](./resources.md), [commands](./commands.md), [queries](./queries.md) e [traceability matrix](./traceability.md).

## Eventos e resposta

O resultado imediato de um comando confirma o resultado conceitual da solicitação: aceita, rejeitada por regra, conflito, ou pendente de decisão quando isso for estado previsto. Eventos produzidos relatam fatos de domínio separados. Não é estabelecido se serão retornados juntos, consumidos de modo assíncrono ou publicados por qualquer mecanismo.

Falha de NotificationDelivery não desfaz reserva, autorização, recebimento, grant ou voto que já tenham ocorrido. Se um comando é negado, não produzir fato de sucesso; tentativa negada sensível pode ser auditada segundo a policy.

## Versionamento e decisões

Contratos públicos devem poder evoluir sem quebrar consumidores existentes. Forma de versionamento e compatibilidade conceitual estão em [versioning.md](./versioning.md). Nenhuma forma técnica (URL, header ou negociação) foi escolhida.

Gaps de domínio são apontados às decisões existentes em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md); não devem ser preenchidos silenciosamente por API.
