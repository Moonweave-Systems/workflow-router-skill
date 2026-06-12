# Workflow Router Map

Use this map after reading `SKILL.md` when the request is broad, ambiguous, or
could trigger more than one workflow.

## Intent Classifier

| User intent | Primary route | Evidence required |
| --- | --- | --- |
| "Build", "fix", "implement", "add" | Implementation workflow | Focused tests, lint/build when relevant, changed files |
| "Debug", "why failing", "diagnose" | Reproduce-minimize-fix workflow | Repro command, failing signal, passing regression check |
| "Review", "audit", "find issues" | Findings-first review | File/line findings, severity ordering, test gaps |
| "PR", "issue", "CI", "GitHub" | GitHub workflow | Repo/branch/PR identity, check or comment evidence |
| "Research", "market", "latest", "best practice" | Research workflow | Current source URLs, credibility notes |
| "Workflow", "agent", "skill", "router" | Workflow-design workflow | Local skill/repo state, selected design, validation path |
| "Mac slow", "storage", "config", "server" | System triage workflow | Live process/config/storage samples |
| "DOCX", "PPTX", "spreadsheet", "PDF" | Artifact workflow | Rendered/artifact-level verification |

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
