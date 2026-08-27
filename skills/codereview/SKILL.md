---
name: codereview
description: One-time, read-only correctness review of a requested change. Invoke explicitly with $codereview.
---

# $codereview

Run once on demand. Do not edit, stage, commit, push, or install anything. Inspect the relevant diff and only the dependency paths needed to judge it. Check correctness, error handling, security boundaries, data/API contracts, concurrency, resource cleanup, and tests. Prioritize concrete regressions over style or unrelated pre-existing issues. Report findings with severity, file/line, impact, and a concise fix direction; report what was inspected and any proof that was run. This skill never becomes a standing workflow gate.
