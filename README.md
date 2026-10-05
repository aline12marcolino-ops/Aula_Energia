# Aula energia
# CP02 — APIs de energia renovável e aprendizado de máquina

Duas tarefas, cada uma com **três algoritmos comparados**:

1. **Classificação (ANEEL):** classificar a fonte de um empreendimento (Solar, Eólica ou Hidráulica) a partir de potência e localização.
2. **Regressão (Open-Meteo):** estimar a radiação solar horizontal (W/m²) em Petrolina (PE) a partir de variáveis meteorológicas e da hora.

## Fontes e período dos dados

| Dado | Fonte | Consulta | Linhas |
|---|---|---|---|
| Empreendimentos de geração | [ANEEL — SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel) (API CKAN/DataStore) | `UFV`, `EOL`, `UHE`, `PCH`, `CGH`, até 1200 registros por sigla (cadastro atual, sem recorte temporal) | 3876 (Solar 1200, Eólica 1200, Hidráulica 1476) |
| Meteorologia horária | [Open-Meteo — Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) | Petrolina (lat −9,39; lon −40,50), 01/04/2025 a 30/06/2025, fuso `America/Recife`, horas de 7h a 17h | 1001 |

As duas APIs são públicas e **não exigem token ou chave**. Os dados do Open-Meteo são históricos estimados por modelos/reanálise, não leituras de um sensor.

## Arquivos

- `CP02_2SEM_APIs_Energia_Renovavel_ML.ipynb`: consulta às APIs, geração dos CSVs, análise, três classificadores, três regressores e conclusões.
- `aneel_classificacao_orange.csv`: dados da Tarefa 1 (`potencia_kw`, `latitude`, `longitude`, `fonte`).
- `meteo_regressao_orange.csv`: dados da Tarefa 2 (`data_hora`, `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora`, `radiacao_w_m2`).

## Como executar

1. Abra o notebook no Google Colab (botão "Open in Colab" no topo) ou em um Jupyter com Python 3.
2. Execute **todas as células em ordem** (Ambiente de execução → Executar tudo). Bibliotecas usadas: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`; todas já vêm no Colab.
3. As células de consulta geram os dois CSVs. Se uma API estiver fora do ar, use os CSVs deste repositório, que reproduzem a consulta.

## Resumo dos resultados

**Tarefa 1 — Classificação** (divisão estratificada 80/20, `random_state=42`, features padronizadas com `StandardScaler` ajustado só no treino; métricas por classe com média `macro`)

| Algoritmo | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Regressão Logística | 0,8247 | 0,8282 | 0,8214 | 0,8197 |
| Árvore de Decisão | 0,9613 | 0,9614 | 0,9610 | 0,9612 |
| **Random Forest** | **0,9755** | **0,9769** | **0,9741** | **0,9753** |

Modelo escolhido: Random Forest. A classe mais confundida é Solar (12 dos 240 exemplos de teste classificados errado).

**Tarefa 2 — Regressão** (divisão temporal: 800 primeiras horas para treino, 201 finais para teste, sem embaralhar)

| Algoritmo | R² | MAE (W/m²) | MSE ((W/m²)²) | RMSE (W/m²) |
|---|---|---|---|---|
| Regressão Linear | 0,3598 | 145,20 | 30034,20 | 173,30 |
| Árvore de Decisão | 0,6741 | 88,79 | 15291,16 | 123,66 |
| **Random Forest** | **0,8442** | **66,80** | **7307,42** | **85,48** |

Modelo escolhido: Random Forest, que captura a relação não linear entre hora, nuvens e radiação.

## Onde estão as respostas

| O que procurar | Onde no notebook |
|---|---|
| Comparação dos 3 classificadores (tabela + matrizes de confusão) | Seção "1_4 Avaliação das Métricas e Matriz de Confusão" |
| Conclusão da classificação | Seção "1_5 Análise dos resultados e Conclusão" |
| Comparação dos 3 regressores (tabela + gráficos de R², MAE, MSE) | Tarefa 2, item 4 |
| Gráfico real × previsto | Tarefa 2, item 4 |
| Conclusão da regressão | Tarefa 2, item 5 |

## Limitações

- Potência e coordenadas não identificam o princípio físico de geração; o modelo pode aprender padrões do cadastro (regiões, faixas de potência) em vez de características da fonte.
- Os dados meteorológicos cobrem uma única localidade e três meses (abril a junho de 2025), então o teste (últimas semanas) tem condições próprias.
- Radiação (W/m²) não é energia (kWh) nem geração fotovoltaica: faltam área e eficiência dos módulos, temperatura, inclinação, sombreamento e perdas do sistema.
