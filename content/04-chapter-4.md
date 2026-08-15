# CHAPTER 4: Quantum Circuits

# Design, Transpilation, and Programming Frameworks

<div class="box box-anecdote">
<p class="box-title"><strong>📜  Richard Feynman's Demand for a New Programming Language — 1985</strong></p>
<p>When David Deutsch published his universal quantum computer model in 1985, Richard Feynman read it with characteristic impatience. He immediately saw a problem that theorists had glossed over: even if one could build such a machine, how would one program it? Classical programming languages — FORTRAN, Pascal, BASIC — described operations on bits that could be copied, read non-destructively, and tested freely. Quantum states could do none of these things. Feynman scribbled in a letter to a colleague: "The real question is not whether nature is quantum mechanical — of course it is. The question is what language you use to talk to it."</p>
<p>That question — what language do you use to talk to a quantum computer? — is what this chapter answers. Over four decades, the answer evolved from abstract mathematical circuit diagrams to practical programming frameworks used by tens of thousands of researchers worldwide. Qiskit, Cirq, Q#, PennyLane, TKET, QuTiP: each represents a different philosophy about what quantum programming should look like and what it should be optimised for. Underlying all of them is the concept of the quantum circuit — the formal structure that connects mathematical gate theory (Chapter 3) to physical hardware execution. Understanding how circuits are designed, simplified, and compiled is the bridge between quantum theory and quantum practice.</p>
</div>

Chapter 3 gave us the vocabulary of quantum gates — the elementary operations that transform qubits. This chapter gives us grammar: the rules and techniques for assembling gates into circuits that are correct, efficient, and executable on real hardware. The journey from a mathematical circuit description to a sequence of microwave pulses on a superconducting qubit involves several transformations — collectively called transpilation — each of which introduces complexity and must be understood if one is to write quantum programs that actually run well.

We then survey the major quantum programming frameworks, understanding not just what they do but why each was designed the way it was, what trade-offs each makes, and when to choose each one. The ability to navigate the quantum software ecosystem is a practical skill that will serve students throughout their careers in quantum computing.

## 4.1 Quantum Circuit Design

A quantum circuit is a sequence of quantum gates applied to a set of qubits, followed by measurements. The circuit model is the dominant computational model for quantum computing — analogous to the program in classical computing. Designing a good quantum circuit means achieving the desired computation with minimal resource use: as few gates as possible, as few qubits as possible, and as shallow a depth as possible. These three objectives often conflict, and circuit design is the art of navigating that trade-off.

### 4.1.1 Circuit Depth, Width, and T-Count

#### Circuit Depth: The Critical Path

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Circuit Depth</strong></p>
<p>The depth of a quantum circuit is the number of layers (time steps) needed to execute it, where a layer consists of all gates that can be executed simultaneously (i.e., gates acting on disjoint qubits). Formally:</p>
<p>A circuit with depth d requires d time steps to execute. On NISQ hardware, this matters critically: decoherence times (T₁, T₂) limit how many time steps can complete before the quantum information is lost. Deep circuits fail on NISQ hardware. Shallow circuits are more robust.</p>
</div>

To compute depth, think of the circuit as a directed acyclic graph (DAG): each gate is a node, and there is an edge from gate A to gate B if A's output qubit is B's input qubit. The depth equals the length of the longest path through this DAG. Equivalently, arrange all gates into the minimum number of columns (layers) such that no two gates in the same column share a qubit — the number of columns is the depth.

#### Circuit Width: Qubit Count

The width of a circuit is simply the number of qubits it uses, including ancilla (helper) qubits. Width is a hard resource constraint: a 100-qubit device cannot run a 101-qubit circuit. Moreover, even circuits with fewer qubits than the device may fail if the physical qubit connectivity forces SWAP routing (Section 4.2) that expands the effective width.

Width and depth trade off: many algorithms can be "unrolled" to use fewer qubits at the cost of deeper circuits (more sequential steps), or parallelised to use more qubits for shallower depth. This space-time trade-off is a constant theme in quantum algorithm design.

#### T-Count: The Fault-Tolerant Cost Metric

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: T-Count</strong></p>
<p>The T-count of a circuit is the number of T gates (and T† gates) in its decomposition into the {H, S, CNOT, T} gate set. T-count is the primary resource metric for fault-tolerant quantum computing because:</p>
<p>Clifford gates {H, S, CNOT} can be implemented transversally in most error-correcting codes — they require O(n) physical operations per logical gate</p>
<p>T gates require magic state distillation — a costly process requiring O(n log n) physical operations per logical T gate (for surface codes)</p>
<p>Magic state distillation factory overhead: each logical T gate requires distilling ~15 noisy physical T states into one clean logical T state (Reed-Muller 15-1 protocol)</p>
<p>Therefore: T-count determines the dominant cost of fault-tolerant algorithms</p>
</div>

<figure class="book-figure">
<img src="content/images/image58.png" alt="Figure 4.1: The three primary circuit complexity metrics. Left: Circuit depth — the number of parallel layers (here, depth=6). Centre: Circuit width — the number of qubits (here, width=5). Right: T-count — the number of non-Clifford T/T† gates (here, T-count=6, highlighted in orange).">
<figcaption>Figure 4.1: The three primary circuit complexity metrics. Left: Circuit depth — the number of parallel layers (here, depth=6). Centre: Circuit width — the number of qubits (here, width=5). Right: T-count — the number of non-Clifford T/T† gates (here, T-count=6, highlighted in orange).</figcaption>
</figure>

Beyond these three primary metrics, several secondary metrics are also tracked:

- Gate count: total number of gates (including single-qubit). Determines runtime on simulators and approximate error.

- CNOT count: number of two-qubit gates. Each CNOT has ~10-50× higher error rate than single-qubit gates on current hardware.

- Multi-qubit gate set fidelity: on real hardware, each 2-qubit gate has an error rate p (typically 0.1%–1%). Circuit fidelity ≈ (1−p)^k where k is the CNOT count.

- Qubit connectivity overhead: extra gates needed to route qubits to adjacent positions on the coupling map.

<div class="box box-example">
<p class="box-title"><strong>Example 4.1: Calculating Circuit Metrics for a GHZ Circuit</strong></p>
<p>Problem: Consider the 4-qubit GHZ state preparation circuit: H on q₀, then CNOT(q₀→q₁), CNOT(q₁→q₂), CNOT(q₂→q₃). Calculate: (a) circuit depth, (b) circuit width, (c) T-count, (d) CNOT count, (e) circuit fidelity if each CNOT has a 0.5% error rate.</p>
<p><strong>Solution:</strong></p>
<p>(a) Circuit depth: Can H and CNOT(q₀→q₁) be parallelised? No — CNOT needs H to complete first (q₀ shared). Can CNOT(q₀→q₁) and CNOT(q₁→q₂) be parallelised? No — q₁ is shared. So execution is strictly sequential: Depth = 1 + 1 + 1 + 1 = 4.</p>
<p>(b) Circuit width = 4 qubits (q₀, q₁, q₂, q₃). No ancilla used.</p>
<p>(c) T-count = 0. The GHZ circuit uses only Clifford gates {H, CNOT} — no T gates. It is a Clifford circuit, efficiently simulable classically.</p>
<p>(d) CNOT count = 3.</p>
<p>(e) Circuit fidelity: F = (1 − 0.005)³ = (0.995)³ ≈ 0.9851 ≈ 98.5%. Good fidelity — this is a shallow, low-CNOT circuit, appropriate for NISQ hardware.</p>
</div>

### 4.1.2 Circuit Identities and Simplification

Circuit simplification — reducing depth, gate count, or T-count while preserving the computation — is one of the most important practical skills in quantum programming. Just as a compiler optimises classical assembly code, a quantum compiler applies circuit identities to reduce resource use. This section catalogues the most important identities.

#### Self-Inverse Gates (Cancellation Pairs)

Any gate U satisfying U² = I can be cancelled when applied consecutively. These cancellations are the simplest and most frequent optimisations:

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p><strong>X·X = I    Y·Y = I    Z·Z = I    H·H = I    CNOT·CNOT = I</strong></p>
<p>SWAP·SWAP = I    CZ·CZ = I</p>
<p>If a gate appears twice in sequence on the same qubit(s) with no intervening gates,</p>
<p>it can be removed entirely — zero gates, zero depth contribution.</p>
</div>

#### Phase Gate Composition

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>T·T = S        (two T gates combine to one S gate)</p>
<p>T·T·T·T = Z    (four T gates = one Z gate)</p>
<p>S·S = Z        S·Z = S†    T·S = T·T·T†</p>
<p>T·T·T·T·T·T·T·T = I    (T has order 8)</p>
<p>Critical implication: T-count can be reduced by collecting and combining T, T†, S gates.</p>
</div>

#### Conjugation Identities

These identities allow basis changes that enable circuit simplification. When a gate A appears between two conjugating gates B on both sides, it can often be replaced by a simpler gate:

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>H·X·H = Z        H·Z·H = X        H·Y·H = −Y</p>
<p>H·Rx(θ)·H = Rz(θ)    H·Rz(θ)·H = Rx(θ)</p>
<p>CNOT_(c→t)·(X_c ⊗ I)·CNOT_(c→t) = X_c ⊗ X_t</p>
<p>CNOT_(c→t)·(I ⊗ Z_t)·CNOT_(c→t) = Z_c ⊗ Z_t</p>
<p>(H⊗H)·CNOT_(c→t)·(H⊗H) = CNOT_(t→c)    (control and target swap!)</p>
</div>

<figure class="book-figure">
<img src="content/images/image59.png" alt="Figure 4.2: Key circuit identities for optimisation. Top-left: H·H = I (adjacent H gates cancel). Top-right: CNOT·CNOT = I (adjacent CNOTs cancel). Bottom-left: phase gate composition (T²=S, T⁴=Z). Bottom-right: conjugation identities (HXH=Z, HZH=X) enabling basis changes.">
<figcaption>Figure 4.2: Key circuit identities for optimisation. Top-left: H·H = I (adjacent H gates cancel). Top-right: CNOT·CNOT = I (adjacent CNOTs cancel). Bottom-left: phase gate composition (T²=S, T⁴=Z). Bottom-right: conjugation identities (HXH=Z, HZH=X) enabling basis changes.</figcaption>
</figure>

#### Commutation Rules

Two gates that commute can be reordered without changing the circuit output. Commutation enables parallelisation (reducing depth) and may enable cancellation after reordering:

- X commutes with CNOT when X acts on the control qubit: (X\_c ⊗ I)·CNOT = CNOT·(X\_c ⊗ I) — FALSE. Actually X on control ANTICOMMUTES with CNOT in the sense X on control → X on target too. See identities above.

- Z commutes with CNOT when Z acts on the control qubit: CNOT\_(c→t)·Z\_c = Z\_c·CNOT\_(c→t)

- X commutes with CNOT when X acts on the target: CNOT\_(c→t)·X\_t = X\_t·CNOT\_(c→t)

- Rz(θ) commutes with CNOT when applied to the control qubit (both are Z-type gates)

- Rx(θ) commutes with CNOT when applied to the target qubit (both are X-type gates)

<div class="box box-example">
<p class="box-title"><strong>Example 4.2: Simplifying a Circuit with Three Redundant Gates</strong></p>
<p>Problem: Simplify the following sequence on qubit q: H, X, H, H, X, H</p>
<p><strong>Step 1 — Identify pairs:</strong></p>
<p>H, X, H = Z    (by conjugation identity HXH = Z)</p>
<p>Remaining circuit: Z, H, X, H</p>
<p><strong>Step 2 — Apply again:</strong></p>
<p>H, X, H = Z    (again by HXH = Z)</p>
<p>Remaining: Z, Z</p>
<p><strong>Step 3 — Cancel:</strong></p>
<p>Z·Z = I    (Z is self-inverse)</p>
<p>Result: the entire 6-gate sequence H·X·H·H·X·H = I — identity! Total gates: 0.</p>
<p>Original: 6 gates, depth 6. Optimised: 0 gates, depth 0. Savings: 100%.</p>
</div>

#### Practical Optimisation Strategy

Modern quantum compilers (Qiskit, TKET, Cirq) apply these identities automatically using a technique called peephole optimisation: scanning the circuit window by window (typically 2–5 gates at a time) and replacing any window that matches a known simplification pattern with a shorter equivalent. The key steps are:

- Gate cancellation: find and remove pairs of inverse gates

- Rotation merging: combine adjacent Rz(θ₁)·Rz(θ₂) = Rz(θ₁+θ₂) into a single rotation

- Commutation-based reordering: move gates past each other to expose new cancellation opportunities

- Basis decomposition: express gates in the hardware-native basis (e.g., {U, CX} for IBM)

- T-gate optimisation (for fault-tolerant): use T-par algorithm or ZX-calculus rewriting to minimise T-count

### 4.1.3 Ancilla Qubits and Uncomputation

Many quantum algorithms require scratch space — extra qubits used for intermediate computations but not part of the final output. These are called ancilla qubits (from the Latin "ancilla," servant). Managing ancilla qubits correctly is one of the most subtle aspects of quantum circuit design, because quantum mechanics imposes a constraint that has no classical analogue: ancilla qubits that remain entangled with the main register corrupt the computation.

#### Why Ancilla Qubits Are Needed

Classical computation frequently uses temporary variables that are overwritten and discarded. In quantum computing, overwriting destroys information, violating unitarity (all quantum gates are reversible). Instead, quantum algorithms use ancilla qubits in the following pattern:

- Begin with ancilla in a known state (typically |0⟩)

- Perform a computation that involves the ancilla (e.g., compute some function into the ancilla)

- Use the result stored in the ancilla for the main computation

- Uncompute: reverse the computation that involved the ancilla, returning it to |0⟩

- The ancilla is now clean (|0⟩) and can be reused or discarded

#### The Uncomputation Principle

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Uncomputation</strong></p>
<p>Let U be any unitary that computes f(x) using ancilla: U|x⟩|0⟩ = |x⟩|f(x)⟩. After using the result |f(x)⟩, the uncomputation step applies U† (the adjoint of U, which reverses the computation):</p>
<p>The full pattern (Bennett's trick, 1973) is: Compute(U) → Use result → Uncompute(U†). The cost is exactly double the computation (one U and one U†), but the ancilla is returned clean. This doubles the gate count but halves the qubit count compared to keeping all ancilla dirty.</p>
</div>

<figure class="book-figure">
<img src="content/images/image60.png" alt="Figure 4.3: Ancilla qubits and uncomputation. Left: without uncomputation — the ancilla remains entangled with inputs after the Toffoli gate, corrupting subsequent use. Right: with uncomputation (Bennett&#x27;s trick) — the compute → copy → uncompute pattern returns the ancilla to |0⟩, safely reusable.">
<figcaption>Figure 4.3: Ancilla qubits and uncomputation. Left: without uncomputation — the ancilla remains entangled with inputs after the Toffoli gate, corrupting subsequent use. Right: with uncomputation (Bennett&#x27;s trick) — the compute → copy → uncompute pattern returns the ancilla to |0⟩, safely reusable.</figcaption>
</figure>

#### Pebbling and Space-Time Trade-offs

The decision of when and how much to uncompute leads to a formal model called quantum pebbling. Like the classical pebble game (used to analyse space-time trade-offs in computation), quantum pebbling tracks which ancilla are "occupied" (dirty) at each step and asks: what is the minimum space (ancilla count) needed for a given time budget?

The key result: any classical computation using S bits of space and T time steps can be simulated reversibly using O(S log T) ancilla qubits and O(T · log T) time steps. This is the Bennett-Fredkin trade-off — increasing time by a logarithmic factor allows exponential savings in ancilla.

<div class="box box-example">
<p class="box-title"><strong>Example 4.3: Uncomputing a Multi-Controlled Gate</strong></p>
<p>Problem: Compute f(a,b,c) = (a AND b) AND c using ancilla qubits and uncomputation. Draw the circuit and count resources.</p>
<p><strong>Strategy:</strong></p>
<p>Step 1: Compute a AND b into ancilla₁: Toffoli(a, b, ancilla₁=0) → ancilla₁ = a·b</p>
<p>Step 2: Compute (a AND b) AND c into output: Toffoli(ancilla₁, c, output=0) → output = (a·b)·c</p>
<p>Step 3: Uncompute ancilla₁: Toffoli(a, b, ancilla₁) → ancilla₁ = 0 (clean)</p>
<p>Resources: width = 5 qubits (3 input + 1 ancilla + 1 output), depth ≈ 3 Toffoli layers, gate count = 3 Toffoli = 18 CNOTs in standard decomposition. Without uncomputation, we would need 5 qubits and the ancilla would be dirty.</p>
</div>

### 4.1.4 Mid-Circuit Measurement and Dynamic Circuits

In the standard quantum circuit model, measurements occur only at the end of the circuit. But real quantum algorithms — particularly quantum error correction and quantum teleportation — require measuring qubits in the middle of the circuit and using the classical results to control subsequent quantum operations. This is called mid-circuit measurement and classical feedforward, and circuits that use these features are called dynamic circuits.

#### What Is Mid-Circuit Measurement?

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Mid-Circuit Measurement</strong></p>
<p>A mid-circuit measurement collapses a qubit to a definite classical state (0 or 1) while the rest of the circuit continues. The measured qubit can then be:</p>
<p>Reset to |0⟩ and reused (qubit recycling — reduces width)</p>
<p>Discarded</p>
<p>Used classically to control subsequent quantum gates (classical feedforward)</p>
<p>The key enabling technology: modern quantum hardware (IBM Heron, Google Willow) can perform mid-circuit measurements with reset in less than 1 μs, and then apply classically-conditioned gates (If-then in OpenQASM 3) within the decoherence time of remaining qubits.</p>
</div>

<figure class="book-figure">
<img src="content/images/image61.png" alt="Figure 4.4 (displayed): Mid-circuit measurement and classical feedforward. Left: quantum teleportation circuit — Alice measures her two qubits mid-circuit; the classical results (0 or 1) control which correction gates (X, Z) Bob applies. Right: feedforward control flow — classical measurement outcome gates future quantum operations.">
<figcaption>Figure 4.4 (displayed): Mid-circuit measurement and classical feedforward. Left: quantum teleportation circuit — Alice measures her two qubits mid-circuit; the classical results (0 or 1) control which correction gates (X, Z) Bob applies. Right: feedforward control flow — classical measurement outcome gates future quantum operations.</figcaption>
</figure>

#### Applications of Dynamic Circuits

- Quantum error correction: measure syndrome qubits without disturbing data qubits; apply corrections classically conditioned on syndrome outcomes. This is essential for fault-tolerant quantum computing.

- Quantum teleportation: Alice measures two qubits in the Bell basis; classical results determine Bob's correction (X and/or Z gates). Without feedforward, teleportation is incomplete.

- Qubit recycling: measure and reset ancilla qubits mid-circuit, reusing physical qubits to simulate wider circuits on narrower hardware. A 50-qubit circuit can sometimes run on 20 qubits by recycling.

- Adaptive algorithms: variational algorithms (VQE, QAOA) where circuit parameters are updated based on intermediate measurement outcomes.

- Repeat-Until-Success (RUS) circuits: try a probabilistic gate repeatedly, measuring to check success, and applying corrections if needed. Efficient non-Clifford gate synthesis.

#### Dynamic Circuits in Qiskit and OpenQASM 3

OpenQASM 3 (Quantum Assembly Language, version 3) added native support for dynamic circuits through if-else statements and real-time classical control. Qiskit's dynamic circuit model translates these to hardware-executable instructions:

```python
# ─────────────────────────────────────────────────────────────────────
# Code 4.1: Dynamic Circuit — Mid-Circuit Measurement and Feedforward
# Implements the correction step of quantum teleportation
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister

# Quantum teleportation of an unknown state |ψ⟩ on q0
# Alice holds q0 (state to teleport) and q1 (her entangled qubit)
# Bob holds q2 (his entangled qubit)
q = QuantumRegister(3, "q")
c = ClassicalRegister(2, "c")          # classical bits for measurement
qc = QuantumCircuit(q, c)

# ── Prepare |ψ⟩ = Ry(π/3)|0⟩ on q0 (unknown state to teleport)
qc.ry(3.14159/3, q[0])

# ── Create Bell pair between Alice (q1) and Bob (q2)
qc.h(q[1])
qc.cx(q[1], q[2])

# ── Alice's Bell measurement (mid-circuit measurement)
qc.cx(q[0], q[1])     # entangle state with Alice's qubit
qc.h(q[0])            # rotate to Bell basis
qc.measure(q[0], c[0])   # mid-circuit measurement — bit 0
qc.measure(q[1], c[1])   # mid-circuit measurement — bit 1

# ── Classical feedforward: Bob applies corrections
# OpenQASM 3 / Qiskit dynamic circuits syntax:
with qc.if_test((c[1], 1)):  # if Alice's second bit is 1:
    qc.x(q[2])              #   Bob applies X correction
with qc.if_test((c[0], 1)):  # if Alice's first bit is 1:
    qc.z(q[2])              #   Bob applies Z correction

# Now q2 holds the teleported state |ψ⟩ (up to global phase)
# Verify: measure q2 in multiple bases and compare with q0 statistics
print(qc.draw())  # draw the dynamic circuit
```

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: Mid-Circuit Measurement Is NOT Always Available</strong></p>
<p>Not all quantum backends support mid-circuit measurement and classical feedforward. Many older IBM backends (ibmq_lima, ibmq_belem) support only final measurement. Always check backend.properties() for "supports_dynamic_reprate" and "supports_classical_control" flags. If mid-circuit measurement is unsupported, deferred measurement (postponing all measurements to the end using the "deferred measurement principle") must be used — but this requires additional ancilla qubits to store intermediate results. The IBM Heron (r2) processor and all IBM Quantum Falcon R8 or later processors support dynamic circuits natively.</p>
</div>

## 4.2 Transpilation: Logical to Physical Circuits

The gap between a quantum algorithm as written in Qiskit (logical circuit) and the microwave pulses that execute on IBM quantum hardware (physical circuit) is bridged by transpilation — a multi-stage compilation process that transforms, optimises, and maps the circuit to the specific constraints of the target hardware. Understanding transpilation is essential for anyone who wants their programs to actually run efficiently on quantum hardware.

### 4.2.1 The Transpilation Problem

#### The Mismatch Between Logical and Physical Circuits

A logical quantum circuit makes no assumptions about hardware. It might use any gate set, connect any two qubits, and have arbitrary depth. A physical quantum processor has three hard constraints that the logical circuit typically violates:

- Gate set constraint: the physical device implements only a specific native gate set (e.g., IBM uses {CX, U} or {ECR, U} depending on processor model). Any gate not in this set must be decomposed.

- Connectivity constraint: two-qubit gates (CNOT, CZ) can only be applied to physically adjacent qubits as defined by the coupling map. A CNOT between non-adjacent qubits requires routing via SWAP gates.

- Calibration constraint: each physical qubit has unique, time-varying error rates, decoherence times (T₁, T₂), and calibrated gate frequencies. The transpiler must map logical qubits to physical qubits optimally given these per-qubit properties.

Transpilation resolves all three mismatches through a pipeline of transformations. Each transformation may increase circuit depth and gate count — the art of transpilation is minimising this overhead.

<figure class="book-figure">
<img src="content/images/image63.png" alt="Figure 4.5: The Qiskit transpilation pipeline. A logical circuit passes through four stages: (1) Unrolling — decompose all gates to the target basis set. (2) Optimisation — apply circuit identities to reduce gate count and depth. (3) Routing — insert SWAP gates to satisfy the coupling map. (4) Final native gate compilation. The transpile() function controls all stages via the optimization_level parameter.">
<figcaption>Figure 4.5: The Qiskit transpilation pipeline. A logical circuit passes through four stages: (1) Unrolling — decompose all gates to the target basis set. (2) Optimisation — apply circuit identities to reduce gate count and depth. (3) Routing — insert SWAP gates to satisfy the coupling map. (4) Final native gate compilation. The transpile() function controls all stages via the optimization_level parameter.</figcaption>
</figure>

#### The Three Transpilation Stages in Detail

Stage 1 — Basis Translation (Unrolling): Every gate in the logical circuit is decomposed into the hardware-native gate set. For IBM hardware, the native set is {U, CX} (or {U, ECR} on newer processors). The U gate is the general single-qubit gate U(θ,φ,λ) from Chapter 3. Any single-qubit gate is expressed as U(θ,φ,λ) for appropriate parameters. Two-qubit gates like Toffoli or SWAP are expressed as sequences of CX gates.

Stage 2 — Optimisation: Circuit identities are applied to reduce gate count and depth. Key optimisations include: rotation gate merging (Rz(θ₁)·Rz(θ₂) → Rz(θ₁+θ₂)), consecutive gate cancellation (U·U† → I), and commutation-based reordering. At higher optimisation levels, more aggressive techniques like template matching (checking if a subcircuit matches a known shorter equivalent) are applied.

Stage 3 — Routing: The most complex stage. The transpiler must map each logical qubit to a physical qubit and insert SWAP gates wherever the circuit requires a two-qubit gate between non-adjacent qubits. Finding the optimal mapping is NP-hard (it is equivalent to a subgraph isomorphism problem), so heuristic algorithms are used. Common routing algorithms include SABRE (Swap-based Bidirectional heuristic search for initial mapping with look-Ahead), basic swap insertion, and lookahead routing.

<div class="box box-example">
<p class="box-title"><strong>Example 4.4: Tracing Transpilation of a CNOT on Non-Adjacent Qubits</strong></p>
<p>Problem: A circuit requires CNOT(q0, q3) on an IBM device where only adjacent qubits are connected (linear chain: q0-q1-q2-q3). Trace the routing process.</p>
<p><strong>The coupling map (linear chain):</strong></p>
<p>q0 — q1 — q2 — q3    (only adjacent pairs can have CNOT)</p>
<p><strong>Step 1: q0 and q3 are not adjacent. CNOT(q0,q3) is not directly executable.</strong></p>
<p><strong>Step 2: SABRE router chooses to move q0's state toward q3 using SWAPs:</strong></p>
<p>SWAP(q0,q1): swaps the states of q0 and q1</p>
<p>SWAP(q1,q2): swaps the states of q1 and q2 (now q0's state is at physical q2)</p>
<p>CNOT(q2,q3): execute — q2 and q3 are adjacent! ✓</p>
<p><strong>Step 3: Resource cost:</strong></p>
<p>2 SWAP gates = 2 × 3 CNOTs = 6 extra CNOTs just for routing!</p>
<p>Original: 1 CNOT. After routing: 7 CNOTs. Overhead: 700%.</p>
<p>This illustrates why coupling map topology matters: a fully-connected device would need 0 SWAPs; a linear chain is the worst case.</p>
</div>

### 4.2.2 Coupling Maps and SWAP Routing

#### Coupling Maps: Physical Qubit Connectivity

The coupling map defines which pairs of physical qubits can have a direct two-qubit gate. It is a graph where nodes are qubits and edges are allowed two-qubit gate connections. Different quantum processors use different graph topologies, each with different routing properties.

| Processor | Topology | SWAP Routing Overhead | Example Devices |
|---|---|---|---|
| IBM Heavy Hex | Hexagonal lattice with extra edges | Low — 2.67 CNOTs/edge average | IBM Eagle, Heron, Flamingo |
| IBM Linear | Chain q0-q1-...-qn | Very High — O(n) SWAPs for distant qubits | Early IBM devices (ibmq_lima) |
| Google Sycamore | 2D grid | Medium — O(√n) SWAPs typical | Google Sycamore 53-qubit |
| Fully Connected | All-to-all (ion traps) | None — no SWAP routing needed | IonQ Aria, Quantinuum H2 |
| All-to-All Subset | Partial graph | Low — few extra SWAPs | Rigetti Aspen-M |

<figure class="book-figure">
<img src="content/images/image64.png" alt="Figure 4.6: Coupling map and SWAP routing. Left: IBM heavy-hex coupling map for 7 qubits — qubit 0 and qubit 4 are not directly connected (dashed red outline). Right: routing CNOT(q0,q4) on a linear chain requires 3 SWAP gates (= 9 CNOTs) overhead — illustrating the severe cost of connectivity mismatch.">
<figcaption>Figure 4.6: Coupling map and SWAP routing. Left: IBM heavy-hex coupling map for 7 qubits — qubit 0 and qubit 4 are not directly connected (dashed red outline). Right: routing CNOT(q0,q4) on a linear chain requires 3 SWAP gates (= 9 CNOTs) overhead — illustrating the severe cost of connectivity mismatch.</figcaption>
</figure>

#### The Heavy-Hex Lattice: IBM's Solution

IBM Quantum uses the heavy-hex coupling map on its Eagle, Osprey, Condor, Heron, and Flamingo processors. The heavy-hex lattice is a subgraph of the hexagonal lattice where each qubit has at most 3 connections. This topology was chosen because:

- It reduces the number of nearest neighbours per qubit from 4 (2D grid) to 2-3, reducing crosstalk errors

- The average SWAP routing overhead is 2.67 CNOTs per edge — better than a linear chain but slightly worse than a full 2D grid

- The graph structure enables efficient implementation of the surface code (the dominant quantum error correction code for superconducting qubits)

- Flag qubits (measurement ancilla) can be placed along the heavy edges, enabling efficient syndrome extraction

#### SABRE: The Routing Algorithm

Qiskit's default routing algorithm is SABRE (Swap-based Bidirectional heuristic search for initial mapping with look-Ahead). SABRE runs in two phases:

- Phase 1 — Initial mapping: use a heuristic to find an initial assignment of logical to physical qubits that minimises expected total SWAP cost. SABRE runs the routing algorithm forward and backward to find a good starting map.

- Phase 2 — SWAP insertion: process gates in topological order. For each two-qubit gate on non-adjacent qubits, insert a SWAP that moves one qubit closer to the other. Use a lookahead heuristic to choose which SWAP minimises future routing costs (not just immediate cost).

SABRE is an approximation algorithm — the routing problem is NP-hard. The quality of routing varies significantly with the circuit and hardware topology. At optimisation level 3, Qiskit runs SABRE multiple times with different random seeds and keeps the best result.

### 4.2.3 Qiskit PassManager and Optimisation Levels

#### The PassManager Architecture

Qiskit's transpiler is built as a PassManager — a pipeline of composable transformation passes, each performing a specific circuit transformation. This architecture allows precise control over the transpilation process: users can add, remove, or reorder passes, insert custom passes, or use pre-configured pipelines.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Qiskit Pass Types</strong></p>
<p>AnalysisPass: reads the circuit and stores data (e.g., compute circuit depth, find commutation groups) — does NOT modify the circuit</p>
<p>TransformationPass: modifies the circuit (e.g., decompose gates, insert SWAPs, merge rotations)</p>
<p>PassManager: an ordered collection of passes with optional repeat-until-fixpoint logic</p>
<p>StagedPassManager: organises passes into named stages (init, layout, routing, translation, optimization, scheduling)</p>
</div>

#### Four Optimisation Levels

The transpile() function exposes four pre-configured optimisation levels via optimization\_level=0/1/2/3:

| Level | Name | Passes Applied | Use Case |
|---|---|---|---|
| 0 | No Optimisation | Trivial layout, basic decomposition, basic routing | Debugging; fastest transpilation; no depth reduction |
| 1 | Light Optimisation | + 1-qubit gate merging, basic routing with SABRE | Quick tests; reasonable results with low compile time |
| 2 | Medium Optimisation | + CommutativeCancellation, Optimize1qGatesDecomposition, better routing | Default level; good balance of quality and compile time |
| 3 | Heavy Optimisation | + Multiple SABRE runs, Clifford simplification, template matching, noise-adaptive routing | Production circuits; maximum depth reduction; slow compile |

```python
# ─────────────────────────────────────────────────────────────────────
# Code 4.2: Qiskit Transpilation — Comparing Optimisation Levels
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_ibm_runtime.fake_provider import FakeSherbrooke  # 127-qubit fake backend
from qiskit.quantum_info import Operator

# ── Build a non-trivial test circuit: 4-qubit QFT-like circuit ────────
qc = QuantumCircuit(4)
qc.h(0)
qc.cx(0, 1)
qc.t(0)
qc.s(1)
qc.h(2)
qc.cx(2, 3)
qc.ccx(0, 1, 2)   # Toffoli — expensive to decompose
qc.h(3)
qc.cx(1, 3)
qc.rz(1.234, 0)

# ── Load fake backend (simulates IBM Eagle 127-qubit processor) ───────
backend = FakeSherbrooke()

# ── Transpile at all four levels ──────────────────────────────────────
results = {}
for level in range(4):
    t_qc = transpile(qc, backend=backend, optimization_level=level, seed_transpiler=42)
    results[level] = {
        'depth':      t_qc.depth(),
        'gate_count': t_qc.size(),
        'cx_count':   t_qc.count_ops().get('cx', 0),
        'ops':        dict(t_qc.count_ops()),
    }

# ── Print comparison table ─────────────────────────────────────────────
print(f"{'Level':>6} {'Depth':>8} {'Gates':>8} {'CX count':>10}")
print("-" * 36)
for lv, r in results.items():
    print(f"{lv:>6} {r['depth']:>8} {r['gate_count']:>8} {r['cx_count']:>10}")

# Expected output (approximate, varies by seed):
# Level   Depth    Gates   CX count
# ------------------------------------
#     0      47      62         18
#     1      31      48         14
#     2      25      41         11
#     3      19      36          9
```

<div class="box box-example">
<p class="box-title"><strong>Example 4.5: Custom PassManager for T-Count Reduction</strong></p>
<p>Problem: Build a custom Qiskit PassManager that: (1) decomposes all gates to {H, S, T, CNOT}, (2) applies rotation merging to reduce T-count, (3) cancels consecutive inverse gates.</p>
</div>

## 4.3 Quantum Programming Frameworks — Survey

Six major quantum programming frameworks are in active use by researchers and engineers worldwide. Each was built with a different primary goal, targets different hardware, and makes different design choices. Understanding all six — not just Qiskit — makes one a more effective quantum programmer and a better judge of which tool fits which problem.

<figure class="book-figure">
<img src="content/images/image65.png" alt="Figure 4.7: Quantum programming framework comparison. Left: radar chart comparing frameworks across six dimensions (hardware access, ML integration, fault-tolerant support, simulation power, ease of use, cross-platform portability). Right: framework release timeline from 2016–2024.">
<figcaption>Figure 4.7: Quantum programming framework comparison. Left: radar chart comparing frameworks across six dimensions (hardware access, ML integration, fault-tolerant support, simulation power, ease of use, cross-platform portability). Right: framework release timeline from 2016–2024.</figcaption>
</figure>

| Framework | Developer | Key Feature | Best For | Qiskit Relation |
|---|---|---|---|---|
| Qiskit ★ | IBM Quantum | Python SDK, OpenQASM 2/3, native hardware access, Runtime primitives | Primary language — this course; complete pipeline from design to hardware | — (this is Qiskit) |
| Cirq | Google | Moment-based circuits, Sycamore hardware, precise gate scheduling | NISQ experiments on Google hardware; fine-grained noise modelling | Complementary; circuits can be converted via pytket |
| Q# | Microsoft | Compiled language, type-safe, adjoint auto-generation, Azure Quantum | Fault-tolerant QC research; production algorithms with resource estimation | Different paradigm; bridges via QIR (Quantum Intermediate Representation) |
| QuTiP | Open Source | Lindblad master equations, density matrices, open system dynamics | Open quantum systems, decoherence modelling, noise characterisation | Complementary: Qiskit for circuits, QuTiP for noise analysis |
| TKET | Quantinuum | Hardware-agnostic compiler, pytket Python interface, many backends | Cross-platform compilation; optimise Qiskit circuits for different hardware | pytket-qiskit plugin converts circuits both ways; used in transpilation |
| PennyLane | Xanadu | Differentiable QC, automatic differentiation, ML integration | Quantum machine learning, variational algorithms with gradient-based optimisers | PennyLane.qiskit device runs PennyLane circuits on IBM hardware |

### 4.3.1 Qiskit — The Primary Framework

#### Architecture and Philosophy

Qiskit (Quantum Information Science Kit) is IBM's open-source Python SDK for quantum computing, first released in 2016 as part of the IBM Quantum Experience. It is the most widely used quantum computing framework in the world, with over 500,000 users and a comprehensive ecosystem spanning circuit construction, simulation, hardware access, and algorithm development.

Qiskit's design philosophy is pragmatic: make it easy to write circuits, simulate them, run them on real hardware, and extract useful results — with minimal friction. The framework is structured in layers:

- qiskit-terra (now qiskit core): circuit building, transpilation, PassManager, OpenQASM 2/3 import/export

- qiskit-aer: high-performance simulators (statevector, QASM, density matrix, pulse-level)

- qiskit-ibm-runtime: access to IBM Quantum hardware; Runtime primitives (Sampler, Estimator)

- qiskit-algorithms: library of quantum algorithms (VQE, QAOA, Grover, Shor, etc.)

- qiskit-nature, qiskit-finance, qiskit-optimization, qiskit-machine-learning: domain-specific libraries

#### OpenQASM 2 and 3

Qiskit uses OpenQASM (Open Quantum Assembly Language) as its intermediate representation. OpenQASM 2 is a simple gate-level language; OpenQASM 3 (released 2021) adds real-time classical control, classical types (int, float, bit arrays), subroutines, delay/duration specifications, and hardware timing control.

```qasm
// OpenQASM 3 example: dynamic circuit with classical control
OPENQASM 3;
include "stdgates.inc";

qubit[3] q;
bit[2] c;

h q[0];
cx q[0], q[1];
measure q[0] -> c[0];   // mid-circuit measurement
measure q[1] -> c[1];

if (c[0] == 1) {
    z q[2];              // classical feedforward
}
if (c[1] == 1) {
    x q[2];
}
```

#### Qiskit Runtime Primitives

Qiskit Runtime (introduced 2022) provides two high-level primitives that abstract away the complexity of running on real hardware:

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Qiskit Runtime Primitives</strong></p>
<p>Sampler: Runs a circuit and returns a probability distribution over measurement outcomes (bit strings). Handles shot noise, readout error mitigation. Returns PrimitiveResult with quasi-probabilities.</p>
<p>Estimator: Estimates the expectation value ⟨ψ|O|ψ⟩ of an observable O (Pauli operator) on a circuit state |ψ⟩. Essential for VQE, QAOA, quantum chemistry. Handles observable decomposition and error mitigation automatically.</p>
<p>Both primitives use error mitigation (Zero Noise Extrapolation, Probabilistic Error Cancellation) automatically when running on real hardware, significantly improving result quality over raw shots.</p>
</div>

```python
# ─────────────────────────────────────────────────────────────────────
# Code 4.3: Qiskit Runtime — Estimator for Expectation Values
# Estimates ⟨Z⊗Z⟩ for the Bell state — should be −1
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.quantum_info import SparsePauliOp
from qiskit_ibm_runtime import QiskitRuntimeService, EstimatorV2 as Estimator
from qiskit_ibm_runtime.fake_provider import FakeSherbrooke
from qiskit_aer import AerSimulator

# ── Build Bell state circuit ───────────────────────────────────────────
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)

# ── Define observable: Z⊗Z ────────────────────────────────────────────
observable = SparsePauliOp("ZZ")   # Z on q1, Z on q0

# ── Estimator on fake backend (no real hardware token needed) ──────────
fake_backend = FakeSherbrooke()
sim = AerSimulator.from_backend(fake_backend)

from qiskit_ibm_runtime import EstimatorV2
from qiskit_ibm_runtime.fake_provider import FakeSherbrookeV2

# With FakeSherbrookeV2 (supports V2 primitives):
estimator = EstimatorV2(backend=FakeSherbrookeV2())

# Transpile the circuit first
from qiskit import transpile
tqc = transpile(qc, FakeSherbrookeV2(), optimization_level=2)

# Run estimation
pub = (tqc, observable)   # Primitive Unified Bloc
job = estimator.run([pub])
result = job.result()

ev = result[0].data.evs   # expectation value
std = result[0].data.stds  # standard deviation
print(f'⟨Z⊗Z⟩ = {ev:.4f} ± {std:.4f}')
# Expected: ⟨Z⊗Z⟩ ≈ −1.0 (Bell state |Φ+⟩ = (|00⟩+|11⟩)/√2)
# Z⊗Z|00⟩ = |00⟩, Z⊗Z|11⟩ = |11⟩ → ⟨Φ+|Z⊗Z|Φ+⟩ = +1
# Correction: actually ⟨Φ+|ZZ|Φ+⟩ = (1+1)/2 = +1, not −1.
# To get −1, use the |Ψ−⟩ state or the XX observable.
```

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: IBM Quantum Network and Cloud Access</strong></p>
<p>IBM Quantum makes its processors available to researchers worldwide through IBM Quantum Network. As of 2024, IBM operates over 100 quantum systems globally, including the 1,121-qubit Condor, the 127-qubit Eagle, and the 133-qubit Heron processors. Access tiers range from free Open Plan (limited queue access) to Premium Plan (dedicated access, priority queuing, custom backend configuration). The IBM Quantum Falcon, Heron, and Eagle series are the production hardware lines; IBM's 2025 roadmap targets a 4,158-qubit Flamingo processor with modular interconnects. For Indian students, IBM's academic partnership with IITs and the National Quantum Mission provides subsidised access; the PARAM Quantum supercomputer (NQM initiative) also targets IBM-compatible hardware by 2027.</p>
</div>

### 4.3.2 Cirq (Google)

#### Design Philosophy: Moment-Based Circuits

Cirq (Circuit Framework), developed by Google's Quantum AI team and released in 2018, takes a fundamentally different approach from Qiskit: rather than abstracting away hardware details, Cirq makes them explicit. Cirq circuits are defined in terms of Moments — discrete time steps where a set of gates execute simultaneously. This gives the programmer precise control over timing, which is critical for noise mitigation on NISQ devices.

Cirq's design targets Google's Sycamore processor (and its successors, including Willow), which uses CZ as the native two-qubit gate (versus IBM's CX/ECR). The choice of CZ reflects Google's superconducting qubit architecture, where controlled-phase gates are more natural than controlled-NOT.

```python
# ─────────────────────────────────────────────────────────────────────
# Code 4.4: Cirq — Moment-Based Bell State Circuit
# ─────────────────────────────────────────────────────────────────────
import cirq

# ── Define qubits on a 2D grid (Sycamore uses GridQubits) ─────────────
q0 = cirq.GridQubit(0, 0)
q1 = cirq.GridQubit(0, 1)

# ── Build circuit using explicit moments ───────────────────────────────
circuit = cirq.Circuit([
    cirq.Moment([cirq.H(q0)]),                # Moment 1: H on q0
    cirq.Moment([cirq.CNOT(q0, q1)]),         # Moment 2: CNOT
    cirq.Moment([cirq.measure(q0, q1, key="result")]),  # Moment 3: measure
])

print(circuit)
# (0, 0): ───H───@───M(result)───
# (0, 1): ───────X───M(result)───

# ── Simulate with Cirq's built-in simulator ────────────────────────────
simulator = cirq.Simulator()
result = simulator.run(circuit, repetitions=1000)
print(result.histogram(key="result"))
# Counter({0: ~500, 3: ~500})  [0=|00⟩, 3=|11⟩ in binary encoding]

# ── Native gate set: decompose CNOT to CZ (Google native) ─────────────
native_circuit = cirq.Circuit(
    cirq.H(q0),
    cirq.H(q1),
    cirq.CZ(q0, q1),   # Native Google gate!
    cirq.H(q1),
    # Note: CNOT = (I⊗H)·CZ·(I⊗H)
)
```

#### Cirq's Noise Modelling Capabilities

One of Cirq's strengths is its fine-grained noise modelling. Every gate in a Cirq circuit can have an associated noise channel attached, allowing realistic device simulation:

- Depolarizing channel: models random Pauli errors with probability p after each gate

- Amplitude damping: models T₁ (energy relaxation) decay

- Phase damping: models T₂ (dephasing) — loss of phase coherence

- Device-specific noise: load calibrated noise data from real Sycamore device into simulation

This makes Cirq particularly powerful for NISQ algorithm research where understanding noise effects on specific circuits is critical.

### 4.3.3 Q# (Microsoft)

#### A Compiled, Type-Safe Quantum Language

Q# (Q Sharp) is Microsoft's quantum programming language, released in 2018 as part of the Microsoft Quantum Development Kit (QDK). Unlike Qiskit (Python-based) and Cirq (Python-based), Q# is a compiled, statically-typed language with full IDE support (Visual Studio, VS Code), designed specifically for quantum computing — not as a library on top of a classical language.

Q#'s design philosophy prioritizes correctness and scalability for large algorithms. Its key features are:

- Static type system: qubit, Result, Bool, Int, Double, Pauli types; type errors caught at compile time

- Adjoint auto-generation: Q# automatically derives the adjoint (inverse) of any operation, enabling uncomputation without manual coding

- Controlled auto-generation: any Q# operation can be automatically promoted to a controlled version

- Resource estimation: the Azure Quantum Resource Estimator gives exact qubit and T-gate counts for a circuit before running it on hardware

- Azure Quantum integration: access to IonQ, Quantinuum, and Pasqal hardware via Azure

```python
// Q# Code 4.5: Bell State in Q#
// Note the static typing, operation syntax, and adjoint support

namespace MPY305.Chapter4 {
    open Microsoft.Quantum.Canon;
    open Microsoft.Quantum.Intrinsic;
    open Microsoft.Quantum.Measurement;

    /// Creates a Bell state |Φ+⟩ = (|00⟩+|11⟩)/√2
    /// Returns measurement results (both bits)
    operation BellStateMeasurement() : (Result, Result) {
        // Allocate 2 qubits (automatically initialised to |0⟩)
        use (q1, q2) = (Qubit(), Qubit());

        // Create Bell state
        H(q1);           // Hadamard on first qubit
        CNOT(q1, q2);    // Entangle with second

        // Measure in Z basis
        let r1 = M(q1);
        let r2 = M(q2);

        // Reset qubits before release (Q# requirement!)
        Reset(q1);
        Reset(q2);

        return (r1, r2);  // Returns (Zero,Zero) or (One,One)
    }

    /// Auto-generated Adjoint: call Adjoint MyOp(q)
    /// Q# derives the inverse circuit automatically
    operation PreparePlusState(q : Qubit) : Unit is Adj + Ctl {
        H(q);   // Adjoint PrepareState automatically generates H† = H
    }
}
```

#### Azure Quantum Resource Estimator

Q#'s killer feature for fault-tolerant quantum computing research is the Azure Quantum Resource Estimator (QRE). Given a Q# program and a target hardware configuration (error rate, physical gate times, error correction code), QRE computes:

- Total physical qubit count needed (including error correction overhead)

- Total T-gate count (the primary runtime bottleneck)

- Total runtime (in microseconds or seconds on a physical device)

- Space-time volume (product of qubits and time, the fundamental resource metric)

This makes Q# uniquely powerful for answering questions like: "How large a quantum computer do we need to break RSA-2048 using Shor's algorithm?" (Answer: approximately 4.5 million physical qubits and 8 hours, per 2022 Microsoft estimates).

### 4.3.4 QuTiP, TKET, and PennyLane

#### QuTiP: Open Quantum Systems Simulation

QuTiP (Quantum Toolbox in Python) is an open-source Python library for simulating open quantum systems — quantum systems that interact with an environment and therefore experience decoherence. Unlike the other frameworks that simulate ideal (closed) quantum circuits, QuTiP works with density matrices and solves the Lindblad master equation:

<div class="box box-equation">
<p><strong>Equation 4.3</strong></p>
<p><strong>dρ/dt = −i[H, ρ] + Σₖ (Lₖ ρ Lₖ† − ½{Lₖ†Lₖ, ρ})</strong></p>
<p>where ρ is the density matrix, H is the Hamiltonian, Lₖ are Lindblad operators</p>
<p>representing decoherence channels (e.g., amplitude damping, dephasing)</p>
</div>

QuTiP is the standard tool for quantum optics, cavity QED, and noise characterisation. It bridges the gap between quantum circuit theory and quantum physics: when you want to know exactly how a qubit decays due to T₁ relaxation over 100 ns, QuTiP gives you the answer. Qiskit Aer's noise models are partly calibrated using QuTiP-based analysis of device data.

#### TKET (Quantinuum): Hardware-Agnostic Compilation

TKET (pronounced "ticket") is a quantum circuit compiler developed by Quantinuum (formerly Cambridge Quantum Computing). Its Python interface is pytket. TKET's defining feature is hardware agnosticism: it has backends for IBM Quantum, Google Cirq, Quantinuum H-series, IonQ, Rigetti, and simulators. It excels as a universal compiler that can optimise circuits for any target hardware.

TKET's key optimisation techniques include:

- Clifford simp: applies Clifford circuit identities aggressively, including graph-state rewriting

- Phase gadget synthesis: rewrites circuits in terms of phase gadgets (diagonal gates represented as a graph) which enable new cancellation opportunities

- ZX-calculus rewriting: transforms circuits to ZX-diagrams and applies diagrammatic rewriting rules before converting back to gates

- Architecture-aware routing: custom routing algorithms for each hardware topology

#### PennyLane (Xanadu): Differentiable Quantum Computing

PennyLane, developed by Xanadu and released in 2019, pioneered the paradigm of differentiable quantum computing — treating quantum circuits as differentiable functions whose gradients can be computed with respect to circuit parameters. This makes quantum circuits directly compatible with classical machine learning optimisers (SGD, Adam, RMSprop).

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Parameter-Shift Rule</strong></p>
<p>For a parameterised gate G(θ) = exp(−iθP/2) where P is a Pauli operator, the gradient of any expectation value with respect to θ is:</p>
<p>This "parameter-shift rule" computes exact gradients using only circuit evaluations (no backpropagation through quantum hardware). PennyLane implements this automatically, enabling gradient-based training of quantum circuits just like training neural networks.</p>
</div>

```python
# ─────────────────────────────────────────────────────────────────────
# Code 4.6: PennyLane — Differentiable Quantum Circuit (VQE-style)
# Minimise ⟨Z⟩ by training circuit parameter θ
# ─────────────────────────────────────────────────────────────────────
import pennylane as qml
import numpy as np

# ── Define device (simulator) ──────────────────────────────────────────
dev = qml.device("default.qubit", wires=1)

# ── Parameterised quantum circuit (quantum node) ───────────────────────
@qml.qnode(dev)
def circuit(theta):
    qml.RY(theta, wires=0)   # Parameterised rotation
    return qml.expval(qml.PauliZ(0))   # ⟨Z⟩

# ── Compute gradient using parameter-shift rule ────────────────────────
theta = np.array(0.5, requires_grad=True)
grad_fn = qml.grad(circuit)

# ── Gradient descent to minimise ⟨Z⟩ ─────────────────────────────────
for step in range(50):
    theta -= 0.1 * grad_fn(theta)   # gradient descent
    if step % 10 == 0:
        print(f'Step {step:3d}: ⟨Z⟩ = {circuit(theta):.4f}, θ = {theta:.4f}')

# ⟨Z⟩ → −1 (minimum) when θ → π (qubit at south pole of Bloch sphere)
# RY(π)|0⟩ = |1⟩, and ⟨1|Z|1⟩ = −1

# ── PennyLane-Qiskit bridge: run PennyLane circuit on IBM hardware ────
# pip install pennylane-qiskit
import pennylane_qiskit
ibm_dev = qml.device("qiskit.ibmq", wires=1,
                      backend='ibm_brisbane',  # real device
                      ibmqx_token="YOUR_TOKEN_HERE")
```

### 4.3.5 Framework Interoperability

#### The Interoperability Problem

A circuit written in Qiskit cannot directly run on Cirq's simulator or be submitted to Quantinuum hardware via Q#. Each framework uses its own circuit representation, gate naming conventions, and data structures. Interoperability — the ability to move circuits between frameworks — is therefore a significant practical challenge.

Several strategies and tools exist to address this:

- OpenQASM 2/3: the lingua franca of quantum circuits. Most frameworks can export to and import from OpenQASM, enabling indirect conversion.

- pytket (TKET): provides direct bridge plugins for all major frameworks. pytket-qiskit, pytket-cirq, pytket-pennylane enable one-line circuit conversion between frameworks.

- QIR (Quantum Intermediate Representation): Microsoft's LLVM-based IR for quantum programs. QIR provides a hardware-agnostic binary format for quantum programs, analogous to LLVM bitcode for classical programs.

- Pennylane device plugins: pennylane-qiskit, pennylane-cirq allow PennyLane circuits to run on IBM or Google backends.

```python
# ─────────────────────────────────────────────────────────────────────
# Code 4.7: Framework Interoperability — Qiskit ↔ Cirq ↔ PennyLane
# ─────────────────────────────────────────────────────────────────────

# ── Method 1: Qiskit → OpenQASM 3 → Cirq ─────────────────────────────
from qiskit import QuantumCircuit
from qiskit.qasm3 import dumps as qasm3_dumps

qc = QuantumCircuit(2)
qc.h(0); qc.cx(0,1)

# Export from Qiskit to OpenQASM 3
qasm_str = qasm3_dumps(qc)
print("OpenQASM 3:")
print(qasm_str)

# ── Method 2: Qiskit ↔ pytket ─────────────────────────────────────────
# pip install pytket pytket-qiskit
from pytket.extensions.qiskit import qiskit_to_tk, tk_to_qiskit

tket_circuit = qiskit_to_tk(qc)        # Qiskit → TKET
qiskit_back  = tk_to_qiskit(tket_circuit)  # TKET → Qiskit

# Apply TKET optimisations to Qiskit circuit:
from pytket.passes import FullPeepholeOptimise
FullPeepholeOptimise().apply(tket_circuit)  # TKET optimises
qiskit_optimised = tk_to_qiskit(tket_circuit)  # back to Qiskit

# ── Method 3: PennyLane running on IBM hardware via plugin ────────────
# import pennylane as qml
# dev = qml.device("qiskit.ibmq", wires=2, backend="ibm_brisbane")
# @qml.qnode(dev)
# def pl_bell():
#     qml.Hadamard(0)
#     qml.CNOT([0,1])
#     return qml.probs(wires=[0,1])

# ── Recommendation summary:
# Qiskit:    Circuit design + IBM hardware → use transpile() + Runtime
# Cirq:      Google hardware + noise modelling → use Moments + Simulator
# Q#:        Resource estimation + fault-tolerant → use QRE + Azure
# PennyLane: ML + gradients + training → use @qnode + qml.grad
# TKET:      Cross-platform optimisation → use pytket bridges
# QuTiP:     Open systems + decoherence → use mesolve() + Lindblad
```

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 4: Quantum Circuit Design, Transpilation & Programming Frameworks

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Define circuit depth formally using the concept of a DAG. Why is circuit depth the critical metric for NISQ hardware? What is the approximate maximum useful depth for IBM superconducting qubits?**

**Q2.  What is T-count? Why is it the dominant resource metric for fault-tolerant quantum computing rather than CNOT count? Describe magic state distillation.**

**Q3.  Explain Bennett's trick (uncomputation) with a concrete example. What happens to a quantum computation if ancilla qubits are not properly uncomputed?**

**Q4.  What are mid-circuit measurements and dynamic circuits? Give two applications where they are essential. How are they implemented in Qiskit?**

**Q5.  Describe the four stages of circuit transpilation. What does each stage accomplish and what is its output?**

**Q6.  What is a coupling map? Give the example of IBM's heavy-hex topology. Why does limited connectivity force SWAP routing, and what is the cost?**

**Q7.  Compare optimization\_level 0, 1, 2, and 3 in Qiskit's generate\_preset\_pass\_manager(). What trade-offs does each level make?**

**Q8.  What is an ISA (Instruction Set Architecture) circuit? Why can only ISA circuits be submitted to IBM Quantum hardware? What three conditions must an ISA circuit satisfy?**

**Q9.  Compare Qiskit, Cirq, and PennyLane on: (a) programming model, (b) key features, (c) best use case, (d) hardware access.**

**Q10.  What is circuit width? What are ancilla qubits and what is their purpose? How does qubit count constrain circuit design on NISQ hardware?**

**Q11.  Explain the circuit identity HXH = Z. How is this identity used in circuit simplification? Give another such identity and its application.**

**Q12.  What is gate cancellation in circuit optimisation? Give two examples of consecutive gates that cancel. Why does this reduce both gate count and error accumulation?**

**Q13.  Describe the SABRE routing algorithm. What problem does it solve and why is it used in Qiskit's transpiler?**

**Q14.  What is peephole optimisation in quantum circuits? Give an example of a 3-gate sequence that can be simplified to 1 gate.**

**Q15.  A quantum algorithm uses 40 Toffoli gates on a circuit with 6 qubits. Calculate: (a) the CNOT count after decomposition, (b) the T-count, (c) the approximate fidelity if CNOT error rate is 0.25%.**

**PART B — MODEL ANSWERS**

**Answer 1:**

Circuit depth = length of the longest path through the circuit's DAG (Directed Acyclic Graph), where nodes are gates and directed edges connect gates that share a qubit. Formally, it equals the minimum number of layers (time steps) needed to execute all gates, where all gates in one layer act on disjoint qubits. Critical for NISQ: decoherence times T₁ ≈ 100–500 μs limit the number of gate layers that complete before the quantum state degrades. With 2-qubit gate time ~300 ns, maximum useful depth ≈ T₂/(gate time) ≈ 100μs/300ns ≈ 333 layers theoretical; in practice ~50–200 two-qubit layers due to gate errors and crosstalk.

**Answer 2:**

T-count = number of T and T† gates in the {H,S,CNOT,T} decomposition. Dominant in fault-tolerant QC because: Clifford gates (H,S,CNOT) can be implemented transversally (fault-tolerantly with no extra overhead) in surface codes; T gates CANNOT be implemented transversally and require magic state distillation — consuming ~15 noisy physical T states (prepared via non-transversal operations) to produce one clean logical T state using the [[15,1,3]] Reed-Muller code. Thus each logical T gate costs ~15× more physical resources than a Clifford gate.

**Answer 3:**

Bennett's trick: Compute U|x⟩|0⟩=|x⟩|f(x)⟩ → use |f(x)⟩ → Uncompute U†|x⟩|f(x)⟩=|x⟩|0⟩. Example: Grover oracle computes f(x) into ancilla, uses it for phase kickback, then uncomputes. If ancilla not uncomputed: the joint state remains Σ\_x αₓ|x⟩|f(x)⟩ — ancilla is entangled with input register. This garbage entanglement means: (1) subsequent operations on the input also affect the ancilla, corrupting the computation; (2) measuring ancilla collapses the input register non-unitarily, destroying quantum information.

**Answer 4:**

Mid-circuit measurements measure one or more qubits during circuit execution, before the final measurement. Classical outcomes are used to control subsequent quantum gates (feedforward). Applications: (1) Quantum error correction: measure ancilla syndrome qubits without measuring data qubits; apply classical correction gates based on syndrome. (2) Quantum teleportation: Bell measurement on two qubits, then classically controlled X and Z corrections on receiving qubit. Qiskit implementation: `if\_test()` in OpenQASM 3.0, available on IBM Quantum hardware via dynamic circuits.

**Answer 5:**

Stage 1 — Basis Translation: converts all gates to hardware native set {U,CX} for IBM, replacing Toffoli→6CNOT+9SQ, H→U(π/2,0,π), etc. Stage 2 — Circuit Optimisation: reduces gate count/depth via cancellation, commutation, rotation merging; higher optimisation\_level applies more aggressive methods. Stage 3 — SWAP Routing: inserts SWAP gates to make all two-qubit operations act between physically adjacent qubits (satisfying coupling map). Stage 4 — Scheduling: assigns precise time slots to each instruction, optimising for parallelism and minimising idle time.

**Answer 6:**

Coupling map: graph where nodes = physical qubits, edges = directly coupled qubit pairs. IBM heavy-hex: hexagonal lattice where each qubit has 2–3 neighbours (degree 2 or 3), chosen to minimise crosstalk while providing sufficient connectivity. When two logical qubits that need to interact are not adjacent in the coupling map, SWAP gates must be inserted: each SWAP costs 3 CNOTs. For a 100-qubit circuit, SWAP routing can increase CNOT count by 3–5× on average, corresponding to the dominant source of overhead on real hardware.

**Answer 7:**

Level 0: minimal compilation — direct mapping, no optimisation, fastest compile time. Level 1: light optimisation — 1-2 peephole passes, basic gate cancellation, moderate noise-aware routing. Level 2: heavy optimisation — multiple passes, gate commutation, rotation merging, more aggressive qubit routing based on error rates. Level 3: maximum optimisation — full optimisation with noise-adaptive routing (selects qubits with best calibration), maximum gate reduction; slowest compile time but best circuit quality. For coursework: level 1 or 2. For hardware runs: level 3.

**Answer 8:**

An ISA circuit is a hardware-ready circuit that satisfies three conditions: (1) Uses only the backend's native gate set (e.g., {U,CX} for IBM Eagle), no abstract gates like H, Toffoli, or 3-qubit gates. (2) All two-qubit gates act between physically adjacent qubit pairs (satisfying the coupling map). (3) Uses physical qubit indices (0, 1, 2, ...) as assigned by the transpiler. Only ISA circuits can be submitted because the quantum control system has no knowledge of abstract gates or logical qubit numbering — it only understands its hardware-specific pulse sequences.

**Answer 9:**

Qiskit: Python library, QuantumCircuit objects, rich Aer simulator with noise models, IBM Quantum hardware access (free tier), largest community, best documentation. Best for IBM hardware, coursework, algorithm development. Cirq: Python library, explicit Moment structure, DensityMatrixSimulator, XEB tools, Google hardware (restricted). Best for research, fine noise control. PennyLane: Python library, automatic differentiation of quantum circuits, PyTorch/JAX/TensorFlow integration, parameter-shift gradients. Best for quantum ML, VQE/QAOA gradient optimisation.

**Answer 10:**

Circuit width = total number of qubits (main + ancilla). Hard NISQ constraint: IBM Heron has 133 physical qubits — a circuit requiring 134 qubits cannot run, period. Ancilla qubits are helper qubits initialised to |0⟩ for intermediate computations: implementing reversible arithmetic, phase kickback target, error syndrome qubits. Wide circuits (many ancilla) increase width, potentially exceeding device limits. Trade-off: adding ancilla qubits can reduce depth (parallelise computation) at cost of increased width.

**Answer 11:**

HXH = Z: the Hadamard conjugation converts X to Z, reflecting a basis change between X and Z measurements. Circuit use: replace H–X–H with Z (3 gates → 1). Replacing: after H, an X gate acts like Z in the computational basis. Another identity: X·H·X = −H — consecutive X gates around H simplify. Also: CNOT·(I⊗H)·CNOT = (I⊗H)·CZ·(I⊗H)^{−1}, converting between CNOT and CZ — used to change between IBM and Google native gate sets during transpilation.

**Answer 12:**

Gate cancellation: consecutive inverse gates can be removed without changing the computation. Examples: (1) H followed immediately by H: H·H = I — both removed. (2) CNOT followed immediately by CNOT (same control/target): CNOT² = I — both removed. (3) X followed by X: X² = I — both removed. Why important: fewer gates → fewer opportunities for errors. In a circuit with 0.3% error per gate, cancelling 10 gate pairs removes 20 gates → fidelity improves from (0.997)^{N} to (0.997)^{N-20}, a measurable improvement.

**Answer 13:**

SABRE (SWAP-based BidiREctional heuristic) solves the qubit routing problem: given a circuit with two-qubit gates and a hardware coupling map with limited connectivity, insert the minimum number of SWAP gates so all two-qubit gates act on adjacent qubits. Algorithm: builds a routing cost function considering both immediate gate satisfaction and future gate dependencies; uses bidirectional search (forward and backward through circuit) to find low-SWAP-count routes. Used in Qiskit level ≥1 because it provides near-optimal SWAP counts (within 1.5× optimal) in polynomial time.

**Answer 14:**

Peephole optimisation: scan 3–5 gate windows in the circuit, matching against known simplifiable patterns and replacing with shorter sequences. Example: T·T·T·T·T·T·T·T = I (T⁸ = I) → 8 T gates reduce to identity. Another: H·T·T·H = H·S·H = Rₓ(π/2) (2 T gates + 2 H → 1 rotation gate). Another: Z·X = iY → 2 gates reduce to 1 (up to global phase). In practice: Qiskit's InverseCancellation and CommutativeCancellation passes implement many such rules automatically.

**Answer 15:**

(a) 40 Toffoli × 6 CNOTs/Toffoli = 240 CNOTs. (b) T-count: 40 × 7 = 280 T gates. (c) Fidelity: F = (1−0.0025)^{240} = 0.9975^{240} = e^{240·ln(0.9975)} = e^{240·(−0.002503)} = e^{−0.601} ≈ 0.548 ≈ 54.8%. A 40-Toffoli circuit has only ~55% fidelity — over half of circuit executions produce wrong results. This motivates the need for error mitigation (ZNE) or error correction even for moderate-size circuits.

## EXERCISES — CHAPTER 4

### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.1</strong></p>
<p>Problem: A 5-qubit circuit has the following gate sequence on the qubits: q0: H, Rz(π/4); q1: H; q2: CNOT(q0,q2) at the same time as H on q1; q3: CNOT(q1,q3); q4: H. All single-qubit gates on q0,q1,q4 can run in parallel. Compute the circuit depth.</p>
<p><strong>Solution — assign layers:</strong></p>
<p>Layer 1: H on q0, H on q1, H on q4 (all single-qubit, disjoint qubits — parallel OK)</p>
<p>Layer 2: Rz(π/4) on q0, CNOT(q1→q3) (q0 and q1,q3 are disjoint — parallel OK)</p>
<p>Layer 3: CNOT(q0→q2) — must wait for Rz to finish on q0; q1,q3 now free</p>
<p>(Note: CNOT(q1→q3) requires H on q1 to complete first — done in layer 1)</p>
<p>Depth = 3.</p>
<p>Count: 7 gates total, depth 3. Parallelism reduced depth from 7 (sequential) to 3 — a 57% depth reduction. This is the power of parallel gate execution.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.2</strong></p>
<p>Problem: Apply the identity HXH = Z to simplify the following 7-gate circuit on one qubit: H, Z, H, X, H, Z, H. Show each step.</p>
<p><strong>Solution:</strong></p>
<p>Start: H · Z · H · X · H · Z · H</p>
<p>Step 1: Identify HZH = X (reading left to right, HZH group):</p>
<p>H · Z · H = X  →  circuit becomes: X · X · H · Z · H</p>
<p>Step 2: X·X = I (two consecutive X gates cancel):</p>
<p>Circuit becomes: H · Z · H</p>
<p>Step 3: H · Z · H = X (conjugation identity):</p>
<p>Final result: X</p>
<p>Summary: 7-gate sequence H·Z·H·X·H·Z·H simplifies to a single X gate. Original depth: 7. Optimised depth: 1. Savings: 86%.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.3</strong></p>
<p>Problem: A circuit requires a CNOT between physical qubits 0 and 5 on a linear chain (q0-q1-q2-q3-q4-q5). How many SWAP gates are needed for routing? What is the total CNOT count (SWAPs + original)?</p>
<p><strong>Solution:</strong></p>
<p>Distance between q0 and q5 on the linear chain: 5 edges apart.</p>
<p>For CNOT between qubits d edges apart, the minimum SWAP overhead is (d-1) SWAP gates.</p>
<p>Here d=5, so minimum SWAPs = 4 SWAP gates.</p>
<p>Circuit: SWAP(q0,q1), SWAP(q1,q2), SWAP(q2,q3), SWAP(q3,q4), then CNOT(q4,q5)</p>
<p>(After 4 SWAPs, q0's state has moved to position q4, adjacent to q5)</p>
<p>Total CNOT count: 4 SWAPs × 3 CNOTs/SWAP = 12 CNOTs (routing) + 1 CNOT (original) = 13 CNOTs total.</p>
<p>This dramatic overhead (13× the original) illustrates why coupling map topology is critical. A fully-connected device would need just 1 CNOT. This is why ion-trap computers (all-to-all connectivity) have a significant advantage for certain algorithms despite lower gate speed.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.4</strong></p>
<p>Problem: Explain what the Qiskit PassManager pass CommutativeCancellation does and give a specific circuit example where it reduces the gate count.</p>
<p><strong>Solution:</strong></p>
<p>CommutativeCancellation works in two phases:</p>
<p>Phase 1 — Commutation analysis: identifies pairs of gates that commute (can be reordered without changing the circuit output).</p>
<p>Phase 2 — Cancellation after reordering: after commuting past intervening gates, checks if the reordered pair are inverses and cancels them.</p>
<p><strong>Example — initial circuit on q0, q1:</strong></p>
<p>T(q0), CNOT(q0,q1), T†(q0), H(q1)</p>
<p>Can T(q0) and T†(q0) cancel directly? NO — CNOT(q0,q1) is between them.</p>
<p>Does T(q0) commute with CNOT(q0,q1)? YES — Rz gates commute with CNOT on the control qubit.</p>
<p>After commutation: T(q0), T†(q0), CNOT(q0,q1), H(q1)</p>
<p>Now T and T† are adjacent → cancel: I, CNOT(q0,q1), H(q1)</p>
<p>Final: CNOT(q0,q1), H(q1)  — reduced from 4 gates to 2 gates (50% reduction).</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.5</strong></p>
<p>Problem: Compare Qiskit's Estimator primitive with the Sampler primitive. When should each be used? What are the input/output types of each?</p>
<p><strong>Solution:</strong></p>
<p>Sampler primitive:</p>
<p>Input: quantum circuit(s) with measurements already in the circuit</p>
<p>Output: quasi-probability distribution over bit strings {0,1}ⁿ</p>
<p>Used for: measuring the probability distribution of measurement outcomes; counting algorithm outputs; quantum state characterisation</p>
<p>Example: counting algorithm counting marked items in Grover search</p>
<p>Estimator primitive:</p>
<p>Input: quantum circuit(s) WITHOUT measurements + observable (Pauli operator or sum)</p>
<p>Output: expectation value ⟨ψ|O|ψ⟩ ± standard deviation</p>
<p>Used for: energy estimation in VQE; cost function in QAOA; any gradient-based optimisation</p>
<p>Example: estimating the Hamiltonian expectation ⟨H⟩ for molecular ground state energy</p>
<p>Key distinction: Sampler needs measurements in the circuit (destructive). Estimator handles multiple observables more efficiently by exploiting operator algebra — it does NOT need to measure in each Pauli basis separately; it can reuse circuit evaluations via operator grouping.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.6</strong></p>
<p>Problem: A student writes a 10-qubit Qiskit circuit using CCX (Toffoli), SWAP, and Rz(θ) gates. They run transpile(qc, backend, optimization_level=0) and get depth=85. They then run optimization_level=3 and get depth=32. Explain what optimisations most likely reduced the depth by 62%.</p>
<p><strong>Solution:</strong></p>
<p>Major optimisations between level 0 and level 3 for this circuit:</p>
<p>1. Toffoli decomposition choice (saves ~10 depth): Level 0 uses the standard 6-CNOT Toffoli decomposition with fixed structure. Level 3 uses the relative-phase Toffoli (uses only 3 CNOTs when the ancilla phase doesn't matter) or the Margolus gate, reducing depth per Toffoli.</p>
<p>2. SWAP routing improvement (saves ~20 depth): Level 0 uses trivial layout + basic SABRE. Level 3 runs SABRE multiple times with different seeds and noise-adaptive qubit mapping, finding better initial qubit placement that needs fewer SWAPs. Each SWAP saved removes 3 CNOTs.</p>
<p>3. Rotation gate merging (saves ~5 depth): Multiple consecutive Rz(θ) gates get merged into a single Rz(θ₁+θ₂+...) gate.</p>
<p>4. Commutative gate cancellation (saves ~8 depth): Rz gates commuted past CNOTs, exposing T·T† pairs that cancel.</p>
<p>5. Template matching (saves ~7 depth): recognises standard 3-gate patterns (e.g., CNOT·H·CNOT sequences) that match known shorter equivalents.</p>
<p>Depth 85 → 32 represents a 62% reduction. At 0.3% CNOT error rate, this could improve fidelity from (0.997)^{85×2} ≈ 60% to (0.997)^{32×2} ≈ 83%.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.7</strong></p>
<p>Problem: Implement the uncomputation pattern to compute f(a,b) = a XOR b using CNOT, copy the result, and uncompute. Verify that the ancilla returns to |0⟩ for all inputs.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 4.8</strong></p>
<p>Problem: Compare the T-count of two circuits that both compute the same function but use different methods: (a) Standard Toffoli decomposition (6 CNOTs, 7 T/T† gates), (b) Selinger's T-count-3 Toffoli (using 3 T/T† gates, relative phase). Which is preferred in a fault-tolerant context and why?</p>
<p><strong>Solution:</strong></p>
<p>Standard Toffoli (T-count = 7):</p>
<p>Uses T·T†·T·T·T†·T·T gate pattern (standard ABC decomposition from Barenco et al.)</p>
<p>Produces exact Toffoli: CCX|abc⟩ = |ab, c⊕(a·b)⟩ for ALL input states</p>
<p>T-count = 7, depth ≈ 11 in the T-layer counting</p>
<p>Selinger's relative-phase Toffoli (T-count = 3):</p>
<p>Produces a gate that is EQUIVALENT to Toffoli when the target qubit starts in |0⟩</p>
<p>If c = |0⟩: output is exactly |ab,a·b⟩</p>
<p>If c ≠ |0⟩: output has a relative phase error — NOT equivalent to Toffoli</p>
<p>T-count = 3 (over 50% reduction)</p>
<p>Fault-tolerant preference: Selinger's T-count-3 version is strongly preferred when the target ancilla starts in |0⟩ (which is the case in almost all practical Toffoli applications). The savings are enormous:</p>
<p>For 1000 Toffoli gates: 7000 T gates → 3000 T gates</p>
<p>In a surface code with 1μs cycle time and 15-1 distillation: 7000 T gates needs 7000×15 = 105,000 distillation rounds ≈ 105 ms; 3000 T gates needs only 45 ms.</p>
<p>Always use the minimal-T-count Toffoli unless the ancilla starting state is unknown.</p>
</div>

### B. Unsolved Problems

Solve the following problems independently. Answers are provided in brackets for self-checking.

1. A circuit has 8 single-qubit gates and 5 CNOT gates arranged so that 4 of the CNOTs can run in parallel with single-qubit gates on other qubits. What is the minimum possible circuit depth? [Answer: minimum depth = 3 (if all non-dependent gates parallelise: one layer of single-qubit gates, one layer with 4 CNOTs + remaining single-qubit gates, one layer with the last CNOT)]

2. How many T gates does a single Toffoli gate contribute to a fault-tolerant circuit in (a) standard decomposition, (b) Selinger relative-phase decomposition? If an algorithm uses 500 Toffoli gates and 200 stand-alone T gates, what is the total T-count in each case? [Answer: (a) 7 per Toffoli → 500×7 + 200 = 3700; (b) 3 per Toffoli → 500×3 + 200 = 1700. Using Selinger saves 2000 T gates = 54% reduction]

3. A coupling map is a cycle graph on 5 qubits: q0-q1-q2-q3-q4-q0 (each qubit connected to two neighbours in a ring). What is the maximum distance between any two qubits? How many SWAP gates are needed to route a CNOT between the two most distant qubits? [Answer: maximum distance = 2 (e.g., q0 to q2 via q1 or via q4); SWAP routing needs 1 SWAP = 3 CNOTs overhead. A ring topology is much better than a linear chain for routing!]

4. Explain why transpile(qc, backend, optimization\_level=3) takes longer than optimization\_level=0 but produces better circuits. What is the time complexity of SABRE routing? [Answer: Level 0: trivial passes, O(n) time. Level 3: multiple SABRE runs O(k·n·G) where k = number of seeds tried (typically 20), G = circuit gate count, n = qubit count; plus template matching O(n²). Slower compile → fewer SWAP gates → faster/better circuit execution on hardware]

5. Write the OpenQASM 3 code for a simple if-else dynamic circuit: measure qubit q0; if the result is 1, apply X to q1; else apply H to q1. [Answer: measure q[0] -> c[0]; if(c[0]==1) { x q[1]; } else { h q[1]; }]

6. A PennyLane circuit has trainable parameter θ. Using the parameter-shift rule, how many circuit evaluations are needed to compute the gradient ∂⟨O⟩/∂θ? How does this compare to classical automatic differentiation? [Answer: 2 circuit evaluations per parameter (one at θ+π/2, one at θ−π/2). For p parameters: 2p evaluations total. Classical autodiff uses 1 forward + 1 backward pass = O(1) gradient evaluations, but cannot be applied to real quantum hardware — the parameter-shift rule is the only hardware-compatible gradient method]

7. What is the Lindblad master equation and what physical processes does it model that are missing from ideal quantum circuit simulation? Give two specific examples of Lindblad operators for superconducting qubits. [Answer: dρ/dt = −i[H,ρ] + Σ(LρL†−½{L†L,ρ}). Models energy relaxation (T₁) and dephasing (T₂ < 2T₁). Lindblad ops: L₁ = √(1/T₁)·σ₋ (amplitude damping = |1⟩→|0⟩ decay); L₂ = √(1/T₂−1/2T₁)·σ\_z (pure dephasing = random phase flip)]

8. In TKET's pytket, what is "phase gadget synthesis"? Why is it a better approach than simple peephole optimisation for reducing CNOT count? [Answer: Phase gadgets represent diagonal unitaries as a graph where nodes = qubits and edges = phase parameters. Multiple phase gadgets can be merged if their graphs overlap, cancelling common edges — this is equivalent to reducing CNOT count without needing to recognise specific gate patterns. It works globally on the circuit, not window-by-window like peephole optimisation, and can find optimisations invisible to local scanning.]

9. A Qiskit circuit uses 3 Rz(θ₁), Rz(θ₂), Rz(θ₃) gates in sequence on the same qubit. After rotation merging, how many gates remain? What is the resulting parameter? [Answer: 1 gate remains: Rz(θ₁ + θ₂ + θ₃). Rotation merging combines any sequence of commuting single-axis rotations into a single rotation. This is applied by Optimize1qGatesDecomposition in Qiskit.]

10. Compare Q# and Qiskit on five specific criteria: (a) language type, (b) adjoint generation, (c) hardware access, (d) best use case, (e) resource estimation. [Answer: (a) Q#: compiled/static vs Qiskit: Python/dynamic. (b) Q#: automatic via "is Adj" vs Qiskit: must manually write inverse circuit. (c) Q#: Azure Quantum (IonQ, Quantinuum, Pasqal) vs Qiskit: IBM Quantum (+ simulators). (d) Q#: fault-tolerant design/resource estimation vs Qiskit: NISQ experiments/education/production on IBM. (e) Q#: Azure QRE gives exact physical qubit + T-gate counts vs Qiskit: no built-in resource estimator (use qiskit.transpiler for proxy counts)]

### C. Multiple Choice Questions

Note: Answers to all MCQs are given at the end of this section.

**Q1. Circuit depth is defined as:**

(a)  The total number of gates in the circuit

(b)  The number of qubits used by the circuit

(c)  The length of the longest sequential path through the circuit DAG

(d)  The number of two-qubit gates

**Q2. The T-count metric is important in fault-tolerant quantum computing because:**

(a)  T gates are the cheapest gates to implement

(b)  T gates require magic state distillation, which dominates the resource cost

(c)  T gates are the only gates that create entanglement

(d)  T gates can be implemented transversally in surface codes

**Q3. Which circuit identity allows two adjacent Hadamard gates on the same qubit to be removed?**

(a)  HXH = Z

(b)  H·H = I

(c)  H†H = 0

(d)  H = (X+Z)/√2

**Q4. In Bennett's uncomputation trick, after performing U and copying the result, U† is applied to:**

(a)  Erase the output register

(b)  Return the ancilla qubits to |0⟩ (unentangle them from the input)

(c)  Compute the adjoint of the output

(d)  Apply error correction to the result

**Q5. Mid-circuit measurement is required for which of the following?**

(a)  Computing Grover's search oracle

(b)  Quantum teleportation with classical feedforward correction

(c)  Creating a Bell state

(d)  Implementing the Toffoli gate

**Q6. The Qiskit transpile() optimization\_level=3 differs from level=0 primarily by:**

(a)  Using a different gate set

(b)  Adding more qubits to the circuit

(c)  Applying more aggressive optimisation passes including multiple SABRE routing runs and template matching

(d)  Disabling SWAP routing

**Q7. The coupling map of a quantum processor defines:**

(a)  The set of native single-qubit gates

(b)  Which pairs of physical qubits can have a direct two-qubit gate

(c)  The clock frequency of each qubit

(d)  The order in which gates are executed

**Q8. SABRE is an algorithm for:**

(a)  Decomposing Toffoli gates into CNOT gates

(b)  Finding the optimal qubit mapping and inserting SWAP gates for routing on a coupling map

(c)  Computing the ZYZ Euler decomposition of single-qubit gates

(d)  Estimating the expectation value of an observable

**Q9. The Qiskit Runtime Estimator primitive is best suited for:**

(a)  Sampling bit string distributions for counting algorithms

(b)  Estimating the expectation value ⟨ψ|O|ψ⟩ of an observable (needed for VQE)

(c)  Drawing circuit diagrams

(d)  Transpiling circuits to a specific backend

**Q10. Cirq's primary structural unit, distinct from Qiskit, is:**

(a)  The QuantumCircuit object

(b)  The Moment — a discrete time step containing simultaneously-executing gates

(c)  The OpenQASM 3 string

(d)  The Clifford tableau

**Q11. Q#'s "is Adj + Ctl" annotation on an operation means:**

(a)  The operation is a Clifford gate

(b)  The adjoint and controlled versions are automatically generated by the compiler

(c)  The operation can only be used with ancilla qubits

(d)  The operation is an adjoint of the Controlled-NOT gate

**Q12. PennyLane's parameter-shift rule computes the gradient of ⟨O⟩ with respect to θ using:**

(a)  Numerical finite differences (⟨O⟩(θ+ε) − ⟨O⟩(θ))/ε

(b)  Classical automatic differentiation (backpropagation)

(c)  Two circuit evaluations at θ+π/2 and θ−π/2

(d)  A single circuit evaluation with a special measurement basis

**Q13. TKET's pytket-qiskit plugin allows:**

(a)  Running Qiskit circuits only on Quantinuum hardware

(b)  Converting circuits between Qiskit and TKET formats, enabling TKET optimisation of Qiskit circuits

(c)  Simulating Qiskit circuits using QuTiP's density matrix solver

(d)  Compiling Qiskit circuits to Q# operations

**Q14. The IBM heavy-hex coupling map topology was chosen over a full 2D grid because:**

(a)  It allows faster gate operation times

(b)  It reduces qubit crosstalk by lowering the number of nearest neighbours, enabling lower error rates

(c)  It requires fewer physical qubits to implement the same algorithm

(d)  It eliminates the need for SWAP routing entirely

**Q15. The Lindblad master equation, used in QuTiP, models quantum systems that are:**

(a)  Isolated from the environment (closed quantum systems)

(b)  Interacting with an environment, experiencing decoherence (open quantum systems)

(c)  Quantum systems with more than 2 dimensions

(d)  Systems in thermal equilibrium only

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 4</strong></p>
<p>Q1: (c) Length of the longest sequential path through the circuit DAG — depth measures the time steps needed, not gate count</p>
<p>Q2: (b) T gates require magic state distillation — Clifford gates can be implemented transversally; T gates cannot, requiring costly magic state factories (O(n log n) physical gates per logical T gate)</p>
<p>Q3: (b) H·H = I — H is self-inverse; two adjacent H gates on the same qubit cancel. Note: HXH = Z (true but this is a conjugation identity, not a cancellation)</p>
<p>Q4: (b) Return ancilla qubits to |0⟩ — Bennett's trick: Compute → Copy → Uncompute (U†). U† reverses the computation, disentangling the ancilla from the input registers</p>
<p>Q5: (b) Quantum teleportation — Alice must measure her qubits and send the classical results to Bob who applies corrections. Without mid-circuit measurement + feedforward, the correction cannot be applied in the same circuit run</p>
<p>Q6: (c) More aggressive optimisation passes — level=3 adds: multiple SABRE seeds (better routing), template matching, Clifford simplification, noise-adaptive layout. Gate set and qubit count remain the same</p>
<p>Q7: (b) Which pairs of physical qubits can have a direct two-qubit gate — it is a graph where edges = allowed CNOT/CZ connections. Single-qubit gates can be applied to any qubit regardless of coupling map</p>
<p>Q8: (b) Finding optimal qubit mapping and inserting SWAP gates — SABRE = Swap-based Bidirectional heuristic search for initial mapping with look-Ahead. It is the standard routing algorithm in Qiskit</p>
<p>Q9: (b) Estimating ⟨ψ|O|ψ⟩ for VQE — Sampler returns bit string distributions (for counting); Estimator returns expectation values (for energy estimation in variational algorithms)</p>
<p>Q10: (b) The Moment — Cirq's key abstraction is explicit time steps (Moments) where non-overlapping gates execute simultaneously. Qiskit uses a higher-level circuit abstraction without explicit timing</p>
<p>Q11: (b) Adjoint and controlled versions are auto-generated — "is Adj" tells Q# compiler to automatically derive the adjoint (inverse circuit); "Ctl" enables controlled version. This eliminates manual inverse circuit coding</p>
<p>Q12: (c) Two circuit evaluations at θ±π/2 — the parameter-shift rule gives exact gradients: ∂⟨O⟩/∂θ = [⟨O⟩(θ+π/2) − ⟨O⟩(θ−π/2)]/2. Works on real hardware unlike backpropagation</p>
<p>Q13: (b) Converting between Qiskit and TKET formats — pytket-qiskit provides qiskit_to_tk() and tk_to_qiskit() functions, enabling use of TKET's superior ZX-calculus and phase gadget optimisations on Qiskit circuits</p>
<p>Q14: (b) Reduces qubit crosstalk by lowering nearest neighbour count — heavy-hex gives each qubit max 3 connections (vs 4 for 2D grid), reducing parasitic ZZ coupling. This enables lower two-qubit gate error rates at the cost of slightly worse connectivity</p>
<p>Q15: (b) Open quantum systems experiencing decoherence — the Lindblad equation generalises the Schrödinger equation to include environment interactions (energy relaxation T₁, dephasing T₂). Closed systems use the unitary Schrödinger equation only</p>
</div>

### D. Theory Questions

- Define circuit depth, circuit width, and T-count. Explain the physical significance of each metric: which limits performance on NISQ devices, which determines fault-tolerant resource requirements, and which governs classical simulation complexity? Give a concrete example showing how each metric affects a real algorithm.

- Explain the uncomputation principle (Bennett's trick) in detail. Why is it necessary for ancilla qubits in quantum algorithms? Derive the resource overhead of uncomputation: if the original computation uses k gates and a ancilla qubits, what does full uncomputation cost in gates and qubits?

- Explain what a coupling map is, why it exists (physical hardware reasons), and how it forces SWAP insertion. Describe the IBM heavy-hex coupling map and explain why it was chosen over a 2D grid. Calculate the worst-case SWAP overhead for a circuit requiring all-to-all connectivity on an n-qubit linear chain.

- Describe the four stages of Qiskit's transpilation pipeline. For each stage, give the specific Qiskit passes involved at optimization\_level=2, explain what it does mathematically, and state what circuit resources it reduces.

- Compare mid-circuit measurement with deferred measurement (end-of-circuit measurement). Under what conditions are they equivalent (same probabilities)? Give an example (quantum teleportation) where mid-circuit measurement with feedforward cannot be replaced by deferred measurement without modifying the algorithm.

- Compare Qiskit and PennyLane for implementing a variational quantum algorithm (VQE). For each framework, describe: (a) how the parameterised circuit is defined, (b) how the gradient is computed, (c) how the classical optimiser is integrated, (d) how to run on real hardware. Which is better for VQE and why?

- Explain Q#'s adjoint auto-generation. What mathematical conditions must an operation satisfy for the Q# compiler to automatically derive its adjoint? Give an example showing the adjoint of a multi-gate sequence and verify by matrix multiplication.

- What is OpenQASM 3? How does it improve on OpenQASM 2? List five specific features added in version 3 and explain how each enables a new class of quantum program that was impossible in QASM 2.

- Explain ZX-calculus as used by TKET for circuit optimisation. What are Z-spiders and X-spiders? State three ZX rewriting rules and show how they can reduce a circuit's CNOT count beyond what peephole optimisation can achieve.

- The Qiskit PassManager architecture separates circuit transformations into composable passes. Explain the distinction between AnalysisPass and TransformationPass. Design a custom 5-pass PassManager for a fault-tolerant compilation pipeline: specify each pass, its purpose, and the order they should run. Justify the ordering.

### E. Programming Assignments

PA4.1. [Transpilation Study]  Write a Qiskit program that: (a) constructs the 5-qubit Grover oracle circuit for target |10101⟩ (use X gates on appropriate qubits + multi-controlled-Z), (b) transpiles the circuit to a fake IBM backend (FakeSherbrooke or FakeNairobi) at all four optimization\_levels, (c) collects metrics for each level: depth, gate\_count, cx\_count, (d) plots a grouped bar chart comparing these metrics across levels. Write a paragraph analysing which pass contributes most to depth reduction between levels. Submit code, chart, and analysis.

PA4.2. [Dynamic Circuits]  Implement the complete quantum teleportation protocol in Qiskit using dynamic circuits (mid-circuit measurement + feedforward). (a) Prepare an arbitrary unknown state |ψ⟩ = Ry(1.23)Rz(0.45)|0⟩ on qubit 0. (b) Create a Bell pair between qubits 1 and 2. (c) Perform Alice's Bell measurement using mid-circuit measurement. (d) Apply Bob's corrections using classical feedforward (qc.if\_test syntax). (e) Run on FakeSherbrooke and verify the teleported state on qubit 2 matches the original by measuring in three bases (Z, X, Y) and comparing statistics. Submit code and verification analysis.

PA4.3. [Framework Comparison]  Implement the same circuit (4-qubit QFT) in three frameworks: (a) Qiskit — use QFT from qiskit.circuit.library, transpile to FakeNairobi at level 2, collect metrics. (b) Cirq — implement QFT manually using controlled-Rz gates in explicit Moments, simulate with cirq.Simulator, measure output state fidelity vs ideal. (c) PennyLane — implement QFT as a differentiable @qnode, compute the gradient of ⟨Z₀⟩ with respect to one rotation parameter. Compare: gate count, depth, ease of implementation, and gradient computation capability. Write a 1500-word report.

### F. Project Suggestions

Project 4.A — Custom Quantum Compiler Pass:  Implement a custom Qiskit TransformationPass that performs T-gate merging optimisation: given a circuit with consecutive T and T† gates separated only by commuting gates, find and cancel them. (a) Implement the pass class inheriting from TransformationPass. (b) Build the commutativity analysis (which gates commute with T?). (c) Benchmark your pass on 10 random circuits of depth 50-100 with 20-30 T/T† gates, measuring T-count reduction. (d) Compare with Qiskit's built-in InverseCancellation pass. Write a 2000-word technical report with benchmarks and code.

Project 4.B — NISQ Noise Study via Qiskit and QuTiP:  Study the effect of T₁ and T₂ decoherence on a Grover search circuit. (a) Build a 3-qubit Grover circuit in Qiskit (one iteration). (b) Run on FakeSherbrooke with the built-in noise model. (c) Build an equivalent simulation in QuTiP using the Lindblad master equation with realistic T₁=100μs, T₂=80μs parameters. (d) Compare the measurement outcome distributions from both simulators. (e) Plot the success probability of Grover search as a function of circuit depth (by adding delay/idle time). Write a 2500-word report explaining the physics of T₁/T₂ decoherence and its quantitative effect on the algorithm.

Project 4.C — Cross-Platform Circuit Portability:  Implement the 3-qubit Bernstein-Vazirani algorithm (finding hidden bitstring s=101) in all four frameworks: Qiskit, Cirq, Q#, and PennyLane. (a) Implement the oracle and full circuit in each framework. (b) Measure: lines of code, programming difficulty (subjective), native gate count after compilation. (c) Run on simulators for each framework and verify correctness. (d) Use pytket to convert the Qiskit circuit to TKET format and apply TKET's FullPeepholeOptimise; compare resulting gate count. (e) Write a 3000-word comparative analysis evaluating each framework for teaching quantum algorithms, including recommendations for Indian universities with limited compute access.

## References and Further Reading

1. Nielsen, M. A. & Chuang, I. L. (2000). Quantum Computation and Quantum Information. Cambridge University Press. [Chapters 4, 5 — circuit model, universality, simulation]

2. Qiskit Documentation (2024). Qiskit 1.x User Guide. https://docs.quantum.ibm.com. [Transpiler, PassManager, Runtime primitives]

3. Cirq Documentation (2024). https://quantumai.google/cirq. [Moment-based circuits, noise models, Sycamore]

4. Microsoft Q# Documentation (2024). https://learn.microsoft.com/azure/quantum. [Q# language, Azure QRE, resource estimation]

5. Bergholm, V. et al. (2022). PennyLane: Automatic differentiation of hybrid quantum-classical computations. arXiv:1811.04968. [Differentiable QC, parameter-shift rule]

6. Cowtan, A. et al. (2020). On the Qubit Routing Problem. Proceedings TQC 2019. [SABRE routing, coupling map optimisation]

7. Bennett, C. H. (1973). Logical Reversibility of Computation. IBM Journal of Research and Development. [Uncomputation, Bennett's trick, ancilla management]

8. Selinger, P. (2013). Quantum circuits of T-depth one. Physical Review A. [T-count optimisation, relative-phase Toffoli]

9. Johansson, J. R. et al. (2013). QuTiP 2: A Python framework for open quantum systems. Computer Physics Communications. [Lindblad master equation, open systems simulation]

10. IBM Quantum Network (2024). https://quantum.ibm.com. [Heavy-hex coupling map, Heron processors, dynamic circuits]
