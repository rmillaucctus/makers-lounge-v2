# Domain: data model and visibility rules

## Core business idea

MakersLounge is trying to turn a set of event-registration responses into a living member directory and project showcase.

The key domain concept is that the historical Luma data is a **longitudinal moat**: every event submission is retained, and the current profile is derived from that history.

## Main records

### `event_response`

One row per Luma submission. The spec describes this as the canonical historical record with fields such as:

- `email` as the cross-event identity key
- `full_name`
- `event_name` and `event_date`
- location / neighbourhood
- `interests`
- `building`
- `needs_help_with`
- `can_help_with`
- `linkedin_url`
- `links`
- `raw` original CSV payload
- `submitted_at` and `imported_at`

This table is intentionally richer than the public profile because it preserves the original event context.

### `member`

One row per unique email, representing the current profile.

The spec includes:

- identity fields: `email`, `clerk_user_id`
- claim state: `status` (`unclaimed` or `claimed`), `claimed_at`
- public/profile fields: `display_name`, `photo_url`, `linkedin_url`, `links`
- community fields: `neighbourhood`, `skills`, `interests`, `building`, `needs_help_with`, `can_help_with`, `bio`
- future billing fields kept in schema only for Phase 3: `plan`, `stripe_customer_id`, `subscription_status`

### `project`

A showcase object attached to a member.

The spec describes:

- `member_id`
- `title`
- `description`
- `screenshot_urls`
- `links`
- `tags`
- `status` (`building`, `launched`, `seeking_collaborators`)

## Visibility rules

The most important product rule is:

> **Private signal, public surface**

That means:

- Private event answers can help with matching and profile preparation.
- Private answers must not be surfaced for unclaimed people.
- Public display for unclaimed people is limited to a minimal card with display name and public LinkedIn URL.
- Full directory and showcase browsing are for claimed members only.
- Claiming is explicit opt-in.

This rule was strengthened in the latest commit history and should be treated as a hard product constraint.

## Identity and lifecycle

- Email is the identity key across events.
- Imported member records start as `unclaimed`.
- Claiming happens via verified email from Clerk.
- Once claimed, the member becomes visible in the directory and can manage their profile and projects.

## Why this domain exists

This design allows the app to preserve trust with people whose data originally came from event registration, while still letting the product derive useful structure from that history.

## Related pages

- [Workflows: import and claim onboarding](../workflows/import-and-claim.md)
- [Operations: testing, errors, and open decisions](../operations/testing-and-risks.md)
