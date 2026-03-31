---
name: figma-create-shadcn-component
description: "Build a Figma component from a ShadCN spec with design system tokens. Use when the user shares a ShadCN component URL or mentions creating/building a ShadCN component in Figma."
context: fork
agent: general-purpose
argument-hint: "[shadcn-component-url]"
allowed-tools:
  - Read
  - WebFetch
  - mcp__figma-console__*
  - mcp__claude_ai_Figma__*
---

# Create ShadCN Component in Figma

Build a ShadCN component as a fully-styled Figma component using the design system's semantic tokens, fonts, and variables.

## Context for the agent

The agent has access to these skill files via `${CLAUDE_SKILL_DIR}`:
- `rules.md` — color token mappings, typography decision rules, and fallback guidance. **Load this at the start** — it governs every styling decision.
- `examples/` — if an example exists for the component being built (e.g., `examples/button.md`), read it before Phase 4. Examples document deliberate deviations and rationale that should inform scoping decisions.
- `references/scoping.md` — detailed variant scoping and property architecture guide. **Read only for complex components** (Phase 4a/4b).

The agent also receives:
- The ShadCN component URL (from `$ARGUMENTS` or the user)
- The target Figma file (from the user or the active figma-console connection)

---

## Agent Workflow

### Phase 1: Gather Context

**Read `rules.md`** from the skill directory. Load it first — it governs every color and typography decision.

**Fetch the ShadCN component spec** from the provided URL using WebFetch. Extract:
- All variants, sizes, and component properties
- Sub-components or compositions shown in examples
- Icon usage — which icons appear and where

### Phase 2: Check Figma File State

Use **figma-console MCP tools** for all Figma operations (preferred over standard Figma MCP). Fall back to `get_design_context` / `get_screenshot` only if figma-console is unavailable.

1. **Check for existing components** — `figma_search_components`. If found, ask the user whether to replace, update, or skip.
2. **Check for the component's page** — find or create a page named after the component. Place inside a Section, never on blank canvas.
3. **Find nested components** — locate sub-components the new component will need.
4. **Survey available icons** — `figma_search_components` and `figma_get_library_components`.

### Phase 3: Icon Handling

For each icon the component needs:
1. Search for an exact match by name using `figma_search_components`
2. If no exact match, try related terms (e.g., "close" for "x", "magnifying-glass" for "search")
3. If a reasonable match exists, use it via `figma_instantiate_component`
4. If nothing close exists, use any available icon as a placeholder. Tell the user which icons were substituted.

### Phase 4: Component Strategy

Assess complexity before building. The goal is to scope what gets built *before* any Figma work begins.

**Simple components** (buttons, badges, inputs, avatars): proceed directly to a brief property architecture check (Phase 4b only), then to creation.

**Complex components** (button groups, dialogs, data tables, command palettes, navigation menus, toolbars): **read `references/scoping.md`** and go through Phase 4a and 4b in full. Do not build anything until the user confirms the scoping spec.

**Always ignore RTL configuration** unless the user explicitly requests it.

#### Phase 4a: Variant Scoping (complex components only)

Follow the detailed guide in `references/scoping.md`. Walk through each dimension (styles, sizes, content patterns, item count, conditional relationships, separate components) with the user. Output a scoping summary and wait for confirmation.

#### Phase 4b: Property Architecture

Follow the guide in `references/scoping.md`. Map each scoped dimension to Figma's component model (variant properties, boolean properties, instance swaps, separate components). Output a property architecture spec and wait for confirmation before building.

### Phase 5: Create the Component

Build according to the confirmed scoping spec and property architecture. Follow the spec — don't improvise new variants or properties.

Use `figma_execute` for component creation (full Plugin API access).

**Color rules** (from rules.md):
- Map every ShadCN color token to the corresponding Figma variable from the Semantic Classes collection
- Use `figma.variables.setBoundVariableForPaint` to bind fills and strokes — never set raw hex values

**Typography rules** (from rules.md):
- Determine the semantic font by **component context first** (what the text IS), not by matching utility classes
- Apply text styles from the Semantic Fonts section rather than setting font properties manually

**Component structure:**
- Use auto-layout for all frames
- Create the variant grid first, then add boolean and instance swap properties to each variant
- For conditional defaults, set the appropriate default state in each variant
- Use `figma_instantiate_component` for nested components
- Name layers clearly — variant names should match the scoping spec
- Property names should be designer-friendly (e.g., "Show trailing icon", not "hasTrailingIcon")

**Separate components:** Build each independently with its own component set. Place on the same page as distinct component sets.

**After each significant creation step**, take a screenshot to verify visually.

**Component description (required):** Use `figma_set_description` on the component set. Include what it is, which ShadCN component it maps to, notable deviations, and the ShadCN docs URL as the Link field.

### Phase 6: Quality Assurance

Run a QA pass after building:

1. **Visual check** — screenshot the complete component set. Check spacing, alignment, colors, typography, icons, auto-layout behaviour.
2. **Property check** — toggle booleans, cycle variants, test text overrides.
3. **Spec comparison** — verify all documented variants are represented, sizes match proportions, interactive states are accounted for.
4. **Design system compliance** — `figma_lint_design` for accessibility and quality checks.
5. **Fix issues** — fix and re-screenshot to verify.

### Phase 7: Report

Present the finished component:
- Final screenshot
- All variants and properties created
- Icons that were substituted
- Deviations from ShadCN spec and reasoning
- Anything needing manual attention
- **Ask the user if they want Developer Notes** — a reference frame on the same page summarising what's included, known gaps, and property differences, aimed at engineers reviewing the design file.

### Phase 8: Developer Notes (if requested)

Create a **Developer Notes** frame on the same page as the component. This is a standalone reference frame (not part of the component set) for engineers.

#### Structure

Build via `figma_execute`. Frame: 600px wide, vertical auto-layout, 32px padding, 24px item spacing, `primaryAxisSizingMode: 'AUTO'`, `counterAxisSizingMode: 'FIXED'`.

Sections separated by 1px divider frames:

1. **Title** — `[Component Name] — Developer Notes`
2. **Intro** — 2-3 sentences: covers common configurations, not every permutation. Goal is a usable design asset, not a 1:1 spec mirror. Nothing limits the ShadCN component in code. May grow over time.
3. **What's included** — Bullet list of current scope (orientation, styles, sizes, slot count, key features).
4. **Known gaps vs ShadCN** — Intentional omissions framed as extensible ("can be added later"), not permanent. We're not recommending removing capabilities from the ShadCN component.
5. **Property differences** — Where Figma properties differ from ShadCN API. Frame as design-file ergonomics, not spec changes. Engineers don't need to match these in code.

#### Styling requirements

All text and colors **must** use the file's existing text styles and color variables — never hardcode font properties or hex values.

**Text styles** — `figma.getLocalTextStylesAsync()`, apply via `setTextStyleIdAsync()`:
- Section titles -> **Subtitle Large**
- Body text and bullets -> **Body Small**

**Color variables** — `figma.variables.setBoundVariableForPaint()`:
- Title fills -> `Content/Default/Primary`
- Body fills -> `Content/Default/Secondary`
- Divider fills -> `Edge/Default`

**Dividers** — 1px tall frames, `layoutSizingHorizontal: 'FILL'`, fill bound to `Edge/Default`.

Screenshot with `figma_capture_screenshot` to verify.

---

## Tool Reference

| Phase | Tools |
|---|---|
| Check file state | `figma_search_components`, `figma_get_library_components`, `figma_execute` |
| Find icons | `figma_search_components`, `figma_get_library_components` |
| Create component | `figma_execute`, `figma_instantiate_component`, `figma_set_fills`, `figma_set_strokes`, `figma_set_text`, `figma_set_description` |
| Bind variables | `figma_get_variables`, `figma_execute` |
| Visual QA | `figma_take_screenshot`, `figma_capture_screenshot`, `figma_lint_design` |
| Node manipulation | `figma_resize_node`, `figma_move_node`, `figma_rename_node`, `figma_create_child` |

If figma-console MCP is unavailable, fall back to:
- `mcp__claude_ai_Figma__get_design_context` for reading designs
- `mcp__claude_ai_Figma__get_screenshot` for screenshots
