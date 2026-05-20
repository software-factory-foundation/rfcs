---
id: SFF-RFC-0004
title: Component Metadata Interchange Format
state: Pre-Draft
working_group: WG-04
working_group_slug: design-and-design-systems
authors: []
created: 2026-05-20
updated: 2026-05-20
---

# Summary

Defines a portable metadata schema for design system components — naming, props, variants, slots, accessibility intent, and intended usage — so that AI agents and design tools can reason about a design system without parsing its rendering layer.

# Motivation

> TODO: working group to draft.

Today an agent that wants to "use the Button component correctly" has to reverse-engineer the design system from React/Vue/Svelte source. A portable metadata layer makes the design system queryable: what variants exist, what props they accept, what accessibility properties they guarantee.

# Detailed Design

> TODO: working group to draft.

## Proposed shape

- `component.id`, `component.name`, `component.kind` (atom, molecule, layout, …)
- `props[]` with types, defaults, and required flags
- `variants[]` (e.g. size, intent)
- `slots[]` — named insertion points
- `a11y` — accessibility commitments and ARIA roles
- `usage_intent` — short human-readable description of when to reach for this component

# Drawbacks

> TODO: working group to draft.

# Alternatives Considered

- Storybook's CSF (Component Story Format) metadata.
- W3C Design Tokens Format.

# Prior Art

> TODO: working group to draft.

# Open Questions

- Should this live in the same file as the component source, in a sidecar, or in a registry?
- How is composition (one component embeds another) represented?
