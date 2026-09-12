# Conventional commit cheat sheet

Quick reference for [git-commit-pr-writer](../SKILL.md). Prefer the repo's
existing `git log` style when it conflicts with this sheet.

## Format

```
type(scope): subject

body (optional)

footer (optional)
```

- **type** — required when using conventional form
- **scope** — optional noun for area (`auth`, `api`, `ui`, `ci`)
- **subject** — imperative, ~50 chars, no trailing period
- **body** — why / context; wrap ~72 chars
- **footer** — trailers only when real (`Fixes #123`, `BREAKING CHANGE: …`)

## Types

| Type | Use for |
|------|---------|
| `feat` | New user-facing capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace; no behavior change |
| `refactor` | Internal restructure; no behavior change |
| `perf` | Performance improvement |
| `test` | Adding or fixing tests |
| `build` | Build system or dependencies |
| `ci` | CI config / scripts |
| `chore` | Maintenance that does not fit above |
| `revert` | Revert a previous commit |

## Subject rules

1. Imperative: "Add rate limit" not "Added rate limit"
2. Do not capitalize the first word after `type(scope):` beyond normal proper nouns
3. No period at the end
4. Focus on intent, not a file dump

## When to omit conventional types

If recent commits are plain sentences (`Fix login timeout on Safari`), mirror
that. Consistency with the repo beats forcing a new convention.

## Breaking changes

Either:

```
feat(api)!: remove legacy /v1/orders endpoint
```

or a footer:

```
BREAKING CHANGE: /v1/orders removed; migrate to /v2/orders.
```

Only mark breaking when the diff actually removes or changes a public contract.

## Multi-commit split hints

| Signal in diff | Action |
|----------------|--------|
| Feature + unrelated bugfix | Two commits |
| Logic + pure formatting | Two commits |
| Feature + its tests | One commit |
| Rename + behavior change in same paths | One commit if inseparable; else rename first |

## Anti-patterns

- `update stuff`, `fix`, `wip`, `temp`
- Subjects that only list filenames
- Ticket IDs guessed from branch names without user confirmation
- Bodies that restate the subject without adding context
