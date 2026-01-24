# feature/disable-upstream-workflows

## Purpose

Remove GitHub workflows that are specific to the upstream `actualbudget/actual` repository, broken without secrets, or too expensive for a fork.

## Changes

Removes 28 workflow files from `.github/workflows/`:

### Publishing Workflows (9)
- `publish-npm-packages.yml` - Publishes to upstream's NPM registry
- `publish-nightly-npm-packages.yml` - Nightly NPM publishing
- `publish-nightly-electron.yml` - Nightly Electron builds
- `publish-nightly-edge.yml` - Nightly edge Docker builds
- `docker-release.yml` - Docker Hub/GHCR publishing
- `docker-edge.yml` - Edge Docker builds
- `electron-master.yml` - Electron + MS Store + Flathub publishing
- `netlify-release.yml` - Upstream Netlify deployment
- `docs-release.yml` - Upstream docs deployment

### Release Automation (3)
- `generate-release-pr.yml` - Upstream release PR automation
- `ai-generated-release-notes.yml` - OpenAI-generated release notes
- `release-notes.yml` - Upstream release notes

### Issue Management (6)
- `issues-remove-help-wanted.yml` - Upstream issue automation
- `issues-feature-implemented.yml` - Upstream feature tracking
- `issues-close-feature-requests.yml` - Upstream feature voting
- `stale.yml` - Upstream stale issue management
- `count-points.yml` - Upstream contributor tracking
- `fork-pr-welcome.yml` - Welcomes PRs to upstream

### Upstream-specific (2)
- `i18n-string-extract-master.yml` - Upstream i18n workflow
- `docs-spelling.yml` - Upstream docs spelling checks

### Broken Without Secrets (4)
- `autofix.yml` - Requires autofix.ci service to be configured
- `size-compare.yml` - Compares against upstream master artifacts
- `vrt-update-apply.yml` - Requires `ACTIONS_UPDATE_TOKEN` secret
- `vrt-update-generate.yml` - Apply step won't work without secret

### Too Expensive (4)
- `codeql.yml` - Scheduled cron job uses Actions minutes
- `e2e-test.yml` - Very expensive (5+ parallel runners per PR)
- `e2e-vrt-comment.yml` - Depends on e2e-test workflow
- `electron-pr.yml` - Very expensive (3 runners per PR)

## Workflows Retained

Only 2 workflows remain for basic CI:

- `build.yml` - Builds API, CRDT, web, and server packages
- `check.yml` - Linting, type checking, and unit tests

## Why

These upstream workflows would either:
1. Fail due to missing secrets/permissions
2. Perform unwanted actions (publishing to upstream registries)
3. Consume excessive GitHub Actions minutes
4. Manage upstream issues/PRs inappropriately
