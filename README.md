# Loop Everything

`loop-everything` is a Codex skill for turning a user's goal, task, target list, or parameter set into a bounded, self-checking agent loop.

It is designed for two modes:

- Active-task mode (default): the agent keeps working in the current task until the goal is complete, blocked, or the user-defined iteration limit is reached.
- Background mode (optional): when explicitly requested, the agent uses a Codex heartbeat or scheduled automation and resumes from compact durable state.

## What it does

The skill converts a natural-language request into a loop contract with:

- an observable objective;
- a finite target or parameter binding;
- a meaningful work unit;
- success criteria and objective evidence;
- hard iteration, time, and cost bounds;
- approval and safety gates;
- compact progress and stop records.

It supports map loops over items, converge loops that improve one goal, and hybrid loops that combine both.

## Use it

Invoke it explicitly with `$loop-everything`, or use a request that clearly asks the agent to loop, repeat, batch, process targets, keep working, or auto-drive a task.

Example:

```text
Use $loop-everything to review these five files, fix the issues you find,
and stop after at most 8 iterations. Verify each fix with the relevant checks.
```

The iteration limit is required for automatic execution. If it is missing, the skill asks for that limit before starting. Clarifying questions are asked one at a time when the target, scope, or acceptance condition is genuinely blocking.

## Loop behavior

Each iteration follows this shape:

1. Check the current state, remaining budget, approvals, and scope.
2. Select the next highest-leverage work unit.
3. Execute it for the current item or parameter binding.
4. Verify the result with concrete evidence.
5. Record the action, evidence, status, and next action.
6. Stop on success, a terminal boundary, or the hard limit; otherwise continue automatically.

The skill does not grant permission for destructive changes, external messages, purchases, deployments, credential use, or other consequential actions that the user did not request.

## Background runs

Background execution is opt-in. A future run is a trigger, not a daemon: the automation must reload the loop state, check the remaining budget before acting, perform a bounded work unit, verify it, persist state, and schedule another run only when needed.

For scheduled work, state is kept under `.loop-everything/` in the project and must not contain passwords, API keys, private keys, tokens, or other secrets. Notifications stay quiet while nothing meaningful changes and fire for progress, completion, failure, or required user input.

Subjective goals without an objective acceptance gate remain in the active task and stop at human review instead of claiming unattended completion.

## Repository layout

```text
.agents/skills/loop-everything/
|-- SKILL.md
|-- agents/openai.yaml
`-- references/background-mode.md
```

Codex discovers the skill from the repository-scoped `.agents/skills` directory. If a skill update does not appear, restart or refresh Codex.

## Validate changes

Run the bundled Codex skill validator from the repository root:

```text
python <skill-creator>/scripts/quick_validate.py .agents/skills/loop-everything
```

Also test realistic prompts for one-shot work, finite target maps, convergence tasks, missing iteration limits, no-progress failures, approval gates, and background resumption.

## Scope

This repository contains an instruction-only skill. It does not run a local watcher or scheduler by itself; active execution is driven by the Codex task, and background execution is driven by the host's automation capability.

