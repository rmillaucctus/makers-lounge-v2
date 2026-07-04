# OpenWiki Quickstart

## What this repository is

MakersLounge v2 is currently centered on a Phase 1 foundation spec for a community directory product. The spec describes a Next.js + Clerk + Neon + Vercel Blob app that imports a historical Luma dataset, lets people claim pre-filled profiles, and publishes a warm directory plus project showcase for claimed members.

The most important product rule is the **private signal, public surface** principle: the repository is designed to use private event-registration answers for matching and profile preparation, while only public information such as a display name and LinkedIn URL may be shown for unclaimed people.

Primary source of truth:
- `docs/superpowers/specs/2026-07-01-makerslounge-phase1-foundation-design.md`

## Start here

1. Read the domain model and privacy rules in [domain/data-model.md](domain/data-model.md).
2. Read the implementation shape in [architecture/stack-and-boundaries.md](architecture/stack-and-boundaries.md).
3. Read the import and onboarding flows in [workflows/import-and-claim.md](workflows/import-and-claim.md).
4. Read the testing and risk notes in [operations/testing-and-risks.md](operations/testing-and-risks.md).

## Repository map

- [Architecture: stack and boundaries](architecture/stack-and-boundaries.md)
- [Domain: data model and visibility rules](domain/data-model.md)
- [Workflows: import and claim onboarding](workflows/import-and-claim.md)
- [Operations: testing, errors, and open decisions](operations/testing-and-risks.md)

## Current codebase shape

At the time of this documentation run, the repository appears to be documentation-led. The only substantive project file discovered was the Phase 1 design spec under `docs/superpowers/specs/`; no application source directories or package manifests were present in the top-level inventory.

That means the OpenWiki here is a synthesis layer over the current design intent, not a map of implemented code yet. Future code work should be checked against the spec and updated here as the repo grows.

## Useful implementation facts

- Stack direction: Next.js App Router on Vercel, Clerk auth, Neon Postgres, Vercel Blob, shadcn/ui, Tailwind v4, tweakcn styling.
- Persistence is centered on three conceptual records: `event_response`, `member`, and `project`.
- Import is meant to be idempotent and rerunnable.
- Claiming is email-based via Clerk verified email.
- Phase 1 intentionally excludes smart matching, Stripe gating, AI concierge, operator agents, messaging, notifications, and mobile.

## Git evidence worth knowing

Recent history shows the spec evolved to make LinkedIn a first-class public field and to formalize the privacy boundary between private event answers and public profile surface. That update is the clearest product constraint in the repository today.

Relevant commit:
- `47bc0ec` — "Refine spec: LinkedIn field + 'private signal, public surface' principle"

## Where to go next

If you are changing the product shape, update the spec in `docs/superpowers/specs/` and then reflect the decision here.

If you are adding implementation code later, create additional OpenWiki pages that cover the actual source tree, scripts, schema, and tests.
