# Case study: FORGE

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [FORGE's git log and docs](https://github.com/yassineaqejjaj/FORGE). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

FORGE is an evaluation lab for AI agents. It answers whether a new agent version is better than another, by scenario, criterion, cost and confidence. It has an agent runner with several adapters, an evaluation engine (rules, LLM judges, multi-judge aggregation, calibration), benchmarks, paired experiments with regression detection, a CLI for CI gating, ingestion of runs executed elsewhere (NOVA), a red-team catalogue, a model arena, retention and forget handling, and a Next.js frontend. It evaluates NOVA's agents but is generic.

**Scale of the work:** 87 commits (61 non-merge, 26 merges) between 2026-09-30 and 2026-10-23, from two committers. 56 commit messages carry a `Co-Authored-By` trailer naming an AI model (31 Sonnet 5.5, 25 Opus 5.5). This is the thinnest history of the three, and many commits have no body, so the table below is short.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | A README stating the question the product answers; no separate spec file | `README.md` |
| 2. Architecture | An architecture contract in the first commit, plus architecture, contracts, agent protocol and workspace documents | `0b0505a`, `docs/ARCHITECTURE.md`, `docs/AGENT_PROTOCOL.md` |
| 3. Agent build | A first interrupted run, then integration of backend modules | `f1240bb`, `a17a491` |
| 5. Verify | About 40 backend test files; documented JSON examples replayed as tests; a CI workflow from 2026-10-02; contract fixtures with ORBIT and NOVA | `519035d`, `docs/OBSERVED_RUNS.md`, `ea8526a` |
| 6. Release gate | A promotion policy that requires calibrated judges, no contamination and replayed regressions; a CI mode that fails on regression | `0182d7b`, `docs/CI.md` |
| 8. Measure | Observability gauges, deployed SHA exposure; no stored evaluation report | `8398b9d` |

Autonomy level used: **not recorded in the repository**. `AUTHOR TO CONFIRM`.

## What the checks caught

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| The first module run was cut off by a quota limit and committed as WIP "unverified" | The run stopped | Not a defect check; the history labels the state | `f1240bb` |
| A race when sibling runs finish concurrently: parent execution or experiment finalisation could be missed | Not stated (same commit as the frontend feature pages) | Not stated; the commit adds a row lock, a worker sweeper and a regression test | `ed9094d` |
| Foundation bugs: `ON CONFLICT` partial index predicate, error details, timeout default | Integration of backend modules | Not stated | `a17a491` |
| Concurrent API replicas could race on Alembic upgrades | Not stated (same commit as the operations guide) | Not stated; fixed with a Postgres advisory lock | `244f7df` |
| The container entrypoint shifted arguments even with no command (POSIX sh) | Not stated | Not stated | `72e8fc6` |
| FORGE read ORBIT's context with the wrong field shapes (excerpt, snapshot content, request id, forgotten) | Not stated; the subject says it now reads the real ORBIT schemas | Contract was assumed rather than pinned; fixtures followed (`ea8526a`, NOV-74) | `3d101e0` |
| Classification levels were lost between ORBIT, FORGE and NOVA, and were "laundered" on promotion | Not stated in the commit; a 51-line classification unit test was added | Not stated | `2cc475f` |
| Outgoing LLM calls (judges, feedback, adapters) were not guarded by classification | Not stated; a 46-line guardrail test was added | Guard existed elsewhere, not on these paths | `4fa851e` |
| The judge alert was wrong; deployed version, queue and pending-capture gauges were missing | Not stated | Not stated | `8398b9d` |
| Fixture bytes differed between SQLite and PostgreSQL runs because JSONB reorders keys | Running the contract tests on both databases (commit body) | Fixtures had not been written with sorted keys | `5e36c3a` |
| Next.js build needed Google Fonts at build time | Not stated | Not stated | `4120574` |
| Layout overflow in run detail and grids | Visual review | Not covered by tests | `3dbb17d`, `1937077`, `244f7df` |

The pattern, where visible: integration with the real counterpart (ORBIT schemas, concurrent workers, two database engines) found what FORGE's own tests did not. Most commit bodies are empty, so "found by" cannot be established for many fixes.

## Process changes this led to

1. **Executable documentation.** The observed-runs contract embeds JSON examples that `test_observed_runs.py` replays, so a doc that drifts from code fails tests (`docs/OBSERVED_RUNS.md`, `505c6ef`).
2. **A promotion policy** (`0182d7b`, `FORGE_PROMOTION_POLICY`): a candidate cannot be promoted without calibrated judges, no contamination and replayed regressions. This puts FORGE's own gate rule on its own release decisions.
3. **A CI gate for agents** (`docs/CI.md`): a baseline against candidate experiment with `--fail-on-regression`, with exit codes for gate failure versus error.
4. **Contract fixtures shared with ORBIT and NOVA** (`ea8526a`, `5e36c3a`, `f46cb0c`), resynced when ORBIT added a field.
5. **Deterministic red-team detectors, no LLM judge** (`86078de`): seven attack families as ordinary scenarios.

## What the process did not catch

- `docs/PORTFOLIO_CASE_STUDY.md` states the Nordalis benchmark is simulated and seeded, so a synthetic pass is not evidence a real agent is ready. Its measures are all **Not measured**.
- There is no stored report of an evaluation run against a real agent in the repo, and no FORGE-side release status matrix like NOVA's and ORBIT's.
- Whether the deployed instance matches the repository cannot be read from the repo.

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion (for example judge calibration thresholds).
- Approximate review time per pull request.
- The autonomy level used, and how the interrupted first run was recovered.
- Anything merged that you later regretted.

## Takeaways for the framework

- **An evaluator should apply its own rule to itself.** FORGE's promotion policy and CI gate are the framework's "autonomy is granted by evidence" idea in product form.
- **Executable docs** (examples replayed as tests) turn a contract into a check.
- **Write commit bodies for fixes.** Here many fixes have no body, which limits what a reconstruction can say about how defects were found. Add "found by" to the fix convention.
- **Do not report synthetic benchmark results as evidence.** The repo says so itself; keep that line.
