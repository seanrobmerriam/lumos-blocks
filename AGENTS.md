# AGENTS.md — Building Sites with Lumos

## Scope

Follow these instructions when creating or editing an Astro site built with Lumos. This is a Lumos site-authoring guide based on the official documentation, reviewed November 8, 2026. It does not define another framework or change Lumos's public API.

Before editing, read the site's package.json, existing page/layout examples, component types, and any local LUMOS.md or applicable agent instructions. Use documentation matching the installed release: the live documentation can expose different release labels on different pages. Do not upgrade dependencies or rename component APIs simply to match a newer example.

## 1. Start with the supplied framework

For a new site, the documented scaffold command is:

```sh
npm create lumos@latest my-site
```

For an existing site, work in its current structure. Use the package manager and scripts already selected by that project; do not introduce another lockfile.

Set the site's identity in src/consts.ts, including SITE_NAME, SITE_DESCRIPTION, SITE_URL, and the locale/noindex settings available in that release. Customize palette values in src/styles/base.css. Replace starter branding and assets before delivery.

Use the supplied Astro components and CSS system. Do not replace the framework with Tailwind, a separate UI kit, or a client-side component library to implement ordinary layouts. Lumos's intended workflow is readable composition with shared design decisions.

Sources: [Installation](https://lumosframework.com/docs/installation/), [Introduction](https://lumosframework.com/docs/).

## 2. Put files where they belong

| Location                 | Use                                                                                      |
| ------------------------ | ---------------------------------------------------------------------------------------- |
| src/pages/               | Routes and page composition. File paths determine URLs.                                  |
| src/layouts/             | Shared document/page shells.                                                             |
| src/components/          | Framework and custom Astro components. Preserve lowercase category directories.          |
| src/styles/base.css      | Tokens, themes, reset, and typography foundation.                                        |
| src/styles/patterns.css  | Shared multi-property patterns.                                                          |
| src/styles/utilities.css | Focused utility classes.                                                                 |
| src/styles/global.css    | Layer order and stylesheet imports.                                                      |
| src/assets/              | Imported media for Astro's asset pipeline.                                               |
| public/                  | Untouched files referenced by public URL, such as favicon, downloads, and social images. |
| src/utils/               | Existing helpers, including slot handling.                                               |

Use the configured @/ alias and exact case-sensitive paths. An image in public does not become optimized merely because an image component references it. Preserve the framework source/version metadata recorded by the scaffold.

Source: [Project structure](https://lumosframework.com/docs/project-structure/).

## 3. Build pages through composition

Use BaseLayout as the normal page shell. Inspect it before adding navigation, footer, metadata, or landmarks; do not duplicate structures it already renders.

Use Section for a full-width band. It owns its background, vertical padding, and automatic centered container. Do not add a second container inside it.

Inside Section, use ContentWrapper for content arrangement and Grid for repeated items. Use named slots, such as column2, instead of reconstructing an existing layout. Group related actions with ButtonWrapper.

Understand spacing ownership before adding margins:

- Section padding controls the band above and below content.
- The container's gap separates its children.
- Typography margins govern heading/body relationships.

Prefer documented spacing steps and layout variants. Avoid wrapper divs that exist only to repair a layout already supported by a component.

Source: [Building a layout](https://lumosframework.com/docs/concepts/layout/).

## 4. Preserve the cascade

```css
@layer base, patterns, components, utilities;
```

| Layer      | Responsibility                              |
| ---------- | ------------------------------------------- |
| base       | Tokens, themes, reset, text styles.         |
| patterns   | Shared arrangements used across components. |
| components | Each component's own styles and variants.   |
| utilities  | Instance-specific adjustments.              |

Keep this declaration before imports. For normal author rules, later layers override earlier ones regardless of selector specificity. Do not solve ordinary overrides with longer selectors or important declarations.

Place custom component styles in a global Astro style block under the components layer:

```astro
<style is:global>
  @layer components {
    .project-panel_body {
      padding: var(--space-5);
      background-color: var(--background-2);
    }
  }
</style>
```

Use component-prefixed block_element class names to prevent collisions. Do not add unlayered styles that unintentionally outrank the system. Keep specificity modest. Remember that important declarations reverse layer precedence; the normal-layer policy is not a universal rule for all CSS declarations.

Source: [The cascade](https://lumosframework.com/docs/concepts/the-cascade/).

## 5. Change tokens instead of scattering values

Use existing tokens for color, type, space, borders, radius, and motion. Change token values without renaming their public identifiers.

Edit fluid min/max inputs when adjusting a scale; preserve its existing clamp calculation. Prefer rem-based layout values so the design responds to the user's font size. Use em for font-relative detail and ch for text measure where appropriate. Avoid fixed pixel layout patches.

Use semantic tokens such as --background, --text, and --button-background in components. Raw swatches are ingredients for theme mappings, not component-level color choices.

Names beginning --_ are private implementation parameters, not stable public customization APIs. Do not couple unrelated components to them. Typography styles are sets of size, weight, line height, spacing, and margins; avoid changing one property in isolation when the intended change is a different text style.

Source: [Tokens](https://lumosframework.com/docs/concepts/tokens/).

## 6. Apply themes at boundaries

Use theme-light, theme-dark, theme-brand, and theme-invert through the documented component theme prop where one exists. On ordinary HTML, apply the class directly.

Themes establish tokens for their subtree. Components should inherit those semantic values rather than carry separate hardcoded light/dark implementations. Use contextual inversion when a surface should contrast with its surrounding theme; check the installed version's actual nested-theme behavior.

A new theme must provide the complete expected semantic token set, including control/link states. Test contrast and interactive states after palette changes. Do not recolor descendants individually to simulate a theme.

Source: [Theming](https://lumosframework.com/docs/concepts/theming/).

## 7. Choose props, utilities, and custom components deliberately

1. Use a supported prop first for variants, theme, spacing, and alignment.
2. Use an existing utility for an isolated instance adjustment.
3. When a combination becomes a repeated design, give it a reusable component or variant instead of repeating a utility stack.

For example, use Section's paddingTop prop for section rhythm; use margin-top-0 for one Heading that needs no top margin. Do not invent a variant for a single trivial exception or use utilities to bypass a supported component decision.

Before creating custom CSS, check the component reference and existing variants. Reuse established patterns when building a genuinely custom component. Keep appearance changes local to their owner rather than patching each page.

Source: [Utility or prop?](https://lumosframework.com/docs/concepts/utilities-or-props/).

## 8. Follow component contracts

- Preserve render-based conditional output where supported.
- Suppress empty content structures. For custom slotted components, use the existing slotContent helper; slot presence alone does not establish visible content.
- Use named slots according to the documented structure.
- Model variant-specific inputs with TypeScript unions so incompatible prop combinations fail checking.
- Merge incoming class utilities and forward appropriate HTML attributes to the root.
- Keep component-only inputs out of forwarded native attributes.
- Read development warnings before adding a workaround. Preserve the documented development-only warnings and default fallback behavior for invalid runtime values.
- Keep private custom properties local and component styles in their layer.

Use the actual installed names, including Nav, Footer, SkipLink, Eyebrow, ButtonWrapper, Accordion, and AccordionItem. Verify a component's current props rather than guessing from its name.

Source: [How components work](https://lumosframework.com/docs/concepts/components/).

## 9. Make grids adapt to their space

Use Grid's columns variant when a design requires explicit counts. Set only breakpoints where the count changes; unspecified larger counts inherit from the previous setting.

The documented thresholds are small at 30rem, medium at 48rem, and large at 64rem, with xsmall below 30rem. Confirm these against the installed component before introducing responsive rules.

Use autofit when remaining items should stretch across available space. Use autofill when empty tracks should remain, especially when filtering could leave a single item that should retain its size. Width settings belong to autofit/autofill; count props belong to columns.

Use the shared gutter and documented gap utilities. Verify long content and narrow containers instead of relying on one desktop viewport.

Source: [Grid](https://lumosframework.com/docs/components/grid/).

## 10. Preserve accessibility while supplying meaningful content

Keep native elements, visible focus, functional skip navigation, landmarks, and reduced-motion behavior. Do not remove these for visual convenience.

The site author must provide:

- Informative alt text for meaningful images; empty alt for decoration.
- A logical heading outline with one page h1. Use tag for semantic level and variant for appearance.
- Descriptive link labels that identify their destinations.
- Usable contrast after theme changes.

Use Button's supported button/link modes according to action versus navigation. Keep form labels and interactive keyboard behavior intact. Verify the rendered site with keyboard navigation and zoom; framework defaults do not prove that custom content and styles are accessible.

Source: [Accessibility](https://lumosframework.com/docs/concepts/accessibility/).

## 11. Verify the production output

Use the project's equivalents of these documented scripts:

```sh
npm run check
npm run build
npm run preview
```

Run the existing formatter as well. Preview the built dist output, not only the development server.

Before delivery, verify SITE_URL, page titles/descriptions, canonical/social URLs, favicon, and social image. Check internal navigation, asset requests, form destinations, keyboard interaction, and layout at 200% zoom. Resolve errors and relevant development warnings.

The documented default produces a static site in dist. Do not add a database, server adapter, or client framework unless the site's requested functionality actually requires it. Configure any deployment for the project's real output mode and host.

Source: [Deploying](https://lumosframework.com/docs/guides/deploying/).

## Completion checklist

- [ ] Used existing components and verified their installed props.
- [ ] Set site metadata and replaced starter identity/assets.
- [ ] Kept page structure and spacing under the correct component's ownership.
- [ ] Used public tokens, documented themes, and the four cascade layers.
- [ ] Avoided unnecessary wrappers, utility stacks, and specificity patches.
- [ ] Preserved empty-state behavior, native semantics, focus, and reduced motion.
- [ ] Checked responsive content and production output.
- [ ] Ran the project's build, type check, and formatter; reported any remaining limitations.
