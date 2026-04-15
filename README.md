# TCC — Protocolo de Diagnóstico Estatístico para Séries Temporais

**Pós-Graduação em Ciência de Dados e IA · SENAI CIMATEC · Salvador, BA**

| | |
|---|---|
| **Aluno** | Vinícius da Fonseca Abreu |
| **Orientador** | Prof. Eldman |
| **Dataset** | M4 Competition — subconjunto mensal |
| **Prazo** | 4 de setembro de 2026 |
| **Sprint atual** | S0 — Kickoff |

---

## O que esse projeto faz

A maioria dos estudos de séries temporais escolhe o modelo por tentativa e erro — aplica ARIMA, testa LSTM, compara os erros. Este trabalho propõe o caminho oposto: **primeiro diagnosticar a série estatisticamente, depois escolher o modelo coerente com o que os dados mostram.**

A hipótese é que séries não-lineares modeladas com ARIMA (que pressupõe linearidade) produzem previsões piores do que séries modeladas com LSTM após confirmação de não-linearidade pelo teste BDS. E que isso pode ser medido objetivamente sobre o dataset M4.

---

## Hipótese de pesquisa

> A seleção de modelo guiada por diagnóstico estatístico — ADF + KPSS + BDS + ARCH + Hurst — produz menor OWA e sMAPE do que o benchmark automático (Auto-ARIMA) aplicado cegamente à mesma amostra do M4 mensal.

---

## Protocolo de diagnóstico

O protocolo funciona como um funil de 4 etapas aplicado a cada série antes de qualquer modelagem:

**Etapa 1 — Estacionariedade**
Testes ADF e KPSS aplicados juntos. Usar os dois evita falso positivo: o ADF testa a presença de raiz unitária, o KPSS testa a estacionariedade. Séries não-estacionárias são diferenciadas até I(0) e a ordem `d` é registrada.

**Etapa 2 — Linearidade**
Teste BDS (Brock-Dechert-Scheinkman) aplicado nos resíduos de um AR ajustado. Rejeição da hipótese nula indica estrutura não-linear — o que invalida o uso de ARIMA e direciona para LSTM, GRU ou N-BEATS.

**Etapa 3 — Heterocedasticidade**
Teste ARCH de Engle. Volatilidade condicional identificada aqui direciona para modelos da família GARCH, isolados ou em combinação com ARIMA.

**Etapa 4 — Memória longa**
Expoente de Hurst combinado com o estimador GPH. Valor acima de 0,7 indica dependência de longo prazo, justificando o uso de ARFIMA em vez de ARIMA padrão.

A partir desse diagnóstico, o modelo aplicado tem justificativa estatística explícita — não é uma escolha arbitrária.

---

## Família de modelos por resultado do diagnóstico

| Perfil da série | Modelo indicado |
|---|---|
| Linear, estacionária | ARIMA / ARMA / ETS / Theta |
| Linear, sazonal | SARIMA / SARIMAX / Prophet / TBATS |
| Não-linear (BDS rejeita H₀) | LSTM / GRU / N-BEATS / XGBoost com lags |
| Heterocedasticidade (ARCH) | ARIMA-GARCH / LSTM-ARIMA híbrido |
| Longa memória (Hurst > 0,7) | ARFIMA / LSTM com janela longa |

---

## Dataset

O [M4 Competition](https://github.com/Mcompetitions/M4-methods) reúne 100.000 séries temporais univariadas reais de múltiplos domínios. Este trabalho usa o **subconjunto mensal** (~48.000 séries) por ser a frequência mais estudada na literatura, o que facilita a comparação de resultados.

---

## Avaliação

Os resultados são comparados usando as três métricas oficiais da competição M4:

- **sMAPE** — Symmetric Mean Absolute Percentage Error
- **MASE** — Mean Absolute Scaled Error
- **OWA** — Overall Weighted Average (combinação das duas anteriores)

A comparação é feita entre o protocolo guiado e o benchmark cego (Auto-ARIMA aplicado a todas as séries sem triagem prévia).

---

## Estrutura do repositório

```
TCC-Ciencia-de-Dados-Senai-Cimatec/
│
├── notebooks/
│   ├── 01_eda.ipynb                  # Análise exploratória do M4 mensal
│   ├── 02_stationarity.ipynb         # Testes ADF + KPSS
│   ├── 03_bds_linearity.ipynb        # Teste BDS
│   ├── 04_arch_hurst.ipynb           # Teste ARCH + expoente de Hurst
│   ├── 05_diagnostic_full.ipynb      # Pipeline diagnóstico integrado
│   ├── 06_arima_results.ipynb        # ARIMA nas séries lineares
│   ├── 07_lstm_results.ipynb         # LSTM nas séries não-lineares
│   ├── 08_garch_results.ipynb        # GARCH nas heterocedásticas
│   └── 09_benchmark_blind.ipynb      # Auto-ARIMA em todas (benchmark)
│
├── src/
│   └── diagnostic_pipeline.py        # Módulo reutilizável do protocolo
│
├── results/
│   └── tabela_comparativa.csv        # sMAPE / MASE / OWA: guiado vs cego
│
├── capitulos/
│   ├── cap1_introducao.docx
│   ├── cap2_revisao_literatura.docx
│   ├── cap3_metodologia.docx
│   ├── cap4_resultados.docx
│   └── cap5_conclusao.docx
│
├── dashboard.html                    # Gantt + links (abrir no navegador)
├── requirements.txt
├── .gitignore
└── README.md
```

> Os PDFs das referências (`refs/`) e o dataset M4 (`data/`) não são versionados — ver `.gitignore`.

---

## Como rodar

**1. Instalar dependências**

```bash
pip install -r requirements.txt
```

**2. Baixar o dataset M4**

```bash
cd data
git clone https://github.com/Mcompetitions/M4-methods temp_m4
cp temp_m4/Dataset/Monthly-train.csv m4_monthly_train.csv
cp temp_m4/Dataset/Monthly-test.csv  m4_monthly_test.csv
rm -rf temp_m4
```

**3. Executar os notebooks em ordem**

Começar pelo `01_eda.ipynb` e seguir a numeração.

---

## Progresso

| Sprint | Período | Entrega | Status |
|--------|---------|---------|--------|
| S0 — Kickoff | 14–27 abr 2026 | Setup, leituras, EDA | 🟡 Em andamento |
| S1 — Revisão de Literatura | 28 abr–18 mai | Cap. 2 completo | ⬜ |
| S2 — Metodologia | 19 mai–7 jun | Cap. 3 + pipeline | ⬜ |
| S3 — Experimentos | 8–28 jun | Tabela de resultados | ⬜ |
| S4 — Análise | 29 jun–19 jul | Cap. 4 + Cap. 5 | ⬜ |
| S5 — Revisão Final | 20 jul–9 ago | TCC formatado | ⬜ |
| Buffer | 10 ago–4 set | Entrega | ⬜ |

---

## Referências principais

- Makridakis, S., Spiliotis, E., Assimakopoulos, V. (2018). *The M4 Competition: Results, findings, conclusion and way forward.* International Journal of Forecasting, 34(4), 802–808.
- Makridakis, S., Spiliotis, E., Assimakopoulos, V. (2020). *The M4 Competition: 100,000 time series and 61 forecasting methods.* International Journal of Forecasting, 36(1), 54–74.
- Broock, W. A., Scheinkman, J. A., Dechert, W. D., LeBaron, B. (1996). *A test for independence based on the correlation dimension.* Econometric Reviews, 15(3), 197–235.
- Engle, R. F. (1982). *Autoregressive conditional heteroscedasticity with estimates of the variance of United Kingdom inflation.* Econometrica, 50(4), 987–1007.
- Box, G. E. P., Jenkins, G. M. (1970). *Time Series Analysis: Forecasting and Control.* Holden-Day.

---

*SENAI CIMATEC · Pós-Graduação em Ciência de Dados e Inteligência Artificial · Salvador, Bahia · 2026*
