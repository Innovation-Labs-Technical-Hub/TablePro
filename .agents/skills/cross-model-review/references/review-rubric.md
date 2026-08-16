# TablePro Review Rubric

## Priority

- `P0`: active data loss, credential exposure, destructive operation without a boundary, or a release-blocking break.
- `P1`: reachable correctness, security, ABI, concurrency, crash, or silent-corruption defect.
- `P2`: material behavior regression, missing failure handling, or a test gap likely to let a defect escape.
- `P3`: maintainability issue with a concrete future failure mode. Do not report taste or formatting.

## Required evidence

Every finding includes:

1. `file:line` and the exact symbol or state transition.
2. A concrete input, sequence, or environment that reaches the failure.
3. Why existing guards or tests do not prevent it.
4. The smallest correct fix and the test that proves it.
5. Whether the finding is confirmed, inferred, or blocked on measurement.

## Review lenses

- User data and destructive SQL safety.
- Credentials, auth, token scope, MCP allowlists, and AI tool permissions.
- Actor isolation, cancellation, late completion, task ownership, and process or C boundaries.
- PluginKit ABI, open plugin types, registry-only builds, and dialect-specific behavior.
- Persistence, sync ordering, schema refresh, cache retention, and migration compatibility.
- AppKit and SwiftUI lifecycle, focus, responder chain, accessibility, and window or tab ownership.
- Missing negative tests, UI automation, probes, docs, localization, and changelog entries.

Return `No findings` when no claim meets the evidence bar.
