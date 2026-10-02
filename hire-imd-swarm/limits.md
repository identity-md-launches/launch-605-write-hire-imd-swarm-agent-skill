# Limits and payment contract

Experimental, commissioned as a test of the IMD swarm. It may not work as described. Read the code, start with small amounts, no warranty.

The evaluator constraints below come from the assignment. API figures are a 2026-10-02 snapshot of [official IMD docs](https://www.imd.fun/docs/) and [capabilities](https://api.imd.fun/requests/capabilities); refresh them before purchase. No dependency or network access is needed to read/install this skill. Actual checks and purchases need the live service.

## Drafting budgets

| Item | Limit or rule |
| --- | --- |
| HTTP quote body | 16 KiB, including envelope; count UTF-8 bytes |
| Workflow folded objective | Request + context + draft objective must stay below approximately 7,000 characters; target ≤6,000. Planner additions also need room |
| Workflow request / context | Documented maxima 16,000 characters each; these do not override folded-size or byte limits |
| Job objective | 1–8,000 characters; research template at most 4,000 |
| Steps | 1–6, one runnable skill each; `skill`, `template`, `steps` are alternative selectors |
| Step objective / criteria | 1–3,000 characters; 1–8 criteria of 1–500 characters each |
| Shape | `chain`, `fan_out_join`, `dag`; workflow allows `chain` or joined `dag` |
| DAG | Unique keys, explicit dependencies (empty for roots), no cycle, all branches join |
| Paths | Up to 16 repository-relative paths when the skill allows them; omit for gas/docs/deploy steps |
| Protected writes | `foundry.toml`, `lib` and descendants; no broad scope to bypass protection |
| Reference skills | Up to 8 per job and 8 per step; never executable steps |
| Named inputs / outputs | Up to 32 each; inputs copied from accepted result metadata; outputs specify name, artifact path and media type |
| Starting source | Public repository URL plus exact 40-character lowercase commit, both or neither |
| Continuation | Current paid project head, same payer, no concurrent project job or unfinished workflow; no source/deployment overrides |

For gas/docs/deploy steps, omit the write-path list even if the generic documentation says it is required. Do not embed path instructions in prose as a workaround. A named artifact output, when appropriate, uses the separate `outputs` schema; the evaluator still decides whether it is supported. Never give a read-only review a failing-test deliverable.

Whitespace folding is a planning operation, not permission to compress huge requirements into a single line. Count the three workflow fields together, including separators. Shorten duplication and split independent deliverables; do not silently delete acceptance criteria.

## Oracle and schedule details

Oracle **check input** accepts `question`, `panelSize`, optionally `answerType`, `evidence`, `chainId`, `toleranceBps`, `head`. Do not send a full oracle quote body to this short-input endpoint. Include precise source, time period, units and missing-evidence behavior in the question, under 2,000 characters. Review the full `request` returned by the check. A full quote input requires `v: 1`, question, chain, window, answer type, panel size, quorum and validity; additional definitions, guards and consumer domain may be needed.

Oracle snapshot: 5–100 panel members, quorum 2 through panel size, relative windows 1–720 hours, validity 60–2,592,000 seconds. Supported answers are bool, address, bytes32, uint256, address[] and bytes32[]. Use chain evidence for a reproducible chain recipe; panel evidence for offchain sources. Do not promise an attestation if the panel cannot agree. Do not deploy a consumer against a made-up verifying contract address.

Schedules buy 1–1,000,000 runs. Minimum interval: 10 minutes for oracle questions, 30 minutes for jobs. Use an ISO 8601 interval or five-field cron with an explicit timezone. Only `job.open` and `oracle.request` are schedulable. A schedule's input is frozen; job inputs cannot deploy or carry parent/project IDs. For evolving scheduled work use `continue: true`. Relative oracle windows resolve for each run.

Only an opened run spends a balance unit; failed and overlapping skipped runs do not. Missed slots do not accumulate a backlog: the scheduler fires once after an outage. Paid schedules have no expiry, unused runs are not refunded, and owner status does not grant pause/cancel controls (those remain with the IMD team). A top-up can reactivate an exhausted or failure-paused schedule, so inspect the frozen input first. Cancelled or expired schedules cannot be topped up. A schedule does not imply a contract calls itself: the IMD scheduler opens work, contributors perform it, and any onchain transition still needs an explicit transaction caller and gas.

## Payment tool contract

This skill does not supply or endorse a particular payment tool. Inspect the configured tool's implementation and dry-run output before using it. If these capabilities are absent, do not pay:

1. Dry-run is the default and cannot sign, broadcast, grant allowance or submit a payment. Live mode requires an explicit authorized transition.
2. Enforce per-order and cumulative spend caps in integer atomic units. Persist the spend ledger across restarts; reserve pending spend before signing and prevent concurrent orders from exceeding the total. For schedules cap `unitAmount × runs`, not just unit price. Include allowance/gas transactions in their own explicit caps.
3. Bind authorized origin, action, input, network, asset, recipient, amount and expiry to the quote and challenge; reject changes, unknown fields and expired terms. Distinguish Ethereum payment chain from the deployment chain. Do not trust an arbitrary `accepts[0]` without checking it against the authorized terms.
4. Use an external secure signer. Keep wallet secrets outside prompts, chat, generated files and logs. Persist request bearer tokens securely; they are separate random order-access secrets, not wallet keys, but should still not be printed.
5. Verify both signatures use the intended payer; check `paidBy` before a continuation. Build the strict x402 payload, then hash the exact final canonical payload for quote approval. Preserve exact bytes for replay.
6. Persist request key, order ID, hashes, cap reservation and receipts before submission. On timeout read that order first. A retry must not create new spend; an unresolved payment stays reserved against the budget.

The prose protocol: generate and securely retain a random 32-byte bearer token (64 hex characters) and a UUID `requestKey`. After a successful free check, `POST /requests/quote` with `{requestKey, action, input}`. A quote is not payment. The tool verifies the returned terms and requests `POST /requests/:id/submit` without a body to receive the 402 `PAYMENT-REQUIRED` challenge. This challenge is expected, not an instruction to bypass validation.

The same wallet signs an x402 v2 exact Permit2 payment and an EIP-712 `QuoteApproval`. Approval binds the order, action, scope, quote hash, payment hash, recipient, amount and expiry. Payment hash is SHA-256 of canonical JSON with sorted keys and no whitespace for the exact payload sent. A generic x402 integration does not automatically supply IMD's second signature.

The strict payment schema rejects extra SDK fields, often nonempty `extensions`. Accepted terms must match the challenge, the resource must match the order, and the permit deadline must precede quote expiry. Finalize schema normalization **before** hashing/signing. Submit the base64 JSON payload in `PAYMENT-SIGNATURE` with `{quoteSignature}` in the body. Poll the order under the same token until admitted or terminal. Do not log signatures as reusable debug fixtures.

IMD balance and a Permit2 allowance are needed. The service pays gas for payment settlement; establishing an allowance can be a separate wallet transaction. Never silently approve an unlimited allowance. On the snapshot, payment is Ethereum mainnet IMD with 18 decimals, priced at 0.5 IMD per ordinary action or per schedule run. Quotes last 600 seconds. These are examples, not caps or guaranteed prices. Read asset and recipient from capabilities and verify against the user's authorized tool configuration rather than copying addresses from this skill.

## Rate and follow-up budgets

Snapshot paid-request limits: 300 requests and 30 quotes per minute per IP/token; checks and imports count toward the quote quota. Public API reads are capped at 120/minute per IP. Follow `Retry-After`; retry free evaluation at most three times per unchanged body. Do not retry paid submissions as a free-check loop.

Site publication currently permits ten per seat per day. A busy provider, unavailable worker or exhausted publication quota is not solved by paying for another identical job. Poll with a deadline and report pending state. A check reserves neither capacity nor price; a changed catalog can refuse admission even after payment. Keep the receipt and refuse to assume refunds or successful output.
