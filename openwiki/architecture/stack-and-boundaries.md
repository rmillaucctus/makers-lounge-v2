---
type: "Reference"
title: "Architecture: stack and boundaries"
description: "Captures the intended implementation stack and the module boundaries described in the Phase 1 foundation spec."
openwiki_generated: true
---

# Architecture: stack and boundaries

## Purpose

This page captures the intended implementation stack and the module boundaries described in the Phase 1 foundation spec.

## Stack direction

The spec calls for:

- **Next.js (App Router)** on Vercel for UI, route handlers, and server actions in one repo.
- **Clerk** for authentication and open sign-up.
- **Neon Postgres** for data storage, accessed through a typed query layer.
- **Vercel Blob** for profile photos and project screenshots.
- **shadcn/ui** with **Tailwind v4** and a **tweakcn** theme for the warm, approachable visual style.
- Project-scoped **MCP servers** for shadcn, Neon, Clerk, and Stripe.

Source: `docs/superpowers/specs/2026-07-01-makerslounge-phase1-foundation-design.md` sections 4 and 9.

## Intended module boundaries

The spec defines the following conceptual boundaries:

- `lib/db` — schema, migrations, and query helpers; the only layer that talks to Neon.
- `lib/import` — CSV parsing and normalization into `EventResponse` rows plus derived `Member` drafts.
- `lib/profile` — computes a current member profile from historical responses.
- `app/(marketing)` — public landing pages.
- `app/(app)` — authenticated surfaces such as directory, profile view, edit, and showcase.
- `app/onboarding/claim` — claim flow.
- `components/ui` — shadcn primitives.
- `components/*` — composed MakersLounge components.

These boundaries matter because the core logic is designed to be unit-testable and to isolate data access from presentation.

## What future agents should watch for

When implementation begins, look for three classes of changes:

1. **Database changes** — should land in the data layer, not in route handlers or UI.
2. **Import/derivation rules** — should remain pure where possible, because the spec expects them to be testable without the app shell.
3. **Visibility rules** — UI and API code must preserve the private/public split described in the domain page.

## Related pages

- [Domain: data model and visibility rules](../domain/data-model.md)
- [Workflows: import and claim onboarding](../workflows/import-and-claim.md)
