# Architecture

## Overview

Openbook is a long-running agent process. Every 30 to 60 seconds, and immediately on breaking news, it runs one cycle of the agent loop. The LLM makes the decisions. Deterministic code provides tools and guardrails.

```
            +------------------------ AGENT LOOP ------------------------+
 events --> | OBSERVE -> PLAN -> INVESTIGATE -> DECIDE -> VERIFY -> ACT  |
            +--------------------------------^---------------------+-----+
                                             |                     |
                                       MEMORY (journal)      DECISION RECORD
                                             ^                     |
                                             +----- outcomes ------+
```

## Components

| Component | Responsibility |
|---|---|
| Agent core | Runs the loop, calls the LLM, enforces the decision schema |
| Tools | The only way the agent reads the world or proposes a trade (see `TOOLS.md`) |
| Fair-value engine | Computes fair price range and confidence per bStock (see `FAIR_VALUE.md`) |
| Prosecutor | Separate LLM call that argues against each proposed trade |
| Guardrails | Code-enforced limits, sizing, loss rules (see `RISK.md`) |
| Execution | Fresh quote, order build, submit, confirm, reconcile |
| Memory | Decision journal with outcomes, retrieval, calibration (see `MEMORY.md`) |
| Point-in-time data layer | Serves only data stamped before the decision time |
| Audit layer | Decision records, version registry, daily traces (see `LOGGING_AND_AUDIT.md`) |
| Ops | Supervisor, heartbeat, alerts, recovery (see `OPERATIONS.md`) |

## Agent loop, step by step

1. **Observe.** Read wallet, gas, US session state, open positions, new events since last cycle.
2. **Plan.** The LLM chooses which bStocks deserve attention this cycle and what to check.
3. **Investigate.** The LLM calls tools: quotes, fair value, news, memory, cost estimate.
4. **Decide.** The LLM outputs a structured decision: action, thesis, fair-price range, confidence, expected edge, exit conditions, evidence list.
5. **Verify.** The prosecutor argues against the decision. The agent answers each objection or withdraws. Guardrails then run as the final check.
6. **Act.** Execution places the order through the Binance Web3 Open APIs. A decision record is written before and after.
7. **Learn.** Outcomes at several horizons are written back to memory.

For open positions, steps 2 to 5 also decide whether to hold or exit, checking whether the original thesis still holds.

## Data flow

```
Binance Web3 APIs --+
Share prices -------+--> point-in-time store --> tools --> agent
Index futures ------+          (timestamps)
News, calendars ----+
BSC RPC ------------+
```

Every fetch is stored raw with its receive time. Data older than a per-source limit is treated as missing. Missing data reduces size or stops the trade. It never triggers a guess.

## Trading behavior

- Long or flat only.
- Buy when a bStock trades below its fair-price range by more than round-trip cost plus a margin that grows as confidence falls.
- Exit when price reaches fair value, when new evidence breaks the thesis, or on a time stop once the US market opens and prices the news.
- When only the bStock moved and futures and related markets did not, treat it as mispricing. When those moved too, treat it as information and follow it.

## Failure behavior

| Failure | Response |
|---|---|
| LLM service down | Fall back to no-news mode with wider ranges, or stand aside |
| Market data stale | Treat as missing, do not trade that bStock |
| Binance API error | Retry with backoff, then stand aside and alert |
| Process crash | Supervisor restarts, agent rebuilds state from log and chain before trading |
| Guardrail breach | Trade rejected, logged with reason |

## Tech stack

Python for agent, tools, guardrails and execution. LLM tool-calling with structured outputs. Postgres or append-only JSONL for records. A supervised process on a hosted server. A BSC RPC endpoint. Third-party services are declared in `manifest.json`.
