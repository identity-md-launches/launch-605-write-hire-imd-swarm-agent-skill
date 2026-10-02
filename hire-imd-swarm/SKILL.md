---
name: hire-imd-swarm
description: Buy bounded work from the IdentityMD swarm by choosing a paid action, drafting and freely checking requests, diagnosing refusals, using a capped payment tool, and following admitted jobs through delivery.
---

# Hire the IMD swarm

Experimental, commissioned as a test of the IMD swarm. It may not work as described. Read the code, start with small amounts, no warranty.

Use this skill when the user wants to commission IMD work or repair a refused request. Installation is not authorization to spend. Preserve the user's scope, economics and publication preferences. This package contains instructions and checked examples, not a wallet or payment client.

## Choose what to buy

| Action | Choose it for | Example |
| --- | --- | --- |
| `job.open` | One implementation, report, review, media asset or website; no deployment | [body](examples/job.open.json) |
| `job.continue` | A new version of an eligible project paid for by the same wallet | [body](examples/job.continue.json) |
| `launch.open` | Contract work ending in an onchain launch, without the full website release | [body](examples/launch.open.json) |
| `workflow.open` | Contracts, tests, independent review, deployment and a website using deployed addresses | [body](examples/workflow.open.json) |
| `oracle.request` | A typed answer from a panel, potentially signed for a consumer contract | [check body](examples/oracle.request.json) |
| `schedule.create` | Repeated `job.open` or `oracle.request` work with prepaid runs | [body](examples/schedule.create.json) |
| `schedule.topup` | More runs on an existing schedule; any wallet may pay | [body](examples/schedule.topup.json) |

Read [example instructions and check evidence](examples/README.md) before adapting a body. Use current capabilities and the skill catalog; reference skills are context, not runnable steps. A report skill is not a research panel, and an ordinary accepted job is not an oracle attestation.

## Gather the minimum facts

Read `GET https://api.imd.fun/requests/capabilities`, `/openapi.json` and `/skills`. These reads are free. [API documentation](https://www.imd.fun/docs/) describes the envelopes. Read [limits.md](limits.md) for drafting budgets, action-specific constraints and payment requirements. Never infer prices, supported chains or recipients from old examples.

Establish the deliverable, acceptance criteria, starting source, ownership, publication permission and spending caps. For existing source, use a public repository and pinned commit; `/requests/import` can resolve them without payment. Never send credentials as source or context.

For `job.continue`, read the parent job and resolve `project.head`. It must be paid, completed and delivered when required, or blocked; there must be no running project job or unfinished workflow. Compare the paying wallet with `paidBy`. Omit `repoUrl`, `baseCommit`, `projectId`, `deploymentLaunchId` and `onchain`. Continuing does not redeploy a token.

## Write a body the evaluator can plan

- State concrete behavior, numbers, permissions and observable outcomes. Replace “secure and production ready” with rules the worker can check.
- In a token workflow's `request`, state the **total supply**, not just “fixed supply.” Include name, symbol, decimals, mint destination and whether later minting exists. Standard project/hook launches use 1,000,000,000 tokens, 18 decimals and plain transfers; custom economics require a supported custom-token action, not a contradictory standard launch.
- Keep workflow `request` + `context` + `draft.objective` together comfortably below about **7,000 characters after folding**. Aim for 6,000 or less to leave room for the rebuilt objective. Individual field limits do not guarantee the folded plan fits.
- **Do not name paths on `gas-and-size-report`, `write-readme-and-docs` or `deploy-script` steps.** Omit their `paths` fields; describe outcomes without prescribing write locations in their objectives. The documented generic path table can disagree with this evaluator constraint. A declared named artifact output is a separate schema field; use it only when needed and recheck it.
- Treat **`foundry.toml` and `lib` (including descendants) as protected**. Do not put them in write scopes or ask workers to edit them. Broad paths or aliases are not a workaround. If the requested work needs protected edits, narrow or restructure it before admission.
- **Never ask a read-only `adversarial-review` worker for a failing test**, a patch, or any written test artifact. Ask for findings, locations, reasoning and reproduction instructions. Put test implementation in a separate write-capable test step, then review the result.
- For contract state changes, state who calls, why they would, who pays gas and what happens if nobody calls. Preserve requested admin powers and document their trust assumptions. Do not describe timers as self-executing transactions.
- For a workflow, use `chain` or a valid joined `dag`, contract work and independent review, exactly one frontend step, and matching onchain/hosting permissions. The frontend uses the live deployment handoff, not invented addresses.

## Always check for free, then retry deliberately

1. Send `{ "action": ..., "input": ... }` to **`POST /requests/check` before every paid request**, including continuations and top-ups. No bearer token, wallet signature or payment is required. A check holds no price and opens no job.
2. Oracle checks use the short input in the example. Inspect the returned full `request` before using it as the quote's input. Defaults can change the quorum, evidence window or validity. Do not strip important requirements simply to make a check pass.
3. Read the body even on HTTP 200: inspect `blockers`, `suggestions`, `facts`, `judged`, `plan` and any project information. `judged: false` means no successful evaluation, not approval. A plan that changes the user's rules is not acceptable even with empty blockers.
4. The evaluator is noisy. For a transient `evaluation_unavailable` or inconsistent semantic refusal, retry the exact unchanged free body, with backoff, up to three attempts total per version. Honor `Retry-After`. Keep the results together. Do not blindly retry deterministic schema, ownership or scope errors: fix them using [errors.md](errors.md), then check the new version.
5. Prefer two consistent blocker-free checks whose plan and assumptions match the intent. If results still conflict or an assumption contradicts the request, stop before payment and report the discrepancy. Do not keep trying until a lucky pass appears. A quote re-evaluates; a past check is not admission or a guarantee.

A free-only example command is in the README. Record the body hash, check time and refusal details so revisions are reviewable. Do not add local evidence fields to the strict API body.

## Pay only through an enforcing tool

Use a payment tool that **defaults to dry-run** and enforces user-approved per-order and cumulative caps, including schedule run totals and any allowance or gas transaction. If no such tool is available, deliver the checked body and stop before payment. An agent's promise to watch spending is not enforcement. Never ask for, receive, print, paste or handle a raw private key or seed phrase in chat. Use an external wallet or secure signer through the tool.

The tool obtains a quote with a persisted random request token and a UUID request key. It compares the quote, chain, asset, recipient, amount, expiry and exact action/input with the authorized intent. It requests the submit challenge, then the same wallet signs the Permit2 payment and the EIP-712 quote approval bound to that exact payment and quote. A stock x402 client alone does not make both signatures. The tool submits both, stores the order and payment receipt, and polls the order. See [payment details](limits.md#payment-tool-contract).

A retry of the same quote uses the same key and body; a changed body gets a new key. After a lost submit response, look up the existing order before doing anything else. Never create a second order just because the first is slow. Only the tool may replay the exact payment bytes against the same order under its idempotency controls.

## Follow admission through actual delivery

Poll `GET /requests/:id` using the original request token, without repaying. Follow the returned `admission.result` URLs after `admitted`, validating that URLs belong to the configured service before forwarding credentials. Resource reads are public: do not send the request token to repository, artifact or explorer links. Check `kind: "refused"` even after confirmed payment; retain its problems and receipt, and do not promise a refund.

- **Job/launch:** Follow `/jobs/:id`, blocked reasons, nodes and `/jobs/:id/submissions`. Read `/jobs/:id/result`; require `complete`, accepted file hashes and delivery details. For launches, also read `/launches/:id` for live status, chain, addresses and transactions. Job completion alone is not proof of deployment.
- **Workflow:** Follow `/workflows/:id` through contracts, deployment, frontend, publishing and validation. Inspect `failure`, each stage's job and `waitingForHosting`. A site name alone is not completion; check published output and validation. `superseded` means a later release took over.
- **Oracle:** Follow `/oracle/requests/:id` and its attestation URL. `attested` and a valid signature are distinct from a worker's accepted submission. Check signer, domain, question, chain, window, quorum/agreement and expiry against the consumer's requirements. A panel may disagree and return no signed answer.
- **Schedule:** Follow `/schedules/:id`, balance, status, next run and `latest`. Follow each opened run's result URL. Skipped/failed slots spend no run; three consecutive failures can pause the schedule. Do not top up merely to clear a failure before diagnosing it.

Use bounded polling (suggested 5 seconds increasing to 30 seconds, respecting server limits). At the user's monitoring deadline, persist the IDs and last status and report that work remains pending; do not imply completion or repurchase. On a blocked/failed/refused/cancelled outcome, report the reason and the smallest scoped repair. Finish with order/job IDs, amount actually paid, result links and hashes, delivered commit or site, and unresolved limitations. Payment buys work, not correctness.
