# Versioning strategy for `gh-custom-actions`

This repo uses a simple versioning approach for GitHub composite actions that balances stability with ease of use.

## Semantic versioning basics

Tags follow a semantic-style pattern:

- `vMAJOR.MINOR.PATCH`
  - **MAJOR** – incompatible / breaking changes (e.g., changing inputs, behavior, or removing fields in a way that breaks existing workflows).
  - **MINOR** – backwards-compatible feature additions or improvements (e.g., new optional inputs, extra flags, better logging).
  - **PATCH** – backwards-compatible bug fixes (e.g., fixing a typo, adjusting a default, improving reliability without changing the contract).

Examples:

- `v1.0.0` – first stable v1 release.
- `v1.1.0` – adds new capabilities but old usage still works.
- `v1.1.1` – fixes a bug; no behavior changes for existing configs.
- `v2.0.0` – breaking changes; workflows may need edits.

The goal: if you use `@v1.x.y`, you should be able to upgrade within the `1.*` line without breaking your workflows, unless you explicitly opt into `@v2`.

## Semantic-ish versions for actions

Each “real” release is tagged with a full version:

- `v1.0.0`
- `v1.0.1`
- `v1.1.0`
- `v2.0.0`
- etc.

These tags are **immutable** once pushed. Workflows can pin exactly to a version if they want zero surprise updates:

    uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1.0.0

## Floating major tags (`v1`, `v2`, …)

In addition to full versions, there are **floating major tags**, like:

- `v1`
- `v2`

A floating tag is just a Git tag that you intentionally **move forward over time** to always point at the latest release in that major line.

Example:

- `v1` always points to the latest `v1.x.y`.
- `v2` will eventually point to the latest `v2.x.y`, etc.

This lets consumers write:

    uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1

and automatically get:

- `v1.0.0`
- later `v1.0.1`
- later `v1.1.0`

without changing their workflows—**but** they will never jump to `v2` until they explicitly change the tag in their workflow.

## Creating the initial tags

When you’re ready to publish the first stable version of an action (or the whole repo):

1. Create the concrete version tag:

    git tag -a v1.0.0 -m "gh-custom-actions v1.0.0"
    git push origin v1.0.0

2. Create the floating major tag (`v1`) pointing at the same commit:

    git tag -a v1 -m "Floating major v1"
    git push origin v1

At this point:

- `v1.0.0` → fixed, immutable release.
- `v1` → floating major tag, currently pointing to `v1.0.0`.

Workflows can safely use `@v1` if they’re okay with minor/patch updates.

## Moving the floating tag when you release a new version

Let’s say you make changes and decide the new release should be `v1.1.0`.

1. Tag the new version:

    git tag -a v1.1.0 -m "gh-custom-actions v1.1.0"
    git push origin v1.1.0

2. Move `v1` to the new release (i.e., make it point at the latest `v1.x`):

    # Move local v1 tag to current HEAD (which should be the v1.1.0 commit)
    git tag -f v1

    # Force-push the updated tag
    git push origin v1 --force

Now:

- `v1.0.0` → old version (still usable if someone pins to it).
- `v1.1.0` → new latest version for the v1 line.
- `v1` → floating major, now pointing to `v1.1.0`.

Any workflow using:

    uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1

will automatically pick up `v1.1.0` the next time it runs.

## Recommended usage patterns

### Pin for stability-critical CI

If a workflow is extremely sensitive to change, pin to a full version:

    uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1.0.0

You can manually bump later when you’re ready.

### Use floating `v1` for most internal projects

For your own repos (like the Pittsburgh in Progress backend/frontend), it’s reasonable to just use:

- `@v1` for `gcloud-wif-auth`
- `@v1` for `deploy-cloud-run`

Example:

    uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1

This gives you bugfixes and small improvements without you needing to touch the workflow every time, as long as you don’t break the v1 contract.

## Summary

- **Concrete tags**: `v1.0.0`, `v1.1.0`, etc. Never move them.
- **Floating majors**: `v1`, `v2`, etc. You move them forward as you ship minor/patch releases.
- Consumers use:
  - `@v1.0.0` when they want **fixed behavior**.
  - `@v1` when they want **“latest v1”** without hopping to v2.

