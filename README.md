# Multi-Agent Workflow Skill

A configurable Pi/OpenCode/Claude Code skill for running multi-agent feature and
PR workflows.

The workflow is driven by role configuration:

- **reviewer**: local review/verdict, no mutation
- **implementer**: minimal code changes or evidence-backed no-change defense
- **communicator**: optional commit/PR-comment drafting from real validation only
- **UI reviewer**: optional UI/UX review for frontend-facing work

## What This Skill Enforces

Local reviewer models stay local. Do **not** post comments like `@sol rereview`
or `ready for Sol rereview`. PR comments should only summarize actual changes,
validation, and evidence-backed no-change decisions.

## Files

```text
skills/multi-agent-workflow/SKILL.md          # the Pi skill
config/multi-agent-workflow.toml              # global default config
examples/project.multi-agent-workflow.toml    # repo override example
```

## Install For Agents

If you are an agent asked to install this skill, do the following from any safe
scratch/checkouts directory. Do not copy files into the target project unless the
user asks for a repo-local override.

```bash
set -euo pipefail

repo_dir="${HOME}/.cache/multi-agent-workflow-skill"
rm -rf "$repo_dir"
git clone https://github.com/stonecharioteer/multi-agent-workflow-skill.git "$repo_dir"

mkdir -p "${HOME}/.pi/agent/skills" "${HOME}/.pi/agent/config"
rm -rf "${HOME}/.pi/agent/skills/multi-agent-workflow"
cp -R "$repo_dir/skills/multi-agent-workflow" "${HOME}/.pi/agent/skills/"
cp "$repo_dir/config/multi-agent-workflow.toml" "${HOME}/.pi/agent/config/multi-agent-workflow.toml"
```

Verify:

```bash
test -f ~/.pi/agent/skills/multi-agent-workflow/SKILL.md
test -f ~/.pi/agent/config/multi-agent-workflow.toml
```

The skill should then appear in Pi's available skills on the next skill refresh
or new agent session.

## Install With Symlinks Instead

Use this if you want updates from a local clone to take effect immediately:

```bash
set -euo pipefail

repo_dir="${HOME}/code/checkouts/stonecharioteer/multi-agent-workflow-skill"
git clone https://github.com/stonecharioteer/multi-agent-workflow-skill.git "$repo_dir"

mkdir -p "${HOME}/.pi/agent/skills" "${HOME}/.pi/agent/config"
ln -sfn "$repo_dir/skills/multi-agent-workflow" "${HOME}/.pi/agent/skills/multi-agent-workflow"
ln -sfn "$repo_dir/config/multi-agent-workflow.toml" "${HOME}/.pi/agent/config/multi-agent-workflow.toml"
```

## Repo Overrides

Global defaults live at:

```text
~/.pi/agent/config/multi-agent-workflow.toml
```

A project can override them by adding either file at the repo root:

```text
.agents/multi-agent-workflow.toml
.multi-agent-workflow.toml
```

For project-specific defaults, copy the example:

```bash
mkdir -p .agents
cp ~/.cache/multi-agent-workflow-skill/examples/project.multi-agent-workflow.toml \
  .agents/multi-agent-workflow.toml
```

Then edit model/runner choices as needed.

## Config Shape

```toml
[agents.reviewer]
runner = "pi"
model = "openai-codex/gpt-5.6-sol"
thinking = "high"
role = "reviewer"

[agents.implementer]
runner = "pi"
model = "xai-auth/grok-4.5"
thinking = "medium"
role = "implementer"

[agents.communicator]
runner = "opencode"
model = "kimi-2.6"
thinking = "medium"
role = "committer-pr-communicator"
optional = true

[agents.ui_reviewer]
runner = "claude-code"
model = "claude.fable.high"
thinking = "high"
role = "ui-reviewer"
optional = true
trigger = "ui-related-work"
```

Runner meanings:

- `pi`: invoke through Pi / `agent_team` / the active Pi harness.
- `opencode`: invoke through OpenCode.
- `claude-code`: invoke through Claude Code.

## Updating An Existing Install

```bash
set -euo pipefail
repo_dir="${HOME}/.cache/multi-agent-workflow-skill"
if [ -d "$repo_dir/.git" ]; then
  git -C "$repo_dir" pull --ff-only
else
  git clone https://github.com/stonecharioteer/multi-agent-workflow-skill.git "$repo_dir"
fi

rm -rf "${HOME}/.pi/agent/skills/multi-agent-workflow"
cp -R "$repo_dir/skills/multi-agent-workflow" "${HOME}/.pi/agent/skills/"
cp "$repo_dir/config/multi-agent-workflow.toml" "${HOME}/.pi/agent/config/multi-agent-workflow.toml"
```

## Agent Checklist

When using this skill in a project:

1. Read global config.
2. Read repo override if present.
3. Merge config with repo/user instructions taking precedence.
4. Use configured runner/model/thinking per role.
5. Keep reviewer-model review local; do not talk to local models in PR comments.
6. Push/comment only after real validation, if the user asked for that mode.
