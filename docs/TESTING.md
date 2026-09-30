# Testing

Development-phase performance is not scored. Testing exists to protect the scored run: no rule breaks, no leakage, no bugs that double-trade, and no surprises in the audit.

The LLM is never tested on historical news, because it may already know what happened next. It is tested only on live, forward paper decisions.

## 1. Unit tests

| Area | Tests |
|---|---|
| Fair-value engine | Session anchors, ratio, corporate actions, futures adjustment, range widening with time |
| Cost estimator | Fees, slippage and gas add up to the round-trip cost |
| Sizing | Never exceeds caps, never above the agent's request, shrinks with lower confidence |
| Memory | Retrieval never returns data after time T, calibration fallback with small samples |
| Decision schema | Rejects missing fields, evidence with `observed_at` after `decided_at`, BUY without positive edge |

## 2. Guardrail tests

Each rule in `RISK.md` has a test that tries to break it.

- Propose a short, a leveraged position, a transfer, an unlisted token: all rejected.
- Propose a size above the per-asset cap, the total cap, the weekend cap: clipped or rejected.
- Simulate a 4% day loss, 10% and 15% drawdowns: entries stop, sizes halve, agent exits itself.
- Propose a BUY inside the earnings window: rejected.
- Propose a BUY with stale or halted data: rejected.
- Propose a BUY with an unresolved blocking prosecutor objection: rejected.
- Try to raise size after a rejection: rejected.

## 3. Leakage tests

- Every tool refuses to return anything stamped after the decision time.
- Feed a tool a future-stamped item: it must be dropped and logged.
- The prosecutor flags evidence stamped at or after `decided_at`.
- No historical news or price reaction is passed to the LLM anywhere in testing.

## 4. Execution tests

Run on tiny real transactions when the wallet and APIs allow it, otherwise on the test environment given at onboarding.

- Fresh quote check cancels when price moved.
- Unique order ID: a retried submit never double-trades.
- A stuck transaction is replaced with the same nonce.
- Reconciliation catches an injected mismatch.
- Restart in the middle of an order: state recovers with no duplicate trade.

## 5. Audit tests

- Every executed order has a decision record and version event.
- Record chain hashes verify end to end.
- A missing record triggers an alert.
- The trace bundle for a day contains every record for that day.
- A dry run of the audit: pick 20 random trades and reconcile them against wallet transactions by hand.
- Secrets scan: no keys or credentials in the repo, logs or prompts.

## 6. Failure injection

- LLM provider timeout: agent falls back or stands aside.
- Stale market data: no trade on that bStock.
- Binance API down: no trading until it returns.
- Server kill: supervisor restarts and recovers.
- RPC down: backup takes over.

## 7. Forward paper run (build week 2)

- Runs on live data with simulated or tiny fills, whichever the rules allow.
- Covers at least 4 US trading days and one weekend, including a night and an after-hours earnings event if one occurs.
- Measures:
  - fair-value error against the next live share price, by session,
  - hit rate by confidence bucket, to seed calibration,
  - cost estimate versus realized cost,
  - decision-record completeness,
  - number of prosecutor challenges and how they resolved.
- Outputs the numbers that set the confidence haircut, entry thresholds and range widths.
- Ends with a release checklist below.

## Release checklist for version 1

- [ ] All guardrail tests pass.
- [ ] All leakage tests pass.
- [ ] Reconciliation clean for the whole paper run.
- [ ] Audit dry run clean.
- [ ] Secrets scan clean.
- [ ] Manifest lists every model, service and data source.
- [ ] Version event written and code committed to the challenge branch.
- [ ] Alerts tested end to end.
- [ ] Operator has the runbook.
