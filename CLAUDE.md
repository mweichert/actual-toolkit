# Actual Budget Fork Workspace

This is a development workspace for the Actual Budget fork, with fork management infrastructure kept **outside** the git repository.

## Directory Structure

```
~/Projects/forks/actualbudget/
├── CLAUDE.md                 # This file
├── fork.yaml                 # Fork composition configuration
├── pyproject.toml            # UV project config
├── scripts/
│   └── build-fork.py         # Fork build script
├── branches/                 # Branch documentation
│   ├── feature/
│   └── bugfix/
└── actual/                   # The git repository (fork of actualbudget/actual)
```

## The Fork

- **Repository**: [mweichert/actual](https://github.com/mweichert/actual)
- **Upstream**: [actualbudget/actual](https://github.com/actualbudget/actual)
- **Description**: A local-first personal finance app

## Packages

Actual Budget is a Yarn 4 monorepo with 11 packages:

| Package | NPM Alias | Purpose |
|---------|-----------|---------|
| `loot-core` | - | Core business logic, database, calculations |
| `desktop-client` | `@actual-app/web` | React UI (Vite bundled) |
| `desktop-electron` | - | Electron wrapper for desktop app |
| `sync-server` | `@actual-app/sync-server` | Express sync server |
| `api` | `@actual-app/api` | Public Node.js API |
| `component-library` | `@actual-app/components` | Shared UI components, icons |
| `crdt` | `@actual-app/crdt` | CRDT sync implementation |
| `docs` | - | Documentation |
| `eslint-plugin-actual` | - | Custom ESLint rules |
| `plugins-service` | - | Plugin service |
| `ci-actions` | - | CI/CD tooling |

## Quick Reference

### Development (inside the repo)

```bash
cd ~/Projects/forks/actualbudget/actual
yarn install
yarn start              # Browser development
yarn start:desktop      # Desktop app
yarn start:server-dev   # With sync server
yarn typecheck          # Type check
yarn lint:fix           # Lint & format
yarn test               # Run all tests
```

### Rebuild the Fork

```bash
cd ~/Projects/forks/actualbudget
uv run scripts/build-fork.py           # Full rebuild
uv run scripts/build-fork.py --dry-run # Preview changes
```

This script:
1. Fetches upstream and resets `master`
2. Rebases each branch onto its base (topologically sorted)
3. Merges all branches into `fork`
4. Tags and pushes everything to origin

## Fork Management

### Branch Structure

| Branch | Purpose |
|--------|---------|
| `master` | Mirror of upstream `actualbudget/actual:master` |
| `fork` | Composed working branch (auto-built) |
| `feature/*` | Fork-only feature branches |
| `bugfix/*` | Fork-only bugfix branches |

### Remotes

- `origin` - mweichert/actual (this fork)
- `upstream` - actualbudget/actual (upstream repo)

### Adding a New Branch

1. Create from master:
   ```bash
   cd ~/Projects/forks/actualbudget/actual
   git checkout -b feature/my-feature master
   ```

2. Make changes, commit, push:
   ```bash
   git add .
   git commit -m "Add my feature"
   git push -u origin feature/my-feature
   ```

3. Create branch documentation:
   ```bash
   # Create ~/Projects/forks/actualbudget/branches/feature/my-feature.md
   ```

   **Important:** Branch docs are referenced when creating PRs, so they should be:
   - Self-contained (don't embed large diagrams - reference separate files)
   - PR-relevant (what changed, why, how to test)
   - Focused on the specific feature/fix

4. Add entry to `fork.yaml`:
   ```yaml
   branches:
     - name: feature/my-feature
       base: master
       description: Description of the feature
       docs: branches/feature/my-feature.md
   ```

5. Rebuild the fork:
   ```bash
   cd ~/Projects/forks/actualbudget
   uv run scripts/build-fork.py
   ```

### Removing a Branch

1. Remove entry from `fork.yaml`
2. Run `uv run scripts/build-fork.py`
3. Optionally delete the branch:
   ```bash
   cd ~/Projects/forks/actualbudget/actual
   git branch -d feature/old
   git push origin --delete feature/old
   ```

## Tagging Releases

The build script automatically tags the fork when changes are detected:
- Format: `{version}-fork.{n}` (e.g., `0.0.1-fork.1`)
- Version comes from `package.json`

## Key Points

- **Fork management files** (this directory) are outside git - they survive branch switches
- **actual/** is the actual repository - keep branches clean
- Never modify `fork` directly - it's rebuilt from component branches
- See `actual/CLAUDE.md` for repository-specific development guidelines
