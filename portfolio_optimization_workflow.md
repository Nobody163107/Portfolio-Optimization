# Portfolio Optimization — Complete Project Workflow

> **Objective:** Minimize portfolio financial risk (variance) subject to return, weight, and budget constraints using 9 stocks + Gold (Markowitz Mean-Variance Framework), implemented in Jupyter Notebooks.

---

## Folder Structure

```
portfolio_optimization/
│
├── data/
│   ├── raw/                        # Downloaded CSVs directly from yfinance (untouched)
│   └── processed/                  # Cleaned, merged, aligned price data
│
├── notebooks/
│   ├── 01_data_collection.ipynb
│   ├── 02_exploratory_analysis.ipynb
│   ├── 03_statistical_analysis.ipynb
│   ├── 04_optimization.ipynb
│   └── 05_visualization.ipynb
│
├── outputs/
│   ├── figures/                    # All saved plots (PNG/HTML)
│   └── results/                    # Optimal weights, frontier data (CSV/JSON)
│
├── requirements.txt
└── README.md
```

---

## Asset Universe

| # | Asset | Ticker | Sector |
|---|-------|--------|--------|
| 1 | Apple | AAPL | Technology |
| 2 | Microsoft | MSFT | Technology |
| 3 | NVIDIA | NVDA | Technology |
| 4 | JPMorgan Chase | JPM | Finance |
| 5 | Goldman Sachs | GS | Finance |
| 6 | ExxonMobil | XOM | Energy |
| 7 | Johnson & Johnson | JNJ | Healthcare |
| 8 | Amazon | AMZN | Consumer |
| 9 | Procter & Gamble | PG | Consumer Staples |
| 10 | Gold ETF | GLD | Commodity (Hedge) |

> **Why Gold?** Gold has a low or negative correlation with equities — it acts as a hedge and makes the covariance matrix more interesting for optimization.

---

## Mathematical Framework

### Notation

| Symbol | Meaning |
|--------|---------|
| `w` | Weight vector (10×1) |
| `μ` | Expected return vector (10×1) |
| `Σ` | Covariance matrix (10×10) |
| `R_target` | Minimum required portfolio return |

### Optimization Problem

```
Minimize:     wᵀ Σ w                    ← Portfolio Variance (Risk)

Subject to:
  wᵀ μ  ≥  R_target                    ← Expected return constraint
  Σ wᵢ  =  1                           ← Budget constraint (fully invested)
  wᵢ   ≥  0    for all i               ← Long-only (no short selling)
  wᵢ   ≤  0.4  (optional cap)          ← Max concentration per asset
```

> This is a **Quadratic Program (QP)** — convex, so a unique global minimum exists.

---

---

## Notebook 1 — Data Collection (`01_data_collection.ipynb`)

### Goal
Download historical adjusted closing prices for all 10 assets and save clean data for downstream use.

### Steps

#### Step 1 — Define your tickers and date range
- Create a Python list of all 10 tickers
- Set a start date (e.g., `2019-01-01`) and end date (e.g., `2024-12-31`) — 5 years of daily data is standard but 3-4 months is enough for this specific purpose


#### Step 2 — Download data using `yfinance`
- Use `yf.download()` with `auto_adjust=True` to get adjusted prices
- The `Adj Close` column accounts for dividends and splits — always use this, not raw `Close`
- Reference: [yfinance docs](https://ranaroussi.github.io/yfinance/index.html)

#### Step 3 — Save raw data to `data/raw/`
- Save each ticker's data individually as `AAPL.csv`, `MSFT.csv`, etc.
- Do NOT modify anything — this is your backup

#### Step 4 — Clean and process
- Merge all assets into a single DataFrame indexed by date
- Keep only the `Adj Close` column for each ticker
- Handle missing values:
  - Use `ffill()` (forward fill) for gaps due to holidays
  - Drop any rows where more than 1 ticker has NaN (early data gaps)
- Rename columns to ticker names for clarity

#### Step 5 — Save processed data to `data/processed/`
- Save the final merged DataFrame as `prices_clean.csv`
- This single file is what all subsequent notebooks will import

### Data Flow

```
yf.download(tickers, start, end)
        ↓
Save raw CSVs → data/raw/TICKER.csv
        ↓
Merge on Date index (outer join)
        ↓
Keep only Adj Close columns
        ↓
Forward-fill gaps → Drop remaining NaNs
        ↓
Save → data/processed/prices_clean.csv
```

### Key Libraries
- `yfinance` — data download
- `pandas` — data manipulation and CSV I/O
- `os` / `pathlib` — directory and file path management

### Reference
- [yfinance multi-ticker download](https://ranaroussi.github.io/yfinance/index.html)
- [Pandas missing data handling](https://pandas.pydata.org/docs/user_guide/missing_data.html)

---

## Notebook 2 — Exploratory Analysis (`02_exploratory_analysis.ipynb`)

### Goal
Understand and visually inspect the data before any computation.

### Steps

#### Step 1 — Load processed data
- Read `data/processed/prices_clean.csv` with date as index
- Check shape: should be ~1250 rows (trading days) × 10 columns

#### Step 2 — Data quality checks
- `df.isnull().sum()` — confirm no missing values remain
- `df.describe()` — check min/max prices for anomalies
- `df.dtypes` — confirm all columns are float

#### Step 3 — Plot raw prices
- Line chart of all 10 assets on the same axes
- Highlights: NVDA vs Gold have very different price scales — mention this

#### Step 4 — Normalize prices (base 100)
- Divide each column by its first value and multiply by 100
- Now all assets start at 100 — easier to compare growth rates
- Formula: `normalized = prices / prices.iloc[0] * 100`

#### Step 5 — Observations to note
- Which assets grew the most? Which were most volatile?
- How does Gold behave relative to tech stocks?
- Any major dips (COVID crash March 2020, rate hike periods 2022)?

### Key Libraries
- `matplotlib.pyplot` — line plots
- `seaborn` — styling
- `pandas` — data loading and normalization

### Reference
- [Matplotlib tutorials](https://matplotlib.org/stable/tutorials/index.html)
- [Seaborn line plot guide](https://seaborn.pydata.org/tutorial/relational.html)

---

## Notebook 3 — Statistical Analysis (`03_statistical_analysis.ipynb`)

### Goal
Compute all required statistics: returns, expected returns, standard deviations, covariance matrix, and correlation matrix.

---

### Part A — Compute Returns

#### Simple Returns vs Log Returns
- **Simple returns:** `r_t = (P_t - P_{t-1}) / P_{t-1}` → use `df.pct_change()`
- **Log returns:** `r_t = ln(P_t / P_{t-1})` → use `np.log(df / df.shift(1))`
- For portfolio optimization, **log returns** are preferred (time-additive, better statistical properties)
- Reference: [Simple vs Log Returns — QuantEcon](https://python.quantecon.org/)

---

### Part B — Expected Returns (Annualized)

#### Formula
```
μ_i = mean(daily log returns of asset i) × 252
```

- `252` = number of trading days in a year
- Store as a Series indexed by ticker name
- This is your **μ vector** used in the optimization constraint

#### Interpretation
- μ_i = 0.15 means asset i is expected to return 15% per year
- Compare across assets — Gold typically has lower expected return but also lower risk

---

### Part C — Standard Deviations (Annualized)

#### Formula
```
σ_i = std(daily log returns of asset i) × √252
```

- Use `ddof=1` (sample standard deviation) in `df.std()`
- Store as a Series — this is per-asset risk

#### Interpretation
- σ_i = 0.20 means ±20% annual volatility (1 standard deviation range)
- Higher σ = higher risk = more volatile asset

---

### Part D — Covariance Matrix (Annualized)

#### Formula
```
Σ = df_returns.cov() × 252
```

- Result is a 10×10 symmetric matrix
- Diagonal entries = variance of each asset (σ²)
- Off-diagonal entries = covariance between pairs of assets
- This matrix **directly enters** the objective function: `wᵀ Σ w`

#### What to look for
- Large positive covariance between AAPL and MSFT (both tech — move together)
- Near-zero or negative covariance between Gold (GLD) and tech stocks (diversification benefit)

#### Reference
- [Pandas .cov() docs](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cov.html)
- [NYU Stern — Covariance in Finance](https://pages.stern.nyu.edu/~adamodar/New_Home_Page/lectures/cov.html)

---

### Part E — Correlation Matrix

#### Formula
```
ρ_{ij} = Cov(i,j) / (σ_i × σ_j)
```

- `df_returns.corr()` computes Pearson correlation by default
- Values range from -1 to +1
- **+1** = perfect positive correlation, **-1** = perfect negative, **0** = no linear relationship

#### Visualization
- Plot as a **heatmap** using `seaborn.heatmap()` with `annot=True`
- Use a diverging colormap (e.g., `coolwarm`) centered at 0

#### Key Insight
- Assets with low/negative correlation reduce portfolio risk when combined
- Gold's low correlation with equities is the core reason it's included

#### Reference
- [Seaborn heatmap docs](https://seaborn.pydata.org/generated/seaborn.heatmap.html)
- [Correlation in Portfolio Context — CFA Institute](https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/portfolio-mathematics)

---

### Summary Table to Generate

At the end of this notebook, compile a summary DataFrame:

| Asset | Expected Return (μ) | Std Dev (σ) | Sharpe (approx) |
|-------|-------------------|------------|-----------------|
| AAPL | 0.XX | 0.XX | 0.XX |
| ... | ... | ... | ... |
| GLD | 0.XX | 0.XX | 0.XX |

> Sharpe ≈ μ / σ (simplified, assuming risk-free rate ≈ 0 or subtract 0.05)

---

## Notebook 4 — Optimization (`04_optimization.ipynb`)

### Goal
Formulate the QP and solve it to find optimal weights. Then trace the **Efficient Frontier** by solving for a range of target returns.

---

### Part A — Single Optimization (Minimum Variance Portfolio)

#### Solver: CVXPY (Recommended)

CVXPY lets you write the optimization in math-like syntax that maps directly to your formulation.

**Conceptual structure (no code — understand the steps):**

1. Define `w` as a CVXPY variable of size 10
2. Define the objective: minimize `quad_form(w, Sigma)` — this is `wᵀ Σ w`
3. Add constraints:
   - `sum(w) == 1` (budget)
   - `mu @ w >= R_target` (return constraint)
   - `w >= 0` (no short selling)
4. Create and solve the `Problem`
5. Extract `w.value` — the optimal weights vector

Reference: [CVXPY Portfolio Optimization Example](https://www.cvxpy.org/examples/finance/portfolio_optimization.html)

---

### Part B — Efficient Frontier

#### Concept
The efficient frontier is the set of portfolios that offer the **maximum return for a given level of risk** (or equivalently, minimum risk for a given return).

#### How to trace it
1. Create a range of R_target values from `min(μ)` to `max(μ)` — e.g., 50 evenly spaced values
2. For each R_target, solve the optimization
3. Record: `portfolio_return = mu @ w.value`, `portfolio_risk = sqrt(wᵀ Σ w)`
4. Plot risk (x-axis) vs return (y-axis) — the curve is the efficient frontier

#### What to also plot
- Individual assets as scatter points on the same graph
- The **Minimum Variance Portfolio** (leftmost point on the curve)
- The **Maximum Sharpe Ratio Portfolio** (tangency point — requires a risk-free rate assumption)

Reference: [Efficient Frontier — Investopedia](https://www.investopedia.com/terms/e/efficientfrontier.asp)

---

### Part C — Alternative Solver: PyPortfolioOpt

If you want a higher-level approach with less boilerplate:
- `EfficientFrontier` class handles the QP internally
- Built-in methods: `min_volatility()`, `max_sharpe()`, `efficient_return()`
- Reference: [PyPortfolioOpt Getting Started](https://pyportfolioopt.readthedocs.io/en/latest/UserGuide.html)

---

### Part D — Save Results

- Save optimal weights to `outputs/results/optimal_weights.csv`
- Save efficient frontier data (risk, return pairs) to `outputs/results/efficient_frontier.csv`

---

### Solver Comparison

| Solver | Level | Best For |
|--------|-------|----------|
| `scipy.optimize.minimize` | Low-level | Learning — full manual control |
| `cvxpy` | Mid-level | Clean math-to-code mapping, good for reports |
| `PyPortfolioOpt` | High-level | Quick results, finance-specific methods |

> **Recommendation:** Use `cvxpy` as primary, mention `PyPortfolioOpt` as a validation check.

---

## Notebook 5 — Visualization (`05_visualization.ipynb`)

### Goal
Present all results visually in a clear, publication-quality format.

---

### Plot 1 — Optimal Weights (Bar Chart)
- X-axis: Asset names
- Y-axis: Weight (0 to 1)
- Highlights which assets the optimizer selected heavily

### Plot 2 — Efficient Frontier Curve
- X-axis: Portfolio Risk (σ)
- Y-axis: Portfolio Return (μ)
- Mark: Minimum variance point, Maximum Sharpe point
- Overlay: Individual asset risk-return scatter

### Plot 3 — Correlation Heatmap
- 10×10 heatmap with `annot=True`
- Diverging colormap (`coolwarm`)
- Clearly shows diversification structure

### Plot 4 — Cumulative Returns Comparison
- Optimal portfolio vs Equal-weight portfolio vs S&P 500 (SPY)
- Shows out-of-sample performance of the optimized weights

### Plot 5 — Risk-Return Summary Table (as figure)
- Table showing μ, σ, and optimal weight for each asset
- Can be rendered as a styled DataFrame or a matplotlib table

### Saving Plots
- Save all figures to `outputs/figures/` as PNG files
- Use `plt.savefig('outputs/figures/plot_name.png', dpi=150, bbox_inches='tight')`

---

## Requirements File

```
# requirements.txt
yfinance>=0.2.0
pandas>=2.0.0
numpy>=1.24.0
matplotlib>=3.7.0
seaborn>=0.12.0
scipy>=1.11.0
cvxpy>=1.4.0
PyPortfolioOpt>=1.5.0
plotly>=5.15.0
jupyterlab>=4.0.0
```

Install all at once:
```bash
pip install -r requirements.txt
```

---

## Key Academic & Reference Sources

| Topic | Source |
|-------|--------|
| Original Markowitz Theory | [Markowitz (1952) — Portfolio Selection, JSTOR](https://www.jstor.org/stable/2975974) |
| Python Finance & Returns | [QuantEcon — Python for Finance](https://python.quantecon.org/) |
| Risk & Return Fundamentals | [NYU Stern — Damodaran](https://pages.stern.nyu.edu/~adamodar/) |
| CVXPY QP Tutorial | [CVXPY Portfolio Optimization](https://www.cvxpy.org/examples/finance/portfolio_optimization.html) |
| PyPortfolioOpt Guide | [PyPortfolioOpt Docs](https://pyportfolioopt.readthedocs.io/en/latest/UserGuide.html) |
| Efficient Frontier Explanation | [Investopedia — Efficient Frontier](https://www.investopedia.com/terms/e/efficientfrontier.asp) |
| Correlation & Covariance | [CFA Institute — Portfolio Mathematics](https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/portfolio-mathematics) |
| yfinance API | [yfinance Documentation](https://ranaroussi.github.io/yfinance/index.html) |
| Pandas Reference | [Pandas Documentation](https://pandas.pydata.org/docs/) |
| Seaborn Heatmap | [Seaborn heatmap](https://seaborn.pydata.org/generated/seaborn.heatmap.html) |

---

## Execution Order

```
01_data_collection.ipynb
        ↓  (produces: data/processed/prices_clean.csv)
02_exploratory_analysis.ipynb
        ↓  (produces: outputs/figures/normalized_prices.png)
03_statistical_analysis.ipynb
        ↓  (produces: μ vector, Σ matrix, correlation heatmap)
04_optimization.ipynb
        ↓  (produces: outputs/results/optimal_weights.csv,
                       outputs/results/efficient_frontier.csv)
05_visualization.ipynb
        ↓  (produces: all final plots in outputs/figures/)
```

---

## Quick Checklist

- [ ] 10 assets downloaded (9 stocks + GLD for Gold)
- [ ] Prices use Adjusted Close (accounts for dividends/splits)
- [ ] Returns computed as log returns, annualized ×252
- [ ] Standard deviations annualized ×√252
- [ ] Covariance matrix annualized ×252
- [ ] Correlation matrix plotted as heatmap
- [ ] Optimization uses `wᵀ Σ w` as objective
- [ ] Three constraints implemented: budget, return, non-negativity
- [ ] Efficient frontier traced over range of R_target values
- [ ] Optimal weights saved to CSV
- [ ] All figures saved to `outputs/figures/`
