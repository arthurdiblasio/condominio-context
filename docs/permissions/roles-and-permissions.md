# Papéis e permissões

Índice de compatibilidade para referências anteriores. A especificação atual é:

- [authorization-model.md](./authorization-model.md): sujeito, gates e decisão conceitual.
- [roles.md](./roles.md): definição e capacidades dos papéis.
- [permissions.md](./permissions.md): taxonomia de actions/capabilities.
- [scopes.md](./scopes.md): limites e abrangência.
- [role-assignment.md](./role-assignment.md): concessão, ciclo e delegação.
- [data-visibility.md](./data-visibility.md): dados pessoais e escopo de leitura.
- [permission-matrix.md](./permission-matrix.md): Role × Permission × Scope.

## Princípios

- `Person` não é `UserAccount`; a conta associa-se à pessoa segundo política.
- `Role` não é `Permission`.
- `RoleAssignment` dá papel à pessoa em escopo e período.
- Uma permission só se aplica a recursos cobertos no tenant correto.
- Nenhuma herança ou prioridade entre papéis é assumida silenciosamente.
- Habilitar feature não concede permissão.
