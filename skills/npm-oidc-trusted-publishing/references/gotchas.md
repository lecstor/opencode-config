# Gotchas and failure modes

These are non-obvious things discovered during the reference migration. Check each one before assuming the setup is broken.

## The npm version floor: 11.5.1

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

Confusingly, provenance signing works on older npm clients because it uses a different code path (sigstore directly). So you'll see "provenance published to transparency log" and then the publish fails. Don't be misled — provenance is not the same as trusted publishing.

## Workflow filename must be basename only

On npmjs.com's trusted publisher config, the **Workflow filename** field takes `release.yml`, not `.github/workflows/release.yml`. Putting in the full path will cause publishes to fail with auth errors that look identical to "token not found." If publishes are failing right after setup, check this field first.

## Environment name: usually leave blank

If the npmjs.com config has an environment name set but the workflow doesn't declare `environment: <name>` at the job level, the OIDC claim won't match and publish fails. Easiest: leave both blank unless the user has a specific reason to use GitHub deployment environments.

## `id-token: write` must be at the job or workflow level

```yaml
permissions:
  id-token: write
  contents: read
```

Without this, the `ACTIONS_ID_TOKEN_REQUEST_URL` environment variable isn't set inside the job, and `npm publish` can't request an OIDC token. Failure mode is another opaque auth error.

## Annotated tags vs lightweight tags

`git push --follow-tags` only pushes **annotated** tags. If the local release flow creates lightweight tags (`git tag v1.2.3` with no `-a`/`-m`), they won't be pushed and the workflow won't trigger. Check with `git cat-file -t <tag>` — annotated tags return `tag`, lightweight return `commit`. Changesets' `changeset tag` creates annotated tags; plain `git tag -a v1.2.3 -m "..."` does too.

## `--no-git-tag` with changesets in CI (Pattern A only)

If the local flow already creates and pushes the tag (which is what triggers CI), the CI publish must run `changeset publish --no-git-tag`. Without that flag, changesets tries to create the tag a second time and fails because it already exists.

Conversely, if the workflow is triggered by merging a release-PR rather than a tag push (Pattern B / release-please style), the publish step should NOT use `--no-git-tag` — let the versioning tool create the tag in CI.

## Deprecation warnings on old action versions

If the user pastes a warning like `Node.js 20 actions are deprecated...`, the fix is NOT adding `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: "true"` as an env var. That's a workaround. The proper fix is bumping the action versions:

- `actions/checkout@v4` -> `@v6` (v5+ uses Node 24)
- `actions/setup-node@v4` -> `@v6` (v5+ uses Node 24)
- `pnpm/action-setup@v4` -> `@v5` (v5 sole change: "Updated the action to use Node.js 24")
