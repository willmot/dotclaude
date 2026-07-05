# dotclaude

My global [Claude Code](https://claude.com/claude-code) configuration — the working
style, delegation system, and model routing that load into every session.

## Layout

```
CLAUDE.md          # global user memory  → ~/.claude/CLAUDE.md
agents/            # subagent definitions → ~/.claude/agents/
├── snr-code-engineer.md   # Opus   — semi-defined work needing judgment
├── jnr-code-engineer.md   # Sonnet — fully-specified mechanical work
├── qa-runner.md           # Sonnet — runs a verification matrix, reports pass/fail
└── web-runner.md          # Haiku  — cheap web fetch / extract grunt work
```

## The idea

A capability-tiered delegation system. The session lead (**Fable 5**, falling back to
**Opus** if Fable is unavailable) scopes, designs, and reviews. Execution is delegated
by *what's missing* from the task:

- decisions named **and** small, clean diff → **jnr** (cheap, mechanical)
- judgment needed, or a large/messy diff → **snr** (stronger model)
- choosing the approach *is* the work — design, architecture, security → stays on the lead

Verification is delegated the same way (`qa-runner`), and the whole thing is tuned for
context-cost discipline: keep the lead's context lean, cap what subagents return.

See [CLAUDE.md](CLAUDE.md) for the full routing rules.

## Install

On a fresh machine:

```bash
git clone https://github.com/willmot/dotclaude ~/Developer/dotclaude
cp ~/Developer/dotclaude/CLAUDE.md ~/.claude/CLAUDE.md
cp -r ~/Developer/dotclaude/agents ~/.claude/
```

Prefer symlinks if you want this repo to stay the source of truth (back up any existing
`~/.claude/CLAUDE.md` and `~/.claude/agents/` first). Restart Claude Code, then run
`/agents` to confirm the four agents registered.
