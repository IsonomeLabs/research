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
