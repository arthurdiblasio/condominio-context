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

A [revisão crítica da arquitetura](../../ARCHITECTURE-CRITICAL-REVIEW.md) classifica a proposta como `NEEDS REVISION` e o status como `ARCHITECTURE_REQUIRES_REVISION`. As propostas ARCH-01..06 continuam propostas, não ADRs aprovados; ARCH-03/04/06 precisam de maior precisão sobre ownership e garantias. TECH-01 é stack informada, não alternativa comparada.

Novas propostas para avaliação, não decisões aprovadas:

| ID candidato | Tema | Questão a fechar |
|---|---|---|
| ARCH-07 | Tenant-scoped persistence contract | Como tornar obrigatório o tenant em todo acesso operacional, enumerar operações globais e testar isolamento; defesa adicional da persistência ainda será avaliada. |
| ARCH-08 | Command consistency, audit e replay | Para cada command exposto, declarar invariantes, atomicidade com audit/publication intent, comportamento pós-falha e semântica de repetição. |
| ARCH-09 | Modelo temporal | Distinguir instantes, datas civis, timezone e períodos de validade/agendamento antes de implementar workflows temporais. |

As decisões de produto e domínio que condicionam essas propostas continuam no [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md); esta revisão não as resolve.

## Dependências de decisão de domínio

OD-01 a OD-18 continuam sendo entradas para capabilities, workflows e consistência. Em particular, operação de voto/apuração, governança de grant, autoridade de conta, concorrência de reservas, uso de acesso, custódia de package, retry de Notification, retenção, habilitação de módulos e efeitos de operações em curso não devem ser implementados como regra consolidada até revisão humana.

## Critério de promoção

Promover uma proposta quando requisitos de negócio, volume, segurança, operação e custo forem conhecidos; documentar trade-offs, responsável e impactos antes de transformar escolha técnica em requisito. Não inferir decisão aprovada só porque a arquitetura contém um diagrama ou exemplo.
