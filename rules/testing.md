---
description: Testing standards for test files
paths:
  - "tests/**"
  - "**/*.test.*"
  - "**/*.spec.*"
---

# Testing

- Write tests that verify behavior, not implementation details.
- Each test should have one clear assertion. Name it after what it proves.
- Use `describe` blocks to group related tests. Use `it` or `test` for individual cases.
- Test the public API of modules, not internal functions.
- Prefer real dependencies over mocks. Only mock external services (APIs, databases).
- Every bug fix must include a regression test that fails without the fix.
- Run the full test suite before marking any task complete: `[your test command]`

<!--
CUSTOMIZATION NOTES (this comment is stripped from context, it's for you):

Replace [your test command] with your actual command (e.g., pytest, npm test, go test ./...).

Adjust the paths globs to match your project structure:
- Python: tests/**, **/*_test.py
- Go: **/*_test.go
- JavaScript/TypeScript: **/*.test.ts, **/*.spec.tsx
-->
