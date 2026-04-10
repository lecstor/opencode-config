---
name: npm-oidc-trusted-publishing
description: Migrate an npm package from long-lived NPM_TOKEN credentials to OIDC trusted publishing on GitHub Actions, with npm provenance attestations. Use this skill whenever the user mentions migrating to trusted publishing, eliminating NPM_TOKEN, setting up OIDC for npm, adding provenance attestations, publishing from CI without a token, or converting an existing release workflow to use a trusted publisher — even if they describe it in their own words like "get rid of the npm token" or "publish from GitHub Actions securely". Also use it proactively when reviewing a release workflow that still uses `NODE_AUTH_TOKEN`/`NPM_TOKEN` and the user is auditing CI secrets.
---

# Migrating npm packages to OIDC trusted publishing

## What this achieves

- **No `NPM_TOKEN` anywhere.** Not on the maintainer's laptop, not in GitHub Actions secrets. The publish workflow exchanges a short-lived GitHub OIDC token for an npm publish token at the moment of `npm publish`.
- **Provenance attestation.** The published package is stamped with a SLSA provenance statement signed via sigstore, so the npmjs.com page shows "Built and signed on GitHub Actions" and links back to the workflow run.
- **Tighter blast radius.** An npm account token authorizes every package the account owns. Removing it means a leaked token can't be used to hijack any package. Trusted publishers are scoped per-package.

## When to use this skill

Trigger this workflow when the user wants to:

- Convert a repo that currently publishes with `NPM_TOKEN` (via `.npmrc` or `NODE_AUTH_TOKEN` in Actions) to OIDC.
- Add npm provenance attestations to an already-CI-published package.
- Delete an old npm automation/classic token but needs CI to keep working.
- Audit a release workflow that still references `secrets.NPM_TOKEN`.

If the user just wants a generic "publish from CI" setup for a brand-new package, this is still the right skill — trusted publishing is now the recommended default.

## Prerequisites to verify before starting

Read the repo and confirm these before touching anything:

1. **Package is already published at least once on npmjs.com.** Trusted publishing cannot bootstrap a brand-new package name — you must publish v0.0.1 (or whatever) the old way first, then configure the trusted publisher. If the package doesn't exist yet, tell the user they need to do one manual `npm publish` first.
2. **Repo is on GitHub.** Trusted publishing currently supports GitHub Actions and GitLab CI/CD. This skill covers GitHub.
3. **User has admin rights on the npm package** (required to configure a trusted publisher on npmjs.com) and on the GitHub repo (required to add workflows).
4. **There is a clear versioning tool.** Changesets, semantic-release, release-please, or plain manual bumps are all fine — the migration is agnostic. Identify which one is in use by reading `package.json` scripts and looking for `.changeset/`, `.release-please-manifest.json`, or similar.
5. **The current release flow is tag-triggered or can be made tag-triggered.** The workflow template below assumes `on: push: tags: ['v*']`. If the user publishes on every push to main, adapt but prefer tag-triggered for safety.

## The migration workflow

Work through these steps in order. Do not skip the verification steps — the failure modes are opaque (see Gotchas below).

### Step 1: Add the publish workflow to the repo

Create `.github/workflows/release.yml`. The **critical** pieces are `id-token: write`, Node 24+, and `NPM_CONFIG_PROVENANCE=true`. Everything else adapts to the repo.

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

      # Node 24, NOT 22. See Gotchas: npm trusted publishing requires
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
      #
      # For changesets: `changeset publish --no-git-tag` because the tag
      # already exists (the tag push is what triggered this workflow).
      # For plain npm: `npm publish --access public`.
      # For semantic-release: use its existing invocation.
      - name: Publish to npm
        run: pnpm exec changeset publish --no-git-tag
        env:
          NPM_CONFIG_PROVENANCE: "true"
```

**Important substitutions:**

- **Package manager.** Replace `pnpm` with `npm ci` / `yarn install --frozen-lockfile`. Drop `pnpm/action-setup` if not using pnpm.
- **The publish command** depends on the versioning tool:
  - Changesets: `changeset publish --no-git-tag` (because tag already exists from local `pnpm release`)
  - Plain: `npm publish --provenance --access public` (the `--provenance` flag is an alternative to the env var; either works)
  - semantic-release: keep existing command — semantic-release natively supports provenance
  - release-please: publish happens automatically when release-please creates the release
- **Build/test steps** should mirror the repo's CI.
- **Do NOT add `NODE_AUTH_TOKEN` or any `env: NPM_TOKEN`** anywhere in this workflow. If the user's old workflow had one, remove it.

### Step 2: Bump any other workflows that touch Node

If the repo has a CI workflow testing against Node 20/22, leave those but add Node 24 to the matrix so CI proves the package works on the version that actually publishes:

```yaml
strategy:
  matrix:
    node-version: [20, 22, 24]
```

Also bump action versions across all workflows while you're in there — the GitHub-hosted Node 20 runners for actions are deprecated:

- `actions/checkout@v4` → `@v6`
- `actions/setup-node@v4` → `@v6`
- `pnpm/action-setup@v4` → `@v5`

The v5/v6 bumps exist specifically to move these actions to Node 24 runtime. v4 still runs Node 20 and will start producing deprecation warnings.

### Step 3: Configure the trusted publisher on npmjs.com

This must happen **before** the first OIDC-based publish runs, or the publish will fail. Walk the user through it (you can't do this from the CLI):

1. Go to `https://www.npmjs.com/package/<package-name>` → **Settings** → **Trusted Publisher** section.
2. Click **Add Trusted Publisher**.
3. Fill in:
   - **Organization or user:** the GitHub org/user that owns the repo (e.g. `lecstor`)
   - **Repository:** the repo name only (e.g. `sync-cf-secrets`)
   - **Workflow filename:** the **basename only**, e.g. `release.yml` — NOT the full path `.github/workflows/release.yml`. This is a common mistake.
   - **Environment name:** leave **blank** unless the workflow uses a GitHub deployment environment. If you set one here, the workflow must reference it with `environment: <name>` at the job level.
4. Save.

If the workflow file is ever renamed, the trusted publisher config on npmjs.com must be updated to match, or publishes start failing with auth errors.

### Step 4: Release a patch version to exercise the new path

Don't migrate and then sit on it — publish something small to prove it works. A patch-level changeset (or equivalent) describing "migrate to OIDC trusted publishing with provenance" is enough.

Cut a release using whatever local flow the repo has (`pnpm release`, `git tag && git push`, etc.). The tag push should trigger the new workflow.

### Step 5: Verify the publish worked correctly

Once the workflow run is green:

```bash
# Confirms provenance signatures are attached to the published tarball
npm view <package> dist.integrity dist.signatures --json

# Shows the actual attestations (should be 2: npm publish attestation + SLSA provenance v1)
curl -s "https://registry.npmjs.org/-/npm/v1/attestations/<package>@<version>" | head -50
```

On the npmjs.com page, the version should now show a **"Built and signed on GitHub Actions"** badge with a link back to the workflow run.

### Step 6: Delete the old NPM_TOKEN

This is the step users most often forget, and it's the whole point of the migration.

1. **Remove from GitHub Actions secrets:**
   - `https://github.com/<org>/<repo>/settings/secrets/actions` → delete `NPM_TOKEN` (or whatever it's named).
   - Also check organization-level secrets if the repo inherits any.

2. **Revoke the token on npmjs.com:**
   - `https://www.npmjs.com/settings/<user>/tokens` → find the token → **Delete**.
   - If unsure which token CI was using, delete the oldest/most-permissive one first and watch for breakage.

3. **Remove from the maintainer's laptop:**
   - Check `~/.npmrc` for `//registry.npmjs.org/:_authToken=...` and remove the line. Verify with `npm whoami` — it should either fail or prompt for `npm login`.
   - Check shell dotfiles (`~/.zshrc`, `~/.bashrc`, `~/.zshenv`) for `NPM_TOKEN=` exports.
   - Check password manager / 1Password entries for stale copies.

**Why this matters, and why it's not optional:** npm account tokens are account-level, not package-level. A single leaked token can publish malicious versions of *every* package the account owns. Setting up a trusted publisher does NOT revoke existing classic tokens — they remain fully functional until explicitly deleted. If the user says "eh, I'll leave the token for now," explain that the migration is only half-done and the threat surface hasn't shrunk.

## Gotchas and failure modes

These are the non-obvious things that bit us during the reference migration. Check each one before assuming the setup is broken.

### The npm version floor: 11.5.1

**Symptom:** workflow succeeds at provenance signing (log shows `Signed provenance statement with source and build information from GitHub Actions` and `Provenance statement published to transparency log: https://search.sigstore.dev/...`) but then fails at the final PUT with:

```
npm error code E404
npm error 404 Not Found - PUT https://registry.npmjs.org/<package>
npm error 404 '<package>@<version>' is not in this registry.
```

**Cause:** The npm client version is < 11.5.1. Trusted publishing support was added in npm 11.5.1. Older clients (including 10.x) don't know about it — they send an unauthenticated PUT, and **npm deliberately returns 404 instead of 401 for unauthenticated writes** (to avoid leaking the existence of private packages). So "not in this registry" actually means "your request was not authenticated."

**The dead-end:** Node 22 LTS ships with npm 10.9.x and will *never* ship with npm 11.x. Node's LTS policy does not bump npm across major versions within an LTS line. Latest 22.22.x is still npm 10.9.7. This is permanent for the life of Node 22 LTS.

**The fix:** Use `node-version: 24` in `setup-node`. Node 24 ships with npm 11.11.0+.

Verify if unsure:

```bash
curl -s https://nodejs.org/dist/index.json | python3 -c "
import json, sys
for r in json.load(sys.stdin)[:20]:
    print(r['version'], 'npm', r.get('npm', 'n/a'))
"
```

Confusingly, provenance signing works on older npm clients because it uses a different code path (sigstore directly). So you'll see "provenance published to transparency log" and then the publish fails. Don't be misled — provenance ≠ trusted publishing.

### Workflow filename must be basename only

On npmjs.com's trusted publisher config, the **Workflow filename** field takes `release.yml`, not `.github/workflows/release.yml`. Putting in the full path will cause publishes to fail with auth errors that look identical to "token not found." If publishes are failing right after setup, check this field first.

### Environment name: usually leave blank

If the npmjs.com config has an environment name set but the workflow doesn't declare `environment: <name>` at the job level, the OIDC claim won't match and publish fails. Easiest: leave both blank unless the user has a specific reason to use GitHub deployment environments.

### `id-token: write` must be at the job or workflow level

```yaml
permissions:
  id-token: write
  contents: read
```

Without this, the `ACTIONS_ID_TOKEN_REQUEST_URL` environment variable isn't set inside the job, and `npm publish` can't request an OIDC token. Failure mode is another opaque auth error.

### Annotated tags vs lightweight tags

`git push --follow-tags` only pushes **annotated** tags. If the local release flow creates lightweight tags (`git tag v1.2.3` with no `-a`/`-m`), they won't be pushed and the workflow won't trigger. Check with `git cat-file -t <tag>` — annotated tags return `tag`, lightweight return `commit`. Changesets' `changeset tag` creates annotated tags; plain `git tag -a v1.2.3 -m "..."` does too.

### `--no-git-tag` with changesets in CI

If the local flow already creates and pushes the tag (which is what triggers CI), the CI publish must run `changeset publish --no-git-tag`. Without that flag, changesets tries to create the tag a second time and fails because it already exists.

Conversely, if the workflow is triggered by merging a release-PR rather than a tag push (release-please style), the publish step should NOT use `--no-git-tag` — let the versioning tool create the tag in CI.

### Deprecation warnings on old action versions

If the user pastes a warning like `Node.js 20 actions are deprecated...`, the fix is NOT adding `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: "true"` as an env var. That's a workaround. The proper fix is bumping the action versions:

- `actions/checkout@v4` → `@v6` (v5+ uses Node 24)
- `actions/setup-node@v4` → `@v6` (v5+ uses Node 24)
- `pnpm/action-setup@v4` → `@v5` (v5 sole change: "Updated the action to use Node.js 24")

## Recovery: when the workflow fails after the tag is pushed

This is the most stressful failure mode and worth rehearsing. The version commit is on main, the `vX.Y.Z` tag is pushed, but nothing was published to npm. If the repo uses changesets, the pending changeset files have already been consumed by the version bump, so `pnpm release` (or equivalent) can't be rerun — it'll fail the "no pending changesets" guard.

Recovery steps:

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

Document this in the repo's README or CONTRIBUTING.md so the next person doesn't have to reason it out under pressure.

## Things NOT to do

- **Don't add `NPM_TOKEN` "just in case."** If OIDC works, the token is unused and is pure risk. If OIDC fails, the token will mask the real failure and you won't notice it stopped working.
- **Don't enable provenance without trusted publishing** in a token-based workflow and call the migration done. You get signatures but the token is still there.
- **Don't use `--provenance` flag AND `NPM_CONFIG_PROVENANCE=true` both.** Either works; doubling up is harmless but clutters the workflow.
- **Don't hardcode the workflow filename** in multiple places. If it must be renamed, remember the trusted publisher config on npmjs.com needs updating too — add a comment in the workflow file itself pointing at this.
- **Don't publish from the maintainer's laptop "just this once"** after migrating. Every manual publish needs an account token, which reintroduces the risk the migration was meant to eliminate. If the workflow is broken, fix it and publish via the workflow.

## Summary checklist

Use this as a final sanity check before declaring the migration done:

- [ ] `.github/workflows/release.yml` exists with `id-token: write` permission
- [ ] Workflow uses `node-version: 24` (not 22)
- [ ] Workflow uses `actions/checkout@v6` or later, `actions/setup-node@v6` or later
- [ ] Publish step has `NPM_CONFIG_PROVENANCE: "true"` (or equivalent flag)
- [ ] No `NODE_AUTH_TOKEN` / `NPM_TOKEN` anywhere in the workflow
- [ ] Trusted publisher configured on npmjs.com with basename-only workflow filename
- [ ] Test publish succeeded and npmjs.com shows "Built and signed on GitHub Actions"
- [ ] `curl .../attestations/<pkg>@<ver>` returns 2 attestations
- [ ] `NPM_TOKEN` deleted from GitHub Actions secrets
- [ ] npm classic token revoked on npmjs.com
- [ ] `~/.npmrc` cleaned up on maintainer's laptop
- [ ] Recovery procedure documented in README/CLAUDE.md
