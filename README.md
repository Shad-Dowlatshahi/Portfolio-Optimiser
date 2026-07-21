# Portfolio-Optimiser
This is a signal-based portfolio allocator combining inverse volatility and return/volatility signals, with an out-of-sample backtest across three market regimes. 

---

## Why This Project

Reading *A Random Walk Down Wall Street* left me with a question: if 
markets are broadly efficient, what role does portfolio construction 
play in generating risk-adjusted returns? Reading *An Introduction to 
Statistical Learning* sharpened this further — the bias-variance 
trade-off and the limits of in-sample performance suggested that any 
allocation model should be judged on out-of-sample robustness, not 
historical fit.

This project is an attempt to build and honestly test a simple, 
interpretable signal-based allocation model — prioritising 
transparency and robustness over complexity.

---

## How It Works

The user specifies:
- A list of tickers
- A start and end date for data collection
- Investable capital
- Risk tolerance (1 = most conservative, 10 = most aggressive)
- Risk-free rate

The model downloads historical price data via yfinance and constructs 
two signals:

**Inverse Volatility** — gives more weight to less volatile assets, 
reducing portfolio risk without requiring return forecasts.

**Return/Volatility** — gives more weight to assets that have 
historically delivered more return per unit of risk.

Both signals are normalised to sum to 1 and combined according to the 
user's risk tolerance — conservative settings weight inverse 
volatility more heavily, aggressive settings weight return/volatility 
more heavily.

The model outputs long-only portfolio weights, monetary allocations, 
expected annual return, expected volatility and Sharpe ratio.

---

## Design Decisions

**Why these two signals?**
Both are interpretable and grounded in financial logic. Inverse 
volatility is a well-established risk-weighting approach. 
Return/volatility is a simple proxy for risk-adjusted momentum. 
Neither requires predicting future returns directly.

**Why not the Kelly Criterion?**
An earlier version included a Kelly fraction as a third signal. It was 
removed after identifying that its core assumption — independently and 
identically distributed returns — cannot be justified for financial 
time series. Keeping it would have added complexity without adding 
rigour, which is precisely the failure mode *An Introduction to 
Statistical Learning* warned against.

**Why long-only?**
Simplicity and interpretability.

---

## Out-of-Sample Backtest

To evaluate the model honestly, signals were calculated using only 
training data and applied to unseen test periods. Three market regimes were tested:

| Period | Training Data | Test Period |
|--------|--------------|-------------|
| Bull Market | 2010–2015 | 2016–2018 |
| COVID Crash & Recovery | 2010–2018 | 2019–2021 |
| Rate Hiking Cycle | 2010–2021 | 2022–2024 |

### Results

| Period | Model Return | Model Sharpe | Benchmark Return | Benchmark Sharpe |
|--------|-------------|--------------|-----------------|------------------|
| Bull Market (2016-2018) | 4.4% | -0.124 | 6.4% | 0.179 |
| COVID Crash & Recovery (2019-2021) | 13.4% | 1.016 | 17.3% | 0.918 |
| Rate Hiking Cycle (2022-2024) | 0.2% | -0.538 | 1.3% | -0.299 |

Benchmark: equal-weight portfolio across the same tickers.

---

## Interpretation

The model behaves consistently with its design. The inverse volatility 
signal tilts the portfolio toward defensive, low-volatility assets — 
primarily bond ETFs (AGG, TIP) — which provides better risk-adjusted 
returns during volatile periods (COVID crash and recovery, Sharpe 
1.016 vs benchmark 0.918) but drags performance in sustained bull 
markets where equities outperform.

The rate hiking cycle (2022-2024) exposes the model's key structural 
vulnerability: heavy bond exposure proved costly when rising rates 
systematically hit fixed income assets — a risk the volatility signal 
does not capture, since bonds had historically low volatility in the 
training data.

---

## Limitations

- Signals calculated using historical data are noisy estimators of 
  future risk and return — past volatility does not guarantee future 
  volatility
- The equal-weight benchmark is a limited comparison point; a 60/40 
  stock/bond benchmark would be more conventional
- Ticker selection itself introduces implicit bias — the model's 
  behaviour is sensitive to which assets are included
- Transaction costs and rebalancing frequency are not modelled





