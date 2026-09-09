# Portability

This skill is organized around the shared Agent Skills SKILL.md convention. The loop contract is portable; installation paths and invocation syntax are host-specific.

## Codex

For a repository-scoped skill, keep the folder at:

    .agents/skills/loop-everything/

Codex may use the optional agents/openai.yaml file for display metadata. Active mode runs inside the current task. Background mode may use a Codex heartbeat or scheduled automation when the user requests it.

## Claude Code

Claude Code discovers project skills from:

    .claude/skills/loop-everything/

Copy or symlink the complete folder there, including references/. Invoke it through Claude Code's native skill mechanism. The core skill does not depend on Claude-only frontmatter such as invocation or execution-context controls; add those only in a host-specific adapter when they are needed.

## OpenCode

OpenCode supports project skills in both of these locations:

    .opencode/skills/loop-everything/
    .agents/skills/loop-everything/

This repository keeps .agents/skills/ as the canonical location so the same checkout can serve Codex and OpenCode. Use the host's native skill loading and invocation behavior.

## Other agents

For an agent that implements the Agent Skills convention, install the complete loop-everything directory in its documented project or user skill directory. Preserve the folder name, SKILL.md, and relative references. If the agent does not support background continuation, use active-task mode or provide the durable state to an external scheduler or runner.

Do not treat a copied SKILL.md as evidence that unattended execution is available. Before every background run, verify that the host has a real trigger, a durable state location, a hard iteration bound, and an objective completion gate.
