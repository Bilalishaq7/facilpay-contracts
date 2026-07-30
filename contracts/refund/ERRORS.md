# Refund Contract Error Codes

This document lists the numeric error codes defined in the refund contract, their symbolic names, and the condition that triggers each error. The refund contract uses two error enums in [contracts/refund/src/lib.rs](src/lib.rs) to stay within Soroban's variant limit.

## Core errors (`CoreError`)

| Error Code | Symbolic Name | Trigger Condition |
| :--- | :--- | :--- |
| 1 | `InvalidAmount` | The refund amount, fee amount, or related numeric value is invalid (for example zero or negative). |
| 2 | `RefundNotFound` | The requested refund ID does not exist in storage. |
| 3 | `Unauthorized` | The caller is not authorized to perform the requested action. |
| 4 | `InvalidPaymentId` | The supplied payment ID is invalid for the operation. |
| 7 | `InvalidStatus` | The refund is in a state that does not allow the requested action. |
| 8 | `AlreadyProcessed` | The refund has already been processed and cannot be processed again. |
| 9 | `RefundExceedsPayment` | The refund amount would exceed the original payment amount. |
| 10 | `TotalRefundsExceedPayment` | The cumulative refunds for the payment would exceed the original payment amount. |
| 11 | `RefundWindowExpired` | The refund request is outside the allowed refund window. |
| 12 | `RefundExceedsPolicy` | The refund violates the merchant or default refund policy limits. |
| 13 | `PolicyNotFound` | No refund policy exists for the requested merchant or context. |
| 14 | `PolicyInactive` | The refund policy exists but is inactive or disabled. |
| 15 | `QuorumNotReached` | An arbitration or review action does not have enough votes to meet quorum. |
| 16 | `NotArbitrator` | The caller is not a registered arbitrator for the operation. |
| 17 | `ContractPaused` | The contract is globally paused. |
| 18 | `FunctionPaused` | The specific function being invoked is paused. |
| 19 | `CaseNotTimedOut` | An arbitration timeout action was attempted before the case had actually timed out. |
| 20 | `BatchRefundTooLarge` | A batch refund request exceeds the configured batch size limit. |
| 21 | `CircularInheritance` | The refund policy inheritance chain contains a circular dependency. |
| 22 | `MaxInheritanceDepth` | The refund policy inheritance chain exceeds the maximum supported depth. |
| 23 | `RefundNotRejected` | The refund is not in the rejected state, so the requested rejection-related action is invalid. |
| 24 | `AppealWindowExpired` | The appeal window for the refund has already expired. |
| 25 | `AppealAlreadyFiled` | An appeal was already filed for the refund. |
| 26 | `RefundRateLimitExceeded` | The refund request exceeds the configured per-customer or global rate limit. |
| 27 | `PaymentContractNotSet` | The payment contract address has not been configured for cross-contract verification. |
| 28 | `PaymentOwnershipMismatch` | The payment referenced by the refund does not belong to the caller or expected owner. |
| 29 | `CircuitBreakerTripped` | The refund circuit breaker is currently open because the configured threshold was exceeded. |
| 30 | `InvalidFeeConfig` | The refund fee configuration is malformed or inconsistent. |
| 31 | `InsufficientTreasuryFees` | There are not enough accumulated treasury fees to satisfy the requested withdrawal. |
| 32 | `AutoApproveThresholdExceedsCeiling` | The merchant attempted to set an auto-approval threshold above the platform ceiling. |

## Extension errors (`ExtError`)

| Error Code | Symbolic Name | Trigger Condition |
| :--- | :--- | :--- |
| 34 | `ArbitratorNotFound` | The referenced arbitrator does not exist in the registry. |
| 35 | `InvalidScoreThreshold` | The arbitrator score threshold provided is invalid or out of bounds. |
| 36 | `AutoRefundTriggerNotFound` | The requested auto-refund trigger does not exist. |
| 37 | `DuplicateAutoRefundTrigger` | An auto-refund trigger with the same identity already exists. |
| 38 | `AddressFlaggedForFraud` | The address is flagged for fraud and cannot proceed with the refund action. |
| 40 | `FraudSignalNotFound` | The requested fraud signal record does not exist. |
| 41 | `HookNotFound` | The requested notification hook was not found. |
| 42 | `MaxHooksPerEventReached` | The maximum number of hooks for an event has already been reached. |
| 43 | `HookNotOwnedBySubscriber` | The hook exists but is not owned by the caller/subscriber. |
| 44 | `CustomerBlockedFromRefund` | The customer is blocked from receiving a refund under the current eligibility rules. |
| 45 | `EligibilityEntryNotFound` | The refund eligibility entry for the merchant/customer pair does not exist. |
| 46 | `TemplateNotFound` | The requested policy template does not exist. |
| 47 | `TemplateInactive` | The requested policy template exists but is inactive. |
| 48 | `RefundCountCapExceeded` | The refund count cap for the payment or merchant has been exceeded. |
| 49 | `RefundAmountCapExceeded` | The refund amount cap for the payment or merchant has been exceeded. |
| 50 | `UnsupportedRefundToken` | The token used for the refund is not supported by the contract. |
| 51 | `InvalidHookAddress` | The notification hook address is invalid or not accepted by the contract. |
| 52 | `VoucherNotFound` | The voucher referenced by the refund flow does not exist. |
| 53 | `VoucherExpired` | The voucher has expired and can no longer be used. |
| 54 | `VoucherAlreadyRedeemed` | The voucher has already been redeemed. |
| 55 | `EvidenceAlreadySubmitted` | The same evidence entry has already been submitted for the dispute. |
| 56 | `CaseAlreadyEscalated` | The arbitration case has already been escalated to a higher tier. |
| 57 | `TierPolicyNotFound` | The requested tier-based refund policy does not exist. |
| 58 | `SchemaAlreadyAtTarget` | The contract schema is already at the target version for migration. |
