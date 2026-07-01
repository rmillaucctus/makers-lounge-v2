# MakersLounge — Phase 1: Foundation (Design Spec)

**Date:** 2026-07-01
**Status:** Draft for review
**Phase:** 1 of 4 (Foundation). Later phases: 2 Smart Matching · 3 Membership/Stripe gating · 4 AI concierge + operator agents.

## 1. Goal

Stand up the MakersLounge app and **activate the 1,200-person Luma dataset**: import it, let existing members **claim** their pre-filled profiles, and give the community a warm, browsable **directory** and **project showcase**. This is the foundation the smart-matching wedge (Phase 2) sits on.

Success = a real Toronto builder can sign in, find their info already waiting, claim/edit it, browse other claimed members, and post a project — using Berto's actual data.

## 2. Non-goals (explicitly later)

- **Smart matching / "people you should meet"** → Phase 2.
- **Stripe checkout, the paid gate, and premium-feature gating** → Phase 3. (Phase 1 *includes the schema fields* `plan`/`stripeCustomerId`/`subscriptionStatus` so Phase 3 is additive, but no billing UI and nothing is gated yet.)
- **AI concierge (assistant-ui)** and **operator agents (Vercel agents)** → Phase 4.
- **Two-sided / recruiter-VC-sponsor access** → later. Phase 1 only keeps data *structured* enough (clean tags) to not foreclose it.

## 3. Users & the consent constraint (hard requirement)

The Luma data came from **event registration forms, not app sign-ups.** Therefore:

- Imported profiles are **`unclaimed`** and are **NOT publicly visible** anywhere in the app until the person signs in and claims them.
- The directory/showcase show **only claimed (opted-in) members**.
- Claiming = explicit opt-in; the claim screen shows what data we have and lets them edit/remove before going live.

This protects trust, which is part of the moat. Non-negotiable for Phase 1.

## 4. Architecture & stack

- **Next.js (App Router)** on Vercel — UI + route handlers/server actions in one repo.
- **Clerk** — auth + open sign-up. Verified email is the identity key for claiming.
- **Neon Postgres** — data. Access via a typed query layer (Drizzle ORM recommended for typed schema + migrations; final ORM choice confirmed in the plan).
- **Vercel Blob** — profile photos + project screenshots (server-side uploads).
- **shadcn/ui** + **Tailwind v4**, themed light-first / simple / rounded / warm via **tweakcn**.
- **Live MCP servers** (shadcn, Neon, Clerk, Stripe) added at **project scope** (committed `.mcp.json`) right after scaffold.

### Module boundaries (each independently understandable/testable)

- `lib/db` — schema + migrations + query helpers (the only place that talks to Neon).
- `lib/import` — CSV parsing + normalization → `EventResponse` rows → derived `Member` drafts. Pure, unit-testable, no UI.
- `lib/profile` — derive a current Member profile from that member's EventResponse history (the "latest wins + merge tags" logic). Pure, unit-testable.
- `app/(marketing)` — public landing.
- `app/(app)` — authed surfaces: directory, profile view, my-profile edit, showcase.
- `app/onboarding/claim` — claim flow.
- `components/ui` — shadcn primitives; `components/*` — composed MakersLounge components.

## 5. Data model (Phase 1)

Emails are the cross-event identity key. Store all timestamps UTC.

**`event_response`** — the longitudinal moat. One row per Luma submission.
- `id`, `email` (lowercased, indexed), `full_name`, `event_name`, `event_date`,
- `neighbourhood`/location, `interests` (text[]), `building` (text), `needs_help_with` (text), `can_help_with` (text), `links` (jsonb),
- `raw` (jsonb — the original CSV row, so we never lose data), `submitted_at`, `imported_at`.

**`member`** — current profile (one per unique email).
- `id`, `email` (unique), `clerk_user_id` (nullable until claimed), `status` (`unclaimed` | `claimed`),
- `display_name`, `photo_url`, `neighbourhood`, `skills` (text[]), `interests` (text[]), `building`, `needs_help_with`, `can_help_with`, `links` (jsonb), `bio`,
- **billing (schema only, unused in P1):** `plan` (default `free`), `stripe_customer_id` (nullable), `subscription_status` (nullable),
- `created_at`, `updated_at`, `claimed_at` (nullable).

**`project`** — showcase.
- `id`, `member_id` (fk), `title`, `description`, `screenshot_urls` (text[]), `links` (jsonb), `tags` (text[]), `status` (`building` | `launched` | `seeking_collaborators`), `created_at`, `updated_at`.

*(`match` table is Phase 2, intentionally omitted here.)*

## 6. CSV import pipeline (`lib/import`)

Luma exports one CSV per event; the same person appears across many. Flow:

1. **Parse** each CSV → rows. Map Luma columns → `EventResponse` fields; keep the whole original row in `raw`.
2. **Normalize** — lowercase email, split interests/skills into clean tags, trim.
3. **Insert** `event_response` rows (idempotent: dedupe on `email + event_name + submitted_at`).
4. **Derive `member` drafts** — group responses by email; build the current profile via `lib/profile` (most-recent answer wins for single fields like `building`/`needs`; union tags for `interests`/`skills`). Upsert `member` with `status = unclaimed`.

Run as a **script / admin route** (not user-facing), re-runnable safely. Column mapping is config-driven since Luma question wording varies per event — we'll confirm the real columns from a sample CSV during implementation.

## 7. Claim-your-profile onboarding

1. Person signs in via Clerk (open sign-up).
2. On first sign-in, match Clerk's **verified email** to an `unclaimed` member.
   - **Match found** → claim screen: "We found your profile from MakersLounge events." Show pre-filled data, let them edit/remove fields, add photo. Confirm → set `clerk_user_id`, `status = claimed`, `claimed_at`. Now visible in the directory.
   - **No match** → fresh profile creation with the same form (empty). Creates a new claimed member.
3. Post-claim → land on their profile / the directory.

## 8. Screens (Phase 1)

1. **Landing** (public) — what MakersLounge is, sign-in CTA. Warm/welcoming tone.
2. **Claim / onboarding** — the flow above.
3. **Directory** (authed) — grid of **claimed** member cards; filter/search by skill, interest, neighbourhood, and "needs help / can help". Postgres filtering (no external search engine in P1).
4. **Member profile** — full profile + their projects.
5. **My profile (edit)** — edit own profile; manage own projects (create/edit/delete, upload screenshots to Blob).
6. **Showcase** — grid of all projects (from claimed members), filter by tag/status, click through to member.

## 9. Styling

Install a **tweakcn** theme tuned to: light-first, generous rounding, warm friendly accent, airy spacing, approachable sans type. Emotional target (from real member words): *Welcoming, Supportive, Talented, Interesting, Ideas, Collaborative.* Install via `npx shadcn@latest add https://tweakcn.com/r/themes/<id>.json` (exact id chosen from the tweakcn gallery during implementation) or hand-tuned OKLCH tokens in `globals.css`.

## 10. Error handling

- **Import**: per-row try/catch; bad rows logged with the offending `raw` payload, import continues; summary report of inserted/skipped/failed.
- **Claim email mismatch**: if a signed-in user expects a match but none exists (e.g., used a different email), offer "start fresh" plus a note to sign in with their event email.
- **Blob uploads**: size/type validation, friendly error, no partial member state.
- **DB**: all writes in the query layer with typed errors; user-facing surfaces show graceful fallbacks.

## 11. Testing

- **Unit** (the pure modules carry the risk): `lib/import` (CSV → normalized EventResponse, dedupe, bad-row handling) and `lib/profile` (history → current profile merge rules). Test with fixture CSVs mirroring real Luma exports.
- **Integration**: claim flow (email match → claimed, no-match → fresh), directory only shows claimed, project CRUD.
- **Manual**: run the real CSV import against a Neon dev branch, spot-check derived profiles.

## 12. Open decisions (resolve during planning)

- Final ORM (Drizzle vs Prisma) — leaning **Drizzle** (typed, light, great with Neon).
- Exact Luma CSV columns → confirm from a sample export before finalizing the mapping.
- Whether the landing/directory is fully private (login required to see anything) or the landing is public with directory behind auth — **assumed: public landing, authed directory.**
- Photo source on claim: allow pulling an avatar, or upload-only — **assumed: upload-only in P1.**

## 13. Out of scope for Phase 1

Smart matching, Stripe checkout/gating, AI concierge, operator agents, messaging/DMs, events/RSVP, notifications, mobile app.
