# Grafo de dependências

## Direção

```mermaid
flowchart TD
    Client[Consumidores]
    API[Interface / API adapter]
    App[Application use cases]
    Domain[Domain model and policies]
    Ports[Application ports]
    Infra[Infrastructure adapters]
    DB[(PostgreSQL)]
    External[External services]

    Client --> API
    API --> App
    App --> Domain
    App --> Ports
    Infra -.implements.-> Ports
    Infra --> Domain
    Infra --> DB
    Infra --> External
```

Setas contínuas mostram chamada/direção conceitual. Infrastructure implementa portas definidas para atender necessidades de Application; o Domain não chama Infrastructure. A relação `Infra → Domain` existe para persistir/reconstituir conceitos e não permite que o domínio importe GORM ou tipos de banco.

## Regras anti-ciclo

- Domain não importa Application, Interface nem Infrastructure.
- Application pode orquestrar mais de um módulo, mas não depende de um adaptador concreto.
- Interface chama casos de uso; não chama repositório nem Aggregate diretamente para contornar guardas.
- Um módulo de domínio não deve depender do serviço HTTP ou do repositório concreto de outro módulo.
- Referências entre módulos usam conceitos/identificadores e contratos estáveis; coordenação entre módulos fica na camada Application.
- Eventos de domínio conectam reações sem criar dependência de retorno entre módulos.
- Um módulo transversal de autorização não precisa carregar todos os agregados: recebe uma descrição contextual do recurso e valida tenant/scope através de fronteiras de aplicação.

## Grafo conceitual da aplicação

```mermaid
flowchart LR
    Interface[Interface/API] --> UseCases[Application]
    UseCases --> Identity[Identity]
    UseCases --> Permissions[Authorization]
    UseCases --> Condo[Condominium]
    UseCases --> Units[Units]
    UseCases --> Reservations[Reservations]
    UseCases --> Access[Access]
    UseCases --> Packages[Packages]
    UseCases --> Assemblies[Assemblies]
    UseCases --> Notifications[Notifications]
    UseCases --> Audit[Audit]
    UseCases --> Finance[Finance - optional]
    Infra[Infrastructure adapters] -.ports.-> UseCases
    Reservations -->|facts| Notifications
    Packages -->|facts| Notifications
    Access -->|facts| Notifications
    Assemblies -->|facts| Notifications
    Identity -->|Person references| Units
    Condo -->|tenant context| Units
    Condo -->|tenant context| Reservations
    Condo -->|tenant context| Access
    Condo -->|tenant context| Packages
    Condo -->|tenant context| Assemblies
```

O desenho representa contextos de negócio e coordenação, não acoplamento obrigatório de pacotes ou chamada síncrona. Dependências específicas devem ser confirmadas ao detalhar os workflows; não se presume que cada seta represente import direto entre módulos.

## Caminho proibido

`Gin → GORM → Domain`, `Domain → PostgreSQL`, `Application → Gin` e `Domain → provider` violam a direção proposta. Consultar [layers](./layers.md) e [módulos](./modules.md).
