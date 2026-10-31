# Case study: SimuLearn

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [SimuLearn's git log and docs](https://github.com/yassineaqejjaj/SimuLearn). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

SimuLearn is a French-language voice simulation prototype for healthcare training (nursing school context). A learner speaks with an AI patient, then receives a competency-by-competency evaluation backed by quotations from their own conversation. A trainer area lets a human review and comment on evaluations without overwriting the AI output. Stack: FastAPI, Next.js, Alembic, Docker Compose, GitHub Actions. Voice (OpenAI GPT-Live) and evaluation (Anthropic Claude) have real providers plus mock providers for development. Data is fictional.

**Scale of the work:** 2 commits. The MVP commit `fc9f071` (2026-10-22) contains everything; `2b2da6c` (2026-10-23) adds a security policy. This is too thin to show a lifecycle, so the study is short and based on the README and docs.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | Product principles and one scenario with a 7-competency rubric in the README; no separate spec file | `README.md` |
| 2. Architecture | Architecture, database, voice-integration, evaluation and security docs | `docs/*.md` |
| 3. Agent build | One MVP commit co-authored by the AI agent | `fc9f071` |
| 5. Verify | Stated: 40 API tests, 7 frontend tests, one Playwright learner path, all on mock providers. Counts are from the README and were not re-run | `README.md` (Tests) |

Autonomy level used: not recorded. `AUTHOR TO CONFIRM`.

Stages 4, 6, 7 and 8 leave no trace. No release gate exists.

## What the checks caught

The history records no defects found and fixed: there is no fix commit and no test report. Nothing is listed here on purpose.

## Process changes this led to

The repo shows design choices rather than process changes:

1. **Evidence-checked AI output.** The evaluation is structured output validated against a schema, with retries, and a deterministic verifier checks that each quoted proof exists in the session and matches the speaker. A rejected proof is removed and logged, and a score with no verified proof is removed. The failure state is visible and can be rerun, never a silent partial evaluation (`docs/evaluation.md`).
2. **Unknowns are not invented.** Claude pricing is `null` in `apps/api/config/pricing.json`, so cost displays as unknown. No latency targets are claimed.
3. **A status table written first.** The README opens with what is tested, what is only written, and what is untested.

## What the process did not catch

Declared by the repo itself:

- The real GPT-Live voice integration was never run against the real service, for lack of API keys.
- The Claude evaluation was tested only with a fake client.
- Keycloak token verification was never tested against a real instance.
- PostgreSQL and Docker Compose were written but not launched; tests and the migration ran on SQLite.
- Two French voices (`sillage`, `brise`) are absent from the documentation consulted.
- Multi-organisation isolation is tested at query level, not by a full HTTP test.
- The mock evaluator is a non-pedagogical heuristic, so the screenshots show the interface, not real evaluation quality.
- Speech interruptions: the server cannot guarantee that unplayed speech is not treated as heard.

## Safety limits and declared limitations

- Fictional scenarios only; the README forbids real patient records.
- Results are "training feedback, not certification"; no global score, each competency separate; unobserved competencies stay "not evaluated".
- No audio is recorded. Learner identity is not sent to the evaluator. Learner export and deletion endpoints exist; retention purge is written but not scheduled.
- `docs/security.md` lists legal review, DPIA, provider contracts and GDPR transfer questions as to do before any real use, and says using these providers does not guarantee compliance.
- Dev-mode login is refused when `APP_ENV=production`.
- `.github/SECURITY.md` (`2b2da6c`) repeats the synthetic-data rule. The repo has no licence yet (README: all rights reserved).

## Review and rejection log

`AUTHOR TO CONFIRM`:

- Agent proposals you rejected or rewrote, and why.
- Architecture choices you made against the agent's suggestion.
- Approximate review time of the MVP commit.
- Whether the screenshots and the UI-to-mockup fidelity were reviewed by you.

## Takeaways for the framework

- Mock-only verification proves the flow, not the integrations. The framework should force a separate "real provider" exit criterion, or an explicit untested label, as this README does.
- Deterministic verification of LLM output (quotes checked against source) is a pattern worth documenting under stage 5.
- With one agent commit, review evidence cannot be reconstructed from history.
