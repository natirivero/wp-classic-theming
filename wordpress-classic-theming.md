---

## name: wordpress-classic-theming

description: WordPress Hybrid theme architecture. Use when generating theme.json, PHP templates, template parts, patterns, and functions.php for WordPress classic (hybrid) themes.

# WordPress Classic (Hybrid) Theming Skill

Comprehensive knowledge for building WordPress **hybrid** themes — PHP templates as the source of truth, `theme.json` as the design system, block patterns and the post editor for editorial content. The repo is the source of truth; the database manages post content only.

## Absolute Rules

- **NO DUPLICATED VARIABLES**: Do not redeclare in `style.css` any color, spacing, or typography value already defined in `theme.json`. Use the matching `--wp--preset--`* CSS custom property instead.
- **NO HARD-CODED FONT SIZES**: Do not set `font-size` in `style.css` or inline in HTML. Use `--wp--preset--font-size--`* variables.
- **NO HARD-CODED SPACING**: Do not use literal `padding` or `margin` values. Use `--wp--preset--spacing--`* variables.
- **NO MIXED OUTPUT**: PHP templates (`front-page.php`, `index.php`, `single.php`, `page.php`, `header.php`, `footer.php`, etc.) are plain HTML/PHP. Never insert `<!-- wp:* -->` comments into PHP templates. Block markup lives only in `patterns/*.php` and in post content saved through the editor.
- **LOAD GOOGLE FONTS LOCALLY, NOT FROM CDN**: Download woff2 files into `assets/fonts/`, declare each family with `fontFace` in `theme.json` (WordPress emits `@font-face` on the front end and in the editor), and never enqueue Google Fonts CSS from `fonts.googleapis.com` or `fonts.gstatic.com`. See "Local Fonts via theme.json".
- **PAGE CONTENT LIVES IN PHP TEMPLATES, NOT IN THE DATABASE**: Static pages (Home, About, Services, Contact, etc.) may exist in the database — the matching PHP template always wins for the route. Write each page's layout and copy directly in its template — `front-page.php` for the homepage, `page-{slug}.php` (e.g., `page-about.php`, `page-services.php`, `page-contact.php`) for every other static page — never source a page body from `the_content()` against the main query. Sub-queries (`new WP_Query()`) for self-contained strips are fine.

## Theme Architecture

### Directory Structure

```
theme-slug/
├── style.css              # REQUIRED. Theme metadata header + custom CSS.
├── theme.json             # REQUIRED. Design system; emits --wp--preset--* CSS vars on the front end.
├── functions.php          # REQUIRED. Theme supports, enqueues, menu/sidebar/pattern-category registration.
├── index.php              # REQUIRED. Ultimate fallback template; runs the loop.
├── header.php             # REQUIRED. Loaded by get_header().
├── footer.php             # REQUIRED. Loaded by get_footer().
├── front-page.php         # REQUIRED. Code-first HTML landing page template.
├── home.php               # REQUIRED. Blog posts index.
├── page.php               # REQUIRED. Generic fallback for pages without a per-slug template.
├── page-{slug}.php        # PREFERRED per static page (page-about.php, page-services.php, page-contact.php) — code-first layout, may include dynamic loops.
├── single.php             # REQUIRED. Single post template.
├── archive.php            # REQUIRED. Category/tag/date archives.
├── search.php             # REQUIRED. Search results page.
├── 404.php                # REQUIRED. Custom 404.
├── comments.php           # INCLUDED. Minimal skeleton; respects discussion settings.
├── sidebar.php            # INCLUDED. Opt-in via is_active_sidebar() guard.
├── searchform.php         # OPTIONAL. Override of get_search_form() output.
├── screenshot.png         # RECOMMENDED. 1200×900 PNG shown in Appearance → Themes.
├── template-parts/
│   ├── header/site-branding.php
│   ├── content/content.php
│   ├── content/content-page.php
│   ├── content/content-single.php
│   ├── content/content-none.php
│   └── footer/site-info.php
├── patterns/              # Auto-discovered by WP 6.0+ (no PHP registration needed).
│   ├── hero.php
│   ├── features.php
│   └── cta.php
└── assets/                # Referenced via get_theme_file_uri().
    ├── images/
    └── js/
```

**Negative rules:**

- Do **NOT** create a `templates/` directory or any `.html` template files.
- Do **NOT** create a root-level `parts/` directory — fragments belong under `template-parts/` via get_template_part()
- Do **NOT** add `customTemplates` or `templateParts` keys to `theme.json` — those are FSE-only.

### Template Hierarchy Cheat-Sheet

WordPress walks this hierarchy per request and uses the first file that exists:


| Request               | First match wins                                                     |
| --------------------- | -------------------------------------------------------------------- |
| Homepage              | `front-page.php` → `home.php` → `index.php`                          |
| Blog index            | `home.php` → `index.php`                                             |
| Single post           | `single-{post-type}.php` → `single.php` → `index.php`                |
| Static page           | `page-{slug}.php` → `page-{id}.php` → `page.php` → `index.php`       |
| Category / Tag / Date | `category-{slug}.php` → `category.php` → `archive.php` → `index.php` |
| Search / 404          | `search.php` / `404.php` → `index.php`                               |


`front-page.php` short-circuits the homepage, and `home.php` renders the blog posts index. `is_home()` is **false** on the homepage — use `is_front_page()` instead.

## theme.json Configuration

The central configuration for a hybrid theme. For more context check ./references/theme.json

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
      "palette": [ /* 5-9 colors: primary, secondary, accent, base, base-2, contrast */ ],
      "defaultPalette": false,
      "defaultGradients": false
    },
    "typography": {
      "fontFamilies": [ /* heading + body font families */ ],
      "fontSizes": [ /* 5-8 step scale: small through huge (sm, p or md, h6-h1) */ ]
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
      "heading": { /* heading font family, line-height 1.1-1.3 */ },
      "link": { /* accent color */ },
      "button": { /* accent background, light text, border-radius */ }
    }
  }
}
```

### Hybrid Theme Caveats

`theme.json` works in classic themes since WP 5.9. Since WP 6.1, `--wp--preset--*` CSS custom properties are emitted on the front end automatically — so PHP templates can use the same design tokens as block patterns.

**CSS variables PHP templates can rely on:** `--wp--preset--color--{slug}`, `--wp--preset--font-size--{slug}`, `--wp--preset--font-family--{slug}`, `--wp--preset--spacing--{slug}`, `--wp--style--global--content-size`, `--wp--style--global--wide-size`. Hand-written `.alignwide` containers should reference `--wp--style--global--wide-size`; `.alignfull` should be `width: 100%`. Don't hard-code `1280px` anywhere.

- `**appearanceTools: true`** lives in `theme.json`, not `functions.php`.
- **For theme.json defaults to apply to `the_content()` block output**, `functions.php` must call `add_theme_support( 'wp-block-styles' )` and `add_editor_style( 'style.css' )`.
- **Do NOT add `customTemplates` or `templateParts` keys** — FSE-only.

## Typography

- **Design file first**: When a design file is available (Figma, Pencil, approved HTML mockups, or `design-tokens.json` from the design package), use its font families, weights, and sizes as the source of truth. Map them into `theme.json` `fontFamilies` and `fontSizes` — they override the generic defaults below.
- **Font size scale** (fallback when no design file): Body: 1rem. Headings: scale modestly (h1 ≤ 2.5–3rem). Cap display text at ~3.5rem max. Avoid sizes above 4rem. A good 6-step scale: 0.875rem / 1rem / 1.25rem / 1.75rem / 2.25rem / clamp(2.5rem, 4vw, 3.5rem).
- **Responsive sizes with `clamp()`**: When the design specifies separate mobile (min) and desktop (max) font sizes for the same token or element, combine them in `theme.json` with `clamp(min, preferred, max)` — mobile value as the minimum, desktop as the maximum. Convert px to rem (÷ 16). Example: h1 at 32px mobile / 56px desktop → `"size": "clamp(2rem, 4vw + 1rem, 3.5rem)"`. Use a single fixed size when the design gives only one breakpoint value.
- **Line height**: Body text: 1.5–1.65. Headings: 1.1–1.3. Never below 1.0. Pull values from the design file when specified.

## Cover Block Pitfalls

Applies to `wp:cover` in block patterns and post content. For the hero in `front-page.php`, use plain CSS (`min-height: 60vh; display: flex; align-items: center;`) — same outcome, no block.

- **Use `60vh`** as the default cover block height — enough visual presence without wasted space. `minHeight` only accepts a number + unit, not `clamp()`.
- Keep top padding modest (~5rem) — the cover's flexbox centering handles vertical positioning.
- Decorative badges (`— Eyebrow —`) must use `display: flex; justify-content: center` — never `inline-flex` (which left-aligns inside cover blocks).

## PHP Templates

PHP templates render every front-end response. WordPress chooses one per request via the template hierarchy.

### The Loop

Canonical pattern used by every template that lists posts:

```php
<?php if ( have_posts() ) : ?>
    <?php while ( have_posts() ) : the_post(); ?>
        <article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
            <header class="entry-header">
                <?php the_title( '<h1 class="entry-title">', '</h1>' ); ?>
            </header>
            <div class="entry-content">
                <?php the_content(); ?>
            </div>
        </article>
    <?php endwhile; ?>
<?php else : ?>
    <p><?php esc_html_e( 'Nothing found.', 'theme-slug' ); ?></p>
<?php endif; ?>
```

### Conditional Tags

Branch behavior in templates: `is_front_page()`, `is_home()` (blog index — **false** on homepage), `is_singular( 'post' | 'page' )`, `is_archive()`, `is_category()`, `is_search()`, `is_404()`, `has_post_thumbnail()`, `comments_open()`, `is_active_sidebar( 'sidebar-1' )`.

### Standard loop-driven templates

`index.php`, `single.php`, `page.php`, `archive.php`, `search.php`, `home.php`, `404.php` all follow the same shape — `get_header()`, a main element, the loop (via a `template-parts/content/`* fragment), pagination, `get_footer()`. One representative skeleton:

```php
<?php get_header(); ?>

<main id="primary" class="site-main">
    <?php if ( have_posts() ) : ?>
        <?php while ( have_posts() ) : the_post(); ?>
            <?php get_template_part( 'template-parts/content/content', get_post_type() ); ?>
        <?php endwhile; ?>
        <?php the_posts_pagination(); ?>
    <?php else : ?>
        <?php get_template_part( 'template-parts/content/content', 'none' ); ?>
    <?php endif; ?>
</main>

<?php get_footer(); ?>
```

Per-template differences from the skeleton above:


| Template      | Difference                                                                                                                                     |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `single.php`  | Drop the `if ( have_posts() )` outer guard. Inside the loop, add `the_post_thumbnail()` and `comments_template()` (guarded by `comments_open() |
| `page.php`    | Generic fallback — `front-page.php` and `page-{slug}.php` win first. Drops pagination. Falls back to `the_content()` from the saved page row.  |
| `archive.php` | Add a `<header class="page-header alignwide">` with `the_archive_title()` and `the_archive_description()` before the loop.                     |
| `search.php`  | Page header uses `printf( esc_html__( 'Search results for: %s' ), esc_html( get_search_query() ) )`.                                           |
| `home.php`    | Page header titled "Latest posts". Loaded when the user navigates to the assigned Posts page.                                                  |
| `404.php`     | No loop. Hand-written copy + `home_url()` link + `get_search_form()`.                                                                          |


### `front-page.php` (hand-written homepage)

The key template for this skill. Plain semantic HTML — no block markup, no `the_content()` against the main query. `WP_Query` sub-queries (latest posts, featured testimonials) are allowed; call `wp_reset_postdata()` after each.

**Rules:**

- No `<!-- wp:* -->` comments anywhere.
- Write the homepage's layout and copy directly as HTML — never `the_content()` on the main query.
- Use `home_url()` / `get_theme_file_uri()` / `esc_url()` for every link and asset reference.
- Use `alignfull` / `alignwide` so the layout matches the editor's wide/full widths.
- Use `wp-element-button` on CTA links so theme.json button styles apply identically to block-rendered buttons.
- Animation, typography, and card classes are the same set used by patterns and `style.css`.

```php
<?php
/**
 * The front page. Hand-written HTML, no block markup.
 */
get_header();
?>

<main id="primary" class="site-main front-page">

    <section class="alignfull hero fade-up">
        <div class="alignwide hero__inner">
            <p class="hero__eyebrow">— Eyebrow —</p>
            <h1 class="hero__title">Headline</h1>
            <p class="hero__lede">Supporting paragraph.</p>
            <p class="hero__cta">
                <a class="wp-element-button" href="<?php echo esc_url( home_url( '/contact/' ) ); ?>">
                    <?php esc_html_e( 'Primary CTA', 'theme-slug' ); ?>
                </a>
            </p>
        </div>
    </section>

    <section class="alignfull features animate-on-scroll">
        <div class="alignwide features__grid">
            <article class="feature-card">
                <h2><?php esc_html_e( 'Fast', 'theme-slug' ); ?></h2>
                <p><?php esc_html_e( 'Lightning-quick load times.', 'theme-slug' ); ?></p>
            </article>
            <!-- repeat -->
        </div>
    </section>

    <?php
    $recent = new WP_Query( array( 'posts_per_page' => 3, 'no_found_rows' => true ) );
    if ( $recent->have_posts() ) :
        ?>
        <section class="alignfull recent-posts animate-on-scroll">
            <div class="alignwide">
                <h2><?php esc_html_e( 'Latest from the blog', 'theme-slug' ); ?></h2>
                <ul class="recent-posts__grid">
                    <?php while ( $recent->have_posts() ) : $recent->the_post(); ?>
                        <li><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></li>
                    <?php endwhile; ?>
                </ul>
            </div>
        </section>
        <?php
    endif;
    wp_reset_postdata();
    ?>

</main>

<?php get_footer(); ?>
```

### `page-{slug}.php` (hand-written page templates)

Preferred for every static page in the site's navigation (About, Services, Contact, etc.). Same pattern as `front-page.php`: code-first layout and copy directly in PHP/HTML, no `<!-- wp:* -->`, no `the_content()` against the main query. Dynamic content is welcome — a team loop, a services grid, a related-posts row — driven by `WP_Query` inside the template. The corresponding "About" / "Services" / "Contact" page rows in the database are just routing anchors.

Rules are identical to `front-page.php` (no block markup, no main-query `the_content()`, `alignfull`/`alignwide` for layout, `wp-element-button` for CTAs, sub-queries fine). Example:

```php
<?php
/**
 * Template: page-about.php
 */
get_header();
?>

<main id="primary" class="site-main page-about">

    <section class="alignfull page-hero fade-up">
        <div class="alignwide page-hero__inner">
            <p class="page-hero__eyebrow">— <?php esc_html_e( 'About us', 'theme-slug' ); ?> —</p>
            <h1 class="page-hero__title"><?php esc_html_e( 'Who we are', 'theme-slug' ); ?></h1>
            <p class="page-hero__lede"><?php esc_html_e( 'One-line value proposition.', 'theme-slug' ); ?></p>
        </div>
    </section>

    <section class="alignfull about-story animate-on-scroll">
        <div class="alignwide about-story__grid">
            <div class="about-story__copy">
                <h2><?php esc_html_e( 'Our story', 'theme-slug' ); ?></h2>
                <p><?php esc_html_e( 'Hand-written narrative paragraph.', 'theme-slug' ); ?></p>
            </div>
            <figure class="about-story__media">
                <img src="<?php echo esc_url( get_theme_file_uri( 'assets/images/about.jpg' ) ); ?>"
                     alt="<?php esc_attr_e( 'Our team at work', 'theme-slug' ); ?>" />
            </figure>
        </div>
    </section>

</main>

<?php get_footer(); ?>
```

The same pattern repeats for `page-services.php`, `page-contact.php`, and so on — one PHP template per static page in the navigation. A request for `/about/` resolves the page row, the hierarchy picks `page-about.php`, and the hand-written HTML is what users see.

## Template Parts (Header, Footer, Sidebar, Fragments)

### `header.php`

```php
<!doctype html>
<html <?php language_attributes(); ?>>
<head>
    <meta charset="<?php bloginfo( 'charset' ); ?>">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <?php wp_head(); ?>
</head>

<body <?php body_class(); ?>>
<?php wp_body_open(); ?>

<a class="skip-link screen-reader-text" href="#primary"><?php esc_html_e( 'Skip to content', 'theme-slug' ); ?></a>

<header id="masthead" class="site-header alignfull">
    <div class="alignwide site-header__inner">
        <div class="site-branding">
            <?php if ( has_custom_logo() ) : ?>
                <?php the_custom_logo(); ?>
            <?php else : ?>
                <p class="site-title"><a href="<?php echo esc_url( home_url( '/' ) ); ?>" rel="home"><?php bloginfo( 'name' ); ?></a></p>
            <?php endif; ?>
        </div>
        <nav class="site-navigation" aria-label="<?php esc_attr_e( 'Primary', 'theme-slug' ); ?>">
            <?php wp_nav_menu( array(
                'theme_location' => 'primary',
                'container'      => false,
                'menu_class'     => 'primary-menu',
                'depth'          => 2,
                'fallback_cb'    => 'wp_page_menu',
            ) ); ?>
        </nav>
    </div>
</header>
```

**Non-negotiable:** `language_attributes()`, `wp_head()` before `</head>`, `body_class()`, `wp_body_open()` right after `<body>`. **No literal `<title>` tag** — `wp_head()` emits it because `add_theme_support( 'title-tag' )` is set.

### `footer.php`

```php
    <footer id="colophon" class="site-footer alignfull">
        <div class="alignwide site-footer__inner">
            <?php wp_nav_menu( array(
                'theme_location' => 'footer',
                'container'      => false,
                'menu_class'     => 'footer-menu',
                'depth'          => 1,
                'fallback_cb'    => false,
            ) ); ?>
            <p class="site-info">&copy; <?php echo esc_html( gmdate( 'Y' ) ); ?> <?php bloginfo( 'name' ); ?>.</p>
        </div>
    </footer>

<?php wp_footer(); ?>
</body>
</html>
```

`wp_footer()` before `</body>` is non-negotiable.

### `sidebar.php` (opt-in)

Only create this file if `functions.php` registers a widget area AND at least one template calls `get_sidebar()`.

```php
<?php if ( ! is_active_sidebar( 'sidebar-1' ) ) return; ?>
<aside id="secondary" class="widget-area" role="complementary">
    <?php dynamic_sidebar( 'sidebar-1' ); ?>
</aside>
```

### `comments.php`

Minimal skeleton — respects WP discussion settings:

```php
<?php
if ( post_password_required() ) return;
?>
<section id="comments" class="comments-area">
    <?php if ( have_comments() ) : ?>
        <h2 class="comments-title">
            <?php printf( esc_html( _n( '1 comment', '%s comments', get_comments_number(), 'theme-slug' ) ), esc_html( number_format_i18n( get_comments_number() ) ) ); ?>
        </h2>
        <ol class="comment-list">
            <?php wp_list_comments( array( 'style' => 'ol', 'short_ping' => true ) ); ?>
        </ol>
        <?php the_comments_navigation(); ?>
    <?php endif; ?>
    <?php comment_form(); ?>
</section>
```

### Fragments via `get_template_part()`

Always under `template-parts/`. Use to dedupe article markup across `single.php` / `index.php` / `archive.php`:

```php
get_template_part( 'template-parts/content/content', get_post_type() );
```

Resolves to `template-parts/content/content-post.php` (with fallback `content.php`) for the `post` post type.

## Block Patterns

Patterns are PHP files in `/patterns/`, auto-discovered by WP 6.0+. Each starts with a header docblock:

```php
<?php
/**
 * Title: Hero Section          (required)
 * Slug: theme-slug/hero        (required; must be namespaced)
 * Categories: featured         (optional, comma-separated)
 * Keywords: hero, intro        (optional)
 * Block Types: core/post-content   (optional)
 * Post Types: page, post       (optional)
 * Inserter: true               (optional; false = registered but hidden)
 * Description: One-line description.
 * Viewport Width: 1400         (optional preview width)
 */
?>
<!-- wp:group {"backgroundColor":"primary","textColor":"light","layout":{"type":"constrained"}} -->
...
<!-- /wp:group -->
```

Missing `Title:` or `Slug:` = silently skipped. **Do NOT** loop through `/patterns/` and call `register_block_pattern()` manually — auto-discovery handles registration. `functions.php` only needs to register a custom pattern category.

## functions.php

`functions.php` declares theme supports, registers menus/sidebars/pattern categories, enqueues assets, and outputs the scroll-observer script.

```php
<?php
/**
 * Theme functions and definitions.
 */

function theme_slug_setup() {
    add_theme_support( 'title-tag' );
    add_theme_support( 'post-thumbnails' );
    add_theme_support( 'automatic-feed-links' );
    add_theme_support( 'html5', array( 'search-form', 'comment-form', 'comment-list', 'gallery', 'caption', 'style', 'script', 'navigation-widgets' ) );
    add_theme_support( 'responsive-embeds' );
    add_theme_support( 'wp-block-styles' );
    add_theme_support( 'align-wide' );
    add_theme_support( 'custom-logo', array( 'height' => 100, 'width' => 400, 'flex-height' => true, 'flex-width' => true ) );
    add_editor_style( 'style.css' );
    load_theme_textdomain( 'theme-slug', get_template_directory() . '/languages' );

    register_nav_menus( array(
        'primary' => __( 'Primary', 'theme-slug' ),
        'footer'  => __( 'Footer',  'theme-slug' ),
    ) );
}
add_action( 'after_setup_theme', 'theme_slug_setup' );

// Sidebars — only if a template calls get_sidebar().
function theme_slug_widgets_init() {
    register_sidebar( array(
        'name'          => __( 'Sidebar', 'theme-slug' ),
        'id'            => 'sidebar-1',
        'before_widget' => '<section id="%1$s" class="widget %2$s">',
        'after_widget'  => '</section>',
        'before_title'  => '<h2 class="widget-title">',
        'after_title'   => '</h2>',
    ) );
}
add_action( 'widgets_init', 'theme_slug_widgets_init' );

// Front-end style.css. Fonts come from theme.json fontFace — no font enqueue here.
function theme_slug_enqueue_assets() {
    wp_enqueue_style( 'theme-slug-style', get_stylesheet_uri(), array(), wp_get_theme()->get( 'Version' ) );
}
add_action( 'wp_enqueue_scripts', 'theme_slug_enqueue_assets' );

// Custom pattern category. Patterns themselves are auto-discovered from /patterns/.
function theme_slug_register_pattern_category() {
    register_block_pattern_category( 'theme-slug', array( 'label' => __( 'Theme Patterns', 'theme-slug' ) ) );
}
add_action( 'init', 'theme_slug_register_pattern_category' );

// Scroll-animation observer — front end only, via wp_footer.
function theme_slug_scroll_animations() {
    ?>
    <script>
    document.addEventListener('DOMContentLoaded', function() {
        var els = document.querySelectorAll('.animate-on-scroll');
        if (!els.length) return;
        var observer = new IntersectionObserver(function(entries) {
            entries.forEach(function(entry) {
                if (entry.isIntersecting) {
                    entry.target.classList.add('is-visible');
                    observer.unobserve(entry.target);
                }
            });
        }, { threshold: 0.15 });
        els.forEach(function(el) { observer.observe(el); });
    });
    </script>
    <?php
}
add_action( 'wp_footer', 'theme_slug_scroll_animations' );
```

**Notes:**

- `appearanceTools` lives in `theme.json`, not here.
- **Do NOT** add `add_theme_support( 'block-templates' )` — that opts into WP 6.7+ PHP-registered block templates and conflicts with this architecture.
- `add_editor_style()` implies `add_theme_support( 'editor-styles' )`.
- Fonts come from `theme.json` `fontFace` (see "Local Fonts via theme.json"). No font enqueues here, no editor-asset hook for fonts. If you ever need editor-only JS/CSS *beyond* fonts, add an `enqueue_block_editor_assets` hook — but `add_editor_style( 'style.css' )` already covers the editor stylesheet, so most themes don't need it.

## Escaping in PHP Templates

Every dynamic value output by a PHP template must be escaped in its context.


| Context                               | Function                                                          |
| ------------------------------------- | ----------------------------------------------------------------- |
| HTML body text                        | `esc_html()`                                                      |
| HTML attribute value                  | `esc_attr()`                                                      |
| URL (href/src)                        | `esc_url()`                                                       |
| Inline JavaScript value               | `esc_js()`                                                        |
| Translated string in body / attribute | `esc_html__()` / `esc_html_e()` — `esc_attr__()` / `esc_attr_e()` |


**Already escaping** (safe to call directly): `the_title()`, `the_permalink()`, `the_content()`, `the_excerpt()`, `wp_nav_menu()`, `the_custom_logo()`, `wp_head()`, `wp_footer()`, `bloginfo()`.

**Need escaping** (wrap with `esc_`*): `get_the_title()`, `get_permalink()`, `get_the_excerpt()`, `bloginfo('description')`, `get_post_meta()`, `get_search_query()`.

**Banned:** `eval()`, `create_function()`, `shell_exec()`, `exec()`, `system()`, `extract()` on user data.

## style.css

```css
/*
Theme Name: Theme Name
Theme URI: https://example.com
Author: Author Name
Description: A beautiful WordPress hybrid theme.
Version: 1.0.0
Requires at least: 6.1
Tested up to: 6.7
Requires PHP: 7.4
License: GNU General Public License v2 or later
Text Domain: theme-slug
*/
```

Always bring across custom CSS from the design into `style.css` that isn't achievable via `theme.json`, especially layout/composition techniques critical to the design's aesthetics (animation, motion, decorative effects).

### Baseline rules every hybrid theme needs

```css
/* Layout containers shared by PHP templates and block content */
.alignfull { width: 100%; }
.alignwide {
  max-width: var(--wp--style--global--wide-size);
  margin-inline: auto;
  padding-inline: var(--wp--preset--spacing--40, 1rem);
}

/* Top-margin resets */
.site-footer { margin-block-start: 0; }
.site-main > :first-child { margin-block-start: 0; }

/* Accessibility: skip link */
.skip-link.screen-reader-text {
  position: absolute; left: -9999px; top: auto;
  width: 1px; height: 1px; overflow: hidden;
}
.skip-link.screen-reader-text:focus {
  left: 6px; top: 7px; width: auto; height: auto; z-index: 100000;
}
```

## Animation & Motion

Animation classes in `style.css` work across two surfaces:

- **PHP templates and `front-page.php`**: apply classes directly on HTML elements (`<section class="alignfull fade-up">`).
- **Block patterns and post-editor content**: apply classes via the `className` JSON attribute on the block, e.g. `<!-- wp:group {"className":"fade-up","align":"full"} -->`.

### Animation classes (`style.css`)

Generate and adapt these — examples only, don't limit yourself to them.

**Entrance animations:**

```css
.fade-up { opacity: 0; transform: translateY(30px); animation: fadeUp 0.6s ease forwards; }
.fade-in { opacity: 0; animation: fadeIn 0.6s ease forwards; }
.slide-in-left { opacity: 0; transform: translateX(-40px); animation: slideIn 0.7s ease forwards; }
.slide-in-right { opacity: 0; transform: translateX(40px); animation: slideIn 0.7s ease forwards; }

@keyframes fadeUp { to { opacity: 1; transform: translateY(0); } }
@keyframes fadeIn { to { opacity: 1; } }
@keyframes slideIn { to { opacity: 1; transform: translateX(0); } }
```

**Staggered children** (delays via nth-child):

```css
.stagger-children > * { opacity: 0; transform: translateY(20px); animation: fadeUp 0.5s ease forwards; }
.stagger-children > *:nth-child(1) { animation-delay: 0.1s; }
.stagger-children > *:nth-child(2) { animation-delay: 0.2s; }
.stagger-children > *:nth-child(3) { animation-delay: 0.3s; }
.stagger-children > *:nth-child(4) { animation-delay: 0.4s; }
```

**Interactive transitions:**

```css
.hover-lift { transition: transform 0.2s ease, box-shadow 0.2s ease; }
.hover-lift:hover { transform: translateY(-4px); box-shadow: 0 12px 24px rgba(0,0,0,0.15); }
.hover-glow { transition: box-shadow 0.3s ease; }
.hover-glow:hover { box-shadow: 0 0 20px rgba(var(--wp--preset--color--accent-rgb, 0,0,0), 0.3); }
```

**Continuous ambient motion:**

```css
.float { animation: float 3s ease-in-out infinite; }
@keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
.pulse-subtle { animation: pulse 2s ease-in-out infinite; }
@keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.7; } }
```

### Scroll-triggered reveals

The IntersectionObserver script in `functions.php` (`wp_footer` hook) adds `.is-visible` to elements with `.animate-on-scroll` as they enter the viewport. Pair with:

```css
.animate-on-scroll { opacity: 0; transform: translateY(30px); transition: opacity 0.6s ease, transform 0.6s ease; }
.animate-on-scroll.is-visible { opacity: 1; transform: translateY(0); }
```

Apply `animate-on-scroll` to section-level wrappers in PHP templates (`<section class="alignfull animate-on-scroll">`) or via `className` in block patterns.

### prefers-reduced-motion (Required)

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
  .animate-on-scroll, .fade-up, .fade-in, .slide-in-left, .slide-in-right, .scale-up {
    opacity: 1 !important;
    transform: none !important;
  }
}
```

### Editor visibility (Required)

The IntersectionObserver runs only on the front end. Without overrides, blocks in post content / patterns that set `opacity: 0` would be invisible in the editor. WordPress wraps editor content in `<div class="editor-styles-wrapper">`, so:

```css
.editor-styles-wrapper .fade-up,
.editor-styles-wrapper .fade-in,
.editor-styles-wrapper .slide-in-left,
.editor-styles-wrapper .slide-in-right,
.editor-styles-wrapper .scale-up,
.editor-styles-wrapper .animate-on-scroll,
.editor-styles-wrapper .stagger-children > * {
  opacity: 1 !important;
  transform: none !important;
  animation: none !important;
  transition: none !important;
}
```

Every entrance class that starts hidden needs a matching `.editor-styles-wrapper` override.

### How much animation

Not every element needs it. Prioritize: hero entrance, scroll reveals on major sections, hover/transition states on cards and buttons, 1–2 decorative ambient motions. Don't animate every heading/paragraph individually — that's noise, not delight.

## Card layouts in rows

For equal-height, equal-width cards (with optional bottom-aligned CTAs), use the appropriate syntax for each surface.

### In block patterns / post content

Use `wp:columns` with `className: "equal-cards"`. Each `wp:column` gets `verticalAlignment: "stretch"` and `width: "X%"` where X = 100 / number_of_cards (2 cards = 50%, 3 = 33.33%, 4 = 25%). All cards must sum to 100%. Images: `style="height:200px;object-fit:cover;width:100%"`.

```css
.equal-cards > .wp-block-column { display: flex; flex-direction: column; flex-grow: 0; }
.equal-cards > .wp-block-column > .wp-block-group { display: flex; flex-direction: column; flex-grow: 1; }
.equal-cards .cta-bottom { margin-top: auto; justify-content: center; }
```

### In `front-page.php` / `page-{slug}.php`

Use CSS grid directly:

```html
<section class="alignfull features">
  <div class="alignwide features__grid">
    <article class="feature-card">
      <h2>Fast</h2><p>Lightning-quick load times.</p>
      <p class="cta-bottom"><a class="wp-element-button" href="/learn-more/">Learn more</a></p>
    </article>
    <!-- repeat -->
  </div>
</section>
```

```css
.features__grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--wp--preset--spacing--50, 2rem);
}
.feature-card { display: flex; flex-direction: column; }
.feature-card .cta-bottom { margin-top: auto; }
```

## Landing Page Composition

Think like a landing page designer, not a template assembler. Every section should be a visually distinct, full-width band that creates rhythm and visual impact as the user scrolls.

### Universal principles

Apply to both `front-page.php` / `page-{slug}.php` (hand-written HTML) and block patterns / post content.

- **Full-bleed wrapper, constrained content**: every major section spans the viewport edge-to-edge, with an inner container that constrains text/cards to the wide content width.
- **Visual rhythm**: alternate visual treatments between adjacent sections (background, layout pattern, image side, grid vs. single column). If two adjacent sections look the same, the page feels monotonous.
- **Section choice is contextual**: portfolios need a project gallery; SaaS needs feature grids; restaurants need imagery and menus; agencies need case studies and social proof. Pick sections that serve the site's primary goal.
- **No decorative HTML comments**: never insert labels like `<!-- Hero Section -->`.
- **No `<!-- wp:html -->` blocks** in patterns/post content — decompose into core blocks.


| Technique               | Implementation                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| Alternating backgrounds | Alternate `backgroundColor` between `background`/`surface`, or `primary`/`secondary` for bold sections. |
| Full-bleed imagery      | Cover blocks (`align: full`) with `overlayColor` from the palette.                                      |
| Edge-to-edge media-text | `wp:media-text` with `align: full` for alternating image/content sides.                                 |
| Bold CTA bands          | Full-width group with `primary`/`accent` background, centered text.                                     |
| Spacer breaks           | `wp:spacer` between sections.                                                                           |


### Syntax: PHP templates (hand-written HTML)

- Every major section: `<section class="alignfull section-name">`.
- Inner container: `<div class="alignwide">`.
- Background colors via CSS classes that reference `var(--wp--preset--color--{slug})`.
- CTAs: `<a class="wp-element-button" href="<?php echo esc_url( ... ); ?>">`.
- Animation classes apply directly on the HTML element.

### Syntax: block patterns / post content

Every major section must be `alignfull` — edge-to-edge. The wrapper fills the viewport; inner content is constrained:

```html
<!-- wp:group {"align":"full","backgroundColor":"...","layout":{"type":"constrained"}} -->
```

**Never use bare `{"layout":{"type":"constrained"}}` without `"align":"full"`** for landing-page sections — the page renders at `contentSize` (800px) and looks narrow. **Columns**: always set `"align":"wide"` on `wp:columns` unless instructed otherwise.

### Common Block Pattern Mistakes

Apply to patterns and post content (NOT to PHP templates, which are hand-written HTML).

**1. Complex layout dumped into an HTML block**

WRONG:

```html
<!-- wp:html -->
<div class="features-grid">
  <div class="feature"><h3>Fast</h3><p>Lightning-quick.</p></div>
  <div class="feature"><h3>Secure</h3><p>Enterprise-grade.</p></div>
</div>
<!-- /wp:html -->
```

RIGHT — each element a selectable, editable core block:

```html
<!-- wp:columns {"align":"wide","className":"features-grid"} -->
<div class="wp-block-columns alignwide features-grid">
  <!-- wp:column --><div class="wp-block-column">
    <!-- wp:heading {"level":3} --><h3 class="wp-block-heading">Fast</h3><!-- /wp:heading -->
    <!-- wp:paragraph --><p>Lightning-quick.</p><!-- /wp:paragraph -->
  </div><!-- /wp:column -->
  <!-- wp:column --><div class="wp-block-column">
    <!-- wp:heading {"level":3} --><h3 class="wp-block-heading">Secure</h3><!-- /wp:heading -->
    <!-- wp:paragraph --><p>Enterprise-grade.</p><!-- /wp:paragraph -->
  </div><!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

**2. Entire styled section in an HTML block**

WRONG:

```html
<!-- wp:html -->
<section class="cta-band" style="background:#1a1a2e;padding:4rem 2rem;text-align:center;">
  <h2 style="color:#fff;">Ready?</h2>
  <a href="#" class="cta-button">Sign Up</a>
</section>
<!-- /wp:html -->
```

RIGHT — use `wp:group` with `className`, style `.cta-band` in `style.css`:

```html
<!-- wp:group {"className":"cta-band","align":"full","backgroundColor":"primary","textColor":"light","layout":{"type":"constrained"}} -->
<div class="wp-block-group alignfull cta-band has-primary-background-color has-light-color has-text-color has-background">
  <!-- wp:heading {"textAlign":"center"} --><h2 class="wp-block-heading has-text-align-center">Ready?</h2><!-- /wp:heading -->
  <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
  <div class="wp-block-buttons">
    <!-- wp:button --><div class="wp-block-button"><a class="wp-block-button__link wp-element-button">Sign Up</a></div><!-- /wp:button -->
  </div>
  <!-- /wp:buttons -->
</div>
<!-- /wp:group -->
```

**3. Decorative HTML comments**

WRONG: `<!-- Hero Section -->` / `<!-- Features Section -->` between blocks. RIGHT: remove them entirely.

## Image Handling

ONLY add user-provided images / image URLs to the initial site build. Stock image URLs often fail to load in the block editor and break the design. Look at user-supplied images carefully and include them where they fit — don't force them in.

### Creating visual richness without images

When no user images are available, convey atmosphere through:

- **CSS Gradients** — linear, radial, conic for depth and color
- **Color Blocks** — bold backgrounds to create hierarchy
- **Typography as Design** — large distinctive headings, creative pairing
- **CSS Patterns** — repeating gradients (stripes, dots, grids)
- **Shadows & Depth** — box-shadow, text-shadow, drop-shadow
- **Borders & Frames** — outlines, decorative frames
- **Spacing & Layout** — generous whitespace or controlled density
- **CSS Pseudo-elements** — `::before` / `::after` for decorative elements
- **Color Overlays** — layered divs with transparency

## Design Package Schema

When building a hybrid theme from approved mockups, generate a `design-package.json` that bridges the design phase and the build phase.

```json
{
  "brief": {
    "siteName": "Name",
    "siteType": "Type",
    "primaryGoal": "Goal",
    "audience": "Audience",
    "tone": "Tone",
    "brandKeywords": "Keywords"
  },
  "tokens": "← full design-tokens.json content",
  "layout": {
    "selectedOption": "Layout name or combined description",
    "hero": { "type": "cover | split | minimal | video", "headline": "", "subheadline": "", "cta": "", "minHeight": "60vh" },
    "sections": [
      { "name": "", "type": "features-grid | testimonials | cta-banner | stats | gallery | text-media | pricing | faq | team | contact", "description": "" }
    ],
    "header": { "style": "minimal | centered | split", "sticky": true, "transparent": false },
    "footer": { "style": "simple | multi-column | minimal", "columns": 3 }
  },
  "templates": [
    { "file": "front-page.php",   "purpose": "Homepage",         "codeFirst": true,  "source": "design/approved/home.html" },
    { "file": "page-about.php",   "purpose": "About page",       "codeFirst": true,  "source": "design/approved/about.html" },
    { "file": "page-services.php","purpose": "Services page",    "codeFirst": true,  "source": "design/approved/services.html" },
    { "file": "page-contact.php", "purpose": "Contact page",     "codeFirst": true,  "source": "design/approved/contact.html" },
    { "file": "page.php",         "purpose": "Generic fallback", "codeFirst": false },
    { "file": "single.php",       "purpose": "Single post",      "codeFirst": false },
    { "file": "archive.php",      "purpose": "Archives",         "codeFirst": false },
    { "file": "search.php",       "purpose": "Search",           "codeFirst": false },
    { "file": "home.php",         "purpose": "Blog index",       "codeFirst": false },
    { "file": "404.php",          "purpose": "Not found",        "codeFirst": true  }
  ],
  "templateParts": [
    { "file": "template-parts/content/content.php",        "purpose": "Default loop entry" },
    { "file": "template-parts/content/content-page.php",   "purpose": "Static page body" },
    { "file": "template-parts/content/content-single.php", "purpose": "Single post body" },
    { "file": "template-parts/content/content-none.php",   "purpose": "Empty-state copy" }
  ],
  "patterns": [
    { "slug": "theme-slug/hero",     "title": "Hero",     "source": "design/approved/hero.html" },
    { "slug": "theme-slug/features", "title": "Features", "source": "design/approved/features.html" }
  ],
  "menus": [
    { "theme_location": "primary", "items": [ { "label": "Home", "url": "/" }, { "label": "About", "url": "/about/" }, { "label": "Contact", "url": "/contact/" } ] },
    { "theme_location": "footer",  "items": [ { "label": "Privacy", "url": "/privacy/" }, { "label": "Terms", "url": "/terms/" } ] }
  ],
  "pages": [
    {
      "title": "About",
      "slug": "about",
      "sourceFile": "<site-path>/design/approved/about.html",
      "renderedBy": "page-about.php",
      "sections": [
        { "type": "hero",          "surface": "hardcoded-html", "notes": "Page-title hero" },
        { "type": "features-grid", "surface": "hardcoded-html", "notes": "3-column feature cards" }
      ]
    }
  ],
  "motion":     { "pageLoad": "", "scrollTriggers": "", "hover": "" },
  "customCSS":  { "global": "", "sections": "", "elements": "" },
  "hybridNotes": [ "Homepage previews require the front-end; the block editor cannot render front-page.php." ]
}
```

