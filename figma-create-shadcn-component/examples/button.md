# Example: Button Component

ShadCN spec: https://ui.shadcn.com/docs/components/radix/button
Figma reference: Appcues Tailwind Design System → Button component set (node 5:1625)

This example demonstrates how a ShadCN component spec gets translated into a Figma component, including deliberate deviations that optimize for design workflow over 1:1 spec parity.

---

## ShadCN Dev Spec

### Props

| Prop | Type | Values | Default |
|------|------|--------|---------|
| `variant` | enum | `default`, `destructive`, `outline`, `secondary`, `ghost`, `link` | `default` |
| `size` | enum | `default`, `xs`, `sm`, `lg`, `icon`, `icon-xs`, `icon-sm`, `icon-lg` | `default` |
| `asChild` | boolean | `true`, `false` | `false` |

Plus all native `<button>` HTML attributes (`disabled`, `type`, `onClick`, `aria-*`, etc.).

### Interaction states (CSS)

Handled via pseudo-classes: `:hover`, `:focus-visible`, `:active`, `:disabled`.

### Icon support

Icons are composed as children using `data-icon="inline-start"` (left) or `data-icon="inline-end"` (right). Any icon, either side.

### Loading

No dedicated prop — composed by placing a spinner inside the button and setting `disabled`.

---

## Figma Component (as built)

### Property architecture

| Property | Figma type | Values | Default |
|----------|------------|--------|---------|
| `variant` | Variant | `default`, `Destructive`, `Outline`, `Secondary`, `Ghost`, `Link`, `Icon` | `default` |
| `size` | Variant | `default`, `sm`, `lg` | `default` |
| `state` | Variant | `default`, `hover` | `default` |
| `left-icon` | Boolean | `true`, `false` | `false` |

### Variant grid

7 variants x 2 states x 3 sizes = 42 theoretical, but `Icon` has no `lg` size → **40 physical variants**.

### Internal structure per variant

- Icon instance child (`RocketLaunch`, hidden by default, toggled by `left-icon` boolean)
- Text child (`"Button"`)
- `Icon` variant: icon instance only, no text child

---

## Gap Analysis and Rationale

### Icon as a variant value (not a size)

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Icon-only buttons | Size values: `icon`, `icon-xs`, `icon-sm`, `icon-lg` | Separate variant value: `Icon` |

**Why the deviation:** In ShadCN, icon-only is a sizing concern — "same button, just square." In Figma, icon-only buttons are frequent in design files, and designers need to switch to them quickly. Making `Icon` a first-class variant value means it appears directly in the variant picker — one click, no digging through size options. This is a workflow optimization: the code architecture treats it as a size, but Figma's component model serves designers better when it's a distinct variant they can see and select at a glance.

### Reduced state coverage

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| States | `:hover`, `:focus-visible`, `:active`, `:disabled` (CSS pseudo-classes) | `default`, `hover` (variant property) |

**Why the deviation:** In everyday design work, buttons appear in `default` and `hover` states — these are the states designers place in mockups, prototypes, and handoff screens. `disabled`, `focus`, and `focus-visible` are real states that need to be designed, but they don't need to live inside the component set.

Adding them would mean: more properties in the properties panel, a larger variant grid, and slower component selection — all for states that appear in maybe 5% of design usage. The tradeoff isn't worth it for a component designers interact with constantly.

**How to handle the missing states:** Design `disabled`, `focus`, and `focus-visible` as standalone frames alongside the component set (same page, outside the component). They serve as spec reference for engineering without bloating the component that designers use hundreds of times per project.

### Reduced size coverage

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Sizes | `default`, `xs`, `sm`, `lg`, `icon`, `icon-xs`, `icon-sm`, `icon-lg` (8) | `default`, `sm`, `lg` (3) |

**Why the deviation:** The `icon-*` sizes are covered by the `Icon` variant (see above). `xs` was omitted because it's uncommon in the design system's usage patterns — three sizes cover the realistic range. Fewer size options means faster variant selection.

### No loading state

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Loading | Composed via spinner child + `disabled` | Not represented |

**Why the deviation:** Loading is a transient interaction state. Designers rarely need to place a loading button in a static mockup. If needed for a specific flow, it can be composed ad-hoc (swap the icon to a spinner, override the text). Not worth a permanent property on every button instance.

### No right-side icon

| Aspect | ShadCN | Figma |
|--------|--------|-------|
| Icon placement | Either side via `data-icon="inline-start/end"` | Left only (`left-icon` boolean) |

**Why the deviation:** Left-icon covers the dominant use case (action icon + label). Right-side icons (typically chevrons or external-link indicators) are less common. Adding a second boolean and instance swap would double the icon-related complexity. If a right-icon pattern becomes frequent, it can be added later as a boolean property without restructuring the component.

### asChild not represented

**No gap.** `asChild` is a React composition pattern (render as `<a>`, `<Link>`, etc.) with no visual difference. Correctly omitted from Figma.

---

## Design Principle

The overarching principle: **code and Figma are different tools with different ergonomics.** A 1:1 port of every ShadCN prop into Figma properties produces a technically complete but practically unusable component. The goal is maximum alignment on the visual variants designers actually place in files, while acknowledging that interaction states, edge-case sizes, and compositional patterns (loading, asChild) belong in code or in reference designs outside the component set.

The test for whether something belongs in the component: *Will a designer reach for this property multiple times per day?* If yes, it's a variant or boolean. If no, it's a reference frame or an ad-hoc override.
