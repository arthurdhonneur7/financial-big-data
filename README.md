# Lead–Lag Effect Analysis Between SPY and AAPL

[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![EPFL](https://img.shields.io/badge/EPFL-Financial%20Big%20Data-red.svg)](https://www.epfl.ch/)

> An intraday analysis of high-frequency lead–lag effects between the S&P 500 ETF (SPY) and Apple Inc. (AAPL) using rolling calibrations and cross-correlation methods.

## Authors

| Name | Email |
|------|-------|
| Raphaël Eliakim | raphael.eliakim@epfl.ch |
| David Mourier | david.mourier@epfl.ch |
| Arthur Dhonneur | arthur.dhonneur@epfl.ch |

**Institution:** École Polytechnique Fédérale de Lausanne (EPFL)  
**Course:** Financial Big Data  
**Date:** January 2026

---

## Abstract

This study investigates the presence of lead–lag effects between the S&P 500 ETF (SPY) and Apple Inc. (AAPL) stock at high-frequency time scales. Using intraday trade data from 2009 sampled at minute and second resolutions, we analyze cross-correlations, rolling window correlations, and volume–price relationships to detect information transmission between these two assets.

**Key Findings:**
- Contemporaneous correlations between AAPL and SPY returns are strong (~0.6–0.8)
- Lead–lag effects are negligible at the minute level
- Marginal lead–lag effects become detectable at the second level (avg. lagged correlations ~0.1)
- Results are consistent with the Efficient Market Hypothesis in highly liquid markets

---

## Table of Contents

- [Features](#features)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Data Requirements](#data-requirements)
- [Usage](#usage)
- [Methodology](#methodology)
- [Results Summary](#results-summary)
- [Technical Challenges](#technical-challenges)
- [References](#references)
- [License](#license)

---

## Features

- **High-frequency data processing** using Polars for memory-efficient computation
- **Multi-resolution analysis** at minute-level and second-level granularity
- **Rolling window correlations** at 30-second, 60-second, and daily horizons
- **Cross-correlation analysis** for lead–lag detection
- **Statistical significance testing** using t-tests, Fisher Z-transformations, and bootstrap methods
- **BBO (Best Bid-Offer) spread analysis** across multiple large-cap equities (AAPL, AMZN, MSFT, GOOGL)
- **Conditional analysis** on high-volume trading days
- **Distributional analysis** with Q-Q plots and ACF analysis

---

## Project Structure

```
Financial Big Data/
├── code/
│   ├── cleaning_data.ipynb      # Data preprocessing and cleaning pipeline
│   └── main_project.ipynb       # Main analysis notebook
├── data/
│   ├── clean/
│   │   ├── merged/
│   │   │   ├── merged_1min.parquet                # Minute-level merged trade data
│   │   │   ├── merged_1sec.parquet                # Second-level merged trade data
│   │   │   ├── merged_BBO_1min.parquet            # Minute-level merged BBO data
│   │   │   ├── merged_BBO_1sec.parquet            # Second-level merged BBO data
│   │   │   ├── merged_BBO_1min_4assets.parquet    # Minute-level merged BBO data (4 assets)
│   │   │   └── merged_BBO_1sec_4assets.parquet    # Second-level merged BBO data (4 assets)
│   │   └── US/
│   │       ├── trade/
│   │       │   ├── trade_AAPL.parquet              # Cleaned trade data (AAPL)
│   │       │   └── trade_SPY.parquet               # Cleaned trade data (SPY)
│   │       └── BBO/
│   │           ├── BBO_AAPL_1.parquet
│   │           ├── BBO_AAPL.parquet
│   │           ├── BBO_AMZN.parquet
│   │           ├── BBO_GOOGL.parquet
│   │           ├── BBO_MSFT.parquet
│   │           └── BBO_SPY.parquet
│   └── raw/
│       ├── FR/
│       └── US/
│           ├── trade/
│           │   ├── AAPL.OQ/
│           │   └── SPY.P/
│           └── BBO/
│               ├── AAPL.OQ/
│               ├── AMZN.OQ/
│               ├── GOOGL.OQ/
│               ├── MSFT.OQ/
│               └── SPY.P/
├── plots/                       # Generated figures
│   ├── fig_acf_AAPL.png
│   ├── fig_acf_SPY.png
│   ├── fig_bbo_crosscorr_*.png
│   ├── fig_cross_correlation_*.png
│   ├── fig_histogram_*.png
│   ├── fig_qqplot_*.png
│   └── fig_rolling_correlation_*.png
├── report/
│   └── Project Requirements.pdf
├── README.md
├── requirements.txt
└── venv/                        # Python virtual environment
```

---

## Installation

### Prerequisites

- Python 3.11 or higher
- pip package manager

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/lead-lag-spy-aapl.git
   cd "Financial Big Data"
   ```

2. **Create and activate virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

### Dependencies

The project requires the following Python packages:

```
polars>=0.20.0
numpy>=1.24.0
scipy>=1.11.0
matplotlib>=3.7.0
pandas>=2.0.0
statsmodels>=0.14.0
pyarrow>=14.0.0
jupyter>=1.0.0
```

---

## Data Requirements

### Trade Data
- **Assets:** SPY (S&P 500 ETF) and AAPL (Apple Inc.)
- **Period:** Calendar year 2009
- **Frequency:** Tick-level data aggregated to minute and second intervals
- **Format:** Parquet files
- **Fields:** timestamp, trade price, trade volume

### BBO Quote Data
- **Assets:** AAPL, AMZN, MSFT, GOOGL
- **Period:** 2015–2017 (varies by asset)
- **Fields:** timestamp, bid price, bid volume, ask price, ask volume

### Data File Naming Convention

Raw trade data files follow this naming pattern:
```
{YYYY-MM-DD}-{TICKER}.{EXCHANGE}-trade.parquet
```

Examples:
- `2009-01-02-AAPL.OQ-trade.parquet` (AAPL on NASDAQ)
- `2009-01-02-SPY.P-trade.parquet` (SPY on NYSE Arca)

### Processed Data

After running `cleaning_data.ipynb`, the following merged files are generated in `data/clean/`:

| File | Description | Observations |
|------|-------------|--------------|
| `merged_1min.parquet` | Minute-level trade data | ~98,000 |
| `merged_1sec.parquet` | Second-level trade data | ~2.3 million |
| `merged_BBO_1min.parquet` | Minute-level BBO quotes | varies |

---

## Usage

### Step 1: Data Cleaning

Run the data cleaning notebook first to preprocess raw data:

```bash
cd code
jupyter notebook cleaning_data.ipynb
```

This notebook will:
- Import and clean trade/BBO data from raw parquet files
- Convert timestamps and filter to regular trading hours (9:30 AM – 4:00 PM ET)
- Aggregate data to minute and second intervals using VWAP
- Compute log returns and normalize all variables (z-score)
- Save cleaned data to `data/clean/`

### Step 2: Main Analysis

Run the main analysis notebook:

```bash
jupyter notebook main_project.ipynb
```

This notebook performs:
- Distributional analysis (histograms, Q-Q plots, ACF)
- Cross-correlation analysis at various lags
- Rolling window correlation analysis
- Volume-return spread analysis
- Conditional high-volume day analysis
- Statistical significance testing
- BBO spread cross-correlation analysis

All generated plots are saved to the `plots/` directory.

### Quick Start Example

```python
import polars as pl

# Load preprocessed data
df_min = pl.read_parquet("../data/clean/merged_1min.parquet")
df_sec = pl.read_parquet("../data/clean/merged_1sec.parquet")

# Select normalized columns for analysis
df_normalized = df_min.select([
    "datetime", 
    "price_AAPL_normalized", 
    "volume_AAPL_normalized", 
    "return_AAPL_normalized",
    "price_SPY_normalized", 
    "volume_SPY_normalized", 
    "return_SPY_normalized"
])

# Compute cross-correlation at lag k
def cross_corr_at_lag(df_1, df_2, lag):
    arr1, arr2 = df_1.to_numpy(), df_2.to_numpy()
    if lag > 0:
        return np.corrcoef(arr1[lag:], arr2[:-lag])[0, 1]
    elif lag < 0:
        return np.corrcoef(arr1[:lag], arr2[-lag:])[0, 1]
    return np.corrcoef(arr1, arr2)[0, 1]
```

---

## Methodology

### 1. Cross-Correlation Analysis

Compute the cross-correlation function between AAPL and SPY returns at various lags:

$$\rho_{XY}(k) = \text{Corr}(X_{t+k}, Y_t) = \frac{\text{Cov}(X_{t+k}, Y_t)}{\sigma_X \sigma_Y}$$

- **Positive lag (k > 0):** Tests if SPY leads AAPL
- **Negative lag (k < 0):** Tests if AAPL leads SPY

### 2. Rolling Window Correlations

Instead of static train/validation/test splits (e.g., 60/20/20), we employ rolling windows to capture time-varying relationships:

$$\rho_{t,w} = \text{Corr}(r^{AAPL}_{t-w:t}, r^{SPY}_{t-w:t})$$

Window sizes analyzed:
- 30 seconds (high-frequency dynamics)
- 60 seconds (short-term patterns)
- 1 day (daily regime behavior)

### 3. Statistical Significance Testing

Multiple testing approaches:
- **t-tests** for mean correlation
- **Fisher Z-transformation** for confidence intervals
- **Bootstrap confidence intervals** (non-parametric)
- **Normality testing** (Shapiro-Wilk / Kolmogorov-Smirnov)

### 4. Return Spread Analysis

Define the return spread as:

$$\Delta r_t = r^{AAPL}_t - r^{SPY}_t$$

Analyze cross-correlation between spread and trading volume to understand how volume shocks affect relative returns.

---

## Results Summary

### Cross-Correlation Results

| Time Scale | Lag | Mean Correlation | 95% CI (Fisher) | Significance |
|------------|-----|------------------|-----------------|--------------|
| Second | 0 | 0.65 | [0.64, 0.66] | Yes (p < 0.001) |
| Second | +1s | 0.10 | [0.09, 0.11] | Yes (p < 0.001) |
| Second | −1s | 0.09 | [0.08, 0.10] | Yes (p < 0.001) |
| Minute | 0 | 0.75–0.80 | — | Yes |
| Minute | +1min | 0.008 | [0.004, 0.012] | Economically negligible* |
| Minute | −1min | −0.011 | [−0.015, −0.007] | Economically negligible* |

*Statistically significant due to large sample size, but too small for profitable trading

### Key Observations

1. **Strong contemporaneous correlation** (0.6–0.8) at all time scales
2. **No significant lead–lag at minute level** — correlations drop to near zero at non-zero lags
3. **Weak but persistent effects at second level** — correlations ~0.1 at ±1 second
4. **High-volume days do not amplify lead–lag effects**
5. **BBO spreads show strong co-movements** with correlations peaking at lag zero

### Implications

- Results support the **Efficient Market Hypothesis** in highly liquid markets
- Exploitable lead–lag relationships likely exist only at **sub-second frequencies**
- Profitable exploitation requires **high-frequency trading infrastructure**

---

## Generated Plots

The `plots/` directory contains all figures generated during analysis:

| Plot | Description |
|------|-------------|
| `fig_histogram_*.png` | Return distribution histograms with Gaussian fit |
| `fig_qqplot_*.png` | Q-Q plots showing fat-tailed behavior |
| `fig_acf_*.png` | Autocorrelation function plots |
| `fig_cross_correlation_*.png` | Cross-correlation at different lags |
| `fig_rolling_correlation_*.png` | Time-varying rolling correlations |
| `fig_bbo_crosscorr_*.png` | BBO spread cross-correlations |

---

## Technical Challenges

1. **Data size and memory management:** Processing 2.3M+ second-level observations required using Polars instead of Pandas
2. **Time zone handling:** Proper alignment of timestamps across data sources (UTC to America/New_York)
3. **Missing data:** Not all seconds/minutes have trades, requiring careful handling during aggregation
4. **Multiple testing:** Controlling for false discoveries across many hypothesis tests
5. **Non-Gaussian distributions:** Fat-tailed returns (kurtosis ~8–12) necessitated robust statistical methods

---

## Suggested Extensions

- **Higher-frequency data:** Millisecond or microsecond analysis
- **Additional assets:** Extend to more S&P 500 constituents
- **Event studies:** Earnings announcements, Fed decisions
- **Machine learning:** Non-linear relationship detection (LSTM, transformers)
- **Trading strategy backtesting:** Profitability analysis after transaction costs
- **Order book data:** Deeper microstructure analysis (LOB dynamics)

---

## References

1. Hasbrouck, J. (2003). Intraday price formation in US equity index markets. *The Journal of Finance*, 58(6), 2375-2400.

2. Chordia, T., Roll, R., & Subrahmanyam, A. (2011). Recent trends in trading activity and market quality. *Journal of Financial Economics*, 101(2), 243-263.

3. Hendershott, T., Jones, C. M., & Menkveld, A. J. (2011). Does algorithmic trading improve liquidity? *The Journal of Finance*, 66(1), 1-33.

4. Cont, R. (2001). Empirical properties of asset returns: stylized facts and statistical issues. *Quantitative Finance*, 1(2), 223-236.

5. Aldridge, I. (2013). *High-frequency trading: a practical guide to algorithmic strategies and trading systems*. John Wiley & Sons.

6. Delattre, S., & Jacod, J. (2013). A central limit theorem for normalized functions of the increments of a diffusion process, in the presence of round-off errors. *Bernoulli*, 19(1), 137-170.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

We thank the Financial Big Data course instructors at EPFL for guidance on project requirements and methodology.

---

<p align="center">
  <i>École Polytechnique Fédérale de Lausanne (EPFL) — Financial Big Data Course — January 2026</i>
</p>