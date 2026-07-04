# Operations: testing, errors, and open decisions

## Testing strategy in the spec

The spec separates testing by risk:

- **Unit tests** for the pure logic in `lib/import` and `lib/profile`.
- **Integration tests** for the claim flow, directory visibility, and project CRUD.
- **Manual validation** by running the real CSV import against a Neon dev branch and spot-checking derived profiles.

## Error handling expectations

The spec calls out several operational behaviors:

- Import should catch bad rows individually, log the offending raw payload, continue processing, and return a summary.
- Claim email mismatches should offer a clear fallback to start fresh and advise signing in with the event email.
- Blob uploads should validate size and type and avoid partial member state.
- DB writes should be centralized in the query layer and produce graceful user-facing fallbacks.

## Open decisions captured in the spec

The spec still leaves a few implementation choices open:

- Final ORM choice, with Drizzle favored over Prisma.
- Exact Luma CSV column mapping, pending a sample export.
- Public vs private landing/directory split, with the current assumption being public landing and authenticated directory.
- Photo handling during claim, with upload-only currently assumed.

## Important non-goals for Phase 1

The spec explicitly excludes:

- smart matching
- Stripe checkout or gating
- AI concierge
- operator agents
- messaging / DMs
- events / RSVP
- notifications
- mobile app

These omissions are important because they keep the initial scope focused on the data foundation and claim flow.

## What future agents should verify first

Before changing behavior in this repo, check whether a decision from the spec has since been implemented or revised. The most likely change points are:

- import normalization rules
- profile derivation rules
- visibility rules for claimed vs unclaimed members
- schema additions for billing fields

## Related pages

- [Architecture: stack and boundaries](../architecture/stack-and-boundaries.md)
- [Domain: data model and visibility rules](../domain/data-model.md)
