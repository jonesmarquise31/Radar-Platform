# 004 — Scheduled Aggregate Refresh Inside the Database

**Date:** May 3, 2026
**Status:** Accepted

## Context

The population aggregate that grounds every diagnosis must refresh after new submissions land. Refresh could be triggered externally (a workflow tool, a cron service, a separate scheduled job in the runtime layer) or internally (a database-native scheduler invoking a SQL function on a fixed cadence).

The choice affects operational surface area, observability, and the failure modes the platform has to guard against.

## Decision

Database-native scheduler. The refresh is a single SQL function. The schedule lives next to the data the function operates on. External orchestration is not introduced.

## Consequences

- **One less moving part.** No external workflow tool to provision, monitor, authenticate, or rotate credentials for. The refresh is part of the data layer's own lifecycle.
- **Schedule lives next to the data.** Both the data and the schedule are in one place. Operators inspecting the database can see the next-refresh time without consulting a separate system.
- **Manual trigger always available.** The owner can invoke the SQL function from any database client at any time. No external tool needs to be reachable for an out-of-cycle refresh.
- **Trade-off acknowledged.** The scheduler is limited by what a database scheduler can do. It does not natively handle multi-step orchestration, cross-system retries, or alerting on failure. If the refresh ever needs cross-system coordination — fan-out to a notification pipeline on completion, dependency on an upstream data source, multi-stage data pipeline — an external scheduler will be the right move at that point. Until then, the database-native option is correct.

---

© 2026 Marquise Jones. All rights reserved.
