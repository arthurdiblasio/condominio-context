# Regras de unidades, vínculos, veículos e pets

## Condomínio, bloco e unidade

- `UNIT-01` Uma unidade pertence a exatamente um condomínio.
- `UNIT-02` Um bloco pertence a exatamente um condomínio; uma unidade associada a bloco deve estar no mesmo condomínio.
- `UNIT-03` Relações históricas encerradas permanecem consultáveis conforme autorização e política de retenção; encerrar vínculo não equivale a apagar a relação histórica.
- A obrigatoriedade de bloco depende do tipo de propriedade e permanece aberta.

## Vínculos semânticos

- `UnitOwnership` representa propriedade/titularidade; não prova moradia nem concede conta ou papel.
- `UnitResidency` representa residência; pode incluir pessoa proprietária, locatária ou dependente conforme vínculo documentado, mas a regra de elegibilidade do vínculo fica a cargo da política local.
- `UnitTenancy` representa locação/ocupação; não prova propriedade e não implica automaticamente `UnitResidency`.
- Vínculos devem indicar a pessoa, unidade, condomínio e período aplicável. Datas futuras e encerramentos devem ser preservados como fatos, sem apagar histórico.
- Copropriedade não pode ser descartada por pressuposto de titular único. Pessoas diferentes podem ter vínculos de propriedade simultâneos; verificação, percentuais e restrições dependem de decisão.
- Sobreposição de residência, locação ou propriedade só é rejeitada se regra local/legal validada proibir aquele tipo de sobreposição. Não existe exclusividade universal definida.
- Proprietário não residente, residente não proprietário, locatário, dependente, múltiplas unidades e residência em múltiplas unidades são cenários não equivalentes. Permissão, direitos e limites relativos a cada um precisam de política aprovada.

## Veículo

- `Vehicle` deve ter vínculo contextual com uma `Person` responsável e com unidade/condomínio quando associado ao uso ou cadastro do condomínio.
- Uma pessoa pode ter mais de um veículo; um cadastro de veículo não deve ser usado como identidade da pessoa.
- Alterar responsável/unidade deve preservar o histórico anterior.
- Veículo temporário/visitante deve ser contextualizado à visita/autorização quando aplicável e não deve criar vínculo residencial.
- Status e identificação conceitual podem ser mantidos; duplicidade de placa, campos obrigatórios, garagem/vaga, compartilhamento entre unidades, veículos temporários e retenção são decisões abertas.

## Pet

- `Pet` é relacionado ao contexto de condomínio e unidade e a uma ou mais pessoas responsáveis, conforme política configurada.
- Atribuir ou encerrar responsabilidade não apaga histórico anterior.
- Não existe limite global de quantidade, espécie, raça ou área de circulação estabelecido.
- Cadastro obrigatório, múltiplos responsáveis, transferência de responsabilidade, saída/óbito e visibilidade são decisões de produto/localidade pendentes.

Ver [domain-model.md](../domain/domain-model.md) e [../../OPEN-DECISIONS.md](../../OPEN-DECISIONS.md).
