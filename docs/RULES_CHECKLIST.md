# Challenge rules checklist

Source: the official DoraHacks page for **Beyond the Benchmark: The Binance Agentic AI Challenge**, read on 30 Sep 2026. Final rules, scoring details and the Participant Guide are issued to finalists at onboarding. Where they differ from this file, they win.

## Timeline

| Phase | What |
|---|---|
| Applications | 9 to 30 Sep 2026, proposal through the Google Form |
| Selection | Fewer than 25 finalist teams |
| Build | 2 weeks |
| Scored run | 14 days, mechanical scoring |
| Audit | Top teams are provisional winners until the audit passes |
| Total | About 7 weeks including onboarding and audit |

Exact dates come at onboarding.

## Team and eligibility

- 1 to 4 people. You may join only 1 team.
- Individuals only. Companies and other legal entities are not eligible.
- Full civil capacity under local law. Identity verification at onboarding. Teams that cannot complete it forfeit their place.
- You hold the rights to every data source, model, API and third-party component.

## How proposals are judged

| Criterion | Our answer |
|---|---|
| Feasibility | Two-week plan in `BUILD_PLAN.md`, tools already scoped |
| Agentic depth | LLM plans, uses tools, keeps memory, verifies with a prosecutor. Not a thin wrapper |
| Risk awareness | Guardrails in `RISK.md`, failure handling in `OPERATIONS.md` |
| Originality | Prosecutor verification, outcome-calibrated memory, session-aware fair-value reasoning |
| Ability to deliver | Roles and milestones in `BUILD_PLAN.md` |

Not considered: proposal length, design polish, brand names, budget.

## Scoring

`FinalScore % = (Final Wallet Equity - Opening Wallet Equity) / Opening Wallet Equity x 100`

- Same opening value for every team.
- Fees and gas reduce the wallet, so they reduce the score.
- Tie-break: lower maximum drawdown.
- The full 14 days count. No leaderboard during the run.
- Not scored: demo polish, spend, call volume, number of trades, development-phase results.

## Rules we must follow

| Rule | Where we handle it |
|---|---|
| Autonomous operation, no manual trading | `OPERATIONS.md`, halt switch never trades |
| Every trade maps to one agent version and one decision record | `LOGGING_AND_AUDIT.md` |
| New versions committed, manifested and logged before trading | `LOGGING_AND_AUDIT.md` |
| All decision-affecting code in the registered repo before it runs, including remote services | `README.md` hard rules |
| No force-pushes or history rewrites on the challenge branch | `LOGGING_AND_AUDIT.md` |
| No secrets in the repo | `RISK.md` security |
| Daily trace uploads to write-once storage, no backfilling | `LOGGING_AND_AUDIT.md` |
| Credentials never in logs | `LOGGING_AND_AUDIT.md` |
| No Binance-internal data, no non-public information, no future information | `FAIR_VALUE.md`, `TESTING.md` |
| Dedicated wallets. No own funds, withdrawals, transfers, bridging, unrelated swaps, arbitrary contract calls | `RISK.md` |
| No leverage, shorting, derivatives | `RISK.md` |
| Repo is private. A Binance audit account gets read-only access | `README.md` |

## Consequences of breaking rules

Manual trading, unmapped trades, fabricated or incomplete decision records, prohibited data, false declarations, credential misuse and material audit gaps lead to ineligibility or disqualification.

## Winner deliverables

- Chain-of-thought output for decisions.
- Full original prompts and tool definitions as deployed.
- Models plus complete inputs and outputs.

All captured from day one. See `LOGGING_AND_AUDIT.md`.

## Questions to answer at onboarding

Ask in the DoraHacks Q&A, Discord or the onboarding session. Do not assume answers.

1. Which bStocks are eligible?
2. What is the wallet's opening asset and value, and what is the quote asset for swaps?
3. What is the valuation source and the treatment of transfers?
4. What is the exact scored window and the schedule for the build phase?
5. What does the Audit Log Specification require, and what are the upload endpoints and grace windows?
6. What is the incident policy?
7. Are raw chain-of-thought outputs required in daily traces or only for winners?
8. What rate limits and fee rules apply to the Web3 Open APIs?
9. What trading hours apply to bStocks?
10. Is a protected or private submit path available on BSC?
11. Does the build phase have a testnet or a sandbox wallet?
12. Are hosted LLM APIs allowed, and how are third-party model records handled?

## Practical notes from the Q&A tab

- The proposal form did not let one applicant replace an uploaded PDF. Upload the final PDF the first time.
