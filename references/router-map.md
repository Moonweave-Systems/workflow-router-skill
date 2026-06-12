# Workflow Router Map

Use this map after reading `SKILL.md` when the request is broad, ambiguous, or
could trigger more than one workflow.

## Canonical Routes

Keep these route names synchronized with `docs/spec.md`. Use exactly one as the
primary route, then add secondary context such as "with GitHub context" only
after the primary route is selected.

| Route | Triggers | Evidence |
| --- | --- | --- |
| Implementation | build, implement, add, refactor, small requested code/doc change | changed paths plus focused tests, build output, or direct command behavior |
| Debugging | debug, diagnose, why failing, broken, "Fix this failing test", "CI failed and fix it" | reproduction signal plus passing regression check |
| Review | review, audit, find issues, inspect diff | findings first with file and line references |
| GitHub | PR, issue, review comments, checks, branch metadata, GitHub summary | repo/branch/PR identity plus structured context |
| Research | market, latest, best practice, compare current options | current source URLs and credibility notes |
| Workflow design | workflow, agent, skill, router, loop, plan the process | selected design plus validation path |
| System triage | Mac slow, storage, config, server, service state | live process, disk, config, or service samples |
| Artifact | DOCX, PDF, PPTX, spreadsheet, UI visual output, prove it visually | rendered or artifact-level proof |

## Specialized Role Selection

Use a specialized role only when it reduces real work:

- `pr_explorer`: read-heavy code tracing, impact surfaces, dependency maps.
- `reviewer`: independent findings-first review after a patch or for a diff.
- `docs_researcher`: official docs, APIs, config, and current best practice.
- `small_fixer`: narrow implementation with minimal diff expectations.

If those roles are unavailable, do the work directly and keep the same evidence
standard.

## Default Workflows

### Implementation

1. Read the touched area and neighboring tests.
2. Identify the smallest behavior change that satisfies the request.
3. Edit only files that trace to the request.
4. Run targeted verification.
5. Report changed paths and evidence.

### Debugging

1. Reproduce the failure or collect the exact live failing signal.
2. Minimize the surface until one plausible cause remains.
3. Patch the cause, not the symptom.
4. Run the failing check again and add a regression check when cheap.

### Review

1. Inspect the diff, branch, PR, or files requested.
2. Lead with actionable bugs and regressions.
3. Include file and line references.
4. Mention missing verification separately.

### Workflow Design

1. Decide whether this should be a skill, plugin, MCP, script, or repo-only doc.
2. Prefer a skill when the value is procedural guidance and routing.
3. Prefer a script when determinism matters more than judgement.
4. Prefer a plugin when distribution must bundle skills, MCPs, or app metadata.
5. Validate with a real prompt or command before calling the design done.

### Research

1. Browse when information may have changed or current market state matters.
2. Prefer official docs, maintained repos, and recent primary sources.
3. Separate market signal from recommendation.
4. End with a decision or a narrow next experiment.

### GitHub

1. Resolve repo, branch, PR, issue, or check identity.
2. Inspect current GitHub context before summarizing or changing anything.
3. Route failures needing a fix into Debugging with GitHub context.
4. Report the structured context and any local verification.

### System Triage

1. Sample live process, disk, memory, config, or service state first.
2. Identify the strongest current signal and separate stale residue from active work.
3. Recommend only reversible local actions unless the user approves risk.
4. Report commands sampled and the concrete culprit or remaining uncertainty.

### Artifact

1. Identify the consumer-visible artifact layer to inspect.
2. Render, parse, screenshot, or inspect artifact cells/XML/pixels as appropriate.
3. Compare the observed artifact behavior to the requested change.
4. Report artifact-level evidence and any unverified gaps.
