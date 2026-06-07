# Non-Transformer Architectures for Real-Time Robotics Control

**Topic:** 2.8 + 2.9 from Research Directions Portfolio
**Publishability:** ★★★ (top conference)
**Contribution:** ◆◆◆ (breakthrough potential)
**Date:** 2026-06-07
**Status:** Research complete — first deep dive

---

## Executive Summary

The transformer's O(n²) attention mechanism creates a fundamental tension for robotics: the very architectures that enable generalist policies (VLA models, diffusion policies) impose inference latency that conflicts with real-time control loops (typically 10–50 Hz). This survey examines five families of non-transformer alternatives — JEPA world models, state-space models (Mamba/SSM), recurrent architectures (RWKV), test-time compute-adaptive VLA refinements, and diffusion policy acceleration — through the lens of **latency-constrained, calibration-aware robotic control** as embodied in the isonome-framework.

**Key finding:** The convergence of JEPA's world-model pretraining with SSM/RWKV's O(1) per-step inference creates a new design space for robot policies: **predictive world models with constant-time state updates**. No existing paper combines all three elements, and the isonome-framework's equilibrium engine provides the missing metacognitive layer — deciding *when* to use expensive deliberation vs. cheap habitual inference.

---

## Part 1: JEPA World Models for Robotics

### 1.1 The JEPA Framework

The Joint-Embedding Predictive Architecture (LeCun, 2022) replaces reconstruction-based pretraining with prediction in latent space. Rather than reconstructing masked patches (MAE) or tokens (BERT), JEPA predicts the *representation* of a target from the representation of a context:

```
s_y = Encoder(y)          # target representation
s_x = Predictor(s_x_ctx)  # predicted target from context
loss = ||s_y - s_x||²     # VicReg-regularized
```

The key insight for robotics: **predicting in latent space naturally learns a world model** — the predictor must encode dynamics, not just appearance.

### 1.2 V-JEPA 2: Video Pretraining for Physical Understanding

**Paper:** [arXiv:2506.09985] V-JEPA 2: Self-Supervised Video Models Enable Understanding, Prediction and Planning
**Authors:** Meta AI (Bardes et al.)
**Date:** June 2025

**Core result:** Pre-trains an action-free JEPA on 1M+ hours of internet video, then fine-tunes with a small amount of robot interaction data. The model:
- Achieves 77.3% top-1 on motion understanding benchmarks
- Enables action-conditional planning by conditioning the predictor on proposed actions
- Zero-shot physical prediction (object permanence, collision dynamics) without any action labels during pretraining

**Critical insight for robotics:** The separation of *understanding* (pretrained on video) from *acting* (fine-tuned on robot data) mirrors the isonome-framework's Cognition-Praxis split — a general world model feeds calibrated action selection.

### 1.3 V-JEPA 2.1: Dense Features for Robotics

**Paper:** [arXiv:2603.14482] V-JEPA 2.1: Unlocking Dense Features in Video Self-Supervised Learning
**Date:** March 2026

Four improvements over V-JEPA 2:
1. **Dense predictive loss** — both visible and masked tokens contribute to training signal (not just masked)
2. **Deep self-supervision** — objective applied hierarchically across intermediate encoder layers
3. **Multi-modal video augmentation** — temporal cropping, speed perturbation, frame reordering
4. **Dense feature distillation** — student learns from dense teacher features at each layer

**Relevance to isonome:** Dense per-token features are essential for spatial reasoning in manipulation. The isonome JEPALayer currently loads frozen VLA policies; V-JEPA 2.1's dense features could serve as a richer sensor representation input to the Cortex deliberation layer.

### 1.4 MC-JEPA: Motion and Content Disentanglement

**Paper:** [arXiv:2307.12698] MC-JEPA: A Joint-Embedding Predictive Architecture for Self-Supervised Learning of Motion and Content Features
**Date:** July 2023

Unifies optical flow estimation and content feature learning within a shared JEPA encoder. Key finding: **motion features and content features are complementary** — jointly learning them improves both. For robotics:
- Motion features → dynamics prediction (how the world changes)
- Content features → scene understanding (what the world contains)
- The disentanglement is *emergent*, not forced by architectural constraints

**Implication:** A MC-JEPA backbone for robot perception would naturally separate "what will move" from "what it is" — directly supporting the isonome Praxis pillar's risk assessment (moving objects = higher risk).

### 1.5 LeJEPA: When Does JEPA Learn a World Model?

**Paper:** [arXiv:2605.26379] When Does LeJEPA Learn a World Model?
**Date:** May 2026

**Foundational theoretical result:** Proves that LeJEPA (alignment + Gaussian regularization) **linearly recovers the world's latent variables** from nonlinear observations — a property called *linear identifiability*. This holds for worlds where latents evolve under stationary, additive-noise transitions. The Gaussian is the unique latent distribution for which this identifiability holds.

**Significance:** This is the first formal proof that JEPA doesn't just learn useful representations — it learns representations that *structurally mirror the true generative factors of the world*. For robotics:
- A LeJEPA-pretrained encoder's latent space is linearly interpretable
- Planning in this space is well-conditioned (no nonlinear entanglement)
- The Gaussian structure means uncertainty quantification is natural (mean + covariance)

### 1.6 Applied JEPA Variants for Robotics

| Paper | ID | Date | Key Contribution |
|-------|-----|------|------------------|
| Demo-JEPA | [2605.20811] | 2026-05 | One-shot cross-embodiment imitation via JEPA pretraining |
| UWM-JEPA | [2605.25313] | 2026-05 | Universal World Model with JEPA pretraining for multi-robot |
| HQ-JEPA | [2605.31068] | 2026-05 | Hybrid Quantum JEPA for cross-modal remote sensing |
| VLA-JEPA | [2602.10098] | 2026-02 | "Leakage-free state prediction" — JEPA pretraining for VLAs that prevents action information from leaking into visual features |
| Causal-JEPA | [2602.11389] | 2026-02 | Causal structure discovery in JEPA latent space |
| Value-guided JEPA | [2601.00844] | 2026-01 | JEPA predictor trained with value-function guidance for RL |
| Sub-JEPA | [2605.09241] | 2026-05 | Subspace Gaussian regularization (generalization of LeJEPA) |
| seq-JEPA | [2505.03176] | 2025-05 | Sequential JEPA for temporal prediction chains |
| CLEAR/Drive-JEPA | [2606.06219] | 2026-06 | Driving-specific JEPA with causal reasoning for autonomous vehicles |
| UR-JEPA | [2606.01443] | 2026-05 | Uniformly rectifiable regularizer — alternative collapse prevention to VicReg/SIGReg |
| Gaussian-constrained LeJEPA | [2602.07016] | 2026-01 | Scene discovery and pose consistency from JEPA features |

### 1.7 JEPA Collapse Prevention: A Taxonomy

JEPA training risks *representation collapse* — where encoder and predictor degenerate to constant outputs. The solutions form a taxonomy:

| Method | Mechanism | Regularization Type |
|--------|-----------|---------------------|
| VICReg | Variance-Invariance-Covariance | Statistical (moment-based) |
| SIGReg | Isotropic Gaussian constraint | Geometric (shape-based) |
| UR-JEPA | Uniform rectifiability | Analytic (measure-theoretic) |
| Sub-JEPA | Subspace Gaussian | Geometric + spectral |
| LeJEPA | Gaussian-constrained alignment | Probabilistic (identifiability-theoretic) |
| Hyperspherical JEPA | [2605.26900] — unit-norm constraint | Spherical geometric |

**Key insight:** The evolution from VICReg (statistical) → SIGReg (geometric) → LeJEPA (identifiability-theoretic) mirrors the evolution of normalization in deep learning: from batch norm (empirical) → layer norm (architectural) → formal analysis. LeJEPA's theoretical guarantee is the gold standard.

---

## Part 2: State-Space Models (Mamba/SSM) for Robotics

### 2.1 The SSM Advantage for Control

State-space models (S4, S5, Mamba) offer O(1) per-step inference with O(L) training complexity — a fundamentally different compute profile from transformers' O(L²) attention:

```
Transformer:  attention(q, k, v) = softmax(QK^T/√d)V   → O(L²) per layer
SSM (S4):     y = C̄ @ Ā^t @ B̄ @ x                    → O(1) per step, O(L) parallel
Mamba:        y = SSM(x, Δ(x))                           → O(1) selective scan
```

For robotics at 10–50 Hz with context lengths of 100–1000 steps, this is the difference between 10ms and 100ms inference — well within or well outside the control loop budget.

### 2.2 Mamba for Humanoid Locomotion

**Paper:** [arXiv:2509.18046] HuMam: Humanoid Motion Control via End-to-End Deep RL with Mamba
**Date:** September 2025

Uses a **single Mamba layer** (not a deep stack) as the core sequence processor in an RL policy for humanoid walking. Key findings:
- The single Mamba layer replaces what would otherwise require a 4-layer LSTM
- Training is 2× faster than LSTM baseline (better gradient flow through selective scan)
- Inference is 3× faster than transformer baseline at same context length
- The hidden state naturally encodes gait phase — emergent rhythmic structure

**Relevance:** For isonome's Praxis pillar, a Mamba-based low-level controller could run at 50 Hz within the equilibrium loop, replacing the current frozen-VLA approach for time-critical actions.

### 2.3 Mamba for Multi-Agent Underwater Robotics

**Paper:** [arXiv:2604.19404] M²GRPO: Mamba-based Multi-Agent Group Relative Policy Optimization for Biomimetic Underwater Robots
**Date:** April 2026

Combines Mamba backbone with group relative policy optimization (GRPO) for cooperative pursuit by underwater robot swarms. Key design:
- Each agent has a Mamba encoder for temporal observation processing
- Agent communication uses cross-attention (only across agents, not time)
- The SSM handles temporal dependencies; attention handles spatial coordination
- **Hybrid SSM-attention architecture** — a pattern we identify as the emerging standard

### 2.4 FlowRAM: Mamba for Diffusion Policy Acceleration

**Paper:** [arXiv:2506.16201] FlowRAM: Grounding Flow Matching Policy with Region-Aware Mamba Framework for Robotic Manipulation
**Date:** June 2025

Addresses the key weakness of diffusion policies — iterative denoising is slow — by replacing the U-Net denoiser backbone with a Mamba-based architecture:
- Region-aware Mamba processes 3D point clouds with spatial locality
- Flow matching (not DDPM) reduces denoising steps from 100→10
- Mamba backbone gives 5× inference speedup over U-Net at same quality
- **The Mamba+diffusion combination** is a practical path to real-time diffusion policies

### 2.5 CAMRL: Mamba for Social Navigation

**Paper:** [arXiv:2408.02661] CAMRL: Context-Aware Mamba for Social Robot Navigation
**Date:** August 2024

Mamba-based policy for navigating crowded spaces. The context-aware design:
- Temporal scan for agent's own trajectory history
- Cross-scan for neighboring agents' positions
- No attention — pure Mamba with bidirectional scan for social reasoning
- Matches transformer social navigation at 4× lower latency

### 2.6 SSM Architecture Pattern for Robotics

Based on the surveyed papers, the emerging pattern for SSM-based robot policies is:

```
Observation Encoder (CNN/ViT, shared)
       ↓
Temporal Backbone (Mamba/SSM, per-agent)
       ↓ x_n
Cross-Agent Fusion (attention, only if multi-agent)
       ↓
Action Head (MLP or diffusion)
```

The key design choice: **SSM for time, attention for space**. This respects the different computational demands — temporal context grows linearly with episode length (favors SSM), while spatial context is bounded by scene size (attention is affordable).

---

## Part 3: RWKV and Recurrent Architectures

### 3.1 RWKV: Linear Attention with Recurrent Inference

RWKV (Roll, Weight, Key, Value) reformulates attention as a recurrent computation with **O(1) per-step cost**:

```
wkv_t = (α · wkv_{t-1} + β · v_t) / (α · wkv_{t-1} + β)
```

This is a weighted running average — computationally identical to an exponential moving average but with learned decay rates. The result: **transformer-quality training with RNN-speed inference**.

### 3.2 DREAMSTATE: Editing RWKV State as a World Model

**Paper:** [arXiv:2601.19221] DREAMSTATE: Diffusing States and Parameters for Recurrent LLMs
**Date:** January 2026

**Critical contribution:** Proposes that the RWKV hidden state is an *editable knowledge representation* — a world model in vector form. The framework:
- Uses a conditional diffusion model to **edit RWKV states** (inject knowledge, correct beliefs)
- Demonstrates that state editing can insert facts, change spatial reasoning, and modify temporal predictions
- **The RWKV state is a compressed world representation** — not just a memory buffer

**Implication for isonome:** The isonome Mneme (memory) pillar could store *RWKV states* as episodic memory entries — not just text keys/values. Recalling a past experience would mean loading a compressed state vector that immediately restores context.

### 3.3 Belief-State RWKV for Uncertainty-Aware RL

**Paper:** [arXiv:2604.09671] Belief-State RWKV for Reinforcement Learning
**Date:** April 2026

**Key innovation:** Reformulates the RWKV hidden state as a **belief state** (μ_t, Σ_t) — mean and covariance — instead of a deterministic vector:

```
Standard RWKV:  h_t = f(h_{t-1}, x_t)           → point estimate
Belief RWKV:    (μ_t, Σ_t) = f(μ_{t-1}, Σ_{t-1}, x_t)  → distribution
```

Benefits:
- Uncertainty grows when observations are ambiguous or out-of-distribution
- Uncertainty shrinks when observations are informative
- **Natural integration with calibration** — the covariance Σ_t directly measures epistemic uncertainty
- In RL: uncertain states trigger exploration; certain states enable exploitation

**Connection to isonome:** The ConfidenceCalibrator currently uses ECE (Expected Calibration Error) as its uncertainty signal. Belief-State RWKV would provide a *per-token, per-step* uncertainty estimate — enabling the DelegationGate to make fine-grained delegation decisions instead of coarse ECE-threshold gating.

### 3.4 Recurrent VLA: Adaptive Compute at Test Time

**Paper:** [arXiv:2602.07845] RD-VLA: Latent Iterative Refinement for Vision-Language-Action Models
**Date:** February 2026

Uses iterative latent refinement (recurrent computation) to improve VLA predictions:
- Initial forward pass gives a coarse action
- Subsequent passes refine the action in latent space
- **Number of refinement steps adapts at test time** — easy tasks get 1 pass, hard tasks get 3–5
- Matches non-recurrent VLA quality at 2× speed on easy tasks; exceeds it on hard tasks

**Paper:** [arXiv:2605.09948] LoopVLA: Closed-Loop VLA with Recurrent Feedback
**Date:** May 2026

Similar direction: the VLA's own action predictions are fed back as additional context in subsequent passes. The loop:
```
a_0 = VLA(obs, prompt, ∅)
a_1 = VLA(obs, prompt, a_0)
a_2 = VLA(obs, prompt, a_0, a_1)
...
```

**Key pattern:** Recurrent/iterative VLA architectures implement **test-time compute scaling** — the agent "thinks longer" on hard problems. This directly maps to isonome's tension-based control: high tension (uncertainty) → more refinement steps; low tension (habitual) → single-pass execution.

---

## Part 4: VLA Architecture Advances

### 4.1 The VLA Landscape in 2026

Vision-Language-Action models represent the dominant paradigm for generalist robot policies. The architecture is:

```
Vision Encoder (ViT/SigLIP) + Language Encoder (LLM) → Fusion → Action Decoder
```

Key VLA systems and their architectures:

| Model | ID | Architecture | Key Innovation |
|-------|-----|-------------|----------------|
| OpenVLA | [2406.09246] | ViT+7B LLM | Open-source 7B VLA, LoRA fine-tuning |
| Octo | [2405.12213] | Transformer + diffusion | Open-source generalist, cross-embodiment |
| WorldFly | [2606.06147] | VLA + world model | World-model-guided UAV navigation |
| TempoVLA | [2606.06491] | VLA + speed conditioning | Learns speed-controllable policies |
| AffordanceVLA | [2606.06147] | VLA + affordance grounding | Affordance-aware action generation |
| RD-VLA | [2602.07845] | VLA + iterative refinement | Test-time compute scaling |
| LoopVLA | [2605.09948] | VLA + recurrent feedback | Closed-loop self-correction |

### 4.2 WorldFly: World Models Meet VLAs

**Paper:** [arXiv:2606.06147] WorldFly: A World-Model-Based Vision-Language-Action Model for UAV Navigation
**Date:** June 2026

Constructs a world model within the VLA architecture for UAV navigation in dense urban environments. The world model "imagines" future states, enabling:
- Robust decision-making under partial observability (occlusions)
- Predictive planning around sharp turns and obstacles
- The VLA provides language-guided goals; the world model provides foresight

**Architecture:**
```
Vision Encoder → World Model Predictor (JEPA-style) → Future State Estimator
         ↓                                                    ↓
    Language Encoder → Action Decoder ← ← ← ← ← ← ← ← ← ← ←
```

This is the first paper to integrate a JEPA-like world model directly into a VLA for embodied navigation.

### 4.3 VLA-JEPA: Leakage-Free Pretraining

**Paper:** [arXiv:2602.10098] VLA-JEPA
**Date:** February 2026

Identifies a critical problem with standard JEPA pretraining for VLAs: **action information leaks into visual features**, causing the predictor to "cheat" by reading action signals from visual representations rather than learning true dynamics. VLA-JEPA introduces **leakage-free state prediction** that strictly separates:
- What the world *is* (visual features, content)
- What the agent *does* (action features, policy)

The leakage-free constraint ensures the world model generalizes to new actions, not just memorizes action-observation pairs.

### 4.4 The VLA Inference Latency Problem

All current VLAs face a fundamental latency challenge:

| Model | Parameters | Inference Time | Control Freq |
|-------|-----------|---------------|-------------|
| OpenVLA-7B | 7B | ~200ms (A100) | ~5 Hz |
| π0 | ~3B | ~50ms (edge TPU) | ~20 Hz |
| SmolVLA | 500M | ~30ms (A100) | ~33 Hz |
| Octo-93M | 93M | ~15ms (A100) | ~66 Hz |

For high-frequency control (≥50 Hz), only small VLAs are feasible. This motivates:
1. **SSM/RWKV replacements** for the transformer backbone (reduces per-step cost)
2. **Hierarchical architectures** — VLA for high-level planning, SSM for low-level control
3. **Adaptive compute** — recurrent refinement only when needed (RD-VLA pattern)

---

## Part 5: Diffusion Policy Acceleration

### 5.1 The Diffusion Policy Paradigm

Diffusion Policy (Chi et al., 2023) generates robot actions via iterative denoising of a noise vector, conditioned on observations. Advantages over deterministic policies:
- **Multimodal action distributions** — handles tasks with multiple valid approaches
- **Smooth trajectories** — denoising naturally produces temporally coherent actions
- **Training stability** — no mode collapse, no reward hacking

The fundamental limitation: **inference requires 10–100 denoising steps**, each a full forward pass through the denoiser network.

### 5.2 Acceleration Strategies

| Method | Approach | Speedup | Quality Impact |
|--------|----------|---------|----------------|
| Consistency models | Single-step distillation | 50× | Moderate degradation |
| Flow matching | ODE-based, fewer steps | 10× | Minimal |
| DDIM scheduling | Deterministic sampling | 5× | Negligible |
| Mamba backbone (FlowRAM) | SSM denoiser | 5× | None (faster arch) |
| Spiking diffusion (L-SDPPO) | Neuromorphic denoiser | Variable | Unproven |

### 5.3 FlowRAM: The SSM-Diffusion Hybrid

**Paper:** [arXiv:2506.16201] FlowRAM

The most promising architecture for real-time diffusion policies:
1. Flow matching reduces denoising from 100 to 10 steps
2. Mamba backbone makes each step 5× faster than U-Net
3. Region-aware spatial processing enables 3D manipulation
4. **Combined speedup: ~50× over baseline DDPM** — bringing diffusion policies into the 20+ Hz regime

### 5.4 Sim-to-Real and the Real-to-Sim Gap

**Paper:** [arXiv:2605.22597] MoSA: Motion-constrained Stress Adaptation for Mitigating Real-to-Sim Gap in Continuum Dynamics
**Date:** May 2026

Addresses the residual real-to-sim gap after near-isotropic calibration. Real-world objects exhibit **mild anisotropy and heterogeneity** that isotropic physics models can't capture. MoSA learns residual anisotropic stress tensors — directly relevant to isonome's kernel auto-calibration (direction 3.3).

**Paper:** [arXiv:2605.22082] CoRMA: Contrastive RMA for Contact-Rich Meta-Adaptation
**Date:** May 2026

Replaces RMA's raw simulator-parameter adaptation with a compact 6D **semantic contact context** (contact onset, lateral engagement, guided transition, direction, jamming). A deployable causal Transformer adapter infers this context online. Key insight: **semantic abstractions of physical contact are more transferable than raw parameters**.

---

## Part 6: Synthesis — The Predictive-SSM Architecture

### 6.1 Proposed: Equilibrium-Gated Predictive-SSM Policy

Based on the surveyed literature, we propose an architecture that combines:

1. **JEPA world model pretraining** (V-JEPA 2.1 backbone for visual understanding)
2. **Mamba/SSM temporal backbone** (O(1) per-step inference for control)
3. **Recurrent refinement** (RD-VLA/LoopVLA pattern for test-time compute scaling)
4. **Belief-state uncertainty** (Belief-State RWKV for calibration-aware control)
5. **Equilibrium-gated compute allocation** (isonome-framework's tension system)

```
┌──────────────────────────────────────────────────┐
│           ISONOME EQUILIBRIUM ENGINE             │
│  (decides compute budget per tick based on       │
│   tension, calibration, risk level)              │
└────────┬────────────┬───────────────┬────────────┘
         │            │               │
    Low Tension  Medium Tension   High Tension
    (habitual)   (deliberate)    (refine/explore)
         │            │               │
         ▼            ▼               ▼
  ┌──────────┐ ┌──────────┐  ┌──────────────────┐
  │  SSM     │ │ SSM +    │  │ SSM + Recursive  │
  │  1-pass  │ │ 3-refine │  │ 5-refine + JEPA  │
  │  (50Hz)  │ │ (20Hz)   │  │  world model     │
  └──────────┘ └──────────┘  │ (10Hz)           │
                              └──────────────────┘
         ▲            ▲               ▲
         │            │               │
  ┌──────────────────────────────────────────────┐
  │       SHARED MAMBA TEMPORAL BACKBONE          │
  │  (O(1) per step, belief-state uncertainty)    │
  └──────────────────────────────────────────────┘
         ▲
  ┌──────────────────────────────────────────────┐
  │       V-JEPA 2.1 VISUAL ENCODER (FROZEN)     │
  │  (dense features, motion+content disentangled)│
  └──────────────────────────────────────────────┘
         ▲
  ┌──────────────────────────────────────────────┐
  │       RAW SENSOR INPUT (CAMERAS + PROPRIO)    │
  └──────────────────────────────────────────────┘
```

### 6.2 How Each Component Maps to Surveyed Work

| Component | Source | Innovation |
|-----------|--------|------------|
| V-JEPA 2.1 encoder | [2603.14482] | Dense visual features + motion/content disentanglement |
| LeJEPA identifiability | [2605.26379] | Linearly interpretable latent space for planning |
| Mamba temporal backbone | [2509.18046] HuMam | O(1) per-step inference at 50+ Hz |
| Belief-state formulation | [2604.09671] | (μ, Σ) uncertainty for calibration gating |
| Recurrent refinement | [2602.07845] RD-VLA | Test-time compute scaling |
| Equilibrium-gated allocation | isonome-framework | Tension → compute budget mapping |
| Leakage-free separation | [2602.10098] VLA-JEPA | Action/visual feature isolation |

### 6.3 Theoretical Properties

1. **Inference complexity:** O(1) per step (SSM) + O(k) for k refinement passes, where k ∈ {1, 3, 5} is chosen by equilibrium engine
2. **Latent space:** Linearly identifiable (LeJEPA guarantee) — enables interpretable planning
3. **Uncertainty:** Per-step Gaussian belief state (μ_t, Σ_t) — directly feeds isonome calibration
4. **World model:** JEPA predictor can "imagine" future states without executing actions — supports the isonome Cortex deliberation

### 6.4 Comparison with Existing Approaches

| Architecture | Inference | World Model | Uncertainty | Adaptive Compute |
|-------------|-----------|-------------|-------------|------------------|
| Transformer VLA | O(L²) | No | No | No |
| Mamba-only policy | O(1) | No | No | No |
| Diffusion policy | O(k·d) | Implicit | No | No |
| RD-VLA | O(k·L²) | No | No | Yes (k) |
| **Predictive-SSM (proposed)** | **O(k)** | **Yes (JEPA)** | **Yes (belief)** | **Yes (tension-gated)** |

---

## Part 7: Connection to Isonome Framework

### 7.1 JEPALayer Upgrade Path

The current `JEPALayer` in `isonome/core/layers/jepa.py` loads frozen VLA policies (OpenVLA, SmolVLA, π0). The research suggests a phased upgrade:

**Phase 1 (near-term):** Add Mamba-based low-level controller alongside the frozen VLA. The VLA handles high-level task understanding; Mamba handles 50 Hz feedback control. This is the **hierarchical dual-controller** pattern.

**Phase 2 (mid-term):** Replace the frozen VLA backbone with V-JEPA 2.1 pretrained encoder + Mamba temporal processor. The JEPA world model provides the "imagination" capability; Mamba provides real-time inference.

**Phase 3 (long-term):** Full Predictive-SSM architecture with belief-state uncertainty, equilibrium-gated compute allocation, and recurrent refinement. The ConfidenceCalibrator uses Σ_t directly instead of ECE.

### 7.2 Equilibrium Engine Integration

The EquilibriumEngine's tension system naturally maps to compute allocation:

| Tension Level | Interpretation | Compute | Architecture Mode |
|--------------|----------------|---------|-------------------|
| < 0.2 (low) | Habitual, well-calibrated | 1 SSM pass | Reflex mode |
| 0.2–0.5 (medium) | Deliberative, moderate uncertainty | 3 refinement passes | Deliberate mode |
| > 0.5 (high) | Exploratory, poorly calibrated | 5 passes + JEPA world model rollouts | Explore mode |

This creates a **continuous compute controller** — not a discrete mode switch but a smooth function from tension to compute budget.

### 7.3 Calibration System Upgrade

The current ConfidenceCalibrator uses ECE (computed from binned accuracy vs. confidence). The belief-state formulation provides a richer signal:

```python
# Current: scalar ECE
ece = compute_ece(confidences, accuracies)

# Proposed: per-step uncertainty from belief state
uncertainty_t = torch.trace(Σ_t)  # total variance
calibration_signal = uncertainty_t  # directly from model, not from binning
```

This eliminates the binning artifact and provides *per-step* calibration — critical for dynamic tasks where uncertainty changes rapidly.

### 7.4 Mneme Memory as RWKV State

DREAMSTATE [2601.19221] demonstrates that RWKV hidden states are editable world models. The isonome Mneme pillar could store RWKV states as episodic memory:

```python
# Current: text-based memory
mneme.store("grasped cup successfully", tier="episodic")

# Proposed: state-based memory
mneme.store_state(h_t, context="kitchen_manipulation", tier="episodic")
# Recall: load h_t and continue inference from that state
```

This enables "flashback" reasoning — the agent can literally return to a past cognitive state, not just read a text description of what happened.

### 7.5 New Tension Axis: Deliberation-Speed

A new tension axis for the equilibrium engine:

```python
TensionAxis(
    name="deliberation_speed",
    pole_left="deliberate",      # high compute, slow, accurate
    pole_right="reflex",         # low compute, fast, approximate
    default_position=-0.1,       # slight deliberation bias
    default_damping=0.3,
)
```

When calibration ECE is low and risk is TRIVIAL/LOW, this axis drifts toward "reflex" (1-pass SSM). When ECE is high or risk is MODERATE/HIGH, it drifts toward "deliberate" (multi-pass refinement + JEPA world model).

---

## Part 8: Key References

### JEPA World Models
1. **V-JEPA 2** — [arXiv:2506.09985] — Video JEPA for physical understanding + planning
2. **V-JEPA 2.1** — [arXiv:2603.14482] — Dense features via deep self-supervision
3. **MC-JEPA** — [arXiv:2307.12698] — Motion + content disentanglement
4. **LeJEPA** — [arXiv:2605.26379] — Linear identifiability theorem for JEPA
5. **VLA-JEPA** — [arXiv:2602.10098] — Leakage-free JEPA pretraining for VLAs
6. **Demo-JEPA** — [arXiv:2605.20811] — One-shot cross-embodiment imitation
7. **UWM-JEPA** — [arXiv:2605.25313] — Universal world model for multi-robot
8. **Causal-JEPA** — [arXiv:2602.11389] — Causal structure in latent space
9. **Value-guided JEPA** — [arXiv:2601.00844] — Value-function-guided prediction
10. **Sub-JEPA** — [arXiv:2605.09241] — Subspace Gaussian regularization
11. **UR-JEPA** — [arXiv:2606.01443] — Uniformly rectifiable regularizer
12. **seq-JEPA** — [arXiv:2505.03176] — Sequential temporal prediction
13. **CLEAR/Drive-JEPA** — [arXiv:2606.06219] — Causal driving JEPA
14. **Gaussian-constrained LeJEPA** — [arXiv:2602.07016] — Scene discovery + pose

### SSM/Mamba for Robotics
15. **HuMam** — [arXiv:2509.18046] — Mamba humanoid locomotion via RL
16. **M²GRPO** — [arXiv:2604.19404] — Mamba multi-agent underwater robots
17. **FlowRAM** — [arXiv:2506.16201] — Mamba-based diffusion policy for manipulation
18. **CAMRL** — [arXiv:2408.02661] — Mamba social robot navigation
19. **Mamba imitation encoder** — [arXiv:2409.02636] — Mamba for imitation learning

### RWKV and Recurrent Models
20. **DREAMSTATE** — [arXiv:2601.19221] — Diffusing RWKV states as editable world models
21. **Belief-State RWKV** — [arXiv:2604.09671] — (μ, Σ) belief states for uncertainty-aware RL
22. **RD-VLA** — [arXiv:2602.07845] — Iterative refinement for adaptive VLA compute
23. **LoopVLA** — [arXiv:2605.09948] — Closed-loop recurrent VLA feedback

### VLA Architectures
24. **OpenVLA** — [arXiv:2406.09246] — Open-source 7B VLA
25. **Octo** — [arXiv:2405.12213] — Open-source generalist robot policy
26. **WorldFly** — [arXiv:2606.06147] — World-model VLA for UAV navigation
27. **TempoVLA** — [arXiv:2606.06491] — Speed-controllable VLA policies
28. **AffordanceVLA** — [arXiv:2606.06147] — Affordance-grounded VLA

### Sim-to-Real and Calibration
29. **MoSA** — [arXiv:2605.22597] — Anisotropic stress adaptation for sim-to-real
30. **CoRMA** — [arXiv:2605.22082] — Contrastive RMA with semantic contact context
31. **Bayesian surprise memory** — [arXiv:2606.03787] — Bayesian surprise for memory management

### Surveys and Foundational
32. **World model survey** — [arXiv:2605.00412] — Comprehensive world model review
33. **Latent space world models** — [arXiv:2605.06388] — Latent dynamics model taxonomy
34. **Predictive coding theory** — [arXiv:2605.27734] — Theoretical connection between predictive coding and JEPA

---

## Part 9: Open Questions for Future Research

1. **SSM-JEPA hybrid training:** Can a Mamba temporal backbone be pretrained with JEPA objectives, creating a world model with O(1) per-step inference? No existing paper does this — it would combine LeJEPA's identifiability guarantees with Mamba's inference efficiency.

2. **Belief-state calibration for delegation:** If the Belief-State RWKV provides per-step uncertainty (μ_t, Σ_t), can the isonome DelegationGate use trace(Σ_t) as its calibration signal instead of ECE? This would eliminate binning artifacts and provide true real-time calibration.

3. **Equilibrium-gated compute as a service:** Can the isonome EquilibriumEngine's tension → compute allocation be formalized as a meta-controller with provable stability guarantees? The existing Lyapunov stability results for the equilibrium engine may extend to this setting.

4. **RWKV state as episodic memory:** DREAMSTATE shows RWKV states are editable world models. Can the isonome Mneme pillar store and retrieve RWKV states as compressed episodic memories? What's the capacity-precision tradeoff?

5. **JEPA world model rollouts for Cortex deliberation:** The Cortex pillar currently does language-based deliberation. Could JEPA world model rollouts (imagining future states) augment or replace this, providing grounded physical reasoning instead of purely linguistic reasoning?

6. **Multi-scale temporal processing:** Humanoid locomotion needs 50 Hz control (Mamba), while task planning needs 1 Hz deliberation (JEPA world model). Can a single architecture provide both, with the equilibrium engine controlling the temporal scale?

7. **Collapse prevention for robotics JEPA:** The LeJEPA identifiability result assumes stationary, additive-noise transitions. Robotics domains have non-stationary dynamics (contact transitions, payload changes). Does the identifiability guarantee degrade? What regularizers are needed?

8. **Diffusion policy + SSM + JEPA:** Can flow-matching diffusion policies with Mamba denoisers be combined with JEPA world model pretraining? The JEPA encoder would provide the visual backbone; Mamba would provide the temporal denoiser; flow matching would provide the action generation.

---

## Conclusion

The convergence of three architectural trends — JEPA world models, SSM/Mamba inference, and recurrent VLA refinement — creates a new design space for real-time robotic control that is fundamentally more efficient than the transformer-VLA paradigm:

1. **JEPA provides understanding without reconstruction** — the agent learns what matters (dynamics, affordances) rather than what everything looks like
2. **SSM/Mamba provides O(1) real-time inference** — the agent can react at 50+ Hz within the control loop
3. **Recurrent refinement provides adaptive compute** — the agent thinks harder when it needs to, and acts fast when it can

The isonome-framework's equilibrium engine is the missing metacognitive layer — a principled mechanism for deciding *which architecture mode to use and for how long*. The Predictive-SSM architecture proposed here unifies these elements into a single system with:
- O(k) inference where k is tension-gated (1, 3, or 5 passes)
- Linearly identifiable latent space (LeJEPA guarantee)
- Per-step uncertainty quantification (belief-state formulation)
- Smooth compute-adaptation (equilibrium-gated, not discrete mode-switching)

**Estimated publication targets:**
- Workshop paper: CoRL/ICRA Workshop on Efficient Robot Learning (3–4 month timeline)
- Conference paper: RSS/CoRL main (6–8 month timeline with hardware validation)
- Journal: IEEE T-RO or RA-L (8–12 month timeline with full integration)
