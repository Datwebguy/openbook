# Tools

The agent reads the world and acts only through these tools. Every call is logged with its inputs, outputs and time.

Common rules:
- Every response includes `as_of` (the time the data is valid for). The data layer refuses to return anything stamped after the decision time.
- Every tool can return `status: "stale" | "missing" | "ok"`. The agent must handle all three.
- Tool definitions are part of the registered agent version.

---

## `get_quotes`

Eligible bStocks, prices, depth and swap quotes through the Binance Web3 Open APIs.

Input:
```json
{ "assets": ["AAPLb"], "size_quote": 500.0 }
```
Output:
```json
{
  "as_of": "iso time",
  "status": "ok",
  "quotes": [
    { "asset": "AAPLb", "mid": 0.0, "bid": 0.0, "ask": 0.0,
      "depth_within_1pct": 0.0,
      "swap_quote": { "in": 0.0, "out": 0.0, "price_impact_pct": 0.0, "expires_at": "iso time" } }
  ]
}
```

## `get_fair_value`

Fair-price range and confidence, with the inputs that produced it. See `FAIR_VALUE.md`.

Input: `{ "asset": "AAPLb" }`

Output:
```json
{
  "as_of": "iso time",
  "status": "ok",
  "session": "regular | pre | after | weeknight | weekend",
  "anchor": { "type": "live_share | extended_hours | futures_adjusted | last_close_news", "price": 0.0 },
  "fair_price": { "low": 0.0, "mid": 0.0, "high": 0.0 },
  "confidence": 0.0,
  "hours_since_live_price": 0.0,
  "adjustments": [ { "kind": "corporate_action | futures | news", "detail": "text", "effect_pct": 0.0 } ],
  "flags": ["halted", "stale_anchor", "holiday"]
}
```

## `get_news`

Company news, earnings releases and macro events, each with its publish time.

Input: `{ "asset": "AAPLb", "since": "iso time", "limit": 20 }`

Output:
```json
{
  "as_of": "iso time",
  "status": "ok",
  "items": [
    { "id": "string", "published_at": "iso time", "source": "string", "headline": "text",
      "summary": "text", "kind": "earnings | guidance | product | legal | macro | other",
      "is_repeat": false }
  ],
  "calendar": [ { "event": "AAPL earnings", "scheduled_at": "iso time" } ]
}
```

The tool does not return price reactions from after the item's publish time.

## `search_memory`

Past decisions in similar situations and how they ended. See `MEMORY.md`.

Input: `{ "asset": "AAPLb", "session": "after", "event_kind": "earnings", "k": 5 }`

Output:
```json
{
  "status": "ok",
  "hits": [
    { "decision_id": "uuid", "decided_at": "iso time", "action": "BUY",
      "raw_confidence": 0.7, "expected_edge_pct": 1.8,
      "outcomes": { "15m_pct": 0.4, "1h_pct": 1.1, "close_pct": 1.6 },
      "lesson": "text or null" }
  ],
  "calibration": { "bucket": "0.6-0.8", "n": 12, "hit_rate": 0.58 }
}
```

## `estimate_cost`

Round-trip fees, slippage and gas for a proposed size.

Input: `{ "asset": "AAPLb", "size_quote": 500.0 }`

Output:
```json
{
  "as_of": "iso time",
  "status": "ok",
  "swap_fee_pct": 0.0, "slippage_pct": 0.0, "gas_quote": 0.0,
  "round_trip_cost_pct": 0.0
}
```

## `get_wallet`

Balances, open positions, exposure and limits used.

Input: `{}`

Output:
```json
{
  "as_of": "iso time",
  "equity": 0.0, "opening_equity": 0.0, "peak_equity": 0.0,
  "drawdown_pct": 0.0, "day_pnl_pct": 0.0,
  "cash": 0.0,
  "positions": [ { "asset": "AAPLb", "qty": 0.0, "avg_cost": 0.0, "value": 0.0, "opened_at": "iso time", "decision_id": "uuid" } ],
  "limits_used": { "per_asset_max_pct": 25, "total_max_pct": 70, "total_used_pct": 0.0, "mode": "normal | half_size | risk_off | weekend" }
}
```

## `propose_trade`

The only path to execution. Sends a decision to the prosecutor and the guardrails.

Input: the decision JSON from `AGENT.md`.

Output:
```json
{
  "status": "executed | rejected | withdrawn",
  "prosecutor": { "verdict": "PASS | CHALLENGE", "objections": [] },
  "guardrails": { "passed": true, "failed_checks": [], "final_size_pct": 0.0 },
  "order": { "client_order_id": "uuid", "tx_hash": "0x...", "filled_qty": 0.0, "avg_price": 0.0, "fees": 0.0, "gas": 0.0 },
  "reason": "text"
}
```

`rejected` is final for that cycle. The agent cannot retry the same decision with a bigger size.

---

## Adding a tool

1. Write the spec here.
2. Implement it in `tools/` with the `as_of` and `status` rules.
3. Add a leakage test that tries to return future data.
4. Ship it as a new agent version.
