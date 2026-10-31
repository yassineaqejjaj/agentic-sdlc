# Case study: Fencing Intelligence

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [Fencing Intelligence's git log and docs](https://github.com/yassineaqejjaj/fencing-intelligence). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

**Evidence caveat.** The public history has 11 commits between 2026-10-09 and 2026-10-23, and the first one (`d4e4532`) publishes the whole tree at once (about 200 files). The build itself is therefore not visible commit by commit. What is visible is the repository's own status documents (dated 8 October 2026) and the nine commits after publication. This study is short for that reason.

## What was built

A self-hosted fencing video analysis platform for épée, foil and sabre. Coaches and athletes upload video, annotate actions, record touches and read statistics computed only from human-validated records. The repo contains a Next.js web app, a FastAPI API, a video worker (FFmpeg), an experimental ML inference service, a shared domain package (statistics formulas, competition logic) and Docker Compose files for local and production-like stacks. A federation pilot slice (competitions, poules, standings, direct-elimination draws, referee assignment) is also documented. The README labels it **experimental beta; detection heuristics are not validated refereeing**.

**Scale of the work:** 11 commits, 2026-10-09 to 2026-10-23. The commit trailers on the post-publication commits name an AI coding tool as co-author (Cursor); no trailer is visible for the squashed first commit.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | A README with explicit non-claims, and a roadmap with a "not a substitute for a referee" section | `README.md`, `docs/ROADMAP.md` |
| 2. Architecture | A decisions log (license, queue, statistics, coaching, computer vision, minors and retention) and a written formula spec for statistics, versioned ("Version: 1") | `docs/DECISIONS.md`, `docs/STATISTICS.md` |
| 3. Agent build | An agent guide with layout and "rules that matter" (validated-only statistics, drafts stay drafts, 404 across organizations, no training without consent) | `AGENTS.md` |
| 5. Verify | API tests (journey, federation, guardian consent, rate limits, heuristics), domain tests, vitest, a Playwright journey, a CI workflow | `services/api/tests`, `packages/fencing-domain/tests`, `.github/workflows/ci.yml` |
| 6. Release gate | A status file that lists "Verified commands" separately from "Not verified here" | `docs/IMPLEMENTATION_STATUS.md` |
| 8. Measure | Prometheus alert rules (ApiDown, ApiErrors, ApiSlow, UploadFailures). No usage or quality metrics in the repo | `infrastructure/monitoring/alerts.yml` per the status file |

Autonomy level used: not recorded in the repo. `AUTHOR TO CONFIRM`.

## What the checks caught

The status file records an explicit "production readiness pass" in which a production-mode stack was booted and the full flow exercised through the web proxy. The defects below are quoted from that pass and from the commit messages after publication. The repo does not say which earlier checks ran before them, so the "why missed" column states only what the sources support.

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| Missing `.dockerignore`: a 1.2 GB build context that included `.env` | Booting `docker-compose.prod.yml` as a staging stack | Not stated. Local runs used no production image build of this kind | squashed into `d4e4532` (status doc) |
| Success notices on the competition page never rendered (cleared synchronously by a refresh) | Same staging pass | Not stated | `d4e4532` (status doc) |
| Login page advertised local demo credentials on production builds | Same staging pass | Not stated. Fixed by gating on `NODE_ENV !== "production"` | `d4e4532` (status doc) |
| Object storage URLs failed on Railway: buckets sign against the virtual host, local MinIO uses path style | First deployment on Railway (per message) | Local stack uses path-style addressing only | `792627f` |
| The production seed for the demo club failed before the coach account could be saved (organization model not imported) | Running the seed on production (per message) | Not stated. The seed had just been added in `8a16f7b` | `9e73fb0` |
| A published feature (athlete linked to a public FIE profile) was reverted the next step, and a migration revision was kept because it had already run in production | Production start, per the message ("so production can still start") | Not stated | `0635845`, `989175e` |

Pattern, limited to what the repo shows: the three staging defects were found by booting the production compose file, and the two deployment defects (`792627f`, `9e73fb0`) by running on the real platform. The status file also records environment-level surprises (ports 8000 and 3000 taken, compose port mappings merging across override files) that were worked around, not fixed.

## Process changes this led to

1. **Verified versus unverified is separated in writing.** `IMPLEMENTATION_STATUS.md` says a feature is listed as tested "only when a command in this session finished successfully" and has a "Not verified here" section (no GitHub-hosted CI run yet, no real-video evaluation, no load test, one advisory in a build-time dependency).
2. **Empty states are a rule, not a nicety.** Rates with a zero denominator are null with reason `no_samples`, charts are not drawn when `data_status` is not `ok`, and this appears in `AGENTS.md`, `DECISIONS.md` and `STATISTICS.md`.
3. **Heuristics are labelled, and their evaluation is declined rather than faked.** See the next section.
4. **A backup drill was executed**, not only scripted: the status file describes a pg_dump, a witness row, a restore that removed it, and one deleted bucket key re-mirrored (1 file, 2072 bytes).

### How heuristics and estimates were validated or labelled

- Movement and scoreboard detectors are called experimental heuristics in the README, `AGENTS.md` and `DECISIONS.md`. Model suggestions stay `draft` until a person validates them, and scoreboard light candidates do not change a bout score.
- Pose estimation uses a third-party landmarker that stores joint positions and "does not name fencing actions".
- A logistic-regression baseline was fit on separated synthetic clusters. The status file says its holdout score "is a pipeline check, not fencing accuracy".
- "No model in this repository has been evaluated on real fencing video. No recognition accuracy is claimed."
- Coaching recommendations require at least five validated samples and quote their source metric; no language model is called.
- Distance stays in pixels until a piste calibration exists.

The repo shows labelling and refusal to claim accuracy. It contains no validation of the heuristics against real footage.

## What the process did not catch

All of this is stated by the repo itself:

- GitHub Actions had not run on a hosted runner at the time of the 8 October status; the roadmap still lists "A GitHub-hosted run of the CI workflow" as next. The README now shows a CI badge, but this study did not verify a green run.
- No load test; concurrent upload and transcode queue depth are unmeasured.
- TLS termination, SMTP against a real relay and a retention schedule are open items in `docs/ROADMAP.md`. Soft-deleted video purge exists as a command, not a schedule.
- A written minor-data policy is listed as needed "before any U11 to U17 pilot".
- The reverted FIE profile feature (`b448aeb`, `0635845`) shows a change reaching the main branch and then being undone; the migration could not be undone (`989175e`).

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Agent proposals you rejected or rewrote, and why. In particular, why the FIE profile link (`b448aeb`) was reverted.
- Architecture choices you made against the agent's suggestion (for example the decisions on license, object storage image and no language model in coaching).
- The autonomy level used per stage.
- Approximate review time per phase, and whether the first commit `d4e4532` was reviewed as a single publication step.
- Anything merged that you later regretted.

## Takeaways for the framework

- **Keep a "verified / not verified" split in the status file.** This repo does it well and it makes the remaining risk visible without extra tooling.
- **A production-mode boot is a distinct check from tests.** Three defects came from it, matching the TrustRail finding.
- **Reverting code does not revert a migration.** Add "applied schema changes" to the rollback check in the release gate.
- **Experimental labels should name what was not done.** "No accuracy claimed" plus "not evaluated on real video" is more useful than "beta".
- **Publish history, not only the result.** A single squashed first commit hides the evidence this framework asks for. Commit per phase.
