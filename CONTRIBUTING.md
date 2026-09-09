# Contributing

Contributions should make the loop more reliable without turning it into an unbounded autonomous runner.

## Guidelines

- Keep the skill domain-agnostic: it should work for coding, research, content, data, and other user-authorized tasks.
- Preserve the active-task default and require a hard user-defined iteration limit for automatic execution.
- Keep the loop contract explicit: objective, work unit, parameters, success evidence, stop rules, and output.
- Prefer objective gates and concrete verification over "looks good" judgments.
- Keep background state compact and free of secrets.
- Ask one blocking clarification at a time; do not turn normal requests into a questionnaire.
- Do not add a daemon, watcher, connector, or external service unless the repository gains a concrete requirement for it.

## Editing the skill

The main instructions live in `.agents/skills/loop-everything/SKILL.md`. Put mode-specific detail in `references/` only when it improves progressive disclosure. Keep `agents/openai.yaml` metadata aligned with the skill description and invocation behavior.

## Validation checklist

Before opening a change:

1. Run `quick_validate.py` against `.agents/skills/loop-everything`.
2. Confirm there are no unfinished scaffold markers or accidental secrets.
3. Exercise at least one map-loop prompt and one converge-loop prompt.
4. Check that the skill stops early on success and stops on its hard iteration limit.
5. Check that a missing target boundary, acceptance gate, or authorization produces a precise clarification or human-review stop.

