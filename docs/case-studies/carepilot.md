# Case study: CarePilot (M3aya Santé)

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [CarePilot's git log and docs](https://github.com/yassineaqejjaj/CarePilot). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

CarePilot (renamed M3aya Santé in the README) is a healthcare administrative-coordination MVP for clinics: patient journeys, a staff worklist, a bilingual (French and Arabic) patient portal, human-approved communications, insurance and finance tracking, and nine reference clinic pathways. The stack is a FastAPI API, a React web app, Keycloak sign-in, PostgreSQL with pgvector, and a controlled AI layer (retrieval over approved procedures only, agents that propose and people that decide). All data is fictional.

**Scale of the work:** 27 commits between 2026-10-22 and 2026-10-29. Six commits carry a `Co-Authored-By` trailer for an AI agent (`7366648`, `72d8954`, `5124c42`, `9c16d38`, `2e0936e`); most later commits carry no trailer, so the history does not show which of them were agent-written. There is no written product spec and no decisions log; the closest artifacts are the README, `docs/architecture.md`, `docs/security.md` and `docs/compliance-morocco.md`.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | A vision statement, nine pathway definitions and a Moroccan compliance review, with no numbered acceptance criteria | `docs/vision.fr.md`, `docs/pathways.md`, `docs/compliance-morocco.md` |
| 2. Architecture | An architecture note, an authentication note, a security note and an integrations review | `docs/architecture.md`, `docs/authentication.md`, `docs/security.md` |
| 3. Agent build | Feature-sized commits (for example `2e0936e`, `0d86586`, `44f3ff3`), with the README updated alongside | git log |
| 5. Verify | Pytest on real pgvector, Playwright browser scenarios, and deployed-environment checks; counts recorded in the README | README "Tests", `tests/`, `apps/web/e2e/`, `scripts/test-deployed.mjs` |
| 6. Release gate | CI for API tests and the frontend build; an explicit list of work remaining before real patients; a statement that no legal compliance is claimed | `e065c54`, README "Remaining work before use with real patients" |
| 7. Deploy | Vercel frontend and Railway API; a reproducible packaging script for the static output | `442ae19`, `33a6b9c`, `scripts/package-web-output.py` |
| 8. Measure | Not present. The README says no user-study results or productivity gains are claimed | README "Clinic usability validation to conduct" |

**How the fictional-data limit was handled** (as the repo shows it):

- A security policy requires synthetic data in demos, tests, screenshots and logs and asks reporters not to submit real records (`cbe2334`, `.github/SECURITY.md`).
- The README banner says the public deployment is a demonstration with simulated SMS, WhatsApp and email delivery, no OpenAI key, and no real patient data to be entered.
- The messaging transport defaults to a console provider; the Brevo adapter is tested only with mocked HTTP calls, and the docs say no live delivery has been verified (`44f3ff3`, `docs/brevo.md`).
- Destructive demo reset is gated behind `DEMO_MODE`, and the browser tests that mutate data are documented as not to be run against a clinic environment.
- Microphone testing uses a generated fictional recording, and the README states this verifies capture and review, not Darija accuracy (`8251b50`).
- AI calls are blocked until a transfer reference is entered, and the README says entering one does not make a transfer lawful.

## What the checks caught

History here is thin: commit bodies are mostly empty, and the repo has no test report or defect log. The rows below are limited to fixes whose diff makes the defect visible. "Found by" is given only when the repo says so; otherwise it is marked as not recorded.

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| The staff-activity metric counted any actor not on a short exclusion list, so patient rights requests and support requests from patient identities could count as staff adoption. It was replaced with an explicit staff-contributor rule and a test. | Not recorded (commit title says "verify staff activity metrics") | Not recorded. The earlier logic was a deny-list and the new test covers cases it did not. | `33a6b9c` |
| The communications screen labelled a provider-accepted message "Envoyé" (sent) with a success tone. A separate "Livré" (delivered) state and an "accepted by provider" label were added, and a heading that implied a patient assistant was reworded. | Not recorded | The two states were conflated in the UI type itself. | `ae0e8d7` |
| A deploy built from the monorepo root can publish no frontend files and overwrite a working deployment. The Vercel root directory, build and output settings were pinned in `vercel.json` and documented. | Not recorded; the README states the failure mode | Not recorded | `442ae19` |
| The README says a deployment marked Ready is not sufficient validation, and the packager now refuses to build when index asset references are missing from the output. | Not recorded | A "Ready" status does not check that the entry assets exist | `33a6b9c` |
| A browser regression test selected `aside` and had to be narrowed to the first match after the navigation and sidebar work. | Running the suite against production, per the commit title | Not recorded; the diff shows only the selector change | `65db194` |

The pattern the repo supports: validation was repeated against the live deployment as well as locally (the README records local pytest counts of 37, 48, 69 and 70 across successive commits, and online scenario counts of 6, 13 and 11). The history does not show which, if any, of those online runs failed first.

## Process changes this led to

1. **Validation sections in the README grew with each release.** Each feature commit records what was run, where (local or deployed) and with which data. Examples: `8251b50`, `65db194`.
2. **Limits are written next to the claim.** The README separates "Verified online" from "not verified" (live messaging, Darija accuracy, malware scanning, scheduled worker) instead of leaving them implicit.
3. **A reproducible release path.** After the Vercel monorepo issue, the static output got an explicit packager and a post-deploy check list (`/`, `/app`, `/operations`, one asset, one authenticated API call) in `442ae19` and `33a6b9c`.
4. **Usability is stated as unmeasured.** The README defines a three-task clinic test and says no results exist.

## What the process did not catch

- Real-patient readiness is explicitly out of scope. The README lists CNDP approvals, malware scanning, backup and restore, robust patient authentication, persistent Keycloak storage, scheduled jobs and incremental migrations as open.
- The initial Alembic revision creates tables from model metadata, so older databases need a manual upgrade path (README "Migration limitation").
- Live messaging delivery and WhatsApp templates were never verified; AI quality was never evaluated on synthetic cases.
- The repo has no recorded rehearsal, no defect log and no decisions log, so most defects found during development left no trace.
- CI (`e065c54`) runs API tests and the frontend build only. Browser tests are not in CI.

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Autonomy level used per stage. The repo does not state it.
- Agent proposals you rejected or rewrote, and why.
- Architecture and compliance choices you made against the agent's suggestion (for example the Moroccan regulatory framing, fictional-data-only deployment).
- Which commits without a trailer were agent-written.
- Approximate review time per release.
- Anything merged that you later regretted.

## Takeaways for the framework

- **A "Ready" deployment is not a release gate.** The repo encodes this as a post-deploy checklist; the framework can make it a stage 6 exit criterion.
- **Name the data limit in the repo, not only in the head.** Synthetic-data policy, a demo-mode flag for destructive resets, a console messaging provider and a mocked-provider test together made the no-real-patient limit enforceable and reviewable.
- **Keep a defect and decisions log even for small projects.** Here the fixes are visible but their discovery is not, which limits what can be learned from the history.
- **Separate states that look alike.** "Accepted by provider" versus "delivered" is a verification claim; the UI should not assert more than the system knows.
