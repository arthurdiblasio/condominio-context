# Regras de identidade e conta

## Conceitos

- `Person`: pessoa do domínio; pode existir sem acesso ao produto.
- `UserAccount`: conta usada para autenticação/acesso e associada a uma `Person` conforme política aprovada.
- `Role`: definição de papel.
- `RoleAssignment`: atribuição contextual de papel à pessoa.
- `Permission`: capacidade específica autorizável.

## Regras

- `IDENTITY-01` A existência de `Person` não exige `UserAccount`.
- `IDENTITY-02` Uma conta não é vínculo de residência, propriedade, locação ou papel.
- `IDENTITY-03` Uma pessoa pode participar de diferentes condomínios sem criar automaticamente acesso para cada um.
- `IDENTITY-04` Ações de usuário autenticado são atribuídas à conta/ator e relacionadas à `Person` conforme a associação vigente no momento.
- `IDENTITY-05` Conta desativada não pode iniciar novas ações autenticadas; desativá-la não apaga a pessoa, vínculos nem auditoria.
- `IDENTITY-06` Alterar ou remover associação conta-pessoa não transfere histórico nem altera titularidade, residência ou locação.
- `IDENTITY-07` Conta suspensa, convite pendente, recuperação e reativação só têm efeitos quando políticas aprovadas definirem seus estados e autoridades.

## Ciclo de conta

Vocabulário de revisão: `invited → active → suspended/deactivated`, com possível reativação conforme política. Não é ainda uma máquina aprovada.

- Ativação exige associação válida a uma `Person` e conclusão dos requisitos de identidade definidos pelo produto.
- Suspensão ou desativação deve impedir novo acesso enquanto vigorar; não deve encerrar automaticamente relações de domínio.
- A desativação preserva histórico de ação e associação anterior conforme regras de privacidade/retention aprovadas.
- Convite expirado, rejeitado, transferência de conta, múltiplas contas por pessoa e conta compartilhada não têm regra aprovada.

## Decisões abertas

Cardinalidade entre Person e UserAccount; método e autoridade para criar/convidar/associar; critérios de ativação; motivos e procedimento de suspensão/desativação; recuperação e reativação; tratamento de conta comprometida. Não decidir aqui senha, OAuth, token ou provedor de identidade.

Ver [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
