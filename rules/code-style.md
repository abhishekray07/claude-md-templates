---
description: Universal code style rules for the entire project
---

# Code Style

- Functions should do one thing. If you need the word "and" to describe it, split it.
- Name variables after what they contain, functions after what they do.
- Don't abbreviate names. `getUserProfile` not `getUsrProf`. Clarity beats brevity.
- No commented-out code. Delete it. Git remembers.
- Handle errors explicitly. Don't swallow exceptions or ignore error returns.
- Keep files under 300 lines. If a file is growing, extract a module.
- Imports go at the top, grouped: stdlib, external packages, internal modules.

<!--
CUSTOMIZATION NOTES (this comment is stripped from context, it's for you):

This rule has no paths field, so it applies to all files every session.
Add language-specific rules for your stack:
- Python: Use type hints for all function signatures.
- TypeScript: Use explicit return types on exported functions.
- Go: Use the standard library over third-party packages when possible.
-->
