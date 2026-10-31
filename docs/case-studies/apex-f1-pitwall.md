# Case study: APEX F1 Pit Wall

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [APEX F1 Pit Wall's git log and docs](https://github.com/yassineaqejjaj/apex-f1-pitwall). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

**Evidence caveat.** The history is thin: 7 commits, 2026-10-17 to 2026-10-23, with one large first commit. There is no spec, decisions log, status file or test report. The study is kept short rather than padded.

## What was built

A French-language race engineering dashboard using public OpenF1 data: an isometric GPS driver tracker with synchronized speed, throttle and brake gauges, a timing tower, driver comparison, telemetry, tyre stints, team radio, championship views, in-browser alert rules, and a deterministic pit-strategy simulation. It has no npm dependencies. A small Node server proxies a whitelist of OpenF1 endpoints (caching anonymous responses, origin allowlist, rate limits); the static frontend is built for Vercel and the server deploys on Railway. The README labels it **beta; simulations are estimates**.

**Scale of the work:** 7 commits, 25 tracked files. The first commit (`99f91b6`) adds 21 files and 292 lines in one step.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | README feature list and an explicit "Model limits" section | `README.md` |
| 2. Architecture | Deployment and security notes in the README (proxy rules, token handling). No separate architecture or decision records | `README.md` |
| 3. Agent build | One large initial build commit, then a feature commit and a restyle | `99f91b6`, `3ebaaf9`, `7dc2984` |
| 5. Verify | Syntax check, 8 unit tests (3 server, 5 strategy model) | `package.json`, `tests/*.test.mjs` |
| 6. Release gate | CI workflow running check, test and build on push and pull request | `c783264` |
| 8. Measure | None in the repo | n/a |

Autonomy level used: not recorded in the repo. `AUTHOR TO CONFIRM`.

## What the checks caught

The repo does not record any defect found by a check: commit messages are feature descriptions and there is no status or test report. The one visible correction is structural: `3ebaaf9` moved the Dockerfile to `server/Dockerfile` and updated `railway.json` and the `check` script in the same commit. The cause is not stated, so it is not listed as a caught defect.

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| None documented | n/a | n/a | n/a |

What the tests do pin down, as evidence of intent rather than caught bugs:

- Strategy model: pit loss and warmup charged once, tyre age not double counted, a late costly stop loses to staying out, out-of-range horizons rejected (`tests/strategy.test.mjs`).
- Server: unsupported endpoints and path traversal rejected, anonymous responses cached while authenticated ones are never shared, only configured origins allowed (`tests/server.test.mjs`).

## Process changes this led to

The history shows one process step: CI added after the build (`c783264`), running syntax check, unit tests and frontend build. Whether the workflow ever ran green is not visible in the repo, and this study did not check it. Nothing else is evidenced.

### How heuristics and estimates were validated or labelled

- The README states: "Strategy results are estimates, not predictions." The status line says "simulations are estimates".
- The model file header says it is deterministic with "No fitted predictions or proprietary tyre data". Degradation is linear and user-specified.
- Listed non-modelled factors: traffic, weather, Safety Car, grip, fuel, mandatory compound rules. The undercut estimate assumes the rival pits one lap later with equal pit losses.
- Sector boundaries and GPS placement are called approximate; missing timing intervals display as a dash.
- Validation of the model beyond its five unit tests (for example against real race outcomes) is not in the repo, so none is claimed here.

## What the process did not catch

- The strategy model's unit tests check its internal arithmetic, not whether its estimates match real races.
- The server proxy has 3 tests; rate limiting is described in the README but no test for it is visible in `tests/server.test.mjs`.
- Live-data behaviour requires an OpenF1 subscription; the README says it may need access, and no test covers it.
- No frontend test covers the UI code.

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion (for example the server proxy design and the decision to use no dependencies).
- The autonomy level used, and review time per commit, including the large first commit `99f91b6`.
- Whether the Dockerfile move in `3ebaaf9` fixed a failed deploy.
- Anything merged that you later regretted.

## Takeaways for the framework

- **Honest estimate labelling can live in the README**, but it is easier to trust with a test report that says what was run.
- **Add CI at the start, not last.** Here it arrived after the features (`c783264`).
- **Unit tests of a simulation prove arithmetic, not fitness.** A framework rule should require a stated validation (or an explicit "not validated") for any heuristic that influences decisions.
- **Thin history is itself a finding.** One 292-line first commit leaves no evidence of intermediate checks; commit per stage if the case is to be evidence-based.
