# Case study: ORBIT

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [ORBIT's git log and docs](https://github.com/yassineaqejjaj/ORBIT). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

ORBIT is a governed context and memory platform for AI agents in a product team. It ingests sources (documents, tickets, meetings, connectors and MCP servers) with personal-data detection and classification, builds a memory with a lifecycle (proposed, validated, replaced, obsolete, forgotten), and assembles context for agents with access control, citations and reason codes. It also ships an MCP server, change feeds and webhooks, an evaluation bench and a Next.js frontend. It is the context layer of the ORION trio with NOVA and FORGE.

**Scale of the work:** 160 commits (130 non-merge, 30 merges) between 2026-09-23 and 2026-10-23, from two committers. 126 commit messages carry a `Co-Authored-By` trailer naming an AI model (96 Opus 5.5, 30 Sonnet 5.5). Docs include an architecture contract (`docs/ARCHITECTURE.md`), an AI and context engineering roadmap with items labelled A to F (`docs/AI_CONTEXT_ENGINEERING.md`, `67691e4`), a contracts document and a status matrix (`docs/STATUS.md`).

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | An "AI and context engineering" roadmap committed before the work, with items A1 to F3 referenced in later commit subjects | `67691e4`, then `bde525c` to `ff3c6fb` |
| 2. Architecture | An architecture contract written in the first commit, and contract files for parallel module work | `09c7959`, `94331d9`, `docs/ARCHITECTURE.md` |
| 3. Agent build | A parallel module run, then feature commits per roadmap item | `6da40d0`, `5420f78`, `57faf00` |
| 5. Verify | A backend test suite of about 70 test files; the first integration commit records "234 passing"; a mypy baseline test; a CI workflow added late | `0b9b7be`, `backend/tests/`, `187a262` |
| 6. Release gate | A status matrix with implemented, tested, `main`, `preprod` and deployed columns | `docs/STATUS.md` |
| 8. Measure | One measured latency budget for the reranker; no stored evaluation report | `115f069` |

Autonomy level used: **not recorded in the repository**. `AUTHOR TO CONFIRM`.

## What the checks caught

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| The parallel module run was cut off twice by an organisation spend limit, leaving code "not yet tested or integrated; several files may be incomplete" | The agent run itself stopped; the author committed the state as WIP with the caveat in the message | Not a defect check: the history shows the work was committed unverified and labelled so | `6da40d0`, `5420f78` |
| Cross-module bugs after integration: query results handled as dicts, `text[]` vs `varchar[]` ACL filters, superseded memory not reported as excluded, audit events leaking restricted documents | First end-to-end integration; integration tests "written but not yet run (Docker was down)" in the previous commit | Integration tests had not been executed until then | `0b9b7be` (after `57faf00`) |
| Two concurrent memory-extraction jobs produced 13 duplicated requirements in the demo | Seen in the Atlas demo data (commit body) | Concurrency was not covered by tests; fixed with a per-project Postgres advisory lock | `a5cea6a` |
| Postgres, OpenSearch and Valkey ran unauthenticated and were published on all interfaces, exposing fictitious C2/C3 demo data to the local network | Not stated in the commit (the body describes the exposure) | Defaults were convenient, not safe | `2715dcf` |
| A context request could fail when learned ranking weights were unavailable | Not stated in the commit | Not stated in the commit | `42aebf8` |
| Reranker defaults (30 passages of 1200 characters) were too slow for the context latency target | Measurement on demo data: p50 1.1 s versus 370 ms p50 and 385 ms p95 at 20 passages of 800 characters (commit body, one machine) | No budget existed until measured | `115f069` |
| The LLM guardrail was not fail-closed for unknown classification levels and remote embeddings | Not stated in the commit; a 137-line test file was added | Not stated | `4cfa47d` |
| "Forget" did not rewrite persisted contexts and snapshots at rest | Not stated in the commit; a 194-line test was added | Forgetting covered live memory only | `0310c1d` |
| `expand_source` did not accept a document id; type ignores were stale | Static typing (mypy: 101 to 31 errors, versioned baseline, per `docs/STATUS.md`) | Types were not checked in CI before `187a262` | `6fa6628` |
| Text tokens of the design system had no contrast check | Added a script that fails below 4.5:1; commit body states all pairs pass | No accessibility check existed | `5c844ef` |
| A duplicate-chunk test depended on which chunk the engine kept; a webhook test left queued deliveries | Test runs | Test design | `1cb528b`, `82100d0` |
| Next.js build required Google Fonts at build time | Offline build (NOV-89 in `docs/STATUS.md`) | Build never ran without network | `61431bc` |

The pattern: the largest correction was integration. The history is unusually honest about it, with two commits that say "unverified" before the commit that makes the suite pass.

## Process changes this led to

1. **Label unverified work.** The two WIP commits state that nothing is integrated or tested, so the history shows the verification debt instead of hiding it.
2. **CI added after the features** (`187a262`): lint, mypy baseline, `alembic check`, pytest, web build, Docker builds and audits. Before that, the only gate visible in the history is local test runs.
3. **A status matrix** with a branch column for the integration branch and a rule that "tested" lists only test files that exist (`docs/STATUS.md`).
4. **Contract fixtures with NOVA and FORGE** (`4e9a6d3`, NOV-74) and drift tests.
5. **Fail-closed defaults** for the guardrail (`4cfa47d`), and localhost-only ports (`2715dcf`).

## What the process did not catch

- `docs/STATUS.md` marks deployment as "to confirm" for almost every row; the repo cannot show what runs in production.
- The status matrix lists items not done: OIDC SSO and MFA, chained audit, retention periods, backups, a Helm chart, continuous deployment, CRM connectors, an externalised authorisation engine and a continuous evaluation loop.
- Object storage was tested against a simulated S3 (`moto`), "no real MinIO" (`docs/STATUS.md`).
- The reranker latency was measured on one machine over 104 chunks of demo data.
- `docs/PORTFOLIO_CASE_STUDY.md` lists quality, safety, resource and latency measures as **Not measured**.

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion.
- Approximate review time per pull request into `preprod`.
- The autonomy level used for the parallel module run, and what you reviewed afterwards.
- Anything merged that you later regretted.

## Takeaways for the framework

- **A parallel agent run needs a verification step before integration.** Here two interrupted runs left unverified code that was only validated at `0b9b7be`. Keep unverified work labelled, as this history does.
- **Concurrency defects appear when you run the product**, not in unit tests (`a5cea6a`).
- **Safe defaults for local demos matter when the demo data models real classes** (`2715dcf`).
- **Add CI before the feature count grows.** CI arrived on 2026-10-15, after about 120 commits.
- **Measure before setting a latency budget** (`115f069`), and store the measurement.
