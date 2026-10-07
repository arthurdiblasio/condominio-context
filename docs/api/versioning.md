# Versionamento conceitual do contrato

## Princípio

Contratos públicos devem evoluir de forma versionável sem quebrar consumidores existentes. Mudança de significado de recurso, command, query, error category ou evento público precisa de revisão de compatibilidade e documentação antes de afetar consumidores.

## Requisitos de compatibilidade

- Mudanças aditivas que não alteram significados existentes podem ser candidatas a evolução compatível, após validar clientes Admin Web, Mobile Resident, Mobile Doorman e Mobile Syndic.
- Remover/renomear uma operação, alterar guardas, permission mapping, estados, erros ou semântica de evento é mudança potencialmente incompatível.
- Mudanças legais/configuráveis devem ser explícitas no contrato e não podem se ocultar em alteração de payload.
- Eventual deprecation deve identificar operação/versão afetada, alternativa e período de migração decidido pelo produto.
- Respostas representativas, transporte, política de compatibilidade binária e migração de clientes não são definidos nesta etapa.

## Escolhas não tomadas

Não selecionar versionamento em URL (`/v1`), header, content negotiation, esquema de compatibilidade, política de suporte ou data de sunset. São decisões posteriores de arquitetura/produto. Esta proposta não gera OpenAPI.
