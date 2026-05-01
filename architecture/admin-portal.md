# The Admin Portal

**Surface:** `/admin` (password-gated, not part of the public surface area)

## Overview

The customer-facing surfaces of Radar (`/quiz`, `/rds`, `/coach`, `/hiring`, `/diagnostic`, `/system`) are the *intake* layer — where buyers self-classify, pay, and get diagnosed. The admin portal is the *operations* layer — where the operator actually runs the business day-to-day. Everything that ends up in the buyer's hands (the diagnostic, the booked call, the support reply, the next-stage unlock) gets routed through here.

It's deliberately on the same domain as the customer surfaces, not a separate operator-only deployment. The data flow goes both ways: customer surfaces write to the same Supabase tables that the admin reads and writes. Splitting domains would add a CORS layer and a deployment boundary for no architectural payoff.

Eight tabs, each backed by a coherent slice of the production data model.

## The eight tabs

**CRM Lead Pipeline.** Every lead from every source rolls into one pipeline view — name, email, phone, source, archetype assignment, stage. Filters by stage and by funnel status. Powers manual outreach decisions: who to follow up with, who to leave alone, who to book a call with.

**DM Pipeline.** A LinkedIn DM parser. The operator pastes their inbound inbox; Claude reads every thread and classifies it (intent, lead score, summary). Threads get logged into the CRM with structured fields so manual follow-up doesn't have to re-read the conversation. This is the only LLM-in-the-loop feature in the admin; it converts unstructured inbound text into structured operator state.

**Connections.** LinkedIn connections classifier. Tag, sort, and segment who you're connected to. Used as the audience-segment substrate for the Content Intel tab.

**Content Intel.** Upload LinkedIn analytics screenshots; the system parses what's working (top hooks, top posting times, top themes) and uses it to plan the next week of posts. Plans flow into a scheduled content calendar.

**Support.** Inbound ticket queue. Reads from the same support tables that customer surfaces write to.

**Calendar.** Booking calendar with follow-up scheduling, plus a workshop / event creation interface. Drives the events the platform runs — intake calls, group sessions, paid workshops.

**Database.** Supabase metrics overview. Read-layer on the whole production database — row counts, recent activity, table-by-table health. The operator's at-a-glance "is the platform ok" view.

**Accounts.** The buyer-state machine. Per-buyer view of tier, current stage in the diagnostic flow, access flags, suspension state, free-month status, and Stripe customer linkage. This is where the operator upgrades a buyer to RDS, marks them complete on phase 1, suspends a stale account, or grants a free-month override.

## The intake-vs-operations split

The two-layer separation matters more than any individual tab. Customer-facing surfaces are lean and one-job: someone hits `/quiz`, the only thing visible is the quiz; someone hits `/rds`, the only thing visible is the RDS flow. There is no "admin" cruft on those pages, no toggles, no preview state, nothing that would let a buyer accidentally see operator UI. Operator UI lives behind `/admin` and is auth-gated.

Same shape as a kitchen and a dining room. The buyer never sees the kitchen. The kitchen is where the work happens.

## The Claude-in-the-loop DM parser

The DM Pipeline is worth naming separately as an architectural pattern.

LinkedIn outbound is high-volume and high-variance: hundreds of conversations, each with its own context, intent, and lead-quality signal. Manually reading each one to extract structured CRM fields is the kind of work that doesn't scale and that the operator burns out on first.

The parser solves this by inserting Claude as a classification layer between the raw inbox and the CRM. The operator pastes the inbox; Claude returns structured fields per thread (intent, summary, lead score, recommended next action); the result lands in the database as queryable state. The operator's job becomes "review the structured output and decide" rather than "read every thread end-to-end."

This is a different shape from how Claude is used on the customer side. The customer side runs Claude as the *product* — diagnostic generation. The admin side runs Claude as a *back-office classifier*. Same model, different role.

## What lives behind the auth layer

Everything in the admin portal is operator-only and password-gated. No customer-facing paths route through `/admin`; no `/admin` routes are linked from any customer surface. The buyer never sees that this layer exists.

That's intentional. The admin portal is infrastructure, not product.
