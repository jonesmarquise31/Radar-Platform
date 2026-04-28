# V40 — Funnel Friction Reduction

**Shipped:** April 2026  
**Surface area:** /quiz (free diagnostic), /rds ($97 paid diagnostic)  
**Deploy hash:** 191911a

## The decision

The free diagnostic at /quiz collected name, email, and phone before the user took a single quiz question. The paid diagnostic at /rds did the same, plus 22 additional quiz questions, all before payment.

A user testing the platform pointed out the obvious: we were asking for too much before delivering anything. Email and phone collection upfront kills conversion. The standard 2026 pattern in high-converting platforms is name-only entry, with email/account creation gated behind delivered value.

## The pattern

The new shape of /quiz:

1. User enters name only
2. User completes the quiz (5 minutes of diagnostic questions)
3. Intermediate state: "Your diagnostic is ready"
4. CTA: "See My Results" → opens account creation modal (email + password only, no phone)
5. Account created → results display

Quiz answers are written to the database the moment the user completes the quiz, with the user_id and lead_id intentionally null. A unique submission_id is stored in browser localStorage. When the user creates an account, the system claims the orphan submission via that ID and links it to the new auth user.

The /rds flow is simpler: phone field removed entirely, full_name and email retained, payment timing unchanged.

## The phased deploy

The change touched two flows but had implications for several others. The deploy was structured in five phases with explicit checkpoints between each:

1. **Audit** — read-only enumeration of every form, every Netlify function, every Supabase column potentially affected. No code changes. The audit revealed the friction was concentrated in only two flows; everything else was already clean. This narrowed scope dramatically.
2. **/quiz refactor** — name-only entry form, intermediate "ready" state, account creation modal, link-user logic to claim orphan submissions by ID
3. **/rds refactor** — phone field removed, function tolerant of missing phone, no schema changes
4. **Regression verification** — smoke test all unrelated flows (/coach, /hiring, /diagnostic, /system, /login) to confirm no breakage
5. **Deploy** — preview first, owner approval gate, production deploy, full smoke test

13/13 PASS on preview. 7/7 PASS on production smoke. Zero regressions.

## The orphan rate trade-off

The new flow introduces an edge case worth naming as an architectural insight, not a bug.

If a user clears their browser data, switches devices, or uses incognito between quiz completion (step 2) and account creation (step 4), the localStorage submission ID is lost. Their quiz answers persist in the database with no path to claim them.

This is a deliberate trade. The friction reduction at the top of the funnel produces dramatically more entries than the orphan rate destroys. The math favors the new pattern. But the orphan rate is now a metric to monitor, not assume away.

The first 30 days post-deploy will produce a real signal. If orphan rate exceeds 25% of total submissions, the next iteration adds an optional email capture at the "ready" state — skippable, but providing a recovery path for users who want their results saved.

## Pattern: Backwards-compatible field removal

Both flows had phone collection that needed to be removed without breaking historical data or downstream consumers. The pattern used:

1. Stop writing to the column. Remove all code paths that populate it.
2. Leave the column nullable in the database. Don't drop it.
3. Update any downstream readers to handle null gracefully.
4. Verify legacy rows still display correctly via existing read paths.

Dropping the column would have broken historical analytics, archived data exports, and any downstream consumers reading the field. The nullable-and-stop-writing pattern preserves history while implementing the new behavior cleanly. The column can be dropped in a future cleanup if it's truly never read again.

## What I'd do differently

The only thing worth flagging: the audit phase saved this entire deploy from being 3-4x larger than it needed to be. The original prompt assumed the friction problem was distributed across all six flows. The audit revealed it was concentrated in two. Always run the read-only audit before scoping the actual work. The cost of audit is trivial. The cost of building the wrong thing is substantial.
