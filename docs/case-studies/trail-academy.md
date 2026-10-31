# Case study: TRAIL Academy

> **Draft v0.1, reconstructed from the repository history.** Everything below that cites a commit comes from [TRAIL Academy's git log and docs](https://github.com/yassineaqejjaj/TRAIL-Academy). Passages marked **`AUTHOR TO CONFIRM`** are things only the author knows (what was rejected, how long reviews took) and are intentionally left open rather than invented.

## What was built

TRAIL Academy is a French-language online learning platform for AI skills: a public catalogue, a learner space (lessons, quizzes, deliverables, certificates with public verification), Stripe Checkout commerce, an instructor admin and CMS, live sessions, enterprise team features, and an AI tutor with cited retrieval in the lesson reader. The stack is Next.js 16 with PostgreSQL (row-level security), Better Auth, S3-compatible storage, self-hosted HLS video and SMTP. Business rules live in tested SQL functions.

**Scale of the work:** 42 commits between 2026-10-23 and 2026-10-31. One initial commit comes from Create Next App; most others carry an AI-agent `Co-Authored-By` trailer, and a few small commits (CI, cover image, docs) do not. Working documents: `TASKS.md` (a status board with a strict legend), `TESTING.md`, `SECURITY.md`, `docs/STRIPE.md`, `docs/GO_LIVE.md` and `docs/TUTEUR_IA.md`.

## How the lifecycle was applied

| Stage | What existed | Evidence |
|---|---|---|
| 1. Intent | `PRODUCT.md`, a product section in the README, a price grid applied in code, and an external audit dated 2026-10-27 whose follow-up is recorded | `PRODUCT.md`, `3f3ae6e`, `8a2e119` |
| 2. Architecture | `ARCHITECTURE.md`, `DATABASE.md`, a stack replacement (Supabase to open-source components) with its reasons in the commit body, and an investigation document written before the tutor was built | `d910ffa`, `1f86fae`, `docs/TUTEUR_IA.md` |
| 3. Agent build | Large phase commits (`cc12ed5` phases A to D, `d910ffa`, `be507dc`), then smaller feature commits | git log |
| 5. Verify | Tests on embedded PostgreSQL with real migrations, a real TCP data-layer suite, signature tests for Stripe, Playwright journeys on a production build, axe accessibility checks in both themes, manual mutation tests of critical rules | `TESTING.md` |
| 6. Release gate | `TASKS.md` legend separates done and tested, built but not run against the real service, and to do. A go-live checklist keeps legal validation as a blocker. | `TASKS.md`, `docs/GO_LIVE.md` |
| 7. Deploy | Vercel and Railway, a CI workflow, a documented successful GitHub Actions run | `d7a1e60`, `5cfbb74`, `c8b870e` |
| 8. Measure | Not measured. The README marks Lighthouse, Core Web Vitals and search ranking as not measured. | README section 9 |

**How the "Stripe in test mode, no live payments" limit was handled** (as the repo shows it):

- `docs/STRIPE.md` tells the operator to stay in test mode, which moves no money, and never to put the secret key in the repository.
- A guard refuses a live key unless both `ALLOW_LIVE_PAYMENTS=true` and `LEGAL_REVIEW_COMPLETED=true` are set (`src/lib/stripe/guard.ts`, `src/lib/readiness.ts`). The readiness check also flags a test key combined with live flags.
- Access is granted only by the signed webhook, idempotently, never by the return page. Signature verification is tested with real signatures (valid, missing, wrong secret, altered body).
- `scripts/verify-stripe-local.mts` (`b6029b5`) creates a real Checkout session against Stripe test mode and replays signed events built from it. It never fills in the card form.
- The docs state what was never run: no card was entered on the Stripe page in that script, and live mode, Managed Payments eligibility and fees are not validated. README section 9 later records a test-mode purchase validated end to end by the owner; the repo has no log of that run.

## What the checks caught

The history is mostly feature commits with short messages. Only fixes whose cause is visible in the repo are listed.

| Defect | Found by | Why earlier checks missed it | Commit |
|---|---|---|---|
| The checkout code assumed the account sells as itself and retried without Managed Payments on a Stripe conflict. Checks against the Stripe API showed Stripe requires a product tax code on each line, forces Stripe Tax on, and rejects custom invoice data and custom text. Checkout was rebuilt for Managed Payments as the default, with a switch for the other model. | Validation against the Stripe API in test mode (commit title) | Unit tests of the builder exercised the first model; the API's real constraints were only visible by calling it | `a513100` |
| The first version of `layout.tsx` failed the CI typecheck because of a layout props typing issue | First CI run | Not run on a real runner before; the commit title says "for CI" | `521d109` |
| Badge contrast and `dl` markup failures | Automated axe checks added in the same commit | No accessibility check existed before | `be507dc` |
| A placeholder favicon and icon shipped and were replaced with the brand mark | Not recorded | Not recorded | `e30e838` |
| Catalogue figures hard-coded in copy ("18", "4 phases") drifted from the catalogue. They were derived from one source and consistency tests were added. | The 2026-10-27 external audit (`docs/GO_LIVE.md`) | Copy was written separately from data | `8a2e119` |
| Pages shown as "available" before purchase opened. Offers now track whether they are open for sale, with three availability states. | Not recorded for the first fix; the 2026-10-27 audit follow-up for the labels | The offer and published states were conflated | `d910ffa`, `a651d08`, `8a2e119` |

The strongest evidence of the checks working is `TESTING.md`: critical rules (quiz marking, admin guard, Stripe session verification, refund revocation, price history, completion, publish readiness) were deliberately broken and the suite was confirmed to fail, then restored. These are self-reported manual tests.

## Process changes this led to

1. **A three-state status legend.** `TASKS.md` separates "done and tested" from "built, not run against the real service", so the unverified part is visible. README section 9 and `TESTING.md` repeat it with a "not tested" list.
2. **Predict what you have not run.** The commit bodies of `cc12ed5` and `d910ffa` state in advance that external integrations were not run against live services; later work then closed some of them (Stripe test mode, Brevo SMTP, deployment, per README section 9).
3. **Write the investigation before the feature.** `docs/TUTEUR_IA.md` (`1f86fae`) states the design, the leak risk (paid content) and the guard (row-level security) before the tutor was built (`8a7fb47`), and records that it has never run with a real model.
4. **Legal and commercial decisions are handed back to the owner.** `docs/GO_LIVE.md` and `TASKS.md` list items the code cannot decide: legal texts, withdrawal rights, domain, review-service promise.
5. **Guards for irreversible actions.** Live payments need two explicit flags; sensitive manual actions need a recorded reason.

## What the process did not catch

- Several integrations were never exercised: S3/MinIO upload, HLS playback of a real encoded video, `docker-compose.yml`, backup restore, Google OAuth, and the LLM-based advisor and tutor (`TESTING.md`, README section 9).
- Stripe live mode, Managed Payments eligibility and fees are not validated, and the legal texts are drafts awaiting a lawyer.
- No manual accessibility audit, Lighthouse run or load test.
- Some docs drifted. `TESTING.md` still gives 514 tests, 9 journeys and a CI success tied to commit `6ac17d7`, while the README and the same file elsewhere say 23 journeys; the `d910ffa` commit message says 185 tests. The counts were reconciled only in later docs commits (`b5b730d`, `f895905`), so the numbers in the repo are not all from the same date.
- Three Playwright journeys depend on seed data and can fail on a non-fresh local database (README section 9).

## Review and rejection log

`AUTHOR TO CONFIRM` (this is the evidence that humans, not the agent, owned the decisions):

- Autonomy level used per stage. The repo does not state it.
- Agent proposals you rejected or rewrote, and why (for example the pricing grid revisions in `3f3ae6e` and `be507dc`, the Supabase replacement decision, the Managed Payments choice).
- The 2026-10-27 external audit: who ran it and which findings you declined.
- The test-mode purchase the README says you validated: when, and what you saw.
- Approximate review time per phase.
- Anything merged that you later regretted.

## Takeaways for the framework

- **Use a status legend that distinguishes "tested" from "built but not run against the real service".** It kept the Stripe, storage and video gaps honest here.
- **Call the real service in test mode early.** The Stripe API rejected assumptions that unit tests had encoded; a sandbox call is cheap compared with a wrong integration model.
- **Put the limit in code, not only in docs.** The two-flag live-payment guard and the test-mode readiness message make "no live payments" a property of the system.
- **Break a rule on purpose to prove the test.** The manual mutation checks are a low-cost way to confirm that a green suite can actually go red.
- **Keep counts in one place.** Test counts repeated across several docs drifted; generate or link them.
