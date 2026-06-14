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

### PATTERN: Velocity reversal as leading oscillation indicator — 2026-06-04
- **What**: Tracking per-axis velocity (finite difference) and counting sign reversals detects oscillation 2-3 ticks before position stddev exceeds threshold; reversal rate > 0.4 predicts imminent oscillation
- **Applies to**: EquilibriumEngine, any feedback loop system that needs preemptive damping
- **Source**: iteration-021-tension-velocity-tracking.md
- **Quality**: ★★★★★ (59 dedicated tests + 1062 full-suite green)
- **Usage count**: 1

### PATTERN: Network graph representation for multi-agent equilibrium — 2026-06-13
- **What**: Agent networks should be modeled as directed graphs (trust, task, knowledge, communication) with spectral analysis (algebraic connectivity λ₂) to detect global network stability that individual pillar equilibrium cannot see
- **Applies to**: RecursiveMAS, DelegationGate, any multi-agent coordination system
- **Source**: content/multi-agent/graph-representations-agent-networks.md
- **Quality**: ★★★ (research-complete, implementation pending)
- **Usage count**: 0

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

### INTEGRATION: TensionVelocityTracker ↔ EquilibriumEngine — 2026-06-04
- **Connection**: Engine feeds position updates to tracker after each apply_feedback/batch; tracker exposes velocity/momentum/reversal data via engine.velocity_tracker property and PillarEquilibriumView
- **Protocol**: Opt-in via enable_velocity_tracking flag or explicit velocity_tracker constructor param; auto-registers all axes; reset co-resets
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

## Session: 2026-06-04 — iter-021 Tension Velocity Tracking & Momentum-Aware Restoration

**Agent:** isonome-framework cron (Role A: Improvement)

### Work Completed
1. **New class: TensionVelocityTracker** (`isonome/equilibrium/velocity.py`): Per-axis velocity tracking via finite difference, reversal detection (sign change + magnitude threshold), momentum scores (velocity × drift-direction → positive=heading home, negative=drifting away), reversal rate computation, oscillation prediction (is_oscillation_imminent — predictive via velocity reversal rate vs post-hoc stddev).
2. **Engine integration**: `velocity_tracker` / `enable_velocity_tracking` params on EquilibriumEngine constructor; tracker fed after apply_feedback and apply_feedback_batch; reset and serialization round-trip support.
3. **PillarEquilibriumView velocity data**: New properties `velocities`, `momentum_scores`, `oscillation_imminent`; convenience methods `get_velocity()`, `get_momentum_score()`, `is_axis_drifting()`; summary includes velocity when available.
4. **Bug fix**: Reversal detection was comparing against `_prev_velocity` (2 updates ago) instead of current `_velocity` (previous update), missing first sign change after steady movement.
5. **59 new tests**: Construction, registration, velocity computation, momentum scores, reversal detection, reversal rate & prediction, window rollover, reset, serialization, engine integration, view integration, adaptive damping coexistence, edge cases.

### Test Count
- Before: 1003 tests
- After: 1062 tests (+59 new)

### Pattern Discovered
Velocity reversal rate is a leading indicator of oscillation — it detects oscillation tendency 2-3 ticks before position stddev exceeds the threshold. This enables preemptive damping.

### Architecture Addition
```
Feedback → Engine.apply_feedback()
 ├── axis.adjust(delta)             # position
 ├── tracker.on_position_update()   # velocity (NEW)
 └── adaptive_damping.on_feedback() # damping
```

### DISCOVERY: Ternary-to-quantum structural isomorphism — 2026-06-05
- **Finding**: BitNet b1.58's ternary weights {-1, 0, +1} map naturally to quantum gate sequences: +1→identity/CZ, -1→Pauli-Z rotation by π, 0→gate omitted (circuit pruning). This gives ~20× depth reduction vs full-precision variable-angle rotations. No existing paper combines 1-bit transformers with quantum attention.
- **Impact**: Enables practical quantum attention layers on NISQ hardware (~32 qubits for 768-dim attention) by exploiting the ternary→fixed-gate compilation. Could reduce qubit overhead by orders of magnitude for hybrid quantum-classical LLMs.
- **Discovered by**: isonome-research agent (arXiv literature survey)
- **Validation**: speculative (theoretical mapping validated against paper math; no hardware implementation yet)
- **Source**: content/qml/1-bit-transformers-qml.md

### DISCOVERY: Low-precision parameters mitigate barren plateaus — 2026-06-05
- **Finding**: Discretized (quantized) variational circuit parameters reduce barren plateau risk in QNNs because the coarser landscape has fewer spurious local minima. Gradient variance scales as O(1/|Θ|^n) for |Θ| discrete levels, vs O(1/2^n) for continuous. For ternary weights (|Θ|=3), this is a meaningful improvement.
- **Impact**: 1-bit quantum layers may train faster than full-precision quantum layers — the quantization that saves memory also helps optimization
- **Discovered by**: isonome-research agent (from arXiv:2206.09313 by Liu, Lin, Jiang)
- **Validation**: confirmed (published in Mach. Learn.: Sci. Technol. 5 015058, 2024)
- **Source**: content/qml/1-bit-transformers-qml.md

### PATTERN: Ternary-to-quantum gate compilation — 2026-06-05
- **What**: Any ternary weight matrix W ∈ {-1,0,+1}^(m×n) can be compiled into a quantum circuit with depth O(m × ρ) where ρ is the sparsity ratio, using only fixed CZ/identity gates (no variable-angle rotations)
- **Applies to**: BitQuant hybrid architecture, any quantum circuit with discretized parameters
- **Source**: content/qml/1-bit-transformers-qml.md
- **Quality**: ★★ (theoretically sound, not yet empirically validated on hardware)
- **Usage count**: 0

[RESEARCH] 2026-06-05 — Deep research on 1-Bit Transformers for QML: surveyed 20+ arXiv papers (BitNet lineage b1.0→b1.58→2B4T, FBI-LLM, BitDistill, BitRL, hardware accelerators, QML encoding/barren plateaus). Identified green-field opportunity: no paper combines 1-bit weights with quantum attention. Proposed BitQuant architecture (1-bit classical FFN + quantum attention), compiled 7-phase research roadmap, documented ternary-to-quantum structural isomorphism and barren plateau mitigation insight. | blockers: 0 | discoveries: 2 | bank entries: 3



[RESEARCH] 2026-06-07 — Deep research on Non-Transformer Architectures for Robotics Control (topics 2.8+2.9): surveyed 34+ arXiv papers across JEPA world models (V-JEPA 2/2.1, LeJEPA, MC-JEPA, VLA-JEPA, Demo-JEPA, UWM-JEPA, Causal-JEPA, Sub-JEPA, UR-JEPA, seq-JEPA, CLEAR/Drive-JEPA, value-guided JEPA, Gaussian-constrained LeJEPA), Mamba/SSM for robotics (HuMam humanoid locomotion, M²GRPO multi-agent underwater, FlowRAM diffusion policy, CAMRL social nav, Mamba imitation encoder), RWKV recurrent architectures (DREAMSTATE editable world model states, Belief-State RWKV for uncertainty-aware RL), VLA advances (OpenVLA, Octo, WorldFly, TempoVLA, AffordanceVLA, RD-VLA iterative refinement, LoopVLA recurrent feedback), and sim-to-real (MoSA anisotropic stress, CoRMA semantic contact context). Proposed Predictive-SSM architecture combining JEPA world model + Mamba temporal backbone + belief-state uncertainty + equilibrium-gated compute allocation. Identified 8 open research questions. | blockers: 0 | discoveries: 4 | bank entries: 5

### DISCOVERY: LeJEPA linear identifiability — 2026-06-07
- **Finding**: LeJEPA (alignment + Gaussian regularization) provably linearly recovers the world's latent variables from nonlinear observations. Among all additive-noise transition worlds, the Gaussian is the unique distribution for which this holds. This means JEPA-pretrained encoders produce structurally interpretable latent spaces.
- **Impact**: Planning in JEPA latent space is well-conditioned — no nonlinear entanglement. Uncertainty quantification is natural (mean + covariance of the Gaussian). This is the first formal proof that JEPA learns a true world model, not just useful features.
- **Discovered by**: isonome-research agent (arXiv:2605.26379)
- **Validation**: confirmed (published theorem with proof)
- **Source**: content/robotics/non-transformer-architectures-robotics.md

### DISCOVERY: DREAMSTATE — RWKV hidden states are editable world models — 2026-06-07
- **Finding**: The RWKV recurrent hidden state can be edited via conditional diffusion to inject knowledge, correct beliefs, and modify spatial/temporal reasoning. The hidden state is not just a memory buffer but a compressed world representation.
- **Impact**: Robot episodic memory can store RWKV states (compressed vector) instead of text descriptions. Recalling a past experience means loading the state vector and continuing inference from that cognitive state — "flashback" reasoning.
- **Discovered by**: isonome-research agent (arXiv:2601.19221)
- **Validation**: confirmed (published with empirical demonstrations)
- **Source**: content/robotics/non-transformer-architectures-robotics.md

### DISCOVERY: Belief-State RWKV provides per-step uncertainty for calibration — 2026-06-07
- **Finding**: Reformulating RWKV hidden state as a belief state (μ_t, Σ_t) — mean and covariance — provides per-step, per-token uncertainty estimation. The covariance Σ_t grows under ambiguity and shrinks under informative observations, naturally tracking epistemic uncertainty.
- **Impact**: The isonome DelegationGate could use trace(Σ_t) as its calibration signal instead of binned ECE, eliminating the binning artifact and providing true real-time calibration. This would enable fine-grained delegation decisions at every control step.
- **Discovered by**: isonome-research agent (arXiv:2604.09671)
- **Validation**: confirmed (published with RL experiments)
- **Source**: content/robotics/non-transformer-architectures-robotics.md

### DISCOVERY: SSM-for-time, attention-for-space is the emerging robotics architecture pattern — 2026-06-07
- **Finding**: Across HuMam, M²GRPO, FlowRAM, and CAMRL, a consistent pattern emerges: Mamba/SSM handles temporal processing (O(1) per step, grows with episode length), while attention (if used) handles spatial/cross-agent coordination (bounded context, O(N²) is affordable). No paper uses attention for temporal processing when SSM is available.
- **Impact**: The isonome JEPALayer architecture should adopt SSM for temporal backbone (replacing the current frozen-VLA-only approach) and reserve any attention for cross-pillar or multi-agent coordination, not per-step inference.
- **Discovered by**: isonome-research agent (synthesis of 4+ papers)
- **Validation**: confirmed (consistent across independent papers)
- **Source**: content/robotics/non-transformer-architectures-robotics.md

### PATTERN: SSM-for-time attention-for-space — 2026-06-07
- **What**: In robot policy architectures, state-space models (Mamba/SSM) should handle temporal sequence processing (O(1) per step, favorable for long episodes), while attention mechanisms should be reserved for spatial coordination (bounded context like scene size or agent count). Never use attention for temporal processing when SSM is available.
- **Applies to**: JEPALayer, Praxis pillar, any robot policy architecture with temporal and spatial components
- **Source**: content/robotics/non-transformer-architectures-robotics.md (synthesis of HuMam, M²GRPO, FlowRAM, CAMRL)
- **Quality**: ★★★★ (consistent across 4+ independent papers, no counterexamples found)
- **Usage count**: 0

### PATTERN: Equilibrium-gated compute allocation — 2026-06-07
- **What**: The number of recurrent refinement passes (1, 3, or 5) in a robot policy should be controlled by the equilibrium engine's tension level: low tension → 1 pass (reflex), medium tension → 3 passes (deliberate), high tension → 5 passes + JEPA world model rollouts (explore). This creates a continuous compute controller rather than a discrete mode switch.
- **Applies to**: Any architecture with test-time compute scaling (RD-VLA, LoopVLA pattern), the isonome EquilibriumEngine
- **Source**: content/robotics/non-transformer-architectures-robotics.md (proposed architecture)
- **Quality**: ★★ (theoretically motivated, not yet empirically validated)
- **Usage count**: 0

### PATTERN: JEPA world model + SSM inference = predictive-SSM — 2026-06-07
- **What**: Combining JEPA-pretrained visual encoder (world model understanding) with Mamba/SSM temporal backbone (O(1) per-step inference) and belief-state uncertainty (calibration) creates a robot policy architecture with linear identifiability, real-time control, and per-step uncertainty — three properties no existing single architecture provides.
- **Applies to**: JEPALayer upgrade path, any real-time robot control system
- **Source**: content/robotics/non-transformer-architectures-robotics.md (proposed architecture)
- **Quality**: ★★ (theoretically sound, not yet implemented)
- **Usage count**: 0

### DISCOVERY: DelegationGate is a proto-recursive spawning mechanism — 2026-06-10
- **Finding**: The isonome DelegationGate already implements the decision logic for recursive agent spawning (DELEGATE → spawn sub-agent, EXECUTE → handle directly), but the actual spawning mechanism (SubAgentPool) does not exist yet. The gate marks actions as "should be delegated" but they sit in limbo as BLOCKED with no real sub-agent to execute them.
- **Impact**: Implementing a SubAgentPool that actually spawns agents when the gate says DELEGATE would close the loop and create a real recursive multi-agent system. This is the single highest-leverage integration for the framework.
- **Discovered by**: isonome-research agent (RecursiveMAS architecture survey)
- **Validation**: confirmed (code review of delegation.py shows DELEGATE decision but no sub-agent spawning)
- **Source**: content/multi-agent/recursive-mas-framework.md

### DISCOVERY: Recursion depth bounding via monotonically increasing depth counter prevents delegation deadlock — 2026-06-10
- **Finding**: Circular delegation (Agent A delegates to Agent B which delegates back to A) is impossible if each delegation carries a monotonically increasing depth counter that the recipient checks against a max_depth bound. Since the counter only increases and is bounded, the recursion must terminate.
- **Impact**: This eliminates an entire class of deadlocks in recursive agent systems without requiring cycle detection or graph analysis. Simpler and more robust than any alternative.
- **Discovered by**: isonome-research agent (RecursiveMAS architecture survey)
- **Validation**: confirmed (proof by construction — depth counter is integer, monotonically increasing, bounded above)
- **Source**: content/multi-agent/recursive-mas-framework.md

### PATTERN: Equilibrium-gated sub-agent lifecycle — 2026-06-10
- **What**: Parent agents should spawn sub-agents only when stress_level < 0.6 (calm enough to manage children), buffer sub-agent outcomes during high tension (stress > 0.3), and terminate non-critical sub-agents when resource pressure is high. This creates natural backpressure: spawning → pressure → termination → calm → spawning.
- **Applies to**: Any recursive agent system with shared resource constraints, the isonome EquilibriumEngine
- **Source**: content/multi-agent/recursive-mas-framework.md (proposed architecture)
- **Quality**: ★★ (theoretically motivated, not yet implemented)
- **Usage count**: 0

### PATTERN: Calibration inheritance with decay for sub-agent bootstrapping — 2026-06-10
- **What**: When a parent agent spawns a sub-agent, the child's ConfidenceCalibrator should be bootstrapped with the parent's recent prediction-outcome pairs at 0.5× confidence decay. This lets the child start in MODERATE mode (not UNCALIBRATED), enabling faster recursive delegation.
- **Applies to**: DelegationGate, any agent system where sub-agents need calibration before they can delegate
- **Source**: content/multi-agent/recursive-mas-framework.md (proposed architecture)
- **Quality**: ★★ (theoretically motivated, not yet implemented)
- **Usage count**: 0

[RESEARCH] 2026-06-10 — Deep research on RecursiveMAS Framework (topic 2.1): surveyed OS process models, actor model (Erlang/OTP supervision trees), hierarchical RL (options framework), agentic frameworks (AutoGen, CrewAI, OpenAI Swarm), recursive neural networks. Mapped existing isonome components to recursive architecture (DelegationGate=spawning decision, EquilibriumEngine=lifecycle governor, MessageBus=inter-agent communication, TensionEventLog=audit trail). Proposed Equilibrium-Gated Recursive Agent Architecture (EGRAA) with 5 phases: sub-agent pool, equilibrium-gated lifecycle, recursion budget with trust decay, calibration inheritance, cross-pillar integration. Identified 6 open research questions. Key finding: DelegationGate is a proto-recursive mechanism — it decides when to delegate but has no actual sub-agent to delegate to. Implementing SubAgentPool is the highest-leverage integration. | blockers: 0 | discoveries: 2 | bank entries: 5

### DISCOVERY: Binary hash codes are exact quantum basis states — lossless isomorphism — 2026-06-11
- **Finding**: Every k-bit semantic hash code b ∈ {0,1}^k maps bijectively to a k-qubit computational basis state |b⟩ = |b₁⟩⊗|b₂⟩⊗...⊗|b_k⟩. This mapping is lossless, preserves Hamming distance, and requires no encoding/decoding overhead. It makes semantic hashing the natural classical interface to quantum memory systems.
- **Impact**: Quantum amplitude amplification can search a database of N semantically-hashed items in O(√N) instead of O(N), and the binary codes serve simultaneously as classical hash keys and quantum state labels. No conversion needed.
- **Discovered by**: isonome-research agent (Semantic Hashing → Quantum survey)
- **Validation**: confirmed (mathematical proof — the isomorphism is trivially exact)
- **Source**: content/qml/semantic-hashing-quantum.md

### DISCOVERY: Semantic hashing can replace Mneme linear scan with O(1) hash table lookup — 2026-06-11
- **Finding**: The HierarchicalMneme.recall() method currently performs O(N·d) linear scan over all entries. With ITQ-based semantic hashing (k=32 bits), recall becomes O(Σ_{j=0}^{r} C(k,j)) = O(k^r) multi-probe hash table lookups, constant regardless of N. For N > 10⁵ entries, this is a 1000×+ speedup.
- **Impact**: Mneme recall at scale becomes practical. Currently, recall latency grows linearly with memory size; with semantic hashing, it stays constant. Critical for long-running agents with large episodic stores.
- **Discovered by**: isonome-research agent (Semantic Hashing → Quantum survey)
- **Validation**: confirmed (complexity analysis based on ITQ + multi-probe LSH theory)
- **Source**: content/qml/semantic-hashing-quantum.md

### PATTERN: Tiered classical-quantum retrieval — 2026-06-11
- **What**: Memory retrieval should use a three-tier strategy: (1) exact match → classical O(1) hash table, (2) approximate match (Hamming radius 1-3) → quantum O(√N) Grover search, (3) broad search (radius 4+) → classical linear scan with early termination. Each tier handles the regime where it's most efficient.
- **Applies to**: QSMI architecture, any hybrid classical-quantum retrieval system, Mneme recall
- **Source**: content/qml/semantic-hashing-quantum.md (proposed architecture)
- **Quality**: ★★ (theoretically motivated, not yet implemented)
- **Usage count**: 0

### PATTERN: Equilibrium-state hashing for analogical reasoning — 2026-06-11
- **What**: Encoding the 8-axis tension vector as a 16-bit binary code (2 bits per axis) enables O(1) retrieval of past equilibrium states with similar tension patterns. This creates an "analogical reasoning" capability: when tension is high, the system can instantly find past episodes with similar tension profiles and learn from how they were resolved.
- **Applies to**: EquilibriumEngine, any system with multi-dimensional state that needs experience replay
- **Source**: content/qml/semantic-hashing-quantum.md (proposed architecture)
- **Quality**: ★★ (theoretically motivated, not yet implemented)
- **Usage count**: 0

[RESEARCH] 2026-06-11 — Deep research on Semantic Hashing → Quantum (topic 1.5): surveyed semantic hashing evolution (Salakhutdinov & Hinton 2007 → ITQ → HashNet → modern contrastive hashing), quantum approximate nearest neighbor (Grover-based QANN, QRAM), and the exact isomorphism between k-bit binary codes and k-qubit basis states. Proposed Quantum-Semantic Memory Index (QSMI) with tiered classical-quantum retrieval (exact→quantum→linear). Mapped 6 isonome integration points (Mneme recall, CalibrationCache, MorphologyAnalyzer, EquilibriumEngine, DelegationGate, TensionEventLog). Key finding: binary hash codes are quantum basis states with zero conversion overhead — semantic hashing IS the classical interface to quantum memory. 8 open research questions, 5-phase roadmap. | blockers: 0 | discoveries: 2 | bank entries: 4
