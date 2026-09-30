# Risk and guardrails

The agent proposes. Code enforces. The agent cannot change or bypass any rule here.

The final score is percent return on the wallet after fees and gas, with lower maximum drawdown as tie-break. These limits protect the wallet without forcing the agent to sit idle. All numbers are starting points. Align them with the official parameters issued at onboarding.

## Hard restrictions

- Eligible bStocks only, as set at onboarding.
- Long or flat only.
- No shorting, leverage, derivatives, transfers, withdrawals, bridging, unrelated swaps or arbitrary contract calls.
- No personal funds.
- No manual trading, ever.

## Caps

| Cap | Value |
|---|---|
| Per bStock | 25% of wallet equity |
| Total invested, market-weighted | 70% of wallet equity |
| Total invested over weekends | 30% |
| Open positions | No fixed count. Total and per-asset caps apply |

Market-weighted means correlated bStocks count together. Three large tech bStocks are treated closer to one position.

## Sizing

Size is a function of calibrated confidence and edge-to-cost ratio, inside the caps.

1. Compute `edge_to_cost = expected_edge_pct / round_trip_cost_pct`.
2. Reject if `edge_to_cost` is below the entry threshold.
3. Scale size up with calibrated confidence and edge-to-cost.
4. Clip to per-asset and total caps and to liquidity limits.

Code may reduce the agent's requested size and never increases it.

## Entry conditions

All must be true:

- Gap below the fair-price range beats round-trip cost plus the confidence margin.
- Data is fresh and no blocking flag is set.
- Prosecutor did not leave a blocking objection.
- Slippage and price impact are inside tolerance.
- No new entry inside the scheduled earnings window.

## Loss rules

| Trigger | Response |
|---|---|
| Down 4% on the day | No new entries until the next day |
| Down 10% from peak equity | All sizes halved |
| Down 15% from peak equity | Agent moves to risk-off and exits positions itself through its normal execution path |

Risk-off is an automatic agent behavior. It is not a manual action. After risk-off the agent stays flat until the next day, then resumes at half size.

## Exit rules

- Price reaches fair-price mid.
- New evidence breaks the thesis.
- Time stop once the US market opens and prices the news.
- Fair value moves down so the position is no longer below it.

Every exit is an agent decision with its own decision record.

## Liquidity and execution limits

- Skip or split an order if expected slippage exceeds tolerance.
- Skip or split an order that is large compared with visible depth.
- Fresh quote required before signing. Cancel if price moved beyond tolerance.
- Costs are part of the decision: fees, slippage and gas reduce the score.

## Halt switch

The operator halt switch stops the agent from acting. It never places or closes a trade. Positions stay open until the agent itself exits them after restart. Use it only for security or integrity problems.

## Security

- The wallet holds only competition funds.
- Keys stay on the server and never appear in logs, prompts or the repo.
- Only approved token contracts and router addresses are allowed. Anything else is rejected.
- API credentials are stored outside the repo.

## Guardrail tests

Each rule needs an automated test that tries to break it. See `TESTING.md`.
