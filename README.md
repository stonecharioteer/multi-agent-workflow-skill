# Multi-Agent Workflow Skill

A configurable Pi/OpenCode/Claude Code skill for running multi-agent feature and
PR workflows.

The workflow is driven by role configuration:

- reviewer: local review/verdict, no mutation
- implementer: minimal code changes or evidence-backed no-change defense
- communicator: optional commit/PR-comment drafting from real validation only
- UI reviewer: optional UI/UX review for frontend-facing work

## Files

```text
skills/multi-agent-workflow/SKILL.md
config/multi-agent-workflow.toml
examples/merrilin.multi-agent-workflow.toml
```

## Install

Copy or symlink the skill into your Pi skill directory:

```bash
mkdir -p ~/.pi/agent/skills
cp -R skills/multi-agent-workflow ~/.pi/agent/skills/
mkdir -p ~/.pi/agent/config
cp config/multi-agent-workflow.toml ~/.pi/agent/config/
```

Per-repo overrides may live at either:

```text
.agents/multi-agent-workflow.toml
.multi-agent-workflow.toml
```

## Policy

The skill explicitly keeps local reviewer models local. Do not post comments like
`@sol rereview`; PR comments should only summarize actual changes, validation,
and evidence-backed no-change decisions.
