# Autonomy levels

Autonomy is a setting per stage (and per area of the codebase), not a mood.

## Definitions

| Level | Agent | Human | Evidence needed to be here |
|---|---|---|---|
| L0 Suggest | Proposes only | Writes and decides | None |
| L1 Assist | Edits in session | Reviews every diff | Spec and acceptance criteria exist |
| L2 Delegate | Completes a scoped task, opens a PR | Reviews the PR, owns the gates | Automated tests cover the area; CI blocks failing changes |
| L3 Supervise | Verifies and merges to a non-production branch | Audits samples, approves release | High coverage, regression evals, reversible deploys, no sensitive data or irreversible actions |

## Hard caps

- **Irreversible actions** (deleting data, payments, external messages, production changes): **L0 or L1**, with explicit human approval per action.
- **Sensitive or regulated data**: **L1** at most. Use synthetic data in development.
- **Security-relevant code** (authentication, authorization, cryptography): **L1** at most, with a security-focused review.

## Promotion

Promote a stage by one level when, over at least 10 consecutive changes:

- no defect escaped that review or checks should have caught;
- the acceptance rate without major rework is stable or improving;
- the automated checks that justify the new level exist and run on every change.

## Demotion

Drop one level immediately after:

- a defect reaching users that the process should have caught;
- a secret, data or permission incident;
- a change merged without the required review or gate.

Record the reason in a decision record. Promote again only through the normal path.
