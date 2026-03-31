# Example: Button Group Component

ShadCN spec: https://ui.shadcn.com/docs/components/radix/button-group
Figma reference: Appcues Tailwind Design System → Button Group component set (node 3158:11275)

This example demonstrates how a complex ShadCN component with a large permutation space gets scoped down to a practical Figma component. Unlike simple components (e.g. Button), complex components require a **variant scoping phase** before building — the skill walked through structured alignment with the designer to decide what to include, what to omit, and what to split into separate components.

---

## ShadCN Dev Spec

### Sub-components

| Component | Purpose |
|-----------|---------|
| `ButtonGroup` | Outer container, `role="group"` |
| `ButtonGroupSeparator` | Visual divider between buttons |
| `ButtonGroupText` | Text slot (supports `asChild`) |

Standard `<Button>` components are placed inside as children.

### Props

| Component | Prop | Type | Values | Default |
|-----------|------|------|--------|---------|
| `ButtonGroup` | `orientation` | enum | `horizontal`, `vertical` | `horizontal` |
| `ButtonGroupSeparator` | `orientation` | enum | `horizontal`, `vertical` | `vertical` |
| `ButtonGroupText` | `asChild` | boolean | `true`, `false` | `false` |

### Style inheritance

ButtonGroup has no `variant` or `size` prop — it inherits styling from whatever variant and size are set on its child `<Button>` components. Any button variant can be used inside a group.

### Separator logic

From the docs: "Buttons with variant `outline` do not need a separator since they have a border. For other variants, a separator is recommended to improve the visual hierarchy."

### Compositions demonstrated in docs

The ShadCN docs show 11 example patterns: basic, orientation, size, nested groups, separator, split button, input integration, input group, dropdown menu, select, and popover. Most of these are compositions (mixed element types in a group), not variants of a single component.

---

## Why This Component Required Variant Scoping

The Button Group is the kind of component where a naive 1:1 port would be impractical. Consider the permutation space:

- **Styles:** ShadCN supports any button variant (default, destructive, outline, ghost, secondary, link) → 6+ options
- **Sizes:** sm, default, lg, icon → 4 options
- **Content patterns per button:** text-only, leading icon + text, text + trailing icon, icon-only → 4 patterns
- **Action count:** 2, 3, 4, 5+ buttons in a group → open-ended
- **Orientation:** horizontal, vertical → 2 options
- **Separator:** present or absent, color varies by style → conditional

Even a modest combination of these dimensions produces hundreds of variants. For Figma, the question isn't "what *can* the component do?" — it's "what will designers *actually use* in their files?"

The skill's Phase 4a (Variant Scoping) walked through each dimension with the designer, proposing defaults and letting them cut. The result was a focused component that covers everyday usage without the overhead of managing hundreds of variants.

---

## Figma Component (as built)

### Property architecture

| Property | Figma type | Values | Default |
|----------|------------|--------|---------|
| `style` | Variant | `outline`, `secondary` | `outline` |
| `size` | Variant | `default`, `sm` | `default` |
| `Show action 3` | Boolean | `true`, `false` | `false` |
| `Show action 4` | Boolean | `true`, `false` | `false` |
| `Show leading icon button` | Boolean | `true`, `false` | `false` |
| `Show trailing icon button` | Boolean | `true`, `false` | `false` |

### Variant grid

2 styles × 2 sizes = **4 physical variants**. All other flexibility is handled via boolean properties and nested component overrides, keeping the grid small.

### Internal structure per variant

Each variant is a horizontal auto-layout frame with `clipsContent: true` and `cornerRadius: 6`:

```
[Leading icon button] [Leading sep] [Action 1] [Sep 1] [Action 2] [Sep 2] [Action 3] [Sep 3] [Action 4] [Trailing sep] [Trailing icon button]
```

- **Action 1–4**: Nested instances of the existing **Button** component (Outline or Secondary variant, matching the group's style). Corner radius overridden to 0 — the outer container handles rounding.
- **Leading/Trailing icon button**: Same Button variant as the group, with `left-icon: true` and text label hidden. This ensures consistent padding and height with the action buttons.
- **Separators**: 1px frames with `Edge/Default` fill, `layoutAlign: STRETCH`. Visibility tied to the same boolean as their adjacent action/icon button.
- Actions 3, 4 and both icon buttons are hidden by default.

### Color token bindings

| Element | Outline style | Secondary style |
|---------|--------------|-----------------|
| Outer container fill | `Surface/Default` | none (transparent) |
| Outer container stroke | `Edge/Default` (1px inside) | none |
| Action button fill | inherited from Button/Outline | inherited from Button/Secondary (`Neutral/Tertiary`) |
| Action button stroke | removed (outer container provides border) | inherited from Button/Secondary (none) |
| Separator fill | `Edge/Default` | `Edge/Default` |
| Text | `Content/Default/Primary` (via Button) | `Content/Default/Primary` (via Button) |

---

## Gap Analysis and Rationale

### Horizontal only — no vertical orientation

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Orientation | `horizontal`, `vertical` | `horizontal` only |

**Why the deviation:** The vertical button group's primary use case (e.g. a +/− stepper, quantity selector) is so different from a horizontal action bar that sharing a component set would make both harder to use. A vertical variant would need different sizing, different icon placement, and different separator behavior — all of which would add conditional complexity for a pattern that's used far less frequently.

**How to handle if needed:** Create a separate `ButtonGroupStepper` component purpose-built for vertical +/− interactions, with its own simpler property set (size, optional label). It shares the conceptual base of a button group but lives as its own component in Figma for usability.

### Two styles, not six

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Styles | Any button variant (default, destructive, outline, ghost, secondary, link) | `outline`, `secondary` |

**Why the deviation:** Button groups are structural UI — they group related actions. In practice, the two most common patterns are outlined (bordered, neutral) and filled secondary (subtle gray). A brand-colored (default/primary) group was considered during scoping but excluded to keep the variant count at 4 instead of 6. Ghost, link, and destructive don't make practical sense as grouped button styles.

**How to handle if needed:** Adding a `brand` style variant is a straightforward extension — add 2 more physical variants (brand/default, brand/sm) with `Brand/Default` fill and `Content/Inverse/Primary` separator color.

### Two sizes, not four

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Sizes | `sm`, `default`, `lg`, `icon` | `default`, `sm` |

**Why the deviation:** Button groups are typically compact UI controls. Large button groups are uncommon. The `icon` size is handled differently (see icon buttons below). Two sizes cover the realistic range.

### Action count via booleans, not dynamic children

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Action count | Unlimited dynamic children | 2 default, up to 4 via booleans |

**Why the deviation:** Figma components need fixed slots — there's no "render N children" like React. Actions 3 and 4 are hidden by default and revealed via `Show action 3` / `Show action 4` booleans. This is a Figma-specific constraint, not a design limitation.

**Sequencing note:** Technically a designer can show action 4 without action 3, creating a gap. In practice, always show them sequentially. A single "Action count" dropdown (values: 2, 3, 4) would prevent this, but it would require tripling the variant grid from 4 to 12 physical variants — the booleans are a pragmatic tradeoff.

### Icon buttons at group level, not per-button

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Icon-only buttons | Any button can be `size="icon"`, placed anywhere in the group | Leading/trailing icon button at group edges only |

**Why the deviation:** In ShadCN, any child button can be icon-only, and you can freely mix text and icon buttons in any order. Exposing per-button icon configuration in Figma (show/hide icon, show/hide label, icon position — for each of 4 buttons) would create an overwhelming number of properties.

Instead, the component offers a simpler model: optional icon-only buttons at the **start** or **end** of the group (the split-button pattern — e.g. a primary action + a chevron trigger). These use the same Button variant as the group with `left-icon: true` and text hidden, so they have consistent padding and height.

**How to handle if needed:** For a one-off mixed layout (e.g. icon button in the middle of the group), detach the instance and swap individual action slots manually.

### Style is group-level, not per-button

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Button styling | Each `<Button>` sets its own `variant` independently | Group `style` variant determines all buttons at once |

**Why the deviation:** ShadCN's flexibility lets you mix outline and ghost buttons in the same group. In practice, button groups use a consistent style. Making style a group-level variant means designers pick it once — not 2–4 times per instance. If a mixed-style group is needed, detach the instance.

### Separator is structural, not a standalone component

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Separator | `<ButtonGroupSeparator>` placed manually between buttons | Built into the component, visibility automatic |

**Why the deviation:** In code, the separator is a component you explicitly place between children. In Figma, separators are structural layers inside the component whose visibility is tied to the adjacent action's boolean. Designers never need to think about separators — they appear automatically when an action is shown.

For both styles, separators use `Edge/Default`. The original scoping considered different separator colors per style (e.g. white separators on brand-colored groups), but since brand style was excluded, `Edge/Default` works for both outline and secondary.

### No ButtonGroupText slot

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Text slot | `<ButtonGroupText>` for embedding labels | Not included |

**Why the deviation:** ButtonGroupText is a composition pattern for embedding labels (e.g. a `<label>` element) inside the group container. This is an uncommon pattern that's better handled as an ad-hoc composition — place a text layer alongside the component — than a permanent property.

### No compositions (input, select, dropdown, popover)

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Compositions | Input, select, dropdown menu, popover demos | Not included |

**Why the deviation:** These are compositions of multiple components, not variants of ButtonGroup. The ShadCN docs show them as usage examples, but in Figma they should be built as one-off frames combining ButtonGroup with Input, Select, DropdownMenu, etc. Trying to build these into the component would turn a focused button group into an unwieldy Swiss Army knife.

---

## Design Principle

The button group exemplifies a different challenge from the button: **combinatorial scoping**. A simple component like Button has a manageable number of variants — you can include most of them. A complex component like ButtonGroup has a permutation space that grows multiplicatively across dimensions, and the skill needs to scope it down *before* building.

The test applied at each dimension: *Is this variant something designers will use regularly, or is it an edge case?* Regular usage becomes a variant or property. Edge cases are handled by extending the component later or detaching for one-off overrides.

The key architectural decisions:
- **Nested Button instances** instead of custom-built action slots. This means the component inherits the Button's existing styling, text styles, and icon support — and stays in sync when Button is updated.
- **Booleans over variants** for action count and icon buttons. This keeps the variant grid at 4 instead of potentially 48+, while still covering 2–4 actions and optional icon buttons.
- **Group-level style** instead of per-button style. A deliberate simplification that matches how button groups are actually used in designs.
