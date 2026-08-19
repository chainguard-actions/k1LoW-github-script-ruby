<!-- markdownlint-disable -->

# Hardening Report: k1LoW--github-script-ruby/v2.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **k1LoW--github-script-ruby/v2.6.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable version tags or branch names instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved.

Failing references include:
- actions/checkout@v6
- ruby/setup-ruby@v1
- k1LoW/octocov-action@v1
- aquasecurity/trivy-action@master
- k1LoW/github-script-ruby@v2
- Songmu/tagpr@v1
- haya14busa/action-update-semver@v1
- docker/setup-buildx-action@v4
- docker/login-action@v4
- docker/build-push-action@v7
- actions/upload-artifact@v7
- actions/download-artifact@v8

Locations:

- `.github/workflows/example-comment.yml:10`
- `.github/workflows/example-pp.yml:12`
- `.github/workflows/integration.yml:14`
- `.github/workflows/ruby-version.yml:13`
- `.github/workflows/tagpr.yml:16`
- `.github/workflows/test.yml:13`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and most jobs also lack job-level `permissions:` blocks. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often `write-all` for older repositories), granting excessive access. Only the `rerun-tests` job in tagpr.yml has a job-level `permissions:` block; all other jobs in all files are missing permissions.

Locations:

- `.github/workflows/example-comment.yml:1`
- `.github/workflows/example-pp.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/ruby-version.yml:1`
- `.github/workflows/tagpr.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a). GitHub Actions performs YAML template substitution before the shell ever sees the command, so any special characters in the value are interpreted by the shell.

1. test.yml line 17: `run: docker run --rm -v ${{ github.workspace }}:/workspace ...` — `github.workspace` is interpolated directly into a shell command.
2. test.yml line 30: `run: docker build ... -t ghcr.io/k1low/github-script-ruby-base:${{ github.sha }}` — `github.sha` is interpolated directly.
3. tagpr.yml line 81: `digest="${{ steps.build.outputs.digest }}"` inside a `run:` block — `steps.build.outputs.digest` is interpolated directly.
4. tagpr.yml line 116: `VERSION=${{ steps.latest_version.outputs.version }}` inside a `run:` block — `steps.latest_version.outputs.version` is interpolated directly.
5. integration.yml line 22: `if [[ "${{ steps.output-set.outputs.result }}" != "hello world" ]]` inside a `run:` block — `steps.output-set.outputs.result` is interpolated directly.

Fix: move values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:30`
- `.github/workflows/tagpr.yml:81`
- `.github/workflows/tagpr.yml:116`
- `.github/workflows/integration.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across all six workflow files:

1. **unpinned-uses**: Pinned all 12 action references to full 40-char SHA digests with original tags preserved as comments. SHAs were resolved via lookup_action_sha.

2. **permissions**: Added top-level `permissions: {}` to all 6 workflow files. Added job-level permissions blocks with minimal required permissions: `contents: read` for most jobs, `contents: write` + `pull-requests: write` for the tagpr job, `packages: write` for docker jobs, and `actions: write` for the rerun-tests job.

3. **script-injection**: Fixed all 5 instances by moving `${{ }}` expressions out of `run:` shell strings into `env:` blocks and referencing them as plain environment variables:
   - test.yml: `github.workspace` → `$WORKSPACE`, `github.sha` → `$GIT_SHA`
   - tagpr.yml: `steps.build.outputs.digest` → `$DIGEST`, `steps.latest_version.outputs.version` → `$VERSION`
   - integration.yml: `steps.output-set.outputs.result` → `$RESULT`

