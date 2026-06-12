# Workflow Router Skill Spec

## Purpose

`workflow-router` is a thin agent-facing routing skill. It helps Codex choose
the correct workflow for broad, ambiguous, or multi-step work, then keeps the
execution bounded and evidence-based.

The skill exists because many real user requests are not single operations.
They combine local repo inspection, implementation, debugging, review, GitHub
context, current research, system triage, and verification. The router should
make the first decision explicit: which workflow is appropriate, what evidence
is required, and where the agent must stop.

## Product Position

This is a workflow contract, not an autonomous agent product. It should remain
small enough to load quickly and specific enough to shape behavior.

The repo is the single source of truth for the skill. Installed copies should
come from this repo by symlink or copy-style install, with `SKILL.md` and
`references/router-map.md` treated as the runtime contract.

## Non-Goals

- Do not create a long-running background agent.
- Do not implement a task queue, daemon, scheduler, or durable workflow engine.
- Do not hide risky actions behind automatic routing.
- Do not replace specialized skills such as debugging, review, GitHub, document,
  or presentation workflows.
- Do not add scripts unless a repeated deterministic step is proven painful.
- Do not add broad role prompts that duplicate existing shared roles.

## Users

Primary user: a local power user who asks Codex to handle practical work across
many repos and machine-level workflows.

Secondary user: another agent instance reading the skill with little prior
context. The skill should help that instance avoid over-planning, blind retries,
missing verification, and incorrect escalation.

## User Workflows

### Workflow Selection

When the user asks what to do, how to proceed, or asks for a broad task to be
handled, the agent should:

1. Inspect the local context if the answer depends on a repo, file, process,
   config, branch, or installed skill state.
2. Classify the request with `references/router-map.md`.
3. Select one primary workflow.
4. Load only the specialized skills required by that route.
5. Execute when the action is reversible and the user asked for execution.
6. Stop at a decision when the user asked only for strategy, options, or market
   research.

### Execution With Verification

When the user asks Codex to proceed, the agent should carry the chosen workflow
through the smallest meaningful verification step. The completion response must
separate:

- what changed
- what passed
- what was blocked or skipped
- remaining risk

### Repair Loop

When a command, test, browser check, or external call fails, the agent should
retry only when the retry is likely to add signal. It should switch strategy
after repeated identical failures and report the strongest gathered evidence
when blocked.

## Routing Contract

The router must map each request to one primary route.

| Route | Trigger examples | Required evidence |
| --- | --- | --- |
| Implementation | build, fix, add, update, refactor | changed paths plus focused tests or build output |
| Debugging | debug, diagnose, why failing, broken | reproduction signal plus passing regression check |
| Review | review, audit, find issues | findings first with file and line references |
| GitHub | PR, issue, comments, CI, checks | repo/branch/PR identity plus structured context |
| Research | market, latest, best practice, compare | current source URLs and credibility notes |
| Workflow design | workflow, agent, skill, router, loop | selected design plus validation path |
| System triage | Mac, storage, slow, server, config | live process, disk, config, or service samples |
| Artifact | DOCX, PDF, PPTX, spreadsheet, visual output | rendered or artifact-level proof |

If two routes seem plausible, choose the route that provides the earliest
verifiable evidence. Example: a failing CI request is a debugging route with
GitHub context, not a general GitHub summary.

## Specialized Skills And Roles

The router may call specialized skills or roles, but it should not fan out by
default. Prefer one specialized path.

Use these patterns:

- Use a review role only after there is a concrete diff, PR, or file set.
- Use a read-heavy explorer when impact analysis is larger than the patch.
- Use official-doc research when current API or platform behavior matters.
- Use a small fixer when the task is bounded and the desired diff is minimal.

If a specialized tool is unavailable, the router should execute the same route
directly and keep the same evidence standard.

## Safety And Risk Gates

The router may proceed without additional confirmation for reversible local
work:

- reading files
- adding new docs or skill files
- small scoped edits
- local tests and validators
- normal commits when the user requested a committed artifact
- normal pushes when the user requested publishing or remote single-source
  setup

The router must pause for explicit confirmation before:

- force push, hard reset, git clean, branch deletion, or history rewrite
- deleting files or directories
- package or dependency installation
- database migrations
- production deploys
- payments, subscriptions, secret access, or external messages
- public API changes when not clearly requested

Secrets must never be committed. Before pushing a new or changed public repo,
run a lightweight secret pattern scan or a stronger available scanner.

## Bounded Loop Rules

The router should enforce these stop rules:

- Retry the same approach at most twice for mechanical or transient failures.
- After the same error appears three times, change strategy.
- Stop after five total repair attempts unless a new signal materially narrows
  the issue.
- Do not convert a blocked verification step into a success claim.

Each iteration must improve one of:

- reproduction fidelity
- fault localization
- implementation correctness
- verification strength

## Verification Requirements

Verification must match the route:

| Route | Minimum verification |
| --- | --- |
| Implementation | targeted tests, typecheck, lint, build, or a direct command proving behavior |
| Debugging | failing signal before fix and passing signal after fix |
| Review | inspected diff or files plus explicit test-gap assessment |
| GitHub | current repo/branch/PR/check data |
| Research | current sources with URLs |
| Workflow design | skill validator, fixture prompts, or a documented evaluation plan |
| System triage | live command output from the machine or host |
| Artifact | rendered, parsed, or consumer-visible artifact evidence |

If verification is impossible, the final answer must say why and preserve the
best available evidence separately from the unverified claim.

## Skill Package Layout

Required runtime files:

- `SKILL.md`: short routing contract loaded when the skill triggers.
- `references/router-map.md`: detailed route table and default workflows.
- `agents/openai.yaml`: UI metadata.

Repository support files:

- `README.md`: public overview and install guidance.
- `LICENSE`: license.
- `docs/spec.md`: product and behavior specification.

Avoid adding `scripts/` until a repeated deterministic operation appears in
real use. The current value is judgment and routing, not automation code.

## Evaluation Fixtures

Future changes should be tested against fixture prompts before release. The
minimum fixture set should cover:

1. "Fix this failing test in the current repo."
2. "Review this PR for regressions."
3. "Research the current best practice for this API."
4. "My Mac feels slow, find the culprit."
5. "Design the workflow before implementing this skill."
6. "The UI change is done, prove it visually."
7. "CI failed on this branch, debug it."
8. "This DOCX output changed, verify the real document."

For each fixture, record:

- selected route
- whether local context was inspected when needed
- whether risky actions were gated
- verification evidence requested or produced
- whether the final answer overclaimed

## Release Criteria

A release is acceptable when:

- `quick_validate.py` passes on the skill folder.
- `SKILL.md` has no placeholders and remains concise.
- `references/router-map.md` contains the current route table.
- fixture review finds no systematic over-routing, over-execution, or missing
  verification pattern.
- public repo scan finds no committed secrets.
- install path resolves to the intended source.

## Open Questions

- Whether to add a tiny evaluation harness after enough real prompts accumulate.
- Whether to publish as a Codex plugin once the skill stabilizes.
- Whether the router should include explicit mappings to local wrapper commands
  under `~/.codex/wrappers`, or keep those in global agent config.
