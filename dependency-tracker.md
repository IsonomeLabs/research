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

### DEP-004: ReflectionPillar (Fourth Pillar) Extraction — 2026-06-14
- **Blocking**: isonome-improvement agent (formalizing four-chamber architecture)
- **Waiting for**: Quadcameral architecture research (theory + integration roadmap)
- **Since**: 2026-06-14
- **Urgency**: medium
- **Research status**: complete (content/multi-agent/quadcameral-reflexive-architecture.md) — unified theory + 5-phase integration roadmap
- **Resolution**: Pending implementation decision by framework team

### DEP-005: Global Workspace Broadcast Mechanism — 2026-06-14
- **Blocking**: isonome-improvement agent (cross-pillar communication upgrade)
- **Waiting for**: Broadcast-based inter-pillar communication design (Global Workspace Theory mapping)
- **Since**: 2026-06-14
- **Urgency**: low
- **Research status**: complete (content/multi-agent/quadcameral-reflexive-architecture.md — Section 3.3)
- **Resolution**: Pending implementation decision by framework team

### DEP-006: EVOC Meta-Reasoning for Deliberation Stopping — 2026-06-14
- **Blocking**: isonome-improvement agent (deliberation budget control)
- **Waiting for**: Expected Value of Computation formalism for LLM deliberation stopping criterion
- **Since**: 2026-06-14
- **Urgency**: low
- **Research status**: complete (content/multi-agent/quadcameral-reflexive-architecture.md — Section 2.6)
- **Resolution**: Pending implementation decision by framework team

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
