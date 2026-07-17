# Multi-Agent Workflow Skill

![status](https://img.shields.io/badge/status-ready-00d084)
![runner: pi](https://img.shields.io/badge/runner-pi-6f42c1)
![runner: opencode](https://img.shields.io/badge/runner-opencode-0969da)
![runner: claude--code](https://img.shields.io/badge/runner-claude--code-f97316)
![license](https://img.shields.io/badge/license-unlicensed-lightgrey)

> A configurable Pi/OpenCode/Claude Code skill for running multi-agent feature
> and PR workflows.

## Mission HUD

| Channel        | Signal                                    | Contract                                       |
| -------------- | ----------------------------------------- | ---------------------------------------------- |
| 🎯 Objective   | Feature, fix, or PR review loop           | Work from current code and repo config         |
| 🧠 Reviewer    | Local verdict, no mutation                | Findings are evidence, not GitHub bot commands |
| 🛠 Implementer  | Minimal safe changes or no-change defense | Parent verifies every diff                     |
| 🛰 Communicator | Optional commit/PR text                   | Reports only real changes and validation       |
| 🎨 UI reviewer | Optional UI/UX pass                       | Trigger only for frontend/user-facing work     |

```text
TASK / PR
  │
  ├─► reviewer ──► blockers / nits / approval
  │       ▲
  │       │
  ├─► implementer ──► fix or evidence-backed defense
  │       │
  │       ▼
  ├─► parent verification ──► tests / lint / diff audit
  │       │
  │       ▼
  └─► optional communicator ──► commit + PR summary from real evidence only
```

## Prime Directive

Local reviewer models stay local. Do **not** post comments like `@sol rereview`
or `ready for Sol rereview`. PR comments should only summarize actual changes,
validation, and evidence-backed no-change decisions.

## Payload

```text
skills/multi-agent-workflow/SKILL.md          # Pi skill
config/multi-agent-workflow.toml              # global default config
examples/project.multi-agent-workflow.toml    # repo override example
```

## Fast Install For Agents

If you are an agent asked to install this skill, run this from a safe scratch or
checkouts directory. Do not copy files into the target project unless the user
asks for a repo-local override.

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

Verify install:

```bash
test -f ~/.pi/agent/skills/multi-agent-workflow/SKILL.md
test -f ~/.pi/agent/config/multi-agent-workflow.toml
```

The skill should appear in Pi's available skills on the next skill refresh or
new agent session.

## Symlink Install For Live Development

Use this if updates from a local clone should take effect immediately:

```bash
set -euo pipefail

repo_dir="${HOME}/code/checkouts/stonecharioteer/multi-agent-workflow-skill"
git clone https://github.com/stonecharioteer/multi-agent-workflow-skill.git "$repo_dir"

mkdir -p "${HOME}/.pi/agent/skills" "${HOME}/.pi/agent/config"
ln -sfn "$repo_dir/skills/multi-agent-workflow" "${HOME}/.pi/agent/skills/multi-agent-workflow"
ln -sfn "$repo_dir/config/multi-agent-workflow.toml" "${HOME}/.pi/agent/config/multi-agent-workflow.toml"
```

## Config Radar

Config is merged in this order; later entries override earlier ones:

| Priority | Location                                       | Scope                   |
| -------: | ---------------------------------------------- | ----------------------- |
|        1 | `~/.pi/agent/config/multi-agent-workflow.toml` | global defaults         |
|        2 | `.agents/multi-agent-workflow.toml`            | repo override           |
|        3 | `.multi-agent-workflow.toml`                   | alternate repo override |
|        4 | current user instruction                       | task override           |

Copy a project override template:

```bash
mkdir -p .agents
cp ~/.cache/multi-agent-workflow-skill/examples/project.multi-agent-workflow.toml \
  .agents/multi-agent-workflow.toml
```

## Agent Roster Config

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

### Runner map

| Runner        | Use for                               | Notes                                 |
| ------------- | ------------------------------------- | ------------------------------------- |
| `pi`          | Pi / `agent_team` / active Pi harness | Provider-qualified models when needed |
| `opencode`    | OpenCode-hosted models                | Model name is runner-local            |
| `claude-code` | Claude Code-hosted models             | Useful for Fable UI review            |

## Worktree Default

Prefer a dedicated Git worktree per task/PR unless the project directives say
otherwise. This keeps agent changes isolated, reduces accidental cross-branch
contamination, and makes cleanup safer.

```bash
git fetch origin
git worktree add ../<repo>-<task-slug> -b <branch-name> origin/<base-branch>
```

Before creating a worktree, inspect project instructions for required checkout
locations, branch naming, or environments. Never move/delete existing worktrees
without explicit user approval.

## Workflow Modes

### Feature mode

```text
clarify behavior → reviewer risk pass → implementer patch → parent validation
→ optional UI review → optional communicator summary
```

### PR review/fix mode

```text
confirm PR branch → reviewer local pass → classify findings → implementer fixes
or defends → parent validation → optional push/comment → repeat until satisfied
```

## Developer HUD

This repo includes pre-commit hooks for small-repo hygiene:

| Hook                                   | Purpose                            |
| -------------------------------------- | ---------------------------------- |
| Prettier Markdown                      | formats `README.md` and skill docs |
| `check-toml` / `check-yaml`            | validates config syntax            |
| merge-conflict / symlink / case checks | catches common repo hazards        |
| EOF / trailing whitespace / LF endings | keeps text files clean             |
| private-key scan                       | blocks accidental key commits      |

Install and run:

```bash
uvx pre-commit install --install-hooks
uvx pre-commit run --all-files
```

Prettier is invoked with `npx --yes prettier --write`, so Node/npm must be
available for Markdown formatting. TOML is currently validated, not reformatted;
add a TOML formatter only if the config surface grows enough to justify another
dependency.

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
7. Report the final reviewer verdict, validation, remaining nits/blockers, and
   next action.
