# Fix-Issue Orchestration

Use the native subagent mechanism of the active platform. Claude Code definitions live in `.claude/agents/`; Codex definitions live in `.codex/agents/`.

## Shared investigation packet

Give every investigator:

- Current behavior and expected behavior.
- Smallest reproduction and environment.
- Acceptance criteria.
- Suspected subsystem, clearly labeled as a hint.
- Current branch and diff scope.

Ask each investigator to return confirmed facts, inferences, unknowns, and concrete evidence. Do not give them the writer's desired solution.

## Agent charters

### Codebase investigator

Answer:

1. What exact entry point and shipping call path produce the behavior?
2. Which owner first has enough information to behave correctly?
3. What is root cause versus downstream symptom?
4. Which sibling paths share the cause?
5. Which project-guide invariants and existing tests apply?

Return file and symbol evidence. Make no edits.

### Platform researcher

For UI work, verify the HIG, AppKit or SwiftUI API, local SDK signature, availability, fallback, focus, accessibility, and responder-chain behavior.

For drivers or dependencies, verify the vendored header and shipped artifact. Build a minimal probe when behavior cannot be proven from source. Return exact URLs, symbols, paths, or measured output. Make no product edits.

### Test strategist

Map the bug to a failing regression test, affected neighboring suites, deterministic UI automation, project generation, build target, plugin build, ABI check, lint scope, quarantine files, and environment traps. Return exact serial commands. Make no edits.

### Collateral hunter

Search the affected subsystem for the same failure class, silent fallbacks, duplicated source-of-truth lists, missing generation or cancellation checks, and drifted sibling implementations. A finding requires `file:line`, a reachable failure scenario, and evidence that no upstream guard prevents it.

### Blueprint critics

Run three independent lenses:

- Existing architecture and ownership.
- Missing scope, edge cases, and compatibility.
- Security, data loss, concurrency, ABI, or native UX.

Critics attack the blueprint, not the problem statement. Give them established measurements so they do not waste work re-deriving settled facts.

## Parallelism

- Claude Code runs the parent session in `ultracode` and keeps the dynamic workflow size unrestricted. Use every genuinely independent lane justified by the issue, including more than eight for a partitionable broad audit.
- Codex runs the parent at `ultra`, workers at `max`, and all 8 configured threads when a broad audit has eight distinct questions.
- Keep one writer per checkout.
- Use isolated worktrees and disjoint files for any additional writer.
- Serialize generation, `xcodebuild`, tests, and ABI checks.

## Cross-vendor review packet

Provide observable behavior, acceptance criteria, base reference, diff scope, applicable invariants, verification results, and one focused threat statement. Do not include the writer's self-review conclusions.

The reviewer returns only P0-P3 findings that satisfy `.agents/skills/cross-model-review/references/review-rubric.md`.
