# Integrating Kelly Criterion with Technical Indicators for VN30

This repository contains the Jupyter notebooks and data used in the project:

> **“Integrating Kelly Criterion with Technical Indicators for VN30 Stock Market”**

The project studies how to combine **RSI** with other classic technical indicators (**MACD, OBV, Bollinger Bands**) and the **Kelly criterion** for position sizing on stocks in the **VN30 index** (30 of the largest and most liquid stocks listed on Ho Chi Minh Stock Exchange).

The code implements the full experimental pipeline from the paper/presentation: signal generation, Kelly-parameter estimation, and backtesting under Vietnam’s **T+2 settlement rule**.

---

## 1. Project Overview

We investigate three indicator combinations:

- **RSI + MACD**
- **RSI + OBV**
- **RSI + Bollinger Bands (BB)**

For each combo, we:

1. Define rule-based **buy/sell signals** using daily data.
2. Estimate **winning rate** `p` and **decimal odds** `b` on a training period.
3. Use the **Kelly fraction**

   \[
   f = \frac{p (1 + b) - 1}{b}
   \]

   to size trades.
4. Backtest strategies on VN30 stocks with an initial budget of **100 million VND**, respecting **T+2 settlement**.

---

## 2. Trading Strategies

All strategies use **daily** prices and indicators.

### 2.1 RSI–MACD

- Indicators: **RSI(14)** and **MACD(12, 26, 9)**.
- **Buy** when:
  - `RSI > 30` **and** MACD line crosses **above** Signal line.
- **Sell** when:
  - `RSI < 70` **and** MACD line crosses **below** Signal line.

### 2.2 RSI–OBV

- Indicators: **RSI(14)** and **OBV slope over 5 days**.
- **Buy** when:
  - `RSI > 30` **and** OBV slope **> 0`.
- **Sell** when:
  - `RSI < 70` **and** OBV slope **< 0`.

### 2.3 RSI–BB (Bollinger Bands)

- Indicators: **RSI(14)** and **Bollinger Bands(15, 2)**.
- **Buy** when:
  - `RSI ≤ 30` **and** price is at or below the **lower band**.
- **Sell** when:
  - `RSI ≥ 70` **and** price is at or above the **upper band**.

---

## 3. Experimental Setup

From the original experiments:

- **Estimation period (for `p` and `b`):**  
  2019-01-01 → 2021-01-02  
- **Backtest period:**  
  2021-01-01 → 2024-01-02  
- **Universe:** 30 VN30 stocks.
- **Initial capital:** 100,000,000 VND.
- **Evaluation horizon for wins/losses:**  
  Every time a trade is opened in the estimation period:
  - Compare price at entry vs. price **20 trading days later**.
  - If later price > entry price ⇒ **win**, else **loss**.
- **Decimal odds `b`:** ratio of **mean profit** to **mean loss** in estimation period.
- **Kelly fraction `f`:** computed from `p` and `b`, then used for position sizing in backtest.

### 3.1 Metrics

Performance is summarized with:

- **AR** – mean return of all 30 VN30 stocks.
- **AP** – mean return of stocks with **positive** returns.
- **AL** – mean return of stocks with **negative** returns.
- **VL** – standard deviation of returns across the 30 stocks.

We also track:

- **Average return (%)** over the period.
- **Maximum loss (%)** (worst drawdown per strategy).

### 3.2 High-level Findings

From the reported results:

- **RSI–MACD** is most effective in **recession/downtrend** periods (e.g. 2022).
- **RSI–BB** and **RSI–OBV** are more suitable in **uptrend** periods (e.g. 2023).
- Applying **Kelly sizing**:
  - Tends to **reduce maximum loss** roughly by half (e.g. −28% → −14%, etc.).
  - Can slightly **reduce average return**, but improves **risk control**, especially in 2022.

---

## 4. Repository Structure

Top-level structure of this repo:

```text
.
├── data/
├── find_pq_kelly_top3_indication/
├── kelly_t2_sell_not_divide/
├── rsi+indicators/
├── rsi_indicator_t2_top3/
├── .gitignore
└── CITATION.cff
```

- `data/`  
  Input data (e.g. VN30 stock list and daily OHLCV series).  
  If your clone does not include data files, place your CSVs here; see the notebooks for the exact file paths they expect.

- `rsi+indicators/`  
  Notebooks that compute **RSI**, **MACD**, **OBV**, **Bollinger Bands**, and generate trading signals for VN30 stocks.

- `find_pq_kelly_top3_indication/`  
  Notebooks that estimate the **winning rate `p`** and **decimal odds `b`** for the top-3 indicator combinations, then compute the **Kelly fractions** used later in backtesting.

- `rsi_indicator_t2_top3/`  
  Notebooks for backtesting the indicator-based strategies under the **T+2** settlement rule, typically without Kelly position sizing.

- `kelly_t2_sell_not_divide/`  
  Notebooks that apply the **Kelly fraction** on top of the indicator strategies and run **T+2** backtests, focusing on capital allocation per trade.

- `CITATION.cff`  
  Machine-readable citation information for the associated paper/presentation.

---

## 5. Getting Started

### 5.1 Clone the repository

```bash
git clone https://github.com/usagichan131/stock-algo.git
cd stock-algo
```

### 5.2 Create a Python environment

You’ll need a recent Python 3 version (3.8+ recommended) plus the usual data-science stack.

Example (using `venv`):

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scriptsctivate
pip install --upgrade pip
```

Install typical dependencies (adjust based on imports in the notebooks):

```bash
pip install numpy pandas matplotlib jupyter ta
# plus any additional packages you see imported in the notebooks
```

### 5.3 Launch Jupyter

```bash
jupyter notebook
# or
jupyter lab
```

Open the notebooks in each folder in this rough order:

1. `rsi+indicators/`
2. `find_pq_kelly_top3_indication/`
3. `rsi_indicator_t2_top3/`
4. `kelly_t2_sell_not_divide/`

Follow the instructions/markdown cells in each notebook to reproduce the calculations and figures.

---

## 6. Data

The experiments use **daily VN30 stock data** over 2019–2024.

Typical layout (you can adapt as needed):

```text
data/
├── vn30_symbols.csv           # List of VN30 tickers
├── prices/
│   ├── <TICKER1>.csv
│   ├── <TICKER2>.csv
│   └── ...
```

Each price CSV usually contains at least:

- `date`
- `open`, `high`, `low`, `close`
- `volume`

If your data comes from another source (e.g. broker API, `vnstock`, etc.), adjust the notebooks’ loading code accordingly.

---

## 7. Reproducing the Experiments

A typical workflow using the notebooks:

1. **Prepare Data**
   - Place historical VN30 data in `data/`.
   - Make sure the column names and file paths match those expected in the notebooks.

2. **Generate Indicator Signals**
   - In `rsi+indicators/`, run the notebooks to compute RSI, MACD, OBV, Bollinger Bands and generate buy/sell signals for each stock.

3. **Estimate Kelly Parameters**
   - In `find_pq_kelly_top3_indication/`, run the notebooks that:
     - Simulate trades over the **2019–2021** estimation window.
     - Compute winning rate `p` and decimal odds `b` from 20-day forward returns.
     - Export or store the resulting **Kelly fractions** for each strategy.

4. **Backtest Without Kelly**
   - In `rsi_indicator_t2_top3/`, backtest RSI-MACD, RSI-OBV and RSI-BB strategies over **2021–2024**:
     - Apply the **T+2** settlement rule.
     - Compute AR, AP, AL, VL, average return, and max loss.

5. **Backtest With Kelly**
   - In `kelly_t2_sell_not_divide/`, re-run the backtests using **Kelly-sized positions**:
     - Compare equity curves and performance metrics to the no-Kelly baseline.
     - Pay attention to changes in **maximum loss** and **return volatility**.

---

## 8. Results Summary (from the paper)

From the reported tables:

- Average returns (2021–2023, across VN30 stocks):

  - RSI–MACD  
    - Without Kelly: **16.6%**  
    - With Kelly: **7.44%**
  - RSI–OBV  
    - Without Kelly: **16.19%**  
    - With Kelly: **14.9%**
  - RSI–BB  
    - Without Kelly: **14.7%**  
    - With Kelly: **12.2%**

- Maximum losses across the period:

  - RSI–MACD  
    - Without Kelly: **−28%**  
    - With Kelly: **−14%**
  - RSI–OBV  
    - Without Kelly: **−29%**  
    - With Kelly: **−17%**
  - RSI–BB  
    - Without Kelly: **−33%**  
    - With Kelly: **−22%**

These numbers illustrate the main trade-off studied in the project: **Kelly sizing reduces downside risk and drawdowns, at the cost of somewhat lower mean returns**.

---

## 9. Limitations & Future Work

Limitations (from the presentation):

- No explicit **portfolio optimization** across stocks; each stock is treated independently.
- Kelly parameters are estimated **once** from a fixed window (no online updating).
- Real-world issues like transaction costs, slippage, and liquidity constraints are kept simple.

Planned / suggested extensions:

- Use **deep learning** or other ML models on top of the indicator signals to adapt to different market regimes.
- Incorporate portfolio construction (e.g. risk parity, CVaR optimization) together with Kelly position sizing.
- Extend to other Vietnamese or regional indices beyond VN30.

---

## 10. Citation

If you use this code or build on this work in your research, please cite the associated paper/presentation. You can also use the `CITATION.cff` file directly with GitHub’s “Cite this repository” feature.

Example BibTeX-style entry (adapt as needed):

```text
@inproceedings{dinh2025kellyvn30,
  title     = {Integrating Kelly Criterion with Technical Indicators for VN30 Stock Market},
  author    = {Vy Dinh Dao Lan and Hang Tran Ngoc and May Vo Huyen Khanh and Hung Tran and Minh Tran Duc and Lan Dang Thu and Van Nhan Vo},
  year      = {2025},
  note      = {DATCOM Lab, College of Technology, National Economics University},
}
```

---

## 11. License

No explicit license file is currently included in this repository.  
If you plan to use or redistribute this code (especially for commercial purposes), please contact the authors or repository owner and/or add an appropriate `LICENSE` file.

---

## 12. Contact

- **Vy Dinh Dao Lan** – <lanvy181004@gmail.com>  
- **Hung Tran** – <hung.tran@neu.edu.vn>  

Or open an issue / pull request in this GitHub repository.
