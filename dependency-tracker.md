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

## Active Blockers

### DEP-002: Predictive-SSM Architecture Implementation — 2026-06-07
- **Blocking**: isonome-improvement agent (JEPALayer upgrade to SSM+JEPA)
- **Waiting for**: Mamba/SSM temporal backbone integration research (architecture spec + training recipe)
- **Since**: 2026-06-07
- **Urgency**: medium
- **Research status**: complete (content/robotics/non-transformer-architectures-robotics.md) — architecture proposed, implementation pending
- **Resolution**: Pending implementation decision by framework team

### DEP-003: Belief-State Calibration for DelegationGate — 2026-06-07
- **Blocking**: isonome-improvement agent (DelegationGate calibration upgrade)
- **Waiting for**: Belief-State RWKV integration with ConfidenceCalibrator (replacing ECE with trace(Σ_t))
- **Since**: 2026-06-07
- **Urgency**: medium
- **Research status**: complete (content/robotics/non-transformer-architectures-robotics.md) — theory established, implementation pending
- **Resolution**: Pending implementation decision by framework team
