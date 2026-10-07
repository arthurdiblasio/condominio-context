# Módulos, features e financeiro opcional

## WF-23 — Habilitar/desabilitar Module ou Feature por condomínio

### Objetivo

Disponibilizar ou retirar capacidade para tenant sem conceder permissões e sem apagar histórico.

### Participantes

Ator de plataforma ou condomínio somente segundo autoridade aprovada; usuários do tenant são afetados pela disponibilidade, mas não ganham grants.

### Contexto/Tenant

Catálogo `Module`/`Feature` pode ser global; `CondominiumModule`/`CondominiumFeature` é específico de um tenant.

### Gatilho e pré-condições

Solicitação de habilitar/desabilitar; feature identificada, dependências/incompatibilidades conhecidas e authority conferida.

### Permissões necessárias

`module.read/enable/disable`, `feature.read/enable/disable` no scope Platform para catálogo ou Condominium/Feature para configuração local. Habilitação não substitui permission funcional em outras ações.

### Dados/conceitos envolvidos

`Module`, `Feature`, `CondominiumModule`, `CondominiumFeature`, configurações, dados históricos, `RoleAssignment`, `AuditLog`.

### Fluxo principal

1. Ator seleciona explicitamente tenant e capability.
2. Valida-se authority, scope e dependências declaradas, incompatibilidades e eventuais operações em andamento.
3. Habilita-se módulo/feature; disponibilidade muda, mas grants permanecem separados e necessários.
4. Na desabilitação, impede-se iniciar operações que dependem da capacidade segundo regra local aprovada.
5. Dados existentes, histórico e auditoria permanecem; não excluir ou alterar status de entidade por efeito colateral.
6. Reativação exige nova validação de configuração; não cria grants automaticamente.

### Fluxos alternativos/validações/transições

Lifecycle de referência: [CondominiumModule e CondominiumFeature](../state-machines/modules.md). Estados de disponibilidade propostos: `ENABLED` e `DISABLED`; `AVAILABLE` descreve catálogo global, e `PENDING` não é adotado sem fluxo de aprovação aprovado. Dependência ausente, conflito ou operações em andamento podem bloquear ou exigir transição controlada conforme policy.

### Eventos/notificações/auditoria/pós-condições

Module/Feature enable/disable requested/accepted/rejected e mudança de configuração. Notificar admins afetados conforme policy. Auditar ator, tenant, estado anterior/novo e dependências. Termina com capability no estado acordado e dados históricos intactos.

### Erros/negações

Permission ausente; tenant ambíguo; dependência não habilitada; incompatibilidade; capability desconhecida; operação dependente em estado impeditivo; tentativa de apagar dados.

### OPEN DECISIONS

OD-11/OD-17: autoridade, defaults, dependências, incompatibilidades, operações pendentes, acesso pós-desabilitação e grants necessários por feature.

---

## WF-24 — Lançar receita/despesa, fornecedor e prestação de contas (módulo opcional)

### Objetivo

Oferecer registro financeiro conceitual para prestação de contas, sem constituir ERP ou contabilidade oficial.

### Participantes

Ator do condomínio com permission financeira; syndic, condominium_admin ou property_manager são candidatos condicionais; leitor de relatório conforme grant independente.

### Contexto/Tenant

Módulo/Feature financeiro habilitado no Condominium selecionado. Receitas, despesas, fornecedores e documentos pertencem a esse tenant.

### Gatilho e pré-condições

Necessidade de registrar movimento, associar fornecedor/documento ou emitir relatório; módulo habilitado; permission e finalidade existentes.

### Permissões necessárias

`finance.income.manage`, `finance.expense.manage`, `finance.supplier.manage`, `finance.read`, `finance.report.read`, `finance.document.export` conforme ação. Feature ativa não concede acesso.

### Dados/conceitos envolvidos

`FinancialModule`, Module/Feature habilitada, `Income`, `Expense`, `Supplier`, `Category`, `FinancialDocument`, `FinancialReport`, `AuditLog`.

### Fluxo principal

Registrar receita ou despesa com dados aprovados; validar tenant, permission, categoria e documento conforme regras definidas; associar fornecedor quando aplicável; preservar alterações e disponibilizar relatórios somente para grants financeiros apropriados.

### Fluxos alternativos e validações

Módulo desabilitado: não aceitar novas operações financeiras; acesso a histórico depois de desativação depende de policy e permissions. Lançamento duplicado, correção, aprovação, fechamento contábil e documento incompleto exigem regras próprias antes de representar valores como oficiais.

### Eventos, notificações, auditoria e pós-condições

Income/Expense/Supplier/Document created/changed/corrected e report generated/exported. Notificações financeiras só conforme policy. Auditar qualquer mudança ou export sensível. Termina com registro/relatório interno marcado de forma compatível com nível de validação; não declarar conformidade contábil sem regra.

### Erros/negações

Tenant errado; módulo desabilitado; permission ausente; category/document inválido; valor/contexto fora da regra; tentativa de tratar como contabilidade oficial sem validação.

### OPEN DECISIONS

OD-12: escopo, campos, aprovações, categorias, documentos, relatórios, responsabilidades e validação contábil/legal.
