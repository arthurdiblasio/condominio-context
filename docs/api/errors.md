# Categorias conceituais de erro da API

Categorias para documentação e alinhamento entre consumidores. Não definem códigos, schema de resposta, HTTP status, retries técnicos ou mensagens exatas.

| Categoria | Significado | Exemplo/limite |
|---|---|---|
| `INVALID_INPUT` | Comando ou filtro não satisfaz requisitos de domínio de forma ou conteúdo. | Período inválido ou campo não permitido; não detalhar dados privados indevidamente. |
| `UNAUTHENTICATED` | Identidade ausente/inválida para operação que exige subject autenticado. | Diferente de sujeito conhecido sem capability. |
| `ACCOUNT_INACTIVE` | UserAccount inexistente para agir, suspensa ou desativada segundo lifecycle aprovado. | Não apagar Person ou vínculos; OD-03 define autoridade/estado. |
| `FORBIDDEN` | Subject autenticado, mas permission ou authority não permitem a ação. | RoleAssignment/Permission ausente ou actor sem authority de grant. |
| `SCOPE_MISMATCH` | Grant existe mas não cobre resource/ação/contexto. | Scope Unit não cobre outra Unit sem regra explícita. |
| `TENANT_MISMATCH` | Contexto selecionado e tenant do resource/referências divergem. | Negação sem confirmar conteúdo/existência de resource alheio. |
| `NOT_FOUND` | Resource não existe ou não é visível ao subject. | Pode ser indistinguível de recurso não revelável em outro tenant conforme privacy. |
| `INVALID_STATE` | Resource está em estado que não permite o command. | Aprovar Reservation já cancelada ou votar em AgendaItem fechado. |
| `BUSINESS_RULE_VIOLATION` | Estado poderia aceitar a operação, mas regra/contexto impede. | Falta de aprovação, disponibilidade, referência incoerente. |
| `CONFLICT` | Operação conflita com fato/decisão concorrente ou recurso já alterado. | Reserva sobreposta proibida, grant concorrente, retirada em revisão. |
| `EXPIRED` | A validade temporal aprovada encerrou. | Authorization, convite, grant ou tentativa; não reativar implicitamente. |
| `REVOKED` | A autorização/relação foi explicitamente revogada. | Diferente de expired, suspended ou account disabled. |
| `MODULE_DISABLED` | A capacidade necessária está desabilitada no Condominium. | Não remover histórico nem grants. |
| `NOT_ELIGIBLE` | Condição de VotingEligibility não satisfeita na pauta. | Não é falha de Permission; acesso técnico pode existir. |
| `DUPLICATE_OPERATION` | A mesma intenção/ocorrência não pode ser aplicada novamente segundo regra de domínio. | Distinguir de NEW_EVENT legítimo; OD-18. |
| `AUDIT_REQUIRED` | A operação exige trilha/autoria e o requisito não pode ser satisfeito. | Não declarar sucesso sem trilha em operação sensível. |
| `DECISION_REQUIRED` | Uma policy aberta impede resolver legitimamente a ação. | Não transformar falta de regra em sucesso, default permissivo ou fallback. |

## Falhas de autorização

O modelo de domínio distingue:

1. identidade ausente;
2. UserAccount inativa;
3. RoleAssignment inexistente, não vigente, suspensa, revogada ou expirada;
4. Permission ausente;
5. Scope incompatível;
6. tenant incompatível;
7. resource fora do alcance;
8. condição de negócio, estado, módulo ou elegibilidade não satisfeitos.

Essa distinção orienta diagnóstico autorizado e audit, mas a resposta externa pode ser minimizada para impedir enumeração/vazamento cross-tenant. Nenhum código HTTP é escolhido.
