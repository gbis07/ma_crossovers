# Moving Average Crossovers
### Systematic Momentum Strategies in Cryptocurrency Markets

A quantitative finance research project investigating the performance of **moving-average crossover momentum signals** in cryptocurrency markets.

The project constructs multiple exponential moving average (EMA) crossover signals, normalizes them for volatility, applies a nonlinear signal transformation, and combines them into a systematic momentum indicator. The resulting signal is evaluated using both **time-series** and **cross-sectional** portfolio construction methods across a diversified universe of cryptocurrencies.

---

## Overview

Moving-average crossover strategies attempt to capture persistent price trends by comparing short-term and long-term estimates of an asset's price level.

This project develops a systematic implementation of this idea by:

- Constructing multiple EMA crossover signals
- Normalizing signals for asset volatility
- Standardizing signals through time
- Applying a bounded nonlinear response function
- Combining multiple crossover horizons into a single momentum signal
- Constructing time-series and cross-sectional portfolios
- Evaluating strategy performance on separate training and validation periods
- Comparing portfolio returns against Bitcoin as a market benchmark

The entire research workflow—from historical data collection to signal generation, portfolio construction, validation, and performance analysis—is contained within a reproducible Jupyter notebook.

---

## Features

- Historical cryptocurrency price collection from Binance
- 30-asset cryptocurrency universe
- Multiple EMA crossover horizons
- Two-stage volatility normalization
- Nonlinear signal transformation
- Composite momentum signal construction
- Time-series momentum portfolio
- Cross-sectional long-short portfolio
- Lagged portfolio weights to prevent look-ahead bias
- Separate training and validation datasets
- Annualized return and volatility calculations
- Sharpe ratio evaluation
- Drawdown analysis
- Benchmark regression against Bitcoin
- HAC/Newey-West robust alpha estimation

---

## Repository Structure

```text
ma_crossovers/
│
├── project.ipynb          # Complete research notebook
├── training_data.pk       # Training-period cryptocurrency prices
├── validation_data.pk     # Out-of-sample validation prices
└── README.md
```

---

# Methodology

## 1. EMA Crossover Signals

Momentum is first measured using the difference between a short-term and long-term exponential moving average:

\[
x_{k,t}
=
EMA(P_t,n_{k,s})
-
EMA(P_t,n_{k,l})
\]

where:

- \(P_t\) is the asset price
- \(n_{k,s}\) is the short EMA horizon
- \(n_{k,l}\) is the long EMA horizon
- \(k\) identifies the crossover specification

The project evaluates three EMA pairs:

```text
(8, 24)
(16, 48)
(32, 96)
```

A positive crossover indicates upward momentum:

\[
x_{k,t} > 0
\]

while a negative crossover indicates downward momentum:

\[
x_{k,t} < 0
\]

Using several horizons allows the strategy to capture trends operating over different time scales rather than relying on a single moving-average specification.

---

## 2. Price Volatility Normalization

Raw moving-average differences are difficult to compare across assets because cryptocurrencies have substantially different price levels and volatility.

Each crossover signal is therefore normalized by rolling price volatility:

\[
y_{k,t}
=
\frac{x_{k,t}}
{\sigma(P_t)}
\]

For cryptocurrency markets, the project uses a **91-day rolling window**, corresponding approximately to three months in a continuously traded market.

This converts the raw crossover into a volatility-adjusted momentum measure.

---

## 3. Signal Normalization

The volatility-adjusted crossover is normalized again using the historical volatility of the signal itself:

\[
z_{k,t}
=
\frac{y_{k,t}}
{\sigma(y_k)}
\]

A **365-day rolling window** is used for cryptocurrency markets.

This second normalization makes the crossover signals more comparable through time and across assets.

---

## 4. Nonlinear Response Function

The normalized crossover is transformed using a nonlinear response function:

\[
u(z)
=
\frac{
z e^{-z^2/4}
}{
\sqrt{2}e^{-1/2}
}
\]

The transformation produces a bounded momentum exposure approximately satisfying:

\[
-1 \leq u(z) \leq 1
\]

Rather than allowing increasingly extreme crossover values to generate increasingly large positions, the nonlinear transformation reduces exposure when signals become unusually large.

This limits the influence of extreme observations and produces a more controlled trading signal.

---

## 5. Composite Momentum Signal

The three transformed EMA crossover signals are combined using equal weighting:

\[
Signal_t
=
\frac{1}{K}
\sum_{k=1}^{K}u_{k,t}
\]

where \(K=3\).

The resulting signal summarizes momentum across short-, medium-, and longer-term EMA horizons.

Positive values indicate bullish momentum, while negative values indicate bearish momentum.

---

# Portfolio Construction

Two portfolio implementations are evaluated.

## Time-Series Portfolio

The time-series strategy determines each asset's exposure directly from its own momentum signal.

Portfolio weights are approximately:

\[
w_{i,t}
=
\frac{Signal_{i,t}}{N_t}
\]

where \(N_t\) represents the number of assets with valid signals.

The strategy therefore:

- goes long assets with positive momentum
- goes short assets with negative momentum
- increases exposure as the momentum signal strengthens
- diversifies exposure across the available universe

Portfolio returns use **lagged weights**:

\[
R_{p,t}
=
\sum_i w_{i,t-1}R_{i,t}
\]

ensuring that today's return is generated using information available before the return occurred.

---

## Cross-Sectional Portfolio

The cross-sectional strategy compares momentum signals across assets.

At each rebalance date:

1. Rank cryptocurrencies by their momentum signal
2. Select the assets with the strongest momentum
3. Select the assets with the weakest momentum
4. Go long the strongest assets
5. Go short the weakest assets
6. Allocate equal absolute weight across positions

The default implementation holds:

```text
3 Long Positions
3 Short Positions
```

This produces a long-short portfolio designed to capture **relative momentum** across the cryptocurrency universe.

As with the time-series portfolio, weights are lagged before calculating realized returns.

---

# Data

Historical daily cryptocurrency prices are collected through the Binance API.

The research universe contains approximately 30 cryptocurrencies, including:

- Bitcoin
- Ethereum
- Solana
- Binance Coin
- Bitcoin Cash
- Aave
- Litecoin
- Avalanche
- Chainlink
- XRP
- Cardano
- Polkadot
- Uniswap
- Filecoin
- Cosmos
- Stellar
- Near
- VeChain
- and others

Data retrieval is parallelized using `ThreadPoolExecutor` to reduce API download time.

---

# Training and Validation

To evaluate whether the strategy generalizes beyond the period used during development, the project separates the analysis into training and validation datasets.

### Training Period

```text
May 2022 – May 2024
```

### Validation Period

```text
August 2024 – August 2026
```

The validation period provides an out-of-sample test of whether the momentum methodology continues to perform after the initial research period.

---

# Performance Evaluation

Both portfolio implementations are evaluated using standard quantitative finance metrics.

Metrics include:

- Cumulative return
- Annualized return
- Annualized volatility
- Sharpe ratio
- Maximum drawdown
- Portfolio turnover
- Benchmark correlation
- Market beta
- Annualized alpha
- Alpha t-statistic

Because cryptocurrency markets trade continuously, annualization assumes:

\[
365
\]

trading periods per year for daily data.

---

# Benchmark Analysis

Bitcoin is used as the primary cryptocurrency market benchmark.

Strategy returns are estimated using:

\[
R_{strategy,t}
=
\alpha
+
\beta R_{BTC,t}
+
\epsilon_t
\]

The regression reports:

- Annualized alpha
- Alpha t-statistic
- Bitcoin beta
- \(R^2\)
- Strategy/Bitcoin correlation

HAC/Newey-West robust standard errors are used to account for potential heteroskedasticity and autocorrelation in strategy returns.

This analysis helps distinguish returns generated by the momentum strategy from simple exposure to the broader cryptocurrency market.

---

# Technologies

- Python
- Pandas
- NumPy
- Statsmodels
- Matplotlib
- python-binance
- concurrent.futures
- Jupyter Notebook

---

# Installation

Clone the repository:

```bash
git clone https://github.com/gbis07/ma_crossovers.git
cd ma_crossovers
```

Install dependencies:

```bash
pip install pandas numpy matplotlib statsmodels python-binance jupyter
```

---

# Binance API Configuration

Historical market data is retrieved using the Binance API.

The notebook expects credentials to be stored as environment variables:

```text
BINANCE_API_KEY
BINANCE_API_SECRET
```

For example, in PowerShell:

```powershell
$env:BINANCE_API_KEY="your_api_key"
$env:BINANCE_API_SECRET="your_api_secret"
```

The included pickle files can also be used to reproduce the existing analysis without repeatedly downloading the historical datasets.

---

# Running the Project

Launch Jupyter:

```bash
jupyter notebook
```

Open:

```text
project.ipynb
```

Execute the notebook from top to bottom.

---

# Research Workflow

```text
Historical Cryptocurrency Prices
              │
              ▼
       EMA Crossovers
              │
              ▼
   Price Volatility Normalization
              │
              ▼
     Signal Normalization
              │
              ▼
 Nonlinear Signal Transformation
              │
              ▼
    Composite Momentum Signal
              │
        ┌─────┴─────┐
        ▼           ▼
   Time-Series   Cross-Sectional
    Portfolio       Portfolio
        │           │
        └─────┬─────┘
              ▼
          Backtesting
              │
              ▼
       Validation Period
              │
              ▼
     Performance Metrics
              │
              ▼
   Bitcoin Benchmark Regression
              │
              ▼
          Drawdowns   
```

---

# Research Highlights

The project demonstrates a complete systematic trading research workflow including:

- financial data engineering
- technical signal construction
- volatility normalization
- nonlinear signal transformations
- multi-horizon signal aggregation
- time-series momentum
- cross-sectional momentum
- long-short portfolio construction
- out-of-sample validation
- systematic backtesting
- benchmark regression
- robust statistical inference

Rather than simply testing whether a moving-average crossover historically generated positive returns, the project treats moving averages as **continuous quantitative signals** that can be normalized, combined, and incorporated into systematic portfolio construction.

---

# Future Improvements

Potential extensions include:

- transaction cost modeling
- explicit turnover penalties
- volatility-targeted portfolio sizing
- alternative EMA horizons
- optimized crossover weighting
- rolling parameter estimation
- walk-forward validation
- dynamic universe selection
- volatility-based asset weighting
- risk parity portfolio construction
- alternative nonlinear signal transformations
- momentum crash analysis
- regime-dependent signals
- comparison against buy-and-hold benchmarks
- comparison with alternative trend-following models
- parameter sensitivity analysis
- live signal generation
- automated execution

---

# Disclaimer

This repository is intended for educational and research purposes only.

Nothing contained herein should be interpreted as financial advice, an investment recommendation, or evidence that historical strategy performance will persist in the future.

---

# Author

**Gianni Bisante**

M.S. Analytics — Georgia Institute of Technology

Interested in:

- Quantitative Research
- Machine Learning
- Statistical Modeling
- Financial Engineering
- Algorithmic Trading