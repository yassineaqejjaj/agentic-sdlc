# Risks specific to agent-driven development

| Risk | What it looks like | Controls |
|---|---|---|
| **Scope drift** | The agent changes files or behavior beyond the task | Task slices in the spec; path allowlists; reviewers reject unrelated diff |
| **Tests written to pass** | Tests assert what the code does, not what it should do | Write acceptance criteria and key tests before implementation; mutation or negative tests for critical logic |
| **Invented dependencies** | Packages, APIs or options that do not exist, or typosquatted names | Verify every new dependency; lock files; dependency review in CI |
| **Leaked secrets** | Keys in code, logs or prompts | Secret scanning in CI; no real credentials in agent sessions; test keys only |
| **Overconfident claims** | Reports success without evidence | Require command output or a CI link as proof of every claim |
| **Unreviewable volume** | Diffs too large to review honestly | Hard size limit per change; split before review |
| **Runaway cost or loops** | Retries, long sessions, large contexts | Budgets, step limits, cost tracking per task |
| **Context rot** | The agent works from stale or contradictory context | Short sessions; versioned `AGENTS.md`; governed, cited context sources |
| **Unsafe actions** | Destructive or external actions taken without approval | Least privilege; approval per irreversible action; action receipts |
| **Skill atrophy** | Reviewers stop understanding the code | Rotate who reviews; reject changes you cannot explain |

Failures in the field should be added here with the control that would have caught them.
