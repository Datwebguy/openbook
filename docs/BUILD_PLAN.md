# Build plan

The challenge gives 2 weeks to build, then a 14-day scored run. Dates are set at onboarding. This plan is in relative days so it can start whenever onboarding ends. If onboarding info arrives later than expected, keep the order and compress.

## Team and roles

Fill in for the form and for the audit.

| Role | Person | Owns |
|---|---|---|
| Lead / agent | [name] | Agent loop, prompts, prosecutor, memory |
| Data and fair value | [name] | Data adapters, point-in-time layer, fair-value engine |
| Execution and risk | [name] | Execution, guardrails, reconciliation |
| Ops and audit | [name] | Logging, version registry, trace upload, monitoring |

A solo builder covers all four roles. Then follow the priority order at the bottom.

## Before day 1 (application phase)

- Submit the proposal form (deadline 30 Sep 2026).
- Set up the private repo and the challenge branch policy (no force-push).
- Prepare `manifest.json` skeleton.

## Week 1: foundations

| Day | Work | Done when |
|---|---|---|
| 1 | Repo layout, secrets handling, config, logging skeleton. Read the Participant Guide and Audit Log Specification when issued | Repo runs, no secrets in git, audit spec reconciled with `LOGGING_AND_AUDIT.md` |
| 2 | Binance Web3 Open API integration: quotes, depth, balances | `get_quotes`, `get_wallet` return real data |
| 3 | Execution on tiny test trades: order builder, submit, confirm, reconcile | One test trade round-trips and reconciles |
| 4 | Point-in-time data layer and share, futures and news adapters | Tools refuse future data (tests pass) |
| 5 | Fair-value engine v1 with session anchors, ratio, corporate actions | `get_fair_value` works for all sessions |
| 6 | Decision record, hash chain, version registry, daily trace upload | Records chain and verify, one trace uploaded |
| 7 | Cost estimator, guardrails v1 (caps, entry rules, loss rules) | All guardrail tests pass |

## Week 2: the agent

| Day | Work | Done when |
|---|---|---|
| 8 | Agent loop with tools and decision schema, first prompts | Agent produces valid decisions end to end on live data |
| 9 | Prosecutor call and answer loop | Objections stored, blocking objections cancel trades |
| 10 | Memory: journal, outcome writer, retrieval, calibration | `search_memory` works, calibration falls back correctly |
| 11 | Recovery, restart, monitoring, alerts, halt switch | Failure-injection tests pass |
| 12 | Start forward paper run (at least 4 US trading days plus a weekend) | Paper run live and logged |
| 13 | Tune from the paper run: thresholds, range widths, haircut. Daily reflection job | Parameters chosen from evidence, not guesses |
| 14 | Release checklist, audit dry run, register version 1, commit and freeze | Checklist all green, version event written |

If the rules require a longer paper run, start it earlier and overlap with days 8 to 11.

## Scored run: 14 days

Operate. Do not redesign.

- Daily checklist from `OPERATIONS.md`.
- Daily trace upload.
- Only fixes and safety changes, each as a new registered version.
- Weekly review of diagnostics: return, drawdown, fees, gas, costs versus estimates, heartbeat gaps.

## After the run

- Give the audit account what it needs.
- Prepare winner deliverables: reasoning output, prompts, tool definitions, model records.
- Write a short report: returns, drawdown, costs, how much the discount to fair value was reduced, lessons.

## Priority order if time is short

Keep these first. Cut from the bottom.

1. Safe execution, reconciliation, no double trades.
2. Guardrails.
3. Decision records, version registry, traces.
4. Fair-value engine.
5. Agent loop with tools.
6. Prosecutor.
7. Memory and calibration.
8. Daily reflection.
9. Polish.

Never cut items 1 to 3.

## Risks and answers

| Risk | Answer |
|---|---|
| Onboarding info arrives late | Build against interfaces. Keep adapters swappable |
| Thin liquidity means few trades | Log every WAIT with its reason. Widen the watchlist within eligible bStocks |
| LLM cost or rate limits | Cap calls per cycle, cache tool outputs within the cycle |
| Model changes behavior | Pin model identifiers in the manifest. Change only via new version |
| Audit gap | Reconcile hourly. Run the audit dry run before the scored run |
