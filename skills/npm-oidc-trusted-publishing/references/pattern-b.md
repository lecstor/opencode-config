# Pattern B: changesets/action on push to main

Fully automated changesets flow. `changesets/action@v1` opens a Version Packages PR automatically when changesets accumulate, and publishes when that PR is merged. Best for multi-contributor repos where the PR review step adds value.

## How it works

On push to main:

1. If there are pending changesets, the action opens (or updates) a "Version Packages" PR that bumps `package.json` and rewrites `CHANGELOG.md`.
2. When that PR is merged, the workflow runs again. With no pending changesets, the action runs the `publish` command, which calls `changeset publish` -> `npm publish` -> OIDC exchange.

You never think about tags. The action creates them.

## Workflow template

```yaml
name: release

on:
  push:
    branches:
      - main

concurrency:
  group: release-${{ github.ref }}

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      # contents: write      — changesets/action pushes version commits and tags
      # pull-requests: write — opens the "Version Packages" PR
      # id-token: write      — OIDC exchange for npm publish
      contents: write
      pull-requests: write
      id-token: write

    steps:
      - uses: actions/checkout@v6

      - uses: pnpm/action-setup@v5
        with:
          version: 10

      - uses: actions/setup-node@v6
        with:
          node-version: 24
          registry-url: https://registry.npmjs.org
          cache: pnpm

      - name: Install dependencies
        run: pnpm install --frozen-lockfile --ignore-scripts

      - name: Test
        run: pnpm test

      - name: Build
        run: pnpm build

      - name: Create release PR or publish to npm
        uses: changesets/action@v1
        with:
          publish: pnpm release   # package.json: "release": "changeset publish"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_CONFIG_PROVENANCE: "true"
```

## Notes

- **`release` script.** Add `"release": "changeset publish"` to `package.json`. The action calls `pnpm release` to publish. (Do NOT call it `"publish"` — that collides with `npm publish` lifecycle hooks and causes surprising recursion.)
- **`GITHUB_TOKEN` is not `NPM_TOKEN`.** `GITHUB_TOKEN` is the automatically-provided Actions token needed for opening PRs and pushing version commits. It does not authenticate to npm. OIDC is still what publishes.
- **No `--no-git-tag`.** That gotcha is only for Pattern A with local-tag changesets. Pattern B lets `changesets/action` create tags after publishing.
- **No NPM_TOKEN env var anywhere.**
- **Contributors add changesets locally** with `pnpm changeset` and commit the generated `.changeset/*.md` file.
- Test/build run on every push to main — that's fine. On the version-PR runs the work is slightly wasted; on the publish run it's necessary.

## Bootstrapping changesets from scratch

If the repo has no versioning tool at all:

```bash
pnpm add -D @changesets/cli
pnpm exec changeset init
```

Then:

- Edit `.changeset/config.json` and set `"access": "public"` (required for unscoped public packages, clearer regardless).
- Add scripts to `package.json`: `"changeset": "changeset"`, `"release": "changeset publish"`.
- **If the package has already been hand-published**, write a retroactive `CHANGELOG.md` entry documenting the shipped version(s) before changesets takes over. The format matches what `changeset version` produces, so future releases append cleanly:
  ```markdown
  # <package-name>

  ## <last-published-version>

  ### Minor Changes

  - Description of what shipped in that version.
  ```
  **Do NOT add changesets describing past work.** Changesets describe pending work, so they'd roll into the *next* release's changelog and misattribute the work to a version where it didn't happen.
- Create a first changeset (`pnpm changeset`) describing the OIDC + changesets migration so there's something concrete to ship on the first test publish.

## Recovery: publish fails after Version Packages PR merged

The Version Packages PR merged, the version commit is on main, but publish failed. You do NOT need to re-tag or revert — `changesets/action` doesn't create the tag until *after* a successful publish, so there's no tag to unwind.

1. Fix the workflow config on main (commit + push) — or if the failure was transient, skip this.
2. Either **re-run the failed workflow run** from the Actions tab, or push any commit to main to re-trigger.
3. With no pending changesets, the action will re-attempt `publish`. It publishes the same version (the one already in `package.json`) and tags it on success.

Do NOT revert the version-bump commit to "reset" the state — the rewritten `CHANGELOG.md` goes with it and the next changeset run will try to bump to the same version again, which will then collide with whatever you publish manually in a panic. Just fix forward.

## Extra checklist items for Pattern B

In addition to the common checklist in SKILL.md:

- [ ] `contents: write` and `pull-requests: write` are in the workflow `permissions`
- [ ] `package.json` has `"release": "changeset publish"` (not `"publish"`)
- [ ] `.changeset/config.json` has `"access": "public"` for a public package
- [ ] `GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}` is set on the `changesets/action` step
- [ ] If migrating an already-published package, a retroactive `CHANGELOG.md` entry was written before changesets took over
