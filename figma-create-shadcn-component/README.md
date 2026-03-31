# figma-create-shadcn-component

A Claude Code skill that builds Figma components from [ShadCN/ui](https://ui.shadcn.com) component specs, applying your design system's semantic tokens, fonts, and variables automatically.

## What it does

Give it a ShadCN component URL and a target Figma file, and it will:

1. Fetch the component spec and identify all variants, sizes, and properties
2. Check your Figma file for existing components, pages, and available icons
3. Scope the component for Figma — deciding what to include, what to simplify, and what to split
4. Build the component with proper variants, properties, and design system tokens
5. Run visual QA, set component documentation, and optionally create developer notes

## Prerequisites

- [Tailwind Design System - Semantic Tokens & Code Sync](https://www.figma.com/community/file/1526688065982358612) — Figma community library providing semantic color variables, Tailwind CSS primitives, and dual-font text styles
- [Figma Console MCP](https://docs.figma-console-mcp.southleft.com/) — MCP server for Claude Code providing Plugin API access, component management, variable binding, and visual validation

## How it works

### Design token mapping

ShadCN uses a flat token model (`--primary`, `--secondary`, `--destructive`). The skill maps these to a richer semantic system — `Brand/Default`, `Neutral/Tertiary`, `Danger/Default` — that supports light/dark modes, hover states, and additional semantics (Warning, Positive) that ShadCN doesn't cover. All color bindings use Figma variables, never hardcoded hex values.

Typography follows the same principle. Rather than mapping ShadCN's Tailwind utility classes directly (e.g., `text-sm font-medium`), the skill determines the semantic font by component context — *what the text is*, not what CSS class it uses. A button label uses the **Button** font style, a dialog title uses **Title Small**, a form label uses **Label**. This preserves the dual-font system where heading/UI fonts stay distinct from body/content fonts.

The full mapping tables live in `rules.md`.

### Complexity-aware scoping

Not every ShadCN component translates directly to Figma. Code components support unlimited dynamic children, conditional rendering, and composition patterns that don't map to Figma's component model. The skill handles this by assessing complexity upfront:

**Simple components** (buttons, badges, inputs, avatars) have a manageable number of variants and a clear structure. The skill builds these directly after a brief property architecture check.

**Complex components** (button groups, dialogs, data tables, navigation menus) have a combinatorial explosion of possible variants — styles x sizes x content patterns x item counts can produce hundreds of permutations. The skill scopes these down before building by walking through each dimension with you: which styles are needed, which sizes, how many slots, what conditional relationships exist between options. The goal is to identify what designers will actually use, not replicate every possible permutation.

This scoping also decides what becomes a Figma variant property (visible in the variant picker) vs a boolean toggle (show/hide a layer) vs a separate component entirely. Getting this mapping right is what makes the difference between a component designers reach for daily and one they avoid.

### Icon handling

When a ShadCN component includes icons, the skill searches your Figma file for matching icon components. If it can't find an exact match, it tries close alternatives (e.g., "arrow-down" for "chevron-down"). If nothing works, it uses a placeholder and tells you which icons need manual replacement.

### Visual QA

The skill takes screenshots after each major creation step and runs a final QA pass checking spacing, alignment, color token bindings, typography styles, and variant completeness. It uses `figma_lint_design` for accessibility checks.

### Component documentation

Every component gets a description set on the component set node — visible when designers hover it in the assets panel. This includes what it is, which ShadCN component it maps to, and notable deviations.

After building, the skill offers to create **Developer Notes** — a standalone reference frame on the same page aimed at engineers reviewing the design file. These summarise what's included, known gaps vs the ShadCN spec (framed as extensible, not permanent), and property differences (framed as design-file ergonomics, not spec changes). All text in the notes uses the file's text styles and color variables.

## Usage

In Claude Code, provide a ShadCN component URL and your target Figma file:

```
Create the button group component from https://ui.shadcn.com/docs/components/radix/button-group
in https://www.figma.com/design/YOUR_FILE_KEY/Your-File-Name
```

The skill handles the rest — fetching the spec, checking your file, scoping, building, QA, and reporting back.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Skill definition — the agent workflow across all phases |
| `rules.md` | Color and typography mapping rules — source of truth for token decisions |
| `references/scoping.md` | Detailed variant scoping and property architecture guide for complex components |
| `examples/button.md` | Reference example — how a simple component was scoped and built, with deviation rationale |
| `examples/button-group.md` | Reference example — how a complex component was scoped and built, with deviation rationale |

## Customisation

If you modify your design system (rename token groups, add new semantic colors, change fonts), update `rules.md` to reflect the changes. The skill reads this file at runtime, so changes take effect immediately.
