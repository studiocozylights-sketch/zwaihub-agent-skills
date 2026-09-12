# ZWAiHub Agent Skills

Free, open [Agent Skills](https://agentskills.io) from [ZWAiHub](https://whop.com/zwaihub).

We deploy this stuff in production for a living. These skills are MIT-licensed, with provenance metadata (`network: none` where applicable), and meant to be installed as-is.

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

```bash
clawhub install git-commit-pr-writer
clawhub install dockerfile-ci-hardener
```

(ClawHub publish pending: GitHub account age gate until ~20 Sep 2026.)

## Provenance

- Open source (MIT)
- Version pinned in each skill's frontmatter `metadata.version`
- Network calls documented in frontmatter (`network: none` for both free skills)
- No secrets required to use

## More kits

Paid deployment kits (scripts + worked example repos): [whop.com/zwaihub](https://whop.com/zwaihub)

---
Built by ZWAiHub — Pakistan.
