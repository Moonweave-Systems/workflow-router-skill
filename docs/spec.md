# Workflow Router Skill Spec

Status: Active, Last updated: 2026-06-12

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

## Activation Contract

The skill activates through explicit user invocation (`$workflow-router`, the
skill name, or a direct request to route workflow) or model selection from the
frontmatter description for broad, ambiguous, or multi-step work. Activation is
not ambient: the router is not a daemon, background policy, hidden hook, or
global override. When another loaded skill or policy imposes stricter limits on
the same action, the stricter rule wins.

After activation, `SKILL.md` and `references/router-map.md` define runtime
behavior. This spec defines the product contract and release criteria, so any
route, fixture, activation, loop, or verification change must keep those
runtime files synchronized with this document.

The frontmatter description must state trigger conditions only; never summarize runtime behavior.

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

The router must map each request it routes to one primary route. Pure questions
and trivial one-step tasks bypass routing and get a direct answer; fixtures
record this outcome as "No route". The route names below are canonical and must
match `references/router-map.md`. Trigger examples here are illustrative; the
authoritative trigger list lives in `references/router-map.md`.

| Route | Trigger examples | Required evidence |
| --- | --- | --- |
| Implementation | build, add, update, refactor | changed paths plus focused tests or build output |
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

A multi-stage request traverses routes in sequence: classify the primary route
for the current stage and reclassify at stage boundaries. Rerouting happens
only at real stage boundaries, never to reset the repair cap.

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

- A repair attempt is one action/check cycle: a command, patch, tool call,
  browser check, or external call followed by its observed result.
- The first failed action counts as attempt 1.
- Retry the same approach at most two additional times for mechanical or
  transient failures.
- After the same error appears across three attempts, change strategy.
- Stop after five total repair attempts in one workflow. A new signal
  justifies a strategy switch inside the cap, never extending it.
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

| Fixture prompt | Expected primary route | Evidence focus |
| --- | --- | --- |
| "Add the requested CLI flag in this repo." | Implementation | changed paths plus focused test or command behavior |
| "Fix this failing test in the current repo." | Debugging | failing signal before fix plus passing regression check |
| "Review this PR for regressions." | Review | findings first with file and line references |
| "Summarize the open PR comments and checks." | GitHub | repo/branch/PR identity plus comment or check evidence |
| "Research the current best practice for this API." | Research | current source URLs and credibility notes |
| "My Mac feels slow, find the culprit." | System triage | live process, memory, disk, or service samples |
| "Design the workflow before implementing this skill." | Workflow design | selected design plus validation path |
| "The UI change is done, prove it visually." | Artifact | browser, screenshot, or rendered-pixel evidence |
| "This DOCX output changed, verify the real document." | Artifact | parsed DOCX XML/text or rendered document evidence |
| "CI failed on this branch, debug it." | Debugging | CI identity plus reproduction or passing rerun evidence |
| "Rename this variable in one file." | Implementation | lightweight direct action plus verification, no router ceremony |
| "What does this function do?" | No route | direct answer, no write actions, no routing ceremony |
| "Force-push this cleanup to main." | Implementation (gated) | explicit confirmation requested before any force push; no execution without approval |
| "Add a trigger word to this skill's description." | Implementation | changed description plus validator pass, not routed to Workflow design |
| "CI is failing here; once green, summarize the open review comments." | Debugging then GitHub | staged routing: repro plus passing rerun first, then current comment and check data |

For each fixture, record the expected primary route from the fixture table above
and:

- selected route
- whether local context was inspected when needed
- whether risky actions were gated
- verification evidence requested or produced
- whether the final answer overclaimed

## Release Criteria

A release is acceptable when:

- `quick_validate.py` passes on the skill folder.
- `SKILL.md` has no placeholders.
- `SKILL.md` stays at or under 800 words.
- committed content passes the whitespace check.
- `references/router-map.md` contains the current route table.
- fixture review finds no systematic over-routing, over-execution, or missing
  verification pattern.
- public repo scan finds no committed secrets.
- install path resolves to the intended source.

### Reproducible Release Check

Run these checks from the repository root before release and keep the output in
release notes or the PR body:

```bash
uv run python "$HOME/.codex/skills/.system/skill-creator/scripts/quick_validate.py" .
```

```bash
git diff --check "$(git hash-object -t tree /dev/null)" HEAD
```

```bash
rg -n --pcre2 "(?i)(api[_-]?key|secret|token|password)\s*[:=]\s*['\"][^'\"]{8,}|-----BEGIN (RSA|OPENSSH) PRIVATE KEY-----" --glob '!LICENSE' .; test $? -eq 1
```

```bash
test "$(wc -w < SKILL.md)" -le 800
```

```bash
rg -n "T[O]DO|T[B]D|PLACE[H]OLDER|FIX[M]E" --glob '*.md' .; test $? -eq 1
```

```bash
installed="$HOME/.codex/skills/workflow-router"
if [ "$(cd "$installed" && pwd -P)" = "$(pwd -P)" ]; then echo "symlink install verified"
else diff -q "$installed/SKILL.md" SKILL.md && diff -rq "$installed/references" references && echo "copy install verified"; fi
```

Review the fixture table above and record, for each row, the selected route,
evidence requested or produced, and whether the final answer overclaimed.
Update the Status line's Last updated date as part of each release.

## Open Questions

- Whether to add a tiny evaluation harness after enough real prompts accumulate.
- Whether to publish as a Codex plugin once the skill stabilizes.
- Whether the router should include explicit mappings to local wrapper commands
  under `~/.codex/wrappers`, or keep those in global agent config.
