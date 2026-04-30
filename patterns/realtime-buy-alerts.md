# Pattern: Realtime Buy Alerts via Stripe Webhook

A serverless function that listens for `checkout.session.completed` events from Stripe and pushes a one-line notification to a chat channel (Telegram, Slack, Discord — wiring is identical) the moment a payment is confirmed. Gives the operator same-second signal on every paid session, including buyers who pay and then bail on any subsequent intake form. Used as the operational dashboard in lieu of a dashboard.

## Why this and not a daily Stripe email digest

Stripe's built-in email notifications fire eventually. For a low-volume product where every buyer matters and the buyer's experience post-payment depends on the operator manually delivering something within hours, "eventually" is not enough. A Telegram notification on a phone, in the same minute as the payment, does three things an email digest doesn't:

1. **Triggers operator attention immediately.** The operator knows there is new work to do without checking anything.
2. **Surfaces bailers.** A buyer who pays and never completes a follow-up intake form still triggers the alert. The operator can manually reach out and recover the submission, or refund and clean up.
3. **Doubles as a smoke test for the live payment flow.** The first time a real buyer's alert lands in chat is also the first time the entire production pipeline (Stripe → webhook → function → chat) is exercised end-to-end.

## The flow

```
Buyer completes Stripe Checkout
              │
              ▼
Stripe sends signed POST to webhook URL
              │
              ▼
Function: verify Stripe signature
              │
              ▼ (if signature valid)
Function: extract session details
              │
              ▼
Function: format and POST to chat API
              │
              ▼
Function: return 200 to Stripe regardless of chat success
```

## The function

```js
const Stripe = require('stripe');

exports.handler = async (event) => {
  if (event.httpMethod !== 'POST') return { statusCode: 405, body: 'Method not allowed' };

  const stripe = Stripe(process.env.STRIPE_SECRET_KEY, { apiVersion: '...' });
  const sig = (event.headers['stripe-signature'] || event.headers['Stripe-Signature']);
  const rawBody = event.isBase64Encoded
    ? Buffer.from(event.body, 'base64').toString('utf8')
    : event.body;

  let stripeEvent;
  try {
    stripeEvent = stripe.webhooks.constructEvent(rawBody, sig, process.env.STRIPE_WEBHOOK_SECRET);
  } catch (err) {
    return { statusCode: 400, body: 'Webhook signature verification failed' };
  }

  if (stripeEvent.type !== 'checkout.session.completed') {
    return { statusCode: 200, body: 'ignored' };
  }

  const session = stripeEvent.data.object;
  const text = formatBuyAlert(session);

  // Fire-and-forget: log result but don't fail the webhook on chat failure.
  try {
    const res = await fetch(`https://api.telegram.org/bot${process.env.TELEGRAM_BOT_TOKEN}/sendMessage`, {
      method: 'POST',
      headers: { 'content-type': 'application/json' },
      body: JSON.stringify({ chat_id: process.env.TELEGRAM_CHAT_ID, text }),
    });
    const result = await res.json().catch(() => ({}));
    console.log('[stripe-webhook] telegram', { ok: result.ok, message_id: result.result && result.result.message_id });
  } catch (err) {
    console.log('[stripe-webhook] telegram error', err.message);
  }

  return { statusCode: 200, body: 'ok' };
};
```

Three details worth calling out:

**The signature verification uses raw bytes, not the parsed body.** Stripe signs the exact bytes that were sent over the wire. If your function framework parses JSON before your handler runs, the parse-and-restringify step changes whitespace and breaks signature verification. On Netlify Functions, the raw body comes in as `event.body` (utf-8 string) or `event.body` base64-decoded (binary). The code above reconstructs the original bytes for signature verification.

**The function returns 200 to Stripe regardless of whether the chat send succeeded.** Stripe retries failed deliveries (non-2xx responses) for up to 3 days. If the chat API is briefly unavailable and you respond with 5xx, Stripe will keep retrying — and each retry will produce a new chat message attempt, potentially a new chat message if Telegram comes back online during the retry window. This produces duplicates. Returning 200 unconditionally on signature-verified events accepts the trade-off: a Telegram outage means a missed alert, but no duplicates and no retry storm.

**The signature secret comes from a webhook endpoint, not from the API secret.** When you create a webhook endpoint in the Stripe dashboard or via API, Stripe generates a `whsec_…` signing secret for that specific endpoint. This is different from your `sk_live_…` API secret. The webhook endpoint's signing secret is what `constructEvent` validates against; the API secret is for outgoing API calls only. Mixing them up produces a "no signatures found matching the expected signature for payload" error that's confusing to debug.

## Setup steps

1. **Create a chat bot** (Telegram example):
   - Message `@BotFather` on Telegram, send `/newbot`, name it whatever
   - Save the bot token (looks like `1234567890:AAH…`)
   - Message `@userinfobot` to get your numeric chat ID
2. **Create the Stripe webhook endpoint** (via API or dashboard):
   ```
   curl -X POST https://api.stripe.com/v1/webhook_endpoints \
     -u sk_live_...: \
     -d url=https://your-site.netlify.app/.netlify/functions/stripe-webhook \
     -d "enabled_events[]=checkout.session.completed"
   ```
   - The response includes a `secret` field starting with `whsec_…` — capture this
3. **Set the four env vars** on the function host:
   - `STRIPE_SECRET_KEY` (existing)
   - `STRIPE_WEBHOOK_SECRET` (the `whsec_…` from step 2)
   - `TELEGRAM_BOT_TOKEN`
   - `TELEGRAM_CHAT_ID`
   - On Netlify: mark secrets with `--secret` flag, scope explicitly to `builds functions runtime` (the default scope for `--secret` does not include `functions`, so the Lambda can't read them at runtime — easy to miss)
4. **Deploy and verify** with two checks:
   - Smoke test: POST garbage with a fake Stripe-Signature header. Expect 400 ("signature verification failed"). Confirms env vars resolve and the verification path runs.
   - Round-trip test: send any test message via the chat API directly to confirm the bot/channel pipeline works before any real buyer triggers it.

## What to put in the message

Keep it scannable. The operator is reading this on a phone, possibly while doing something else. Five lines maximum:

```
NEW DECODE  ·  $47.00
Email: buyer@example.com
Name: Sample Buyer
Time: Apr 29, 11:42 PM PT
Session: cs_live_a1b2c3d4e5f6g7...
```

Time should be in the operator's timezone, not UTC. Truncate the session ID to ~24 chars + ellipsis — the full ID is 50+ chars and isn't useful in the alert; the operator will look up details in Stripe Dashboard if they need them.

## What this pattern doesn't solve

- **Outage on the chat channel** — alerts during the outage are lost. For low-volume products this is acceptable; for high-stakes flows, add a redundant channel (email digest, second chat platform).
- **Operator timezone shifts.** If the operator travels and the function is hardcoded to one timezone, alert timestamps are confusing. Use an env var if this matters.
- **Auth.** The webhook endpoint URL is public. If Stripe's signature verification fails, the request is rejected — but you should also not expose any other sensitive operations behind the same path. The function is a one-job endpoint.

## Used in

- Radar Decode launch — Stripe webhook for `checkout.session.completed`, posting to a Telegram bot already in use for other Radar operational alerts

## Related patterns

- [Verify-before-write with Stripe](./stripe-verify-before-write.md) — for any write paths that depend on payment, separate from the webhook
- [Idempotent submissions via unique constraint + pre-check](./idempotent-submissions.md) — the webhook fires once per event, but downstream writes still need idempotency
