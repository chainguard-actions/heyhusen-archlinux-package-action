<!-- markdownlint-disable -->

# Hardening Report: heyhusen--archlinux-package-action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **heyhusen--archlinux-package-action/v3.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in .github/workflows/main.yml use mutable version tags instead of full 40-character SHA commit digests, making the workflow vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: actions/checkout@v6, redhat-actions/podman-login@v1, docker/metadata-action@v6, redhat-actions/buildah-build@v2, redhat-actions/push-to-registry@v2, tj-actions/branch-names@v9, mindsers/changelog-reader-action@v2, ncipollo/release-action@v1.

Locations:

- `.github/workflows/main.yml:20`
- `.github/workflows/main.yml:52`
- `.github/workflows/main.yml:59`
- `.github/workflows/main.yml:70`
- `.github/workflows/main.yml:82`
- `.github/workflows/main.yml:91`
- `.github/workflows/main.yml:100`
- `.github/workflows/main.yml:110`
- `.github/workflows/main.yml:121`
- `.github/workflows/main.yml:127`
- `.github/workflows/main.yml:133`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings in the `test` job. (1) `run: wget https://github.com/${{ github.repository_owner }}/plenti-bin/...` — the repository owner is injected directly into a shell command. (2) `REPOSITORY_OWNER=${{ github.repository_owner }}` and `REPOSITORY_NAME=${{ github.event.repository.name }}` — github context values are interpolated directly into shell variable assignments inside a run: block. These values flow through YAML template substitution before the shell sees them, allowing an attacker to inject shell metacharacters.

Locations:

- `.github/workflows/main.yml:101`
- `.github/workflows/main.yml:105`
- `.github/workflows/main.yml:108`

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansions of user-controlled inputs in entrypoint.sh. (1) `gpg --keyserver $INPUT_PGPKEYSERVER --recv-keys $key` — both $INPUT_PGPKEYSERVER and $key (derived from $INPUT_PGPKEYS) are unquoted, allowing shell word-splitting and glob expansion on attacker-controlled values. (2) `cd $INPUT_PATH` — unquoted expansion of the user-supplied path input. (3) `makepkg $INPUT_FLAGS` — unquoted expansion of the user-supplied flags input, allowing injection of arbitrary makepkg arguments or shell metacharacters. These variables are set from action inputs (passed as Docker args) and must be double-quoted.

Locations:

- `entrypoint.sh:15`
- `entrypoint.sh:24`
- `entrypoint.sh:78`

### permissions (severity: medium)

Missing permissions: .github/workflows/main.yml has no top-level `permissions:` key, and the `pretest` and `test` jobs have no job-level `permissions:` key either. This means those jobs run with the default (broad) token permissions. Only the `publish` job (packages: write) and `release` job (contents: write) define explicit permissions. All jobs should declare minimal permissions.

Locations:

- `.github/workflows/main.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, permissions

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all 8 action references in .github/workflows/main.yml to full 40-char SHAs with tag comments preserved: actions/checkout@d23441a..., redhat-actions/podman-login@4934294..., docker/metadata-action@dc80280..., redhat-actions/buildah-build@7a95fa7..., redhat-actions/push-to-registry@5ed88d2..., tj-actions/branch-names@5250492..., mindsers/changelog-reader-action@1faaf50..., ncipollo/release-action@339a818...

2. **script-injection (workflow)**: Moved all ${{ github.repository_owner }} and ${{ github.event.repository.name }} expressions from run: shell strings into step env: blocks. The wget step and the 'Update action.yml' step now reference plain env vars ($REPOSITORY_OWNER, $REPOSITORY_NAME) in the shell.

3. **script-injection (entrypoint.sh)**: Quoted $INPUT_PGPKEYSERVER and $key in the gpg command; quoted $INPUT_PATH in the cd command; replaced bare `makepkg $INPUT_FLAGS` with a bash array approach (`read -ra makepkg_flags <<< "$INPUT_FLAGS"` then `makepkg "${makepkg_flags[@]}"`) to safely pass flags as separate arguments.

4. **permissions**: Added top-level `permissions: {}` to deny all by default, and added `permissions: contents: read` to the pretest and test jobs which had no permissions defined.

