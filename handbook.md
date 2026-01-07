# Agent Handbook

This handbook applies to both Claude Code and Codex (and other AI coding assistants).

Entry points:
- Claude Code: `CLAUDE.md`
- Codex: `AGENTS.md`

## Interaction

- Address me as "Stefan"
- If you're having trouble, stop and ask for help

## References

Detailed guidance in `docs/`:

- `docs/code-style.md` - Code style and conventions
- `docs/testing.md` - TDD practices
- `docs/source-control.md` - Git and version control
- `docs/python.md` - Python-specific guidance
- `docs/using-uv.md` - UV package manager
- `docs/docker-uv.md` - Docker with UV

## Skills

Shared skills in `.agents/skills/`:

- `rust-style-python/` - Type-safe Python patterns

## Project-Local Extensions

Projects can add their own docs and skills alongside this handbook:

```
project/
├── .agents/              # This handbook (submodule)
├── .agents-local/        # Project-specific extensions
│   ├── docs/             # Project-specific documentation
│   └── skills/           # Project-specific skills
├── .claude/skills/       # Symlinks for Claude Code discovery
│   ├── rust-style-python -> ../../.agents/skills/rust-style-python
│   └── my-skill -> ../../.agents-local/skills/my-skill
├── .codex/skills/        # Symlinks for Codex discovery
│   └── (same structure)
├── CLAUDE.md
└── AGENTS.md
```

**Why symlinks?** Claude Code looks for skills in `.claude/skills/` and Codex in `.codex/skills/`. Symlinks let both tools discover skills from shared and local locations.