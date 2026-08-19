<!-- markdownlint-disable -->

# Hardening Report: heyhusen--archlinux-package-action/v2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **heyhusen--archlinux-package-action/v2.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All uses: references in .github/workflows/main.yml use mutable version tags instead of pinned 40-character SHA commits. This exposes the workflow to supply-chain attacks if any of these actions are compromised or their tags are moved. Affected references: actions/checkout@v4 (×4), redhat-actions/podman-login@v1, docker/metadata-action@v5, redhat-actions/buildah-build@v2, redhat-actions/push-to-registry@v2, tj-actions/branch-names@v8, mindsers/changelog-reader-action@v2, ncipollo/release-action@v1.

Locations:

- `.github/workflows/main.yml:18`
- `.github/workflows/main.yml:47`
- `.github/workflows/main.yml:55`
- `.github/workflows/main.yml:65`
- `.github/workflows/main.yml:76`
- `.github/workflows/main.yml:83`
- `.github/workflows/main.yml:91`
- `.github/workflows/main.yml:100`
- `.github/workflows/main.yml:118`
- `.github/workflows/main.yml:123`
- `.github/workflows/main.yml:130`

### missing-permissions (severity: medium)

The workflow file .github/workflows/main.yml has no top-level permissions: key and no job-level permissions: block on any of its four jobs (pretest, publish, test, release). Without explicit permissions, the GITHUB_TOKEN is granted its default (broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/main.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell commands. In the 'test' job, ${{ github.repository_owner }} is injected directly into a wget URL: `run: wget https://github.com/${{ github.repository_owner }}/plenti-bin/raw/...`. In the same job's next step, ${{ github.repository_owner }} and ${{ github.event.repository.name }} are assigned directly to shell variables without quoting: `REPOSITORY_OWNER=${{ github.repository_owner }}` and `REPOSITORY_NAME=${{ github.event.repository.name }}`. An attacker who can control these values (e.g. via a fork with a crafted repository name) could inject arbitrary shell commands.

Locations:

- `.github/workflows/main.yml:96`
- `.github/workflows/main.yml:103`
- `.github/workflows/main.yml:106`

### script-injection (severity: high)

Sub-rule (b): In entrypoint.sh, the workflow-controlled env var $INPUT_FLAGS is expanded unquoted in the shell command `makepkg $INPUT_FLAGS`. INPUT_FLAGS is populated from the inputs.flags action input (set by the calling workflow), making it untrusted data. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, etc.) from the value, enabling command injection. The safe form would be `makepkg "$INPUT_FLAGS"` or, since flags is a space-separated list, use an array.

Locations:

- `entrypoint.sh:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all four findings in .github/workflows/main.yml and entrypoint.sh:
1. unpinned-uses: Pinned all 8 unique action references (actions/checkout@v4 ×4, redhat-actions/podman-login@v1, docker/metadata-action@v5, redhat-actions/buildah-build@v2, redhat-actions/push-to-registry@v2, tj-actions/branch-names@v8, mindsers/changelog-reader-action@v2, ncipollo/release-action@v1) to their full 40-character SHA commits with original tags preserved as comments.
2. missing-permissions: Added top-level `permissions: {}` and job-level permissions (pretest: contents:read, publish: contents:read + packages:write, test: contents:read, release: contents:write).
3. script-injection (workflow): Moved ${{ github.repository_owner }} and ${{ github.event.repository.name }} into step env: blocks in the 'test' job's 'Get sample PKGBUILD' and 'Update action.yml' steps, referencing them as plain env vars in shell.
4. script-injection (entrypoint.sh): Replaced unquoted `makepkg $INPUT_FLAGS` with array-based expansion: `read -ra FLAGS_ARRAY <<< "$INPUT_FLAGS"` then `makepkg "${FLAGS_ARRAY[@]}"` to safely handle space-separated flags without shell metacharacter injection.

