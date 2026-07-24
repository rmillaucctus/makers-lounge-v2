---
type: "Reference"
title: "Workflows: import and claim onboarding"
description: "Outlines the CSV import pipeline and claim onboarding workflow for the MakersLounge project."
openwiki_generated: true
---

# Workflows: import and claim onboarding

## CSV import pipeline

The spec describes a rerunnable admin-side import flow for Luma exports.

### Intended steps

1. Parse each CSV into rows and map Luma columns into `EventResponse` fields.
2. Normalize values such as email casing and tag lists.
3. Insert `event_response` rows idempotently, deduping on `email + event_name + submitted_at`.
4. Group rows by email and derive the current `member` draft using the most recent response for single-value fields and merged tags for multi-value fields.
5. Upsert `member` with `status = unclaimed`.

### Why this workflow matters

The app depends on historical event data being preserved, not collapsed away. That is why the import pipeline retains the raw CSV row and treats derived member state as a projection over history.

### Implementation notes from the spec

- The import path should be script/admin-route based, not user-facing.
- The mapping is expected to be config-driven because Luma question wording varies across events.
- The import should continue after bad rows and report inserted/skipped/failed counts.

## Claim onboarding

### Intended claim flow

1. A person signs in through Clerk.
2. The app uses the verified email to look for a matching `unclaimed` member.
3. If found, the user sees a pre-filled claim screen and can edit or remove fields before opting in.
4. Confirming the claim sets `clerk_user_id`, flips `status` to `claimed`, and records `claimed_at`.
5. If no match exists, the user creates a fresh profile.

### Why this workflow matters

The claim flow is how the product turns event signups into actual members while preserving consent. The user must explicitly opt in before the profile becomes visible to others.

## User-facing surfaces tied to these workflows

The Phase 1 spec names these surfaces:

- public landing
- claim / onboarding
- authenticated directory
- member profile
- my profile edit
- project showcase

## Future-agent guidance

When working on these flows, verify both of the following:

- The import still honors idempotency and history retention.
- The claim flow still respects the visibility boundary between unclaimed and claimed profiles.

## Related pages

- [Domain: data model and visibility rules](../domain/data-model.md)
- [Operations: testing, errors, and open decisions](../operations/testing-and-risks.md)
