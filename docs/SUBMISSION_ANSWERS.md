# Proposal form answers

Form: Beyond the Benchmark, The Proposal. Fill the [brackets] before you submit. Upload `Openbook-Architecture.pdf` for the architecture field. The form did not let one applicant replace an uploaded PDF, so upload the final one.

Applications close 30 Sep 2026.

---

## Your Proposal

bStocks trade around the clock, but their real shares trade only in US market hours. When that market is closed or on-chain liquidity is thin, bStock prices drift from fair value and sellers get less than the stock is worth. Openbook is an LLM-powered agent that reasons about each bStock's fair value from the live share price, extended-hours trading, index futures and breaking news. It plans what to investigate, uses tools, and forms a thesis with a fair-price range and a confidence. A separate prosecutor model then argues against every trade before it runs, and code guardrails cap the risk. The agent buys through the Binance Web3 Open APIs when a bStock is below fair value by more than trading costs, exits as prices converge or its thesis breaks, and learns from each outcome through memory. Every trade is tied to a registered agent version and a decision record.

## System architecture

Upload `Openbook-Architecture.pdf`.

## Data / tools used

Binance Web3 Open APIs (bStocks prices, depth, quotes, execution, wallet state). Public or licensed data: regular and extended-hours prices of the underlying shares, index futures, company news, earnings and macro calendars, and a BSC RPC for gas and confirmations. The agent uses these tools: `get_quotes`, `get_fair_value`, `get_news`, `search_memory`, `estimate_cost`, `get_wallet` and `propose_trade`. It is built in Python with LLM tool-calling, a decision-journal memory and an append-only log.

## Position & risk management

The agent proposes and code enforces. Eligible bStocks only, long or flat, with no shorting, leverage or transfers. Position size follows calibrated confidence and edge-to-cost ratio, within caps of 25% of the wallet per bStock and 70% total, or 30% over weekends. Loss limits: a 4% daily loss stops new entries, a 10% drawdown from peak halves sizes, and a 15% drawdown puts the agent into risk-off, where it exits positions itself. Exits also trigger when price reaches fair value or new evidence breaks the thesis. It skips trades on stale data, a halted share, thin liquidity or a failed prosecutor check. The operator switch only halts the agent and never trades.

## How would you prevent future leakage and overfitting?

A point-in-time layer serves only data stamped before each decision, and the prosecutor checks evidence timestamps. Only public or licensed data is used. The LLM is never tested on past news, because it may already know the outcome, so it is evaluated only on live forward paper decisions. Few parameters, each with an economic reason. The fair-value engine is checked with walk-forward tests, and configuration is frozen when each agent version is registered.

## Reproducibility plan

Every trade maps to one registered agent version and one unique write-once decision record. It holds the evidence with timestamps, the reasoning, the prosecutor verdict, guardrail checks, the order and the BSC transaction hash. All code is committed to the registered repo before it runs, and history is never rewritten. Daily traces go to write-once storage. Full prompts, tool definitions and every model input and output are captured, so any decision can be audited against wallet transactions.

## Team capability + intended scope

[Your name] builds tokenized-stock and AI agent products, including [project name] for the Solana STOCKLANA hackathon, which gives direct experience with tokenized-stock pricing, on-chain execution and autonomous agents. Scope: a two-week build, then an unattended 14-day scored run with daily traces, health monitoring and recovery. I will provide repo access, manifests and full model records for the audit, and I have the rights to every data source and model used.

---

## Consistency check

These numbers must match `RISK.md` and the PDF: 25% per bStock, 70% total, 30% weekend total, 4% daily loss, 10% drawdown halves sizes, 15% drawdown risk-off. If you change one, change it everywhere.
