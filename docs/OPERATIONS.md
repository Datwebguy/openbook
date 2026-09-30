# Operations runbook

The agent runs unattended. No manual trading, ever. Operator actions are limited to monitoring, halting, restarting and shipping new registered versions.

## Hosting

- One hosted server with process supervision. Consider a warm standby only if it cannot double-trade (single-writer lock on the wallet).
- Keys and API credentials stored outside the repo, loaded from a secret store or environment.
- Outbound access to: Binance Web3 Open APIs, BSC RPC, share and futures data, news, the LLM provider.

## Health monitoring

| Signal | Alert when |
|---|---|
| Heartbeat | Missed for 2 minutes |
| Cycle time | Over 3x normal |
| Binance API errors | More than 3 in a row |
| LLM errors or timeouts | More than 3 in a row |
| Data freshness | Any source beyond its limit |
| Reconciliation | Any mismatch or unmapped transaction |
| Loss rules | Daily loss, drawdown or risk-off triggered |
| Trace upload | A daily upload fails |

Alerts go to the operator's phone.

## Recovery

| Situation | What happens |
|---|---|
| Process crash | Supervisor restarts. Agent rebuilds state from the decision log and on-chain balances, reconciles, then resumes. |
| Server lost | Restore from the write-once log and repo. Confirm wallet balances before starting. |
| LLM provider down | Agent uses no-news mode with wider ranges, or stands aside. Alert the operator. |
| Binance API down | Retry with backoff. No trading until quotes and execution both work. |
| RPC down | Switch to a backup RPC. Confirm nonce and pending transactions before submitting. |
| Stuck transaction | Replace it with the same nonce and higher gas. Never resend as a new order. |
| Unknown transaction on the wallet | Halt, investigate, log an incident. |

Recovery logic is part of the agent's code. It is not a manual trade.

## Halt switch

- `ops halt` stops the agent from starting new cycles and from placing orders.
- It does not close or open positions. That would be manual trading.
- After a halt, positions stay open until the agent exits them itself after restart.
- Use it only for security or integrity problems, and log the reason as an incident.

## Restart

1. Verify the repo is on the registered version.
2. Rebuild state from the log and the chain.
3. Run reconciliation. Resolve mismatches before trading.
4. Start the agent.
5. Log a restart event.

## Shipping a new version during the run

Only for fixes or safety issues.

1. Fix on a branch, test, merge to the challenge branch. No force-push.
2. Update the manifest.
3. Write a version event.
4. Deploy. The old version stops before the new one trades.
5. Confirm the first decision under the new version maps correctly.

Avoid strategy changes during the scored run.

## Incident handling

An incident is any of: outage over 5 minutes, unmapped transaction, guardrail failure, data breach risk, a version deployed incorrectly.

1. Halt if safety or integrity is at risk.
2. Write an incident record with time, cause and effect.
3. Upload an extra trace bundle.
4. Fix, register a version if code changed, resume.

## Daily checklist

- Heartbeat green, no open alerts.
- Daily trace uploaded.
- Reconciliation clean.
- Drawdown and exposure inside limits.
- Reflection report read. Review-list items triaged.
- Wallet balances match the log.

## Cost watch

Data, model and hosting costs are on us. Track model spend and API rate limits. Do not cut logging to save cost.
