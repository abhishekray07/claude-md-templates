# CLAUDE.md Cheatsheet

## The Hierarchy

```
~/.claude/CLAUDE.md     Global     Your preferences          Every project    ~15 lines
.claude/CLAUDE.md       Project    Team context + rules       Committed        40-60 lines
./CLAUDE.local.md       Local      Your personal overrides    Gitignored       ~10 lines
```

## Project CLAUDE.md Sections (in order)

```markdown
## Project        <- Name + one sentence
## Stack          <- Language, framework, DB, deploy target
## Structure      <- Key directories only (5-7 lines)
## Commands       <- Copy-paste: dev, build, test, lint
## Verification   <- Steps to run before committing
## Conventions    <- 3-5 team rules
## Don't          <- Anti-patterns + what to do instead
```

## The Self-Improvement Loop

```
1. Claude makes a mistake
2. You correct it
3. You say: "Update CLAUDE.md so you don't make that mistake again"
4. Claude writes a rule for itself
5. You review and edit the rule
6. Repeat — CLAUDE.md gets smarter over time
```

## Commands to Know

| Command       | What it does                                    |
|---------------|------------------------------------------------|
| `/cost`       | Shows token usage for the current session       |
| `/context`    | Visualize current context usage as a colored grid |
| `/clear`      | Clears the session (start fresh)                |
| Shift+Tab x2  | Toggle Plan Mode (plan before acting)           |
| fn x2 (macOS) | Voice dictation (3x faster than typing)         |

## Do vs Don't

| Do | Don't |
|----|-------|
| Keep it under 60-80 lines | Write a 200-line manual |
| Copy-paste commands | Say "run the tests" vaguely |
| Point to docs: "see docs/auth.md" | @-mention docs (embeds entire file) |
| Say what to do instead | Only say "never do X" |
| Update after every mistake | Write it once and forget it |
| Use a linter for formatting rules | Put formatting rules in CLAUDE.md |
| Start lean, add when needed | Front-load every possible rule |

## Quick Copy: Verification Section

```markdown
## Verification
After every change, run in this order:
1. `[typecheck command]` — fix type errors
2. `[test command]` — fix failing tests
3. `[lint command]` — fix lint errors
4. `[build command]` — confirm it builds
```

## Quick Copy: Progressive Disclosure

```markdown
## References
- For auth flows or StripeErrors: see docs/stripe-guide.md
- For database migrations: see docs/migration-guide.md
- For deployment: see docs/deploy.md
```

---

*From the CLAUDE.md Starter Kit by [Claude Code Camp](https://claudecodecamp.com)*
