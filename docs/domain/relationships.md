# Relacionamentos do domínio

## 1. Plataforma e tenant

```mermaid
flowchart TD
    Platform --> CA[Condominium A]
    Platform --> CB[Condominium B]
    Platform --> CC[Condominium C]
    CA --> DA[Dados e vínculos do tenant A]
    CB --> DB[Dados e vínculos do tenant B]
    CC --> DC[Dados e vínculos do tenant C]
```

`Platform` agrega condomínios. Cada condomínio é uma fronteira de contexto para blocos, unidades, áreas comuns e relações operacionais. Uma `Person` pode participar de vários tenants por vínculos separados; isso não funde nem compartilha os dados de cada condomínio.

```mermaid
classDiagram
    Platform "1" --> "*" Condominium
    Condominium "1" --> "*" Block
    Condominium "1" --> "*" Unit
    Block "0..1" --> "*" Unit
    Condominium "1" --> "*" CommonArea
```

A cardinalidade Block-Unit e a possibilidade de unidade sem bloco devem ser confirmadas para todos os tipos de propriedade.

## 2. Identidade, conta e atribuição contextual

```mermaid
flowchart LR
    Person -->|zero ou uma ou mais contas conforme política| UserAccount
    Person --> RoleAssignment
    RoleAssignment --> Role
    RoleAssignment --> Scope
    Role --> Permission
    Scope --> TenantResource[Platform / Condominium / Block / Unit / Feature]
```

- `Person` é a pessoa do domínio.
- `User` é o termo corrente para identidade de acesso; `UserAccount` explicita essa conta e não representa uma pessoa adicional.
- `RoleAssignment` atribui `Role` a uma pessoa em contexto `Scope`.
- `Permission` descreve ação autorizável; não é papel nem vínculo com unidade.
- A conta não cria automaticamente vínculos, papéis ou permissões.

Relação pessoa-conta (uma ou várias, possibilidade de contas compartilhadas, associação e desativação) é uma política ainda aberta.

## 3. Pessoa e vínculos com unidade

```mermaid
flowchart LR
    Person --> UnitOwnership
    UnitOwnership --> Unit
    Person --> UnitResidency
    UnitResidency --> Unit
    Person --> UnitTenancy
    UnitTenancy --> Unit
    Person --> Proxy
    Proxy --> AssemblyOrAgenda[Assembly / AgendaItem]
```

`UnitOwnership`, `UnitResidency` e `UnitTenancy` são relações semanticamente separadas. Podem coexistir para uma pessoa e unidade quando os fatos assim determinarem. Nenhum vínculo implica UserAccount, papel ou outro vínculo. `Proxy` representa delegação delimitada e não altera titularidade ou residência.

## 4. Pessoa visitante, visita, autorização e ocorrência de acesso

```mermaid
flowchart TD
    Person --> Visit
    Host[Person anfitriã / Unit] --> Visit
    Visit -. opcional .-> Event
    Visit -. opcional .-> Reservation
    Visit -. opcional .-> AccessAuthorization
    AccessAuthorization -. pode existir sem .-> Visit
    AccessAuthorization -. optional context .-> AccessEvent
    Visit -. optional context .-> AccessEvent
    AccessEvent --> Entry[ENTRY]
    AccessEvent --> Exit[EXIT]
    AccessEvent --> Denied[DENIED]
```

- Visit descreve intenção/contexto planejado ou registrado.
- AccessAuthorization é a permissão pretendida durante condições/período definidos.
- AccessEvent é um fato observado. Não é prova automática de autorização.
- Uma autorização pode não resultar em evento ENTRY; uma ocorrência DENIED pode acontecer sem autorização.
- `Visitor` e `Guest` classificam `Person` por contexto e não duplicam identidade.

## 5. Reserva, evento e convidados

```mermaid
flowchart TD
    Unit --> Reservation
    Person --> Reservation
    CommonArea --> Reservation
    Reservation -. pode contextualizar .-> Event
    Event --> GuestPerson[Person como convidada]
    Event -. pode relacionar .-> AccessAuthorization
    Reservation -. pode relacionar .-> AccessAuthorization
```

Reserva associa solicitante, unidade, área e período. `Event` pode dar contexto ao uso; pessoas convidadas são relações contextuais. Aprovação, pagamento, capacidade, lista obrigatória e cancelamento seguem configuração/regra que ainda precisa ser decidida.

## 6. Pacote e histórico

```mermaid
flowchart TD
    Unit --> Package
    Recipient[Person destinatária] --> Package
    Package --> PE1[PackageEvent: RECEIVED]
    Package --> PE2[PackageEvent: NOTIFIED]
    Package --> PE3[PackageEvent: CONFIRMED]
    Package --> PE4[PackageEvent: PICKED_UP]
    Package --> PE5[PackageEvent: CANCELLED]
    Actor[Person/ator registrador] --> PE1
    Actor --> PE4
```

`PackageEvent` registra fatos com ator e instante apropriados para cada ocorrência. `PackageDelivery` não é uma entidade principal. `Notification` e suas tentativas de entrega mantêm ciclo próprio e podem ser referenciadas pelo evento de notificação do pacote.

## 7. Notificação e canais de entrega

```mermaid
flowchart LR
    Subject[Evento de domínio: Package, Reservation, Assembly...] --> Notification
    Recipient[Person destinatária] --> Notification
    Notification --> Push[NotificationDelivery: Push]
    Notification --> WhatsApp[NotificationDelivery: WhatsApp]
    Notification --> Email[NotificationDelivery: Email]
    Notification --> InApp[NotificationDelivery: In-app]
```

Uma `Notification` expressa uma intenção de comunicação. Cada `NotificationDelivery` é tentativa/resultado independente; canal não identifica provedor. A notificação pode ter zero, uma ou várias tentativas conforme política ainda a decidir.

## 8. Assembleia, presença, elegibilidade e voto

```mermaid
flowchart TD
    Assembly --> Participant
    Assembly --> AgendaItem
    Assembly --> Proxy
    Assembly --> Quorum
    Participant --> Presence[Presença registrada]
    AgendaItem --> VotingEligibility
    Participant --> VotingEligibility
    Proxy -. pode representar conforme regra .-> Participant
    VotingEligibility --> Vote
    AgendaItem --> Vote
```

- `Participant` indica associação/participação; presença é fato separado.
- `VotingEligibility` é determinada por pauta e não é sinônimo de presença.
- `Proxy` representa somente dentro de sua validade e escopo decididos.
- `Vote` registra o voto efetivamente realizado; elegibilidade não implica voto.
- `Quorum` é apurado segundo regra humana/jurídica ainda não fechada.

## 9. Módulos e features por condomínio

```mermaid
flowchart LR
    Module --> Feature
    Condominium --> CondominiumModule
    CondominiumModule --> Module
    Condominium --> CondominiumFeature
    CondominiumFeature --> Feature
```

`Module` agrupa capacidades conceituais; `Feature` é uma capacidade. `CondominiumModule` e `CondominiumFeature` expressam habilitação contextual por tenant. Dependências, defaults e autoridade de alteração não estão decididos. Módulo não precisa ser entidade operacional independente.

## 10. Regras de integridade conceitual

1. Não usar `UserAccount ↔ Unit` como substituto de vínculo de domínio: utilizar `UnitOwnership`, `UnitResidency` ou `UnitTenancy` conforme o fato.
2. Não representar Visitor e Guest como pessoas duplicadas quando se referem à mesma `Person`.
3. Não inferir entrada física a partir de `AccessAuthorization`; registrar `AccessEvent`.
4. Não apagar o histórico do pacote ao alterar seu status atual; registrar `PackageEvent`.
5. Não tratar `Notification` e `NotificationDelivery` como uma só ocorrência.
6. Não inferir elegibilidade, presença ou voto uns dos outros.
7. Toda relação operacional deve preservar o condomínio de contexto e impedir acesso transversal sem autorização explícita.
8. Papéis, permissões e vínculos não são intercambiáveis.

## Referências

- [domain-model.md](./domain-model.md)
- [responsibility-matrix.md](./responsibility-matrix.md)
- [../../BUSINESS-RULES.md](../../BUSINESS-RULES.md)
- [../../PERMISSIONS.md](../../PERMISSIONS.md)
- [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md)
