# workflow-router

Agent skill for routing broad or ambiguous agent work into a concrete workflow.

This is a thin control layer, not a background autopilot. It helps Codex choose
between implementation, debugging, review, GitHub, research, system triage, and
artifact workflows, then requires bounded retries and real verification before
claiming completion.

## What It Is For

Use this skill when a task needs:

- workflow selection before execution
- coordination between local repo work, GitHub context, research, or review
- bounded repair loops instead of repeated blind retries
- verification evidence tied to the requested behavior
- a single agent-facing source of truth for personal workflow policy

## Install

For a copy-style global install:

```bash
npx skills add https://github.com/Moonweave-Systems/workflow-router-skill -g -y --copy
```

For a local single-source install during development, symlink the repo root:

```bash
ln -sfn /Users/choemun-yeong/workspace/projects/agent-tools/ai-skills-dev/my-skills/workflow-router-skill ~/.codex/skills/workflow-router
```

## Use

Example prompts:

```text
Use $workflow-router to decide the right workflow and carry this task through verification.
```

```text
Use $workflow-router to design the agent workflow before making changes.
```

The skill activates through explicit `$workflow-router` use or model selection
for broad, ambiguous, or multi-step work. It is not a background policy or
daemon.

## Repository Layout

```text
.
├── SKILL.md                  # Agent-facing routing contract
├── agents/openai.yaml        # UI metadata for skill listings
├── docs/spec.md              # Product and release contract
├── references/router-map.md  # Detailed routing table
├── LICENSE                   # MIT license
└── README.md                 # Public repo overview
```

## Verification

Validate the skill package:

```bash
uv run python "$HOME/.codex/skills/.system/skill-creator/scripts/quick_validate.py" .
```

## License

MIT
