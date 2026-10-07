# Acesso e autorização

Documento introdutório; regras detalhadas estão em [access.md](./access.md), [permissions.md](./permissions.md) e [tenancy.md](./tenancy.md).

- `AccessAuthorization` expressa uma permissão de acesso no seu contexto e período.
- `AccessEvent` registra fato observado (`ENTRY`, `EXIT`, `DENIED`); autorização não comprova ocorrência.
- A ação deve ser permitida por `RoleAssignment`, `Permission` e `Scope` válidos.
- Expiração/revogação impede novo uso nas condições afetadas, mas não apaga fatos de acesso anteriores.
- Dados exibidos à portaria devem se limitar ao que é necessário à função e ao tenant.

Regras de autorização, hardware e integração física não se confundem: o domínio não escolhe tecnologia de identificação ou controle.
