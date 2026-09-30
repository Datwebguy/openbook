# Memory

Memory lets the agent learn during the run without changing its code. It stores what it decided, why, and what happened.

## What is stored

For every decision (including WAIT with a real consideration):

| Field | Meaning |
|---|---|
| `decision_id`, `decided_at`, `asset`, `action` | Identity |
| `session`, `event_kind` | Situation tags used for retrieval |
| `thesis`, `raw_confidence`, `expected_edge_pct` | What the agent believed |
| `calibrated_confidence` | After calibration |
| `outcomes` | Realized results at 15 minutes, 1 hour, next session close, and at exit |
| `lesson` | Optional one-line lesson from reflection |

Outcomes are written only after their horizon has passed, so memory never contains future data relative to a later decision.

## Retrieval

`search_memory` returns the most similar past decisions by:

1. same event kind,
2. same session type,
3. same asset, then similar assets,
4. similar raw confidence and edge.

At the start of the run memory is empty. The paper run in the build phase seeds it with forward decisions. Seeded entries are marked `phase: paper` and kept separate in reports.

## Calibration

Confidence is bucketed, for example 0.0 to 0.2, 0.2 to 0.4, up to 1.0. For each bucket the system tracks how often the expected edge was realized.

- Calibrated confidence = observed hit rate for that bucket, smoothed toward the raw value when the sample is small.
- With fewer than N samples in a bucket (start with N = 10), use the raw value reduced by a fixed haircut.
- Calibration is computed by code, not by the LLM.

Sizing uses the calibrated value. Overconfidence therefore shrinks size automatically.

## Daily reflection

Once a day the reflection prompt reviews recent decisions and outcomes and writes up to five lessons as data.

- Lessons are stored in memory and retrieved with similar situations.
- Lessons cannot change prompts, tools, limits or code.
- If the reflection suggests a change, it goes on a review list for the operator. Any accepted change ships as a new registered agent version.

## Storage

One table or JSONL file for decisions, one for outcomes, one for lessons. Append-only. Corrections are new rows that reference the old one.

## Leakage rules

- Retrieval only returns decisions with `decided_at` and all used outcomes earlier than the current decision time.
- Lessons are stamped with the time they were written, and only retrieved after that time.
- No memory is preloaded from historical market data or historical news.

## Tests

- A query at time T never returns an outcome recorded after T.
- Calibration with tiny samples falls back to the haircut rule.
- Lessons cannot modify configuration.
