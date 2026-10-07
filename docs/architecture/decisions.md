# Decisões e questões de arquitetura

Este registro cobre propostas arquiteturais/técnicas desta etapa. Decisões de produto, domínio, governança ou validação jurídica permanecem em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md); nenhuma foi alterada por este documento.

## Propostas para revisão

| ID | Classe | Proposta | Estado |
|---|---|---|---|
| ARCH-01 | ARCHITECTURAL | Iniciar como monólito modular com boundaries de domínio e dependência invertida. | Proposta; validar crescimento, equipe e modelo de implantação. |
| ARCH-02 | ARCHITECTURAL | Commands e Queries separados semanticamente, sem CQRS completo no MVP. | Proposta; revisar se leitura/escala divergir materialmente. |
| ARCH-03 | ARCHITECTURAL | Application coordena casos de uso, autorização contextual e fronteiras transacionais; Domain permanece independente. | Proposta; respeita regras atuais. |
| ARCH-04 | ARCHITECTURAL | Audit obrigatório origina-se nos casos de uso e não apenas na camada HTTP. | Requisito arquitetural decorrente das regras de auditoria; mecanismo de persistência aberto. |
| ARCH-05 | ARCHITECTURAL | Transactional Outbox somente para efeitos assíncronos cuja publicação durável for necessária; incluir no MVP condicional à necessidade de NotificationDelivery confiável. | Proposta condicional. |
| ARCH-06 | ARCHITECTURAL | Contratos de persistência/serviços são específicos às necessidades; não adotar GenericRepository ou Unit of Work universal sem necessidade. | Proposta; revisar com implementação dos casos de uso. |
| TECH-01 | TECHNICAL | Go, Gin, GORM e PostgreSQL são stack de referência informada para implementação futura. | Entrada do Prompt 08; validar e detalhar em projeto de aplicação. |

## Questões técnicas abertas

- Estratégia e formato de ID (UUID, versão ordenável, ULID ou outra).
- Unidade concreta de transação e necessidade de Unit of Work após mapear ports e aggregates.
- Persistência/entrega durável e eventuais garantias/ordenação do outbox.
- Provedor de autenticação, gestão de sessão/credencial e composição do principal.
- Estratégia de isolamento no adaptador de banco; não assumir RLS ou isolamento por schema.
- Paginação e limites de consulta.
- Invalidação/validade de qualquer cache.
- Tecnologia de scheduler, filas, storage e observabilidade, se necessárias.
- Retenção de logs técnicos e política de redaction, distinta da política legal de AuditLog.

## Revisão crítica

A [revisão crítica da arquitetura](../../ARCHITECTURE-CRITICAL-REVIEW.md) permanece registro histórico com assessment `NEEDS REVISION` e status `ARCHITECTURE_REQUIRES_REVISION`; o resultado desta etapa está em [ARCHITECTURE-REMEDIATION.md](../../ARCHITECTURE-REMEDIATION.md). As propostas ARCH-01..06 continuam propostas, não ADRs aprovados. TECH-01 é stack informada, não alternativa comparada.

Novas propostas para avaliação, não decisões aprovadas:

| ID candidato | Tema | Questão a fechar |
|---|---|---|
| ARCH-07 | Tenant-scoped persistence contract | Requisitos obrigatórios estão descritos; selecionar forma concreta de ports/defesa, classificar todos os recursos e validar com testes. |
| ARCH-08 | Command consistency, audit e replay | Matriz e semântica conceitual estão descritas; selecionar mecanismo de atomicidade, identidade/recuperação e fechar cobertura de commands por release. |
| ARCH-09 | Modelo temporal | Tipos e contexto de timezone estão descritos; fechar fonte/configuração da zona, instante de referência, bordas por workflow e resolução de ambiguidade. |
| ARCH-10 | Actor e autorização de jobs | Distinguir human/system/service actors e novas decisões de negócio de consequências já autorizadas; fechar autoridade/revalidação e envelope por classe de job. |
| ARCH-11 | Atomicidade de AuditLog obrigatório | Tornar obrigatório que fato de sucesso e AuditLog requerido confirmem/falhem juntos; selecionar contrato transacional concreto na implementação. |

## Remediação proposta — não aprovada como ADR

A [remediação arquitetural](../../ARCHITECTURE-REMEDIATION.md) fortalece documentalmente o contrato: tenant obrigatório/fail-closed para toda persistência tenant-scoped; uma fronteira de Application para commands/jobs/assistência; distinção de actor e subject; atomicidade do fato e AuditLog obrigatório; semântica por operação para concorrência/replay; e modelo temporal explícito.

Isso resolve os requisitos conceituais descritos por ARCH-07/08/09/10/11, mas **não aprova** escolhas de implementação, ferramentas ou regras de negócio. Os itens seguem propostas para revisão humana. Em particular, a forma do contrato de persistência, mecanismo de consistência, operação/outbox, clock/timezone, identidade de operação e authority para actors não humanos precisam de decisão e validação no projeto de aplicação.

As decisões de produto e domínio que condicionam essas propostas continuam no [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md); esta revisão não as resolve.

## Dependências de decisão de domínio

OD-01 a OD-18 continuam sendo entradas para capabilities, workflows e consistência. Em particular, operação de voto/apuração, governança de grant, autoridade de conta, concorrência de reservas, uso de acesso, custódia de package, retry de Notification, retenção, habilitação de módulos e efeitos de operações em curso não devem ser implementados como regra consolidada até revisão humana.

## Critério de promoção

Promover uma proposta quando requisitos de negócio, volume, segurança, operação e custo forem conhecidos; documentar trade-offs, responsável e impactos antes de transformar escolha técnica em requisito. Não inferir decisão aprovada só porque a arquitetura contém um diagrama ou exemplo.
