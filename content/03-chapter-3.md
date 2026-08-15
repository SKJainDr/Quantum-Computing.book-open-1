# UNIT II · Chapter 3: Quantum Gates

# Single-Qubit Gates, Multi-Qubit Gates, and Universality

<div class="box box-anecdote">
<p class="box-title"><strong>📜  Rolf Landauer and the Physical Nature of Information — Yorktown Heights, 1961</strong></p>
<p>Rolf Landauer, working at IBM's Thomas J. Watson Research Center in 1961, made a profound discovery: erasing a bit of information necessarily dissipates a minimum energy of k_B T ln 2, where k_B is Boltzmann's constant and T is the temperature. This is Landauer's principle. The physical intuition is simple — a classical AND gate takes two bits in and gives one bit out; that lost bit of information must become heat.</p>
<p>Charles Bennett, also at IBM, showed in 1973 that computation itself need not be thermodynamically irreversible if every gate is logically reversible — meaning every output uniquely determines the inputs. The quantum gates of this chapter are exactly this: unitary transformations are bijections on the state space, so every quantum gate is automatically reversible. The Toffoli gate (Section 3.2.3) is the logical key: it makes classical AND gate reversible, unlocking the ability to run any classical algorithm on a quantum computer with zero logical heat dissipation. Feynman would later cite Landauer's work as one of the conceptual foundations of his 1982 quantum simulator proposal.</p>
</div>

Quantum gates are the elementary operations of quantum computation. Just as classical logic gates (AND, OR, NOT) transform classical bits, quantum gates transform qubits — but with a crucial difference: quantum gates are always reversible, always unitary, and can create and manipulate superposition and entanglement in ways impossible for classical logic.

This chapter develops the full vocabulary of quantum gates with rigorous mathematical precision. We begin with the single-qubit universe — the Pauli matrices, the Hadamard gate, phase gates, and rotation operators — and build to the multi-qubit realm where entanglement appears. The chapter culminates in one of the most beautiful theorems in computer science: any quantum computation can be performed using just three types of gate.

Every formula here connects to physical reality: each gate corresponds to a physical operation on a quantum system — a microwave pulse on a superconducting qubit, a laser pulse on a trapped ion. Understanding gates mathematically is understanding what quantum computers actually do.

## 3.1 Single-Qubit Gates

A single-qubit gate is a 2×2 unitary matrix U acting on the two-dimensional Hilbert space ℂ². Unitarity — the requirement that U†U = UU† = I — guarantees two things: the transformation is reversible (the gate has an inverse U†), and it preserves the Born rule (probabilities still sum to one after the gate acts). This section develops all the essential single-qubit gates used in quantum computing.

### 3.1.1 The Pauli Gates: X, Y, Z

The Pauli matrices are the three fundamental single-qubit operators, named after Wolfgang Pauli who introduced them in 1927 in the context of electron spin. In quantum computing they serve as the elementary gates from which more complex operations are constructed.

#### Matrix Representations

<div class="box box-equation">
<p><strong>Equation 3.1</strong></p>
<p><strong>Pauli-X (NOT gate):</strong></p>
<img class="eq-block" src="content/images/eq-image13.png" alt="equation">
<p><strong>Pauli-Y:</strong></p>
<img class="eq-block" src="content/images/eq-image14.png" alt="equation">
<p><strong>Pauli-Z:</strong></p>
<img class="eq-block" src="content/images/eq-image15.png" alt="equation">
</div>

<figure class="book-figure">
<img src="content/images/image16.png" alt="Figure 3.1: Bloch sphere representation of Pauli gates. X performs a π-rotation about the x-axis (|0⟩↔|1⟩), Y about the y-axis (with phase), and Z about the z-axis (phase flip of |1⟩).">
<figcaption>Figure 3.1: Bloch sphere representation of Pauli gates. X performs a π-rotation about the x-axis (|0⟩↔|1⟩), Y about the y-axis (with phase), and Z about the z-axis (phase flip of |1⟩).</figcaption>
</figure>

#### Key Algebraic Identities

The Pauli matrices satisfy a rich set of identities that are used constantly in quantum computing. We state them all:

- Self-inverse: X² = Y² = Z² = I (every Pauli is its own inverse)

- Anticommutation: XY = −YX = iZ, YZ = −ZY = iX, ZX = −XZ = iY

- The cyclic identity: XY = iZ, YZ = iX, ZX = iY (and reversal picks up a minus sign)

- Hermitian: X = X†, Y = Y†, Z = Z† (Pauli gates are self-adjoint, so they are both unitary and Hermitian)

- Traceless: Tr(X) = Tr(Y) = Tr(Z) = 0

- Determinant: det(X) = det(Y) = det(Z) = −1

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: The Pauli Group</strong></p>
<p>The Pauli group 𝒫₁ for one qubit consists of {±I, ±iI, ±X, ±iX, ±Y, ±iY, ±Z, ±iZ} — 16 elements in total. The factors of ±1 and ±i arise as global phases and are needed for the group to close under multiplication. For n qubits, the n-qubit Pauli group 𝒫ₙ consists of all n-fold tensor products of Pauli matrices with ±1 and ±i prefactors, giving 4^(n+1) elements. The Pauli group plays a central role in quantum error correction: stabiliser codes are defined by abelian subgroups of 𝒫ₙ.</p>
</div>

#### Bloch Sphere Action

On the Bloch sphere, where a qubit state is written as |ψ⟩ = cos(θ/2)|0⟩ + e^(iφ)sin(θ/2)|1⟩ corresponding to the unit vector n̂ = (sin θ cos φ, sin θ sin φ, cos θ):

- X performs a π-rotation about the x-axis: (θ,φ) → (π−θ, π+φ). Most visibly: north pole |0⟩ → south pole |1⟩.

- Y performs a π-rotation about the y-axis: (θ,φ) → (π−θ, −φ). It flips the qubit with an additional phase: Y|0⟩ = i|1⟩, Y|1⟩ = −i|0⟩.

- Z performs a π-rotation about the z-axis: (θ,φ) → (θ, φ+π). The north and south poles are eigenstates: Z|0⟩ = |0⟩, Z|1⟩ = −|1⟩.

<div class="box box-equation">
<p><strong>Example 3.1: Verifying XY = iZ by Matrix Multiplication</strong></p>
<p>We verify the fundamental Pauli relation XY = iZ directly.</p>
<p><strong>Compute XY:</strong></p>
<img class="eq-block" src="content/images/eq-image17.png" alt="equation">
<p><strong>Now compute iZ:</strong></p>
<img class="eq-block" src="content/images/eq-image18.png" alt="equation">
<p><strong>✓</strong></p>
<p>Conclusion: XY = iZ. ✓  Note: order matters =&gt;  YX = −iZ.</p>
</div>

<div class="box box-equation">
<p><strong>Example 3.2: Pauli-X Applied to a Superposition State</strong></p>
<p>Problem: Apply the Pauli-X gate to |ψ⟩ = (1/√3)|0⟩ + i√(2/3)|1⟩. Find the output state and verify normalisation.</p>
<p><strong>Solution:</strong></p>
<p>Represent |ψ⟩ as a column vector: |ψ⟩ = [1/√3, i√(2/3)]ᵀ</p>
<img class="eq-block" src="content/images/eq-image19.png" alt="equation">
<p><strong>Apply X:</strong></p>
<img class="eq-block" src="content/images/eq-image20.png" alt="equation">
<p>So, X|ψ⟩ = i√(2/3)|0⟩ + (1/√3)|1⟩. The amplitudes are swapped (this is the quantum NOT gate action).</p>
<p>Verify normalisation: |i√(2/3)|² + |1/√3|² = 2/3 + 1/3 = 1. ✓</p>
<p>Physical interpretation: X maps |0⟩→|1⟩ and |1⟩→|0⟩, so in the superposition it simply permutes the two amplitudes.</p>
</div>

### 3.1.2 The Hadamard Gate

The Hadamard gate is arguably the most important single-qubit gate in quantum computing. It is the superposition creator: it transforms definite classical states into equal superpositions, enabling quantum parallelism. Named after the French mathematician Jacques Hadamard, though its application to quantum computing was developed by Deutsch, Jozsa, and others in the early 1990s.

#### Matrix and Action

<div class="box box-equation">
<p><strong>Equation 3.2</strong></p>
<img class="eq-block" src="content/images/eq-image21.png" alt="equation">
<img class="eq-block" src="content/images/eq-image22.png" alt="equation">
<img class="eq-block" src="content/images/eq-image23.png" alt="equation">
<img class="eq-block" src="content/images/eq-image24.png" alt="equation">
<img class="eq-block" src="content/images/eq-image25.png" alt="equation">
<img class="eq-block" src="content/images/eq-image26.png" alt="equation">
</div>

<figure class="book-figure">
<img src="content/images/image27.png" alt="Figure 3.2: Hadamard gate action. Left: complete mapping of the six cardinal states. Right: H² = I demonstrating self-inverse property. The Hadamard rotates the Bloch sphere by π about the (x+z)/√2 axis.">
<figcaption>Figure 3.2: Hadamard gate action. Left: complete mapping of the six cardinal states. Right: H² = I demonstrating self-inverse property. The Hadamard rotates the Bloch sphere by π about the (x+z)/√2 axis.</figcaption>
</figure>

#### Key Identities Involving H

The Hadamard gate transforms between the X and Z bases, giving rise to the most-used identities in quantum circuit simplification:

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>HXH = Z        HZH = X        HYH = −Y</p>
<p>H(XZ)H = ZX = −XZ    (phase factor arising from anticommutation)</p>
<p>HH = I        (H is both unitary and Hermitian: H† = H)</p>
</div>

The identity HXH = Z is profoundly useful: it means a Z gate can be converted to an X gate (and vice versa) simply by conjugating with H. This is used constantly in circuit compilation to simplify circuits involving different Pauli bases.

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: H² = I Does NOT Mean H is Trivial</strong></p>
<p>Students sometimes think that since H² = I, applying H twice does nothing, and therefore H is somehow "weak." This misses the point entirely. H² = I means H is its own inverse — which makes it especially elegant and useful. The key fact is what H does to one application: it maps |0⟩ to a 50/50 superposition. Two applications return to the start. This is completely different from a classical identity operation at each step — after the first H, the system genuinely is in superposition, and quantum evolution between the two H gates operates on that superposition.</p>
</div>

<div class="box box-equation">
<p><strong>Example 3.3: Conjugation Identity HXH = Z</strong></p>
<p>Problem: Verify HXH = Z by direct matrix multiplication.</p>
<p><strong>Solution:</strong></p>
<p><strong>Step 1 — Compute XH:</strong></p>
<img class="eq-block" src="content/images/eq-image28.png" alt="equation">
<p><strong>Step 2 — Compute H(XH):</strong></p>
<img class="eq-block" src="content/images/eq-image29.png" alt="equation">
<p><strong>✓</strong></p>
<p>Physical meaning: The Hadamard basis-change lets us implement a Z rotation by doing an X rotation in the Hadamard-conjugated basis. This is the basis of many circuit optimisations.</p>
</div>

### 3.1.3 Phase Gates: S and T

Phase gates add a relative phase between the |0⟩ and |1⟩ components of a qubit state. While global phases are unobservable, relative phases are physically significant — they determine interference patterns and are the mechanism by which quantum algorithms like the QFT and Grover's algorithm create constructive and destructive interference.

#### Definitions and Matrices

<div class="box box-equation">
<p><strong>Equation 3.3</strong></p>
<p>S gate (π/2 phase gate):</p>
<img class="eq-block" src="content/images/eq-image30.png" alt="equation">
<p>T gate (π/4 phase gate, also called π/8 gate in some texts):</p>
<img class="eq-block" src="content/images/eq-image31.png" alt="equation">
<img class="eq-block" src="content/images/eq-image32.png" alt="equation">
<p>where</p>
<img class="eq-block" src="content/images/eq-image33.png" alt="equation">
</div>

<figure class="book-figure">
<img src="content/images/image34.png" alt="Figure 3.3: Phase gates S and T act in the equatorial plane of the Bloch sphere, rotating the |+⟩ state by π/2 (S) and π/4 (T) respectively. The x-axis corresponds to the |+⟩ state.">
<figcaption>Figure 3.3: Phase gates S and T act in the equatorial plane of the Bloch sphere, rotating the |+⟩ state by π/2 (S) and π/4 (T) respectively. The x-axis corresponds to the |+⟩ state.</figcaption>
</figure>

#### Relative Phase vs Global Phase

A global phase e^(iα) applied to an entire quantum state |ψ⟩ → e^(iα)|ψ⟩ is physically unobservable: no measurement can detect it. However, a relative phase between computational basis states is observable:

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>α|0⟩ + β|1⟩  →  (after Z gate)  →  α|0⟩ − β|1⟩</p>
<p>The relative phase between |0⟩ and |1⟩ changes from 0 to π.</p>
<p>(After Hadamard): H(α|0⟩ − β|1⟩) ≠ H(α|0⟩ + β|1⟩)</p>
<p>The phase becomes amplitude → measurable!</p>
</div>

#### Importance of T Gate for Universality

The T gate (also called the non-Clifford gate) plays a special role in quantum computing theory. It is the generator of the non-Clifford operations needed for universal quantum computation. The Clifford gates {H, S, CNOT} are efficiently simulable on a classical computer (Gottesman-Knill theorem, Section 3.3.3). Adding T to the Clifford set breaks this classical simulability and achieves universality. In fault-tolerant quantum computation, T gates are typically the most expensive operations to implement via magic state distillation, making T-count a critical circuit complexity metric.

<div class="box box-equation">
<p><strong>Example 3.4: Computing T|+⟩ and Comparing with S|+⟩</strong></p>
<p>Problem: Compute T|+⟩ and S|+⟩. Show that the output states have different relative phases. Then determine whether a standard Z-basis measurement can distinguish them.</p>
<p><strong>Solution — T|+⟩:</strong></p>
<p>|+⟩ = (1/√2)(|0⟩ + |1⟩) = (1/√2)[1, 1]ᵀ</p>
<img class="eq-block" src="content/images/eq-image35.png" alt="equation">
<img class="eq-block" src="content/images/eq-image36.png" alt="equation">
<p><strong>Solution — S|+⟩:</strong></p>
<p>S|+⟩ = (1/√2)(|0⟩ + i|1⟩)  [since S|1⟩ = i|1⟩]</p>
<p><strong>Z-basis measurement probabilities:</strong></p>
<p>For T|+⟩: P(0) = |1/√2|² = 1/2, P(1) = |e^(iπ/4)/√2|² = 1/2</p>
<p>For S|+⟩: P(0) = 1/2, P(1) = |i/√2|² = 1/2</p>
<p>Both give the same measurement statistics in the Z basis! The relative phase is undetectable in the Z basis.</p>
<p><strong>However, applying H before measurement (measuring in the X basis):</strong></p>
<p>HT|+⟩: probability of |0⟩ = |(1+e^(iπ/4))|²/4 = (1+cos(π/4))/2 ≈ 0.854 ≠ 0.5</p>
<p>HS|+⟩: probability of |0⟩ = |(1+i)|²/4 = 2/4 = 0.5 = |0⟩ measurement prob.</p>
<p>The relative phase IS detectable via interference — but only by measuring in the right basis.</p>
</div>

### 3.1.4 Rotation Gates: Rx(θ), Ry(θ), Rz(θ)

The rotation gates are the continuous-parameter family of single-qubit gates. They implement rotations by angle θ about the x, y, and z axes of the Bloch sphere. They are the physical gate set for most quantum hardware — a microwave pulse rotates the Bloch vector on a transmon qubit, and the pulse angle is exactly the θ parameter.

#### Definitions via Matrix Exponential

<div class="box box-equation">
<p><strong>Equation 3.4</strong></p>
<p>From the Lie group theory: rotation by θ about axis n̂ = e^(-iθσ_n/2)</p>
<img class="eq-block" src="content/images/eq-image37.png" alt="equation">
<p>Rx(θ) = exp(-iθX/2) = cos(θ/2)I − i·sin(θ/2)X</p>
<img class="eq-block" src="content/images/eq-image38.png" alt="equation">
<p>Ry(θ) = exp(-iθY/2) = cos(θ/2)I − i·sin(θ/2)Y</p>
<img class="eq-block" src="content/images/eq-image39.png" alt="equation">
<img class="eq-block" src="content/images/eq-image40.png" alt="equation">
<p>Rz(θ) = exp(-iθZ/2) =</p>
</div>

<figure class="book-figure">
<img src="content/images/image41.png" alt="Figure 3.4: Rotation gates on the Bloch sphere. Left: Rx(θ) rotates about x-axis — note the angle parameter is twice the Bloch angle. Right: Rz(θ) rotates about z-axis with no matrix mixing. Special values: Rx(π)=−iX, Ry(π)=−iY, Rz(π)=−iZ.">
<figcaption>Figure 3.4: Rotation gates on the Bloch sphere. Left: Rx(θ) rotates about x-axis — note the angle parameter is twice the Bloch angle. Right: Rz(θ) rotates about z-axis with no matrix mixing. Special values: Rx(π)=−iX, Ry(π)=−iY, Rz(π)=−iZ.</figcaption>
</figure>

#### Special Values

Important special values link rotation gates back to Pauli and phase gates:

- Rx(π) = −iX        Ry(π) = −iY        Rz(π) = −iZ     [up to global phase: physical Pauli gates]

- Rz(π/2) = e^(-iπ/4)S        Rz(π/4) = e^(-iπ/8)T     [S and T as special Rz cases, up to global phase]

- Rx(π/2)|0⟩ = (|0⟩ − i|1⟩)/√2 = |−i⟩        [creates the |y−⟩ eigenstate]

- Ry(π/2)|0⟩ = (|0⟩ + |1⟩)/√2 = |+⟩           [same result as H, up to global phase]

<div class="box box-equation">
<p><strong>Example 3.5: Verify Rx(π/2) and Rx(π) Explicitly</strong></p>
<p>Compute Rx(π/2) and Rx(π). Verify Rx(π) = −iX.</p>
<p><strong>Rx(π/2):</strong></p>
<p>cos(π/4) = 1/√2,    sin(π/4) = 1/√2</p>
<img class="eq-block" src="content/images/eq-image42.png" alt="equation">
<p><strong>Rx(π):</strong></p>
<p>cos(π/2) = 0,    sin(π/2) = 1</p>
<img class="eq-block" src="content/images/eq-image43.png" alt="equation">
<p>✓</p>
<p>Global phase −i is unobservable, confirming Rx(π) implements the X gate physically.</p>
</div>

### 3.1.5 ZYZ Euler Decomposition and the Universal Single-Qubit Gate U(θ,φ,λ)

Every 2×2 unitary matrix (every single-qubit gate) can be decomposed as a sequence of at most three rotation gates. This is the quantum analogue of Euler angle decomposition for 3D rotations, and it forms the foundation of single-qubit circuit compilation.

#### ZYZ Decomposition Theorem

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: ZYZ Decomposition</strong></p>
<p>For any 2×2 unitary matrix U ∈ U(2), there exist real angles α, β, γ, δ such that:</p>
<p>The proof follows from the structure of SU(2) ≅ S³ (the 3-sphere): any special unitary is a rotation in 3D via the covering map SU(2) → SO(3), and any 3D rotation has an Euler angle decomposition. The global phase lifts SU(2) to U(2).</p>
</div>

#### IBM Native Gate: U(θ,φ,λ)

IBM Quantum systems express every single-qubit gate as a single physical parametric gate U, defined as:

<div class="box box-equation">
<p><strong>Equation 3.6</strong></p>
<img class="eq-block" src="content/images/eq-image44.png" alt="equation">
<p><strong>Special cases:</strong></p>
<p>X = U(π, 0, π)       Y = U(π, π/2, π/2)     Z = U(0, 0, π)</p>
<p>H = U(π/2, 0, π)     S = U(0, 0, π/2        T = U(0, 0, π/4)</p>
<p>Rx(θ) = U(θ, −π/2, π/2)           Ry(θ) = U(θ, 0, 0)</p>
</div>

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: IBM Quantum Native Gate Set</strong></p>
<p>IBM Quantum hardware (Heron, Eagle, Falcon processors) implements all single-qubit operations as a single physical U gate using a shaped microwave pulse. The three parameters θ, φ, λ correspond to the pulse amplitude, initial phase, and final phase of the microwave envelope. In Qiskit, when you write "circuit.h(0)" you are telling the compiler to emit a U(π/2, 0, π) gate, which is then translated to a specific microwave pulse at the calibrated frequency of qubit 0. Understanding U(θ,φ,λ) is therefore not just mathematical abstraction — it is literally the physical operation executed on real quantum hardware.</p>
</div>

<div class="box box-equation">
<p><strong>Example 3.6: Express H and S as U(θ,φ,λ) Gates</strong></p>
<p>Problem: Express the Hadamard gate H and the S gate in U(θ,φ,λ) form. Verify by explicit matrix expansion.</p>
<p><strong>Hadamard H = U(π/2, 0, π):</strong></p>
<img class="eq-block" src="content/images/eq-image45.png" alt="equation">
<p>✓</p>
<p><strong>S gate = U(0, 0, π/2):</strong></p>
<img class="eq-block" src="content/images/eq-image46.png" alt="equation">
<img class="eq-block" src="content/images/eq-image47.png" alt="equation">
<p>✓</p>
</div>

```python
# ─────────────────────────────────────────────────────────────────────
# Code 3.1: Single-Qubit Gates in Qiskit
# Demonstrates Pauli, Hadamard, Phase, Rotation, and U gates
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.quantum_info import Statevector, Operator
import numpy as np

# ── Part 1: Basic single-qubit gates ──────────────────────────────────
qc = QuantumCircuit(1)
qc.x(0)           # Pauli-X gate
qc.y(0)           # Pauli-Y gate
qc.z(0)           # Pauli-Z gate
qc.h(0)           # Hadamard gate
qc.s(0)           # S gate (√Z)
qc.sdg(0)         # S† gate (S-dagger)
qc.t(0)           # T gate (⁴√Z)
qc.tdg(0)         # T† gate

# ── Part 2: Rotation gates ────────────────────────────────────────────
qc2 = QuantumCircuit(1)
qc2.rx(np.pi/4, 0)   # Rx(π/4) — rotate 45° about x-axis
qc2.ry(np.pi/3, 0)   # Ry(π/3) — rotate 60° about y-axis
qc2.rz(np.pi/2, 0)   # Rz(π/2) — rotate 90° about z-axis

# ── Part 3: Universal U gate ──────────────────────────────────────────
qc3 = QuantumCircuit(1)
theta, phi, lam = np.pi/2, 0, np.pi  # U(π/2, 0, π) = Hadamard
qc3.u(theta, phi, lam, 0)

# ── Part 4: Verify HXH = Z using Statevector ─────────────────────────
test = QuantumCircuit(1)
test.h(0)
test.x(0)
test.h(0)
# This should equal Z (up to global phase)
op_hxh = Operator(test)
op_z   = Operator.from_label("Z")
print("HXH equal to Z?", op_hxh.equiv(op_z))   # True

# ── Part 5: State after H gate ───────────────────────────────────────
sv = QuantumCircuit(1)
sv.h(0)
state = Statevector.from_instruction(sv)
print(f"|+> state: {state}")   # [0.707+0j, 0.707+0j]

# ── Part 6: Bloch sphere visualisation ───────────────────────────────
from qiskit.visualization import plot_bloch_multivector
sv2 = QuantumCircuit(1)
sv2.ry(np.pi/3, 0)   # Tilt qubit from north pole
sv2.rz(np.pi/4, 0)   # Add azimuthal phase
state2 = Statevector.from_instruction(sv2)
fig = plot_bloch_multivector(state2)
fig.savefig("bloch_single.png", dpi=150)
```

## 3.2 Multi-Qubit Gates

Multi-qubit gates act on two or more qubits simultaneously. They are the mechanism by which qubits become correlated — and specifically by which entanglement is created. Without multi-qubit gates, quantum computation would reduce to independent single-qubit processing, with no computational advantage over classical. The gates in this section are the building blocks of every quantum algorithm.

Multi-qubit states live in the tensor product Hilbert space: a two-qubit system lives in ℂ² ⊗ ℂ² = ℂ⁴ (four-dimensional), a three-qubit system in ℂ⁸, and n qubits in ℂ^(2ⁿ). Gates on this space are 2ⁿ × 2ⁿ unitary matrices.

### 3.2.1 The CNOT (CX) Gate

#### Definition and Matrix

The CNOT (Controlled-NOT, or CX) gate is the most important two-qubit gate. It acts on a control qubit and a target qubit: if the control is |1⟩, it flips (applies X to) the target; if the control is |0⟩, the target is unchanged.

<div class="box box-equation">
<p><strong>Equation 3.7</strong></p>
<p>CNOT|c,t⟩ = |c, t⊕c⟩    where ⊕ is addition mod 2 (XOR)</p>
<p>In the computational basis {|00⟩, |01⟩, |10⟩, |11⟩}:</p>
<img class="eq-block" src="content/images/eq-image48.png" alt="equation">
<p>Truth Table:</p>
<img class="eq-block" src="content/images/eq-image49.png" alt="equation">
<p>(target flips only when control=|1⟩)</p>
</div>

<figure class="book-figure">
<img src="content/images/image50.png" alt="Figure 3.5: CNOT gate. Left: truth table showing the target flip when control=|1⟩. Right: H⊗I followed by CNOT creates the Bell state |Φ⁺⟩ = (|00⟩+|11⟩)/√2 — the prototype entangled state.">
<figcaption>Figure 3.5: CNOT gate. Left: truth table showing the target flip when control=|1⟩. Right: H⊗I followed by CNOT creates the Bell state |Φ⁺⟩ = (|00⟩+|11⟩)/√2 — the prototype entangled state.</figcaption>
</figure>

#### Bell State Creation: H + CNOT

The most important two-gate sequence in quantum computing: applying H to the control qubit, then CNOT, creates a maximally entangled Bell state from a product input. This is the fundamental entanglement-creation primitive.

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>Step 1: H⊗I acts on |00⟩:</p>
<p>|00⟩ → (|0⟩+|1⟩)/√2 ⊗ |0⟩ = (|00⟩+|10⟩)/√2</p>
<p>Step 2: CNOT acts on (|00⟩+|10⟩)/√2:</p>
<p><strong>(|00⟩+|10⟩)/√2 → (|00⟩+|11⟩)/√2 = |Φ⁺⟩</strong></p>
<p>This Bell state is maximally entangled: reduced density matrix of either qubit is I/2</p>
</div>

#### CNOT Identities

- CNOT² = I        (applying CNOT twice returns to the original state)

- CNOT is equivalent to classical XOR on computational basis states

- (H⊗H)·CNOT\_(c→t)·(H⊗H) = CNOT\_(t→c)        (Hadamard conjugation reverses control and target)

- CNOT can be used to copy classical information: CNOT|x,0⟩ = |x,x⟩ (but NOT quantum states — no-cloning theorem)

<div class="box box-example">
<p class="box-title"><strong>Example 3.7: The Four Bell States via H+CNOT with Different Inputs</strong></p>
<p>Problem: Compute the output of the H+CNOT circuit for all four two-qubit basis inputs: |00⟩, |01⟩, |10⟩, |11⟩.</p>
<p><strong>For |00⟩:</strong></p>
<p>H|0⟩⊗|0⟩ → (|0⟩+|1⟩)/√2 ⊗ |0⟩ → CNOT → (|00⟩+|11⟩)/√2 = |Φ⁺⟩</p>
<p><strong>For |01⟩:</strong></p>
<p>H|0⟩⊗|1⟩ → (|0⟩+|1⟩)/√2 ⊗ |1⟩ → CNOT → (|01⟩+|10⟩)/√2 = |Ψ⁺⟩</p>
<p><strong>For |10⟩:</strong></p>
<p>H|1⟩⊗|0⟩ → (|0⟩−|1⟩)/√2 ⊗ |0⟩ → CNOT → (|00⟩−|11⟩)/√2 = |Φ⁻⟩</p>
<p><strong>For |11⟩:</strong></p>
<p>H|1⟩⊗|1⟩ → (|0⟩−|1⟩)/√2 ⊗ |1⟩ → CNOT → (|01⟩−|10⟩)/√2 = |Ψ⁻⟩</p>
<p>The four Bell states {|Φ⁺⟩, |Φ⁻⟩, |Ψ⁺⟩, |Ψ⁻⟩} form an orthonormal basis for ℂ² ⊗ ℂ² and are the maximally entangled states used in quantum teleportation, superdense coding, and quantum key distribution.</p>
</div>

### 3.2.2 Controlled-Z, SWAP, and iSWAP

#### Controlled-Z (CZ) Gate

<div class="box box-equation">
<p><strong>Equation 3.8</strong></p>
<img class="eq-block" src="content/images/eq-image51.png" alt="equation">
<p>Applies Z to target only when control=|1⟩</p>
<p>Equivalently: CZ = (I⊗H)·CNOT·(I⊗H)</p>
<p>CZ is symmetric in control and target!</p>
</div>

The CZ gate is preferred over CNOT on some hardware platforms (e.g., Google Sycamore, which uses CZ as its native 2-qubit gate). Its symmetric action (either qubit can be called the "control") is particularly elegant. Note that CZ|11⟩ = −|11⟩ — it applies a phase flip to the |11⟩ component only.

#### SWAP Gate

<div class="box box-equation">
<p><strong>Equation 3.9</strong></p>
<img class="eq-block" src="content/images/eq-image52.png" alt="equation">
<p>Swaps the states of two qubits</p>
<p>SWAP = CNOT_(12)·CNOT_(21)·CNOT_(12)</p>
<p>(three CNOT decomposition)</p>
</div>

SWAP gates are critical in quantum circuit compilation: when two logically connected qubits are not physically adjacent on a device, SWAP gates must be inserted to bring them together. This is called SWAP routing and is one of the main sources of circuit overhead on real hardware (Section 3.3 of Chapter 4).

#### iSWAP Gate

<div class="box box-equation">
<p><strong>Equation 3.10</strong></p>
<img class="eq-block" src="content/images/eq-image53.png" alt="equation">
<p>Swaps with a phase factor of i</p>
<p>Native gate of some superconducting processors</p>
</div>

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: CNOT and CZ Are Equivalent — Not the Same</strong></p>
<p>CNOT and CZ are equivalent in the sense that each can be converted to the other using single-qubit gates: CZ = (I⊗H)·CNOT·(I⊗H). However, on real hardware they are different physical operations, have different error rates, and require different pulse sequences. When targeting a specific hardware platform (IBM uses CNOT/CX, Google uses CZ), always compile to the correct native 2-qubit gate. Failing to do so can cause the compiler to insert unnecessary conversions, increasing circuit depth and error.</p>
</div>

### 3.2.3 Toffoli (CCX) Gate: Reversible AND

The Toffoli gate, proposed by Tommaso Toffoli in 1980, is the three-qubit gate that makes quantum computing classically universal. It is sometimes called the CCX (controlled-controlled-X) gate: it applies X (NOT) to the target qubit only when both control qubits are in state |1⟩.

<div class="box box-equation">
<p><strong>Equation 3.11</strong></p>
<p>CCX|c₁,c₂,t⟩ = |c₁,c₂, t⊕(c₁·c₂)⟩</p>
<p>As a truth table modification: only |110⟩↔|111⟩ are swapped</p>
<p>All other 3-qubit basis states are unchanged</p>
</div>

<figure class="book-figure">
<img src="content/images/image54.png" alt="Figure 3.6: Multi-qubit gates. The Toffoli gate truth table (left): only |110⟩↔|111⟩ are exchanged — quantum AND gate. The SWAP gate circuit decomposition (right): three CNOT gates implement SWAP.">
<figcaption>Figure 3.6: Multi-qubit gates. The Toffoli gate truth table (left): only |110⟩↔|111⟩ are exchanged — quantum AND gate. The SWAP gate circuit decomposition (right): three CNOT gates implement SWAP.</figcaption>
</figure>

#### Toffoli Gate Decomposition

The Toffoli gate can be decomposed into single-qubit gates and CNOTs. The standard decomposition requires 6 CNOTs:

```python
# Standard Toffoli decomposition in Qiskit (6 CNOT gates)
from qiskit import QuantumCircuit
import numpy as np

# ── Method 1: Direct Toffoli gate ────────────────────────────────────
qc = QuantumCircuit(3)
qc.ccx(0, 1, 2)   # Toffoli: control=q0,q1, target=q2

# ── Method 2: Manual decomposition (educational) ─────────────────────
qc2 = QuantumCircuit(3)
qc2.h(2)           # Hadamard on target
qc2.cx(1, 2)       # CNOT: c=q1, t=q2
qc2.tdg(2)         # T† on target
qc2.cx(0, 2)       # CNOT: c=q0, t=q2
qc2.t(2)           # T on target
qc2.cx(1, 2)       # CNOT: c=q1, t=q2
qc2.tdg(2)         # T† on target
qc2.cx(0, 2)       # CNOT: c=q0, t=q2
qc2.t(1)           # T on q1
qc2.t(2)           # T on target
qc2.h(2)           # Hadamard on target
qc2.cx(0, 1)       # CNOT: c=q0, t=q1
qc2.t(0)           # T on q0
qc2.tdg(1)         # T† on q1
qc2.cx(0, 1)       # CNOT: c=q0, t=q1
# Total: 6 CNOTs + 7 single-qubit gates
```

#### Why Toffoli Is Special: Reversible Classical Computation

The Toffoli gate achieves something remarkable: it allows any classical Boolean circuit to be implemented reversibly. The key insight:

- Classical AND gate: (a,b) → a∧b is IRREVERSIBLE (two bits in, one bit out; information destroyed)

- Toffoli gate: (a,b,c=0) → (a,b, a∧b) is REVERSIBLE (three bits in, three bits out; information preserved)

- Classical NOT is already reversible; combined with Toffoli, the gate set {Toffoli, NOT} is universal for reversible classical computation

- Any classical circuit → equivalent reversible circuit using Toffoli + ancilla qubits → quantum circuit

This means quantum computers are at least as powerful as classical computers for any computable function — they can simulate any classical algorithm with polynomial overhead. The question of quantum advantage is about whether quantum computers can do certain things FASTER, not whether they can do more.

<div class="box box-example">
<p class="box-title"><strong>Example 3.8: Implementing Classical AND Reversibly with Toffoli</strong></p>
<p>Problem: Show how to compute a AND b reversibly using the Toffoli gate. Implement in Qiskit and verify for all 4 input combinations.</p>
<p><strong>Strategy: Use Toffoli with c=|0⟩ as ancilla. Output: |a,b, a∧b⟩</strong></p>
</div>

### 3.2.4 Fredkin (CSWAP) Gate

The Fredkin gate, named after Ed Fredkin who proposed it in 1982, is the controlled-SWAP gate. It swaps the states of two target qubits conditionally, based on a control qubit.

<div class="box box-equation">
<p><strong>Equation 3.12</strong></p>
<p>Fredkin: CSWAP|c,a,b⟩ = |c, a⊕c(a⊕b), b⊕c(a⊕b)⟩</p>
<p>If c=|0⟩: targets unchanged: |0,a,b⟩ → |0,a,b⟩</p>
<p>If c=|1⟩: targets swapped:   |1,a,b⟩ → |1,b,a⟩</p>
<p>CSWAP = CNOT_(32)·CCNOT_(132)·CNOT_(32)   (one Toffoli + two CNOTs)</p>
</div>

#### Applications of CSWAP

- Quantum multiplexer: routes qubit states to different outputs based on control qubit, analogous to a classical 2:1 MUX

- Fingerprinting and SWAP test: applying CSWAP with control in |+⟩ and measuring the control qubit tests whether two quantum states are equal (SWAP test)

- Reversible sorting: CSWAP is the quantum equivalent of a conditional swap in sorting networks (e.g., quantum bubble sort)

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: The SWAP Test</strong></p>
<p>Given two unknown quantum states |φ⟩ and |ψ⟩, the SWAP test determines |⟨φ|ψ⟩|² without knowing the states explicitly. Circuit: prepare ancilla in |+⟩, apply CSWAP with ancilla as control and |φ⟩, |ψ⟩ as targets, apply H to ancilla, measure ancilla. P(ancilla=0) = (1 + |⟨φ|ψ⟩|²)/2. If P(0)=1, the states are identical; if P(0)=1/2, they are orthogonal. This non-destructive overlap estimation is used in quantum machine learning algorithms.</p>
</div>

## 3.3 Universality, the Solovay-Kitaev Theorem, and the Clifford Group

The central theorem of quantum circuit complexity asserts that any quantum computation, no matter how complex, can be performed using only three types of gate: Hadamard (H), the T gate (π/8 gate), and the two-qubit CNOT gate. This is quantum universality, and understanding it deeply — not just as a fact but as a mathematical result — is essential for anyone working on quantum algorithm design or hardware compilation.

### 3.3.1 What Is a Universal Gate Set?

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Universal Gate Set — Formal Definition</strong></p>
<p>A finite set of quantum gates 𝒢 is universal for quantum computation if, for any n-qubit unitary operator U ∈ U(2ⁿ) and any precision ε &gt; 0, there exists a circuit C using gates from 𝒢 (and their inverses) such that:</p>
<p>In other words, any target unitary can be approximated to any desired precision using gates from 𝒢. The number of gates required depends on both the target U and the desired precision ε.</p>
</div>

<figure class="book-figure">
<img src="content/images/image55.png" alt="Figure 3.7: The {H, T, CNOT} universal gate set. Any quantum algorithm can be compiled into these three gate types. The Solovay-Kitaev theorem guarantees efficient approximation: the gate count grows only as O(polylog(1/ε)).">
<figcaption>Figure 3.7: The {H, T, CNOT} universal gate set. Any quantum algorithm can be compiled into these three gate types. The Solovay-Kitaev theorem guarantees efficient approximation: the gate count grows only as O(polylog(1/ε)).</figcaption>
</figure>

#### Examples of Universal Gate Sets

The following gate sets are all universal for quantum computation:

| Gate Set | Comment | Used By |
|---|---|---|
| {H, T, CNOT} | Standard universal set; minimum for fault-tolerant QC | Theoretical standard |
| {H, S, T, CNOT} | Clifford + T gate set; T is expensive in FT-QC | Fault-tolerant hardware |
| {U, CX} | IBM native: any single-qubit + CNOT; most efficient | IBM Quantum |
| {CZ, H, T} | CZ replaces CNOT; equivalent power | Google Sycamore |
| {Rx, Ry, CX} | Rotation + CNOT; hardware-natural for ion traps | IonQ, Quantinuum |
| {H, T, Toffoli} | Classically universal + Hadamard; T-count efficiency | Reversible computing |

### 3.3.2 The Solovay-Kitaev Theorem

Knowing that a gate set is universal tells us any unitary can be approximated — but it does not tell us how efficiently. The Solovay-Kitaev theorem is the quantitative universality result that makes quantum computing practical.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Solovay-Kitaev Theorem (Statement)</strong></p>
<p>Let 𝒢 be a finite universal gate set closed under inverses, and let U be any single-qubit unitary. Then there exists a sequence S of gates from 𝒢 such that:</p>
<p>This is dramatically better than the naive estimate of O(1/ε). It means approximating a unitary to precision ε = 10⁻¹⁰ requires only about 40³·⁹⁷ ≈ 10⁷ gates in the worst case — not 10¹⁰ gates. The algorithm to find this approximation runs in time polynomial in log(1/ε) as well.</p>
</div>

The practical implication: to compile an arbitrary quantum algorithm into the {H, T, CNOT} gate set, each single-qubit rotation Rz(θ) is replaced by a sequence of H and T gates approximating it to the required precision. The Solovay-Kitaev algorithm finds this sequence efficiently. Modern compilers (Qiskit, TKET) implement improved variants of this algorithm.

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: T-Count Optimisation in Fault-Tolerant Quantum Computing</strong></p>
<p>In fault-tolerant quantum computers, the most expensive operations are T gates. While Clifford gates (H, S, CNOT) can be implemented transversally with very low overhead in most error-correcting codes, T gates require magic state distillation — a resource-intensive process that requires preparing and purifying many noisy T states. A single logical T gate may require thousands of physical operations. As a result, quantum algorithm designers actively minimise the T-count of their circuits. For example, Grover's oracle typically requires many Toffoli gates, each of which decomposes into 7 T gates. A 100-qubit Grover instance with 1000 Toffolis requires approximately 7,000 T gates — and the total physical resources for fault-tolerant implementation scale linearly with this T-count.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 3.9: Approximating Rz(π/5) with {H, T} Gates</strong></p>
<p>Problem: Estimate how many H and T gates are needed to approximate Rz(π/5) to precision ε = 0.01 using the Solovay-Kitaev theorem. Compare with naive count.</p>
<p><strong>Solovay-Kitaev estimate:</strong></p>
<p>Gate count ≤ O(log^c(1/ε)) = O(log^{3.97}(100))</p>
<p>log₂(100) ≈ 6.64</p>
<p>6.64^{3.97} ≈ 6.64^4 ≈ 1,948</p>
<p>So Rz(π/5) can be approximated to precision 0.01 using roughly O(2000) H and T gates.</p>
<p><strong>Naive estimate (grid approximation):</strong></p>
<p>If we try all sequences of length k, the number of gate sequences is 2^k (for {H,T}). For the approximation to work by random coverage of SU(2), we need coverage at scale ε, requiring k ≈ 1/ε = 100 gates.</p>
<p><strong>Comparison: S-K gives O(2000) — seems worse! But note:</strong></p>
<p>• The S-K algorithm is constructive — it finds the approximating sequence in time poly(log(1/ε))</p>
<p>• Naive search requires checking all sequences — which is exponential in k</p>
<p>• In practice, for {H,T}, optimal sequences have been tabulated and typical T-counts are 3×log₁₀(1/ε) — about 6 T gates for ε=10⁻²</p>
</div>

### 3.3.3 The Clifford Group and the Gottesman-Knill Theorem

#### The Clifford Group

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Clifford Group</strong></p>
<p>The n-qubit Clifford group 𝒞ₙ is the group of unitary operators that map the n-qubit Pauli group 𝒫ₙ to itself under conjugation:</p>
<p>For single qubits, the Clifford group 𝒞₁ is generated by {H, S, X}. This group has 24 elements (the single-qubit Clifford group is isomorphic to the symmetric group S₄ × Z₂). For n qubits, the Clifford group is generated by {H, S, CNOT}.</p>
<p><strong>Key fact: Every element of the Clifford group maps Paulis to Paulis. For example:</strong></p>
<p>HXH† = Z    HZH† = X    SXS† = Y    SZS† = Z</p>
</div>

#### Gottesman-Knill Theorem

The Gottesman-Knill theorem is one of the most important results in quantum computing theory. It tells us precisely which quantum circuits can be simulated efficiently on a classical computer.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Gottesman-Knill Theorem (Statement)</strong></p>
<p>Any quantum circuit that:</p>
<p>Begins with computational basis inputs (|0⟩^⊗n),</p>
<p>Uses only Clifford gates (H, S, CNOT, Pauli gates, measurements), and</p>
<p>Ends with computational basis measurements,</p>
<p>can be simulated classically in polynomial time O(n²) using the stabiliser formalism.</p>
<p><strong>Corollary: Clifford circuits alone provide NO exponential quantum advantage. To achieve exponential speedup, at least one non-Clifford gate (typically T) is necessary.</strong></p>
</div>

#### Stabiliser Formalism

The Gottesman-Knill theorem exploits the stabiliser representation of quantum states. Instead of tracking the full 2ⁿ-dimensional state vector (which is exponentially large), one tracks the n generators of the stabiliser group — the set of Pauli operators that leave the state invariant:

<div class="box box-equation">
<p><strong>Equation 3.16</strong></p>
<p>Stabiliser group of |ψ⟩: Stab(|ψ⟩) = {P ∈ 𝒫ₙ : P|ψ⟩ = |ψ⟩}</p>
<p>Example: |00⟩ has stabilisers +Z₁ and +Z₂</p>
<p>Bell state |Φ⁺⟩ has stabilisers +X₁X₂ and +Z₁Z₂</p>
<p>(The Bell state is the +1 eigenstate of X⊗X and Z⊗Z)</p>
</div>

Under Clifford gates, stabilisers transform as P → CPC†, which remains a Pauli operator. So n generators update as n Pauli strings — an O(n²) operation per gate step. This is the classical simulation efficiency.

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: The Clifford + Measurement Boundary Is Sharp</strong></p>
<p>Students sometimes think: "if Clifford circuits are easy to simulate, why not just use them?" The answer is that Clifford circuits cannot perform universal computation. Specifically, no Clifford-only circuit can compute an AND gate on arbitrary inputs — the output is always a linear function over GF(2). The moment you add a single T gate, the circuit becomes universal and (apparently) hard to simulate classically. This sharp boundary — between efficiently simulable Clifford circuits and hard-to-simulate universal circuits — is believed to be real (it is related to the conjectured hardness of simulating quantum circuits), but proving it would have major consequences in computational complexity theory.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 3.10: Verifying HXH† = Z Using Stabilisers</strong></p>
<p>Problem: Verify the Clifford conjugation relation HXH = Z by computing the action on the stabiliser of |0⟩.</p>
<p><strong>The stabiliser of |0⟩:</strong></p>
<p>|0⟩ is the +1 eigenstate of Z: Z|0⟩ = +|0⟩. So Stab(|0⟩) = ⟨Z⟩.</p>
<p><strong>Apply H: |0⟩ → |+⟩. What stabilises |+⟩?</strong></p>
<p>Compute HZH: HZH = (HXH)^(-1)... actually let's compute directly:</p>
<p>HZH = (1/√2)[1 1][1  0][1  1] = (1/√2)[1 1][1  −1] = [1  0] — wait, let's be careful:</p>
<p>HZH = X   (as we verified in Example 3.3, HXH = Z, so also HZH = X by symmetry)</p>
<p><strong>So H maps stabiliser Z → HZH = X.</strong></p>
<p>This means |+⟩ is stabilised by X: X|+⟩ = X(|0⟩+|1⟩)/√2 = (|1⟩+|0⟩)/√2 = |+⟩. ✓</p>
<p>This is exactly the stabiliser update rule: when H acts on a qubit, its stabiliser P → HPH. Since HZH = X, the stabiliser Z of |0⟩ becomes X, which is the stabiliser of |+⟩. The Gottesman-Knill theorem tells us we can track all these stabiliser updates with O(n²) classical overhead.</p>
</div>

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 3: Quantum Gates: Single-Qubit, Multi-Qubit, and Universality

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Define a single-qubit gate mathematically. What physical condition must it satisfy and why does this condition have two physical consequences?**

**Q2.  Write the Pauli-X gate matrix. What is its action on |0⟩ and |1⟩? Why is it called the quantum NOT gate? What is its Bloch sphere geometric interpretation?**

**Q3.  State any three algebraic properties of Pauli matrices and prove one of them (e.g., XY = iZ) by explicit matrix multiplication.**

**Q4.  Define the Hadamard gate H. Prove H² = I by matrix multiplication. State the three conjugation identities HXH, HZH, and HYH.**

**Q5.  Define the S and T phase gates. What are their matrix representations? State the composition relations T² = S, S² = Z, T⁸ = I. Why is the T gate essential for universal quantum computation?**

**Q6.  Write the general rotation gate Rₓ(θ) using the matrix exponential. What is its geometric interpretation on the Bloch sphere? Give the special case Rₓ(π) in terms of Pauli matrices.**

**Q7.  State the ZYZ Euler decomposition theorem. Write IBM's universal U(θ,φ,λ) gate. Express X, H, and S as special cases of U(θ,φ,λ).**

**Q8.  Define the CNOT gate. Write its matrix representation and truth table. Show that CNOT applied to (H⊗I)|00⟩ produces the Bell state |Φ+⟩.**

**Q9.  What is the Toffoli (CCX) gate? How many CNOT gates does its standard decomposition require? What is its T-count? Why is it called a reversible AND gate?**

**Q10.  State the Solovay-Kitaev theorem. What does it guarantee about the efficiency of gate set compilation? Why is the {H, T, CNOT} set universal?**

**Q11.  State the Gottesman-Knill theorem. What is the Clifford group? Why can Clifford circuits be simulated efficiently classically, and what does adding T gates change?**

**Q12.  A circuit contains 30 Toffoli gates. (a) How many CNOTs result from decomposition? (b) If each CNOT has 0.4% error, what is the circuit fidelity? (c) What does this reveal about NISQ limitations?**

**Q13.  What is the physical difference between global phase and relative phase? Why is global phase physically unobservable? Give an example where relative phase determines a measurement outcome.**

**Q14.  Describe the CZ gate. Show that CZ = (I⊗H)·CNOT·(I⊗H). Why is CZ preferred over CNOT on some hardware platforms?**

**Q15.  What is the SWAP gate? How many CNOTs are needed to implement it? Why is SWAP routing necessary in quantum computing and what is its cost on circuit depth?**

**PART B — MODEL ANSWERS**

**Answer 1:**

A single-qubit gate is a 2×2 unitary matrix U acting on the state space ℂ² of a single qubit. Mathematical condition: U†U = UU† = I₂. Two physical consequences: (1) Reversibility: every gate has exact inverse U†; applying U† after U returns to original state. (2) Norm-preservation: probabilities sum to one before and after the gate, since ||U|ψ⟩||² = ⟨ψ|U†U|ψ⟩ = ⟨ψ|ψ⟩ = 1.

**Answer 2:**

X = [[0,1],[1,0]]. X|0⟩ = |1⟩, X|1⟩ = |0⟩ — it swaps the computational basis states, analogous to classical NOT. It is the quantum NOT gate because it flips the state: 0 becomes 1 and 1 becomes 0, just as NOT flips a classical bit. Bloch sphere: rotation by π (180°) about the x-axis, mapping north pole ↔ south pole.

**Answer 3:**

Three properties: (1) Self-inverse: X² = Y² = Z² = I. (2) Anticommutation: XY = −YX = iZ. (3) Hermitian: P† = P. Proof of XY = iZ: XY = [[0,1],[1,0]][[0,−i],[i,0]] = [[0×0+1×i, 0×(−i)+1×0],[1×0+0×i, 1×(−i)+0×0]] = [[i,0],[0,−i]] = i[[1,0],[0,−1]] = iZ ✓.

**Answer 4:**

H = (1/√2)[[1,1],[1,−1]]. H² = (1/2)[[1,1],[1,−1]][[1,1],[1,−1]] = (1/2)[[1+1, 1−1],[1−1, 1+1]] = (1/2)[[2,0],[0,2]] = I₂ ✓. H is self-inverse (H⁻¹ = H). Conjugation identities: HXH = Z (interchanges X and Z bases); HZH = X; HYH = −Y (Y is antisymmetric under the H basis change).

**Answer 5:**

S = [[1,0],[0,i]]: adds phase i to |1⟩, leaves |0⟩ unchanged. T = [[1,0],[0,exp(iπ/4)]]: adds phase exp(iπ/4) to |1⟩. T² = diag(1, exp(iπ/2)) = diag(1,i) = S ✓. S² = diag(1,i²) = diag(1,−1) = Z ✓. T⁸ = diag(1, exp(i8π/4)) = diag(1, exp(2πi)) = I ✓. T is essential because the Clifford group {H,S,CNOT} is efficiently classically simulable (Gottesman-Knill). Adding T breaks this simulability and achieves universal quantum computation.

**Answer 6:**

Rₓ(θ) = exp(−iθX/2) = cos(θ/2)·I − i·sin(θ/2)·X = [[cos(θ/2), −i·sin(θ/2)],[−i·sin(θ/2), cos(θ/2)]]. Bloch sphere: rotates the Bloch vector about the x-axis by angle θ. Special case Rₓ(π): Rₓ(π) = cos(π/2)·I − i·sin(π/2)·X = −iX (Pauli-X up to a global phase of −i). This is why X and Rₓ(π) implement the same physical gate up to a global phase.

**Answer 7:**

ZYZ theorem: any U ∈ U(2) = exp(iα)·Rz(β)·Ry(γ)·Rz(δ). IBM's U(θ,φ,λ) = [[cos(θ/2), −exp(iλ)sin(θ/2)],[exp(iφ)sin(θ/2), exp(i(φ+λ))cos(θ/2)]]. Special cases: X = U(π,0,π); H = U(π/2,0,π); S = U(0,0,π/2). The U gate is the single physical gate used by IBM to implement all single-qubit operations in one hardware operation.

**Answer 8:**

CNOT = [[1,0,0,0],[0,1,0,0],[0,0,0,1],[0,0,1,0]]. Truth table: |00⟩→|00⟩, |01⟩→|01⟩, |10⟩→|11⟩, |11⟩→|10⟩ (XOR). Bell state creation: (H⊗I)|00⟩ = (|0⟩+|1⟩)/√2 ⊗ |0⟩ = (|00⟩+|10⟩)/√2. CNOT maps: |00⟩→|00⟩ and |10⟩→|11⟩. Result: (|00⟩+|11⟩)/√2 = |Φ+⟩ ✓.

**Answer 9:**

Toffoli (CCX): applies X to target qubit only when BOTH control qubits are |1⟩. Standard decomposition: 6 CNOTs + 9 single-qubit gates. T-count: 7. Reversible AND: CCX|a,b,0⟩ = |a,b,a∧b⟩ — computes AND of a and b into the third qubit while preserving a and b. The original inputs are recoverable from the output, making it reversible (unlike classical AND which destroys the inputs).

**Answer 10:**

Solovay-Kitaev theorem: for any finite universal gate set 𝒢 closed under inverses, and any U ∈ SU(2), there exists an approximating sequence of length O(log^c(1/ε)) (c ≈ 3.97) that approximates U to precision ε. Guarantees: no gate set is weaker than any other; the cost of switching is at most polylogarithmic. {H,T,CNOT} is universal because: H creates superposition, T provides non-Clifford phase (breaking classical simulability), and CNOT creates entanglement — these three together can approximate any quantum computation.

**Answer 11:**

Gottesman-Knill theorem: any quantum circuit using only Clifford gates {H, S, X, Y, Z, CNOT, Pauli measurements} can be simulated classically in O(n²) time using the stabiliser formalism. Clifford group: generated by {H, S, CNOT} — maps Paulis to Paulis under conjugation. Classical simulation exploits a compact O(n²) binary matrix representation of the state. Adding T gates: T conjugation doesn't map Paulis to Paulis (T·X·T† = (X+Y)/√2 is not Pauli), breaking the stabiliser structure and enabling universal but classically hard simulation.

**Answer 12:**

(a) 30 Toffoli × 6 CNOTs = 180 CNOTs. (b) Fidelity = (1−0.004)^180 = 0.996^180 = e^{180·ln(0.996)} = e^{−0.721} ≈ 0.487 ≈ 48.7%. (c) A 30-Toffoli circuit has only ~49% fidelity on current NISQ hardware — the circuit is essentially useless for most algorithms. This demonstrates why only very shallow circuits (depth < ~50) are reliable on today's NISQ hardware, and why fault-tolerant error correction is necessary for meaningful computation.

**Answer 13:**

Global phase e^{iα}: multiplies the entire state vector by a complex number of magnitude 1. Unobservable because: P(outcome m) = |⟨m|e^{iα}|ψ⟩|² = |e^{iα}|²|⟨m|ψ⟩|² = |⟨m|ψ⟩|² — the global phase cancels. Relative phase: phase difference between components of a superposition, e.g., α = 1/√2 and β = ±1/√2 giving |+⟩ vs |−⟩. Example: H|+⟩ = |0⟩ (relative phase 0) but H|−⟩ = |1⟩ (relative phase π). The relative phase determines the measurement outcome after the Hadamard.

**Answer 14:**

CZ gate: applies phase −1 to |11⟩ only: CZ = diag(1,1,1,−1). Symmetric: CZ = (I⊗H)·CNOT·(I⊗H). Proof: (I⊗H)·CNOT·(I⊗H)|ab⟩: H changes |b⟩ to ±|0⟩,±|1⟩ → CNOT acts → H restores. For |11⟩: H|1⟩=|−⟩; CNOT flips to |−⟩; H|−⟩=|1⟩ with phase −1. Result: CZ|11⟩ = −|11⟩ ✓. CZ preferred on Google Sycamore and some other platforms because it is symmetric (no control/target distinction), reducing the need for qubit routing.

**Answer 15:**

SWAP: exchanges qubit states, SWAP|ab⟩=|ba⟩. Minimum 3 CNOTs needed (provably optimal): SWAP = CNOT(0,1)·CNOT(1,0)·CNOT(0,1). SWAP routing is necessary because not all logical qubit pairs are physically adjacent on the coupling map. When a two-qubit gate requires non-adjacent qubits, SWAP gates must move qubit states to adjacent positions. Cost: each SWAP adds 3 CNOTs to circuit depth, tripling the two-qubit gate count for remote operations — the dominant source of circuit overhead on real hardware.

## EXERCISES — CHAPTER 3

### A. Solved Problems

<div class="box box-equation">
<p><strong>Solved Problem 3.1</strong></p>
<p>Problem: Verify that the CNOT gate is unitary. Show CNOT† = CNOT (it is self-inverse, i.e., CNOT² = I).</p>
<p><strong>Solution:</strong></p>
<p>CNOT = [1 0 0 0; 0 1 0 0; 0 0 0 1; 0 0 1 0] (rows/columns in basis |00⟩,|01⟩,|10⟩,|11⟩)</p>
<p>Unitarity: CNOT† = (CNOT*)ᵀ. Since CNOT has all real entries, CNOT† = CNOTᵀ.</p>
<p>The matrix is symmetric (CNOTᵀ = CNOT), so CNOT† = CNOT.</p>
<p>Compute CNOT²:</p>
<img class="eq-block" src="content/images/eq-image56.png" alt="equation">
<p>✓</p>
<p>(Row 3: [0,0,0,1]×CNOT = [0,0,0,1]×([row 3 col j]) = row 4 = [0,0,0,1] then (row4 ×CNOT) = row 3... explicitly: row 3 of CNOT² = (row 3 of CNOT)×CNOT = [0,0,0,1]×CNOT = row 4 of CNOT = [0,0,1,0]. So (CNOT²)₃₃=1 ✓)</p>
<p>Conclusion: CNOT is real, symmetric, and its own inverse. It is a real symmetric involution.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 3.2</strong></p>
<p>Problem: Express the CZ gate in terms of CNOT and single-qubit gates. Prove that CZ is symmetric (either qubit can be the "control").</p>
<p><strong>Solution — CZ from CNOT:</strong></p>
<p>CZ = (I⊗H)·CNOT·(I⊗H)</p>
<p>Proof: Apply (I⊗H) first: this converts the target qubit from Z basis to X basis.</p>
<p>Then CNOT applies X (in Z basis) = Z (in X basis, after conjugation) to the target.</p>
<p>Final (I⊗H) converts back.</p>
<p><strong>Verify on |11⟩:</strong></p>
<p>Step 1: (I⊗H)|11⟩ = |1⟩ ⊗ H|1⟩ = |1⟩ ⊗ |−⟩ = |1⟩(|0⟩−|1⟩)/√2</p>
<p>Step 2: CNOT|1,(|0⟩−|1⟩)/√2⟩ = |1,(|1⟩−|0⟩)/√2⟩ = −|1⟩(|0⟩−|1⟩)/√2</p>
<p>Step 3: (I⊗H) × (−|1⟩(|0⟩−|1⟩)/√2) = −|1⟩H(|0⟩−|1⟩)/√2 = −|1⟩|1⟩ = −|11⟩</p>
<p>This matches CZ|11⟩ = −|11⟩. ✓</p>
<p><strong>Symmetry proof:</strong></p>
<p>CZ matrix = diag(1,1,1,−1). The matrix is diagonal and symmetric, so it commutes with swapping the qubit order: Swap·CZ·Swap = CZ. ✓</p>
<p>CZ acts symmetrically on both qubits: it only cares whether BOTH are |1⟩.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 3.3</strong></p>
<p>Problem: Show that the SWAP gate can be decomposed as SWAP = CNOT₁₂·CNOT₂₁·CNOT₁₂. Verify on the basis state |10⟩.</p>
<p><strong>Solution:</strong></p>
<p>Goal: Show that this three-CNOT sequence implements SWAP on all basis states.</p>
<p><strong>Track |10⟩ through the circuit:</strong></p>
<p>Start: |10⟩ (q1=|1⟩, q2=|0⟩)</p>
<p>After CNOT₁₂ (control=q1, target=q2): |1, 0⊕1⟩ = |11⟩</p>
<p>After CNOT₂₁ (control=q2, target=q1): |1⊕1, 1⟩ = |01⟩</p>
<p>After CNOT₁₂ (control=q1, target=q2): |0, 1⊕0⟩ = |01⟩</p>
<p>Result: |10⟩ → |01⟩ = SWAP|10⟩. ✓</p>
<p>Physical cost: 3 CNOT gates. This is optimal — no circuit using fewer than 3 CNOTs can implement SWAP (proven by counting entangling power).</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 3.4</strong></p>
<p>Problem: Compute the matrix representation of the Toffoli gate explicitly in the 8×8 basis {|000⟩,...,|111⟩}. Verify it is unitary.</p>
<p><strong>Solution:</strong></p>
<p>The Toffoli gate flips the target (q₃) only when both controls (q₁,q₂) are |1⟩. So only the |110⟩ and |111⟩ rows are swapped:</p>
<p>Toffoli = 8×8 identity matrix with rows/columns for |110⟩ and |111⟩ swapped:</p>
<p>Unitarity: The matrix is a permutation matrix (exactly one 1 per row and column). All permutation matrices are unitary: U†U = UᵀU = I (since each column has exactly one non-zero entry). det(Toffoli) = +1 (even permutation). ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 3.5</strong></p>
<p>Problem: A quantum circuit uses 50 Toffoli gates. How many CNOT gates does this require in the standard decomposition? If each CNOT has a 0.3% error rate, what is the circuit fidelity?</p>
<p><strong>Solution:</strong></p>
<p>Standard Toffoli decomposition: 6 CNOTs + 7 single-qubit gates</p>
<p>Total CNOTs from 50 Toffolis: 50 × 6 = 300 CNOT gates</p>
<p><strong>Circuit fidelity:</strong></p>
<p>F = (1 − p)^k = (1 − 0.003)^300 = (0.997)^300</p>
<p>ln(0.997) ≈ −0.003004</p>
<p>ln(F) = 300 × (−0.003004) = −0.9013</p>
<p>F = e^(−0.9013) ≈ 0.406 ≈ 40.6%</p>
<p>Conclusion: A circuit with 50 Toffoli gates has only about 40.6% fidelity on current NISQ hardware with 0.3% two-qubit gate error rates. This is a severe limitation. Fault-tolerant quantum computation, which requires 0.1% logical gate error rates, would be needed for reliable execution.</p>
</div>

<div class="box box-equation">
<p><strong>Solved Problem 3.6</strong></p>
<p>Problem: Using the ZYZ decomposition theorem, find the ZYZ decomposition of the Hadamard gate H. Express as e^(iα) Rz(β) Ry(γ) Rz(δ).</p>
<p><strong>Solution:</strong></p>
<p>We need α, β, γ, δ such that e^(iα) Rz(β) Ry(γ) Rz(δ) = H = (1/√2)[1 1; 1 −1]</p>
<p>Key insight: H = Ry(π/2) × Rz(π) × (global phase):</p>
<p>Rz(π) = [e^(-iπ/2)  0; 0  e^(iπ/2)] = [−i  0; 0  i] = −i × [1 0; 0 −1] = −iZ</p>
<p>Ry(π/2) = [1/√2  −1/√2; 1/√2  1/√2]</p>
<img class="eq-block" src="content/images/eq-image57.png" alt="equation">
<p>So Ry(π/2) × Rz(π) = −i × H (global phase −i = e^(-iπ/2)).</p>
<p>Therefore: H = e^(iπ/2) × Rz(0) × Ry(π/2) × Rz(π)</p>
<p>→ α = π/2, β = 0, γ = π/2, δ = π</p>
<p>Verify: This is consistent with the IBM gate U(π/2, 0, π) = H, since Euler angles and U parameters are related by (β+δ)/2 = φ+λ/2 and γ = θ.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 3.7</strong></p>
<p>Problem: Prove that the single-qubit Clifford group has exactly 24 elements. List them.</p>
<p><strong>Solution — Counting argument:</strong></p>
<p>The Clifford group 𝒞₁ acts on the Pauli group 𝒫₁ = {X, Y, Z} (ignoring phases). This action defines a permutation of the 3 Paulis, so 𝒞₁/U(1) ⊆ Sym({X,Y,Z}) × sign. However, not all permutations preserve the Pauli commutation relations.</p>
<p>The group of sign-preserving permutations of (X,Y,Z) that preserve XY = iZ is exactly the rotation group of the octahedron, which has 24 elements (the chiral octahedral group O ≅ S₄).</p>
<p><strong>Explicit generators from {H, S, I}:</strong></p>
<p>• I (identity): order 1</p>
<p>• X, Y, Z (Paulis): each order 2; total 3 elements</p>
<p>• H: maps X↔Z, fixes Y (with sign change): order 2</p>
<p>• S: maps X→Y, Y→−X, fixes Z: order 4 (S⁴=I)</p>
<p>• SH: order 3; (SH)³ = I</p>
<p>• HS: order 4; combinations of H and S generate 24 elements</p>
<p>The 24 elements can be parameterised as: {I, X, Y, Z, H, SH, HS, XH, YH, ZH, HX, HY, HZ, SXH, ...} — they correspond to the 24 rotations that map the 6 faces of a cube to themselves.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 3.8</strong></p>
<p>Problem: Construct a circuit that prepares the 3-qubit GHZ state |GHZ⟩ = (|000⟩ + |111⟩)/√2 using H and CNOT gates only. Verify by computing the output state.</p>
<p><strong>Solution — Circuit:</strong></p>
<p>1. Apply H to q₀: |000⟩ → (|0⟩+|1⟩)/√2 ⊗ |00⟩ = (|000⟩+|100⟩)/√2</p>
<p>2. Apply CNOT(q₀→q₁): (|000⟩+|100⟩)/√2 → (|000⟩+|110⟩)/√2</p>
<p>3. Apply CNOT(q₀→q₂): (|000⟩+|110⟩)/√2 → (|000⟩+|111⟩)/√2 = |GHZ⟩ ✓</p>
<p>The GHZ state is a maximally entangled 3-qubit state. Measuring any one qubit in the Z basis collapses the others to the same state — a genuine 3-qubit entanglement that cannot be decomposed into bipartite entanglement.</p>
</div>

### B. Unsolved Problems

Solve the following problems independently. Answers are provided in brackets for self-checking.

1. Compute Y² and verify Y² = I. Then verify the commutation relation YX = −XY. [Answer: Y² = [0 −i][i 0]^T = I ✓; YX = [0 −i;i 0]×[0 1;1 0] = [−i 0;0 i] = −iZ; XY = iZ; so YX = −XY ✓]

2. Express the S gate in terms of T gates. How many T gates equal one S gate? How many equal one Z gate? [Answer: S = T², so 2 T gates = 1 S gate; Z = S² = T⁴, so 4 T gates = 1 Z gate]

3. A qubit begins in |ψ⟩ = (√3/2)|0⟩ + (1/2)|1⟩. Apply the sequence H, S, H. Find the final state. [Answer: H|ψ⟩ = ((√3+1)/2√2)|0⟩ + ((√3−1)/2√2)|1⟩; S adds phase to |1⟩ component; final H gives back a specific superposition — compute step by step to get ((√3+1)|0⟩ + i(√3−1)|1⟩)/(2√2)]

4. Verify the conjugation relation SXS† = Y by matrix multiplication. [Answer: SXS† = [1 0;0 i][0 1;1 0][1 0;0 −i] = [0 1;i 0][1 0;0 −i] = [0 −i;i 0] = Y ✓]

5. Show that the 4 Bell states form an orthonormal basis for ℂ² ⊗ ℂ². [Answer: Compute all 4 inner products ⟨Φ±|Ψ±⟩ etc.; each pair is orthogonal; each is normalised; 4 orthonormal vectors in ℂ⁴ span it. Hint: write all 4 Bell states explicitly and take pairwise inner products to confirm ⟨B\_i|B\_j⟩ = δ\_{ij}]

6. The Hadamard gate can be written as H = (X+Z)/√2. Verify this identity by substituting the Pauli matrices. [Answer: (X+Z)/√2 = ([0 1;1 0]+[1 0;0 −1])/√2 = [1 1;1 −1]/√2 = H ✓]

7. How many distinct states can be reached from |0⟩ by applying the 24 single-qubit Clifford group elements? [Answer: The 24 Cliffords map |0⟩ to the 6 stabiliser states {|0⟩, |1⟩, |+⟩, |−⟩, |i⟩, |−i⟩} (the faces of the octahedron). Each state is reached by 24/6 = 4 different Clifford elements]

8. Compute Rz(π/4)·Rz(π/4) and verify it equals the T gate (up to global phase). [Answer: Rz(π/4) = [e^(-iπ/8) 0; 0 e^(iπ/8)]; Rz(π/4)² = [e^(-iπ/4) 0; 0 e^(iπ/4)] = e^(-iπ/4)[1 0; 0 e^(iπ/2)] = e^(-iπ/4)×T; so T = e^(iπ/4)Rz(π/4)² ✓]

9. A circuit uses 20 H gates, 10 S gates, 15 CNOT gates, and 8 T gates. (a) Is this a Clifford circuit? (b) Can it be efficiently simulated classically? (c) What is the T-count? [Answer: (a) No — T gates are non-Clifford; (b) No — not classically efficiently simulable (though heuristic methods may work); (c) T-count = 8]

10. Using the Solovay-Kitaev theorem with c=3.97, estimate the number of {H,T} gates needed to approximate Ry(0.001) to precision ε = 10⁻⁶. [Answer: k ≤ O(log^{3.97}(10⁶)); log₂(10⁶) ≈ 19.93; 19.93^3.97 ≈ 19.93⁴ ≈ 158,000 gates. In practice, optimised methods give ~3×6 = 18 T gates for this precision]

### C. Multiple Choice Questions

Note: Answers to all MCQs are given at the end of this section.

**Q1. The Pauli-Y gate applied to |1⟩ gives:**

(a)  |0⟩

(b)  i|0⟩

(c)  −i|0⟩

(d)  −|0⟩

**Q2. Which of the following gates is NOT in the Clifford group?**

(a)  Hadamard H

(b)  T gate

(c)  CNOT

(d)  Pauli-Z

**Q3. The Hadamard gate satisfies H² = I. What does this imply about its eigenvalues?**

(a)  Both eigenvalues are +1

(b)  Both eigenvalues are −1

(c)  Eigenvalues are +1 and −1

(d)  Eigenvalues are +i and −i

**Q4. The CNOT gate can be used to create entanglement because:**

(a)  It is a two-qubit gate

(b)  It can map product states to entangled states

(c)  It has determinant −1

(d)  It has non-zero trace

**Q5. Which statement about the T gate is CORRECT?**

(a)  T is a Clifford gate

(b)  T² = Z

(c)  T⁸ = I

(d)  T and S† together generate a universal gate set

**Q6. The Gottesman-Knill theorem states that Clifford circuits can be:**

(a)  Simulated classically in exponential time

(b)  Simulated classically in polynomial time (O(n²))

(c)  Not simulated classically

(d)  Simulated in O(n log n) time

**Q7. How many CNOT gates are needed to implement a SWAP gate?**

(a)  1

(b)  2

(c)  3

(d)  4

**Q8. The ZYZ decomposition U = e^(iα) Rz(β) Ry(γ) Rz(δ) requires how many real parameters to specify any single-qubit unitary (ignoring global phase)?**

(a)  2

(b)  3

(c)  4

(d)  Infinitely many

**Q9. The Solovay-Kitaev theorem guarantees approximation of any unitary to precision ε using at most O(polylog(1/ε)) gates. What constant c satisfies this bound?**

(a)  c = 1 (linear in log(1/ε))

(b)  c ≈ 2

(c)  c ≈ 3.97

(d)  c is not bounded (it depends on the target gate)

**Q10. Which two-qubit gate has the property that CZ|11⟩ = −|11⟩ and CZ|xy⟩ = |xy⟩ for all other basis states?**

(a)  CNOT

(b)  Controlled-Z (CZ)

(c)  SWAP

(d)  Toffoli

**Q11. The Toffoli gate is universal for which class of computation?**

(a)  Universal quantum computation alone

(b)  Classical reversible computation (with NOT gate)

(c)  Only linear Boolean operations

(d)  Clifford group operations only

**Q12. The Bell state |Ψ⁺⟩ = (|01⟩ + |10⟩)/√2 is produced from which input by the H+CNOT circuit?**

(a)  |00⟩

(b)  |01⟩

(c)  |10⟩

(d)  |11⟩

**Q13. Which of the following is TRUE about global vs relative phases?**

(a)  Both are physically observable

(b)  Global phase is observable; relative phase is not

(c)  Relative phase is observable; global phase is not

(d)  Neither is physically observable

**Q14. The SWAP test (using CSWAP gate) allows us to estimate:**

(a)  The fidelity of a quantum channel

(b)  The overlap |⟨φ|ψ⟩|² between two unknown quantum states

(c)  The purity of a mixed state

(d)  The T-count of a circuit

**Q15. For the gate set {H, T, CNOT}, the T gate is special because:**

(a)  It is the only gate not in U(2)

(b)  Without T, the set cannot generate non-Clifford operations needed for universality

(c)  T is the cheapest gate in fault-tolerant quantum computing

(d)  T commutes with all Pauli gates

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 3</strong></p>
<p>Q1: (c) −i|0⟩ — Y = [0 −i; i 0]; Y|1⟩ = [−i; 0] = −i|0⟩</p>
<p>Q2: (b) T gate — T is a non-Clifford gate; it does not map all Paulis to Paulis under conjugation</p>
<p>Q3: (c) Eigenvalues +1 and −1 — Any involution (A² = I) has eigenvalues ±1. H has eigenvalues +1 (eigenvector |+⟩ rotated) and −1</p>
<p>Q4: (b) It can map product states to entangled states — the H+CNOT sequence maps |00⟩ (product) to |Φ⁺⟩ (entangled)</p>
<p>Q5: (c) T⁸ = I — T = diag(1, e^(iπ/4)); T⁸ = diag(1, e^(i2π)) = diag(1,1) = I. Note: T² = S (not Z), S² = Z, T⁴ = Z; (b) is WRONG: T² = S ≠ Z</p>
<p>Q6: (b) Simulated classically in polynomial time O(n²) — via the stabiliser tableau representation; O(n²) bits per gate step</p>
<p>Q7: (c) 3 — SWAP = CNOT₁₂·CNOT₂₁·CNOT₁₂ (three CNOTs, provably optimal)</p>
<p>Q8: (b) 3 — SU(2) is a 3-dimensional Lie group. Three real angles (β, γ, δ) specify any U up to global phase; adding α gives 4 for full U(2)</p>
<p>Q9: (c) c ≈ 3.97 — The Solovay-Kitaev theorem with current best known c ≈ 3.97 (though recent work has improved to c &lt; 2 with different methods)</p>
<p>Q10: (b) Controlled-Z (CZ) — CZ = diag(1,1,1,−1) in {|00⟩,|01⟩,|10⟩,|11⟩} basis; only |11⟩ gets phase −1</p>
<p>Q11: (b) Classical reversible computation (with NOT gate) — {Toffoli, NOT} is universal for reversible classical Boolean computation. For universal quantum computation one also needs H.</p>
<p>Q12: (b) |01⟩ — H|0⟩⊗|1⟩ → (|0⟩+|1⟩)/√2⊗|1⟩ → CNOT → (|01⟩+|10⟩)/√2 = |Ψ⁺⟩</p>
<p>Q13: (c) Relative phase is observable; global phase is not — global phase e^(iα)|ψ⟩ leaves all measurement probabilities unchanged; relative phase changes interference and is detectable by measuring in different bases</p>
<p>Q14: (b) The overlap |⟨φ|ψ⟩|² — SWAP test gives P(0) = (1 + |⟨φ|ψ⟩|²)/2, so |⟨φ|ψ⟩|² = 2P(0) − 1</p>
<p>Q15: (b) Without T, the set cannot generate non-Clifford operations needed for universality — {H, CNOT} without T generates only the Clifford group, which is classically simulable and not computationally universal</p>
</div>

### D. Theory Questions

- State and prove the identity HXH = Z. Use your result to explain why conjugation by H transforms between the X and Z eigenbases. What is the physical significance of this transformation for quantum circuit design?

- Explain the distinction between a relative phase and a global phase in quantum mechanics. Give a circuit example where (a) only the global phase changes (unobservable) and (b) the relative phase changes (observable). Include the measurement that would distinguish case (b).

- Explain in detail why the Toffoli gate makes quantum computers at least as powerful as classical computers. What is Landauer's principle, and how does the Toffoli gate relate to it? Why does reversible computing matter for quantum hardware?

- State the Gottesman-Knill theorem precisely. What is the "stabiliser formalism"? Explain why the stabiliser of the Bell state |Φ⁺⟩ is generated by {X⊗X, Z⊗Z}, and show that these Pauli operators indeed stabilise the Bell state.

- Derive the ZYZ Euler decomposition of an arbitrary single-qubit unitary U ∈ SU(2). Why are exactly three angles needed? What is the relationship between ZYZ angles and the IBM U(θ,φ,λ) gate parameters?

- Explain why the T-count is the most important complexity metric in fault-tolerant quantum computing. What is magic state distillation? How does the overhead of T gate implementation compare to that of Clifford gates in a surface code implementation?

- Prove that the 4 Bell states form an orthonormal basis for ℂ² ⊗ ℂ². Show explicitly that the H+CNOT circuit maps the computational basis to the Bell basis via a unitary transformation. What is the matrix of this unitary in the computational basis?

- Compare the CZ gate and the CNOT gate: (a) matrix representations, (b) which qubit is the "control," (c) how to convert one to the other, (d) which platforms use each as a native 2-qubit gate, and (e) error rate considerations on current hardware.

- State the Solovay-Kitaev theorem and explain its practical implications for quantum circuit compilation. If we want to approximate every gate in an n-gate circuit to precision ε/n (so the total circuit error is at most ε), how does the total gate count scale with n and ε?

- Explain the role of the Hadamard gate in the following contexts: (a) creating the Fourier transform basis, (b) the Deutsch algorithm, (c) phase kickback, and (d) the HXH=Z conjugation identity used in circuit simplification. Why is H described as the "backbone" of quantum algorithms?

### E. Programming Assignments

PA3.1. [Qiskit — Gate Verification]  Write a Qiskit program that verifies all 12 Pauli gate identities: X²=Y²=Z²=I, XY=iZ, YZ=iX, ZX=iY, and their reverses. For each identity, (a) compute the left-hand side using Qiskit's Operator class, (b) compute the right-hand side, and (c) check equality with Operator.equiv(). Print a verification table. Submit your code and output.

PA3.2. [Qiskit — Bloch Sphere Study]  Create a Qiskit program that: (a) starts with |0⟩, (b) applies each of the 24 single-qubit Clifford group elements in sequence, (c) plots the Bloch sphere at each step showing the state evolution, and (d) identifies which of the 6 stabiliser states {|0⟩, |1⟩, |+⟩, |−⟩, |i⟩, |−i⟩} each Clifford maps |0⟩ to. Create an animated or multi-panel figure. Submit code, figure, and a table of Clifford → output state mappings.

PA3.3. [Qiskit — Toffoli Verification and Quantum AND]  (a) Verify the Toffoli gate decomposition (6 CNOTs + 7 single-qubit gates) by running both the direct Toffoli circuit and its decomposition on the AerSimulator for all 8 input states |000⟩ through |111⟩, confirming identical outputs. (b) Build a 3-qubit quantum adder for 1-bit numbers using Toffoli (for carry) and CNOT (for XOR), implement it in Qiskit, and verify it adds 0+0, 0+1, 1+0, 1+1 correctly. Submit complete code and verification output.

### F. Project Suggestions

Project 3.A — Universal Gate Set Analysis:  Implement the Solovay-Kitaev algorithm (or use Qiskit's SK\_Algorithm if available) to approximate five specific rotation gates — Rz(π/5), Rz(π/7), Rz(π/11), Rx(π/3), Ry(2π/9) — to precision ε = 0.001 and ε = 0.0001 using the {H, T} gate set. For each: (a) find the approximating sequence, (b) verify the approximation error, (c) count the T gates used, (d) compare with the Solovay-Kitaev bound. Write a 2000-word report analysing the trade-off between gate count and precision.

Project 3.B — Stabiliser State Classification:  Write a Python simulation of the stabiliser formalism using the Tableau representation. Implement: (a) encoding of n-qubit stabiliser states as 2n×(2n+1) binary tableau, (b) Clifford gate updates (H, S, CNOT) on the tableau, (c) measurement in the Z basis with tableau update, (d) simulation of a 10-qubit random Clifford circuit of depth 50. Compare runtime with the Statevector simulator. Write a report on the efficiency gain and its theoretical basis (Gottesman-Knill theorem). Submit code and performance analysis.

Project 3.C — Bell Inequality Violation on IBM Hardware:  Prepare the Bell state |Φ⁺⟩ = (|00⟩+|11⟩)/√2 on an actual IBM Quantum processor. Measure in four different bases to compute the CHSH value S = E(a,b) − E(a,b') + E(a',b) + E(a',b'). Classically S ≤ 2; quantum mechanics predicts S ≤ 2√2 ≈ 2.828. Report: (a) circuit implementation in Qiskit, (b) raw counts from hardware, (c) CHSH S value obtained, (d) comparison with simulator, (e) effect of noise and how to mitigate it. Minimum 2500 words with figures.

## References and Further Reading

1. Nielsen, M. A. & Chuang, I. L. (2000). Quantum Computation and Quantum Information. Cambridge University Press. [Chapters 1, 4 — the standard reference for all material in this chapter]

2. Kitaev, A. Y., Shen, A. H., & Vyalyi, M. N. (2002). Classical and Quantum Computation. AMS. [Solovay-Kitaev theorem; stabiliser formalism]

3. Gottesman, D. (1997). Stabilizer Codes and Quantum Error Correction. PhD Thesis, Caltech. [Stabiliser formalism; Gottesman-Knill theorem]

4. Landauer, R. (1961). Irreversibility and heat generation in the computing process. IBM Journal of Research and Development, 5(3), 183–191. [Landauer's principle; reversible computation]

5. Toffoli, T. (1980). Reversible Computing. ICALP Proceedings, Springer. [Toffoli gate and reversible classical computing]

6. Qiskit Documentation (2024). quantum.ibm.com. [U gate, native gate sets, Qiskit 1.x API]

7. Harrow, A. W. (2002). Solovay-Kitaev Theorem. Course Notes, MIT. [Constructive proof and complexity analysis]
