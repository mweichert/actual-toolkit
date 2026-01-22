# Actual Budget Architecture

## Deployment Modes

Actual Budget can run in two modes:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DESKTOP MODE (Electron)                            │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      desktop-electron                                │   │
│   │                                                                      │   │
│   │   ┌─────────────────────────────────────────────────────────────┐   │   │
│   │   │              desktop-client (@actual-app/web)                │   │   │
│   │   │                      React UI                                │   │   │
│   │   └─────────────────────────┬───────────────────────────────────┘   │   │
│   │                             │                                        │   │
│   │                             ▼                                        │   │
│   │   ┌─────────────────────────────────────────────────────────────┐   │   │
│   │   │                    loot-core                                 │   │   │
│   │   │           (runs in Electron main process)                    │   │   │
│   │   │                                                              │   │   │
│   │   │    SQLite DB stored locally   ◄───►   Optional sync to      │   │   │
│   │   │                                        remote server         │   │   │
│   │   └─────────────────────────────────────────────────────────────┘   │   │
│   └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   Everything runs locally - no server required                              │
└─────────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                           WEB MODE (Browser + Server)                       │
│                                                                             │
│   ┌───────────────────────────────┐      ┌───────────────────────────────┐ │
│   │           BROWSER             │      │           SERVER              │ │
│   │                               │      │                               │ │
│   │  ┌─────────────────────────┐  │      │  ┌─────────────────────────┐  │ │
│   │  │     desktop-client      │  │      │  │      sync-server        │  │ │
│   │  │    (@actual-app/web)    │  │ HTTP │  │  (@actual-app/sync-     │  │ │
│   │  │       React UI          │◄─┼──────┼─▶│       server)           │  │ │
│   │  └────────────┬────────────┘  │      │  └───────────┬─────────────┘  │ │
│   │               │               │      │              │                │ │
│   │               ▼               │      │              ▼                │ │
│   │  ┌─────────────────────────┐  │      │  ┌─────────────────────────┐  │ │
│   │  │       loot-core         │  │      │  │       loot-core         │  │ │
│   │  │  (runs in Web Worker)   │  │ Sync │  │   (budget storage)      │  │ │
│   │  │                         │◄─┼──────┼─▶│                         │  │ │
│   │  │   IndexedDB (local)     │  │ CRDT │  │   SQLite (server)       │  │ │
│   │  └─────────────────────────┘  │      │  └─────────────────────────┘  │ │
│   │                               │      │                               │ │
│   └───────────────────────────────┘      └───────────────────────────────┘ │
│                                                                             │
│   sync-server serves the web UI and stores/syncs budget data                │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Package Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             PACKAGES                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  APPLICATIONS                                                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │ desktop-electron│  │   sync-server   │  │       api       │             │
│  │                 │  │ @actual-app/    │  │  @actual-app/   │             │
│  │ Electron shell  │  │  sync-server    │  │      api        │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
│           │                    │                    │                       │
│           │ embeds             │ serves             │ wraps                 │
│           ▼                    ▼                    │                       │
│  UI LAYER                                           │                       │
│  ┌──────────────────────────────────────────┐       │                       │
│  │          desktop-client                  │       │                       │
│  │          @actual-app/web                 │       │                       │
│  │                                          │       │                       │
│  │  ┌────────────────────────────────────┐  │       │                       │
│  │  │  component-library                 │  │       │                       │
│  │  │  @actual-app/components            │  │       │                       │
│  │  └────────────────────────────────────┘  │       │                       │
│  └─────────────────────┬────────────────────┘       │                       │
│                        │                            │                       │
│                        │ imports                    │                       │
│                        ▼                            ▼                       │
│  CORE LAYER                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                           loot-core                                   │  │
│  │                                                                       │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                │  │
│  │  │   server/    │  │   shared/    │  │    types/    │                │  │
│  │  │  - api.ts    │  │  - query.ts  │  │  - models    │                │  │
│  │  │  - rules/    │  │  - months.ts │  │  - handlers  │                │  │
│  │  │  - accounts/ │  │              │  │              │                │  │
│  │  └──────┬───────┘  └──────────────┘  └──────────────┘                │  │
│  │         │                                                             │  │
│  │         ▼                                                             │  │
│  │  ┌──────────────┐                                                     │  │
│  │  │     crdt     │  @actual-app/crdt - sync protocol                  │  │
│  │  └──────────────┘                                                     │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## How the API Package Fits In

```
┌────────────────┐
│ External       │                    ┌─────────────────────────────────────┐
│ Scripts /      │───▶ @actual-app/api │                                     │
│ Node.js Apps   │         │          │  loot-core                          │
└────────────────┘         │          │                                     │
                           │ send()   │  ┌─────────────────────────────┐    │
                           └──────────┼─▶│  server/api.ts (handlers)   │    │
                                      │  └─────────────────────────────┘    │
                                      │                                     │
                                      └─────────────────────────────────────┘

The api package is a thin RPC client - all logic lives in loot-core handlers.
```

## Package Descriptions

| Package | NPM Alias | Purpose |
|---------|-----------|---------|
| `loot-core` | - | Core business logic, database, calculations, API handlers |
| `desktop-client` | `@actual-app/web` | React UI (Vite bundled) |
| `desktop-electron` | - | Electron wrapper for desktop app |
| `sync-server` | `@actual-app/sync-server` | Express server - serves web UI & syncs data |
| `api` | `@actual-app/api` | Public Node.js API for external integrations |
| `component-library` | `@actual-app/components` | Shared UI components, icons |
| `crdt` | `@actual-app/crdt` | CRDT sync implementation |

## Key Insight

**`loot-core` is the central hub** - all runtime packages depend on it:

- `api` package is a thin RPC client that calls `loot-core` handlers via `send()`
- `desktop-client` imports business logic directly from `loot-core`
- `sync-server` uses `loot-core` for budget storage and sync
- Adding new API functionality requires modifying `loot-core` handlers
