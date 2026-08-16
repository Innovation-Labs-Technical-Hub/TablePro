---
name: fix-issue
description: >-
  Root-cause fix workflow for the TablePro macOS app. Use whenever the user wants to fix a
  GitHub issue (by number or URL) or a described bug, behaviour gap, or UX problem, and cares
  about doing it the right way: native AppKit/SwiftUI, Apple HIG, clean architecture, full
  scope, no quick patches. It orchestrates the investigation as a multi-agent workflow
  (codebase tracing, platform/API research, competitor UX, collateral defect hunting),
  synthesizes a refactor-aware blueprint, has that blueprint attacked, implements to
  TablePro's standards, builds and tests and lints, opens the pull request, and then ships
  every other verified defect it found as its own follow-up PR without stopping to ask.
  Trigger on things like "fix issue #1234", "fix this bug", "this should behave like a native
  app", "do this properly / natively", or any non-trivial defect or behaviour gap in the app.
  Prefer this over an ad-hoc fix when the change touches UI behaviour, architecture, or
  anything the user expects to match Apple conventions.
---

# Fix Issue

A disciplined way to fix a TablePro problem so the result is correct, native, and complete, not a patch over a symptom. The core idea: understand before you build, build the version Apple would ship, prove it against the artifact we actually ship, and leave the subsystem better than the issue found it.

Low-quality fixes fail for five reasons: the author did not trace how the code actually behaves, did not check what the platform documents as correct, believed a plausible claim instead of measuring it, stopped at the first change that made the symptom disappear, or never built the result. This skill attacks all five.

**It runs to completion with no approval gate.** Invoking it is the authorization to investigate, implement, verify, commit, push, open the pull request, and then open further pull requests for the other defects the investigation turned up. Do not stop to ask whether to proceed. Stop only when verification fails, and say exactly what failed.

## When to use this

Use it for any non-trivial fix: a bug, a wrong behaviour, a UX gap, "make this work like \<other app\>", or "do this the proper native way". It matters most when the change touches UI behaviour, AppKit/SwiftUI internals, a driver plugin, or anything the user expects to match Apple conventions.

Skip the investigation workflow for genuinely trivial edits (a typo, a renamed constant, a one-line guard with an obvious cause). Still run Phase 3 onward for those, because the branch, build, test, lint, and CHANGELOG rules apply to every change. If you are unsure whether a fix is trivial, it is not.

Run at high effort. The investigation and the adversarial passes are where the quality comes from, and they are worth their cost.

## The standard this skill holds to

Two documents define done, and neither is optional:

- `CLAUDE.md` at the repo root: principles, mandatory rules (CHANGELOG, localization, docs, lint, tests, conventional commits, writing style), and the **Invariants** section listing patterns that have caused real bugs.
- `references/quality-bar.md`: the condensed "is this fix actually done" checklist, the refactor-vs-patch decision, and the native/HIG bar.

Read `quality-bar.md` early. The user's standing preference is a complete, Apple-correct fix grounded in documented APIs. Never pitch a phased, minimal, or quick-win version as the answer, and never implement the user's literal UI suggestion when research says a different native mechanism is correct. Say what the correct approach is, then build that.

### Measure, do not assume

The single highest-value habit in this skill. When the fix depends on how a binary dependency, a C library, or a system framework actually behaves, **write a probe and run it against the artifact we ship**, rather than trusting documentation, an agent's report, or your own recall.

A probe is cheap: a C file compiled against the real `Libs/*.a` and the vendored header, a `swiftc` harness, a SQL statement run through the vendored CLI. It routinely overturns claims that three independent sources agreed on. Treat an unverified claim as a hypothesis no matter how confidently it was stated, including when you stated it.

When the probe settles a fact that the codebase then hard-codes by hand, commit the probe as a script under `scripts/` so a future dependency bump re-checks it instead of trusting a transcription. `scripts/check-pluginkit-abi.sh` and `scripts/check-duckdb-value-api.sh` are the shape.

## Phase 0: Intake

Get a precise problem statement, and a safe place to work, before touching anything.

1. **Read the report.** Given an issue number or URL: `gh issue view <number> --repo TableProApp/TablePro --comments`. Read the body and every comment; reporters often clarify the real complaint in follow-ups. Given a chat description: restate it in one sentence naming the observable wrong behaviour against the expected behaviour.
2. **Capture the specifics.** Reproduction steps, screenshots, database type, macOS version. These shape what the investigators look for.
3. **Treat any code pointer in the issue as a hint, not a fact.** Reporters point at the wrong file often. Verify it and follow the evidence where it actually leads.
4. **Check the tree.** Run `git branch --show-current` and `git status --porcelain`. If the tree is dirty with unrelated work, ask the user how to proceed before creating a branch. Never silently stash someone else's uncommitted changes; a mid-session branch move drops uncommitted edits to tracked files.

End Phase 0 with a written problem statement: what happens now, what should happen, and the smallest reproduction. If the expected behaviour is genuinely ambiguous, meaning two reasonable readings lead to different fixes, resolve it with `AskUserQuestion` now, before spending the investigation on a guess. That is the one question worth asking, and it is about the requirement, never about permission to proceed.

## Phase 1: Investigation workflow

Run the investigation with the `Workflow` tool. This skill's instructions are the opt-in the tool requires, so no further user consent is needed. The full script, with the agent charters written out, is in `references/orchestration.md`. Read it before calling.

The script fans out four investigators, then adversarially verifies what the collateral hunter found:

| Role | Answers |
| --- | --- |
| Codebase tracer | How does the relevant code actually work today? Which files, types, and call paths are involved? Where is the real cause, as opposed to the symptom? |
| Platform researcher | What do Apple's HIG and framework docs, or the vendored header of the dependency in play, say the correct behaviour and the right API are? |
| Competitor / UX researcher | How do TablePlus, DataGrip, Postico, and Sequel Ace handle this? What interaction do users expect? |
| Collateral hunter | What **else** is wrong in the subsystem this fix touches? This feeds Phase 6, and it is the reason the skill leaves the area better than it found it. |

Give every agent the Phase 0 problem statement verbatim and a sharp question. A vague brief produces a vague report. Require concrete evidence: `file:line` for code, a doc URL or exact symbol name for platform claims, a named source for competitor behaviour, a reproduction for a collateral finding.

### Orchestration notes

- **Hardcode the inputs in the script.** The `args` parameter has failed to reach the script global before. Paste the problem statement into the script as a string.
- **The workflow runs in the background.** You are notified when it completes. Do not report, assume, or invent its results before the notification arrives.
- **Its report goes to you, not the user.** Relay what matters in your own words.
- **Verify load-bearing claims yourself.** An agent's confident report is evidence, not proof. Anything the design depends on gets a probe, a header grep, or a read of the actual file. Agents in this session have been wrong in both directions on exactly the facts that decided the architecture.
- **Do not chase parallelism at the cost of the brief.** If the `Workflow` tool is unavailable, run the same charters with the `Agent` tool in one message, or in sequence yourself. Parallelism is a latency optimization; the evidence bar is what determines the fix.

## Phase 2: Synthesis and challenge

You own the blueprint. You have every report plus the conversation context the subagents never saw, so write it yourself rather than handing it to a fresh agent that would re-derive everything.

The blueprint must answer:

- **Root cause**, stated plainly and separated from the symptom.
- **Refactor vs. patch.** Can the current structure express the correct behaviour cleanly, or does the relevant code need restructuring to do this properly? This is the most important call in the skill. If the existing design cannot express the right behaviour, say "refactor X" instead of bolting a special case onto a broken shape. Criteria are in `references/quality-bar.md`.
- **The native, HIG-correct design**, naming the specific AppKit/SwiftUI API or dependency call and the documented behaviour it follows. Prefer a documented platform API over a hand-rolled equivalent.
- **Full scope.** Every file to create or change, in implementation order, plus the edge cases and the TablePro invariants from `CLAUDE.md` the change must respect.
- **Blast radius.** The reported symptom is usually one instance of a class. Say how many cases the root cause actually covers, and cover all of them.
- **Tests** that would have caught the bug: the unit test always, plus `TableProUITests` automation when the fix changes a user flow and that flow runs deterministically. If it does not run deterministically, say so and why, so it can go in the PR description. Name the CHANGELOG and `docs/` updates the fix requires.
- **The collateral register.** Every finding that is real but is not the reported bug, with its evidence and its disposition. Phase 6 consumes this.

**Then have the blueprint attacked**, with a second `Workflow` call that runs three critics on distinct lenses: does it fight existing codebase patterns, what scope is missing, and is the refactor-vs-patch call right. Script in `references/orchestration.md`. Fold what survives into the blueprint. Skip the challenge only for a contained single-file fix.

Do not present the blueprint for approval. Write it down, act on it, and let the PR description carry it.

## Phase 3: Implementation

- **Branch first.** `git checkout -b <type>/<slug>` off the current base in the main checkout. Default to the main checkout, not a worktree; use a worktree only when the tree already holds unrelated in-flight work and the fix needs its own PR, and say so before doing it.
- **Follow the blueprint's file order.** Do the refactor it calls for. Do not quietly downgrade to a patch because the refactor turned out to be more work.
- **Regenerate after adding a file.** A new `.swift` file is not compiled until `scripts/generate-project.sh` runs. The symptom is `cannot find 'X' in scope` from the callers, which reads like the code was never written.
- **Honour the mandatory rules as you go**, not as cleanup: `String(localized:)` for user-facing strings and never with interpolation, `CHANGELOG.md` under `[Unreleased]`, `docs/` for shortcut, UI, settings, or driver changes, OSLog instead of `print`, no comments, early returns, explicit access control.
- **Write the tests the blueprint specified**, unit and UI both. UI suites subclass `UITestCase`; a bare `XCUIApplication()` or `: XCTestCase` under `TableProUITests/` fails a source-scanning guard test, because storage isolation depends on the launch path. When a test fails, fix the source. Never bend a test to match wrong output.
- **Run `Skill(swiftui-pro)`** when the change adds or reworks SwiftUI views, before you consider the code done.

## Phase 4: Verify

You build and test this yourself. Do not hand unverified code back and ask the user to surface compile errors. The full playbook, including the environment setup that makes local `xcodebuild` and `swiftlint` work, is in `references/verification.md`.

The short form:

```bash
export DEVELOPER_DIR=/Applications/Xcode-beta.app/Contents/Developer
xcodebuild -project TablePro.xcodeproj -scheme TablePro -configuration Debug build -skipPackagePluginValidation
xcodebuild -project TablePro.xcodeproj -scheme TablePro test -skipPackagePluginValidation -only-testing:TableProTests/<SuiteYouTouched>
xcodebuild -project TablePro.xcodeproj -scheme TablePro test -skipPackagePluginValidation -only-testing:TableProUITests/<SuiteYouTouched>
swiftlint lint --strict <changed files>
```

Non-obvious rules that decide whether the result means anything: run only the suites you touched and their neighbours, never the whole target; check the executed-test count before believing `TEST FAILED`; cross-reference `.github/macos-test-quarantine.txt` and `.github/macos-ui-test-quarantine.txt` before blaming yourself for a failure; never run two `xcodebuild` invocations at once; and build the `AllPlugins` scheme yourself if the change touched a registry-only plugin, because PR CI never compiles those.

Where the fix rests on how a dependency behaves, finish with the before-and-after probe from "Measure, do not assume". A probe that reproduces the bug on the old path and shows every case correct on the new one is the strongest evidence a PR can carry.

UI tests have their own trap list, including an accessibility tree that differs between this machine and the CI runner, and a SwiftUI container identifier that silently erases every child's. Read `references/verification.md` before writing one.

## Phase 5: Review, commit, and open the primary PR

1. **Self-review the diff.** Run `Skill(code-review)` on the change. Fix what it finds, or say why a finding does not apply. Treat its findings on your own edits as seriously as its findings on old code; this pass has already caught a CHANGELOG heading deleted by a careless `Edit`.
2. **Check the CHANGELOG survived.** After any edit to `CHANGELOG.md`, run `grep -n '^## \[' CHANGELOG.md` and confirm the released version headings are still there. An `Edit` whose `new_string` drops the trailing context silently folds a shipped release into `[Unreleased]`, and the next release notes then re-ship it.
3. **Writing-style gate.** Stage the change, then run the grep from `CLAUDE.md` over the staged diff for em dashes and banned filler words. Rewrite every hit that is on an added line.
4. **Verify the branch, in its own call, immediately before committing.** `git branch --show-current`. The checkout can move between turns, and chaining `commit && push` has already pushed straight to `main` once. Never chain them.
5. **Commit.** Conventional Commits: single line, no body, canonical scope from `CLAUDE.md`. Never pass `-c user.email` or `-c user.name`; the repo identity is already correct and overriding it has shipped unattributed commits.
6. **Push.** `git push -u origin <branch>`. If SSH fails, port 22 is blocked here; push over HTTPS with the `gh` credential helper instead:
   ```bash
   git -c credential.helper='!gh auth git-credential' push https://github.com/TableProApp/TablePro.git <branch>
   ```
7. **Open the PR.**
   ```bash
   gh pr create --repo TableProApp/TablePro --base main --head <branch> --title "<commit subject>" --body-file <file>
   ```
   Write the body to a file rather than passing it inline, so the writing-style grep can run over it first. The body states the root cause, the fix, and what you built and tested, and it closes the issue with `Fixes #<number>`. If a UI flow could not get deterministic automation, say so here: the PR description is the only place that exemption is recorded.

If the fix is stacked on another in-flight branch, base the PR on that branch instead of `main` and say so, rather than dragging the other work into this PR.

## Phase 6: Ship the collateral findings

**An investigation that finds three defects and fixes one has failed at the part that mattered most.** Everything in the collateral register that clears the bar below gets fixed now, in its own pull request, without asking. Do not report a real defect as future work, do not file it as an issue to be triaged, and do not wait for the primary PR to be reviewed or merged.

### Disposition

Sort every register entry into exactly one of three:

1. **Fold into the primary PR.** The primary fix is wrong, unsafe, or incomplete without it. The test: would shipping the primary fix alone make this defect more likely to bite, or leave the same class of bug latent? If yes, it is not collateral, it is scope. In the DuckDB timestamp fix, routing common types through an existing cast path made that path's row-misalignment and unbound-parameter bugs go from rare to routine, so both belonged in the primary PR.
2. **Its own follow-up PR, shipped in this session.** A verified defect that the primary PR should not carry because it is independently reviewable. This is the default for anything that clears the bar.
3. **Report only.** It needs a product decision, or the evidence did not survive verification. Say so plainly in the final report with what you found and why you stopped.

### The bar for a follow-up PR

All of these, or it drops to disposition 3:

- It is a **defect or a concrete correctness risk**, with a failure scenario someone could actually hit. Not naming, not taste, not "I would have structured this differently".
- The evidence **survived verification**. A plausible-sounding claim nobody confirmed is a hypothesis. Probe it or drop it.
- It lives **in this repository**. Another target counts: `TableProMobile`, a registry-only plugin, and `scripts/` are all in scope.
- Fixing it **does not require a product decision**: no changing a UX users rely on, no new feature, no altering a shipped default.

Size is not a reason to skip. A finding too large for one reviewable PR gets split into a sequence of PRs, not deferred. A finding that needs a documented heavy process, such as a PluginKit ABI bump, follows that process in `CLAUDE.md`; that is what the process is for.

### How to ship them

- **Primary PR first, always.** It is what the user asked for.
- **One branch and one PR per finding.** Never batch unrelated fixes into a single PR, and never sneak one into the primary PR after the fact.
- **Base each on `main`**, unless it genuinely depends on the primary branch, in which case base it there and say so in the body.
- **Each follow-up runs Phases 3 through 5 in full.** Tests, build, `AllPlugins` when a registry plugin moved, lint, CHANGELOG, code review, style gate. A PR nobody asked for is not exempt from the quality bar; it is the one most likely to be judged on it.
- **Each body says why it exists**: found while investigating `#<issue>`, what the defect is, its failure scenario, and how it was verified. Link the primary PR.
- **Autonomy is not a licence for risk.** The standing safety rules hold: nothing destructive or irreversible, no force-push, no rewriting history, no touching release tags, no publishing plugins or libraries. Those still get asked about.

## Phase 7: Report

Close with a table of every pull request opened: the primary one and each follow-up, what each fixes, and its verification status. Then, in your own words, the root cause and how the fix maps to it, plainly what you built and tested, and anything left at disposition 3 with the reason.

If part of the work was blocked, say which part and why, and confirm everything else shipped.

## Reference files

- `references/quality-bar.md`: definition of done, refactor-vs-patch criteria, native/HIG bar, mandatory-rules checklist. Read early.
- `references/orchestration.md`: the investigation and challenge workflow scripts, with the agent charters written out. Read before Phase 1.
- `references/research-sources.md`: Apple documentation map, dependency headers, research tools, competitor apps, and what counts as evidence. The investigators use this.
- `references/verification.md`: build, test, and lint playbook, including the environment setup and the failures that are not yours. Read before Phase 4.
