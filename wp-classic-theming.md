---

## name: wordpress-classic-theming

description: WordPress hybrid classic theme development with phased design-token, component-library, layout, and template generation. Use when generating or evolving theme.json, reusable PHP components, layouts, template parts, functions.php, and classic PHP templates for WordPress themes where PHP templates drive main site pages and Gutenberg manages blog/editorial content.

# WordPress Classic Hybrid Theming

Build WordPress hybrid themes where PHP templates are the source of truth for main site pages, `theme.json` is the design system, reusable PHP components/layouts carry the interface, and the Block Editor manages blog/editorial content.

Read this skill first, then read and follow `theme-creation-flow.md` for execution order and approval checkpoints. The flow file controls sequencing only; all architecture, WordPress, theme.json, CSS, typography, component, and template rules in this skill still apply.

## Absolute Rules

- **No duplicated variables**: Do not redeclare in `style.css` any color, spacing, typography, radius, shadow, or width value already defined in `theme.json`. Use matching WordPress preset custom properties whenever possible.
- **No hard-coded font sizes**: Do not set literal font sizes in `style.css` or inline HTML. Use `--wp--preset--font-size--`* variables.
- **No hard-coded spacing**: Do not use literal padding or margin values where a spacing token exists. Use `--wp--preset--spacing--`* variables.
- **No mixed output**: PHP templates are plain HTML/PHP. Never insert `<!-- wp:* -->` comments into PHP templates. Do not generate a `patterns/` folder; block markup belongs only in saved editor content.
- **Load Google fonts locally, not from CDN:**: Download only `.woff2` files into `assets/fonts/`, declare each family with `fontFace` in `theme.json`, and never use `.ttf`, `.otf`, `fonts.googleapis.com`, or `fonts.gstatic.com`.
- **Page content lives in PHP templates, not in the database**: Static pages may exist in the database as routing anchors. The matching PHP template wins for the route and contains the page layout/copy.
- **Validate components before templates**: Before generating templates, identify reusable UI patterns and implement them as shared components. Templates should reuse approved components whenever practical instead of duplicating markup.
- **Never silently invent important states**: Missing states must be reported as inferred, omitted, or missing. Implement a state only when it is shown in the design, required for accessibility, or required by the component's actual behavior.

## Mandatory Workflow

Use `theme-creation-flow.md` for the full workflow.

Phase order:

1. Design Tokens
2. Component Library
3. Layouts
4. Templates

Stop after Phases 1, 2, and 3. Wait for the exact approval phrase:

- `Approve Phase 1`
- `Approve Phase 2`
- `Approve Phase 3`

Do not continue automatically after casual approval language.

## Design Source Priority

Use this priority when extracting components and states:

1. Components, UI Kit, or Design System page.
2. Components inferred from page designs.
3. Explicitly documented assumptions.

Most projects include a Pencil or Figma page containing buttons, cards, navigation, form elements, component variants, hover states, focus states, and active states. Treat that page as the primary component source of truth.

If the component library is incomplete, infer from page designs and report the inference.

## Theme Architecture

Prefer this structure for component-first hybrid themes:

```text
theme-slug/
|-- style.css
|-- theme.json
|-- functions.php
|-- index.php
|-- header.php
|-- footer.php
|-- front-page.php
|-- home.php
|-- page.php
|-- page-{slug}.php
|-- single.php
|-- archive.php
|-- search.php
|-- 404.php
|-- comments.php
|-- searchform.php
|-- component-library.php
|-- layout-library.php
|-- components/
|   |-- buttons.php
|   |-- cards.php
|   |-- navigation.php
|   |-- forms.php
|   `-- content-card.php
|-- layouts/
|   |-- hero.php
|   |-- service-grid.php
|   |-- cta.php
|   `-- blog-preview.php
|-- template-parts/
|   |-- header/site-branding.php
|   |-- content/content.php
|   |-- content/content-page.php
|   |-- content/content-single.php
|   |-- content/content-none.php
|   `-- footer/site-info.php
`-- assets/
    |-- fonts/
    |-- images/
    `-- js/
```

Classic theme compatibility:

- Keep WordPress-resolved template files at the theme root: `front-page.php`, `page.php`, `page-{slug}.php`, `single.php`, `archive.php`, `home.php`, `index.php`.
- Do not create FSE `.html` templates.
- Do not create a `patterns/` directory. Block layouts confuse this workflow and should stay out of generated theme structure.
- Do not add `customTemplates` or `templateParts` keys to `theme.json`.
- Do not add `add_theme_support( 'block-templates' )`.
- If a project wants a `templates/` directory, use it only for optional PHP assembly helpers, never for FSE `.html` templates.
- Do not create a root-level `parts/` directory. Fragments belong under `template-parts/` and are loaded with `get_template_part()`.

## Template Hierarchy Cheat Sheet

WordPress walks this hierarchy per request and uses the first file that exists:


| Request               | First match wins                                                        |
| --------------------- | ----------------------------------------------------------------------- |
| Homepage              | `front-page.php` -> `home.php` -> `index.php`                           |
| Blog index            | `home.php` -> `index.php`                                               |
| Single post           | `single-{post-type}.php` -> `single.php` -> `index.php`                 |
| Static page           | `page-{slug}.php` -> `page-{id}.php` -> `page.php` -> `index.php`       |
| Category / Tag / Date | `category-{slug}.php` -> `category.php` -> `archive.php` -> `index.php` |
| Search / 404          | `search.php` / `404.php` -> `index.php`                                 |


`front-page.php` short-circuits the homepage. `home.php` renders the blog posts index. `is_home()` is false on the homepage; use `is_front_page()` instead.

## theme.json Configuration

The central configuration for a hybrid theme. For more context check `./references/theme.json`. Generate `theme.json` entirely before CSS, components, layouts, or templates.

### Schema and Version

```json
{
  "$schema": "https://schemas.wp.org/trunk/theme.json",
  "version": 3
}
```

### Settings

```json
{
  "settings": {
    "appearanceTools": true,
    "layout": { "contentSize": "800px", "wideSize": "1280px" },
    "color": {
      "palette": [ /* 5-9 colors using design-provided hex, rgb(), or rgba(): primary, secondary, accent, base, base-2, contrast */ ],
      "defaultPalette": false,
      "defaultGradients": false
    },
    "typography": {
      "fontFamilies": [ /* heading + body font families */ ],
      "fontSizes": [ /* small/body plus heading-6 through heading-1 tokens */ ]
    },
    "spacing": {
      "units": ["px", "em", "rem", "%", "vw", "vh"],
      "spacingSizes": [ /* 6 steps from compact to spacious */ ]
    }
  }
}
```

### Styles

```json
{
  "styles": {
    "color": { /* background + text from palette */ },
    "typography": { /* body font family, medium size, line-height 1.5-1.65 */ },
    "elements": {
      "heading": { /* shared heading defaults only */ },
      "h1": { /* heading-1 size/weight/line-height */ },
      "h2": { /* heading-2 size/weight/line-height */ },
      "h3": { /* heading-3 size/weight/line-height */ },
      "h4": { /* heading-4 size/weight/line-height */ },
      "h5": { /* heading-5 size/weight/line-height */ },
      "h6": { /* heading-6 size/weight/line-height */ },
      "link": { /* accent color */ },
      "button": { /* accent background, light text, border-radius */ }
    }
  }
}
```

### Heading Elements

Define a shared `heading` baseline, then isolate `h1` through `h6`. Do not rely on grouped heading cascades for individual heading sizes or weights.

```json
{
  "styles": {
    "elements": {
      "heading": {
        "typography": {
          "fontFamily": "var(--wp--preset--font-family--heading)",
          "fontWeight": "400",
          "lineHeight": "1.2",
          "letterSpacing": "-0.02em"
        },
        "color": {
          "text": "var(--wp--preset--color--contrast)"
        }
      },
      "h1": {
        "typography": {
          "fontSize": "var(--wp--preset--font-size--heading-1)",
          "fontWeight": "400",
          "lineHeight": "1.18",
          "letterSpacing": "1.2em"
        }
      },
      "h2": {
        "typography": {
          "fontSize": "var(--wp--preset--font-size--heading-2)",
          "fontWeight": "500",
          "lineHeight": "1.3",
        }
      },
      "h3": {...},
      "h4": {...},
      "h5": {...},
      "h6": {...}
      }
    }
  }
}
```

Hybrid caveats:

- `theme.json` works in classic themes since WordPress 5.9.
- WordPress emits `--wp--preset--*` CSS variables on the front end.
- PHP templates can rely on color, font-size, font-family, spacing, content width, and wide width variables, such as:  `--wp--preset--color--{slug}`, `--wp--preset--font-size--{slug}`, `--wp--preset--font-family--{slug}`, `--wp--preset--spacing--{slug}`, `--wp--style--global--content-size`, `--wp--style--global--wide-size`
- `appearanceTools` belongs in `theme.json`, not `functions.php`.
- `functions.php` must call `add_theme_support( 'wp-block-styles' )` and `add_editor_style( 'style.css' )` so editor content matches the front end.

## Typography

- When a design file is available, use the design file first. Map font families, weights, sizes, and line heights into `theme.json`.
- Style `h1` through `h6` individually in `styles.elements`; keep `heading` only for shared defaults such as family/color.
- When the design specifies separate mobile (min) and desktop (max) font sizes for the same token or element, combine them in `theme.json` with `clamp(min, preferred, max)` — mobile value as the minimum, desktop as the maximum. Convert px to rem (÷ 16). Example: h1 at 32px mobile / 56px desktop → `"size": "clamp(2rem, 4vw + 1rem, 3.5rem)"`
- Convert px to rem by dividing by 16 unless the design system specifies otherwise.
- Fallback body size is `1rem`.
- **Line height**: Body text: 1.5–1.65. Headings: 1.1–1.3. Never below 1.0. Pull values from the design file when specified.

## Font Face Architecture

- Use WOFF2 only. Never download or declare `.ttf` or `.otf` files.
- Store fonts in `assets/fonts/` and declare them in `theme.json` `typography.fontFamilies[].fontFace`.
- Each `fontFace` entry should define `fontFamily`, `fontStyle`, `fontWeight`, `src`, and `fontDisplay: "swap"`.
- Use theme.json presets in CSS: `var(--wp--preset--font-family--heading)` and `var(--wp--preset--font-family--body)`.

## Components

Components are reusable UI primitives created in Phase 2 before templates.

Common components:

- Button
- Card
- Header
- Footer
- Navigation
- Mobile navigation
- Hero
- Form field
- Form message
- CTA block
- Blog card
- Pagination

Component rules:

- Prefer PHP functions or partials in `components/`.
- Prefix functions with the theme slug.
- Escape dynamic values by context.
- Use token-driven classes and CSS variables.
- Use semantic, descriptive class names. Do not use BEM naming conventions.
- Keep states close to the component implementation.
- Create forced-state helper classes for `component-library.php` when needed.
- Do not create every possible state for every component.
- Include baseline accessible states for standard interactive controls, such as buttons and links.
- Include behavior-specific states only when the component needs them, such as open/closed for mobile navigation or error/success for validated forms.
- Omit irrelevant states and report the omission when it may affect review.
- Do not hide missing states. Report them.

Example component function shape:

```php
<?php
function theme_slug_button( array $args = array() ) {
    $args = wp_parse_args( $args, array(
        'label'    => '',
        'url'      => '#',
        'variant'  => 'primary',
        'disabled' => false,
    ) );

    $classes = array( 'button', 'button--' . sanitize_html_class( $args['variant'] ) );
    if ( $args['disabled'] ) {
        $classes[] = 'is-disabled';
    }
    ?>
    <a class="<?php echo esc_attr( implode( ' ', $classes ) ); ?>"
       href="<?php echo esc_url( $args['url'] ); ?>"
       <?php echo $args['disabled'] ? 'aria-disabled="true" tabindex="-1"' : ''; ?>>
        <?php echo esc_html( $args['label'] ); ?>
    </a>
    <?php
}
```

## Component Library Page

Create a dedicated review template in Phase 2, usually `component-library.php` or `design-system.php`.

It should show:

- Every component.
- Every variant.
- Every available state.
- Every inferred state, labeled in the page or in the phase report.
- Desktop and mobile navigation examples.
- Open and closed menu states.
- Form states shown in the design or required by the form behavior.

Use helper classes such as `.is-hover`, `.is-focus`, `.is-current`, `.is-open`, and `.is-disabled` to make visual QA possible.

## Layouts

Layouts are reusable compositions created in Phase 3 from approved components.

Examples:

- Hero section
- Services grid
- Team grid
- CTA section
- Blog preview
- Testimonial row
- Contact band

Create a dedicated live review page in Phase 3, usually `layout-library.php`, `layout-showcase.php`, or `section-library.php`.

It should show:

- Every reusable layout.
- Relevant variants.
- Realistic sample content.
- Desktop and mobile behavior.
- The approved components used inside each layout.
- Any inferred layout behavior, documented in the phase report.

Layout rules:

- Put reusable layouts in `layouts/`.
- Use components from `components/`.
- Do not introduce new component states during layout work.
- Use `alignfull` wrappers and `alignwide` inner containers.
- Keep layout CSS token-driven.
- Do not generate production templates during Phase 3.
- Treat the layout showcase as a QA page, not as the homepage or a static page template.

## PHP Templates

PHP templates render front-end routes through the WordPress template hierarchy.

Standard loop-driven templates include `index.php`, `single.php`, `page.php`, `archive.php`, `search.php`, `home.php`, and `404.php`.

Canonical loop:

```php
<?php if ( have_posts() ) : ?>
    <?php while ( have_posts() ) : the_post(); ?>
        <?php get_template_part( 'template-parts/content/content', get_post_type() ); ?>
    <?php endwhile; ?>
    <?php the_posts_pagination(); ?>
<?php else : ?>
    <?php get_template_part( 'template-parts/content/content', 'none' ); ?>
<?php endif; ?>
```

Conditional tags:

- `is_front_page()`
- `is_home()`
- `is_singular()`
- `is_archive()`
- `is_category()`
- `is_search()`
- `is_404()`
- `has_post_thumbnail()`
- `comments_open()`
- `is_active_sidebar()`

## front-page.php

The homepage is a hand-written PHP template.

Rules:

- Use plain semantic HTML and PHP.
- Do not use block comments.
- Do not call `the_content()` against the main query.
- Use approved layouts and components from `layouts/` and `components/`.
- Use `home_url()`, `get_theme_file_uri()`, and `esc_url()` for links and assets.
- Use `alignfull` and `alignwide`.
- Use `wp-element-button` or approved button components for CTAs.
- Use `WP_Query` for dynamic strips when needed and call `wp_reset_postdata()`.

## page-{slug}.php

Use one hand-written PHP template per static page in the site navigation when a specific layout is required.

Rules match `front-page.php`:

- No block markup.
- No main-query `the_content()`.
- Approved layouts/components only.
- Dynamic sub-queries are allowed.
- Database page rows are routing anchors.

## Header, Footer, Sidebar, and Comments

`header.php` must include:

- `language_attributes()`
- `bloginfo( 'charset' )`
- responsive viewport meta
- `wp_head()`
- `body_class()`
- `wp_body_open()`
- skip link
- accessible primary navigation

Do not add a literal `<title>` tag. `add_theme_support( 'title-tag' )` lets WordPress emit it.

`footer.php` must include:

- Footer navigation when registered.
- Current year and site name.
- `wp_footer()` before `</body>`.

`sidebar.php` is opt-in. Only create it if `functions.php` registers a widget area and at least one template calls `get_sidebar()`.

`comments.php` should be minimal and respect WordPress discussion settings.

## functions.php

`functions.php` declares theme supports, menus, sidebars, pattern categories, and assets.

Required setup:

```php
add_theme_support( 'title-tag' );
add_theme_support( 'post-thumbnails' );
add_theme_support( 'automatic-feed-links' );
add_theme_support( 'html5', array( 'search-form', 'comment-form', 'comment-list', 'gallery', 'caption', 'style', 'script', 'navigation-widgets' ) );
add_theme_support( 'responsive-embeds' );
add_theme_support( 'wp-block-styles' );
add_theme_support( 'align-wide' );
add_editor_style( 'style.css' );
```

Rules:

- Register primary and footer menus when used.
- Register sidebars only when used.
- Enqueue `style.css` with `wp_enqueue_style()`.
- Do not enqueue Google Fonts CSS.
- Do not register block pattern categories.
- Do not add `add_theme_support( 'block-templates' )`.
- Add front-end JavaScript only when needed.

## Escaping in PHP Templates

Escape every dynamic value by context.


| Context                   | Function                        |
| ------------------------- | ------------------------------- |
| HTML body text            | `esc_html()`                    |
| HTML attribute value      | `esc_attr()`                    |
| URL                       | `esc_url()`                     |
| Inline JavaScript value   | `esc_js()`                      |
| Translated body text      | `esc_html__()` / `esc_html_e()` |
| Translated attribute text | `esc_attr__()` / `esc_attr_e()` |


Already escaping or safe to call directly in normal contexts:

- `the_title()`
- `the_permalink()`
- `the_content()`
- `the_excerpt()`
- `wp_nav_menu()`
- `the_custom_logo()`
- `wp_head()`
- `wp_footer()`

Usually wrap with an escaping function:

- `get_the_title()`
- `get_permalink()`
- `get_the_excerpt()`
- `bloginfo( 'description' )`
- `get_post_meta()`
- `get_search_query()`

Banned:

- `eval()`
- `create_function()`
- `shell_exec()`
- `exec()`
- `system()`
- `extract()` on user data

## style.css

`style.css` contains:

- Required WordPress theme header.
- Layout and component CSS not covered by `theme.json`.
- State styling.
- Responsive behavior.
- Accessibility helpers.
- Motion classes.
- Modern Grid and Flexbox layout. Avoid legacy float/position fallbacks.
- Native CSS nesting when it keeps related selectors readable.
- Semantic, descriptive class names; reject BEM naming.
- Tablet breakpoint: `@media (max-width: 1024px)`.
- Mobile breakpoint: `@media (max-width: 600px)`.

Baseline CSS:

```css
.alignfull {
  width: 100%;
}

.alignwide {
  max-width: var(--wp--style--global--wide-size);
  margin-inline: auto;
  padding-inline: var(--wp--preset--spacing--40, 1rem);
}

.site-footer {
  margin-block-start: 0;
}

.site-main > :first-child {
  margin-block-start: 0;
}

.skip-link.screen-reader-text {
  position: absolute;
  left: -9999px;
  top: auto;
  width: 1px;
  height: 1px;
  overflow: hidden;
}

.skip-link.screen-reader-text:focus {
  left: 6px;
  top: 7px;
  width: auto;
  height: auto;
  z-index: 100000;
}
```

Do not apply inline left/right padding to first-level section wrappers. Center constrained inner containers with `max-width` and `margin: 0 auto`; apply padding on inner containers when needed.

## Animation and Motion

Animation classes work across PHP templates and editor-rendered content.

Rules:

- Prefer subtle motion.
- Prioritize hero entrance, major section reveals, and hover transitions on cards/buttons.
- Do not animate every heading or paragraph individually.
- Include `prefers-reduced-motion`.
- Include editor visibility overrides for any class that starts hidden.

Required reduced motion guard:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }

  .animate-on-scroll,
  .fade-up,
  .fade-in,
  .slide-in-left,
  .slide-in-right,
  .scale-up {
    opacity: 1 !important;
    transform: none !important;
  }
}
```

Required editor visibility pattern:

```css
.editor-styles-wrapper .fade-up,
.editor-styles-wrapper .fade-in,
.editor-styles-wrapper .slide-in-left,
.editor-styles-wrapper .slide-in-right,
.editor-styles-wrapper .scale-up,
.editor-styles-wrapper .animate-on-scroll {
  opacity: 1 !important;
  transform: none !important;
  animation: none !important;
  transition: none !important;
}
```

## Card Layouts

For equal-height PHP template cards:

```css
.service-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--wp--preset--spacing--50, 2rem);
}

.service-card {
  display: flex;
  flex-direction: column;
}

.service-card .cta-bottom {
  margin-top: auto;
}
```

## Landing Page Composition

When no design file is available, think like a landing page designer, not a template assembler.

Rules:

- Use full-bleed section wrappers and constrained inner content.
- Create visual rhythm between adjacent sections.
- Choose sections based on the site's goal.
- Use context-specific sections such as project galleries, service grids, case studies, menus, features, testimonials, social proof, and CTAs.
- Avoid decorative HTML comments.
- Avoid introducing layout patterns after Phase 3 approval.

PHP section pattern:

```html
<section class="alignfull section-name">
  <div class="alignwide section-name__inner">
    ...
  </div>
</section>
```

## Image Handling

Only add user-provided images or image URLs to the initial site build unless the user explicitly asks for sourced or generated imagery.

When no images are available, create visual richness through:

- CSS gradients
- Color blocks
- Typography
- CSS patterns
- Shadows
- Borders
- Spacing
- Pseudo-elements
- Color overlays

Do not force unrelated stock imagery into the theme.

## Phase Reporting

At the end of each phase, use the report format from `theme-creation-flow.md`.

Always include:

- Files created or changed.
- Design sources used.
- Components or states inferred.
- Assumptions.
- Required next approval phrase.

If a requested template needs a component that was not approved, stop and report the new need instead of implementing it silently.