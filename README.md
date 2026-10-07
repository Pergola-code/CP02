# Checkpoint 02 — Regressão linear com PIB e Índice ABCR

**Integrantes**

| Nome | RM |
|---|---|
| Felipe Macedo | RM 570990 |

Atividade da disciplina MLAM (1CCPJ, 2026). Enunciado original: [prof-atritiack/CHECKPOINT_02_MLAM_2026_2](https://github.com/prof-atritiack/CHECKPOINT_02_MLAM_2026_2).

## A tarefa

Investigar se existe relação entre a atividade econômica brasileira e o fluxo de veículos nas rodovias, e construir um modelo de regressão linear em Python que estime o Índice ABCR a partir do índice de volume do PIB.

- O **PIB** é o valor dos bens e serviços finais produzidos no país em um período. O **índice de volume** acompanha a produção descontando a variação de preços; na série usada, a média de 1995 vale 100.
- O **Índice ABCR** mede o fluxo de veículos que passa pelas praças de pedágio das rodovias concedidas.

## Conteúdo do repositório

| Arquivo | O que é |
|---|---|
| `CP2_regressao_PIB_ABCR.ipynb` | Notebook com fontes, preparação dos dados, gráficos, modelo, métricas e conclusão |
| `dados/pib_trimestral_sidra1620.csv` | PIB: índice de volume trimestral, 2006–2025 (IBGE) |
| `dados/abcr_0826.xlsx` | Planilha histórica do Índice ABCR, como baixada do site |
| `dados/abcr_mensal_original_total_brasil.csv` | Série mensal extraída da planilha (gerada pelo notebook) |
| `dados/base_anual_pib_abcr.csv` | Tabela final `Ano`, `PIB_indice`, `ABCR_indice` (gerada pelo notebook) |

## Fontes

| Indicador | Fonte | Série |
|---|---|---|
| PIB | IBGE, [SIDRA — Tabela 1620](https://sidra.ibge.gov.br/tabela/1620) | Série encadeada do índice de volume trimestral (base: média 1995 = 100), Brasil, PIB a preços de mercado, sem ajuste sazonal |
| Fluxo de veículos | ABCR, [Índice ABCR](https://melhoresrodovias.org.br/indice-abcr_2/) → "Ver histórico" | Série original (sem ajuste sazonal), Brasil, fluxo total, número-índice mensal |

Consulta do PIB pela API do SIDRA (07/10/2026):
`https://apisidra.ibge.gov.br/values/t/1620/n1/all/v/583/p/200601-202504/c11255/90707`

## Passo a passo

1. **Pesquisa dos dados** — 20 anos completos em comum, 2006 a 2025, séries sem ajuste sazonal.
2. **Organização da base** — `PIB_indice` é a média dos 4 índices trimestrais; `ABCR_indice` é a média dos 12 índices mensais. O notebook confere se todos os trimestres e meses existem e se os valores são números-índice (não variações percentuais).
3. **Análise da relação** — gráfico de dispersão (PIB no eixo horizontal, ABCR no vertical) e correlação de Pearson.
4. **Treinamento** — `LinearRegression` do scikit-learn, `X = PIB_indice`, `y = ABCR_indice`. Treino com os 16 primeiros anos (2006–2021) e teste com os 4 últimos (2022–2025), sem embaralhar.
5. **Avaliação** — MAE, MSE e R² no teste, com tabela de valores observados e previstos.

## Decisões e justificativas

- **Média anual simples** para alinhar frequências diferentes (PIB trimestral, ABCR mensal), como pede o enunciado.
- **Corte cronológico manual** em vez de `train_test_split`, que embaralha por padrão e deixaria o modelo treinar com anos posteriores aos que ele prevê.
- **Série original, não dessazonalizada**, nas duas fontes: como a base é anual, a sazonalidade se cancela dentro do ano.
- **MAE e erro percentual lidos junto com o R²**: com só 4 pontos de teste, o R² oscila muito e pode ficar negativo mesmo com erros pequenos em pontos de índice.

## Resultados

Base anual (20 anos, 2006–2025) em `dados/base_anual_pib_abcr.csv`. Correlação de Pearson entre `PIB_indice` e `ABCR_indice`: **r = 0,964**.

Reta ajustada no treino (2006–2021): `ABCR = 1,2145 × PIB − 56,78` (R² no treino = 0,901).

| Métrica | Teste (2022–2025) |
|---|---|
| MAE | 4,95 |
| MSE | 25,94 |
| R² | 0,485 |

| Ano | PIB_indice | ABCR observado | ABCR previsto | Erro | Erro % |
|---|---|---|---|---|---|
| 2022 | 178,05 | 154,12 | 159,46 | +5,34 | +3,47% |
| 2023 | 183,82 | 163,49 | 166,47 | +2,98 | +1,82% |
| 2024 | 190,10 | 168,88 | 174,10 | +5,22 | +3,09% |
| 2025 | 194,45 | 173,12 | 179,38 | +6,26 | +3,62% |

## Conclusões

- **Há relação positiva e forte** entre PIB e fluxo de veículos (r = 0,964): anos de maior atividade econômica são anos de mais veículos nas rodovias pedagiadas.
- **As estimativas ficaram próximas dos dados reais, mas com viés.** O erro médio foi de 4,95 pontos de índice, cerca de 3% do nível do período, e nenhum ano errou mais de 3,6%.
- **O R² de 0,485 no teste parece baixo, mas precisa de contexto:** ele é calculado com 4 pontos que variam pouco entre si, então um erro de 5 pontos já consome metade da variância. MAE e erro percentual descrevem melhor a proximidade.
- **O modelo superestimou o fluxo nos quatro anos.** Erro sempre no mesmo sentido indica mudança de patamar: para um mesmo PIB, o fluxo recente é menor que antes (2013: PIB 174,26 e ABCR 153,59; 2021: PIB 172,83 e ABCR 144,99). Mudança de hábitos de deslocamento após a pandemia é uma hipótese, não algo que estes dados comprovem.
- **O teste é uma extrapolação:** os quatro anos têm PIB acima do máximo do treino (175,14 em 2014).
- **O treino contém quebras que uma reta não captura**, sobretudo 2020, quando o ABCR anual caiu 13,9% e o PIB 3,3%.
- **Correlação alta não demonstra causa e efeito.** Parte dela vem da tendência de crescimento comum às duas séries, e o Índice ABCR é um dos insumos usados pelo IBGE no cálculo do PIB.

## Dificuldades e soluções

| Dificuldade | Solução |
|---|---|
| A Tabela 1620 tem várias variáveis e setores | Consulta fixada na variável 583 e no setor 90707, conferindo a unidade "Número-índice" |
| PIB trimestral e ABCR mensal | Redução das duas séries para anual, com conferência de períodos completos |
| Planilha da ABCR com muitas séries e cabeçalho em várias linhas | Função de leitura que localiza datas e cabeçalho e informa a coluna escolhida |
| Células vazias na coluna de datas da planilha são lidas como `NaT`, que o Python trata como data, e a leitura perdia o cabeçalho | Filtro com `pd.notna` na identificação das datas |
| `train_test_split` embaralha os dados | Corte por posição com `iloc` |
| R² instável com 4 pontos | Interpretação apoiada em MAE e erro percentual |

## Como executar

```bash
pip install pandas scikit-learn matplotlib openpyxl
jupyter notebook CP2_regressao_PIB_ABCR.ipynb
```

No Google Colab: envie o notebook e a pasta `dados/` e use "Executar tudo".
