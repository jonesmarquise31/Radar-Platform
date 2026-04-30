# V41 — The Radar Decode

**Shipped:** April 2026
**Surface area:** New standalone product on a separate domain (`radar-decode.netlify.app`), independent from `workforceradar.com`
**Stack:** Single-file static landing, two Netlify Functions, additive Supabase table, Stripe webhook → Telegram alerts

## The decision

Two pressures landed in the same week and they pointed at the same answer.

The first was a beta tester's critique of the main platform, which named three real problems: a 5-minute diagnostic was generating archetype assignments from a single sentence of buyer input (overreach); the platform branded itself as surveillance/intelligence but never parsed real artifacts (LinkedIn profiles, resumes, work history); and there was no proof beat between diagnosis and payment, which collapsed the buyer's journey from curiosity straight to the highest-friction commitment.

The second was a distribution surge on LinkedIn — profile views and engagement up multiple-X over a four-week window, opening a 30-90 day period of elevated conversion. Fixing the main platform's three issues was a multi-week refactor. Capturing the distribution window required something live in 24 hours.

The decision: ship a new offer that bypasses all three critiques as a separate product, in parallel with the main platform fix. A standalone tier that costs less, demands less, and proves the methodology before asking for the bigger commitment.

## The product shape

**The Radar Decode** — $47 one-time. The buyer sends a LinkedIn URL and a resume. Within 48 hours they receive a one-page PDF written by hand, naming exactly how recruiters and hiring systems are currently parsing their profile, the specific signals causing them to be filtered or undervalued, and three repositioning moves anchored to their actual artifacts.

This shape addresses each of the beta critiques directly. No archetype is generated from one sentence — there is no archetype, just a personalized read. The artifacts being parsed are the buyer's real LinkedIn and resume, not a 23-question quiz proxy. And the Decode itself is the proof beat — the buyer holds a written deliverable in their hand before any upsell to the larger system is offered.

It also creates a clean tier structure for the line:

```
$47    Radar Decode        Entry, fast, artifact-anchored read
$97    Radar RDS           Full diagnostic with archetype classification
$1,497 Full System         Engagement: 3C Framework + AMR + delivery
```

## The architecture

The product is deployed as a separate site, not a subdomain of `workforceradar.com`. DNS configuration would have consumed hours and could have missed the launch window. A Netlify-assigned subdomain went live within 30 minutes of the first commit. Custom DNS can be added later without changing any application code.

The two Netlify Functions are isolated from the main platform's function set. The Supabase work is purely additive: a new table (`decode_submissions`), a new private storage bucket (`decode-resumes`), no modification to any existing tables, RLS policies, or auth configuration. The main platform's flows are untouched and the rollback for the entire Decode tier is "delete the Netlify site" — nothing is entangled.

```
Buyer journey:

  Landing page  →  Stripe Checkout  →  /thanks intake form  →  Decode delivered (manual)
       ↓                  ↓                     ↓                       ↓
  CTA click       payment_status:    multipart upload to       PDF email from
  → Stripe URL    "paid" event       submit-decode function    operator inbox
                       ↓                     ↓
                  stripe-webhook       verifies session,
                  → Telegram alert     writes row + file
```

### The two functions

**`submit-decode.js`** — receives the post-payment intake form. It verifies the Stripe session is real, paid, and matches the configured price ($4,700 in cents) before it accepts any data. If the session check fails or the price doesn't match, the submission is rejected before a row is written or a file is uploaded. The buyer's resume is uploaded to the private storage bucket under a path namespaced by their session ID; the row in `decode_submissions` references the storage path. Two layers of idempotency — a pre-write lookup on `stripe_session_id` and a unique-constraint catch (Postgres error code `23505`) at insert time — collapse double-submits, retries, and network blips down to one canonical row.

**`stripe-webhook.js`** — receives `checkout.session.completed` events directly from Stripe with signed payloads. It verifies the signature using the webhook signing secret, then formats a one-line buy alert and posts to a Telegram bot. The function returns 200 to Stripe whether or not the Telegram POST succeeded — Telegram failures should not cause Stripe to retry the entire event in a loop.

### The notification design

The webhook fires the moment Stripe confirms payment, before the buyer has completed the intake form. This was a deliberate choice: the operator gets real-time signal on every paid session, including buyers who pay and then bail on the intake form (the orphan-rate population from V40). A second notification could be added on form completion if dual-signal becomes useful, but starting with one alert at the buy moment kept the surface area small.

## The phased build

The Decode shipped in five phases, each gated on explicit verification before the next began:

1. **Schema.** Created `decode_submissions` table with CHECK constraint on a delivery status enum, two indexes for the queries the operator will actually run, and RLS enabled with zero policies (deny everything except the service role). Created the private `decode-resumes` storage bucket, also deny-all-except-service-role.
2. **Function.** Wrote `submit-decode.js` with multipart parsing, server-side Stripe verification, validation order (mime → empty → size), filename sanitization, idempotency, and a tightly-controlled error vocabulary that never leaks internal state.
3. **Thank-you page.** Static HTML with vanilla JS, mobile-first. Reads `session_id` from the URL query string; if absent, hides the form and shows a contact email instead. The submit handler distinguishes JSON server errors, non-JSON server errors (502s, function timeouts, HTML error pages), and network failures — each gets a different message so the buyer can route the problem.
4. **Deploy + env.** Set four environment variables on the production Netlify site (Supabase URL + service role key, Stripe secret key, expected price in cents). Caught a real bug here: secrets set with the `--secret` flag did not default to the `functions` scope, so the function could not see them at runtime. Re-set with explicit scope `builds functions runtime`. The smoke test (POST with empty body returning 400, not 500) confirmed the env vars resolved correctly before any real traffic landed.
5. **Buy alerts.** Created the Stripe webhook endpoint via the Stripe API, captured the signing secret, set it as another scoped Netlify secret, wrote and deployed `stripe-webhook.js`, sent a sample-format message via the Telegram bot to confirm the pipeline before any real buyer hit it.

## Trade-offs

### Live mode from day one

The launch went straight to live Stripe mode, not test mode. The reasoning: clients were already being DM'd the link as the platform was being built. Test-mode purchases would have produced test events that wouldn't validate against the live webhook's signing secret, and a test-then-cutover would have introduced its own risk. The risk of a real-mode early purchase was bounded by the operator's ability to refund. The ability to manage refunds is a built-in Stripe feature; the ability to test in production was made non-trivial by the timing.

### Smoke test as substitute for end-to-end test

A dummy purchase + form submission + storage check was scoped and then deferred. The reasoning: each layer was verified independently — function smoke test confirms env vars resolve, Stripe webhook confirms signature verification logic, Telegram bot confirms the notification pipeline, the schema and storage bucket confirm the write surface — and the Telegram alert provides same-second feedback on real production traffic. If a real buyer's flow breaks, the operator knows before the buyer can email about it.

This is not a substitute for end-to-end testing in general. It is a substitute when the launch window is shorter than the test cycle, monitoring is good enough to convert real users into the test, and the worst-case failure (a manual cleanup for the first one or two buyers) is acceptable.

### The error vocabulary

The submit function returns specific 400 messages for client mistakes ("Resume must be a PDF, DOC, or DOCX") and a single generic 500 message ("Server is not configured for submissions yet") for any server-side problem. The 500 message intentionally never reveals which env var is missing or what the underlying error is. The thank-you page distinguishes three error categories: server returned JSON with an `error` field (show that), server returned non-JSON or 5xx (show "Server error (status code)" with the operator's email as escape hatch), or fetch itself rejected (show "Network error"). This means the buyer gets actionable feedback without internal state leaking, and silent failures can't happen because every code path produces some signal.

## Patterns shipped

Three patterns from this build are documented separately in [/patterns](../patterns):

- **[Verify-before-write with Stripe](../patterns/stripe-verify-before-write.md)** — every write that depends on payment must re-verify the payment server-side, even when the same field is also presented client-side
- **[Idempotent submissions via unique constraint + pre-check](../patterns/idempotent-submissions.md)** — two-layer collapse of double-submits, retries, and concurrent-request races
- **[Real-time buy alerts via Stripe webhook → Telegram](../patterns/realtime-buy-alerts.md)** — same-second signal on every paid session, regardless of subsequent form-completion state

## What the buyer sees

The landing page is a single static HTML file, all CSS inline, no framework, no build step. Page weight on first paint is under 6KB before fonts. Mobile-first; the H1, subhead, accent rule, primary CTA, and reassurance line all fit above the fold on a 375×667 viewport (the iPhone SE / typical LinkedIn-in-app size) with 225+ pixels of clearance. Three sections walk the buyer through what they're getting (HOW IT WORKS), what's in the deliverable (WHAT YOU GET), and who is writing it (WRITTEN BY). Two CTAs, both pointing to the same Stripe Checkout link. No popups, no chat widgets, no exit-intent modals.

## What I'd do differently

Two things from this launch worth carrying forward:

The audit phase from V40 worked again. Before writing any code I enumerated every potential surface — every form, every function, every column — and rejected all the work that didn't need to happen. Most of the Decode tier was carved out by what we explicitly chose not to build: no quiz, no archetype generation, no account creation, no auth, no email verification, no shopping cart, no admin dashboard. The audit isn't only useful when scoping a fix. It's also useful when scoping a launch.

The thing I'd change: I shipped a Telegram-only alert pipeline, but a redundancy second channel (email digest at end of day, or Slack mirror) would catch the failure mode where Telegram itself is the problem. For a one-buyer-a-day rate that's tolerable. If the Decode tier scales, the second channel becomes load-bearing.
