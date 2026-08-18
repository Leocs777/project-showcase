# Investor Lab Case Study

Investor Lab is a private, local-first investment research and paper-trading workspace for web and iPhone. It turns market, company, portfolio, options, and intraday evidence into reproducible planning scenarios while preserving the full decision trail. The project is in personal beta testing; it is not a public advisory service and it does not implement live-account order routing.

![Investor Lab iOS research dashboard](../assets/investor-lab-ios.png)

Production source and personal financial data remain private. This page documents the product scope and engineering decisions without exposing credentials, brokerage records, or internal history.

## Product Summary

The core problem is not obtaining one more indicator. It is keeping the assumptions behind a decision consistent across research, risk sizing, execution rehearsal, and later review.

- **Explainable stock research** — deterministic Buy candidate, Watch, Avoid, Hold, Reduce, or Sell / exit-review states with factor scores, supporting and counter-evidence, data provenance, and explicit invalidation rules.
- **Price planning** — pullback buy zones, breakout triggers, ATR-based risk references, two reward/risk targets, and position sizing tied to saved account limits.
- **Options workspace** — manual or imported option chains, DTE/spread/volume filters, IV term structure and skew, portfolio Greeks, earnings flags, and flexible one-to-six-leg payoff scenarios.
- **Day-trade planning** — opening-range breakout, VWAP pullback, and premarket-momentum setups with no-trade conditions, replay, worksheet reviews, and daily-loss stops.
- **Portfolio intelligence** — exposure, concentration, correlations, stress scenarios, action queues, performance history, alerts, and paper-only rebalance calculations.
- **Shared Web/iOS history** — watchlists, plans, reports, reviews, alerts, settings, and an append-only journal synchronize through one local backend.

## Architecture

```text
SwiftUI iOS ─┐
             ├─ authenticated local API ─ SQLite + append-only revision ledger
Web client ──┘             │
                           ├─ Alpha Vantage: end-of-day OHLCV + earnings
                           ├─ SEC EDGAR: filings + company facts
                           └─ Alpaca Paper/IEX: quotes, options, account + Paper orders
```

The backend is a dependency-free Python service with a responsive web client and a native SwiftUI client. SQLite is the system of record. Clients synchronize from a monotonically increasing revision cursor instead of replacing whole account snapshots, which keeps Web and iOS behavior consistent and makes changes auditable.

Provider credentials stay in the macOS Keychain. Browser authentication uses an HttpOnly, SameSite cookie with CSRF protection; iOS stores a device-only session token in Keychain. Passwords and session tokens are stored only as salted or cryptographic hashes.

## Engineering Highlights

### Making analysis explainable and reproducible

Strategy Lab uses versioned templates rather than a hidden model response. Balanced, Growth, Value, Income, Momentum, and custom profiles assign explicit weights across technical structure, fundamentals, valuation, position risk, and execution cost. Every saved decision retains the model version, configuration hash, thresholds, factor contributions, evidence timestamps, and change explanation.

The displayed price levels are scenarios, not forecasts. The pullback zone references the 20-day average and ATR; the breakout trigger references prior highs; the stop reference combines 50-day structure and ATR; targets use the saved minimum reward/risk. The separation prevents a convenient target from silently changing the underlying score.

### Rejecting weak data before producing an actionable state

Daily bars pass depth, freshness, OHLC consistency, calendar-gap, and split-scale discontinuity checks. A decision can remain visible for research while being marked non-actionable when evidence is stale or incomplete. SEC filings are cached and compared using only information that would have been available at the decision date.

Walk-forward tests use historical closes, filing availability, fees, slippage, liquidity surcharges, and a benchmark when SPY data is present. Later highs and lows can resolve saved targets and stops into a validation history, but the product does not present backtests as guarantees.

### Treating broker connectivity as a safety boundary

Alpaca integration is fixed to the Paper host. There is no live-account URL or live-order route. Paper order submission, replacement, and cancellation require synchronized account state, an enabled safety control, typed confirmation, per-order notional limits, and daily-loss checks. Each request first creates an auditable order intent; broker responses and fills are reconciled back into the local journal.

The default state is locked. Read-only account, position, order, quote, and option-chain synchronization can be tested without enabling order mutations.

### Keeping a local system operational

The app records collection runs, cache coverage, data quality, provider readiness, and database health in one settings surface. It creates verified SQLite backups, retains a configurable history, and requires a schema-compatible file plus an exact typed phrase before restore. A pre-restore safety backup is created before the active database is replaced.

## Technical Stack

| Layer | Choices |
| --- | --- |
| iOS | SwiftUI, URLSession, Keychain, local notifications |
| Web | Semantic HTML, CSS, vanilla JavaScript, server-sent events |
| Backend | Python standard library, threaded HTTP server, deterministic analysis engine |
| Data | SQLite, append-only journal and sync revision ledger, verified backups |
| Market research | Alpha Vantage EOD, SEC EDGAR, Alpaca IEX and option snapshots |
| Execution boundary | Alpaca Paper only; typed confirmations and synchronized risk controls |
| Localization | English and Simplified Chinese across Web and iOS |

## Testing and Current Status

The current private build passes 34 automated API and domain tests, database integrity checks, Web/iOS build verification, and physical-iPhone installation. The product is now in personal end-to-end testing with cached research data. Alpaca Paper connectivity and small simulated-order rehearsals remain deliberately separate from any use of real capital.

Current public status: **private beta, personal use, paper-only execution**.

## What I Owned

- Product scope, information architecture, and risk boundaries
- Responsive web client and native SwiftUI iOS client
- Authentication, sync protocol, SQLite schema, migrations, backup and restore
- Deterministic stock, options, day-trade, and portfolio analysis workflows
- Alpha Vantage, SEC EDGAR, and Alpaca Paper/IEX integrations
- Bilingual interface, device testing, and release-readiness tooling

## Engineering Notes

**A recommendation is only useful if it is inspectable.** Scores, thresholds, price scenarios, and data timestamps are stored with the decision so a later review can reconstruct what the system knew.

**The execution boundary should be structural.** Keeping the broker host fixed to Paper and omitting a live route is stronger than relying on a warning dialog around the same endpoint.

**Local-first still needs operations.** A private single-user app requires migrations, backup verification, health reporting, stale-data handling, and recovery paths if it is expected to hold a trustworthy decision history.
