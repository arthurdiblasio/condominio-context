# Infrastructure e fronteiras externas

## Papel

Infrastructure contém adaptações técnicas à persistência e serviços. É substituível, não decide regras de domínio e implementa somente capacidades necessárias aos casos de uso.

Stack informada: Go, Gin, GORM e PostgreSQL. Gin é adaptador de Interface/API; GORM/PostgreSQL são opções de persistência em Infrastructure. Não há implementação, schema nem configuração executável aqui.

## Repositories

Repositories expressam necessidades reais de agregado/caso de uso, com tenant e fronteira explícitos. Candidatos incluem `ReservationRepository`, `PackageRepository`, `AssemblyRepository`, conforme necessidade concreta.

- Evitar `GenericRepository<T>` universal e CRUD público genérico.
- Para recurso tenant-scoped, tenant validado ou garantia equivalente é obrigatório no contrato de toda leitura/escrita; tenant não pode ser parâmetro opcional. A forma literal do contrato permanece livre.
- Operação conceitualmente equivalente a `FindByID(resourceID)` é proibida se puder retornar recurso tenant-scoped sem limitar/verificar tenant. Operação tenant-scoped deve exigir TenantContext/tenant validado ou garantia equivalente.
- Operações de Platform/global devem ser separadas e declaradas; não usar método unscoped tenant-scoped como shortcut de suporte.
- Validar que resource e todas as referências/joined resources pertencem ao tenant indicado antes de retornar/prosseguir.
- Aplicar isolamento antes de paginação, agregação, busca, exportação, relatório ou cache; não filtrar somente após carregar dados.
- Queries complexas podem usar consultas de leitura/projeções específicas sem carregar agregado inteiro.
- Repositories não são endpoints nem camada para esconder regra de autorização.
- Definição do contrato pertence ao lado interno (Application port, ou Domain quando sua linguagem exigir); implementação concreta pertence a Infrastructure.

### Classificação de acesso

Cada operação de persistência deve identificar explicitamente:

- `TENANT-SCOPED`: exige TenantContext validado na leitura/escrita; ausência/mismatch falha fechada.
- `PLATFORM-SCOPED`: opera só em dados globais nomeados (por exemplo catálogo); não adquire acesso operacional a tenants.
- `CROSS-TENANT`: operação excepcional, explícita, por tenant-alvo e propósito, com permission de plataforma e auditoria. Não é busca sem filtro nem agregação implícita.

Nenhuma consulta/unscoped interface é criada para “conveniência”. A arquitetura não escolhe RLS, schema por tenant ou outra defesa física, mas exige testes de integração que demonstrem que omissão/mismatch não expõe dados.

## GORM vs Domain

Persistence models podem existir futuramente separados das entidades de domínio. Mapper converte entre ambos. Tags, callbacks e lifecycle de GORM não definem invariantes nem transições de domínio. Evitar transação/sessão ORM vazando para Domain ou contratos públicos de Application.

## Integrações externas

| Capacidade | Fronteira |
|---|---|
| WhatsApp, e-mail, Push e canais adicionais | Adaptadores de envio por canal sob política de Notification/Delivery; domínio não conhece provider. |
| Storage de documentos/evidências | Porta de armazenamento com metadados de tenant, finalidade, classificação e referência no domínio; binários permanecem fora de regras que não os interpretam. |
| Clock / TimeProvider | Fornece instantes a Application; Domain recebe instante explícito e aplica regras temporais. |
| ID generation | Serviço/adaptador de geração escolhido após decidir formato, ordenação e exposição. |
| Messaging / event publication | Adaptador opcional; introduzido apenas para requisito de entrega assíncrona, possivelmente com outbox. |
| External services | Portas específicas por necessidade, sem SDK/provider em Domain ou Application. |

Documentos pessoais, procurações, atas, comprovantes, evidências de pacote e documentos financeiros requerem política de classificação, acesso, retenção e exclusão/anonimização antes de definir storage. Nenhum armazenamento é escolhido.

## IDs

Não há padrão UUID, ULID ou outro registrado como decisão vigente. UUIDs são amplamente suportados pelo PostgreSQL e evitam depender de sequência centralizada, mas UUIDv4 não tem ordenação temporal e formatos ordenáveis como UUIDv7/ULID podem revelar informação de ordem/tempo e exigem decidir geração/canonicalização.

**Recomendação provisória:** comparar identificadores opacos, distribuição, índices/ordenação, interoperabilidade, geração offline e risco de exposição antes de fixar formato; não usar identificadores como segredo ou prova de autorização. A escolha final é uma decisão técnica pendente.

## Configuration

Manter separados:

- configuração global da Platform (catálogo e capacidades globais);
- configuração de produto/tenant por Condominium;
- habilitação `CondominiumModule`/`CondominiumFeature`;
- regras de negócio configuráveis e seus valores aprovados;
- dados operacionais do domínio.

Feature flag não concede Permission, não altera retrospectivamente fatos e não substitui configuração auditável. Defaults e dependências seguem OD-11/17.
