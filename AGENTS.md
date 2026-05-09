# tebanashi

## Repository Rules

- When the user invokes AI-DLC, read and follow `.aidlc/aws-aidlc-rules/aws-aidlc-rules/core-workflow.md` to start the workflow.
- AI-DLC related documentation should be generated in Japanese.

## Commit Rules

- Use Conventional Commits.
- Keep each commit focused on one logical change.
- Prefer these types:
  - `feat:`
  - `fix:`
  - `docs:`
  - `test:`
  - `refactor:`
  - `chore:`
  - `ci:`
  - `build:`
- Work from the latest `main`.
- Before opening a PR, check existing open or recently merged PRs.
- Keep PRs focused and small.
- PR titles should follow Conventional Commits, e.g. `feat: add intake unit`.
- If implementation changes architecture, contracts, or behavior, update related AI-DLC documents in the same PR.

## Branch Strategy

This repository follows a lightweight trunk-based workflow.

- `main` is the single integration branch.
- Always start work from the latest `main`.
- Keep branches short-lived.
- Keep changes focused and incremental.
- Merge completed work back into `main` quickly.
- Avoid long-running or phase-based branches.

Preferred branch types:

- `docs/<topic>`
- `feature/<unit-name>`
- `fix/<topic>`

Examples:

- `docs/application-design`
- `feature/foundation`
- `feature/intake-card-creation`
- `fix/browser-gate`
