# Case study: ClassMind

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [ClassMind's git log and docs](https://github.com/yassineaqejjaj/ClassMind). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

ClassMind is a French-language prototype that helps a teacher analyse assessment results. A teacher imports a CSV (pseudonymised pupils), maps questions to competencies, gets statistics computed in Python, and generates three exercise variants with an LLM. Every calculation in the generated exercises is checked with SymPy; a contradicted answer blocks validation and export. The teacher must validate the series before a PDF is produced. Stack: FastAPI, React/Vite, Keycloak, Podman/Caddy; the LLM is behind a `LLMProvider` interface with an OpenAI implementation. Data is fictional; scope is one 7th-grade-equivalent maths chapter (French "5e", relative numbers).

**Scale of the work:** 5 commits over 2026-10-22 to 2026-10-23. `f029509` is the MVP; `d2a4796` adds the README with screenshots; `fd309ad` adds a local demo; `fa1a9fb` adds CI (PR #1); `850cbab` adds a security policy. Early history is a single large commit, so stage-by-stage evidence is thin.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | Design principles and a scope-limits list in the README; no separate spec file | `README.md` |
| 2. Architecture | Architecture doc and a preliminary regulatory risk analysis with a residual-risk table | `docs/architecture.md`, `docs/analyse-risque-reglementaire.md` |
| 3. Agent build | MVP commit co-authored by the AI agent | `f029509` |
| 5. Verify | Stated: 35 backend tests (isolation, JWT, CSV import, analysis, SymPy checks, validation flow, cost cap, rejection), frontend CSV tests, lint and build. Counts are from the README and were not re-run. An evaluation-cases module exists in `ai/evaluation/test_cases.py`; no results are recorded | `README.md` (Tests) |
| 6. Release gate | CI gates on API tests, lint, frontend tests and build | `fa1a9fb` |

Autonomy level used: not recorded. `AUTHOR TO CONFIRM`.

## What the checks caught

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| The first CI attempt for the API job needed a different install step: the second bullet of the PR message says it must "install declared dependencies without unsupported editable package build". The workflow now reads dependencies from `pyproject.toml` with a comment that the API is a source tree, not an installable distribution | First CI run on a PR (the PR has two sub-commits, squashed) | Tests ran locally with `uv`; nothing had exercised a clean pip install on a runner | `fa1a9fb` |

This is the only defect the history shows. The commit message does not describe the original failure; the cause is read from the final workflow text.

## Process changes this led to

1. **LLM output is checked by something that is not the LLM.** Statistics are computed in Python; generated calculations are verified by SymPy; non-verifiable questions are labelled "not automatically verifiable" and left to human review.
2. **Human validation is a gate.** Validation and rejection apply to the whole series, are recorded, and export is only possible after validation.
3. **Dev shortcuts are scoped.** `fd309ad` adds a local demo (`scripts/local-demo.sh`) with a dev login limited to `vite dev` with `VITE_DEV_AUTH=1`, absent from production builds; the README says never to expose it on a network. The AI stays real in that demo (OpenAI if configured).

## What the process did not catch

Declared by the repo itself:

- Playwright tests are written but were not run in CI.
- No formal WCAG 2.2 AA audit.
- SymPy covers single-expression numeric calculations only.
- Validation and rejection apply to the whole series, not per variant.
- Question-to-competency mapping is manual; the CSV does not contain the question text.
- Scope is one chapter and one establishment per token.
- The PDF export test needs Pango and is skipped when it is absent; it is said to run in the container image.

## Safety limits and declared limitations

- Fictional data only. Pupils are pseudonyms (random identifiers); the mapping table stays with the teacher outside the tool. The README states pseudonymised data is still personal data and recommends a DPIA.
- No pupil data is sent to the LLM; only competency, difficulty, type, count, filtered teacher constraints (emails and phone numbers refused) and aggregated statistics. Calls use `store=False`; logs hold operation, tokens and cost, never content.
- Education AI can be high risk under the EU AI Act (Annex III). The design is a teacher aid: groups are "proposals", AI interpretations "hypotheses", no automatic decision, no causal attribution of progress. The risk analysis says it must be reviewed by a DPO before use with real data, and lists provider terms (DPA, transfers, retention) as still to verify.
- Per-establishment monthly LLM cost cap; retention purge command exists and is to be scheduled.
- `.github/SECURITY.md` (`850cbab`) repeats the synthetic-data rule.

## Review and rejection log

`AUTHOR TO CONFIRM`:

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion.
- Approximate review time of the MVP commit.
- Why the Playwright tests were not wired into CI.

## Takeaways for the framework

- Pair every generative step with a non-generative check where one exists (here SymPy), and label honestly what cannot be checked.
- A written regulatory risk analysis with a residual-risk table is a useful stage 2 artifact for education and health prototypes.
- Tests that exist but never run in CI should count as unverified in the status board.
