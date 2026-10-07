# Domínio independente da infraestrutura

## Modelo de domínio

O Domain é a fonte das entidades, value concepts, fronteiras de agregados candidatas, políticas, invariantes, transições e eventos/fatos de negócio descritos em [DOMAIN.md](../../DOMAIN.md), [BUSINESS-RULES.md](../../BUSINESS-RULES.md) e [domain model](../domain/domain-model.md).

Regras de aplicação do princípio:

- Domain expressa `Person`, `Condominium`, `Unit`, `Reservation`, `Package`, `AccessAuthorization`, `Assembly` e conceitos correlatos conforme modelo aprovado.
- Invariantes e transições ficam próximas do conceito que protegem; não em handler HTTP nem em callback de ORM.
- Regras configuráveis recebem configuração contextual explicitamente; o domínio não inventa defaults para OD abertas.
- Condições temporais usam instantes fornecidos pela operação; domínio não lê relógio de sistema diretamente.
- Eventos representam fatos ocorridos após aceitação de regra; comando, tentativa de operação, audit e Notification são conceitos distintos.

## Domain Entity ≠ persistence model

Uma entidade de domínio não deve ser criada para satisfazer GORM. Um modelo de persistência pode refletir chaves, índices, relações, colunas e necessidades do PostgreSQL; não se torna automaticamente entidade/agregado de domínio.

Manter mapeamento explícito quando as formas divergirem:

- Domain protege semântica e invariantes.
- Persistence model protege forma de gravação/reconstrução.
- Mapper/adaptador é responsabilidade de Infrastructure.
- Mudança de schema não deve redefinir silenciosamente regra, estado ou nome do domínio.

Trade-off: mapeamento separado custa código e manutenção, mas reduz acoplamento do domínio ao ORM e permite testar regras sem banco. Compartilhar estrutura só deve ser considerado quando preserva independência sem introduzir tags, convenções ou lifecycle do ORM no modelo conceitual; não é a premissa arquitetural.

## Fronteiras de agregado

Usar os candidatos documentados em [domain-model.md](../domain/domain-model.md): Condominium não contém todos os dados do tenant; Unit não agrega automaticamente relações; Reservation, Package, Assembly e Notification protegem apenas as invariantes que suas regras aprovadas exigirem. A arquitetura não transforma toda relação em carregamento de agregado nem toda ação em transação ampla.

As fronteiras finais de PackageEvent, Vote, Participant e NotificationDelivery dependem de semântica, volume e consistência definidas; requisitos de auditoria/histórico persistem em qualquer escolha.

## Proibições arquiteturais

Domain não importa nem executa Gin, GORM, PostgreSQL, HTTP, email, WhatsApp, Push, Redis, AWS, storage, broker ou scheduler. Nenhuma indisponibilidade técnica é convertida em fato de domínio ocorrido.
