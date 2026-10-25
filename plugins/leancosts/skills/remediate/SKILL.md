---
name: remediate
description: Take ONE leancosts savings opportunity from the register to executed, from the terminal — triage it, file and approve its change request, dry-run the script leancosts rendered, apply it with the user's own cloud credentials after they say so, and confirm the receipt landed. Use when the user says "fix the top leancosts opportunity", "remediate a leancosts finding", "apply a leancosts saving", "run a leancosts change request", or passes an opportunity id.
---

# Remediate one leancosts opportunity

leancosts is **read-only by construction**: it never touches the cloud. It renders a script; the user's own
credentials run it; a scoped receipt proves it ran. You orchestrate the leancosts MCP tools and run the one
script, nothing more. **One opportunity per invocation.** Any refusal or failure: stop, show where the
change request stands (`get_change_request`), and end. Never loop to the next one.

## 1. Pick the opportunity

- With an id from the user, use it.
- Without one, call `list_opportunities` (view `register`, open rows) and show the **top 5 by monthly
  value**: title, resource, estimate, confidence. Ask which one. Wait.

Then call `get_opportunity` and show its calculation (the formula and the numbers behind the claim).

## 2. First confirmation: "Accept and file a change request?"

Ask exactly that and wait for a literal yes. If the answer is no, offer to record the decision and end:
`defer_opportunity` with a return date they give ("not now": the row returns on that date, or earlier if the
cost changes materially; `resume_opportunity` brings it back sooner). Use `reject_opportunity`
only if they say it should never come back: it records a decision about this type of finding, so ask whether
it covers this resource, this account or the whole organization (`scope`), and whether the finding does not apply (a DR replica, a compliance hold, a false positive: send "not_applicable") or is valid and the cost is accepted (send "valid"). Never choose the `verdict` yourself. Say that `withdraw_wont_fix_decision` undoes it.

On a yes, stopping at the first error:

1. `accept_opportunity` — only if the row is `surfaced` or `deferred`; skip it if already `accepted`.
2. `draft_change_request` — for a size-up row, ask the user for the target SKU; never invent one.
3. `submit_change_request`
4. `approve_change_request` — if it is refused because the author cannot approve their own request, **stop**:
   tell the user self-approval is off for their organization (Admin → Workspace → Approvals) or a second
   person must approve. Never work around dual control.

## 3. Get the script

Call `get_change_request_execution_bundle`. If it has no executable payload, stop: the change is applied from
its guidance, and the user records it afterwards with `record_change_request_execution`.

Otherwise save `script.content` to `script.fileName` in the current directory and make it executable. If
`requiredInputs` is not empty, ask the user for each value and export it; never guess one.

## 4. Check whose cloud this is

Before anything runs, show which identity the script will use and ask the user to confirm it is the right
account:

- `provider: "azure"` → `az account show --query "{name:name, id:id, tenant:tenantId}"`
- `provider: "aws"` → `aws sts get-caller-identity`
- `provider: "gcp"` → `gcloud config list account project`

## 5. Dry run

Run `./<fileName> --dry-run`. It runs every pre-flight and prints the commands; it changes nothing. Show the
output. If it fails, stop.

## 6. Second confirmation: "Apply?"

Only after a clean dry run, ask "Apply these changes to <account>?" and wait for a literal yes in the
conversation. No yes, no apply.

Then:

1. `mint_execution_receipt_token` — returns a token scoped to this one change request. Do not print it; export
   it for the run only.
2. `LEANCOSTS_RECEIPT_TOKEN=<token> ./<fileName>` — the script applies the change and reports back itself.

## 7. Confirm it landed

- `get_change_request` — the status must read `executed`. If it does not, the report-back did not land: show
  the script's last lines and stop.
- `list_ledger_entries` — show the draft impact entry this change created. Its saving is measured later from
  the bill, not claimed now.

End with one line: what changed, where, and the estimate the bill will now confirm or correct.
