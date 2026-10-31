# Stage contracts

| # | Stage | Entry | Activities | Exit criterion | Proof | Default level |
|---|---|---|---|---|---|---|
| 1 | Intent | A problem worth solving | Define users, outcome, acceptance criteria, non-goals | Criteria are testable; non-goals listed | `SPEC.md` | L0 to L1 |
| 2 | Architecture | Approved spec | Boundaries, data model, risks, key decisions | Risks named; decisions recorded | Decision records | L0 |
| 3 | Agent build | Spec and task slice | Implement in small steps with tests | Change is small, scoped and passes local tests | Pull request | L1 to L2 |
| 4 | Human review | Pull request | Read the diff; check security, data, scope, tests | Reviewer can explain the change; checklist complete | Review checklist | Human |
| 5 | Verify | Reviewed change | Run tests, end-to-end checks, evals, regression comparison | All required checks green; evals within thresholds | CI and eval report | Automated |
| 6 | Release gate | Verified candidate | Check gate criteria, limitations, rollback | Gate criteria met; known limits documented | Release record | Human |
| 7 | Deploy | Approved release | Pipeline, preview, versioning | Reproducible, versioned deployment | Pipeline log | Automated |
| 8 | Measure | Live release | Track usage, quality, cost, incidents | Findings feed the next intent | Metrics and post-mortems | Human |

## Rules

- A stage cannot start without its entry condition.
- A stage cannot close without its proof.
- Small changes can use a short form, but never skip stages 4 and 5.
- Findings from stage 8 always create an item in stage 1.
