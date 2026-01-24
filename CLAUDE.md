# Actual Budget Toolkit

This is the development workspace for the Actual Budget fork and related tools.

**Repository**: [mweichert/actual-toolkit](https://github.com/mweichert/actual-toolkit)

## Directory Structure

```
~/Projects/forks/actualbudget/
├── CLAUDE.md                 # This file
├── ARCHITECTURE.md           # Package architecture diagrams
├── fork.yaml                 # Fork composition configuration
├── pyproject.toml            # UV project config (Python dependencies)
├── scripts/
│   └── build-fork.py         # Fork build script
├── branches/                 # Branch documentation
│   ├── feature/
│   └── bugfix/
├── actual/                   # Git submodule → mweichert/actual (the fork)
└── actual-mcp/               # Git submodule → mweichert/actual-mcp
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

- **actual/** and **actual-mcp/** are git submodules - work in them directly
- Never modify `fork` branch directly - it's rebuilt from component branches
- See `actual/CLAUDE.md` for Actual Budget development guidelines
- See `actual-mcp/CLAUDE.md` for MCP server development guidelines

## Important Notes

### The `fork` branch is ephemeral

**Never commit directly to the `fork` branch.** It gets completely rebuilt from scratch by `build-fork.py`:

1. The script deletes `fork` and recreates it from `master`
2. It merges each feature branch listed in `fork.yaml`
3. Any commits made directly to `fork` will be **permanently lost**

If you need to add something to the fork:
1. Create a feature branch from `master`
2. Make your changes there
3. Add the branch to `fork.yaml`
4. Run the build script

### Before rebuilding the fork

Before running `build-fork.py`, verify that all changes in the current `fork` branch are accounted for in feature branches:

```bash
cd ~/Projects/forks/actualbudget/actual
git log --oneline master..fork
```

Each commit should trace back to a branch in `fork.yaml`. If you see orphaned commits, they will be lost on rebuild.

## MCP Server (`actual-mcp/`)

An MCP server that exposes the Actual Budget API for LLM agents.

### Tools

| Tool | Description |
|------|-------------|
| `list_api_methods` | Discover available methods with documentation |
| `call_api_method` | Call any API method dynamically |

### Quick Start

Run directly from GitHub:
```bash
npx github:mweichert/actual-mcp
```

Or from local clone:
```bash
cd ~/Projects/forks/actualbudget/actual-mcp
npm install && npm run build
ACTUAL_SERVER_URL=http://localhost:5006 npm start
```

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ACTUAL_SERVER_URL` | Yes | URL of your Actual Budget server |
| `ACTUAL_PASSWORD` | No | Server password |
| `ACTUAL_DATA_DIR` | No | Local data directory (default: `/data`) |
| `ACTUAL_BUDGET_ID` | No | Budget ID to auto-load |

### Letta Core Integration

Add to MCP configuration:

```json
{
  "mcp_server_type": "stdio",
  "command": "node",
  "args": ["/path/to/actual-mcp/dist/index.js"],
  "env": {
    "ACTUAL_SERVER_URL": "http://actual-server:5006",
    "ACTUAL_PASSWORD": "your-password"
  }
}
```

See `actual-mcp/README.md` for full documentation.

## Deployment Workflow

### Deploying actual-mcp changes

When making changes to `actual-mcp/`:

1. **Make and commit changes** in actual-mcp submodule
2. **Tag release**: `git tag v0.x.0 && git push origin main --tags`
3. **GitHub Action** creates draft release automatically
4. **Update submodule** in toolkit:
   ```bash
   cd ~/Projects/forks/actualbudget
   git submodule update --remote actual-mcp
   git add actual-mcp
   git commit -m "Update actual-mcp submodule (description)"
   git push
   ```
5. **Clear npx cache** (for testing): `rm -rf ~/.npm/_npx/`

### If actual/ fork changes are needed

1. Create/update feature branch in `actual/`
2. Add to `fork.yaml` if new branch
3. Run `uv run scripts/build-fork.py`
4. Wait for GitHub Action to create release with `actual-app-api.tgz`
5. Update `actual-mcp/package.json` with new tarball URL
6. Continue with actual-mcp deployment steps above
