# 📊 Data Model: Autonomous Day Trading Agent

## 1. Overview

This data model defines the schema for storing and managing:

- Real-time and historical market data
- Technical indicators and model predictions
- Trade decisions and executions
- Strategy performance metrics
- Audit logs for compliance and debugging

---

## 2. Entity Relationship Diagram (ERD) Overview

[Symbol]───< [MarketData]
│
├───< [TechnicalIndicator]
│
├───< [ModelPrediction]
│
├───< [TradeDecision] ───< [TradeExecution]
│
└───< [StrategyPerformance]


---

## 3. Entities and Attributes

### 🟦 Symbol
Stores metadata about a financial instrument.

| Field          | Type     | Description                     |
|----------------|----------|---------------------------------|
| symbol_id      | UUID     | Unique identifier               |
| ticker         | String   | Stock ticker (e.g., AAPL)       |
| exchange       | String   | Exchange name (e.g., NASDAQ)    |
| asset_type     | String   | Stock, ETF, Crypto, etc.        |
| is_active      | Boolean  | If symbol is tradable           |
| created_at     | DateTime | Record creation timestamp       |

---

### 📈 MarketData
Stores OHLCV and live tick data.

| Field          | Type     | Description                     |
|----------------|----------|---------------------------------|
| market_data_id | UUID     | Unique identifier               |
| symbol_id      | UUID     | FK to Symbol                    |
| timestamp      | DateTime | Timestamp of quote              |
| open           | Float    | Opening price                   |
| high           | Float    | Highest price                   |
| low            | Float    | Lowest price                    |
| close          | Float    | Closing price                   |
| volume         | Float    | Volume traded                   |
| interval       | String   | 1min, 5min, 1h, 1d              |
| source         | String   | Data provider (e.g., Polygon)   |

---

### 🔍 TechnicalIndicator
Pre-computed indicators used by the model.

| Field             | Type     | Description                   |
|-------------------|----------|-------------------------------|
| indicator_id      | UUID     | Unique ID                     |
| symbol_id         | UUID     | FK to Symbol                  |
| timestamp         | DateTime | Timestamp                     |
| indicator_name    | String   | e.g., RSI, MACD               |
| indicator_value   | Float    | Computed value                |
| period            | Int      | Lookback window size          |
| computed_by       | String   | "engine", "external", etc.    |

---

### 🧠 ModelPrediction
Stores ML output for future price movement or trade score.

| Field            | Type     | Description                   |
|------------------|----------|-------------------------------|
| prediction_id    | UUID     | Unique ID                     |
| symbol_id        | UUID     | FK to Symbol                  |
| timestamp        | DateTime | Prediction time               |
| predicted_price  | Float    | Target price forecast         |
| probability_up   | Float    | Likelihood of price increase  |
| confidence_score | Float    | Model confidence [0, 1]       |
| model_version    | String   | Git hash or semantic version  |

---

### 🧾 TradeDecision
Stores the agent’s internal trade decisions.

| Field            | Type     | Description                       |
|------------------|----------|-----------------------------------|
| decision_id      | UUID     | Unique ID                         |
| symbol_id        | UUID     | FK to Symbol                      |
| timestamp        | DateTime | Decision time                     |
| action           | String   | BUY, SELL, HOLD                   |
| size             | Float    | Quantity (e.g., # of shares)      |
| predicted_roi    | Float    | Expected ROI of the trade         |
| model_reference  | UUID     | FK to ModelPrediction             |
| approved         | Boolean  | Passed all risk filters?          |

---

### ✅ TradeExecution
Logs of executed trades.

| Field            | Type     | Description                     |
|------------------|----------|---------------------------------|
| execution_id     | UUID     | Unique ID                       |
| decision_id      | UUID     | FK to TradeDecision             |
| symbol_id        | UUID     | FK to Symbol                    |
| executed_price   | Float    | Filled price                    |
| quantity         | Float    | Final executed size             |
| status           | String   | FILLED, REJECTED, CANCELED      |
| broker           | String   | Alpaca, IBKR, etc.              |
| latency_ms       | Int      | Milliseconds to execute         |
| timestamp        | DateTime | Trade execution time            |

---

### 📊 StrategyPerformance
Tracks ongoing strategy-level metrics.

| Field              | Type     | Description                    |
|--------------------|----------|--------------------------------|
| strategy_id        | UUID     | Unique strategy identifier     |
| symbol_id          | UUID     | FK to Symbol                   |
| window_start       | DateTime | Analysis window start          |
| window_end         | DateTime | Analysis window end            |
| total_trades       | Int      | Number of trades               |
| win_rate           | Float    | % of profitable trades         |
| avg_roi            | Float    | Mean ROI across trades         |
| max_drawdown       | Float    | Worst peak-to-trough drop      |
| sharpe_ratio       | Float    | Sharpe Ratio                   |

---

### 📜 AuditLog
All system events for traceability.

| Field           | Type     | Description                    |
|-----------------|----------|--------------------------------|
| log_id          | UUID     | Unique ID                      |
| timestamp       | DateTime | Event time                     |
| event_type      | String   | MODEL_RUN, TRADE_SUBMIT, etc.  |
| actor           | String   | Agent, human override, system  |
| payload         | JSON     | Structured metadata            |
| success         | Boolean  | If the event completed         |

---

## 4. Notes
- All `timestamp` fields should be stored in UTC.
- Indexing should exist on `symbol_id`, `timestamp`, and `strategy_id` for performance.
- Consider adding a `backtest_session_id` to each entity to support simulation environments.

---

## 5. Next Steps
- Define schema in a Postgres-compatible DDL
- Optionally, map to ORM models for Python (e.g., SQLAlchemy or Django ORM)
- Configure Kafka or Redis for real-time event streaming (for low-latency updates)

---
