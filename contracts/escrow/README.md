# Escrow Contract

This contract manages secure, conditional fund holding for the Facil-Pay ecosystem, ensuring assets are only released when agreed-upon conditions are met by all parties.

## Public Functions

- create_escrow: Initializes a new escrow agreement with locked funds, terms, and designated participants.
- elease_escrow: Releases the held funds to the recipient once the agreed-upon conditions are successfully met.
- dispute_escrow: Flags the escrow transaction for administrative arbitration if participants cannot reach a consensus.
- clawback: Reverts the funds back to the original sender if the escrow conditions expire or fundamentally fail.
- approve_multisig: Records an approval signature from a required participant for multi-signature escrow setups.
- add_observer: Assigns a read-only role to a specific address for auditing and compliance tracking.

## Codebase Architecture & File Structure

- **`src/lib.rs`**: Active, primary implementation of the Escrow contract used in builds, deployments, and testing.
- **`src/lib_stable.rs`**: Legacy / deprecated snapshot of an older escrow contract implementation. It is not referenced in `Cargo.toml`, nor is it maintained or included in active build targets. Follow-up removal of this file is proposed as it is dead code.

---
[⬅ Back to Main README](../../README.md)
