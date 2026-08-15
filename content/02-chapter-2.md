# CHAPTER 2: The Language of Quantum Information

# Mathematics and the Qubit

<div class="box box-anecdote">
<p class="box-title"><strong>📜  Dirac's Bracket Notation — Cambridge, 1939</strong></p>
<p>Paul Adrien Maurice Dirac (1902–1984) — theorist who predicted antimatter, formulated the relativistic wave equation, and received the Nobel Prize at 31 — was legendarily laconic. Colleagues at Cambridge jokingly defined the 'dirac' as the minimum unit of conversation (one word per hour). In 1939, he published a paper introducing the bracket (bra-ket) notation universally used by quantum physicists and quantum computing researchers to this day. He chose the word 'bracket' for the inner product ⟨φ|ψ⟩ and split it into a 'bra' ⟨φ| and a 'ket' |ψ⟩. When Dirac was asked how he preferred to spend his time, he reportedly said: 'Just calculating.' This chapter is dedicated to that calculation.</p>
</div>

Every quantum computation is a manipulation of quantum states. Before writing a single gate or running a single circuit, we must speak the language of quantum information fluently. This chapter builds that language — not just mathematically but physically — so that every equation connects to something real happening inside a quantum computer.

### 2.1 Hilbert Spaces and Dirac Notation

#### 2.1.1 What Is a Hilbert Space?

A Hilbert space ℋ is the mathematical arena of quantum mechanics: a complex vector space with an inner product, complete in the sense that every convergent sequence of vectors has a limit inside the space. For a single qubit: ℋ = ℂ² (two-dimensional complex vector space).

The key properties we use:

- Linearity: if |ψ₁⟩ and |ψ₂⟩ are states and α, β ∈ ℂ, then α|ψ₁⟩ + β|ψ₂⟩ is a valid (unnormalised) state.

- Inner product ⟨φ|ψ⟩ ∈ ℂ: satisfies ⟨φ|ψ⟩ = ⟨ψ|φ⟩\*, linearity in second argument, ⟨ψ|ψ⟩ ≥ 0.

- Norm: ||ψ|| = √⟨ψ|ψ⟩. Physical states are normalised: ⟨ψ|ψ⟩ = 1.

#### 2.1.2 Dirac Notation — Complete Reference

| Symbol | Name | Mathematical Meaning |
|---|---|---|
| \|ψ⟩ | Ket | Column vector in ℋ. Qubit: \|ψ⟩ = [α, β]ᵀ where α,β ∈ ℂ |
| ⟨ψ\| | Bra | Dual (row) vector: complex conjugate transpose. ⟨ψ\| = [α*, β*] |
| ⟨φ\|ψ⟩ | Braket (inner product) | Complex number: Σᵢ φᵢ*ψᵢ. Probability amplitude for \|ψ⟩ found in \|φ⟩ |
| \|φ⟩⟨ψ\| | Outer product | Matrix operator on ℋ. \|φ⟩⟨ψ\|\|χ⟩ = ⟨ψ\|χ⟩\|φ⟩ |
| ⟨ψ\|A\|φ⟩ | Matrix element | When ψ=φ: expectation value ⟨A⟩ = ⟨ψ\|A\|ψ⟩ |
| Σᵢ \|i⟩⟨i\| = I | Completeness | For any orthonormal basis {\|i⟩}: decomposition of identity |

<div class="box box-generic">
<p class="box-title"><strong>|0⟩ = [1, 0]ᵀ    |1⟩ = [0, 1]ᵀ    Orthonormality: ⟨i|j⟩ = δᵢⱼ</strong></p>
<p>Completeness: |0⟩⟨0| + |1⟩⟨1| = I₂ (2×2 identity matrix)</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 2.1  Inner Product Calculations</strong></p>
<p>Compute ⟨+|−⟩ and ⟨i|−i⟩ where |+⟩=(|0⟩+|1⟩)/√2, |−⟩=(|0⟩−|1⟩)/√2, |i⟩=(|0⟩+i|1⟩)/√2, |−i⟩=(|0⟩−i|1⟩)/√2.  ⟨+|−⟩ = (1/√2)⟨0|+⟨1|) · (1/√2)(|0⟩−|1⟩) = (1/2)(⟨0|0⟩−⟨0|1⟩+⟨1|0⟩−⟨1|1⟩) = (1/2)(1−0+0−1) = 0. ✓ Orthogonal X-basis states.  ⟨i|−i⟩ = (1/√2)(⟨0|−i⟨1|) · (1/√2)(|0⟩−i|1⟩) = (1/2)(⟨0|0⟩−i⟨0|1⟩−i⟨1|0⟩+i²⟨1|1⟩) = (1/2)(1−0−0+(−1)) = 0. ✓ Orthogonal Y-basis states.</p>
</div>

### 2.2 The Qubit: State Space and Physical Realisations

#### 2.2.1 The General Qubit State

A classical bit has a definite value: 0 or 1. A qubit is a quantum two-level system whose state is any normalised vector in ℂ²:

<div class="box box-generic">
<p class="box-title"><strong>|ψ⟩ = α|0⟩ + β|1⟩  where  α, β ∈ ℂ  and  |α|² + |β|² = 1</strong></p>
<p>Born rule: P(0) = |α|²,  P(1) = |β|².  Amplitudes, not probabilities — phase matters!</p>
</div>

<figure class="book-figure">
<img src="content/images/image7.png" alt="Figure 2.1: Classical Bit vs Quantum Qubit: Fundamental Differences">
<figcaption>Figure 2.1: Classical Bit vs Quantum Qubit: Fundamental Differences</figcaption>
</figure>

<div class="box box-example">
<p class="box-title"><strong>Example 2.2  Computing Measurement Probabilities</strong></p>
<p>State: |ψ⟩ = (√3/2)|0⟩ + (1/2)|1⟩. Find P(0), P(1). Verify normalisation. P(0) = |√3/2|² = 3/4 = 0.750. P(1) = |1/2|² = 1/4 = 0.250. Normalisation: 3/4 + 1/4 = 1 ✓ Expectation value of Z: ⟨Z⟩ = P(0) − P(1) = 3/4 − 1/4 = 1/2. This state is closer to |0⟩ than |1⟩, consistent with 75% probability of measuring 0.</p>
</div>

<div class="box box-anecdote">
<p class="box-title"><strong>📜  IBM's Transmon Qubit — The Physics Behind the Quantum Computer</strong></p>
<p>IBM's quantum processors use transmon qubits — electrical circuits containing Josephson junctions cooled to 15 millikelvin (colder than outer space). The Josephson junction is two superconductors separated by a thin insulating barrier; Cooper pairs (pairs of electrons) quantum-tunnel through it. The non-linear inductance creates an anharmonic energy spectrum: the gap between levels 0 and 1 (around 5 GHz) differs from the gap between levels 1 and 2, enabling selective driving of only the |0⟩↔|1⟩ transition. Control pulses are microwave signals at the qubit frequency, shaped using DRAG (Derivative Removal via Adiabatic Gate) pulses to avoid exciting higher levels. The qubit is read out by coupling it to a microwave resonator — the resonator frequency shifts depending on the qubit state, and a weak measurement pulse reveals the state without directly destroying it.</p>
</div>

### 2.3 The Bloch Sphere: Geometry of a Qubit

#### 2.3.1 Parametrisation

Global phase is unobservable — multiplying |ψ⟩ by e^(iθ) changes no measurement outcome. Choosing α real and positive, we parametrise every distinct pure qubit state as:

<div class="box box-generic">
<p class="box-title"><strong>|ψ⟩ = cos(θ/2)|0⟩ + e^{iφ}sin(θ/2)|1⟩</strong></p>
<p>Bloch vector: x=sinθcosφ, y=sinθsinφ, z=cosθ.  |r|=1 (pure state), |r|&lt;1 (mixed state).</p>
</div>

<figure class="book-figure">
<img src="content/images/image8.png" alt="Figure 2.2: The Bloch Sphere: Complete Geometric Representation of a Single Qubit">
<figcaption>Figure 2.2: The Bloch Sphere: Complete Geometric Representation of a Single Qubit</figcaption>
</figure>

| State | Amplitude Form | θ | φ | Bloch Sphere Location |
|---|---|---|---|---|
| \|0⟩ | \|0⟩ | 0 | — | North pole (+z) |
| \|1⟩ | \|1⟩ | π | — | South pole (−z) |
| \|+⟩ | (\|0⟩+\|1⟩)/√2 | π/2 | 0 | Equator, +x direction |
| \|−⟩ | (\|0⟩−\|1⟩)/√2 | π/2 | π | Equator, −x direction |
| \|i⟩ | (\|0⟩+i\|1⟩)/√2 | π/2 | π/2 | Equator, +y direction |
| \|−i⟩ | (\|0⟩−i\|1⟩)/√2 | π/2 | 3π/2 | Equator, −y direction |

<div class="box box-example">
<p class="box-title"><strong>Example 2.3  Finding Bloch Sphere Angles</strong></p>
<p>Find θ and φ for |ψ⟩ = (1/√3)|0⟩ + √(2/3)·e^(iπ/3)|1⟩. α = 1/√3, β = √(2/3)·e^(iπ/3). Since α is real positive: cos(θ/2) = 1/√3, sin(θ/2) = √(2/3). θ/2 = arccos(1/√3) ≈ 54.74°, so θ ≈ 109.47°. φ = arg(β) − arg(α) = π/3 − 0 = π/3 = 60°. Bloch vector: x = sin(109.47°)cos(60°) = 0.943×0.5 = 0.472, y = sin(109.47°)sin(60°) = 0.943×0.866 = 0.816, z = cos(109.47°) = −0.333. |r| = √(0.472²+0.816²+0.333²) = √(0.223+0.666+0.111) = √1 = 1 ✓</p>
</div>

### 2.4 Density Matrices and Mixed States

<div class="box box-generic">
<p class="box-title"><strong>Density matrix: ρ = |ψ⟩⟨ψ| (pure state)  or  ρ = Σᵢ pᵢ|ψᵢ⟩⟨ψᵢ| (mixed state)</strong></p>
<p>Properties: ρ=ρ† (Hermitian),  ρ≥0 (positive semidefinite),  Tr(ρ)=1.  Purity: Tr(ρ²)∈[1/d,1].</p>
</div>

<figure class="book-figure">
<img src="content/images/image9.png" alt="Figure 2.3: Density Matrices: Pure States, Mixed States, and Entanglement">
<figcaption>Figure 2.3: Density Matrices: Pure States, Mixed States, and Entanglement</figcaption>
</figure>

<div class="box box-example">
<p class="box-title"><strong>Example 2.4  Computing Purity and Von Neumann Entropy</strong></p>
<p>Mixed state: ρ = 0.6|0⟩⟨0| + 0.4|1⟩⟨1| = diag(0.6, 0.4). Purity: Tr(ρ²) = 0.6² + 0.4² = 0.36 + 0.16 = 0.52. Less than 1 → mixed state. Von Neumann entropy: eigenvalues λ₁=0.6, λ₂=0.4. S(ρ) = −(0.6 log₂0.6 + 0.4 log₂0.4) = −(0.6×(−0.737) + 0.4×(−1.322)) = 0.442 + 0.529 = 0.971 bits. Maximum entropy for qubit = 1 bit. This state has 97.1% of maximum entropy — close to maximally mixed.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 2.5  Purity of Reduced State after Entanglement</strong></p>
<p>Compute the purity of qubit 0's reduced state for |Ψ-⟩ = (|01⟩−|10⟩)/√2. Full density matrix: ρ = |Ψ-⟩⟨Ψ-| = (1/2)(|01⟩−|10⟩)(⟨01|−⟨10|). Reduced density matrix: ρ₀ = Tr₁(ρ). Trace out qubit 1: ρ₀ = ⟨0|₁ρ|0⟩₁ + ⟨1|₁ρ|1⟩₁ = (1/2)|1⟩⟨1| + (1/2)|0⟩⟨0| = I/2. Purity: Tr(ρ₀²) = Tr((I/2)²) = Tr(I/4) = 1/4+1/4 = 1/2. Since purity &lt; 1, qubit 0 is in a mixed state — a direct signature of entanglement with qubit 1. Note: ALL four Bell states give ρ₀ = I/2 (maximally mixed single qubit).</p>
</div>

### 2.5 Linearity — The First Fundamental Principle

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  PRINCIPLE 1 — LINEARITY</strong></p>
<p>All quantum operations are linear maps. The Schrödinger equation iℏ d|ψ⟩/dt = H|ψ⟩ is linear. This means: U(α|ψ₁⟩+β|ψ₂⟩) = αU|ψ₁⟩+βU|ψ₂⟩ for any gate U. Consequence: applying U to a superposition of 2ⁿ basis states simultaneously applies U to all 2ⁿ — quantum parallelism. But measurement gives only ONE outcome: the algorithm must use interference to amplify the correct answer.</p>
</div>

<div class="box box-generic">
<p class="box-title"><strong>Quantum Parallelism: U(1/√2ⁿ Σₓ|x⟩) = 1/√2ⁿ Σₓ(U|x⟩)</strong></p>
<p>All 2ⁿ values of Uf computed in one step — but only ONE readable at measurement time.</p>
</div>

<figure class="book-figure">
<img src="content/images/image10.png" alt="Figure 2.4: The Four Fundamental Principles of Quantum Computing">
<figcaption>Figure 2.4: The Four Fundamental Principles of Quantum Computing</figcaption>
</figure>

<div class="box box-example">
<p class="box-title"><strong>Example 2.6  Quantum Parallelism Demonstrated</strong></p>
<p>Suppose f:{0,1}² → {0,1} is computed by the gate Uf|x⟩|y⟩ = |x⟩|y⊕f(x)⟩. Step 1: Prepare |ψ₀⟩ = H⊗H|00⟩ = (1/2)(|00⟩+|01⟩+|10⟩+|11⟩) — equal superposition of all 4 inputs. Step 2: Apply Uf: Uf|ψ₀⟩⊗|0⟩ = (1/2)(|00,f(00)⟩+|01,f(01)⟩+|10,f(10)⟩+|11,f(11)⟩). Result: All four values f(00), f(01), f(10), f(11) are computed in ONE oracle call. Classical: 4 separate evaluations required. BUT: Measuring the output register gives only ONE (x, f(x)) pair randomly. The information is in the amplitude structure, not directly readable — hence algorithms must use interference to extract useful global properties of f.</p>
</div>

### 2.6 Reversibility — The Second Fundamental Principle

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  PRINCIPLE 2 — REVERSIBILITY</strong></p>
<p>All quantum gates are unitary: U†U = UU† = I. Every gate has exact inverse U†. No information is destroyed until measurement. Classical irreversible gates (NAND, AND, OR) cannot be directly quantum gates — they must be replaced with reversible equivalents (CNOT, Toffoli). Thermodynamically: reversible computation requires no energy dissipation (Landauer's principle). Practically: ancilla qubits used in computations must be 'uncomputed' to avoid entangling them with the main register.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 2.7  Verifying Unitarity and Computing Gate Inverses</strong></p>
<p>The Hadamard gate H = (1/√2)[[1,1],[1,−1]]. Is it unitary? H†H: H is real and symmetric (H†=H), so H†H = H² = (1/2)[[1,1],[1,−1]][[1,1],[1,−1]] = (1/2)[[1+1,1−1],[1−1,1+1]] = (1/2)[[2,0],[0,2]] = I₂ ✓ H is its own inverse: H† = H. Applying H twice returns to original state.  The CNOT gate: CNOT = diag block [[I,0],[0,X]] on the 4D space. CNOT†=CNOT (Hermitian and unitary). CNOT² = I ✓. CNOT is also self-inverse.  Physical meaning: applying H twice undoes the Hadamard — this is used in Deutsch's algorithm where the second H 'undoes' the superposition, concentrating amplitude on the correct answer.</p>
</div>

### 2.7 Superposition — The Third Fundamental Principle

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  PRINCIPLE 3 — SUPERPOSITION</strong></p>
<p>A qubit α|0⟩+β|1⟩ is in a coherent linear combination of both basis states simultaneously. This is NOT a classical probability distribution. The complex phases of α and β are physically real — they govern quantum interference. Measurement collapses the superposition to one outcome (Born rule). The |+⟩ = (|0⟩+|1⟩)/√2 state gives 50% Z-basis outcomes but 100% X-basis outcome +1 — impossible for any classical probability mixture of |0⟩ and |1⟩.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 2.8  Superposition vs Classical Probability — Experimental Distinction</strong></p>
<p>Consider state A: quantum superposition |+⟩ = (|0⟩+|1⟩)/√2. Consider state B: classical mixture ρ_cl = (1/2)|0⟩⟨0| + (1/2)|1⟩⟨1| (50/50 coin flip).  Z-basis measurement: Both give P(0)=P(1)=0.5. IDENTICAL — cannot distinguish.  X-basis measurement (apply H then measure Z): • State A: H|+⟩ = H·H|0⟩ = |0⟩ → P(0)=1, P(1)=0. Always gives 0! • State B: H ρ_cl H† = (1/2)H|0⟩⟨0|H† + (1/2)H|1⟩⟨1|H† = (1/2)|+⟩⟨+| + (1/2)|−⟩⟨−| → P(0)=0.5, P(1)=0.5.  Conclusion: A single X-basis measurement immediately distinguishes quantum superposition from classical mixture. This is why quantum computers can be faster — they exploit coherent superposition, not classical probability.</p>
</div>

### 2.8 Entanglement — The Fourth Fundamental Principle

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  PRINCIPLE 4 — ENTANGLEMENT</strong></p>
<p>Two or more qubits can be in a non-separable state — one that cannot be written as a product of single-qubit states. Entangled states are a uniquely quantum resource with no classical analogue. Measuring one entangled qubit instantaneously determines correlated outcomes for the others, regardless of separation. The Bell states are the canonical maximally entangled two-qubit states. Entanglement enables teleportation, superdense coding, quantum cryptography, and exponential speedups.</p>
</div>

<div class="box box-generic">
<p class="box-title"><strong>|Φ+⟩=(|00⟩+|11⟩)/√2                          |Φ-⟩=(|00⟩-|11⟩)/√2</strong></p>
<p><strong>|Ψ+⟩=(|01⟩+|10⟩)/√2                          |Ψ-⟩=(|01⟩-|10⟩)/√2</strong></p>
<p>Bell states: orthonormal basis for ℂ⁴. Created by H(q0)+CNOT(q0,q1) from computational basis.</p>
</div>

<figure class="book-figure">
<img src="content/images/image11.png" alt="Figure 2.5: Quantum Entanglement: Bell States, Measurement, and Entropy">
<figcaption>Figure 2.5: Quantum Entanglement: Bell States, Measurement, and Entropy</figcaption>
</figure>

<div class="box box-anecdote">
<p class="box-title"><strong>📜  Einstein's 'Spooky Action' and the Nobel Prize 2022</strong></p>
<p>In 1935, Einstein, Podolsky, and Rosen published the EPR paper arguing that if quantum mechanics was complete, measuring one particle of an entangled pair would instantly affect the other — 'spooky action at a distance.' Einstein believed this proved quantum mechanics needed hidden variables. In 1964, John Bell proved that no local hidden-variable theory can reproduce all quantum predictions — the Bell inequality provides a testable criterion. Alain Aspect (1982) and others demonstrated experimental violations of Bell inequalities. Loophole-free tests were conducted in 2015. John Clauser, Alain Aspect, and Anton Zeilinger received the Nobel Prize in Physics 2022 for this work. Entanglement is real, non-local, and foundational to quantum computing.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 2.9  Proving Entanglement by the Separability Test</strong></p>
<p>Show that |Φ+⟩ = (|00⟩+|11⟩)/√2 is entangled. Assume |Φ+⟩ = (α|0⟩+β|1⟩)⊗(γ|0⟩+δ|1⟩) = αγ|00⟩+αδ|01⟩+βγ|10⟩+βδ|11⟩. Matching coefficients: αγ=1/√2, αδ=0, βγ=0, βδ=1/√2. From αδ=0: α=0 or δ=0. If α=0: αγ=0 ≠ 1/√2. If δ=0: βδ=0 ≠ 1/√2. Contradiction! No solution exists. Therefore |Φ+⟩ is ENTANGLED — it cannot be written as a product of single-qubit states. □</p>
</div>

### 2.9 Measurement Theory and the No-Cloning Theorem

#### 2.9.1 Projective Measurements

A projective measurement associates with a Hermitian observable A = Σₘ aₘ Πₘ (spectral decomposition). Measuring |ψ⟩:

- Outcome aₘ with probability p(aₘ) = ⟨ψ|Πₘ|ψ⟩ = ||Πₘ|ψ⟩||².

- Post-measurement state: Πₘ|ψ⟩/√p(aₘ) — collapsed to eigenspace of aₘ.

- Expectation value: ⟨A⟩ = ⟨ψ|A|ψ⟩ = Σₘ aₘ p(aₘ).

<figure class="book-figure">
<img src="content/images/image12.png" alt="Figure 2.6: Quantum Measurement and the No-Cloning Theorem">
<figcaption>Figure 2.6: Quantum Measurement and the No-Cloning Theorem</figcaption>
</figure>

<div class="box box-example">
<p class="box-title"><strong>Example 2.10  Measurement Probabilities in Different Bases</strong></p>
<p>State: |ψ⟩ = (1+i)/2 |0⟩ + 1/√2 |1⟩. Find P(0) and P(1) in Z and X bases. Normalisation check: |(1+i)/2|² + |1/√2|² = (1+1)/4 + 1/2 = 1/2 + 1/2 = 1 ✓  Z-basis: P(0) = |(1+i)/2|² = |1+i|²/4 = 2/4 = 1/2. P(1) = 1/√2|² = 1/2.  X-basis (apply H first): H|ψ⟩ = (1/√2)[[1,1],[1,−1]]·[(1+i)/2, 1/√2]ᵀ. H|ψ⟩[0] = (1/√2)((1+i)/2 + 1/√2) = (1/√2)(0.707∠45° + 0.707) ≈ 0.924+0.354i. |H|ψ⟩[0]|² = 0.924² + 0.354² = 0.854 + 0.125 = 0.979 ≈ 0.979 (P(+) in X basis). P(−) ≈ 0.021. Different from Z-basis — confirming the non-trivial phase structure.</p>
</div>

#### 2.9.2 The No-Cloning Theorem

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  No-Cloning Theorem — Proof and Implications</strong></p>
<p>Theorem: No unitary U exists such that U|ψ⟩|0⟩ = |ψ⟩|ψ⟩ for ALL states |ψ⟩. Proof: Suppose U exists. For any |ψ⟩,|φ⟩: U|ψ⟩|0⟩=|ψ⟩|ψ⟩ and U|φ⟩|0⟩=|φ⟩|φ⟩. Take inner product: ⟨φ|ψ⟩·⟨0|0⟩ = ⟨φ|ψ⟩² (using U†U=I). So ⟨φ|ψ⟩ = ⟨φ|ψ⟩² → ⟨φ|ψ⟩(⟨φ|ψ⟩−1)=0 → ⟨φ|ψ⟩=0 or 1. Contradiction for general states (e.g., ⟨+|0⟩=1/√2). □ Implications: (1) QKD security: eavesdropper cannot copy quantum keys. (2) Error correction uses entanglement, not repetition. (3) Teleportation requires destroying source.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 2.11  Applying the No-Cloning Theorem to QKD Security</strong></p>
<p>In BB84 QKD, Alice sends qubits in random states from {|0⟩,|1⟩,|+⟩,|−⟩}. Eve intercepts qubit in state |+⟩ = (|0⟩+|1⟩)/√2 and tries to clone it. Attempt: Eve uses CNOT as copier: CNOT|+⟩|0⟩ = CNOT·(|0⟩+|1⟩)/√2·|0⟩ = (|00⟩+|11⟩)/√2 = |Φ+⟩. This is a Bell state, NOT |+⟩|+⟩ = (|0⟩+|1⟩)/√2 ⊗ (|0⟩+|1⟩)/√2 = (|00⟩+|01⟩+|10⟩+|11⟩)/2. Eve has created entanglement, not a clone. When she measures her ancilla and forwards the qubit to Bob, the qubit sent to Bob is now disturbed (from quantum measurement collapse), introducing errors that Alice and Bob can detect by computing the quantum bit error rate (QBER). This is why BB84 is information-theoretically secure.</p>
</div>

### 2.10 Multi-Qubit Systems and Tensor Products

<div class="box box-generic">
<p class="box-title"><strong>n-qubit Hilbert space: ℋₙ = ℋ₁⊗ℋ₂⊗...⊗ℋₙ, dimension 2ⁿ</strong></p>
<p>General state: |ψ⟩ = Σₓ∈{0,1}ⁿ αₓ|x⟩ with Σ|αₓ|²=1. For n=50: 2⁵⁰≈10¹⁵ amplitudes.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 2.12  Tensor Products and Separability</strong></p>
<p>Compute (|+⟩⊗|0⟩) explicitly and check if (|00⟩+|10⟩+|01⟩+|11⟩)/2 is separable. |+⟩⊗|0⟩ = (|0⟩+|1⟩)/√2 ⊗ |0⟩ = (|0⟩⊗|0⟩ + |1⟩⊗|0⟩)/√2 = (|00⟩+|10⟩)/√2. This state has coefficients: α₀₀=1/√2, α₁₀=1/√2, α₀₁=α₁₁=0. Product form confirmed ✓  Is (|00⟩+|01⟩+|10⟩+|11⟩)/2 = |+⟩⊗|+⟩ separable? Coefficients: αₓᵧ = 1/2 for all x,y ∈ {0,1}. Try (α|0⟩+β|1⟩)⊗(γ|0⟩+δ|1⟩): need αγ=αδ=βγ=βδ=1/2. From αγ=αδ: α(γ−δ)=0. If α≠0: γ=δ. Similarly βγ=βδ → β(γ−δ)=0, consistent. Let γ=δ=1/√2, α=β=1/√2. Check: αγ=1/2 ✓. YES, this state = |+⟩⊗|+⟩ is separable.</p>
</div>

### 2.11 First Qiskit Programs

⊕ Lab Connection: These programs correspond to Lab Experiments 1 (AQLL §1) and 9 (own-code Bell states).

```python
● ● ●   Python / Qiskit
# Chapter 2 Qiskit Programs — States, Measurements, and Bell Pairs
from qiskit import QuantumCircuit
from qiskit.quantum_info import Statevector, DensityMatrix, partial_trace, entropy
from qiskit_aer import AerSimulator
from qiskit.visualization import plot_bloch_multivector
import numpy as np
 
# Program 2.1: Six Cardinal Qubit States
states = {
    '|0>': QuantumCircuit(1),
    '|1>': QuantumCircuit(1),
    '|+>': QuantumCircuit(1),
    '|->': QuantumCircuit(1),
    '|i>': QuantumCircuit(1),
    '|-i>': QuantumCircuit(1)
}
states['|1>'].x(0)                           # X gate: |0> -> |1>
states['|+>'].h(0)                            # H gate: |0> -> |+>
states['|->'].x(0); states['|->'].h(0)        # X then H -> |->
states['|i>'].h(0); states['|i>'].s(0)        # H then S -> |i>
states['|-i>'].h(0); states['|-i>'].sdg(0)    # H then S† -> |-i>
 
print('State  | α (Re+iIm)  | β (Re+iIm)  | P(0)   | P(1)')
print('-'*65)
for name, qc in states.items():
    sv = Statevector(qc)
    a, b = sv[0], sv[1]
    print(f'{name:6} | {a.real:+.4f}{a.imag:+.4f}i | {b.real:+.4f}{b.imag:+.4f}i | {abs(a)**2:.4f} | {abs(b)**2:.4f}')
 
# Program 2.2: Bell State |Phi+> with Entanglement Verification
qc_bell = QuantumCircuit(2, 2)
qc_bell.h(0)         # H on qubit 0: creates superposition
qc_bell.cx(0, 1)     # CNOT: control=0, target=1: creates entanglement
 
sv_bell = Statevector(qc_bell)
print(f'Bell |Phi+> statevector: {sv_bell.data.round(4)}')
 
# Verify entanglement via reduced density matrix
dm_bell = sv_bell.to_density_matrix()
rho_q0 = partial_trace(dm_bell, [1])       # Trace out qubit 1
purity = float(np.real(np.trace(rho_q0.data @ rho_q0.data)))
S_q0 = float(entropy(rho_q0, base=2))
print(f'Qubit 0 purity: {purity:.6f}  (expected 0.5 for entangled qubit)')
print(f'Entanglement entropy S(Q0): {S_q0:.6f} ebits  (expected 1.0)')
 
# Program 2.3: Measurement statistics
qc_meas = qc_bell.copy()
qc_meas.measure([0, 1], [0, 1])
sim = AerSimulator()
counts = sim.run(qc_meas, shots=4096).result().get_counts()
print(f'Bell state measurement (4096 shots): {counts}')
print('RULE: Only |00> and |11> ever appear — perfect entanglement correlation')
 
# Program 2.4: Density matrix purity scan
print('\nPurity scan for mixtures p|0><0| + (1-p)|1><1|:')
for p in [0.0, 0.25, 0.5, 0.75, 1.0]:
    rho = np.array([[p, 0], [0, 1-p]])
    purity = np.trace(rho @ rho)
    S = -(p*np.log2(p+1e-15) + (1-p)*np.log2(1-p+1e-15))
    print(f'  p={p:.2f}: Purity={purity:.4f}, S(rho)={S:.4f} bits')
```

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 2: The Language of Quantum Information: Mathematics and the Qubit

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Define a Hilbert space. What are the three key properties of the inner product ⟨φ|ψ⟩ that make it suitable for quantum mechanics? Write the normalisation condition for a physical qubit state.**

**Q2.  Complete the Dirac notation reference table: define the ket |ψ⟩, bra ⟨ψ|, braket ⟨φ|ψ⟩, outer product |φ⟩⟨ψ|, and matrix element ⟨ψ|A|φ⟩. Write the completeness relation for the computational basis {|0⟩, |1⟩}.**

**Q3.  Write the general qubit state equation including the Born rule. A qubit is in state |ψ⟩ = (√3/2)|0⟩ + (1/2)|1⟩. Calculate P(0), P(1), and the expectation value ⟨Z⟩. Verify normalisation.**

**Q4.  Write the Bloch sphere parametrisation of a qubit state. Identify the Bloch sphere coordinates (θ, φ) and Bloch vector components for each of the six cardinal states: |0⟩, |1⟩, |+⟩, |−⟩, |i⟩, |−i⟩.**

**Q5.  Define the density matrix for a pure state and a mixed state. List the three mathematical properties every valid density matrix must satisfy. Define purity Tr(ρ²) and state its range. What does purity = 1 versus purity < 1 indicate?**

**Q6.  State Principle 1 (Linearity) precisely. Write the equation demonstrating quantum parallelism for a gate U acting on an n-qubit equal superposition. Explain why quantum parallelism alone does NOT give exponential speedup.**

**Q7.  State Principle 2 (Reversibility) and its mathematical condition. What does it mean physically that all quantum gates are unitary? Show that the Hadamard gate H is both unitary and self-inverse by computing H².**

**Q8.  Explain how quantum superposition differs from a classical probability mixture. Describe the specific X-basis measurement experiment that distinguishes |+⟩ = (|0⟩+|1⟩)/√2 from the classical mixture ρ = (1/2)|0⟩⟨0| + (1/2)|1⟩⟨1|. What are the respective measurement outcomes?**

**Q9.  Write all four Bell states. State Principle 4 (Entanglement) and explain why entanglement is not the same as classical correlation. Describe the separability test and apply it to |Φ+⟩ = (|00⟩+|11⟩)/√2 to prove it is entangled.**

**Q10.  State and prove the No-Cloning Theorem. Identify the step in the proof that requires linearity/unitarity. Give three specific technological implications of the theorem.**

**Q11.  Define Von Neumann entropy S(ρ). For the mixed state ρ = diag(0.6, 0.4), compute: (a) purity Tr(ρ²), (b) S(ρ) in bits, and (c) determine how close it is to maximum entropy for a qubit.**

**Q12.  Write the n-qubit Hilbert space dimension formula. For n = 50 qubits: (a) state the number of amplitudes, (b) how much classical memory would be needed to store the full state, and (c) how many classical bits can be extracted per measurement?**

**Q13.  Describe how a transmon superconducting qubit works: (a) what physical object acts as the qubit, (b) what temperature is required and why, (c) how are quantum gates applied, and (d) how is the qubit state read out?**

**Q14.  For a projective measurement with Hermitian observable A = Σₘ aₘ Πₘ, write expressions for: (a) the probability of outcome aₘ, (b) the post-measurement state, and (c) the expectation value ⟨A⟩. Using Z = |0⟩⟨0| − |1⟩⟨1|, prove that ⟨Z⟩ = 2P(0) − 1.**

**Q15.  Explain the concept of quantum entanglement entropy. For the Bell state |Ψ−⟩ = (|01⟩−|10⟩)/√2: (a) compute the reduced density matrix ρ₀ of qubit 0, (b) find its purity, and (c) state what the purity tells you about the entanglement of the two qubits.**

**PART B — MODEL ANSWERS**

**Answer 1:**

A Hilbert space ℋ is a complex vector space equipped with an inner product, complete in the metric defined by that inner product. For a single qubit: ℋ = ℂ². Three key inner product properties: (1) Conjugate symmetry: ⟨φ|ψ⟩ = ⟨ψ|φ⟩\*. (2) Linearity in second argument: ⟨φ|(α|ψ₁⟩+β|ψ₂⟩) = α⟨φ|ψ₁⟩+β⟨φ|ψ₂⟩. (3) Positive definiteness: ⟨ψ|ψ⟩ ≥ 0, with equality iff |ψ⟩ = 0. Normalisation condition: |α|²+|β|² = 1 for |ψ⟩ = α|0⟩+β|1⟩.

**Answer 2:**

Ket |ψ⟩: column vector in ℋ representing the quantum state, e.g., [α, β]ᵀ. Bra ⟨ψ|: conjugate transpose row vector [α\*, β\*]. Braket ⟨φ|ψ⟩: inner product — a complex number representing probability amplitude. Outer product |φ⟩⟨ψ|: a matrix (linear operator on ℋ). Matrix element ⟨ψ|A|φ⟩: transition amplitude; when ψ=φ gives expectation value. Completeness: |0⟩⟨0| + |1⟩⟨1| = I₂ (resolution of identity).

**Answer 3:**

General qubit: |ψ⟩ = α|0⟩ + β|1⟩, Born rule: P(0) = |α|², P(1) = |β|². For (√3/2)|0⟩+(1/2)|1⟩: P(0) = |√3/2|² = 3/4. P(1) = |1/2|² = 1/4. Verification: 3/4 + 1/4 = 1 ✓. Expectation ⟨Z⟩ = P(0) − P(1) = 3/4 − 1/4 = 1/2. Positive ⟨Z⟩ indicates the state is biased toward |0⟩, consistent with 75% measurement probability.

**Answer 4:**

|ψ⟩ = cos(θ/2)|0⟩ + e^{iφ}sin(θ/2)|1⟩. Cardinal states: |0⟩: θ=0, r=(0,0,+1), North pole. |1⟩: θ=π, r=(0,0,−1), South pole. |+⟩: θ=π/2,φ=0, r=(+1,0,0), +X equator. |−⟩: θ=π/2,φ=π, r=(−1,0,0), −X equator. |i⟩: θ=π/2,φ=π/2, r=(0,+1,0), +Y equator. |−i⟩: θ=π/2,φ=3π/2, r=(0,−1,0), −Y equator.

**Answer 5:**

Pure state: ρ = |ψ⟩⟨ψ| (rank-1 projector, outer product of the state with itself). Mixed state: ρ = Σᵢ pᵢ|ψᵢ⟩⟨ψᵢ| where pᵢ ≥ 0, Σpᵢ = 1. Three properties: (1) Hermitian: ρ = ρ†. (2) Positive semidefinite: all eigenvalues ≥ 0. (3) Unit trace: Tr(ρ) = 1. Purity Tr(ρ²) ∈ [1/d, 1] for d-dimensional system. Purity = 1: pure state (on Bloch sphere surface, maximum quantum coherence). Purity < 1: mixed state (inside Bloch sphere, partial decoherence).

**Answer 6:**

Principle 1 (Linearity): U(α|ψ₁⟩+β|ψ₂⟩) = αU|ψ₁⟩+βU|ψ₂⟩ for any quantum gate U. Quantum parallelism: U·(1/√2^n Σ\_x|x⟩) = 1/√2^n Σ\_x(U|x⟩) — evaluates U on all 2^n inputs in one step. However, quantum parallelism alone does NOT give exponential speedup because the Holevo bound limits measurement to only n classical bits per measurement. You cannot 'read out' all 2^n outputs. Speedup requires interference to amplify the correct answer's probability.

**Answer 7:**

Principle 2: U†U = UU† = I (unitarity condition). Physically: quantum evolution is time-reversible (every gate has exact inverse U†), information-preserving (no information is destroyed until measurement), and norm-preserving (probabilities sum to one). H = (1/√2)[[1,1],[1,−1]]; H† = H (real symmetric, so H† = Hᵀ = H). H² = (1/2)[[1,1],[1,−1]]·[[1,1],[1,−1]] = (1/2)[[2,0],[0,2]] = I₂ ✓. H is both unitary and Hermitian (self-adjoint).

**Answer 8:**

The quantum superposition |+⟩ has off-diagonal density matrix elements (coherences ρ₀₁ = ρ₁₀ = 1/2) encoding quantum interference. The classical mixture ρ\_cl = I/2 has all off-diagonal elements zero. X-basis measurement: apply H then measure Z. For |+⟩: H|+⟩ = |0⟩ — outcome always 0, P(0) = 1. For classical mixture ρ\_cl: H·(I/2)·H† = I/2 — outcome random, P(0) = P(1) = 1/2. The guaranteed outcome for |+⟩ vs. random for ρ\_cl is the experimental signature of quantum coherence.

**Answer 9:**

Bell states: |Φ+⟩=(|00⟩+|11⟩)/√2, |Φ−⟩=(|00⟩−|11⟩)/√2, |Ψ+⟩=(|01⟩+|10⟩)/√2, |Ψ−⟩=(|01⟩−|10⟩)/√2. Entanglement is not classical correlation because it violates Bell inequalities — impossible for any classical hidden-variable model. Separability test: assume |Φ+⟩=(a|0⟩+b|1⟩)⊗(c|0⟩+d|1⟩). Matching coefficients: ac=1/√2, ad=0, bc=0, bd=1/√2. From ad=0: a=0 or d=0. If a=0: ac=0≠1/√2. If d=0: bd=0≠1/√2. Contradiction — |Φ+⟩ is entangled.

**Answer 10:**

Statement: No unitary U can clone arbitrary unknown states: U|ψ⟩|0⟩ = |ψ⟩|ψ⟩ for all |ψ⟩. Proof: Assume U exists. Apply to superposition |ψ⟩+|φ⟩: linearity gives (U|ψ⟩|0⟩+U|φ⟩|0⟩)/√2 = (|ψ⟩|ψ⟩+|φ⟩|φ⟩)/√2. But cloning (|ψ⟩+|φ⟩)/√2 gives ((|ψ⟩+|φ⟩)⊗(|ψ⟩+|φ⟩))/2, which includes cross terms |ψ⟩|φ⟩ and |φ⟩|ψ⟩. Contradiction. Critical step: the linearity/unitarity of U is used in the superposition argument. Implications: (1) no quantum backup possible, (2) QKD eavesdropping is detectable, (3) quantum teleportation must destroy the original.

**Answer 11:**

Von Neumann entropy: S(ρ) = −Tr(ρ log₂ ρ) = −Σᵢ λᵢ log₂ λᵢ where {λᵢ} are eigenvalues. For ρ = diag(0.6, 0.4): (a) Purity = Tr(ρ²) = 0.6² + 0.4² = 0.36 + 0.16 = 0.52. (b) S(ρ) = −0.6 log₂(0.6) − 0.4 log₂(0.4) = 0.6×0.737 + 0.4×1.322 = 0.442 + 0.529 = 0.971 bits. (c) Maximum entropy for a qubit = 1 bit (at ρ = I/2). This state has entropy 0.971/1.0 = 97.1% of maximum — it is nearly maximally mixed.

**Answer 12:**

n-qubit Hilbert space dimension: 2^n. For n = 50: (a) Amplitudes = 2^50 ≈ 1.126×10^15 (over one quadrillion complex amplitudes). (b) Classical memory: each complex amplitude = 16 bytes (double precision); total = 16 × 2^50 ≈ 1.8×10^16 bytes = 18 petabytes — exceeds combined capacity of all Earth's data centers. (c) Classical bits per measurement = 50 (Holevo bound — despite quadrillion amplitudes, only 50 bits are accessible per shot).

**Answer 13:**

(a) Physical qubit: a Josephson junction circuit (transmon) — an anharmonic LC oscillator whose quantised energy levels |0⟩ and |1⟩ encode the qubit. (b) Temperature: ~15 mK, colder than deep space. Required because thermal energy k\_BT must be much less than qubit energy splitting hf ≈ 20 μeV (~5 GHz); at 15 mK, k\_BT ≈ 0.13 μeV — suppressing thermal excitation of |1⟩. (c) Gates: shaped microwave pulses at the qubit resonance frequency (~5 GHz), DRAG envelopes, duration 10–50 ns. (d) Readout: dispersive coupling to a microwave resonator; qubit state shifts resonator frequency by ±χ; probing resonator transmission reveals qubit state without directly measuring it.

**Answer 14:**

(a) P(aₘ) = Tr(Πₘρ) = ⟨ψ|Πₘ|ψ⟩. (b) Post-measurement: ρ' = ΠₘρΠₘ / P(aₘ). (c) ⟨A⟩ = Tr(Aρ) = Σₘ aₘ P(aₘ). Proof ⟨Z⟩ = 2P(0)−1: ⟨Z⟩ = Tr(Zρ) = ⟨ψ|Z|ψ⟩ = ⟨ψ|(|0⟩⟨0|−|1⟩⟨1|)|ψ⟩ = |⟨0|ψ⟩|² − |⟨1|ψ⟩|² = P(0)−P(1) = P(0)−(1−P(0)) = 2P(0)−1 ✓.

**Answer 15:**

Entanglement entropy measures quantum correlations: S(ρ\_A) = −Tr(ρ\_A log₂ ρ\_A) where ρ\_A is the reduced density matrix of subsystem A. For |Ψ−⟩ = (|01⟩−|10⟩)/√2: (a) ρ\_full = |Ψ−⟩⟨Ψ−|; tracing out qubit 1: ρ₀ = Tr₁(ρ\_full) = (1/2)|0⟩⟨0| + (1/2)|1⟩⟨1| = I/2. (b) Purity = Tr(ρ₀²) = Tr(I²/4) = Tr(I/4) = 1/2. (c) Purity = 1/2 (less than 1) means qubit 0 is maximally mixed — it has no definite quantum state on its own. This indicates maximum entanglement between the two qubits: all the quantum information is stored in correlations, none in either qubit alone.

<div class="box box-generic">
<p class="box-title"><strong>END OF CHAPTER 2 — EXERCISES &amp; PROBLEMS</strong></p>

</div>

#### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.1</strong></p>
<p><strong>Problem:</strong> Find β such that the state |ψ⟩ = (3+4i)/5·|0⟩ + β|1⟩ is normalised with β real and positive. Find P(0) and P(1).</p>
<p><strong>Solution:</strong> Normalisation: |(3+4i)/5|² + |β|² = 1. |(3+4i)/5|² = (3²+4²)/25 = 25/25 = 1. So |β|² = 0, hence β = 0. Therefore |ψ⟩ = (3+4i)/5·|0⟩ + 0·|1⟩ — this is just |0⟩ with a phase (3+4i)/5. Note: (3+4i)/5 is a unit complex number since |(3+4i)/5|=5/5=1. So |ψ⟩ = e^(iθ)|0⟩ where θ = arctan(4/3). P(0) = 1, P(1) = 0. Global phase does not affect measurements.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.2</strong></p>
<p><strong>Problem:</strong> Find the Bloch vector (x,y,z) for |ψ⟩ = cos(π/6)|0⟩ + sin(π/6)·e^(i·3π/4)|1⟩. Verify |r|=1.</p>
<p><strong>Solution:</strong> θ = 2×π/6 = π/3, φ = 3π/4. x = sin(π/3)cos(3π/4) = (√3/2)(−1/√2) = −√3/(2√2) = −√6/4 ≈ −0.612. y = sin(π/3)sin(3π/4) = (√3/2)(1/√2) = √3/(2√2) = √6/4 ≈ 0.612. z = cos(π/3) = 1/2 = 0.500. |r|² = 6/16 + 6/16 + 1/4 = 12/16 + 4/16 = 16/16 = 1 ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.3</strong></p>
<p><strong>Problem:</strong> For ρ = 0.8|+⟩⟨+| + 0.2|−⟩⟨−|, compute the density matrix, purity, and Von Neumann entropy.</p>
<p><strong>Solution:</strong> |+⟩⟨+| = (1/2)[[1,1],[1,1]], |−⟩⟨−| = (1/2)[[1,−1],[−1,1]]. ρ = 0.8×(1/2)[[1,1],[1,1]] + 0.2×(1/2)[[1,−1],[−1,1]] = [[0.5,0.3],[0.3,0.5]]. Eigenvalues: det(ρ−λI)=0: (0.5−λ)²−0.09=0 → λ = 0.5±0.3. So λ₁=0.8, λ₂=0.2. Purity: Tr(ρ²) = λ₁² + λ₂² = 0.64+0.04 = 0.68. S(ρ) = −(0.8 log₂0.8 + 0.2 log₂0.2) = −(0.8×(−0.322) + 0.2×(−2.322)) = 0.258+0.464 = 0.722 bits.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.4</strong></p>
<p><strong>Problem:</strong> Prove that ⟨A⟩ = 2P(0)−1 where P(0) is the probability of outcome 0 for the Pauli Z operator A=Z=|0⟩⟨0|−|1⟩⟨1|, measured on state |ψ⟩=α|0⟩+β|1⟩.</p>
<p><strong>Solution:</strong> ⟨Z⟩ = ⟨ψ|Z|ψ⟩ = ⟨ψ|(|0⟩⟨0|−|1⟩⟨1|)|ψ⟩ = ⟨ψ|0⟩⟨0|ψ⟩ − ⟨ψ|1⟩⟨1|ψ⟩ = |⟨0|ψ⟩|² − |⟨1|ψ⟩|² = |α|² − |β|² = P(0) − P(1). Since P(0)+P(1)=1: P(1)=1−P(0). So ⟨Z⟩ = P(0)−(1−P(0)) = 2P(0)−1 ✓ Corollary: P(0) = (1+⟨Z⟩)/2. This is used in quantum state tomography to reconstruct states from measurements.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.5</strong></p>
<p><strong>Problem:</strong> Show that the Hadamard gate swaps the X and Z axes on the Bloch sphere. Specifically prove that HXH† = Z and HZH† = X.</p>
<p><strong>Solution:</strong> H = (1/√2)[[1,1],[1,−1]]. H† = H (Hermitian). HXH = (1/√2)[[1,1],[1,−1]] · [[0,1],[1,0]] · (1/√2)[[1,1],[1,−1]]. First: XH = (1/√2)[[1,1],[1,−1]] right product → actually compute HX: HX = (1/√2)[[0+1,1+0],[0−1,1−0]] = (1/√2)[[1,1],[−1,1]]. HXH = (1/2)[[1,1],[−1,1]][[1,1],[1,−1]] = (1/2)[[2,0],[0,−2]] = [[1,0],[0,−1]] = Z ✓ Similarly HZH = (1/√2)[[1,1],[1,−1]]·[[1,0],[0,−1]]·(1/√2)[[1,1],[1,−1]] = [[0,1],[1,0]] = X ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.6</strong></p>
<p><strong>Problem:</strong> The density matrix of a state is ρ = (1/4)[[3,1+i],[1−i,1]]. Is this a valid density matrix? Is it pure?</p>
<p><strong>Solution:</strong> Check Hermitian: ρ† should equal ρ. ρ†[0][1] = conj(ρ[1][0]) = conj(1−i) = 1+i = ρ[0][1] ✓ Check Tr(ρ)=1: (3+1)/4 = 1 ✓ Check positive semidefinite: eigenvalues of (1/4)[[3,1+i],[1−i,1]]. det((1/4)ρ−(λ/4)I)=0: (3−λ)(1−λ)−|1+i|²=0: λ²−4λ+(3−2)=0: λ²−4λ+1=0: λ=(4±√12)/2=2±√3. λ₁=2+√3≈3.73, λ₂=2−√3≈0.27. But these are eigenvalues of 4ρ, so actual eigenvalues are (2±√3)/4. λ₁=(2+√3)/4≈0.933, λ₂=(2−√3)/4≈0.067. Both positive ✓ Valid density matrix. Purity: λ₁²+λ₂² = 0.933²+0.067² = 0.870+0.004 = 0.874 &lt; 1 → MIXED state.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.7</strong></p>
<p><strong>Problem:</strong> A qubit is measured in the X basis (|+⟩,|−⟩) and gives outcome |+⟩. It is then measured again in the Z basis. What are the probabilities P(0) and P(1)?</p>
<p><strong>Solution:</strong> After X-basis measurement, the state collapses to |+⟩ = (|0⟩+|1⟩)/√2 (regardless of previous state). Now measure in Z basis on |+⟩: P(0) = |⟨0|+⟩|² = |1/√2|² = 1/2. P(1) = |⟨1|+⟩|² = |1/√2|² = 1/2. Result: 50/50 — completely random in Z basis. This illustrates the Heisenberg uncertainty principle for complementary observables X and Z: perfect knowledge of X (eigenstate |+⟩) means complete uncertainty in Z.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.8</strong></p>
<p><strong>Problem:</strong> The state |ψ⟩ = (|000⟩+|001⟩+|010⟩+|011⟩)/2 is a 3-qubit state. Is it entangled? If not, write it as a product state.</p>
<p><strong>Solution:</strong> Try |ψ⟩ = |q₀⟩⊗|q₁⟩⊗|q₂⟩. Amplitudes: α₀₀₀=α₀₀₁=α₀₁₀=α₀₁₁=1/2, all others = 0. Note: all amplitudes with leading bit 0 (|0xx⟩) are 1/2, and all with leading bit 1 (|1xx⟩) are 0. This means qubit 0 is in state |0⟩ (no |1⟩ component). For qubits 1 and 2: remaining state = (|00⟩+|01⟩+|10⟩+|11⟩)/2 = (|0⟩+|1⟩)/√2 ⊗ (|0⟩+|1⟩)/√2 = |+⟩⊗|+⟩. Therefore |ψ⟩ = |0⟩ ⊗ |+⟩ ⊗ |+⟩ — fully separable, NOT entangled.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 2.9</strong></p>
<p><strong>Problem:</strong> A qubit is in the state ρ = 0.7|0⟩⟨0| + 0.3|1⟩⟨1| (a statistical mixture). Compute the purity and Von Neumann entropy.</p>
<p><strong>Solution:</strong> ρ = [[0.7, 0], [0, 0.3]] (diagonal, since |0⟩ and |1⟩ are the basis used). Purity: Tr(ρ²) = 0.7² + 0.3² = 0.49 + 0.09 = 0.58. Less than 1, confirming a mixed state. Eigenvalues: λ₁ = 0.7, λ₂ = 0.3. S(ρ) = −(0.7 log₂0.7 + 0.3 log₂0.3) = −(0.7×(−0.515) + 0.3×(−1.737)) = 0.3605 + 0.5211 = 0.882 bits. Note: maximum entropy for a qubit is 1 bit (maximally mixed ρ = I/2). This state has 88.2% of maximum entropy.</p>
</div>

#### B. Unsolved Problems

Answers provided in brackets for self-checking.

**1.** Find α (real, non-negative) such that |ψ⟩=α|0⟩+(i/2)|1⟩+(1/2)|1⟩...wait: find real α≥0 for |ψ⟩=α|0⟩+(3+4i)/10|1⟩ to be normalised. Find P(0) and P(1).  [Answer: α = √(1−25/100) = √(75/100) = √3/2 ≈ 0.866; P(0) = 3/4, P(1) = 1/4]

**2.** Compute ⟨X⟩, ⟨Y⟩, ⟨Z⟩ for the state |ψ⟩ = cos(π/4)|0⟩ + i·sin(π/4)|1⟩. What are the Bloch sphere angles θ and φ?  [Answer: ⟨X⟩=0, ⟨Y⟩=1, ⟨Z⟩=0; θ=π/2, φ=π/2 (the +Y axis, state |i⟩)]

**3.** A density matrix is ρ=[[0.7,0.3],[0.3,0.3]]. Find the Bloch vector (x,y,z).  [Answer: x=2×Re(ρ₀₁)=0.6, y=−2×Im(ρ₀₁)=0, z=ρ₀₀−ρ₁₁=0.4; |r|=√(0.36+0+0.16)=√0.52≈0.72 (mixed state inside sphere)]

**4.** Show that H|+⟩=|0⟩ and H|−⟩=|1⟩, confirming that H maps the X-basis to the Z-basis and vice versa.  [Answer: H|+⟩=H(H|0⟩)=H²|0⟩=|0⟩; H|−⟩=H(H|1⟩)=H²|1⟩=|1⟩ (using H²=I)]

**5.** Compute the Von Neumann entropy of the state with eigenvalues {0.9, 0.08, 0.02} (3-dimensional system). Is it pure or mixed?  [Answer: S = −(0.9log₂0.9+0.08log₂0.08+0.02log₂0.02) ≈ 0.137+0.358+0.114 = 0.609 bits; Mixed (purity = 0.81+0.0064+0.0004 = 0.817)]

**6.** Three systems A, B, C are in state |ψ\_ABC⟩ = (|0\_A⟩|Φ+\_BC⟩)/1. Is A entangled with BC? Is B entangled with C?  [Answer: A is separable from BC (product state |0⟩\_A ⊗ |Φ+⟩\_BC); B and C are maximally entangled (Bell pair |Φ+⟩)]

**7.** The no-cloning theorem states ⟨φ|ψ⟩ = ⟨φ|ψ⟩². Find all values of ⟨φ|ψ⟩ satisfying this equation. What are the only states that CAN be cloned?  [Answer: ⟨φ|ψ⟩=0 (orthogonal states) or ⟨φ|ψ⟩=1 (identical states); only orthogonal basis states can be cloned]

**8.** Verify that the S gate (S=diag(1,i)) satisfies S²=Z. Starting from |+⟩, apply S and find the resulting Bloch vector.  [Answer: S²=diag(1,i²)=diag(1,−1)=Z ✓; S|+⟩=(|0⟩+i|1⟩)/√2=|i⟩; Bloch vector=(0,1,0) on +Y axis]

**9.** A quantum channel applies the transformation ρ → (1−p)ρ + p(XρX)/2 + p(ZρZ)/2 for some error probability p. Show that for p=1, ρ → I/2 (maximally mixed). What is the purity for general p starting from a pure state?  [Answer: p=1: ρ→0ρ+(XρX+ZρZ)/2; for ρ=|0⟩⟨0|: X|0⟩⟨0|X=|1⟩⟨1|, Z|0⟩⟨0|Z=|0⟩⟨0|, average=I/2 ✓; Purity=(1−p)²+p²/2]

**10.** The Bell state |Ψ-⟩=(|01⟩-|10⟩)/√2 is the 'singlet state.' Show that it is rotationally invariant: for ANY unit vector n̂, the expected value of n̂·σ⊗n̂·σ is −1 (perfect anti-correlation in every basis).  [Answer: ⟨Ψ-|n̂·σ⊗n̂·σ|Ψ-⟩ = −⟨Ψ-|σ·σ|Ψ-⟩ = −Tr(P) where... result = −1 by SU(2) invariance of singlet; anti-correlated in every direction]

#### C. Multiple Choice Questions

Note: Answers to all MCQs are given at the end of this section.

**Q1.** A qubit state |ψ⟩=α|0⟩+β|1⟩ satisfies which condition?

(a)  |α|+|β|=1

(b)  α²+β²=1

(c)  |α|²+|β|²=1

(d)  α\*+β\*=1

**Q2.** The Bloch sphere represents:

(a)  All quantum states including mixed states

(b)  All physically distinct pure single-qubit states

(c)  The probability distribution over |0⟩ and |1⟩

(d)  The physical location of a qubit inside a quantum computer

**Q3.** Von Neumann entropy S(ρ)=0 means:

(a)  The system has zero energy

(b)  The state is pure

(c)  The system is maximally mixed

(d)  The temperature is zero

**Q4.** The inner product ⟨i|−i⟩ where |i⟩=(|0⟩+i|1⟩)/√2 and |−i⟩=(|0⟩−i|1⟩)/√2 equals:

(a)  1

(b)  0

(c)  i

(d)  1/2

**Q5.** A qubit in state |+⟩=(|0⟩+|1⟩)/√2 is measured in the X basis. The outcome is:

(a)  +1 with probability 0.5

(b)  −1 with probability 0.5

(c)  +1 with probability 1.0

(d)  Random — 50/50 between +1 and −1

**Q6.** The no-cloning theorem is a consequence of:

(a)  Heisenberg uncertainty principle only

(b)  Linearity and unitarity of quantum operations

(c)  The measurement postulate only

(d)  The second law of thermodynamics

**Q7.** For the Bell state |Φ+⟩=(|00⟩+|11⟩)/√2, measuring qubit 0 and finding '1' collapses qubit 1 to:

(a)  |0⟩

(b)  |1⟩

(c)  |+⟩

(d)  A random superposition

**Q8.** The purity Tr(ρ²) of the maximally mixed qubit state ρ=I/2 is:

(a)  1

(b)  0.5

(c)  0

(d)  0.25

**Q9.** Which of the following is an entangled state?

(a)  (|0⟩+|1⟩)/√2 ⊗ |0⟩

(b)  (|00⟩+|01⟩)/√2

(c)  (|00⟩+|11⟩)/√2

(d)  (|0⟩+|1⟩)/√2 ⊗ (|0⟩+|1⟩)/√2

**Q10.** The Hadamard gate H satisfies:

(a)  H²=0

(b)  H=H†=H⁻¹ (H is Hermitian, unitary, and self-inverse)

(c)  H is not Hermitian

(d)  H⁴=I only

**Q11.** The Bloch vector (x,y,z) for the state |0⟩ is:

(a)  (1,0,0)

(b)  (0,1,0)

(c)  (0,0,1)

(d)  (0,0,−1)

**Q12.** Quantum superposition differs from classical probability because:

(a)  Superposition allows more than two outcomes

(b)  The complex phase of amplitudes creates interference effects

(c)  Superposition can violate energy conservation

(d)  Classical probability doesn't obey the Born rule

**Q13.** The completeness relation for the computational basis is:

(a)  |0⟩⟨0|·|1⟩⟨1|=I

(b)  ⟨0|0⟩+⟨1|1⟩=I

(c)  |0⟩⟨0|+|1⟩⟨1|=I

(d)  Neither, completeness only holds in infinite dimensions

**Q14.** A qubit is prepared in |+⟩, then a Z gate is applied, then measured in X basis. The outcome is:

(a)  +1 with probability 1

(b)  −1 with probability 1

(c)  50/50 random

(d)  0 with probability 1

**Q15.** The reduced density matrix of qubit 0 in the Bell state |Φ+⟩=(|00⟩+|11⟩)/√2 is:

(a)  [[1,0],[0,0]] = |0⟩⟨0|

(b)  [[0.5,0.5],[0.5,0.5]] = |+⟩⟨+|

(c)  [[0.5,0],[0,0.5]] = I/2

(d)  [[1,1],[1,1]]/2

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 2</strong></p>
<p><strong>Q1: (c) |α|²+|β|²=1  —</strong> Normalisation condition: total probability must equal 1; amplitudes are complex numbers</p>
<p><strong>Q2: (b) All physically distinct pure single-qubit states  —</strong> After removing global phase, 2 real parameters (θ,φ) map to Bloch sphere surface</p>
<p><strong>Q3: (b) The state is pure  —</strong> S(ρ)=0 iff ρ=|ψ⟩⟨ψ| is a rank-1 projector — pure state with zero entropy</p>
<p><strong>Q4: (b) 0  —</strong> ⟨i|−i⟩=(1/2)(1+i)(1−i)·real part... = 0; Y-basis states are orthogonal</p>
<p><strong>Q5: (c) +1 with probability 1.0  —</strong> H|+⟩=|0⟩, so X-basis measurement of |+⟩ always gives +1 (eigenvalue of X)</p>
<p><strong>Q6: (b) Linearity and unitarity  —</strong> Inner product argument: ⟨φ|ψ⟩ = ⟨φ|ψ⟩² requires ⟨φ|ψ⟩=0 or 1</p>
<p><strong>Q7: (b) |1⟩  —</strong> In |Φ+⟩ the amplitudes pair: |00⟩ and |11⟩ — measuring 1 on q0 forces q1=|1⟩</p>
<p><strong>Q8: (b) 0.5  —</strong> Tr((I/2)²)=Tr(I/4)=2/4=0.5=1/d for d=2</p>
<p><strong>Q9: (c) (|00⟩+|11⟩)/√2  —</strong> Bell state |Φ+⟩ — proved non-separable by coefficient matching argument</p>
<p><strong>Q10: (b) H=H†=H⁻¹  —</strong> H is Hermitian (H=H†), unitary (H†H=I), and involutory (H²=I), so H is its own inverse</p>
<p><strong>Q11: (c) (0,0,1)  —</strong> z=⟨Z⟩=1 for |0⟩ (north pole). x=⟨X⟩=0, y=⟨Y⟩=0</p>
<p><strong>Q12: (b) Complex phase of amplitudes creates interference  —</strong> The relative phase between α and β is physically real and creates interference — impossible classically</p>
<p><strong>Q13: (c) |0⟩⟨0|+|1⟩⟨1|=I  —</strong> Outer product sum over orthonormal basis equals identity — completeness relation</p>
<p><strong>Q14: (b) −1 with probability 1  —</strong> Z|+⟩=|−⟩; H|−⟩=|1⟩; measuring in Z gives 1 → X-eigenvalue −1 with probability 1</p>
<p><strong>Q15: (c) [[0.5,0],[0,0.5]]=I/2  —</strong> Trace out qubit 1: ρ₀=⟨0|₁|Φ+⟩⟨Φ+|₁|0⟩+⟨1|₁|Φ+⟩⟨Φ+|₁|1⟩=(1/2)|0⟩⟨0|+(1/2)|1⟩⟨1|=I/2</p>
</div>

#### D. Theory Questions

- State and prove the no-cloning theorem. Identify each step that requires linearity and unitarity. Give three specific applications of the no-cloning theorem in quantum technology.

- Explain why quantum superposition is physically different from a classical probability mixture. Describe a specific experiment (which observable measurement) that can distinguish |+⟩=(|0⟩+|1⟩)/√2 from the classical mixture ρ=(1/2)|0⟩⟨0|+(1/2)|1⟩⟨1|. What does this imply about the physical reality of quantum phases?

- Define the density matrix. What conditions must a valid density matrix satisfy? Give the density matrix for: (a) the pure state |ψ⟩=cos(π/3)|0⟩+sin(π/3)|1⟩, (b) the maximally mixed state, (c) a 30%/70% statistical mixture of |0⟩ and |+⟩. For each, compute purity and Von Neumann entropy.

- Describe the Bloch sphere representation of a qubit. How are the four fundamental principles (Linearity, Reversibility, Superposition, Entanglement) each visible in the Bloch sphere picture? What happens to the Bloch vector of an entangled qubit?

- State the four fundamental principles of quantum computing. For each, give: (a) a precise mathematical statement, (b) one example of how it appears in a specific quantum algorithm, (c) a specific circuit-level consequence for gate design.

- Explain the concept of quantum entanglement using the Bell state |Φ+⟩. Why is entanglement not the same as classical correlation? State Bell's theorem and explain how experimental violations of Bell's inequality prove that no local hidden-variable theory can reproduce quantum mechanics.

- What is quantum state tomography? Describe the protocol for reconstructing a single-qubit density matrix from measurements in Z, X, and Y bases. How many measurements are needed, and what is the accuracy achievable with N total shots?

- Explain the Holevo bound and the no-communication theorem. Why does quantum entanglement not allow faster-than-light communication, even though measuring one entangled qubit instantly collapses the other?

- Prove that for any two orthonormal bases {|u⟩,|v⟩} and {|+⟩,|−⟩}, the completeness relations |u⟩⟨u|+|v⟩⟨v|=I and |+⟩⟨+|+|−⟩⟨−|=I both equal the identity. What is the physical significance of the completeness relation?

- Describe how a transmon superconducting qubit works physically: what is the Josephson junction, why is anharmonicity essential, how are gates applied, and how is the state read out? Compare with trapped-ion qubits on three technical dimensions.

#### E. Programming Assignments

- [Qiskit] For each of the six cardinal Bloch sphere states (|0⟩,|1⟩,|+⟩,|−⟩,|i⟩,|−i⟩): (a) create the circuit, (b) compute statevector, (c) compute ⟨X⟩,⟨Y⟩,⟨Z⟩ as expectation values, (d) verify these equal (sinθcosφ, sinθsinφ, cosθ), (e) plot all six states on the Bloch sphere in a single figure using plot\_bloch\_multivector.

- [Qiskit] Implement density matrix purity scanning: for p ∈ {0, 0.1, 0.2, ..., 1.0}, create ρ = p|0⟩⟨0| + (1−p)|1⟩⟨1|. For each: compute purity Tr(ρ²), Von Neumann entropy S(ρ), and Bloch vector length |r|. Plot all three quantities vs p and verify they all agree at p=0 (pure |1⟩), p=0.5 (maximally mixed), and p=1 (pure |0⟩).

- [Qiskit] Run AQLL §2 (density matrix module). Record the purity and entropy from the AQLL output for the GHZ+i state. Then independently create the GHZ+i state in Qiskit: qc.h(0), qc.p(pi/2,0), qc.cx(0,1), qc.cx(0,2), qc.cx(0,3). Compute its full 16×16 density matrix and plot it using matplotlib.imshow() for Re(ρ), Im(ρ), and |ρᵢⱼ|. Verify purity=1 and entropy=0.

#### F. Project Suggestions

- Project 2.A — Quantum State Tomography: Implement full 2-qubit quantum state tomography. Measure in all 9 Pauli basis combinations (XX, XY, XZ, YX, YY, YZ, ZX, ZY, ZZ). Reconstruct ρ = (1/4)Σᵢⱼ⟨Pᵢ⊗Pⱼ⟩(Pᵢ⊗Pⱼ). Apply to the Bell state |Φ+⟩ on (a) Qiskit simulator (8192 shots) and (b) IBM Quantum hardware (1024 shots). Compute trace distance D(ρ\_ideal, ρ\_reconstructed) = (1/2)Tr|ρ−σ| for both. Write a 12-page report including the theory of tomography, your implementation, results, and analysis of hardware noise effects.

- Project 2.B — Entanglement Measures: Study four measures of bipartite entanglement: (a) entanglement entropy S(ρ\_A), (b) concurrence C(ρ), (c) negativity N(ρ), (d) entanglement of formation E\_F(ρ). Implement each in Python for general 2-qubit states. Generate 1000 random 2-qubit states and compute all four measures. Plot their correlations and discuss whether they are equivalent. Identify a state where they disagree most. Write a 15-page report.

- Project 2.C — Bell Inequality Experimental Simulation: Implement the full CHSH experiment in Qiskit. Prepare |Φ+⟩ and measure correlations E(a,b), E(a,b'), E(a',b), E(a',b') at optimal angles. Compute |S| = |E(a,b)−E(a,b')+E(a',b)+E(a',b')|. Show |S|>2 (Bell violation). Add depolarising noise at levels p=0,0.05,0.1,...,0.5 and find the critical p\_c where |S|=2 (violation disappears). Execute the ideal circuit on IBM Quantum hardware and compare with simulation. Write a 15-page report.

## REFERENCES AND FURTHER READING

### Primary Textbooks

- Nielsen M.A. and Chuang I.L. (2010). Quantum Computation and Quantum Information (10th Anniversary Ed.). Cambridge University Press. — The definitive graduate reference. Chapters 1–2 and 8–11 cover all Unit I material in full rigor.

- Mermin N.D. (2007). Quantum Computer Science: An Introduction. Cambridge University Press. — Exceptionally clear logical development; best for Sections 2.1–2.9 of this chapter.

- Hidary J.D. (2021). Quantum Computing: An Applied Approach (2nd Ed.). Springer. — Strong Qiskit integration; recommended for programming assignments.

- Sutor R. (2019). Dancing with Qubits. Packt Publishing. — Accessible introduction with IBM hardware focus.

### Research Papers

- Feynman R.P. (1982). 'Simulating Physics with Computers.' Int. J. Theor. Phys. 21, 467–488.

- Deutsch D. (1985). 'Quantum Theory, the Church-Turing Principle and the Universal Quantum Computer.' Proc. R. Soc. A 400, 97–117.

- Shor P.W. (1994). 'Algorithms for Quantum Computation.' FOCS 1994, 124–134.

- Grover L.K. (1996). 'A Fast Quantum Mechanical Algorithm for Database Search.' STOC 1996, 212–219.

- Wootters W.K. & Zurek W.H. (1982). 'A Single Quantum Cannot Be Cloned.' Nature 299, 802–803.

- Bell J.S. (1964). 'On the Einstein-Podolsky-Rosen Paradox.' Physics 1(3), 195–200.

- Aspect A., Dalibard J., Roger G. (1982). 'Experimental Test of Bell's Inequalities Using Time-Varying Analyzers.' PRL 49, 1804.

- Preskill J. (2018). 'Quantum Computing in the NISQ Era and Beyond.' Quantum 2, 79.
