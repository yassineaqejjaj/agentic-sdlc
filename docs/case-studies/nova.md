# Case study: NOVA

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [NOVA's git log and docs](https://github.com/yassineaqejjaj/NOVA). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

NOVA is the personal AI product agent of the ORION ecosystem (with ORBIT for governed context and FORGE for evaluation). It turns an intent into a plan, runs versioned Skills through specialist agents, and produces structured, editable Artifacts. An engineering path (SDLC Autopilot) goes from spec to pull request, review, CI and merge. The repository holds a Python backend, a Next.js web app, a Skills catalogue and a contract-test suite for the links to ORBIT and FORGE.

**Scale of the work:** 234 commits (140 non-merge, 94 merges) between 2026-10-03 and 2026-10-23, from two committers. 135 commit messages carry a `Co-Authored-By` trailer naming an AI model (104 Sonnet 5.5, 31 Opus 5.5), so an agent took part in most of the history. The repo does not contain a written spec with numbered acceptance criteria comparable to TrustRail's; the closest artifacts are `docs/STATUS.md`, `docs/ENGINEERING.md`, `docs/ARCHITECTURE.md` and one ADR (`docs/adr/001-identite-federee.md`, added in `95269a1`).

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | Ticket references (NOV-68 to NOV-114) in commit subjects; no standalone spec file found | commit log |
| 2. Architecture | An architecture document, a data inventory, an isolation document and one ADR on federated identity | `docs/ARCHITECTURE.md`, `docs/ISOLATION.md`, `docs/adr/001-identite-federee.md` (`95269a1`) |
| 3. Agent build | Feature work in small commits, often grouped by ticket | `git log`, Co-Authored-By trailers |
| 4. Human review | Work went through pull requests into a `preprod` integration branch; the status matrix refers to PR numbers | `docs/STATUS.md` |
| 5. Verify | Unit, integration, contract and cross-instance tests; a CI workflow; a CI check that refuses a Skill change without a version bump | `tests/`, `.github/workflows/ci.yml`, `9508893` |
| 6. Release gate | A status matrix with four separate columns: implemented, tested, on `main`, deployed. A freeze rule is written down | `docs/STATUS.md` |
| 8. Measure | Token and latency figures in two commit bodies; an SDLC run evaluation (success rate, CI repairs, cost per run) | `b25a575`, `7222f12`, `279d1b2` |

Autonomy level used: **not recorded in the repository**. `AUTHOR TO CONFIRM`.

## What the checks caught

Many defects were fixed with the real failure described in the commit body. Where the commit does not say how the defect was found, the table says so instead of guessing.

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| A truncated forced tool call came back as `{}`, the repair round failed the same way, and retries regenerated everything (about 10 minutes per step) | Real runs on a small model (commit body names the 4096-token budget) | Not stated in the commit | `4f1c7f9` |
| Skill sync raced when API workers started together (unique violation crashed startup) | Not stated in the commit | Not stated in the commit | `9508893` |
| Anthropic rejected `temperature` for some models and refused a forced `tool_choice` | Real provider calls; the commits describe the fallbacks only | Not stated in the commit | `91be4e6`, `8d2a008` |
| The CI repair loop read only check annotations (often runner deprecation warnings), so it "fixed blind" | Real failed GitHub Actions runs | The repair step had not been fed real job logs | `396007b` |
| ORBIT changed `timings.rounds` to a list; NOVA expected numbers, so every ORBIT retrieval failed with a generic error | Integration with the real ORBIT | Contract was not yet pinned by fixtures; ORBIT/FORGE/NOVA fixtures and traversal tests were added later (`ee47cba`, NOV-74) | `d3891b6` |
| Observed capture sent the redacted request text to FORGE, contradicting the docs and risking C2/C3 material leaving NOVA | Review of behaviour against the documented contract (commit body) | Code and docs disagreed | `1ef3057` |
| Context handed over by FORGE fell back to classification C1; unclassified context is now read as C3 | Not stated in the commit | Not stated in the commit | `4d58de2` |
| SDLC auto-merge accepted "no CI" as green | Not stated in the commit; the fix adds integration tests | Not stated in the commit | `1bd5ed7` |
| ORBIT clearance was not enforced on derived Artifacts, lists, replay and stored extracts | Ticket NOV-75 in `docs/STATUS.md`; the ORBIT repo's status page mentions an ORION audit dated 13 October, which may be the origin (not confirmed) | Not stated in the commit | `3fb21e7` |
| Model calls via the user's own keys and voice were not guarded by classification | Not stated in the commit | Guard covered the default path only | `48888ca` |
| Training never promoted a release: each cycle changed one Skill but FORGE compared all of the agent's Skills, so its statistical test stayed inconclusive | **Production data of 20 October** (commit body) | Not stated | `7afbb11` |
| Every training cycle lost 1 to 3 baseline runs to an inference server error; retries were 1 s and 2 s | **Production data of 20 October** (commit body) | Not stated | `e086449` |
| CI flakes: link-health reset depended on machine uptime; a Playwright mock raced on `route.fetch` | CI | Test design | `447e310`, `e4fd6f1` |
| Generated dev build files, including Next.js development preview keys, were tracked in git | Ticket NOV-92 in `docs/STATUS.md`; how it was noticed is not stated | Files had been committed earlier; history was not rewritten | `81a60ad` |
| A wrong ORBIT password returned 401, which the web client treated as an expired session and redirected to login | Not stated in the commit | Not stated in the commit | `1ed5806` |
| A 300-frame prototype lost everything after its header: icons and chart strokes consumed a 120-node budget | Real Figma read (commit body) | Not stated | `3117cd2` |

Two commits record measured gains, taken from runs with a real small model: PRD output tokens 97.8k to 27.7k (702 s to 206 s) in `b25a575`, and PRD input tokens 130k to 61k in `7222f12`. The commit bodies say "measured on real runs"; the repo has no stored report, so treat the numbers as author-reported.

The pattern: **the defects with the most information in their commit bodies were found against real providers, a real ORBIT/FORGE or production data.** The status matrix itself shows the same stance: "CI green establishes only the checks in that workflow" (`docs/PORTFOLIO_CASE_STUDY.md`).

## Process changes this led to

1. **Four-column status matrix.** `docs/STATUS.md` separates implemented, tested, on `main` and deployed, names the test files, and says a row is never "deployed" only because it is on `main`.
2. **Immutable Skill versions enforced by CI.** `9508893` adds a hash lock; CI fails when a released Skill version's content changes.
3. **Contract fixtures between the three products.** `ee47cba` and `e3aba5e` (NOV-74) add shared fixtures and traversal tests after the `timings.rounds` break.
4. **README facts are tested.** `0fa309a` and `b3c30e8` add tests that check README claims (links, headings, character limit) against the repository.
5. **Classification (C0 to C3) became a cross-cutting guard** after several separate fixes (`4d58de2`, `3fb21e7`, `48888ca`, `ba3d82e`). Each guard has its own configuration variable in `docs/STATUS.md`.
6. **A policy gate for learned changes** (`33578bb`): hold a learned policy without out-of-production proof; default mode is `warn`, not `enforce`.

## What the process did not catch

- Several issues surfaced only in production data on 20 October (`7afbb11`, `e086449`), after the training loop had shipped.
- Dev key material (Next.js preview keys) stayed tracked until `81a60ad`, and `docs/STATUS.md` says it remains in git history because history was not rewritten.
- The status matrix lists items not yet done: an LLM classification ceiling PR still open at the time of writing, a FORGE-side calibrated promotion gate outside NOVA's scope, and an SDLC agent with no proof source in `enforce` mode.
- The status file says production deployment was not re-checked from the repository.
- `docs/PORTFOLIO_CASE_STUDY.md` lists quality, safety, resource and latency measures as **Not measured**. No evaluation report is stored in the repo.

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion.
- Approximate review time per pull request into `preprod`.
- Which commits were agent-authored versus hand-written, and the autonomy level per stage.
- Anything merged that you later regretted (the tracked dev keys are one candidate).

## Takeaways for the framework

- **Keep a status matrix that separates "written", "tested", "merged" and "deployed".** NOVA's matrix is the clearest instance in the three repos.
- **Pin cross-product contracts with shared fixtures before integrating**, not after the first break (`d3891b6`).
- **Production data is a verify stage.** The two 20 October fixes show a loop that was green in tests and inconclusive in real use.
- **A safety property such as data classification needs a single enforcement point.** It took four or more separate fixes to cover derived content, own keys, voice and capture.
- Store measurements when you quote them. Token figures live only in commit bodies here.
