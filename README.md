# 1. An End-to-End Machine Learning Framework for Market Time-Series Decision Making

This project implements an end-to-end machine learning–driven decision framework
for market time-series data. It automatically collects market data, engineers features, trains predictive models,
generates daily signals, integrates with broker APIs for execution, and maintains a full portfolio 
state on a cloud environment.

---

# 2. Data Sources & Storage

## 2.1 Market Data Sources
Daily market data is collected from licensed APIs, including:

- OHLCV (open, high, low, close, volume) for selected equities  
- Market index proxies (e.g., SP500)  
- Volatility indices (e.g., VIX proxy)  
- News & sentiment metadata  

## 2.2 Processing & Storage

All incoming data is:

- cleaned and aligned by (date, ticker)
- processed with time-series–aware transformations
- lagged using time-indexed shifts to avoid lookahead bias
- enriched with rolling & cross-sectional statistics  
- saved to versioned CSV files for reproducibility

---

# 3. Machine Learning Models

## 3.1 XGBoost Classifier

A supervised learning pipeline predicts next-day movement using:

- TimeSeriesSplit cross-validation  
- Optuna hyperparameter search  
- Out-of-Fold (OOF) evaluation  
- Final full-history retraining  
- Score binning for downstream logic  

Outputs include prediction probabilities per ticker per day.

## 3.2 Market Regime Detection

A lightweight model produces **Bull / Neutral / Bear** market states based on:

- trend indicators  
- volatility signals  
- momentum estimates  
- optional model-confidence adjustment  

Regime is used as a constraint/gating mechanism for trading execution.

---

# 4. Walk-Forward Pipeline

The system performs strict **walk-forward forecasting**:

1. Train on all data up to date *t*  
2. Generate predictions for date *t + 1*  
3. Append market-state label  
4. Produce T+1 execution metadata (`t1_px`, `date_next`)

The output is a consolidated `wf_df` containing:

- Date  
- Ticker  
- Model probability  
- MarketState  
- Next-day execution price info  

This guarantees no data leakage and realistic evaluation under deployment-like constraints.

---

# 5. Backtesting Engine

A custom execution simulator models a real trading account:

- T+1 open-price execution  
- Regime-dependent execution parameters  
- Position lifecycle management  
- Risk & exit constraints
- Time-based exit  
- Cooldown system to avoid rapid re-entries  
- Daily mark-to-market equity calculation  

Outputs:

- equity curve  
- trade log  
- daily account summary  

---

# 6. Trading Automation

## 6.1 Execution & Broker Integration
The live trading engine supports:

- automated order routing  
- buy/sell execution  
- position & cash state tracking  
- manual override with synchronization  
- email notifications for trade events  

## 6.2 Google Cloud VM Hosting

Daily automation includes:

1. Scheduled VM startup before market open  
2. Data ingestion + feature computation  
3. Daily model refresh  
4. scheduled pre-market execution
5. Intraday (5-minute) sell monitoring  
6. Automatic VM shutdown after market close  

This keeps operational cost minimal while maintaining reliability.

---

# 7. Performance Summary (High-Level Only)

The model exhibits:

- strong Out-of-Fold AUC  
- stable performance across most regimes  
- sensitivity to macro-volatility environments  
- consistent walk-forward behavior without leakage  

(Feature definitions and trading rules intentionally omitted.)

---

# 8. Repository Structure (Simplified)

```text
project/
│
├── data/ # Processed market data & intermediate outputs
├── models/ # Saved ML artifacts (XGBoost models, parameters)
│
├── src/
│ ├── data_pipeline/ # Daily ingestion, feature computation (time-series safe)
│ ├── model_training/ # XGBoost training & evaluation
│ ├── walk_forward/ # Rolling prediction generator
│ ├── backtest/ # T+1 execution simulator
│ ├── execution/ # Broker integration (buy/sell automation)
│ └── helper_utils/ # Shared utilities, env loaders, helpers
│
├── cloud/ # GCP startup scripts, cron tasks, automation configs
└── README.md
```

# 9. Key Highlights

The system is designed with production-level structure while maintaining strict data integrity guarantees.

### **Key characteristics:**

- **Fully automated end-to-end ML trading pipeline**  
  From data ingestion → model training → prediction → execution.

- **Strict walk-forward framework**  
  Ensures every trading-day prediction is generated without future leakage.

- **Realistic execution modeling**  
  Backtest replicates live behavior closely using T+1 fills, slippage/fees, cooldowns, and timed exits.

- **Regime-aware decision framework**  
  Market-state signals constrain trading activity during adverse environments.

- **Cloud-native automation**  
  Daily operations run on a scheduled GCP VM, minimizing manual intervention.

- **Live trading integration**  
  Alpaca API handles real executions with synchronized position and trade logging.

- **Scalable modular structure**  
  New models, features, or execution rules can be introduced with minimal refactoring.

---

# 10. Disclaimer

This repository presents the **framework and engineering components** of the system.  
Certain details—including feature definitions, weighting logic, and trading-specific parameters—are intentionally omitted to protect proprietary research and to comply with API and data-use restrictions.

The project is intended purely for educational, research, and portfolio demonstration purposes, **not** as investment advice.
