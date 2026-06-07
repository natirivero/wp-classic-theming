# WordPress Classic Theming Agent

This agent builds WordPress **hybrid (classic) themes** — PHP templates as the source of truth, `theme.json` as the design system, and block patterns for editorial content.

## Where to start

### Design file present

If the project directory contains a design file (e.g. `.pen`, Figma export, HTML mockup, or `design-tokens.json`):

1. Read [wp-classic-theming.md](wp-classic-theming.md) for architecture and coding rules.
2. **Follow [theme-creation-flow.md](theme-creation-flow.md) strictly** — phased checkpoints, design extraction, and approval gates exist to translate the design faithfully before templates are built.

### No design file (starting from scratch)

If there is no design package yet:

1. Start with [site-specification.md](site-specification.md) to infer site type, audience, tone, layout, and typography from the user's description.
2. Proceed to [wp-classic-theming.md](wp-classic-theming.md) for theme generation.
3. **[theme-creation-flow.md](theme-creation-flow.md) is not required** — the phased design-review workflow assumes extractable design artifacts. Without them, work through site specification and theme build directly.

## Flow control flag

Users can override the default flow behavior with a request flag:


| Flag                        | Effect                                                                                                                                                                                 |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *(default, design present)* | Strict phased flow per [theme-creation-flow.md](theme-creation-flow.md)                                                                                                                |
| *(default, no design)*      | Skip strict flow; use site specification → theme build                                                                                                                                 |
| `--skip-strict-flow`        | Skip phased approvals and checkpoint stops even when a design file exists. Use for scratch builds, quick iterations, or when the user wants end-to-end generation without phase gates. |


When `--skip-strict-flow` is set, still read [wp-classic-theming.md](wp-classic-theming.md) in full. Use any available design file or site specification as input, but do not stop for "Approve Phase N" unless the user asks to resume strict flow.