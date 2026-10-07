# Regras de visitas e controle de acesso

## Conceitos e distinções

- `Person` é a pessoa.
- `Visit` é a visita planejada ou registrada.
- `AccessAuthorization` é uma permissão contextual e temporal.
- `AccessEvent` é um fato observado: `ENTRY`, `EXIT` ou `DENIED`.
- `Visitor` e `Guest` descrevem papéis contextuais da mesma `Person`; não são identidades independentes.

## Regras

- `ACCESS-01` Uma autorização identifica pessoa autorizada ou critério de identificação aprovado, contexto/escopo e período de validade.
- `ACCESS-02` Uma autorização pode existir sem `Visit` e sem evento de entrada.
- `ACCESS-03` `ENTRY` só pode ser permitido quando as condições de autorização exigidas pela política local forem satisfeitas ou houver exceção explicitamente autorizada.
- `ACCESS-04` `ENTRY` registrado é fato distinto da autorização e deve apontar para seu contexto conhecido; não se pode inferir autorização retroativamente.
- `ACCESS-05` Uma tentativa sem permissão ou após expiração pode ser registrada como `DENIED`, não como `ENTRY`.
- `ACCESS-06` `EXIT` pode ser observado sem `ENTRY` prévio registrado. Isso deve ser documentado como divergência/ocorrência operacional, não usado para fabricar ou reescrever o histórico.
- `ACCESS-07` Revogação ou expiração impede novas autorizações de uso no período/escopo afetado; não apaga AccessEvents anteriores.
- `ACCESS-08` Referência opcional a `Visit`, `Event` ou `Reservation` não amplia o período autorizado além da janela válida aprovada.
- `ACCESS-09` Funcionário de portaria só visualiza e registra dados necessários às suas permissões e tarefas no condomínio de contexto.

## Ciclo conceitual de autorização

Vocabulário candidato: `DRAFT → ACTIVE → EXPIRED`, ou saída por `REVOKED`/`CANCELLED`. `ACTIVE` não significa que houve entrada. `EXPIRED` deriva do fim da validade; `REVOKED` registra revogação anterior; `CANCELLED` pode representar cancelamento antes do uso. Quem emite/revoga, janela inclusiva, uso único/repetido, aprovação, exceções e transições de draft dependem de decisão local/humana.

Hardware, QR Code, credenciais, leitura biométrica e integração física não são regras de domínio.

Ver [domain-model.md](../domain/domain-model.md) e [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
