# Regras de multi-tenancy

## Regras

- `TENANCY-01` A operação que cria, consulta, altera ou encerra dado operacional deve identificar o `Condominium` alvo.
- `TENANCY-02` O contexto de um pedido não autoriza inferir ou substituir o condomínio a partir apenas de uma pessoa, unidade, bloco ou papel homônimo.
- `TENANCY-03` Todo `Block`, `Unit`, `CommonArea`, vínculo com unidade e registro operacional deve referenciar o mesmo condomínio de contexto.
- `TENANCY-04` `Person` pode participar de diversos condomínios; cada vínculo e `RoleAssignment` conserva o seu próprio tenant.
- `TENANCY-05` A existência de `RoleAssignment` em A não concede autorização em B, mesmo que papel e pessoa sejam idênticos.
- `TENANCY-06` Ações de plataforma que atravessam tenants requerem papel, permissão, escopo platform e finalidade explícita; não são acesso implícito de suporte.
- `TENANCY-07` Dados de negócio, acesso, visita, unidade, pacote, votação e notificação associados a um condomínio não são compartilhados com outro por padrão.
- `TENANCY-08` Dados da própria plataforma — identidade global de conta quando assim decidida, catálogo de módulos/features e planos conceituais — podem ser globais; relações de habilitação e dados de uso são contextuais ao condomínio.

## Compartilhamento

Compartilhamento entre tenants só pode ocorrer por operação autorizada e finalidade documentada. Não transfere propriedade, vínculo, papel, autorização, evento ou histórico. Dados agregados/anônimos só podem ser compartilhados quando a política de privacidade aprovar que não há identificação ou inferência de condomínio/pessoa.

## Invariantes de isolamento

- Uma operação nunca combina dados de tenants diferentes como se fossem um só contexto.
- Uma `Person` comum a dois condomínios não elimina segregação entre seus registros.
- Uma relação com `Unit` tem que respeitar o tenant daquela unidade.
- A ausência de autorização cross-tenant é tratada como negação, não como busca global de fallback.

Ver também [domain-model.md](../domain/domain-model.md) e [../permissions/roles-and-permissions.md](../permissions/roles-and-permissions.md).
