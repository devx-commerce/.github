# devx-commerce / .github

Organization-level configuration for the devx-commerce GitHub org. This repository is **public by design** - it contains no sensitive information. Credentials, tokens, and environment config must never be added here.

---

## What this repo does

GitHub treats this repository specially. Files placed here are automatically inherited across the organization:

| File | Effect |
|---|---|
| `SECURITY.md` | Shown on every repo's Security tab as the vulnerability reporting guide |
| `.github/ISSUE_TEMPLATE/` | Issue templates available in every repo that has no templates of its own |
| `.github/workflows/devx-security-scanners.yml` | Reusable security scanner called by opted-in repos |

---

## Security Scanner

### How it works

Repos opt in with a single 10-line caller workflow. The full scanner logic lives here - opted-in repos just point to it.

```
devx-commerce/.github
  └── .github/workflows/devx-security-scanners.yml   ← scanner logic (here)

devx-commerce/any-repo
  └── .github/workflows/security.yml                 ← 10-line caller
```

**Caller workflow** (copy this into any repo to enable scanning):

```yaml
name: Security Scan

on:
  pull_request:
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * 1'

jobs:
  security:
    uses: devx-commerce/.github/.github/workflows/devx-security-scanners.yml@main
    permissions:
      contents: write
      pull-requests: write
      checks: write
      issues: write
```

### What runs

| Scanner | Finds |
|---|---|
| **Trivy** | Dependency CVEs - CRITICAL and HIGH only |
| **TruffleHog** | Secrets and credentials in git history |
| **Semgrep** | Code-level issues - injections, insecure configs, hardcoded tokens |

### Where findings appear

- **PR comment** - collapsible summary from all three scanners, updates on every push
- **Inline diff annotations** - findings pinned directly on changed lines in the Files tab
- **Check run** - blocks merge if CRITICAL or HIGH findings exist
- **GitHub Issues** - one issue per finding, auto-closed when the finding is resolved
- **Step Summary** - full scan report in the Actions run UI
- **Artifacts** - raw reports stored for 90 days

### Self-healing repos

On every default-branch scan, the workflow syncs `SECURITY.md` and the issue template from this repo into the calling repo if they are missing or outdated. No manual setup needed in each repo.

### Updating the scanner

Any change pushed to `main` here takes effect on the next scan run across all opted-in repos. No re-deployment, no changes needed in individual repos.

---

## Trivy version

Trivy is pinned to `v0.69.3` with a verified checksum. Do not switch to the `aquasecurity/trivy-action` marketplace action - it pulls `latest` by default.

To upgrade: download the new release, run `sha256sum` on the archive, update `TRIVY_VERSION` and `EXPECTED` in the workflow.

---

## Security policy

See [SECURITY.md](./SECURITY.md) for how to report a vulnerability to the devx-commerce team.

---

## License

MIT License - see [LICENSE](./LICENSE) for details.
