# Logging and audit

Binance verifies the record. The audit checks that every scored trade maps to a registered agent version and a unique decision record, and that nothing was hidden or changed later. This file defines what we record.

The official Audit Log Specification is issued at onboarding. When it arrives, this file must be reconciled against it. Where they differ, the official spec wins.

## What the audit checks (from the challenge page)

- Every scored trade maps to a registered version and a unique decision record.
- No unmapped trades.
- No unexplained transfers.
- No version gaps.
- No manual intervention.
- No prohibited data.
- No material misrepresentation.

## Decision record

One write-once record per decision that leads to a trade, and also for decisions to WAIT or reject that the agent seriously considered.

```json
{
  "decision_id": "uuid",
  "agent_version": "v1.0.0+<git sha>",
  "decided_at": "iso time",
  "asset": "AAPLb",
  "action": "BUY",
  "reason_summary": "one plain sentence",
  "thesis": "text",
  "fair_price": { "low": 0.0, "mid": 0.0, "high": 0.0 },
  "market_price": 0.0,
  "raw_confidence": 0.0,
  "calibrated_confidence": 0.0,
  "evidence": [ { "source": "", "ref": "", "observed_at": "", "summary": "" } ],
  "tool_calls": [ { "tool": "", "input": {}, "output": {}, "called_at": "" } ],
  "prosecutor": { "verdict": "", "objections": [], "agent_answers": [] },
  "guardrails": { "passed": true, "checks": [], "final_size_pct": 0.0 },
  "order": { "client_order_id": "", "tx_hash": "", "filled_qty": 0.0, "avg_price": 0.0, "fees": 0.0, "gas": 0.0 },
  "model_calls": [ { "role": "agent|prosecutor|reflection", "model": "", "prompt_ref": "", "input_ref": "", "output_ref": "" } ],
  "record_hash": "sha256 of the record",
  "prev_record_hash": "sha256 of the previous record"
}
```

Records are chained with `prev_record_hash`, so any edit breaks the chain and is detectable.

## Agent version registry

A version is any unique combination of code, prompts, tool definitions, model identifiers and parameters.

To register a version:

1. Commit all code to the registered challenge branch. No force-pushes. No history rewrites.
2. Update `manifest.json` with the version, models, third-party services and data sources.
3. Write a **version event** to the log with the git SHA, manifest hash, time and reason.
4. Only then may the version trade.

Includes code running on remote services. Third-party services are declared in the manifest, not exposed as source.

## Daily traces

- Upload a trace bundle once a day to write-once storage, and once after any material incident.
- No end-of-event backfilling. If uploads fail, buffer locally and upload inside the published grace window.
- A trace bundle contains the decision records, version events, health and incident events and wallet snapshots for the period.
- Credentials never appear in logs.

## Full model records

Winners must deliver:

- the chain-of-thought output the agent produced for its decisions,
- full original prompts and tool definitions exactly as deployed,
- models plus complete inputs and outputs (for third-party API models, complete request and response records).

Capture all of this from day one. Do not rely on being able to reconstruct it later.

Reasoning output is stored in a separate field or file referenced from the decision record, and is not required in daily traces unless the official spec says so.

## Reconciliation

A job runs hourly and after every fill:

- compares wallet on-chain balances with the position tracker,
- maps every on-chain transaction to a decision record,
- raises an alert on any unmapped transaction or unexplained balance change.

## Retention

Keep everything for the whole challenge and audit period. Do not delete or rewrite.

## Never log

Keys, seed phrases, API secrets, personal data.
