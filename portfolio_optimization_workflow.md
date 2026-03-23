# Portfolio Optimization — Complete Project Workflow

> **Objective:** Minimize portfolio financial risk (variance) subject to return, weight, and budget constraints using 9 stocks + Gold (Markowitz Mean-Variance Framework), implemented in Jupyter Notebooks.

---

## Folder Structure (Actual Git Repository)

```
Portfolio-Optimization/
│
├── .gitignore
├── README.md                              # Project documentation
├── requirements.txt                       # Python dependencies
├── portfolio_optimization_workflow.md     # This workflow guide
│
└── notebooks/
    ├── 1_data_collection.ipynb            # Data acquisition & return computation
    ├── 2_stat_analysis.ipynb              # Statistical analysis + optimization
    ├── Data.csv                           # Summary stats (first approach)
    ├── returns_data.csv                   # Full daily returns matrix (all 10 assets)
    └── summary_data.csv                   # Annualized return & STD summary
```

> **Note:** The project uses a streamlined 2-notebook structure. Data files (CSVs) are stored alongside the notebooks for simplicity.

---

## Asset Universe

| #   | Asset             | Ticker | Sector            |
| --- | ----------------- | ------ | ----------------- |
| 1   | Apple             | AAPL   | Technology        |
| 2   | Microsoft         | MSFT   | Technology        |
| 3   | NVIDIA            | NVDA   | Technology        |
| 4   | JPMorgan Chase    | JPM    | Finance           |
| 5   | Goldman Sachs     | GS     | Finance           |
| 6   | ExxonMobil        | XOM    | Energy            |
| 7   | Johnson & Johnson | JNJ    | Healthcare        |
| 8   | Amazon            | AMZN   | Consumer          |
| 9   | Procter & Gamble  | PG     | Consumer Staples  |
| 10  | Gold ETF          | GLD    | Commodity (Hedge) |

---

## Mathematical Framework

### Notation

| Symbol     | Meaning                           |
| ---------- | --------------------------------- |
| `w`        | Weight vector (10×1)              |
| `μ`        | Expected return vector (10×1)     |
| `Σ`        | Covariance matrix (10×10)         |
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

## 1. Data Collection (1_data_collection.ipynb)

### Goal
Download historical adjusted closing prices for 10 assets, compute returns, and save clean data for downstream use.

### Steps
1. **Define Tickers**: Create a Python list of the 10 portfolio tickers (AAPL, MSFT, NVDA, JPM, GS, XOM, JNJ, AMZN, PG, GLD).
2. **Download Data**: Use yfinance to fetch 3 months of daily data for each ticker.
3. **Compute Returns**: Extract the closing prices and compute daily percentage returns using pct_change().
4. **Calculate Stats**: Annualize the mean daily returns and standard deviations (assuming 252 trading days).
5. **Save Data**: Export the full daily returns to `notebooks/returns_data.csv` and the summary statistics to `notebooks/summary_data.csv` and `notebooks/Data.csv`.

---

## 2. Statistical Analysis and Optimization (2_stat_analysis.ipynb)

### Goal
Compute the covariance matrix and implement the Markowitz portfolio optimization algorithm to find optimal weights.

### Steps
1. **Load Data**: Read the generated `notebooks/returns_data.csv` and summary data.
2. **Covariance Matrix**: Calculate the annualized covariance matrix from the daily returns.
3. **Setup CVXPY Problem**:
   - **Decision Variable**: w, a 10x1 weight vector.
   - **Objective**: Minimize portfolio variance (cp.Minimize(w.T @ cov_matrix @ w)).
   - **Constraints**: 
     - Sum of weights = 1 (fully invested)
     - Expected return >= target return
     - All weights >= 0 (no short selling)
4. **Solve for 15% Target Return**: Run the optimizer to find the minimum variance portfolio achieving at least a 15% annualized return.
5. **Solve for 10% Target Return**: Run the optimizer again for a 10% target return.
6. **Comparison**: Display side-by-side tables comparing asset allocations and portfolio metrics between scenarios.

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

| Topic                          | Source                                                                                                                                                |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Original Markowitz Theory      | [Markowitz (1952) — Portfolio Selection, JSTOR](https://www.jstor.org/stable/2975974)                                                                 |
| Python Finance & Returns       | [QuantEcon — Python for Finance](https://python.quantecon.org/)                                                                                       |
| Risk & Return Fundamentals     | [NYU Stern — Damodaran](https://pages.stern.nyu.edu/~adamodar/)                                                                                       |
| CVXPY QP Tutorial              | [CVXPY Portfolio Optimization](https://www.cvxpy.org/examples/finance/portfolio_optimization.html)                                                    |
| PyPortfolioOpt Guide           | [PyPortfolioOpt Docs](https://pyportfolioopt.readthedocs.io/en/latest/UserGuide.html)                                                                 |
| Efficient Frontier Explanation | [Investopedia — Efficient Frontier](https://www.investopedia.com/terms/e/efficientfrontier.asp)                                                       |
| Correlation & Covariance       | [CFA Institute — Portfolio Mathematics](https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/portfolio-mathematics) |
| yfinance API                   | [yfinance Documentation](https://ranaroussi.github.io/yfinance/index.html)                                                                            |
| Pandas Reference               | [Pandas Documentation](https://pandas.pydata.org/docs/)                                                                                               |
| Seaborn Heatmap                | [Seaborn heatmap](https://seaborn.pydata.org/generated/seaborn.heatmap.html)                                                                          |

---

## Execution Order

```
1_data_collection.ipynb
        ↓  (produces: notebooks/returns_data.csv,
                       notebooks/summary_data.csv,
                       notebooks/Data.csv)
2_stat_analysis.ipynb
        ↓  (loads:    notebooks/returns_data.csv, notebooks/Data.csv)
        ↓  (computes: covariance matrix, μ vector,
                       optimal weights for 15% and 10% targets)
        ↓  (outputs:  comparison tables with allocation differences)
```

**Note**: The project uses a streamlined 2-notebook pipeline. `1_data_collection.ipynb` handles data acquisition and return computation, and `2_stat_analysis.ipynb` performs statistical analysis and optimization.

---

## Quick Checklist

- [x] 10 assets downloaded (9 stocks + GLD for Gold)
- [x] Prices use Close prices (3-month daily data via yfinance)
- [x] Returns computed as simple returns (`pct_change()`), annualized ×252
- [x] Standard deviations annualized ×√252
- [x] Covariance matrix annualized ×252
- [ ] Correlation matrix plotted as heatmap (planned for visualization)
- [x] Optimization uses `wᵀ Σ w` as objective via CVXPY
- [x] Three constraints implemented: budget, return, non-negativity
- [x] Two scenarios solved: 15% and 10% target returns
- [ ] Efficient frontier traced over range of R_target values (future enhancement)
- [ ] Optimal weights saved to CSV (results displayed in notebook output)
- [ ] All figures saved to output directory (future enhancement)
