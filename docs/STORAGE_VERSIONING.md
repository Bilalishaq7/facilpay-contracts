# Storage Versioning Guide

This document explains how storage schema versions are tracked across FacilPay Soroban smart contracts on Stellar and provides a workflow guide for contributors when modifying stored data shapes.

---

## 📐 Overview

Smart contracts maintain persistent storage on the Stellar ledger. As FacilPay contracts evolve, data structures (such as structs, enums, or ledger key patterns) may change. 

To prevent data corruption, deserialization errors, or unexpected state when upgrading contract code on-chain, FacilPay contracts implement explicit **Storage Schema Versioning** and **Migration Protocols**.

---

## ⚙️ How Storage Versioning Works

### 1. Schema Version State
Each contract tracks its active schema version using a integer storage key (starting at `1`):

| Contract | Storage Key Type | Storage Key | Initial Version |
| :--- | :--- | :--- | :--- |
| **Payment** | `DataKey::Config` | `ConfigKey::SchemaVersion` | `1` |
| **Refund** | `SystemKey` | `SystemKey::SchemaVersion` | `1` |
| **Escrow** | `DataKey::Config` | `ConfigKey::SchemaVersion` | `1` |

### 2. Inspection Endpoint
All contracts expose a public view function to check the current storage version:

```rust
pub fn get_schema_version(env: Env) -> u32
```

When uninitialized or first deployed, this defaults to `INITIAL_SCHEMA_VERSION` (version `1`).

---

## 🔄 Migration Patterns

FacilPay employs two distinct migration strategies depending on contract complexity and stored entry volume:

### Pattern A: Direct Schema Upgrade (`Payment` & `Refund` Contracts)

For contracts with global state or configuration-level changes, an admin executes a direct schema migration:

```rust
pub fn migrate_schema(env: Env, admin: Address, target_version: u32) -> Result<(), Error>
```

- **Authorization**: Must be signed by an authorized admin.
- **Validation**: Ensures `target_version > current_version`. If current version is already at or above `target_version`, it returns `Error::SchemaAlreadyAtTarget`.
- **Action**: Updates the contract's schema version entry in instance storage to `target_version`.

### Pattern B: Batch / Multi-Step Migration (`Escrow` Contract)

For contracts managing large collections of individual records (e.g. escrows) that cannot be migrated within a single transaction's resource limits:

1. **`begin_migration(env, admin)`**:
   - Pauses new escrow creation to freeze state.
   - Initializes a `MigrationStatus` struct tracking `total_count`, `migrated_count`, and `started_at`.
2. **`migrate_escrow(env, admin, escrow_id)` / `migrate_escrow_batch(env, admin, escrow_ids)`**:
   - Iterates through target records, reading and rewriting them under the updated data layout (applying default values or struct transforms).
   - Flags each record with `EscrowMigrated(escrow_id) = true` to prevent double-migration.
   - Increments `migrated_count`.
3. **`complete_migration(env, admin)`**:
   - Verifies all items are migrated (`migrated_count >= total_count`).
   - Unpauses creation and advances `SchemaVersion` to `MIGRATION_TARGET_SCHEMA_VERSION`.

---

## 🛠️ Contributor Guide: Changing Stored Data Shapes

When introducing changes to any on-chain struct, enum variant, or storage key format, follow this step-by-step workflow:

### Step 1: Increment Schema Version Constants
Update or define the target schema version constant in the target contract's `lib.rs`:

```rust
const MIGRATION_TARGET_SCHEMA_VERSION: u32 = 2;
```

### Step 2: Implement Compatibility & Data Transformation
- If modifying an existing struct, ensure default values or fallback handling exist for missing fields when reading pre-migration data.
- Write transformation functions if data must be reshaped or converted into a new layout upon migration.

### Step 3: Implement or Update Migration Functions
- For direct migrations, extend `migrate_schema()` to execute necessary state transformations before updating `SchemaVersion`.
- For collection migrations, update the individual/batch migration handlers (e.g., `migrate_escrow`).

### Step 4: Write Unit Tests
Contributors **must** include comprehensive tests verifying storage versioning and migration behavior. Follow the existing test suites as reference examples:

- **Payment Contract**: [`contracts/payment/src/schema_version_test.rs`](../contracts/payment/src/schema_version_test.rs)
  - Tests schema initialization to `1`.
  - Tests rejection when migrating to an already reached version (`SchemaAlreadyAtTarget`).
- **Refund Contract**: [`contracts/refund/src/schema_version_test.rs`](../contracts/refund/src/schema_version_test.rs)
  - Tests version initialization and `migrate_schema` authorization/guards.
- **Escrow Contract**: [`contracts/escrow/src/migration_test.rs`](../contracts/escrow/src/migration_test.rs)
  - Tests full migration lifecycle (`begin_migration`, single/batch migration, `complete_migration`, paused state guards, double-migration guards).

### Checklist for PR Review
- [ ] Schema version constant incremented if storage layout changed.
- [ ] Migration function updated to handle data transformation.
- [ ] Unit tests added in `schema_version_test.rs` or `migration_test.rs`.
- [ ] Documentation updated if breaking changes affect upstream SDK/API callers.
