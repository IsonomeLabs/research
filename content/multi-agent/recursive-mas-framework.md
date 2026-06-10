# Recursive Multi-Agent Systems (RecursiveMAS) Framework

**Topic:** 2.1 from Research Directions Portfolio
**Publishability:** ★★☆ (workshop/conference)
**Contribution:** ◆◆◆ (breakthrough potential)
**Date:** 2026-06-10
**Status:** Research complete — first deep dive

---

## Executive Summary

Current multi-agent frameworks treat agents as flat peers or rigid hierarchies. Neither model captures the reality of autonomous systems that must *dynamically* spawn, monitor, and terminate sub-agents based on their own calibration state, task complexity, and equilibrium tension. The isonome framework already implements a proto-recursive architecture: `DelegationGate` decides whether the parent agent should delegate high-risk actions to sub-agents; `EquilibriumEngine` provides tension-gated control that could regulate sub-agent lifecycle; and `MessageBus` provides typed inter-layer communication. But these components are not yet composed into a recursive multi-agent system (RecursiveMAS).

This survey examines the landscape of recursive agent architectures, drawing from operating system process models, actor frameworks, recursive neural networks, hierarchical reinforcement learning, and emerging agentic frameworks (AutoGen, CrewAI, OpenAI Swarm). The key finding: **no existing framework provides recursive agent spawning with equilibrium-gated lifecycle management and calibration-driven delegation in a single architecture**. We propose the **Equilibrium-Gated Recursive Agent Architecture (EGRAA)**, which composes the isonome framework's existing DelegationGate, EquilibriumEngine, TensionEventLog, and MessageBus into a recursive multi-agent system where:

1. Parent agents spawn sub-agents via DelegationGate (calibration-gated)
2. Sub-agent lifecycles are regulated by the parent's equilibrium state (tension-gated)
3. Inter-agent communication flows through the MessageBus with schema-versioned messages
4. Sub-agent outcomes feed back to the parent's calibrator (closing the metacognitive loop)
5. Recursion depth is bounded by resource limits and a trust-decay function

---

## Part 1: The Problem Space

### 1.1 Why Flat Peer Architectures Fail

Flat multi-agent systems (all agents are peers, no hierarchy) fail for three reasons:

1. **No accountability**: When all agents are peers, no agent is responsible for the outcome of a delegated task. If a sub-agent fails, the parent cannot be held accountable because there is no parent.

2. **No resource governance**: Peers compete for shared resources (compute, API calls, memory) without a governor. In the isonome framework, the EquilibriumEngine already tracks 8 tension axes — adding resource contention as a tension axis would enable equilibrium-gated resource allocation, but only if there is a parent-child relationship.

3. **No calibration inheritance**: A well-calibrated parent agent should be able to bootstrap its sub-agents' calibration. In a flat system, each agent must calibrate independently, wasting time and API calls.

### 1.2 Why Rigid Hierarchies Fail

Fixed hierarchies (manager → worker → sub-worker) fail for three different reasons:

1. **No dynamic scaling**: A rigid hierarchy cannot spawn additional workers for a burst of high-risk actions. The isonome DelegationGate is *already* dynamic — it delegates when calibration is poor and executes directly when calibration is good. A rigid hierarchy cannot match this flexibility.

2. **No self-termination**: Sub-agents in a rigid hierarchy persist until the manager explicitly terminates them. In a recursive architecture, sub-agents should self-terminate when their task is complete or when the parent's equilibrium tension drops (indicating the crisis that prompted spawning has resolved).

3. **No cross-level feedback**: In a rigid hierarchy, feedback flows only upward (worker → manager). A recursive architecture needs bidirectional feedback: the parent's calibrator learns from sub-agent outcomes (already implemented via `DelegationOutcome`), and the sub-agent's equilibrium state influences the parent's tension (not yet implemented).

### 1.3 The Recursive Agent Thesis

A recursive multi-agent system is one where agents can contain other agents. The key design principle:

> **Control loops within control loops.**

Each agent runs its own Soma → JEPA → Cortex → Reflex pipeline (see `isonome/core/agent.py`). A recursive agent contains a *pool* of sub-agents, each running their own pipeline. The parent's Cortex layer decides *when* to delegate; the parent's Praxis pillar decides *what* to delegate; the parent's EquilibriumEngine decides *whether the system is calm enough* to accept sub-agent feedback without destabilization.

The recursion terminates when:
- The delegation gate is WELL_CALIBRATED (no need for sub-agents)
- The maximum recursion depth is reached (resource bound)
- The parent's stress level exceeds a threshold (system is too overwhelmed to manage children)

---

## Part 2: Existing Approaches

### 2.1 Operating System Process Models

The most mature recursive execution model is the OS process tree:

```
init (PID 1)
 ├── sshd (PID 42)
 │   └── bash (PID 100)
 │       └── python agent.py (PID 200)
 └── cron (PID 55)
     └── update_agent.sh (PID 300)
```

**Key properties:**
- **Parent-child relationship**: Parent spawns child; child inherits parent's environment (uid, cwd, file descriptors)
- **Signal propagation**: Parent can send SIGTERM to child; child's exit status propagates to parent via `wait()`
- **Resource limits**: `setrlimit()` bounds CPU, memory, and file descriptors per process
- **Orphan adoption**: If parent dies, child is reparented to init

**Applicability to RecursiveMAS:**
- ✅ Parent-child spawning model → `DelegationGate` decides when to spawn
- ✅ Signal propagation → sub-agent outcomes propagate via `record_outcome()`
- ✅ Resource limits → `EquilibriumEngine` stress level bounds sub-agent count
- ❌ Orphan adoption → agents don't have an "init" process; if the parent crashes, sub-agents must self-terminate (safety requirement for robotics)
- ❌ Shared memory → OS processes share memory via mmap; agents share state via `MessageBus` publish/subscribe

### 2.2 Actor Model (Hewitt, 1973; Erlang/OTP)

The actor model is the theoretical foundation for concurrent recursive systems:

```
Actor A
 ├── spawn Actor B
 │   └── spawn Actor C
 └── send(message, Actor B)
```

**Key properties:**
- **Encapsulation**: Each actor has private state; communication only via async messages
- **Location transparency**: Actors don't know if their peers are local or remote
- **Supervision trees** (Erlang/OTP): If a child actor crashes, the supervisor restarts it with a configurable strategy (one-for-one, one-for-all, rest-for-one)

**Applicability to RecursiveMAS:**
- ✅ Message-passing communication → `MessageBus` with typed `Channel` enum
- ✅ Supervision trees → parent agent monitors sub-agent health; can restart or terminate
- ✅ Location transparency → agents could be remote (robot fleet) or local (same process)
- ❌ No calibration concept → actors don't have a metacognitive model; the isonome `ConfidenceCalibrator` adds a layer not present in actor systems
- ❌ No equilibrium gating → actors spawn based on programmer logic, not system state

### 2.3 Hierarchical Reinforcement Learning (HRL)

HRL decomposes tasks into temporal abstractions:

```
Policy π(high)
 ├── option o₁ (go to kitchen)
 │   ├── option o₁.₁ (open door)
 │   └── option o₁.₂ (walk to counter)
 └── option o₂ (pick up cup)
```

**Key properties:**
- **Temporal abstraction**: Options span multiple time steps
- **Goal-conditioned policies**: Each sub-policy is conditioned on a sub-goal
- **Intrinsic motivation**: Sub-policies learn from their own reward signals

**Applicability to RecursiveMAS:**
- ✅ Hierarchical task decomposition → DelegationGate decides which actions to delegate
- ✅ Goal-conditioned delegation → each `DelegationRecord` includes `action_description` as the sub-goal
- ✅ Intrinsic feedback → `DelegationOutcome` feeds back to calibrator
- ❌ Fixed option set → HRL typically pre-defines options; RecursiveMAS needs dynamic sub-agent creation
- ❌ No resource management → HRL doesn't bound the number of concurrent sub-policies

### 2.4 Agentic Frameworks (AutoGen, CrewAI, OpenAI Swarm)

#### AutoGen (Microsoft, 2023)

Multi-agent conversation framework where agents chat in rounds:

```python
agent_a = AssistantAgent("A", llm_config=...)
agent_b = AssistantAgent("B", llm_config=...)
groupchat = GroupChat(agents=[agent_a, agent_b])
manager = GroupChatManager(groupchat)
manager.initiate_chat(agent_a, message="Solve this problem")
```

**Limitations for RecursiveMAS:**
- Flat peer model (no true parent-child)
- No calibration or metacognitive awareness
- Conversation-based (not suitable for real-time robotics at 50 Hz)

#### CrewAI (2023)

Role-based agent crews with sequential/hierarchical processes:

```python
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.hierarchical,
    manager_llm="gpt-4",
)
```

**Limitations for RecursiveMAS:**
- Rigid hierarchy (cannot dynamically spawn/terminate)
- No equilibrium or tension concept
- Synchronous execution model (not real-time)

#### OpenAI Swarm (2024)

Lightweight multi-agent orchestration with handoffs:

```python
def transfer_to_agent_b():
    return agent_b

agent_a = Agent(name="A", functions=[transfer_to_agent_b])
```

**Limitations for RecursiveMAS:**
- Handoff-based (agents transfer control, not spawn sub-agents)
- No feedback loop from sub-agent to parent
- No resource governance or equilibrium awareness

### 2.5 Recursive Neural Networks and Tree-Structured Models

Recursive Neural Networks (Socher et al., 2011) process tree-structured inputs by applying the same neural function recursively at each node. The key insight: **the same computational primitive is applied at every level of the hierarchy**.

**Applicability to RecursiveMAS:**
- ✅ Same primitive at every level → each agent runs the same Soma→JEPA→Cortex→Reflex pipeline
- ✅ Variable-depth computation → recursion depth adapts to task complexity
- ❌ Fixed topology → recursive NNs operate on known tree structures; RecursiveMAS must handle dynamic topologies where agents spawn and terminate at runtime
- ❌ No feedback from leaves to root during computation → recursive NNs pass gradients after forward pass; RecursiveMAS needs real-time bidirectional feedback

---

## Part 3: The Equilibrium-Gated Recursive Agent Architecture (EGRAA)

### 3.1 Architecture Overview

```
┌─────────────────────────────────────────────────┐
│ Parent Agent                                      │
│                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │ Cognition │  │  Praxis  │  │    Mneme     │   │
│  │  Pillar   │  │  Pillar  │  │    Pillar    │   │
│  └─────┬─────┘  └─────┬────┘  └──────┬──────┘   │
│        │              │               │           │
│  ┌─────▼──────────────▼───────────────▼──────┐   │
│  │         EquilibriumEngine                  │   │
│  │  ┌─────────────┐  ┌────────────────────┐  │   │
│  │  │ DelegationGate│  │ TensionEventLog   │  │   │
│  │  └──────┬───────┘  └────────────────────┘  │   │
│  └─────────┼──────────────────────────────────┘   │
│            │ spawn                                  │
│  ┌─────────▼──────────────────────────────────┐   │
│  │         Sub-Agent Pool                      │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐ │   │
│  │  │ SubAgent1│  │ SubAgent2│  │ SubAgent3│ │   │
│  │  │ (C,P,M) │  │ (C,P,M) │  │ (C,P,M) │ │   │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘ │   │
│  └───────┼──────────────┼──────────────┼──────┘   │
│          │ outcome      │ outcome      │          │
│  ┌───────▼──────────────▼──────────────▼──────┐   │
│  │       Sub-Agent Feedback Aggregator         │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

### 3.2 Component Mapping to Existing Isonome Code

| EGRAA Component | Isonome Implementation | Status |
|---|---|---|
| Delegation decision | `DelegationGate.check()` | ✅ Implemented (iter-019) |
| Delegation outcome feedback | `DelegationGate.record_outcome()` | ✅ Implemented (iter-020) |
| Calibration-driven spawning | `DelegationMode.OVERCONFIDENT/UNDERCONFIDENT` | ✅ Implemented |
| Tension-gated lifecycle | `EquilibriumEngine.view_for(pillar).stress_level` | ⚠️ Needs extension |
| Sub-agent pool management | `PraxisPillar` manages action batch | ❌ Not yet a pool |
| Inter-agent messaging | `MessageBus` with `Channel` enum | ✅ Implemented (needs sub-agent channels) |
| Cross-agent calibration inheritance | `ConfidenceCalibrator` is per-agent | ❌ Not yet implemented |
| Recursion depth bounding | No depth tracker exists | ❌ Not yet implemented |
| Resource governance via equilibrium | `explore_exploit` tension axis partially models this | ❌ Needs explicit resource axis |

### 3.3 Sub-Agent Lifecycle

A sub-agent goes through these phases:

1. **SPAWNED**: Parent's DelegationGate decides to delegate; sub-agent is created with a scoped task and calibration bootstrap from parent.

2. **BOOTING**: Sub-agent initializes its own Soma→JEPA→Cortex→Reflex pipeline. The parent's `Agent.boot()` runs this phase.

3. **RUNNING**: Sub-agent executes its delegated task. Parent monitors via heartbeat signals on the MessageBus.

4. **REPORTING**: Sub-agent completes its task and produces a `DelegationOutcome`. This flows back to the parent's `DelegationGate.record_outcome()`.

5. **TERMINATED**: Sub-agent is cleaned up. Its equilibrium state summary is archived in the parent's Mneme for future pattern matching.

**Tension-gated termination**: If the parent's stress level exceeds a threshold (default 0.6), the parent can terminate all non-critical sub-agents to reduce resource pressure. This is analogous to the OOM killer in Linux — sacrifice non-essential work to preserve the core.

### 3.4 Calibration Inheritance

When a parent spawns a sub-agent, the sub-agent's ConfidenceCalibrator should be bootstrapped with the parent's calibration data, not started from scratch:

```python
class SubAgentBootstrapper:
    """Bootstrap a sub-agent's calibrator from parent data."""

    def __init__(self, parent_calibrator: ConfidenceCalibrator):
        self._parent = parent_calibrator

    def bootstrap(self, child_calibrator: ConfidenceCalibrator) -> None:
        """Transfer parent's calibration history to child.

        Strategy: copy the parent's last N prediction-outcome pairs
        with a decay factor. The child inherits the parent's
        calibration distribution but with reduced confidence
        (the parent's model was trained on different actions).

        Decay factor: 0.5 — the child "half-trusts" the parent's
        calibration because the action distributions may differ.
        """
        # Get parent's recent predictions
        parent_predictions = self._parent.recent_predictions(n=20)
        for pred in parent_predictions:
            child_calibrator.record(
                confidence=pred.confidence * 0.5,  # Decay
                correct=pred.correct,
            )
```

**Why this matters**: Without calibration inheritance, a sub-agent starts in `UNCALIBRATED` mode (DelegationMode). It takes `min_predictions=10` successful predictions before the sub-agent can make calibrated delegation decisions of its own. With inheritance, the sub-agent starts closer to `MODERATE` mode, enabling faster recursive delegation (the sub-agent can spawn its own sub-agents sooner).

### 3.5 Recursion Depth Bounding

Unbounded recursion is dangerous in agent systems — a poorly-calibrated agent might delegate to a sub-agent that delegates to another sub-agent, consuming resources without limit.

**Proposed bounding mechanism:**

```python
class RecursionBudget:
    """Bounds recursion depth and total sub-agent count."""

    def __init__(
        self,
        max_depth: int = 3,
        max_sub_agents: int = 5,
        trust_decay: float = 0.7,
    ):
        self._max_depth = max_depth
        self._max_sub_agents = max_sub_agents
        self._trust_decay = trust_decay  # Per level

    def can_spawn(self, current_depth: int, active_children: int) -> bool:
        if current_depth >= self._max_depth:
            return False
        if active_children >= self._max_sub_agents:
            return False
        return True

    def child_budget(self, parent_budget: "RecursionBudget") -> "RecursionBudget":
        """Derive a child's budget from the parent's.

        Each level gets a reduced budget:
        - max_depth decreases by 1
        - max_sub_agents *= trust_decay
        """
        return RecursionBudget(
            max_depth=parent_budget._max_depth - 1,
            max_sub_agents=max(
                1,
                int(parent_budget._max_sub_agents * self._trust_decay),
            ),
            trust_decay=parent_budget._trust_decay,
        )
```

**Trust decay**: Each level of recursion reduces the maximum sub-agent count by 30% (default `trust_decay=0.7`). This means:
- Level 0 (parent): up to 5 sub-agents
- Level 1 (child): up to 3 sub-agents (5 × 0.7 = 3.5 → 3)
- Level 2 (grandchild): up to 2 sub-agents (3 × 0.7 = 2.1 → 2)
- Level 3: cannot spawn (max_depth=3 reached)

Total theoretical agents in tree: 1 + 5 + 15 + 30 = 51 (bounded).

### 3.6 Equilibrium-Gated Sub-Agent Management

The parent's EquilibriumEngine controls sub-agent lifecycle through three mechanisms:

#### 3.6.1 Stress-Gated Spawning

The parent should not spawn sub-agents during high stress (it's overwhelmed):

```python
def can_spawn_sub_agent(self) -> bool:
    """Only spawn when calm enough to manage children."""
    view = self._engine.view_for(Pillar.PRAXIS)
    if view.stress_level > 0.6:
        return False  # Too stressed to manage children
    if view.is_highly_stressed:
        return False
    return self._delegation_gate.compute_mode() in (
        DelegationMode.OVERCONFIDENT,
        DelegationMode.UNDERCONFIDENT,
    )
```

#### 3.6.2 Tension-Gated Feedback Processing

When the parent is in a high-tension state, sub-agent outcomes should be queued, not immediately processed. Processing outcomes during high tension could destabilize the equilibrium:

```python
class SubAgentFeedbackAggregator:
    """Buffers sub-agent outcomes and processes them when calm."""

    def __init__(self, engine: EquilibriumEngine, gate: DelegationGate):
        self._engine = engine
        self._gate = gate
        self._pending: deque[DelegationOutcome] = deque(maxlen=200)

    def receive_outcome(self, outcome: DelegationOutcome) -> None:
        self._pending.append(outcome)

    def process_pending(self) -> int:
        """Process pending outcomes when system is calm enough.

        Only processes when stress_level < 0.3 (same threshold
        as the SchemaMigrationController — maintenance tasks
        happen during calm periods).

        Returns:
            Number of outcomes processed.
        """
        view = self._engine.view_for(Pillar.PRAXIS)
        if view.stress_level > 0.3:
            return 0

        processed = 0
        while self._pending:
            outcome = self._pending.popleft()
            self._gate.record_outcome(outcome)
            processed += 1
            # Re-check stress after each outcome (could shift)
            if self._engine.view_for(Pillar.PRAXIS).stress_level > 0.3:
                break
        return processed
```

#### 3.6.3 Resource Tension Axis

We propose adding a new tension axis to the EquilibriumEngine specifically for sub-agent resource management:

| Axis | Default | Owner | Description |
|---|---|---|---|
| `sub_agent_pressure` | 0.0 | Praxis | Fraction of max sub-agents currently active. 0.0 = none; 1.0 = all slots full. |

When `sub_agent_pressure` is high:
- The parent should not spawn new sub-agents (even if DelegationGate says DELEGATE)
- Existing sub-agents with low-priority tasks should be terminated
- The Praxis pillar should emit feedback pulling this axis back toward 0.0

This creates a **natural backpressure mechanism**: spawning sub-agents increases pressure; completing tasks reduces it. The equilibrium engine self-regulates the sub-agent population.

### 3.7 Inter-Agent Communication

#### 3.7.1 Channel Extension for Sub-Agents

The current `Channel` enum supports inter-layer communication within a single agent:

```python
class Channel(str, Enum):
    SENSORS = "sensors"
    REFLEX_OUTPUT = "reflex_output"
    JEPA_ADJUSTMENT = "jepa_adjustment"
    CORTEX_ADVICE = "cortex_advice"
    PLASTICITY_PATCHES = "plasticity_patches"
    ERROR = "error"
```

For RecursiveMAS, we need additional channels:

```python
class Channel(str, Enum):
    # Existing channels (intra-agent)
    SENSORS = "sensors"
    REFLEX_OUTPUT = "reflex_output"
    JEPA_ADJUSTMENT = "jepa_adjustment"
    CORTEX_ADVICE = "cortex_advice"
    PLASTICITY_PATCHES = "plasticity_patches"
    ERROR = "error"

    # New channels (inter-agent)
    DELEGATION_REQUEST = "delegation_request"      # Parent → Child: task assignment
    DELEGATION_OUTCOME = "delegation_outcome"       # Child → Parent: task result
    SUB_AGENT_HEARTBEAT = "sub_agent_heartbeat"     # Child → Parent: alive signal
    SUB_AGENT_TERMINATION = "sub_agent_termination"  # Parent → Child: shutdown signal
    CALIBRATION_SYNC = "calibration_sync"           # Parent ↔ Child: calibration bootstrap
```

**Design rationale**: Adding inter-agent channels to the existing `Channel` enum (rather than creating a separate bus) keeps the communication model unified. The schema evolution patterns from our earlier research (topic 2.5) apply: new channel types should be added with backward-compatible defaults, and unknown channels should be gracefully ignored by old agents.

#### 3.7.2 Message Schema for Delegation

Delegation messages must carry enough context for the sub-agent to act independently:

```python
@dataclass
class DelegationMessage:
    """Message sent from parent to sub-agent via DELEGATION_REQUEST channel."""
    task_id: UUID
    action_description: str
    action_risk: int
    calibration_snapshot: dict  # Parent's ECE, bias, mode
    equilibrium_snapshot: dict  # Parent's tension state at delegation time
    recursion_depth: int        # Current depth in the agent tree
    resource_budget: dict       # Derived RecursionBudget for child
    deadline_tick: int          # Parent expects result by this tick
    schema_version: int = 1    # For schema evolution (see topic 2.5)
```

The `schema_version` field follows the **Default Factory Fields** pattern from our schema evolution research: old sub-agents that don't know about new fields will use defaults.

---

## Part 4: Cross-Pillar Integration with Sub-Agents

### 4.1 Cognition → Sub-Agent: Task Decomposition

The Cognition pillar's `RecursiveReasoningEngine` should decompose complex actions into sub-tasks suitable for delegation:

```
Parent Cognition:
  "Deploy robot fleet to building B"
  → Sub-task 1: "Navigate to building B entrance" (MODERATE risk)
  → Sub-task 2: "Scan for obstacles" (LOW risk)
  → Sub-task 3: "Enter building B" (HIGH risk)
```

Sub-tasks 1 and 2 are executed directly (LOW/MODERATE risk + well-calibrated). Sub-task 3 is delegated (HIGH risk + overconfident calibration).

### 4.2 Praxis → Sub-Agent: Action Dispatch

The Praxis pillar's `DelegationGate` already dispatches actions based on risk and calibration. The extension is to dispatch to *real sub-agents* instead of marking them as `DELEGATE` and leaving them in limbo:

**Current flow:**
```
PraxisPillar.execute_batch(actions)
  → gate.check(action) → DELEGATE
  → action is marked BLOCKED
  → (nothing happens — no actual sub-agent exists)
```

**Proposed flow:**
```
PraxisPillar.execute_batch(actions)
  → gate.check(action) → DELEGATE
  → SubAgentPool.spawn(action, budget=recursion_budget.child_budget())
  → sub-agent runs action
  → SubAgentFeedbackAggregator receives outcome
  → gate.record_outcome(outcome) → calibrator updates
```

### 4.3 Mneme → Sub-Agent: Memory Inheritance

Sub-agents should inherit relevant memories from their parent's Mneme:

```python
class MemoryInheritance:
    """Selects parent memories relevant to a delegated task."""

    def select_for_delegation(
        self,
        mneme: HierarchicalMneme,
        action_description: str,
        max_entries: int = 5,
    ) -> list[MnemeEntry]:
        """Select the most relevant episodic memories for the sub-agent.

        Strategy:
        1. Recall memories matching action description keywords
        2. Filter for high-significance entries (sig > 0.7)
        3. Prefer recently-stored entries (recency bias)
        4. Limit to max_entries to prevent information overload
        """
        candidates = mneme.recall(action_description, max_results=20)
        # Filter and sort
        relevant = [
            e for e in candidates
            if e.significance > 0.7
        ]
        relevant.sort(key=lambda e: e.created_at, reverse=True)
        return relevant[:max_entries]
```

### 4.4 Sub-Agent → Parent: Equilibrium Feedback

Sub-agents' equilibrium states should propagate upward as a summary signal:

```python
@dataclass
class SubAgentEquilibriumSummary:
    """Summary of a sub-agent's equilibrium state for parent reporting."""
    sub_agent_id: UUID
    stress_level: float        # 0.0-1.0
    dominant_axis: str         # Most-stressed tension axis
    mode: str                  # DelegationMode name
    active_ticks: int          # How long this sub-agent has been running
    calibration_ece: float     # Sub-agent's own calibration quality
```

When multiple sub-agents report high stress simultaneously, the parent's `sub_agent_pressure` axis should reflect this — creating a cascading backpressure effect.

---

## Part 5: Convergence Properties

### 5.1 Why Recursive Delegation Converges

The key property that ensures convergence: **each level of recursion reduces the action space**.

- Level 0: Parent handles all actions (full action space)
- Level 1: Sub-agents handle delegated high-risk actions (subset of actions)
- Level 2: Sub-sub-agents handle the highest-risk subset of the subset

At each level, the delegated actions are *strictly harder* (higher risk) but *strictly fewer*. The recursion terminates when:
1. The action space is exhausted (no more actions to delegate)
2. The recursion budget is exhausted (max depth reached)
3. The sub-agent is well-calibrated (DelegationMode → WELL_CALIBRATED, no more delegation needed)

This is analogous to **quicksort recursion**: each level partitions into smaller subproblems, and the recursion terminates when the subproblem size is 1.

### 5.2 Equilibrium Stability with Sub-Agents

Adding sub-agents introduces a new source of tension: the parent must manage its own state AND its children's states. We analyze stability using the existing equilibrium framework:

**Without sub-agents**: The engine has 8 axes. Stress is RMS drift across all 8.

**With sub-agents**: We add `sub_agent_pressure` as a 9th axis. The stress computation now includes sub-agent resource pressure:

```
stress = sqrt((Σ drift²) / 9)  # was / 8
```

**Stability guarantee**: If the parent's `sub_agent_pressure` is regulated by the EquilibriumEngine (via adaptive damping, velocity tracking, etc.), then the parent's stress level will converge even in the presence of sub-agents. The sub-agents' outcomes feed back as calibration signals, which gradually improve the parent's delegation decisions, which reduces unnecessary spawning, which reduces `sub_agent_pressure`.

**Potential instability**: If sub-agent outcomes are processed during high tension, they can cause oscillation (stress → spawn → outcome → more stress). The `SubAgentFeedbackAggregator` (Section 3.6.2) prevents this by buffering outcomes during high tension and processing them only when calm.

### 5.3 Deadlock Prevention

Recursive agent systems can deadlock: Agent A delegates to Agent B, which delegates back to Agent A (circular dependency).

**Prevention mechanism**: The `RecursionBudget.max_depth` counter is *monotonically increasing* along any delegation chain. Since each delegation increments the depth, and the depth is bounded, circular delegation is impossible — Agent B at depth 1 cannot delegate back to Agent A at depth 0 because that would require decreasing the depth counter, which the system never does.

Additionally, each `DelegationMessage` carries a `recursion_depth` field that the recipient's `DelegationGate` checks:

```python
def check(self, action, *, risk_value=None, recursion_depth=0) -> DelegationDecision:
    if recursion_depth >= self._max_recursion_depth:
        return DelegationDecision.EXECUTE  # Cannot delegate further
    # ... normal delegation logic
```

---

## Part 6: Implementation Roadmap

### Phase 1: Sub-Agent Pool (2-3 days)
- Implement `SubAgentPool` class in `isonome/praxis/`
- Wire pool to `DelegationGate` — DELEGATE decisions spawn sub-agents
- Add `DELEGATION_REQUEST` and `DELEGATION_OUTCOME` channels to `MessageBus`
- Tests: sub-agent spawning, outcome recording, pool capacity

### Phase 2: Equilibrium-Gated Lifecycle (1-2 days)
- Add `sub_agent_pressure` tension axis to `EquilibriumEngine`
- Implement `SubAgentFeedbackAggregator` with tension-gated processing
- Implement stress-gated spawning (Section 3.6.1)
- Tests: spawning under stress, feedback buffering, pressure axis

### Phase 3: Recursion Budget (1 day)
- Implement `RecursionBudget` with trust decay
- Add `recursion_depth` to `DelegationGate.check()`
- Implement `DelegationMessage` with schema versioning
- Tests: depth bounding, budget derivation, deadlock prevention

### Phase 4: Calibration Inheritance (1-2 days)
- Implement `SubAgentBootstrapper` with decayed parent calibration
- Implement `MemoryInheritance` for sub-agent memory bootstrapping
- Implement `SubAgentEquilibriumSummary` for upward feedback
- Tests: calibration bootstrap, memory selection, equilibrium propagation

### Phase 5: Cross-Pillar Integration (2-3 days)
- Wire Cognition → Praxis task decomposition for delegation
- Wire sub-agent outcomes → Mneme storage with "delegated" tag
- Wire sub-agent equilibrium summaries → parent's tension axes
- Full integration tests: end-to-end recursive delegation pipeline

**Total estimated effort: 7-11 days for core implementation.**

---

## Part 7: Open Research Questions

1. **Calibration decay rate for inheritance**: We propose 0.5 decay for bootstrapping child calibrators from parent data. What is the optimal decay rate? Too high → child overtrusts parent; too low → child starts nearly uncalibrated. Needs empirical validation.

2. **Optimal recursion depth**: We propose max_depth=3 with trust_decay=0.7. What is the right depth for different task complexities? Simple tasks may only need depth 1; complex multi-robot coordination may need depth 5.

3. **Sub-agent termination criteria**: When should the parent terminate a sub-agent? Options: task complete, timeout exceeded, parent stress too high, sub-agent stress too high. The right termination policy depends on the cost of interrupted vs. wasted work.

4. **Cross-level equilibrium coupling**: Should the parent's `sub_agent_pressure` axis be coupled to the child's stress level? If so, what coupling function? Linear? Exponential? This affects stability properties.

5. **Sub-agent memory persistence**: Should sub-agent memories persist after termination? If so, where? In the parent's Mneme? In a separate shared memory? This affects the framework's memory architecture.

6. **Competing delegation**: When multiple parents want to delegate to the same sub-agent pool, how should conflicts be resolved? This is the multi-agent resource allocation problem, which the equilibrium engine is uniquely positioned to solve.

---

## Part 8: Relationship to Other Isonome Research

| Research Topic | Connection to RecursiveMAS |
|---|---|
| 1.3 1-Bit Transformers for QML | Sub-agents could use 1-bit quantized VLA backends for low-latency inference |
| 1.5 Semantic Hashing → Quantum | Sub-agent memory lookup could use semantic hashing for fast retrieval |
| 2.2 Quadcameral Architecture | Each sub-agent has its own 4-layer cognition (perception, deliberation, action, reflection) |
| 2.5 Schema Evolution | DelegationMessage uses schema_version; sub-agent communication must be evolution-safe |
| 2.7 Graph Representations | Agent trees can be modeled as directed graphs for resilience analysis |
| 3.3 Kernel Auto-Calibration | Sub-agents need their own SomaKernels; parent's kernel could bootstrap child's |
| Predictive-SSM (from 2.8/2.9) | Sub-agents using SSM temporal backbones could run faster per-tick than attention-based agents |

---

## References

1. Hewitt, C., Bishop, P., & Steiger, R. (1973). "A Universal Modular ACTOR Formalism for Artificial Intelligence." IJCAI.
2. Socher, R., et al. (2011). "Parsing Natural Scenes and Natural Language with Recursive Neural Networks." ICML.
3. Sutton, R. S., Precup, D., & Singh, S. (1999). "Between MDPs and Semi-MDPs: A Framework for Temporal Abstraction in Reinforcement Learning." Artificial Intelligence.
4. Kleppmann, M. (2017). "Designing Data-Intensive Applications." O'Reilly.
5. Wu, Q., et al. (2023). "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation." Microsoft Research.
6. OpenAI (2024). "Swarm: Educational framework exploring ergonomic, lightweight multi-agent orchestration."
7. Isonome Framework, iteration-019: Calibration-Gated Delegation (DelegationGate).
8. Isonome Framework, iteration-020: Delegation Outcome Tracking (DelegationOutcome + record_outcome).
9. Isonome Framework, iteration-021: Tension Velocity Tracking (TensionVelocityTracker).
10. Isonome Framework, iteration-028: Velocity-Aware Adaptive Damping (preemptive oscillation detection).
11. Isonome Framework, iteration-032: Event Log Analysis API (cross-pillar conflict detection).
12. Isonome Research, topic 2.5: Schema Evolution in Distributed Agent Systems (BCBM pattern).
