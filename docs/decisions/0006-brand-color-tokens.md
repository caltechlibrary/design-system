# 6. Keep brand colors as a named palette, and build role tokens on top of it

- Status: proposed
- Date: 2026-11-12

## Context and Problem Statement

The design system needs colors that every app can share. Today the brand colors live as Sass variables in libguine's `scss/common/_colors.scss`, and they disappear when the SCSS compiles. No other app can use them.

We are moving CSS out of libguine and into the design system in small slices, starting with these colors. That first slice raised three questions:

Consideration 1: What should the tokens be named? This is hard to change later, because every app will reference these names.

Consideration 2: Some colors in the palette aren't really brand colors. `$monza-alert` and `$burnt-alert` are darker shades that exist only so alert text passes contrast.

Consideration 3: The web components in this repository define their own `--cl-*` colors (`--cl-dark-blue`, `--cl-teal` and others). They came over from CL-web-components and look like design-system tokens, but they aren't.

## Decision Drivers

- Token names will be used by every app, so they need to stay stable.
- We don't want to name or design every color role before we know what the roles are. Roles should come from the CSS as each slice is moved.
- The same file has to work on Springshare pages, which load it with `<link>`, and inside web components, which load it in their shadow DOM.
- Moving the colors shouldn't require renaming them in libguine.
- Someone reading an app's CSS should be able to tell a raw brand color from a role.

## Considered Options

1. Plain color names (`--cl-dark-blue`, `--cl-teal`)
2. Keep the Sass names (`--tarawera`, `--jungle`)
3. Role names (`--cl-primary`, `--cl-danger`)
4. Two layers: a brand palette that keeps the Sass names, with role tokens added on top as roles come up

## Decision Outcome

**Chosen: Option 4.**

### Option 1: plain color names — rejected

Easy to read, but the palette has several similar colors (three greens, a gray, a brown), so the names would need qualifiers and would invite debate. A name like `--cl-teal` also becomes wrong if the color changes.

### Option 2: keep the Sass names, palette only — rejected on its own

Every name is already unique, and libguine's switch becomes a find-and-replace. On its own, though, it gives apps no way to say what a color is *for*. Option 4 keeps these names for the palette and adds that missing layer.

### Option 3: role names — rejected

Names that describe purpose survive color changes. But most of the palette is the five site-section colors, which don't have roles, and some colors play several roles: `$tarawera` is the footer background, the info alert, the search button and a section color. Choosing roles first would be a design project of its own.

### Option 4: a named palette, with role tokens on top — chosen

```css
/* layer 1: brand palette, one entry per brand color, no prefix */
--monza: #cf0a2c;

/* layer 2: role tokens, added in later slices, with a --cl- prefix */
--cl-alert-danger-text: /* derived from --monza */;
```

- **The palette keeps the Sass names, without a prefix.** `$jungle` becomes `--jungle`.
- **Role tokens get a `--cl-` prefix** (Caltech Library). The difference in prefix is how you tell the two layers apart: `var(--jungle)` in an app's CSS is a raw brand color, and `var(--cl-footer-bg)` is a role. Role names are generic words like `primary` and `text-color` that other themes and libraries also use, so they need the prefix to avoid clashing on pages we don't fully control, like Springshare's. The palette names are distinctive enough without one.
- **The palette holds brand colors only.** It starts with the colors from libguine's `_colors.scss`, and more brand colors can be added as they come up (neutrals such as white, black or a gray, for example). `$monza` is in even though libguine uses it only for the danger alert today, because it's the brand red.
- **Shades made for a role are role tokens, not palette colors.** `$monza-alert` and `$burnt-alert` wait for the alert slice and will become role tokens there.
- **Roles are added as slices reveal them,** not designed up front.
- **libguine's palette is the source.** The web components in this repository consume the tokens. Their own `--cl-*` colors aren't part of the design system and will switch to the tokens later.

The tokens are published as a static file, `tokens.css`, built into `dist/` and released with the rest of the design system. The variables sit on `:root, :host`. `:root` applies when a site loads the file with `<link>`, and `:host` applies when a web component loads it inside its shadow DOM.

## Consequences

Good:

- The palette is settled now, without waiting for roles to be designed.
- libguine can switch to the tokens without renaming anything.
- A role's color can change without touching the palette, and a brand color can change in one place.
- One file serves both Springshare pages and web components.

Bad, and accepted:

- **The palette names don't say what color they are.** Nobody can guess the color of `--mulled` from its name. Role tokens are where readable names belong.
- **Two naming rules instead of one.** Palette tokens have no prefix, while role tokens use `--cl-`. A palette name seen in an app's CSS also doesn't show that it comes from the design system.
- **Until role tokens exist, apps use palette names directly.** Each slice should add the roles it finds, so this shrinks over time.
- **The web components keep their own colors for now,** including a `--cl-teal` (`#008080`) that differs from the brand teal, `--jungle` (`#19747d`). That gets settled when the components switch to the tokens.

## More Information

- Still open, and tracked in the slice: Sass can't run its color functions (`darken()`, `mix()`, `color.adjust()`) on a CSS variable, so some libguine lines can't use the tokens directly yet. Whether role tokens calculate their shades (`color-mix()`) or pick from fixed shades in the palette gets decided in the alert slice.
- The tokens are versioned and released with the rest of the design system, per ADR 0005.
