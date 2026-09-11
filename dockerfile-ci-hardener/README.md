# dockerfile-ci-hardener

Free [Agent Skill](https://agentskills.io) from ZWAiHub: audit local Dockerfiles and CI workflows (GitHub Actions primary), then produce a concrete checklist and proposed diffs.

## Install

Copy this directory into your agent's skills path, or follow your agent's skill-install docs. Directory name must remain `dockerfile-ci-hardener` (matches `name` in `SKILL.md`).

## What it does

- Hardens Dockerfiles: non-root, multi-stage, pinning, secrets, `.dockerignore`, HEALTHCHECK, layer hygiene
- Hardens CI: least-privilege permissions, pin actions by SHA, secret hygiene, fork safety, OIDC preference, fail-closed gates
- Outputs findings table + unified diffs for human review

## What it does not do

- No network calls by default
- Does not apply changes without review
- Does not run clusters or claim compliance certifications

## Layout

```
dockerfile-ci-hardener/
├── SKILL.md
├── LICENSE
├── README.md
└── references/
    ├── dockerfile-checklist.md
    └── github-actions-checklist.md
```

## License

MIT © 2026 ZWAiHub

## Links

- Kits: https://whop.com/zwaihub
- Source: github.com/studiocozylights-sketch/zwaihub-agent-skills
