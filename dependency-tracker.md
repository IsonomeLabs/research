# Dependency Tracker

Tracks what's blocked and what it's waiting for.
The improvement cron checks this FIRST — if a blocker was resolved, it acts immediately.

## Active Blockers

<!-- Format:
### DEP-001: [short title] — YYYY-MM-DD
- **Blocking**: [which agent]
- **Waiting for**: [research topic slug or external input]
- **Since**: YYYY-MM-DD
- **Urgency**: critical | high | medium
- **Research status**: queued | in-progress (by which cron) | complete
- **Resolution**: [empty until resolved]
-->

## Resolved Blockers

<!-- Move here when research completes -->

## Stale Resolution Queue

<!-- Items that were resolved but the improvement cron hasn't acted on yet.
     The improvement cron MUST clear this queue before going SILENT. -->
