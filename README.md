# ZWAiHub Agent Skills

Free, open [Agent Skills](https://agentskills.io) from [ZWAiHub](https://whop.com/zwaihub).

We deploy this stuff in production for a living. These skills are MIT-licensed, signed by provenance metadata (`network: none` where applicable), and meant to be installed as-is.

## Free skills

| Skill | What it does |
| --- | --- |
| [`git-commit-pr-writer`](./git-commit-pr-writer/) | Conventional commits + PR titles/bodies from real `git` state |
| [`dockerfile-ci-hardener`](./dockerfile-ci-hardener/) | Audit/harden Dockerfiles and CI (GitHub Actions primary) |

## Install

Each skill is a folder with a `SKILL.md`. Point your agent at the folder, or add this repo as a Claude Code marketplace:

```text
/plugin marketplace add studiocozylights-sketch/zwaihub-agent-skills
```

Then install the free-skills plugin from the marketplace browser.

## Provenance

- Open source (MIT)
- Version pinned in each skill's frontmatter `metadata.version`
- Network calls documented in frontmatter (`network: none` for both free skills)
- No secrets required to use

## More kits

Paid deployment kits (scripts + worked example repos): [whop.com/zwaihub](https://whop.com/zwaihub)

---
Built by ZWAiHub — Pakistan.
