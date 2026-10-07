# Regras de módulos e features

## Conceitos

- `Module`: agrupamento conceitual de capacidades; não necessariamente entidade operacional autônoma.
- `Feature`: capacidade específica que pode ser oferecida/habilitada.
- `CondominiumModule`: habilitação de módulo no escopo de condomínio.
- `CondominiumFeature`: habilitação de feature no escopo de condomínio.

## Regras

- `MODULE-01` A disponibilidade de capacidade em um condomínio deve ser determinada por habilitação aplicável naquele condomínio.
- `MODULE-02` Habilitar recurso em um tenant não habilita o mesmo recurso em outro.
- `MODULE-03` Habilitação de módulo não concede permissão de uso; autorização continua exigindo role, permission e scope.
- `MODULE-04` Desabilitar um módulo/feature impede novas operações que dependem dele segundo política aprovada, mas não apaga automaticamente dados, evidências ou histórico.
- `MODULE-05` Habilitação/desabilitação e mudança de configuração devem preservar quem alterou, quando, contexto e estado anterior/posterior para auditoria.
- `MODULE-06` Uma feature pode depender de módulo ou outra feature somente quando dependência for declarada; não inferir incompatibilidades.
- `MODULE-07` Não iniciar operação incompatível com estado de habilitação, salvo exceção explícita e autorizada.

## Decisões pendentes

Quem pode ativar/desativar; defaults e dependências; efeito sobre operações em andamento; leitura/exportação após desativação; reativação; conflito de habilitação com plano comercial. `FinancialModule` permanece opcional, sem pressupor ERP, contabilidade oficial ou cobrança.
