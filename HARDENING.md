<!-- markdownlint-disable -->

# Hardening Report: heyhusen--archlinux-package-action/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **heyhusen--archlinux-package-action/v2.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in .github/workflows/main.yml use mutable tags instead of pinned 40-character SHA commits, making the workflow vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references: `actions/checkout@v4` (lines 20, 55, 92, 116), `redhat-actions/podman-login@v1` (line 57), `docker/metadata-action@v5` (line 63), `redhat-actions/buildah-build@v2` (line 74), `redhat-actions/push-to-registry@v2` (line 82), `tj-actions/branch-names@v8` (line 120), `mindsers/changelog-reader-action@v2` (line 124), `ncipollo/release-action@v1` (line 130).

Locations:

- `.github/workflows/main.yml:20`
- `.github/workflows/main.yml:55`
- `.github/workflows/main.yml:57`
- `.github/workflows/main.yml:63`
- `.github/workflows/main.yml:74`
- `.github/workflows/main.yml:82`
- `.github/workflows/main.yml:92`
- `.github/workflows/main.yml:116`
- `.github/workflows/main.yml:120`
- `.github/workflows/main.yml:124`
- `.github/workflows/main.yml:130`

### permissions (severity: medium)

missing-permissions: .github/workflows/main.yml has no top-level `permissions:` key and none of its four jobs (pretest, publish, test, release) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/main.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands.

1. Line 95 — `run: wget https://github.com/${{ github.repository_owner }}/plenti-bin/...` — `${{ github.repository_owner }}` is interpolated directly into a shell command.

2. Line 98 — `REPOSITORY_OWNER=${{ github.repository_owner }}` inside a `run:` block — the expression is substituted before the shell sees the script.

3. Line 101 — `REPOSITORY_NAME=${{ github.event.repository.name }}` inside a `run:` block — `github.event.repository.name` is attacker-influenced and interpolated directly.

Sub-rule (b): After the expressions are assigned to shell variables, those variables are expanded unquoted:
- Line 99: `REPOSITORY_OWNER=$(echo $REPOSITORY_OWNER | ...)` — `$REPOSITORY_OWNER` is unquoted.
- Line 102: `REPOSITORY_NAME=$(echo $REPOSITORY_NAME | ...)` — `$REPOSITORY_NAME` is unquoted.

All of these allow shell metacharacter injection.

Locations:

- `.github/workflows/main.yml:95`
- `.github/workflows/main.yml:98`
- `.github/workflows/main.yml:99`
- `.github/workflows/main.yml:101`
- `.github/workflows/main.yml:102`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/main.yml:

1. **unpinned-uses**: Pinned all 8 unique action references to full 40-char SHAs with tag comments preserved: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 (v4), redhat-actions/podman-login@4934294ad0449894bcd1e9f191899d7292469603 (v1), docker/metadata-action@c299e40c65443455700f0fdfc63efafe5b349051 (v5), redhat-actions/buildah-build@7a95fa7ee0f02d552a32753e7414641a04307056 (v2), redhat-actions/push-to-registry@5ed88d269cf581ea9ef6dd6806d01562096bee9c (v2), tj-actions/branch-names@dde14ac574a8b9b1cedc59a1cf312788af43d8d8 (v8), mindsers/changelog-reader-action@1faaf50aa09d5793d9a100819973df801febfb31 (v2), ncipollo/release-action@339a81892b84b4eeb0f6e744e4574d79d0d9b8dd (v1).

2. **permissions**: Added top-level `permissions: {}` and job-level blocks: pretest/test get `contents: read`; publish gets `contents: read` + `packages: write`; release gets `contents: write`.

3. **script-injection**: Moved all `${{ github.repository_owner }}` and `${{ github.event.repository.name }}` expressions out of `run:` shell strings into `env:` blocks, and ensured all variable expansions in the shell scripts are properly double-quoted.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in entrypoint.sh:
1. Line 14: `gpg --keyserver $INPUT_PGPKEYSERVER --recv-keys $key` → quoted both variables as `"$INPUT_PGPKEYSERVER"` and `"$key"`.
2. Line 24: `cd $INPUT_PATH` → `cd "$INPUT_PATH"`.
3. Line 72: `makepkg $INPUT_FLAGS` → used `read -ra makepkg_flags <<< "$INPUT_FLAGS"` followed by `makepkg "${makepkg_flags[@]}"`. This safely splits the space-separated flags into an array (preserving correct argument boundaries for makepkg) while preventing shell metacharacter injection from attacker-controlled input.

