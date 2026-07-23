# Codex start instruction

Open or create the local repository `global-ai-mt5-trader` and use the attached `PROJECT_SPEC.md` as the authoritative requirement.

Your current task is **Phase 1 only: deterministic scanner**.

Before editing:

1. Inspect the current repository, environment and existing files.
2. Summarize what already exists.
3. Write a short Phase 1 implementation plan.
4. Identify the exact files you will create or modify.

Then implement Phase 1 end to end.

Required Phase 1 scope:

- Python package and configuration structure;
- strict provider-independent schemas;
- historical/mock data adapters only;
- deterministic indicators and feature engineering;
- 0–100 opportunity scoring with configurable weights;
- ranking by asset class and final candidate limit;
- correlation and duplicated-exposure filter;
- deterministic market-regime classifier;
- approved strategy interface and registry stubs, including `NO_TRADE`;
- tests, fixtures, README and local scripts.

Do not implement:

- live APIs;
- news providers;
- LLM/API calls;
- Binance WebSockets or order flow;
- MT5 or MQL5;
- order execution;
- real-money trading;
- complex UI.

Engineering requirements:

- Use Python type hints and Pydantic validation.
- Keep calculations deterministic.
- Reject stale, incomplete, duplicated, future-dated and invalid data.
- Avoid look-ahead bias and data leakage.
- Do not hide exceptions or skip failing tests.
- Do not invent credentials or paid-provider assumptions.
- Prefer a small maintainable dependency set.

Completion requirements:

1. Run formatting, type checks and the complete test suite.
2. Fix all failures caused by your work.
3. Show the commands and final results.
4. List every created or changed file.
5. State known limitations honestly.
6. Create one focused Git commit if the repository is initialized and Git identity is available; otherwise provide the exact commit command and explain why it was not run.
7. Stop after Phase 1. Do not begin Phase 2 without a new instruction.
