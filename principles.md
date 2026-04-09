# CLAUDE.md Principles

> Everything we know about writing effective CLAUDE.md files — from Anthropic's official guidance, HumanLayer's research, Boris Cherny's team, and the open-source community.

---

## The Attention Budget

Claude Code's system prompt already contains **~50 instructions**. Frontier models can reliably follow **~150-200 instructions** total. That means your CLAUDE.md gets roughly 100-150 instruction slots before things start getting ignored.

Worse: Claude Code's harness tells the model your CLAUDE.md *"may or may not be relevant."* If your file has too many instructions, Claude doesn't selectively ignore the bad ones — it starts ignoring **all of them** uniformly.

**The math is simple: every line you add makes every other line less likely to be followed.**

One more thing to internalize: **CLAUDE.md is advisory, not enforced.** Claude reads your instructions and tries to follow them, but there's no guarantee of strict compliance. If something must happen every time with zero exceptions (formatting, linting, security checks), use a [hook](https://code.claude.com/docs/en/hooks-guide) instead. Hooks are deterministic — they run shell commands at specific lifecycle points and are guaranteed to execute. CLAUDE.md is for guidance and judgment calls. Hooks are for rules that cannot be broken.

Source: [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [Anthropic — "Best Practices"](https://code.claude.com/docs/en/best-practices)

---

## What to Include vs Exclude (Anthropic Official)

This is from Anthropic's own best practices documentation:

| Include | Exclude |
|---------|---------|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred runners | Detailed API documentation (link to it instead) |
| Repository etiquette (branch naming, PRs) | Information that changes frequently |
| Architectural decisions specific to project | Long explanations or tutorials |
| Developer environment quirks | File-by-file descriptions of the codebase |
| Common gotchas or non-obvious behaviors | Self-evident practices like "write clean code" |

**The test:** Before adding a rule, ask: "Could Claude figure this out by reading my code?" If yes, don't add it.

Source: [Anthropic — "Claude Code Best Practices"](https://code.claude.com/docs/en/best-practices)

---

## Emphasis Keywords Work

Anthropic confirms that adding emphasis to critical rules improves adherence:

- `IMPORTANT:` for rules that must not be skipped
- `YOU MUST` for non-negotiable constraints
- `NEVER` for hard prohibitions (always pair with what to do instead)

Use these sparingly. If every rule is "IMPORTANT," none of them are.

Source: [Anthropic — "Claude Code Best Practices"](https://code.claude.com/docs/en/best-practices)

---

## The 3-Level Hierarchy

```
~/.claude/CLAUDE.md          -> Global (every project, ~15 lines)
.claude/CLAUDE.md            -> Project (committed to git, 40-60 lines)
./CLAUDE.local.md            -> Local (project root, gitignored, ~10 lines)
```

**Global:** Personal preferences that apply everywhere. "Ask before committing." "Run tests after changes." "Keep code simple."

**Project:** Team context. Stack, structure, commands, conventions. This is the high-leverage file — it turns Claude from a generic assistant into a teammate who knows your codebase.

**Local:** Your personal setup. Your terminal, your MCP servers, your editor quirks. Things your teammates don't need.

### Where Auto Memory Fits

Claude Code has a fourth memory layer you don't write — **auto memory**. When you correct Claude or it discovers something useful (build commands, debugging patterns, your preferences), it saves notes to `~/.claude/projects/<project>/memory/`. These notes are loaded at the start of every session alongside your CLAUDE.md files.

| | CLAUDE.md | `.claude/rules/` | Auto Memory |
|--|-----------|-------------------|-------------|
| **Who writes it** | You | You | Claude |
| **What it contains** | Instructions, project context | Scoped coding standards | Learnings, patterns, build commands |
| **Loaded** | Once in `messages[0]` | Re-injected on tool results | First 200 lines of `MEMORY.md` at session start |
| **Shared via git** | Yes (project), No (local/global) | Yes | No — machine-local only |

**What this means for your setup:**

- **Don't duplicate what Claude will learn.** If Claude discovers your test command by running it, it saves that to auto memory. You don't also need it in CLAUDE.md — unless you want the whole team to have it immediately.
- **CLAUDE.md is for the team, auto memory is for you.** Put shared standards in CLAUDE.md/rules. Let auto memory handle your personal workflow patterns.
- **They share the attention budget.** Auto memory's `MEMORY.md` is loaded alongside your CLAUDE.md. If both are long, you're consuming more of the instruction budget. Run `/memory` to audit what's loaded.
- **The self-improvement loop feeds auto memory.** When you say "remember that for next time," Claude writes to auto memory. When you say "update CLAUDE.md," it writes to the project file your team shares. Use the right one for the audience.

You can browse, edit, or delete auto memory files at any time — they're plain markdown in `~/.claude/projects/<project>/memory/`. Run `/memory` in a session to see everything that's loaded.

Source: [Anthropic — "How Claude remembers your project"](https://code.claude.com/docs/en/memory)

---

## Module-Specific CLAUDE.md Files (Advanced)

For larger codebases, you can place CLAUDE.md files in subdirectories:

```
.claude/CLAUDE.md              -> Project root (always loaded)
src/auth/CLAUDE.md             -> Auth module (loaded when working in src/auth/)
src/api/CLAUDE.md              -> API module (loaded when working in src/api/)
src/billing/CLAUDE.md          -> Billing module (loaded on demand)
```

Claude Code loads these **on demand** when it's working in that directory. This is the scaling strategy for monorepos and large codebases — keep the root file lean, push domain-specific rules into the modules that need them.

**When to use this:**
- Your root CLAUDE.md is pushing past 80 lines
- Different parts of the codebase have different conventions
- You have complex domain areas (auth, billing, data pipeline) with specific gotchas

**When NOT to use this:**
- Your project is small/medium (one CLAUDE.md is enough)
- Rules apply universally (those go in root)

---

## Organize Rules with `.claude/rules/`

For teams that outgrow a single CLAUDE.md, the `.claude/rules/` directory lets you split instructions into focused, modular files. Every `.md` file in this directory is automatically loaded into Claude's context. No configuration needed.

```
your-project/
├── .claude/
│   ├── CLAUDE.md           # Main project instructions (always loaded)
│   └── rules/
│       ├── code-style.md   # Universal code style (loaded every session)
│       ├── testing.md      # Testing conventions (loaded every session)
│       ├── security.md     # Security requirements (loaded every session)
│       └── frontend/
│           └── react.md    # React patterns (loaded every session)
```

### Path-Scoped Rules (Conditional Loading)

The real power: rules can be **scoped to specific files** using YAML frontmatter. These rules only load when Claude works with files matching the pattern — not every session.

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All API endpoints must include input validation
- Use the standard error response format from src/lib/errors.ts
- Include OpenAPI documentation comments
```

This rule only loads when Claude reads a file matching `src/api/**/*.ts`. It won't consume context when Claude is working in unrelated directories.

**Gotcha:** Path-scoped rules are triggered by the **Read tool only** — not by Write or Edit. If Claude edits a file without reading it first, the rule won't activate. Once triggered, rules stay in context for the rest of the session (they don't "scope out"). See [How Rules Are Actually Loaded](#how-rules-are-actually-loaded-under-the-hood) for the full mechanics.

Supported glob patterns:

| Pattern | Matches |
|---------|---------|
| `**/*.ts` | All TypeScript files in any directory |
| `src/**/*` | All files under `src/` |
| `*.md` | Markdown files in the project root |
| `src/components/*.tsx` | React components in a specific directory |
| `**/*.{ts,tsx}` | TypeScript and TSX files everywhere |

Rules without a `paths` field load unconditionally every session, just like CLAUDE.md. Note: use `paths` (not `globs`) for conditional loading — the `globs` key loads rules unconditionally at session start regardless of patterns. Anthropic's docs support multiple `paths` patterns and brace expansion (e.g., `"**/*.{ts,tsx}"`), though some users have reported inconsistent matching ([GitHub issue #45587](https://github.com/anthropics/claude-code/issues/45587)).

### When to Use Rules vs CLAUDE.md

| Use CLAUDE.md for | Use `.claude/rules/` for |
|-------------------|-------------------------|
| Project overview, stack, commands | Topic-specific conventions |
| Verification steps | Path-scoped rules for different areas |
| Instructions that apply everywhere | Rules that only matter in certain directories |
| Team onboarding context | Rules that different team members maintain |
| Context Claude needs once per session | Standards Claude needs reinforced near tool output |

The mental model: **CLAUDE.md is your onboarding brief. Rules are your team's coding standards handbook, split by chapter.** Under the hood, CLAUDE.md loads once (cheap, cached), while rules re-inject near tool output (higher attention, but higher token cost). Keep rule files short and few. See [How Rules Are Actually Loaded](#how-rules-are-actually-loaded-under-the-hood) for the mechanics.

### User-Level Rules

Personal rules in `~/.claude/rules/` apply to every project on your machine. Use them for preferences that aren't project-specific, like your preferred debugging workflow or code review habits. User-level rules load before project rules, so project rules take priority when they conflict.

Source: [Anthropic — "How Claude remembers your project"](https://code.claude.com/docs/en/memory)

---

## How Rules Are Actually Loaded (Under the Hood)

> **Caveat:** This section is sourced from community reports and GitHub issues, not official Anthropic documentation. The implementation details below were observed by users inspecting API traffic and may change between Claude Code versions. The official behavior is documented at [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory). When this section conflicts with official docs, trust the official docs.

### CLAUDE.md: Loaded Once in the First Message

Your CLAUDE.md content is placed into `messages[0]` — the very first user message — wrapped in a `<system-reminder>` block tagged `# claudeMd`. It sits alongside other context like the current date and available skills. Because it's at a fixed position in the conversation, it **benefits from prompt caching** (90% cost discount on Anthropic's API).

The three tiers load eagerly at session start:
- `~/.claude/CLAUDE.md` (user-level)
- `.claude/CLAUDE.md` or `./CLAUDE.md` (project-level)
- Subdirectory `CLAUDE.md` files load lazily via `nested_traversal` when Claude reads a file in that directory

### Rules: Re-Injected on Every Tool Result

This is the critical difference. `.claude/rules/` files are **re-injected as `<system-reminder>` blocks attached to tool call results** throughout the conversation. This has two major implications:

1. **Higher attention:** Rules appear fresh in context near the tool output Claude is processing — they get more attention than CLAUDE.md sitting at the top of a long conversation.
2. **Higher token cost:** Community reports show that 11 rule files (~6,200 tokens) re-injected across 30 tool calls consumed **~93,000 tokens — nearly half the 200K context window.** ([GitHub issue #32057](https://github.com/anthropics/claude-code/issues/32057))

Here's the real token breakdown from a typical session:

| Component | Tokens | % of 200K Context |
|-----------|-------:|-------------------:|
| Initial load (system prompt + CLAUDE.md + rules) | ~43K | 21% |
| Rule re-injections (~30 tool calls) | ~93K | 46% |
| Actual conversation content | ~50K | 25% |

**The takeaway:** Every rule file you add has a multiplied cost. One extra 500-token rule file doesn't cost 500 tokens — it costs 500 × (number of tool calls in your session). Keep your rule count low and each file focused.

### Path-Scoped Rules: Trigger on Read Only

Path-scoped rules (with `paths:` frontmatter) activate when Claude **reads** a file matching the glob pattern. Important gotchas:

- **Only the Read tool triggers loading.** Write, Edit, and MultiEdit do **not** trigger path-scoped rules. If Claude edits a file without reading it first, the rule won't load. ([GitHub issue #23478](https://github.com/anthropics/claude-code/issues/23478))
- **Once loaded, rules never "scope out."** A rule triggered by reading `src/api/handler.ts` stays in context for the rest of the session, even when Claude moves to unrelated files.
- **Multiple `paths:` patterns:** Anthropic's docs show multiple patterns and brace expansion working (e.g., `"src/**/*.{ts,tsx}"`). However, some users have reported inconsistent matching with multiple patterns ([GitHub issue #45587](https://github.com/anthropics/claude-code/issues/45587)). If your rule isn't triggering, try consolidating patterns.
- **`globs:` is not the same as `paths:`.** Using `globs:` in frontmatter loads the rule **unconditionally at session start** regardless of the patterns. Use `paths:` for conditional loading.

### Parent Directory Inheritance

Claude Code walks up from your working directory to the filesystem root, loading `.claude/rules/` and `CLAUDE.md` from every ancestor directory. In a monorepo, a sub-project inherits all ancestor rules. To exclude irrelevant ancestor files, use `claudeMdExcludes` in `.claude/settings.local.json` with glob patterns matching the files to skip. ([Anthropic docs](https://code.claude.com/docs/en/memory), [GitHub issue #34209](https://github.com/anthropics/claude-code/issues/34209))

### System Prompt Always Wins

Claude Code's built-in system prompt instructions take precedence over your CLAUDE.md and rules when they conflict. Don't try to override built-in behaviors like tool usage patterns or safety guidelines — your instructions will be silently ignored. ([GitHub issue #38491](https://github.com/anthropics/claude-code/issues/38491))

### Practical Implications

| Decision | Recommendation | Why |
|----------|---------------|-----|
| Where to put project context | CLAUDE.md | Loaded once, cached, cheap |
| Where to put coding standards | `.claude/rules/` | Higher attention near tool output |
| How many rule files | 3-5 max | Each file has multiplied token cost |
| Path-scoped vs always-loaded | Path-scoped when possible | Limits re-injection to relevant sessions |
| Long rule vs short rule | Short (under 30 lines) | Re-injected on every tool call |

Source: [GitHub issue #32057](https://github.com/anthropics/claude-code/issues/32057), [GitHub issue #44045](https://github.com/anthropics/claude-code/issues/44045), [GitHub issue #23478](https://github.com/anthropics/claude-code/issues/23478), [GitHub issue #45587](https://github.com/anthropics/claude-code/issues/45587), [GitHub issue #34209](https://github.com/anthropics/claude-code/issues/34209)

---

## Writing Rules Claude Actually Follows

Rules are only useful if Claude follows them. Here's what the evidence says works.

### Be Specific Enough to Verify

Claude follows concrete instructions better than vague ones. For each rule, ask: "Could I check whether Claude followed this by looking at the output?"

```markdown
# Bad — too vague to verify
- Write good error handling

# Good — specific and verifiable
- Catch specific exceptions, not bare `Exception`. Log the full context: what was
  attempted, with what arguments, for which user. Return a user-facing message.
```

### Use Emphasis Keywords Sparingly

Anthropic confirms that emphasis improves adherence — but only when used sparingly:

- `IMPORTANT:` for rules that must not be skipped
- `YOU MUST` for non-negotiable constraints
- `NEVER` for hard prohibitions (always pair with what to do instead)

If every rule is "IMPORTANT," none of them are. Reserve emphasis for 2-3 rules that truly matter. Everything else should stand on its own clarity.

### Always Provide the Alternative

Every prohibition needs a replacement. This is the "Don't X, Do Y" rule applied to rule files:

```markdown
# Bad — Claude doesn't know what to do
- NEVER use `any` in TypeScript

# Good — Claude knows exactly what to do
- NEVER use `any` in TypeScript — use `unknown` and narrow the type with
  type guards or assertions
```

### One Topic Per Rule File

Each file in `.claude/rules/` should cover one topic. `testing.md` covers testing conventions. `api-design.md` covers API patterns. Don't create a single `everything.md` — that defeats the purpose of modular rules.

Keep individual rule files under 30 lines. Rule files are re-injected on every tool call (see [How Rules Are Actually Loaded](#how-rules-are-actually-loaded-under-the-hood)), so every line has a multiplied token cost. If a rule file is getting long, it's probably covering multiple topics.

### The Total Budget Still Applies — And Rule Files Cost More Than You Think

Splitting instructions across CLAUDE.md and multiple rule files doesn't give you more instruction slots. Anthropic recommends keeping each CLAUDE.md file under 200 lines. HumanLayer's community research suggests frontier models reliably follow ~150-200 total instructions — beyond that, adherence drops uniformly.

But with rule files, the cost is **multiplied.** Unlike CLAUDE.md (loaded once in `messages[0]`), rule files are re-injected as `<system-reminder>` blocks on every tool call. A 500-token rule file doesn't cost 500 tokens — it costs 500 × (number of tool calls). In a real session with 30 tool calls and 11 rule files, re-injections consumed **93K tokens — 46% of the context window.** ([GitHub issue #32057](https://github.com/anthropics/claude-code/issues/32057))

**Target: 3-5 rule files, each under 30 lines. Keep root CLAUDE.md under 60 lines.** Use path-scoping aggressively so rules only load when relevant.

Be ruthless. For each rule in each file, ask: "Would Claude make a mistake without this?" If Claude already does it correctly, delete the rule.

Source: [Anthropic — "Best Practices"](https://code.claude.com/docs/en/best-practices), [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [GitHub issue #32057](https://github.com/anthropics/claude-code/issues/32057)

---

## Progressive Disclosure

Don't stuff everything into CLAUDE.md. Keep task-specific docs in separate files and tell Claude *when* to read them:

```markdown
## References
- For auth flows or StripeErrors: see docs/stripe-guide.md
- For database migrations: see docs/migration-guide.md
- For deployment: see docs/deploy.md
```

This is **dramatically cheaper** than `@docs/stripe-guide.md`, which embeds the entire file into context every single session whether Claude needs it or not.

The mental model: CLAUDE.md is the index. The docs/ folder is the library. Claude pulls books off the shelf when it needs them.

### @import Syntax

CLAUDE.md files can also import other files directly using `@path/to/file` syntax. Imported files are expanded and loaded into context at launch alongside the CLAUDE.md that references them.

```markdown
See @README.md for project overview and @package.json for available npm commands.

# Additional Instructions
- Git workflow: @docs/git-instructions.md
- Personal overrides: @~/.claude/my-project-instructions.md
```

Use this for files Claude needs **every session** (like a README or package.json). For domain-specific docs that are only sometimes relevant, the "pitch" approach above is still better.

Source: [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [Shrivu Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature), [Anthropic — "How Claude remembers your project"](https://code.claude.com/docs/en/memory)

---

## The Self-Improvement Loop

From Boris Cherny's 10 tips from the Claude Code team (tip #3):

> "Invest in your CLAUDE.md. After every correction, end with: 'Update your CLAUDE.md so you don't make that mistake again.' Claude is eerily good at writing rules for itself. Ruthlessly edit your CLAUDE.md over time. Keep iterating until Claude's mistake rate measurably drops."

This turns CLAUDE.md from a static config into a **living document that gets smarter over time**. One engineer on the team tells Claude to maintain a notes directory for every task, updated after every PR, then points CLAUDE.md at it.

The cycle:
1. Claude makes a mistake
2. You correct it
3. You say: "Update CLAUDE.md so you don't make that mistake again"
4. Claude writes a specific rule
5. You review and edit the rule (make it sharper)
6. The mistake never happens again

Over weeks, your CLAUDE.md accumulates the team's institutional knowledge.

Source: [Boris Cherny's team tips, tip #3](https://x.com/bcherny/status/2017742747067945390) (from the [full thread](https://x.com/bcherny/status/2017742741636321619))

---

## The Verification Loop

Boris's #1 tip for quality: **give Claude a way to verify its own work.**

> "Probably the most important thing to get great results out of Claude Code — give Claude a way to verify its work. If Claude has that feedback loop, it will 2-3x the quality of the final result." — Boris Cherny ([source](https://x.com/bcherny/status/2007179832300581177))

Anthropic's official best practices agree: "Include tests, screenshots, or expected outputs so Claude can check itself. This is the single highest-leverage thing you can do." ([source](https://code.claude.com/docs/en/best-practices))

Put your verification commands in CLAUDE.md so Claude can run them:

```markdown
## Verification
After every change, run in this order:
1. `npx tsc --noEmit` — fix type errors
2. `npm test` — fix failing tests
3. `npm run lint` — fix lint errors
4. `npm run build` — confirm it builds
```

When Claude has a feedback loop, it produces **2-3x better results**. It will run tests, find issues, and fix them before you even review.

Source: [Boris Cherny's personal setup thread](https://x.com/bcherny/status/2007179832300581177), [Anthropic Best Practices](https://code.claude.com/docs/en/best-practices)

---

## Architecture Diagrams (HumanLayer's Pattern)

HumanLayer uses ASCII architecture flow diagrams in their CLAUDE.md:

```
Claude Code -> MCP Protocol -> hlyr -> JSON-RPC -> hld -> Cloud API
```

This is a cheap way to give Claude spatial understanding of your system. One line of ASCII replaces paragraphs of explanation. Especially useful for:
- Monorepos with multiple services
- Request/response flows
- Data pipelines
- Module dependency chains

Source: [HumanLayer's CLAUDE.md](https://github.com/humanlayer/humanlayer/blob/main/CLAUDE.md)

---

## The "Don't X, Do Y" Rule

Every prohibition should include the alternative:

```markdown
# Bad — Claude doesn't know what to do instead
- Don't use `any` in TypeScript

# Good — Claude knows exactly what to do
- Don't use `any` — use `unknown` and narrow the type
```

Negative-only instructions leave Claude guessing. Always provide the positive alternative.

---

## Never Send an LLM to Do a Linter's Job

These do NOT belong in CLAUDE.md:
- "Use 2-space indentation"
- "Always add trailing commas"
- "Sort imports alphabetically"
- "Use single quotes"

Use a linter/formatter for deterministic rules. Save CLAUDE.md for judgment calls that require understanding context.

Source: [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## Matt Pocock's Plan Loop

A tiny addition that changes planning behavior:

```markdown
## Plan Mode
- Make the plan extremely concise. Sacrifice grammar for the sake of concision.
- At the end of each plan, give me a list of unresolved questions to answer, if any.
```

The "unresolved questions" pattern forces Claude to surface what it doesn't know before proceeding. Two lines, massive impact.

Source: [Matt Pocock — "My AGENTS.md file"](https://www.aihero.dev/my-agents-md-file-for-building-plans-you-actually-read)

---

## Real-World Benchmarks

| Source | Lines | Key Pattern |
|--------|-------|-------------|
| [HumanLayer](https://github.com/humanlayer/humanlayer/blob/main/CLAUDE.md) | 57 | ASCII diagrams, TODO priority system, progressive disclosure |
| [Boris Cherny's team](https://x.com/bcherny/status/2007179832300581177) | ~83 | Self-improvement, plan mode, verification loop (shared team CLAUDE.md, private repo) |
| [ChrisWiles](https://github.com/ChrisWiles/claude-code-showcase/blob/main/CLAUDE.md) | 80 | Quick Facts block, skill activation mapping |
| [Cloudflare](https://github.com/cloudflare/templates/blob/main/CLAUDE.md) | 230 | Enterprise monorepo — too long for most projects |
| [Anthropic's example](https://code.claude.com/docs/en/best-practices) | ~10 | Deliberately tiny — just code style + workflow |

The sweet spot: **40-80 lines** for a single project. Under 60 is ideal.

---

## Skill Activation Mapping (Advanced)

From ChrisWiles — tell Claude which skill to load for which task:

```markdown
## Skills
- Creating tests -> use `testing-patterns` skill
- Building forms -> use `formik-patterns` skill
- GraphQL operations -> use `graphql-schema` skill
- Debugging issues -> use `systematic-debugging` skill
```

This is lightweight progressive disclosure that costs almost nothing in context.

Source: [ChrisWiles/claude-code-showcase](https://github.com/ChrisWiles/claude-code-showcase)

---

## TODO Priority System (HumanLayer's Pattern)

Standardize how TODOs are annotated across the team:

```markdown
## TODO Annotations
- `TODO(0)`: Critical — never merge with these
- `TODO(1)`: High — architectural flaws, major bugs
- `TODO(2)`: Medium — minor bugs, missing features
- `TODO(3)`: Low — polish, tests, documentation
- `TODO(4)`: Questions/investigations needed
- `PERF`: Performance optimization opportunities
```

Claude will use these consistently when writing code if they're in your CLAUDE.md.

Source: [HumanLayer's CLAUDE.md](https://github.com/humanlayer/humanlayer/blob/main/CLAUDE.md)

---

## Anti-Patterns

| Anti-pattern | Why it fails | What to do instead |
|---|---|---|
| Relying on `/init` without editing | The generated file is generic and often too long ([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)) | Use `/init` as a starting point ([Anthropic recommends it](https://code.claude.com/docs/en/best-practices)), then ruthlessly edit down |
| Over 100 lines | Instructions get ignored uniformly | Keep root under 60, use module-specific files |
| Personality instructions | "Be a senior engineer" wastes tokens | Claude already has strong system-level behavior |
| @-mentioning docs | Embeds entire file every session | Pitch Claude on *when* to read: "For X, see docs/Y.md" |
| Code snippets | They go stale fast | Use file:line references instead |
| Formatting rules | Linters are deterministic, LLMs aren't | Use prettier/eslint/ruff |
| Negative-only rules | Claude doesn't know what to do | "Don't X — do Y instead" |
| Duplicate rules | Wastes attention budget | Each rule in exactly one place |
| Treating it as documentation | It's an onboarding brief, not a wiki | Keep it lean, link to docs/ |

---

## Troubleshooting: Claude Isn't Following My Rules

"Claude ignores my CLAUDE.md" is the #1 complaint in the community. Here's a diagnostic checklist based on real failure modes.

### 1. Your file is too long

This is the most common cause. Anthropic recommends keeping each CLAUDE.md file under 200 lines and splitting larger instruction sets into imports or `.claude/rules/`. Community research from HumanLayer suggests frontier models reliably follow ~150-200 total instructions — beyond that, adherence drops uniformly, not selectively. Run `/memory` to see what's loaded.

**Fix:** Keep root CLAUDE.md under 60 lines (HumanLayer's benchmark). Anthropic says to target under 200 lines per file. For each rule, ask: "Would Claude make a mistake without this?" If not, delete it.

### 2. Your rules conflict with each other

If your CLAUDE.md says "use Prettier" and a rule file says "use ESLint --fix for formatting," Claude picks one arbitrarily. Review all loaded files for contradictions.

**Fix:** Run `/memory` to see every loaded file. Search for overlapping topics. Each instruction should live in exactly one place.

### 3. Your rules are too vague

"Write clean code" and "handle errors properly" are not instructions Claude can follow consistently. They're aspirations. Claude needs concrete, verifiable rules.

**Fix:** Rewrite vague rules as specific, checkable statements. "Catch specific exceptions, not bare `Exception`" beats "handle errors properly."

### 4. Context window is full

In long sessions, the context window fills with conversation, file contents, and command output. When this happens, Claude's adherence to instructions in conversation can drop. Note: CLAUDE.md and `.claude/rules/` files are not affected by compaction — they're re-read from disk every time. But instructions you gave *in conversation* (not in files) will be lost after `/compact`.

**Fix:** Put durable instructions in CLAUDE.md or rule files, not in chat messages. Run `/clear` between unrelated tasks. For long sessions, use `/compact` to free space — your file-based instructions will survive.

### 5. You need a hook, not a rule

If a rule must be followed 100% of the time with zero exceptions, CLAUDE.md is the wrong mechanism. CLAUDE.md is advisory. Hooks are deterministic.

**Fix:** Convert non-negotiable rules to hooks. Auto-formatting after edits, blocking writes to protected files, running linters — these should be hooks in `.claude/settings.json`, not lines in CLAUDE.md.

### 6. Claude can't find your file

CLAUDE.md and rule files must be in the right location. Files in the wrong directory won't be loaded.

**Fix:** Run `/memory` in Claude Code to see exactly which files are loaded. Verify your CLAUDE.md is at `./CLAUDE.md` or `./.claude/CLAUDE.md`, and rules are in `.claude/rules/`. Check that path-scoped rules have correct glob patterns in their frontmatter.

### 7. Too many rule files are eating your context

If you have many rule files, they get re-injected as `<system-reminder>` blocks on every tool call result. Community reports show 11 rule files consumed ~93K tokens (46% of the 200K context) over a 30-tool-call session. Your conversation runs out of room for actual work.

**Fix:** Consolidate to 3-5 rule files max. Use path-scoping so rules only load when relevant. Keep each rule file under 30 lines. See [How Rules Are Actually Loaded](#how-rules-are-actually-loaded-under-the-hood) for the full token math.

### 8. Your rule conflicts with Claude's built-in system prompt

Claude Code's system prompt includes strong instructions about tool usage, safety, and code patterns. When your CLAUDE.md or rules conflict with these built-in instructions, the system prompt wins — silently.

**Fix:** Don't try to override built-in behaviors (like "always use the Read tool before editing"). Instead, work with the grain. If Claude keeps doing something despite your instructions, it may be following system prompt instructions that take priority.

### 9. Path-scoped rules aren't triggering

Your rule has `paths:` frontmatter but Claude isn't following it. Common causes: Claude used Write/Edit without first reading the file (only Read triggers path-scoped rules), you used `globs:` instead of `paths:` (which loads unconditionally), or your glob patterns don't match the files Claude is accessing.

**Fix:** Ensure `paths:` (not `globs:`) is your frontmatter key. Test your glob patterns against actual file paths. If Claude needs to follow a rule when editing, the file should be read first — or consider making the rule always-loaded instead. If multiple patterns aren't matching, try brace expansion (`"**/*.{ts,tsx}"`) or consolidate patterns.

### 10. Inherited rules from parent directories

In a monorepo, Claude walks up from your working directory to the filesystem root, loading `.claude/rules/` and `CLAUDE.md` from every ancestor directory. Rules you didn't write may be loading into your sub-project.

**Fix:** Run `/memory` to see all loaded files, including inherited ones. Use `claudeMdExcludes` in `.claude/settings.local.json` to skip specific ancestor files:

```json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/path/to/other-team/.claude/rules/**"
  ]
}
```

Source: [Anthropic — "How Claude remembers your project"](https://code.claude.com/docs/en/memory), [GitHub issue #668](https://github.com/anthropics/claude-code/issues/668), [GitHub issue #19635](https://github.com/anthropics/claude-code/issues/19635), [GitHub issue #32057](https://github.com/anthropics/claude-code/issues/32057), [GitHub issue #34209](https://github.com/anthropics/claude-code/issues/34209), [GitHub issue #38491](https://github.com/anthropics/claude-code/issues/38491)

---

## Further Reading

- [Anthropic — "How Claude remembers your project"](https://code.claude.com/docs/en/memory) — Official docs on CLAUDE.md, `.claude/rules/`, auto memory, @imports
- [Anthropic — "Claude Code Best Practices"](https://code.claude.com/docs/en/best-practices) — Official guidance, include/exclude table, emphasis keywords
- [Anthropic — "Effective Context Engineering for AI Agents"](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Context rot, attention budget, just-in-time context
- [Anthropic — "Automate workflows with hooks"](https://code.claude.com/docs/en/hooks-guide) — Deterministic enforcement, when to use hooks instead of rules
- [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — Instruction limits, progressive disclosure, leverage diagram
- [Boris Cherny's team tips](https://x.com/bcherny/status/2017742741636321619) — 10 tips from the Claude Code team (Jan 31, 2026)
- [Boris Cherny's personal setup](https://x.com/bcherny/status/2007179832300581177) — How the creator uses Claude Code (Jan 2, 2026)
- [Boris Cherny's setup (summary)](https://paddo.dev/blog/how-boris-uses-claude-code/) — paddo.dev's analysis of Boris's workflow
- [Shrivu Shankar — "How I Use Every Claude Code Feature"](https://blog.sshh.io/p/how-i-use-every-claude-code-feature) — Practitioner deep dive
- [josix/awesome-claude-md](https://github.com/josix/awesome-claude-md) — Curated collection of real CLAUDE.md files from open-source projects
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — The main awesome list for Claude Code resources
- [Matt Pocock — "My AGENTS.md file"](https://www.aihero.dev/my-agents-md-file-for-building-plans-you-actually-read) — Plan loop rules
- [GitHub issue #32057](https://github.com/anthropics/claude-code/issues/32057) — Rule re-injection on tool calls, token cost analysis
- [GitHub issue #44045](https://github.com/anthropics/claude-code/issues/44045) — `messages[0]` structure showing `<system-reminder>` content blocks
- [GitHub issue #23478](https://github.com/anthropics/claude-code/issues/23478) — Path-scoped rules only trigger on Read, not Write/Edit
- [GitHub issue #45587](https://github.com/anthropics/claude-code/issues/45587) — Multiple `paths:` patterns, `globs:` vs `paths:` behavior
- [GitHub issue #34209](https://github.com/anthropics/claude-code/issues/34209) — Parent directory traversal and rule inheritance

---

*From the CLAUDE.md Starter Kit by [Claude Code Camp](https://claudecodecamp.com)*
