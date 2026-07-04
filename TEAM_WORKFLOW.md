# AI Development Workflow

## Recommended AI roles

- Owner: kazuma. Decides what to build and gives final approval.
- Lead AI: coordinates the work, edits files, and keeps the project moving.
- Implementation AI: builds a specific feature or fix in a clearly assigned area.
- Review AI: checks changed code for bugs, regressions, security issues, and missing edge cases.
- Test AI: designs or runs verification steps and reports what still needs checking.
- UI/UX AI: reviews app screens, copy, layout, and usability when frontend work is involved.

## How to use AI role separation

Use one Lead AI by default. Add separate AI agents only when the task benefits from parallel checking or split ownership.

Good examples:

- One AI implements a feature while another reviews likely risks.
- One AI works on frontend while another checks backend assumptions.
- One AI writes tests while another fixes the implementation.
- One AI reviews UI/UX after the screen is built.

Avoid splitting work when the task is small, because coordination can cost more than it saves.

## Branch naming

Use short names that show what the work is for.

- `feature/login-screen`
- `fix/save-error`
- `review/data-check`
- `test/user-flow`

## GitHub settings

Protect the `main` branch.

- Require a pull request before merging.
- Require approval from an administrator.
- Do not allow direct pushes to `main`.

## Pull request checklist

- What changed is written clearly.
- Risky files or behavior are named.
- Review AI checked bugs and regressions when needed.
- Test AI checked or described verification steps when needed.
- Owner approved before merge.
