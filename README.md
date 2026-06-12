# workflow-router

> Route broad, ambiguous, or multi-step agent work into the right workflow — then hold the agent to bounded repair loops and real verification before it can claim completion.

[![License: MIT](https://img.shields.io/badge/License-MIT-0F766E.svg)](LICENSE)
[![Agent skill](https://img.shields.io/badge/agent%20skill-Codex-0F766E.svg)](SKILL.md)

`workflow-router` is a thin control layer for [OpenAI Codex](https://github.com/openai/codex) agents, packaged as an agent skill. It is not an autonomous agent, daemon, or background policy: it activates on explicit request or description match, makes the first decision explicit — *which workflow handles this request* — and enforces evidence standards until the work is done.

## Why

Real requests are rarely one operation. "CI failed on this branch, fix it" mixes GitHub context, debugging, implementation, and verification. Without a routing contract, agents over-plan, blind-retry, skip verification, or quietly convert a blocked check into a success claim. This skill pins down three things:

1. **One primary route per request** — chosen up front, from a canonical table.
2. **Bounded repair loops** — a hard attempt cap instead of indefinite retries.
3. **Route-matched verification** — completion requires evidence, not memory.

## The eight routes

| Route | Typical triggers | Must produce |
| --- | --- | --- |
| Implementation | build, implement, add, refactor | changed paths plus focused tests or build output |
| Debugging | debug, why failing, "CI failed and fix it" | reproduction signal plus passing regression check |
| Review | review, audit, find issues | findings first, with file and line references |
| GitHub | PR metadata, issues, comments, checks | repo/branch/PR identity plus structured context |
| Research | latest, best practice, compare options | current source URLs plus credibility notes |
| Workflow design | workflow, agent, skill, router | selected design plus validation path |
| System triage | Mac slow, storage, config, server | live process, disk, or config samples |
| Artifact | DOCX, PDF, PPTX, spreadsheet, visuals | rendered or artifact-level proof |

Pure questions and trivial one-step tasks bypass routing entirely, and
multi-stage requests traverse routes in sequence. The authoritative trigger
list and per-route default workflows live in
[`references/router-map.md`](references/router-map.md).

## Guardrails

- **Loop discipline** — a repair attempt is one action/check cycle. Retry the same approach at most two additional times, switch strategy after the same error appears across three attempts, and stop at five total attempts per workflow. A new signal justifies a strategy switch inside the cap, never an extension.
- **Risk gate** — destructive or externally visible work (force push, deletions, package installs, migrations, deploys, payments, secret access) pauses for explicit confirmation. Secrets are never committed; pushes are preceded by a secret scan.
- **Completion gate** — verification runs before any completion claim, output is confirmed directly, and passing evidence is reported separately from blocked or skipped checks.

## Install

Copy-style global install:

```bash
npx skills add https://github.com/Moonweave-Systems/workflow-router-skill -g -y --copy
```

Single-source install for development — symlink the repo root from inside a clone:

```bash
ln -sfn "$(pwd)" ~/.codex/skills/workflow-router
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

## Repository layout

```text
.
├── SKILL.md                  # Runtime routing contract (≤ 800 words)
├── references/router-map.md  # Canonical route table + default workflows
├── agents/openai.yaml        # UI metadata for skill listings
├── docs/spec.md              # Product spec, fixtures, release criteria
├── LICENSE                   # MIT license
└── README.md                 # Public repo overview
```

## Releasing (maintainers)

`docs/spec.md` defines the release criteria and a reproducible check suite:
validator, whitespace, secret scan, SKILL.md word budget, and install-path
verification. Run the commands in the "Reproducible Release Check" section of
[`docs/spec.md`](docs/spec.md) from the repository root before pushing a
release.

## License

[MIT](LICENSE)
