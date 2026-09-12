# GitHub Actions / CI audit checklist

Primary target: `.github/workflows/*`. Apply analogous judgment to GitLab CI
and other systems (see bottom). Mark `pass` / `fail` / `n/a` / `unknown`.
Propose a concrete diff for every `fail`.

## Permissions and trust boundaries

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-PERM-01 | Top-level `permissions` set | Explicit map; not relying on wide repo defaults |
| G-PERM-02 | Least privilege | Jobs only get the scopes they need (`contents: read` baseline) |
| G-PERM-03 | Write scoped to deploy jobs | `contents: write`, `packages: write`, `id-token: write` only where required |
| G-FORK-01 | Fork PRs cannot consume secrets for deploy/publish | No deploy on untrusted `pull_request` from forks |
| G-FORK-02 | `pull_request_target` used safely or avoided | If present: must not run untrusted code with secrets; document risk |
| G-ENV-01 | Production deploys use protected environments | Environment + required reviewers when org supports it |

## Action and tool pinning

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-PIN-01 | Third-party `uses:` pinned by full commit SHA | Not `@v1` / `@main` alone; tag comment allowed beside SHA |
| G-PIN-02 | Container images in jobs pinned thoughtfully | Tag or digest; avoid `image: something:latest` for security-sensitive jobs |
| G-PIN-03 | Language/tool versions pinned | `node-version`, `go-version`, setup-* inputs not floating carelessly |

## Secrets handling

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-SEC-01 | No secret echo / debug print | No `echo ${{ secrets.* }}`, no `set -x` around secrets |
| G-SEC-02 | Secrets not passed to untrusted steps | Untrusted PR code does not receive deploy credentials |
| G-SEC-03 | Prefer OIDC to static cloud keys | AWS/GCP/Azure via `id-token: write` + federation when possible |
| G-SEC-04 | Long-lived keys justified | If static keys remain, document rotation owner and residual risk |
| G-SEC-05 | Masking | Sensitive script output masked; artifacts do not contain secrets |

## Cache and artifacts

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-CACHE-01 | Cache keys scoped appropriately | Include OS, lockfile hash, relevant inputs |
| G-CACHE-02 | Cache poisoning considered | Privileged jobs do not blindly trust caches from untrusted PRs |
| G-ART-01 | Artifacts retain retention limits | Retention set; no secrets in uploaded artifacts |
| G-ART-02 | Publish jobs separated | Build on PR; publish/push image only on trusted refs |

## Fail-closed behavior

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-FAIL-01 | Shell steps fail closed | `bash -euo pipefail` or equivalent |
| G-FAIL-02 | Security gates not silently skipped | No unexplained `continue-on-error: true` on scan/sign/lint gates |
| G-FAIL-03 | Required checks documented | What must pass before merge/deploy is clear from workflow or residual notes |


## Expression / script injection

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-INJ-01 | Untrusted input not spliced into `run:` | `github.event.*`, `head_ref`, PR title/body, issue body, etc. go through `env:` and `"$VAR"` — never `${{ }}` inside shell text |
| G-INJ-02 | `github.sha` / trusted context in scripts | Prefer `env:` + quoted expansion even for trusted contexts; document if raw `${{ }}` remains in non-shell fields only |

## Workflow triggers

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-TRIG-01 | `on:` is intentional | No overly broad `workflow_dispatch` + secrets without protection |
| G-TRIG-02 | `workflow_run` / `repository_dispatch` reviewed | Cannot be abused to escalate from low-trust events |
| G-TRIG-03 | Cron jobs least privilege | Scheduled jobs do not carry unnecessary write tokens |

## Docker-related CI

| ID | Check | Pass criteria |
|----|-------|---------------|
| G-DOC-01 | Image tags immutable when publishing | Prefer git SHA tags; avoid movable `latest` as sole tag |
| G-DOC-02 | Registry auth minimal | Push credentials only on publish job; OIDC to cloud registries preferred |
| G-DOC-03 | Build context sane | Does not pack secrets; uses repo `.dockerignore` |

## GitLab CI and others (brief map)

| Theme | GitLab CI notes |
|-------|-----------------|
| Permissions | Restrict job token; protected branches/environments for prod vars |
| Pinning | Pin `image:` by digest; pin included templates where possible |
| Forks / MRs | Masked/protected variables not available to untrusted MRs |
| Secrets | No `echo` of variables; prefer short-lived OIDC/federated tokens |
| Fail closed | Do not `allow_failure: true` on security jobs without documenting |

Jenkins / Azure Pipelines / CircleCI: same principles — pin versions, least
privilege credentials, separate PR vs release, no secret logging, fail closed.

## Report guidance

For each `fail`, include finding ID, severity, file + job/step name, and a
proposed YAML diff. Example SHA comments in diffs must be labeled as
placeholders if not verified from local vendor copies — this skill does not
network to resolve current action SHAs.

Out of scope without evidence: org-level GHAS settings, SSO, branch protection
UI config, cloud IAM policy outside the workflow file.
