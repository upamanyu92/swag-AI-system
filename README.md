# swag-AI-system

Autonomous, continuous-learning algorithmic trading architecture for the Indian equity market (NSE/BSE).

## Objective

Build an always-online, zero-prompt trading agent that:

1. Continuously evaluates current holdings for hold/scale/liquidate actions.
2. Continuously scans the full Indian equity universe to identify the best active opportunities.
3. Produces live, actionable recommendations on a real-time dashboard.
4. Learns from outcomes and safely updates policy over time.

## Core System Design

### 1) Dual-factor daily analysis

- **Factor 1: Portfolio intelligence**
  - Ingest broker portfolio state (positions, average price, unrealized PnL, margin headroom).
  - Re-evaluate holdings using fundamentals, momentum, order-book pressure, and sentiment.
  - Generate actions: hold, add, trim, hedge, or exit.

- **Factor 2: Market-wide active opportunity discovery**
  - Scan NSE/BSE symbols with pre-filters (volume shockers, breakouts, abnormal volatility, sentiment spikes).
  - Route shortlisted symbols through the full model + agent consensus pipeline.
  - Output the top conviction opportunities with risk-aware position sizing.

### 2) Event-driven data and execution topology

| Component | Technology | Primary Function in Trading System |
| :--- | :--- | :--- |
| API Gateway | FastAPI / Nginx | Serves as the single entry point, managing SSE connections for streaming live price updates and routing traffic to backend services. |
| Order Management System (OMS) | Python Microservice | Validates incoming orders against margin limits, performs exchange order-routing logic, and publishes order states (placed, executed) to Kafka. |
| Kafka Event Stream | Apache Kafka | Acts as the durable backbone, decoupling ingestion from analytics. Real-time tick data, order executions, and sentiment scores are published to partitioned topics, ensuring fault tolerance. |
| Message Broker | Redis Pub/Sub | Handles transient live data that does not require durable storage, broadcasting sub-millisecond price updates to the user dashboard. |
| Exchange Gateway Processor | WebSocket Clients | Translates internal normalized order formats into broker-specific payloads (e.g., Zerodha, FYERS) and maintains persistent market feeds. |

- **API Gateway (FastAPI/Nginx):** single ingress; streams live updates to clients (SSE/WebSocket).
- **Kafka:** durable event backbone for ticks, order states, sentiment scores, model outputs.
- **Redis Pub/Sub:** ultra-low-latency fanout for dashboard state changes.
- **OMS microservice:** pre-trade risk checks, routing, compliance checks, broker order execution.
- **Exchange gateways:** persistent broker/exchange WebSocket adapters (e.g., Zerodha, FYERS).
- **TimescaleDB:** historical tick/orderbook/sentiment storage for replay + retraining.
- **PostgreSQL OrderDB:** transactional order lifecycle + audit trail.

### 3) Multi-modal feature fusion

Unified state representation combines:

- **Technical/microstructure:** OHLCV features, RSI/MACD/EMA, depth imbalance from Level-2 order book.
- **Fundamental:** valuation, leverage, cash-flow, quality and trend scores from screening providers.
- **Sentiment/NLP:** India-focused FinBERT-style sentiment probabilities + optional emotion dimensions.

Fusion is performed with a multi-input architecture (e.g., 1D-CNN/LSTM branches + dense fundamental branch + transformer sentiment embeddings) followed by joint decision layers.

### 4) Multi-agent decision orchestration

Role-specialized agents collaborate on each decision:

- Fundamental Analyst
- Sentiment Strategist
- Quantitative Technician
- Portfolio & Risk Manager

Agent disagreements are resolved through a stateful orchestration graph with conditional routing and risk-weighted consensus.

### 5) Continuous learning with DRL

- DRL environment modeled as MDP using market state, position state, and execution constraints.
- Candidate algorithms: PPO, SAC, A2C (FinRL/Stable-Baselines3 style setup).
- Reward integrates risk-adjusted return, slippage, drawdown control, and compliance penalties.
- Walk-forward/out-of-sample validation required to reduce regime-overfit risk.

### 6) MLOps and safe deployment

- **Airflow DAGs** orchestrate nightly/periodic retraining and evaluation.
- **Drift detection** (data, concept, prediction drift) triggers retraining when thresholds breach.
- **Champion-Challenger** deployment:
  - Champion executes live orders.
  - Challenger runs shadow inference on the same feed.
  - Promote only when challenger outperforms under risk constraints.
- **MLflow** tracks model versions, metrics, artifacts, and promotion decisions.

## Real-time dashboard behavior

No chat prompt required. The system continuously publishes:

- Current holdings diagnostics and action recommendations.
- Most active opportunities and top conviction picks.
- Risk state (exposure, margin usage, concentration).
- Model/agent confidence and rationale signals.

## Compliance-first execution (SEBI-aligned)

Execution layer must enforce:

- Static IP and broker/exchange policy controls.
- Immutable order/audit trail.
- Order-rate throttling (including hard orders-per-second safeguards).
- Pre-trade risk checks before every live order.
- Policy-level penalties for attempted non-compliant actions.

## Minimal implementation roadmap

1. Establish event bus + normalized market data schema.
2. Build OMS/risk/compliance guardrails before autonomous execution.
3. Add feature pipelines (technical, fundamental, sentiment) and fusion model.
4. Integrate multi-agent orchestration for decision consensus.
5. Add DRL training loop + replay + walk-forward evaluation.
6. Add drift monitoring, retraining DAGs, and champion-challenger release flow.
7. Expose zero-prompt real-time dashboard.

## References

1. CFA Institute Research (RL in investment management)
2. FinRL framework literature (Columbia University)
3. CrewAI documentation
