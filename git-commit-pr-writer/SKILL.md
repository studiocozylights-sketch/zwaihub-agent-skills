---
name: git-commit-pr-writer
description: >
  Write conventional git commit messages and pull request titles/bodies from real
  repository state. Use when the user asks to commit, draft a commit message,
  write a PR description, open a pull request, summarize staged changes, or
  match existing commit style. Inspects git status, diff, and log first; never
  invents file changes or ticket IDs. Keywords: git commit, conventional commits,
  PR description, pull request, changelog, commit message, gh pr create.
license: MIT
metadata:
  author: ZWAiHub
  version: "1.0.0"
  network: none
  provenance: open-source
---

# Git Commit & PR Description Writer

Write accurate commit messages and PR descriptions grounded in the actual
working tree. Prefer matching the repository's existing style when one is clear.

## When to use

- User asks to commit, draft a message, or summarize changes
- User asks for a PR title/body or to open a pull request
- Multiple files changed and the message needs a clear scope
- Repo already uses conventional commits (or a close variant)

## When not to use

- No git repository, or git is unavailable
- User only wants code edits with no commit/PR text
- Secrets or credentials appear in the diff — stop and warn; do not draft a
  commit that would record them

## Provenance

- **Network:** none. This skill does not call external APIs.
- **License:** MIT (open source). Version pinned in frontmatter metadata.
- **Tools expected:** `git` CLI. Optional: `gh` for creating PRs when the user
  explicitly asks. Do not push unless the user explicitly requests it.

## Hard limits

- Do **not** invent file changes, hunks, or behavior not present in `git status`
  / `git diff`.
- Do **not** invent ticket IDs, issue numbers, or co-authors.
- Do **not** push, force-push, or amend published history unless the user
  explicitly asks and it is safe.
- Do **not** skip hooks (`--no-verify`) unless the user explicitly requests it.
- Do **not** commit `.env`, key files, or other secrets. Warn if they appear
  staged or unstaged.

## Workflow

### 1. Inspect before writing

Run these in parallel when possible:

```bash
git status
git diff          # unstaged
git diff --staged # staged
git log -8 --oneline
```

If nothing is staged and the user wants a commit, either stage relevant paths
(with user intent) or draft from the unstaged diff and state that staging is
still required.

Read enough of the diff to name the *why*, not only the file list. For large
diffs, summarize by concern (API, UI, tests, config) from the paths and hunks.

### 2. Match repo style

From `git log`:

- If recent commits use `type(scope): subject` → use Conventional Commits.
- If they use a prefix like `[fix]` or `Fix:` → mirror that pattern.
- If subjects are short imperative sentences without types → keep that form.
- If mixed, prefer Conventional Commits unless the majority clearly differs.

See [references/commit-style.md](references/commit-style.md) for the type cheat
sheet.

### 3. Write the commit message

**Subject rules**

- Imperative mood: "Add", "Fix", "Remove" — not "Added" / "Fixes"
- ~50 characters when practical; hard stop around 72
- No trailing period
- Lowercase after the type/scope prefix when using conventional form
- Scope optional; use when it clarifies (e.g. `auth`, `api`, `docs`)

**Body (optional)**

- Blank line after subject
- Explain motivation and tradeoffs; wrap ~72 chars
- Bullet lists for multiple related changes in one commit
- Footer only for real trailers (`Fixes #123` only if the user supplied the ID)

**Conventional form**

```
type(scope): subject

Optional body explaining why.

Optional footer
```

Common types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`,
`build`, `ci`, `chore`, `revert`.

### 4. Split vs single commit

Prefer **one concern per commit**. Split when:

- Unrelated fixes sit beside a feature
- Mixed formatting-only changes with logic changes
- Independent modules could revert separately

Keep a single commit when files are one logical change (e.g. feature + its
tests + minimal docs).

When splitting, propose an ordered list of commits with paths for each; do not
claim a split was performed until the user agrees and you stage accordingly.

### 5. PR title and body

**Title:** Same rules as a commit subject; often matches the primary commit or
summarizes the branch. Prefer conventional form if the repo uses it.

**Body structure** (adapt to repo templates if `.github/PULL_REQUEST_TEMPLATE*`
exists — read and fill those sections instead of inventing a parallel format):

```markdown
## Summary
- 1–3 bullets of what changed and why

## Test plan
- [ ] Concrete steps to verify
- [ ] Edge cases touched by the diff

## Breaking changes
- None
```

or list real breaking changes (API removals, config renames, migration needed).

Ground every bullet in the diff. If tests were not added, say so honestly in
the test plan rather than inventing coverage.

### 6. Creating the commit or PR

Only when the user asks to create it:

- Stage intended paths (`git add …`), not blanket `git add .` unless requested
- Commit with a HEREDOC so formatting stays intact:

```bash
git commit -m "$(cat <<'EOF'
type(scope): subject

Body if needed.

EOF
)"
```

- For PRs: push only if needed and requested; use `gh pr create` with title and
  body via HEREDOC. Confirm base branch from repo default or user instruction.

After commit, show `git status` (and the new hash/subject) so the user can
verify.

## Quality checklist

Before handing text back:

- [ ] Message matches inspected diff (no phantom files)
- [ ] Style matches recent `git log` when a pattern exists
- [ ] Subject is imperative, concise, no trailing period
- [ ] Multi-concern work was split or explicitly kept together with rationale
- [ ] No invented ticket IDs or test results
- [ ] Secrets not included in the commit

## Examples

**Diff:** auth middleware rejects expired JWT; test added.

```
fix(auth): reject expired JWT in middleware

Return 401 when exp is in the past. Add unit coverage for skew boundary.
```

**Diff:** README install section only.

```
docs: clarify local install steps in README
```

**PR summary bullets** should name behavior ("Rate-limit login to 5/min per IP"),
not only filenames.

---
Built by ZWAiHub — we deploy this stuff in production for a living.
More kits: https://whop.com/zwaihub · Source: github.com/studiocozylights-sketch/zwaihub-agent-skills
