# Case study: MedGuard AI

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [MedGuard AI's git log and docs](https://github.com/yassineaqejjaj/medguard-ai). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

MedGuard AI is an open-source "AI assurance platform for healthcare". It registers AI applications under test, runs synthetic scenario campaigns against them, applies deterministic checks plus a local LLM judge (Qwen3-8B), routes doubtful or critical results to human reviewers, compares versions, applies a release gate (PASS, FAIL, REVIEW REQUIRED) with a CI endpoint, and produces reports with a hash-chained audit trail. Stack: FastAPI, PostgreSQL with row-level security, Celery/Valkey, Next.js, Keycloak (OIDC), Docker Compose. A fictional RAG assistant ships in two versions, the second with deliberate regressions. Everything runs on synthetic data and the README states it issues no medical or regulatory certification.

**Scale of the work:** 4 commits, all on 2026-10-23. One MVP commit (`d686857`, 02:43) contains the whole platform, followed by two CI fixes, a README update and a security policy. The history is too coarse to show how the work was staged; the study is therefore built mostly from the docs.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | Product scope and a "trust model" in the README; no separate spec file in the repo | `README.md` |
| 2. Architecture | Architecture doc with data model and decisions; security control table mapping each control to the test that covers it | `docs/architecture.md`, `docs/security.md` |
| 3. Agent build | One MVP commit co-authored by the AI agent | `d686857` |
| 5. Verify | Stated counts: 90 API tests passed and 1 skipped, 7 web tests, 5 Playwright journeys; `pip-audit` and `pnpm audit --prod` reported clean on 2026-10-22. These figures come from the README; they were not re-run for this study | `README.md` (Testing) |
| 6. Release gate | The product itself implements a gate, and the repo states limits plainly: a PASS "is not a guarantee of clinical safety" | `docs/evaluation-limits.md`, `docs/compliance-readiness.md` |

Autonomy level used: not recorded in the repository. `AUTHOR TO CONFIRM`.

Not present as artifacts: a status board, a dated decisions log, a test report file. Stages 4 (human review), 7 (deploy) and 8 (measure) leave no trace in the history.

## What the checks caught

The history shows one defect found by running the real pipeline. A second is recorded in a doc, without a commit.

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| The CI workflow used `pnpm/action-setup` without pointing at `apps/web/package.json`, in both jobs that install web dependencies. The fix adds `package_json_file` to both | Running the GitHub Actions workflow (inferred from the fix landing two minutes after the MVP commit) | The workflow was written in the MVP commit and could only be exercised on a runner. The root cause is inferred from the diff; the commit message does not describe it | `a353047` |
| A multicast address bypassed the SSRF guard on outbound calls | A unit test during development, per `docs/security.md` | Caught by the right test layer; no commit isolates the fix | in `d686857` |

The README's CI status moved from "GitLab CI written, not run" to "GitHub Actions: green" in the next commit (`049afff`), so the claim "green" followed the fix. Whether the run was green is not verifiable from the repository alone.

The pattern is thin: one infrastructure defect that only a real runner could expose.

## Process changes this led to

The history is too short to show process changes over time. What the repo does show is a design that encodes the framework's concerns:

1. **Evidence per control.** `docs/security.md` lists each control with the file and test that cover it, and lists gaps (no pen test, demo credentials, Keycloak in `start-dev`).
2. **A status table with "never exercised" entries.** The README says the real Qwen3-8B judge was never run in the test environment (tested against a stub server), the GPU path was never exercised, Podman was not used, and the GitLab CI file was never run on GitLab.
3. **The judge is measured, not trusted.** A deterministic failure cannot be overridden by the judge; critical failures need a human decision; judge-versus-human agreement (Cohen's kappa, false passes) is computed, and flagged indicative below 30 reviewed cases.

## What the process did not catch

All items below are declared by the repo itself, not discovered by this study:

- The real LLM judge was never run; judge behaviour is untested against the intended model.
- Consistency across rephrasings is declared as a criterion but not implemented.
- Evidence-archive restore was not rehearsed (database restore was).
- No penetration test; imported files get no malware scan; the audit chain cannot stop a database owner who recomputes hashes.
- The demo realm ships users with known passwords and Keycloak runs in dev mode in Compose.

## Safety limits and declared limitations

- Synthetic data only, stated in the README, `.github/SECURITY.md` (`1a1dbe4`) and `docs/evaluation-limits.md`. The security policy tells reporters not to submit real patient data.
- `docs/compliance-readiness.md` says the platform is not certified or presented as compliant with GDPR, HDS or the EU AI Act, and lists what the operator still has to do.
- The risk classification of a tested application (AI Act, MDR) is left to legal and clinical judgement, outside the tool.

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion.
- Approximate review time of the MVP commit. It is a single large commit, so how it was reviewed matters.
- Anything merged that you later regretted.

## Takeaways for the framework

- A single large agent commit leaves almost no review evidence. Prefer small commits per capability, so the history itself shows stages 3 to 5.
- Declaring "never exercised" items in a status table is a strong honesty practice and worth making a template field.
- A CI file written but never run is an untested artifact; the one defect here came from exactly that.
