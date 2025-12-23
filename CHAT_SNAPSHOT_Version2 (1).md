# Chat snapshot (project state) — 2025-12-23

> Purpose: This file captures the minimal, critical context needed to resume work if the Copilot chat disappears.

## Assistant operating profile (how the assistant should behave)
- Act as a **professional software engineer with extensive experience**, building **world-class** applications.
- Be **direct/critical and pragmatic** (“суров”), focused on security and correctness.
- Provide **clever, out-of-the-box ideas**, but only when they are **implementable and justified**.
- Use **proven, maintainable, production-grade techniques** (no experimental hacks).
- Work **step-by-step**:
  - Ask clarifying questions when needed
  - Wait for answers before generating lots of extra content
  - Keep changes small, testable, and secure

## Project overview
- Project type: Auction platform (P2P auctions) with:
  - Pre-bids window before live auction
  - Live bidding with anti-sniping
  - Currency: EUR
  - Payments/Stripe: **planned later** (for now build full app without Stripe)

## Tech stack
- Frontend / Fullstack: Next.js (App Router)
- Auth: Supabase Auth (Google OAuth) — **working**
- Styling: Tailwind CSS
- DB: Supabase Postgres (tables already created)


## Current Next.js dependencies (from package.json screenshot)
- `next`: `16.1.1`
- `react`: `19.2.3`
- `react-dom`: `19.2.3`
- `@supabase/supabase-js`: `^2.89.0`
- Tailwind + eslint + typescript dev deps are present


## Business rules (confirmed)
- Currency: EUR
- Pre-bids:
  - Active window: **2 days** before auction start datetime
- Live bidding:
  - Short live window (previous target ~10 minutes, may be refined in code)
- Anti-sniping:
  - Each valid bid in the last moments extends by **+30 seconds**
  - Total extension cap: **max +5 minutes** (overall)
- Stripe:
  - Not integrated yet
  - Plan: build entire app flow first, then connect payments at the end

## Security requirements / direction
- Goal: maximum security
- Writes should go through Next.js route handlers (not direct client writes)
- RLS should be enabled and enforced in Supabase
- Avoid using Supabase service role key for normal user writes (service role bypasses RLS)
- Next planned improvement:
  - Add server-side Supabase auth/session handling (cookies) via `@supabase/ssr`
  - Create `src/lib/supabase/server.ts` for server client
  - Build route handlers that:
    - require authenticated user
    - validate input (server-side)
    - perform DB writes under RLS

## Immediate next steps (planned)
1) Verify RLS is enabled for key tables:
   - `auctions`, `participants`, `bids`, plus read tables (`auction_images`, `tags`, `auction_tags`)
2) Add server-side auth tooling:
   - Install `@supabase/ssr`
   - Create `src/lib/supabase/server.ts` (createServerClient + cookie store)
3) Add first protected API endpoint to verify server-side auth works:
   - Example: `GET /api/me` returns current user/profile
4) Build UI flows (no Stripe yet):
   - Auction listing page
   - Auction details page
   - Create auction flow (seller)
   - Join auction flow (participants)
   - Place pre-bid / live bid flow
   - Realtime updates for bids
5) Finalization logic (later):
   - Cron endpoint to finalize auctions, pick winner, write audit_logs, etc.

## Open questions to resolve when resuming
- Confirm RLS policies and exact columns/constraints in the existing tables.
- Decide whether writes will:
  - use direct Supabase calls from route handlers under the user session (preferred),
  - or use RPC functions for critical transitions.
- Confirm the exact “live window duration” and how to store anti-sniping state (end_at adjustments).