# Estrutura conceitual do projeto

Esta árvore descreve responsabilidades futuras e **não cria diretórios nem código** no repositório de domínio. É uma proposta para o repositório separado `condominio-api`.

```text
condominio-api/
  cmd/                         # composição e inicialização do processo
  internal/
    interfaces/
      http/                    # adaptador Gin: parsing, auth boundary, responses
    application/
      usecases/                # commands e queries por workflow
      ports/                   # contratos necessários à aplicação
    domain/
      identity/
      condominium/
      units/
      permissions/
      reservations/
      access/
      packages/
      assemblies/
      notifications/
      audit/
      finance/                 # opcional; fora do MVP proposto
    infrastructure/
      persistence/             # PostgreSQL/GORM, mapping e repositories
      integrations/            # adapters externos
      time/                    # Clock/TimeProvider
      identifiers/             # gerador de IDs, se escolhido
      jobs/                    # execução agendada/assíncrona, se aprovada
      observability/           # logs, metrics e tracing
```

Os nomes não são estrutura aprovada de pacotes. Módulos podem ser agrupados de outra forma desde que mantenham responsabilidades e direção de dependência.

## Composição

O ponto de inicialização futuro conecta interfaces com casos de uso e injeções de ports/adapters. Essa composição não deve mover regras para o bootstrap nem introduzir dependências de Infrastructure nos módulos de Domain/Application.

## Estratégia de testes

- Domain: testes determinísticos de invariantes, transições, fatos e policies, sem banco ou relógio global.
- Application: testes de casos de uso com ports controladas, cobrindo autorização, tenant, erros, transação e efeitos.
- Infrastructure: testes de integração para persistência/mapeamento e providers quando implementados.
- Interface: testes de parsing/mapeamento/contrato; nenhuma regra de negócio deve existir somente aqui.
- Cruzados: cenários de WF-01..26 priorizados por risco, incluindo tentativa cross-tenant, auditoria, concorrência e operação repetida.

Mocks não devem fazer um teste passar se a garantia exige consistência real no adaptador escolhido.

## Paginação, busca e versionamento

- A camada de query entrega ordenação determinística e filtros tenant-aware, preservando limites do [API Contract](../api/queries.md).
- Estratégia de paginação (cursor, offset, limites) continua em aberto; os clientes não devem receber coleções ilimitadas.
- PostgreSQL é candidato inicial suficiente para busca estruturada e filtros tenant-scoped; avaliar índice/consulta antes de adotar Elasticsearch/OpenSearch. Busca textual complexa, ranking, idioma e latência podem reabrir a decisão.
- Evolução de contrato segue requisitos de compatibilidade do [versionamento](../api/versioning.md); nenhuma convenção de URL/header foi escolhida.

## Tarefas de fundo e tempo

| Ação temporal candidata | Regra de domínio vs disparo técnico |
|---|---|
| Expiração de AccessAuthorization, convite ou RoleAssignment | Validade/instante efetivo são regras de domínio; scheduler pode iniciar a avaliação/registro conforme policy aprovada. |
| Reservation com prazo, lembrete ou expiração | Definir prazo e consequência em OD-04; scheduler não aprova nem cancela por convenção. |
| NotificationDelivery e retry | Cada tentativa é fato independente; retry, limites e canais seguem OD-08/18. |
| Lembretes/ações relacionados à Assembly | Calendário e finalidade seguem procedimento aprovado; fechamento/apuração não são inferidos pelo scheduler. |
| Visit `NO_SHOW` | Exige janela e critério aprovados; ausência de evento não basta por si só. |

`Clock`/TimeProvider fornece o instante de avaliação; scheduler técnico apenas desperta e invoca um caso de uso idempotente com tenant, finalidade e ator/processo explicitamente atribuídos. Uma tarefa não herda a identidade de uma requisição anterior. Não executar transição irreversível sem regra aprovada.
