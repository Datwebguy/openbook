# Openbook

An LLM-powered agent that works out what every bStock should be worth at every moment, including when the US market is closed, and trades only when the market price is wrong by more than the cost of trading. Every trade is traceable and audit-ready.

Built for **Beyond the Benchmark: The Binance Agentic AI Challenge** (DoraHacks).

## The problem

bStocks trade on BNB Smart Chain around the clock, but the real shares behind them trade only in US market hours. When the US market is closed, or on-chain liquidity is thin, nobody reliably prices bStocks using all the information available. Prices drift from fair value and sellers get less than the stock is worth.

Openbook reasons about each bStock's fair value, buys when the market price is below it by more than trading costs, and exits when price converges or the thesis breaks.

## What makes it agentic

- The LLM **decides**: it plans what to investigate, calls tools, weighs conflicting evidence, forms a thesis with a fair-price range and confidence, and chooses to trade, wait or exit.
- A separate **prosecutor** call argues against every trade before it runs.
- **Memory** of past decisions and outcomes calibrates confidence and size.
- **Code guardrails** enforce hard limits the model cannot override.

## Status of key facts

| Item | Status |
|---|---|
| Applications | Close 30 Sep 2026 |
| Scored run | 14 days, dates set at onboarding |
| Eligible instruments, quote asset, valuation method, audit log spec | **Issued at onboarding. Not known yet.** See `docs/RULES_CHECKLIST.md` |
| Wallet | Dedicated competition wallet, same opening value for every team. No personal funds |

## Documents

| File | What it covers |
|---|---|
| `docs/ARCHITECTURE.md` | Components, agent loop, data flow |
| `docs/AGENT.md` | Agent behavior contract and decision schema |
| `docs/PROMPTS.md` | Draft prompts: agent, prosecutor, reflection |
| `docs/TOOLS.md` | Tool specs with input and output schemas |
| `docs/FAIR_VALUE.md` | Fair-value engine specification |
| `docs/RISK.md` | Guardrails, limits and loss rules |
| `docs/MEMORY.md` | Decision journal, retrieval, calibration |
| `docs/LOGGING_AND_AUDIT.md` | Decision records, versions, traces, deliverables |
| `docs/OPERATIONS.md` | Runbook, monitoring, recovery |
| `docs/RULES_CHECKLIST.md` | Challenge rules mapped to the design, plus onboarding questions |
| `docs/TESTING.md` | Paper run, leakage tests, guardrail tests |
| `docs/BUILD_PLAN.md` | Two-week build plan with acceptance criteria |
| `docs/SUBMISSION_ANSWERS.md` | Final proposal form answers |
| `CLAUDE.md` | Instructions for the coding agent building this repo |

## Proposed repo layout

```
openbook/
  agent/          loop, planner, decision schema, prosecutor
  tools/          one module per tool in TOOLS.md
  fairvalue/      engine, session logic, corporate actions
  guardrails/     limits, sizing, kill and halt logic
  memory/         journal store, retrieval, calibration
  execution/      quotes, order builder, submit, reconcile
  data/           adapters and point-in-time store
  audit/          decision records, version registry, trace upload
  ops/            supervisor, heartbeat, alerts
  tests/
  docs/           these files
  manifest.json   declared third-party services and models
```

## Hard rules (never break)

1. No manual trading. Every trade comes from the registered agent.
2. Long or flat only. No shorting, leverage, derivatives, transfers, bridging or unrelated swaps.
3. Never use personal funds.
4. Every trade maps to one registered agent version and one unique decision record.
5. Commit code before it runs. Never rewrite history.
6. Never commit secrets or keys.
7. Use only public or legally authorized data. No future information.
