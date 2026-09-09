---
name: loop-everything
description: Turn a user goal, task, target list, or parameter set into a bounded, self-checking agent loop. Use when the user wants the agent to keep working automatically across iterations, and keep the loop inside the active task by default; use background scheduling only when explicitly requested.
---

# Loop Everything

Convert an ordinary request into an explicit loop contract, then drive the work until the objective is reached, a stop condition fires, or the user must decide something. The loop is agent-driven: after each verified iteration, choose the next useful action without waiting for a new user prompt.

## Defaults and boundaries

- Run in the active task by default.
- Require a user-defined maximum iteration count before starting an automatic loop. If it is missing, ask one concise question for that limit and wait. Do not ask a batch of setup questions.
- Treat the iteration limit as a hard cap. Stop as soon as the objective is satisfied; never spend the remaining iterations merely to use the budget.
- Include a time or cost bound when the work can consume substantial tools, tokens, compute, or external services. The iteration cap alone is not a license to run indefinitely inside one iteration.
- A loop authorizes only the work already authorized by the user. It does not authorize destructive changes, external messages, purchases, deployments, credential use, or other consequential actions that were not clearly requested.
- Do not turn an unbounded word such as "everything" into unlimited work. First derive a finite scope from the request or ask for the missing scope.
- Keep automatic skill selection enabled. This skill should trigger for requests to loop, repeat, batch, iterate, process all targets, keep working, auto-drive, or continue until done, but not for a one-off task that has no repeat behavior.

## Build the loop contract

Before the first iteration, normalize the request into these fields. Keep the contract in task context; for a background run, persist the minimum state described in [background-mode.md](references/background-mode.md).

1. **Objective** - the desired end state, written as an observable result.
2. **Loop kind** - choose one:
   - **Map**: apply the same work unit to each item in a finite target or parameter set.
   - **Converge**: repeatedly inspect and improve one goal until its success criteria are met.
   - **Hybrid**: map over items, then run a bounded verification or improvement pass when the request clearly requires both.
3. **Work unit** - the smallest meaningful action completed once per iteration. Do not count a tool call, status message, or empty retry as an iteration.
4. **Inputs and parameters** - the finite items, range, or named parameter values. Preserve the user's ordering when it carries meaning. Avoid silently constructing a large Cartesian product; ask before doing so when its size is material.
5. **Success criteria** - concrete evidence that allows the agent to stop early.
6. **Stop rules** - maximum iterations, user-requested time/cost bounds, repeated no-progress threshold, safety or approval boundary, and terminal errors.
7. **Output** - the artifact, change, report, or per-item result the user expects.

Infer fields that are clear from context. Ask one question at a time only for a missing field that blocks safe or meaningful progress. Prefer a narrow question such as "What is the maximum number of iterations?" over a questionnaire. If a non-blocking preference is missing, choose a reasonable default and state it briefly in the first progress update.

For a map loop, ensure each iteration has a deterministic parameter binding and an item-level status: `pending`, `running`, `completed`, `failed`, or `skipped`. For a converge loop, maintain a compact progress record containing the current state, the last action, evidence of change, and the next hypothesis or action.

## Readiness and activation

Do a lightweight readiness check before acting:

- If one direct pass can satisfy the request, run one bounded iteration and finish. A loop should reduce operator effort, not add ceremony to a one-shot task.
- For unattended background execution, require all four pieces: a clear objective, durable state, a hard bound, and an objective gate that can reject an incomplete result. If any piece is missing, keep the work in the active task and ask for the missing decision when necessary.
- Defining a loop does not activate a future run. Only the active-task request or an explicit background/schedule request activates it.
- Before every iteration, check the terminal state, remaining budget, approval requirements, and whether the next action is still in scope. If no action is appropriate, record a no-action outcome and stop or wait; do not burn an iteration on an empty retry.

## Active-task loop protocol

## Active-task loop protocol

Use this protocol when the user has not explicitly asked for a later or scheduled run:

1. **Normalize and scope.** State the objective, loop kind, work unit, iteration cap, and success criteria in one compact checkpoint. If the request is ambiguous in a way that changes the target, ask the next single blocking question instead of guessing.
2. **Initialize.** Set the iteration counter to zero. For a map loop, enumerate or validate the finite items before acting. For a converge loop, inspect the current state and establish a baseline.
3. **Execute one iteration.** Choose the highest-leverage next action that advances the objective. Bind the current parameters, make the requested change or analysis, and avoid unrelated cleanup.
4. **Verify.** Check the result against the success criteria using concrete evidence: tests, file state, query output, comparison, or another domain-appropriate check. A claimed action without verification is not progress. For consequential or non-trivial work, use a separate checker skill or fresh verification context when available; the maker should not be the sole authority that its own result is correct.
5. **Record.** Increment the counter and record the parameter binding, action, evidence, status, and next action. Record no-action decisions with their reason. Report only meaningful milestones while the loop is running.
6. **Decide.** Stop immediately on success. Otherwise continue automatically if iterations remain and the next action is clear. If the last iteration made no meaningful progress, change strategy once; if progress is still impossible, stop as `blocked` rather than repeating the same action.
7. **Finish.** End with a concise summary of the objective, iterations used versus allowed, completed and failed items, evidence, remaining work, and the exact reason for stopping.

Respect normal approval and safety boundaries at every iteration. If an action needs user authorization, pause with one precise question naming the action and its consequence. A user's approval to loop does not imply approval for later side effects.

## Parameter and target handling

- If the user supplies a list, use one iteration per list item unless they specify another grouping.
- If the user supplies a range or parameter domain, make the bounds and step explicit before starting. Cap the expanded set at the user's iteration limit.
- If the user names several independent targets, keep their results separate so one failure does not erase successful work on other targets.
- If "everything" means all discoverable items, first inventory the relevant finite scope and show the count. If discovery itself is open-ended or costly, ask the user to define the boundary.
- For recoverable per-item failures, record the failure and continue when the user's objective still benefits from processing the remaining items. Stop the whole loop for systemic failures, safety boundaries, or missing credentials/inputs.
- Never quietly skip an item. Mark it with a reason and include it in the final report.

## Background and scheduled mode

Use background mode only when the user explicitly asks the agent to continue later, run on a schedule, monitor something, or keep working without the active task. Read [background-mode.md](references/background-mode.md) for the state and automation rules.

Prefer a heartbeat attached to the current task when the user wants continuation of this same work. Use a standalone scheduled automation only when the user asks for independent recurring runs or a specific schedule. Create or update the automation through the host's automation tool; do not invent a cron expression in the user-facing response when the host can represent the schedule directly.

The background prompt must carry the objective, loop kind, iteration cap, current state location, stop rules, and notification policy. Each run must rehydrate state, perform at most the remaining budget, verify progress, persist state, and stop or schedule the next continuation. Keep notifications quiet while the state is unchanged or non-actionable; notify on meaningful progress, completion, failure, or required user input.

If the host cannot provide background execution, say so and continue in the active task when possible. Do not simulate persistence by promising to return later.

## Compact progress format

Use a small, stable record so the loop can resume without rereading a long transcript:

```text
Loop: <objective>
Mode: active | heartbeat | scheduled
Kind: map | converge | hybrid
Iteration: <used>/<limit>
Current binding: <item or parameters>
Last evidence: <verified result>
Next action: <one action>
Status: running | completed | blocked | failed | stopped
```

Keep the user-facing update shorter when no decision is needed. Always surface the stop reason when the loop ends.
