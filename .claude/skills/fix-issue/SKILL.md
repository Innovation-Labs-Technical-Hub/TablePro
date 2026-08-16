---
name: fix-issue
description: High-compute root-cause workflow for TablePro bugs, GitHub issues, behavior gaps, regressions, and native macOS UX problems. Use when the user asks to fix an issue number, URL, crash, incorrect behavior, data or query bug, plugin defect, concurrency problem, or non-trivial UI gap. It runs parallel evidence gathering, adversarial plan review, one-writer implementation, targeted verification, and independent Claude/Codex review while preserving user changes and external-action boundaries.
---

# Fix Issue

Load `/tablepro-engineering` first. Read `references/quality-bar.md`, `references/orchestration.md`, and the relevant sections of the shared project guide.

Run the parent Claude session in `ultracode`. Keep dynamic workflow sizing unrestricted. Spend compute on independent evidence and adversarial review, not duplicated edits.

## Phase 0: Intake and worktree safety

1. Inspect `git branch --show-current` and `git status --short`.
2. Treat existing changes as user-owned. Do not stash, reset, switch branches over them, or include them in the fix.
3. If given an issue number or URL, read the full issue and comments with the available GitHub integration or `gh issue view`.
4. Write one precise problem statement: current behavior, expected behavior, smallest reproduction, environment, and acceptance criteria.
5. Treat reporter code pointers and proposed fixes as hypotheses.

Ask a question only when repository evidence cannot choose between materially different product outcomes. Do not ask routine permission to investigate, implement, or run local verification.

## Phase 1: Parallel investigation

Use the platform's project agents. Run independent read-only lanes concurrently:

- `codebase-investigator`: trace the shipping call path, state transitions, root cause, blast radius, and tests.
- `platform-researcher`: establish the documented Apple or dependency contract and measure uncertain behavior.
- `test-strategist`: identify regression coverage and exact serial verification commands.
- A second codebase investigator: hunt sibling paths and concrete collateral defects in the affected subsystem.
- `plugin-abi-reviewer` for PluginKit, driver, registry, or public plugin API work.

For broad, high-risk, or architectural issues, let ultracode add every useful non-overlapping lane. Do not impose an agent-count or usage-saving cap. Give every lane the same problem statement. Require file and symbol evidence, URLs or exact SDK symbols, reproduction steps, and explicit uncertainty labels.

The main agent reads the actual files behind every load-bearing claim. Agent agreement does not replace verification.

## Phase 2: Blueprint and challenge

Synthesize one blueprint containing:

- Root cause separated from symptom.
- Targeted fix versus refactor decision, with the ownership boundary that makes the behavior correct.
- Full affected path, including callers, state, persistence, plugins, docs, localization, and migration or ABI effects.
- Relevant project-guide invariants.
- Unit, integration, UI, probe, build, lint, and ABI evidence required.
- A collateral register split into required scope, independently useful findings, and unverified hypotheses.

Before editing, send the blueprint to 3 independent read-only critics:

1. Architecture and existing-pattern critic.
2. Missing-scope, edge-case, and backwards-compatibility critic.
3. Adversarial correctness, security, data-loss, concurrency, or ABI critic.

Verify objections before changing the blueprint. A measured fact outranks a critic.

## Phase 3: Implement with one writer

- Keep exactly one writer in the active checkout.
- Use a separate worktree only for a truly independent writer with disjoint file ownership.
- Follow the blueprint's dependency order. Do not downgrade a required refactor into a special-case patch.
- Add the regression test before or with the fix.
- Regenerate generated Xcode projects after source-file, project YAML, or configuration changes.
- Handle changelog, docs, localization, logging, and tests as part of the implementation.
- Preserve all unrelated changes already in the tree.

For SwiftUI work, read `.claude/skills/swiftui/SKILL.md`. For SwiftData work, read `.claude/skills/swiftdata/SKILL.md`. Apply TablePro's deployment targets and hybrid AppKit/SwiftUI architecture over generic defaults.

## Phase 4: Verify

Read `references/verification.md` before running commands.

Run serially:

1. Targeted unit or integration suite.
2. App build.
3. Deterministic affected UI suite.
4. `AllPlugins` when a registry-only plugin changed.
5. PluginKit ABI check when shared plugin API changed.
6. Strict lint on changed Swift files.
7. Before-and-after probe when dependency or binary behavior is load-bearing.

Confirm the intended tests executed. Never run two `xcodebuild` processes concurrently. Do not report an unrun check as passing.

## Phase 5: Independent review

Load `/cross-model-review`.

- If Claude wrote the change, run Codex review.
- If Codex wrote the change, run Claude Opus review in read-only plan mode.
- For high-risk work, add one focused adversarial pass.
- Validate every finding, fix confirmed P0-P2 issues, and rerun affected verification.
- Prevent review recursion. A reviewer never invokes the other vendor or another reviewer.

## Phase 6: Collateral findings

- Fold a collateral defect into the primary fix only when shipping without it leaves the same root cause, makes the fix unsafe, or prevents verification.
- Record independently useful defects with evidence and a proposed disposition.
- Do not silently expand into unrelated implementation, branches, commits, or pull requests. Implement separate collateral work only when the user's request explicitly covers that broader scope.
- Drop speculative, unreachable, style-only, or product-decision findings.

## Phase 7: External actions

Implementation and local verification do not imply authorization to commit, push, open a pull request, tag, publish, or release.

When the user explicitly requests those actions:

1. Recheck the branch and staged diff immediately before committing.
2. Keep the commit atomic and use a one-line Conventional Commit.
3. Never chain commit and push.
4. Include root cause, behavior, tests, and limitations in the pull request body.
5. Follow `/release` only for an explicit release request.

## Final report

Report the root cause, implemented behavior, files changed, verification evidence, independent review result, remaining risk, and collateral findings not implemented. State blockers precisely.

## References

- `references/quality-bar.md`: root-cause, native UX, evidence, and done criteria.
- `references/orchestration.md`: agent charters and review packet format for Claude and Codex.
- `references/research-sources.md`: authoritative platform, SDK, dependency, and competitor sources.
- `references/verification.md`: build, test, lint, UI-test, and environment guidance.
