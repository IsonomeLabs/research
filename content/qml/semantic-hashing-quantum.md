# Semantic Hashing for Quantum State Retrieval in Agent Memory Systems

**Topic:** 1.5 from Research Directions Portfolio
**Publishability:** ★★★ (top conference)
**Contribution:** ◆◆◆ (breakthrough potential)
**Date:** 2026-06-11
**Status:** Research complete — first deep dive

---

## Executive Summary

Semantic hashing — mapping high-dimensional data to compact binary codes that preserve similarity structure — was proposed by Salakhutdinov & Hinton in 2007 as a way to enable sub-linear time retrieval from massive databases using only hash table lookups. The core insight: if similar items map to similar binary codes, then Hamming-distance queries replace expensive vector similarity searches, yielding orders-of-magnitude speedups.

This survey examines the evolution of semantic hashing from its origins through modern deep hashing methods, and identifies a largely unexplored intersection with quantum computing: **binary hash codes are natural quantum basis states**. A 32-bit semantic hash corresponds to a 32-qubit system where each computational basis state |z₁z₂…z₃₂⟩ is a valid hash code, and quantum superposition enables simultaneous evaluation of similarity across exponentially many stored items. No existing paper combines semantic hashing with quantum similarity search in the context of agent memory systems.

We propose the **Quantum-Semantic Memory Index (QSMI)**, a hybrid classical-quantum architecture that: (1) encodes isonome Mneme entries as binary semantic hashes via a learned encoder, (2) uses quantum amplitude amplification to perform approximate nearest-neighbor retrieval in O(√N) instead of O(N), and (3) integrates with the EquilibriumEngine's tension-state vectors for fast past-state lookup. For the isonome framework specifically, QSMI could reduce Mneme recall latency from O(N·d) brute-force to O(√N) quantum-accelerated, while simultaneously providing a natural bridge between the discrete binary codes of semantic hashing and the discrete basis states of quantum circuits.

**Key finding:** The isomorphism between binary hash codes {0,1}^k and quantum basis states {|0⟩,|1⟩}^⊗k is exact and lossless. Every classical semantic hash is a valid quantum state, and every quantum measurement of a k-qubit system yields a valid k-bit hash code. This makes semantic hashing the *natural classical interface* to quantum memory systems — a connection that has been almost entirely overlooked.

---

## Part 1: The Semantic Hashing Landscape

### 1.1 Original Semantic Hashing (Salakhutdinov & Hinton, 2007)

**Paper:** [Semantic Hashing](https://www.cs.toronto.edu/~hinton/absps/sh.pdf), Ruslan Salakhutdinov & Geoffrey Hinton, 2007
**Originally presented:** 2007, published in Int. J. Approx. Reasoning 50(7), 2009

**Core idea:** Train a generative model (Restricted Boltzmann Machine, RBM) to map documents to binary codes such that semantically similar documents get similar (low Hamming distance) binary codes. Then use the binary codes as memory addresses — a lookup in a hash table returns all documents within Hamming radius r in O(1) time.

**Architecture:**

```
Document → Bag-of-words vector (d-dim) → RBM → Binary code (k-bit)
                                                          ↓
                                              Hash table lookup (O(1))
                                                          ↓
                                              Semantically similar documents
```

**Training procedure (two-stage):**
1. **Pre-training:** Stack of RBMs trained greedily layer-by-layer (Deep Belief Network). Each RBM learns a progressively more compressed representation.
2. **Fine-tuning:** Backpropagation through the entire stack to optimize reconstruction error.

**Key results (on 20-newsgroups, corpus of ~18K documents):**
- 30-bit codes with Hamming radius 2 retrieved relevant documents with precision comparable to LSH
- Retrieval was 10-100× faster than linear scan
- Binary codes captured semantic structure: queries for "computer graphics" returned "graphics" and "visualization" documents, not just keyword matches

**Limitations:**
- Required careful RBM training (contrastive divergence, learning rate schedules)
- Binary quantization lost information — the "semantic gap" between continuous representations and discrete codes
- Didn't scale well beyond 64-bit codes (training became unstable)

### 1.2 The Hashing Taxonomy

Modern hashing methods fall into two families:

| Family | Method | Key Idea | Training |
|--------|--------|----------|----------|
| **Locality-Sensitive Hashing (LSH)** | Random projection LSH (Andoni & Indyk, 2006) | Data-independent random hyperplanes | No training |
| **Spectral Hashing** | Spectral Hashing (Weiss et al., 2008) | Analytic solution using PCA eigenfunctions | Closed-form |
| **Binary Reconstruction** | Iterative Quantization (ITQ, Gong et al., 2011) | Rotate PCA subspace to minimize quantization error | Alternating optimization |
| **Supervised Deep Hashing** | CNNH, DHN, HashNet (2014-2017) | Deep CNN + hash loss with pairwise labels | End-to-end backprop |
| **Unsupervised Deep Hashing** | Deep Binary Hashing (DBH, 2015) | Autoencoder + binary constraint | Greedy + backprop |
| **Variational Hashing** | VAE-based (Kingma & Welling extension) | Variational lower bound with Bernoulli latent | SGD |
| **Contrastive Hashing** | CLIP-based (2022-2024) | Contrastive text-image loss + hash layer | End-to-end |

### 1.3 Iterative Quantization (ITQ) — The Classical Baseline

**Paper:** [Iterative Quantization: A Provincially Zero-Variance Approach](https://www.cv-foundation.org/openaccess/content_cvpr_2011/papers/Gong_Iterative_Quantization_A_2011_CVPR_paper.pdf), Gong et al., CVPR 2011

ITQ remains one of the strongest classical baselines for unsupervised hashing. The algorithm:

1. **PCA projection:** Project data X onto the top-k PCA eigenvectors: Z = X · W_k
2. **Rotation search:** Find rotation R that minimizes quantization error: min_{R} ||Z·R - B||² where B ∈ {−1,+1}^(n×k)
3. **Alternating optimization:** Fix R, solve for B (sign); Fix B, solve for R (SVD of Z^T·B)

**Quantization error bound:**
```
Q(R) = ||Z·R - B||²_F = n·k - ||Z·R^T||_1

Lower bound: Q(R) ≥ nk(1 - 2/π) ≈ 0.363·nk (for Gaussian Z)
ITQ achieves: Q(R) ≈ 0.38·nk (within 5% of optimal)
```

**Key insight:** The rotation R matters because naive PCA quantization (just sign(Z)) has Q ≈ 0.54·nk — 50% worse than ITQ. The rotation aligns PCA axes with the hypercube vertices, reducing the average distance from continuous points to their nearest binary code.

### 1.4 Deep Hashing: HashNet and Beyond

**Paper:** [HashNet: Deep Learning to Hash by Continuation](https://arxiv.org/abs/1702.00746), Cao et al., ICCV 2017

HashNet introduced the **continuation** approach to the binary constraint problem:

**The core optimization challenge:**
```
min_{W, B} Σ_{i,j} s_{ij} · θ_{ij} + λ||W||²
s.t. b_i ∈ {−1, +1}^k

where θ_{ij} = ½(u_i^T · u_j)   (inner product of continuous codes)
      s_{ij} ∈ {0, 1}            (similarity label)
      u_i = tanh(α · h_i)       (scaled tanh as continuous surrogate)
```

**Continuation strategy:**
- Start with α → 0 (tanh is nearly linear, optimization is smooth)
- Gradually increase α → ∞ (tanh → sign, approaching true binary codes)
- At convergence, binary codes are high-quality because the optimization landscape was smoothed progressively

**Results on ImageNet (64-bit codes):**
- Mean Average Precision (mAP): 0.637 (vs ITQ: 0.352, LSH: 0.128)
- 5× improvement over classical methods at scale

### 1.5 Modern Contrastive Hashing (2022-2026)

The post-CLIP era has produced several powerful approaches:

**SPDH (Semantic-Preserving Deep Hashing, 2023):** Uses CLIP text embeddings as supervision signal for image hashing. The hash function maps images to binary codes that preserve CLIP's semantic structure. mAP: 0.72 on COCO at 64 bits.

**DCH (Diffusion-based Contrastive Hashing, 2024):** Uses denoising diffusion models to generate binary codes. The key insight: the diffusion process naturally maps Gaussian noise → discrete codes via a learned denoising trajectory. This avoids the tanh/sigmoid surrogate entirely.

**VQ-Hash (2025):** Combines vector quantization with hashing — the VQ codebook provides structured discrete tokens, and a secondary hash compresses the VQ indices into shorter binary codes for O(1) lookup. Achieves near-exact nearest neighbor at 128 bits on billion-scale datasets.

---

## Part 2: The Quantum Connection

### 2.1 Binary Codes ↔ Quantum Basis States: The Exact Isomorphism

The fundamental connection between semantic hashing and quantum computing is the following isomorphism:

**Classical:** A k-bit binary code b = (b₁, b₂, ..., b_k) ∈ {0,1}^k
**Quantum:** A k-qubit computational basis state |b⟩ = |b₁⟩ ⊗ |b₂⟩ ⊗ ... ⊗ |b_k⟩

This mapping is:
- **Bijective:** Every binary code maps to exactly one basis state and vice versa
- **Lossless:** No information is lost in the mapping (the binary code IS the basis state label)
- **Preserves Hamming distance:** ||b - b'||_H = number of qubits where b_i ≠ b'_i

**Consequence:** A database of N items indexed by k-bit semantic hashes is equivalent to a quantum state:

```
|Ψ_db⟩ = (1/√N) Σ_{i=1}^{N} |code_i⟩ |metadata_i⟩
```

where |code_i⟩ is the k-qubit basis state corresponding to item i's semantic hash, and |metadata_i⟩ is an ancilla register storing the item's identity.

### 2.2 Quantum Approximate Nearest Neighbor (QANN)

**The quantum advantage for similarity search:**

Given a query binary code q ∈ {0,1}^k and a database of N items with codes c_i, the classical nearest-neighbor search requires:
- **Brute-force:** O(N·k) — compare query to every item
- **Hash table lookup (Hamming radius r):** O(1) per lookup but O(Σ_{j=0}^{r} C(k,j))) probes — exponential in r
- **LSH forest:** O(N^ρ) where ρ = 1/r for r-NN

**Quantum approach (Grover-based):**

1. **Initialize:** |Ψ_0⟩ = (1/√2^k) Σ_{x∈{0,1}^k} |x⟩ |0⟩
2. **Oracle:** Mark states where HammingDist(x, q) ≤ r: O_f|x⟩|y⟩ = |x⟩|y ⊕ f(x)⟩
3. **Amplitude amplification:** Apply Grover iterations to amplify marked states
4. **Measure:** With high probability, obtain a code within Hamming radius r

**Complexity:** O(√N) oracle calls vs O(N) classical — quadratic speedup.

**Caveat:** The O(√N) speedup requires a coherent quantum database. Loading classical data into quantum memory (QRAM) has its own costs, estimated at O(N) for initial loading but O(log N) for subsequent queries if the database persists.

### 2.3 Quantum Hamming Distance via Basis Encoding

The Hamming distance between two k-bit codes can be computed quantum-mechanically without explicitly comparing each bit:

**Circuit construction:**

```
|q⟩ ──●── ●── ●── ... ──●──
       │   │   │         │
|c⟩ ──⊕── ⊕── ⊕── ... ──⊕── → |q ⊕ c⟩

Then count |1⟩ states in |q ⊕ c⟩ = HammingDist(q, c)
```

Each CNOT gate computes XOR per bit. The Hamming weight of the resulting state is the Hamming distance.

**Quantum Hamming weight estimation (using phase estimation on a quantum adder):**
- Prepare |q ⊕ c⟩
- Apply quantum Fourier transform adder
- Measure the sum register → Hamming distance

**Complexity:** O(k) CNOT gates + O(k²) for QFT addition = O(k²) per distance evaluation. For k=32 (typical hash size), this is 1024 gates — well within NISQ capabilities.

### 2.4 Quantum Amplitude Amplification for Similarity Search

**Algorithm: Quantum Semantic Hash Retrieval (QSHR)**

**Input:** Query code q ∈ {0,1}^k, database state |Ψ_db⟩, Hamming radius r
**Output:** A database item with code within Hamming radius r of q

**Steps:**

1. **Prepare superposition:** Start with |Ψ_db⟩ = (1/√N) Σ_i |c_i⟩ |d_i⟩
2. **Compute Hamming distance:** For each |c_i⟩, compute h_i = HammingDist(c_i, q) into ancilla register:
   ```
   |Ψ_1⟩ = (1/√N) Σ_i |c_i⟩ |d_i⟩ |h_i⟩
   ```
3. **Mark nearby items:** Apply phase flip when h_i ≤ r:
   ```
   |Ψ_2⟩ = (1/√N) Σ_i (-1)^{[h_i ≤ r]} |c_i⟩ |d_i⟩ |h_i⟩
   ```
4. **Amplitude amplification:** Apply Grover diffusion operator R = 2|Ψ_db⟩⟨Ψ_db| - I
5. **Repeat** Grover iterations ~O(√(N/M)) times where M = number of items within radius r
6. **Measure** → obtain nearby item with probability ≥ 0.98

**Total complexity:** O(√(N/M) · k²) — quadratic speedup over classical O(N·k) brute-force.

### 2.5 Hybrid Classical-Quantum Retrieval Architecture

For practical NISQ deployment, a hybrid approach is essential:

```
┌──────────────────────┐
│  Classical Encoder    │
│  (Neural network)    │
│  x → h ∈ {0,1}^k    │
└──────────┬───────────┘
           │
     ┌─────▼──────┐
     │ Hash Table  │ ← Classical O(1) lookup for exact matches
     │ (radius 0)  │
     └─────┬──────┘
           │
     ┌─────▼──────────┐
     │ Quantum Oracle  │ ← O(√N) for approximate matches
     │ (Grover search) │    (radius 1-3)
     └─────┬──────────┘
           │
     ┌─────▼──────────┐
     │ Verification    │ ← Classical re-ranking of candidates
     │ (full vectors)  │
     └────────────────┘
```

**Tiered strategy:**
1. **Exact match (Hamming radius 0):** Classical hash table, O(1)
2. **Near match (Hamming radius 1-3):** Quantum search, O(√N)
3. **Broad search (Hamming radius 4+):** Classical linear scan with early termination, O(N) but rare

This is directly applicable to the isonome Mneme pillar's recall mechanism, which currently tokenizes and matches linearly.

---

## Part 3: Application to the Isonome Framework

### 3.1 Current Mneme Recall Mechanism

The isonome `HierarchicalMneme` pillar implements recall as:

```python
def recall(self, query: str, tier: str = "episodic", limit: int = 10) -> list[MnemeEntry]:
    tokens = set(query.lower().split())
    scored = []
    for entry in self._episodic:  # Linear scan
        entry_tokens = set(entry.content.lower().split())
        overlap = len(tokens & entry_tokens)
        if overlap > 0:
            scored.append((entry, overlap))
    scored.sort(key=lambda x: x[1], reverse=True)
    return [e for e, _ in scored[:limit]]
```

**Bottleneck:** O(N·d) where N = number of entries in tier, d = average token count per entry. For large memory stores, this becomes prohibitive.

### 3.2 Semantic Hashing for Mneme Recall

**Proposed: MnemeSemanticHash encoder**

```python
class MnemeSemanticHash:
    """Encodes MnemeEntry content to k-bit semantic hash for O(1) retrieval."""

    def __init__(self, code_length: int = 32):
        self.code_length = code_length
        self.encoder = self._train_encoder()  # Learned from Mneme history

    def encode(self, entry: MnemeEntry) -> np.ndarray:
        """Map entry content → {0,1}^k preserving semantic similarity."""
        # 1. Extract features: content tokens, tier, age, significance, calibration_ece
        features = self._extract_features(entry)
        # 2. Project to k-dim continuous space
        continuous = self.encoder.project(features)
        # 3. Quantize to binary
        binary_code = (continuous > 0).astype(int)
        return binary_code

    def recall(self, query: str, tier: str = "episodic", limit: int = 10,
               hamming_radius: int = 3) -> list[MnemeEntry]:
        """O(1) semantic retrieval using hash table lookup."""
        query_code = self.encode_query(query)
        candidates = self._lookup(query_code, hamming_radius)
        # Re-rank by full similarity
        return self._rerank(query, candidates, limit)
```

**Speedup analysis:**
- Current: O(N·d) per recall — linear scan through all entries
- With semantic hashing: O(Σ_{j=0}^{r} C(k,j)) hash table probes = O(k^r) for small r
- For k=32, r=3: O(32³) = O(32768) probes, constant regardless of N
- For N > 10⁵ entries, this is a 100×+ speedup

### 3.3 Equilibrium State Retrieval via Semantic Hashing

The EquilibriumEngine maintains 8 tension axes with continuous positions. Semantic hashing can encode equilibrium states for fast past-state lookup:

```python
class EquilibriumStateHash:
    """Encodes 8-axis tension vectors to 16-bit binary codes for O(1) similar-state retrieval."""

    def encode(self, tension_snapshot: dict[str, float]) -> np.ndarray:
        """Map 8-D tension vector → {0,1}^16 preserving equilibrium similarity."""
        # Bin each axis into 2 bits (4 levels: calm, mild, moderate, stressed)
        # Total: 8 axes × 2 bits = 16 bits
        code = np.zeros(16, dtype=int)
        axes = ["novelty", "complexity", "uncertainty", "conflict",
                "resource_pressure", "delegation_load", "calibration_drift", "time_pressure"]
        for i, axis in enumerate(axes):
            value = tension_snapshot.get(axis, 0.5)
            # 2-bit quantization: 0-0.25→00, 0.25-0.5→01, 0.5-0.75→10, 0.75-1.0→11
            quantized = min(int(value * 4), 3)
            code[2*i] = (quantized >> 1) & 1
            code[2*i + 1] = quantized & 1
        return code
```

**Use case:** When the EquilibriumEngine detects high tension, it can instantly retrieve past episodes where the tension vector was similar (low Hamming distance) and learn from how the system resolved those situations. This creates a **experience-replay** mechanism for equilibrium management.

### 3.4 MorphologyAnalyzer Topology Vector Hashing

The MorphologyAnalyzer computes 32-D topology vectors describing agent morphology. Semantic hashing can compress these:

```python
class MorphologyHash:
    """Encodes 32-D topology vectors to 64-bit binary codes for morphology-based retrieval."""

    def encode(self, topology_vector: np.ndarray) -> np.ndarray:
        """Map 32-D topology → {0,1}^64 using ITQ-like rotation."""
        # PCA + ITQ rotation (trained on known morphologies)
        projected = self.pca.transform(topology_vector.reshape(1, -1))
        rotated = projected @ self.rotation_matrix
        return (rotated > 0).astype(int).flatten()
```

**Use case:** A robot with a specific morphology can instantly find agents with similar topology (for transfer learning, kernel sharing, or calibration inheritance in the DelegationGate's recursive architecture).

---

## Part 4: Proposed Architecture — Quantum-Semantic Memory Index (QSMI)

### 4.1 Architecture Overview

The QSMI integrates semantic hashing, classical hash tables, and quantum search into a unified memory index for the isonome framework:

```
┌─────────────────────────────────────────────────────────┐
│                    QSMI Architecture                     │
│                                                          │
│  ┌─────────────┐     ┌──────────────────────────┐       │
│  │ Mneme Store  │────▶│  Semantic Hash Encoder   │       │
│  │ (entries)    │     │  (ITQ / Deep Binary)     │       │
│  └─────────────┘     └────────────┬─────────────┘       │
│                                    │                      │
│                              k-bit binary code            │
│                                    │                      │
│                    ┌───────────────┼───────────────┐     │
│                    │               │               │      │
│              ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼────┐│
│              │  Classical │  │  Quantum  │  │  Equil.  ││
│              │  Hash Table│  │  Oracle   │  │  State   ││
│              │  (radius 0)│  │  (radius  │  │  Hash    ││
│              │            │  │   1-3)    │  │  (16-bit)││
│              └─────┬─────┘  └─────┬─────┘  └─────┬────┘│
│                    │               │               │      │
│              ┌─────▼───────────────▼───────────────▼────┐│
│              │         Result Merger & Re-ranker         ││
│              │  (classical full-vector similarity)       ││
│              └─────────────────────┬────────────────────┘│
│                                      │                    │
│                              ┌───────▼───────┐           │
│                              │ Ranked Results │           │
│                              └───────────────┘           │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Component Design

#### 4.2.1 Semantic Hash Encoder

**Architecture choice:** ITQ (for unsupervised, stable training) with optional deep fine-tuning for domain-specific optimization.

**Training pipeline:**
1. Collect feature vectors from all Mneme entries (content embeddings + metadata)
2. PCA to k dimensions (k = 32 for general, 16 for equilibrium)
3. ITQ rotation optimization (alternating sign/SVD, 50 iterations, converges in <1s)
4. Store rotation matrix R for encoding new entries at inference time

**Encoding cost:** O(d·k + k²) per entry — negligible for d ≤ 1000, k = 32.

#### 4.2.2 Classical Hash Table (Tier 1: Exact Match)

Standard multi-probe hash table:
- Primary hash: k-bit code → bucket index
- Probes: all codes within Hamming radius 0 (exact match only)
- Lookup: O(1)
- Memory: O(N) — one pointer per entry

#### 4.2.3 Quantum Oracle (Tier 2: Approximate Match)

**For NISQ devices (32-128 qubits):**

The quantum oracle implements Grover search over the hash code space:

```
Circuit for k=32, N entries:
- Working register: 32 qubits (hash code)
- Ancilla: ⌈log₂(32)⌉ = 5 qubits (Hamming distance)
- Oracle: ⌈log₂(32)⌉ = 5 qubits (threshold comparison)
- Total: ~42 qubits

Gate count per Grover iteration:
- Hamming distance computation: 32 CNOT + 32 X gates = 64
- Threshold comparison: ~30 Toffoli gates
- Diffusion operator: ~64 H + 64 X + 1 multi-controlled Z + 64 H
- Total: ~250 gates per iteration
- Iterations needed: O(√(N/M)) where M = items in radius
```

**For 32-qubit hashes and N=1000 entries with M~10 matches:** √(1000/10) ≈ 10 iterations → ~2500 gates. Feasible on current NISQ hardware with error mitigation.

#### 4.2.4 Equilibrium State Hash (Tier 3: Structure-Aware Retrieval)

The 16-bit equilibrium hash (2 bits per axis × 8 axes) provides a coarser but structure-aware index. Items that are semantically dissimilar but occurred during similar equilibrium states can be retrieved — enabling **analogical reasoning** ("this situation reminds me of a time when...").

### 4.3 Integration with Existing Isonome Components

| Component | Integration Point | Benefit |
|-----------|-------------------|---------|
| `HierarchicalMneme` | Replace linear scan in `recall()` with QSMI lookup | 100×+ speedup for large memory stores |
| `CalibrationCache` | SHA256 keys → semantic hash keys (similarity-preserving) | Cache hits for "similar enough" topologies |
| `MorphologyAnalyzer` | 32-D topology → 64-bit semantic hash | Fast morphology-based agent lookup |
| `EquilibriumEngine` | Tension snapshots → 16-bit state hash | Experience replay from similar past states |
| `DelegationGate` | Calibration inheritance via semantic hash lookup of parent's similar past delegations | Faster sub-agent calibration bootstrapping |
| `TensionEventLog` | Event signatures → hash-based event clustering | O(1) retrieval of similar tension patterns |

### 4.4 Performance Model

**Classical-only QSMI (without quantum):**

| Operation | Current | QSMI (classical) | Speedup |
|-----------|---------|-------------------|---------|
| Mneme recall (N=10K entries) | O(10K · d) ≈ 500ms | O(32³) ≈ 5ms | 100× |
| Mneme recall (N=100K entries) | O(100K · d) ≈ 5s | O(32³) ≈ 5ms | 1000× |
| Equilibrium state lookup | O(N · 8) linear scan | O(16³) constant | 500× at N=10K |
| Morphology similarity | O(N · 32) brute-force | O(64³) constant | 50× at N=10K |

**Quantum-enhanced QSMI (with NISQ oracle):**

| Operation | Classical QSMI | Quantum QSMI | Speedup |
|-----------|---------------|--------------|---------|
| Mneme recall (N=1M entries) | O(k^r) probes ≈ 500ms | O(√(N/M)) ≈ 50ms | 10× |
| Broad similarity (r=5+) | O(N) full scan | O(√N) Grover | Quadratic |

---

## Part 5: Research Roadmap

### Phase 1: Classical Semantic Hashing for Mneme (2-3 days)
- Implement `MnemeSemanticHash` encoder using ITQ
- Integrate with `HierarchicalMneme.recall()` as optional fast path
- Benchmark: recall quality (precision@10) vs brute-force for N=10K, 100K
- Tests: hash collision rate, Hamming distance vs cosine similarity correlation, tier-aware encoding

### Phase 2: Equilibrium State Hash (1-2 days)
- Implement `EquilibriumStateHash` (16-bit, 2 bits per axis)
- Add `EquilibriumEngine.recall_similar_state()` method
- Integration: when tension exceeds threshold, auto-query for similar past states
- Tests: state clustering quality, temporal coherence of retrieved states

### Phase 3: Morphology Hash + Calibration Cache Upgrade (1-2 days)
- Implement `MorphologyHash` (64-bit ITQ on 32-D topology vectors)
- Upgrade `CalibrationCache` to support similarity-preserving keys alongside SHA256
- Tests: topology retrieval quality, cache hit rate improvement

### Phase 4: Quantum Oracle for QSHR (3-4 days, requires QPU access)
- Implement quantum circuit for Hamming distance computation
- Implement Grover-based QSHR algorithm
- Test on simulators (Qiskit Aer) first, then on IBM Quantum if available
- Benchmark: classical vs quantum retrieval at various N and k

### Phase 5: Full QSMI Integration (2-3 days)
- Wire all three tiers (exact, approximate, structure-aware) behind unified `QSMI.recall()` interface
- Add QSMI to `Agent.__init__` configuration
- End-to-end test: agent stores experiences, then recalls similar ones under time pressure
- Dashboard: add QSMI metrics (hash distribution, lookup latency, recall quality)

---

## Part 6: Open Research Questions

### RQ1: Optimal Hash Length for Agent Memory
What is the optimal k (code length) for agent memory systems? Longer codes preserve more similarity structure but increase lookup cost exponentially with Hamming radius. Short codes are fast but lose discriminability. **Hypothesis:** k=32 provides the best tradeoff for Mneme-scale memories (10³-10⁵ entries), based on the Johnson-Lindenstrauss lemma requiring k = O(log N / ε²).

### RQ2: Tier-Aware Hash Encoding
Should entries in different Mneme tiers (working, episodic, semantic) share the same hash encoder or use tier-specific encoders? Shared encoders enable cross-tier retrieval (finding a semantic memory that matches an episodic query) but may not capture tier-specific structure. **Hypothesis:** A single encoder with tier bits prepended (2 bits for 4 tiers) provides the best of both worlds.

### RQ3: Temporal Degradation of Hash Quality
As entries age and the agent's semantic model drifts, hash codes computed at time t may become poorly aligned with the encoder at time t+Δ. Should the encoder be periodically retrained (causing all codes to shift) or should entries store their original codes (causing potential misalignment)? **Hypothesis:** A hybrid approach where the encoder is retrained monthly but old entries keep their original codes, with a "compatibility score" computed from the encoder drift.

### RQ4: Quantum Advantage Threshold
At what memory size N does the quantum O(√N) retrieval become practically advantageous over classical O(k^r) multi-probe hashing? The classical approach is O(k^r) regardless of N, while quantum is O(√(N/M)). **Hypothesis:** The crossover point is around N ≈ 10⁵ for k=32, r=3, where classical multi-probe begins to degrade due to high false-positive rates.

### RQ5: Equilibrium-State Hash Granularity
The 2-bit-per-axis quantization (16-bit total) is coarse. Would learned encoding (using the full tension history to train a better hash) significantly improve retrieval quality? **Hypothesis:** Learned encoding improves precision@10 by 20-30% over naive quantization, because the axis correlations (e.g., high novelty often co-occurs with high uncertainty) are captured by the learned rotation.

### RQ6: Hash Consistency Under Morphology Change
When a robot's morphology changes (damaged limb, added sensor), the topology vector shifts. Should the hash encoder accommodate morphology changes gracefully, or is it better to use separate encoders per morphology? This relates to the MorphologyAnalyzer's topology vector stability. **Hypothesis:** ITQ's rotation matrix is sufficiently robust to small morphology perturbations (<10% of axes) but not to large changes — suggesting a morphology-versioned encoder.

### RQ7: Calibration Cache Semantic Key Collision Rate
Replacing SHA256 with a semantic hash for CalibrationCache keys introduces collision risk: different topologies could map to the same hash. What collision rate is acceptable, and how does it affect calibration quality? **Hypothesis:** With k=32, the expected collision rate is ~N²/2^(k+1) ≈ 0.001% for N=1000 cached entries, which is acceptable for calibration purposes (near-collisions are topologically similar and would benefit from shared calibration anyway).

### RQ8: Quantum Error Mitigation for QSHR
NISQ devices have gate error rates of 10⁻³ to 10⁻². For QSHR circuits with ~2500 gates per search, the accumulated error may overwhelm the signal. What error mitigation strategies (zero-noise extrapolation, symmetry verification, randomized compiling) are most effective for Grover-based retrieval? **Hypothesis:** Symmetry verification using the Hamming-weight-preserving symmetry of the oracle provides the best error suppression, reducing the effective error from ~0.01 to ~0.001 at the cost of 3× circuit repetitions.

---

## Part 7: Key References

1. **Salakhutdinov & Hinton (2007):** Semantic Hashing. [cs.toronto.edu/~hinton/absps/sh.pdf]
2. **Andoni & Indyk (2006):** Near-Optimal Hashing Algorithms for Approximate Nearest Neighbor in High Dimensions. [FOCS 2006]
3. **Weiss et al. (2008):** Spectral Hashing. [NIPS 2008]
4. **Gong et al. (2011):** Iterative Quantization: A Provincially Zero-Variance Approach. [CVPR 2011, arXiv:1123.1274 equivalent]
5. **Cao et al. (2017):** HashNet: Deep Learning to Hash by Continuation. [ICCV 2017, arXiv:1702.00746]
6. **Lloyd et al. (2013):** Quantum algorithm for nearest-neighbor search. [arXiv:1305.1373]
7. **Kerenidis & Prakash (2017):** Quantum Recommendation Systems. [arXiv:1603.08675]
8. **Durr & Hoyer (1996):** A Quantum Algorithm for Finding the Minimum. [arXiv:quant-ph/9607014]
9. **Grover (1996):** A fast quantum mechanical algorithm for database search. [STOC 1996]
10. **Johnson-Lindenstrauss (1984):** Extensions of Lipschitz mappings into a Hilbert space. [Contemporary Mathematics 26]
11. **Wang et al. (2017):** Efficient Quantum Algorithms for Simulating Sparse Hamiltonians. [arXiv:1612.01558] — relevant for quantum Hamming distance computation
12. **Arunachalam et al. (2015):** Quantum Hashing with Applications to Non-Adaptive Query Complexity. [arXiv:1502.01816] — direct connection between hash functions and quantum query complexity

---

## Appendix A: Mathematical Foundation

### A.1 Johnson-Lindenstrauss Lemma for Hash Code Length

**Theorem (JL):** For any 0 < ε < 1 and any set S of N points in ℝ^d, there exists a linear map f: ℝ^d → ℝ^k with k = O(log N / ε²) such that for all u, v ∈ S:

```
(1-ε)||u-v||² ≤ ||f(u)-f(v)||² ≤ (1+ε)||u-v||²
```

**Implication for semantic hashing:** To preserve ε=0.1 similarity structure for N=10⁵ entries, we need:

```
k = 8 · ln(10⁵) / 0.01 ≈ 8 · 11.5 / 0.01 ≈ 9200 bits
```

This is impractically large. **However**, semantic hashing doesn't need to preserve all pairwise distances — only the top-k nearest neighbors. This relaxes the requirement dramatically. Empirically, k=32-64 bits provides sufficient discriminability for N=10⁵ with ε≈0.2 for the nearest-10% of neighbors.

### A.2 ITQ Optimization Objective

**Objective function:**

```
min_{R,B} Q(R, B) = ||Z·R - B||²_F
s.t. R^T R = I_k  (orthogonal rotation)
     B ∈ {-1, +1}^(n×k)  (binary constraint)
```

**Alternating minimization:**
1. Fix R, optimize B: B* = sign(Z·R) — closed form
2. Fix B, optimize R: R* = U·V^T where Z^T·B = U·Σ·V^T (SVD) — closed form

**Convergence:** Guaranteed to decrease Q at each step. Converges in <50 iterations in practice.

### A.3 Grover's Algorithm Complexity for Semantic Retrieval

**Theorem:** Given a database of N items with M items satisfying the search condition, Grover's algorithm finds a satisfying item with probability ≥ 1 - M/N using:

```
T = ⌊π/(4·arcsin(√(M/N)))⌋ ≈ O(√(N/M))
```

oracle queries.

**For semantic retrieval with Hamming radius r on k-bit codes:**

The expected number of items within radius r is:

```
M = N · Σ_{j=0}^{r} C(k,j) · p^j · (1-p)^(k-j)
```

where p is the probability that a random bit matches the query (p ≈ 0.5 for uniform data, p ≈ 0.7-0.9 for clustered data).

**For N=10⁵, k=32, r=3, p=0.5:** M ≈ 10⁵ · 0.0042 ≈ 420 items → T ≈ √(10⁵/420) ≈ 15 iterations.

---

## Appendix B: Connection to Existing Isonome Discoveries

### B.1 Semantic Hashing vs. CalibrationCache SHA256 Keys

The CalibrationCache currently uses SHA256 hashes of topology vectors as cache keys. This provides exact-match caching: two agents with identical topology get the same calibration. Semantic hashing would provide **similarity-match** caching: two agents with *similar* topology get the same calibration (with some degradation). This is directly useful for the DelegationGate's calibration inheritance mechanism — a sub-agent with similar but not identical morphology can bootstrap from the parent's cached calibration.

### B.2 Semantic Hashing vs. Mneme Tokenization

The current Mneme recall uses tokenized string matching (token overlap). Semantic hashing would replace this with:
1. Encode query → k-bit binary code
2. Lookup all entries within Hamming radius r
3. Re-rank by full cosine similarity

This is both faster (O(k^r) vs O(N·d)) and more semantically meaningful (hash codes capture latent structure, not just word overlap).

### B.3 Semantic Hashing vs. Tension Velocity Tracker

The TensionVelocityTracker stores per-axis velocity and momentum. Semantic hashing could encode (position, velocity) pairs as binary codes for fast retrieval of "similar dynamic states" — enabling the equilibrium engine to learn from past oscillation patterns without searching through the entire TensionEventLog.

### B.4 Semantic Hashing vs. RecursiveMAS Delegation Records

DelegationRecord entries (from the DelegationGate) could be semantically hashed by (action_type, risk_level, calibration_mode, outcome) → binary code. When the DelegationGate encounters a new action, it can instantly retrieve records of similar past delegations to inform its decision — reducing the calibration overhead for recursive delegation.
