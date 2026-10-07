# Visibilidade de dados pessoais e recursos

## Princípio

Autorização de leitura não é apenas `person.read`. A decisão deve considerar sujeito, recurso/pessoa consultada, condomínio, unidade/vínculo, campos, finalidade, ação e escopo. A matriz não é um catálogo de campos nem autorização geral para consultar pessoa.

## Regras conceituais

- `VISIBILITY-01` Leitura exige finalidade funcional e permission apropriada; vínculo com condomínio, unidade ou papel pode ser condição adicional.
- `VISIBILITY-02` Person não pode consultar dados de outra pessoa somente por morarem no mesmo condomínio.
- `VISIBILITY-03` Owner/resident/tenant podem acessar informações próprias ou da unidade relacionada somente nos limites de policy e vínculo válido.
- `VISIBILITY-04` Portaria vê somente atributos necessários à decisão operacional presente (por exemplo nome, unidade, identificação mínima e condição da autorização, se política aprovar).
- `VISIBILITY-05` Acesso de portaria não implica acesso a finanças, documentos de titularidade, dados pessoais completos, auditoria completa ou histórico integral da pessoa.
- `VISIBILITY-06` Admin/syndic não recebe automaticamente visibilidade irrestrita; cada finalidade e permission deve ser definida.
- `VISIBILITY-07` Financeiro, voto secreto, documentos de identidade, evidências e auditoria são áreas de sensibilidade elevada e requerem permissions específicas.
- `VISIBILITY-08` Dados cross-tenant não se tornam visíveis por ser a mesma Person, staff externo ou property manager; requer autorização de cada tenant ou grant cross-tenant específico.
- `VISIBILITY-09` Respostas e relatórios devem limitar exposição aos dados necessários, inclusive quando a permission da ação existe.

## Exemplo conceitual para portaria

| Dado | Necessidade de portaria | Modelo inicial |
|---|---|---|
| Nome usado na visita/autorização | Pode ser necessário à conferência. | `CONDITIONAL`, conforme operação e política. |
| Unidade de destino/responsável | Pode ser necessária ao encaminhamento. | `CONDITIONAL`, apenas no tenant correto. |
| Validade/status da autorização | Necessária para conferir permissão. | `CONDITIONAL`, limitada a autorização relevante. |
| Dados financeiros | Sem finalidade operacional padrão identificada. | `DENY` por padrão. |
| Documentos pessoais completos | Não necessários por padrão à portaria. | `DENY` por padrão; exceção requer decisão e scope específico. |
| Histórico completo da pessoa/acessos | Não necessário por padrão à conferência atual. | `DENY` por padrão; acesso excepcional precisa grant/auditoria. |

## Decisões necessárias

Definir conjunto de campos por papel/workflow, finalidade, dados sensíveis, exportação e regra para dados de unidade compartilhados. Não definir base legal ou períodos de retenção sem revisão competente.
