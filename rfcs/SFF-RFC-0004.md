---
id: SFF-RFC-0004
title: Component Metadata Interchange Format
state: Pre-Draft
working_group: WG-04
working_group_slug: design-and-design-systems
authors: []
created: 2026-05-20
updated: 2026-05-22
---

# Summary

Defines a portable metadata schema for design system components — naming, props, variants, slots, accessibility intent, and intended usage — so that AI agents and design tools can reason about a design system without parsing its rendering layer.

# Motivation

> TODO: working group to draft.

Today an agent that wants to "use the Button component correctly" has to reverse-engineer the design system from React/Vue/Svelte source. A portable metadata layer makes the design system queryable: what variants exist, what props they accept, what accessibility properties they guarantee.

## Related Work

Three traditions inform component metadata, and the format must combine the conventions of all three.

**Design token specs** standardise the *atoms* below components. The [W3C Design Tokens Format Module (DTCG)](https://www.designtokens.org/tr/drafts/format/) — a W3C Community Final Specification — established the `$`-prefixed reserved-property discipline (`$value`, `$type`, `$description`, `$extensions`) and reference resolution (`{group.token}`). [Figma Variables](https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables) contributes a first-class *modes* matrix (light / dark / brand); [Style Dictionary](https://styledictionary.com/info/tokens/) (Apache-2.0) is the canonical build pipeline emitting platform-specific outputs from one DTCG source; [Adobe Spectrum](https://github.com/adobe/spectrum-design-data) (Apache-2.0) layers per-token-type JSON Schema validation; [Tokens Studio](https://docs.tokens.studio/tokens/json-schema) introduces orthogonal *theme groups* (color mode × brand × density).

**Component metadata formats** describe individual components. [Storybook CSF](https://storybook.js.org/docs/api/csf) (MIT) makes metadata executable — `args` / `argTypes` drive control widgets — but ES-module-shaped, not JSON. [react-docgen / vue-docgen](https://react-docgen.dev/) (MIT) derive metadata from source AST + PropTypes / TS / JSDoc; no hand-authored manifest. [web-types.json](https://github.com/JetBrains/web-types) (JetBrains, Apache-2.0) introduces three namespaces (`html`, `css`, `js`) so non-element symbols (Vue directives, mixins) coexist with components. The [Custom Elements Manifest](https://github.com/webcomponents/custom-elements-manifest) (BSD-3-Clause) is the most complete structural model — modules → declarations → `kind` (class / variable / function), with CustomElement adding `tagName`, `attributes`, `members`, `events`, `slots`, `cssParts`, `cssProperties`, `cssStates`.

**UI library registries and primitives metadata** describe whole component *collections*. [Radix UI](https://www.radix-ui.com/primitives) (MIT) exposes component state as DOM `data-*` attributes (`data-state`, `data-orientation`) — state is observable without instrumentation, but there is no separate manifest. [shadcn's registry.json / registry-item.json](https://ui.shadcn.com/docs/registry/registry-item-json) (MIT) is *distribution-oriented* (copy-paste delivery + dependency graph) with an explicit `meta` extension point for LLMs. [MUI](https://mui.com/material-ui/all-components/) (MIT) is TypeScript-first; consumer-facing JSON is a doc-build byproduct. [Headless UI](https://headlessui.com) and [Ariakit](https://ariakit.org) (MIT) wire WAI-ARIA correctness in code rather than in metadata — *behavioural* metadata lives in the implementation itself.

The gap SFF-RFC-0004 fills sits between the three traditions. **No surveyed format has structured a11y declarations**; every one treats accessibility either as free-text description (CEM, web-types) or implicit-in-code (Radix, Ariakit). **Variants are weakly modeled everywhere** — Storybook encodes them as argTypes (control widgets, not semantic variants), shadcn buckets at file-type granularity, CEM not at all. This is the clearest opportunity for SFF-RFC-0004 to differentiate.

# Detailed Design

> TODO: working group to draft.

## Proposed shape (informed by prior art)

The schema SHOULD adopt the DTCG `$`-prefix discipline (any non-`$` key is a nested group / component), the CEM structural model (modules → declarations → fields), and the web-types three-namespace pattern for extensibility.

**Identity and structural envelope (CEM-shaped):**

- `$schema` — stable URL of the SFF component-metadata JSON Schema.
- `name`, `version`, `description` — package-level (mirrors `web-types.json`).
- `modules[]` → `declarations[]` → `kind: component | primitive | layout | utility`.
- Per component: `tagName?`, `displayName`, `description`.

**Props, slots, events, variants:**

- `props[]` — each is `{ name, type, default?, required, description, deprecated? }` (CEM + react-docgen pattern).
- `slots[]` — named insertion points (CEM + Vue / web-component standard).
- `events[]` — `{ name, type, description }`.
- `variants[]` — **first-class semantic variants** (novel). Each variant is `{ axis, value, prop_bindings, preview? }`. Axes form a matrix (Tokens Studio theme-group pattern lifted to components).

**Accessibility (novel — clearest differentiator):**

- `a11y` — structured: `{ role, aria_attributes[], keyboard_interactions[], focus_management, screen_reader_announcement? }`. No surveyed format has this; CEM and web-types use free-text only.

**Composition and polymorphism:**

- `composes[]` — `{ component_ref, slot? }` — explicit composition graph.
- `polymorphic` — declares `as` / `asChild` patterns portably across React, Vue, web components.

**Distribution-oriented metadata (shadcn-shaped, optional):**

- `files[]`, `dependencies[]`, `registry_dependencies[]`, `css_vars`, `meta` (for LLM hints).

## Source and authoring

The format MUST be expressible as a sidecar JSON file (`<component>.component.json`) alongside source — derivable by static-analysis tools (react-docgen-style) or hand-authored (CEM-style). A single component MAY ship both paths.

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- **[Storybook CSF](https://storybook.js.org/docs/api/csf)** (MIT). Executable metadata, ES module — strong for tooling but not a portable JSON interchange.
- **[W3C Design Tokens Format Module](https://www.designtokens.org/tr/drafts/format/)**. Token-level only — orthogonal to components, but the `$`-prefix discipline is adopted here.
- **[Custom Elements Manifest](https://github.com/webcomponents/custom-elements-manifest)** (BSD-3-Clause). Most complete structural model; the v0 envelope adopts its modules → declarations → fields shape.
- **[web-types.json](https://github.com/JetBrains/web-types)** (Apache-2.0). Three-namespace pattern (HTML / CSS / JS) for extensibility.
- **[shadcn registry](https://ui.shadcn.com/docs/registry/registry-item-json)** (MIT). Distribution-oriented; the `meta` extension point pattern is adopted as an optional layer.
- **[react-docgen](https://react-docgen.dev/) / [vue-docgen](https://vue-styleguidist.github.io/docs/Docgen.html)** (MIT). Source-derived JSON; useful as a generator into this format.
- **[Adobe Spectrum tokens](https://github.com/adobe/spectrum-design-data)** (Apache-2.0). Per-token-type JSON Schema validation pattern.

# Prior Art

> TODO: working group to draft.

| Source | Stream | Has machine schema? | Variants / slots? | A11y declarations? | Portable across libs? | Contribution to v0 |
|---|---|---|---|---|---|---|
| [DTCG Format](https://www.designtokens.org/tr/drafts/format/) | Tokens | Yes (JSON Schema) | N/A | No | Yes — cross-tool | `$`-prefix discipline; composite types |
| [Figma Variables](https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables) | Tokens | Partial (export) | Modes only | No | With export | Modes matrix |
| [Style Dictionary](https://styledictionary.com/info/tokens/) | Tokens | Yes (DTCG-compatible) | N/A | No | Yes | Build pipeline |
| [Adobe Spectrum](https://github.com/adobe/spectrum-design-data) | Tokens | Yes (per-type JSON Schema) | Component schemas too | No | Yes | Strong typing |
| [Tokens Studio](https://docs.tokens.studio/tokens/json-schema) | Tokens | Yes (DTCG-opt-in) | Theme groups (orthogonal) | No | Yes | Multi-axis themes |
| [Storybook CSF](https://storybook.js.org/docs/api/csf) | Component | Partial (ES module, not JSON) | args / argTypes for variants | No | Cross-framework | Executable metadata |
| [react- / vue-docgen](https://react-docgen.dev/) | Component | Yes (JSON, convertible to JSON Schema) | Slots (Vue), events (Vue) | No (only JSDoc text) | Per-framework | Derived from source |
| [web-types.json](https://github.com/JetBrains/web-types) | Component | Yes (JSON Schema) | Slots, events, attrs | Indirect (descriptions) | Framework-agnostic | Three-namespace pattern |
| [Custom Elements Manifest](https://github.com/webcomponents/custom-elements-manifest) | Component | Yes (JSON Schema v2.1.0) | Slots, events, CSS parts / states | Indirect (description) | Standards-aligned | Structural model |
| [Radix Primitives](https://www.radix-ui.com/primitives) | Library | No manifest | DOM `data-state` / `data-*` | Yes (WAI-ARIA wired) | No (code-only) | State on DOM |
| [shadcn registry](https://ui.shadcn.com/docs/registry/registry-item-json) | Library | Yes (registry-item.json schema) | type + files, deps graph | No | Yes (registry protocol) | `meta` for LLMs |
| [MUI API](https://mui.com/material-ui/all-components/) | Library | Partial (internal JSON from TS) | Slots prop pattern | No formal | No portable artefact | TS-first |
| [Headless UI](https://headlessui.com) / [Ariakit](https://ariakit.org) | Library | No manifest | Render-prop slots / store | Yes (WAI-ARIA implicit) | No | Behaviour-as-code |

# Open Questions

- Should this live in the same file as the component source, in a sidecar, or in a registry? *Recommendation from synthesis: sidecar JSON, with optional registry aggregation.*
- How is composition (one component embeds another) represented? *Recommendation: explicit `composes[]` array with `{ component_ref, slot? }`.*
- Source-derived (react-docgen) vs hand-authored (CEM / web-types) vs both?
- Library-distribution metadata (shadcn `meta`, registry deps) vs introspection metadata (CEM) — one envelope or two?
- How to express polymorphic `asChild` / `as` patterns portably across React, Vue, web-component runtimes?
- Should `variants[]` be normative (every component MUST declare its variant matrix) or descriptive?

# References

- W3C Design Tokens Community Group. *Design Tokens Format Module (Final).* https://www.designtokens.org/tr/drafts/format/
- Figma. *Modes for variables.* https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables
- Amazon. *Style Dictionary.* https://styledictionary.com/
- Adobe. *Spectrum design data.* https://github.com/adobe/spectrum-design-data
- Tokens Studio. *JSON Schema.* https://docs.tokens.studio/tokens/json-schema
- Storybook. *Component Story Format (CSF).* https://storybook.js.org/docs/api/csf
- react-docgen. https://react-docgen.dev/
- vue-docgen-api. https://vue-styleguidist.github.io/docs/Docgen.html
- JetBrains. *web-types.* https://github.com/JetBrains/web-types
- Web Components Community Group. *Custom Elements Manifest.* https://github.com/webcomponents/custom-elements-manifest
- Radix UI Primitives. https://www.radix-ui.com/primitives
- shadcn. *registry-item.json docs.* https://ui.shadcn.com/docs/registry/registry-item-json
- MUI. *Material UI components.* https://mui.com/material-ui/all-components/
- Headless UI. https://headlessui.com
- Ariakit. https://ariakit.org
