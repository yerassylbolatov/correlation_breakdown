# Correlation Breakdown & Diversification Failure — 2004–2009

A Python-based quantitative risk analysis project that measures how asset correlations evolved before and during the 2007–2009 financial crisis, demonstrating that diversification — the foundational principle of portfolio risk management — collapsed exactly when it was needed most.

The core question this project answers: **if assets were supposedly uncorrelated, why did everything fall together in 2008?**

---

## Motivation

Modern portfolio theory tells investors to diversify — hold assets that move independently so losses in one are offset by gains in another. This works well in normal markets. But during the 2008 financial crisis, correlations between almost all asset classes spiked simultaneously, rendering diversification ineffective.

This project quantifies that breakdown using rolling correlation analysis, static regime comparison, and PCA — revealing not just that correlations increased, but **when**, **how fast**, and **which pairs were most affected**.

A key finding: the crisis did not produce a single correlation spike. The data reveals **two distinct waves of panic** separated by a deceptive calm — a pattern that lulled risk managers into complacency before the worst of the crisis hit.

---

## Tickers Analyzed

| Ticker | Description | Role in Analysis |
|--------|-------------|-----------------|
| `SPY`  | S&P 500 ETF | Market benchmark |
| `XLF`  | Financials sector ETF | Crisis epicenter |
| `GLD`  | Gold ETF | Flight-to-safety asset |
| `XLP`  | Consumer staples ETF | Defensive sector |
| `BAC`  | Bank of America | Name-brand bank, high crisis exposure |

---

## Project Structure

### `correlation_report(tickers, start_date, end_date)`

The core computation function. Downloads returns for all tickers and computes the full correlation analysis.

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `tickers` | List of ticker symbols (e.g. `["SPY", "XLF", "GLD", "XLP", "BAC"]`) |
| `start_date` | Start date in `"YYYY-MM-DD"` format |
| `end_date` | End date in `"YYYY-MM-DD"` format |

**Returns a dict containing:**

| Key | Description |
|-----|-------------|
| `returns` | DataFrame of log returns, one column per ticker |
| `corr_matrix` | Static correlation matrix for the full period |
| `rolling_corr` | DataFrame of all pairwise 60-day rolling correlations |
| `avg_corr` | Series — mean of all pairwise correlations per day |
| `variance_explained` | PCA variance explained ratio per component |
| `cumulative_var` | Cumulative variance explained |
| `n_components_80` | Number of components needed to explain 80% of variance |

---

### `plot_correlation_report(calm, stressed)`

Takes two results dicts and produces a 2×2 figure:

- **Top left** — Correlation heatmap for the calm period. Color scale from red (−1) to green (+1).
- **Top right** — Correlation heatmap for the stressed period. Same scale for direct visual comparison.
- **Bottom left** — 60-day rolling average pairwise correlation over the full 2004–2009 period, with vertical markers at the February 2007 first spike and the Lehman collapse (September 15, 2008).
- **Bottom right** — PCA bar chart showing variance explained by each principal component, calm vs stressed side by side.

---

## Key Concepts

### Correlation
Measures how two assets move together, ranging from −1 (perfectly opposite) to +1 (perfectly synchronized). Diversification relies on holding assets with low or negative correlations — losses in one asset are offset by gains in another.

### Rolling Correlation
A 60-day rolling window computes correlation using only the most recent 60 trading days, updated daily. This captures how correlations evolve over time rather than averaging across the full period — revealing spikes, troughs, and regime changes that static analysis hides.

### Average Correlation Index
The mean of all pairwise correlations on a given day — a single number summarizing how synchronized the portfolio is. Rising average correlation means diversification is shrinking. This is the project's primary time series and its clearest visual result.

### PCA — Principal Component Analysis
Identifies the hidden common drivers of portfolio variance. In a diversified portfolio, multiple independent factors drive returns. When correlations spike, a single dominant factor — systemic market fear — overwhelms everything else and explains most of the variance.

---

## Key Findings

### Two Waves of Panic

The rolling correlation chart reveals a pattern the textbook version of the crisis misses:

```
Early 2007       →  First correlation spike (~0.73) — subprime fears emerge
Mid 2007–2008    →  False calm, correlation drops to ~0.25
Sep 2008 onward  →  Second spike (~0.68) — Lehman collapse, acute crisis phase
```

The trough between the two spikes — where correlations fell back to near-calm levels — represents a period when risk managers believed the worst was over. The data shows it was a temporary reprieve before the most acute phase of the crisis.

### Gold as a Diversifier

GLD (gold) showed near-zero or slightly negative correlation with equities in both periods, confirming its role as a flight-to-safety asset. During the stressed period, GLD vs BAC correlation was −0.08 and GLD vs XLF was −0.06 — one of the few pairs that maintained genuine diversification benefit during the crisis.

### Defensive Stocks Failed to Protect

XLP (consumer staples) — traditionally considered a defensive, low-beta sector — saw its correlation with SPY rise from 0.76 to 0.85 during the acute crisis. Even assets designed to be stable got dragged down by systemic selling pressure.

### Correlation Matrix Comparison

| Pair | Calm (2004–2006) | Stressed (2008–2009) | Change |
|------|-----------------|----------------------|--------|
| SPY vs XLF | 0.85 | 0.85 | unchanged |
| SPY vs XLP | 0.76 | 0.85 | +0.09 |
| SPY vs GLD | 0.09 | 0.04 | slight decrease |
| BAC vs XLF | 0.79 | 0.89 | +0.10 |
| GLD vs XLF | 0.05 | −0.06 | decoupled |

---

## Example Usage

```python
TICKERS = ["SPY", "XLF", "GLD", "XLP", "BAC"]

# compute regime results
calm     = correlation_report(TICKERS, "2004-01-01", "2006-12-31")
stressed = correlation_report(TICKERS, "2008-09-01", "2009-03-31")

# visualize
plot_correlation_report(calm, stressed)

# print PCA summary
print(f"Calm     — PC1 explains: {calm['variance_explained'][0]:.1%}")
print(f"Stressed — PC1 explains: {stressed['variance_explained'][0]:.1%}")
print(f"Calm     — components to reach 80%: {calm['n_components_80']}")
print(f"Stressed — components to reach 80%: {stressed['n_components_80']}")
```

---

## Sample Output

```
Calm     — PC1 explains: 63.3%
Stressed — PC1 explains: 65.6%
Calm     — components to reach 80%: 2
Stressed — components to reach 80%: 2
```

The rolling average correlation chart is the project's strongest visual result — showing the two-wave panic pattern and the false calm of mid-2008 that static analysis cannot reveal.

---

## Connection to Portfolio Projects

This is the third project in a series analyzing the 2007–2009 financial crisis from complementary angles:

| Project | Question |
|---------|----------|
| VaR Crisis Simulation | Did standard risk models fail during the crisis? |
| GARCH Volatility Modeling | Why did they fail — what is volatility clustering? |
| Correlation Breakdown | What else broke — and why couldn't diversification save portfolios? |

Together these three projects form a complete quantitative narrative of the crisis: model failure, its statistical cause, and the portfolio-level consequence.

---

## Dependencies

```
yfinance
numpy
pandas
matplotlib
scipy
seaborn
scikit-learn
```

Install with:
```bash
pip install yfinance numpy pandas matplotlib scipy seaborn scikit-learn
```

---

## Author

Built as a quantitative finance portfolio project focusing on correlation dynamics, diversification failure, and the application of PCA to financial returns during the 2007–2009 financial crisis.
