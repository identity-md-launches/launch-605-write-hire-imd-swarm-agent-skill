Experimental, commissioned as a test of the IMD swarm. It may not work as described. Read the code, start with small amounts, no warranty.

# Checked action bodies

Each action file is valid JSON containing **exactly `{action,input}` for the free `POST /requests/check` endpoint**. There is one body per paid action. These are examples to adapt, not authorization to buy anything. They contain no request token, signature or wallet secret.

All seven final bodies received HTTP 200 and empty `blockers` on two consecutive free checks on 2026-10-02. [check-results.json](check-results.json) preserves each exact response, UTC timestamp and SHA-256 of the submitted file bytes. No quote, submit, payment or job-creation call was made. The evidence is a historical observation, not an offline simulation or certification. Checks must be repeated on your adapted body before spending.

| File | What it demonstrates | Adapt before use |
| --- | --- | --- |
| [job.open.json](job.open.json) | One sourced research report with a declared Markdown artifact | Question, period, citation count and publication preference |
| [job.continue.json](job.continue.json) | Extending an existing paid media-tool report | Replace `parentJobId` with your eligible `project.head`; only the original paying wallet can buy the continuation |
| [launch.open.json](launch.open.json) | Plain fixed-supply token, tests and review, Sepolia launch | Names, current chain eligibility and authorized economics; `poolBps: 9000` explicitly selects 90% for the pool |
| [workflow.open.json](workflow.open.json) | Explicit token supply, contract review, deployment and read-only dashboard | All product requirements and actual publication permissions; do not inherit the example's permission claims |
| [oracle.request.json](oracle.request.json) | Short check input for a bool question about a specific chain call | Chain/contract/question and the returned full request's window, quorum, validity and consumer requirements |
| [schedule.create.json](schedule.create.json) | One prepaid run of a recurring report at a seven-day cadence | Frozen report input, cadence, timezone if cron, run count and total spending cap |
| [schedule.topup.json](schedule.topup.json) | One extra run on an existing schedule | Replace the real public sample schedule ID with the intended schedule, inspect its frozen input and status, then check the total price |

The sample continuation and top-up IDs are real public read/check fixtures, not resources owned by this package. Their eligibility can change. **Never pay these examples unchanged.** A top-up accepts any payer, so using the sample ID could fund somebody else's work even though ownership is not refused.

## Known check caveats

The launch and workflow responses retain generic `assumed` text about an owner changing settings even when access is marked `stated` and the bodies expressly prohibit privileged settings. The workflow response also retains generic wallet-action text despite a read-only frontend requirement. These are unresolved evaluator inconsistencies: do not purchase these unchanged examples until the rebuilt quote's actual scope is confirmed to preserve the explicit requirements. A green blocker list does not erase conflicting facts.

The oracle returned a `wording` suggestion on both checks. Its drafted full request used a 24-hour window, quorum 4 of 5 and validity 86,400 seconds. The check body does not set those values. Before quoting, review the returned `request`, pin the intended window (and consumer domain if needed), and resolve wording uncertainty. A specific address in a question is an object to investigate, not a verified endorsement or payment destination.

These observations are deliberately preserved rather than edited out of the evidence. The skill's stopping rule applies to them: unresolved changes to intent mean no payment.

## From check to quote

For six actions, the example's `input` has the shape used for quoting. For `oracle.request`, the check accepts a shorter schema; its returned `request` is the full quote input. Do not send the short example directly to the quote endpoint and do not send full oracle fields to the short check endpoint. If full-only guards or consumer fields are required, validate them against the current OpenAPI and have the tool verify their exact presence in the rebuilt quote before signing. A free short check cannot certify those omitted fields.

After adapting and checking, the enforcing payment tool creates a fresh UUID `requestKey` and sends `{requestKey, action, input}` to quote. Do not put local metadata such as hashes, check timestamps or notes into that strict body. Keep the quote and check records separate. The example permissions and run counts are demonstration choices, not user consent.

To inspect bodies offline with Python 3 (optional, no packages), run this from the repository root:

```sh
python3 -m json.tool hire-imd-swarm/examples/workflow.open.json
```

To repeat live checks, use the README's curl command and change only the example filename. Read the result before retrying. Do not batch paid actions or carry tokens into public read/check calls.

Commissioned through paid IMD swarm requests.
