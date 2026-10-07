# Camadas e responsabilidades

## Interface / API

É o limite de entrada e saída do sistema. Adaptadores HTTP podem usar Gin, mas a camada permanece substituível e não constitui a aplicação inteira.

Responsabilidades:

- receber a representação externa e fazer parsing;
- validar forma, formato e campos permitidos;
- encaminhar uma intenção para o caso de uso correspondente;
- obter evidências de autenticação segundo mecanismo futuro e construir dados de identidade/contexto não confiáveis até validação;
- serializar resultados e mapear categorias de erro conceituais;
- propagar identificador de correlação, sem usá-lo como autoridade ou identidade.

Não decide elegibilidade, disponibilidade, grants, estados válidos, isolamento tenant, auditoria obrigatória nem política de retry.

## Application

Implementa casos de uso que representam commands e queries semânticos.

Responsabilidades:

- orquestrar carregamento e coordenação de agregados;
- resolver/validar TenantContext e efetuar autorização contextual;
- solicitar decisões de domínio e aplicar transições permitidas;
- definir a fronteira transacional conceitual;
- coordenar repositórios/portas de leitura e escrita;
- exigir e coordenar registros de auditoria;
- recolher fatos/eventos produzidos e encaminhar efeitos após persistência;
- traduzir falhas de portas em erros de aplicação sem mascarar erro ou produzir sucesso falso.

Application não depende de Gin, GORM, SQL ou SDK de provedor. Um caso de uso pode depender de contratos abstratos que Infrastructure implementa.

## Domain

Define entidades/conceitos, value concepts, agregados e suas invariantes, políticas de domínio, guardas, transições e fatos/eventos de domínio.

Domain:

- recebe instante/contexto necessários como dados explícitos;
- não consulta relógio global, banco, rede, ambiente ou provedor;
- não importa Gin, GORM, PostgreSQL, HTTP, WhatsApp, e-mail, Push, Redis ou AWS;
- não decide política jurídica ou local ainda marcada `OPEN DECISION`;
- não usa Permission como substituto de elegibilidade ou de condição de negócio.

## Infrastructure

Implementa adaptadores e detalhes substituíveis exigidos pelas portas da aplicação:

- persistência PostgreSQL via GORM ou outro adaptador futuro;
- mapeamento entre persistência e domínio;
- envio de notificações e acesso a serviços externos;
- relógio, geração de identificadores, armazenamento documental;
- execução de tarefas e entrega de eventos quando aprovadas;
- logging, métricas e tracing técnicos.

Infrastructure não redefine regras, autoriza operações por conta própria nem transforma falhas em resultado de sucesso.

## Regra de dependência

- Interface depende de contratos de Application.
- Application depende de Domain e das abstrações necessárias às suas operações.
- Domain depende apenas de seus próprios conceitos e de bibliotecas neutras se aprovadas.
- Infrastructure implementa abstrações consumidas por Application e pode referenciar Domain para mapear seus conceitos.
- Nenhuma camada interna depende de Gin; Domain não depende de GORM/PostgreSQL.

## Validação e erros entre camadas

- **Input validation:** Interface verifica estrutura e forma de entrada.
- **Authorization validation:** Application determina identidade, grants, scope, tenant e acesso ao recurso.
- **Business validation:** Domain avalia invariantes, estado, período, elegibilidade e regras configuradas.
- **Infrastructure failure:** falha de banco ou serviço externo é propagada como falha técnica explícita, não como validação de negócio.

| Categoria de erro | Origem/responsabilidade | Tratamento conceitual |
|---|---|---|
| Validation Error | Interface para formato; Domain/Application para restrições semânticas do command. | Não confundir campo malformado com regra de negócio. |
| Domain Error | Domain, quando estado ou invariantes impedem transição. | Application preserva a categoria e não persiste sucesso parcial. |
| Authorization Error | Application, a partir do sujeito, permission, scope, recurso e tenant. | Minimizar resposta externa e preservar contexto para audit/diagnóstico autorizado. |
| Application Error | Application, quando contexto, workflow ou decisão requerida impede o caso de uso. | Propagar decisão/falha; nunca usar resposta de sucesso alternativa. |
| Infrastructure Error | Adapter de Infrastructure. | Converter apenas para erro técnico conhecido, mantendo causa; não reclassificar como erro de domínio. |

Mapeamento para categorias de API e limites de exposição seguem [API errors](../api/errors.md); nenhum status HTTP é definido.
