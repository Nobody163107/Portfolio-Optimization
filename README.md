# Portfolio Optimization Project

## Overview

This project implements a **Markowitz Mean-Variance Portfolio Optimization** model to construct an optimal investment portfolio. Using modern portfolio theory, the algorithm maximizes expected returns while minimizing risk through efficient asset allocation across a diversified set of securities.

The project demonstrates practical application of quadratic programming in finance, utilizing historical market data to compute optimal portfolio weights that balance risk and return.

## Features

- **Data Collection**: Automated download of historical stock prices using Yahoo Finance API
- **Statistical Analysis**: Computation of expected returns, volatilities, and covariance matrices
- **Portfolio Optimization**: Quadratic programming solver for efficient frontier calculation
- **Risk Management**: Incorporation of correlation analysis and diversification principles
- **Visualization**: Interactive plots for portfolio performance and risk-return analysis

## Mathematical Framework

### Objective Function
Minimize portfolio variance (risk):
$$\min \frac{1}{2} \mathbf{w}^T \Sigma \mathbf{w}$$

Subject to:
- Expected return constraint: $\mathbf{w}^T \mu \geq r_{\text{target}}$
- Budget constraint: $\mathbf{w}^T \mathbf{1} = 1$
- Non-negativity: $\mathbf{w} \geq 0$

Where:
- $\mathbf{w}$: Portfolio weights vector
- $\Sigma$: Covariance matrix of asset returns
- $\mu$: Expected returns vector
- $r_{\text{target}}$: Minimum required return

### Decision Variables
- $w_i$: Weight allocated to asset $i$ (fraction of total portfolio)
- Constraints: $0 \leq w_i \leq 1$ and $\sum w_i = 1$

## Asset Universe

The portfolio consists of 10 assets across different sectors:

| Asset | Ticker | Sector |
|-------|--------|--------|
| Apple Inc. | AAPL | Technology |
| Microsoft Corp. | MSFT | Technology |
| NVIDIA Corp. | NVDA | Technology |
| JPMorgan Chase | JPM | Financial Services |
| Goldman Sachs | GS | Financial Services |
| Exxon Mobil | XOM | Energy |
| Johnson & Johnson | JNJ | Healthcare |
| Amazon.com | AMZN | Consumer Discretionary |
| Procter & Gamble | PG | Consumer Staples |
| SPDR Gold Shares | GLD | Commodities |

## Project Structure

```
Portfolio-Optimization/
├── README.md                           # Project documentation
├── requirements.txt                    # Python dependencies
├── portfolio_optimization_workflow.md  # Detailed implementation guide
├── test.py                             # API connectivity test
├── data/
│   ├── raw/                           # Raw data storage
│   └── processed/                     # Cleaned and processed data
├── notebooks/
│   ├── 1_data_collection.ipynb        # Data acquisition and preprocessing
│   ├── 2_stat_analysis.ipynb          # Statistical analysis and optimization
│   └── test.ipynb                     # Testing and validation
└── outputs/
    ├── figures/                       # Generated plots and visualizations
    └── results/                       # Optimization results and reports
```

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd Portfolio-Optimization
```

2. Create a virtual environment:
```bash
python -m venv pfo
source pfo/bin/activate  # On Windows: pfo\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

## Usage

### Data Collection
Run the data collection notebook to fetch historical price data:
```bash
jupyter notebook notebooks/1_data_collection.ipynb
```

### Statistical Analysis and Optimization
Execute the analysis notebook for portfolio optimization:
```bash
jupyter notebook notebooks/2_stat_analysis.ipynb
```

## Dependencies

- **yfinance**: Financial data download
- **pandas**: Data manipulation and analysis
- **numpy**: Numerical computations
- **cvxpy**: Convex optimization
- **matplotlib/seaborn**: Data visualization
- **scipy**: Scientific computing
- **plotly**: Interactive visualizations

## Results

The optimization produces:
- **Optimal Portfolio Weights**: Asset allocation percentages
- **Efficient Frontier**: Risk-return tradeoff curve
- **Risk Metrics**: Portfolio volatility, Sharpe ratio
- **Performance Analysis**: Historical backtesting results

## Methodology

1. **Data Acquisition**: Download 3-month daily price data for selected assets
2. **Return Calculation**: Compute daily returns and annualize statistics
3. **Covariance Estimation**: Build covariance matrix from historical returns
4. **Optimization**: Solve quadratic programming problem for optimal weights
5. **Validation**: Analyze portfolio performance and risk characteristics

## Key Insights

- Diversification significantly reduces portfolio risk
- Technology sector shows highest expected returns but also highest volatility
- Gold (GLD) serves as an effective hedge against market downturns
- Optimal portfolios balance growth potential with risk management

## Future Enhancements

- Extend analysis period for more robust statistical estimates
- Implement multi-period optimization
- Add transaction cost constraints
- Incorporate macroeconomic factors
- Develop web-based portfolio dashboard

## References


- Modern Portfolio Theory principles
- CVXPY documentation for optimization
- Yahoo Finance API for market data

## Project Summary

This portfolio optimization project demonstrates the practical application of Modern Portfolio Theory through:

- **Data-Driven Analysis**: Systematic collection and processing of financial time series
- **Statistical Modeling**: Computation of risk and return metrics for asset allocation
- **Optimization Techniques**: Implementation of quadratic programming for efficient portfolios
- **Risk Management**: Diversification strategies to balance return and volatility

The project showcases skills in quantitative finance, data analysis, and optimization programming, making it an excellent demonstration of technical expertise for CV purposes.

## Contact

For questions or collaboration opportunities, please refer to the project documentation or reach out through professional channels.

---

*Built with Python, CVXPY, and financial data analysis best practices.* 