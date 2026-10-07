# Modelo temporal arquitetural

## Propósito e limites

Este documento define distinções temporais necessárias para modelar validade, agenda, expiração e fatos. Não escolhe biblioteca, banco, serialização, formato de timestamp, timezone provider ou política de negócio. Regras específicas de reservas, acesso, assembleia e vínculos continuam sujeitas a OD-01/04/05/09.

## Tipos conceituais

| Tipo | Significado | Exemplos de uso |
|---|---|---|
| `INSTANT` | Um ponto único na linha do tempo, independente de calendário local. Deve ser comparável globalmente e preservar o instante ocorrido. | Registro real de ENTRY/EXIT, recebimento/retirada de Package, audit, alteração de grant. |
| `LOCAL DATE` | Data civil sem horário ou instante implícito. | Dia de assembleia, data de vencimento civil, quando a regra não exige hora. |
| `LOCAL TIME` | Hora de relógio sem data/instante. | Horário de funcionamento recorrente, se política local assim definir. |
| `DATE-TIME WITH TIMEZONE` | Data/hora civil associada a zona identificada, necessária para resolver o instante conforme regra de calendário. Offset isolado não substitui necessariamente a zona para recorrência/regras locais. | Início agendado de assembleia/reserva em contexto condominial. |
| `DURATION` | Quantidade de tempo decorrido, distinta de calendário civil. | Tempo transcorrido entre dois instants ou prazo em horas, se policy assim definir. |
| `INTERVAL` | Período delimitado por começo e fim, com semântica de fronteira especificada por regra. | Janela de reserva, período de vigência ou bloqueio. |
| `VALIDITY WINDOW` | Janela durante a qual um grant/autorização/convite/operação pode ser usado. | `RoleAssignment`, `AccessAuthorization`, convite de conta. Não implica consumo/uso. |

Um valor temporal deve ter tipo declarado; não converter implicitamente `LOCAL DATE`, `LOCAL TIME` ou `DATE-TIME WITH TIMEZONE` em `INSTANT`. `INSTANT` registra ocorrência; horário agendado expressa intenção civil e zona.

## Zona de calendário

Quando uma regra de condomínio depender de data/hora local, o caso de uso usa o timezone configurado para aquele Condominium como contexto de calendário. Não presumir timezone do servidor, navegador, telefone, conexão ou processo. Cada tenant pode ter sua própria zona.

Fonte da zona, identificador/formato, alteração histórica da zona e tratamento de hora local inexistente ou repetida durante transições de horário permanecem questões técnicas/operacionais a definir antes de implementar agenda local. A zona precisa permanecer explícita na interpretação da hora; converter para UTC não apaga a referência civil original quando ela for necessária para auditoria.

## Fronteiras e validade

- Cada workflow define explicitamente se os limites inicial/final são inclusivos ou exclusivos; não existe convenção global presumida neste documento.
- Validação é feita contra um único instante de referência capturado para a decisão, não múltiplas leituras implícitas do relógio durante o mesmo comando.
- A fonte desse instante é fornecida ao caso de uso por uma abstração temporal; o Domain não consulta relógio de sistema, timezone de host ou serviço externo.
- Uma tarefa de expiração pode materializar condição derivada, mas não antecipa nem posterga validade; só executa transição automática já permitida pela regra.
- Instante do fato, instante de decisão e janela de validade são fatos/valores distintos quando necessários para explicar causalidade.
- Correção de horário/fato mantém o valor original referenciado; não reescreve a ocorrência como se o novo instante sempre tivesse existido.
- Divergência de relógio/fonte em evento físico requer regra de reconciliação e audit, não ajuste silencioso.

## OPEN QUESTIONS TÉCNICAS / DE PRODUTO

- Produto/governança: timezone do condomínio, limites de janela, validade de convites/autorização/reservas e resolução de ambiguidade de horário dependem das decisões de domínio aplicáveis.
- Arquitetura/implementação: representação da zona, fonte confiável de instante, precisão, clock abstraction, persistência e serialização serão propostas em decisão técnica após os requisitos de workflow.
- Não foi escolhido comportamento inclusivo/exclusivo comum, pois isso pode alterar validade e alocação do recurso.
