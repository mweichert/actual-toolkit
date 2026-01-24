# feature/api-i18next-init

## Summary

Initializes i18next in the `@actual-app/api` package so that error messages from loot-core are properly returned.

## Problem

When loot-core's error functions (like `getSyncError`, `getUploadError`, etc.) call `t()` from i18next, the library returns empty strings if it hasn't been initialized. This causes API consumers like the MCP server to receive empty error messages.

## Solution

Add i18next initialization at the top of `packages/api/index.ts`, before any imports that might use `t()`. The configuration:

- Uses English (`lng: 'en'`)
- Disables namespace/key separators to allow natural-language keys
- No translation resources needed (English keys return themselves)

## Files Changed

- `packages/api/i18n.ts` - New file with i18next initialization
- `packages/api/index.ts` - Import i18n at the very top
- `packages/api/package.json` - Add i18next dependency

## Testing

1. Build the API package: `yarn workspace @actual-app/api build`
2. Run tests: `yarn workspace @actual-app/api test`
3. Use the API and trigger an error - verify the error message is not empty
