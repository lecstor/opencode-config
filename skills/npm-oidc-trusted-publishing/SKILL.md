---
name: npm-oidc-trusted-publishing
description: Migrate an npm package from long-lived NPM_TOKEN credentials to OIDC trusted publishing on GitHub Actions, with npm provenance attestations. Use this skill whenever the user mentions migrating to trusted publishing, eliminating NPM_TOKEN, setting up OIDC for npm, adding provenance attestations, publishing from CI without a token, or converting an existing release workflow to use a trusted publisher — even if they describe it in their own words like "get rid of the npm token" or "publish from GitHub Actions securely". Also use it proactively when reviewing a release workflow that still uses `NODE_AUTH_TOKEN`/`NPM_TOKEN` and the user is auditing CI secrets.
---

# Migrating npm packages to OIDC trusted publishing

## What this achieves

- **No `NPM_TOKEN` anywhere.** The publish workflow exchanges a short-lived GitHub OIDC token for an npm publish token at the moment of `npm publish`.
- **Provenance attestation.** The package is stamped with a SLSA provenance statement signed via sigstore — npmjs.com shows "Built and signed on GitHub Actions" and links back to the workflow run.
- **Tighter blast radius.** npm account tokens authorize every package the account owns. Trusted publishers are scoped per-package.

## Prerequisites

1. **Package already published at least once on npmjs.com.** Trusted publishing can't bootstrap a new package name — one manual `npm publish` is needed first.
2. **Repo is on GitHub.** (This skill covers GitHub Actions.)
3. **User has admin rights** on the npm package and the GitHub repo.
4. **There is a versioning tool — or you're adding one now.** Check for `.changeset/`, `.release-please-manifest.json`, or similar. If none, the fastest path is adding changesets alongside the migration.
5. **The release flow is either tag-triggered or uses `changesets/action` on push to main.**

## Migration steps

### Step 1: Pick a pattern and add the publish workflow

The **critical** pieces regardless of pattern: `id-token: write` permission, Node 24+, and `NPM_CONFIG_PROVENANCE=true`.

- **Pattern A (tag-triggered)** — read `references/pattern-a.md`. Works with manual bumps, semantic-release, release-please, and changesets with a local release script. Best for solo maintainers or small teams.
- **Pattern B (changesets/action on push to main)** — read `references/pattern-b.md`. Fully automated: opens Version Packages PRs, publishes on merge. Best for multi-contributor repos.

**For solo maintainers using changesets:** Pattern A + a local release script is usually the right call. The Version Packages PR from Pattern B is overhead when you're the only person merging.

### Step 2: Bump action versions and CI matrix

Add Node 24 to the CI test matrix (keep 20/22 for compatibility testing). Bump action versions across all workflows:

- `actions/checkout@v4` -> `@v6`
- `actions/setup-node@v4` -> `@v6`
- `pnpm/action-setup@v4` -> `@v5`

### Step 3: Configure the trusted publisher on npmjs.com

This must happen **before** the first OIDC-based publish. Walk the user through it (can't be done from CLI):

1. Go to `https://www.npmjs.com/package/<package-name>` -> **Settings** -> **Trusted Publisher**.
2. Fill in:
   - **Organization or user:** the GitHub org/user (e.g. `lecstor`)
   - **Repository:** repo name only (e.g. `sync-cf-secrets`)
   - **Workflow filename:** **basename only** — `release.yml`, NOT `.github/workflows/release.yml`
   - **Environment name:** leave **blank** unless using GitHub deployment environments
3. Save.

### Step 4: Test publish

Publish a patch version to exercise the new path. Don't migrate and sit on it.

### Step 5: Verify

```bash
npm view <package> dist.integrity dist.signatures --json
curl -s "https://registry.npmjs.org/-/npm/v1/attestations/<package>@<version>" | head -50
```

Should show 2 attestations (npm publish + SLSA provenance v1) and a "Built and signed on GitHub Actions" badge on npmjs.com.

### Step 6: Delete the old NPM_TOKEN

This is the whole point — don't skip it.

1. **GitHub Actions secrets** — delete `NPM_TOKEN` (check org-level secrets too).
2. **npmjs.com** -> Settings -> Tokens -> **Delete** the classic token.
3. **Maintainer's laptop** — remove `//registry.npmjs.org/:_authToken=...` from `~/.npmrc`. Check shell dotfiles for `NPM_TOKEN=` exports.

npm account tokens are account-level, not package-level. A leaked token can hijack *every* package the account owns. Setting up a trusted publisher does NOT revoke existing tokens — they stay functional until deleted.

## Gotchas

Read `references/gotchas.md` for the full list. The most common:

- **npm >= 11.5.1 required.** Node 22 LTS is permanently stuck on npm 10.9.x. Use Node 24.
- **404 means unauthenticated**, not "package not found." npm hides auth failures behind 404 to protect private package names.
- **Workflow filename on npmjs.com must be basename only** — `release.yml`, not the full path.
- **`id-token: write`** must be in the workflow permissions or OIDC silently fails.

## Things NOT to do

- Don't add `NPM_TOKEN` "just in case" — it masks OIDC failures and is pure risk.
- Don't enable provenance without trusted publishing and call it done — the token is still there.
- Don't publish from the maintainer's laptop after migrating — it reintroduces the token.

## Checklist

- [ ] `.github/workflows/release.yml` exists with `id-token: write`
- [ ] Workflow uses `node-version: 24` (not 22)
- [ ] Workflow uses `actions/checkout@v6`+, `actions/setup-node@v6`+
- [ ] Publish step has `NPM_CONFIG_PROVENANCE: "true"`
- [ ] No `NODE_AUTH_TOKEN` / `NPM_TOKEN` in the workflow
- [ ] Trusted publisher configured on npmjs.com (basename-only workflow filename)
- [ ] Test publish succeeded with provenance badge
- [ ] `NPM_TOKEN` deleted from GitHub Actions secrets
- [ ] npm classic token revoked on npmjs.com
- [ ] `~/.npmrc` cleaned up on maintainer's laptop
- [ ] Recovery procedure documented in repo README

See the pattern-specific reference file for additional checklist items.
