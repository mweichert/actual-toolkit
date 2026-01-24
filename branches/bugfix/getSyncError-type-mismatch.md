# Bugfix: getSyncError Type Mismatch

## Summary

Fixes a bug where `getSyncError()` receives an object instead of a string, causing all sync errors to display "unknown problem" instead of the actual error reason.

## Problem

The `sync-budget` handler returns errors as objects:
```typescript
{ error: { message: string; reason: string; meta: unknown } }
```

But `getSyncError()` expected a string:
```typescript
getSyncError(error: string, id: string)
```

This caused string comparisons like `error === 'out-of-sync-migrations'` to always fail, falling through to the generic "unknown problem" message.

## Solution

Modified `getSyncError()` to handle both string and object inputs:

```typescript
export function getSyncError(
  error: string | { reason?: string },
  id: string,
) {
  const reason = typeof error === 'string' ? error : error?.reason;
  // ... comparisons now use `reason` instead of `error`
}
```

## Files Changed

- `packages/loot-core/src/shared/errors.ts`

## Testing

- TypeScript check passes
- All loot-core tests pass
- Sync errors now display meaningful messages instead of "unknown problem"
