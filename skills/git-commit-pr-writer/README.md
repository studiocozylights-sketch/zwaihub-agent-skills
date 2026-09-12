# git-commit-pr-writer

Agent skill (agentskills.io) that drafts **conventional commit messages** and
**PR titles/bodies** from real `git status` / `diff` / `log` — not from guesses.

## When to use

Ask the agent to commit, write a commit message, or draft/open a PR. The skill
inspects the repo first, matches existing commit style when present, and stays
within clear limits (no invented changes, ticket IDs, or pushes).

## Install

Copy or clone this folder into your agent's skills directory so
`git-commit-pr-writer/SKILL.md` is discoverable. Directory name must stay
`git-commit-pr-writer` to match the skill `name`.

Requires `git` on PATH. Optional: GitHub CLI (`gh`) if you want the agent to
create PRs when you ask.

## Contents

| Path | Role |
|------|------|
| `SKILL.md` | Instructions for the agent |
| `references/commit-style.md` | Conventional-commit cheat sheet |
| `LICENSE` | MIT |

## License

MIT © 2026 ZWAiHub
