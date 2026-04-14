# Pattern A: tag-triggered publish

Works with plain manual version bumps, semantic-release, release-please, **and changesets with a local release script**. The maintainer runs a local release command, which tags and pushes; the tag push triggers CI. Best for solo maintainers or small teams where the overhead of automated Version Packages PRs isn't worth it.

## Workflow template

```yaml
name: release

on:
  push:
    tags:
      - "v*"

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      # id-token: write is required for OIDC trusted publishing to npm.
      # This lets the publish step exchange a short-lived GitHub Actions
      # OIDC token for an npm publish token — no long-lived NPM_TOKEN
      # secret exists anywhere.
      id-token: write
      contents: read

    steps:
      - uses: actions/checkout@v6

      # Only if the repo uses pnpm:
      - uses: pnpm/action-setup@v5
        with:
          version: 10

      # Node 24, NOT 22. See gotchas.md: npm trusted publishing requires
      # npm >= 11.5.1, and Node 22 LTS is permanently stuck on npm 10.9.x.
      # Latest Node 24 ships with npm 11.11.0.
      - uses: actions/setup-node@v6
        with:
          node-version: 24
          registry-url: https://registry.npmjs.org
          cache: pnpm   # or "npm" or "yarn"

      - name: Install dependencies
        run: pnpm install --frozen-lockfile --ignore-scripts

      - name: Typecheck
        run: pnpm typecheck

      - name: Test
        run: pnpm test

      - name: Build
        run: pnpm build

      # NPM_CONFIG_PROVENANCE=true stamps the package with a provenance
      # attestation. This is SEPARATE from trusted publishing — it uses
      # sigstore, not the trusted-publisher token exchange. Both work
      # together; you want both.
      - name: Publish to npm
        run: npm publish --access public
        env:
          NPM_CONFIG_PROVENANCE: "true"
```

## Substitutions

- **Package manager.** Replace `pnpm` with `npm ci` / `yarn install --frozen-lockfile`. Drop `pnpm/action-setup` if not using pnpm.
- **The publish command** depends on the versioning tool:
  - Plain: `npm publish --access public` (omit `--access public` for scoped-private or already-public packages)
  - Changesets driven by local `pnpm release`: `pnpm exec changeset publish --no-git-tag` (tag already exists — see the `--no-git-tag` gotcha in `gotchas.md`)
  - semantic-release: keep existing command — semantic-release natively supports provenance
  - release-please: publish happens automatically when release-please creates the release
- **Build/test steps** should mirror the repo's CI.
- **Do NOT add `NODE_AUTH_TOKEN` or any `env: NPM_TOKEN`** anywhere in this workflow.

## Pattern A + changesets: local release script

When using changesets with Pattern A, add a `scripts/release.js` that automates the full local flow: guard -> test -> version -> commit -> confirm -> push + tag. This gives you changesets' structured CHANGELOG without the Version Packages PR overhead.

The script should:

1. **Guard:** clean tree, on main, not behind origin, pending changesets exist.
2. **Safety gate:** run tests and build before touching versions.
3. **Consume changesets:** `changeset version` to bump `package.json` + `CHANGELOG.md`.
4. **Commit:** stage only `package.json`, `CHANGELOG.md`, `.changeset` and commit as `Version X.Y.Z`.
5. **Prompt:** show the commit diff and ask for confirmation — this is the last abort window. Everything before is local-only and reversible with `git reset --hard HEAD^`.
6. **Push + tag:** push main, run `changeset tag` (creates annotated `vX.Y.Z` tag), push tags.

The tag push triggers the release workflow. The publish step in the workflow becomes `pnpm exec changeset publish --no-git-tag` (the tag already exists).

Add `"release": "node scripts/release.js"` to `package.json`. The contributor workflow becomes:

```bash
pnpm changeset       # describe the change (interactive: patch/minor/major)
# ... commit the changeset file with the PR ...
pnpm release         # when ready: guards, tests, versions, tags, publishes via CI
```

A reference implementation of `scripts/release.js` is in the [lecstor/sync-cf-secrets](https://github.com/lecstor/sync-cf-secrets) repo.

## Recovery: tag-triggered publish fails

The `vX.Y.Z` tag is pushed but the workflow failed before the publish succeeded. If the repo uses changesets, the pending changeset files have already been consumed by the version bump, so `pnpm release` (or equivalent) can't be rerun — it'll fail the "no pending changesets" guard.

```bash
# 1. Delete the tag locally and on the remote
git tag -d vX.Y.Z
git push origin :refs/tags/vX.Y.Z

# 2. Fix whatever broke (e.g. wrong Node version, missing id-token permission,
#    wrong workflow filename in trusted publisher config), commit on main
git commit -am "fix(ci): ..."
git push origin main

# 3. Re-tag HEAD (annotated!) and push, which re-triggers the release workflow.
#    Use the SAME version number — this is not a new version, just a retry
#    with a fixed tree.
git tag -a vX.Y.Z -m "Version X.Y.Z"
git push origin vX.Y.Z
```
