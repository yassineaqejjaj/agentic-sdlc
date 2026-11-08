# AGENTS.md

## Project
One paragraph: purpose, stack, how to run and test.

## Rules for the agent
- Work only inside the task slice; do not edit unrelated files.
- Add or update tests with every change.
- Show command output as proof; do not claim success without it.
- Do not add dependencies without stating why and verifying they exist.
- Never use real credentials or real personal data.
- Ask before any irreversible or external action.

## Architecture diagrams

Author architecture, component, deployment, sequence, data-flow and state diagrams in Mermaid. Embed them in Markdown using fenced `mermaid` blocks, or keep Mermaid source for rendered application pages. Keep Mermaid as the editable source of truth; screenshots, brand artwork and directory listings are separate documentation assets.

## Commands
- Install:
- Test:
- Lint:
- Build:

## Boundaries
Paths the agent must not modify: secrets, migrations, release config.

## Definition of done
Tests green, lint clean, diff small, spec criteria met, limits noted.

