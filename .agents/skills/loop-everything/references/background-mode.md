# Background mode

Use this reference only when the user explicitly requests work after the active task, recurring execution, monitoring, or a scheduled continuation.

## Choose the host mode

- Use a heartbeat attached to the current task when the work should continue the same conversation and project context.
- Use a standalone scheduled automation when the user asks for an independent task per run or gives a recurring schedule that should not depend on the current conversation staying active.
- If the user asks for a schedule but does not provide enough timing detail, ask one question for the schedule. Do not guess a business-critical time zone or recurrence.

## State

Heartbeat runs may use the current task context, but still keep a compact progress record in the conversation or in a project-local state file when the run spans substantial work. Standalone scheduled runs need a durable, project-local state file. Use a clearly named `.loop-everything/` directory and do not store secrets in it.

The state should contain only what is needed to resume:

```json
{
  "loop_id": "short-stable-id",
  "objective": "observable desired end state",
  "kind": "map",
  "mode": "heartbeat",
  "work_unit": "one meaningful action",
  "parameters": {"items": ["a", "b"]},
  "max_iterations": 10,
  "iterations_used": 2,
  "time_budget": "optional absolute or per-run bound",
  "cost_budget": "optional token, compute, or external-service bound",
  "status": "running",
  "success_criteria": ["evidence-based condition"],
  "stop_rules": ["hard iteration cap"],
  "current_binding": "b",
  "last_evidence": "verified result",
  "next_action": "one next action",
  "history": []
}
```

Keep history compact: one entry per iteration with the binding, action, evidence, and status. Never persist API keys, private keys, passwords, tokens, or other sensitive values. If the state file is missing or contradictory, pause and ask the user rather than restarting from an unknown point.

For a subjective objective with no objective gate, do not claim unattended completion. Run in the active task and stop at a human-review gate, or ask the user to define an acceptance signal before scheduling it.

## Automation behavior

When creating or updating an automation, its human-readable prompt should tell the future run to:

1. Load the loop state and verify that the project and target still match.
2. Stop if the state is `completed`, `blocked`, `failed`, or `stopped`.
3. Check the hard iteration cap before doing work.
4. Check the time/cost bound and approval boundary before doing work.
5. Perform one meaningful work unit, verify it, update state, and decide whether another run is needed.
6. Ask the user one precise question if a decision or authorization is required.

Do not create duplicate automations for the same `loop_id`. Prefer updating the existing automation when the user changes the limit, schedule, target, or notification preference. Preserve the user's notification intent: stay quiet while unchanged, and notify only for meaningful progress, completion, failure, or required user action unless the user explicitly asks for periodic updates.
