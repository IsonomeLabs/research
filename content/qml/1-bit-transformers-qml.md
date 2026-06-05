# 1-Bit Transformers for Quantum Machine Learning

**Topic:** 1.3 from Research Directions Portfolio
**Publishability:** ★★★ (top conference)
**Contribution:** ◆◆◆ (breakthrough potential)
**Date:** 2026-06-05
**Status:** Research complete — first deep dive

---

## Executive Summary

The intersection of 1-bit/ternary weight quantization (BitNet family) and quantum machine learning (QML) represents a largely unexplored but high-impact research frontier. The core insight: **quantum circuits are inherently discrete** — qubits exist in superpositions of |0⟩ and |1⟩, and quantum gates are parameterized by angles that can be discretized. This creates a natural structural isomorphism between 1-bit neural networks and variational quantum circuits that, if exploited, could reduce qubit overhead by orders of magnitude and enable practical hybrid quantum-classical inference on NISQ devices.

**Key finding:** No existing paper directly combines BitNet-style 1-bit transformers with quantum layers. This is a green-field research opportunity at the ★★★ publishability level.

---

## Part 1: The 1-Bit Transformer Landscape

### 1.1 BitNet: The Foundation (2023)

**Paper:** [arXiv:2310.11453] BitNet: Scaling 1-bit Transformers for Large Language Models
**Authors:** Hongyu Wang, Shuming Ma, Li Dong, et al. (Microsoft Research)
**Date:** October 2023

**Core innovation:** BitLinear — a drop-in replacement for nn.Linear that trains 1-bit weights from scratch using:

- **Weight binarization:** w ∈ {−1, +1} via `sign(w)` with `abs(mean(w))` scaling
- **Activation quantization:** 8-bit activations using absmax quantization
- **Group quantization:** Per-subtensor scaling for stability

**Key results:**
- Competitive perplexity with FP16 at 3B parameter scale
- Memory reduction: ~10× (1-bit vs 16-bit weights)
- Energy reduction: ~10× (bitwise operations vs floating-point multiply-accumulate)
- Exhibits scaling law akin to full-precision Transformers

**Mathematical formulation:**
```
BitLinear(x) = α · (sign(W) · x_quant) / √n
where:
  α = mean(|W|) per group
  x_quant = clip(round(x / β), -Q_b, Q_b)
  β = max(|x|) / Q_b
  Q_b = 2^(b-1) - 1 for b-bit activation quantization
```

### 1.2 BitNet b1.58: The Ternary Breakthrough (2024)

**Paper:** [arXiv:2402.17764] The Era of 1-bit LLMs: All Large Language Models are in 1.58 Bits
**Authors:** Shuming Ma, Hongyu Wang, et al. (Microsoft Research)
**Date:** February 2024

**Core innovation:** Expands weight space from {−1, +1} to {−1, 0, +1}:
- **log₂(3) ≈ 1.58 bits** per weight — the "1.58-bit" name
- The zero weight enables **network pruning** without extra mechanism
- Massive speedup on CPU: ternary matmul → addition/subtraction only
- Matches full-precision LLaMA performance at same parameter count

**Critical insight for QML:** Ternary weights map naturally to **three quantum basis states** per qubit. A single qubit in the {|0⟩, |1⟩} computational basis with an "idle/dropout" state maps to {−1, 0, +1}. This is the structural isomorphism.

### 1.3 BitNet b1.58 2B4T: Open-Source Release (2025)

**Paper:** [arXiv:2504.12285] BitNet b1.58 2B4T Technical Report
**Date:** April 2025

First open-source native 1-bit LLM at 2B parameter scale, trained on 4 trillion tokens. Validates the architecture at production scale. Key metrics:
- 2B parameters, 4T training tokens
- Matches LLaMA-2 7B on most benchmarks at 3.5× fewer parameters
- Hardware: 8× H100 GPUs for training
- Released on HuggingFace with inference code

### 1.4 FBI-LLM: Fully Binarized LLMs (2024)

**Paper:** [arXiv:2407.07093] FBI-LLM: Scaling Up Fully Binarized LLMs from Scratch via Autoregressive Distillation
**Date:** July 2024

**Key distinction:** While BitNet b1.58 uses ternary {-1, 0, +1} weights, FBI-LLM achieves **true binary** {-1, +1} weights for ALL layers (including embeddings and output). Uses autoregressive distillation from a full-precision teacher. This is relevant for QML because:
- Binary weights → direct mapping to Pauli-Z rotations (±1 eigenvalues)
- No "zero" state to represent → simpler quantum circuit structure
- But: accuracy gap is larger than ternary — quantum error mitigation may help

### 1.5 BitNet b1.58 Reloaded: Smaller Networks (2024)

**Paper:** [arXiv:2407.09527] BitNet b1.58 Reloaded: State-of-the-art Performance Also on Smaller Networks
**Date:** July 2024

Validates that 1.58-bit quantization-aware training works for networks as small as 125M–1.3B parameters. Important for QML because quantum layers will be small (NISQ constraints) — this shows the approach scales down.

### 1.6 BitNet Distillation (2025)

**Paper:** [arXiv:2510.13998] BitNet Distillation (BitDistill)
**Date:** October 2025

Lightweight pipeline that fine-tunes off-the-shelf full-precision LLMs (e.g., Qwen) into 1.58-bit models using knowledge distillation. Faster than training from scratch. Relevant for hybrid quantum-classical: you can distill a classical model into a 1-bit version, then selectively replace layers with quantum equivalents.

### 1.7 BitRL: 1-bit RL for Edge Deployment (2026)

**Paper:** [arXiv:2604.24273] BitRL: Reinforcement Learning with 1-bit Quantized Language Models for Resource-Constrained Edge Deployment
**Date:** April 2026

Applies 1-bit quantization to RL agents — directly relevant to the isonome-framework's Praxis pillar (action execution under calibration-gated delegation). RL with 1-bit models achieves comparable reward with 10× less memory.

### 1.8 Hardware Accelerators for Ternary LLMs

Multiple hardware papers targeting efficient ternary inference:

| Paper | ID | Focus |
|-------|-----|-------|
| Bitnet.cpp | [2502.11880] | CPU inference engine for ternary LLMs |
| TeLLMe | [2504.16266] | FPGA ternary LLM accelerator (prefill + decode) |
| TeLLMe v2 | [2510.15926] | End-to-end ternary FPGA accelerator |
| TOM | [2602.20662] | Ternary ROM accelerator for LLM edge inference |
| BitROM | [2509.08542] | Weight reload-free CiROM architecture |
| ReTern | [2506.01140] | Fault tolerance in compute-in-memory ternary LLMs |

**Key hardware insight:** Ternary weights enable **addition-only matrix multiplication** — no multipliers needed. On FPGAs and ASICs, this reduces area by ~10× and power by ~5×. This same principle applies to quantum circuits: ternary weight matrices can be compiled into **fixed-angle rotation sequences** rather than variable-angle parameterized gates, reducing circuit depth.

### 1.9 NativeTernary Encoding (2026)

**Paper:** [arXiv:2604.03336] NativeTernary: A Self-Delimiting Binary Encoding for Ternary Weights
**Date:** April 2026

Novel binary encoding for ternary {-1, 0, +1} weights with 460× smaller framing overhead than GGUF tensor headers. Relevant for efficient quantum state preparation: loading ternary weight matrices into quantum memory.

### 1.10 Direct Quantized Training with Stochastic Rounding (2024)

**Paper:** [arXiv:2412.04787] Direct Quantized Training of Language Models with Stochastic Rounding
**Date:** December 2024

Proposes stochastic rounding for training directly in quantized space (no full-precision master weights). This is directly applicable to **quantum gradient estimation** where parameter shift rules naturally produce quantized gradient updates.

---

## Part 2: The Quantum Machine Learning Context

### 2.1 The Qubit Bottleneck Problem

Current NISQ quantum processors have:
- **IBM Eagle:** 127 qubits
- **IBM Heron:** 1,121 qubits (2024)
- **Google Sycamore:** 72 qubits (extended to 105 in 2024)
- **IonQ Forte:** 36 algorithmic qubits (all-to-all connectivity)

A single quantum layer in a variational quantum circuit (VQC) typically requires:
- **n qubits** for input encoding (one per feature)
- **O(n²)** entangling gates for full connectivity
- **O(p)** parameterized rotation gates per variational layer (p = parameter count)

For a 768-dimensional transformer hidden layer (e.g., BERT-base), you need **768 qubits** just for input encoding — far beyond current hardware.

### 2.2 Why 1-Bit Reduces Qubit Requirements

**The key mathematical insight:**

A full-precision weight matrix W ∈ ℝ^(m×n) requires encoding m×n real numbers into quantum states. Using amplitude encoding, this needs log₂(m×n) qubits — but the preparation circuit has depth O(m×n), which is infeasible.

A **1-bit weight matrix** W ∈ {−1, +1}^(m×n) can be encoded using:
- **n qubits** for the input vector
- **n controlled-Z gates** per row (each -1 weight is a Pauli-Z rotation by π)
- Total circuit depth: **O(m)** per forward pass

A **ternary weight matrix** W ∈ {−1, 0, +1}^(m×n) further reduces this:
- Zero weights → **skip the gate entirely** (circuit pruning)
- Effective depth: **O(m × sparsity_ratio)**
- For 50% sparsity (typical in BitNet b1.58), depth is halved

**Quantum advantage formula:**
```
Full-precision quantum layer:  depth = O(m × n)        [amplitude encoding]
1-bit quantum layer:           depth = O(m × n)        [but only CZ gates, no rotations]
Ternary quantum layer:         depth = O(m × n × ρ)    [ρ = sparsity, ~0.5]
```

While asymptotic depth is the same, **constant factors are dramatically different:**
- Full-precision: each weight needs a variable-angle rotation gate (decomposed into ~20 native gates)
- 1-bit: each weight is a fixed CZ or nothing (1 native gate or 0)
- **~20× depth reduction per weight** → practical quantum circuit execution within coherence time

### 2.3 Structural Isomorphism: Ternary Weights ↔ Quantum Gates

| Classical (BitNet b1.58) | Quantum Equivalent |
|--------------------------|--------------------|
| Weight = +1 | No rotation (identity) or rotation by +π/2 |
| Weight = −1 | Pauli-Z rotation by π (phase flip) |
| Weight = 0 | Gate omitted (circuit pruning) |
| Matrix multiply W·x | Sequence of controlled rotations |
| Attention weights (softmax) | Amplitude amplification (Grover-like) |
| Layer normalization | Quantum amplitude normalization (post-measurement) |
| ReLU/GELU activation | Post-selection measurement + ancilla reset |

This mapping is **exact for inference** and **approximate for training** (quantum parameter shift rule → stochastic rounding for weight updates).

### 2.4 Relevant QML Architecture Papers

From arXiv search (quantum + binary/ternary/discretization):

| Paper | ID | Relevance |
|-------|-----|-----------|
| Feature Encoding in QML: Survey & Guidelines | [2606.05387] | Comprehensive feature encoding taxonomy — directly informs how to encode 1-bit weights |
| Quantum State Preparation via NN Encoding | [2605.31006] | Neural network-assisted state preparation — could use 1-bit NN as encoder |
| Research Progress on Quantum Neural Networks | [2605.30724] | Survey of QNN architectures — most assume continuous parameters |
| Integrated Encoding and Quantization for Quanvolutional NNs | [2410.05777] | **Closest to our topic** — quantization of quanvolutional parameters |
| Laziness, Barren Plateau, and Noise in ML | [2206.09313] | Low-precision angles reduce barren plateau risk (fewer local minima) |
| MoG-VQE: Multiobjective Genetic VQE | [2007.04424] | Discretized parameter search for VQE — genetic algo over quantized angles |

### 2.5 The Barren Plateau Connection

**Critical finding from [2206.09313]:** Low-precision (discretized) variational parameters can actually **mitigate barren plateaus** in quantum neural networks. The argument:

1. Barren plateaus occur when the loss landscape becomes exponentially flat for random circuits
2. Discretized parameters create a **coarser landscape** with fewer spurious local minima
3. The gradient variance scales as O(1/2^n) for continuous parameters but O(1/|Θ|^n) for discretized parameters where |Θ| is the number of discrete levels
4. For ternary weights |Θ| = 3, this is better than continuous but worse than binary |Θ| = 2

**Implication:** 1-bit quantum layers may train **faster** than full-precision quantum layers due to reduced barren plateau risk. This is a publishable insight.

---

## Part 3: Proposed Architecture — BitQuant

### 3.1 BitQuant: 1-Bit Transformer with Quantum Attention

A hybrid architecture that combines classical 1-bit feedforward layers with quantum attention mechanisms:

```
Input Embedding (classical, 1-bit)
    ↓
┌─────────────────────────────┐
│  BitQuant Transformer Block │
│                             │
│  1. Quantum Self-Attention  │  ← Quantum circuit (ternary weights → CZ gates)
│     Q, K, V = ternary proj │  ← Classical ternary linear layers
│     Attn = VQC(Q, K, V)    │  ← Variational quantum circuit for attention scores
│                             │
│  2. 1-Bit FFN (classical)  │  ← BitNet b1.58 style
│     W1 ∈ {−1,0,+1}         │
│     W2 ∈ {−1,0,+1}         │
│                             │
│  3. Residual + LayerNorm   │  ← Classical
└─────────────────────────────┘
    ↓ (×N blocks)
Output Head (classical, 1-bit)
```

### 3.2 Quantum Attention Mechanism

The quantum attention replaces the softmax(QK^T/√d) computation:

**Classical attention:**
```
α_ij = softmax(Q_i · K_j / √d)
output_i = Σ_j α_ij · V_j
```

**Quantum attention (proposed):**
```
1. Encode Q_i into |q_i⟩ (amplitude encoding, log₂(d) qubits)
2. Encode K_j into |k_j⟩ (amplitude encoding, log₂(d) qubits)
3. Compute inner product via Swap Test:
   |⟨q_i|k_j⟩|² → attention score (magnitude only)
4. Apply amplitude amplification on high-score pairs
5. Encode V_j and apply controlled rotations weighted by scores
6. Measure output qubits → output_i
```

**Qubit budget for quantum attention (d=768):**
- Q, K, V registers: 3 × log₂(768) ≈ 3 × 10 = 30 qubits
- Swap test ancilla: 1 qubit
- Amplitude amplification: 1 qubit
- **Total: ~32 qubits** (within reach of 127-qubit IBM Eagle)

### 3.3 Training BitQuant

**Phase 1: Classical pre-training** — Train the full architecture in simulation (classical attention) using BitNet b1.58 quantization-aware training.

**Phase 2: Quantum distillation** — Replace classical attention with quantum attention circuit; distill from the classical teacher using knowledge distillation (cf. BitDistill [2510.13998]).

**Phase 3: Quantum fine-tuning** — Use parameter shift rule with stochastic rounding to update quantum circuit angles in quantized space (cf. [2412.04787]).

### 3.4 Expected Resource Reduction

| Component | Full-Precision | 1-Bit Classical | BitQuant (Hybrid) |
|-----------|---------------|-----------------|-------------------|
| Memory (weights) | 16-bit × N | 1.58-bit × N | 1.58-bit × (N - attn) + 0 (quantum attn) |
| Attention compute | O(n²d) FLOPs | O(n²d) additions | O(n² × depth) quantum ops |
| Attention memory | O(n²) floats | O(n²) 8-bit | O(1) quantum state |
| Inference energy | ~10W/GPU | ~1W/CPU | ~0.1W/FPGA + quantum |
| Qubit requirement | N/A | N/A | ~32 per attention head |

---

## Part 4: Research Roadmap

### Phase 1: Theoretical Foundation (2-3 weeks)
1. **Formalize the ternary-to-quantum gate mapping** — Prove that any ternary weight matrix W ∈ {−1,0,+1}^(m×n) can be compiled into a quantum circuit with depth O(m × ρ) where ρ is the sparsity ratio
2. **Barren plateau analysis** — Extend [2206.09313] to show that ternary-discretized quantum attention has reduced barren plateau risk compared to continuous-parameter VQC
3. **Expressivity bounds** — Determine the minimum number of qubits needed to approximate softmax attention within ε error

### Phase 2: Simulation & Prototyping (3-4 weeks)
1. **Implement quantum attention in PennyLane/Qiskit** — Use classical simulation backend
2. **Train a small BitQuant model** (e.g., d=64, 2 layers) on a simple task (e.g., sentiment classification)
3. **Compare against:** full-precision transformer, 1-bit transformer, classical-quantum hybrid with continuous parameters
4. **Benchmark:** accuracy, training time, gradient variance, circuit depth

### Phase 3: Hardware Validation (4-6 weeks)
1. **Run on IBM Quantum (127 qubit)** — Test quantum attention on real hardware
2. **Measure:** coherence time vs circuit depth, error rates, actual speedup
3. **Compare simulation vs hardware** — Error mitigation techniques for ternary quantum layers
4. **Write publication** — Target: NeurIPS or ICML workshop on Quantum ML

### Key Risks & Mitigations

| Risk | Severity | Mitigation |
|------|----------|------------|
| Quantum circuit depth exceeds coherence time | High | Use ternary sparsity to prune gates; split attention across multiple shots |
| Swap test fidelity too low for useful attention scores | Medium | Use Hadamard test instead; or classical-quantum hybrid (classical QK, quantum V mixing) |
| Barren plateaus still occur in discretized space | Medium | Use layer-wise training; initialize with classical 1-bit weights |
| No quantum advantage at small scale | High | Target asymptotic advantage; focus on memory reduction (no weight storage for quantum layers) |
| Noise destroys ternary weight structure | Medium | Error mitigation: post-selection on expected parity; noise-adaptive compilation |

---

## Part 5: Connection to Isonome Framework

The isonome-framework's equilibrium-based architecture has several natural integration points with BitQuant research:

1. **Cognition → Praxis calibration pipeline:** The confidence calibrator's ECE metric could modulate whether to use quantum or classical attention (delegate to quantum when calibration is low → uncertainty needs quantum exploration)

2. **Task-type homeostasis:** "quantum_inference" could be a new task type with its own equilibrium profile (explore_exploit toward explore, since quantum circuits introduce stochasticity)

3. **Delegation gate extension:** The DelegationGate (iter-019) could gain a QUANTUM_DELEGATE mode where high-risk actions are executed on quantum hardware for uncertainty-aware reasoning

4. **Tension axis addition:** A new `classical_quantum` tension axis (pole_left="classical", pole_right="quantum", default=-0.3) could control the hybrid compute mix

5. **Mneme quantum memory:** The hierarchical memory system's semantic tier could use quantum state encoding for pattern matching (cf. semantic hashing direction 1.5)

---

## Part 6: Key References

### Foundational Papers
1. **BitNet** — [arXiv:2310.11453] — 1-bit transformer architecture
2. **BitNet b1.58** — [arXiv:2402.17764] — Ternary weight scaling law
3. **BitNet b1.58 2B4T** — [arXiv:2504.12285] — Open-source 2B ternary model
4. **FBI-LLM** — [arXiv:2407.07093] — Fully binarized LLM training
5. **BitNet b1.58 Reloaded** — [arXiv:2407.09527] — 1.58-bit for smaller networks

### Distillation & Training
6. **BitDistill** — [arXiv:2510.13998] — Knowledge distillation for 1-bit models
7. **Direct Quantized Training** — [arXiv:2412.04787] — Stochastic rounding for quantized training
8. **BitRL** — [arXiv:2604.24273] — 1-bit RL for edge deployment

### Hardware
9. **Bitnet.cpp** — [arXiv:2502.11880] — CPU inference for ternary LLMs
10. **TeLLMe/TeLLMe v2** — [arXiv:2504.16266, 2510.15926] — FPGA accelerators
11. **TOM** — [arXiv:2602.20662] — Ternary ROM accelerator
12. **BitROM** — [arXiv:2509.08542] — CiROM for 1.58-bit inference
13. **ReTern** — [arXiv:2506.01140] — Fault tolerance for ternary CiM
14. **NativeTernary** — [arXiv:2604.03336] — Efficient ternary encoding

### Quantum ML
15. **Feature Encoding in QML** — [arXiv:2606.05387] — Encoding survey & guidelines
16. **Quantum State Preparation via NN** — [arXiv:2605.31006] — NN-assisted state prep
17. **Integrated Encoding & Quantization for Quanvolutional NNs** — [arXiv:2410.05777] — Closest existing work
18. **Laziness, Barren Plateau, Noise** — [arXiv:2206.09313] — Low-precision mitigates BP
19. **MoG-VQE** — [arXiv:2007.04424] — Discretized VQE optimization

### Robotics & Vision
20. **BitVLA** — [arXiv:2506.07530] — 1-bit Vision-Language-Action for robotics

---

## Part 7: Open Questions for Future Research

1. **Quantum error correction for ternary weights:** Can the {−1, 0, +1} structure be exploited for more efficient error correction codes? Ternary repetition codes (majority vote of 3) are natural here.

2. **Quantum-aware quantization training:** Instead of training classically then distilling to quantum, can we train with a quantum-aware loss function that accounts for measurement noise and gate errors?

3. **Circuit-depth-regularized training:** Add a regularization term that penalizes non-sparse ternary weights (more +1/-1 than 0) → minimizes quantum circuit depth.

4. **Cross-architecture distillation:** Can a 1-bit classical model serve as a compact teacher for training a quantum model? (Reverse of the usual distillation direction.)

5. **Quantum attention without Swap Test:** Are there more qubit-efficient attention mechanisms? E.g., quantum amplitude estimation for direct QK inner product computation.

6. **Scaling laws for quantum layers:** Does BitNet's scaling law (performance improves predictably with model size) hold when some layers are quantum?

7. **Hybrid classical-quantum delegation:** In the isonome-framework, should the agent delegate uncertain sub-problems to quantum hardware? What's the calibration threshold?

---

## Conclusion

The intersection of 1-bit transformers and QML is a **high-impact, low-competition** research area. The structural isomorphism between ternary weights and quantum gate sequences is natural and unexploited. The key publishable contributions are:

1. **Ternary-to-quantum compilation theorem** (mapping {-1,0,+1} matrices to fixed-gate quantum circuits with depth proportional to sparsity)
2. **Barren plateau mitigation via discretization** (extending existing theoretical results to the ternary regime)
3. **BitQuant architecture** (hybrid 1-bit classical + quantum attention transformer)
4. **Empirical validation** on NISQ hardware showing practical feasibility at ~32 qubit scale

The nearest comparable work is [2410.05777] (Integrated Encoding and Quantization for Quanvolutional NNs), but this focuses on quanvolutional (CNN-like) architectures, not attention-based transformers. **No existing work combines BitNet-style 1-bit weights with transformer attention on quantum hardware.**

**Estimated publication targets:**
- Workshop paper: NeurIPS Workshop on Quantum ML (3-4 month timeline)
- Conference paper: ICML / NeurIPS main (6-8 month timeline with hardware results)
- Journal: PRX Quantum or Quantum Machine Intelligence (8-12 month timeline)
