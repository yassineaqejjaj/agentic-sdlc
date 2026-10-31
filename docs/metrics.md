# Metrics

All values are **not yet measured** for the author's projects. Publish a metric only with its method and period.

| Metric | Definition | Why it matters |
|---|---|---|
| Acceptance rate | Agent changes kept without major rework / changes proposed | Is autonomy deserved? |
| Review time | Median minutes from pull request to decision | Is review sustainable? |
| Escaped defects | Defects found after release per release | Are the gates working? |
| Rework ratio | Lines changed within 14 days of merge / lines merged | Hidden quality problems |
| Gate failures | Releases blocked at stage 6 / releases attempted | Are gates real? |
| Cost per shipped change | Model and compute cost / merged changes | Is it economical? |
| Incident count | Security, data or permission incidents | Drives demotion |

## Method

Record per change: task id, level, review minutes, outcome, rework within 14 days, cost. Report monthly per project with sample sizes. Never compare projects with different risk levels without saying so.
