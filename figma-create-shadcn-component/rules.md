# ShadCN → Figma Component Rules

Guidance for building Figma components from ShadCN component specs, adapted to the design system.

## Core Principles

1. **Use semantic tokens, not ShadCN defaults.** Every color and font in a ShadCN component must be replaced with the design system equivalent. ShadCN's styling is a starting point, not the output.

2. **Use the dual-font system.** The design system uses separate font families for headings/interactive UI and body/content text. Never mix them within a single text role. Check the `font/family/heading` and `font/family/sans` variables for the current fonts.

3. **Apply Figma text styles, not manual properties.** Always use the text styles from the Semantic Fonts section. Do not manually set font size, weight, or line height unless no semantic style fits.

4. **Bind to Figma variables.** Use color variables from the Semantic Classes collection (Content/*, Surface/*, Edge/*, Brand/*, etc.) for all color properties. Never hardcode color values.

---

## Color Token Mapping

When a ShadCN component references a color token (e.g., `bg-primary`, `text-muted-foreground`, `border-border`), replace it with the corresponding Figma variable from the Semantic Classes collection.

### Surfaces and foregrounds

| When ShadCN uses | Use Figma variable | When to apply |
|---|---|---|
| `--background` | `Surface/Default` | Page backgrounds, app shells, root containers |
| `--foreground` | `Content/Default/Primary` | Default text and icons on any background surface |
| `--card` | `Surface/Default` | Card fills, container backgrounds — cards are elevated by border and shadow, not color |
| `--card-foreground` | `Content/Default/Primary` | Text and icons inside cards |
| `--popover` | `Surface/Default` | Dropdown menus, dialog surfaces, tooltip backgrounds, popovers |
| `--popover-foreground` | `Content/Default/Primary` | Text and icons inside floating surfaces |
| `--muted` | `Surface/Secondary` | De-emphasized areas, disabled surfaces, grouped content backgrounds, sidebar fills |
| `--muted-foreground` | `Content/Default/Secondary` | Helper text, placeholder text, secondary descriptions |

### Actions and emphasis

| When ShadCN uses | Use Figma variable | When to apply |
|---|---|---|
| `--primary` | `Brand/Default` | Primary CTA fills, brand-colored buttons, active indicators |
| `--primary-foreground` | `Content/Inverse/Primary` | Text and icons on brand-colored surfaces |
| `--secondary` | `Neutral/Tertiary` | Secondary button fills, low-emphasis actions, supporting surfaces |
| `--secondary-foreground` | `Content/Default/Primary` | Text on secondary surfaces |
| `--accent` | `Neutral/Tertiary` | Hover and focus backgrounds on ghost buttons, menu items, list items |
| `--accent-foreground` | `Content/Default/Primary` | Text on accent/hover surfaces |
| `--destructive` | `Danger/Default` | Destructive button fills, error badges, critical alert backgrounds |
| `--destructive-foreground` | `Content/Inverse/Primary` | Text on destructive surfaces |

### Borders and focus

| When ShadCN uses | Use Figma variable | When to apply |
|---|---|---|
| `--border` | `Edge/Default` | Card borders, dividers, separators, table rules |
| `--input` | `Edge/Secondary` | Form input outlines, select borders, textarea boundaries |
| `--ring` | `Misc/Ring` | Focus rings on buttons, inputs, and other interactive elements |

### Data visualization

| When ShadCN uses | Use Figma variable | When to apply |
|---|---|---|
| `--chart-1` | `Misc/Chart 1` | Primary data series |
| `--chart-2` | `Misc/Chart 2` | Second data series |
| `--chart-3` | `Misc/Chart 3` | Third data series |
| `--chart-4` | `Misc/Chart 4` | Fourth data series |
| `--chart-5` | `Misc/Chart 5` | Fifth data series |

### Sidebar (create mappings only when building sidebar components)

| When ShadCN uses | Use Figma variable | When to apply |
|---|---|---|
| `--sidebar` | `Surface/Default` or `Surface/Secondary` | Sidebar background — ask the user if the sidebar should match or contrast with the page |
| `--sidebar-foreground` | `Content/Default/Primary` | Sidebar text and icons |
| `--sidebar-primary` | `Brand/Default` | Active sidebar items |
| `--sidebar-primary-foreground` | `Content/Inverse/Primary` | Text on active sidebar items |
| `--sidebar-accent` | `Neutral/Tertiary` | Hovered or selected sidebar items |
| `--sidebar-accent-foreground` | `Content/Default/Primary` | Text on hovered/selected sidebar items |
| `--sidebar-border` | `Edge/Default` | Sidebar dividers |
| `--sidebar-ring` | `Misc/Ring` | Focus rings within the sidebar |

---

## Typography Mapping

When importing a ShadCN component, replace Tailwind typography utility classes with the closest semantic font style from the design system.

### Context-first decision rules

The semantic font is determined by **what the text is and what component it belongs to** — not by the size or weight ShadCN happens to use. Always ask "what role does this text play?" before looking at utility classes.

| Context | Semantic Font |
|---|---|
| Button label | **Button** |
| Small button label | **Button Small** |
| Dialog, sheet, or card title | **Title Small** or **Subtitle Large** |
| Section heading within a component | **Subtitle** or **Subtitle Small** |
| Form input or textarea content | **Input** |
| Label next to a form control | **Label** or **Label Bold** |
| Description, helper text, supporting paragraph | **Body Small** or **Body** |
| Badge, tag, or small metadata | **Caption** or **Caption Bold** |
| Toast/alert title | **Subtitle Small** |
| Toast/alert message body | **Body Small** |
| Table header cell | **Label Bold** |
| Table body cell | **Body Small** |
| Dropdown or select menu item | **Body Small** |
| Tooltip content | **Caption** or **Body Small** |
| Tab label | **Button** |
| Breadcrumb or navigation link | **Label** or **Label Bold** |
| Code or code snippet | **Code** |
| Large page title or hero heading | **Display**, **Title Large**, or **Title** |

### Utility class reference (secondary)

Use this table only when the component context above is ambiguous.

| ShadCN Utilities | Semantic Font |
|---|---|
| `text-5xl font-bold tracking-tight` or larger | **Display** |
| `text-4xl font-bold tracking-tight` | **Title Large** |
| `text-3xl font-semibold tracking-tight` | **Title** |
| `text-xl font-semibold` | **Title Small** |
| `text-lg font-semibold` | **Subtitle Large** |
| `text-base font-semibold` | **Subtitle** |
| `text-sm font-semibold` | **Subtitle Small** |
| `text-lg font-normal` | **Body Large** |
| `text-base font-normal` | **Body** |
| `text-sm font-normal` | **Body Small** |
| `text-sm font-medium` (in a button) | **Button** |
| `text-xs font-medium` (in a small button) | **Button Small** |
| `text-sm font-medium` (not a button) | **Label Bold** |
| `text-sm font-normal` (form labels) | **Label** |
| `text-sm` (inputs) | **Input** |
| `text-xs font-normal` | **Caption** |
| `text-xs font-medium` | **Caption Bold** |
| `font-mono` | **Code** |

### Typography rules

- **Never mix font families within a single text role.** The heading font is for headings and interactive UI. The body font is for body content, labels, and captions.
- **Preserve heading tracking.** Display, Title, and Title Large have negative letter spacing defined in their styles. Do not override this.
- **Accept minor size differences.** If the nearest semantic font is 1-2px off from ShadCN's default, use the semantic font anyway. Consistency across the system matters more than pixel-matching ShadCN.

---

## Extending Beyond ShadCN

The design system has tokens that ShadCN does not cover. When building or customising components, use these directly rather than constraining to ShadCN's flat model.

| Scenario | Use Figma variable |
|---|---|
| Warning or caution state | `Warning/Default` for fills, `Content/Warning/Primary` for text |
| Success or positive state | `Positive/Default` for fills, `Content/Positive/Primary` for text |
| Softer brand fill (selected state, tag, badge) | `Brand/Secondary` or `Brand/Tertiary` |
| Brand-colored text or icon | `Content/Brand/Primary` |
| Error or danger text | `Content/Danger/Primary` |
| Hover states | Use the corresponding `*/Default Hover`, `*/Secondary Hover`, or `*/Tertiary Hover` token rather than opacity or shade utilities |
| Strong neutral fill (solid badge, dark button) | `Neutral/Default` |
| High-emphasis border (active/focused input) | `Edge/Tertiary` |

---

## Fallback Behaviour

When no semantic font or color token clearly fits:

1. **Check the Tailwind CSS Variables collection in Figma.** It contains all Tailwind utility values (font sizes, weights, line heights, tracking, colors) as Figma variables. These can be applied directly and will match ShadCN's utilities exactly. Use these as a fallback to preserve the intended styling while still binding to variables rather than hardcoding values.

2. **When in doubt, ask the user.** If there are two reasonable choices — whether for a font style (e.g., Subtitle Small vs Label Bold) or a color token (e.g., Surface/Secondary vs Neutral/Tertiary) — present both options and ask the user which fits their intent. Do not guess.
