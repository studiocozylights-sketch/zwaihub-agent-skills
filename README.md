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
/plugin marketplace add zwaihub/zwaihub-agent-skills
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
- Version: root `VERSION` is the plugin release; skill frontmatter `metadata.version` tracks it (lockstep for this free catalog)
- Network policy in each skill frontmatter: `none` | `optional` | `required` (machine-readable)
- No secrets required to use

## More kits

Give away the skill; sell the deployment. Paid kits (scripts + worked example
repos): [whop.com/zwaihub](https://whop.com/zwaihub) (coming soon).

---
Built by ZWAiHub — Pakistan.
