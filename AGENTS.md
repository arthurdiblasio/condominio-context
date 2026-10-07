# AGENTS.md

## Objetivo

Este repositório define a fundamentação documental do produto Condomínio. Ele deve orientar futuras implementações em `condominio-api`, `condominio-admin` e `condominio-mobile` sem criar regras de negócio improvisadas.

## Regras para qualquer IA

1. Ler este repositório antes de alterar regras de negócio.
2. Nunca inventar regra não documentada.
3. Procurar entidades existentes antes de criar novas.
4. Procurar workflows existentes antes de modificar processos.
5. Respeitar permissões e governança de acessos.
6. Respeitar multi-tenancy e isolamento entre condomínios.
7. Registrar decisões novas e ambíguas em `OPEN-DECISIONS.md`.
8. Não alterar regras sem documentação formal.
9. Separar domínio de implementação.
10. Não criar regra jurídica sem validação.

## Restrições de implementação

A IA não deve criar:

- Go
- React
- Flutter
- SQL
- migrations
- endpoints
- código de aplicação
- estruturas de banco de dados

A IA deve priorizar:

- documentação em Markdown;
- diagramas Mermaid quando úteis;
- estruturas declarativas em JSON/YAML quando isso facilitar entendimento;
- clareza conceitual e semântica de domínio.

## Política de decisão

- Quando uma regra não estiver definida, registrar como `OPEN DECISION`.
- Quando houver ambiguidade, documentar alternativas e escolher a opção mais neutral, sem transformar hipótese em regra.
- Nunca assumir que existe apenas um condomínio.
- Validar todas as interações em escopo de condomínio, unidade, pessoa e permissão.
- Tratar `User`, `Resident`, `Owner`, `Tenant`, `Syndic` e demais papéis como conceitos separáveis.

## Áreas de responsabilidade

A documentação deve manter consistência em:

- domínio;
- regras de negócio;
- permissões;
- workflows;
- notificações;
- decisões abertas;
- auditoria e compliance.

## Produção de documentação

- Usar terminologia consistente em todo o repositório.
- Manter explicação de estados, transições e papéis.
- Explicitar escopo e limitações dos conceitos.
- Registrar gatilhos relevantes para notificação e auditoria.
- Incluir referências cruzadas entre documentos quando apropriado.

## Status esperado

Este repositório deve permanecer em estado de documentação de produto, não de implementação técnica.
