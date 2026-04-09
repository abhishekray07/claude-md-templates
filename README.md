# The CLAUDE.md Starter Kit

> Get better output from Claude Code in 5 minutes. No plugins, no MCP servers, no configuration — just markdown files in the right places.

---

## The 3-Level Hierarchy

Claude Code reads instructions from three locations. Each has a different scope:

```
~/.claude/CLAUDE.md          -> Global: your personal preferences (every project)
.claude/CLAUDE.md            -> Project: shared with your team (committed to git)
./CLAUDE.local.md            -> Local: your personal overrides (gitignored)
```

**Global** = rules you'd repeat across every project ("always run tests," "prefer simple code").
**Project** = context your whole team benefits from (stack, structure, commands, conventions).
**Local** = stuff only you need (your terminal, your MCP servers, your editor quirks).

## Quick Start

### 1. Create your global file (~2 min)

```bash
mkdir -p ~/.claude
cp global/CLAUDE.md ~/.claude/CLAUDE.md
```

Edit it with your personal preferences. Keep it under 15 lines.

### 2. Create your project file (~3 min)

```bash
mkdir -p .claude
# Pick the template closest to your stack:
cp project/nextjs-typescript.md .claude/CLAUDE.md   # Next.js / React / TypeScript
cp project/python-fastapi.md .claude/CLAUDE.md      # Python / FastAPI
cp project/generic.md .claude/CLAUDE.md              # Anything else
```

Fill in the blanks. Commit it to git so your team gets the same behavior.

### 3. Create your local overrides (~1 min)

```bash
cp local/local.md ./CLAUDE.local.md
echo "CLAUDE.local.md" >> .gitignore
```

Add your personal setup. This file lives at the project root (not inside `.claude/`) and is never shared.

### 4. Add rules for your codebase (~2 min, optional)

```bash
mkdir -p .claude/rules
cp rules/code-style.md .claude/rules/code-style.md
cp rules/testing.md .claude/rules/testing.md
# Edit placeholders like [your test command]
```

Rules are modular instruction files. Path-scoped rules only load when Claude works with matching files. See `principles.md` for when and why to use rules vs CLAUDE.md.

Your final setup should look like this:

```
your-project/
├── .claude/
│   ├── CLAUDE.md              # Project instructions (committed)
│   └── rules/
│       ├── code-style.md      # Always loaded
│       └── testing.md         # Loaded when working with test files
├── CLAUDE.local.md            # Your personal overrides (gitignored)
└── ...
```

## How It Works

Claude Code automatically reads these files at the start of every session. All discovered files are **concatenated into context** — they don't override each other. When instructions conflict, Claude sees the last one it read. Within each directory, `CLAUDE.local.md` loads after `CLAUDE.md`, so your personal notes get the final word at that level.

**Important:** Claude Code's system prompt already contains ~50 instructions. That's a third of the ~150-200 instruction limit frontier models can reliably follow. Your CLAUDE.md must be lean. Every line competes for attention.

**Auto memory:** Claude also maintains its own notes at `~/.claude/projects/<project>/memory/` — build commands it discovers, patterns from your corrections, debugging insights. These load at session start too. You don't need to put things in CLAUDE.md that Claude will learn on its own. Run `/memory` to see everything that's loaded. See `principles.md` for when to use CLAUDE.md vs auto memory.

## The Self-Improvement Loop

This is the single most impactful habit you can build:

After every correction you give Claude, end with:

> "Update CLAUDE.md so you don't make that mistake again."

Claude is good at writing rules for itself. Over time, your CLAUDE.md becomes a living document that gets smarter with every session.

## What Goes Where (Decision Guide)

| Rule | Where it goes | Why |
|------|--------------|-----|
| "Run tests after changes" | Global | You want this everywhere |
| "Use shadcn/ui components" | Project | Team convention |
| "I use Ghostty terminal" | Local | Only you need this |
| "Never use `any` in TypeScript" | Project | Team standard |
| "Ask before committing" | Global | Personal preference |
| "I have Context7 MCP configured" | Local | Your setup, not theirs |
| "Prices are in src/lib/config" | Project | Domain knowledge |
| "Keep code simple" | Global | Universal preference |

## Common Mistakes

**Too long.** If your project CLAUDE.md is over 80 lines, Claude starts ignoring parts of it. HumanLayer keeps theirs under 60 lines. That's a good benchmark.

**Personality instructions.** "Be a senior engineer" or "Think step by step" wastes tokens. Claude Code already has strong system-level instructions.

**@-mentioning docs.** Writing `@docs/api-guide.md` embeds the entire file into context every single session. Instead, *pitch* Claude on when to read it: "For Stripe integration issues, see docs/stripe-guide.md."

**Formatting rules without a formatter.** If you have a linter/formatter configured, use a hook to run it — don't burn CLAUDE.md lines on what a tool enforces. But if you *don't* have a formatter, concrete style rules like "Use 2-space indentation" are exactly the kind of specific instruction Anthropic recommends.

**Duplicate rules.** If your global file says "run tests" and your project file also says "run tests," you've wasted tokens saying the same thing twice.

## Scaling Up: Module-Specific Files

For larger codebases, you can place CLAUDE.md files in subdirectories:

```
.claude/CLAUDE.md              -> Project root (always loaded)
src/auth/CLAUDE.md             -> Auth module (loaded when working in src/auth/)
src/api/CLAUDE.md              -> API module (loaded when working in src/api/)
```

Claude Code loads these **on demand** — only when working in that directory. This keeps your root file lean while giving Claude deep context where it matters.

Use this when your root CLAUDE.md pushes past 80 lines, or when different parts of the codebase have different conventions.

## Scaling Up: Rules

For rules that only apply to specific parts of your codebase, use `.claude/rules/`:

```
.claude/rules/
  testing.md          -> Only activates for test files
  api-design.md       -> Only activates for API route handlers
  code-style.md       -> No path restriction = applies everywhere
```

Rules use YAML frontmatter with `paths` globs to scope when they activate. See `rules/` in this repo for annotated guides you can adapt to your stack, and `principles.md` for the full guide on writing effective rules.

**CLAUDE.md vs rules:** Use CLAUDE.md for project-wide context (stack, commands, structure). Use rules for focused standards that apply to specific file types or directories.

## Included Files

| File | What it is | When to use it |
|------|-----------|---------------|
| `global/CLAUDE.md` | Personal preferences template | Copy to `~/.claude/CLAUDE.md` |
| `project/nextjs-typescript.md` | Next.js/React/TS project template | Copy to `.claude/CLAUDE.md` |
| `project/python-fastapi.md` | Python/FastAPI project template | Copy to `.claude/CLAUDE.md` |
| `project/generic.md` | Fill-in-the-blank for any stack | Copy to `.claude/CLAUDE.md` |
| `local/local.md` | Personal overrides template | Copy to `.claude/local.md` |
| `rules/testing.md` | Testing standards (path-scoped) | Copy to `.claude/rules/testing.md`, edit placeholders |
| `rules/api-design.md` | API design standards (path-scoped) | Copy to `.claude/rules/api-design.md`, adjust paths |
| `rules/code-style.md` | Universal code style standards | Copy to `.claude/rules/code-style.md` |
| `workflows/self-improvement-rules.md` | Structured rules for planning, verification, self-correction | Paste sections into your CLAUDE.md |
| `workflows/prompting-patterns.md` | 11 copy-paste prompts from the Claude Code team | Use directly in your sessions |
| `cheatsheet.md` | One-page reference card | Bookmark or print |
| `principles.md` | The full research — why these patterns work | Read when you want to go deeper |

## The Principles (Go Deeper)

**`principles.md`** contains everything we know about writing effective CLAUDE.md files:

- The attention budget (why less is more)
- Advisory vs deterministic: CLAUDE.md vs hooks
- Anthropic's official include/exclude table
- `.claude/rules/` — path-scoped modular rules with YAML frontmatter
- Writing rules Claude actually follows (with bad/good examples)
- Emphasis keywords that actually work (`IMPORTANT`, `YOU MUST`)
- Module-specific CLAUDE.md files for scaling
- Progressive disclosure patterns and `@import` syntax
- Architecture diagrams (HumanLayer's pattern)
- The "Don't X, Do Y" rule
- Troubleshooting: "Claude isn't following my rules" diagnostic checklist
- Matt Pocock's plan loop
- Real-world benchmarks from HumanLayer, Boris Cherny, Cloudflare, ChrisWiles
- Skill activation mapping
- TODO priority systems
- Every anti-pattern and what to do instead

If you only read one file in this kit besides the templates, read that one.

## Sources

This starter kit is based on:
- [Anthropic — "Claude Code Best Practices"](https://code.claude.com/docs/en/best-practices) — Official guidance, include/exclude table, rules
- [Anthropic — "Effective Context Engineering"](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Context rot, attention budget, just-in-time context
- [Anthropic — "Memory and Project Configuration"](https://code.claude.com/docs/en/memory) — CLAUDE.md hierarchy, rules, auto memory
- [Anthropic — "Hooks Guide"](https://code.claude.com/docs/en/hooks-guide) — Deterministic lifecycle hooks
- [Boris Cherny's team tips](https://x.com/bcherny/status/2017742741636321619) — 10 tips from the Claude Code team (Jan 31, 2026)
- [Boris Cherny's personal setup](https://x.com/bcherny/status/2007179832300581177) — How the creator uses Claude Code (Jan 2, 2026)
- [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — Instruction limits, progressive disclosure, leverage diagram
- [Matt Pocock — "My AGENTS.md file"](https://www.aihero.dev/my-agents-md-file-for-building-plans-you-actually-read) — Plan loop rules
- [josix/awesome-claude-md](https://github.com/josix/awesome-claude-md) — Curated collection of real CLAUDE.md files from open-source projects
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — The main awesome list for Claude Code resources
- Real CLAUDE.md files from [HumanLayer](https://github.com/humanlayer/humanlayer/blob/main/CLAUDE.md), [Cloudflare](https://github.com/cloudflare/templates/blob/main/CLAUDE.md), [ChrisWiles](https://github.com/ChrisWiles/claude-code-showcase) (5.2k stars), and the community

## Future Work

- [ ] Before/after examples showing real CLAUDE.md files improved with these principles
- [ ] Stack-specific rule templates (Go, Rust, Ruby on Rails, Django)
- [ ] Migration guide: "You have a 200-line CLAUDE.md — here's how to split it into rules"
- [ ] Hooks cookbook: common lifecycle hooks paired with CLAUDE.md patterns
- [ ] Video walkthrough of setting up the full hierarchy from scratch

---

*Built by [Claude Code Camp](https://claudecodecamp.com) — a weekly newsletter teaching developers how to get 10x more from Claude Code.*
