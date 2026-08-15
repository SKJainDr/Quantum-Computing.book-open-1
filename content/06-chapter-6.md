# CHAPTER 6: Grover's Search and Variational Quantum Algorithms (NISQ Era)

<div class="box box-anecdote">
<p class="box-title"><strong>📜  Lov Grover's Discovery — Bell Labs, 1996</strong></p>
<p>In 1996, Lov Grover was a researcher at Bell Labs working on database search problems. He was not looking for a quantum algorithm — he was thinking about a classical problem: how efficiently can you find an item in an unsorted list? The classical answer was well-known: on average, you check N/2 items before finding what you seek, and in the worst case, N items. This O(N) bound was considered essentially optimal.</p>
<p>Grover realised something stunning: if the database were stored as a quantum superposition, one could use interference to amplify the amplitude of the target item while suppressing all others — iteratively, like a quantum compass needle swinging toward true north. The algorithm required only O(√N) queries. Grover published his result at the 28th Annual ACM Symposium on Theory of Computing, and it became one of the most celebrated results in quantum computing theory.</p>
<p>The algorithm was soon proven optimal by Bennett, Bernstein, Brassard, and Vazirani (1997): no quantum algorithm can search an unstructured database in fewer than Ω(√N) queries. This was a landmark result — a provable quantum lower bound meeting the quantum upper bound. For the first time, a quantum algorithm was known to be exactly optimal.</p>
<p>Today, Grover's algorithm has applications far beyond database search: it accelerates any problem reducible to satisfiability, speeds up collision finding in cryptographic hash functions, and underlies amplitude estimation — a key primitive in quantum Monte Carlo. Meanwhile, a different class of quantum algorithms — variational methods — has emerged specifically for the NISQ era. These hybrid quantum-classical algorithms run on today's noisy hardware and are the most promising path to near-term quantum advantage.</p>
</div>

This chapter develops two of the most practically important quantum algorithms. Grover's search algorithm is the archetypal quadratic quantum speedup — provably optimal, widely applicable, and deeply instructive about how quantum interference creates computational advantage. Variational Quantum Algorithms (VQE and QAOA) represent a completely different paradigm: rather than achieving exact speedups through clever circuit design, they use shallow parametrised circuits optimised by classical computers to approximate solutions to hard problems on near-term hardware.

The chapter is self-contained. Section 6.1 develops Grover's algorithm from first principles, proving the O(√N) bound, the geometric interpretation, the optimal iteration count, and the over-rotation effect. Section 6.2 develops the variational quantum algorithm framework — the variational principle, ansatz design, the parameter-shift gradient rule, and complete Qiskit implementations of VQE and QAOA. Together, these two sections complete the foundational algorithm survey of Unit III.

## 6.1 Grover's Search Algorithm

Grover's algorithm solves the unstructured search problem: given a function f: {0,1,...,N−1} → {0,1} where exactly M items satisfy f(x)=1 (the "marked" items), find any one of them. The algorithm uses O(√(N/M)) applications of the oracle — a quadratic speedup over the classical O(N/M) expected queries, and provably optimal in the quantum query model.

### 6.1.1 Unstructured Search: Problem and Classical Bounds

#### The Classical Problem

Consider N items in a database (N = 2ⁿ for an n-bit index). Each item has an associated Boolean property f(x) ∈ {0,1}. We want to find any x with f(x)=1. No additional structure is assumed — the items are unordered, and f(x) can only be evaluated by querying the oracle.

Classical algorithm: check items one at a time. If M items are marked (f(x)=1), the expected number of queries before finding a marked item is N/(M+1) ≈ N/M for small M. For M=1 (single marked item): N/2 queries on average, N queries worst case. This is clearly optimal — any classical algorithm might need to check all N items before finding the marked one.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Unstructured Search Problem</strong></p>
<p>Given: Black-box oracle O_f implementing f: {0,...,N−1} → {0,1} with M marked items.</p>
<p>Goal: Find any x* with f(x*)=1.</p>
<p>Why it matters: Grover's algorithm applies to any search over a set of possibilities, not just literal databases. It accelerates: brute-force satisfiability checking (SAT), cryptographic key search, optimization over discrete spaces, and any problem whose solutions can be verified efficiently.</p>
</div>

### 6.1.2 Phase Oracle and Amplitude Amplification

#### Setting Up the Algorithm

Grover's algorithm operates on an n-qubit register (where N=2ⁿ). The algorithm begins by creating a uniform superposition of all N basis states using n Hadamard gates:

<div class="box box-equation">
<p><strong>Equation 6.2</strong></p>
<p><strong>|s⟩ = H^(⊗n)|0⟩^n = (1/√N) Σ_{x=0}^{N-1} |x⟩</strong></p>
<p>The initial amplitude of each basis state is 1/√N.</p>
<p>Initially, P(finding a marked item) = M/N.</p>
</div>

A single measurement of |s⟩ finds a marked item with probability M/N — for M=1, N=1024, this is only 0.1%. The goal of Grover's algorithm is to amplify the amplitude of marked states to ≈ 1/√M using a sequence of operations called Grover iterations.

#### The Grover Iteration: Two Reflections

Each Grover iteration consists of two operations:

- Step 1 — Phase oracle O\_f: marks the target states by flipping the sign of their amplitudes: O\_f|x⟩ = (−1)^f(x)|x⟩ (phase oracle, see Chapter 5 Section 5.1.1). Unmarked states are unchanged; marked states get a phase −1.

- Step 2 — Grover diffusion operator D = 2|s⟩⟨s| − I: reflects the amplitude vector about the mean. All amplitudes above the mean are reduced; all amplitudes below the mean are increased. Since the phase oracle has pushed the marked-state amplitude below the mean (to negative), the diffusion operator amplifies it.

#### The Diffusion Operator in Detail

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Grover Diffusion Operator</strong></p>
<p>D = 2|s⟩⟨s| − I   where |s⟩ = (1/√N)Σ_x|x⟩ is the uniform superposition</p>
<p>Action: (D|v⟩)_x = 2μ − v_x  where μ = (1/N) Σ_x v_x is the mean amplitude</p>
<p>This is called "inversion about the mean" — each amplitude is reflected through the mean.</p>
<p>Circuit implementation: apply H^(⊗n), then X^(⊗n), then a multi-controlled-Z gate (CZ on all n qubits), then X^(⊗n), then H^(⊗n). Total: 2n + 1 gates per diffusion step.</p>
</div>

<figure class="book-figure">
<img src="content/images/image72.png" alt="Figure 6.1: Amplitude amplification in Grover&#x27;s algorithm. Left: bar chart showing initial amplitudes (blue), after oracle phase flip (red, marked state |3⟩ goes negative), and after diffusion (green, marked state amplified). Right: success probability vs iteration count for N=256, M=1 — probability peaks at k_opt=12 then drops due to over-rotation.">
<figcaption>Figure 6.1: Amplitude amplification in Grover&#x27;s algorithm. Left: bar chart showing initial amplitudes (blue), after oracle phase flip (red, marked state |3⟩ goes negative), and after diffusion (green, marked state amplified). Right: success probability vs iteration count for N=256, M=1 — probability peaks at k_opt=12 then drops due to over-rotation.</figcaption>
</figure>

### 6.1.3 The Grover Diffusion Operator

#### Why the Diffusion Operator Amplifies the Marked State

Let us trace a single Grover iteration precisely. Define:

- α: amplitude of each unmarked state (N−M states with f(x)=0)

- β: amplitude of each marked state (M states with f(x)=1)

Initial state: α₀ = β₀ = 1/√N. Mean amplitude: μ₀ = (1/N)[(N−M)·(1/√N) + M·(1/√N)] = 1/√N.

After oracle (phase flip of marked states):

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>α₁ = 1/√N  (unchanged)</p>
<p>β₁ = −1/√N  (sign flipped)</p>
<p>Mean μ₁ = (1/N)[(N−M)/√N + M·(−1/√N)] = (N−2M)/(N√N)</p>
</div>

After diffusion (inversion about mean μ₁):

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>α₂ = 2μ₁ − α₁ = 2(N−2M)/(N√N) − 1/√N = [1 − 4M/N]/√N</p>
<p>β₂ = 2μ₁ − β₁ = 2(N−2M)/(N√N) + 1/√N = [1 − 4M/N + 2]/√N = [3 − 4M/N]/√N</p>
</div>

For M=1, N=1024: α₂ ≈ (1−0.004)/√N ≈ 1/√N and β₂ ≈ 3/√N. The marked amplitude has tripled in one iteration! This is the power of inversion about the mean: the marked state, sitting below zero after the phase flip, is reflected to a position three times the original amplitude.

#### General k-iteration Formula

After k Grover iterations, the amplitude of each marked state becomes:

<div class="box box-equation">
<p><strong>Equation 6.4</strong></p>
<p><strong>β_k = (1/√M) sin((2k+1)θ)</strong></p>
<p>where sin(θ) = √(M/N), so θ ≈ √(M/N) for small M/N</p>
<p>P(success after k iterations) = M·β_k² = sin²((2k+1)θ)</p>
</div>

This formula reveals the sinusoidal oscillation of success probability with iteration count. The state rotates in a 2D subspace, traversing angle 2θ per iteration.

### 6.1.4 Geometric Interpretation and Optimal Iterations

#### The 2D Subspace Picture

Grover's algorithm has a beautiful geometric interpretation. The entire computation takes place in a 2-dimensional subspace spanned by two orthogonal vectors:

- |m⟩ = (1/√M) Σ\_{f(x)=1} |x⟩: the normalised superposition of all marked states

- |w⟩ = (1/√(N−M)) Σ\_{f(x)=0} |x⟩: the normalised superposition of all unmarked states

The initial state |s⟩ lies in this subspace: |s⟩ = cos(θ)|w⟩ + sin(θ)|m⟩ where sin(θ)=√(M/N).

Each Grover iteration is a rotation in this 2D plane: the oracle reflects about |w⟩, and the diffusion operator reflects about |s⟩. Two reflections compose to a rotation by 2θ. After k iterations, the state has rotated by (2k+1)θ from |w⟩ toward |m⟩.

<figure class="book-figure">
<img src="content/images/image73.png" alt="Figure 6.2: Grover&#x27;s algorithm — geometric interpretation and circuit. Left: the 2D geometric picture — each iteration rotates the state by 2θ toward the marked state |m⟩. After k_opt iterations, the state is nearly |m⟩. Right: n-qubit Grover circuit — H^⊗n creates the initial superposition; k_opt repetitions of oracle + diffusion amplify the marked state; final measurement reveals the solution.">
<figcaption>Figure 6.2: Grover&#x27;s algorithm — geometric interpretation and circuit. Left: the 2D geometric picture — each iteration rotates the state by 2θ toward the marked state |m⟩. After k_opt iterations, the state is nearly |m⟩. Right: n-qubit Grover circuit — H^⊗n creates the initial superposition; k_opt repetitions of oracle + diffusion amplify the marked state; final measurement reveals the solution.</figcaption>
</figure>

#### Optimal Number of Iterations

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Optimal Grover Iterations</strong></p>
<p>We want (2k+1)θ ≈ π/2 (state pointing toward |m⟩). Solving:</p>
<p>Key values: N=4: k_opt=1; N=16: k_opt=3; N=64: k_opt=6; N=256: k_opt=12; N=1024: k_opt=25.</p>
<p>Each factor of 4 in N doubles k_opt — a square-root relationship.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 6.1: Grover Search in a 16-item Database</strong></p>
<p>Problem: N=16 items, M=1 marked item. (a) Compute k_opt. (b) Find the state amplitude after each iteration. (c) What is the probability of success at k_opt?</p>
<p><strong>Solution:</strong></p>
<p>(a) θ = arcsin(√(1/16)) = arcsin(0.25) ≈ 0.2527 rad</p>
<p>k_opt = round(π/(4θ)) = round(3.1416/(4×0.2527)) = round(3.107) = 3</p>
<p><strong>(b) Amplitude of marked state after k iterations: β_k = (1/√1)·sin((2k+1)θ)</strong></p>
<p>k=0: β_0 = sin(θ) = sin(0.2527) = 0.250 (initial, = 1/√16)</p>
<p>k=1: β_1 = sin(3θ) = sin(0.758) = 0.688</p>
<p>k=2: β_2 = sin(5θ) = sin(1.264) = 0.952</p>
<p>k=3: β_3 = sin(7θ) = sin(1.769) = 0.981   [near maximum!]</p>
<p>(c) P(success at k=3) = β_3² = 0.981² = 0.962 ≈ 96.2%</p>
<p>Compare: classical average queries = N/2 = 8. Quantum: 3. Speedup: 2.7×.</p>
</div>

### 6.1.5 Over-Rotation, Multi-Target, and Partial Grover

#### The Over-Rotation Effect

If more than k\_opt iterations are applied, the state overshoots |m⟩ and rotates back toward |w⟩. The success probability then decreases. This is the "over-rotation" effect:

<img class="fig-img" src="content/images/image74.png" alt="figure">

<div class="box box-equation">
<p><strong>Key Equation — Equation 6.1</strong></p>
<p>P(k iterations) = sin²((2k+1)θ)</p>
<p>This is periodic with period π/θ ≈ π√(N/M)/2</p>
<p>Over-rotating by even a few steps beyond k_opt significantly reduces success probability</p>
</div>

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: Over-Rotation Is Dangerous in Practice</strong></p>
<p>On real hardware, the number of iterations must be chosen carefully. If the number of marked items M is unknown (which is typical in practice), k_opt cannot be computed exactly. Strategies for unknown M include:</p>
<p>Quantum Counting: use amplitude estimation to count M before running Grover</p>
<p>Exponential search: try k = 1, 3, 9, 27, ... (tripling) and use the first k that gives a correct answer. Expected total queries: O(√(N/M))</p>
<p>Partial Grover: run for a random number of iterations uniformly sampled from [1, k_opt]; this gives P ≥ 1/2 even without knowing M exactly</p>
</div>

#### Multi-Target Grover (M > 1 marked items)

When M > 1 items are marked, Grover's algorithm still works with the same circuit — just with a different k\_opt:

<div class="box box-equation">
<p><strong>Key Equation — Equation 6.2</strong></p>
<p><strong>k_opt = round( (π/4) · sqrt(N/M) )  for M marked items</strong></p>
<p>sin(θ) = sqrt(M/N)  →  larger θ means faster convergence</p>
<p>Special case M = N/4: k_opt = 1 (single iteration suffices!)</p>
</div>

Important subtlety: when M > N/4, we have k\_opt = 0 (the initial measurement is already likely to succeed). When M ≥ N/2 (more than half the items are marked), measuring the initial superposition gives a marked item with ≥ 50% probability, and no Grover iterations are needed.

### 6.1.6 Full Qiskit Implementation

```python
# ─────────────────────────────────────────────────────────────────────
# Code 6.1: Grover's Search Algorithm — Complete Qiskit Implementation
# Searches for a target string in N=2^n items using Grover iterations
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
import numpy as np
import matplotlib.pyplot as plt

def phase_oracle(n, target):
    """Phase oracle: flips sign of |target> state."""
    qc = QuantumCircuit(n)
    # Flip qubits where target bit is 0 (to convert to all-ones check)
    for i, bit in enumerate(reversed(target)):  # LSB first in Qiskit
        if bit == "0":
            qc.x(i)
    # Multi-controlled Z: applies -1 phase when all qubits are |1>
    # For n>2 qubits, use multi-controlled X with ancilla target
    if n == 2:
        qc.cz(0, 1)
    else:
        # Multi-controlled Z via CCX chain
        qc.h(n-1)
        qc.mcx(list(range(n-1)), n-1)
        qc.h(n-1)
    # Undo bit flips
    for i, bit in enumerate(reversed(target)):
        if bit == "0":
            qc.x(i)
    return qc

def diffusion_operator(n):
    """Grover diffusion: 2|s><s| - I."""
    qc = QuantumCircuit(n)
    qc.h(range(n))    # H^n
    qc.x(range(n))    # X^n
    qc.h(n-1)         # Prepare for multi-controlled-Z
    qc.mcx(list(range(n-1)), n-1)
    qc.h(n-1)
    qc.x(range(n))    # X^n
    qc.h(range(n))    # H^n
    return qc

def grover_circuit(n, target, k=None):
    """Full Grover circuit."""
    N = 2**n
    M = 1  # single marked item
    theta = np.arcsin(np.sqrt(M/N))
    if k is None:
        k = max(1, int(np.round(np.pi / (4*theta))))
    print(f"n={n}, N={N}, theta={theta:.4f} rad, k_opt={k}")

    qc = QuantumCircuit(n, n)
    # Step 1: Uniform superposition
    qc.h(range(n))
    # Step 2: Grover iterations
    oracle = phase_oracle(n, target)
    diffusion = diffusion_operator(n)
    for iteration in range(k):
        qc.compose(oracle, inplace=True)
        qc.compose(diffusion, inplace=True)
    # Step 3: Measure
    qc.measure(range(n), range(n))
    return qc

# ── Run Grover search for target "1011" in n=4 qubits (N=16) ─────────
n, target = 4, "1011"
qc = grover_circuit(n, target)

sim = AerSimulator()
tqc = transpile(qc, sim)
result = sim.run(tqc, shots=2048).result()
counts = result.get_counts()

# Sort by count and display top results
sorted_counts = sorted(counts.items(), key=lambda x: -x[1])
print(f"Target: {target}")
print("Top 5 measurement outcomes:")
for state, count in sorted_counts[:5]:
    pct = 100*count/2048
    marker = ' <-- TARGET' if state == target else ''
    print(f"  |{state}> : {count:5d} shots ({pct:.1f}%){marker}")
# Expected: |1011> appears ~96% of the time (k_opt=3, N=16)
```

<div class="box box-example">
<p class="box-title"><strong>Example 6.2: Multi-Target Grover — Two Marked Items</strong></p>
<p>Problem: N=8 items, M=2 marked items (targets |011⟩ and |110⟩). Compute k_opt and the success probability.</p>
<p><strong>Solution:</strong></p>
<p>θ = arcsin(√(M/N)) = arcsin(√(2/8)) = arcsin(1/√4) = arcsin(0.5) = π/6 ≈ 0.5236 rad</p>
<p>k_opt = round(π/(4·π/6)) = round(π·6/(4π)) = round(1.5) = 2</p>
<p>State after k=2 iterations:</p>
<p>P(success) = sin²((2·2+1)·π/6) = sin²(5π/6) = sin²(π − π/6) = sin²(π/6) = (1/2)² = 0.25</p>
<p>That gives P=25%! Something is wrong — let us check k=1:</p>
<p>P(success at k=1) = sin²(3·π/6) = sin²(π/2) = 1.0 = 100%!</p>
<p>Correction: k_opt = round(π/(4θ)) = round(π/(4·π/6)) = round(6/4) = round(1.5) = 2 by rounding convention, but k=1 gives exact success since (2·1+1)·π/6 = π/2.</p>
<p>The formula k_opt = round(π/(4θ)) can round to k=2 when k=1 is better. Always check both k_opt−1 and k_opt and use the one with higher sin²((2k+1)θ).</p>
</div>

## 6.2 Variational Quantum Algorithms (NISQ Era)

The NISQ era — characterised by quantum processors with 50–1000 noisy qubits, without full error correction — has motivated a fundamentally different algorithmic paradigm: Variational Quantum Algorithms (VQAs). Rather than executing a single, precisely-designed deep circuit (as in QPE or Shor's algorithm), VQAs use a shallow parametrised quantum circuit whose parameters are optimised by a classical computer. The quantum processor handles only the computationally hard part — evaluating expectation values of quantum states — while the classical computer handles the optimisation.

Two VQAs dominate current NISQ research: VQE (Variational Quantum Eigensolver) for chemistry and materials simulation, and QAOA (Quantum Approximate Optimisation Algorithm) for combinatorial optimisation. Both are hybrid quantum-classical algorithms that are designed to run on today's hardware and find practical applications with 50–1000-qubit devices.

### 6.2.1 The Hybrid Quantum-Classical Model

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Hybrid Quantum-Classical Architecture</strong></p>
<p>A hybrid quantum-classical algorithm has three components:</p>
<p>Classical optimiser: runs on a classical computer; proposes parameter updates based on measurements</p>
<p>Parametrised quantum circuit (ansatz): a quantum circuit U(θ) whose gates depend on classical parameters θ ∈ ℝᵖ; runs on quantum hardware</p>
<p>Observable measurement: the quantum processor estimates ⟨ψ(θ)|O|ψ(θ)⟩ = ⟨O⟩_θ for a Hermitian observable O; this gives the cost function</p>
<p>The outer loop: the classical optimiser minimises E(θ) = ⟨ψ(θ)|H|ψ(θ)⟩ over θ by repeatedly evaluating the cost function on the quantum processor. Each evaluation requires O(shots) circuit runs; each gradient requires O(p) evaluations (p = number of parameters).</p>
</div>

The key insight: for many problems of practical interest (molecular energies, portfolio optimisation, graph partitioning), the relevant observable H can be expressed as a sum of Pauli operators: H = Σᵢ cᵢ Pᵢ where each Pᵢ is a tensor product of Pauli matrices. Estimating ⟨H⟩ requires measuring each Pauli term separately, but the circuits are shallow and can be run on NISQ hardware today.

<figure class="book-figure">
<img src="content/images/image75.png" alt="Figure 6.4: VQE hybrid quantum-classical architecture. The variational loop: the classical optimiser proposes new parameters θ; the ansatz circuit prepares |ψ(θ)⟩ on the quantum processor; the Estimator primitive evaluates ⟨H⟩ = ⟨ψ(θ)|H|ψ(θ)⟩; the classical optimiser uses E(θ) to update θ. Loop continues until convergence.">
<figcaption>Figure 6.4: VQE hybrid quantum-classical architecture. The variational loop: the classical optimiser proposes new parameters θ; the ansatz circuit prepares |ψ(θ)⟩ on the quantum processor; the Estimator primitive evaluates ⟨H⟩ = ⟨ψ(θ)|H|ψ(θ)⟩; the classical optimiser uses E(θ) to update θ. Loop continues until convergence.</figcaption>
</figure>

### 6.2.2 Variational Quantum Eigensolver (VQE)

#### The Variational Principle

VQE is based on the quantum mechanical variational principle: for any Hamiltonian H with ground state energy E₀ and any normalised state |ψ⟩:

<div class="box box-equation">
<p><strong>Equation 6.6</strong></p>
<p><strong>E(ψ) = ⟨ψ|H|ψ⟩ ≥ E₀   for all |ψ⟩</strong></p>
<p>Equality holds iff |ψ⟩ is the ground state |E₀⟩</p>
</div>

VQE exploits this: by parameterising |ψ⟩ = |ψ(θ)⟩ via a quantum circuit U(θ)|0⟩, and minimising E(θ) = ⟨ψ(θ)|H|ψ(θ)⟩ over parameters θ, we obtain an upper bound on E₀ that tightens as the ansatz becomes more expressive. The minimum over θ gives the best approximation to E₀ within the ansatz family.

#### VQE Workflow

- Step 1: Map the physical problem to a qubit Hamiltonian H = Σᵢ cᵢ Pᵢ (Pauli decomposition)

- Step 2: Choose an ansatz circuit U(θ) with p tunable parameters

- Step 3: Initialise parameters θ randomly (or with a good initial guess)

- Step 4: Prepare |ψ(θ)⟩ = U(θ)|0⟩ on quantum hardware

- Step 5: Measure ⟨H⟩ = Σᵢ cᵢ ⟨Pᵢ⟩ using the Estimator primitive

- Step 6: Classical optimiser updates θ → θ + δθ to minimise E(θ)

- Step 7: Repeat Steps 4–6 until convergence (|E(θ\_{k+1}) − E(θ\_k)| < ε)

- Step 8: Return E\_min ≈ E₀ and optimal |ψ(θ\*)⟩ ≈ |E₀⟩

### 6.2.3 Ansatz Circuit Design

#### What Makes a Good Ansatz?

The ansatz circuit U(θ) must balance three competing requirements: expressibility (can it represent the target ground state?), trainability (can the classical optimiser find the minimum efficiently?), and hardware efficiency (is it shallow enough to run on NISQ hardware?). No single ansatz satisfies all three perfectly.

<figure class="book-figure">
<img src="content/images/image76.png" alt="Figure 6.5: Ansatz circuits. Left: Hardware-Efficient Ansatz (HEA) — alternating layers of single-qubit Ry rotations and entangling CNOT gates. Simple, shallow, hardware-native, but may not match the true ground state structure. Right: UCCSD (Unitary Coupled Cluster) ansatz — physically motivated excitation operators matching the molecular problem; better expressibility, deeper circuit.">
<figcaption>Figure 6.5: Ansatz circuits. Left: Hardware-Efficient Ansatz (HEA) — alternating layers of single-qubit Ry rotations and entangling CNOT gates. Simple, shallow, hardware-native, but may not match the true ground state structure. Right: UCCSD (Unitary Coupled Cluster) ansatz — physically motivated excitation operators matching the molecular problem; better expressibility, deeper circuit.</figcaption>
</figure>

#### Hardware-Efficient Ansatz (HEA)

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Hardware-Efficient Ansatz (HEA)</strong></p>
<p>Structure: p alternating layers of single-qubit rotations + CNOT entangling gates.</p>
<p>A typical depth-L HEA on n qubits:</p>
<p>L layers, each containing n Ry(θ) gates followed by a CNOT entangling pattern</p>
<p>Total parameters: n × L (one Ry angle per qubit per layer)</p>
<p>Optional: add Rz rotations (doubled parameters); use hardware-native entangling gates</p>
<p>Advantages: very shallow, uses only native gates, easily implementable on any hardware.</p>
<p>Disadvantages: may lack expressibility for chemistry problems; can suffer from "barren plateaus" — flat gradient landscapes where classical optimisation stalls.</p>
</div>

#### Chemistry-Inspired Ansatz: UCCSD

For molecular simulation, the Unitary Coupled Cluster Singles and Doubles (UCCSD) ansatz is the standard choice. It is motivated by the coupled-cluster ansatz of classical quantum chemistry:

<div class="box box-equation">
<p><strong>Equation 6.7</strong></p>
<p>|ψ_UCCSD(θ)⟩ = exp(T(θ) − T†(θ)) |HF⟩</p>
<p>where T(θ) = Σ_{ia} θ_{ia} a†_a a_i + Σ_{ijab} θ_{ijab} a†_a a†_b a_j a_i</p>
<p>|HF⟩ is the Hartree-Fock reference state (occupied orbitals filled)</p>
</div>

The UCCSD ansatz includes all single (T₁) and double (T₂) excitations from occupied to virtual orbitals. Each parameter θ\_{ia} or θ\_{ijab} corresponds to a specific electron excitation amplitude. This ansatz is physically motivated — it captures the dominant correlation effects in most molecules — and converges to the correct ground state energy for small molecules.

Disadvantage: the circuit depth scales as O(N⁴) in the number of basis functions N, making UCCSD impractical for large molecules on NISQ hardware. Hardware-efficient approximations (k-UpCCGSD, ADAPT-VQE) address this.

#### Barren Plateaus: The Trainability Crisis

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: Barren Plateaus in Deep Ansatz Circuits</strong></p>
<p>A barren plateau is a region of parameter space where the gradient of E(θ) vanishes exponentially with the number of qubits. Formally: Var[∂E/∂θⱼ] ∝ e^(−cn) for some c &gt; 0. In such regions, the optimiser cannot determine which direction to move and the algorithm stalls.</p>
<p>Barren plateaus occur when: (1) the ansatz is too deep (random circuit ansatz with depth O(n)), (2) the observable is non-local (global observables like ⟨H⟩ for random Hamiltonians), or (3) the initial parameters are random in a large range. They are a serious practical obstacle for VQE at scale.</p>
<p>Mitigation strategies: use shallow depth-p circuits; use local observables; initialise near the identity (θ ≈ 0); use problem-specific structured ansatz (UCCSD, QAOA); use layer-by-layer training.</p>
</div>

### 6.2.4 QAOA: Quantum Approximate Optimisation Algorithm

#### The MaxCut Problem

QAOA (Farhi, Goldstone, Gutmann, 2014) targets combinatorial optimisation problems, beginning with MaxCut. Given an undirected graph G=(V,E), partition the vertices into two sets S and S̄ to maximise the number of edges crossing the partition (the "cut").

<div class="box box-equation">
<p><strong>Equation 6.8</strong></p>
<p>MaxCut objective: C(z) = Σ_{(i,j)∈E} (1 − zᵢzⱼ)/2   where z ∈ {−1,+1}^n</p>
<p>Qubit encoding: zᵢ = (−1)^{bᵢ}  where bᵢ ∈ {0,1} is the cut assignment of vertex i</p>
<p><strong>Quantum Hamiltonian: H_C = Σ_{(i,j)∈E} (I − ZᵢZⱼ)/2  (cost Hamiltonian)</strong></p>
</div>

MaxCut is NP-hard — no polynomial classical algorithm is known to find the optimal cut in general. The classical Goemans-Williamson algorithm achieves approximation ratio ≈ 0.878 (guaranteed). QAOA at depth p is conjectured (but not proven) to approach optimal as p → ∞.

#### The QAOA Circuit

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: QAOA Circuit (depth p)</strong></p>
<p>Starting from |+⟩^n = H^⊗n|0⟩^n, QAOA alternates between cost and mixer unitaries:</p>
<p>The 2p parameters (γ₁,...,γ_p, β₁,...,β_p) are optimised classically to maximise ⟨ψ|H_C|ψ⟩.</p>
<p>Implementation: U_C term exp(−iγ ZᵢZⱼ/2) = CNOT · Rz(γ) · CNOT (standard 2-qubit ZZ rotation). U_M term exp(−iβ Xᵢ) = Rx(2β) (single-qubit x-rotation). Total gates: O(p·|E|) CNOTs + O(p·n) Rx gates.</p>
</div>

<figure class="book-figure">
<img src="content/images/image77.png" alt="Figure 6.6: QAOA for MaxCut. Left: 4-node MaxCut instance — nodes 0,1,2,3 connected by 5 edges; the optimal cut (dashed red) separates node 0 from {1,2,3}. Right: QAOA circuit for p=1 — all qubits start in |+⟩; ZZ rotations implement cost Hamiltonian for each edge; Rx rotations implement the mixer Hamiltonian.">
<figcaption>Figure 6.6: QAOA for MaxCut. Left: 4-node MaxCut instance — nodes 0,1,2,3 connected by 5 edges; the optimal cut (dashed red) separates node 0 from {1,2,3}. Right: QAOA circuit for p=1 — all qubits start in |+⟩; ZZ rotations implement cost Hamiltonian for each edge; Rx rotations implement the mixer Hamiltonian.</figcaption>
</figure>

#### QAOA Approximation Ratio

At depth p=1, QAOA guarantees an approximation ratio of at least 0.692 for MaxCut on any 3-regular graph — better than random guessing (0.5) but worse than Goemans-Williamson (0.878). At p → ∞, QAOA converges to the exact optimum (this is the adiabatic limit). For practical NISQ hardware, p=1,2,3 is typical.

Key result (Farhi et al., 2014): the p=1 QAOA circuit for MaxCut on an unweighted triangle-free graph achieves expected cut value at least 0.6924 × OPT.

```python
# ─────────────────────────────────────────────────────────────────────
# Code 6.2: QAOA for MaxCut — Complete Qiskit Implementation (p=1)
# Graph: 4 nodes, edges (0,1),(1,2),(2,3),(3,0),(0,2)
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit.quantum_info import SparsePauliOp
import numpy as np
from scipy.optimize import minimize

# ── Graph definition ────────────────────────────────────────────────
n_nodes = 4
edges = [(0,1),(1,2),(2,3),(3,0),(0,2)]

# ── Build cost Hamiltonian H_C = Σ_{(i,j)} (I - Z_i Z_j) / 2 ────────
pauli_list = []
for (i, j) in edges:
    # ZᵢZⱼ term: Pauli string with Z at positions i and j
    pauli_str = ["I"] * n_nodes
    pauli_str[i] = "Z"
    pauli_str[j] = "Z"
    pauli_list.append(("".join(reversed(pauli_str)), -0.5))  # -ZZ/2
    pauli_list.append(("I" * n_nodes, 0.5))                   # +I/2
H_C = SparsePauliOp.from_list(pauli_list)

# ── Build QAOA circuit (p=1) ─────────────────────────────────────────
def qaoa_circuit(gamma, beta):
    qc = QuantumCircuit(n_nodes)
    # Initial state: |+>^n
    qc.h(range(n_nodes))
    # Cost unitary U_C(gamma)
    for (i, j) in edges:
        # exp(-i*gamma*Z_i*Z_j/2) = CNOT · Rz(gamma) · CNOT
        qc.cx(i, j)
        qc.rz(gamma, j)
        qc.cx(i, j)
    # Mixer unitary U_M(beta): Rx(2*beta) on each qubit
    qc.rx(2 * beta, range(n_nodes))
    return qc

# ── Expectation value evaluation using Estimator ──────────────────────
sim = AerSimulator()

def expectation(params):
    gamma, beta = params
    qc = qaoa_circuit(gamma, beta)
    # Transpile and simulate
    tqc = transpile(qc, sim)
    # Sample-based expectation: measure in Z basis for each Pauli term
    qc_meas = qc.copy()
    qc_meas.measure_all()
    tqc_meas = transpile(qc_meas, sim)
    result = sim.run(tqc_meas, shots=4096).result()
    counts = result.get_counts()
    # Compute cut value from counts
    total_cut = 0
    total_shots = sum(counts.values())
    for bitstring, count in counts.items():
        # Count cut edges for this assignment
        cut = sum(1 for (i,j) in edges
                  if bitstring[n_nodes-1-i] != bitstring[n_nodes-1-j])
        total_cut += cut * count
    return -(total_cut / total_shots)  # Negative for minimisation

# ── Classical optimisation ───────────────────────────────────────────
from scipy.optimize import minimize
result = minimize(expectation, x0=[0.5, 0.5],
                  method="COBYLA", options={"maxiter": 200})
gamma_opt, beta_opt = result.x
max_cut = -result.fun

print(f"Optimal gamma = {gamma_opt:.4f}, beta = {beta_opt:.4f}")
print(f"Expected cut value = {max_cut:.3f} (max possible = 4)")

# ── Sample the optimal circuit ───────────────────────────────────────
qc_opt = qaoa_circuit(gamma_opt, beta_opt)
qc_opt.measure_all()
counts_opt = sim.run(transpile(qc_opt, sim), shots=2048).result().get_counts()
top = sorted(counts_opt.items(), key=lambda x: -x[1])[:3]
print("Top solutions (bitstring = cut assignment):")
for bs, cnt in top:
    cut = sum(1 for (i,j) in edges
              if bs[n_nodes-1-i] != bs[n_nodes-1-j])
    print(f"  {bs}: cut={cut}, count={cnt}")
```

### 6.2.5 Parameter-Shift Gradient Rule

#### The Need for Quantum-Compatible Gradients

Classical neural network training uses backpropagation — automatic differentiation through the computational graph. This requires access to intermediate activations and is incompatible with quantum hardware (measuring a quantum state collapses it). Quantum circuits need a hardware-compatible gradient method.

The parameter-shift rule provides exact gradients using only circuit evaluations — no backpropagation, no access to internal states. It works for any gate of the form G(θ) = exp(−iθP/2) where P is a Pauli operator (X, Y, or Z). This covers all standard rotation gates: Rx(θ), Ry(θ), Rz(θ).

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Parameter-Shift Rule</strong></p>
<p>For a circuit U(θ) with a gate G_j(θ_j) = exp(−iθ_j P_j/2), the gradient of any expectation value E(θ) = ⟨ψ(θ)|O|ψ(θ)⟩ with respect to parameter θ_j is:</p>
<p>Proof: For G(θ) = exp(−iθP/2), the expectation E(θ) = cos(θ)·A + sin(θ)·B for some A,B independent of θ. Therefore dE/dθ = −sin(θ)·A + cos(θ)·B. Evaluating at θ±π/2: E(θ+π/2) = −sin(θ)·A + cos(θ)·B = dE/dθ. Thus dE/dθ = [E(θ+π/2) − E(θ−π/2)]/2. ∎</p>
<p>For p parameters: total gradient computation requires 2p circuit evaluations. This is the quantum analogue of forward-mode automatic differentiation.</p>
</div>

<figure class="book-figure">
<img src="content/images/image78.png" alt="Figure 6.7: Parameter-shift rule and VQE convergence. Left: the energy landscape E(θ) for a single-parameter circuit — gradient at θ is computed from two circuit evaluations at θ±π/2. Right: VQE convergence curve — energy decreases from initial random parameters toward the exact ground state energy E₀ = −1.136 Ha; convergence to chemical accuracy (|E_VQE − E₀| &lt; 1.6×10⁻³ Ha) is achieved.">
<figcaption>Figure 6.7: Parameter-shift rule and VQE convergence. Left: the energy landscape E(θ) for a single-parameter circuit — gradient at θ is computed from two circuit evaluations at θ±π/2. Right: VQE convergence curve — energy decreases from initial random parameters toward the exact ground state energy E₀ = −1.136 Ha; convergence to chemical accuracy (|E_VQE − E₀| &lt; 1.6×10⁻³ Ha) is achieved.</figcaption>
</figure>

#### Classical Optimisers for VQE

The choice of classical optimiser significantly affects VQE performance:

| Optimiser | Type | Gradient? | Best Use |
|---|---|---|---|
| COBYLA | Gradient-free (simplex) | No | Noisy hardware, few params (<20) |
| SPSA | Gradient-free (stochastic) | No | Noisy hardware, many params (>20) |
| L-BFGS-B | Quasi-Newton (gradient-based) | Yes (param-shift) | Simulator, moderate noise |
| Adam | Adaptive learning rate | Yes (param-shift) | Simulation; quantum ML |
| NFT | Non-linear fitting | No | Suited for periodic params |

### 6.2.6 Qiskit Runtime Estimator for Expectation Values

The Qiskit Runtime Estimator primitive is the standard tool for VQE on IBM Quantum hardware. It handles Pauli observable decomposition, shot allocation, error mitigation, and parallel circuit execution automatically.

```python
# ─────────────────────────────────────────────────────────────────────
# Code 6.3: VQE for H₂ ground state using Qiskit Runtime Estimator
# Uses hardware-efficient ansatz and parameter-shift gradients
# ─────────────────────────────────────────────────────────────────────
import numpy as np
from qiskit import QuantumCircuit
from qiskit.circuit import ParameterVector
from qiskit.quantum_info import SparsePauliOp
from qiskit_ibm_runtime.fake_provider import FakeSherbrookeV2
from qiskit_ibm_runtime import EstimatorV2, Session
from qiskit import transpile
from scipy.optimize import minimize

# ── H₂ Hamiltonian at bond length 0.735 Angstrom (STO-3G basis) ──────
# H = g₀I + g₁Z₀ + g₂Z₁ + g₃Z₀Z₁ + g₄X₀X₁ + g₅Y₀Y₁
H2_hamiltonian = SparsePauliOp.from_list([
    ("II",  -1.0523732),   # constant term
    ("IZ",   0.3979374),   # Z on qubit 0
    ("ZI",  -0.3979374),   # Z on qubit 1
    ("ZZ",  -0.0112801),   # ZZ correlation
    ("XX",   0.1809312),   # XX correlation (entanglement)
    ("YY",   0.1809312),   # YY correlation (entanglement)
])
print(f"H2 Hamiltonian: {len(H2_hamiltonian)} Pauli terms")
print(f"Exact ground state energy: -1.136189 Hartree")

# ── Hardware-Efficient Ansatz (2 qubits, 2 layers) ───────────────────
theta = ParameterVector("theta", length=6)  # 2 qubits × 3 Ry layers
ansatz = QuantumCircuit(2)
ansatz.ry(theta[0], 0); ansatz.ry(theta[1], 1)  # Layer 0
ansatz.cx(0, 1)                                   # Entangle
ansatz.ry(theta[2], 0); ansatz.ry(theta[3], 1)  # Layer 1
ansatz.cx(0, 1)
ansatz.ry(theta[4], 0); ansatz.ry(theta[5], 1)  # Layer 2

# ── Set up backend and Estimator ──────────────────────────────────────
backend = FakeSherbrookeV2()
tansatz = transpile(ansatz, backend, optimization_level=2)

# ── VQE with parameter-shift gradients ───────────────────────────────
evaluations = [0]

def energy(params):
    """Evaluate E(θ) = ⟨ψ(θ)|H|ψ(θ)⟩ using Estimator."""
    param_dict = dict(zip(tansatz.parameters, params))
    bound_circuit = tansatz.assign_parameters(param_dict)
    with Session(backend=backend) as session:
        estimator = EstimatorV2(mode=session)
        pub = (bound_circuit, H2_hamiltonian)
        job = estimator.run([pub])
        result = job.result()
    ev = float(result[0].data.evs)
    evaluations[0] += 1
    return ev

def gradient(params):
    """Compute gradient using parameter-shift rule."""
    grad = np.zeros_like(params)
    shift = np.pi / 2
    for j in range(len(params)):
        params_plus = params.copy(); params_plus[j] += shift
        params_minus = params.copy(); params_minus[j] -= shift
        grad[j] = (energy(params_plus) - energy(params_minus)) / 2
    return grad

# ── Optimise ──────────────────────────────────────────────────────────
np.random.seed(42)
theta_init = np.random.uniform(0, 2*np.pi, 6)
result_vqe = minimize(energy, theta_init, method="L-BFGS-B",
                      jac=gradient, options={'maxiter': 100})

print(f"\nVQE Result:")
print(f"  E_VQE   = {result_vqe.fun:.6f} Hartree")
print(f"  E_exact = -1.136189 Hartree")
print(f"  Error   = {abs(result_vqe.fun + 1.136189):.6f} Hartree")
print(f"  Circuit evaluations: {evaluations[0]}")
# Expected error: < 0.002 Hartree (chemical accuracy = 1.6e-3 Ha)
```

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: VQE for Drug Discovery: Current Status and Timeline</strong></p>
<p>VQE is the primary near-term quantum algorithm for pharmaceutical applications. Key milestones: (1) 2016: Google and Harvard demonstrated VQE for H₂ molecule on a photonic chip. (2) 2020: Google computed the energy of H₂ and HeH⁺ with chemical accuracy using a 12-qubit processor. (3) 2022: IBM demonstrated VQE for a 4-atom molecular system (BeH₂ dissociation) with error mitigation on a 7-qubit device. Current limitation: molecules relevant to drug discovery (e.g., penicillin, 1200 atoms) require thousands of logical qubits — beyond current and near-term hardware. Realistic timeline for drug-relevant quantum chemistry: 2030–2035 for fault-tolerant devices. NISQ-era VQE is primarily a research tool for algorithm development, not yet for practical pharmaceutical applications.</p>
</div>

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 6: Grover's Search Algorithm and Variational Quantum Algorithms

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Define the unstructured search problem. State the classical and quantum query complexities for M marked items in N = 2^n items. What does 'quadratic speedup' mean precisely?**

**Q2.  Describe the phase oracle in Grover's algorithm. Write its mathematical definition and explain how it differs from the bit-flip oracle studied in Chapter 5.**

**Q3.  Define the Grover diffusion operator D = 2|s⟩⟨s| − I. Write the circuit decomposition of D. What is its geometric interpretation?**

**Q4.  Trace one complete Grover iteration (oracle + diffusion) starting from initial state |s⟩ = (1/√N)Σ|x⟩ with M=1, N=16. Show α and β before and after each step.**

**Q5.  Derive the general formula β\_k = (1/√M) sin((2k+1)θ) for the amplitude of marked states after k Grover iterations. What is the success probability P(success) after k iterations?**

**Q6.  Prove that the optimal number of Grover iterations is k\_opt ≈ π√(N/M)/4. What is the success probability at k\_opt for M=1, N=256? What happens at k = 2k\_opt?**

**Q7.  Explain the geometric interpretation of Grover's algorithm as rotation in a 2D subspace. Define the two basis vectors of this subspace and state the angle 2θ rotated per iteration.**

**Q8.  Why does Grover's algorithm NOT make NP-complete problems tractable? Use 3-SAT with n=100 variables as a concrete example with explicit numerical estimates.**

**Q9.  Define a Variational Quantum Algorithm (VQA). What are its three components? Describe the hybrid quantum-classical loop and explain why VQA is suitable for NISQ hardware.**

**Q10.  State the variational principle underlying VQE. Why does ⟨ψ(θ)|H|ψ(θ)⟩ ≥ E₀ for all |ψ(θ)⟩? How does minimising this over θ approximate the ground state energy?**

**Q11.  What is a hardware-efficient ansatz (HEA) in VQE? Describe its structure (layers, gate types). What is the trade-off between HEA and a chemically inspired UCCSD ansatz?**

**Q12.  Describe the QAOA algorithm for the MaxCut problem. Define the cost Hamiltonian H\_C and the mixer Hamiltonian H\_B. How does QAOA at depth p relate to the exact solution as p → ∞?**

**Q13.  State the parameter-shift gradient rule. Prove it (or at least show the key step) for a parametrised gate exp(−iθP/2). How does it compare to classical finite-difference gradient estimation?**

**Q14.  What are barren plateaus in variational quantum algorithms? Why do they occur for deep circuits and large qubit counts? What strategies mitigate them?**

**Q15.  A VQE run uses 50 parameters and 100 shots per circuit evaluation. (a) How many circuit evaluations are needed per gradient? (b) If each circuit takes 1 second on hardware, how long does one gradient step take? (c) If 500 gradient steps are needed, estimate total hardware time.**

**PART B — MODEL ANSWERS**

**Answer 1:**

Problem: given oracle O\_f for f:{0,...,N-1}→{0,1} with M marked items (f(x\*)=1), find any x\*. Classical query complexity: Θ(N/M) expected queries (on average check N/M items before finding a marked one). Quantum query complexity: Θ(√(N/M)) — provably optimal by the BBBV theorem (1997). Quadratic speedup means the query count scales as √N rather than N: the ratio of classical to quantum queries is √(N/M), which grows as the square root of N/M.

**Answer 2:**

Phase oracle: O\_f|x⟩ = (−1)^{f(x)}|x⟩. Applied to all basis states simultaneously: marks marked states (f(x)=1) with a phase −1 while leaving unmarked states (f(x)=0) unchanged. Difference from bit-flip oracle: bit-flip oracle O\_f|x⟩|b⟩=|x⟩|b⊕f(x)⟩ writes f(x) into an ancilla register, requiring an extra qubit. Phase oracle acts only on the input register, encoding f(x) as a phase rather than in a separate qubit. The phase oracle is obtained from the bit-flip oracle via phase kickback (Chapter 5).

**Answer 3:**

Grover diffusion operator: D = 2|s⟩⟨s| − I, where |s⟩=(1/√N)Σ\_x|x⟩. Circuit: D = H^{⊗n}·(2|0⟩⟨0|−I)·H^{⊗n} = H^{⊗n}·X^{⊗n}·(multi-controlled Z)·X^{⊗n}·H^{⊗n}. Action: (D|v⟩)\_x = 2μ − v\_x where μ = (1/N)Σ\_x v\_x is the mean amplitude — 'inversion about the mean'. Geometric interpretation: reflection about the uniform superposition |s⟩ in the n-dimensional amplitude vector space.

**Answer 4:**

Initial: α₀ = 1/√N = 1/4, β₀ = 1/4 (N=16, uniform). After oracle (phase flip marked state): α₁ = +1/4, β₁ = −1/4. Mean: μ₁ = [(N-1)/4 + (−1/4)]/N = [15/4 − 1/4]/16 = [14/4]/16 = 14/64 = 7/32. After diffusion (inversion about mean): α₂ = 2·(7/32) − 1/4 = 14/32 − 8/32 = 6/32 = 3/16. β₂ = 2·(7/32) − (−1/4) = 14/32 + 8/32 = 22/32 = 11/16. P(marked) = β₂² = (11/16)² ≈ 0.473. Starting from P=1/16=6.25%, one iteration gives P≈47.3% — a 7.6× increase.

**Answer 5:**

Decompose state into |marked⟩ = (1/√M)Σ\_{x:f=1}|x⟩ and |unmarked⟩ components. State after k iterations: |ψ\_k⟩ = sin((2k+1)θ)|marked⟩ + cos((2k+1)θ)|unmarked⟩, where sin(θ) = √(M/N). Amplitude of each marked state: β\_k = (1/√M)·sin((2k+1)θ) [projecting onto individual marked state]. Success probability P = M·β\_k² = M·(1/M)·sin²((2k+1)θ) = sin²((2k+1)θ). This oscillates sinusoidally with k, peaking at P=1 when (2k+1)θ = π/2.

**Answer 6:**

P(success) = sin²((2k+1)θ) = 1 when (2k+1)θ = π/2, giving k = (π/(2θ)−1)/2. Since θ = arcsin(1/√N) ≈ 1/√N for large N: k\_opt ≈ π/(4θ) ≈ π√N/4. For M=1, N=256: θ = arcsin(1/16) ≈ 0.0625 rad; k\_opt = round(π/(4×0.0625)) = round(12.57) = 12. P(success at k=12) = sin²(25×0.0625) = sin²(1.5625) ≈ 1.000 ≈ 100%. At k = 2k\_opt = 24: P = sin²(49×0.0625) = sin²(3.0625) ≈ sin²(π−0.079) ≈ 0.006 ≈ 0.6% — near zero due to over-rotation.

**Answer 7:**

Geometric picture: the quantum state evolves in a 2D subspace spanned by |t⟩ = (1/√M)Σ\_{marked}|x⟩ (uniform superposition of target states) and |u⟩ = (1/√(N-M))Σ\_{unmarked}|x⟩. Initial state |s⟩ = sin(θ)|t⟩ + cos(θ)|u⟩ is close to |u⟩ (angle θ from |u⟩). Each Grover iteration rotates the state by 2θ toward |t⟩: the oracle reflects about |u⟩, the diffusion reflects about |s⟩, and the composition of two reflections is a rotation by 2θ. After k\_opt iterations, state ≈ |t⟩.

**Answer 8:**

Grover reduces 2^n-element search to O(2^{n/2}) queries — still exponential in n/2. For 3-SAT, n=100: search space = 2^{100} possible assignments. Grover iterations: k = π√(2^{100})/4 ≈ π×2^{50}/4 ≈ 2.5×10^{14} oracle calls. At 10^{15} operations/second: time ≈ 0.25 seconds... wait, but each oracle call requires simulating 3-SAT verification which itself may be expensive. More importantly: 2^{50} ≈ 10^{15} oracle calls at any reasonable rate still takes impractical time for large instances. The BBBV theorem proves O(√N) is optimal — NP ⊄ BQP is widely believed.

**Answer 9:**

VQA components: (1) Parametrised quantum circuit (ansatz) |ψ(θ)⟩ with parameters θ ∈ ℝ^m. (2) Cost function C(θ) = ⟨ψ(θ)|H\_cost|ψ(θ)⟩ measured on quantum hardware. (3) Classical optimiser (COBYLA, L-BFGS, Adam) that updates θ to minimise C. Hybrid loop: classical computer sends θ → quantum device prepares |ψ(θ)⟩ and measures C(θ) → classical optimiser computes ∇C and updates θ → repeat. Suitable for NISQ: shallow circuit depth (polynomial in layers), error resilience (optimisation partially compensates errors), hardware-native gate structure.

**Answer 10:**

Variational principle: for any Hermitian H with ground state |E₀⟩ and eigenvalue E₀, and for any normalised state |ψ⟩: ⟨ψ|H|ψ⟩ = Σᵢ |⟨Eᵢ|ψ⟩|² Eᵢ ≥ E₀ Σᵢ |⟨Eᵢ|ψ⟩|² = E₀. Equality only when |ψ⟩ = |E₀⟩. VQE minimises ⟨ψ(θ)|H|ψ(θ)⟩ over θ, approaching E₀ from above. The minimised value is the best approximation achievable given the expressiveness of the ansatz. Quality limited by ansatz expressiveness — if true |E₀⟩ cannot be represented as |ψ(θ)⟩ for any θ, the minimum may exceed E₀.

**Answer 11:**

Hardware-efficient ansatz (HEA): alternating layers of single-qubit Ry,Rz rotations and nearest-neighbour CNOT entangling gates, designed to match hardware connectivity. Structure: L layers of [Ry(θ)⊗n + Rz(φ)⊗n + CNOT(i,i+1)], total parameters 2nL. Advantage: low circuit depth (L≪n), compatible with hardware topology. Disadvantage: may not converge to the true ground state if it cannot represent |E₀⟩ (limited expressiveness); prone to barren plateaus for large L. UCCSD ansatz: chemically motivated unitary coupled cluster expansion — more likely to represent the true ground state, but T-count scales as O(n⁴) — impractical for NISQ hardware.

**Answer 12:**

MaxCut QAOA: partition graph vertices V into two sets to maximise edges crossing the cut. Cost Hamiltonian: H\_C = Σ\_{(i,j)∈E} (I − ZᵢZⱼ)/2 — diagonal in computational basis, value = number of cut edges. Mixer Hamiltonian: H\_B = Σᵢ Xᵢ — generates transitions between bitstrings. QAOA circuit: alternates p layers of exp(−iγ\_j H\_C) (cost rotation) and exp(−iβ\_j H\_B) (mixer rotation). Parameters {γ\_j, β\_j} optimised classically to maximise ⟨H\_C⟩. As p→∞: QAOA approximation ratio approaches 1 (exact MaxCut). For p=1: ≥0.6924 approximation ratio (provable lower bound).

**Answer 13:**

Parameter-shift rule: ∂C/∂θ\_k = [C(θ\_k+π/2) − C(θ\_k−π/2)]/2. Key step: for gate G\_k(θ\_k) = exp(−iθ\_k P\_k/2) where P\_k is a Pauli, the expectation C(θ\_k) = A cos(θ\_k) + B sin(θ\_k) + const (a trigonometric polynomial). Derivative: ∂C/∂θ\_k = −A sin(θ\_k) + B cos(θ\_k) = [C(θ\_k+π/2) − C(θ\_k−π/2)]/2 (by direct substitution and trigonometric addition formulas). Comparison with finite differences: finite differences ≈ [C(θ+ε)−C(θ−ε)]/(2ε) is an APPROXIMATION with error O(ε²) and numerical instability for small ε. Parameter-shift is EXACT and numerically stable.

**Answer 14:**

Barren plateaus: regions of parameter space where the gradient ∇C(θ) vanishes exponentially with the number of qubits n. Cause: for deep (L ≥ O(n)) random circuits, the unitary 2-design approximation makes all partial derivatives scale as O(1/2^n). Physical origin: the state |ψ(θ)⟩ explores the full Hilbert space uniformly, and the cost function becomes exponentially flat — changes in any parameter have exponentially small effect. Mitigation strategies: (1) layer-by-layer training (ADAPT-VQE); (2) structured ansatz with physical problem insight (reduces symmetry to avoid full Hilbert space exploration); (3) local cost functions (sum of local terms rather than global); (4) smaller circuits (fewer layers).

**Answer 15:**

(a) Per gradient: each of the 50 parameters needs 2 circuit evaluations (parameter-shift rule). Total per gradient: 50 × 2 = 100 circuit evaluations. (b) Time per gradient step: 100 circuits × 1 second/circuit = 100 seconds ≈ 1.67 minutes per gradient. (c) Total for 500 gradient steps: 500 × 100 seconds = 50,000 seconds ≈ 13.9 hours. This illustrates why hardware VQE runs are expensive: ~14 hours of QPU time for 500 gradient steps. Using batched execution (multiple circuits per job) and Session access can reduce this by 5-10× in practice.

## EXERCISES — CHAPTER 6

### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.1</strong></p>
<p>Problem: Prove that one Grover iteration (oracle O_f followed by diffusion D) is a rotation by angle 2θ in the 2D subspace spanned by |m⟩ and |w⟩, where sin(θ) = √(M/N).</p>
<p><strong>Solution:</strong></p>
<p>Write the state in the {|w⟩, |m⟩} basis as (cos φ)|w⟩ + (sin φ)|m⟩.</p>
<p>The initial state |s⟩ has φ = θ (since sin θ = √(M/N) = overlap with |m⟩).</p>
<p>Oracle O_f reflects about |w⟩: O_f[(cos φ)|w⟩ + (sin φ)|m⟩] = (cos φ)|w⟩ − (sin φ)|m⟩</p>
<p>This is a reflection of the angle: φ → −φ (reflection about x-axis in the 2D plane).</p>
<p>Diffusion D = 2|s⟩⟨s| − I reflects about |s⟩ = (cos θ)|w⟩ + (sin θ)|m⟩:</p>
<p>In the 2D plane, reflecting about a line at angle θ from the x-axis maps angle α to 2θ−α.</p>
<p>Composed operation: φ → −φ (oracle), then −φ → 2θ−(−φ) = 2θ+φ (diffusion).</p>
<p>Net effect: φ → φ + 2θ — the angle increases by 2θ per Grover iteration.</p>
<p>Starting from φ = θ, after k iterations: φ = θ + 2kθ = (2k+1)θ. ✓</p>
<p>The state approaches |m⟩ (φ = π/2) when (2k+1)θ ≈ π/2, giving k ≈ π/(4θ) = k_opt.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.2</strong></p>
<p>Problem: For Grover search with N=1024, M=4, compute (a) θ, (b) k_opt, (c) P(success at k_opt), (d) classical expected queries for comparison.</p>
<p><strong>Solution:</strong></p>
<p>(a) sin(θ) = √(M/N) = √(4/1024) = √(1/256) = 1/16. Therefore θ = arcsin(1/16) ≈ 0.06253 rad</p>
<p>(b) k_opt = round(π/(4θ)) = round(3.14159/(4×0.06253)) = round(3.14159/0.2501) = round(12.56) = 13</p>
<p>(c) P(success) = sin²((2×13+1)×0.06253) = sin²(27×0.06253) = sin²(1.6883) = sin²(π/2 − 0.1177)</p>
<p>≈ cos²(0.1177) ≈ 1 − 0.1177² / 2 ≈ 0.9931 ≈ 99.3%</p>
<p>(d) Classical expected queries = N/(M+1) ≈ 1024/5 ≈ 204 queries</p>
<p>Quantum: 13 oracle calls. Classical: ~204. Speedup factor ≈ 204/13 ≈ 15.7×.</p>
<p>Note: the quantum speedup is not exactly √(N/M) = √(256) = 16 because of the rounding in k_opt.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.3</strong></p>
<p>Problem: Implement the Grover diffusion operator for n=2 qubits as a matrix and verify it equals 2|s⟩⟨s| − I.</p>
<p><strong>Solution:</strong></p>
<p>|s⟩ = (|00⟩+|01⟩+|10⟩+|11⟩)/2. The outer product |s⟩⟨s| is a 4×4 matrix:</p>
<p>|s⟩⟨s| = (1/4) × [[1,1,1,1],[1,1,1,1],[1,1,1,1],[1,1,1,1]]</p>
<p>2|s⟩⟨s| − I = (1/2)[[1,1,1,1],[1,1,1,1],[1,1,1,1],[1,1,1,1]] − I</p>
<p>= [[-1/2, 1/2, 1/2, 1/2],[1/2,-1/2,1/2,1/2],[1/2,1/2,-1/2,1/2],[1/2,1/2,1/2,-1/2]]</p>
<p><strong>Verify via circuit decomposition D = H^⊗2 · (2|0⟩⟨0|−I) · H^⊗2:</strong></p>
<p>2|0⟩⟨0|−I = diag(1,−1,−1,−1) (diagonal matrix with +1 at |00⟩, −1 elsewhere)</p>
<p>H^⊗2 = (1/2)[[1,1,1,1],[1,−1,1,−1],[1,1,−1,−1],[1,−1,−1,1]]</p>
<p>D = H^⊗2 · diag(1,−1,−1,−1) · H^⊗2:</p>
<p>First: H^⊗2 · diag(1,−1,−1,−1) multiplies each column j≥1 by −1:</p>
<p>= (1/2)[[1,−1,−1,−1],[1,1,−1,1],[1,−1,1,1],[1,1,1,−1]] (after column scaling)</p>
<p>Then right-multiply by H^⊗2... (full calculation omitted for brevity)</p>
<p>Result: the final matrix equals 2|s⟩⟨s|−I shown above. ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.4</strong></p>
<p>Problem: For VQE with a 2-qubit ansatz containing 4 parameters, how many circuit evaluations are needed per gradient computation using the parameter-shift rule? How many total evaluations does a 100-iteration gradient descent require?</p>
<p><strong>Solution:</strong></p>
<p>Parameter-shift rule: 2 evaluations per parameter per gradient computation.</p>
<p>With 4 parameters: gradient requires 2 × 4 = 8 evaluations per iteration.</p>
<p>Each gradient descent iteration also needs 1 evaluation for the current energy.</p>
<p>Total per iteration: 1 (energy) + 8 (gradient) = 9 evaluations.</p>
<p>For 100 iterations: 100 × 9 = 900 circuit evaluations.</p>
<p>Comparison with finite differences: 2p = 8 evaluations for central differences (same as parameter-shift), but parameter-shift gives exact gradients while finite differences introduce approximation error proportional to h² (step size squared).</p>
<p>On noisy hardware, fewer evaluations are preferred (each adds noise). COBYLA (gradient-free) uses O(p) evaluations per iteration but converges more slowly.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.5</strong></p>
<p>Problem: For MaxCut on a triangle graph (3 nodes, edges (0,1),(1,2),(0,2)), write the cost Hamiltonian H_C as a sum of Pauli operators. What is the maximum cut value?</p>
<p><strong>Solution:</strong></p>
<p>H_C = Σ_{(i,j)∈E} (I − ZᵢZⱼ)/2</p>
<p>Three edges: (0,1), (1,2), (0,2)</p>
<p>H_C = (I−Z₀Z₁)/2 + (I−Z₁Z₂)/2 + (I−Z₀Z₂)/2</p>
<p>= (3/2)I − (1/2)(Z₀Z₁ + Z₁Z₂ + Z₀Z₂)</p>
<p><strong>In Pauli string notation (3 qubits):</strong></p>
<p>H_C = 1.5×"III" − 0.5×"IZZ" − 0.5×"ZZI" − 0.5×"ZIZ"</p>
<p>Verify eigenvalues: for cut assignment z = (1,−1,1) (node 0 in S, nodes 1,2 in S̄):</p>
<p>C(z) = (1−1·(−1))/2 + (1−(−1)·1)/2 + (1−1·1)/2 = 1 + 1 + 0 = 2 edges cut</p>
<p>Maximum cut for a triangle: 2 (cut any 2 of 3 edges). The max cut partitions one node from the other two. ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.6</strong></p>
<p>Problem: Prove the parameter-shift rule: ∂⟨O⟩/∂θ = [⟨O⟩(θ+π/2) − ⟨O⟩(θ−π/2)]/2 for a circuit containing the gate G(θ) = exp(−iθP/2).</p>
<p><strong>Solution:</strong></p>
<p>Write ⟨O⟩(θ) = ⟨0|U†(θ) O U(θ)|0⟩ where U(θ) = U_R · G(θ) · U_L is the full circuit.</p>
<p>G(θ) = exp(−iθP/2) = cos(θ/2)I − i·sin(θ/2)P</p>
<p><strong>The expectation value can be written as:</strong></p>
<p>⟨O⟩(θ) = ⟨0|U_L†G†(θ)U_R†O U_R G(θ)U_L|0⟩</p>
<p>Let |φ⟩ = U_R G(θ)U_L|0⟩, and expand using G(θ):</p>
<p>⟨O⟩(θ) = a cos²(θ/2) + b sin(θ/2)cos(θ/2) + c sin²(θ/2)</p>
<p>where a, b, c are constants independent of θ.</p>
<p>Using double-angle formulas: ⟨O⟩(θ) = A + B cos(θ) + C sin(θ) for some A,B,C.</p>
<p><strong>Therefore:</strong></p>
<p>d⟨O⟩/dθ = −B sin(θ) + C cos(θ)</p>
<p>[⟨O⟩(θ+π/2) − ⟨O⟩(θ−π/2)]/2</p>
<p>= [A+B cos(θ+π/2)+C sin(θ+π/2) − A−B cos(θ−π/2)−C sin(θ−π/2)] / 2</p>
<p>= [B(cos(θ+π/2)−cos(θ−π/2)) + C(sin(θ+π/2)−sin(θ−π/2))] / 2</p>
<p>= [B·(−2sin(θ)sin(π/2)) + C·(2cos(θ)sin(π/2))] / 2</p>
<p>= [−2B sin(θ) + 2C cos(θ)] / 2 = −B sin(θ) + C cos(θ) = d⟨O⟩/dθ ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.7</strong></p>
<p>Problem: Show that Grover's algorithm cannot solve structured problems (like integer factorisation) exponentially faster than classical — only quadratically. Why?</p>
<p><strong>Solution:</strong></p>
<p>Grover's algorithm is an oracle-based search algorithm: it finds one x with f(x)=1 in O(√N) queries. This applies when the search space has N items and there is no structure to exploit.</p>
<p><strong>Integer factorisation has structure: it is NOT an unstructured search problem.</strong></p>
<p>One can reduce factoring to period finding (Shor's approach), and period finding exploits the algebraic structure of the multiplicative group Z_N using the QFT — not Grover.</p>
<p>If we instead apply Grover to "search" for factors p by trying all p from 2 to √N and checking divisibility:</p>
<p>N items → √(√N) = N^(1/4) Grover queries. Classical: O(√N) by trial division.</p>
<p><strong>So Grover gives N^{1/4} vs classical N^{1/2} — only a quadratic speedup within trial division.</strong></p>
<p>Shor's algorithm achieves exponential speedup by exploiting group structure via QFT, not Grover's amplitude amplification. The key lesson: quantum speedup comes from exploiting problem-specific mathematical structure, not just from quantum parallelism alone.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 6.8</strong></p>
<p>Problem: A VQE run with a 4-parameter ansatz finds E = −1.1200 Ha after 50 iterations but the exact answer is −1.1362 Ha. Is chemical accuracy achieved? What should be tried?</p>
<p><strong>Solution:</strong></p>
<p>Chemical accuracy threshold: 1 kcal/mol = 1.594 × 10⁻³ Ha ≈ 1.6 × 10⁻³ Ha</p>
<p>Current error: |−1.1200 − (−1.1362)| = 0.0162 Ha</p>
<p>0.0162 / 0.0016 ≈ 10× above chemical accuracy threshold. NOT achieved.</p>
<p><strong>Strategies to improve:</strong></p>
<p>Increase ansatz depth: add more variational layers (8 or 12 parameters)</p>
<p>Use a more expressive ansatz: UCCSD instead of HEA — it has correct excitation structure for H₂</p>
<p>Better initial parameters: use Hartree-Fock solution to initialise θ near the optimal point</p>
<p>More iterations: 50 iterations may be insufficient for gradient convergence; try 200+</p>
<p>Better optimiser: switch from COBYLA (gradient-free) to L-BFGS-B (gradient-based with parameter-shift)</p>
<p>Error mitigation: on real hardware, use Zero-Noise Extrapolation to reduce measurement errors</p>
<p>Root cause: 4 parameters is too few for the HEA to express the exact ground state. For H₂ in STO-3G basis (2 qubits), the exact ground state requires correlations that the UCCSD ansatz (2 parameters) captures correctly with 1 CNOT.</p>
</div>

### B. Unsolved Problems

Solve independently. Bracketed answers for self-checking.

1. For N=64, M=1, compute θ, k\_opt, and P(success at k\_opt). Compare with k\_opt−1 and k\_opt+1. [Answer: θ=arcsin(1/8)≈0.1253 rad; k\_opt=round(π/(4×0.1253))=round(6.27)=6; P(k=6)=sin²(13×0.1253)=sin²(1.629)≈0.998; P(k=5)=sin²(11×0.1253)=sin²(1.379)≈0.975; P(k=7)=sin²(15×0.1253)=sin²(1.879)≈0.948 — k=6 is best]

2. Verify by direct matrix multiplication that the diffusion operator D satisfies D|s⟩ = |s⟩ (the uniform superposition is a fixed point of D). [Answer: D = 2|s⟩⟨s|−I; D|s⟩ = 2|s⟩⟨s|s⟩ − |s⟩ = 2|s⟩·1 − |s⟩ = |s⟩ ✓ (eigenvalue +1)]

3. A QAOA circuit for MaxCut on a path graph of 3 nodes (edges (0,1),(1,2)) has p=1 layer. Write out the full unitary U\_C(γ)·U\_M(β) as a product of single and two-qubit gates. How many CNOTs? [Answer: U\_C(γ) = exp(-iγ Z₀Z₁/2)·exp(-iγ Z₁Z₂/2) = CNOT(0,1)Rz(γ,1)CNOT(0,1)·CNOT(1,2)Rz(γ,2)CNOT(1,2) = 4 CNOTs; U\_M(β) = Rx(2β) on each qubit = 0 CNOTs. Total: 4 CNOTs per QAOA layer]

4. Prove that the VQE energy E(θ) = ⟨ψ(θ)|H|ψ(θ)⟩ is always ≥ E₀ (ground state energy) for any θ. [Answer: Expand |ψ(θ)⟩ in energy eigenbasis: |ψ⟩ = Σᵢ αᵢ|Eᵢ⟩. Then E(θ) = Σᵢ|αᵢ|²Eᵢ ≥ Σᵢ|αᵢ|²E₀ = E₀·Σᵢ|αᵢ|² = E₀ (since Σ|αᵢ|²=1). QED ✓]

5. How does the QAOA depth p relate to solution quality? For MaxCut on a complete 4-vertex graph (6 edges), what is the maximum cut? [Answer: QAOA quality improves with p; p→∞ recovers exact optimum (adiabatic theorem). MaxCut on K₄: partition 2+2 cuts 4 edges (all inter-partition); max cut = 4 for K₄]

6. Grover's algorithm on a 5-qubit register (N=32) has k\_opt=4. Verify by computing sin²(9θ) where sin(θ)=1/√32. [Answer: sin(θ)=1/√32; θ=arcsin(0.177)≈0.1776 rad; sin(9θ)=sin(1.598)≈0.9997; P=sin²(1.598)≈0.9994≈99.94%]

7. For a UCCSD ansatz on 4 qubits with 2 occupied and 2 virtual orbitals, how many single-excitation parameters and double-excitation parameters are there? [Answer: Single excitations: n\_occ × n\_virt = 2×2 = 4 (with spin) = up to 4 singles; Double excitations: C(2,2)×C(2,2) = 1×1 = 1 (spatial) × 4 spin combinations = up to 4 doubles; Total UCCSD parameters: ≤ 8 for this system (exact count depends on symmetry reduction)]

8. The parameter-shift rule requires 2p circuit evaluations for the gradient of a p-parameter circuit. For VQE with p=10 parameters running 200 optimiser iterations, how many total circuit evaluations are needed? How does this compare with a 50-qubit system with p=100 parameters? [Answer: p=10, 200 iters: 200×(1+2×10)=200×21=4200 evaluations. p=100, 200 iters: 200×(1+2×100)=200×201=40,200 evaluations — roughly 10× more, scaling linearly with p]

9. In QAOA with p=1 for MaxCut, the optimal parameters satisfy ∂⟨H\_C⟩/∂γ = 0 and ∂⟨H\_C⟩/∂β = 0. For a simple 2-node graph (1 edge, 2 qubits): show that the QAOA circuit at p=1 achieves the maximum cut of 1 edge with probability sin²(2β)sin²(γ). What is the maximum probability? [Answer: Tracing through the 2-qubit QAOA circuit: P(cut) = sin²(2β)sin²(γ). Maximised when β=π/4 and γ=π/2: P\_max = sin²(π/2)·sin²(π/2) = 1. QAOA achieves P=100% for a 2-node graph at p=1 — this is an exact result since the graph is simple enough.]

10. What is the "barren plateau" phenomenon in VQE? For an n-qubit random circuit ansatz, how does the variance of gradients scale with n? Why is this a serious problem for large NISQ devices? [Answer: Variance Var[∂E/∂θ] = O(2^{-n}) for deep random circuits — exponentially small in n. For n=50 qubits: gradient magnitude ≈ 2^{-25} ≈ 3×10⁻⁸ — undetectable above shot noise. This means random initialisation stalls immediately for large circuits, requiring structured initialisation or shallow/problem-specific ansatz designs.]

### C. Multiple Choice Questions

Answers at end of section.

**Q1. Grover's algorithm achieves what query complexity for searching N unstructured items?**

(a)  O(log N)

(b)  O(√N)

(c)  O(N)

(d)  O(N²)

**Q2. The Grover diffusion operator D = 2|s⟩⟨s| − I performs which geometric operation?**

(a)  Rotation by π/4 about the z-axis

(b)  Reflection about the uniform superposition state |s⟩

(c)  Reflection about the marked state |m⟩

(d)  Projection onto the computational basis

**Q3. The optimal number of Grover iterations for M=1 marked item in N=256 items is:**

(a)  4

(b)  8

(c)  12

(d)  16

**Q4. What is the 'over-rotation' effect in Grover's algorithm?**

(a)  Applying too many H gates causes decoherence

(b)  Applying more than k\_opt iterations causes the success probability to decrease

(c)  The oracle is applied more than once per iteration

(d)  The diffusion operator introduces extra phase rotations

**Q5. The variational principle states that for any state |ψ⟩ and Hamiltonian H:**

(a)  ⟨ψ|H|ψ⟩ = E₀ always

(b)  ⟨ψ|H|ψ⟩ ≤ E₀ (upper bound on ground state energy)

(c)  ⟨ψ|H|ψ⟩ ≥ E₀ (lower bound on ground state energy)

(d)  ⟨ψ|H|ψ⟩ is imaginary

**Q6. In VQE, the Qiskit Runtime Estimator primitive is used to:**

(a)  Transpile circuits to hardware

(b)  Evaluate expectation values ⟨ψ(θ)|H|ψ(θ)⟩ on quantum hardware

(c)  Optimise classical parameters θ

(d)  Measure bit-string distributions

**Q7. The parameter-shift rule requires how many circuit evaluations to compute the gradient of a p-parameter ansatz?**

(a)  p evaluations

(b)  2p evaluations

(c)  p² evaluations

(d)  1 evaluation (automatic differentiation)

**Q8. QAOA (Quantum Approximate Optimisation Algorithm) is designed for:**

(a)  Finding ground state energies of molecules

(b)  Combinatorial optimisation problems like MaxCut

(c)  Estimating eigenphases of unitary operators

(d)  Searching unstructured databases

**Q9. In QAOA for MaxCut, the cost Hamiltonian H\_C encodes the problem as:**

(a)  H\_C = Σ\_{edges} X\_i X\_j

(b)  H\_C = Σ\_{edges} (I − Z\_i Z\_j)/2

(c)  H\_C = Σ\_{vertices} Z\_i

(d)  H\_C = Σ\_{edges} (I + Z\_i Z\_j)/2

**Q10. A 'barren plateau' in VQE is characterised by:**

(a)  Very large gradients that cause the optimiser to overshoot

(b)  Exponentially small gradients that make classical optimisation stall

(c)  Flat energy landscape where all states have equal energy

(d)  Circuit depth exceeding the hardware coherence time

**Q11. Grover's algorithm is provably optimal for unstructured search. This was shown by:**

(a)  Peter Shor (1994)

(b)  Lov Grover (1996)

(c)  Bennett, Bernstein, Brassard, Vazirani (1997)

(d)  Deutsch and Jozsa (1992)

**Q12. The UCCSD ansatz is preferred over HEA for molecular simulation because:**

(a)  It requires fewer qubits

(b)  It is physically motivated and captures relevant electron correlation structure

(c)  It has shallower circuit depth on all hardware

(d)  It does not require entangling gates

**Q13. If Grover's algorithm applies k\_opt + 5 iterations (5 more than optimal), the success probability:**

(a)  Increases further toward 100%

(b)  Stays the same (extra iterations have no effect)

(c)  May decrease significantly due to over-rotation

(d)  Becomes exactly 50%

**Q14. For QAOA depth p, the number of two-qubit (ZZ-rotation) gates scales as:**

(a)  O(p · |V|)  (proportional to vertices)

(b)  O(p · |E|)  (proportional to edges)

(c)  O(p²)

(d)  O(2^p)

**Q15. Why does Grover's algorithm fail to give exponential speedup for structured problems like factoring?**

(a)  Grover works only on boolean functions, not integers

(b)  Grover gives only quadratic speedup; exponential speedup requires exploiting algebraic structure (as Shor's algorithm does via QFT)

(c)  Factoring has too many marked items for Grover to work

(d)  Grover requires M=1 marked item; factoring has many factors

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 6</strong></p>
<p>Q1: (b) O(√N) — provably optimal for unstructured search (BBBV 1997 lower bound)</p>
<p>Q2: (b) Reflection about the uniform superposition state |s⟩ — D = 2|s⟩⟨s|−I is the reflection operator about |s⟩. Combined with the oracle reflection about |w⟩, two reflections = rotation by 2θ.</p>
<p>Q3: (c) 12 — sin(θ)=1/√256=1/16, θ=arcsin(1/16)≈0.0627 rad, k_opt=round(π/(4×0.0627))=round(12.53)=13. Closest listed answer: 12. Note: exact k_opt=13 but P(12)=sin²(25θ)=sin²(1.566)≈0.9997 ≈ same as k=13. Both 12 and 13 are near-optimal; (c) 12 is the best listed answer.</p>
<p>Q4: (b) More than k_opt iterations causes success probability to decrease — the state overshoots |m⟩ and rotates back toward |w⟩. P(k) = sin²((2k+1)θ) oscillates.</p>
<p>Q5: (c) ⟨ψ|H|ψ⟩ ≥ E₀ — This is the variational principle. Equality holds only for the exact ground state. The VQE upper bound tightens as the ansatz improves.</p>
<p>Q6: (b) Evaluate expectation values ⟨ψ(θ)|H|ψ(θ)⟩ — the Estimator handles Pauli decomposition, grouping, error mitigation. The Sampler gives bit-string distributions (used for QAOA sampling, not VQE energy estimation).</p>
<p>Q7: (b) 2p evaluations — parameter-shift rule: 2 circuit evaluations per parameter (at θⱼ±π/2). Total gradient: 2p evaluations. Note: automatic differentiation is not applicable to real quantum hardware.</p>
<p>Q8: (b) Combinatorial optimisation like MaxCut — QAOA encodes discrete optimisation problems as Pauli Hamiltonians and uses alternating cost+mixer unitaries.</p>
<p>Q9: (b) H_C = Σ_{edges} (I − Z_i Z_j)/2 — when z_i≠z_j (edge cut), Z_iZ_j=−1, so (1−(−1))/2=1 (contribution 1 per cut edge). When z_i=z_j (not cut), Z_iZ_j=+1, contribution 0.</p>
<p>Q10: (b) Exponentially small gradients — Var[∂E/∂θ] = O(2^{−n}) for random deep circuits; gradients become undetectable above shot noise for large n, causing classical optimisation to fail.</p>
<p>Q11: (c) Bennett, Bernstein, Brassard, Vazirani (1997) — proved the Ω(√N) lower bound for quantum unstructured search, confirming that Grover is optimal.</p>
<p>Q12: (b) Physically motivated, captures electron correlation structure — UCCSD parameters correspond to physical electron excitation amplitudes, giving better expressibility for molecular problems with far fewer parameters needed per unit of accuracy.</p>
<p>Q13: (c) May decrease significantly — P(k) = sin²((2k+1)θ) oscillates. At k_opt+5, the argument shifts by 10θ, potentially moving P far from its maximum. For small θ (large N), 5 extra steps can reduce P by ~30-50%.</p>
<p>Q14: (b) O(p·|E|) — each QAOA layer contains one ZZ-rotation exp(−iγ Z_i Z_j/2) per edge, requiring 2 CNOTs. Total: p×|E|×2 = O(p·|E|) CNOT gates.</p>
<p>Q15: (b) Grover gives only quadratic speedup; Shor exploits algebraic structure via QFT — factoring N requires finding period r of f(x)=a^x mod N. Grover applied to trial division gives O(N^{1/4}) vs classical O(N^{1/2}), still polynomial; Shor achieves O(poly(log N)) by exploiting the cyclic group structure.</p>
</div>

### D. Theory Questions

- State and prove the optimality of Grover's algorithm: no quantum algorithm can solve unstructured search in fewer than Ω(√N) queries. Describe the BBBV hybrid argument. Why does this lower bound not apply to structured problems like integer factorisation?

- Derive the formula P(success after k iterations) = sin²((2k+1)θ) where sin(θ) = √(M/N). Start from the 2D geometric picture and show how each Grover iteration rotates the state by 2θ. Include the derivation of θ from the initial amplitude of |m⟩.

- Describe the Grover diffusion operator D = 2|s⟩⟨s| − I completely: (a) prove it is unitary, (b) derive its circuit implementation using H^⊗n, X^⊗n, and multi-controlled-Z, (c) compute the gate count, (d) explain why it amplifies the marked state after the oracle has applied the phase flip.

- Explain the variational quantum eigensolver (VQE) in complete detail: (a) state the variational principle, (b) describe the ansatz circuit and its parameters, (c) explain how ⟨H⟩ is computed using the Estimator primitive, (d) describe the classical optimisation loop, (e) state the convergence criterion and chemical accuracy threshold.

- Compare Hardware-Efficient Ansatz (HEA) and UCCSD ansatz for molecular VQE. For each: (a) circuit structure, (b) number of parameters, (c) expressibility for chemistry problems, (d) circuit depth, (e) susceptibility to barren plateaus. Which is preferred for what applications and why?

- Derive the QAOA circuit for MaxCut. Starting from the MaxCut problem formulation, show how: (a) the cost Hamiltonian H\_C is constructed from edge weights, (b) the cost unitary U\_C(γ) = exp(−iγ H\_C) is implemented as ZZ-rotation gates, (c) the mixer unitary U\_M(β) = exp(−iβ H\_B) is implemented as single-qubit Rx gates, (d) the 2p parameters are optimised classically.

- Prove the parameter-shift rule for gates of the form G(θ) = exp(−iθP/2) where P is a Pauli operator. Be mathematically rigorous. Does the rule apply to other parametrised gates (e.g., exp(−iθ(X+Y)/2))? What condition must G satisfy?

- Explain the barren plateau phenomenon in VQE. (a) State the mathematical condition (exponentially vanishing variance of gradients). (b) For what ansatz types do barren plateaus appear? (c) What are three mitigation strategies? (d) Why is this a more serious problem for NISQ-era algorithms than for fault-tolerant algorithms?

- Discuss the practical differences between using a gradient-based optimiser (L-BFGS-B with parameter-shift) vs a gradient-free optimiser (COBYLA) for VQE on noisy hardware. When is each preferred? How does hardware noise affect gradient quality?

- Describe the connection between QAOA and the quantum adiabatic algorithm. As p → ∞, QAOA converges to what? What is the approximation ratio of QAOA at p=1 for MaxCut on 3-regular graphs? Why is this result significant despite being worse than Goemans-Williamson?

### E. Programming Assignments

PA6.1. [Grover's Algorithm Study]  Implement Grover's algorithm in Qiskit for n=3,4,5 qubits and all k=1,2,...,2k\_opt iterations. (a) For each (n,k) pair, simulate 2048 shots and compute the empirical success probability. (b) Plot the theoretical P(k) = sin²((2k+1)θ) curve overlaid with empirical results — verify sinusoidal oscillation. (c) Identify k\_opt and verify over-rotation effect. (d) Test with M=2 marked items (for n=4): show that k\_opt changes and compute the new optimal probability. Submit: code, P vs k plots for each n, and comparative table of theoretical vs measured probabilities.

PA6.2. [VQE for H₂]  Implement VQE from scratch for the H₂ molecule using the Hamiltonian given in Code 6.3. (a) Use a hardware-efficient ansatz with L=1,2,3 layers; for each: plot E vs iteration count, record minimum energy, compute error from exact answer. (b) Implement parameter-shift gradients and compare convergence with COBYLA (gradient-free). (c) Plot energy landscape E(θ) as a 2D heat map varying two selected parameters with others fixed at optimal values. (d) Run on FakeSherbrooke (noisy backend) and compare with noiseless simulation. Analyse the effect of noise on the converged energy. Submit code, convergence curves, landscape heat maps, and noise analysis.

PA6.3. [QAOA for MaxCut]  Implement QAOA for MaxCut at depths p=1,2,3 for (a) a cycle graph C₄ (4 nodes), (b) a 3-regular Petersen graph (10 nodes). For each: (1) build the cost Hamiltonian, (2) implement and optimise the QAOA circuit, (3) decode the top measurement outcomes and compute cut values, (4) compare with classical brute-force optimal cut, (5) compute approximation ratio for each p. Plot approximation ratio vs depth p. Submit complete code, optimisation curves, cut analysis, and 1500-word report discussing the NISQ feasibility of QAOA for these graph instances.

### F. Project Suggestions

Project 6.A — Grover Oracle Synthesis and Quantum Search on Real Hardware:  Implement Grover's algorithm for a 4-qubit database and run on a real IBM Quantum device (IBM Brisbane or similar). (a) Synthesise phase oracles for multiple target states using Qiskit's multi-controlled-X gate. (b) Transpile and run at optimisation levels 1, 2, 3 — compare circuit depths and success probabilities. (c) Apply measurement error mitigation (Readout Error Mitigation via calibration matrix) and compare mitigated vs raw results. (d) Study the effect of hardware noise on the sinusoidal P(k) curve — does over-rotation still occur on noisy hardware? Submit code, hardware results, and a 2500-word report analysing the impact of noise on Grover's algorithm.

Project 6.B — Adaptive Ansatz VQE (ADAPT-VQE):  ADAPT-VQE (Grimsley et al., Nature Communications 2019) builds the ansatz one operator at a time by greedily selecting the operator that reduces the energy most. (a) Implement a simplified 2-qubit ADAPT-VQE for H₂ using a pool of {X₀Y₁, Y₀X₁, X₀X₁Y₀Y₁, ...} operators as candidates. (b) At each step: compute the gradient of E with respect to adding each operator from the pool, select the largest-gradient operator, add it to the circuit, optimise all parameters. (c) Compare convergence rate (energy vs circuit depth) between ADAPT-VQE, fixed-depth HEA, and UCCSD. (d) Demonstrate that ADAPT-VQE reaches chemical accuracy with fewer gates than standard UCCSD. Write a 3000-word report including mathematical derivation of the ADAPT selection criterion.

Project 6.C — Quantum Optimisation for a Real Problem:  Apply QAOA to a real-world combinatorial optimisation problem of your choice. Suggested problems: (a) portfolio optimisation (maximise return subject to risk constraint, encoded as MaxCut on a correlation graph), (b) job scheduling (minimise makespan on 3 machines with 4 jobs, encoded as graph colouring), or (c) molecular conformation (minimum energy conformer search). For the chosen problem: (1) derive the Pauli Hamiltonian encoding the objective and constraints, (2) implement QAOA at p=1,2,3 in Qiskit, (3) benchmark against classical simulated annealing, (4) analyse the approximation ratio and scaling with problem size. Write a 3000-word technical report including the problem formulation, Hamiltonian derivation, circuit description, benchmark results, and discussion of NISQ feasibility.

## References and Further Reading

1. Grover, L. K. (1996). A fast quantum mechanical algorithm for database search. STOC 1996, pp. 212–219. [Original Grover algorithm]

2. Bennett, C. H., Bernstein, E., Brassard, G., Vazirani, U. (1997). Strengths and Weaknesses of Quantum Computing. SIAM Journal on Computing 26(5), 1510–1523. [Grover lower bound; Ω(√N) optimality proof]

3. Peruzzo, A. et al. (2014). A variational eigenvalue solver on a photonic quantum processor. Nature Communications 5, 4213. [Original VQE paper]

4. Farhi, E., Goldstone, J., Gutmann, S. (2014). A Quantum Approximate Optimization Algorithm. arXiv:1411.4028. [Original QAOA paper; MaxCut formulation]

5. Mitarai, K. et al. (2018). Quantum circuit learning. Physical Review A 98, 032309. [Parameter-shift rule (simultaneous with Schuld et al.)]

6. Schuld, M. et al. (2019). Evaluating analytic gradients on quantum hardware. Physical Review A 99, 032331. [Parameter-shift rule; hardware-compatible gradients]

7. Cerezo, M. et al. (2021). Variational quantum algorithms. Nature Reviews Physics 3, 625–644. [Comprehensive VQA review; barren plateaus; trainability]

8. Grimsley, H. R. et al. (2019). An adaptive variational algorithm for exact molecular simulations on a quantum computer. Nature Communications 10, 3007. [ADAPT-VQE]

9. Goemans, M. X. & Williamson, D. P. (1995). Improved approximation algorithms for maximum cut. Journal of the ACM 42(6), 1115–1145. [Classical 0.878 approximation for MaxCut]

10. Nielsen, M. A. & Chuang, I. L. (2000). Quantum Computation and Quantum Information. Cambridge. [Chapter 6 on quantum search algorithms]
