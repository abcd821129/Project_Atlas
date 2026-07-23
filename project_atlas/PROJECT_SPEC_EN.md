# Global AI MT5 Trader — Project Specification

## 1. Goal

Build a macOS-based system that scans a controlled global market universe, ranks trading opportunities, selects an approved strategy autonomously, validates the decision through deterministic risk controls, and later routes approved signals to an MT5 broker demo account.

The system must not claim guaranteed profit. Version 1 must not trade a real-money account.

## 2. Core architecture

```text
Market data + news + macro calendar + Binance order flow
                     ↓
             Data normalization
                     ↓
        Deterministic global scanner
                     ↓
        Ranked candidate instruments
                     ↓
       Deep multi-timeframe analysis
                     ↓
       Autonomous strategy selection
                     ↓
       Deterministic decision validator
                     ↓
          Independent hard risk gate
                     ↓
          MT5 symbol mapping + EA
                     ↓
        MT5 broker demo execution
                     ↓
        Logging, review and research
```

Principles:

- Programs scan broadly; the AI compares only a small candidate set.
- AI may select only registered and tested strategies.
- AI never controls position size or hard risk limits.
- Any missing, stale, conflicting, or invalid data results in `NO_TRADE`.
- MT5 is the execution layer, not the global scanning layer.

## 3. Version 1 market universe

### Forex
EURUSD, GBPUSD, USDJPY, AUDUSD, USDCAD, USDCHF, EURJPY, GBPJPY.

### Commodities
XAUUSD, XAGUSD, USOIL, UKOIL.

### Indices
US500, NAS100, US30.

### Crypto
BTCUSDT and ETHUSDT. Binance order flow may be used only for corresponding crypto instruments.

### Stocks
Up to 200 liquid US stocks, prioritizing major Nasdaq-100 and S&P 500 constituents, unusual-volume stocks, and liquid stocks with major news or earnings events.

All symbols must be configurable. Do not hard-code the universe in strategy logic.

## 4. Repository structure

```text
global-ai-mt5-trader/
├── PROJECT_SPEC.md
├── README.md
├── pyproject.toml
├── .env.example
├── .gitignore
├── config/
│   ├── settings.yaml
│   ├── universe.yaml
│   ├── opportunity_weights.yaml
│   ├── strategies.yaml
│   ├── risk_limits.yaml
│   ├── symbol_mapping.yaml
│   └── data_providers.yaml
├── src/global_ai_mt5/
│   ├── app/
│   ├── data_layer/
│   ├── scanner/
│   ├── analysis/
│   ├── strategies/
│   ├── orderflow/
│   ├── intelligence/
│   ├── execution/
│   └── review/
├── mql5/Experts/Global_AI_Trader_Bridge.mq5
├── tests/
├── data/
├── logs/
├── reports/
├── strategy_versions/
└── scripts/
```

Use Python type hints, Pydantic schemas, structured logging, dependency injection where useful, and deterministic tests.

## 5. Standard data models

Create provider-independent schemas for:

- instrument metadata;
- OHLCV bars;
- quotes and spreads;
- trades;
- order-book snapshots and updates;
- news events;
- macro events;
- provider health;
- scan results;
- strategy evaluations;
- final decisions;
- execution reports;
- trade outcomes.

Normalize timestamps to UTC. Preserve source name, source timestamp, receipt timestamp, latency and quality score.

Reject stale, incomplete, duplicated, future-dated or logically invalid data.

## 6. Global scanner

Do not call an LLM for every instrument. Calculate deterministic features and an `OpportunityScore` from 0 to 100.

Feature families:

- trend: EMA structure, slope, ADX and price structure;
- momentum: RSI, MACD and KDJ, without double-counting correlated indicators;
- volatility: ATR percentile, Bollinger bandwidth and volatility expansion;
- structure: Donchian position, range break, HH/HL/LH/LL;
- volume: volume anomaly and VWAP relationship;
- event context: news and macro-risk flags;
- execution quality: spread, liquidity and data quality;
- multi-timeframe context.

Initial configurable weights:

- trend 20%;
- volatility 15%;
- volume 10%;
- structure 15%;
- event context 15%;
- multi-timeframe 15%;
- liquidity and cost 10%.

Per scan:

1. Rank within each asset class.
2. Retain no more than three candidates per class.
3. Merge candidates.
4. Remove excessive correlation and duplicated directional exposure.
5. Return no more than ten candidates for deep analysis.

## 7. Market regimes

Classify candidates as:

- `TRENDING_UP`;
- `TRENDING_DOWN`;
- `RANGING`;
- `BREAKOUT_UP`;
- `BREAKOUT_DOWN`;
- `HIGH_VOLATILITY`;
- `LOW_VOLATILITY`;
- `EVENT_RISK`;
- `UNCERTAIN`.

Classification must be deterministic in Version 1. Conflicting or low-confidence states become `UNCERTAIN`.

## 8. Approved strategy registry

Version 1 strategies:

- `TREND_FOLLOWING`;
- `TREND_PULLBACK`;
- `BREAKOUT`;
- `MEAN_REVERSION`;
- `VWAP_REVERSION`;
- `ORDERFLOW` — BTC and ETH only;
- `NO_TRADE`.

All strategies implement one interface and return a structured evaluation containing compatibility, score, confidence, action, entry zone, invalidation level, target, reason codes and explanation.

The AI must not invent or run unregistered strategies.

## 9. Autonomous strategy selection

For every deeply analyzed candidate:

1. Determine market regime.
2. Exclude incompatible strategies.
3. Score each remaining registered strategy.
4. Apply conservative performance priors from sufficiently large historical samples.
5. Penalize event risk, poor liquidity, excessive spread, stale data and conflicting signals.
6. Select the highest valid strategy only when it clears configured thresholds.
7. Select `NO_TRADE` when scores are too low, too close, unstable or conflicting.

The router output must include every candidate strategy score and rejection reason.

Strategy switching controls:

- minimum hold bars;
- switch cooldown bars;
- minimum score improvement;
- maximum daily switches;
- no immediate reversal solely because the selected strategy changed.

AI may lower a strategy's priority or recommend suspension. It may not modify core strategy code, hard risk limits or live deployment state.

## 10. News and macro events

News is context, not an order trigger.

Extract:

- affected instruments and asset classes;
- publication time and event time;
- source and reliability;
- duplicate status;
- independent confirmation count;
- novelty;
- direction and estimated impact;
- estimated duration;
- whether price appears to have already reacted.

High-impact macro events include central-bank decisions, CPI, PPI, payrolls, GDP, PMI, oil inventories, major earnings and material regulation.

Version 1 should block new entries around configured high-impact windows rather than attempt to predict releases.

## 11. Binance order flow

Only BTCUSDT and ETHUSDT initially.

Implement:

- WebSocket connection and reconnection;
- local order-book reconstruction from snapshot plus ordered deltas;
- sequence-gap detection and forced resynchronization;
- best bid/ask;
- trades and aggressor classification;
- bid/ask volume;
- bar delta and cumulative delta;
- order-book imbalance;
- large-trade detection;
- liquidity walls and gaps;
- cancellation velocity;
- absorption and delta divergence features.

An unsynchronized order book is unusable and must force `NO_TRADE` for order-flow strategies.

Footprint Version 1 outputs structured data, CSV and database records. Complex heatmap visualization is deferred.

## 12. AI decision layer

The AI receives only structured summaries of the top candidates, not raw full-market feeds or every tick.

The final response must be strict JSON containing:

- scan ID;
- selected symbol;
- asset class;
- market regime and confidence;
- selected registered strategy and version;
- action: BUY, SELL or HOLD;
- confidence and opportunity score;
- timeframe;
- entry zone;
- stop loss and take profit proposal;
- news and macro risk;
- order-flow confirmation where applicable;
- reason codes;
- rejected candidates and reasons;
- data-quality status.

A deterministic validator checks schema, prices, timestamps, allowed symbols, registered strategy, logical stop/target placement and signal age. Invalid output becomes `NO_TRADE`.

## 13. MT5 integration

Use a Python service on macOS and an MQL5 EA bridge inside MT5. Do not depend on direct macOS use of the MetaTrader5 Python package for execution.

The EA must:

- report available broker symbols and contract specifications;
- report account type, quote, spread, trading session and positions;
- receive only validated decisions;
- calculate position size deterministically;
- enforce hard risk limits;
- prevent duplicate orders;
- send demo orders to the broker server;
- verify retcodes and actual positions;
- ensure stop loss and take profit are active;
- manage existing positions safely when Python is unavailable;
- persist and recover state after restart.

Symbol mapping must be explicitly confirmed. Name similarity alone must never authorize a trade.

Operating modes:

- `SIGNAL_ONLY`;
- `DEMO_LIVE`;
- `REAL_LIVE`, permanently disabled in Version 1.

A real-money account must be rejected in code and must not be enabled by a configuration change.

## 14. Independent hard risk gate

Initial defaults, configurable but not AI-editable:

- risk per trade: 0.25%;
- daily loss limit: 1%;
- maximum account drawdown: 5%;
- maximum consecutive losses: 3;
- maximum open positions: 2;
- maximum correlated positions: 1;
- maximum one position per asset class initially;
- minimum AI confidence: 75;
- minimum opportunity score: 70;
- all trades require a broker-side stop loss.

Prohibited:

- martingale;
- unlimited grids;
- averaging down;
- loss-based position increases;
- AI-selected position size;
- removal or widening of a stop beyond approved risk;
- automatic risk-limit increases;
- automatic switch to a real-money account.

Any failed check produces `STOP_NEW_ENTRIES`.

## 15. Review and controlled learning

Record each decision and trade with the complete information set available at decision time.

Analyze performance by:

- instrument;
- strategy;
- market regime;
- asset class;
- session;
- volatility state;
- event-risk state;
- data provider;
- execution cost;
- spread and slippage;
- backtest versus demo divergence.

Track sample size, expectancy, profit factor, payoff ratio, maximum drawdown, consecutive losses, Sharpe and Sortino where statistically meaningful.

Use `Champion` and `Challenger` versions:

- Champion is immutable while running.
- Challenger may be generated, backtested, tested out of sample, walk-forward tested and shadow run.
- Version 1 must not automatically promote Challenger to Champion.
- Hard risk controls cannot be changed by either version.
- Suspended strategies do not automatically resume.

## 16. Development phases

### Phase 1 — deterministic scanner only

Build:

- package structure and configuration;
- standard schemas;
- historical/mock data adapters;
- indicators and features;
- opportunity scoring;
- candidate ranking;
- correlation filtering;
- deterministic market-regime classifier;
- approved strategy interfaces and registry stubs;
- unit tests and fixtures.

Do not add live providers, news, LLM calls, Binance WebSockets, MT5 or order execution in Phase 1.

### Phase 2 — live market-data adapters

Add controlled providers, health checks, rate limiting, reconnect logic and data-quality monitoring.

### Phase 3 — Binance order flow

Add synchronized order book, trades, delta and footprint features.

### Phase 4 — news, macro and AI decision layer

Add event analysis, candidate comparison, strict JSON decision and deterministic validation.

### Phase 5 — MT5 `SIGNAL_ONLY`

Add EA bridge, symbol mapping and chart/status panel without orders.

### Phase 6 — broker demo execution

Add hard risk gate, sizing, duplicate protection, broker demo orders, fill verification and local position safety.

### Phase 7 — review and controlled research

Add reporting, performance tracking, Champion/Challenger and shadow validation.

## 17. Phase 1 acceptance criteria

Phase 1 is complete only when:

- all package imports work;
- configuration is validated;
- deterministic fixtures produce repeatable results;
- indicators have edge-case tests;
- data-quality failures are tested;
- opportunity scores remain within 0–100;
- ranking enforces per-class and total limits;
- correlation filter prevents concentrated candidates;
- market-regime tests cover all states, including `UNCERTAIN`;
- strategy registry rejects unknown strategies;
- tests pass without network access;
- README explains installation and Phase 1 commands;
- one intentional Git commit is created.

Stop after Phase 1 and report:

- files created or changed;
- architecture decisions;
- commands run;
- test results;
- known limitations;
- next recommended task.
