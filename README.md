# Claude Skills

A collection of custom skills for [Claude Code](https://claude.ai/claude-code) that extend its capabilities with specialized workflows.

## Skills

| Skill | Description |
|---|---|
| [figma-create-shadcn-component](./figma-create-shadcn-component/) | Build Figma components from ShadCN specs with design system token mapping |

## Installation

Each skill is self-contained in its own directory. To install a skill:

1. Copy the skill directory into `~/.claude/skills/`
2. Install any prerequisites listed in the skill's README
3. The skill will be available in all Claude Code sessions

```bash
# Example: install the figma-create-shadcn-component skill
cp -r figma-create-shadcn-component ~/.claude/skills/
```

## Structure

Each skill directory contains:

```
skill-name/
  SKILL.md     # Skill definition (required)
  README.md    # Documentation, prerequisites, design decisions
  rules.md     # Reference files (if needed)
  ...          # Any other companion files
```
