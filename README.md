# Statistical Arbitrage & Mean Reversion Strategies

A collection of quantitative statistical arbitrage and mean reversion strategies built on cointegration (Engle-Granger, Johansen) and state filtering (Kalman Filter), featuring complete backtesting, transaction cost controls, and systematic comparisons against Buy & Hold benchmarks.

> ** Disclaimer:** This repository is strictly for educational and quantitative research purposes. None of the strategies published here constitute financial advice. Results are based on historical backtests and do not guarantee future returns. Please review the Limitations and Warnings section before running any code in live environments.

---

##  Repository Structure

```text
.
├── 01-mean-reversion-basics/
│   ├── MeanReversionV1.ipynb          # Fundamentals: OU process, stationarity, half-life
│   └── Correlation.ipynb              # Basic correlation / mean reversion implementation
│
├── 02-kalman-bitcoin-mstr/
│   ├── MeanReversionStrategy-Bitcoin.ipynb
│   └── Bitcoin_Strategy_MeanReversion_.pdf
│
├── 03-kalman-currencies-nzd-ars/
│   ├── Kalman_currencies.ipynb
│   └── Kalman_Currencies.pdf
│
└── README.md

```

*Note: Reorganize paths according to your upload preferences — the structure suggested above makes each strategy self-contained (notebook + report).*

---

##  Common Theoretical Framework

All strategies share the same underlying model for the spread: an **Ornstein-Uhlenbeck process**:

$$dX_t = \theta(\mu - X_t)dt + \sigma dW_t$$

where $\theta$ controls the speed of mean reversion, $\mu$ represents the equilibrium level, and $\sigma$ denotes the volatility of the spread. The primary distinction between strategies lies in how the spread is constructed:

| Method | Parameter Assumption | Strategies Applied |
| --- | --- | --- |
| **OLS / Engle-Granger** | Static hedge ratio, estimated via simple regression | `Correlation.ipynb`, base of `MeanReversionV1.ipynb` |
| **Johansen (VECM)** | Static, multivariate hedge ratio (cointegration rank) | EWC–EWA–IGE |
| **Kalman Filter** | Dynamic hedge ratio (random walk state space) | BTC–MSTR, NZD/USD–ARS/USD |

Entry and exit signals are generated based on the **z-score** of the spread's deviation relative to its moving average, applying a 1-period lag to eliminate lookahead bias.

---

##  Included Strategies

### 1. Johansen Cointegration — EWC / EWA / IGE

A triplet of ETFs (Canada, Australia, and Global Natural Resources) tied together through their structural exposure to commodities. Trivariate cointegration via Johansen test, with the spread constructed using the primary eigenvector and entry/exit signals driven by z-scores ($\pm2.0$ / $0.0$).

* **Data:** MetaTrader5, H1, Apr-2010 to Apr-2020 (~17,500 obs.)
* **Costs:** 0.10% per transaction
* **Backtest Results:** Annualized return **+3.52%**, Sharpe **0.52**, Volatility **6.81%**, Max Drawdown **−5.80%**
* **Context:** All three underlying assets held individually (Buy & Hold) yielded negative returns and negative Sharpe ratios during the same timeframe.

### 2. Kalman Filter — Bitcoin (BTC-USD) vs. MicroStrategy (MSTR)

Dynamic hedge ratio estimated via Kalman Filter (modeled as a random walk) between BTC-USD and its leveraged corporate proxy MSTR, designed to adapt dynamically to capital structure shifts (dilution, convertible debt issuances, treasury accumulation).

* **Data:** Yahoo Finance, Daily
* **Costs:** 5 bps per leg
* **Backtest Results:** Annualized return **−5.58%**, Sharpe **−0.18**, Volatility **30.61%**, Max Drawdown **−27.15%**
* **Context:** Underperformed cash, but delivered drastically lower drawdown and volatility compared to Buy & Hold exposures in BTC (−51.17% DD) and MSTR (−76.53% DD).

### 3. Kalman Filter — NZD/USD vs. ARS/USD

The same Kalman Filter framework applied to an extreme FX pair: NZD (G10, commodity-linked currency) against ARS (emerging market, hyperinflationary regime).

* **Data:** Yahoo Finance, Daily
* **Costs:** 1.5 bps per leg
* **Backtest Results:** Annualized return **−2.27%**, Sharpe **−0.35**, Volatility **6.49%**, Max Drawdown **−13.59%**
* **Context:** Defensive outperformance compared to standalone ARS/USD (Sharpe −2.44, DD −88.33%), though still negative in absolute terms.

### 4. Mean Reversion / Correlation — Base

An introductory notebook detailing the broader quantitative workflow: core definitions of mean reversion, stationarity testing (ADF, Hurst exponent, Variance Ratio), choosing hedge ratios via OLS, and assembling a strategy from scratch. Serves as a prerequisite before moving to Johansen/Kalman implementations.

---

##  Comparative Results Table

| Strategy | Assets | Annual Return | Sharpe | Annual Vol. | Max DD |
| --- | --- | --- | --- | --- | --- |
| **Johansen** | EWC / EWA / IGE | **+3.52%** | **0.52** | **6.81%** | **−5.80%** |
| **Kalman** | BTC-USD / MSTR | −5.58% | −0.18 | 30.61% | −27.15% |
| **Kalman** | NZD/USD / ARS/USD | −2.27% | −0.35 | 6.49% | −13.59% |

> Only the Johansen strategy (EWC-EWA-IGE) achieves absolute profitability within the backtest. Both Kalman Filter strategies register negative returns, though with substantially reduced volatility and drawdowns compared to unhedged exposure in the underlying assets.

---

##  Tech Stack

* **Python 3.10+**
* `pandas`, `numpy` — Data manipulation and arithmetic
* `statsmodels` (`coint_johansen`, `adfuller`, `VECM`) — Cointegration and stationarity testing
* `arch` — Supplementary unit root tests
* `yfinance` — Daily market data (BTC-USD, MSTR, FX)
* `MetaTrader5` (`mt5.copy_rates_range`) — Intraday (H1) ETF data
* `matplotlib` — Equity curve visualization
* `python-dotenv` — Environment configuration (`.env`, never push credentials to repository)

### Quick Start

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install pandas numpy statsmodels arch yfinance MetaTrader5 matplotlib python-dotenv

```

For strategies leveraging MetaTrader5, create an untracked `.env` file containing:

```env
MT5_LOGIN=your_login
MT5_PWD=your_password

```

---

##  How to Use This Repository

1. Clone the repository and configure the virtual environment.
2. Launch the notebook corresponding to your research topic (`jupyter lab`).
3. Adjust execution dates or asset universe to reproduce or expand findings.
4. Review the supplementary PDFs in each folder for exhaustive details: economic rationale, mathematical formulations, data pre-processing steps, backtest engine mechanics, and core constraints.

---

##  Limitations and Warnings

These strategies represent research workflows rather than production-ready trading systems. Consider the following constraints before running live setups:

* **In-Sample Cointegration Parameter Estimation (Johansen):** Cointegration vectors are fitted once across the entire historical period and held static, resulting in de facto lookahead bias during estimation. Rigorous evaluation requires a walk-forward framework.
* **Selection Bias:** Asset pairs/triplets were selected using prior domain knowledge regarding economic correlations without adjustments for multiple testing across wider universes.
* **Execution Latency:** Backtests evaluate signals at candle close (H1 or Daily); intra-bar movements that trigger and settle deviations prior to close remain uncaptured, overstating real-world execution capacity.
* **Simplified Transaction Models:** Commissions and slippage are modeled as fixed percentages without accounting for order-book depth, market impact, short borrow fees (critical for ARS/USD), or capital controls.
* **Fee Sensitivity:** For the Johansen implementation, doubling transaction fees (0.10% → 0.20%) cuts the Sharpe ratio from 0.52 to ~0.28, nearing economic viability limits.
* **State Noise Calibration (Kalman):** Tuning process noise $Q$ is critical — excessively high values overfit transient noise, while overly low values prevent adaptation to structural shifts (e.g., MSTR NAV premium changes, capital controls in ARS).
* **Out-of-Sample Validation:** None of the four strategies have been deployed out-of-sample or via paper/live execution; reported figures are strictly historical (2010–2020 for EWC/EWA/IGE, 2025–2026 for BTC/MSTR and FX).

---

##  Roadmap

* [ ] Walk-forward re-estimation for Johansen vectors (rolling estimation windows)
* [ ] Multiple-testing corrections across larger universe scans
* [ ] Regime-switching filters (HMM) to halt trading during structural trend shifts
* [ ] Sub-hourly intraday backtesting (M15/M30) to minimize live execution drift
* [ ] Multi-asset basket expansion within the Kalman Filter framework

---

##  License

**MIT** — Free for educational and research usage. See `LICENSE`.

---

##  Contact

**Jan Gómez Escobar** — Quantitative Research Division

Reference Repository / Source Code: [github.com/spinortechnologies](https://www.google.com/search?q=https://github.com/spinortechnologies)
