# Case study: TrustRail

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [TrustRail's git log and docs](https://github.com/yassineaqejjaj/TrustRail). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

TrustRail controls what AI agents may do on behalf of a person or company, and records verifiable receipts. V1 is a monorepo: an enforcement gateway API, a policy engine, signed (Ed25519) receipts, a human approval queue, a Next.js dashboard, TypeScript and Python SDKs, an MCP adapter and two synthetic sandboxes. It covers two use cases: customer-support refunds and B2B SaaS account management. No real money moves.

**Scale of the work:** 33 commits between 2026-09-21 and 2026-09-22, built with an AI coding agent under a written spec (`docs/PRODUCT_SPEC.md`) with 15 mandatory capabilities (V01 to V15) and six phase gates.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | A 549-line product spec with mandatory capabilities and numbered phase gates | `docs/PRODUCT_SPEC.md` |
| 2. Architecture | A threat model, an architecture document and a dated, append-only decisions log | `docs/THREAT_MODEL.md`, `docs/DECISIONS.md` |
| 3. Agent build | Phase-by-phase implementation, with a status board updated after every phase | `docs/BUILD_STATUS.md` |
| 5. Verify | Unit, integration and concurrency tests on a real Postgres, a Playwright suite, cross-language receipt checks | `docs/TEST_REPORT.md` |
| 6. Release gate | Phase gates marked *met* only when the specified test had actually run | `docs/BUILD_STATUS.md` |
| 8. Measure | Real latency aggregates (p50, p95, p99) in the dashboard | V15 |

Autonomy level used: **L2 (Delegate)** for implementation, **L0** for the architecture and threat model. `AUTHOR TO CONFIRM`.

## What the checks caught

Green unit tests were not enough. Each of these defects passed the earlier checks and was found only when something was executed for real.

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| A required file was never copied into the production build output, so any `dist/` start would have failed | First real `docker compose up` | No test suite ran the compiled output | `cf16a50` |
| Wrong Postgres volume path, missing base config in each image, and a key-file permission mismatch between services | First real `docker compose up` | The Docker files had been written but never run, which the decisions log had flagged in advance | `cf16a50` |
| CI failed on a version conflict between two pnpm settings | First real GitHub Actions run | The workflow had never run on a real runner | `80fb109` |
| Session cookies silently dropped once the web app and API sat on different domains | Real deployment | Local development shared one origin | `d62c99d` |
| The signing-key bootstrap trusted a database row even when the key file was gone from a replaced volume | Real deployment | Tests never replaced the volume | `d62c99d` |
| A button component dropped a React ref, so approve and reject dialogs failed about half the time | Rehearsing the demo script in a browser | Tests passed; the failure was intermittent and interactive | `40a7f99` |
| The dashboard showed "€NaN" and "€0.00" for actions that have no amount, and a dialog always said "refund" | Walking through the second use case live | The refund shape had leaked into UI that now served five action types | `c4aa191` |
| Authentication and row-level security deadlocked on agent credentials | Gateway integration tests | Caught by the right test layer, before any manual run | build log |

The pattern: **the cheapest defects were caught by tests, the expensive ones by running the real thing.** That is the case for a stage between "tests pass" and "release".

## Process changes this led to

1. **"Executed evidence" became part of Verify.** A capability counts as done only if a test *that was actually run* covers it. `BUILD_STATUS.md` defines `done` this way and keeps `partial` and `todo` honest.
2. **Predict, then check.** The decisions log recorded in advance that the never-run Docker setup would surface real bugs, and it did. Writing the expected failure down before running turned the run into a test of the plan.
3. **Rehearsal is a gate.** The demo script was timed and walked through live against a freshly seeded stack, which found three UI defects that no automated test covered.
4. **Limitations are written down.** The status file lists known simplifications, such as header-based network isolation without Docker (later verified in Compose) and ambiguous budget semantics for mixed grants, instead of leaving them implicit.

## What the process did not catch

- The spec's last item, a recorded demo with a human presenter, was never an engineering task and remained open.
- The second use case was covered by integration tests and live runs but not by Playwright at the time of the status entry.
- Several simplifications remained by design, including a worker that scales linearly with the number of organizations and polling-based event streaming.

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion.
- Approximate review time per phase.
- Anything merged that you later regretted.

## Takeaways for the framework

- Add an explicit **"run it for real"** exit criterion to stage 5 for anything that ships: build output, containers, CI, deployment and an interactive walkthrough.
- Keep a **status board with a strict definition of done**. It makes unverified work visible.
- **Tests and real execution catch different classes of defects.** Neither replaces the other.
