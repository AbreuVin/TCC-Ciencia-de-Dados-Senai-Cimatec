# TCC — Protocolo de Diagnóstico Estatístico para Seleção de Modelos em Séries Temporais

**Pós-Graduação em Ciência de Dados e IA · SENAI CIMATEC · Salvador, BA**

| | |
|---|---|
| **Aluno** | Vinícius da Fonseca Abreu |
| **Orientador** | Prof. Eldman |
| **Dataset** | M4 Competition — subconjunto mensal (~48.000 séries) |
| **Prazo** | 4 de setembro de 2026 |

---

## Hipótese

A seleção de modelo guiada por diagnóstico estatístico — detecção de sazonalidade (STL) + ADF + KPSS + BDS + ARCH + Hurst/GPH — produz menor OWA do que o benchmark automático (Auto-ARIMA aplicado cegamente) sobre o subconjunto mensal da Competição M4.

---

## Protocolo de diagnóstico (5 etapas)

**Etapa 0 — Sazonalidade**
Força sazonal STL: F_S > 0,64 → flag `sazonal=True` propagado ao modelo final.

**Etapa 1 — Estacionariedade**
ADF + KPSS aplicados em conjunto. Concordância entre os dois fornece evidência robusta. Série não-estacionária é diferenciada; a ordem `d` é registrada.

**Etapa 2 — Linearidade**
Teste BDS nos resíduos de um AR(p). Rejeição consistente indica estrutura não-linear → direciona para LSTM.

**Etapa 3 — Heterocedasticidade condicional**
Teste ARCH de Engle com q=12. Rejeição → direciona para ARIMA-GARCH.

**Etapa 4 — Memória longa**
Expoente de Hurst (R/S) + estimador GPH. H > 0,7 com d ∈ (0; 0,5) confirmado → direciona para ARFIMA.

---

## Família de modelos

| Perfil | Critério | Modelo |
|--------|----------|--------|
| Linear estacionário | ADF+KPSS concordam; BDS não rejeita; ARCH não rejeita; H ≤ 0,7 | ARIMA |
| Sazonal | Etapas 1–4 sem redirecionamento; F_S > 0,64 | SARIMA |
| Não-linear | BDS rejeita H₀ consistentemente | LSTM |
| Heterocedástico | ARCH rejeita H₀ | ARIMA-GARCH |
| Memória longa | H > 0,7 e GPH confirma d ∈ (0; 0,5) | ARFIMA |

---

## Baselines de comparação

- **Auto-ARIMA** — seleção automática de (p,d,q) sem diagnóstico prévio
- **Método Theta** — vencedor da M3, competitivo na M4

---

## Estrutura do repositório

```
TCC Data Science/
├── latex/
│   ├── main.tex          # documento principal
│   └── referencias.bib   # referências BibTeX (28 entradas)
├── notebooks/            # (a criar) experimentos por etapa
├── src/                  # (a criar) módulo do protocolo
├── requirements.txt
├── .gitignore
└── README.md
```

> PDFs de referência (`refs/`) e dataset M4 (`data/`) não são versionados — ver `.gitignore`.

---

## Como rodar

```bash
pip install -r requirements.txt
```

Dataset M4 (subconjunto mensal):

```bash
git clone https://github.com/Mcompetitions/M4-methods temp_m4
cp temp_m4/Dataset/Train/Monthly-train.csv data/
cp temp_m4/Dataset/Test/Monthly-test.csv data/
rm -rf temp_m4
```

---

## Progresso

| Sprint | Período | Entrega | Status |
|--------|---------|---------|--------|
| S0 — Kickoff | 14–27 abr 2026 | Setup, leituras, documento LaTeX | Concluído |
| S1 — Metodologia | 28 abr–18 mai | Cap. 1, 2 e 3 em LaTeX | Em andamento |
| S2 — Experimentos | 19 mai–7 jun | Pipeline de diagnóstico + resultados | Pendente |
| S3 — Análise | 8–28 jun | Cap. 4 + análises complementares | Pendente |
| S4 — Revisão Final | 29 jun–9 ago | TCC formatado + entrega | Pendente |

---

## Referências principais

- Makridakis, S., Spiliotis, E., Assimakopoulos, V. (2020). *The M4 Competition: 100,000 time series and 61 forecasting methods.* International Journal of Forecasting, 36(1), 54–74.
- Broock, W. A., Scheinkman, J. A., Dechert, W. D., LeBaron, B. (1996). *A test for independence based on the correlation dimension.* Econometric Reviews, 15(3), 197–235.
- Hyndman, R. J., Athanasopoulos, G. (2018). *Forecasting: Principles and Practice* (2nd ed.). OTexts.
- Box, G. E. P., Jenkins, G. M. (1970). *Time Series Analysis: Forecasting and Control.* Holden-Day.

---

*SENAI CIMATEC · Pós-Graduação em Ciência de Dados e Inteligência Artificial · Salvador, Bahia · 2026*
