# Theme Creation Flow

Use this workflow for WordPress classic hybrid theme creation when design review should happen before full page/template generation.

This workflow is mandatory when referenced by `wp-classic-theming.md`.

## Core Rule

Stop after each phase and wait for explicit approval before continuing.

Do not continue automatically after phrases such as:

- looks good
- continue
- nice
- ship it
- proceed

Only advance when the user writes the explicit phase approval phrase:

- Approve Phase 1
- Approve Phase 2
- Approve Phase 3

Phase 4 does not require a later phase approval unless the user asks for additional phased work.

## Source Priority

When extracting design decisions, use this priority order:

1. Components, UI Kit, or Design System page from Pencil, Figma, or the provided design package.
2. Components inferred from page designs.
3. Explicitly documented assumptions.

Never silently invent important component states. If a state is inferred, report it in this form:

```text
Inferred component state:
Service Card Hover

Reason:
No hover state was provided in the design files.
```

If a component is missing from the UI kit but appears in a page design, identify the source page and explain the inference.

## Before Phase 1

Inspect available design files before generating code.

Look for:

- Dedicated Components, UI Kit, or Design System pages.
- Buttons, cards, navigation, forms, headers, footers, heroes, CTA blocks, and content cards.
- Variants such as primary, secondary, compact, featured, dark, light, image, text-only.
- States shown in the design, plus interaction states that are semantically required for the component to function.
- Mobile, tablet, and desktop behavior.
- Existing theme files if enhancing an existing theme.

Report:

- Design sources found.
- Component sources found.
- Missing states that matter.
- Assumptions required before coding.

Keep assumptions explicit and short. Do not let an assumption become hidden implementation behavior.

## Phase 1: Design Tokens

Generate the design system foundation.

Create or update:

- `theme.json`
- Color palette
- Gradients
- Typography scale
- Spacing system
- Shadows
- Radius values
- Container widths

Rules:

- Use the design file as the source of truth when available.
- Map reusable values into `theme.json` settings and styles.
- Support color presets from the design as hex, `rgb()`, or `rgba()`.
- Define shared heading defaults plus isolated `h1` through `h6` element styles in `theme.json`.
- Use WordPress preset variables for CSS values whenever possible.
- Do not duplicate theme.json values in `style.css`.
- Do not generate components, layouts, or templates yet unless minimal scaffolding is required to verify tokens.
- If local fonts are needed, use WOFF2 only, plan `assets/fonts/` and `fontFace` declarations, and do not enqueue Google Fonts from a CDN.

Deliverables for review:

- Summary of extracted tokens.
- Files created or changed.
- Any token assumptions.
- Any mismatches between design file values and WordPress theme.json constraints.

Stop and wait for:

```text
Approve Phase 1
```

## Phase 2: Component Library

Generate reusable components before generating pages.

Examples:

- Buttons
- Cards
- Navigation
- Hero variants
- Forms
- CTA blocks
- Footers
- Headers
- Blog/content cards
- Pagination
- Search form

Create a dedicated visual review page: `component-library.php`

The page must visually display the component system before page templates are generated.

The showcase page must include:

- All reusable components.
- All variants.
- All documented states.
- Only inferred states that are necessary for the component's real interaction model, clearly labeled.
- Desktop and mobile navigation examples where possible.
- Open and closed mobile navigation states when the design or navigation behavior requires them.

Use helper classes when needed to force states for review:

- `.is-hover`
- `.is-focus`
- `.is-active`
- `.is-current`
- `.is-open`
- `.is-disabled`
- `.is-loading`
- `.has-error`
- `.has-success`

These helper classes should exist only to make visual QA possible. They should mirror real pseudo-class or JavaScript states instead of creating separate designs.

Component implementation rules:

- Put reusable PHP component renderers or partials in `components/`.
- Keep component CSS grouped and named consistently in `style.css` or a theme stylesheet that is enqueued by `functions.php`.
- Use approved `theme.json` tokens for colors, spacing, typography, shadows, radii, and widths.
- Use semantic, descriptive class names and native CSS nesting where helpful. Do not use BEM naming.
- Implement states only when they are shown in the design, required by accessibility, or required by the component's behavior.
- Do not create every possible state for every component. For example, a static informational card may not need disabled, loading, error, or open/closed states.
- For standard interactive controls, include baseline accessible states such as `:hover` and `:focus-visible` unless the design explicitly prevents a distinct hover treatment.
- For behavioral components, include only the states the behavior needs, such as open/closed for a mobile menu or error/success for a validated form.
- If a potentially useful state is not shown and not required, report it as omitted instead of inventing it.
- Do not introduce page-specific layout decisions here.
- Do not generate full page templates yet.

Deliverables for review:

- Component source list: UI kit, page-inferred, or assumption.
- Component files created.
- Showcase page route/file.
- State inventory.
- Inferred state report.
- Known gaps.

Stop and wait for:

```text
Approve Phase 2
```

## Phase 3: Layouts

Generate reusable layout compositions using approved components.

Examples:

- Hero sections
- Service grids
- Team grids
- CTA sections
- Blog previews
- Testimonial rows
- Feature comparisons
- Contact strips

Create a dedicated visual review page for layouts: `layout-library.php`

Its purpose is to review full layout compositions before templates are generated.

This page should display:

- Every reusable layout.
- Relevant layout variants.
- Desktop and mobile responsive behavior.
- Layouts using realistic sample content.
- Layouts using approved components only.
- Any inferred layout behavior, clearly labeled in the phase report.

Rules:

- Layouts must compose approved Phase 2 components.
- Avoid introducing new visual patterns here.
- If a new component need appears, stop and report it instead of burying it inside a layout.
- Put reusable layouts in `layouts/`.
- Keep layout CSS token-driven.
- Use `alignfull` wrappers and `alignwide` inner containers where appropriate.
- Use modern Grid and Flexbox. Avoid float/position fallbacks.
- Use `@media (max-width: 1024px)` for tablet and `@media (max-width: 600px)` for mobile.
- Do not apply inline left/right padding to first-level section wrappers; center constrained inner containers with `max-width` and `margin: 0 auto`.
- Preserve accessibility and responsive behavior from the component layer.
- Do not generate production page templates yet.
- The layout showcase is a review surface, not a replacement for `front-page.php`, `page-{slug}.php`, or other Phase 4 templates.

Deliverables for review:

- Layout files created.
- Layout showcase page route/file.
- Components used by each layout.
- Any design-file source for each layout.
- Responsive behavior notes.
- Any new needs discovered that should return to Phase 2.

Stop and wait for:

```text
Approve Phase 3
```

## Phase 4: Templates

Generate WordPress templates after tokens, components, and layouts are approved.

Generate as needed:

- `front-page.php`
- `home.php`
- `page.php`
- `page-{slug}.php`
- `single.php`
- `archive.php`
- `search.php`
- `404.php`
- `index.php`
- `header.php`
- `footer.php`
- `comments.php`
- `searchform.php`
- `template-parts/`

Rules:

- Templates should primarily assemble approved layouts and components.
- Reuse approved components whenever practical instead of duplicating markup.
- Do not introduce new visual systems in templates.
- If a template requires a new component or state, stop and report it.
- Static pages should be code-first PHP templates, not database-sourced page bodies.
- Blog/editorial content remains managed through the Block Editor.
- PHP templates must not contain `<!-- wp:* -->` block comments.
- Do not generate a `patterns/` folder. Block markup belongs only in saved editor content.

Deliverables:

- Template files created.
- Template-to-layout map.
- Component reuse summary.
- Remaining assumptions or content placeholders.
- Verification notes.

## Required Review Summary Format

At the end of each phase, report:

```text
Phase completed:
Phase N - Name

Created/changed:
- file
- file

Design sources used:
- source

Inferred items:
- item and reason

Needs user review:
- item

Next required approval:
Approve Phase N
```

For Phase 4, replace `Next required approval` with `Ready for final review`.

## Workflow Integrity

If the user asks for the entire theme in one prompt, still follow the phases.

If the user explicitly asks to skip checkpoints, explain that this workflow requires approval gates and ask whether they want to use a different skill or workflow.

If a design file lacks a component library, continue by inferring from page designs, but label every important inferred component or state.

If no design file is available, use documented assumptions and keep the first pass conservative.
