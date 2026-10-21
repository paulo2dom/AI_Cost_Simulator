# Simulador de Custo de IA 2028

![Vista geral do simulador](docs/screenshot.png)

## O que é

O Simulador de Custo de IA 2028 é uma ferramenta interativa que projeta, mês a mês, o custo de modelos de linguagem (LLM) de uma empresa de software entre **novembro de 2026 e dezembro de 2028**.

Junta numa só página as variáveis que mais vão pesar nas faturas de IA nos próximos dois anos:

- o preço por milhão de tokens, que pode descer, estabilizar ou subir;
- o volume de tokens gerado por agentes autónomos e por janelas de contexto de 100k+;
- a alternativa de servir parte da carga em GPUs NVIDIA Blackwell próprias;
- o efeito do custo da energia e da sustentabilidade no preço da nuvem.

Com estes dados, o simulador mostra quando e porquê a despesa com IA deixa de ser sustentável para o negócio.

## Propósito

Este simulador partilha uma **base de cálculo aberta** sobre o custo da IA generativa numa empresa de software. Serve para fazer simulações com diferentes valores e testar hipóteses. O objetivo é ajudar cada pessoa que o use a pensar no tema e a construir a sua própria opinião crítica sobre a evolução destes custos.

> **Os valores calculados destinam-se a investigação, não a orçamentos.** Não devem ser usados diretamente como previsão ou orçamento, mas a lógica de cálculo e os pressupostos podem apoiar quem esteja a preparar um.

Na prática, o simulador permite:

- **Perceber a mecânica do custo**, ou seja, como o preço por token, o volume, a cache, os agentes e o contexto longo se combinam numa fatura.
- **Testar hipóteses sobre o futuro**, como uma descida contínua de preços, uma subida depois dos IPOs dos laboratórios frontier ou um choque energético, e comparar o efeito de cada uma.
- **Questionar os próprios pressupostos.** A análise de sensibilidade mostra quais pesam mais no resultado e, portanto, quais merecem ser validados com dados reais.
- **Explorar limites**, isto é, em que condições o custo de IA começa a pôr em causa a margem de uma empresa de software.
- **Comparar abordagens** (API, aluguer ou compra de GPUs), vendo os custos e as limitações de cada uma.

Todos os pressupostos e fórmulas estão expostos neste guia e no código, para que possam ser verificados, contestados e melhorados.

### Questões em aberto

Três questões motivaram este simulador. Nenhuma tem resposta certa, e é para as explorar que os cenários existem:

1. **O volume vai crescer mais depressa do que o preço desce?** Um agente que executa 30 passos reenvia o contexto acumulado em cada passo, por isso pode consumir centenas de vezes mais tokens do que um pedido de chat. Assim, o custo por tarefa pode subir mesmo que o preço por token continue a descer.
2. **Os preços atuais são sustentáveis?** Há indicações de que os laboratórios frontier operam com prejuízo ou margens apertadas. A entrada em bolsa pode trazer pressão para subir margens, ou a concorrência pode continuar a empurrar os preços para baixo.
3. **Quanto da restrição energética chega ao preço?** O défice de ligação à rede, os prazos longos para equipamento elétrico e a subida da eletricidade junto dos grandes centros de dados podem refletir-se no preço da nuvem, com intensidade incerta.

### Para quem

Para qualquer pessoa que queira explorar o tema com números, por exemplo:

- quem acompanha a área financeira ou de controlo de gestão e quer perceber a ordem de grandeza destes custos;
- responsáveis de produto e de engenharia que pensam em arquiteturas de agentes, mix de modelos e uso de contexto longo;
- equipas de infraestrutura e FinOps que querem comparar API, aluguer e GPUs próprias;
- equipas de governação de IA e de sustentabilidade interessadas na relação entre IA, energia e emissões;
- estudantes, investigadores e curiosos que queiram pôr à prova as suas próprias hipóteses.

## Como funciona

O ecrã tem duas zonas. À esquerda ficam os **pressupostos**, agrupados em seis secções. À direita ficam os **resultados**, que se atualizam sempre que um valor muda. Não há botão de calcular.

| Resultado | Pergunta a que responde |
|---|---|
| **Fatura mensal · dez/2028** | Quanto vou pagar por mês no fim do horizonte, e quanto isso representa da receita? |
| **Indicadores** | Qual o custo acumulado, o volume de tokens por dia, o preço médio por milhão de tokens, as margens e o custo de cada execução de agente? |
| **Pontos de ruptura** | Em que mês passo o alerta, a margem bruta mínima, a margem operacional zero e o orçamento? Quanto podem os preços subir antes de dar prejuízo? |
| **Comparação de cenários** | Como mudam as datas de ruptura entre deflação, base, pós-IPO e choque energético? |
| **Fatura por origem** | Que parte do custo vem do uso interativo, do contexto longo e dos agentes? |
| **IA em % da receita** | Quando cruza cada cenário os limiares de alerta e de ruptura? |
| **Volume diário de tokens** | Como evolui o consumo, e que parte é dos agentes? |
| **Índice de preço** | Que trajetória de preço por token assume cada cenário? |
| **Só API vs. híbrido** | Compensa migrar parte da carga para GPUs Blackwell, e a partir de quando? |
| **Sensibilidade (tornado)** | Que pressuposto, se estiver errado, mais altera o custo acumulado? |
| **Mapa de ruptura** | Que combinações de preço e volume levam a operação a prejuízo? |
| **Projeção mensal** | Que valores tem cada mês? A tabela pode ser copiada para Excel. |

### Estados

Cada mês recebe um estado, avaliado do mais grave para o menos grave:

| Estado | Condição |
|---|---|
| **Ruptura operacional** | A margem operacional fica negativa: o custo de IA consome todo o resultado. |
| **Ruptura de margem** | A margem bruta fica abaixo do mínimo definido (por defeito 60%, abaixo do habitual em SaaS). |
| **Alerta** | O custo de IA passa a percentagem da receita definida como limiar (por defeito 15%). |
| **Saudável** | Nenhuma das condições anteriores. |

## Pressupostos

Os valores por defeito descrevem uma **empresa SaaS fictícia de média dimensão**. Os preços de API e de GPU são referências de outubro de 2026. Todos os valores são editáveis, e cada campo aceita um valor exato ou ajuste por cursor.

### 1. Cenários de preço

O preço por token segue um **índice** (100 = outubro de 2026) que combina três efeitos:

- uma **tendência anual antes do IPO** dos laboratórios frontier;
- um **salto no mês do IPO** e uma **tendência depois do IPO**, que representam a pressão dos investidores por margem;
- um **fator energia**: a subida anual da eletricidade nos centros de dados, multiplicada pelo peso da energia no custo de inferência.

| Parâmetro | Deflação contínua | Base | Alta pós-IPO | Choque energético |
|---|---:|---:|---:|---:|
| Tendência antes do IPO (%/ano) | −40 | −20 | −10 | 0 |
| Salto no IPO (%) | 0 | 0 | +30 | +10 |
| Tendência após o IPO (%/ano) | −35 | −15 | 0 | +10 |
| Eletricidade de datacenter (%/ano) | 2 | 4 | 5 | 15 |
| Peso da energia na inferência (%) | 10 | 15 | 15 | 35 |

O que cada cenário representa:

- **Deflação contínua:** a descida histórica mantém-se, com concorrência intensa, incluindo modelos abertos e chineses, e eficiência de hardware.
- **Base:** a descida abranda à medida que os ganhos fáceis se esgotam.
- **Alta pós-IPO:** os laboratórios deixam de subsidiar preços depois de entrarem em bolsa.
- **Choque energético:** a escassez de energia e de capacidade nos centros de dados torna-se o fator dominante e o seu custo passa para o cliente.

Pressupostos globais:

- **IPO:** junho de 2027.
- **Prémio de região UE / energia verde:** 10%, o custo adicional de endpoints europeus ou de regiões com energia renovável.
- **Câmbio:** 1,16 USD por euro. Os preços de API e de GPU são introduzidos em dólares e os resultados aparecem em euros.

### 2. Empresa e limiares

| Parâmetro | Defeito | Nota |
|---|---:|---|
| Receita mensal | 2,0 M€ | Cerca de 24 M€ de receita recorrente anual |
| Crescimento da receita | 30%/ano | |
| COGS sem IA | 18% da receita | Alojamento, suporte, licenças |
| Opex | 55% da receita | I&D, vendas e marketing, G&A |
| Alerta | IA > 15% da receita | |
| Margem bruta mínima | 60% | |
| Orçamento mensal de IA | 600 k€ | |

### 3. Uso interativo (copilotos, chat)

| Parâmetro | Defeito |
|---|---:|
| Utilizadores ativos | 8 000 |
| Crescimento de utilizadores | 3%/mês |
| Pedidos por utilizador | 12/dia |
| Contexto de um pedido normal | 8k tokens |
| Pedidos de contexto longo | 10% |
| Contexto de um pedido longo | 150k tokens |
| Tokens de saída por pedido | 800 |
| Crescimento anual do contexto | 25% |
| Taxa de cache | 50% |

O **crescimento do contexto** representa a tendência de enviar cada vez mais documentos, histórico e código em cada pedido. Com 25% ao ano, um pedido de 150k tokens passa os 200k durante o horizonte e começa a pagar a sobretaxa de contexto longo.

### 4. Agentes autónomos

| Parâmetro | Defeito |
|---|---:|
| Execuções por dia | 2 000 |
| Crescimento das execuções | 8%/mês |
| Teto de execuções | 60 000/dia |
| Passos por execução | 30 |
| Contexto inicial | 20k tokens |
| Contexto acrescentado por passo | 3k tokens |
| Saída por passo | 600 tokens |
| Taxa de cache | 75% |
| Receita por execução | 1,50 € |

Em cada passo, o agente reenvia o contexto inicial mais tudo o que acumulou nos passos anteriores. Com os valores por defeito, uma execução consome cerca de **1,9 milhões de tokens de entrada**. Duplicar o número de passos mais do que duplica o consumo. A taxa de cache dos agentes é mais alta do que a do uso interativo porque o prefixo da conversa se repete de passo para passo.

### 5. Mix de modelos e preços de API

Valores em USD por milhão de tokens. As quotas são normalizadas automaticamente, por isso não precisam de somar 100.

| Escalão | Quota | Entrada | Cache | Saída | Referência (out/2026) |
|---|---:|---:|---:|---:|---|
| Frontier | 25% | 4,00 | 0,20 | 20,00 | Claude Opus 5.5 |
| Intermédio | 50% | 2,00 | 0,20 | 10,00 | Claude Sonnet 5.5 |
| Económico | 25% | 0,30 | 0,03 | 2,50 | Gemini 3.5 Flash-Lite |

- **Tráfego em batch:** 15%, com 50% de desconto (prática comum na OpenAI, Anthropic, Google e Mistral).
- **Sobretaxa acima de 200k tokens:** 50%, aplicada aos pedidos cujo contexto passa esse limite. Alguns fornecedores cobram o dobro na entrada a partir de 200k.

### 6. Infraestrutura própria (NVIDIA Blackwell)

Opcional. Quando está ligada, uma fração do tráfego não-frontier passa para modelos abertos servidos em GPUs próprias, em compra ou em aluguer. As margens e as rupturas passam a usar o custo híbrido.

| Plataforma | Capex all-in/GPU | Potência/GPU (sistema) | Débito (tokens eq/s) | Aluguer |
|---|---:|---:|---:|---:|
| B200 (HGX/DGX 8 GPU) | 65 k USD | 1,4 kW | 12 000 | 5,50 USD/h |
| **B300 (HGX/DGX 8 GPU)** (defeito) | 80 k USD | 1,7 kW | 18 000 | 6,94 USD/h |
| GB300 NVL72 (rack) | 60 k USD | 1,9 kW | 24 000 | 5,74 USD/h |

| Parâmetro | Defeito | Parâmetro | Defeito |
|---|---:|---|---:|
| Modelo de aquisição | Compra | Intensidade carbónica | 110 gCO₂/kWh |
| Utilização média | 55% | Preço do carbono | 80 €/t |
| Amortização | 36 meses | Preço do hardware | −10%/ano |
| Manutenção e colocation | 12% do capex/ano | Ganho de eficiência | +35%/ano |
| Eletricidade | 0,13 €/kWh | Tráfego migrável | 35% |
| PUE | 1,25 | Equipa MLOps | 45 k€/mês |
| Peso de 1 token de saída | 10 tokens de entrada | Mínimo | 1 nó (8 GPUs) |

O **capex all-in** inclui servidor, rede e armazenamento repartidos por GPU, e não só a GPU isolada. A **intensidade carbónica** por defeito reflete uma rede com muita energia renovável, como a portuguesa. O **ganho de eficiência** representa a chegada de novas gerações (Blackwell Ultra, Rubin).

## Como o modelo calcula

**Volume diário**

- Interativo: `utilizadores × pedidos/dia × contexto`, com o contexto a crescer `(1 + crescimento)^(meses/12)`.
- Agentes: `execuções × Σ (contexto inicial + incremento × passo)` sobre todos os passos, mais `execuções × passos × saída por passo`. As execuções crescem a uma taxa mensal composta até ao teto.

**Custo de API**

```
custo = [ I × ((1 − cache) × p_entrada + cache × p_cache)
        + I_>200k × (1 − cache) × p_entrada × sobretaxa
        + O × p_saída × (1 + sobretaxa × I_>200k / I) ]
        × índice de preço × (1 + prémio UE) × (1 − batch × 50%) ÷ câmbio
```

`I` são os tokens de entrada e `O` os de saída. Os preços são a média ponderada pelas quotas dos três escalões.

**Índice de preço**

```
índice(t) = (1 + tendência_pré)^(min(t, IPO)/12)
          × [se t ≥ IPO] (1 + salto_IPO) × (1 + tendência_pós)^((t − IPO)/12)
          × (1 + peso_energia × ((1 + subida_eletricidade)^(t/12) − 1))
```

**Infraestrutura própria**

- Custo por GPU (compra): `capex/amortização + capex × manutenção/12 + kW × PUE × 730 h × (€/kWh + gCO₂/kWh × €/t ÷ 10⁶)`.
- Custo por GPU (aluguer): `preço/hora × 730 × fator energia`.
- Capacidade: `débito × (1 + eficiência)^(t/12) × utilização × 3600 × 730`.
- Tokens equivalentes: tokens sem cache × 1, tokens em cache × 0,1, tokens de saída × peso de saída.
- Custo híbrido: `custo de API − custo evitado + GPUs × custo por GPU + equipa`. O custo evitado usa o preço médio dos escalões intermédio e económico.

**Margens**

- `receita = receita base × (1 + crescimento)^(t/12) + execuções × receita por execução`
- `margem bruta = 1 − COGS sem IA − custo de IA / receita`
- `margem operacional = margem bruta − opex`
- `folga de preço = receita × (1 − COGS − opex) / custo de IA − 1`: quanto os preços ainda podem subir antes de a margem operacional ficar negativa.

## O que mostram os valores por defeito

Com os pressupostos por defeito:

- O volume passa de cerca de **6 para 49 mil milhões de tokens por dia**, e os agentes chegam a 85% do total.
- No **cenário Base** não há ruptura até dez/2028, mas o custo de IA passa o alerta de 15% da receita em nov/2028.
- Na **Alta pós-IPO** o alerta chega em dez/2027 e a ruptura de margem bruta em ago/2028.
- No **Choque energético** a margem bruta rompe em jul/2028 e a margem operacional fica negativa em nov/2028.
- A **estratégia híbrida** com B300 fica mais barata do que usar só API a partir de set/2027.

Estes números ilustram a mecânica do modelo para uma empresa fictícia. Não são uma previsão.

## Considerações

### Na interpretação

- **Preço por token e custo por tarefa são coisas diferentes.** Um modelo mais caro por token que resolve a tarefa em menos passos pode sair mais barato. O simulador trabalha com tokens: se mudar de modelo, ajuste também os passos e o contexto.
- **Os agentes dominam a fatura.** Na maioria das configurações, o crescimento das execuções, os passos por execução e o contexto por passo pesam mais do que o preço por token. A sensibilidade mostra isto: as barras mais longas costumam ser a cache dos agentes e os passos por execução.
- **A cache é a alavanca mais barata.** Subir a taxa de cache dos agentes de 75% para 90% pode cortar uma parte significativa do custo sem mudar de fornecedor. Estruturar os prompts com prefixos estáveis tem impacto financeiro direto.
- **O contexto longo tem um custo escondido.** Quando o contexto médio passa os 200k tokens, a sobretaxa aplica-se ao pedido inteiro. Resumir histórico, recuperar só os excertos relevantes (RAG) e limitar o contexto evitam este degrau.
- **Os limiares são decisões de gestão.** Uma margem bruta mínima de 60% e um alerta de 15% são pontos de partida. Ajuste-os ao perfil de margem da empresa e ao que o mercado espera.
- **A data de ruptura importa mais do que o valor exato.** Com tanta incerteza, o útil é saber se a ruptura acontece em 2027 ou em 2028 em cada cenário, e quanto tempo há para agir.

### Na infraestrutura própria

- **Utilização é tudo.** Uma GPU a 30% de utilização custa quase o dobro por token do que a 55%. Cargas irregulares favorecem a API ou o aluguer.
- **Há uma escala mínima.** Um nó de 8 GPUs e uma equipa MLOps são custos fixos. Abaixo de um certo volume, o modelo híbrido sai mais caro do que usar só API, e o simulador mostra o mês em que isso se inverte.
- **Qualidade.** Os modelos abertos não substituem os modelos frontier em todas as tarefas. A fração migrável deve vir de testes reais de qualidade, não do custo.
- **Prazos e risco tecnológico.** O hardware tem prazos de entrega, exige refrigeração líquida (B300, NVL72) e desvaloriza com cada nova geração. O aluguer reduz este risco à custa de um preço por hora mais alto.

### Em governação, soberania e sustentabilidade

- **Dependência de fornecedor.** Concentrar a carga num único escalão frontier expõe a empresa ao cenário pós-IPO. Um mix com alternativas reduz essa exposição.
- **Proteção de dados e soberania.** Endpoints na UE e infraestrutura própria têm um prémio de custo (o parâmetro de região UE), mas podem ser exigidos pelo RGPD, pela NIS2 ou por contratos com clientes.
- **Energia e emissões.** Com infraestrutura própria, o simulador estima as emissões mensais (tCO₂e) a partir da potência, do PUE e da intensidade carbónica da rede. Com API, as emissões ficam do lado do fornecedor, mas o custo de energia continua a chegar via preço.

## Limitações

- **Débito por GPU, capex all-in e peso da energia** são estimativas de ordem de grandeza. O débito real depende do modelo aberto, da quantização, da relação entre entrada e saída e do software de serviço (vLLM, TensorRT-LLM, SGLang). Use cotações e benchmarks próprios antes de decidir.
- O custo de infraestrutura própria usa, em cada mês, o hardware e o preço desse mês. Não modela uma frota comprada numa data e que depois envelhece, nem prazos de entrega.
- O mapa preço × volume e a sensibilidade mantêm a receita constante. Na prática, parte do aumento de custo pode ser repassada ao cliente.
- Não inclui taxas de escrita em cache, limites de pedidos (*rate limits*), compromissos de volume nem descontos negociados.
- Os cenários são hipóteses exploratórias, não previsões. Os resultados servem para investigação e reflexão, não para serem usados diretamente como orçamento, e o simulador não constitui aconselhamento financeiro.

## Utilização prática

- **Começar pelo cenário Base** com dados reais da empresa (receita, utilizadores, execuções de agente) e só depois comparar com os outros cenários.
- **Mudar um pressuposto de cada vez** para perceber o efeito isolado nas datas de ruptura.
- **Copiar tabela** exporta a projeção mensal para colar em Excel.
- **Repor valores** volta aos pressupostos por defeito.
- Os valores introduzidos ficam guardados apenas no browser de quem os introduz e não são enviados para nenhum servidor.

## Fontes dos valores de referência

Valores consultados em outubro de 2026:

- Preços de API por milhão de tokens (6/out/2026): [ActuIA](https://www.actuia.com/en/news/ai-api-pricing-in-october-2026-050-to-50-for-output/)
- Preço por hora de B200 e B300 em nuvem (8/out/2026): [DEV Community](https://dev.to/viswaretas_kotra/gpu-cloud-pricing-october-2026-what-an-h100-h200-b200-or-b300-costs-per-hour-5a25); GB300: [ComputePrices](https://computeprices.com/gpus/gb300)
- Preços de compra de B200/B300, DGX e NVL72: [Tech Insider](https://tech-insider.org/nvidia-blackwell-gpu-pricing/)
- Subsídio de preços e pressão pré-IPO: [Axios](https://www.axios.com/2026/03/12/ai-models-costs-ipo-pricing), [CNBC](https://www.cnbc.com/2026/05/20/cheap-ai-could-derail-openai-and-anthropics-ipos.html)
- Restrições energéticas, PUE e défice de rede: [Tech Insider](https://tech-insider.org/ai-data-center-power-crisis-2026/), [CNBC / Goldman Sachs](https://www.cnbc.com/2026/02/12/electricity-price-data-center-ai-inflation-goldman.html)
