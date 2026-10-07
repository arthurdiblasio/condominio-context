# Regras de auditoria

## Diferenças conceituais

- `AuditLog`: trilha de quem fez/solicitou ação, em qual recurso e escopo, quando e com que resultado.
- `Domain Event`: mudança/fato de domínio que expressa algo relevante ocorrido no ciclo do negócio.
- `Operational Event`: observação do processo operacional, como `AccessEvent` ou `PackageEvent`.

Um evento operacional pode motivar uma entrada de auditoria, mas esses conceitos não são equivalentes. A trilha de auditoria não deve ser usada para inventar evento de domínio que não ocorreu.

## Regras

- `AUDIT-01` Registrar ação sensível com actor (ou ator indisponível explicitamente indicado), ação, recurso, instante, condomínio/escopo, resultado e contexto necessário.
- `AUDIT-02` Auditar criação, alteração e encerramento lógico de dados e vínculos sensíveis.
- `AUDIT-03` Auditar concessão, alteração, suspensão e revogação de `RoleAssignment` e permissões.
- `AUDIT-04` Auditar criação/revogação de autorização, `ENTRY`, `EXIT`, `DENIED` e divergências de acesso quando registradas.
- `AUDIT-05` Auditar eventos do pacote, confirmação, retirada, contestação e correção.
- `AUDIT-06` Auditar criação/alteração/cancelamento de reserva e bloqueio de área.
- `AUDIT-07` Auditar assembleia, presença, alteração de elegibilidade, procuração, voto, correção e apuração.
- `AUDIT-08` Auditar habilitação/desabilitação de módulos/features e alterações administrativas sensíveis.
- `AUDIT-09` Alterações corretivas devem preservar referência ao registro corrigido e a autoria da correção; não sobrescrever silenciosamente a história.
- `AUDIT-10` A leitura de auditoria é ela própria sujeita a permission/scope e finalidade; acesso de plataforma não deve ser implícito.

## Limites

Campos exatos, imutabilidade, retenção, exportação, acesso a detalhes pessoais e armazenamento são decisões de política/implementação futuras. Nenhum mecanismo técnico é escolhido aqui.
