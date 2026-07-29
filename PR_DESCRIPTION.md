# Fix Architecture Issues #348, #483, #484, #485

## Summary

This PR addresses four critical architecture issues in the payment platform contracts:
1. **#348**: Unbounded payment forward chain depth risking instruction-budget exhaustion
2. **#483**: Admin address synchronization issues between coordinator and child contracts
3. **#484**: No distinct pauser role for emergency operations
4. **#485**: Missing bulk-resume function to match bulk-pause

---

## Issue #348: Payment Forward Chain Max-Hop Limit

### Problem
Loop detection traverses unlimited depth on every validation call. If a chain of 100 hops is configured, validation touches 100 storage entries sequentially, risking instruction-budget exhaustion.

### Solution
- Added `ConfigKey::MaxForwardDepth` storage key (default: 5 hops)
- Added admin functions:
  - `set_max_forward_depth(admin, max_depth)` - Configure maximum chain depth
  - `get_max_forward_depth()` - Query current limit
- Updated `set_payment_forward()` to enforce depth limit at write time
- Chains exceeding `max_depth` are rejected with `ForwardLoop` error before storage

### Files Changed
- `contracts/payment/src/lib.rs`:
  - Added `MaxForwardDepth` to `ConfigKey` enum
  - Modified `set_payment_forward()` to read and enforce max depth
  - Added `set_max_forward_depth()` and `get_max_forward_depth()` functions

---

## Issue #483: Admin Address Synchronization

### Problem
Admin contract stores a single address, but payment/escrow use independent multisig admin lists. If the coordinator's admin isn't in a child's admin list, `emergency_pause_all` fails during security incidents.

### Solution
**Operational requirement**: The pauser address MUST be added to payment and escrow multisig admin lists during deployment. Child contracts already validate `config.admins.contains(&admin)`.

### Files Changed
None (operational/deployment requirement)

### Deployment Checklist
1. Initialize payment contract with pauser in multisig admin list
2. Initialize escrow contract with pauser in multisig admin list
3. Initialize refund contract with pauser as admin
4. Initialize admin contract with pauser parameter
5. Verify pauser can call `pause_contract` on all three children

---

## Issue #484: Separate Pauser Role

### Problem
No privilege separation - `emergency_pause_all` requires the same highest-privilege admin key used for contract configuration, exposing it to operational risk.

### Solution
- Added `DataKey::Pauser` to admin contract
- Updated `initialize()` to accept separate `pauser: Address` parameter
- Modified `emergency_pause_all()` and `emergency_unpause_all()` to authorize against `pauser`
- Enables privilege separation:
  - **Admin role**: High-privilege - manages contract wiring
  - **Pauser role**: Lower-privilege - can pause/unpause during incidents

### Files Changed
- `contracts/admin/src/lib.rs`:
  - Added `Pauser` variant to `DataKey` enum
  - Updated `initialize()` signature
  - Updated `emergency_pause_all()` authorization
  - Updated `emergency_unpause_all()` authorization
  - Updated test with separate admin and pauser

---

## Issue #485: Bulk-Resume Counterpart

### Problem
No `emergency_unpause_all` - operators must manually unpause payment, escrow, and refund individually after incidents, creating operational toil and risk of inconsistent state.

### Solution
- Added `emergency_unpause_all(pauser)` function to admin contract
- Mirrors `emergency_pause_all` by calling `unpause_contract()` on all three children
- Maintains state consistency across platform
- Authorization via pauser role

### Files Changed
- `contracts/admin/src/lib.rs`:
  - Added `emergency_unpause_all()` function
  - Updated test to verify unpause functionality

---

## Testing

Updated test in `contracts/admin/src/lib.rs`:
```rust
#[test]
fn test_initialize_and_pause_all() {
    let admin = Address::generate(&env);
    let pauser = Address::generate(&env);
    
    client.initialize(&admin, &pauser, &payment_contract, 
                     &escrow_contract, &refund_contract);
    
    client.emergency_pause_all(&pauser, &reason);
    client.emergency_unpause_all(&pauser);
}
```

All contracts compile successfully:
- ✅ `contracts/admin` - No errors
- ✅ `contracts/payment` - No errors
- ✅ `contracts/escrow` - No errors
- ✅ `contracts/refund` - No errors

---

## Breaking Changes

1. **Admin Contract**: `initialize()` signature changed
   - Old: `initialize(admin, payment, escrow, refund)`
   - New: `initialize(admin, pauser, payment, escrow, refund)`

2. **Admin Contract**: Authorization changed
   - `emergency_pause_all()` now checks `pauser` (not `admin`)
   - `emergency_unpause_all()` now checks `pauser` (not `admin`)

---

## Migration Guide

### Existing Deployments
1. **Issue #348**: Default max depth is 5. Call `set_max_forward_depth()` to customize
2. **Issues #483-485**: Requires admin contract redeployment with new signature

### New Deployments
Follow deployment checklist in Issue #483 section

---

Closes #348  
Closes #483  
Closes #484  
Closes #485
