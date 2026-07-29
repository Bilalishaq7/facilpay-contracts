# Escrow Contract

Part of the [FacilPay smart contracts](../../README.md) suite on Stellar/Soroban.

## Multisig release threshold

Multi-party escrows can require a minimum approval weight before funds are released. Configure the release threshold as a basis-point value where `10000` means full approval and `5000` means 50% approval.

- Valid values are in the range `(0, 10000]`.
- The contract rejects `0` and any value outside this range with `InvalidThreshold` via the validation logic in [src/lib.rs](src/lib.rs).
- The minimum safe threshold is `1` bps, but in practice you should use a higher value for any escrow that holds meaningful funds. Setting the threshold too low can let a small group of signers release funds with weak consensus, which increases the risk of misuse or compromise.
- Recommended defaults are `10000` for high-value or sensitive escrows, and `5000` or higher for majority-based approval policies.

## Events

All events are emitted via `#[contractevent]` structs published through the Soroban event system. Each event's **topic** is a `Symbol` matching the struct name. Off-chain consumers (Horizon, indexers) can subscribe to these topics.

The sections below group events by domain. Within each section the table lists the event, its topic, payload fields, and the conditions under which it fires.

### Escrow Lifecycle

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `EscrowCreated` | `"EscrowCreated"` | `escrow_id: u64`, `customer: Address`, `merchant: Address`, `amount: i128`, `token: Address`, `release_timestamp: u64` | A new escrow is created via `create_escrow_with_multisig` or `create_conditional_escrow`. |
| `EscrowReleased` | `"EscrowReleased"` | `escrow_id: u64`, `recipient: Address`, `amount: i128`, `token: Address` | A locked escrow is released to the merchant (or an override recipient). |
| `EscrowDisputed` | `"EscrowDisputed"` | `escrow_id: u64`, `disputed_by: Address` | A party files a dispute on an escrow — status moves to `Disputed`. |
| `EscrowResolved` | `"EscrowResolved"` | `escrow_id: u64`, `released_to_merchant: bool`, `amount: i128` | An escrow is resolved via refund, manual resolution, or auto-resolution. |
| `EscrowExpired` | `"EscrowExpired"` | `escrow_id: u64`, `refunded_to: Address`, `amount: i128` | An escrow expires and funds are refunded to the customer. |
| `EscrowFeeCollected` | `"EscrowFeeCollected"` | `escrow_id: u64`, `fee_amount: i128`, `recipient: Address` | A fee is deducted from the escrow amount on release (`fee_amount > 0`). |
| `EscrowFeesWithdrawn` | `"EscrowFeesWithdrawn"` | `amount: i128`, `withdrawn_by: Address` | An admin withdraws accumulated escrow fees from the contract. |
| `EscrowFeeConfigUpdated` | `"EscrowFeeConfigUpdated"` | `fee_bps: i128` | The escrow fee basis-point configuration is updated. |

### Multi-Party Escrow

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `MultiPartyEscrowCreated` | `"MultiPartyEscrowCreated"` | `escrow_id: u64`, `participant_count: u32` | A multi-party escrow is created with multiple participants. |
| `ParticipantApproved` | `"ParticipantApproved"` | `escrow_id: u64`, `approver: Address` | A participant approves release of a multi-party escrow. |
| `MultiPartyEscrowReleased` | `"MultiPartyEscrowReleased"` | `escrow_id: u64` | All multi-party approval conditions are met and the escrow is fully released. |
| `WeightUpdated` | `"WeightUpdated"` | `escrow_id: u64`, `participant: Address`, `weight_bps: u32` | A participant's voting weight is updated by the customer. |
| `ThresholdUpdated` | `"ThresholdUpdated"` | `escrow_id: u64`, `threshold_bps: u32` | The multi-party approval threshold is updated. |

### Multi-Token Escrow

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `MultiTokenEscrowCreated` | `"MultiTokenEscrowCreated"` | `escrow_id: u64`, `customer: Address`, `merchant: Address`, `token_count: u32`, `release_timestamp: u64` | A multi-token escrow is created. |
| `MultiTokenEscrowReleased` | `"MultiTokenEscrowReleased"` | `escrow_id: u64`, `merchant: Address`, `token_count: u32` | A multi-token escrow is fully released. |

### Collateral

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `CollateralDeposited` | `"CollateralDeposited"` | `escrow_id: u64`, `party: Address`, `amount: i128` | A party deposits collateral when initiating a dispute. |
| `CollateralForfeited` | `"CollateralForfeited"` | `escrow_id: u64`, `party: Address`, `amount: i128` | Collateral is forfeited because the dispute resolution goes against the depositor. |
| `CollateralReturned` | `"CollateralReturned"` | `escrow_id: u64`, `party: Address`, `amount: i128` | Collateral is returned because the dispute resolution favors the depositor. |

### Disputes, Evidence & Appeals

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `EvidenceSubmitted` | `"EvidenceSubmitted"` | `escrow_id: u64`, `submitter: Address`, `ipfs_hash: String` | A party submits evidence (IPFS hash) for a disputed escrow. |
| `EvidenceDeadlineSet` | `"EvidenceDeadlineSet"` | `escrow_id: u64`, `deadline: u64`, `set_at: u64` | The 7-day evidence submission deadline is set when a dispute is first opened. |
| `EvidenceDeadlineExceeded` | `"EvidenceDeadlineExceeded"` | `escrow_id: u64`, `deadline: u64`, `submitted_at: u64` | A party attempts to submit evidence after the deadline — the submission is rejected. |
| `DisputeEscalated` | `"DisputeEscalated"` | `escrow_id: u64`, `level: u64` | A dispute is escalated to a higher review level. |
| `DisputeRecommendationGenerated` | `"DisputeRecommendationGenerated"` | `escrow_id: u64`, `outcome: DisputeOutcome`, `confidence_bps: u32` | An automated dispute recommendation is generated based on reputation scores. |
| `DisputeAppealFiled` | `"DisputeAppealFiled"` | `appeal_id: u64`, `escrow_id: u64`, `appellant: Address`, `filed_at: u64`, `appeal_deadline: u64` | A party files an appeal against a dispute resolution. |
| `AppealResolved` | `"AppealResolved"` | `appeal_id: u64`, `escrow_id: u64`, `in_favor_of: Address`, `resolved_at: u64` | An appeal is resolved. |
| `TimeoutResolutionTriggered` | `"TimeoutResolutionTriggered"` | `escrow_id: u64`, `favor: AutoResolveFavor`, `resolved_at: u64` | A dispute is automatically resolved due to timeout. |

### Multi-Party Dispute

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `MultiPartyDisputeRaised` | `"MultiPartyDisputeRaised"` | `escrow_id: u64`, `raised_by: Address` | A dispute is raised in a multi-party escrow. |
| `MultiPartyDisputeVoteCast` | `"MultiPartyDisputeVoteCast"` | `escrow_id: u64`, `voter: Address`, `favor_merchant: bool` | A participant casts a vote in a multi-party dispute. |
| `MultiPartyDisputeResolved` | `"MultiPartyDisputeResolved"` | `escrow_id: u64`, `favor_merchant: bool`, `resolved_at: u64` | A multi-party dispute is resolved when quorum is reached. |

### Vesting & Milestones

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `VestingScheduleCreated` | `"VestingScheduleCreated"` | `escrow_id: u64`, `total_amount: i128` | A vesting escrow with a release schedule is created. |
| `VestedAmountReleased` | `"VestedAmountReleased"` | `escrow_id: u64`, `amount: i128`, `released_at: u64` | A vested amount from a vesting escrow is released. |
| `MilestoneReleased` | `"MilestoneReleased"` | `escrow_id: u64`, `milestone_id: u64`, `amount: i128` | A specific milestone in the vesting schedule is released. |
| `MilestoneApproved` | `"MilestoneApproved"` | `escrow_id: u64`, `milestone_id: u64`, `approved_by: Address` | An admin approves a milestone in a vesting escrow. |

### Conditional Escrow

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `ConditionEvaluated` | `"ConditionEvaluated"` | `escrow_id: u64`, `met: bool` | The on-chain condition of a conditional escrow is checked. |
| `ConditionalReleaseExecuted` | `"ConditionalReleaseExecuted"` | `escrow_id: u64`, `released_to: Address` | The condition is met and the conditional escrow is released to the merchant. |

### Templates

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `TemplateCreated` | `"TemplateCreated"` | `template_id: u64`, `owner: Address` | A new escrow template is created. |
| `TemplateDeactivated` | `"TemplateDeactivated"` | `template_id: u64` | An escrow template is deactivated. |
| `EscrowCreatedFromTemplate` | `"EscrowCreatedFromTemplate"` | `escrow_id: u64`, `template_id: u64`, `customer: Address` | An escrow is created from a template. |

### Renewal

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `EscrowRenewed` | `"EscrowRenewed"` | `escrow_id: u64`, `renewed_by: Address`, `new_expiry_timestamp: u64`, `renewal_fee: i128`, `renewal_count: u32` | An escrow's expiry is extended (renewed). |
| `EscrowRenewalConfigUpdated` | `"EscrowRenewalConfigUpdated"` | `max_renewals: u32`, `renewal_fee_bps: u32`, `min_renewal_period: u64`, `max_renewal_period: u64`, `updated_by: Address` | The escrow renewal configuration is updated. |

### Beneficiary

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `BeneficiaryTransferred` | `"BeneficiaryTransferred"` | `escrow_id: u64`, `old_merchant: Address`, `new_merchant: Address` | An escrow's beneficiary (merchant) is transferred to a new address. |

### Watchdog

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `WatchdogReleaseTriggered` | `"WatchdogReleaseTriggered"` | `escrow_id: u64`, `released_to: Address`, `triggered_by: Address` | A watchdog triggers the release of a stale/overdue escrow. |

### Reputation & Tenure

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `ReputationUpdated` | `"ReputationUpdated"` | `address: Address`, `old_score: i64`, `new_score: i64` | A participant's reputation score changes after a dispute resolution. |
| `ReputationConfigUpdated` | `"ReputationConfigUpdated"` | `win_reward: i64`, `loss_penalty: i64`, `completion_reward: i64`, `dispute_initiation_penalty: i64` | The reputation scoring configuration is updated. |
| `TenureConfigUpdated` | `"TenureConfigUpdated"` | `base_score: u32`, `weight_per_day: u32`, `max_bonus_days: u32` | The tenure reputation bonus configuration is updated. |
| `TenureBonusGranted` | `"TenureBonusGranted"` | `escrow_id: u64`, `participant: Address`, `bonus: u32` | A tenure bonus is granted to a participant after a dispute-free completion. |
| `ReputationDecayed` | `"ReputationDecayed"` | `address: Address`, `old_score: i128`, `new_score: i128`, `days_inactive: u64` | A participant's reputation decays due to inactivity. |
| `AnalyticsReset` | `"AnalyticsReset"` | `reset_by: Address`, `reset_at: u64` | Contract analytics counters are reset by an admin. |

### Admin Governance (Multi-Sig)

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `ActionProposed` | `"ActionProposed"` | `proposal_id: String`, `proposer: Address`, `action_type: ActionType` | An admin proposes a new multi-sig governance action. |
| `ActionApproved` | `"ActionApproved"` | `proposal_id: String`, `approver: Address`, `approval_count: u32` | An admin approves a pending governance proposal. |
| `ActionExecuted` | `"ActionExecuted"` | `proposal_id: String` | A governance proposal reaches the required approvals and is executed. |
| `ActionRejected` | `"ActionRejected"` | `proposal_id: String`, `rejected_by: Address` | An admin rejects a pending governance proposal. |
| `AdminAdded` | `"AdminAdded"` | `admin: Address` | An admin is added to the multi-sig (on initialization, via add_admin, or succession activation). |
| `AdminRemoved` | `"AdminRemoved"` | `admin: Address` | An admin is removed from the multi-sig. |

### Succession

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `SuccessorDesignated` | `"SuccessorDesignated"` | `successor: Address`, `activatable_after: u64` | A new admin successor is designated. |
| `SuccessionActivated` | `"SuccessionActivated"` | `new_admin: Address`, `activated_at: u64` | The designated successor activates their succession. |
| `SuccessionRevoked` | `"SuccessionRevoked"` | `revoked_by: Address` | An admin revokes a succession plan. |

### TimeLock

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `TimeLockActionQueued` | `"TimeLockActionQueued"` | `action_id: u64`, `escrow_id: u64`, `executable_after: u64` | A time-locked action is queued for an escrow. |
| `TimeLockActionExecuted` | `"TimeLockActionExecuted"` | `action_id: u64`, `escrow_id: u64`, `executed_at: u64` | A time-locked action is executed after its delay. |
| `TimeLockActionCancelled` | `"TimeLockActionCancelled"` | `action_id: u64`, `cancelled_by: Address` | A queued time-locked action is cancelled by an admin. |
| `TimeLockConfigUpdated` | `"TimeLockConfigUpdated"` | `delay: u64`, `grace_period: u64` | The time-lock configuration is updated. |

### Pause / Unpause

| Event | Topic | Payload Fields | When Fired |
|-------|-------|----------------|------------|
| `ContractPausedEvent` | `"ContractPausedEvent"` | `paused_by: Address`, `reason: String`, `paused_at: u64` | The entire contract is paused. |
| `ContractUnpausedEvent` | `"ContractUnpausedEvent"` | `unpaused_by: Address`, `unpaused_at: u64` | The entire contract is unpaused. |
| `FunctionPausedEvent` | `"FunctionPausedEvent"` | `function_name: String`, `paused_by: Address`, `reason: String` | A specific contract function is paused. |
| `FunctionUnpausedEvent` | `"FunctionUnpausedEvent"` | `function_name: String`, `unpaused_by: Address` | A specific contract function is unpaused. |
