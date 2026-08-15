# CHAPTER 8: Qiskit Aer Noise Simulation

# and Hardware Benchmarking

# Kraus Operators · Depolarising · Amplitude/Phase Damping · Qiskit Aer · QV · CLOPS · NISQ

<div class="box box-anecdote">
<p class="box-title"><strong>📜  The Noise Problem — Bell Labs, 1940s to Today</strong></p>
<p>Noise has been the central adversary of information technology since its beginning. Claude Shannon's 1948 paper "A Mathematical Theory of Communication" proved a remarkable theorem: no matter how noisy a classical communication channel is (provided its capacity is non-zero), it is possible to transmit information with arbitrarily low error rate — using error-correcting codes and sufficient redundancy. Shannon's theorem gave us digital communications, reliable hard drives, and deep-space telemetry. Every classical computer chip today operates error-free despite billions of transistors, precisely because classical errors are manageable.</p>
<p>Quantum noise is more subtle and more vicious. In the classical world, an error is a bit flip: 0 becomes 1. You detect and correct it by majority vote. In the quantum world, errors are continuous: any rotation of the Bloch vector is an error. The qubit can partially flip, partially dephase, partially leak to |2⟩ — all simultaneously, in superposition. And you cannot simply measure the qubit to check for errors: measurement destroys the quantum information. For decades after Feynman proposed quantum computers in 1982, many physicists believed quantum error correction was fundamentally impossible.</p>
<p>In 1995, Peter Shor proved them wrong. He showed that quantum error correction is possible — that quantum codes can protect arbitrary quantum states against arbitrary noise, using entangled ancilla qubits as "parity checkers" without measuring the data qubits directly. The quantum information science that followed built the theoretical foundation for fault-tolerant quantum computing. This chapter is the practical engineering sequel: we learn to characterise, model, simulate, and measure quantum noise using the language of quantum channels, the tools of Qiskit Aer, and the benchmarks of Quantum Volume and CLOPS.</p>
</div>

Chapter 8 completes the treatment of quantum noise begun in Chapter 7. We start with a unified framework for all noise channels through the operator-sum representation, then analyse in full depth the three physically most important channels: the depolarising channel (symmetric noise), the amplitude damping channel (T1 energy relaxation), and the phase damping channel (T2 dephasing). We develop the complete Qiskit Aer toolkit for noise simulation — from simple depolarising errors to calibration-accurate backend models — and introduce error mitigation techniques that reduce effective noise without full error correction. We close with a rigorous treatment of Quantum Volume and CLOPS as hardware benchmarks, a survey of randomised benchmarking, and a frank analysis of NISQ era limitations and the path to fault-tolerant quantum computing.

## 8.1 Quantum Noise: A Unified Framework

The quantum computing noise problem is fundamentally different from classical noise. Classical computers use voltage levels well above thermal noise, and digital logic regenerates clean signals at each gate. Classical error rates per operation approach 10⁻¹⁵ — so low that one can build a computer with 10¹⁰ transistors operating for years without a single error. Quantum computers currently achieve error rates of 10⁻³ to 10⁻² per two-qubit gate — roughly 10¹²–10¹³ times worse. Understanding why quantum noise is harder to eliminate, and how to model it precisely, is the starting point for all practical quantum computing.

### 8.1.1 Why Ideal Unitary Evolution Is Insufficient

Quantum mechanics describes the evolution of a closed (isolated) system by a unitary operator U: |ψ⟩ → U|ψ⟩. This is perfectly reversible, information-preserving, and noise-free. Real quantum systems are never perfectly isolated. Every qubit is surrounded by an environment — substrate phonons, background radiation, control electronics, neighbouring qubits — and interacts with it continuously. This interaction transfers quantum information from the qubit to the environment, causing decoherence: the loss of quantum coherence that makes quantum computation possible.

The density matrix formalism (Chapter 2) handles this naturally. A pure state |ψ⟩ has density matrix ρ = |ψ⟩⟨ψ| with Tr(ρ²) = 1. After interacting with an environment and tracing over environmental degrees of freedom, the qubit state becomes a mixed state with Tr(ρ²) < 1: information has leaked into the environment and is no longer accessible. This entropy increase is irreversible — the qubit has become entangled with the macroscopic environment.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Why Unitary Evolution Cannot Describe Noise</strong></p>
<p>Suppose a qubit starts in pure state |ψ⟩ and interacts with an environment via unitary U_SE (on system+environment). After tracing over the environment:</p>
<p>The entanglement with the environment means the qubit state alone cannot be described by a pure state after the interaction. The most general physically allowed transformation of a qubit density matrix is a quantum channel (CPTP map), not a unitary.</p>
<p>Consequence for computation: every gate operation on real hardware is not U but U followed by a noise channel ε. The effective operation is ε∘U: first apply the intended unitary, then apply noise. Qiskit Aer noise simulation implements exactly this model.</p>
</div>

### 8.1.2 The Operator-Sum Representation Revisited

The Kraus operator (operator-sum) representation introduced in Chapter 7 is not just a mathematical convenience — it is the unique canonical form for any physically allowed quantum operation. In this section we develop it more deeply, establishing the connection between Kraus operators and physical noise processes, and proving the key properties.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Kraus Representation: Canonical Form and Physical Meaning</strong></p>
<p>Every quantum channel ε on a d-dimensional system can be written:</p>
<p>Physical derivation from system-environment coupling: Let the joint system+environment start in ρ_SE = ρ_S ⊗ |0_E⟩⟨0_E| (environment in a known pure state — a standard assumption called the "system-environment initial decorrelation"). After unitary evolution U_SE on the joint system, the reduced state of the system is:</p>
<p>Non-uniqueness: The Kraus representation is NOT unique. Given one set {Kₖ}, any other set {K̃ₖ} = {Σⱼ uₖⱼ Kⱼ} (where u is any unitary matrix) gives the same channel. This freedom corresponds to choosing different bases for the environment — different "measurement records" — all producing the same physical noise channel.</p>
<p>Special cases: (1) Unitary evolution: single K₀ = U, completeness U†U = I ✓. (2) Complete decoherence: K₀ = |0⟩⟨0|, K₁ = |1⟩⟨1|, gives diagonal ρ. (3) Measurement in Z basis: same Kraus operators, trace = 1 only on average.</p>
</div>

The Choi-Kraus isomorphism gives a deep connection: every CPTP map on a d-dimensional system corresponds uniquely to a positive semidefinite d²×d² matrix (the Choi matrix J(ε) = (I⊗ε)(|Φ+⟩⟨Φ+|), where |Φ+⟩ is a maximally entangled state). The Kraus operators are the square roots of J(ε). This connection is exploited in quantum process tomography (see Project 8.B).

### 8.1.3 Single-Qubit Pauli Channels: Complete Treatment

A Pauli channel is any quantum channel whose Kraus operators are multiples of the four Pauli matrices {I, X, Y, Z}. Pauli channels form a natural basis for qubit noise because any single-qubit error can be decomposed into Pauli errors through the Pauli channel representation. They are also the standard noise model used in Qiskit Aer's pauli\_error function.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: General Single-Qubit Pauli Channel</strong></p>
<p>The most general single-qubit Pauli channel is:</p>
<p>Action on the Bloch vector r = (r_x, r_y, r_z): The Pauli channel contracts the Bloch vector along each axis:</p>
<p>This shows that Pauli channels produce an ellipsoidal deformation of the Bloch sphere. The special cases are: (1) Bit-flip: λ_x=1, λ_y=λ_z=1-2p. (2) Phase-flip: λ_z=1, λ_x=λ_y=1-2p. (3) Depolarising: λ_x=λ_y=λ_z=1-4p/3.</p>
</div>

<figure class="book-figure">
<img src="content/images/image86.png" alt="Figure 8.1: Complete quantum noise channel taxonomy. Top row: Bloch sphere cross-sections (xz plane) showing the deformed ellipsoids for bit-flip (compressed vertically, z-axis), phase-flip (compressed horizontally, x-axis), and depolarising (uniform contraction) channels at p=0.3. The dashed circle is the ideal unit sphere. Bottom row: State fidelity vs error probability p for each channel on different input states, showing that each channel preferentially preserves certain basis states (|+⟩ is immune to bit-flip; |0⟩ is immune to phase-flip) — the foundation of quantum error correction code design.">
<figcaption>Figure 8.1: Complete quantum noise channel taxonomy. Top row: Bloch sphere cross-sections (xz plane) showing the deformed ellipsoids for bit-flip (compressed vertically, z-axis), phase-flip (compressed horizontally, x-axis), and depolarising (uniform contraction) channels at p=0.3. The dashed circle is the ideal unit sphere. Bottom row: State fidelity vs error probability p for each channel on different input states, showing that each channel preferentially preserves certain basis states (|+⟩ is immune to bit-flip; |0⟩ is immune to phase-flip) — the foundation of quantum error correction code design.</figcaption>
</figure>

<div class="box box-example">
<p class="box-title"><strong>Example 8.1: Pauli Channel Bloch Vector Contraction</strong></p>
<p>Problem: A qubit in state |+⟩ = (|0⟩+|1⟩)/√2 (Bloch vector r = (1,0,0)) undergoes a Pauli channel with p_I=0.6, p_X=0.1, p_Y=0.1, p_Z=0.2. Find the output Bloch vector and state.</p>
<p><strong>Solution:</strong></p>
<p>Compute shrinkage factors:</p>
<p>λ_x = p_I + p_X - p_Y - p_Z = 0.6 + 0.1 - 0.1 - 0.2 = 0.4</p>
<p>λ_y = p_I - p_X + p_Y - p_Z = 0.6 - 0.1 + 0.1 - 0.2 = 0.4</p>
<p>λ_z = p_I - p_X - p_Y + p_Z = 0.6 - 0.1 - 0.1 + 0.2 = 0.6</p>
<p>Output Bloch vector: r' = (λ_x·1, λ_y·0, λ_z·0) = (0.4, 0, 0)</p>
<p>Output density matrix: ρ' = (I + 0.4·X)/2 = [[0.5, 0.2], [0.2, 0.5]]</p>
<p>State fidelity with |+⟩: F = ⟨+|ρ'|+⟩ = (1 + λ_x·1)/2 = (1+0.4)/2 = 0.7</p>
<p>The state has degraded from pure |+⟩ (fidelity 1.0) to mixed state (fidelity 0.7). The dominant shrinkage along x shows that all four Pauli errors contribute to degrading the |+⟩ state.</p>
</div>

## 8.2 The Depolarising Channel: Deep Analysis

The depolarising channel is the most symmetric and most commonly used noise model in quantum computing. It models a process in which the qubit's quantum state is partially replaced by random noise in the most symmetric way — with equal probability of X, Y, or Z errors. It is the default noise model used in Qiskit's randomised benchmarking (RB) protocol and serves as the standard benchmark model for gate error characterisation.

### 8.2.1 Definition, Kraus Operators, and Bloch Sphere Action

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Depolarising Channel: Complete Specification</strong></p>
<p>The single-qubit depolarising channel with parameter p ∈ [0, 3/4] is defined by:</p>
<p>Bloch sphere action: The Bloch vector r contracts uniformly:</p>
<p>Fidelity of depolarised state with ideal input |ψ⟩: F = ⟨ψ|ε_D(|ψ⟩⟨ψ|)|ψ⟩ = 1 - 2p/3. This is the fidelity formula used in randomised benchmarking to extract the gate error rate.</p>
</div>

The equivalence between the two forms of the depolarising channel deserves careful derivation. Starting from the Pauli form and using the identity Σ\_P P·ρ·P = 2I for any density matrix ρ (where the sum is over {I,X,Y,Z}):

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>(1-p)ρ + (p/3)(XρX+YρY+ZρZ) = (1-p)ρ + (p/3)(2I-ρ)</p>
<p>= (1-p-p/3)ρ + (2p/3)I</p>
<p>= (1-4p/3)ρ + (4p/3)(I/2)  ✓</p>
</div>

This confirms the equivalence. The second form shows clearly that the depolarising channel is a convex combination of the identity (weight 1-4p/3) and the maximally mixed state (weight 4p/3). At the "depolarising threshold" p = 3/4, the channel outputs I/2 regardless of input — complete loss of quantum information.

### 8.2.2 Two-Qubit Depolarising Channel

For two-qubit gates (CX, CZ, iSWAP), noise affects both qubits simultaneously. The two-qubit depolarising channel applies a random two-qubit Pauli error {I,X,Y,Z}⊗{I,X,Y,Z} (15 non-identity elements) with equal probability p/15 each, leaving the state unchanged with probability (1-p):

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Two-Qubit Depolarising Channel</strong></p>
<p>Fidelity of output with ideal 2-qubit state: F = 1 - 16p/15. For p = 0.005 (0.5% two-qubit gate error): F = 1 - 16×0.005/15 = 1 - 0.00533 ≈ 0.9947 per gate.</p>
<p>In Qiskit Aer: depolarizing_error(p, num_qubits=2) creates the two-qubit depolarising channel with the 16-Kraus-operator form automatically.</p>
</div>

<figure class="book-figure">
<img src="content/images/image87.png" alt="Figure 8.2: Kraus operator analysis and noise error budget. Left: Action of the amplitude damping channel on |1⟩⟨1| as γ varies from 0 to 1 — the |1⟩ population (red) decreases as exp(−t/T1) while the |0⟩ population (blue) grows; off-diagonal coherences (green, dashed) for a |+⟩ input decay as √(1−γ). Centre: Circuit fidelity F=(1−p_total)^n vs circuit depth n for four representative noise levels — the log scale reveals that all fidelities converge to near zero beyond ~50–200 layers for NISQ-era parameters. Right: Typical error budget for a two-qubit CX gate on IBM Falcon (~0.40% total), showing gate calibration errors dominate, with T1/T2 and crosstalk as secondary contributions.">
<figcaption>Figure 8.2: Kraus operator analysis and noise error budget. Left: Action of the amplitude damping channel on |1⟩⟨1| as γ varies from 0 to 1 — the |1⟩ population (red) decreases as exp(−t/T1) while the |0⟩ population (blue) grows; off-diagonal coherences (green, dashed) for a |+⟩ input decay as √(1−γ). Centre: Circuit fidelity F=(1−p_total)^n vs circuit depth n for four representative noise levels — the log scale reveals that all fidelities converge to near zero beyond ~50–200 layers for NISQ-era parameters. Right: Typical error budget for a two-qubit CX gate on IBM Falcon (~0.40% total), showing gate calibration errors dominate, with T1/T2 and crosstalk as secondary contributions.</figcaption>
</figure>

### 8.2.3 Depolarising Channel in Randomised Benchmarking

Randomised benchmarking (RB) is the gold standard for characterising gate error rates in practice. The key insight is that for Clifford gates (the 24-element single-qubit Clifford group), the average error channel across all random Clifford gates approaches a depolarising channel — even if the actual noise on each gate is quite different from depolarising. This "twirling" property makes RB remarkably robust.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Randomised Benchmarking Protocol</strong></p>
<p>Standard RB protocol for single-qubit gates:</p>
<p>Prepare qubit in |0⟩.</p>
<p>Apply m random Clifford gates C₁, C₂, ..., Cₘ.</p>
<p>Compute and apply the inverse Cₘ₊₁ = (Cₘ···C₁)⁻¹.</p>
<p>Measure P(|0⟩). Repeat ~100 times for each m.</p>
<p>Sweep m from 1 to ~500. For each m, average P(|0⟩) over all random sequences.</p>
<p>Under the depolarising approximation, the survival probability decays as:</p>
<p>The error rate per Clifford r is extracted by fitting the decay curve. A single Clifford gate on IBM hardware averages ~1.5 native gates, so the error per native gate ≈ r/1.5.</p>
<p>Why RB is robust: It is insensitive to state preparation and measurement (SPAM) errors (captured by A and B), to slow drift (averaged over many sequences), and to coherent gate errors (which average to incoherent noise under Clifford randomisation). This makes RB more reliable than individual gate tomography for benchmarking production hardware.</p>
</div>

## 8.3 Amplitude and Phase Damping: Physical T1/T2 Channels

The depolarising channel is a useful model for benchmarking and RB analysis, but it is not physically accurate for superconducting or trapped-ion qubits. The physical noise processes are more specific: T1 energy relaxation (amplitude damping) and T2 dephasing (phase damping) operate via distinct physical mechanisms with distinct Kraus operators. Understanding these channels precisely is essential for designing noise-aware algorithms and for the Qiskit Aer thermal\_relaxation\_error model.

### 8.3.1 Amplitude Damping (T1) — Full Kraus Treatment

The amplitude damping channel models the spontaneous emission process: the qubit in excited state |1⟩ emits a photon (or phonon) and decays irreversibly to the ground state |0⟩. This is the dominant energy relaxation mechanism in superconducting qubits. At zero temperature (the relevant limit for qubits cooled to ~15 mK), excitation is one-directional: |1⟩ → |0⟩ only.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Amplitude Damping Channel: Full Specification</strong></p>
<p>Kraus operators at zero temperature (spontaneous emission only):</p>
<p>Action on a general density matrix ρ = [[ρ₀₀, ρ₀₁],[ρ₁₀, ρ₁₁]]:</p>
<p>Element-by-element interpretation:</p>
<p>ρ₁₁ → (1−γ)·ρ₁₁: excited state population decays by factor (1−γ) = exp(−t/T1)</p>
<p>ρ₀₀ → ρ₀₀ + γ·ρ₁₁: ground state gains the decayed excited population</p>
<p>ρ₀₁, ρ₁₀ → √(1−γ)·ρ₀₁: off-diagonal coherences decay by factor √(1−γ) = exp(−t/2T1)</p>
<p>This off-diagonal decay of √(1−γ) = exp(−t/2T1) is the T1 contribution to T2: it gives a 1/(2T1) contribution to the total dephasing rate 1/T2 = 1/(2T1) + 1/T_φ.</p>
</div>

#### Amplitude Damping on the Bloch Sphere

The amplitude damping channel acts asymmetrically on the Bloch sphere. Let r = (r\_x, r\_y, r\_z) be the input Bloch vector. The output is:

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>r_x' = √(1-γ)·r_x   (equatorial shrinkage)</p>
<p>r_y' = √(1-γ)·r_y   (equatorial shrinkage)</p>
<p>r_z' = (1-γ)·r_z + γ  (z-axis shift toward north pole)</p>
</div>

The z-component shift r\_z → (1−γ)r\_z + γ is crucial. This shifts the Bloch sphere centre toward the north pole (+z = |0⟩ ground state) by amount γ. For a qubit starting at the south pole r\_z = −1 (|1⟩ state): r\_z' = −(1−γ) + γ = −1 + 2γ. At γ=1: r\_z' = +1 (north pole, |0⟩). The decay is always toward the ground state — this is thermalization at T=0. The channel is NOT unital (does not preserve the maximally mixed state): ε\_AD(I/2) ≠ I/2.

<div class="box box-example">
<p class="box-title"><strong>Example 8.2: Amplitude Damping on |+⟩ State</strong></p>
<p>Problem: Apply the amplitude damping channel with γ = 0.2 to the state |+⟩ = (|0⟩+|1⟩)/√2. Find the output density matrix and Bloch vector. Compute the fidelity of the output with |+⟩.</p>
<p><strong>Solution:</strong></p>
<p>Input: ρ = |+⟩⟨+| = (1/2)[[1,1],[1,1]]. Bloch vector: r = (1,0,0).</p>
<p>Output density matrix using the formula:</p>
<p>ρ₀₀' = ρ₀₀ + γ·ρ₁₁ = 0.5 + 0.2×0.5 = 0.6</p>
<p>ρ₁₁' = (1−γ)·ρ₁₁ = 0.8×0.5 = 0.4</p>
<p>ρ₀₁' = √(1−γ)·ρ₀₁ = √0.8×0.5 = 0.4472×0.5 = 0.2236</p>
<p>ρ' = [[0.6, 0.2236],[0.2236, 0.4]]</p>
<p>Bloch vector: r_x' = √0.8×1 = 0.894; r_y' = 0; r_z' = 0.8×0 + 0.2 = 0.2</p>
<p>|r'| = √(0.894² + 0 + 0.2²) = √(0.799 + 0.04) = √0.839 ≈ 0.916 &lt; 1 (mixed state)</p>
<p>Fidelity: F = ⟨+|ρ'|+⟩ = (1/2)(ρ₀₀'+ρ₁₁'+ρ₀₁'+ρ₁₀') = (0.6+0.4+2×0.2236)/2 = 1.4472/2 ≈ 0.724</p>
<p>The fidelity with |+⟩ dropped from 1.0 to 0.724 due to amplitude damping. Notice the state is now mixed (Bloch vector length &lt; 1) and has acquired a positive z-component — it has partially relaxed toward the ground state.</p>
</div>

### 8.3.2 Phase Damping (T2) — Pure Dephasing Channel

The phase damping channel (also called dephasing or transverse relaxation) models the loss of phase coherence between |0⟩ and |1⟩ without any energy exchange. It is the quantum channel corresponding to the T\_φ contribution in the T2 decomposition (1/T2 = 1/2T1 + 1/T\_φ). Unlike amplitude damping, phase damping is symmetric between |0⟩ and |1⟩ — both are ground states from an energy perspective, but they differ in phase.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Phase Damping Channel</strong></p>
<p>The phase damping channel is defined by Kraus operators:</p>
<p>Action on density matrix ρ = [[ρ₀₀, ρ₀₁],[ρ₁₀, ρ₁₁]]:</p>
<p>Bloch sphere action: r_x, r_y → √(1-λ)·r_x, √(1-λ)·r_y; r_z → r_z.</p>
<p>The Bloch sphere is compressed in the equatorial plane (x-y) while the z-axis is preserved. This is a unital channel (preserves I/2): ε_PD(I/2) = I/2 ✓.</p>
<p>Relationship to T2: The combined T1+T2 channel (amplitude damping + phase damping in sequence) gives total off-diagonal decay exp(−t/T2) = exp(−t/2T1)·exp(−t/T_φ), consistent with 1/T2 = 1/(2T1) + 1/T_φ.</p>
</div>

The physical origin of pure dephasing in superconducting qubits is fluctuating qubit frequency. If the qubit frequency ω₀₁ fluctuates with a random process δω(t), the qubit phase evolves as φ(t) = ∫₀ᵗ δω(t')dt'. After averaging over many repetitions, the off-diagonal density matrix element decays as ρ₀₁(t) = ρ₀₁(0)·⟨exp(iφ(t))⟩ = ρ₀₁(0)·exp(−t/T\_φ) for Gaussian noise with Lorentzian spectrum. The T\_φ parameter depends on the noise power spectral density at low frequencies.

### 8.3.3 Combined T1+T2 Channel (Generalised Amplitude Damping)

In real hardware, T1 and T2 processes occur simultaneously and continuously. The Qiskit Aer thermal\_relaxation\_error function combines both into a single channel that models the exact physical evolution for a qubit with given T1, T2, at a given temperature, for a gate of given duration.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Combined T1+T2 Channel: Kraus Operators</strong></p>
<p>For a qubit at finite temperature T (with excited state population p_excited = exp(-ħω/k_BT)/(1+exp(-ħω/k_BT)) ≈ 0 at 15 mK), the combined T1+T2 channel has 3 Kraus operators (for the zero-temperature case):</p>
<p>Completeness verification: K₀†K₀ + K₁†K₁ + K₂†K₂</p>
<p>This 3-Kraus form is exactly what Qiskit Aer's thermal_relaxation_error implements internally (for the zero-temperature limit). At finite temperature, two additional Kraus operators appear for |0⟩→|1⟩ thermal excitation.</p>
</div>

## 8.4 Qiskit Aer Noise Simulation: Complete Toolkit

Qiskit Aer provides the most comprehensive open-source quantum noise simulation framework available. Its noise model system is both physically rigorous and practically convenient: it can simulate anything from a simple depolarising error on all gates to a fully calibrated noise model reconstructed from real IBM hardware calibration data. In this section we develop the complete toolkit, progressing from simple to complex.

### 8.4.1 NoiseModel Architecture and Gate Error Assignment

A NoiseModel object is a collection of quantum error channels (QuantumError objects) assigned to specific gates and qubits. When AerSimulator runs a circuit with a NoiseModel, it inserts the specified error channel after each gate matching the assignment. The error channel is specified as a Kraus operator set, which Aer samples stochastically during simulation (for density matrix or stabiliser methods) or applies exactly (for statevector+Kraus methods).

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: NoiseModel Assignment Methods</strong></p>
<p>The four key methods for assigning errors to a NoiseModel:</p>
<p>Error composition: Multiple errors can be composed (applied in sequence) using error1.compose(error2), or tensored (applied to different qubits) using error1.tensor(error2) or error1.expand(error2). This allows combining thermal relaxation with depolarising calibration errors.</p>
</div>

<figure class="book-figure">
<img src="content/images/image88.png" alt="Figure 8.3: Error characterisation and mitigation techniques. Left: Randomised benchmarking decay curves for four hardware generations — survival probability P(|0⟩) vs Clifford gate count m, fitted to A·p^m + B. The extracted error rate r = 1 − p gives average Clifford gate fidelity. Centre: Zero-Noise Extrapolation (ZNE) — the observable ⟨Z⟩ is measured at noise scaling factors λ = 1, 1.5, 2, 2.5, 3 (by gate folding) and extrapolated to λ=0 (zero noise) using polynomial fit. The star marks the ZNE-corrected estimate, close to the true ideal value (dashed). Right: Two-qubit readout calibration matrix M showing P(measured state | prepared state) for 2% per-qubit readout error — off-diagonal elements are the error probabilities that must be inverted to mitigate SPAM errors.">
<figcaption>Figure 8.3: Error characterisation and mitigation techniques. Left: Randomised benchmarking decay curves for four hardware generations — survival probability P(|0⟩) vs Clifford gate count m, fitted to A·p^m + B. The extracted error rate r = 1 − p gives average Clifford gate fidelity. Centre: Zero-Noise Extrapolation (ZNE) — the observable ⟨Z⟩ is measured at noise scaling factors λ = 1, 1.5, 2, 2.5, 3 (by gate folding) and extrapolated to λ=0 (zero noise) using polynomial fit. The star marks the ZNE-corrected estimate, close to the true ideal value (dashed). Right: Two-qubit readout calibration matrix M showing P(measured state | prepared state) for 2% per-qubit readout error — off-diagonal elements are the error probabilities that must be inverted to mitigate SPAM errors.</figcaption>
</figure>

### 8.4.2 thermal\_relaxation\_error: T1/T2 Simulation

The thermal\_relaxation\_error function is the physically most accurate noise model for superconducting qubits. It constructs the combined T1+T2 Kraus channel for a gate of specified duration, exactly implementing the 3-Kraus model derived in Section 8.3.3.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 8.1 — Complete T1/T2 Noise Simulation with thermal_relaxation_error
# Models physically realistic qubit decoherence for superconducting devices.
# Includes: T1 relaxation + T2 dephasing + depolarising gate calibration errors.
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit_aer.noise import (NoiseModel, thermal_relaxation_error,
                               depolarizing_error, ReadoutError)
from qiskit.quantum_info import DensityMatrix, state_fidelity, Statevector
import numpy as np

# ── Hardware parameters (IBM Eagle, typical qubit) ──────────────────────
T1      = 200e-6    # T1 = 200 μs (amplitude relaxation)
T2      = 120e-6    # T2 = 120 μs (total dephasing; must satisfy T2 ≤ 2T1)
t_1q    = 50e-9     # Single-qubit gate duration
t_cx    = 300e-9    # CX gate duration
t_meas  = 800e-9    # Measurement window duration

# Derived: pure dephasing time T_phi from 1/T2 = 1/(2T1) + 1/T_phi
T_phi = 1.0 / (1.0/T2 - 1.0/(2*T1))
print(f'Pure dephasing T_phi = {T_phi*1e6:.1f} μs')

# ── Create thermal relaxation error channels ────────────────────────────
# Internally creates 3 Kraus operators: K0, K1 (T1), K2 (T_phi)
err_th_1q = thermal_relaxation_error(T1, T2, t_1q)    # for 1Q gates
err_th_cx = thermal_relaxation_error(T1, T2, t_cx)    # for CX gate (per qubit)
err_th_m  = thermal_relaxation_error(T1, T2, t_meas)  # for measurement

# ── Add depolarising calibration errors (separate from coherence) ───────
err_dep_1q = depolarizing_error(3e-4, 1)    # 0.03% 1Q calibration error
err_dep_cx = depolarizing_error(3e-3, 2)    # 0.30% CX calibration error

# Compose: apply thermal relaxation THEN depolarising
err_1q_full = err_th_1q.compose(err_dep_1q)
# For 2Q gates: tensor the 1Q thermal errors (both qubits decohere), then compose 2Q depo
err_cx_full = err_th_cx.expand(err_th_cx).compose(err_dep_cx)

# ── Build the complete noise model ──────────────────────────────────────
nm = NoiseModel()
nm.add_all_qubit_quantum_error(err_1q_full, ['h','x','y','z','s','sdg','t','tdg','rx','ry','rz'])
nm.add_all_qubit_quantum_error(err_cx_full, ['cx', 'cz'])

# Readout error: 1.5% SPAM error per qubit
p_ro = 0.015
ro_err = ReadoutError([[1-p_ro, p_ro], [p_ro, 1-p_ro]])
nm.add_all_qubit_readout_error(ro_err)

# ── Simulate GHZ state fidelity ─────────────────────────────────────────
n_qubits = 4
qc_ghz = QuantumCircuit(n_qubits)
qc_ghz.h(0)
for i in range(n_qubits-1): qc_ghz.cx(i, i+1)
qc_ghz.save_density_matrix()

# Ideal GHZ state
ideal_ghz = Statevector.from_label("0"*n_qubits).evolve(qc_ghz)

# Noisy simulation
sim = AerSimulator(method="density_matrix", noise_model=nm)
result = sim.run(transpile(qc_ghz, sim)).result()
dm_noisy = DensityMatrix(result.data()["density_matrix"])

# Ideal simulation
sim0 = AerSimulator(method="statevector")
dm_ideal = DensityMatrix(ideal_ghz)

F = state_fidelity(dm_noisy, dm_ideal)
print(f'{n_qubits}-qubit GHZ fidelity (T1/T2+depolarising): {F:.4f}')
# Expected output: ~0.90-0.93 for 4-qubit GHZ with 3 CX gates and the above parameters

# ── Scan fidelity vs circuit depth (T1/T2 noise) ────────────────────────
depths = [1, 2, 3, 5, 8, 12]
fidelities = []
for d in depths:
    qc_d = QuantumCircuit(2)
    for _ in range(d): qc_d.cx(0,1); qc_d.cx(1,0)
    qc_d.save_density_matrix()
    dm_d = DensityMatrix(sim.run(transpile(qc_d,sim)).result().data()["density_matrix"])
    dm_i = DensityMatrix(AerSimulator(method="density_matrix").run(transpile(qc_d,AerSimulator(method="density_matrix"))).result().data()["density_matrix"])
    fidelities.append(state_fidelity(dm_d, dm_i))
    print(f'Depth {d*2:2d} CX gates: F = {fidelities[-1]:.4f}')
```

### 8.4.3 depolarizing\_error, pauli\_error, and ReadoutError

Beyond thermal relaxation, Qiskit Aer supports several other QuantumError types that cover the full range of hardware noise. Each is appropriate for different noise characterisation scenarios.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 8.2 — depolarizing_error, pauli_error, ReadoutError: Full Examples
# ─────────────────────────────────────────────────────────────────────────
from qiskit_aer.noise import (NoiseModel, depolarizing_error,
                               pauli_error, ReadoutError, coherent_unitary_error)
import numpy as np

# ── 1. Depolarising error (standard RB / benchmarking model) ────────────
err_1q_depo = depolarizing_error(0.002, 1)    # 0.2% error, 1-qubit gate
err_2q_depo = depolarizing_error(0.005, 2)    # 0.5% error, 2-qubit gate
print('1Q depo Kraus count:', len(err_1q_depo.to_kraus()))  # should print 4

# ── 2. Pauli error (asymmetric Pauli noise from RB characterisation) ─────
# Specify probability of each Pauli error individually
# Useful when X, Y, Z errors have different rates (non-depolarising)
err_pauli_1q = pauli_error([('X', 0.003),   # 0.3% bit-flip error
                            ('Z', 0.005),   # 0.5% phase-flip error
                            ('Y', 0.001),   # 0.1% combined error
                            ('I', 0.991)])  # 99.1% no error

# Verify: probabilities sum to 1
print(f'Pauli error probs sum: {0.003+0.005+0.001+0.991}')  # 1.000

# ── 3. ReadoutError (SPAM — state preparation and measurement) ───────────
# Assignment matrix: [[P(0|0), P(1|0)], [P(0|1), P(1|1)]]
# Different error rates for 0→1 vs 1→0 misclassification
err_ro = ReadoutError([[0.985, 0.015],   # 98.5% correct when |0>; 1.5% flip to 1
                       [0.025, 0.975]])  # 97.5% correct when |1>; 2.5% flip to 0
print('Readout assignment matrix:')
print(np.array(err_ro.probabilities))

# ── 4. Coherent error (miscalibrated rotation) ───────────────────────────
# Over-rotation by epsilon (coherent gate error, not decoherence)
epsilon = 0.02    # 1.15° over-rotation in pi-pulse
theta_actual = np.pi + epsilon    # intended: pi-pulse; actual: pi + 0.02
# Coherent over-rotation about X axis:
U_overrot = np.array([[np.cos(theta_actual/2), -1j*np.sin(theta_actual/2)],
                       [-1j*np.sin(theta_actual/2), np.cos(theta_actual/2)]])
# Ideal X gate:
U_ideal = np.array([[0, -1j], [-1j, 0]])
# Error gate: U_err = U_actual @ U_ideal_dagger
U_err = U_overrot @ U_ideal.conj().T
err_coherent = coherent_unitary_error(U_err)

# ── 5. Build a mixed noise model using all error types ───────────────────
nm_full = NoiseModel()
nm_full.add_all_qubit_quantum_error(err_1q_depo.compose(err_pauli_1q), ['h', 'x', 'rx', 'ry'])
nm_full.add_all_qubit_quantum_error(err_2q_depo, ['cx', 'cz'])
nm_full.add_all_qubit_readout_error(err_ro)

# ── 6. Demonstrate readout error mitigation (calibration matrix inversion)
from qiskit import QuantumCircuit

# Build calibration circuits: one per computational basis state
cal_states = ['00', '01', '10', '11']
cal_counts = {}
shots = 10000

# Simulate each calibration circuit (prepare state, measure immediately)
# In practice: submit these to real hardware to get the actual cal matrix
print("\nReadout mitigation: calibration matrix approach")
print("1. Run calibration circuits on hardware")
print("2. Build M[i,j] = P(measured i | prepared j)")
print("3. For new measurements: corrected_counts = M_inverse @ raw_counts")
```

### 8.4.4 Backend-Based and Custom Noise Models

For the most accurate simulation of a specific IBM device, Qiskit provides the ability to build a NoiseModel directly from backend calibration data. The FakeProvider backends (FakeSherbrookeV2, FakeKyotoV2, etc.) contain frozen calibration data from real IBM devices, enabling realistic simulation without queue time.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 8.3 — Backend-Based Noise Model from IBM Calibration Data
# Uses FakeSherbrookeV2 (127-qubit Eagle R3) calibration data.
# ─────────────────────────────────────────────────────────────────────────
from qiskit_ibm_runtime.fake_provider import FakeSherbrookeV2
from qiskit_aer import AerSimulator
from qiskit_aer.noise import NoiseModel
from qiskit import QuantumCircuit, transpile
from qiskit.quantum_info import Statevector

# ── Load fake backend (frozen IBM hardware calibration snapshot) ─────────
backend = FakeSherbrookeV2()

# Auto-build noise model from calibration data
# Includes: per-qubit T1, T2, gate times, gate error rates, readout errors
noise_model = NoiseModel.from_backend(backend)

# Inspect the noise model
print('Backend name:', backend.name)
print('Number of qubits:', backend.num_qubits)
print('Basis gates with errors:', noise_model.basis_gates)
print('Number of noisy qubits:', len(noise_model.noise_qubits))

# ── Build calibration-matched simulator ──────────────────────────────────
sim_noisy = AerSimulator.from_backend(backend)   # alternative: includes coupling map
# OR: AerSimulator(noise_model=noise_model)

# ── Run a benchmark: compare ideal vs calibrated noise for Bell state ────
qc_bell = QuantumCircuit(2, 2)
qc_bell.h(0)
qc_bell.cx(0, 1)
qc_bell.measure_all()

# Transpile to backend's native gate set and coupling map
qc_t = transpile(qc_bell, sim_noisy, optimization_level=3)
print(f'Bell circuit depth after transpile: {qc_t.depth()}')
print(f'CX gate count after transpile: {qc_t.count_ops().get("cx",0)}')

# Ideal simulation
result_ideal = AerSimulator().run(transpile(qc_bell, AerSimulator()), shots=10000).result()
counts_ideal = result_ideal.get_counts()

# Noisy simulation
result_noisy = sim_noisy.run(qc_t, shots=10000).result()
counts_noisy = result_noisy.get_counts()

print(f'Ideal Bell counts: {counts_ideal}')
print(f'Noisy Bell counts: {counts_noisy}')

# Compute state fidelity from counts (approximate)
n_corr = counts_noisy.get("00",0) + counts_noisy.get("11",0)
print(f'Bell correlation fidelity: {n_corr/10000:.4f}')
# Expected: ~0.97-0.99 (high fidelity for just 1 CX gate on best qubits)
```

### 8.4.5 Error Mitigation: Zero-Noise Extrapolation and Readout Correction

Error mitigation techniques reduce the effective noise in quantum computations without requiring full quantum error correction. They are practical tools for NISQ-era algorithms that improve output accuracy at the cost of additional classical or quantum overhead. The two most widely used techniques are Zero-Noise Extrapolation (ZNE) and readout error mitigation.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Zero-Noise Extrapolation (ZNE)</strong></p>
<p>ZNE assumes that the noisy expectation value ⟨O⟩(λ) is a smooth function of a noise scaling parameter λ, with the ideal value at λ=0:</p>
<p>Implementation via gate folding: To scale noise by factor λ &gt; 1, replace each gate G with G·G†·G·G†···G (λ repetitions of the gate and its inverse). This multiplies the gate time (and hence coherence decay) by λ while keeping the ideal unitary action unchanged (G†G = I). Run the circuit at λ = 1, 1.5, 2, 2.5, 3 and fit the observable vs λ, then extrapolate to λ=0.</p>
<p>Overhead: ZNE requires approximately 2K circuit executions (K noise levels × same number of shots). For K=3, overhead is 3× total shots. Accuracy improves with more noise levels but the extrapolation error grows with noise.</p>
<p>Limitations: ZNE assumes the noise can be scaled smoothly (not always true for non-Markovian or coherent noise). It does not work if the circuit fidelity at high λ is too low (signal buried in noise). Effective for reducing errors by 2–5× on shallow NISQ circuits.</p>
</div>

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 8.4 — Zero-Noise Extrapolation (ZNE) using Qiskit IBM Runtime
# Demonstrates manual ZNE via gate folding and polynomial extrapolation.
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit_aer.noise import NoiseModel, depolarizing_error
from qiskit.quantum_info import SparsePauliOp
import numpy as np
from scipy.optimize import curve_fit

# ── Define the circuit and target observable ─────────────────────────────
def build_circuit(n_folds):
    """Bell state circuit with gate folding for ZNE."""
    qc = QuantumCircuit(2)
    # Base gate sequence: H + CX
    qc.h(0)
    qc.cx(0, 1)
    # Gate folding: repeat G†G (n_folds-1) additional times
    for _ in range(n_folds - 1):
        qc.cx(0, 1)  # G†
        qc.cx(0, 1)  # G (net effect: identity)
    qc.save_density_matrix()
    return qc

# ── Noise model (0.5% CX depolarising) ──────────────────────────────────
nm_zne = NoiseModel()
nm_zne.add_all_qubit_quantum_error(depolarizing_error(0.005, 2), ["cx"])
sim_zne = AerSimulator(method="density_matrix", noise_model=nm_zne)

# ── Run at different noise scaling factors ───────────────────────────────
scale_factors = [1, 2, 3, 4, 5]   # λ = 1, 2, 3, 4, 5 (odd folds only)
ZZ_values = []   # measure ZZ = Z₀⊗Z₁

for n_folds in scale_factors:
    qc = build_circuit(n_folds)
    result = sim_zne.run(transpile(qc, sim_zne)).result()
    from qiskit.quantum_info import DensityMatrix, Statevector
    dm = DensityMatrix(result.data()["density_matrix"])
    # ZZ expectation = ⟨00|ρ|00⟩ + ⟨11|ρ|11⟩ - ⟨01|ρ|01⟩ - ⟨10|ρ|10⟩
    ZZ = (dm.data[0,0] + dm.data[3,3] - dm.data[1,1] - dm.data[2,2]).real
    ZZ_values.append(ZZ)
    print(f'  λ={n_folds}: ⟨ZZ⟩ = {ZZ:.4f}')

# ── ZNE: polynomial extrapolation to λ=0 ─────────────────────────────────
# Fit ⟨ZZ⟩(λ) = a + b*λ + c*λ² (quadratic)
coeffs = np.polyfit(scale_factors, ZZ_values, 2)
ZZ_zne = np.polyval(coeffs, 0)   # extrapolate to λ=0

# Ideal value: Bell state has ⟨ZZ⟩ = 1.0
ZZ_ideal = 1.0
ZZ_raw   = ZZ_values[0]   # unmitigated (λ=1)

print(f'\nIdeal ⟨ZZ⟩:       {ZZ_ideal:.4f}')
print(f'Raw (noisy) ⟨ZZ⟩:  {ZZ_raw:.4f}')
print(f'ZNE estimate:      {ZZ_zne:.4f}')
print(f'ZNE error:         {abs(ZZ_zne-ZZ_ideal):.4f}  (vs raw error: {abs(ZZ_raw-ZZ_ideal):.4f})')
# ZNE should reduce the error by ~2-5× for this simple case
```

## 8.5 Quantum Volume: Theory and Measurement Protocol

Quantum Volume (QV) is the most widely adopted single-number benchmark for quantum hardware quality. Introduced by IBM in 2019 (Cross et al., Physical Review A), it is designed to answer the practical question: "What is the largest computation this processor can reliably perform?" — where "computation" is defined as a square random quantum circuit of equal width (qubits) and depth (layers).

QV is valuable precisely because it is a composite metric: a processor cannot achieve high QV by excelling in only one dimension. High qubit count alone is not enough (if fidelity is poor). High gate fidelity alone is not enough (if connectivity requires excessive SWAP routing). High connectivity alone is not enough (if readout errors are too large). QV naturally captures the interplay of all these factors.

### 8.5.1 QV Definition and Heavy-Output Generation Probability

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Quantum Volume: Formal Definition</strong></p>
<p>The Quantum Volume of a quantum computer is:</p>
<p>Heavy-Output Generation Probability (HOGP): For a random n-qubit circuit U, the ideal output probability distribution P_ideal(x) has median value m. The heavy outputs are bitstrings x with P_ideal(x) &gt; m. Exactly 2^n/2 = half of all bitstrings are heavy outputs.</p>
<p>The heavy-output probability of the ideal distribution is:</p>
<p>The hardware HOGP is: h_hw = Σ_{x: P_ideal(x)&gt;m} P_hw(x), where P_hw(x) is the measured probability from hardware execution. If the hardware perfectly replicates the ideal distribution, h_hw = h_ideal ≈ 0.818. With noise, h_hw decreases toward 0.5.</p>
<p>QV pass threshold: A processor achieves QV = 2^n if h_hw &gt; 2/3 for all 100 random n-qubit circuits with 97.5% statistical confidence (one-sided z-test).</p>
</div>

The 2/3 threshold is chosen to lie between the random baseline (0.5) and the ideal quantum value (~0.818), requiring the hardware to be closer to ideal than to random. The 97.5% confidence requirement ensures statistical rigour: with 100 circuits and typically ~1000 shots per circuit, the sample estimate of h\_hw must be above 2/3 with high confidence.

### 8.5.2 QV Measurement Protocol — Step-by-Step

We now describe the full QV protocol as implemented in practice by IBM Quantum and as reproducible using Qiskit Experiments.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: QV Protocol: Complete Implementation</strong></p>
<p>Input: A quantum processor P with N physical qubits. Target: find maximum n for which QV = 2^n.</p>
<p>Select n qubits: Choose the best n qubits from the processor based on T1, T2, and gate error rates. For n=6 on a 127-qubit device, this is the 6 qubits with highest fidelity.</p>
<p>Generate circuits: Create 100 random "model circuits" of n qubits × n layers. Each layer consists of: (a) a uniformly random permutation of the n qubits into ⌊n/2⌋ pairs, (b) a uniformly random SU(4) gate applied to each pair (sampled from the Haar measure on SU(4)). This is the QV circuit structure.</p>
<p>Classical simulation: For each circuit, simulate the ideal output distribution P_ideal(x) for all 2^n bitstrings using a classical simulator (this is feasible up to n≈20 on a standard computer). Compute the median m and identify the heavy-output set.</p>
<p>Transpile: Transpile each circuit to the native gate set and qubit connectivity of the target processor at the highest optimisation level. Record the final circuit depth.</p>
<p>Execute on hardware: Run each transpiled circuit with ≥1000 shots (more is better for statistical confidence). Record the outcome counts.</p>
<p>Compute HOGP: For each circuit, compute h_hw = (number of shots hitting heavy outputs) / (total shots). Average h_hw over all 100 circuits.</p>
<p>Statistical test: Compute the mean h̄_hw and standard error σ = √(h̄(1-h̄)/N_shots). Test if h̄_hw − 2·σ &gt; 2/3 (lower confidence bound above threshold). If yes, the processor achieves QV ≥ 2^n. Try n+1.</p>
</div>

<figure class="book-figure">
<img src="content/images/image89.png" alt="Figure 8.4: NISQ algorithm landscape and hardware scaling roadmap. Left: Log-log plot of required circuit depth vs qubit count for representative quantum algorithms. The NISQ-reachable region (shaded blue, below the ~230-gate depth limit) contains shallow VQE and QAOA circuits; the fault-tolerant region (shaded red, above ~230 gates) includes deep QFT and Shor&#x27;s algorithm. Right: Historical and projected qubit counts for IBM (blue) and Google (red) from 2019 to 2033 on a log scale. IBM&#x27;s roadmap targets &gt;100,000 physical qubits by 2030 via modular quantum-centric supercomputing; Google&#x27;s Willow processor achieved 105 qubits in 2024 with sub-threshold error correction.">
<figcaption>Figure 8.4: NISQ algorithm landscape and hardware scaling roadmap. Left: Log-log plot of required circuit depth vs qubit count for representative quantum algorithms. The NISQ-reachable region (shaded blue, below the ~230-gate depth limit) contains shallow VQE and QAOA circuits; the fault-tolerant region (shaded red, above ~230 gates) includes deep QFT and Shor&#x27;s algorithm. Right: Historical and projected qubit counts for IBM (blue) and Google (red) from 2019 to 2033 on a log scale. IBM&#x27;s roadmap targets &gt;100,000 physical qubits by 2030 via modular quantum-centric supercomputing; Google&#x27;s Willow processor achieved 105 qubits in 2024 with sub-threshold error correction.</figcaption>
</figure>

### 8.5.3 QV Across Platforms and Hardware Generations

Quantum Volume has become the de facto standard for comparing quantum hardware, with results reported by IBM, IonQ, Quantinuum, Rigetti, and others. The progression reveals both the rapid improvement in hardware quality and the fundamental differences between qubit platforms.

| Company/Platform | Processor | QV (Year) | Physical Qubits (at QV test) | Key Enabling Factor |
|---|---|---|---|---|
| IBM Quantum | Falcon r4 | 8 (2019) | 5 of 27 | First QV measurement; proof of concept |
| IBM Quantum | Falcon r5 | 64 (2020) | 5 of 27 | CNOT gate improvement; better calibration |
| IBM Quantum | Eagle r1 | 128 (2021) | 5 of 127 | Heavy-hex topology reduces crosstalk |
| IBM Quantum | Eagle r3 | 512 (2022) | 5 of 127 | Improved T1/T2; better pulse calibration |
| IBM Quantum | Heron r1 | 2048 (2023) | 12 of 133 | Cross-resonance gate refinement; no ZZ crosstalk |
| IBM Quantum | Heron r2 | 4096 (2024) | 12 of 133 | New coupler design; T1>500μs on best qubits |
| IonQ | Forte | 512 (2022) | 9 of 32 | All-to-all connectivity; 99.9% 2Q fidelity |
| Quantinuum | H1-1 | 2048 (2021) | 10 of 20 | QCCD architecture; 99.8% 2Q gate fidelity |
| Quantinuum | H2-2 | 65536 (2024) | 12 of 56 | Mid-circuit measurement; highest QV recorded |

The most striking entry in this table is Quantinuum H2-2 reporting QV = 65,536 = 2^16 in 2024. This means their processor reliably executes 16-qubit, 16-layer random circuits — a feat requiring ~128 entangling gates with high fidelity. This is possible because trapped-ion qubits with all-to-all connectivity and 99.8% two-qubit fidelity require minimal SWAP routing overhead. The QV test uses only 12 of the 56 available qubits, selecting the best-performing subset.

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: QV and India's Quantum Computing Programmes</strong></p>
<p>India's National Quantum Mission has set an explicit target: indigenous quantum processors with 50–1,000 qubits and QV &gt; 100 by 2028. The Technology Innovation Hub on Quantum Communication at IIT Madras is developing superconducting qubits; the National Centre of Excellence in Quantum Technologies at IISc focuses on superconducting and photonic platforms. The QV metric provides a clear, internationally comparable benchmark for assessing progress of these programmes.</p>
<p>For M.Sc. Physics students: understanding QV is directly relevant for evaluating research proposals, reading hardware benchmarking papers, and designing algorithms that work within the QV-implied capability of available hardware. The ability to compute QV-equivalent circuit estimates (Section 8.5.2) is a practical skill for quantum algorithm developers.</p>
</div>

## 8.6 CLOPS, Other Benchmarks, and NISQ Limitations

### 8.6.1 CLOPS Definition and Practical Impact

While Quantum Volume measures the quality (how reliably circuits execute), CLOPS (Circuit Layer Operations Per Second) measures throughput — how fast circuits are processed from submission to result in a real workload. Introduced by IBM in 2021, CLOPS captures the full system performance including gate execution, classical control latency, compilation overhead, and cloud interface delays. For variational algorithms like VQE and QAOA that require thousands of circuit executions per optimisation run, CLOPS is often the dominant factor determining practical wall-clock time.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: CLOPS: Definition and Calculation</strong></p>
<p>Worked example: A processor runs M=100 templates × K=10 updates × S=100 shots × D=8 layers (QV=256) in T=25 seconds total: CLOPS = (100×10×100×8)/25 = 3,200 CLOPS.</p>
<p>The CLOPS formula counts the total number of (circuit layer × shot) operations — measuring how many QV-circuit-equivalent layers are completed per second in a realistic workload.</p>
</div>

The practical significance of CLOPS is enormous for variational algorithms. A QAOA circuit with D=12 layers running 500 optimiser iterations × 2,000 shots each requires 500×2,000×12 = 12,000,000 circuit-layer executions. At 15,000 CLOPS (IBM Heron R2): 12,000,000/15,000 = 800 seconds ≈ 13 minutes. At 850 CLOPS (IBM Falcon): 12,000,000/850 = 14,118 seconds ≈ 3.9 hours. The 17× CLOPS improvement makes the difference between a practical daily workflow and an overnight job.

### 8.6.2 Error Rates, Gate Benchmarks, and Other Metrics

Beyond QV and CLOPS, several other benchmarks are used to characterise specific aspects of hardware performance. A complete hardware specification reports multiple metrics.

<figure class="book-figure">
<img src="content/images/image90.png" alt="Figure 8.5: Quantum error correction — surface code and fault-tolerance threshold. Left: Surface code (d=3) lattice diagram — 9 data qubits (blue circles, labelled D) form the computational lattice; X-stabiliser ancillas (green squares) detect Z-type errors; Z-stabiliser ancillas (orange squares) detect X-type errors. The logical qubit is encoded in the collective state of all 9 data qubits. Centre: Logical error rate p_L vs physical error rate p for code distances d=3,5,7,9. Below the ~1% fault-tolerance threshold, increasing d exponentially suppresses p_L. Above threshold, QEC makes things worse. Right: Physical qubits required for RSA-2048 Shor&#x27;s algorithm vs physical error rate — ranging from 20 million qubits at p=0.1% to 1.5 million at p=0.01%.">
<figcaption>Figure 8.5: Quantum error correction — surface code and fault-tolerance threshold. Left: Surface code (d=3) lattice diagram — 9 data qubits (blue circles, labelled D) form the computational lattice; X-stabiliser ancillas (green squares) detect Z-type errors; Z-stabiliser ancillas (orange squares) detect X-type errors. The logical qubit is encoded in the collective state of all 9 data qubits. Centre: Logical error rate p_L vs physical error rate p for code distances d=3,5,7,9. Below the ~1% fault-tolerance threshold, increasing d exponentially suppresses p_L. Above threshold, QEC makes things worse. Right: Physical qubits required for RSA-2048 Shor&#x27;s algorithm vs physical error rate — ranging from 20 million qubits at p=0.1% to 1.5 million at p=0.01%.</figcaption>
</figure>

| Metric | What It Measures | Typical Values (IBM Heron 2024) | How It's Measured |
|---|---|---|---|
| QV | Overall circuit quality (fidelity × depth × connectivity) | 4096 (n_eff=12) | HOGP on random square circuits |
| CLOPS | Throughput: circuit layers per second end-to-end | 15,000 | QV-protocol timing benchmark |
| 1Q Gate Fidelity | Average single-qubit gate error rate | 99.95% (err < 0.05%) | Randomised benchmarking (RB) |
| 2Q Gate Fidelity (CX) | Two-qubit entangling gate error rate | 99.7% (err ~0.3%) | Interleaved RB or gate tomography |
| T1 (median) | Energy relaxation time | ~200 μs (best ~500 μs) | Inversion recovery |
| T2 (median) | Total dephasing time | ~150 μs | Ramsey / Hahn echo |
| Readout Fidelity | P(correct) on measurement | 98–99.5% | Prepare \|0⟩ / \|1⟩, measure |
| SPAM Error | State prep + measurement combined | 1–2% | Calibration matrix method |
| Cross-entropy Benchmarking | Circuit fidelity vs ideal (Google metric) | via RCS circuits | Random circuit sampling + XEB |
| Leakage | Rate of \|0⟩/\|1⟩ → \|2⟩ transitions | <0.05% per gate | Leakage RB protocol |

### 8.6.3 NISQ Era: Limitations and the Path Forward

The NISQ era is not a temporary inconvenience but a fundamental phase of quantum computing development. Understanding its precise limitations — and distinguishing what is achievable from what is not — is essential for every quantum computing student and practitioner.

#### The Fault-Tolerance Threshold

Quantum error correction works only if the physical error rate p is below the fault-tolerance threshold p\_th. For the surface code (the leading QEC code for practical hardware), the threshold is approximately p\_th ≈ 1%. This means: if each physical qubit operation has error rate < 1%, and you have enough physical qubits (roughly 1,000 per logical qubit), you can build a logical qubit with arbitrarily low error rate. Below threshold, adding more physical qubits per logical qubit exponentially reduces the logical error rate:

<div class="box box-equation">
<p><strong>Equation 8.18</strong></p>
<p><strong>p_L ≈ (p / p_th)^((d+1)/2)  [below threshold, surface code distance d]</strong></p>
<p>For p=0.003 (0.3%), p_th=0.01: p_L ≈ (0.3)^((d+1)/2)</p>
<p>At d=5: p_L ≈ (0.3)^3 = 0.027    [still 2.7% logical error — not good enough]</p>
<p>At d=9: p_L ≈ (0.3)^5 = 0.0024   [0.24% logical error — approaching useful]</p>
<p>At d=17: p_L ≈ (0.3)^9 ≈ 2×10⁻⁵  [20 ppm — sufficient for Shor's algorithm]</p>
</div>

This exponential suppression is the fundamental promise of fault-tolerant quantum computing. But the overhead is severe: at p = 0.3% and p\_th = 1%, a logical qubit with p\_L < 10⁻¹⁰ requires distance d ≈ 25, meaning 2d²−1 = 1,249 physical qubits per logical qubit. For Shor's algorithm on RSA-2048 (requiring ~4,000 logical qubits), this needs ~5 million physical qubits — more than 3,600× what exists today.

#### What NISQ Computers Can Do

- Variational algorithms (VQE, QAOA) for chemistry and optimisation with < 100 qubits and < 100 circuit layers — potentially giving heuristic advantage for specific instances

- Quantum simulation of specific physical systems (Hubbard model, Ising chains) using analog or digital-analog approaches

- Error-mitigated expectation value estimation for shallow circuits

- Quantum sensing and metrology (using quantum entanglement for precision measurement — already demonstrated beyond classical limits)

- Quantum communication and QKD (using single photons and entanglement — already commercially deployed)

#### What NISQ Computers Cannot Currently Do

- Run Shor's algorithm on cryptographically relevant RSA key sizes (needs ~5–20 million physical qubits)

- Run Grover's algorithm at scale (needs fault-tolerant qubits for the quadratic speedup to materialise in practice)

- Classically-intractable quantum chemistry simulation for drug discovery (needs thousands of logical qubits)

- Outperform classical computers reliably on practical optimisation problems

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: Google Willow: Below-Threshold QEC — A 2024 Milestone</strong></p>
<p>In December 2024, Google published results from their Willow processor demonstrating two key milestones: (1) Quantum supremacy redux: Willow performed a random circuit sampling task in 5 minutes that would take the fastest classical supercomputer 10^25 years using the best-known classical algorithms. (2) More importantly for practical QC: Willow demonstrated the first experimental evidence of below-threshold quantum error correction. When the surface code distance increased from d=3 to d=5 to d=7, the logical error rate decreased at each step — the first time a quantum processor has shown the exponential suppression predicted by theory. This does not yet mean fault-tolerant quantum computing is achieved (the logical error rates are still too high for practical algorithms), but it validates the surface code approach and demonstrates that the fault-tolerance threshold is reachable.</p>
<p>What this means for the field: the 10-year timeline to fault-tolerant quantum computing with millions of physical qubits is beginning to feel achievable rather than speculative. For students entering quantum computing research and industry today, there is a realistic probability of working on fault-tolerant quantum computers before the end of their careers.</p>
</div>

<figure class="book-figure">
<img src="content/images/image91.png" alt="Figure 8.6: Noise effects in variational algorithms and quantum process tomography. Left: VQE energy landscape ⟨H⟩(θ) for one parameter θ at four noise levels — noise shrinks the oscillation amplitude, making the minimum shallower and harder to find. The ideal minimum (dashed) is increasingly obscured. Centre: VQE convergence curves — the ideal curve converges to E = −0.80; noisy curves converge to a higher &quot;noise floor&quot; that increases with error rate, with p=3% effectively preventing convergence. Right: Process matrix χ (real part) of a noisy X gate (5% depolarising) reconstructed via quantum process tomography. The dominant element χ_XX ≈ 0.95 confirms an X gate with 5% error; off-diagonal leakage into I, Y, Z components is visible at the ~0.017 level.">
<figcaption>Figure 8.6: Noise effects in variational algorithms and quantum process tomography. Left: VQE energy landscape ⟨H⟩(θ) for one parameter θ at four noise levels — noise shrinks the oscillation amplitude, making the minimum shallower and harder to find. The ideal minimum (dashed) is increasingly obscured. Centre: VQE convergence curves — the ideal curve converges to E = −0.80; noisy curves converge to a higher &quot;noise floor&quot; that increases with error rate, with p=3% effectively preventing convergence. Right: Process matrix χ (real part) of a noisy X gate (5% depolarising) reconstructed via quantum process tomography. The dominant element χ_XX ≈ 0.95 confirms an X gate with 5% error; off-diagonal leakage into I, Y, Z components is visible at the ~0.017 level.</figcaption>
</figure>

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 8: Noise Channels, Qiskit Aer Simulation, and Hardware Benchmarks

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Explain why unitary evolution alone is insufficient to describe real quantum systems. What physical process causes decoherence, and how does it manifest in the density matrix formalism?**

**Q2.  State the operator-sum (Kraus) representation of a quantum channel. Derive the Kraus operators from the system-environment model, and state the completeness condition and its physical meaning.**

**Q3.  Define the depolarising channel and write its Kraus operators. Derive the Bloch vector contraction factor (1−4p/3). What is the process fidelity F = 1−2p/3 and what does it represent?**

**Q4.  Write the Kraus operators for the amplitude damping channel (T₁ channel). Show that they preserve the trace (completeness) and derive the effect on populations ρ₀₀ and ρ₁₁.**

**Q5.  Write the Kraus operators for the phase damping channel (T₂ dephasing). Show that it preserves populations while decaying coherences. What is the combined T₁+T₂ channel?**

**Q6.  Describe Qiskit Aer's NoiseModel architecture. How do you add a thermal relaxation error to a two-qubit gate? Write the Qiskit code for thermal\_relaxation\_error(t1, t2, gate\_time).**

**Q7.  What is Zero-Noise Extrapolation (ZNE) error mitigation? Describe gate folding as a noise amplification method. Why is ZNE more practical than full quantum error correction for NISQ hardware?**

**Q8.  Define the heavy-output generation probability in the Quantum Volume protocol. Why is the threshold set at 2/3? What does it measure physically?**

**Q9.  Describe the randomised benchmarking (RB) protocol. Write the fidelity decay model F(k) = A·p^k + B. How does RB extract the average Clifford gate error r?**

**Q10.  A single-qubit depolarising channel has p = 0.005 (0.5%). Compute: (a) the Bloch vector contraction factor, (b) the output state for input |+⟩ = (|0⟩+|1⟩)/√2, (c) the purity of the output.**

**Q11.  For an amplitude damping channel with T₁ = 200 μs and gate time t = 500 ns, compute γ and the resulting density matrix for initial state |1⟩.**

**Q12.  What is the Choi-Kraus isomorphism? How does it connect quantum channels to positive semidefinite matrices? Why is it useful for quantum process tomography?**

**Q13.  Compare readout error and gate error in quantum circuits. Write the Qiskit Aer ReadoutError model. How does measurement error mitigation (matrix inversion) correct for readout errors?**

**Q14.  Explain the two-qubit depolarising channel. Write its effect on the fidelity. For a CNOT gate with p = 0.01, compute the process fidelity.**

**Q15.  What is the NISQ era's honest assessment of quantum error correction progress? What gate error rates have been achieved, what is the surface code threshold, and what qubit count is needed for fault-tolerant RSA factoring?**

**PART B — MODEL ANSWERS**

**Answer 1:**

Unitary evolution describes closed quantum systems perfectly. Real qubits interact continuously with their environment (substrate phonons, control line noise, background radiation). This interaction causes energy exchange (T₁) and phase randomisation (T₂). Mathematically: the joint system+environment evolves unitarily U\_SE, but tracing over environmental degrees of freedom leaves the system in a mixed state: ρ\_S' = Tr\_E[U\_SE(ρ\_S⊗|0\_E⟩⟨0\_E|)U\_SE†]. The result has Tr(ρ\_S'²) < 1 (mixed state), meaning quantum information has leaked into the environment irreversibly. No single unitary U\_S can describe ρ\_S': the operation is a quantum channel (CPTP map), not a unitary.

**Answer 2:**

Kraus representation: ε(ρ) = Σ\_k K\_k ρ K\_k†. Derivation: joint evolution U\_SE with environment starting in |0\_E⟩. Define K\_k = ⟨k\_E|U\_SE|0\_E⟩ where {|k\_E⟩} is any ONB of the environment. Then ρ\_S' = Σ\_k K\_k ρ K\_k†. Completeness condition: Σ\_k K\_k†K\_k = Σ\_k ⟨0\_E|U\_SE†|k\_E⟩⟨k\_E|U\_SE|0\_E⟩ = ⟨0\_E|U\_SE†(Σ\_k |k\_E⟩⟨k\_E|)U\_SE|0\_E⟩ = ⟨0\_E|U\_SE†U\_SE|0\_E⟩ = I\_S. Physical meaning: completeness ensures total probability is preserved: Tr(ρ\_S') = Tr(Σ\_k K\_k ρ K\_k†) = Tr(ρ Σ\_k K\_k†K\_k) = Tr(ρ·I) = 1.

**Answer 3:**

Depolarising: ε\_D(ρ) = (1−p)ρ + (p/3)(XρX+YρY+ZρZ). Kraus: K₀=√(1−p)I, K₁=√(p/3)X, K₂=√(p/3)Y, K₃=√(p/3)Z. Bloch vector: r\_out = (1−p)r + (p/3)(X-action+Y-action+Z-action of r). XrX component: (r\_x,−r\_y,−r\_z). YrY: (−r\_x,r\_y,−r\_z). ZrZ: (−r\_x,−r\_y,r\_z). Sum: (−r\_x,−r\_y,−r\_z). So r\_out = (1−p)r + (p/3)(−r) = (1−p−p/3)r = (1−4p/3)r ✓. Process fidelity F = Tr(U†ε\_D(U·ρ·U†)) averaged over pure states = 1 − 2p/3 for single-qubit: measures how well the channel approximates the identity operation.

**Answer 4:**

Amplitude damping Kraus operators: K₀ = [[1,0],[0,√(1−γ)]], K₁ = [[0,√γ],[0,0]], where γ = 1−exp(−t/T₁). Completeness: K₀†K₀+K₁†K₁ = diag(1,1−γ) + diag(0,γ) = I ✓. Effect on populations: ρ₀₀' = ⟨0|ε(ρ)|0⟩ = ρ₀₀ + γρ₁₁ (|0⟩ population increases as |1⟩ decays). ρ₁₁' = ⟨1|K₀ρK₀†|1⟩ = (1−γ)ρ₁₁ (|1⟩ population decreases exponentially). Coherences: ρ₀₁' = √(1−γ)·ρ₀₁ (also decay, at rate exp(−t/(2T₁))).

**Answer 5:**

Phase damping Kraus operators: K₀ = [[1,0],[0,√(1−λ)]], K₁ = [[0,0],[0,√λ]], where λ = 1−exp(−t/T₂\_pure). Effect: ρ₀₀' = ρ₀₀ (K₀ and K₁ leave |0⟩⟨0| unchanged). ρ₁₁' = (1−λ)ρ₁₁ + λρ₁₁ = ρ₁₁ (populations preserved!). ρ₀₁' = √(1−λ)·ρ₀₁ (coherences decay). Combined T₁+T₂: combine amplitude damping (γ=1−exp(−t/T₁)) and phase damping (λ=1−exp(−t/T₂\_pure)) channels. Net coherence decay: ρ₀₁(t) = ρ₀₁(0)·exp(−t/(2T₁))·√(1−λ) ≈ exp(−t/T₂) where 1/T₂ = 1/(2T₁)+1/T₂\_pure.

**Answer 6:**

NoiseModel architecture: a dictionary mapping gate names and qubit subsets to QuantumError objects. Gate error assignment: `noise\_model.add\_quantum\_error(error, gate\_name, qubits)` for specific qubits; `add\_all\_qubit\_quantum\_error(error, gate\_name)` for all qubits. Thermal relaxation code: `from qiskit\_aer.noise import thermal\_relaxation\_error; e = thermal\_relaxation\_error(t1=200e-6, t2=100e-6, time=300e-9); noise\_model.add\_all\_qubit\_quantum\_error(e, ['cx'])`. This attaches a combined T₁+T₂ channel calibrated to the specified coherence times and gate duration. Two-qubit thermal: use `.expand()` to tensor product single-qubit errors.

**Answer 7:**

ZNE: run circuit at amplified noise levels c=1,2,3,... (using gate folding: replace gate G with G·G†·G to triple gate noise without changing unitary), measure expectation C(c). Fit polynomial to {(c, C(c))} and extrapolate to c=0. Gate folding: G·G†·G = G (since G†G=I): the computation is identical but noise is approximately 3× larger. ZNE advantage over full error correction: requires no extra physical qubits, works on current NISQ hardware without syndrome measurement infrastructure. Limitation: assumes noise scales linearly with c (not always true); extrapolation error increases with noise level.

**Answer 8:**

Heavy outputs for circuit C: bitstrings x with p\_ideal(x) > median({p\_ideal(y)}). Heavy-output probability: P\_hw = Σ\_{x∈H(C)} count(x)/shots. Threshold 2/3: analytical calculation shows ideal (noise-free) quantum circuits produce heavy outputs with P\_ideal ≈ 0.847 (derived from distribution of squared magnitudes of Haar-random unitary matrix elements). Completely depolarised circuits produce P = 0.5 (uniform distribution). Threshold 2/3 ≈ (0.847+0.5)/2 is the midpoint, ensuring the hardware provides genuinely non-classical computational capability beyond random guessing.

**Answer 9:**

RB protocol: (1) Prepare |0⟩. (2) Apply k random Clifford gates C₁,C₂,...,C\_k. (3) Apply C\_{k+1} = (C\_k·...·C₁)† (the inverse circuit). (4) Measure; record P(0). (5) Repeat for many random sequences and average. (6) Repeat for multiple values of k. Fidelity model: F(k) = A·p^k + B, where p = 1−r and r is the average error per Clifford (depolarising model). A and B are SPAM error parameters. Extract r: plot F vs k, fit exponential decay, extract p from decay rate. RB is SPAM-robust because A,B absorb preparation and measurement errors independently of k.

**Answer 10:**

(a) Contraction factor: 1−4p/3 = 1−4×0.005/3 = 1−0.00667 = 0.9933. (b) |+⟩ has Bloch vector r\_in = (1,0,0). Output: r\_out = 0.9933·(1,0,0) = (0.9933,0,0). Output state: ρ = (I + 0.9933·X)/2 = [[0.5, 0.4967],[0.4967, 0.5]]. (c) Purity: Tr(ρ²) = 2×0.5²+2×0.4967² = 0.5+0.4934 = 0.9934... wait: Tr(ρ²) = (1+|r|²)/2 = (1+0.9933²)/2 = (1+0.9866)/2 = 0.9933. The purity is 0.9933 — almost pure but slightly mixed due to the 0.5% depolarising noise.

**Answer 11:**

γ = 1−exp(−t/T₁) = 1−exp(−500×10⁻⁹/200×10⁻⁶) = 1−exp(−0.0025) ≈ 0.002497. For initial state |1⟩: ρ\_in = [[0,0],[0,1]]. After amplitude damping: ρ₀₀' = ρ₀₀ + γρ₁₁ = 0 + 0.002497×1 = 0.002497. ρ₁₁' = (1−γ)ρ₁₁ = 0.9975×1 = 0.9975. ρ₀₁' = √(1−γ)×ρ₀₁ = 0. Final: ρ = [[0.002497, 0],[0, 0.9975]]. The qubit remains predominantly in |1⟩ but has decayed slightly toward |0⟩ in 500 ns — negligible for T₁ = 200 μs.

**Answer 12:**

Choi-Kraus isomorphism: every CPTP map ε on a d-dimensional system corresponds uniquely to a positive semidefinite (d²×d²) Choi matrix J(ε) = (I⊗ε)(|Φ+⟩⟨Φ+|), where |Φ+⟩ = (1/√d)Σᵢ|ii⟩ is maximally entangled. The Kraus operators are obtained as square roots of J(ε). Completeness J(ε) ≥ 0 and Tr\_B(J(ε)) = I/d ensure the channel is CPTP. Useful for process tomography: prepare all d² input states, apply the channel, measure. The d²×d² set of output density matrices determines J(ε) completely, from which Kraus operators are extracted.

**Answer 13:**

Readout error: measurement device misclassifies qubit state with probabilities p(1|0) (read 1 when qubit is in |0⟩) and p(0|1) (read 0 when qubit is in |1⟩). Qiskit: `ReadoutError([[1-p10, p10],[p01, 1-p01]])` as a 2×2 confusion matrix. Mitigation: measure calibration circuits (all-zeros, all-ones) to determine confusion matrix A. For output counts vector v\_raw: v\_mitigated = A⁻¹·v\_raw (pseudoinverse for stability). Practical: compute calibration circuits for all 2^n basis states (expensive for large n); use tensored readout mitigation (assumes independent qubit errors) for scalability.

**Answer 14:**

Two-qubit depolarising: ε\_D2(ρ) = (1−p)ρ + (p/15)Σ\_{P∈P₂\{II}} PρP†. Applies random non-identity two-qubit Pauli error with probability p/15 each. 15 non-identity Paulis in P₂ = {I,X,Y,Z}⊗{I,X,Y,Z} minus II. Process fidelity: F = 1−16p/15. For p = 0.01: F = 1 − 16×0.01/15 = 1 − 0.01067 = 0.9893. This represents 1.07% infidelity per CNOT — comparable to IBM Heron's ~0.15% to ~0.5% CNOT error rate in practice.

**Answer 15:**

Progress (2024): IBM Heron two-qubit gate error ~0.15% (1.5×10⁻³), below surface code threshold of ~1%. Single-qubit error ~0.01%. Threshold significance: below threshold, logical error rate decreases exponentially with code distance d. IBM Heron at 0.15% error: code distance d≈15 gives logical error ~10⁻¹³ per gate (highly fault-tolerant). Physical qubits needed per logical qubit: ~2d² ≈ 450 at d=15. For Shor on RSA-2048: ~4000 logical qubits × 450 = ~1.8 million physical qubits. IBM Heron has 133 qubits. Gap: factor ~14,000. Timeline: at current scaling (~2× qubits/year): ~14 years to reach 1.8M qubits. But control systems, classical overhead, and logical gate compilation also require massive development.

## EXERCISES — CHAPTER 8

### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.1</strong></p>
<p>Problem: A single-qubit Pauli channel has p_I=0.90, p_X=0.04, p_Y=0.02, p_Z=0.04. (a) Write the channel in operator-sum form. (b) Compute the Bloch sphere shrinkage factors λ_x, λ_y, λ_z. (c) Compute the fidelity of the output with input |0⟩.</p>
<p><strong>Solution:</strong></p>
<p>(a) ε(ρ) = 0.90·ρ + 0.04·XρX + 0.02·YρY + 0.04·ZρZ</p>
<p>Kraus operators: K₀=√0.90·I, K₁=√0.04·X, K₂=√0.02·Y, K₃=√0.04·Z</p>
<p>Verify: 0.90+0.04+0.02+0.04 = 1.00 ✓</p>
<p>(b) Shrinkage factors:</p>
<p>λ_x = p_I+p_X-p_Y-p_Z = 0.90+0.04-0.02-0.04 = 0.88</p>
<p>λ_y = p_I-p_X+p_Y-p_Z = 0.90-0.04+0.02-0.04 = 0.84</p>
<p>λ_z = p_I-p_X-p_Y+p_Z = 0.90-0.04-0.02+0.04 = 0.88</p>
<p>(c) Input |0⟩ has Bloch vector r=(0,0,1). Output: r'=(0,0,0.88).</p>
<p>Density matrix: ρ' = (I+0.88·Z)/2 = [[0.94,0],[0,0.06]]</p>
<p>Fidelity: F = ⟨0|ρ'|0⟩ = 0.94.</p>
<p>The |0⟩ fidelity drops from 1.0 to 0.94 due to X and Y errors (X flips |0⟩→|1⟩; Z preserves |0⟩).</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.2</strong></p>
<p>Problem: The amplitude damping channel with γ=0.15 acts on ρ = [[0.7, 0.4],[0.4, 0.3]]. Compute the output density matrix and verify trace preservation and positive semi-definiteness.</p>
<p><strong>Solution:</strong></p>
<p>Using ε_AD formula:</p>
<p>ρ₀₀' = ρ₀₀ + γ·ρ₁₁ = 0.7 + 0.15×0.3 = 0.7 + 0.045 = 0.745</p>
<p>ρ₁₁' = (1-γ)·ρ₁₁ = 0.85×0.3 = 0.255</p>
<p>ρ₀₁' = √(1-γ)·ρ₀₁ = √0.85×0.4 = 0.9220×0.4 = 0.3688</p>
<p>Output: ρ' = [[0.745, 0.369],[0.369, 0.255]]</p>
<p>Trace: 0.745 + 0.255 = 1.000 ✓</p>
<p>Determinant: 0.745×0.255 − 0.369² = 0.190 − 0.136 = 0.054 &gt; 0 ✓</p>
<p>The output is a valid density matrix: Tr=1, positive semi-definite.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.3</strong></p>
<p>Problem: In randomised benchmarking, the survival probability fits to P(m) = 0.5 + 0.5·p_rb^m where p_rb is the decay parameter. A fit gives p_rb = 0.9985. (a) Extract the average Clifford gate error rate r. (b) If each Clifford gate averages 1.5 CX gates, what is the per-CX error rate? (c) How many Clifford gates can be applied before P(m) &lt; 0.6?</p>
<p><strong>Solution:</strong></p>
<p>(a) r = 1 − p_rb = 1 − 0.9985 = 0.0015 = 0.15% per Clifford gate</p>
<p>(b) Per-CX error rate ≈ r / 1.5 = 0.0015/1.5 = 0.001 = 0.10%</p>
<p>(c) P(m) = 0.5+0.5·p_rb^m &lt; 0.6: 0.5·p_rb^m &lt; 0.1 → p_rb^m &lt; 0.2</p>
<p>m·ln(0.9985) &lt; ln(0.2): m &gt; ln(0.2)/ln(0.9985) = −1.609/(−0.001501) ≈ 1072</p>
<p>After ~1,072 Clifford gates, P(|0⟩) &lt; 0.6 — the signal is barely above random (0.5).</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.4</strong></p>
<p>Problem: A qubit has T1=300μs, T2=200μs, and undergoes a gate of duration t=500ns. (a) Compute the amplitude damping parameter γ. (b) Compute the pure dephasing parameter λ using T_φ. (c) Write the three Kraus operators of the combined T1+T2 channel.</p>
<p><strong>Solution:</strong></p>
<p>(a) γ = 1 − exp(−t/T1) = 1 − exp(−500e-9/300e-6) = 1 − exp(−0.001667) ≈ 0.001665</p>
<p>(b) T_φ: 1/T_φ = 1/T2 − 1/(2T1) = 1/200 − 1/600 = 3/600 − 1/600 = 2/600 μs⁻¹</p>
<p>T_φ = 300 μs</p>
<p>λ = 1 − exp(−t/T_φ) = 1 − exp(−500e-9/300e-6) ≈ 0.001667</p>
<p>(c) Three Kraus operators (zero-temperature):</p>
<p>K₀ = [[1, 0], [0, √((1-γ)(1-λ))]] = [[1,0],[0,√(0.998335×0.998333)]] ≈ [[1,0],[0,0.99834]]</p>
<p>K₁ = [[0, √γ], [0, 0]] = [[0, 0.04080], [0, 0]]     (T1 decay)</p>
<p>K₂ = [[0,0],[0,√((1-γ)·λ)]] = [[0,0],[0,√(0.998335×0.001667)]] ≈ [[0,0],[0,0.04081]]  (dephasing)</p>
<p>Verify: K₀†K₀+K₁†K₁+K₂†K₂ = diag(1, (1-γ)(1-λ)+γ+(1-γ)λ) = diag(1,1) = I ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.5</strong></p>
<p>Problem: Compute the QV protocol's HOGP for a simple 2-qubit circuit with ideal output distribution P(00)=0.6, P(01)=0.05, P(10)=0.05, P(11)=0.3. (a) Find the median and heavy outputs. (b) Compute the ideal h_ideal. (c) If hardware measures P_hw(00)=0.52, P_hw(01)=0.08, P_hw(10)=0.08, P_hw(11)=0.32, compute h_hw. Does this pass QV?</p>
<p><strong>Solution:</strong></p>
<p>(a) Sorted probabilities: 0.05, 0.05, 0.30, 0.60. Median = (0.05+0.30)/2 = 0.175.</p>
<p>Heavy outputs (P &gt; 0.175): |00⟩ (P=0.60) and |11⟩ (P=0.30).</p>
<p>(b) h_ideal = P(00) + P(11) = 0.60 + 0.30 = 0.90 &gt; 2/3 ✓ (ideal value)</p>
<p>(c) Hardware HOGP: h_hw = P_hw(heavy outputs) = P_hw(00) + P_hw(11) = 0.52 + 0.32 = 0.84</p>
<p>h_hw = 0.84 &gt; 2/3 = 0.667 ✓ — this circuit PASSES the QV test.</p>
<p>The hardware achieves 84% of the heavy-output probability compared to the ideal 90%, demonstrating good but not perfect agreement with the ideal distribution.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.6</strong></p>
<p>Problem: Implement ZNE in Qiskit Aer for a single-qubit Z-basis measurement after a noisy X gate. Use gate folding at λ=1,2,3 and linear extrapolation to estimate ⟨Z⟩_ideal. Depolarising error p=0.05.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.7</strong></p>
<p>Problem: The surface code has fault-tolerance threshold p_th ≈ 1%. A processor has physical error rate p = 0.003 (0.3%). (a) Compute the logical error rate p_L for code distances d=3,5,7,9. (b) How many physical qubits are needed per logical qubit for d=7? (c) For what target p_L would d=7 suffice for a 1,000-gate logical algorithm?</p>
<p><strong>Solution:</strong></p>
<p>(a) p_L ≈ (p/p_th)^((d+1)/2) with p/p_th = 0.3/1.0 = 0.3:</p>
<p>d=3: p_L ≈ 0.3² = 0.09      (9% — far too high)</p>
<p>d=5: p_L ≈ 0.3³ = 0.027     (2.7% — still high)</p>
<p>d=7: p_L ≈ 0.3⁴ = 0.0081    (0.81% per logical gate)</p>
<p>d=9: p_L ≈ 0.3⁵ = 0.00243   (0.24% per logical gate)</p>
<p>(b) Physical qubits for surface code distance d=7: 2d²-1 = 2×49-1 = 97 physical qubits per logical qubit.</p>
<p>(c) For a 1,000-gate algorithm with d=7 (p_L=0.0081 per gate): total logical error probability ≈ 1−(1−0.0081)^1000 ≈ 1−exp(−8.1) ≈ 1−0.0003 ≈ 99.97% chance of error. This is completely unreliable. We need p_L &lt; 10⁻⁶ per gate for a 1,000-gate algorithm to have &gt;99.9% success. That requires d≥17 at p=0.3% (p_L ≈ 0.3^9 ≈ 2×10⁻⁵).</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 8.8</strong></p>
<p>Problem: A system completes the CLOPS benchmark protocol (M=100, K=10, S=100, D=log₂(QV)) in T=40 seconds with QV=1024. (a) Compute CLOPS. (b) For a VQE run requiring 300 iterations × 500 shots × 10 circuit layers, compute the estimated runtime. (c) IBM Heron (CLOPS=15,000): how much faster?</p>
<p><strong>Solution:</strong></p>
<p>(a) D = log₂(1024) = 10 layers</p>
<p>CLOPS = (100×10×100×10)/40 = 10,000,000/40 = 250,000 CLOPS</p>
<p>(b) VQE total circuit-layer operations: 300×500×10 = 1,500,000</p>
<p>Runtime on this system: 1,500,000 / 250,000 = 6 seconds</p>
<p>(c) IBM Heron CLOPS = 15,000 (much lower than 250,000 here)</p>
<p>Runtime on Heron: 1,500,000 / 15,000 = 100 seconds</p>
<p>This system is 100/6 ≈ 17× faster than Heron for this workload.</p>
<p>Note: 250,000 CLOPS would be exceptional — this corresponds to a theoretical fast classical simulator with D=10; typical quantum hardware achieves 850–15,000. The exercise illustrates how CLOPS scales with QV and timing.</p>
</div>

### B. Unsolved Problems

Solve independently. Bracketed answers for self-checking.

1. A Pauli channel acts on ρ=|+⟩⟨+| with p\_I=0.85, p\_X=0.05, p\_Y=0.05, p\_Z=0.05. Compute the output Bloch vector and fidelity F with |+⟩. [Answer: λ\_x=0.85+0.05-0.05-0.05=0.80; r'=(0.80,0,0); F=(1+0.80)/2=0.90]

2. Show that the two-qubit depolarising channel with p=0 is the identity. Also show that at p=15/16, the output is (I₄/4) — the maximally mixed 2-qubit state. [Answer: At p=0: ε(ρ)=(1-0)ρ=ρ ✓; At p=15/16: ε(ρ)=(1-15/16)ρ+(15/16)(1/15)Σ\_{P≠II}PρP†=(1/16)ρ+(1/16)(Σ\_P PρP†−ρ)=(1/16)Σ\_P PρP†=(1/16)·4I₄/4·4·Tr[ρ]=I₄/4 ✓]

3. Apply the amplitude damping channel with γ=0.25 to the maximally mixed state ρ=I/2=[[0.5,0],[0,0.5]]. Show the output is NOT I/2, confirming the channel is not unital. [Answer: ρ₀₀'=0.5+0.25×0.5=0.625; ρ₁₁'=0.75×0.5=0.375; output=[[0.625,0],[0,0.375]]≠I/2 ✓ (non-unital)]

4. A processor achieves HOGP = 0.71 ± 0.025 (mean ± standard error) on 7-qubit QV circuits. Does it achieve QV=128? [Answer: Lower confidence bound = 0.71 − 2×0.025 = 0.66 < 2/3 = 0.667. Marginally below threshold — does NOT achieve QV=128 at 97.5% confidence. More shots needed.]

5. Compute the fidelity F = 1 − 2p/3 of the depolarising channel for p=0.002, 0.005, 0.01, 0.02. Plot trend. [Answer: F=0.99867, 0.99667, 0.99333, 0.98667. Each doubling of p reduces F by approximately 2/3 × Δp]

6. The phase damping channel with λ=0.1 acts on ρ=|+⟩⟨+|=(1/2)[[1,1],[1,1]]. Find the output and Bloch vector. Then verify 1/T2 = 1/(2T1) + 1/T\_φ gives the correct total dephasing when combined with amplitude damping at γ=0.05. [Answer: ρ'=(1/2)[[1,√0.9],[√0.9,1]]; r'=(√0.9,0,0)=(0.949,0,0); Combined fidelity with |+⟩: F=√(1-γ)·√(1-λ)/1 ≈ 0.975×0.949 ≈ 0.925]

7. Implement in Qiskit Aer a noise model where qubit 0 has T1=100μs, T2=80μs and qubit 1 has T1=200μs, T2=150μs. Simulate a Bell state circuit and compute the output density matrix. Verify trace=1 and compare off-diagonal coherence to the ideal. [Answer: Code exercise — expected Bell state density matrix ρ₀₁ element will be reduced from ideal 0.5 to approximately 0.5×exp(-t\_cx/T2\_eff)≈0.5×exp(-300e-9/80e-6)≈0.5×0.9963≈0.498]

8. For the QV pass condition h\_hw > 2/3, derive the minimum number of shots N needed to detect h\_hw = 0.70 with 97.5% confidence (one-sided z-test). [Answer: Null: h=2/3; test statistic z=(h\_hw-2/3)/√(h\_hw(1-h\_hw)/N). For z>1.96 (97.5%): N > (1.96)²×0.70×0.30/(0.70-0.667)² = 3.84×0.21/(0.033)² ≈ 3.84×0.21/0.00109 ≈ 740 shots minimum per circuit]

9. The surface code threshold is p\_th ≈ 0.01. A processor has p=0.005. For a logical qubit with p\_L < 10⁻⁸, what code distance d is needed? How many physical qubits does this require? [Answer: (p/p\_th)^((d+1)/2) < 10⁻⁸: (0.5)^((d+1)/2) < 10⁻⁸; (d+1)/2 × ln(0.5) < -18.4; (d+1)/2 > 26.5; d+1 > 53; d ≥ 53. Physical qubits: 2×53²-1 = 5617 per logical qubit]

10. Using the CLOPS formula, compute CLOPS for IBM Heron R2 if: QV=4096, total time for M=100 templates × K=10 updates × S=100 shots is 8 seconds. Verify this is consistent with the reported ~15,000 CLOPS. [Answer: D=log₂(4096)=12; CLOPS=(100×10×100×12)/8=12,000,000/8=1,500,000. Note: this is much higher than 15,000 because the S=100 shots/circuit contributes linearly. If adjusted for real-hardware overhead, the effective figure is lower. The official IBM CLOPS benchmark definition weights differently; 15,000 is the hardware-overhead-included figure.]

### C. Multiple Choice Questions

Circle the best answer. Answers collected at end of section.

**Q1. The completeness condition for Kraus operators Σₖ Kₖ†Kₖ = I ensures that the quantum channel:**

- (a) Preserves all pure states exactly

- (b) Is trace-preserving (total probability is conserved)

- (c) Is unitary (reversible)

- (d) Has at most 2 Kraus operators

**Q2. For the single-qubit depolarising channel at p = 3/4, the output for any input ρ is:**

- (a) The identity operation — output equals input

- (b) The maximally mixed state I/2

- (c) The ground state |0⟩⟨0|

- (d) Undefined — p cannot equal 3/4

**Q3. The amplitude damping channel is NON-UNITAL because:**

- (a) It has 2 Kraus operators instead of 4

- (b) It does not preserve the maximally mixed state I/2

- (c) Its Kraus operators are not Hermitian

- (d) It cannot be written in Kraus form

**Q4. In randomised benchmarking, the decay constant p\_rb is related to the average Clifford error rate r by:**

- (a) r = p\_rb

- (b) r = 1 − p\_rb

- (c) r = p\_rb²

- (d) r = 1 − p\_rb²

**Q5. The phase damping channel with λ = 1 (maximum dephasing) acting on ρ = |+⟩⟨+| gives:**

- (a) |+⟩⟨+| (unchanged)

- (b) |0⟩⟨0| (decayed to ground state)

- (c) I/2 (maximally mixed)

- (d) |−⟩⟨−| (phase flipped)

**Q6. The three Kraus operators of the combined T1+T2 channel (zero temperature) are K₀, K₁, K₂. Which process does K₁ = [[0,√γ],[0,0]] model?**

- (a) Dephasing without energy loss (T\_φ process)

- (b) Energy relaxation from |1⟩ to |0⟩ (T1 process)

- (c) Thermal excitation from |0⟩ to |1⟩

- (d) Phase randomisation from flux noise

**Q7. Quantum Volume QV = 512 means the processor reliably executes random circuits of size:**

- (a) 512 qubits, 1 layer

- (b) 9 qubits, 9 layers (since 2^9=512)

- (c) 512 qubits, 512 layers

- (d) 1 qubit, 512 layers

**Q8. In Zero-Noise Extrapolation (ZNE), gate folding works because:**

- (a) It reduces the total number of gates needed

- (b) Replacing G with G·G†·G increases noise without changing the ideal unitary

- (c) It improves gate fidelity by averaging coherent errors

- (d) It applies error correction codes to the circuit

**Q9. The readout calibration matrix M[i,j] = P(measure i | prepared j) is used to:**

- (a) Calibrate the qubit frequency

- (b) Correct measurement statistics by matrix inversion: corrected = M⁻¹ × raw counts

- (c) Improve T1 relaxation times

- (d) Set the CX gate duration

**Q10. For the surface code with distance d below threshold, the logical error rate scales as:**

- (a) p\_L ≈ (p/p\_th)^d linearly in d

- (b) p\_L ≈ (p/p\_th)^((d+1)/2) — exponential suppression

- (c) p\_L ≈ p (unchanged by error correction)

- (d) p\_L ≈ d·p (error rate grows with d)

**Q11. The Qiskit Aer function depolarizing\_error(p, num\_qubits=2) creates:**

- (a) Independent p/2 error on each qubit separately

- (b) A 2-qubit depolarising channel with 16 Kraus operators (15 two-qubit Paulis)

- (c) The amplitude damping channel for a CX gate

- (d) A readout error with probability p

**Q12. CLOPS measures quantum hardware performance by:**

- (a) The maximum single-gate fidelity achievable

- (b) Circuit layer operations per second, including all compilation and communication overhead

- (c) The ratio of quantum speedup over classical for benchmark circuits

- (d) Number of physical qubits per logical qubit

**Q13. A quantum channel is completely positive (CP) rather than just positive because:**

- (a) All eigenvalues of Kraus operators must be positive

- (b) The channel must remain valid when applied to part of a larger entangled system

- (c) The trace of the output must be exactly 1

- (d) The channel can only be applied to pure states

**Q14. In the QV heavy-output generation probability test, the "heavy outputs" are:**

- (a) The n most likely bitstrings in the ideal distribution

- (b) All bitstrings with ideal probability above the median — exactly half of all 2^n bitstrings

- (c) Bitstrings with probability > 1/n

- (d) Only the most probable single bitstring

**Q15. Google's Willow processor (2024) demonstrated below-threshold quantum error correction, meaning:**

- (a) The processor has zero logical errors

- (b) Increasing the surface code distance d decreased the logical error rate, as predicted by theory

- (c) The processor is now fault-tolerant for Shor's algorithm

- (d) The physical error rate dropped below 10⁻⁶

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 8</strong></p>
<p><strong>Q1: (b)   Q2: (b)   Q3: (b)   Q4: (b)   Q5: (c)</strong></p>
<p><strong>Q6: (b)   Q7: (b)   Q8: (b)   Q9: (b)   Q10: (b)</strong></p>
<p><strong>Q11: (b)  Q12: (b)  Q13: (b)  Q14: (b)  Q15: (b)</strong></p>
<p>Q5 Detail: Phase damping with λ=1 fully dephases: off-diagonal elements × √(1-1)=0, diagonal unchanged. ρ'=[[1/2,0],[0,1/2]]=I/2 for |+⟩ input. Q10 Detail: Below-threshold exponential suppression is the defining property of topological codes. Q13 Detail: Complete positivity (vs mere positivity) is required because qubits can be entangled with ancillas; the channel must map valid joint states to valid joint states.</p>
</div>

### D. Theory Questions

- Explain the distinction between a positive map and a completely positive (CP) map on quantum states. Why must physical quantum channels be CP rather than just positive? Give an example of a positive but not CP map.

- Derive the Bloch sphere action of the general single-qubit Pauli channel (p\_I, p\_X, p\_Y, p\_Z). Show that the three special cases — bit-flip, phase-flip, and depolarising — are recovered as special cases of this formula.

- State and prove the completeness condition for Kraus operators from first principles. Start from the system-environment model U\_SE acting on ρ\_S ⊗ |0\_E⟩⟨0\_E| and derive Σₖ Kₖ†Kₖ = I.

- Describe the randomised benchmarking protocol in full detail. Explain why Clifford group randomisation converts arbitrary gate errors into an effective depolarising channel (the "twirling" argument). What are the advantages of RB over gate set tomography?

- Compare Zero-Noise Extrapolation (ZNE) and Probabilistic Error Cancellation (PEC) as error mitigation strategies. For each: state the principle, the quantum overhead, the classical processing required, and the key limitation. Under what conditions would you prefer each?

- Explain the Quantum Volume benchmark protocol in full. Why does QV = 2^n rather than simply n? What does the heavy-output generation probability (HOGP) test measure, and why is 2/3 chosen as the threshold?

- The surface code threshold is p\_th ≈ 1% for independent Pauli noise. Explain qualitatively why a threshold exists: why does error correction help below p\_th but hurt above it? How does the logical error rate scale with code distance below threshold?

- Derive the three Kraus operators of the combined T1+T2 channel from the physical model of amplitude damping (T1) and pure dephasing (T\_φ) applied simultaneously. Verify the completeness condition explicitly.

- Explain how CLOPS (Circuit Layer Operations Per Second) differs from gate speed metrics like nanoseconds-per-gate. What components of the quantum computation pipeline does CLOPS capture? Give a worked example comparing two processors with different QV and CLOPS for a VQE problem.

- Describe the NISQ era limitations that prevent current quantum computers from running Shor's algorithm on RSA-2048. Be quantitative: how many physical qubits are needed, what error rate is required, what code distance is needed, and how far are current processors from these targets?

### E. Programming Assignments

PA8.1 — Complete Noise Channel Simulation Suite. Build a comprehensive Qiskit Aer noise simulation programme that: (a) Creates three noise channels: (i) a Pauli channel with p\_X=0.003, p\_Y=0.001, p\_Z=0.005; (ii) an amplitude damping channel with γ=0.02; (iii) a phase damping channel with λ=0.03. (b) Applies each channel to the four states |0⟩, |1⟩, |+⟩, |i⟩ and computes the output density matrix. (c) For each channel×input combination, compute: (i) output Bloch vector, (ii) state fidelity with input, (iii) purity Tr(ρ²). (d) Produces a 4×4 table (channels × input states) of fidelities. (e) Plots the Bloch vector shrinkage as a 3D bar chart. Submit: code, tables, plots, and 500-word analysis discussing which input states are most/least affected by each noise channel and why.

PA8.2 — Randomised Benchmarking Implementation. Implement a standard single-qubit randomised benchmarking (RB) experiment in Qiskit Aer: (a) Generate 50 random Clifford gate sequences for each of m = 1, 2, 4, 8, 16, 32, 64, 128, 256 (using qiskit.circuit.library.random\_clifford or manual Clifford circuit construction). (b) For each sequence, append the inverse Clifford gate and measure. (c) Simulate with AerSimulator using three noise models: (i) ideal (no noise), (ii) depolarising p=0.002, (iii) depolarising p=0.008. (d) Fit each RB decay curve P(m) = A·p\_rb^m + B using scipy.optimize.curve\_fit. (e) Extract and compare the error rate r = 1 − p\_rb for each noise model against the theoretical expectation. Submit: code, RB decay curves (all three on one plot), table of r values, and 300-word comparison of measured vs expected error rates.

PA8.3 — QV Estimation with Qiskit Aer. Implement the Quantum Volume estimation protocol: (a) For n = 2, 3, 4, 5 qubits: generate 100 random n-qubit, n-layer model circuits using random two-qubit SU(4) gates (use qiskit.quantum\_info.random\_unitary(4) for each 2-qubit pair per layer). (b) Classically simulate each circuit with AerSimulator(method="statevector") to get P\_ideal(x). Identify heavy outputs (P > median). (c) Run same circuits with AerSimulator using FakeSherbrookeV2 noise model to get P\_hw(x). (d) Compute heavy-output generation probability h\_hw = Σ\_{heavy x} P\_hw(x) for each circuit. (e) Average h\_hw over 100 circuits; compute mean and 95% confidence interval. (f) Determine maximum n where h̄\_hw − 2σ > 2/3. Report your estimated QV and compare with FakeSherbrooke's documented QV. Submit: code, h\_hw distribution plots per n, table of mean HOGP and confidence intervals, and comparison with official QV.

### F. Project Suggestions

Project 8.A — Noise Channel Tomography on Real IBM Hardware. Connect to a real IBM Quantum backend and perform quantum process tomography (QPT) to characterise a noisy gate: (a) Choose a two-qubit gate (CX or ECR) on the best qubit pair. (b) Implement QPT: for the 16 two-qubit input state combinations (tensor products of |0⟩, |1⟩, |+⟩, |i⟩), apply the gate and measure in all 9 two-qubit Pauli basis combinations. (c) Reconstruct the 4×4 process matrix χ from the 16×9 = 144 measurement outcomes using linear inversion. (d) Compute the gate fidelity F\_gate = Tr(χ\_ideal · χ\_measured) and the diamond norm distance from the ideal gate. (e) Decompose χ into Kraus operators and identify the dominant error type. Write a 3,000-word report covering: QPT methodology, experimental results, comparison with IBM-reported gate fidelity, identification of dominant noise sources, and recommendations for which qubits give the best performance.

Project 8.B — Error Mitigation Study: ZNE vs Readout Correction for VQE. Implement a complete error mitigation study for a 4-qubit VQE calculation: (a) Design a 4-qubit VQE circuit for the H₂ molecule (minimum: Ry+CX ansatz, 2 parameters). (b) Simulate the energy landscape ⟨H⟩(θ₁,θ₂) under: (i) ideal, (ii) depolarising p=0.005, (iii) depolarising p=0.005 + readout error 2%, (iv) thermal relaxation (T1=100μs, T2=80μs). (c) Apply ZNE with gate folding λ=1,2,3 to scenario (ii). (d) Apply readout correction to scenario (iii) using calibration matrix inversion. (e) Apply both ZNE and readout correction to scenario (iv). (f) For each scenario, run COBYLA optimisation to find the minimum energy and compare with the exact ground state energy. Write a 2,500-word analysis covering: which error source dominates, how much error mitigation improves the energy estimate, the classical overhead required, and whether error mitigation is sufficient for chemical accuracy (1 kcal/mol ≈ 1.6 mHartree).

Project 8.C — Comprehensive Hardware Benchmark Report. Perform a systematic benchmarking study comparing three IBM backends accessible via Qiskit IBM Runtime: (a) Collect from each backend: T1, T2 (all qubits), 1Q gate fidelity (RB), 2Q gate fidelity (interleaved RB on 3 qubit pairs), readout fidelity (calibration matrix). (b) Compute per-backend statistics: median/mean/best/worst qubit for each metric, qubit quality score ranking. (c) Implement simplified QV estimation (n=3,4,5) on each backend using AerSimulator with the backend noise model. (d) Measure approximate CLOPS by timing a VQE circuit execution loop (100 templates × 10 updates × 100 shots). (e) For a specific algorithm (QAOA depth-3 on Max-Cut), predict and measure the algorithm fidelity on each backend. Write a 4,000-word hardware benchmark report with: comparative tables, radar charts of all metrics, QV estimates, CLOPS measurements, algorithm performance predictions vs actuals, and a recommendation of which backend is best suited for which algorithm class.

## References and Further Reading

1. Nielsen, M. A. & Chuang, I. L. (2000). Quantum Computation and Quantum Information. Cambridge University Press. [Chapter 8: Quantum noise and quantum operations — the definitive reference for Kraus operators and quantum channels]

2. Shor, P. W. (1995). Scheme for reducing decoherence in quantum computer memory. Physical Review A, 52, R2493. [Original quantum error correction paper]

3. Cross, A. W., Bishop, L. S., Sheldon, S., Nation, P. D., & Gambetta, J. M. (2019). Validating quantum computers using randomized model circuits. Physical Review A, 100, 032328. [Quantum Volume definition and protocol]

4. Wack, A. et al. (2021). Quality, Speed, and Scale: Three Key Attributes to Measure the Performance of Near-Term Quantum Computers. arXiv:2110.14108. [CLOPS definition and IBM benchmarking framework]

5. Magesan, E., Gambetta, J. M., & Emerson, J. (2012). Efficient measurement of quantum gate error by interleaved randomized benchmarking. Physical Review Letters, 109, 080505. [Interleaved randomised benchmarking]

6. Temme, K., Bravyi, S., & Gambetta, J. M. (2017). Error mitigation for short-depth quantum circuits. Physical Review Letters, 119, 180509. [Zero-noise extrapolation and probabilistic error cancellation]

7. Google Quantum AI (2024). Quantum error correction below the surface code threshold. Nature, 614, 676. [Google Willow below-threshold QEC demonstration]

8. Fowler, A. M., Martinis, J. M. et al. (2012). Surface codes: Towards practical large-scale quantum computation. Physical Review A, 86, 032324. [Surface code resource estimates for Shor's algorithm]

9. Preskill, J. (2018). Quantum Computing in the NISQ Era and Beyond. Quantum, 2, 79. [NISQ definition and near-term outlook]

10. Qiskit Aer Documentation (2024). https://qiskit.github.io/qiskit-aer/ [Official reference for NoiseModel, thermal\_relaxation\_error, and all noise simulation functions]
