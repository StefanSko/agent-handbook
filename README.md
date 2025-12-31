# Agent Handbook

Shared configuration for AI coding assistants (Claude Code, Codex, etc.).

## Quick Start

```bash
# Add to your project as a submodule
git submodule add https://github.com/StefanSko/agent-handbook.git .agents

# Copy the template config files
cp .agents/setup/CLAUDE.md.template CLAUDE.md
cp .agents/setup/AGENTS.md.template AGENTS.md

# Create skill symlinks for tool discovery
mkdir -p .claude/skills .codex/skills
ln -s ../../.agents/skills/rust-style-python .claude/skills/
ln -s ../../.agents/skills/rust-style-python .codex/skills/
```

## Structure

```
your-project/
├── .agents/                  # This repo (submodule)
│   ├── handbook.md           # Main instructions
│   ├── docs/                 # Detailed guides
│   │   ├── code-style.md
│   │   ├── testing.md
│   │   ├── source-control.md
│   │   ├── python.md
│   │   ├── using-uv.md
│   │   └── docker-uv.md
│   ├── skills/               # Shared skills
│   │   └── rust-style-python/
│   └── setup/                # Templates
├── .agents-local/            # Project-specific (optional)
│   └── skills/
├── .claude/skills/           # Symlinks for Claude Code
├── .codex/skills/            # Symlinks for Codex
├── CLAUDE.md                 # Entry point for Claude Code
└── AGENTS.md                 # Entry point for Codex
```

## Adding Project-Local Skills

```bash
# Create local skills directory
mkdir -p .agents-local/skills/my-skill

# Create the skill
cat > .agents-local/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: What this skill does
---
# My Skill
Instructions here...
EOF

# Symlink for discovery
ln -s ../../.agents-local/skills/my-skill .claude/skills/
ln -s ../../.agents-local/skills/my-skill .codex/skills/
```

## Updating the Submodule

```bash
git submodule update --remote .agents
git add .agents
git commit -m "chore: update agent-handbook submodule"
```

## Cloning a Project with This Submodule

```bash
git clone --recurse-submodules <your-project-url>

# Or if already cloned:
git submodule update --init --recursive
```
