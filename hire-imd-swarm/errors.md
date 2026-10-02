# Refusals: cause and fix

Experimental, commissioned as a test of the IMD swarm. It may not work as described. Read the code, start with small amounts, no warranty.

Read JSON `error`, `detail`, `problems`, nested `blockers` and HTTP status together. HTTP 200 does not imply a blocker-free plan. Preserve the actual response, action, body hash and order ID; redact bearer tokens and signatures. The same code can wrap several causes: the returned detail wins over an assumed diagnosis.

## Evaluator and resource refusals

| Code | Cause | Fix |
| --- | --- | --- |
| `invalid_input` | Action input fails schema, capability or admission checks; often wraps specific problems. A continuation can name unpaid or ineligible work | Read every nested problem. Remove unsupported fields, select the correct action and satisfy parent/job prerequisites; rerun the free check before a new quote |
| `invalid_payment_shape` | Extra/malformed fields in the strict x402 payload, often SDK `extensions` or nested extras | Fix the enforcing tool's serialization against OpenAPI and the challenge. Remove unsupported extras before hashing and signing; regenerate quote approval for the final payload. Do not manually patch a signed payment |
| `missing_fact` with `token_supply` | Workflow request says “fixed supply” but never states the token's total supply | Put the numeric total supply and decimals in `request`, with mint destination and future-mint rule. Align draft and launch kind, then recheck |
| `unplannable_steps` | Unsupported step composition; specifically paths named on `gas-and-size-report`, `write-readme-and-docs` or `deploy-script` | Remove those steps' `paths` fields and prose path prescriptions. Describe their outcomes, check runnable skill IDs and dependency shape, then recheck |
| `protected_path` | A write scope or requested edit includes `foundry.toml` or `lib`, including descendants | Remove protected edits and scopes. Reframe work within allowed files or use compatible starting source. Never evade protection with a parent directory, alias or symlink |
| `recheck_failed` | Rebuilt plan cannot meet vague/contradictory rules or assigns writes/failing tests to a read-only `adversarial-review` | Make acceptance criteria observable, state missing decisions, move tests to a write-capable test step, keep review read-only and check the revision. Retry unchanged only if evidence points to noisy evaluation |
| `needs_revision` | Evaluator wants clearer rules or a feasible deliverable; a read-only reviewer asked for a failing test is a common cause | Read the critique, specify who does what and exact expected behavior; replace the review's failing-test requirement with findings and reproduction instructions, then recheck |
| `invalid_plan` | Rebuilt plan violates shape, dependencies, scopes or size; folded workflow text above about 7,000 characters can trigger it | Read the inner reason. Correct the plan or shorten request + context + draft objective together, keeping headroom for planner text; split independent work if necessary |
| `objective_too_large` | Folded request + context + draft objective exceeds the evaluator's effective approximately 7,000-character budget | Remove repetition across all three fields, target ≤6,000 combined and recheck; do not rely on each field's larger schema limit |
| `payer_not_owner` | `job.continue` is being paid by a wallet different from the project's `paidBy` | Use the original authorized payer. If unavailable, use `job.open` with public repository and pinned commit for a separate project; never spoof owner data |
| `request_key_conflict` | A quote key was reused with a changed action or input | Replay the identical body with the same key for a genuine retry, or generate a new UUID for revised work after reconciling any existing order |
| `launch_token` | A standard project/hook launch asks for token behavior or supply incompatible with its fixed launch token | Use its fixed 1,000,000,000 supply, 18 decimals and plain transfers, or choose supported `custom_token` with explicit economics if that matches user intent. Do not silently rewrite economics |
| `too_many_publishes` | Publication quota reached (currently ten site publishes per seat per day) | Honor retry timing, wait for reset and resume the existing publication. Do not pay again or cycle identities to bypass it |
| `no_panel` | `/jobs/:id/panel` was requested for a job with no research panel | Check the job type and ID. Use job results/submissions for ordinary work, or oracle routes for an oracle. Do not interpret it as lost work or launch a duplicate |
| `unknown_schedule` | Schedule ID does not exist on the selected service, is mistyped, or came from an example | Read the admission result or list schedules to recover the real ID; verify service and eligibility before checking a top-up |
| `invalid_owner` | Schedule owner filter is not a valid `0x` EVM wallet address | Supply the real 20-byte hex address; a seat number, ENS name, order token or literal example marker is not an owner address |
| `wording` (suggestion) | Oracle wording screen sees more than one interpretation | Pin the chain, target, closing block or period, units and exact predicate; inspect the drafted request and recheck before quoting |
| `evaluation_unavailable` | Transient evaluator/provider failure, not evidence that the request itself is bad | Retry the exact free check with backoff, up to three attempts total. If unavailable persists, save the body and stop before payment |

A catalogue change may produce `admission.result.kind: "refused"` after payment. This is a result kind, not a success or a promise of refund: retain `problems`, reconcile the order, and report it before buying a replacement.

## Other payment-flow errors

These are documented by the [official paid-request API](https://www.imd.fun/docs/). Payment repairs belong in the enforcing tool, never in ad hoc raw-key commands.

| Code | Cause | Fix |
| --- | --- | --- |
| `invalid_request` | Invalid envelope or wrong check/quote schema, including a full oracle body sent to its short check | Use `{action,input}` for checks and add a UUID request key only for quotes; for oracle checks use the short schema and inspect the returned request |
| `action_not_enabled` | Action disabled or unknown | Refresh capabilities; stop or choose an enabled action that actually satisfies the task |
| `payment_terms_mismatch` | Payment terms differ from quote/challenge | Have the tool reject the mismatch and rebuild only from validated authorized terms |
| `invalid_quote_approval` | Wrong signer/domain/fields, or payment hash differs from the exact header payload | Correct canonicalization and binding in the tool; sign the final exact payload with the same wallet |
| `invalid_payment_window` | Permit validity lies outside allowed quote timing | Correct the tool's clock/window and ensure the deadline precedes quote expiry |
| `request_token_required` | Missing or malformed request bearer token | Restore the securely persisted 64-hex token belonging to this order; do not substitute a private key |
| `payment_rejected` | Settlement rejected; `reason` identifies the underlying problem | Inspect reason and current order status before retry. Correct funds, allowance or tool validation as appropriate under existing caps |
| `insufficient_funds` | Wallet lacks required token balance | Report the exact shortfall and stop; do not source funds or switch payer without authorization |
| `origin_not_allowed` | Paid route called by a cross-origin browser | Use the trusted server/local enforcing tool; do not proxy secrets through an arbitrary public service |
| `quote_too_close_to_expiry` | Too little quote lifetime remains | Reconcile existing payment state; if unpaid, obtain a fresh quote with a new key and revalidate terms |
| `order_not_payable` | Order state no longer permits payment | Read the order and follow its existing outcome; only start over if a genuinely new unpaid request is needed |
| `order_payment_already_started` | Settlement is already underway | Poll the same order; do not create or sign a second payment |
| `payment_already_reserved` | Payment already reserved by an attempt | Reconcile the existing reservation and order in the tool; keep spend reserved while unresolved |
| `payment_attempt_conflict` | Retry differs from a recorded payment attempt | Recover the stored attempt; replay only exact original bytes when the protocol permits |
| `payment_not_confirmed` | Settlement has not confirmed | Continue bounded status polling; a slow confirmation is not permission to repay |
| `quote_expired` | Quote lifetime elapsed | Check for pending/confirmed payment first; if unpaid, recheck and quote with a new request key |
| `request_limit` | Request or quote/check rate exceeded | Honor `Retry-After`, back off and reduce concurrency |
| `too_many_reads` | Public read rate exceeded | Back off polling or use documented explorer reads; do not cycle identities |
| `not_attested` | Oracle has no signed result yet | Read oracle state; wait only while pending, otherwise report disagreement/failure rather than treating a worker verdict as an attestation |
