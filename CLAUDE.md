# TablePro Claude Code Instructions

@AGENTS.md

`AGENTS.md` is the shared source of truth. Follow it before the Claude-specific orchestration below.

## Runtime profile

- Run Opus in `ultracode` for every TablePro task. This repository persists `ultracode: true`, which means `xhigh` reasoning plus standing dynamic-workflow orchestration; `effortLevel: xhigh` is the fallback. Do not replace it with `max` or a lower effort unless the user explicitly asks.
- Keep the ultracode workflow size `unrestricted`. Scale independent lanes to the problem instead of a token budget, while avoiding duplicate work.
- For non-trivial tasks, use dynamic workflows and the project agents in `.claude/agents/`. Run independent read-only lanes in parallel, then synthesize their evidence in the main thread before editing.
- Prefer `codebase-investigator`, `platform-researcher`, `test-strategist`, `adversarial-reviewer`, and `plugin-abi-reviewer` over generic agents when their lens applies.
- Keep one writer in the current checkout. Use an isolated worktree for any additional writer.
- Use `/clear` between unrelated tasks and `/compact` within a long task. Project instructions survive compaction.

## Shared skills

- `/tablepro-engineering` loads the shared TablePro workflow from `.agents/skills/tablepro-engineering/`.
- `/cross-model-review` loads the shared Claude and Codex review protocol.
- `/fix-issue` is the high-compute root-cause workflow for bugs and behavior gaps.
- `/release` is destructive and may run only after an explicit release request.

## Claude and Codex pairing

- The `codex@openai-codex` plugin is enabled for this repository.
- For an ambiguous root cause or high-risk design, request a fresh Codex read-only investigation before editing: `/codex:rescue --background --fresh Read-only investigation: <question>. Do not edit files.` Omit `--effort` so it inherits the repository's `gpt-5.6-sol` `ultra` profile. Retrieve the result with `/codex:status` and `/codex:result`.
- Never use the rescue command's default write-capable mode while Claude owns the current checkout. A deliberate writer handoff requires an isolated worktree or an explicit ownership transfer.
- After Claude implements a medium-risk change, use the Codex review capability once before handoff.
- For high-risk changes, run both the normal Codex review and one focused adversarial review. Give the second pass a narrow threat statement such as data loss, actor isolation, ABI breakage, SQL dialect drift, or MCP privilege.
- Keep Codex background reviews read-only. Claude remains the writer and validates each finding against the code. Claude's own ultracode workflow lanes may gather review evidence, but they must not start another cross-vendor review.
- Never ask Codex to invoke Claude, and never start another cross-model review from a review session.

## Autonomy

Work through analysis, implementation, and local verification without asking routine permission. Do not infer permission to commit, push, open pull requests, publish, tag, or release. Preserve user changes already in the tree.
