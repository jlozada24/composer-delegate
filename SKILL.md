---
name: composer-delegate
description: Delegate work to Cursor's Composer model through the cursor-agent CLI while the lead model directs and reviews. Use only when the user explicitly invokes /composer-delegate. Covers the one request it is invoked with.
---

# Composer Delegate

You are the lead. Composer 2.5, run through the `cursor-agent` CLI, is the worker. The goal is to move work the lead would otherwise do onto the cheaper worker with no loss in output quality.

## 1. Activation

Activate only when the user explicitly invokes `/composer-delegate <request>`. It covers that one request, including its clarifications and corrections, and ends when the request completes. A general request to "delegate" something does not activate it.

This skill grants no new permission. Approvals the user or repository requires for edits, commits, or pushes still apply.

## 2. Decide how to delegate

Classify the request as full, split, or none before doing substantive work on it, and tell the user which one you chose in one line.

### 2a. Full delegation

Composer does the whole task. The lead only reviews.

- Use it when the task is bounded and has a known procedure or a sufficient contract: mechanical or prescribed edits, extraction, locating symbols, focused checks, and implementation against a clear spec with checks.
- A short prompt or a small file does not mean the task is easy. Judge by whether the procedure and the acceptance checks are clear.
- Do not duplicate the work. The lead does not rediscover or re-implement what Composer did.

### 2b. Split delegation

The lead does the thinking and Composer does the volume. Use it when the task needs real judgment but most of its tokens are reading, implementing, or testing.

- **Discovery:** send a read-only Composer call first. Ask for a compact overview with exact anchors (file paths and line numbers), important findings, and open questions, with no whole-file dumps. Then read only the anchored sections yourself.
- **Judgment:** isolate the uncertain decision and resolve it. Write the result as a concrete contract: the decision, the anchors, the interfaces or signatures, and the acceptance checks.
- **Execution:** Composer implements and tests against that contract.
- **Partition, don't duplicate:** the lead and Composer never both discover, implement, or review the same work.
- **Review in proportion to remaining risk:** evaluate the receipt, the diff, and the check results. Do not repeat Composer's investigation or run an automatic full second pass.
- **Stuck worker:** refine the assignment with a specific correction. Do not take over the implementation.
- **Count total cost:** lead tokens, repeated context, handoff writing, retries, and review all count, not only Composer's price. Do not claim measured savings without evidence.

### 2c. No delegation

The lead does the task directly.

- Use it when the task is mostly judgment with little execution volume: ambiguous design, cross-cutting decisions, or difficult diagnosis where the reasoning is the work.
- Use it when writing the handoff would cost more than doing the task.
- Say so in one line and proceed without Composer.

## 3. Write the handoff

Composer starts with no knowledge of this conversation. Every prompt is self-contained and states:

- the workspace path
- the bounded task and the intended result
- evidence and anchors already known (file paths and line numbers)
- the files Composer owns; for read-only work, that it must not edit anything
- the acceptance checks to run
- the constraints: no commits, no branch changes, no edits outside the owned files, and stop and report if the work needs a change outside them
- the receipt to return: files changed, checks run with their results, and blockers

## 4. Run Composer

Read-only (discovery, extraction, checks that write nothing):

```bash
cursor-agent --model composer-2.5 --print --output-format text --workspace "$PWD" --trust --mode plan -- "<prompt>"
```

Edits:

```bash
cursor-agent --model composer-2.5 --print --output-format text --workspace "$PWD" --trust --force -- "<prompt>"
```

The command prints Composer's final answer when it finishes. Allow several minutes for it; run long calls in the background and wait for them to exit. Do not add other flags or substitute another model. If `cursor-agent` is missing or not logged in, report that and stop.

## 5. Review

- Read the receipt and `git diff`, and run the acceptance checks. Keep the review in proportion to the remaining risk, as in 2b.
- Send corrections to Composer as a new bounded call that names the specific defect and includes the anchors. Do not patch the files yourself.
- If Composer fails the same correction twice, stop and report it to the user.
- Report to the user what changed, the check results, and anything unresolved.
