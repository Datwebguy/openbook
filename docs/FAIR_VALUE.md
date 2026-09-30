# Fair-value engine

The engine is a tool the agent calls. It gives the agent a fair-price range and a confidence. The agent may override the anchor with evidence and must say why.

## Outputs

For each bStock: `fair_price {low, mid, high}`, `confidence`, `session`, `anchor`, `hours_since_live_price`, `adjustments`, `flags`.

## Anchor by US session

Times are US Eastern.

| Session | Anchor | Base confidence |
|---|---|---|
| Regular, 9:30 to 16:00 | Live share price | High |
| Pre-market and after-hours | Extended-hours price, weighted by volume | Medium |
| Weeknights | Last close adjusted by index-futures move times stock beta | Lower |
| Weekends | Last close adjusted by news | Lowest |

Beta is estimated from past data only and refreshed on a schedule. It is never fit on data after the decision time.

## Adjustments applied in order

1. **Token-to-share ratio.** Convert the share price to the bStock's unit.
2. **Corporate actions.** Dividends, splits and other changes that affect the token.
3. **Session anchor.** As in the table above.
4. **Futures adjustment.** Only when the share is not trading live.
5. **News adjustment.** From the agent's structured reading of events (see below).

## News adjustment

The agent reads news and returns a structured event for each relevant item: stock, direction (bullish or bearish), magnitude class (minor, major, critical), and whether it is new or a repeat.

The engine converts the event to a fair-price shift using the stock's own past reactions to the same event kind, scaled by its volatility. The LLM does not choose the price move.

Rules:
- A repeated item causes no shift.
- Conflicting items widen the range instead of shifting the mid.
- A single unconfirmed source gives a small shift. A second independent source lifts it to the full shift.

## Confidence range

The range starts narrow and widens with:

- each hour since the last live share price, scaled by volatility,
- news uncertainty,
- thin or stale inputs,
- weekends and holidays.

Entry margin required on top of round-trip cost grows as confidence falls.

## Flags and stop conditions

The engine returns flags and the agent must respect them:

| Flag | Meaning | Effect |
|---|---|---|
| `halted` | Underlying share halted or trading irregular | No trades on that bStock |
| `stale_anchor` | Anchor older than its limit | No trades on that bStock |
| `holiday` | US market holiday | Use weekend-style range |
| `earnings_soon` | Scheduled release within the window | No new entries before it, reaction traded after |
| `ratio_unknown` | Token-to-share ratio not confirmed | No trades on that bStock |

## Mispricing versus information

Before calling a gap mispricing, the engine reports whether related markets moved too:

- index futures move since last close,
- other tokenized versions of the same stock, if available.

If they moved, the gap is likely information. The agent should follow it, not fade it.

## Validation

- Walk-forward tests on past data for the price model only.
- No use of the LLM on historical news in any test, because it may already know the outcomes.
- Forward paper run checks fair-value error against realized prices at the next live session.
- Track the error by session type. Widen ranges where the error is larger than the range implies.

## Open items before build

- Confirm the bStock unit and token-to-share ratio from Binance.
- Confirm which share and futures data sources are legally usable and their delays.
- Confirm how dividends are handled on the token.
