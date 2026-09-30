# Agent behavior contract

## Role

The agent is the decision-maker. It is not a wrapper that turns signals into orders. It plans, investigates with tools, forms a view, and decides. Code around it limits what it can do.

## What the agent may do

- Choose which bStocks to examine each cycle.
- Call any tool in `TOOLS.md`.
- Override the fair-value engine's anchor when it has evidence, and it must state why.
- Propose one of: `BUY`, `HOLD`, `EXIT`, `WAIT`.
- Explain every decision in a short reason summary.

## What the agent may not do

- Trade outside eligible bStocks.
- Short, use leverage, derivatives, transfers, bridging or unrelated swaps.
- Move money out of the wallet.
- Change limits, prompts, tools or code during the run. Changes ship as a new registered version.
- Use information stamped after the decision time.
- Place an order any way other than `propose_trade`.

## Cycle inputs

`wallet`, `positions`, `session` (regular, pre, after, closed, weekend), `new_events`, `memory_hits`, `limits_used`.

## Decision schema

Every cycle produces one decision per bStock the agent acted on or seriously considered.

```json
{
  "decision_id": "uuid",
  "agent_version": "v1.0.0+<git sha>",
  "decided_at": "2026-10-20T14:02:11Z",
  "asset": "AAPLb",
  "action": "BUY | HOLD | EXIT | WAIT",
  "thesis": "one or two sentences",
  "fair_price": { "low": 0.0, "mid": 0.0, "high": 0.0 },
  "market_price": 0.0,
  "gap_after_costs_pct": 0.0,
  "confidence": 0.0,
  "expected_edge_pct": 0.0,
  "requested_size_pct_of_wallet": 0.0,
  "exit_conditions": {
    "take_profit": "price reaches fair mid",
    "thesis_break": ["list of events that invalidate the thesis"],
    "time_stop": "US open + 30 min"
  },
  "evidence": [
    { "source": "tool name", "ref": "id", "observed_at": "iso time", "summary": "text" }
  ],
  "override_of_engine": null,
  "reason_summary": "concise sentence for the audit"
}
```

Rules:

- `confidence` is between 0 and 1 and is calibrated by memory before sizing.
- Every `evidence` item must have `observed_at` earlier than `decided_at`.
- `BUY` needs a positive `gap_after_costs_pct`.
- `WAIT` is a valid and often correct decision. It is logged with its reason.

## Prosecutor exchange

After `BUY` or `EXIT` proposals, the prosecutor receives only the decision packet and returns:

```json
{
  "verdict": "PASS | CHALLENGE",
  "objections": [
    { "type": "stale_data | priced_in | thin_market | halted | contradiction | timestamp | other",
      "detail": "text", "severity": "minor | major | blocking" }
  ]
}
```

The agent must answer each objection or withdraw. A `blocking` objection that is not resolved cancels the trade. The verdict and the answers are stored in the decision record.

## Confidence and sizing

1. The agent states raw confidence.
2. `search_memory` returns similar past decisions and their outcomes.
3. Calibration maps raw confidence to a calibrated value (see `MEMORY.md`).
4. Guardrails compute size from calibrated confidence and edge-to-cost ratio, within caps (see `RISK.md`).

The agent proposes a size. Code sets the final size. Code may reduce it and never increase it.

## Reason summary

One sentence, plain language, no chain-of-thought. Example: "AAPLb trades 2.1% below a fair range raised by a confirmed earnings beat, and futures are flat, so the gap is not explained by the news."

The full reasoning output is captured separately in the decision record for the audit and winner deliverables.

## Versioning

Any change to prompts, tool definitions, model, parameters or code is a new agent version. It must be committed, added to the manifest and logged in a version event before it trades. See `LOGGING_AND_AUDIT.md`.
