# MVP — Análise de Risco, Custo e Tempo para Implementação de Nova Unidade

**Disciplina:** Sistemas de Suporte à Decisão  
**Autora:** Isadora Messenberg (231013387) 

---

## O problema

Uma empresa de armazenamento de dados na nuvem precisa decidir sobre a implementação de uma **nova unidade** (data center). A decisão envolve três dimensões que competem entre si:

- **Risco** — qual a probabilidade de falha dos equipamentos escolhidos?
- **Custo** — quanto custa adquirir, operar e repor a infraestrutura ao longo do tempo?
- **Tempo** — quando planejar manutenções preventivas e reposições?

Sem dados históricos, essas escolhas viram achismo. Este MVP mostra como usar dados operacionais reais para transformar a decisão em uma recomendação parametrizada e defensável.

---

## A base de dados

Foram utilizados **14 dias de dados operacionais da Backblaze** (janeiro/2017), disponibilizados publicamente no Kaggle. A Backblaze é uma empresa real de armazenamento em nuvem que publica diariamente o estado de todos os HDs dos seus data centers.

- **~1 milhão de linhas** (cada linha = 1 HD em 1 dia)
- **~73 mil HDs únicos** de diferentes modelos e capacidades
- **~90 atributos SMART** por HD (temperatura, setores defeituosos, horas ligado, erros de leitura etc.)
- **45 falhas** registradas no período
- **Variável-alvo:** `failure` (0 = operacional, 1 = falhou naquele dia)

A base é ideal para o problema: reflete exatamente o tipo de dado operacional que uma empresa de armazenamento usa para tomar decisões sobre expansão de capacidade.

---

## Hipóteses

1. **Nem todo modelo de HD tem o mesmo risco.** Existem diferenças significativas de AFR (Annualized Failure Rate) entre modelos, o que permite escolher os mais confiáveis para a nova unidade.
2. **Os atributos SMART antecipam falhas.** Sinais como setores realocados, temperatura e horas ligadas devem se destacar como preditores.
3. **O custo total (TCO) não é dominado apenas pelo preço de compra.** Reposições e downtime somam um valor comparável ao CAPEX ao longo de 3 anos.
4. **Manutenção preventiva vale a pena.** Substituir HDs em janelas planejadas reduz o custo de downtime frente a reagir a falhas imprevistas.

---

## Análises realizadas

### 1. Análise Descritiva — *o que aconteceu*
- Composição do parque de HDs (modelos, capacidades, participação relativa)
- Cálculo do **AFR por modelo** — a métrica-padrão da indústria de storage
- Segmentação por faixa de risco: baixo (<1%), médio (1–3%), alto (>3%)

### 2. Análise Preditiva — *o que vai acontecer*
- Modelo **Random Forest** treinado sobre 1M+ registros para prever falhas a partir dos sinais SMART
- Modelo **Regressão Logística** como baseline linear interpretável
- **Ranking de importância** dos atributos SMART — qual sinal monitorar primeiro
- Uso de `class_weight='balanced'` para lidar com o forte desbalanceamento (0,004% de falhas)

### 3. Análise Prescritiva — *o que fazer*
- **Dimensionamento** da nova unidade: quantos HDs de cada modelo comprar para atingir 5.000 TB úteis com fator de redundância 1,5x
- **TCO em 3 anos** = CAPEX (aquisição) + OPEX (reposições esperadas com base no AFR) + custo estimado de downtime
- **Ranking por TCO/TB útil** — recomenda o modelo que entrega o menor custo total por TB de capacidade
- **Cronograma de manutenção preventiva** a cada 6 meses, com estoque de segurança (spare) dimensionado

Todos os parâmetros do modelo prescritivo (preço por TB, custo de troca, custo por hora de downtime, capacidade alvo, horizonte de planejamento) são configuráveis no notebook — o mesmo código roda para qualquer cenário de expansão.

---

## Principais resultados

- **AFR varia mais de 100x entre modelos** no dataset — de 0% (HGST HMS5C4040ALE640) a mais de 50% em modelos residuais. Escolher bem o modelo é a alavanca de risco mais importante.
- **Atributos SMART mais informativos:** horas ligado (idade), temperatura, setores pendentes de realocação e erros não corrigíveis. Esses quatro sinais devem entrar no monitoramento diário da nova unidade.
- **O modelo preditivo com apenas 14 dias é ilustrativo** — em produção, seria treinado sobre 1–2 anos de histórico, como recomenda a própria Backblaze.
- **TCO total em 3 anos** para o modelo recomendado ficou dominado por CAPEX, com OPEX de reposição e downtime somando frações menores quando o AFR é baixo. Isso valida a hipótese de que priorizar confiabilidade compensa financeiramente.

---

## Conclusões

1. **Dados operacionais internos são a alavanca principal** para decisões de infraestrutura. Sem histórico de falhas, qualquer escolha de fornecedor é aposta.
2. **A recomendação final não depende do menor preço de aquisição**, e sim do menor TCO por TB útil entregue no horizonte de planejamento. O CAPEX baixo pode ser anulado por OPEX alto quando o AFR é ruim.
3. **A manutenção preventiva a cada 6 meses** é uma resposta razoável ao AFR observado, mas o próximo passo é usar o modelo preditivo para *disparar substituições individualmente*, quando a probabilidade prevista de falha de um HD específico passa de um limiar.
4. **Como sistema de suporte à decisão, o MVP atende três critérios essenciais:** é *transparente* (fórmulas explícitas), *parametrizável* (o decisor ajusta cenários) e *acionável* (entrega números concretos — modelo, quantidade, custo, cronograma).

---

# Próximos passos

- Estender a série histórica para 1–2 anos (mais falhas → modelo preditivo mais robusto)
- Integrar preços reais de fornecedores em vez do valor médio de mercado
- Rodar **análise de sobrevivência (Kaplan-Meier)** para curvas de risco por idade do HD
- Simular cenários com **Monte Carlo** variando AFR, preço e demanda
- Construir dashboard operacional que recebe leituras SMART em tempo real e dispara alertas

---

## 📁 Estrutura do repositório

```
├── README.md                           # este arquivo
├── MVP_NovaUnidade_DataCenter.ipynb    # notebook com todas as análises
└── dados/                              # base do Kaggle (não versionada, baixar do link abaixo)
```

**Fonte dos dados:** [Backblaze Hard Drive Data — Kaggle](https://www.kaggle.com/datasets/backblaze/hard-drive-test-data)
