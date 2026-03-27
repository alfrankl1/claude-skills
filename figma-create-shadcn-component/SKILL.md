---
name: figma-create-shadcn-component
description: "Build a Figma component from a ShadCN component spec, applying design system tokens and semantic fonts. Use this skill when the user shares a ShadCN component URL (ui.shadcn.com/docs/components/...) and wants it created as a Figma component, or when they mention creating/importing/building a ShadCN component in Figma. Also trigger when the user says things like 'make me a button component from shadcn', 'import this shadcn component into figma', 'create the dialog component', or references any shadcn/ui component by name alongside Figma."
---

# Create ShadCN Component in Figma

This skill spawns an agent that takes a ShadCN component spec and builds it as a fully-styled Figma component using the design system's semantic tokens, fonts, and variables.

## How to invoke

Spawn a general-purpose agent with the full workflow below. The agent needs access to: WebFetch (to read the ShadCN spec), Read (to load rules.md), and all figma-console MCP tools.

Pass the agent:
- The ShadCN component URL from the user
- The path to `rules.md` in this skill's directory: `~/.claude/skills/figma-create-shadcn-component/rules.md`
- The target Figma file (from the user or from the active figma-console connection)

---

## Agent Workflow

### Phase 1: Gather Context

**Read the design system rules** from `rules.md` in this skill's directory. This file contains all color token mappings, typography decision rules, and fallback guidance. Load it at the start — it governs every styling decision.

**Fetch the ShadCN component spec** from the provided URL using WebFetch. Extract:
- All variants (e.g., default, destructive, outline, secondary, ghost, link for a button)
- All sizes (e.g., default, sm, lg, icon)
- Component properties and their types (boolean, enum, slot)
- Any sub-components or compositions shown in the examples
- Icon usage — which icons appear and where

### Phase 2: Check Figma File State

Use **figma-console MCP tools** for all Figma operations. These are the preferred toolset — they provide 90+ purpose-built tools with full Plugin API access via `figma_execute`, real-time screenshots via `figma_take_screenshot`, and component search via `figma_search_components`. Only fall back to the standard Figma MCP (get_design_context, get_screenshot) if figma-console is unavailable.

1. **Check for existing components** — use `figma_search_components` to see if this component (or parts of it) already exists. If it does, inform the user and ask whether to replace, update, or skip.

2. **Check for the component's page** — look for a page named after the component. If it doesn't exist, create one. Place the component inside a Section on this page, never on blank canvas.

3. **Find nested components** — use `figma_search_components` to locate any sub-components the new component will need (e.g., if building a Dialog, check if a Button component already exists to use inside it).

4. **Survey available icons** — use `figma_search_components` and `figma_get_library_components` to understand what icon components are available in the file or its libraries.

### Phase 3: Icon Handling

ShadCN components often include icons (chevrons, close buttons, search icons, etc.). For each icon the component needs:

1. **Search for an exact match** by name (e.g., "chevron-down", "x", "search", "settings") using `figma_search_components`
2. **If no exact match, search for close alternatives** — try related terms (e.g., "arrow-down" for "chevron-down", "close" for "x", "magnifying-glass" for "search", "cog" for "settings")
3. **If a reasonable match exists**, use it via `figma_instantiate_component`
4. **If nothing close exists**, use any available icon as a placeholder. Inform the user: list which icons were substituted and what the intended icon was, so they can manually swap them later

### Phase 4: Component Strategy

Assess the component's complexity before building it.

**Simple components** (buttons, badges, inputs, avatars — few variants, clear structure): proceed directly to creation. No need to check with the user.

**Complex components** (dialogs, data tables, command palettes, navigation menus — many variants, nested sub-components, or ambiguous structure): pause and present a strategy to the user before building. Include:

- **Variant vs property decisions** — What becomes a Figma variant (visually distinct states shown in the variant picker) vs a component property (toggleable options like "show icon", "show description")? As a rule of thumb: visual style differences (size, color scheme) are variants; optional content slots (icon, description, close button) are boolean properties.
- **Single vs multiple components** — If cramming everything into one component set would make it unwieldy (e.g., 50+ variants, deeply nested conditional content), recommend splitting. For example, a Dialog might be one component for the shell and separate components for DialogHeader, DialogFooter, DialogContent. Check with the user before splitting.
- **Usability recommendations** — Anything that would make the Figma component more practical for designers: sensible default values, clear property names, logical variant ordering.

Wait for the user to confirm or adjust the strategy before proceeding.

**Always ignore RTL configuration** unless the user explicitly requests it.

### Phase 5: Create the Component

Use `figma_execute` for component creation — it gives full access to the Figma Plugin API for creating frames, component sets, variants, auto-layout, and binding variables.

**Color rules** (from rules.md):
- Map every ShadCN color token to the corresponding Figma variable from the Semantic Classes collection
- Use `figma.variables.setBoundVariableForPaint` to bind fills and strokes to variables — never set raw hex values
- For states beyond ShadCN's model (warning, positive), use the extended tokens documented in rules.md

**Typography rules** (from rules.md):
- Determine the semantic font by **component context first** (what the text IS), not by matching ShadCN's utility classes
- A button label always uses the **Button** font style, a form label uses **Label**, table headers use **Label Bold**, etc.
- Apply text styles from the Semantic Fonts section rather than setting font properties manually
- If no semantic font fits, fall back to the Tailwind CSS Variables collection in Figma

**Component structure:**
- Use auto-layout for all frames
- Set up component properties (boolean toggles, instance swaps, text properties) to match the ShadCN component's API
- Create variants for visual style differences (variant, size)
- Use `figma_instantiate_component` for nested components (icons, sub-components found in Phase 2-3)
- Name layers clearly — variant names should match ShadCN's naming (e.g., "variant=destructive, size=lg")

**After each significant creation step**, take a screenshot with `figma_take_screenshot` to verify the result visually before continuing. This catches layout, spacing, and color issues early.

### Phase 6: Quality Assurance

After the component is fully built, run a QA pass:

1. **Visual check** — `figma_take_screenshot` of the complete component set. Review:
   - Spacing and alignment are consistent across variants
   - Colors match the design system tokens (not ShadCN defaults)
   - Typography uses semantic font styles
   - Icons are properly sized and aligned
   - Auto-layout behaves correctly (resizes, wraps as expected)

2. **Property check** — Verify each component property works:
   - Toggle boolean properties and confirm the right layers show/hide
   - Cycle through variants and confirm each is visually distinct
   - Check that text overrides work without breaking layout

3. **Spec comparison** — Compare the Figma component against the ShadCN spec:
   - All documented variants are represented
   - Sizes match the spec's proportions (padding, height, font size)
   - Interactive states (hover, focus, disabled) are accounted for where applicable

4. **Design system compliance** — Run `figma_lint_design` for accessibility and quality checks

5. **Fix issues** — If anything fails QA, fix it and re-screenshot to verify the fix

### Phase 7: Report

Present the finished component to the user:

- Show the final screenshot
- List all variants and properties created
- Note any icons that were substituted (and what the intended icon was)
- Note any deviations from the ShadCN spec and the reasoning
- Flag anything that needs the user's manual attention

---

## Tool Reference

These are the key figma-console MCP tools the agent should use at each phase:

| Phase | Tools |
|---|---|
| Check file state | `figma_search_components`, `figma_get_library_components`, `figma_execute` (to find/create pages) |
| Find icons | `figma_search_components`, `figma_get_library_components` |
| Create component | `figma_execute` (Plugin API), `figma_instantiate_component`, `figma_set_fills`, `figma_set_strokes`, `figma_set_text` |
| Bind variables | `figma_get_variables`, `figma_execute` (for `setBoundVariableForPaint`) |
| Visual QA | `figma_take_screenshot`, `figma_lint_design` |
| Node manipulation | `figma_resize_node`, `figma_move_node`, `figma_rename_node`, `figma_create_child` |

If figma-console MCP is unavailable, fall back to:
- `mcp__claude_ai_Figma__get_design_context` for reading designs
- `mcp__claude_ai_Figma__get_screenshot` for screenshots
- Standard Figma REST API tools for other operations
