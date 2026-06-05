# Cross-Agent Knowledge Bank

Shared intelligence across all cron agents.
**FORCING FUNCTION**: Every agent MUST add at least one entry per active session,
or add a dated note explaining why there's nothing new. No silent runs on the bank.

## Blocker Registry

<!-- Format:
### BLOCKER: [short description] — YYYY-MM-DD
- **Blocking**: which agent/cron (e.g., isonome-improvement)
- **Depends on**: research topic slug
- **Unblock condition**: what needs to be true
- **Urgency**: critical | high | medium
- **Status**: blocked | research-in-progress | resolved
- **Resolved by**: research doc path (when done)
-->

## Architecture Patterns

<!-- Format:
### PATTERN: [name] — YYYY-MM-DD
- **What**: one-sentence description
- **Applies to**: which agents/systems
- **Source**: research doc or discovery context
- **Quality**: ★★★ (how well-validated this pattern is)
- **Usage count**: how many times it's been reused
-->

### PATTERN: Per-axis adaptive damping — 2026-06-03
- **What**: Each tension axis gets its own dynamic damping coefficient that boosts on oscillation and decays on stability, replacing the static base damping only when needed
- **Applies to**: EquilibriumEngine, any system with feedback loops that can oscillate
- **Source**: iteration-015-adaptive-damping-controller.md
- **Quality**: ★★★★★ (75 dedicated tests + 704 full-suite green)
- **Usage count**: 1

### PATTERN: Calibration-driven weight rebalancing — 2026-06-03
- **What**: When calibration ECE > 0.15, attention weights α↔β shift based on direction: overconfident → boost α (surprisal), underconfident → boost β (MI), zero-sum, capped at 0.12
- **Applies to**: AttentionEquilibriumSystem, any weighted scoring system with metacognitive feedback
- **Source**: iteration-016-calibration-weight-rebalancing.md
- **Quality**: ★★★★★ (26 dedicated tests + 732 full-suite green)
- **Usage count**: 1

## Discoveries

<!-- Format:
### DISCOVERY: [title] — YYYY-MM-DD
- **Finding**: one sentence
- **Impact**: what changes because of this
- **Discovered by**: which agent
- **Validation**: confirmed | speculative
- **Source**: research doc path
-->

### DISCOVERY: Auto-registration prevents KeyError in sparse-axis controllers — 2026-06-03
- **Finding**: When a controller tracks per-axis state and some axes are only contacted via feedback (not pre-registered), the stable feedback path skips writing to the state dict, causing KeyError on read
- **Impact**: Any per-axis controller must either pre-register all axes or auto-register on first contact — the stable path won't create entries
- **Discovered by**: isonome-improvement
- **Validation**: confirmed (bug found in AdaptiveDampingController, test added)
- **Source**: iteration-015-adaptive-damping-controller.md

## Integration Points

<!-- Format:
### INTEGRATION: [system A] ↔ [system B] — YYYY-MM-DD
- **Connection**: what flows between them
- **Protocol**: how (signal, file, API, shared state)
- **Status**: active | proposed | broken
-->

### INTEGRATION: AdaptiveDampingController ↔ EquilibriumEngine — 2026-06-03
- **Connection**: Engine reads effective_damping per axis before adjust(); controller receives on_feedback after each adjust
- **Protocol**: Shared state via engine.adaptive_damping attribute; opt-in via enable_adaptive_damping flag
- **Status**: active

### INTEGRATION: Calibration Weight Rebalance ↔ AttentionEquilibriumSystem — 2026-06-03
- **Connection**: _modulate_weights() reads calibration state; collect_garbage() reports rebalance deltas in GarbageCollectionReport
- **Protocol**: Internal method call; calibration state set via set_calibration_state(); zero-sum α↔β shift composed additively with tension weights
- **Status**: active

## Quality Feedback (cross-agent ratings)

<!-- Format:
### RATING: [producing agent]'s [output] — YYYY-MM-DD
- **Rated by**: [consuming agent]
- **Score**: 1-5 (1=useless, 3=useful, 5=breakthrough)
- **What worked**: specific things that were valuable
- **What didn't**: specific things that were wrong or missing
- **Would reuse**: yes | with modifications | no
-->

## Session Summaries

<!-- Every agent adds one line per session -->
<!-- Format: [AGENT] YYYY-MM-DD HH:MM — [did this] | blockers: [N] | discoveries: [N] | bank entries: [N] -->
[IMPROVEMENT] 2026-06-03 00:00 — feat: AdaptiveDampingController (per-axis dynamic damping, 75 tests, 3 bug fixes, 704/704 green) | blockers: 0 added/0 resolved | discoveries: 1 | bank entries: 3
[IMPROVEMENT] 2026-06-03 01:00 — feat: calibration-driven weight rebalancing (_compute_calibration_weight_rebalance, α↔β zero-sum shift, 26 new tests, 732/732 green) | blockers: 0 added/0 resolved | discoveries: 1 | bank entries: 2

## Session Summaries
[IMPROVEMENT] 2026-06-03 12:00 — Fixed convergence_ratio sentinel (1.0→np.inf) and double-adjust_default bug in apply_task_type_profile; +4 tests (invariant, inf sentinel, single-adjust, exact 1/3) | blockers: 0 added/resolved | discoveries: 2 (convergence_ratio semantic ambiguity, pre-adaptation intermediate-corruption via clamping) | bank entries: 1

[IMPROVEMENT] 2026-06-04 03:46 — feat: RecursiveReasoningEngine to_dict/from_dict serialization, recall tokenization fix (strip punctuation), ActionRisk enum fix (MEDIUM→MODERATE), ruff lint cleanup (42+ unused imports, duplicate methods, E741), frozendict utility class, 31 cross-pillar integration tests (cognition→praxis, praxis→mneme, mneme→cognition, full pipeline, serialization round-trips) — 794/794 green | blockers: 0 added/0 resolved | discoveries: 3 (recall punctuation mismatch, ActionRisk enum naming inconsistency, duplicate to_dict/from_dict in HierarchicalMneme) | bank entries: 2

### DISCOVERY: Recall punctuation mismatch breaks memory retrieval — 2026-06-03
- **Finding**: HierarchicalMneme.recall() tokenizes with simple split() while store() preserves punctuation, causing entries like "Action 'deploy' completed successfully" to be unsearchable by the word "deploy" (stored as "'deploy'" with quotes)
- **Impact**: Memory retrieval is unreliable for any entry with quoted/punctuated content — critical for action tracking
- **Discovered by**: isonome-improvement agent (cross-pillar integration testing)
- **Fix**: strip punctuation from tokens in recall() so bare words match punctuated entries

### DISCOVERY: ActionRisk enum naming inconsistency (MEDIUM vs MODERATE) — 2026-06-03
- **Finding**: Praxis orchestrator defined ActionRisk with MODERATE but some code referenced MEDIUM, causing KeyError in risk assessment
- **Impact**: Risk assessment on actions with medium risk would crash; cross-pillar tests caught this
- **Discovered by**: isonome-improvement agent (cross-pillar integration testing)
- **Fix**: Aligned all references to use MODERATE (the actual enum value)

[DASHBOARD] 2026-06-04 04:50 — feat: Developer Visualization Dashboard (HTML/JS frontend + Python HTTP server) under /root/isonome-framework/dashboard/. 8 tension axis bars, stress semicircular gauge, pillar activity cards, attention budget utilization, mneme 3-tier memory display, calibration ECE/bias/MCE metrics. Dark theme, 1s polling, demo mode with simulated agent ticks. Fixed: _mneme→mneme attribute access, stats dict handling, ConfidenceCalibrator API (compute_ece/compute_bias/compute_mce). Role rotation counter-based (was "A", now "2"). | blockers: 0 added/0 resolved | discoveries: 2 (MnemePillar.mneme not _mneme, _stats.working_count always 0 — use len(_working) instead) | bank entries: 2

### DISCOVERY: MnemePillar exposes mneme not _mneme — 2026-06-04
- **Finding**: MnemePillar's HierarchicalMneme instance is accessible as .mneme (public), not ._mneme (private). Code that tried _mneme got AttributeError.
- **Impact**: Any dashboard or external tool accessing the mneme must use .mneme
- **Discovered by**: isonome-dashboard agent
- **Validation**: confirmed (tested with live agent)
- **Fix**: Changed all _mneme references to mneme

### DISCOVERY: MnemeStats.working_count always returns 0 — 2026-06-04
- **Finding**: HierarchicalMneme._stats.working_count, episodic_count, semantic_count are always 0 even after storing entries. The actual tier sizes are available via len(_working), len(_episodic), len(_semantic).
- **Impact**: Stats counters are unreliable for monitoring; must use len() on tier containers
- **Discovered by**: isonome-dashboard agent
- **Validation**: confirmed (7 entries stored, stats showed 0, len(_working) showed 7)
- **Fix**: Dashboard uses len() as primary with stats as fallback

[IMPROVEMENT] 2026-06-04 06:00 — feat: Calibration-Based Rehearsal Scheduling (iter-018). RehearsalScheduler class with interval computation (significance factor, calibration modes, tension modulation). Two integration methods on HierarchicalMneme: get_rehearsal_candidates (urgency-sorted) and rehearse_due_candidates. Fixed 3 bugs from incomplete prior run: missing set_calibration_state signature, inverted significance formula (1/(0.5+sig)→0.5+sig), inverted tension modifier sign (-0.15→+0.15). 27 new tests, 821/821 green. | blockers: 0 added/0 resolved | discoveries: 3 | bank entries: 3

### DISCOVERY: Inverted significance factor in RehearsalScheduler — 2026-06-04
- **Finding**: The significance_factor formula used 1/(0.5+significance) which made high-significance entries get SHORTER intervals — the exact opposite of the intended behavior (high sig = more stable = rehearse less often)
- **Impact**: Any system using the RehearsalScheduler for interval computation would under-rehearse important memories and over-rehearse unimportant ones
- **Discovered by**: isonome-improvement (test failure analysis)
- **Validation**: confirmed (test_significance_factor failed with inverted formula)
- **Fix**: Changed to 0.5 + significance (range 0.5-1.5), so high sig → larger factor → longer interval

### DISCOVERY: Tension modifier sign inversion in RehearsalScheduler — 2026-06-04
- **Finding**: tension_modifier = -0.15 * consolidate_prune_position gave consolidate (neg position) a positive modifier (longer intervals) and prune (pos position) a negative modifier (shorter intervals) — the exact opposite of the docstring intent
- **Impact**: Consolidate mode would extend rehearsal intervals (weakening memories) while prune mode would shorten them (strengthening memories that should decay)
- **Discovered by**: isonome-improvement (test failure analysis)
- **Validation**: confirmed (test_consolidate_shortens_interval and test_prune_extends_interval both failed)
- **Fix**: Changed to +0.15 * consolidate_prune_position so consolidate→shorter, prune→longer

### DISCOVERY: Double-counting rehearsal spacing in interval formula — 2026-06-04
- **Finding**: The original interval formula multiplied effective_half_life (which already includes 1.5^rehearsal_count) by a separate rehearsal_expansion of 1.3^rehearsal_count, double-counting the spacing effect
- **Impact**: Intervals for well-rehearsed entries would be ~1.5^n × 1.3^n = 1.95^n instead of the intended 1.5^n, making them rehearse far less often than expected
- **Discovered by**: isonome-improvement (test_rehearsal_expansion ratio was 7.4x instead of ~2.2x)
- **Validation**: confirmed (ratio calculation showed compound growth)
- **Fix**: Removed separate rehearsal_expansion factor; effective_half_life's built-in 1.5^count is sufficient


---

## Session: 2026-06-04 — Isonome Iter-019 (Role A: Improvement)

### Completed: Calibration-Gated Delegation (iter-019)

**Feature**: DelegationGate in `isonome/praxis/delegation.py` — when the confidence calibrator's ECE exceeds a threshold (default 0.15), high-risk actions are delegated to subagents instead of executed directly.

**Key Design Decisions**:
- 5 operating modes: UNCALIBRATED, WELL_CALIBRATED, MODERATE, OVERCONFIDENT, UNDERCONFIDENT
- Overconfident systems delegate at MODERATE+ risk (threshold=2) — they over-estimate capability
- Underconfident systems delegate at HIGH+ risk (threshold=3) — risk classifications still trusted
- TRIVIAL risk always executes directly regardless of calibration
- Phase 1.7 runs after risk gate (Phase 1) and confidence gate (Phase 1.5), before parallelism (Phase 2)
- Delegated actions are marked BLOCKED and tracked via DelegationRecord for feedback

**Cross-pillar pipeline**: Cognition (ECE/bias) → Praxis (delegation decision) → Cognition (delegation record feedback)

**Tests**: 895 total (74 new), all passing. Commit ab16d3b pushed to origin main.

**Bug Found & Fixed**: PraxisPillar.__init__ referenced `delegation_gate` without it being a parameter — added the parameter and docstring.

**Pattern Discovered**: Delegation gate and risk gate interact — actions blocked by the risk gate are never considered for delegation. This is correct: the risk gate already prevents execution, so delegating would be redundant.


## Session: 2026-06-04 — iter-020 Delegation Outcome Tracking

**Agent:** isonome-framework cron (Role A: Improvement)
**Commit:** ea88642 + fac9a45 + d1522a5 (pushed to origin/main)

### Work Completed
1. **Fixed stress feedback test bug** (3eecbde): Tests `test_stress_feedback_when_drifted` and `test_stress_feedback_disabled` were broken because `EquilibriumEngine.__init__` resets all axis positions to `default_position`. Fixed by using `engine.apply_feedback()` to create real drift after construction.
2. **Implemented iter-020: Delegation Outcome Tracking** (ea88642): New `DelegationOutcome` dataclass + `record_outcome()` on `DelegationGate` that feeds back to the calibrator + dynamic ECE threshold adaptation based on delegation accuracy (α=0.02, bounds [0.05, 0.5], rolling window of 50). Full serialization round-trip support.
3. **Cross-pillar integration tests** (fac9a45): 7 tests for the delegation → calibrator feedback loop using real `ConfidenceCalibrator`.
4. **Serialized iteration MD** (d1522a5): iteration-020-delegation-outcome-tracking.md

### Key Discovery
- `EquilibriumEngine.__init__` always resets axis positions to `default_position` (line 684-686). Tests that need non-default positions must use `apply_feedback()` after construction, not set `position` in the axis constructor. This is a known pitfall from the skill doc.

### Test Count
- Before: 957 tests
- After: 1003 tests (+46 new)

### Quality Rating
- iter-020 delegation outcome tracking: **4/5** — Clean implementation, well-tested, conservative adaptation. Would be 5/5 with a live simulation demo.

