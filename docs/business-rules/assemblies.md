# Regras de assembleias e votação

## Conceitos separados

`Assembly` contém `AgendaItem`s. `Participant` representa relação com a assembleia; convocação/inscrição, presença, representação, elegibilidade, votação e voto registrado são fatos diferentes. `VotingEligibility` é avaliada no contexto de uma pauta. `Proxy` tem outorgante, representante, escopo e validade. `Quorum` e `Vote` dependem de regra aplicável.

## Regras estruturais

- `ASSEMBLY-01` Cada `AgendaItem`, elegibilidade, proxy e voto está associado ao condomínio e à assembleia corretos.
- `ASSEMBLY-02` Presença não implica direito de voto; ausência de registro de presença não deve ser convertida em presença presumida.
- `ASSEMBLY-03` Elegibilidade é avaliada por pessoa e pauta; resultado para uma pauta não se estende automaticamente a outra.
- `ASSEMBLY-04` Um voto só pode ser registrado para pauta em votação segundo regra vigente e após elegibilidade aplicável ser reconhecida.
- `ASSEMBLY-05` Proxy só representa dentro do escopo e período válidos reconhecidos para a assembleia/pauta; não é transferível automaticamente entre assembleias ou itens.
- `ASSEMBLY-06` Voto registrado, correção, invalidação e resultado devem permanecer auditáveis. Correção não deve apagar o registro original sem trilha.
- `ASSEMBLY-07` Quórum e resultado devem ser calculados segundo regra identificável; sem regra aprovada, o resultado não deve ser apresentado como deliberação validada.
- `ASSEMBLY-08` Convocado/convidado, participante, presente, representado, elegível, votante e pessoa com voto registrado não são categorias intercambiáveis.

## Ciclo conceitual

Ver [state machines de Assembly, AgendaItem, Participant, VotingEligibility, Proxy e Vote](../state-machines/communications-and-assemblies.md). Assembly e pauta têm ciclos separados; `VOTING` pertence a AgendaItem, não à Assembly. `Vote` é fato registrado, não ciclo `pending/open/closed`; presença, representação, elegibilidade, resultado e publicação também não são intercambiáveis. A abertura exige as condições previamente aprovadas; fechamento impede novos votos naquela etapa sem apagar os existentes. Critérios de convocação, presença, proxy, elegibilidade, início/fim, quórum, voto secreto/aberto, opções, ponderação, correção, reabertura e publicação são decisões humanas/legais.

## Decisões dependentes de legislação

Elegibilidade do proprietário, inadimplência, peso por unidade, representação, quórum, validade de convocação e efeitos da deliberação exigem revisão jurídica e regras do condomínio. Este documento não define aconselhamento ou regra legal universal.

Ver [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
