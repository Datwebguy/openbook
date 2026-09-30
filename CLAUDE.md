# CLAUDE.md

Instructions for the coding agent working on the Openbook repo.

## What this project is

Openbook is an LLM-powered agent that trades eligible bStocks on BNB Smart Chain for the Binance Agentic AI Challenge. Read `README.md` first, then `docs/ARCHITECTURE.md`, `docs/RISK.md` and `docs/LOGGING_AND_AUDIT.md` before changing anything.

## Rules that override everything else

1. **No manual trading path.** Never add code, scripts, CLI commands or endpoints that place, cancel or close a trade outside the agent's `propose_trade` path. The halt switch only halts.
2. **Long or flat only.** Never add shorting, leverage, derivatives, transfers, bridging, unrelated swaps or arbitrary contract calls.
3. **Never use personal funds.** Do not write code that funds the wallet.
4. **No secrets in git.** Keys, seed phrases, API secrets and tokens live outside the repo. Run the secrets scan before every commit.
5. **No history rewrites.** Never force-push or rebase the challenge branch. Merge only.
6. **Commit before it runs.** All code that affects decisions, trades or logging must be committed to the challenge branch before it runs anywhere, including remote services.
7. **Every trade needs a decision record and a registered agent version.** If a change could produce a trade without one, stop.
8. **No future information.** Every data path goes through the point-in-time layer. Never pass historical news or post-decision price reactions to the LLM.
9. **Never log credentials.**

If a task seems to require breaking one of these, stop and ask.

## Working style

- Small, focused commits with clear messages. One concern per commit.
- Write or update tests with every change. Guardrail, leakage and audit tests are mandatory for their areas (see `docs/TESTING.md`).
- Prefer boring, explicit code over clever code. Money is involved.
- Match the existing style, names and comment density of the code around you.
- Use types and validate every external input. Reject and log, never guess.
- Every external call needs a timeout, bounded retries and an error path.
- Money and quantities use decimals, not floats, in execution and accounting.
- All timestamps are UTC ISO 8601 with a timezone.

## Where things live

| Area | Path | Spec |
|---|---|---|
| Agent loop, prosecutor | `agent/` | `docs/AGENT.md`, `docs/PROMPTS.md` |
| Tools | `tools/` | `docs/TOOLS.md` |
| Fair value | `fairvalue/` | `docs/FAIR_VALUE.md` |
| Guardrails | `guardrails/` | `docs/RISK.md` |
| Memory | `memory/` | `docs/MEMORY.md` |
| Execution | `execution/` | `docs/RISK.md`, `docs/OPERATIONS.md` |
| Audit | `audit/` | `docs/LOGGING_AND_AUDIT.md` |
| Ops | `ops/` | `docs/OPERATIONS.md` |

## Changing behavior

Any change to prompts, tool definitions, model identifiers, parameters or code that affects decisions is a **new agent version**:

1. Bump the version.
2. Update `manifest.json`.
3. Add a version event.
4. Run the release checklist in `docs/TESTING.md`.

During the scored run, ship only fixes and safety changes.

## Before finishing a task

- Tests pass, including guardrail, leakage and audit tests for anything you touched.
- No new secrets, no real keys in fixtures.
- Docs updated if behavior or interfaces changed.
- If the change affects limits, update `docs/RISK.md`, `docs/SUBMISSION_ANSWERS.md` and the architecture PDF numbers together.

## Open facts, not assumptions

These are unknown until onboarding. Do not hardcode guesses. Make them config with clear names and fail loudly when unset:

- eligible bStocks,
- wallet opening asset and quote asset,
- bStock trading hours,
- valuation method,
- rate limits and fee rules,
- audit log format and upload endpoints.
