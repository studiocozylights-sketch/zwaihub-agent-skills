# ZWAiHub Agent Skills

Free, open [Agent Skills](https://agentskills.io) from ZWAiHub.

We deploy this stuff in production for a living. These skills are MIT-licensed, with provenance metadata in frontmatter (network policy is stated per skill — read it; some optional git/`gh` steps need a remote), and meant to be installed as-is.

## Free skills

| Skill | What it does |
| --- | --- |
| [`git-commit-pr-writer`](./skills/git-commit-pr-writer/) | Conventional commits + PR titles/bodies from real `git` state |
| [`dockerfile-ci-hardener`](./skills/dockerfile-ci-hardener/) | Audit/harden Dockerfiles and CI (GitHub Actions primary) |

## Install

### Claude Code marketplace

```text
/plugin marketplace add studiocozylights-sketch/zwaihub-agent-skills
/plugin install zwaihub-free-skills@zwaihub-agent-skills
```

### ClawHub / OpenClaw

Not published on ClawHub yet (GitHub account age gate until ~20 Sep 2026).
When live, install will look like:

```bash
# Available after ClawHub publish (~20 Sep 2026) — do not run until then
# clawhub install git-commit-pr-writer
# clawhub install dockerfile-ci-hardener
```

## Provenance

- Open source (MIT) — see root `LICENSE`
- Version pinned in each skill's frontmatter `metadata.version`
- Network policy documented in each skill's frontmatter (local-first; optional remote only when the user asks)
- No secrets required to use

## More kits

Paid deployment kits (scripts + worked example repos) ship on our Whop storefront
when live. Until then, treat this repo as the free catalog only — do not trust a
placeholder Whop URL.

---
Built by ZWAiHub — Pakistan.
