---
name: dockerfile-ci-hardener
description: >-
  Audit and harden Dockerfiles and CI pipelines (GitHub Actions primary) for
  security and least privilege. Use when reviewing Dockerfiles, docker-compose,
  GitHub Actions workflows, GitLab CI, build pipelines, image digests, non-root
  users, secret handling, action pinning by SHA, OIDC cloud auth, .dockerignore,
  multi-stage builds, or producing concrete hardening diffs and checklists.
license: MIT
metadata:
  author: ZWAiHub
  version: "1.0.2"
  network: none-by-default
  provenance: open-source
---

# Dockerfile & CI Pipeline Hardener

Teach the agent to audit local Dockerfiles and CI workflow files, then produce a
concrete checklist and proposed diffs. Read-only by default; never apply changes
without explicit human review.

## When to use

- User asks to harden, secure, or review a Dockerfile / CI pipeline
- PR or repo contains `Dockerfile*`, `.dockerignore`, or `.github/workflows/*`
- Questions about non-root containers, pinned actions, secrets in builds, OIDC

## When not to use

- Live cluster / Kubernetes / runtime incident response
- Claiming SOC2, CIS, PCI, or other compliance certification
- Blindly rewriting production pipelines without a reviewable diff

## Provenance and limits

- **Network:** none by default for the audit itself (read local Dockerfiles /
  workflows only). Do not fetch images, pull actions, or call APIs unless the
  user explicitly asks you to verify a digest/tag online.
- **Inputs:** local files only (`Dockerfile*`, `.dockerignore`, `.github/workflows/*`,
  `.gitlab-ci.yml`, similar CI configs the user points at).
- **Outputs:** audit findings + proposed patches. Do not auto-apply; do not
  commit or push unless the user explicitly asks after reviewing the diff.
- **Honest limits:** this skill does not run production clusters, does not scan
  registries, does not prove compliance, and does not replace a human security review.

## Cold start

1. Locate inputs in the workspace (ask if ambiguous):
   - `Dockerfile`, `Dockerfile.*`, `*/Dockerfile`
   - `.dockerignore`
   - `.github/workflows/*.yml` / `*.yaml`
   - Optionally `.gitlab-ci.yml`, `Jenkinsfile`, `azure-pipelines.yml`
2. Read each file fully before advising.
3. Run the checks in `references/dockerfile-checklist.md` and
   `references/github-actions-checklist.md`.
4. Emit a structured report (format below).
5. Propose unified diffs or side-by-side snippets for every fix you recommend.
6. Stop for review. Do not modify files unless the user confirms.

## Output format

```markdown
## Scope
- Files reviewed: …
- Assumptions: …

## Findings
| ID | Severity | File | Check | Issue | Recommendation |
|----|----------|------|-------|-------|----------------|
| D01 | high | Dockerfile | non-root | runs as root | add USER … |
| G01 | high | ci.yml | pin-actions | @v4 floating tag | pin to SHA |

## Proposed diffs
… unified diffs …

## Residual risk / out of scope
- Items not verified locally (e.g. registry scan, cluster RBAC)
- Items requiring human policy decisions
```

Severity: `critical` | `high` | `medium` | `low` | `info`.

## Dockerfile hardening (apply checklist)

Work through `references/dockerfile-checklist.md`. Priorities:

1. **No secrets in image layers** — never `ENV`/`ARG` passwords, tokens, private keys.
   Prefer BuildKit secret mounts (`RUN --mount=type=secret`) when build-time
   credentials are unavoidable; document that the secret must not land in a layer.
2. **Non-root runtime** — create a user/group, `USER` before `CMD`/`ENTRYPOINT`.
   Fix ownership on copied artifacts (`COPY --chown=`).
3. **Multi-stage builds** — compile/test in a builder stage; copy only runtime
   artifacts into a slim/distroless final stage.
4. **Base images** — prefer official slim/distroless where fit. Pin thoughtfully:
   digest (`image@sha256:…`) for max reproducibility, or major/minor tags with a
   documented update process — never `latest` in production Dockerfiles.
5. **COPY vs ADD** — use `COPY` unless you need ADD's archive URL semantics
   (usually you do not; avoid remote URLs in ADD).
6. **Layer hygiene** — combine `apt-get update && apt-get install`, then clean
   (`rm -rf /var/lib/apt/lists/*`) in the same `RUN`. Minimize layers that
   invalidate often (deps before app source).
7. **`.dockerignore`** — exclude `.git`, secrets, local env files, build
   artifacts, `node_modules`, `.env*`, credentials.
8. **HEALTHCHECK** — add only when the image has a usable probe binary and the
   orchestrator does not already own health checks. Distroless/scratch have no
   `/bin/sh`, so shell-form `HEALTHCHECK CMD curl …` fails at runtime. Prefer
   orchestrator probes, or exec-form against a binary that exists in the image.
   Mark as n/a on distroless when health lives outside the container.
9. **Package managers** — pin versions where practical; avoid `curl | bash`
   installers without checksum verification.

### Example patterns (illustrative)

These are **patterns**, not a drop-in app. They will not build as-is without your
own `package.json`, lockfile, and `dist/` output. Adapt them; do not paste blind.

Non-root + multi-stage (Node-style sketch). Distroless Node images already set
`ENTRYPOINT` to `node` and run as UID 65532 on the `:nonroot` tag — do not invent
a `USER nonroot` name unless that user exists in the image. Install **build**
deps in the builder stage; copy only runtime artifacts into the final stage.

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-bookworm-slim AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --omit=dev

FROM gcr.io/distroless/nodejs22-debian12:nonroot
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3000
# ENTRYPOINT is already node; CMD is the script path
CMD ["dist/server.js"]
```

Apt clean in one layer (pin a real package version from `apt-cache policy`, not a
glob like `curl=8.*` — apt does not accept that form):

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/*
```

## CI pipeline hardening (GitHub Actions primary)

Work through `references/github-actions-checklist.md`. Priorities:

1. **Least privilege `permissions`** — set top-level `permissions:` to the minimum
   (`contents: read` default; add write only on jobs that need it). Prefer job-level
   grants over repo-wide defaults.
2. **Pin actions by full commit SHA** — not mutable tags (`@v4`). Comment the
   tag next to the SHA for humans.
3. **Secrets** — never `echo` secrets; mask them; do not pass secrets to
   untrusted / fork PRs. Prefer environment protection rules for deploy jobs.
4. **Fork PRs** — do not publish images, deploy, or use write tokens on
   `pull_request` from forks. Use `pull_request_target` only with extreme care
   (never checkout untrusted code and run it with secrets).
5. **OIDC over long-lived cloud keys** — for AWS/GCP/Azure, prefer
   `id-token: write` + cloud OIDC federation instead of static access keys in secrets.
6. **Caches** — scope caches safely; do not restore caches from untrusted PRs
   into privileged jobs in ways that allow cache poisoning.
7. **Fail closed** — `set -euo pipefail` in shell steps; treat scan failures as
   blocking where policy requires; do not `continue-on-error: true` on security gates
   without documenting why.
8. **Artifact / registry publish** — tag immutably; sign if the org already uses
   signing; restrict who can push.

### GitHub Actions sketch

```yaml
name: build
on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      # id-token: write   # enable only for OIDC deploy jobs
    steps:
      - name: Checkout
        # Replace <40-char-sha> with a commit you verified (e.g. git ls-remote
        # for tag v4.2.2). Never invent a SHA or mislabel the tag comment.
        uses: actions/checkout@<40-char-sha>  # v4.2.2
      - name: Build image
        env:
          IMAGE_TAG: ${{ github.sha }}
        run: docker build -t "app:${IMAGE_TAG}" .
```

Do not paste a concrete SHA into this sketch from memory. Verify the pin, then
substitute. Never splice untrusted `github.event.*` values into `run:` — put them
in `env:` and expand as shell variables (see checklist G-INJ-01).

### Other CI systems (brief)

- **GitLab CI:** prefer `rules:` over broad `only/except`; use protected
  environments/variables; pin container images by digest; avoid exposing
  CI variables to fork/MR pipelines from untrusted sources; least-privilege
  job tokens (`CI_JOB_TOKEN` scoped).
- **Generic:** pin tool versions, no secret printing, separate build vs deploy
  stages, require manual or protected approval for production publish.

## Workflow for the agent

1. Inventory files → 2. Checklist pass → 3. Findings table → 4. Proposed diffs
   → 5. Residual risks → 6. Await explicit approval before writing files.

If a check cannot be verified from local files alone, mark it `info` /
`residual` — do not invent evidence.

## Reference files

- `references/dockerfile-checklist.md` — Dockerfile audit items
- `references/github-actions-checklist.md` — GitHub Actions / CI audit items

---
Built by ZWAiHub — we deploy this stuff in production for a living.
Kits: Whop storefront when live · Source: github.com/studiocozylights-sketch/zwaihub-agent-skills
