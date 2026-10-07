# Workflow de RoleAssignment

## WF-09 — Conceder, alterar, suspender, reativar, revogar ou deixar expirar um papel

### Objetivo

Gerir papel de uma Person em scope e tenant definidos sem confundir papel, permission, vínculo residencial ou `Proxy`.

### Participantes

- Solicitante: pessoa com finalidade legítima; papéis candidates incluem `condominium_admin`, `syndic` e `platform_admin`, sem authority presumida.
- Concessor/revisor: ator com `role_assignment.grant`, `.suspend`, `.reactivate` ou `.revoke` explicitamente atribuído no scope; recebedor é Person.
- `Proxy` de assembleia não participa como concessor de RoleAssignment.

### Contexto/Tenant

Scope explícito: Platform, Condominium, Block, Unit ou Feature. Assignment tenant-scoped fica limitado ao tenant indicado. Manager de A/B/C não recebe D.

### Gatilho e pré-condições

Necessidade de atribuir, atualizar, suspender, reativar ou revogar um papel; Person/Role/resource/scope identificáveis; ator com autoridade não superior ao seu grant ou com delegação explícita válida.

### Permissões necessárias

Permissions específicas do catálogo `role_assignment.read/grant/suspend/reactivate/revoke`; concessão e mudança de scope são elevadas/críticas. A existência do papel não concede as próprias permissions administrativas.

### Dados/conceitos envolvidos

`Person`, `UserAccount` se acesso autenticado, `Role`, `RoleAssignment`, `Permission`, `Scope`, tenant, vigência, delegação, `AuditLog`.

### Fluxo principal

1. Ator seleciona target Person, Role, Scope e tenant explicitamente.
2. Avalia-se permission de grant e limite de autoridade do ator; rejeita-se escalação superior sem delegação.
3. Verificam-se consistência tenant-resource, escopo suportado, vigência e incompatibilidades declaradas.
4. Concede-se ou altera-se assignment com condições e período aprovados.
5. Suspension/revoke impede novas ações baseadas no grant após efetividade definida; expiration encerra validade pelo período configurado.
6. Reativação exige nova validação/authority; não restaura grants separados expirados ou revogados.
7. Notifica-se a pessoa somente conforme política; a mensagem não ativa o grant.

### Fluxos alternativos

- Grant sem conta: pode representar autorização futura ou papel nominal, mas se isso é permitido precisa de OD-03; não cria capacidade autenticada.
- Pedido de papel no mesmo ou maior nível: negar ou encaminhar para delegação aprovada.
- Alteração de tenant/scope: tratar como revogação/novo grant ou alteração apenas conforme política, sem transferência implícita.
- Role incompatível: conflito de papéis é encaminhado à política; não se resolve por prioridade automática.

### Validações e transições

Lifecycle de referência: [RoleAssignment](../state-machines/identity-and-reservations.md#roleassignment). `ACTIVE`, `SUSPENDED`, `REVOKED`, `EXPIRED`; `PENDING` somente quando política criar grant futuro/aguardando validação. `REVOKED` e `EXPIRED` encerram o ciclo; somente suspensão pode ser reativada pelo fluxo autorizado. `ACTIVE` requer pessoa, papel, scope e vigência coerentes. Nenhum estado ou grant atravessa tenant automaticamente.

### Eventos/fatos e notificações

Assignment requested/granted/changed/suspended/reactivated/revoked/expired como fatos separados; notificação é opcional.

### Auditoria

Obrigatória para criação/alteração de grant, mudança de scope, delegação, suspensão, reativação e revogação. Registrar concessor, recebedor, papel, scope, tenant, vigência, resultado e base/contexto; restringir consulta da trilha.

### Pós-condições/término

Assignment vigente conforme operação, ou sem acesso novo após revogação/suspensão/expiração; histórico de decisões permanece. Término ocorre após mudança aceita ou negação/encaminhamento.

### Erros/negações

Permission ausente; actor sem autoridade; autoelevação; superioridade de grant; scope/tenant incoerente; Role inexistente; vigência inválida; conta inativa para ação autenticada; regra de conflito indefinida.

### OPEN DECISIONS

OD-02, OD-14, OD-15, OD-16 e OD-03 em [OPEN-DECISIONS.md](../../OPEN-DECISIONS.md): autoridade por role/scope, autoatribuição, precedência, herança, delegation, ciclo e grant sem conta.

## Delegação de autoridade versus Proxy

Delegação de RoleAssignment, se aprovada, transfere só permissions enumeradas, scope, vigência e possibilidade de revogação; não deve ser transitiva por padrão. `Proxy` é representação para assembleia/ato definido e não concede conta, papéis administrativos ou acesso a outros recursos. Legalidade e limites do Proxy seguem [assemblies.md](./assemblies.md).
