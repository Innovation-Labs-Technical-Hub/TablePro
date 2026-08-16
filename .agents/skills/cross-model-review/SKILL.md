---
name: cross-model-review
description: Independent cross-vendor code review protocol for TablePro changes. Use after Claude Code or Codex implements a medium-risk change, and always for data loss, destructive SQL, credentials, auth, MCP, AI permissions, sync, migrations, plugin ABI, concurrency, C boundaries, signing, or release automation. It invokes the other vendor read-only, prevents review recursion, validates findings, and produces evidence-ranked results.
---

# Cross-model Review

Read `references/review-rubric.md`. Keep the reviewer independent and read-only.

## Prepare the review packet

Provide the reviewer with:

- Observable behavior and acceptance criteria.
- Base reference and exact diff scope.
- Relevant project-guide invariants.
- Tests, builds, probes, and lint already run.
- A focused threat statement for high-risk changes.

Do not prime the reviewer with the writer's preferred conclusion.

## Pre-implementation second opinion

For an ambiguous root cause, architectural fork, or high-risk design, use the other vendor before editing:

- Claude writer: `/codex:rescue --background --fresh Read-only investigation: <problem and narrow question>. Do not edit files.` Omit `--effort` so the run inherits Codex `ultra`. Then use `/codex:status` and `/codex:result`.
- Codex writer: invoke Claude with `--model opus --effort ultracode --permission-mode plan --no-session-persistence`, the read-only tool allowlist below, the acceptance criteria, and one narrow design question.

The second opinion is evidence, not authority. Verify it against the repository or an authoritative source. Do not let both vendors write in the same checkout.

## When Claude Code is the writer

Use the enabled `codex@openai-codex` plugin:

```text
/codex:review --background
/codex:adversarial-review --background "Focus on <specific threat>"
/codex:status
/codex:result
```

Run the normal review once for medium-risk changes. Add one focused adversarial pass for high-risk changes. Do not enable an automatic review gate that can loop between agents.

## When Codex is the writer

Invoke Claude Code in non-persistent, read-only plan mode from the repository root. Keep the user's `ultracode` profile so Claude can orchestrate independent read-only review lenses:

```bash
claude -p --model opus --effort ultracode --permission-mode plan --no-session-persistence --allowedTools "Read,Grep,Glob,Bash,Workflow" --disallowedTools "Write,Edit,NotebookEdit,Agent" "Review this working tree against its merge base. You may use an ultracode workflow for independent read-only evidence lanes. Do not edit files, invoke Codex, or start another cross-vendor review. Read AGENTS.md, the relevant project-guide sections, the diff, callers, and tests. Report only actionable correctness, security, data-loss, concurrency, ABI, behavior, and missing-test findings. Rank each finding P0-P3 and include file:line, evidence, failure scenario, and the smallest valid fix. State explicitly when no findings survive verification."
```

For high-risk work, run one second call with a narrow threat statement. Do not reuse or continue the first review session.

If Claude CLI is unavailable, use Codex's project `adversarial_reviewer` agent and disclose that the review was not cross-vendor.

If Codex's sandbox blocks Claude authentication or macOS Keychain access, request approval to run the same read-only command in the host environment. Do not modify credentials or Keychain state. If approval is unavailable, use the local fallback and report the limitation.

## Resolve findings

1. Verify each finding in source, tests, SDK documentation, headers, or a probe.
2. Reject speculation and style-only preferences.
3. Fix confirmed P0-P2 findings in the writer session.
4. Re-run affected verification.
5. Re-review only when the fix materially changes the design or a high-risk boundary.

Never ask a reviewer to review its own output or invoke the other vendor. The writer owns the final decision.
