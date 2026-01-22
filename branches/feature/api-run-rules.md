# feature/api-run-rules

## Purpose

Add API methods to programmatically run transaction rules, allowing clients to find matching transactions, preview rule effects, and apply rules to specific transactions.

See [ARCHITECTURE.md](../../ARCHITECTURE.md) for package dependency diagram.

## New API Methods

| Method | Description |
|--------|-------------|
| `getTransactionsMatchingRule(ruleId)` | Find all transactions matching a rule's conditions |
| `previewRuleOnTransactions(ruleId, transactionIds)` | Preview what changes would occur (dry-run) |
| `applyRuleToTransactions(ruleId, transactionIds)` | Apply rule actions and save to database |

## Changes

### `packages/loot-core/src/types/api-handlers.ts`
Added type definitions for three new API handlers.

### `packages/api/methods.ts`
Added public method wrappers that call the new handlers.

### `packages/loot-core/src/server/api.ts`
Implemented handlers:
- `api/rule-get-matching-transactions` - Uses `conditionsToAQL()` to convert rule conditions to database queries
- `api/rule-preview` - Uses `Rule.exec()` to evaluate changes without persisting
- `api/rule-apply-to-transactions` - Uses `rule-apply-actions` handler to apply and save

### `packages/api/methods.test.ts`
Added 3 tests covering all new functionality.

## Usage Examples

```typescript
import * as api from '@actual-app/api';

// Find transactions that match a rule
const matches = await api.getTransactionsMatchingRule(ruleId);
console.log(`Found ${matches.length} matching transactions`);

// Preview what changes a rule would make
const preview = await api.previewRuleOnTransactions(ruleId, [tx1.id, tx2.id]);
for (const { transaction, changes } of preview) {
  console.log(`Transaction ${transaction.id} would change:`, changes);
}

// Apply rule to specific transactions
const result = await api.applyRuleToTransactions(ruleId, [tx1.id]);
console.log(`Updated ${result.updated} transactions`);
```

## Why

Previously, rules could only be run automatically during transaction import. This feature enables:
- Programmatic rule application for automation scripts
- Selective application of rules to specific transactions
- Preview mode for testing rule behavior before applying
- Building custom UIs for rule management

## Testing

```bash
cd ~/Projects/forks/actualbudget/actual
corepack yarn workspace @actual-app/api test
corepack yarn typecheck
```
