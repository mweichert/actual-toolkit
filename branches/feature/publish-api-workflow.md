# feature/publish-api-workflow

## Purpose

Add a GitHub Action workflow that publishes the `@actual-app/api` package to GitHub Releases when fork tags are pushed.

## Changes

Adds `.github/workflows/publish-api.yml`:

- **Trigger**: Push of tags matching `*-fork*` (e.g., `v24.12.0-fork.1`, `0.0.1-fork.3`)
- **Action**: Builds the API package and attaches the `.tgz` artifact to the GitHub Release

## Workflow Steps

1. Checkout the repository
2. Set up the build environment (Node.js, Yarn, dependencies)
3. Build the `@actual-app/api` package
4. Pack it as `actual-app-api.tgz`
5. Create/update the GitHub Release with the tarball attached

## Why

This enables downstream projects to install the fork's API package directly from GitHub Releases:

```bash
npm install https://github.com/mweichert/actual/releases/download/0.0.1-fork.3/actual-app-api.tgz
```

This is necessary because the fork doesn't publish to NPM.
