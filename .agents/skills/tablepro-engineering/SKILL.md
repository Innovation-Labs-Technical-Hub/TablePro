---
name: tablepro-engineering
description: End-to-end engineering workflow for the TablePro macOS and iOS repository. Use for any TablePro feature, bug fix, refactor, test, build, plugin, driver, AI, MCP, sync, storage, UI, documentation, or release-preparation task that reads or changes repository files. It routes agents to project invariants, coordinates high-compute investigation, enforces one-writer ownership, and requires build, test, lint, and review evidence.
---

# TablePro Engineering

Follow `AGENTS.md` first. Use this workflow for every repository change.

## 1. Establish scope

1. Inspect the branch and `git status --short`.
2. Separate task files from pre-existing user changes. Never include or alter unrelated changes.
3. State the current behavior, expected behavior, acceptance criteria, and risk level.
4. Search the project reference before proposing a design:

```bash
rg -n '^#{2,4} |<symbol>|<issue>|<subsystem>' .agents/skills/tablepro-engineering/references/project-guide.md
```

Read only matching sections and nearby invariants unless the task is a broad audit.

## 2. Load domain guidance

- Project generation, commands, architecture, style, changelog, localization, and CI: search the matching heading in `references/project-guide.md`.
- Plugin or `TableProPluginKit`: read `Plugin System`, `PluginKit ABI`, `DatabaseType`, and plugin CI sections.
- AppKit, SwiftUI, editor, windows, tabs, split views, or grid UI: read `Editor Architecture`, relevant `Invariants`, `Main Coordinator Pattern`, and `Window Close`.
- Sync, CloudKit, storage, connection lifecycle, schema, cancellation, or persistence: search the exact service or invariant name and read the full paragraph.
- Driver or database behavior: read `DatabaseType`, the driver-specific invariant, the vendored header, build script, and sibling driver implementation.
- AI or MCP: inspect `TablePro/Core/AI`, `TablePro/Core/MCP`, the provider disclosure, tool policy, token scopes, connection allowlists, and `docs/external-api/`.
- Verification: read `.claude/skills/fix-issue/references/verification.md` before running builds or tests.
- Platform research: read `.claude/skills/fix-issue/references/research-sources.md`.

## 3. Investigate with independent agents

For non-trivial work, delegate independent read-only lanes before editing:

1. Trace the real code path and state transitions with file and symbol evidence.
2. Verify Apple APIs, SDK availability, dependency headers, or shipped binary behavior.
3. Find existing tests, missing failure coverage, and deterministic verification commands.
4. Challenge architecture, ownership, cancellation, concurrency, security, and backwards compatibility.
5. Add a plugin ABI or UI/HIG specialist when that domain is involved.

In Claude Code, let `ultracode` scale an unrestricted dynamic workflow to the number of genuinely independent lanes the task supports. In Codex, let the parent `ultra` profile delegate automatically and use all 8 configured threads for broad audits with eight distinct questions; workers run at `max`. Give every lane the same problem statement and a narrow question. Require confirmed facts, inferences, and unknowns to be labeled. The main agent verifies the reports and owns the plan.

## 4. Design the complete fix

Write a plan that identifies:

- Root cause and why it produces the symptom.
- Correct ownership boundary and whether the shape needs a targeted fix or refactor.
- Full caller, state, persistence, plugin, and documentation blast radius.
- Relevant project-guide invariants.
- Tests that fail before and pass after.
- Exact build, lint, ABI, or UI verification.

Prefer a measured fact over consensus between agents. Build a small probe when SDK, database, C library, ABI, or binary behavior remains uncertain.

## 5. Implement with one writer

- Keep one writer in the active checkout.
- Use isolated worktrees for additional writers, with disjoint file ownership.
- Preserve existing abstractions when they express the correct behavior. Refactor when they cannot.
- Update tests, docs, localization, and changelog as part of the change, not as later cleanup.
- Regenerate the project immediately after source-file or project-configuration changes.

## 6. Verify serially

1. Run the smallest relevant test suite early.
2. Run the app build.
3. Run affected unit and deterministic UI suites.
4. Build `AllPlugins` for registry-only plugin changes.
5. Run the PluginKit ABI check for shared plugin API changes.
6. Run strict lint on changed Swift files.
7. Inspect the diff and confirm intended tests actually executed.

Never run two `xcodebuild` processes concurrently.

## 7. Review across models

Load `$cross-model-review` for high-risk changes and whenever the other vendor is available. The reviewer stays read-only, receives the acceptance criteria and verification evidence, and reports prioritized findings with file and line evidence. Validate findings before changing code.

## 8. Hand off

Report:

- Root cause and implemented behavior.
- Files and ownership boundaries changed.
- Tests, builds, lint, probes, and reviews run with results.
- Remaining risk and commands not run.
- No claim beyond the evidence collected.
