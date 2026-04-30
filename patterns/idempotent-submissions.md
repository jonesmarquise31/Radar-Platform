# Pattern: Idempotent Submissions

A submission flow — buyer pays, then submits a form, function writes a row plus an associated file — needs to handle three real-world cases without producing duplicates or partial state:

1. **The double-submit.** Buyer hits "Submit" twice in quick succession. Browser fires both requests before the first response lands.
2. **The retry-on-network-blip.** First request times out from the buyer's perspective; they hit submit again. The first request actually succeeded server-side.
3. **The concurrent race.** Two requests with the same session ID arrive simultaneously (rare, but real for buyers using multiple devices or for sessions left open in two tabs).

A submission flow that ignores these produces duplicate rows, orphaned files, and operator confusion. The pattern below collapses all three cases into one canonical write.

## The two-layer collapse

Use two layers of idempotency, both keyed on the same unique business identifier (here, `stripe_session_id`):

### Layer 1 — Pre-write lookup

Before the write, check if a row already exists for this identifier:

```js
const { data: existing, error: lookupErr } = await supabase
  .from('decode_submissions')
  .select('id')
  .eq('stripe_session_id', session_id)
  .maybeSingle();

if (lookupErr) {
  return resp(500, { success: false, error: 'Could not save your submission.' });
}
if (existing) {
  return resp(200, { success: true, alreadySubmitted: true });
}
```

This catches the second request when the first request has already completed. Most retry-on-blip cases land here.

### Layer 2 — Unique-constraint catch

The pre-check is not enough on its own. Two requests can pass the pre-check at the same time (both see no existing row) and then both attempt to insert. This is the concurrent race. The fix is at the database level:

```sql
create table decode_submissions (
  id                 uuid primary key default gen_random_uuid(),
  stripe_session_id  text not null unique,
  ...
);
```

The `unique` constraint on `stripe_session_id` means that even if two inserts race past the pre-check, only one succeeds. The other gets a `unique_violation` error, which Postgres reports as SQLSTATE `23505`. Catch it and treat it as a successful write:

```js
const { error: insertErr } = await supabase.from('decode_submissions').insert({...});
if (insertErr) {
  if (insertErr.code === '23505') {
    return resp(200, { success: true, alreadySubmitted: true });
  }
  return resp(500, { success: false, error: 'Could not save your submission.' });
}
```

Together: the pre-check makes the common case fast (no DB write, immediate success response). The unique-constraint catch makes the race case correct (DB write blocked, success response anyway).

## The file upload concern

If the submission has an associated file upload (a resume, an image, a PDF) the order of operations matters. The naive flow is:

1. Insert row
2. Upload file
3. Return success

This produces an inconsistent state if the upload fails after the row is written. Better:

1. Pre-check existing row → return success if found
2. Upload file with `upsert: true` (overwrite is safe; we know we have the right session)
3. Insert row
4. On unique-violation at insert, return success anyway — file is in place, row is in place from the previous successful request

The `upsert: true` flag on the file upload is what makes this safe under retry. A retry that lands at step 2 with the file already present overwrites the same path with the same file (or a slightly different file from the same buyer's same session, which is fine — the canonical row references the current file). A retry that lands at step 3 collapses via the unique-violation catch.

The path itself should be namespaced by the unique identifier:

```js
const storagePath = `${session_id}/${safeFilename}`;
```

This guarantees no path collision between buyers, and makes the file path predictable for operator queries ("show me everything from session `cs_live_...`").

## Why one layer alone is insufficient

**Pre-check alone:** loses to the concurrent race. Both requests pass the pre-check, both insert, you have two rows.

**Unique constraint alone:** correct, but every retry burns an INSERT (which produces lock contention on the unique index) and forces the function to interpret a 23505 error code as success — which works but is harder to reason about than an explicit pre-check.

Both layers together: cheap fast-path for common retries (pre-check), correctness floor for races (unique constraint), no duplicates ever.

## What this pattern doesn't solve

This pattern handles same-identifier double-submits. It does not handle:

- **Different identifier, same buyer.** A buyer who pays twice (creates two Stripe sessions) and submits both. Both submissions are valid; both rows are correct.
- **Server-side processing failures after the row lands.** If the row is written but downstream delivery (email, PDF generation) fails, the row exists but the deliverable doesn't reach the buyer. Solve this with a `decode_status` column and operator monitoring, not with idempotency.
- **Adversarial replay.** If an attacker captures a valid request and replays it, they don't create a duplicate row (idempotency catches it) but they also don't add new value. This is a feature, not a bug — but it's worth noting the pattern doesn't authenticate the request, only deduplicates it.

## Used in

- `submit-decode` function (Radar Decode intake form) — pre-check on `stripe_session_id` + unique constraint catch + file upload with `upsert: true`

## Related patterns

- [Verify-before-write with Stripe](./stripe-verify-before-write.md) — the verification that runs *before* the idempotency layer
