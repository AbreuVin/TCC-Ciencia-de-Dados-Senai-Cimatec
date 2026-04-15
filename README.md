# Protocolo de Diagnóstico Estatístico para Seleção de Modelos de Séries Temporais
### Aplicação ao Dataset M4 Mensal

<p align="center">
  <img src="https://img.shields.io/badge/Status-Em%20desenvolvimento-yellow?style=flat-square"/>
  <img src="https://img.shields.io/badge/Python-3.10%2B-blue?style=flat-square&logo=python"/>
  <img src="https://img.shields.io/badge/Institui%C3%A7%C3%A3o-SENAI%20CIMATEC-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Sprint%20atual-S0%20%E2%80%94%20Kickoff-teal?style=flat-square"/>
  <img src="https://img.shields.io/badge/Prazo-04%2F09%2F2026-critical?style=flat-square"/>
</p>

---

## Sobre o Projeto

**TCC de Pós-Graduação em Ciência de Dados e Inteligência Artificial**  
Centro Universitário SENAI CIMATEC — Salvador, Bahia  

**Aluno:** Vinícius da Fonseca Abreu  
**Orientador:** Prof. Eldman  
**Prazo de entrega:** 4 de setembro de 2026  

---

## Hipótese de Pesquisa

> *"A seleção de modelo de previsão guiada por um protocolo sistemático de diagnóstico estatístico — composto pelos testes ADF, KPSS, BDS, Engle ARCH e expoente de Hurst — produz menor erro de previsão (OWA/sMAPE) do que a aplicação do benchmark automático utilizado na competição M4."*

A ideia central é que a maioria dos estudos aplica modelos de séries temporais sem verificar previamente se os dados satisfazem os pré-requisitos estatísticos desses modelos. Por exemplo, ARIMA pressupõe linearidade — mas e se a série for não-linear? O teste BDS detecta isso antes. Este trabalho propõe e valida um protocolo de triagem que **afunila a escolha do modelo a partir das propriedades da série**, e não do gosto do pesquisador.

---

## Dataset

**M4 Competition** — subconjunto mensal  
- 48.000 séries temporais univariadas reais  
- Domínios: finanças, macro, micro, indústria, demografia, outros  
- Referência padrão em forecasting acadêmico (Makridakis et al., 2018/2019)  
- Download: [github.com/Mcompetitions/M4-methods](https://github.com/Mcompetitions/M4-methods)

---

## Protocolo de Diagnóstico (Contribuição Original)

```
Série temporal bruta
        │
        ▼
┌───────────────────────────────────────────────────────┐
│  ETAPA 1 — Estacionariedade: ADF + KPSS (conjunto)    │
│  → Não estacionária: diferenciar até I(0)              │
└───────────────────────────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────────────────────────┐
│  ETAPA 2 — Linearidade: Teste BDS                      │
│  → Rejeita H₀ (p < 0.05): estrutura não-linear         │
└───────────────────────────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────────────────────────┐
│  ETAPA 3 — Heterocedasticidade: Teste ARCH (Engle)     │
│  → Rejeita H₀: volatilidade condicional presente       │
└───────────────────────────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────────────────────────┐
│  ETAPA 4 — Memória Longa: Expoente de Hurst + GPH      │
│  → Hurst > 0.7: dependência de longa duração           │
└───────────────────────────────────────────────────────┘
        │
        ▼
  FAMÍLIA DE MODELO ESTATISTICAMENTE COERENTE
  ┌────────────────┬───────────────────┬──────────────────────┬──────────────────────┐
  │ Linear + I(0)  │ Linear + Sazonal  │  Não-linear (BDS)    │  ARCH (Volatilidade) │
  ├────────────────┼───────────────────┼──────────────────────┼──────────────────────┤
  │ ARIMA / ARMA   │ SARIMA / SARIMAX  │  LSTM / GRU / N-BEATS│  ARIMA-GARCH         │
  │ ETS / Theta    │ Prophet / TBATS   │  XGBoost (lags)      │  LSTM-ARIMA híbrido  │
  └────────────────┴───────────────────┴──────────────────────┴──────────────────────┘
        │
        ▼
  AVALIAÇÃO: sMAPE / MASE / OWA vs. benchmark cego (Auto-ARIMA M4)
```

---

## Estrutura do Repositório

```
TCC-Ciencia-de-Dados-Senai-Cimatec/
│
├── refs/                              # Referências bibliográficas (PDFs)
│   ├── camada1_M4/                    # Dataset M4 + código oficial
│   ├── camada2_papers/                # Artigos principais sobre M4
│   ├── camada3_testes_estatisticos/   # BDS, ADF, ARCH, Hurst, preprocessing
│   ├── camada4_surveys/               # Surveys gerais de séries temporais
│   └── camada5_tccs_brasil/           # TCCs e dissertações brasileiras
│
├── data/
│   └── m4_monthly.csv                 # Dataset M4 mensal (48k séries)
│
├── notebooks/
│   ├── 01_eda.ipynb                   # Análise exploratória inicial
│   ├── 02_stationarity.ipynb          # Testes ADF + KPSS
│   ├── 03_bds_linearity.ipynb         # Teste BDS (linearidade)
│   ├── 04_arch_hurst.ipynb            # Teste ARCH + expoente de Hurst
│   ├── 05_diagnostic_full.ipynb       # Pipeline diagnóstico integrado
│   ├── 06_arima_results.ipynb         # Resultados ARIMA (séries lineares)
│   ├── 07_lstm_results.ipynb          # Resultados LSTM (séries não-lineares)
│   ├── 08_garch_results.ipynb         # Resultados GARCH (heteroced.)
│   └── 09_benchmark_blind.ipynb       # Benchmark cego: Auto-ARIMA em todas
│
├── src/
│   └── diagnostic_pipeline.py         # Módulo Python do protocolo
│
├── results/
│   └── tabela_comparativa.csv         # sMAPE / MASE / OWA: guiado vs cego
│
├── capitulos/                         # Capítulos do TCC (LaTeX/Word)
│   ├── cap1_introducao.docx
│   ├── cap2_revisao_literatura.docx
│   ├── cap3_metodologia.docx
│   ├── cap4_resultados.docx
│   └── cap5_conclusao.docx
│
├── download_refs.py                   # Script de download das referências
├── fix_downloads.py                   # Script de correção de downloads
├── dashboard.html                     # Gantt + links (abrir no navegador)
└── README.md
```

---

## Roadmap — Sprints

| Sprint | Período | Objetivo | Status |
|--------|---------|----------|--------|
| **S0 — Kickoff** | 14–27 abr 2026 | Setup RAG, leituras M4, EDA inicial | 🟡 Em andamento |
| **S1 — Lit. Review** | 28 abr–18 mai | Cap.2 revisão de literatura (16–20p) | ⬜ Pendente |
| **S2 — Metodologia** | 19 mai–7 jun | Cap.3 + pipeline diagnóstico | ⬜ Pendente |
| **S3 — Experimentos** | 8–28 jun | Rodar protocolo + modelos + benchmark | ⬜ Pendente |
| **S4 — Análise** | 29 jun–19 jul | Cap.4 resultados + Cap.5 conclusão | ⬜ Pendente |
| **S5 — Revisão** | 20 jul–9 ago | ABNT, abstract, revisão geral | ⬜ Pendente |
| **Buffer** | 10 ago–4 set | Ajustes pós-orientador + entrega | ⬜ Pendente |

---

## Instalação e Uso

### Requisitos

```bash
Python 3.10+
pip install -r requirements.txt
```

### `requirements.txt`

```text
pandas>=2.0
numpy>=1.24
statsmodels>=0.14        # ADF, KPSS, ARIMA
arch>=6.0                # BDS, ARCH, GARCH
hurst>=0.0.5             # Expoente de Hurst
tensorflow>=2.13         # LSTM
scikit-learn>=1.3
matplotlib>=3.7
seaborn>=0.12
jupyter>=1.0
tqdm>=4.65
requests>=2.31
pmdarima>=2.0            # Auto-ARIMA benchmark
```

### Baixar referências

```bash
pip install requests tqdm
python download_refs.py
```

### Baixar dataset M4

```bash
# Opção 1 — GitHub oficial
cd data
git clone https://github.com/Mcompetitions/M4-methods temp_m4
cp temp_m4/Dataset/Monthly-train.csv m4_monthly_train.csv
cp temp_m4/Dataset/Monthly-test.csv  m4_monthly_test.csv
rm -rf temp_m4

# Opção 2 — Kaggle CLI
pip install kaggle
kaggle datasets download yogesh94/m4-forecasting-competition-dataset -p data/
```

---

## Referências Principais

> Para a lista completa, ver `refs/README.md`

- **Makridakis, S., Spiliotis, E., Assimakopoulos, V.** (2018). The M4 Competition: Results, findings, conclusion and way forward. *International Journal of Forecasting*, 34(4), 802–808.
- **Makridakis, S., Spiliotis, E., Assimakopoulos, V.** (2020). The M4 Competition: 100,000 time series and 61 forecasting methods. *International Journal of Forecasting*, 36(1), 54–74.
- **Broock, W. A., Scheinkman, J. A., Dechert, W. D., LeBaron, B.** (1996). A test for independence based on the correlation dimension. *Econometric Reviews*, 15(3), 197–235.
- **Engle, R. F.** (1982). Autoregressive conditional heteroscedasticity with estimates of the variance of United Kingdom inflation. *Econometrica*, 50(4), 987–1007.
- **Box, G. E. P., Jenkins, G. M.** (1970). *Time Series Analysis: Forecasting and Control*. Holden-Day.

---

## Licença

Este repositório é de uso acadêmico. Os dados do M4 Competition estão sujeitos à licença do repositório original ([Mcompetitions/M4-methods](https://github.com/Mcompetitions/M4-methods)).

---

<p align="center">
  <sub>SENAI CIMATEC · Pós-Graduação em Ciência de Dados e IA · Salvador, BA · 2026</sub>
</p>
#   T C C - C i e n c i a - d e - D a d o s - S e n a i - C i m a t e c  
 