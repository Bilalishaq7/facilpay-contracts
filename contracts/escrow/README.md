# Escrow Contract

This contract manages secure, conditional fund holding for the Facil-Pay ecosystem, ensuring assets are only released when agreed-upon conditions are met by all parties.

## Public Functions

- create_escrow: Initializes a new escrow agreement with locked funds, terms, and designated participants.
- 
elease_escrow: Releases the held funds to the recipient once the agreed-upon conditions are successfully met.
- dispute_escrow: Flags the escrow transaction for administrative arbitration if participants cannot reach a consensus.
- clawback: Reverts the funds back to the original sender if the escrow conditions expire or fundamentally fail.
- pprove_multisig: Records an approval signature from a required participant for multi-signature escrow setups.
- dd_observer: Assigns a read-only role to a specific address for auditing and compliance tracking.
## Escalation timeout vs. appeal expiry

The escrow dispute flow has two separate timeout paths, and they apply in different dispute rounds.

- Escalation timeout applies while the escrow is in the Disputed state after a party escalates the dispute. `escalate_dispute` increments `escalation_level`, captures `escalated_at`, and adds a deadline at `now + escalation_timeout`. When that deadline is processed, `trigger_timeout_resolution` resolves the dispute under the configured `auto_resolve_in_favor_of` policy. This timeout is tied to the escalation event, not to a filed appeal.
- Appeal expiry applies only after the dispute has entered the Appeal round. An appeal can be filed only while the dispute round is not Final and the time since `dispute_started_at` is still within the 72-hour appeal window. The appeal stores `appeal_deadline = filed_at + 259200`, and if that deadline passes without a resolution, `expire_appeal` rejects the pending appeal, advances the dispute round to Final, and leaves the prior outcome as the effective final disposition.

These are distinct timers rather than one combined timeout. Escalation timeout is measured from the escalation timestamp on a disputed escrow, while appeal expiry is measured from the appeal filing deadline in the Appeal round. In practice, they are not both expected to fire for the same dispute state: the escalation path resolves the Disputed state before a valid appeal round is entered, and the appeal-expiry path only exists once an appeal has already been filed.
---
[⬅ Back to Main README](../../README.md)
