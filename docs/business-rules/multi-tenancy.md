# Multi-tenancy

Documento introdutório para regras de isolamento, com detalhamento vigente em [tenancy.md](./tenancy.md).

- A plataforma pode atender vários condomínios.
- Cada operação tenant-scoped requer contexto de condomínio explícito e coerente.
- Pessoa comum a vários condomínios não une nem compartilha os dados operacionais desses tenants.
- Atribuição de papel ou relação com unidade em um condomínio não autoriza acesso em outro.
- Ação cross-tenant requer permissão explícita de plataforma, finalidade identificável e auditoria conforme escopo.

Ver também [relationships.md](../domain/relationships.md) e [PERMISSIONS.md](../../PERMISSIONS.md).
