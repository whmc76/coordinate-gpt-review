---
name: coordinate-gpt-review
description: Use when a user wants Codex to coordinate GPT or ChatGPT review on local files, uploaded materials, copied context, review packs, prompts, screenshots, or generated artifacts while preserving evidence, implementation ownership, verification, and final decisions.
---

# Coordinate GPT Review

## Overview

Use this skill for GPT review loops where the main agent must keep ownership of the task: review scope, browser GPT evidence, implementation, local verification, and final decisions. GPT feedback is advisory until it proves it saw the supplied materials and the main agent verifies any accepted change locally.

Real GPT review means a browser ChatGPT/GPT conversation was actually sent and the latest response was extracted. Internal critique, simulated review, another Codex subagent's opinion, or a locally generated "second review" is not GPT review and must be labeled internal or simulated.

Original author: whmc76. This skill is open source under Apache License 2.0 with a NOTICE file; downstream copies and modified versions should preserve the original author, license, and NOTICE attribution.

## Decision

Use this skill when the work needs an external GPT pass and at least one condition is true:

- The review requires uploading a file, zip, screenshot, doc, sheet, slide, or copied material through the in-app browser.
- The review needs a clear original user input -> Codex change -> GPT suggestion -> Codex accept/reject loop.
- The feedback must be captured as evidence before the main agent acts on it.
- The user asks whether GPT/browser operations should be handled by a subagent.

Do not use this for ordinary web research, local test results, or direct code review that can be completed inside the current workspace.

## Role Split

Main agent owns:

- defining the review question and acceptance criteria;
- creating or approving the focused review pack;
- submitting to and extracting from ChatGPT in the in-app browser by default;
- deciding whether GPT feedback is valid;
- implementing accepted changes;
- running local verification;
- making final status, release, or readiness decisions.

Liaison subagent owns:

- organizing the review prompt or focused review pack;
- updating this skill or related non-browser instructions;
- structuring already extracted GPT/user feedback;
- performing non-browser file operations and evidence packaging;
- browser operations only when subagent browser capability has already been verified in the current environment, or the user explicitly asks the subagent to try.

If subagent tools are unavailable or unreliable for browser work, keep the browser phase in the main conversation and tell the user which evidence status applies.

If the user explicitly asks for liaison/subagent browser operation, the main agent may dispatch the liaison after recording the browser target and fallback plan. The liaison must not simulate feedback if browser attach, upload, send, wait, or extraction fails.

## Main-Conversation GPT Review Loop

When a task requires submitting material to ChatGPT or extracting feedback from ChatGPT through the in-app browser, the recommended path in this environment is main-conversation browser control by the main agent.

- Use the main conversation for in-app browser ChatGPT submit, wait, and extraction unless subagent browser capability has already been verified for this task or the user explicitly requests a subagent attempt.
- Use subagents for prompt preparation, review-pack organization, skill updates, structured summaries of existing feedback, and non-browser file work.
- Keep one review scope/version label for every round, such as `market-v0`, `carriers-v0`, or `lesson-sample-v0`, and include that scope/version in prompts, raw evidence, handoffs, and final summaries.
- Preserve the review chain explicitly: original user input, Codex change, GPT suggestion, and Codex accept/reject decision.
- Save raw GPT output and a structured handoff only after a real browser GPT response has been extracted; otherwise keep the status as `not sent`, `sent but not extracted`, or `simulated/internal only`.
- Treat main-agent browser work as the normal path, not an exceptional fallback, when subagent browser reliability is unknown.

## Evidence Contract

Use these exact status distinctions:

- `not sent`: no browser prompt was submitted to GPT.
- `sent but not extracted`: the prompt was submitted, but the latest GPT response was not captured.
- `extracted valid`: the latest GPT response was extracted and passes the validity gate.
- `extracted invalid`: the latest GPT response was extracted but fails the validity gate.
- `simulated/internal only`: feedback came from Codex/internal reasoning, not from browser GPT.

Never label `simulated/internal only` as GPT, ChatGPT, external review, GPT second review, or liaison evidence. It cannot satisfy this skill.

Evidence files are created only after actual GPT response extraction. Do not claim raw feedback files, handoff files, attachment evidence, or saved response paths unless those files exist.

## Capability Preflight

Before any browser GPT round, the main agent must record the browser target, review scope/version, and expected handoff path. Use the most concrete available target in this order:

1. current or selected in-app browser tab;
2. exact ChatGPT conversation URL;
3. new ChatGPT conversation URL.

Dispatch a liaison for browser operations only when subagent browser capability is verified or the user explicitly asks the subagent to try. If a previous liaison in the same task failed with browser attach timeout, DOM snapshot failure, clipboard/input timeout, or in-app browser visibility limitations, do not repeat the same dispatch unchanged. Move browser work to the main conversation unless the user asks for another subagent attempt with a changed target.

## Workflow

1. Define a focused review target.
   - Ask one concrete question: blocker check, specific implementation review, artifact review, status validation, or follow-up confirmation.
   - Keep the attachment small. Prefer focused zips over full project dumps.
   - Include files GPT needs to inspect: source, schemas, reports, README, test summaries, and prior feedback only when relevant.

2. Prepare the browser round.
   - Use the main conversation for ChatGPT in-app browser operations by default.
   - If dispatching a liaison is justified, give the subagent the review pack path, target GPT/browser conversation if known, exact prompt, validity gate, and expected handoff path.
   - Do not ask the liaison to implement changes or decide final status.
   - If the user already opened a ChatGPT/browser page, use the current/selected tab or the exact conversation URL.
   - The liaison must not simulate feedback if browser attach, upload, send, wait, or extraction fails.

3. Upload or paste through the in-app browser.
   - Attach the actual file or paste the actual content. Never paste a local filesystem path as a substitute for upload.
   - If attachment paste hangs, inspect the composer before retrying; ChatGPT may display random attachment names.
   - If a large zip fails repeatedly, create a smaller focused zip and retry.

4. Send a scoped prompt.
   - State the review target and the current boundary or status.
   - Ask GPT to cite files, fields, IDs, command outputs, screenshots, or exact artifact details it inspected.
   - Ask GPT to separate blockers from non-blockers.
   - Ask GPT not to recommend readiness/status upgrades without evidence from the attached material.

5. Wait for the complete response.
   - Poll until the stop/streaming control disappears or the page shows completion.
   - Do not summarize partial output.
   - Extract the newest GPT answer from the page, not an older thread message.

6. Apply the validity gate.
   - Valid: GPT mentions specific uploaded files, fields, IDs, status values, source behavior, screenshots, or report output.
   - Invalid: GPT gives generic praise, generic advice, assumes unseen files, ignores the attachment, or recommends a status upgrade without evidence.
   - Low confidence: GPT saw the artifact but gives comments that cannot be checked locally.

7. Save evidence.
   - Save the raw GPT response.
   - Save a concise summary with validity judgment.
   - Link the attachment or focused pack and any files GPT cited.
   - Save nothing, or mark planned paths as not created, when no actual response was extracted.

8. Main agent acts only after local verification.
   - Convert accepted feedback into local changes or explicit deferrals.
   - Run the narrowest verification tied to the accepted feedback.
   - Record residual risk and whether another focused GPT pass is needed.

## Iterative Review Loop

When the user gives substantive feedback after a GPT review, or expands the work with new modules such as toys, apps, frontend pages, files, or lesson sections, start a new review round.

- First integrate the user feedback into a fresh, reviewable Codex plan or artifact. Do not send raw deltas alone when the new scope needs a coherent proposal.
- Assign each round a review scope/version, such as `market-v0`, `carriers-v0`, or `lesson-sample-v0`, and use that label in prompts, handoffs, evidence files, and final summaries.
- Treat scope changes as new evidence states. Do not apply a previous GPT answer to the new scope; the changed scope returns to `not sent` or `sent but not extracted` until GPT has reviewed it.
- Each GPT handoff must state what changed since the previous round: newly added material, removed material, changed constraints, and open questions.
- The final Codex proposal must separate: original user input, Codex change, GPT suggestion, and Codex accept/reject decision, with brief reasons for rejected or deferred advice.
- If the liaison fails, keep using the failure and fallback rules below, but do not claim the new scope was GPT-improved unless the new scope reached `extracted valid` or a clearly labeled low-confidence extraction.

## Liaison Failure and Fallback

If the liaison cannot attach to the browser, including `Timed out waiting for the Browser webview to attach for this browser-use page`, the main agent must report:

- `liaison failed`;
- the exact blocker/error text;
- the current evidence status from the Evidence Contract.

Then move the ChatGPT browser work to the main conversation unless the user explicitly asks for another subagent attempt. Main-agent browser work is the recommended path in the current environment, not an exception. The final answer must state whether real GPT evidence came from main-agent browser work, liaison extraction, or manual user copy.

Do not repeatedly dispatch subagents against the same ChatGPT page after attach timeout, DOM snapshot failure, clipboard/input timeout, or IAB visibility errors. At that point the first bad boundary is the liaison-to-browser layer, not GPT itself. Move to one of these explicit states:

- `liaison failed; sent but not extracted`: prompt was sent elsewhere but liaison cannot extract it.
- `liaison failed; not sent`: liaison never reached the composer.
- `main-agent browser round`: the main agent will send or extract in the main conversation.
- `manual export requested`: ask the user to copy the GPT answer when neither liaison nor main-agent browser control can extract it.

## Response Extraction Ladder

Use this order when extracting an already generated ChatGPT answer:

1. Wait until the composer no longer shows `Stop`, `Stop generating`, or a streaming/working control.
2. Prefer main-agent extraction in the main conversation when in-app browser work is needed.
3. Use liaison extraction only when subagent browser capability is verified or explicitly requested.
4. If the normal ChatGPT page times out, retry the exact conversation URL with `?mweb_fallback=1` or keep the existing mweb fallback URL if the user already opened it.
5. If DOM snapshot fails, use visible DOM/accessibility extraction. If that truncates the answer, use browser selection/copy or ask the user for manual copy.
6. Save raw and handoff files only after the extracted text contains the latest GPT answer and passes or fails the validity gate.

The extraction output must include source identity: `main-agent browser`, `liaison`, or `manual user copy`.

## Subagent Prompt Template

```md
You are the GPT review liaison. Do not use the in-app browser unless the main agent states that subagent browser capability is verified or that the user explicitly requested a subagent attempt. Otherwise prepare materials, structure existing feedback, or perform non-browser file work only.

Review pack: <absolute path or copied material source>
Browser target: <current/selected tab, exact ChatGPT conversation URL, or new ChatGPT conversation>
Review target: <one concrete question>
Prompt to send:
<exact GPT prompt>
Scope/version: <round label>

Validity gate:
- Treat feedback as valid only if GPT cites concrete files, fields, IDs, status values, screenshots, source behavior, or report output from the supplied materials.
- Mark generic or evidence-free feedback invalid.

Return:
- uploaded/pasted material identity;
- exact prompt sent;
- raw latest GPT response location;
- completion status: not sent | sent but not extracted | extracted valid | extracted invalid;
- blocker list;
- non-blocker list;
- evidence GPT cited;
- validity judgment;
- recommended main-agent next action.

If browser attach, upload, send, wait, or extraction fails, return `liaison failed` with the exact blocker/error text. Do not simulate feedback. Do not implement changes. Do not decide final project status.
```

## Handoff Format

```md
# GPT Review Handoff

## Material
- Attachment/content:
- GPT conversation:
- Prompt sent:

## Validity
- Judgment: valid | low-confidence | invalid
- Evidence GPT cited:
- Missing evidence:

## Blockers
- [ ] Issue:
  - GPT evidence:
  - Local files/fields to verify:

## Non-blockers
- [ ] Issue:
  - GPT evidence:
  - Suggested timing:

## Accepted or Deferred
- Accepted changes:
- Deferred/rejected changes:
- Main-agent verification required:

## Next Action
- Re-review needed:
- Blocks implementation/status:
```

## Browser Control Notes

When browser control is required, the main agent should normally use the available in-app browser tooling or the `browser:control-in-app-browser` skill in the main conversation. Prefer accessibility roles and DOM snapshots for ChatGPT composer state. After paste/upload attempts, inspect the current composer before retrying to avoid duplicate attachments or stale prompts.

Subagent browser operations are conditional, not default. Use them only after verified subagent browser capability or explicit user instruction.

ChatGPT pages may expose more reliable controls through mobile web fallback. If the user or browser is already at a URL containing `mweb_fallback=1`, keep that target instead of normalizing it away.

## Common Mistakes

| Mistake | Correction |
|---|---|
| Pasting `C:\...file.zip` or `/path/file.zip` into GPT | Upload the actual file or paste actual content. |
| Treating GPT's confidence as evidence | Require concrete references to supplied material. |
| Letting liaison decide readiness or release status | Main agent decides after local verification. |
| Sending the whole repo for a narrow question | Build a focused review pack. |
| Acting on partial streaming text | Wait for completion, then extract the latest response. |
| Re-uploading blindly after timeout | Inspect composer state first; random zip names can still mean attachment success. |
| Calling internal critique "GPT review" | Label it `simulated/internal only`; it does not satisfy this skill. |
| Treating subagent browser work as the default | Use the main conversation for in-app browser ChatGPT submit/wait/extract unless subagent browser capability is verified or the user explicitly requests a subagent attempt. |
| Repeating the same failed liaison dispatch | Do not keep sending subagents to the same ChatGPT page after attach, DOM, clipboard/input, or visibility failures; move browser work to the main conversation. |
| Final answer implies liaison evidence after main-agent browser work | Say real GPT evidence came from main-agent browser extraction. |
| Treating browser attach timeout as GPT failure | Label the first bad boundary as liaison-to-browser attach; GPT may still have generated a response. |
| Losing the mweb fallback URL | Preserve `mweb_fallback=1` when it is the current working ChatGPT target. |
| Forgetting scope/version or decision trace | Preserve scope/version and separate original user input, Codex change, GPT suggestion, and Codex accept/reject decision. |
