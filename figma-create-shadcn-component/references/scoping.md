# Complex Component Scoping Guide

This reference covers Phase 4a (Variant Scoping) and Phase 4b (Property Architecture) in detail. Read this when the component is complex — button groups, dialogs, data tables, command palettes, navigation menus, toolbars, or anything with many variants, nested sub-components, or conditional relationships between options.

---

## Phase 4a: Variant Scoping

For complex components, the ShadCN spec describes every possible permutation. A button group alone could have styles x sizes x content patterns x action counts = hundreds of variants. The job here is to narrow that down to what the user will actually use in their designs.

Walk through each dimension with the user, proposing sensible defaults based on the ShadCN spec and common usage patterns. Present it as a structured checklist they can confirm or adjust:

1. **Styles/visual variants** — Which visual styles from the ShadCN spec are needed? Not all of them — ask which ones. For example: "The spec supports default, secondary, outline, ghost, destructive, and link. Which of these do you need for this component?"

2. **Sizes** — Which size variants? Many components only need 1-2 sizes in practice. Propose the most common subset (e.g., "default and sm" rather than "sm, default, lg, icon").

3. **Content patterns** — What goes inside each item? For components with flexible content slots (like a button group where each action could be text-only, icon-only, leading icon + text, or text + trailing icon), ask which patterns matter. These often become boolean properties rather than variants — but the user needs to confirm which patterns to support.

4. **Item/action count** — For group-type components (button groups, breadcrumbs, tabs, steppers), how many items should the component support? This is a Figma-specific concern — the component needs a fixed set of slots that designers can show/hide. Propose a range (e.g., "2 to 4 actions, with extra actions hidden by default").

5. **Conditional relationships** — Identify dependencies between options and confirm them. These are critical because they affect which layers exist and how they're styled. For example:
   - "Outline style -> no separator between items"
   - "Filled style -> separator visible, using `Edge/Default`"
   - "Filled brand style -> separator visible, using `Content/Inverse/Primary` (white on dark)"
   - "Vertical orientation -> only used for +/- toggle pattern, probably a separate component"

   Present these as if/then rules so the user can verify the logic.

6. **Separate components** — Some use cases that technically fall under the same ShadCN component are different enough in Figma that they should be their own component. The test: if a use case would require so many conditional layers and variant overrides that it becomes confusing for designers, split it out. Propose splits when you see them (e.g., "A vertical +/- stepper control shares the button group base, but in Figma it would be cleaner as its own component — fewer properties, purpose-built for that use case"). Let the user decide.

**Output of Phase 4a:** A scoping summary like this (adapt the dimensions to the component):

```
Component: Button Group
Styles: outline, filled-neutral, filled-brand
Sizes: default, sm
Content: text-only, leading icon + text, text + trailing icon
Action count: 2, 3, 4
Conditional rules:
  - outline -> no separator
  - filled-neutral -> separator (Edge/Default)
  - filled-brand -> separator (Content/Inverse/Primary)
Separate components:
  - Vertical +/- toggle -> own component (ButtonGroupStepper)
```

Wait for the user to confirm or adjust before proceeding.

---

## Phase 4b: Property Architecture

With the scope defined, plan how each dimension maps to Figma's component model. This matters because the wrong mapping creates either an unusable number of variants or a component where designers can't find what they need.

**Figma has four mechanisms for component flexibility — use each for what it's good at:**

| Mechanism | Use when | Example |
|---|---|---|
| **Variant property** | The option changes the component's visual appearance in a way designers need to see in the variant picker. These multiply the number of physical variants, so keep the set small. | `style=outline / filled / filled-brand`, `size=default / sm` |
| **Boolean property** | The option shows or hides a layer. This avoids creating separate variants for every show/hide combination — one component covers many configurations by toggling layers on or off. | `Show leading icon=true/false`, `Show action 3=true/false`, `Show separator=true/false` |
| **Instance swap property** | A nested component slot that designers swap out (e.g., picking which icon to use). | `Leading icon` (swap to any icon component), `Trailing icon` |
| **Separate component** | The use case has diverged enough that sharing a component set would confuse designers. It might share underlying structure, but in Figma it lives as its own component with its own simpler property set. | A vertical +/- stepper split from a horizontal button group |

**Plan the architecture:**

1. List every variant property and its values. Count the resulting variant grid (e.g., 3 styles x 2 sizes = 6 physical variants). If the grid exceeds ~20 variants, look for dimensions that can move to boolean/instance swap properties instead.

2. List every boolean property — each one is a layer that gets shown or hidden. Name them clearly from the designer's perspective (e.g., `Show trailing icon`, not `hasTrailingIcon`).

3. List every instance swap property and what it swaps (e.g., "Leading icon -> any icon component").

4. List any conditional rules that the component's internal logic needs to handle. In Figma, these are implemented by setting default visibility and values per variant. For example: "In the `style=outline` variant, the separator layer's default visibility is off. In `style=filled-brand`, the separator layer is visible and its fill is bound to `Content/Inverse/Primary`."

5. List any separate components and their own (simpler) property sets.

**Output of Phase 4b:** A property architecture spec like this:

```
Button Group — Property Architecture

Variant properties (6 physical variants):
  - style: outline | filled | filled-brand
  - size: default | sm

Boolean properties:
  - Show action 3 (default: false)
  - Show action 4 (default: false)
  - Show leading icon (default: false)
  - Show trailing icon (default: false)

Instance swap properties:
  - Leading icon -> icon component
  - Trailing icon -> icon component

Conditional defaults per variant:
  - style=outline -> separator layer hidden
  - style=filled -> separator visible, fill bound to Edge/Default
  - style=filled-brand -> separator visible, fill bound to Content/Inverse/Primary

Separate components:
  - ButtonGroupStepper (vertical +/- toggle)
    Variant properties: size (default | sm)
    Boolean properties: Show label (default: false)
```

Present this to the user and wait for confirmation before building.
