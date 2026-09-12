# Dockerfile audit checklist

Use against every `Dockerfile*` and `.dockerignore` in scope. Mark each item
`pass` / `fail` / `n/a` / `unknown` (needs human input). Propose a concrete
diff for every `fail`.

## Base image and pinning

| ID | Check | Pass criteria |
|----|-------|---------------|
| D-BASE-01 | No `latest` (or unpinned floating tag) in production final stage | Tag is major/minor or digest; update process noted if not digest |
| D-BASE-02 | Prefer official / trusted base | Known official image or org-approved internal base |
| D-BASE-03 | Slim/distroless where fit | Final stage is minimal for the runtime (not full SDK unless required) |
| D-BASE-04 | Digest pin when reproducibility required | `image@sha256:…` or documented why tag is acceptable |

## Multi-stage and layers

| ID | Check | Pass criteria |
|----|-------|---------------|
| D-STAGE-01 | Multi-stage for compiled or heavy build tooling | Builder tools absent from final image |
| D-STAGE-02 | Only runtime artifacts copied to final stage | No source toolchains, `.git`, test caches in final |
| D-LAYER-01 | Package install + cleanup in same `RUN` | `apt-get` lists (or equiv) removed in same layer |
| D-LAYER-02 | Dependency copy before app source | Lockfiles/manifests copied first to maximize cache hits |
| D-LAYER-03 | Minimal number of mutating RUNs | No unnecessary layer churn; related commands combined where clear |

## Users and filesystem

| ID | Check | Pass criteria |
|----|-------|---------------|
| D-USER-01 | Non-root `USER` before CMD/ENTRYPOINT | Final stage drops privileges |
| D-USER-02 | Writable paths owned correctly | `COPY --chown=` or explicit `chown` for app dirs the process needs |
| D-USER-03 | No unnecessary `sudo` / setuid | Not installed or not required at runtime |

## Secrets and build args

| ID | Check | Pass criteria |
|----|-------|---------------|
| D-SEC-01 | No secrets in `ENV` / `ARG` | No passwords, tokens, keys, `.pem` contents |
| D-SEC-02 | No secrets via `COPY` of credential files | `.env`, key files excluded or mounted as build secrets |
| D-SEC-03 | Build-time creds use secret mounts when needed | `RUN --mount=type=secret` (BuildKit) documented; not layered |
| D-SEC-04 | Debug flags off in final image | No `NODE_ENV=development`, open debug ports, etc. by default |

## COPY / ADD / network in build

| ID | Check | Pass criteria |
|----|-------|---------------|
| D-COPY-01 | Prefer `COPY` over `ADD` | `ADD` only with documented reason |
| D-COPY-02 | No remote URL `ADD` | Dependencies fetched with package manager + verification |
| D-COPY-03 | `.dockerignore` present and useful | Excludes `.git`, secrets, local junk, heavy deps not needed in context |
| D-NET-01 | No unverified `curl \| bash` | Checksums/signatures or package manager used |

## Runtime posture

| ID | Check | Pass criteria |
|----|-------|---------------|
| D-RUN-01 | `HEALTHCHECK` considered | Present when appropriate; `n/a` if orchestrator-only health is intentional |
| D-RUN-02 | `EXPOSE` documents ports only | Not a security boundary; ports match what the app listens on |
| D-RUN-03 | Read-only root feasible | Noted as residual if app requires writes; suggest explicit volumes |
| D-RUN-04 | Signal / PID 1 behavior | Use a proper init or runtime that handles signals if needed (`tini`, distroless expectations documented) |

## Supply chain (local evidence only)

| ID | Check | Pass criteria |
|----|-------|---------------|
| D-SUP-01 | Lockfiles copied and used | `npm ci`, `pip install -r`, `go mod`, etc. — not floating `npm install` without lock |
| D-SUP-02 | OS packages pinned or constrained where practical | Version pins or clear org policy; note residual if rolling |
| D-SUP-03 | No committed private keys in build context | Confirmed via `.dockerignore` + absence in COPY paths |

## Report guidance

For each `fail`, include:

1. Finding ID (e.g. `D-USER-01`)
2. Severity (`critical` if secrets in image; `high` for root + public service; etc.)
3. File + line reference
4. Proposed diff

Out of scope without local evidence: registry CVE scans, signed image policy in the cluster, runtime admission controllers.
