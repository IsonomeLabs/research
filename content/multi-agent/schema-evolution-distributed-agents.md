# Schema Evolution in Distributed Agent Systems

**Topic:** 2.5 from Research Directions Portfolio
**Publishability:** ★★☆ (workshop/conference)
**Contribution:** ◆◆◆ (breakthrough potential)
**Date:** 2026-06-08
**Status:** Research complete — first deep dive

---

## Executive Summary

As agent networks grow from single-process prototypes to distributed fleets running for days or weeks, the schemas that define inter-agent communication — message formats, memory structures, API contracts, and capability descriptors — inevitably change. New fields are added, old fields are deprecated, enum values are renamed, and structural invariants evolve. Unlike traditional microservices, agent systems face three unique challenges: (1) agents may be offline or unreachable during a schema migration, (2) agents spawn and terminate dynamically, making coordinated rollouts impossible, and (3) memory schemas (like the isonome Mneme's episodic/semantic tiers) accumulate persistent state that must remain queryable across schema versions.

This survey examines the landscape of schema evolution for distributed agent systems, drawing from database migration theory, Protocol Buffers/Avro versioning, REST API evolution, and event sourcing patterns. The key finding: **no existing framework provides schema evolution with graceful degradation for agents that are temporarily offline or running different versions simultaneously**. We propose the **Backward-Compatible Belief Migration (BCBM)** pattern, which combines lazy migration (migrate on read, not on write), schema-aware serialization with default factories, and an equilibrium-gated migration controller that delays non-critical migrations during high-tension periods.

---

## Part 1: The Problem Space

### 1.1 Why Schema Evolution is Hard for Agents

Traditional software handles schema changes via coordinated deployment: stop all instances, migrate the database, deploy new code. Agent systems cannot do this because:

1. **No deployment coordination**: Agents in a swarm may be running different code versions. A new agent with schema v2 must communicate with an old agent running schema v1.

2. **Persistent memory across versions**: The isonome Mneme pillar stores episodic memories that persist across sessions. If the `MnemeEntry` schema changes (e.g., adding a `calibration_ece_at_store` field), old entries must still be readable.

3. **Offline agents**: In a robot fleet, some agents may be in a dead zone or powered off when a schema migration is deployed. They must recover gracefully when they come back online.

4. **Dynamic spawning**: Sub-agents created by `DelegationGate` may expect a specific schema version for the delegation request. If the parent has upgraded, the child's schema may not match.

5. **Equilibrium sensitivity**: Schema migration during high-tension periods could destabilize the agent. An agent exploring a novel environment should not also be dealing with format changes.

### 1.2 Schema Types in Agent Systems

| Schema Type | Example (isonome) | Volatility | Migration Impact |
|---|---|---|---|
| Message schemas | `Signal`, `Feedback` | High (new channels, fields) | Transient — old messages drop |
| Memory schemas | `MnemeEntry`, `DelegationRecord` | Medium (new metadata) | Persistent — old entries must remain readable |
| Capability schemas | `ActionRisk` enum, `Pillar` enum | Low (rare additions) | Cascading — enum changes break switch statements |
| State schemas | `AgentState`, `TensionSnapshot` | Medium (new axes) | Critical — serialization round-trips must survive |
| Config schemas | `EquilibriumConfig`, layer params | Low (gradual additions) | Low — defaults handle missing fields |

---

## Part 2: Existing Approaches

### 2.1 Protocol Buffers: Forward + Backward Compatibility

Google's Protocol Buffers (protobuf) enforce rules that maintain compatibility across versions:

- **New fields**: Always allowed. Old code ignores unknown fields; new code uses defaults for missing fields.
- **Removed fields**: Must be reserved (field number + name) to prevent reuse.
- **Enum changes**: New values can be added; old code treats unknown values as "unknown" sentinel.
- **Type changes**: Generally prohibited (string ↔ int32 breaks wire format).

**Applicability to agents:** The "add fields with defaults" pattern works for forward compatibility, but protobuf requires a schema registry and code generation step that doesn't fit dynamic agent spawning. The isonome framework uses Python dataclasses + dict serialization, not protobuf.

### 2.2 Apache Avro: Schema-in-the-Message

Avro embeds the writer's schema in each message, allowing the reader to resolve differences dynamically:

```json
{
  "schema": {"type": "record", "name": "Signal", "fields": [...]},
  "content": {...}
}
```

Resolution rules: missing fields get defaults, extra fields are ignored, type promotions (int → long) are allowed.

**Applicability to agents:** Schema-in-the-message is elegant but has overhead (every message carries its schema). For high-frequency control loops (50 Hz in robotics), this overhead is unacceptable. Better suited for low-frequency coordination messages.

### 2.3 REST API Evolution: Versioned Endpoints

REST APIs handle evolution through URL versioning (`/v1/agents`, `/v2/agents`) or content negotiation (`Accept: application/vnd.isonome.v2+json`).

**Applicability to agents:** URL versioning is too coarse for agent-to-agent communication. Content negotiation is more flexible but adds latency per message. The isonome MessageBus uses typed channels, not HTTP — there's no place to put version headers.

### 2.4 Event Sourcing: Append-Only with Projections

Event sourcing stores all changes as an immutable log. Schema changes are handled by creating new projections (views) over the same event stream:

```
Events: [v1_event, v1_event, v2_event, v1_event, v2_event]
                                ↑ schema change
Projection v1: reads v1 events, skips v2
Projection v2: reads both, maps v1 → v2 on the fly
```

**Applicability to agents:** The isonome `TensionEventLog` is already an append-only event log. Schema evolution could be handled by versioned projections over the event log — the log is immutable, only the reading code changes. This is the most promising approach for event-based components.

### 2.5 Database Migration: Lazy + Eager Strategies

Database migrations come in two flavors:

- **Eager (expand/contract)**: Add new column (expand) → migrate data → drop old column (contract). Requires two deployments.
- **Lazy (migrate-on-read)**: Add new column → read old data in new format on access → eventually all data is migrated.

**Applicability to agents:** Lazy migration is the right default for agent memory systems. The isonome Mneme pillar should not eagerly rewrite all stored entries when a schema changes — it should migrate individual entries when they are recalled, and batch-migrate only during idle periods (low tension).

---

## Part 3: Academic Foundations

### 3.1 Schema Evolution in Data-Intensive Systems (Kleppmann, 2017)

Martin Kleppmann's "Designing Data-Intensive Applications" (O'Reilly, 2017) codifies the principle of **evolvability**: the ability to accommodate change with minimal disruption. Key concepts:

- **Backward compatibility**: New code can read old data.
- **Forward compatibility**: Old code can read new data.
- **Full compatibility** = backward + forward — the gold standard for rolling upgrades.

For agent systems, **forward compatibility** is the harder requirement: old agents must not crash when they encounter messages from newer agents with additional fields.

### 3.2 SemVer and the Compatibility Contract

Semantic Versioning (SemVer) encodes compatibility in version numbers:

- **MAJOR**: Breaking changes (no forward compatibility)
- **MINOR**: Additive changes (forward compatible)
- **PATCH**: Bug fixes (fully compatible)

For agent systems, we need a **weaker contract**: MINOR changes are the only allowed changes within a running fleet. MAJOR changes require a fleet restart or a dual-write transition period.

### 3.3 CRDTs for Schema Coordination

Conflict-Free Replicated Data Types (CRDTs) provide a theoretical foundation for schema coordination in eventually consistent systems. A **schema CRDT** would allow each agent to register its current schema version, and the fleet would converge on the maximum version that all online agents support.

**Key insight:** CRDTs solve the "which version should we use?" problem without a central coordinator. The isonome MessageBus could use a grow-only counter (G-Counter) CRDT for schema version tracking — each agent increments its counter on upgrade, and the fleet's effective version is `min(all_counters)`.

### 3.4 Live Schema Migration in Distributed Databases

CockroachDB and TiDB support online schema changes using the "absorb" pattern:

1. Publish new schema alongside old schema (dual-write period)
2. Backfill existing data to new schema
3. Flip reads to new schema
4. Stop writing to old schema
5. Remove old schema

This 5-phase approach ensures zero downtime. For agent systems, the equivalent would be:

1. Deploy new code that writes both old and new format
2. Let old entries be lazily migrated on read
3. Once all entries are migrated (or a TTL expires), stop writing old format
4. Remove old format support

---

## Part 4: Schema Evolution Patterns for Agent Systems

### 4.1 Pattern: Default Factory Fields

**Problem:** Adding a new field to a message/memory schema breaks deserialization of old entries that lack the field.

**Solution:** Every new field must have a default factory:

```python
@dataclass
class MnemeEntry:
    content: str
    significance: float
    tier: str = "episodic"
    # v2 addition:
    calibration_ece_at_store: float | None = None  # default=None
    # v3 addition:
    rehearsal_count: int = 0  # default=0
```

**Rule:** Never add a required field without a default. This is the single most important schema evolution rule.

**Isonome-specific:** The `Feedback` dataclass, `TensionSnapshot`, `DelegationRecord`, and `MnemeEntry` should all follow this pattern. A lint rule could enforce: `no new fields without defaults on @dataclass used in serialization`.

### 4.2 Pattern: Enum Reservation

**Problem:** Adding a new enum value (e.g., `ActionRisk.CRITICAL`) causes `KeyError` in old agents that use switch-style dispatch.

**Solution:** All enum dispatch must include an `else`/default branch that handles unknown values:

```python
# BAD: will crash on unknown risk level
if risk == ActionRisk.LOW:
    ...
elif risk == ActionRisk.MODERATE:
    ...
elif risk == ActionRisk.HIGH:
    ...

# GOOD: graceful degradation
if risk == ActionRisk.LOW:
    ...
elif risk in (ActionRisk.MODERATE, ActionRisk.HIGH):
    ...
else:
    # Unknown risk → treat conservatively
    delegate_to_subagent(action)
```

**Rule:** Never use exhaustive enum dispatch without a fallback. The `ActionRisk.MEDIUM` → `MODERATE` bug (discovered in iter-019) is a direct instance of this pattern violated.

### 4.3 Pattern: Lazy Migration on Read

**Problem:** Eagerly migrating all stored data when a schema changes blocks the agent and wastes resources.

**Solution:** Migrate individual entries when they are accessed, not when the schema changes:

```python
class HierarchicalMneme:
    SCHEMA_VERSION = 3

    def recall(self, query: str, max_results: int = 5) -> list[MnemeEntry]:
        entries = self._search_tiers(query, max_results)
        # Lazy migration: upgrade entries on access
        return [self._migrate_entry(e) for e in entries]

    def _migrate_entry(self, entry: MnemeEntry) -> MnemeEntry:
        if entry.schema_version < self.SCHEMA_VERSION:
            entry = self._apply_migrations(entry)
            self._replace_entry(entry)  # write back migrated version
        return entry
```

**Tension-gated variant:** During high tension (stress_level > 0.5), skip migration and return the entry as-is. The migration can happen later when the agent is calmer.

### 4.4 Pattern: Dual-Write Transition

**Problem:** Changing a message format in a way that's not backward compatible (e.g., renaming a field, changing a type).

**Solution:** Write both old and new formats during a transition period:

```python
class Signal:
    def serialize(self) -> dict:
        # Dual-write: include both old and new field names
        d = {
            "type": self.type.value,
            "sender": self.sender.value,  # v1 field name
            "source_pillar": self.sender.value,  # v2 field name
            "payload": self.payload,
        }
        return d

    @classmethod
    def deserialize(cls, data: dict) -> "Signal":
        # Read from either field name
        sender = data.get("source_pillar") or data.get("sender")
        ...
```

**Transition lifecycle:**
1. Phase 1: New code writes both, reads from new (fallback to old)
2. Phase 2: All agents upgraded — stop writing old field
3. Phase 3: Remove old field from serialization

### 4.5 Pattern: Schema-Version-Aware MessageBus

**Problem:** The MessageBus doesn't know which schema versions each agent speaks.

**Solution:** Add schema version negotiation to the bus:

```python
class MessageBus:
    def __init__(self, maxsize: int = 100) -> None:
        self._agent_versions: dict[str, int] = {}  # agent_id → schema_version
        ...

    def register_agent(self, agent_id: str, schema_version: int) -> None:
        self._agent_versions[agent_id] = schema_version

    @property
    def fleet_schema_version(self) -> int:
        """Minimum schema version across all registered agents."""
        if not self._agent_versions:
            return 0
        return min(self._agent_versions.values())
```

Publishers serialize messages at `fleet_schema_version` (the minimum), ensuring all subscribers can read them. This is the CRDT approach — the fleet converges on the weakest common denominator.

---

## Part 5: Isonome-Specific Analysis

### 5.1 Current Schema Vulnerability Assessment

| Component | Schema Stability | Risk | Mitigation Priority |
|---|---|---|---|
| `Signal` | Low — new channels added often | Medium — old agents drop unknown channels | Low (already handled by Channel enum) |
| `Feedback` | Medium — new feedback types possible | High — old agents can't parse new feedback | Medium |
| `TensionSnapshot` | Medium — new axes added (e.g., deliberation_speed) | High — serialization breaks | High |
| `MnemeEntry` | High — memory persists across versions | Critical — old memories become unreadable | Critical |
| `DelegationRecord` | Medium — outcome tracking added in iter-020 | High — delegation audit trail breaks | High |
| `AgentState` | Medium — new pillars, new config | High — state restoration fails | High |
| `ActionRisk` enum | Low — but CRITICAL may be added | Medium — switch dispatch breaks | Medium |
| `Pillar` enum | Very Low — only 3 pillars | Low — but adding a 4th would be catastrophic | Low |
| `TensionEventLog` | Medium — new event types | Low — event log is append-only | Low |

### 5.2 The MnemeEntry Migration Problem

The `MnemeEntry` (stored in `HierarchicalMneme`) is the most critical schema evolution challenge because:

1. **Volume**: A long-running agent accumulates thousands of episodic entries
2. **Persistence**: Entries survive agent restarts (if serialized to disk)
3. **Query requirement**: Old entries must be searchable even after schema changes
4. **Cross-tier migration**: An entry may migrate from working → episodic → semantic with different schemas expected at each tier

**Proposed solution:** Add a `schema_version: int` field to `MnemeEntry` and implement lazy migration in `recall()`:

```python
@dataclass
class MnemeEntry:
    content: str
    significance: float = 0.5
    tier: str = "episodic"
    schema_version: int = 1  # NEW: tracks entry's schema version
    created_at: float = 0.0
    rehearsal_count: int = 0
    calibration_ece_at_store: float | None = None  # v2 field
```

Migration registry:

```python
MIGRATIONS = {
    (1, 2): lambda e: MnemeEntry(
        **{**e.__dict__, "calibration_ece_at_store": None, "schema_version": 2}
    ),
}
```

### 5.3 TensionSnapshot Serialization

`TensionSnapshot` is serialized in the dashboard's snapshot endpoint. Adding a new tension axis (like the proposed `deliberation_speed` from the robotics research) would break deserialization of old snapshots.

**Current state:** `TensionSnapshot` uses `to_dict()` / `from_dict()` — these must be forward-compatible.

**Proposed rule:** `from_dict()` must ignore unknown keys and provide defaults for missing keys. This is already partially implemented but should be enforced by convention and tested.

### 5.4 DelegationRecord Evolution

The `DelegationRecord` was added in iter-020 with `DelegationOutcome` tracking. If future iterations add fields (e.g., `reasoning_trace`, `estimated_duration`), old records must remain deserializable.

**Proposed solution:** Follow the Default Factory Fields pattern — all new fields on `DelegationRecord` must have defaults.

### 5.5 Equilibrium-Gated Migration Controller

**Novel contribution:** Schema migration should be tension-aware. The EquilibriumEngine's stress level determines when migrations happen:

```python
class SchemaMigrationController:
    """Tension-gated schema migration for Mneme and other persistent stores."""

    def __init__(self, engine: EquilibriumEngine):
        self._engine = engine
        self._pending_migrations: list[Migration] = []
        self._migration_count: int = 0

    def register_migration(self, migration: Migration) -> None:
        self._pending_migrations.append(migration)

    def should_migrate(self) -> bool:
        """Only migrate when the agent is in a calm state."""
        if not self._engine:
            return True  # no engine → no gate
        view = self._engine.view_for(Pillar.COGNITION)
        return view.stress_level < 0.3  # calm enough to migrate

    def run_pending_migrations(self, max_entries: int = 100) -> int:
        if not self.should_migrate():
            return 0
        migrated = 0
        for migration in self._pending_migrations:
            migrated += migration.execute(max_entries=max_entries)
        self._migration_count += migrated
        return migrated
```

**Design rationale:** Schema migration is a bookkeeping task, not a control-critical operation. During high tension (novel environment, calibration failure), the agent should focus on adaptation — not on reformatting old memories. The equilibrium engine provides a natural "when is it safe to do maintenance?" signal.
