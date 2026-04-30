# Pattern: Verify-Before-Write with Stripe

When a server-side action depends on a Stripe payment having succeeded, the action must re-verify the payment via the Stripe API at the moment of the write — even if the client is also presenting the same field. The client cannot be trusted to honestly report payment state, even if "the client" is your own front-end code running on your own domain.

## The shape of the bug

Imagine a flow where a buyer pays via Stripe Checkout, redirects back to your site with `?session_id=cs_live_…` in the URL, and submits a form. The form POSTs to your function with the session ID and the buyer's data. The naive write looks like this:

```js
// VULNERABLE — do not ship this
const { session_id, linkedin_url, resume } = parsed;
await db.from('submissions').insert({
  session_id,
  linkedin_url,
  resume_path,
});
```

This is open to anyone who guesses the function URL. A request like:

```
POST /.netlify/functions/submit-decode
Content-Type: multipart/form-data

session_id=cs_live_anything
linkedin_url=https://linkedin.com/in/attacker
resume=<any file>
```

…lands a row in your database without payment having occurred. The session ID is just a string the client provided; nothing about its presence in the request implies a successful Stripe charge.

## The fix

Every server-side action gated on payment must, at the moment of the write, retrieve the Stripe session by ID and check three things:

```js
const stripe = Stripe(process.env.STRIPE_SECRET_KEY, { apiVersion: '...' });

let session;
try {
  session = await stripe.checkout.sessions.retrieve(session_id);
} catch (err) {
  return resp(400, { success: false, error: 'Payment session not found.' });
}

if (session.payment_status !== 'paid') {
  return resp(402, { success: false, error: 'Payment is not marked complete yet.' });
}
if (session.amount_total !== expectedAmountCents) {
  return resp(402, { success: false, error: 'Payment amount does not match this offer.' });
}

// Now safe to write
```

The three checks each block a distinct attack:

1. **Retrieve succeeds** — proves the session ID corresponds to a real Stripe session in the merchant's account
2. **`payment_status === 'paid'`** — proves the session resulted in a successful charge, not a `'unpaid'` or `'no_payment_required'` state
3. **`amount_total === expectedAmountCents`** — proves the buyer paid the price for *this* offer, not a different (cheaper) offer in the same Stripe account

The third check is easy to forget and important. If the merchant has multiple Payment Links for products at different prices, an attacker who paid $1 for a different product in the same Stripe account could submit that session ID against the $47 offer's intake form. The amount comparison rejects this.

## Where the verification belongs

Three things to know about placement:

**It belongs in every write path that's gated on payment, not just the obvious one.** If a single function handles intake, every code path through that function that produces a row or upload must run the verification. If two functions both write payment-gated state, both verify.

**It does not belong in middleware that you forget about later.** Putting the check in a shared "verify session" wrapper that runs before the main handler is fine, but only if the wrapper is mandatory — not opt-in via a flag. Verify-before-write is a write-time invariant, and the closer it sits to the actual write, the harder it is to accidentally bypass.

**It is not redundant with Stripe webhooks.** A webhook handler verifies a Stripe-signed payload as proof that the buyer did pay. A verify-before-write check verifies that the session the *current request* is invoking is paid. Both have to exist. The webhook tells you what happened; the verify-before-write protects what you do next.

## Why client-side verification doesn't substitute

A pattern occasionally seen: validate the session ID's format on the client, only allow submission if it matches `cs_(live|test)_[a-zA-Z0-9]+`, and skip server-side verification. This protects against nothing. Anyone can hand-craft a session ID matching the regex; the check is cosmetic.

A more sophisticated pattern: have the client call a `/verify-session` function and only allow submission if that returns OK. This protects against nothing either, as soon as the attacker bypasses your front-end entirely and POSTs directly. The attacker controls what their client does.

Server-side verification is not optional and is not made redundant by anything the client can do.

## Cost

A Stripe API call adds 100-300ms to the function response time. For an interactive flow this is invisible to the buyer. The cost is real but trivial in context.

## Used in

- `submit-decode` function (Radar Decode intake form) — verifies session is paid and matches `DECODE_PRICE_AMOUNT_CENTS` before writing to `decode_submissions` or uploading to storage

## Related patterns

- [Idempotent submissions via unique constraint + pre-check](./idempotent-submissions.md) — once verification passes, the write itself needs to be idempotent
- [Realtime buy alerts via Stripe webhook](./realtime-buy-alerts.md) — the webhook is independent of, and additive to, this pattern
