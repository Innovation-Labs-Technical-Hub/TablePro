# Fix Quality Bar

## Definition of done

The fix addresses the root cause at the correct ownership boundary, preserves native behavior and project invariants, includes the regression test, builds, passes targeted verification, survives independent review, and reports limitations honestly.

## Refactor versus targeted fix

Refactor when the current shape cannot express correct behavior without a special case, models a multi-state domain as a boolean, places ownership in the wrong layer, or leaves the same failure class in sibling paths.

Use a targeted fix when the architecture is sound and the defect is local, such as a wrong comparison, missing guard, stale mapping, or incorrect ordering. Small is good only when it is complete.

## Evidence

- Cite code with file and symbol, plus a reachable state transition.
- Cite platform behavior with the exact API and authoritative documentation.
- Verify availability in the installed SDK for TablePro's deployment target.
- Verify C and database behavior against vendored headers and shipped artifacts.
- Measure ambiguous behavior with a minimal probe.
- Treat agent reports and competitor descriptions as hypotheses until verified.

## Native UX

- Prefer documented AppKit, SwiftUI, and system behavior over hand-rolled approximations.
- Preserve keyboard access, focus, selection, undo, IME, UTF-16 range handling, accessibility, and responder-chain behavior.
- Use AppKit where the repository deliberately uses it to avoid a known SwiftUI lifecycle or sizing failure.
- Treat competitor behavior as research input. The macOS HIG and verified user requirements decide the design.

## Mandatory completion checks

- User-visible changelog entry when required.
- Relevant product or external API docs.
- Correct localization without interpolated keys.
- Unit coverage and deterministic UI automation when applicable.
- Project generation after source or configuration changes.
- App build, targeted tests, strict lint, and plugin or ABI checks where required.
- Independent other-vendor review for high-risk changes.
- Clean diff review that preserves unrelated user work.

## Collateral findings

Fold a finding into the fix only when it is required for correctness, safety, or verification of the requested behavior. Record other verified findings separately. Do not broaden the implementation, create branches, commit, push, or open follow-up pull requests unless the user's request explicitly authorizes that scope.
