# figma-create-shadcn-component

A Claude Code skill that builds Figma components from [ShadCN/ui](https://ui.shadcn.com) component specs, applying your design system's semantic tokens, fonts, and variables automatically.

## What it does

Give it a ShadCN component URL and a target Figma file, and it will:

1. Fetch the component spec and identify all variants, sizes, and properties
2. Check your Figma file for existing components, pages, and available icons
3. Create the component with proper Figma variants and component properties
4. Apply your design system tokens (not ShadCN defaults) for all colors and typography
5. Run visual QA and report the result

## Prerequisites

### 1. Tailwind Design System Figma Library

This skill is designed to work with the **Tailwind Design System - Semantic Tokens & Code Sync** Figma community file:

https://www.figma.com/community/file/1526688065982358612

This library provides:
- **Semantic color variables** (Surface, Edge, Brand, Neutral, Positive, Warning, Danger, Content) with Light/Dark modes
- **Tailwind CSS primitive variables** as a complete Figma variable collection
- **Semantic font styles** with a dual-font system (heading font + body font)
- **Design-to-code sync** — variables map directly to Tailwind CSS utility classes

The skill's `rules.md` contains the mapping between ShadCN tokens and this library's semantic variables. If you customise your library (rename groups, change token structure), update `rules.md` to match.

### 2. Figma Console MCP

This skill requires the **Figma Console MCP** server for Claude Code:

https://docs.figma-console-mcp.southleft.com/

Figma Console MCP provides 90+ purpose-built tools for design creation, component management, variable binding, and visual validation — including `figma_execute` for full Plugin API access and `figma_take_screenshot` for visual QA.

Install and configure it before using this skill. The standard Figma MCP is supported as a fallback but has significantly fewer capabilities.

### 3. Cookiecutter Starter Repo (Optional)

For engineers who want the full design-to-code pipeline, there's a companion cookiecutter repo that scaffolds a Tailwind + ShadCN project pre-configured with the same semantic token structure:

> *Coming soon — link will be added here when available*

This gives you a codebase where the Figma variables and the CSS variables are already aligned, so components built with this skill can be implemented in code with matching token names.

## Key Design Decisions

### Context-first typography

ShadCN components use raw Tailwind utility classes for fonts (e.g., `text-sm font-medium`). This skill does **not** blindly map those utilities. Instead, it determines the semantic font by asking *"what is this text?"*:

- A button label always uses the **Button** font style, regardless of what ShadCN's CSS says
- A dialog title uses **Title Small** or **Subtitle Large**
- A form label uses **Label** or **Label Bold**
- Helper text uses **Body Small**

This ensures the dual-font system is respected — heading/UI fonts stay distinct from body/content fonts.

### Design system tokens over ShadCN defaults

ShadCN uses a flat token model (`--primary`, `--secondary`, `--destructive`). This skill maps those to a richer semantic token system:

| ShadCN | Design System | Why |
|---|---|---|
| `--primary` | `Brand/Default` | Brand identity, not generic "primary" |
| `--destructive` | `Danger/Default` | Part of a full semantic set (also Warning, Positive) |
| `--secondary` | `Neutral/Tertiary` | Correct visual weight for low-emphasis actions |
| `--background` | `Surface/Default` | Named to avoid `bg-background` redundancy in Tailwind |
| `--border` | `Edge/Default` | Named to avoid `border-border` redundancy in Tailwind |
| `--muted` | `Surface/Secondary` | Maps to the right surface tier |

The design system also extends beyond ShadCN with tokens ShadCN doesn't have: `Warning`, `Positive`, `Brand/Secondary`, `Brand/Tertiary`, hover states, and semantic content colors.

### Smart complexity gating

Not all components need the same level of planning:

- **Simple components** (Button, Badge, Input, Avatar): the skill builds them immediately
- **Complex components** (Dialog, DataTable, Command, NavigationMenu): the skill pauses and proposes a component strategy — what becomes a variant vs a property, whether to split into multiple Figma components — and waits for your confirmation

### Graceful icon handling

When a ShadCN component includes icons, the skill searches your Figma file for matching icon components. If it can't find an exact match, it tries close alternatives (e.g., "arrow-down" for "chevron-down"). If nothing works, it uses any available icon as a placeholder and tells you which icons need manual replacement.

### Visual QA built in

The skill takes screenshots after each major creation step and runs a final QA pass checking spacing, alignment, color tokens, typography, and variant completeness. It uses `figma_lint_design` for accessibility checks.

## Usage

In Claude Code, provide a ShadCN component URL and your target Figma file:

```
Create the button group component from https://ui.shadcn.com/docs/components/radix/button-group
in https://www.figma.com/design/YOUR_FILE_KEY/Your-File-Name
```

The skill handles the rest — fetching the spec, checking your file, building the component, and reporting back.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Skill definition — the prompt and workflow the agent follows |
| `rules.md` | Color and typography mapping rules — the source of truth for token decisions |
| `README.md` | This file |

## Customisation

If you modify your design system (rename token groups, add new semantic colors, change fonts), update `rules.md` to reflect the changes. The skill reads this file at runtime, so changes take effect immediately.
