# Unit III - CHAPTER 5: Quantum Algorithms

# Oracle Models, Quantum Fourier Transform & Phase Estimation

<div class="box box-anecdote">
<p class="box-title"><strong>📜  Peter Shor's Discovery — Bell Labs, April 1994</strong></p>
<p>On a spring afternoon in 1994, Peter Shor sat in a lecture at a workshop on quantum computing organised by Bell Labs. He had been thinking for weeks about a talk by Dan Simon outlining a quantum algorithm exponentially faster than any classical one for a certain algebraic problem. Shor saw the pattern immediately: Simon's algorithm exploited a hidden group structure using quantum parallelism and interference. Integer factorisation — the problem underlying RSA cryptography — had exactly such a hidden group structure. He went home that evening and began working. Within days he had the skeleton of what would become the most famous quantum algorithm in history.</p>
<p>The algorithm uses the Quantum Fourier Transform as its computational engine — the same mathematical object that underlies every algorithm in this chapter. The QFT is to quantum computing what the FFT is to classical signal processing: the universal tool that converts between two complementary representations of information. Understanding the QFT deeply is understanding the heart of quantum algorithmic speedup.</p>
<p>This chapter builds the QFT from first principles, develops the oracle-based algorithm framework that preceded it historically, and culminates in Quantum Phase Estimation — the subroutine that appears inside Shor's algorithm, quantum chemistry simulations, and the HHL linear systems algorithm. These are not disparate topics: they are a coherent mathematical story about what quantum interference can compute.</p>
</div>

The algorithms in this chapter answer a fundamental question: for which computational problems does quantum mechanics provide a genuine speedup over the best possible classical algorithm? This is not merely asking whether quantum computers are "faster" in some vague sense — it requires proving lower bounds (no classical algorithm can do better) and constructing quantum algorithms that achieve those bounds. Remarkably, for several important problem classes, we have definitive answers.

The chapter is organised as a progression from simple to complex. We begin with the oracle model — the theoretical framework for measuring algorithmic efficiency by query count — and develop four oracle-based algorithms (Deutsch, Deutsch-Jozsa, Bernstein-Vazirani, Simon) that demonstrate exponential and polynomial quantum speedups. We then develop the Quantum Fourier Transform as a standalone circuit, before showing how it combines with controlled unitary operations to produce Quantum Phase Estimation — one of the most powerful subroutines in all of quantum computing.

## 5.1 Oracle-Based Algorithms and Quantum Speedup

The oracle model of computation provides a clean, hardware-independent framework for studying quantum speedups. Rather than asking "how many arithmetic operations does an algorithm use?" — which depends on implementation details — we ask "how many times does an algorithm query a black-box function f?" This query complexity perspective reveals quantum speedup in its purest form, stripped of implementation concerns.

### 5.1.1 The Quantum Oracle Model

#### The Classical Oracle

In classical computation, an oracle for a Boolean function f: {0,1}ⁿ → {0,1}ᵐ is a black box that, given any input x, returns f(x) in one step. The query complexity of an algorithm is the number of calls to this oracle. Classical randomised algorithms can often compute properties of f using fewer queries than deterministic algorithms; quantum algorithms can use even fewer.

Why study query complexity at all? Because many important problems — database search, period finding, learning hidden functions — can be abstracted as oracle problems where the oracle represents some unknown structure we are trying to discover. Query complexity lower bounds are often provable, giving us absolute statements like "no quantum algorithm can find f's period using fewer than k queries."

#### The Bit-Flip Oracle

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Bit-Flip Oracle (Standard Oracle)</strong></p>
<p>The bit-flip oracle O_f for a function f: {0,1}ⁿ → {0,1} acts on an (n+1)-qubit register as:</p>
<p>When b=0: O_f|x⟩|0⟩ = |x⟩|f(x)⟩ — the oracle writes f(x) into the ancilla.</p>
<p>O_f is a unitary operation: applying it twice returns the original state (O_f² = I), since b⊕f(x)⊕f(x) = b. It maps computational basis states to computational basis states without superposition.</p>
</div>

#### The Phase Oracle

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Phase Oracle</strong></p>
<p>The phase oracle O_f for a function f: {0,1}ⁿ → {0,1} acts on the n-qubit input register alone:</p>
<p>The phase oracle marks solution states with a negative amplitude — it does not move them to a different basis state. This is the form used in Grover's algorithm and the Deutsch-Jozsa algorithm. It requires no ancilla qubit in the circuit description, though one is used in the implementation.</p>
</div>

<figure class="book-figure">
<img src="content/images/image66.png" alt="Figure 5.1: Quantum oracle models. Left: bit-flip oracle — target qubit b becomes b⊕f(x), used to compute f(x) into a register. Centre: phase oracle — applies phase (−1)^f(x) to the input register, with no ancilla. Right: phase kickback mechanism — initialising the target in |−⟩ and applying the bit-flip oracle produces the phase oracle action automatically.">
<figcaption>Figure 5.1: Quantum oracle models. Left: bit-flip oracle — target qubit b becomes b⊕f(x), used to compute f(x) into a register. Centre: phase oracle — applies phase (−1)^f(x) to the input register, with no ancilla. Right: phase kickback mechanism — initialising the target in |−⟩ and applying the bit-flip oracle produces the phase oracle action automatically.</figcaption>
</figure>

### 5.1.2 Phase Kickback: The Engine of Oracle Algorithms

Phase kickback is the quantum mechanical mechanism that converts a bit-flip oracle into a phase oracle. It is one of the most important and non-classical effects in quantum computing, and it underlies virtually every quantum algorithm that uses an oracle.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Phase Kickback Theorem</strong></p>
<p>Let O_f be the bit-flip oracle: O_f|x⟩|b⟩ = |x⟩|b⊕f(x)⟩. Initialise the target qubit in |−⟩ = (|0⟩−|1⟩)/√2. Then:</p>
<p>Proof: The bit-flip oracle acts on |x⟩(|0⟩−|1⟩)/√2 as:</p>
<p>O_f|x⟩(|0⟩−|1⟩)/√2 = |x⟩(|0⊕f(x)⟩−|1⊕f(x)⟩)/√2</p>
<p>Case f(x)=0: |x⟩(|0⟩−|1⟩)/√2 = |x⟩|−⟩ = (−1)⁰|x⟩|−⟩   ✓</p>
<p>Case f(x)=1: |x⟩(|1⟩−|0⟩)/√2 = −|x⟩|−⟩ = (−1)¹|x⟩|−⟩  ✓</p>
<p>In both cases the target qubit |−⟩ is unchanged — the phase (−1)^f(x) "kicks back" into the amplitude of the input register. The ancilla is separable from the input and can be discarded.</p>
</div>

The physical intuition: |−⟩ is an eigenstate of the X gate (X|−⟩ = −|−⟩). When the oracle applies X to the target (when f(x)=1), X acting on its own eigenstate |−⟩ produces the eigenvalue −1 as a phase factor on the entire state. This eigenvalue "kicks back" into the input register as a global phase relative to the f(x)=0 terms — making it a relative (and hence observable) phase.

This is why all oracle algorithms begin with the same two-step preparation: (1) put the input register in uniform superposition with H^⊗n, (2) put the target qubit in |−⟩ = H|1⟩. After these two steps, a single application of the bit-flip oracle simultaneously evaluates f on all 2ⁿ inputs and encodes the results as phases — all in one quantum operation.

### 5.1.3 Deutsch's Algorithm: The First Quantum Speedup

#### The Problem

Deutsch's algorithm (David Deutsch, 1985) is the simplest quantum algorithm demonstrating a speedup over the best classical algorithm. The problem: given an oracle for a function f: {0,1} → {0,1}, determine whether f is constant (f(0)=f(1)) or balanced (f(0)≠f(1)). Classically, you must query f twice (once for f(0), once for f(1)) — one query cannot distinguish the four possible functions. Deutsch's algorithm decides with one query.

#### The Circuit

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>Initial state: |0⟩|1⟩</p>
<p>After H⊗H:  (|0⟩+|1⟩)/√2 ⊗ (|0⟩−|1⟩)/√2 = |+⟩|−⟩</p>
<p>After O_f:  (−1)^f(0)|0⟩|−⟩/√2 + (−1)^f(1)|1⟩|−⟩/√2  [by kickback]</p>
<p>= (−1)^f(0) [(|0⟩ + (−1)^(f(0)⊕f(1))|1⟩)/√2] |−⟩</p>
<p>After H on first qubit and measuring:</p>
<p>If f constant: f(0)⊕f(1)=0 → |+⟩ → H → |0⟩  [measure 0]</p>
<p>If f balanced: f(0)⊕f(1)=1 → |−⟩ → H → |1⟩  [measure 1]</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 5.1: Full Deutsch Algorithm for f(x) = x (balanced)</strong></p>
<p>Problem: Apply Deutsch's algorithm to f(x) = x (balanced: f(0)=0, f(1)=1). Trace through all four steps.</p>
<p><strong>Step 1 — Initial state: |ψ₀⟩ = |0⟩|1⟩</strong></p>
<p><strong>Step 2 — Apply H⊗H:</strong></p>
<p>|ψ₁⟩ = H|0⟩ ⊗ H|1⟩ = [(|0⟩+|1⟩)/√2] ⊗ [(|0⟩−|1⟩)/√2]</p>
<p>= (1/2)(|00⟩ − |01⟩ + |10⟩ − |11⟩)</p>
<p><strong>Step 3 — Apply bit-flip oracle O_f (f(x)=x means O_f|x,b⟩=|x,b⊕x⟩):</strong></p>
<p>|ψ₂⟩ = (1/2)(|0,0⊕0⟩ − |0,1⊕0⟩ + |1,0⊕1⟩ − |1,1⊕1⟩)</p>
<p>= (1/2)(|00⟩ − |01⟩ + |11⟩ − |10⟩)</p>
<p>= (1/2)(|0⟩(|0⟩−|1⟩) − |1⟩(|0⟩−|1⟩))</p>
<p>= [(|0⟩−|1⟩)/√2] ⊗ [(|0⟩−|1⟩)/√2] = |−⟩|−⟩</p>
<p>(−1)^f(0) factor: (−1)^0 = 1 on |0⟩; (−1)^f(1) = (−1)^1 = −1 on |1⟩ ✓</p>
<p><strong>Step 4 — Apply H to first qubit:</strong></p>
<p>H|−⟩ = H[(|0⟩−|1⟩)/√2] = (H|0⟩−H|1⟩)/√2 = (|+⟩−|−⟩)/√2 · √2 = |1⟩</p>
<p>Wait — more carefully: H|−⟩ = |1⟩ (since H maps |−⟩ → |1⟩)</p>
<p>Measurement: first qubit is |1⟩ → outcome = 1 → f is balanced. ✓</p>
<p>Conclusion: one oracle query determined that f(x)=x is balanced. Classical minimum: 2 queries.</p>
</div>

### 5.1.4 Deutsch-Jozsa Algorithm: Exponential Speedup

#### Generalising to n Bits

The Deutsch-Jozsa algorithm (Deutsch and Jozsa, 1992, improved by Cleve et al. 1998) is the n-qubit generalisation of Deutsch's algorithm. The problem: given an oracle for f: {0,1}ⁿ → {0,1}, where f is promised to be either constant (all 0s or all 1s) or balanced (exactly half the inputs map to 0, half to 1), determine which case holds.

Classical deterministic complexity: in the worst case, you must query 2^(n-1)+1 inputs before being certain. A randomised algorithm can distinguish with high probability using O(1) queries (with small error probability). Quantum algorithm: exactly 1 query — and the answer is certain, not probabilistic.

#### The n-Qubit Algorithm

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Deutsch-Jozsa Algorithm</strong></p>
<p>Input: n-qubit query register initialised to |0⟩^n; 1-qubit ancilla initialised to |1⟩.</p>
<p>Step 1: Apply H^⊗n to query register and H to ancilla:</p>
<p>|0⟩^n|1⟩ → (1/√2^n) Σ_{x∈{0,1}^n} |x⟩ ⊗ |−⟩</p>
<p>Step 2: Apply bit-flip oracle O_f (phase kickback converts to phase oracle):</p>
<p>→ (1/√2^n) Σ_{x} (−1)^f(x)|x⟩ ⊗ |−⟩</p>
<p>Step 3: Apply H^⊗n to query register (discard ancilla):</p>
<p>→ Σ_{z∈{0,1}^n} [ (1/2^n) Σ_x (−1)^(f(x)+x·z) ] |z⟩</p>
<p>Step 4: Measure query register.</p>
<p>Proof: If f is constant, (−1)^f(x) = ±1 for all x, so the sum is ±2^n, and its squared magnitude divided by 2^(2n) is 1. If f is balanced, exactly half the terms are +1 and half are −1, so the sum is 0.</p>
</div>

<figure class="book-figure">
<img src="content/images/image67.png" alt="Figure 5.2: Deutsch-Jozsa algorithm. Left: the n-qubit circuit — H gates create superposition, the phase oracle applies (−1)^f(x), H gates create interference. Right: amplitude analysis for n=3 — constant f (blue) concentrates all amplitude at |000⟩; balanced f (green) distributes amplitude elsewhere with zero probability at |000⟩.">
<figcaption>Figure 5.2: Deutsch-Jozsa algorithm. Left: the n-qubit circuit — H gates create superposition, the phase oracle applies (−1)^f(x), H gates create interference. Right: amplitude analysis for n=3 — constant f (blue) concentrates all amplitude at |000⟩; balanced f (green) distributes amplitude elsewhere with zero probability at |000⟩.</figcaption>
</figure>

The key insight is interference: the final H^⊗n layer is a quantum interference step that either constructively combines all amplitudes at |0⟩^n (constant case) or destructively cancels all amplitude at |0⟩^n (balanced case). No intermediate measurement is needed — the entire computation is coherent.

<div class="box box-example">
<p class="box-title"><strong>Example 5.2: Deutsch-Jozsa for n=2, f(x) = x₁ ⊕ x₂ (balanced)</strong></p>
<p>Problem: Apply D-J algorithm to f(00)=0, f(01)=1, f(10)=1, f(11)=0 (balanced).</p>
<p><strong>Step 1 — After H^⊗2 on |00⟩: uniform superposition over 4 states</strong></p>
<p>|ψ₁⟩ = (|00⟩ + |01⟩ + |10⟩ + |11⟩)/2 ⊗ |−⟩</p>
<p><strong>Step 2 — After phase oracle (multiply by (−1)^f(x) per basis state):</strong></p>
<p>|ψ₂⟩ = (|00⟩ − |01⟩ − |10⟩ + |11⟩)/2 ⊗ |−⟩</p>
<p>[since f(00)=0,f(01)=1,f(10)=1,f(11)=0: signs are +−−+]</p>
<p><strong>Step 3 — After H^⊗2 on query register:</strong></p>
<p>H^⊗2 · (|00⟩−|01⟩−|10⟩+|11⟩)/2</p>
<p>H|0⟩=(|0⟩+|1⟩)/√2, H|1⟩=(|0⟩−|1⟩)/√2</p>
<p>= H⊗H · (1/2)[(|00⟩−|01⟩) − (|10⟩−|11⟩)]</p>
<p>= (1/2)[(|0⟩+|1⟩)(|0⟩−|1⟩)/√2·√2·2 − ...]   [careful calculation]</p>
<p>Direct: apply H⊗H to each basis state and collect:</p>
<p>Coefficient of |00⟩ after H^⊗2: (1/4)(1·1 − 1·(−1) − (−1)·1 + (−1)·(−1)) = (1+1+1+1)/4 = 0</p>
<p>Coefficient of |11⟩ after H^⊗2: (1/4)(1·1 − 1·(−1)·(−1) − ...) = compute to find 1</p>
<p>[More carefully: H^⊗2 maps basis state |z⟩ to Σ_x (−1)^(x·z)|x⟩/2; so amplitude at |00⟩ is (1/4)Σ_x(−1)^f(x) = (1+(-1)+(-1)+1)/4 = 0] ✓</p>
<p>Measurement: P(|00⟩) = 0. We do NOT get |00⟩, confirming f is balanced. ✓</p>
</div>

```python
# ─────────────────────────────────────────────────────────────────────
# Code 5.1: Deutsch-Jozsa Algorithm in Qiskit
# Tests both constant and balanced oracles for n=3
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator

def deutsch_jozsa(n, oracle_type="balanced", balanced_pattern=None):
    """
    n: number of input qubits
    oracle_type: "constant_0", "constant_1", or "balanced"
    balanced_pattern: list of n bits s where f(x) = s·x mod 2
    Returns: "constant" or "balanced" based on measurement
    """
    qc = QuantumCircuit(n+1, n)

    # ── Step 1: Prepare ancilla in |1⟩, then H on all ─────────────────
    qc.x(n)              # ancilla: |0⟩ → |1⟩
    qc.h(range(n+1))     # H on query register + ancilla → |+⟩^n|−⟩

    # ── Step 2: Apply oracle ───────────────────────────────────────────
    if oracle_type == "constant_1":
        qc.x(n)          # constant f=1: flip ancilla (adds global −1 phase)
    elif oracle_type == "balanced":
        pattern = balanced_pattern or [1]*n  # default: f(x) = x₀ (first bit)
        for i, bit in enumerate(pattern):
            if bit:
                qc.cx(i, n)  # CNOT when s_i = 1: implements f(x) = s·x mod 2
    # constant_0: oracle does nothing (identity)

    # ── Step 3: H on query register ────────────────────────────────────
    qc.h(range(n))

    # ── Step 4: Measure query register ─────────────────────────────────
    qc.measure(range(n), range(n))

    # ── Simulate ───────────────────────────────────────────────────────
    sim = AerSimulator()
    tqc = transpile(qc, sim)
    result = sim.run(tqc, shots=1).result()
    counts = result.get_counts()
    outcome = list(counts.keys())[0]

    if outcome == "0" * n:
        return "constant"
    else:
        return "balanced"

# ── Test all cases ────────────────────────────────────────────────────
n = 3
print(f"n={n} qubits:")
print(f"  Constant-0 oracle: {deutsch_jozsa(n, oracle_type=chr(99)+chr(111)+chr(110)+chr(115)+chr(116)+chr(97)+chr(110)+chr(116)+chr(95)+chr(48))}")
print(f"  Constant-1 oracle: {deutsch_jozsa(n, oracle_type=chr(99)+chr(111)+chr(110)+chr(115)+chr(116)+chr(97)+chr(110)+chr(116)+chr(95)+chr(49))}")
print(f"  Balanced (s=110): {deutsch_jozsa(n, oracle_type=chr(98)+chr(97)+chr(108)+chr(97)+chr(110)+chr(99)+chr(101)+chr(100), balanced_pattern=[1,1,0])}")

# Simpler call syntax:
print(deutsch_jozsa(3, "constant_0"))    # -> constant
print(deutsch_jozsa(3, "balanced", [1,0,1]))  # -> balanced
```

### 5.1.5 Bernstein-Vazirani Algorithm

#### The Problem: Finding a Hidden Bit String

The Bernstein-Vazirani problem (Bernstein and Vazirani, 1993) asks: given an oracle for f: {0,1}ⁿ → {0,1} where f(x) = s·x mod 2 = (s₁x₁ + s₂x₂ + ... + sₙxₙ) mod 2 for some unknown string s ∈ {0,1}ⁿ, find s.

This is an inner-product function: f(x) returns the dot product of x with the hidden string s modulo 2. Note that knowing f(x) for any particular x only reveals one bit of information about s — the bit s·x. To determine all n bits of s classically, one must query f with n carefully chosen inputs (the standard basis vectors e₁, e₂, ..., eₙ). So the classical query complexity is exactly n.

Quantum algorithm: exactly 1 query returns s completely.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Bernstein-Vazirani Algorithm</strong></p>
<p>The circuit is identical to Deutsch-Jozsa:</p>
<p>1. Prepare |+⟩^n|−⟩</p>
<p>2. Apply oracle O_f</p>
<p>3. Apply H^⊗n to query register</p>
<p>4. Measure</p>
<p>Analysis: The oracle applies (−1)^(s·x) to basis state |x⟩. After H^⊗n, the amplitude at basis state |z⟩ is:</p>
<p>The measurement outcome is s with certainty — the algorithm outputs the entire hidden string in a single query.</p>
</div>

<figure class="book-figure">
<img src="content/images/image68.png" alt="Figure 5.3: Bernstein-Vazirani and Simon&#x27;s algorithm. Left: BV circuit for hidden string s=1101 — identical structure to Deutsch-Jozsa, but the oracle implements f(x)=s·x mod 2. Output directly reveals s. Right: Simon&#x27;s algorithm structure — two registers, two sets of measurements, requiring n circuit repetitions to find the period s via linear algebra.">
<figcaption>Figure 5.3: Bernstein-Vazirani and Simon&#x27;s algorithm. Left: BV circuit for hidden string s=1101 — identical structure to Deutsch-Jozsa, but the oracle implements f(x)=s·x mod 2. Output directly reveals s. Right: Simon&#x27;s algorithm structure — two registers, two sets of measurements, requiring n circuit repetitions to find the period s via linear algebra.</figcaption>
</figure>

<div class="box box-example">
<p class="box-title"><strong>Example 5.3: Bernstein-Vazirani with s = 101</strong></p>
<p>Problem: Apply BV algorithm to f(x) = 1·x₀ + 0·x₁ + 1·x₂ mod 2 (hidden string s=101).</p>
<p><strong>After H^⊗3 on |000⟩: uniform superposition over 8 states</strong></p>
<p>|ψ₁⟩ = (1/2√2) Σ_{x∈{0,1}³} |x⟩ = (1/2√2)(|000⟩+|001⟩+...+|111⟩)</p>
<p><strong>After phase oracle (applies (−1)^(1·x₀+0·x₁+1·x₂)):</strong></p>
<p>f(000)=0, f(001)=1, f(010)=0, f(011)=1</p>
<p>f(100)=1, f(101)=0, f(110)=1, f(111)=0</p>
<p>|ψ₂⟩ = (1/2√2)(|000⟩−|001⟩+|010⟩−|011⟩−|100⟩+|101⟩−|110⟩+|111⟩)</p>
<p><strong>After H^⊗3:</strong></p>
<p>Amplitude at |z⟩ = (1/8) Σ_x (−1)^((101·x)⊕(z·x)) = (1/8) Σ_x (−1)^(x·(101⊕z))</p>
<p>= 1 if 101⊕z = 000 (i.e., z=101); = 0 otherwise</p>
<p>Measurement: outcome |101⟩ with probability 1. s = 101. ✓</p>
</div>

### 5.1.6 Simon's Algorithm (Overview)

#### The Problem: Finding a Hidden Period

Simon's algorithm (Daniel Simon, 1994) solves a more complex hidden structure problem: given f: {0,1}ⁿ → {0,1}ⁿ that is promised to be periodic with period s (meaning f(x) = f(x⊕s) for all x, where ⊕ is bitwise XOR), find s. This is a "hidden subgroup problem" — one of the most important algorithmic problem classes in quantum computing.

Classical query complexity: Ω(2^(n/2)) queries (birthday paradox bound — you need to find two inputs x₁,x₂ with f(x₁)=f(x₂), which on average requires ~√(2^n) queries). Quantum complexity: O(n) queries and O(n²) classical post-processing.

#### The Quantum Part of Simon's Algorithm

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p><strong>Repeat n times:</strong></p>
<p>1. Start with |0⟩^n|0⟩^n</p>
<p>2. Apply H^⊗n to first register: (1/√2^n) Σ_x |x⟩|0⟩</p>
<p>3. Apply oracle U_f: (1/√2^n) Σ_x |x⟩|f(x)⟩</p>
<p>4. Measure second register → collapses to some value y = f(x₀)</p>
<p>First register collapses to (|x₀⟩ + |x₀⊕s⟩)/√2</p>
<p>5. Apply H^⊗n to first register and measure → get y_i with y_i · s = 0 mod 2</p>
</div>

After n repetitions: we have n linear equations y\_i · s = 0 (mod 2) in n unknowns (the bits of s). Solve this system using Gaussian elimination over GF(2) to find s. With high probability, the n vectors y\_i are linearly independent, giving a unique solution.

Simon's algorithm was historically pivotal: Shor's factoring algorithm is a continuous-group generalisation of Simon's discrete-group algorithm. Understanding Simon's is a direct path to understanding Shor's.

## 5.2 The Quantum Fourier Transform

The Quantum Fourier Transform (QFT) is the quantum analogue of the Discrete Fourier Transform (DFT). It is the single most powerful subroutine in quantum computing: it underlies Shor's factoring algorithm, Quantum Phase Estimation, and the Hidden Subgroup Problem framework. Understanding the QFT deeply — not just as a circuit diagram but as a mathematical transformation with geometric meaning — is essential for every quantum algorithm researcher.

### 5.2.1 Classical DFT vs Quantum QFT

#### The Discrete Fourier Transform

The classical DFT transforms a vector of N complex numbers (a\_0, a\_1, ..., a\_{N-1}) into another vector (A\_0, A\_1, ..., A\_{N-1}):

<div class="box box-equation">
<p><strong>Equation 5.6</strong></p>
<p><strong>A_k = (1/√N) Σ_{j=0}^{N-1} a_j · e^(2πijk/N)   for k = 0, 1, ..., N-1</strong></p>
<p>Equivalently: the DFT matrix F_N has entry (F_N)_{kj} = e^(2πijk/N)/√N</p>
</div>

The classical Fast Fourier Transform (FFT) computes the DFT of N=2ⁿ points in O(N log N) = O(n·2ⁿ) arithmetic operations. This is the fundamental algorithm of signal processing.

#### The Quantum Fourier Transform

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: QFT Definition</strong></p>
<p>The QFT on N=2ⁿ dimensional Hilbert space acts on basis states as:</p>
<p>By linearity, the QFT on a general state |ψ⟩ = Σ_j a_j |j⟩ gives:</p>
<p>QFT|ψ⟩ = Σ_k A_k |k⟩    where A_k = (1/√N) Σ_j a_j e^(2πijk/N)</p>
<p>The amplitudes of the output are the DFT of the amplitudes of the input. The QFT is the unitary matrix F_N — the same matrix as the classical DFT, but acting on a quantum state vector rather than a classical data array.</p>
</div>

#### Critical Distinction: QFT vs Classical FFT

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: QFT Does NOT Compute DFT Faster</strong></p>
<p>The QFT is NOT a faster way to compute the DFT of a classical signal. The reasons are subtle but important:</p>
<p>The QFT acts on quantum amplitudes, which cannot be directly read out. Measuring the output state gives only one sample from the output distribution — not all N output values.</p>
<p>Classically loading N input values into a quantum state requires O(N) operations (no speedup in data preparation).</p>
<p>The speedup comes from using the QFT as a subroutine in a larger quantum algorithm where the input state is prepared efficiently and the output is used in a way that extracts useful information from a single measurement.</p>
<p>The QFT is useful because it efficiently computes a transformation on quantum amplitudes that would require O(N) classical operations — but only when those amplitudes are already in a quantum computer and only when we need specific properties of the output, not all N output values.</p>
</div>

<figure class="book-figure">
<img src="content/images/image69.png" alt="Figure 5.4: QFT circuit and complexity comparison. Left: 3-qubit QFT circuit using Hadamard gates and controlled phase rotations R_k = diag(1, e^(2πi/2^k)). Right: complexity comparison — classical FFT requires O(N·log N) operations where N=2^n, while the QFT uses only O(n²) gates — an exponential gap.">
<figcaption>Figure 5.4: QFT circuit and complexity comparison. Left: 3-qubit QFT circuit using Hadamard gates and controlled phase rotations R_k = diag(1, e^(2πi/2^k)). Right: complexity comparison — classical FFT requires O(N·log N) operations where N=2^n, while the QFT uses only O(n²) gates — an exponential gap.</figcaption>
</figure>

### 5.2.2 QFT Circuit: O(n²) Controlled Rotations

#### Derivation of the QFT Circuit

To derive the QFT circuit, we rewrite the QFT in terms of single-qubit operations using the binary fraction representation. For an n-bit input j = j\_{n-1}j\_{n-2}...j\_1j\_0 (binary), define the binary fraction 0.j\_l j\_{l+1}...j\_m = j\_l/2 + j\_{l+1}/4 + ... + j\_m/2^(m-l+1). Then:

<div class="box box-equation">
<p><strong>Equation 5.8</strong></p>
<p>QFT|j_{n-1}...j_1j_0⟩ =</p>
<p>(|0⟩+e^(2πi·0.j_0)|1⟩)/√2 ⊗ (|0⟩+e^(2πi·0.j_1j_0)|1⟩)/√2 ⊗ ... ⊗ (|0⟩+e^(2πi·0.j_{n-1}...j_0)|1⟩)/√2</p>
</div>

This product form reveals the circuit structure: each output qubit k is a single-qubit state that depends on the input bits through controlled phase rotations. The controlled rotation gate R\_k applies a phase of e^(2πi/2^k) to |1⟩:

<div class="box box-equation">
<p><strong>Equation 5.9</strong></p>
<p><strong>R_k = [[1, 0], [0, e^(2πi/2^k)]]    = [[1, 0], [0, e^(iπ/2^{k-1})]]</strong></p>
<p>Special cases: R_1 = Z (phase π),  R_2 = S (phase π/2),  R_3 = T (phase π/4)</p>
</div>

#### The QFT Algorithm

For an n-qubit register, the QFT requires n(n-1)/2 controlled-R\_k gates plus n Hadamard gates:

- For qubit j (from j=1 to j=n): apply H to qubit j; then apply controlled-R\_k from qubit j+k-1 controlling qubit j, for k=2,3,...,n-j+1.

- Total gates: n H-gates + n(n-1)/2 controlled rotations = O(n²) gates.

- Compare: classical FFT on N=2ⁿ points needs O(n·2ⁿ) operations — exponentially more.

However, note the caveat from Section 5.2.1: this O(n²) vs O(n·2ⁿ) comparison is valid only for the gate count of the circuit. The QFT cannot be directly used to speed up classical DFT computation.

### 5.2.3 Bit-Reversal and the SWAP Network

After the controlled rotation network, the QFT output is in bit-reversed order: the most significant qubit (q\_{n-1}) holds the least significant bit of the output. This is because the product-form derivation naturally produces the output with the bits in reverse order.

To correct this, a SWAP network is applied after the rotations: swap qubit 1 with qubit n, qubit 2 with qubit n-1, and so on. This requires ⌊n/2⌋ SWAP gates. Each SWAP costs 3 CNOTs, adding O(n) CNOTs total — negligible compared to the O(n²) rotation gates.

<div class="box box-equation">
<p><strong>Equation 5.10</strong></p>
<p>Total QFT circuit: n H + n(n-1)/2 controlled-R_k + floor(n/2) SWAPs</p>
<p>Total gates: O(n²)   Total depth: O(n) if parallelised, O(n²) sequentially</p>
</div>

### 5.2.4 Qiskit Implementation from Scratch

```python
# ─────────────────────────────────────────────────────────────────────
# Code 5.2: Quantum Fourier Transform — Implementation from Scratch
# Builds QFT without using Qiskit's built-in QFT library
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit.quantum_info import Statevector
import numpy as np

def qft_rotations(qc, n):
    """Apply QFT rotation network to qubits 0..n-1 (without SWAP network)."""
    if n == 0:
        return qc
    n -= 1  # act on qubit n (0-indexed)
    qc.h(n)  # Hadamard on qubit n
    for qubit in range(n):
        # Controlled R_k: control = qubit, target = n
        k = n - qubit + 1  # phase = 2*pi / 2^k
        qc.cp(2*np.pi / 2**k, qubit, n)  # cp = controlled-phase gate
    qft_rotations(qc, n)  # recurse on remaining qubits
    return qc

def swap_registers(qc, n):
    """Apply SWAP network for bit-reversal."""
    for qubit in range(n//2):
        qc.swap(qubit, n-qubit-1)
    return qc

def qft(qc, n):
    """Full QFT on first n qubits of circuit qc."""
    qft_rotations(qc, n)
    swap_registers(qc, n)
    return qc

# ── Test: QFT on |001⟩ (j=1) ────────────────────────────────────────
n = 3
qc = QuantumCircuit(n)
qc.x(0)   # prepare |001⟩ = |j=1⟩ (LSB first in Qiskit)
qft(qc, n)

state = Statevector.from_instruction(qc)
print("QFT|1> amplitudes (n=3, N=8):")
for k, amp in enumerate(state):
    if abs(amp) > 1e-10:
        phase_angle = np.angle(amp)
        print(f"  |{k:03b}> = {abs(amp):.4f} * exp({phase_angle:.4f}i)")
# Expected: all amplitudes equal 1/√8 = 0.3536, phases = 2πi*k*j/N = 2πi*k/8

# ── Verify against Qiskit's built-in QFT ──────────────────────────────
from qiskit.circuit.library import QFT
qc_builtin = QuantumCircuit(n)
qc_builtin.x(0)
qc_builtin.append(QFT(n), range(n))

state_builtin = Statevector.from_instruction(qc_builtin)
fidelity = abs(state.inner(state_builtin))**2
print(f"\nFidelity between scratch QFT and built-in QFT: {fidelity:.6f}")
# Expected: fidelity = 1.000000
```

### 5.2.5 QFT on Periodic States and Period Finding

#### QFT Reveals Hidden Periodicity

The most powerful application of the QFT is period finding: given a quantum state with a periodic amplitude pattern, the QFT transforms it into a state concentrated at frequencies that reveal the period. This is the mathematical heart of Shor's algorithm.

Consider a quantum state |ψ⟩ = (1/√(N/r)) Σ\_{k=0}^{N/r-1} |kr + x₀⟩ — a uniform superposition over multiples of r, starting at offset x₀. The QFT of this state is:

<div class="box box-equation">
<p><strong>Key Equation — Equation 5.1</strong></p>
<p>QFT|ψ⟩ = (1/√(N/r)) Σ_{j} (1/√N) Σ_k e^(2πi(kr+x₀)j/N) |j⟩</p>
<p><strong>= (e^(2πix₀j/N)/√r) Σ_{m=0}^{r-1} |mN/r⟩    (non-zero only at j = mN/r)</strong></p>
<p>The QFT output has non-zero amplitude only at multiples of N/r.</p>
<p>Measuring the QFT output gives j = mN/r with equal probability for m=0,1,...,r-1.</p>
<p>From j and N, recover r using continued fractions: r = N/gcd(j, N) or continued fraction expansion of j/N.</p>
</div>

<figure class="book-figure">
<img src="content/images/image70.png" alt="Figure 5.5: QFT on periodic states. Top: input states periodic with period r=8 (left) and r=16 (right) in a 64-state register. Bottom: after QFT, probability is concentrated at frequency indices that are multiples of N/r = 8 and 4 respectively, revealing the period from the peak positions.">
<figcaption>Figure 5.5: QFT on periodic states. Top: input states periodic with period r=8 (left) and r=16 (right) in a 64-state register. Bottom: after QFT, probability is concentrated at frequency indices that are multiples of N/r = 8 and 4 respectively, revealing the period from the peak positions.</figcaption>
</figure>

#### Connection to Shor's Algorithm

Shor's algorithm (1994) reduces integer factorisation to the period-finding problem. Given a number N to factor and a coprime a<N, the function f(x) = aˣ mod N is periodic with some period r (called the order of a modulo N). Finding r gives factors of N via gcd(a^(r/2) ± 1, N). The quantum speedup comes from using the QFT to find r exponentially faster than any classical algorithm.

The period-finding subroutine of Shor's algorithm is exactly QPE (Section 5.3) applied to the unitary U: |x⟩ → |ax mod N⟩. This is why QPE is called the "quantum computational primitive" — it is the module that gives quantum computers their exponential advantage for algebraic problems.

## 5.3 Quantum Phase Estimation

Quantum Phase Estimation (QPE) is one of the most important subroutines in all of quantum computing. Given a unitary operator U and one of its eigenstates |u⟩, QPE estimates the phase φ in the eigenvalue equation U|u⟩ = e^(2πiφ)|u⟩ to t bits of precision using t ancilla qubits. QPE is the computational core of Shor's algorithm, the HHL linear systems algorithm, quantum chemistry (VQE-QPE hybrid), and quantum simulation.

### 5.3.1 The Eigenphase Problem

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Eigenphase Estimation Problem</strong></p>
<p>Given: A unitary operator U (implementable as a quantum circuit) and an eigenstate |u⟩ such that:</p>
<p>Goal: Estimate φ to t bits of precision, i.e., find the best t-bit approximation φ̃ ≈ φ.</p>
<p>Resource requirement: t ancilla (counting register) qubits + 1 eigenstate register (size depends on U).</p>
<p>Precision: the best t-bit approximation has error |φ − φ̃| ≤ 1/2^t.</p>
<p>Probability: with t+O(1) ancilla, the algorithm succeeds with probability &gt; 8/π² ≈ 81%.</p>
</div>

Why is this useful? Many important physical quantities are eigenvalues of Hermitian operators: the energy of a molecule is an eigenvalue of its Hamiltonian H; the phase of a quantum gate is an eigenvalue of the gate unitary. QPE computes these eigenvalues to exponential precision in the number of ancilla qubits — a task that is classically exponentially hard for large systems.

### 5.3.2 QPE Circuit: Controlled-U^k Gates + QFT†

#### The QPE Algorithm

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: QPE Circuit Construction</strong></p>
<p>The QPE circuit has three stages:</p>
<p>Stage 1 — Prepare the counting register in uniform superposition:</p>
<p>H^⊗t |0⟩^t = (1/√2^t) Σ_{k=0}^{2^t-1} |k⟩</p>
<p>Stage 2 — Apply controlled-U^(2^j) gates for j=0,1,...,t-1:</p>
<p>Ancilla qubit j controls U^(2^j) on the eigenstate register.</p>
<p>Because U|u⟩ = e^(2πiφ)|u⟩, we have U^(2^j)|u⟩ = e^(2πi·2^j·φ)|u⟩.</p>
<p>After all controlled gates, the counting register state is:</p>
<p><strong>(1/√2^t) Σ_{k=0}^{2^t-1} e^(2πiφk) |k⟩  =  QFT|φ̃⟩ (approximately)</strong></p>
<p>Stage 3 — Apply QFT† (inverse QFT) to the counting register:</p>
<p>QFT†(QFT|φ̃⟩) = |φ̃⟩  →  measuring the counting register gives φ̃ ≈ φ</p>
</div>

<figure class="book-figure">
<img src="content/images/image71.png" alt="Figure 5.6: Quantum Phase Estimation circuit and precision analysis. Left: QPE circuit with t ancilla qubits — each ancilla qubit j controls U^(2^j) on the eigenstate. After QFT†, measurement gives a t-bit estimate of φ. Right: precision scales as 1/2^t — adding one ancilla qubit doubles the precision, so t=14 qubits achieves precision better than 10^-4.">
<figcaption>Figure 5.6: Quantum Phase Estimation circuit and precision analysis. Left: QPE circuit with t ancilla qubits — each ancilla qubit j controls U^(2^j) on the eigenstate. After QFT†, measurement gives a t-bit estimate of φ. Right: precision scales as 1/2^t — adding one ancilla qubit doubles the precision, so t=14 qubits achieves precision better than 10^-4.</figcaption>
</figure>

#### Why the Circuit Works: Detailed Analysis

Let φ = 0.φ₁φ₂...φₜ... in binary (so φ = φ₁/2 + φ₂/4 + ...). After Stage 2, the counting register is in state:

<div class="box box-equation">
<p><strong>Equation 5.12</strong></p>
<p>(1/√2^t) Σ_{k=0}^{2^t-1} e^(2πiφk) |k⟩</p>
<p>= (|0⟩+e^(2πi·2^{t-1}·φ}|1⟩)/√2 ⊗ (|0⟩+e^(2πi·2^{t-2}·φ}|1⟩)/√2 ⊗ ... ⊗ (|0⟩+e^(2πiφ)|1⟩)/√2</p>
</div>

Comparing this with the QFT output form (Section 5.2.2): this is exactly QFT|2^t φ⟩ (if φ has an exact t-bit representation). Therefore, applying QFT† recovers |2^t φ⟩ exactly, and measuring gives the t-bit binary representation of φ.

If φ does not have an exact t-bit representation, QPE gives the best t-bit approximation with high probability. The error probability decreases as we add more ancilla qubits.

#### Implementing Controlled-U^k

A key implementation challenge in QPE is implementing the controlled-U^(2^k) gates. For each ancilla qubit k, we need a circuit that applies U^(2^k) to the eigenstate register conditioned on the ancilla being |1⟩. Several approaches:

- Direct construction: for simple U (like phase gates on a single qubit), U^(2^k) can be computed in closed form and implemented as a standard gate.

- Repeated squaring: compute U, U² = U·U, U⁴ = U²·U², ... by repeated matrix squaring. This gives U^(2^k) using k squarings, each of which can be compiled as a circuit.

- Quantum modular exponentiation: for U: |x⟩ → |ax mod N⟩ in Shor's algorithm, use classical modular exponentiation algorithms adapted to reversible quantum circuits.

```python
# ─────────────────────────────────────────────────────────────────────
# Code 5.3: Quantum Phase Estimation — Complete Implementation
# Estimates eigenphase of T gate: T|1> = e^(i*pi/4)|1> => phi = 1/8
# ─────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit.circuit.library import QFT
from qiskit_aer import AerSimulator
from qiskit.quantum_info import Statevector
import numpy as np

def qpe_circuit(unitary_gate, eigenstate_prep, t):
    """
    Build QPE circuit.
    unitary_gate: a 1-qubit gate U
    eigenstate_prep: circuit preparing the eigenstate of U
    t: number of ancilla (counting) qubits
    """
    n_eigenstate = eigenstate_prep.num_qubits
    qc = QuantumCircuit(t + n_eigenstate, t)

    # Eigenstate qubits: 0..n_eigenstate-1
    # Ancilla (counting) qubits: n_eigenstate..n_eigenstate+t-1
    eig_qubits  = list(range(n_eigenstate))
    count_qubits = list(range(n_eigenstate, n_eigenstate + t))

    # Stage 1: Prepare eigenstate and counting register
    qc.compose(eigenstate_prep, qubits=eig_qubits, inplace=True)
    qc.h(count_qubits)   # uniform superposition

    # Stage 2: Controlled-U^(2^k) gates
    for k, ctrl in enumerate(count_qubits):
        # Apply U^(2^k) controlled on count_qubits[k]
        repetitions = 2**k
        for _ in range(repetitions):
            # Append controlled version of U
            qc.append(unitary_gate.control(1), [ctrl] + eig_qubits)

    # Stage 3: Inverse QFT on counting register
    qc.append(QFT(t, inverse=True), count_qubits)

    # Measure counting register
    qc.measure(count_qubits, range(t))
    return qc

# ── Estimate phase of T gate ──────────────────────────────────────────
# T|1> = e^(i*pi/4)|1> so phi = (pi/4)/(2*pi) = 1/8 = 0.125
# Binary: phi = 0.001  (so t=3 ancilla should give exact result)

from qiskit.circuit.library import TGate
T_gate = TGate()   # the T gate

# Prepare eigenstate |1> (T gate eigenstate)
eigenstate_circuit = QuantumCircuit(1)
eigenstate_circuit.x(0)   # |1>

t_ancilla = 5   # 5-bit precision: can resolve 1/32 = 0.03125
qc_qpe = qpe_circuit(T_gate, eigenstate_circuit, t_ancilla)

# Simulate
sim = AerSimulator()
tqc = transpile(qc_qpe, sim)
result = sim.run(tqc, shots=1000).result()
counts = result.get_counts()

# Decode most likely outcome
top_outcome = max(counts, key=counts.get)
phi_estimate = int(top_outcome, 2) / 2**t_ancilla
print(f'Most frequent: {top_outcome} -> phi_estimate = {phi_estimate}')
print(f'True phi = 1/8 = {1/8}')
print(f'Error = {abs(phi_estimate - 1/8):.6f}')
# Expected: phi_estimate = 0.125 (= 4/32), error = 0
```

### 5.3.3 Precision, Qubit Count, and Error Analysis

#### The Precision-Resource Trade-off

The fundamental trade-off in QPE: more ancilla qubits give higher precision, but require more controlled-U operations (and hence longer circuits). Specifically:

- t ancilla qubits → t-bit binary approximation of φ → precision 1/2^t

- Probability of success (finding the best t-bit approximation): P ≥ 1 − 1/(2(ε−1)) for any ε > 0, where we use t + ⌈log₂(2+1/(2ε))⌉ ancilla to achieve error at most ε/2^t

- For 99% success probability: use t + 4 ancilla qubits (the extra 4 provide the probability buffer)

- Controlled-U^(2^k) cost: implementing U^(2^k) requires 2^k applications of U sequentially, so total controlled-U operations = Σ\_{k=0}^{t-1} 2^k = 2^t − 1 = O(2^t)

The O(2^t) cost of implementing all controlled-U^k operations is the bottleneck: it grows exponentially in the number of ancilla qubits. For QPE to be efficient, the controlled-U operations must be efficiently implementable (polynomial in the system size). This is true for phase gates (trivially), for modular arithmetic (Shor's algorithm), and for Hamiltonian simulation (quantum chemistry).

<div class="box box-example">
<p class="box-title"><strong>Example 5.4: QPE for a Phase Gate — Hand Calculation</strong></p>
<p>Problem: Apply QPE with t=3 ancilla to the gate U = Rz(π/2) = diag(1, e^(iπ/2)) = diag(1, i). The eigenstate is |1⟩ with eigenvalue e^(iπ/2) = e^(2πi·(1/4)). So φ = 1/4 = 0.01 in binary. Predict the measurement outcome.</p>
<p><strong>Stage 1 — Initial state after H^⊗3 on ancilla and |1⟩ on eigenstate:</strong></p>
<p>(1/2√2)(|000⟩+|001⟩+|010⟩+|011⟩+|100⟩+|101⟩+|110⟩+|111⟩)|1⟩</p>
<p><strong>Stage 2 — After controlled-U^(2^k) gates:</strong></p>
<p>U^1|1⟩ = e^(iπ/2)|1⟩ = i|1⟩; U^2|1⟩ = e^(iπ)|1⟩ = −|1⟩; U^4|1⟩ = e^(i2π)|1⟩ = |1⟩</p>
<p>Ancilla register state: (1/2√2) Σ_{k=0}^{7} e^(2πi·k/4)|k⟩ ⊗ |1⟩</p>
<p>(since φ=1/4, e^(2πiφk) = e^(2πi·k/4) = i^k)</p>
<p>= (1/2√2)(|0⟩ + i|1⟩ − |2⟩ − i|3⟩ + |4⟩ + i|5⟩ − |6⟩ − i|7⟩) ⊗ |1⟩</p>
<p><strong>Stage 3 — After QFT† on ancilla:</strong></p>
<p>The state (1/2√2) Σ_k i^k |k⟩ = QFT|2⟩ (in 8-point QFT, peak at k=2)</p>
<p>So QFT†[QFT|2⟩] = |2⟩ = |010⟩</p>
<p>Measurement: |010⟩ with probability 1. Decoded: φ̃ = 2/8 = 1/4. ✓</p>
<p>The 3-bit estimate is exact because φ=1/4=0.010 is a 3-bit fraction.</p>
</div>

### 5.3.4 Applications: Quantum Chemistry and Hamiltonian Simulation

#### Quantum Chemistry via QPE

The most commercially important application of QPE is molecular energy estimation. For a molecule with Hamiltonian H (a Hermitian matrix whose eigenvalues are the molecular energy levels), QPE applied to the unitary U = e^(−iHt) estimates the ground state energy E₀:

<div class="box box-equation">
<p><strong>Equation 5.13</strong></p>
<p><strong>U|E₀⟩ = e^(−iE₀t)|E₀⟩    so eigenphase φ = E₀t/(2π)</strong></p>
<p>QPE with t ancilla gives E₀ to precision 2π/(t·Δt)</p>
<p>where Δt is the simulation time step</p>
</div>

The challenge: preparing the ground state |E₀⟩ (which is typically unknown). In practice, an initial trial state with sufficient overlap with |E₀⟩ is used; the probability of finding the ground energy is equal to the squared overlap. For small molecules, classical methods can prepare reasonable initial states; for large molecules, this remains an active research area.

#### Hamiltonian Simulation

Quantum phase estimation for Hamiltonian simulation uses the Lie-Trotter-Suzuki product formula or more sophisticated quantum simulation algorithms (Qubitization, QSVT) to implement e^(−iHt). The key result:

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: QPE for Drug Discovery and Materials Science</strong></p>
<p>The FeMo cofactor of nitrogenase (the enzyme that fixes atmospheric nitrogen into ammonia) has approximately 50 electrons in its active site. Classical simulation of its ground state energy requires diagonalising a 2^50 × 2^50 matrix — computationally intractable. A quantum computer using QPE with ~100 logical qubits (plus error correction overhead) could compute this energy exactly, potentially enabling the design of room-temperature nitrogen fixation catalysts. Current estimate (Reiher et al., Science 2017): 111 logical qubits needed for active-space FeMo cofactor simulation. With surface code error correction at 10^-3 physical error rate: approximately 400,000 physical qubits. IBM's quantum roadmap targets this scale by 2030.</p>
</div>

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 5: Oracle Algorithms, Quantum Fourier Transform & Quantum Phase Estimation

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Define the bit-flip oracle and phase oracle for a function f: {0,1}^n → {0,1}. Write the mathematical definition of each. How is the phase oracle obtained from the bit-flip oracle using phase kickback?**

**Q2.  State and prove the Phase Kickback theorem. What specific ancilla preparation is required, and what happens physically when f(x) = 1?**

**Q3.  Describe Deutsch's algorithm step by step. Why does it require only 1 oracle query while the classical minimum is 2?**

**Q4.  State the Deutsch-Jozsa problem. Write the general n-qubit circuit and prove that P(measure |0⟩^n) = 1 if and only if f is constant.**

**Q5.  State the Bernstein-Vazirani problem. Prove that the algorithm outputs the hidden string s with certainty using 1 oracle query (show the amplitude calculation at |z⟩ after H^⊗n).**

**Q6.  Describe Simon's algorithm. What is the promised period structure? What is the classical vs quantum query complexity? How is classical post-processing used to extract s?**

**Q7.  Define the Quantum Fourier Transform on n qubits. Write its action on basis state |j⟩. How does its complexity O(n²) compare to classical FFT's O(n·2^n)?**

**Q8.  Derive the QFT product representation and explain how it directly suggests the QFT circuit construction. What is the role of controlled phase rotation gates R\_k = diag(1, exp(2πi/2^k))?**

**Q9.  Define Quantum Phase Estimation. What is the eigenphase problem? Describe the QPE circuit (control register, eigenstate register, controlled-U^k gates, inverse QFT, measurement).**

**Q10.  For QPE with n control qubits, what is the precision of the phase estimate? If U = S (phase gate) and |ψ⟩ = |1⟩, what phase φ should QPE output? With n = 2 control qubits, what binary output do you expect?**

**Q11.  Compare the four oracle algorithms (Deutsch, Deutsch-Jozsa, Bernstein-Vazirani, Simon) in a table: (a) problem, (b) classical complexity, (c) quantum complexity, (d) speedup type.**

**Q12.  Explain how the Quantum Fourier Transform is used inside Shor's factoring algorithm. What mathematical operation does Shor need, and how does QPE/QFT provide it?**

**Q13.  What is the practical limitation of QFT's exponential speedup? Why can't we use QFT as a general FFT replacement in classical signal processing?**

**Q14.  Trace the Deutsch-Jozsa algorithm for n=2 and f(x) = x₁ ⊕ x₂ (balanced). Show the state at each step and verify P(|00⟩) = 0.**

**Q15.  What applications use Quantum Phase Estimation? Explain the quantum chemistry application: how is the molecular ground state energy extracted using QPE and the time evolution operator?**

**PART B — MODEL ANSWERS**

**Answer 1:**

Bit-flip oracle: O\_f|x⟩|b⟩ = |x⟩|b⊕f(x)⟩ — writes f(x) into ancilla register by XOR. Unitary because O\_f² = I (XOR is self-inverse). Phase oracle: O\_f|x⟩ = (−1)^{f(x)}|x⟩ — marks states with f(x)=1 by phase −1. Obtaining phase from bit-flip: initialise ancilla in |−⟩ = (|0⟩−|1⟩)/√2 and apply the bit-flip oracle: O\_f|x⟩|−⟩ = (−1)^{f(x)}|x⟩|−⟩ (Phase Kickback theorem). The phase 'kicks back' into the input register amplitude.

**Answer 2:**

Phase Kickback theorem: O\_f|x⟩|−⟩ = (−1)^{f(x)}|x⟩|−⟩. Proof: O\_f|x⟩(|0⟩−|1⟩)/√2 = |x⟩(|0⊕f(x)⟩−|1⊕f(x)⟩)/√2. Case f(x)=0: |x⟩(|0⟩−|1⟩)/√2 = (+1)|x⟩|−⟩. Case f(x)=1: |x⟩(|1⟩−|0⟩)/√2 = (−1)|x⟩|−⟩. Both cases give (−1)^{f(x)}|x⟩|−⟩ ✓. Physical interpretation: |−⟩ is the −1 eigenstate of X. When f(x)=1, the oracle applies X to |−⟩, and the eigenvalue −1 appears as a phase factor on the input register amplitude.

**Answer 3:**

Deutsch's algorithm for f:{0,1}→{0,1}: Step 1: Prepare |0⟩|1⟩. Step 2: Apply H⊗H → |+⟩|−⟩. Step 3: Apply bit-flip oracle O\_f → (−1)^{f(0)}|0⟩|−⟩/√2 + (−1)^{f(1)}|1⟩|−⟩/√2 = (−1)^{f(0)}(|0⟩+(−1)^{f(0)⊕f(1)}|1⟩)/√2 · |−⟩. Step 4: Apply H to first qubit and measure. If f constant (f(0)⊕f(1)=0): state before H is |+⟩ → H → |0⟩, measure 0. If balanced (f(0)⊕f(1)=1): state is |−⟩ → H → |1⟩, measure 1. Classical requires 2 queries (f(0) then f(1)). Quantum uses 1: the H gates create superposition and interference that encodes the GLOBAL property f(0)⊕f(1) in a single query.

**Answer 4:**

Problem: given f:{0,1}^n→{0,1} promised constant or balanced, determine which with 1 query. Circuit: |0⟩^n|1⟩ → H^{⊗n}⊗H → O\_f → H^{⊗n} → measure. After H^{⊗n}: (1/√2^n)Σ\_x|x⟩ ⊗ |−⟩. After O\_f: (1/√2^n)Σ\_x(−1)^{f(x)}|x⟩ ⊗ |−⟩. After H^{⊗n}: amplitude at |0⟩^n = (1/2^n)Σ\_x(−1)^{f(x)}. For constant f with value c: = (1/2^n)·2^n·(−1)^c = ±1, so P(|0^n⟩) = 1. For balanced f: equal numbers of +1 and −1 terms, sum = 0, P(|0^n⟩) = 0 ✓.

**Answer 5:**

Problem: find hidden s ∈ {0,1}^n where f(x) = s·x mod 2 = (Σᵢ sᵢxᵢ) mod 2. After H^{⊗n}, oracle, H^{⊗n}: amplitude at |z⟩ = (1/2^n)Σ\_x(−1)^{s·x+z·x} = (1/2^n)Σ\_x(−1)^{x·(s⊕z)}. Using the identity Σ\_x(−1)^{x·c} = 2^n δ\_{c,0}: amplitude at |z⟩ = δ\_{z,s}. So P(|z⟩) = 1 if z = s, 0 otherwise. The measurement outcome is s with certainty. Classical: n queries needed (one per bit of s, using basis vectors). Quantum: 1 query reveals the entire hidden string.

**Answer 6:**

Problem: given 2-to-1 function f with f(x)=f(x⊕s) for unknown period s ∈ {0,1}^n, find s. Classical: Ω(2^{n/2}) queries (birthday paradox — wait for collision f(x₁)=f(x₂)). Quantum: O(n) queries. Algorithm: prepare (1/√2^n)Σ\_x|x⟩|0⟩, apply U\_f, measure second register → collapses first to (|x₀⟩+|x₀⊕s⟩)/√2. Apply H^{⊗n} → obtain y\_i with y\_i·s = 0 mod 2. Repeat n times to collect n equations. Gaussian elimination over GF(2) (classical, O(n³)) solves for s uniquely with probability ≥ 1 − n/2^n.

**Answer 7:**

QFT: |j⟩ → (1/√2^n) Σ\_{k=0}^{2^n-1} exp(2πijk/2^n) |k⟩. The QFT maps basis state |j⟩ to a superposition of all basis states with frequency-dependent phases. For a superposition input Σ\_j α\_j|j⟩, QFT outputs Σ\_k β\_k|k⟩ where β\_k = (1/√2^n)Σ\_j α\_j exp(2πijk/2^n) — exactly the DFT of {α\_j}. Complexity: QFT requires n Hadamards + n(n−1)/2 controlled-R\_k gates = O(n²) gates. Classical FFT on N=2^n points: O(N log N) = O(n·2^n) operations. Ratio: QFT/FFT ≈ n²/(n·2^n) = n/2^n → exponentially fewer gates.

**Answer 8:**

Product representation: QFT|j⟩ = (1/√2^n) ⊗\_{k=1}^{n} (|0⟩ + exp(2πij/2^k)|1⟩). Each k-th factor involves only the phase exp(2πij/2^k), which depends on specific bits of j. Circuit: for qubit m, apply H to create (|0⟩+exp(2πij\_m/2)|1⟩), then apply controlled-R\_k gates from less significant qubits to add phases exp(2πi·j\_{m+k}/2^{k+1}) for k=1,...,n-m. Each controlled-R\_k gate adds one bit of phase information from the source qubit. After processing all qubits and applying a bit-reversal SWAP network, the full product state is achieved.

**Answer 9:**

QPE estimates φ in U|ψ⟩ = exp(2πiφ)|ψ⟩. Eigenphase problem: find φ given U and |ψ⟩. Circuit: n-qubit control register + m-qubit eigenstate register. Prepare |+⟩^n for control, |ψ⟩ for eigenstate. Apply controlled-U^{2^j} (controlled by control qubit j) to eigenstate: state becomes (1/√2^n)Σ\_k exp(2πiφk)|k⟩ ⊗ |ψ⟩. Apply QFT† (inverse QFT) to control register: when φ = p/2^n exactly, maps to |p⟩. Measure control register: obtain n-bit binary approximation to φ. Precision: ε ≤ 1/2^n, so n = log₂(1/ε) control qubits needed for precision ε.

**Answer 10:**

For S gate: S|1⟩ = i|1⟩ = exp(iπ/2)|1⟩, so φ = 1/4 (since exp(2πiφ) = exp(iπ/2) → φ = 1/4). With n=2 control qubits: 2-bit binary representation of 1/4 = 0.01 (binary) = |01⟩. So QPE should output the state |01⟩ in the control register with probability 1. Measurement: outcome 01 (binary) = 1 (decimal), corresponding to φ = 1/4·(1/2^2)... wait: outcome k gives φ ≈ k/2^n = 1/4 at k=1 with n=2: 1/2² = 0.25 ✓. So control register measures |01⟩ → binary 01 → φ = 1/4 ✓.

**Answer 11:**

Comparison table: Deutsch — determine constant/balanced f:{0,1}→{0,1}; Classical 2 queries; Quantum 1 query; Speedup: constant factor (×2). Deutsch-Jozsa — constant or balanced f:{0,1}^n→{0,1}; Classical 2^{n-1}+1; Quantum 1; Speedup: exponential. Bernstein-Vazirani — find hidden s in f=s·x mod 2; Classical n queries; Quantum 1; Speedup: factor n. Simon — find period of 2-to-1 f; Classical Ω(2^{n/2}); Quantum O(n); Speedup: exponential.

**Answer 12:**

Shor's algorithm: to factor N, compute f(x) = a^x mod N for randomly chosen a. Find the period r of this function (f(x+r) = f(x)). Given r, extract prime factors via gcd(a^{r/2}±1, N). The period r is found by Quantum Phase Estimation: the eigenvalues of the modular exponentiation unitary U\_a|y⟩=|ay mod N⟩ encode the periods as phases. QPE with sufficient precision extracts r in polynomial time O(n³). The QFT inside QPE is what achieves the exponential speedup over classical period-finding methods.

**Answer 13:**

QFT's exponential speedup applies to quantum amplitudes in superposition — it operates on all 2^n amplitude values simultaneously. The limitation: we cannot read out the 2^n output amplitudes. Due to the Holevo bound, measuring the QFT output gives only n classical bits per shot. This means QFT cannot replace classical FFT for applications requiring all N output values (signal processing, image processing). The speedup is only useful when combined with algorithms (Shor, QPE) where the specific structure of the QFT output can be measured efficiently.

**Answer 14:**

f(x)=x₁⊕x₂: f(00)=0, f(01)=1, f(10)=1, f(11)=0. After H^{⊗2}|00⟩: (|00⟩+|01⟩+|10⟩+|11⟩)/2. After phase oracle: (|00⟩−|01⟩−|10⟩+|11⟩)/2 (signs: f(x) = 0,1,1,0). After H^{⊗2}: amplitude at |z⟩ = (1/4)Σ\_x(−1)^{f(x)+x·z}. At z=00: (1/4)(1−1−1+1)=0 ✓. P(|00⟩)=0 ✓ — f is balanced.

**Answer 15:**

QPE applications: (1) Shor's factoring: period r of modular exponentiation; (2) Quantum chemistry: given time evolution unitary U=exp(−iH\_mol·t), the eigenphase φ = E₀t/ℏ (where E₀ is ground state energy). Apply QPE to U with eigenstate prepared by VQE or HF state → measure control register → extract φ → E₀ = φ·ℏ/t. This gives molecular ground state energy to n-bit precision, crucial for drug design and materials simulation; (3) HHL algorithm: QPE is used to invert eigenvalues of the coefficient matrix A in Ax=b.

## EXERCISES — CHAPTER 5

### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.1</strong></p>
<p>Problem: Prove that the phase oracle O_f (O_f|x⟩ = (−1)^f(x)|x⟩) is unitary.</p>
<p><strong>Solution:</strong></p>
<p>We need to show O_f†O_f = I. Since O_f acts on the computational basis as O_f|x⟩ = (−1)^f(x)|x⟩, it is a diagonal matrix:</p>
<p>O_f = diag((−1)^f(0), (−1)^f(1), ..., (−1)^f(2^n-1))</p>
<p>All diagonal entries are ±1, so the matrix is real and diagonal.</p>
<p>O_f† = (O_f*)^T = O_f  (since each entry is ±1 = its own complex conjugate and the matrix is diagonal).</p>
<p>O_f · O_f = diag((−1)^(2f(0)), ...) = diag(1, ..., 1) = I  ✓</p>
<p>Alternatively: O_f² = I is immediate from (−1)^f(x) · (−1)^f(x) = (−1)^(2f(x)) = 1.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.2</strong></p>
<p>Problem: Apply Deutsch's algorithm to the constant function f(x) = 1 (for x=0,1). Trace all steps and give the measurement outcome.</p>
<p><strong>Solution:</strong></p>
<p>Step 1 — Initial state: |0⟩|1⟩</p>
<p>Step 2 — Apply H⊗H: |+⟩|−⟩ = (|0⟩+|1⟩)/√2 · (|0⟩−|1⟩)/√2</p>
<p>Step 3 — Apply oracle O_f (f constant=1: bit-flip oracle flips target for ALL inputs):</p>
<p>O_f|x,b⟩ = |x, b⊕1⟩. Acting on (|0⟩+|1⟩)|−⟩/√2:</p>
<p>O_f (|0⟩+|1⟩)/√2 · |−⟩ = ? Let's use phase kickback:</p>
<p>(−1)^f(0)|0⟩|−⟩/√2 + (−1)^f(1)|1⟩|−⟩/√2 = −|0⟩|−⟩/√2 − |1⟩|−⟩/√2</p>
<p>= −(|0⟩+|1⟩)/√2 · |−⟩ = −|+⟩|−⟩</p>
<p>(The global phase −1 is unobservable.)</p>
<p>Step 4 — Apply H to first qubit: H|+⟩ = |0⟩</p>
<p>Measurement: outcome 0 → f is constant. ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.3</strong></p>
<p>Problem: For the Deutsch-Jozsa algorithm with n=3 and constant f(x)=0, compute the amplitude at each of the 8 basis states after the second Hadamard layer. Verify that |0⟩^3 has amplitude 1.</p>
<p><strong>Solution:</strong></p>
<p>After H^⊗3 (first layer): |+⟩^3 = (1/2√2) Σ_x |x⟩</p>
<p>After phase oracle (f=0 constant): (−1)^0 = +1 for all x, so state unchanged.</p>
<p>State: (1/2√2) Σ_{x=0}^{7} |x⟩</p>
<p>After H^⊗3 (second layer): amplitude at basis state |z⟩ is:</p>
<p>A_z = (1/8) Σ_{x=0}^{7} (−1)^(x·z) = (1/8) Σ_x (−1)^(x·z)</p>
<p>For z=000: A_{000} = (1/8)(8 × 1) = 1  ✓  [all terms are (+1)^0 = 1]</p>
<p>For z≠000: A_z = (1/8) Σ_x (−1)^(x·z). Since z has at least one bit = 1, summing (−1)^(x·z) over all x ∈ {0,1}^3 gives exactly 0 (equal numbers of +1 and −1).</p>
<p>Therefore: state after second H layer = |000⟩ with amplitude 1. Measurement gives 000 with P=1. ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.4</strong></p>
<p>Problem: The QFT on n=2 qubits (N=4). Write out the full 4×4 QFT matrix and verify it is unitary by computing QFT · QFT†.</p>
<p><strong>Solution:</strong></p>
<p>QFT matrix: F_{kj} = (1/√4) e^(2πijk/4) = (1/2) e^(iπjk/2)</p>
<p>The 4th root of unity is ω = e^(2πi/4) = e^(iπ/2) = i.</p>
<p>F = (1/2) [[1, 1, 1, 1], [1, i, -1, -i], [1, -1, 1, -1], [1, -i, -1, i]]</p>
<p>Verify F†F = I by computing (F†F)_{jk} = Σ_m F†_{jm} F_{mk} = Σ_m (F*)_{mj} F_{mk}</p>
<p>= (1/4) Σ_m e^(-2πijm/4) e^(2πikm/4) = (1/4) Σ_m e^(2πi(k-j)m/4)</p>
<p>= δ_{jk}  (orthogonality of characters of Z_4 — sum is 4 if j=k, else 0)</p>
<p>Therefore F†F = I, confirming unitarity. ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.5</strong></p>
<p>Problem: In QPE with t=4 ancilla qubits, what is the precision? If the true phase is φ=3/16=0.1875, what measurement outcomes are possible and with what probability?</p>
<p><strong>Solution:</strong></p>
<p>Precision with t=4: 1/2^4 = 1/16 = 0.0625</p>
<p>φ = 3/16 = 0.0011 in 4-bit binary → exact 4-bit representation!</p>
<p>Since φ = 3/16 is exactly representable in 4 bits: measurement gives |3⟩ = |0011⟩ with probability 1.</p>
<p>Decoded: φ̃ = 3/16. Error = 0. ✓</p>
<p>If instead φ = 1/5 = 0.2 (not exactly representable in 4 bits):</p>
<p>Nearest 4-bit fractions: 3/16=0.1875 and 4/16=0.25. The algorithm gives one of these with probability &gt; 8/π² ≈ 81%.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.6</strong></p>
<p>Problem: Implement a 4-qubit QFT in Qiskit from scratch (without using the QFT library). Count the gates and compare with the formula n(n+1)/2 + n/2.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.7</strong></p>
<p>Problem: Simon's algorithm is run 3 times on a 3-qubit oracle with period s, yielding measurement results y₁=110, y₂=011, y₃=101. Set up and solve the linear system over GF(2) to find s.</p>
<p><strong>Solution — Linear system y_i · s = 0 mod 2:</strong></p>
<p>y₁·s = 0:  1·s₀ + 1·s₁ + 0·s₂ = 0  →  s₀ + s₁ = 0 (mod 2)  [Eq.1]</p>
<p>y₂·s = 0:  0·s₀ + 1·s₁ + 1·s₂ = 0  →  s₁ + s₂ = 0 (mod 2)  [Eq.2]</p>
<p>y₃·s = 0:  1·s₀ + 0·s₁ + 1·s₂ = 0  →  s₀ + s₂ = 0 (mod 2)  [Eq.3]</p>
<p><strong>Gaussian elimination over GF(2):</strong></p>
<p>From Eq.1: s₁ = s₀. From Eq.2: s₂ = s₁ = s₀. Check Eq.3: s₀+s₂ = s₀+s₀ = 0 ✓</p>
<p>Two solutions: s₀=0 gives s=(0,0,0) [trivial]; s₀=1 gives s=(1,1,1)</p>
<p>Since Simon's algorithm guarantees s ≠ 0, the answer is s = 111.</p>
<p>Verification: y₁·111 = 1+1+0 = 0 (mod 2) ✓; y₂·111 = 0+1+1 = 0 ✓; y₃·111 = 1+0+1 = 0 ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 5.8</strong></p>
<p>Problem: A quantum register of n=5 qubits holds the periodic state |ψ⟩ = (1/√8) Σ_{k=0}^{7} |4k⟩ (period r=4, N=32). After QFT, which basis states have non-zero probability? What period can be extracted?</p>
<p><strong>Solution:</strong></p>
<p>The state |ψ⟩ has N=32, r=4, and N/r = 8 non-zero terms.</p>
<p>By the QFT period-finding formula, non-zero probability after QFT at indices j where j = m · N/r for integer m:</p>
<p>j = m × 32/4 = 8m  for m = 0, 1, 2, 3</p>
<p>So non-zero states: |0⟩, |8⟩, |16⟩, |24⟩  (4 states, each with probability 1/4)</p>
<p>Period extraction: from any measurement j ∈ {8, 16, 24}:</p>
<p>j/N = 8/32 = 1/4, 16/32 = 1/2, 24/32 = 3/4</p>
<p>Continued fraction of 1/4: denominator = 4 = r ✓</p>
<p>Continued fraction of 1/2: denominator = 2, but r=4 is a multiple of 2 (need multiple runs)</p>
<p>Continued fraction of 3/4: numerator/denominator = 3/4, denominator = 4 = r ✓</p>
<p>With 2-3 measurements, the period r=4 can be reliably extracted using GCD and continued fractions.</p>
</div>

### B. Unsolved Problems

Solve independently. Bracketed answers for self-checking.

1. Verify that the bit-flip oracle O\_f with f(x)=x₁ (the second bit of a 2-bit input) satisfies O\_f|01⟩|0⟩ = |01⟩|0⟩ and O\_f|10⟩|0⟩ = |10⟩|1⟩. [Answer: f(01)=0 so target unchanged: |01⟩|0⟩; f(10)=1 so target flips: |10⟩|1⟩]

2. For Deutsch's algorithm with f(0)=1 and f(1)=0 (balanced), trace the state after each step and verify the measurement outcome is 1 (balanced). [Answer: After oracle, state is −|−⟩|−⟩ (global phase); after H, first qubit = |1⟩; measurement = 1 ✓]

3. For Bernstein-Vazirani with n=4 and hidden string s=1001, which Hadamard gate output amplitude is non-zero? What is the measurement outcome? [Answer: Amplitude 1 at |z⟩=|1001⟩ only; measurement outcome = 1001 with P=1]

4. Write the 2-qubit QFT matrix explicitly using ω=e^(iπ/2)=i and verify F†F=I. [Answer: F=(1/√2)[[1,1],[1,−1]]⊗ ... wait this is 1-qubit H; 2-qubit QFT: F=(1/2)[[1,1,1,1],[1,i,−1,−i],[1,−1,1,−1],[1,−i,−1,i]]; F†F: each column is orthonormal ✓]

5. A 5-qubit QFT has how many (a) H gates, (b) controlled phase gates, (c) SWAP gates? What is the total gate count? [Answer: (a) 5 H gates; (b) n(n-1)/2 = 10 controlled-phase gates; (c) floor(5/2) = 2 SWAP gates; total = 17 gates]

6. QPE with t=6 ancilla estimates φ=0.333... (=1/3). What is the best 6-bit approximation? What is the error? [Answer: 1/3 ≈ 0.010101 in binary ≈ 21/64 = 0.328125; closest 6-bit fraction to 1/3 is 21/64; error = 1/3 − 21/64 = 1/192 ≈ 0.0052]

7. How many controlled-U operations are needed in QPE with t=10 ancilla? How does this scale with precision? [Answer: Total controlled-U ops = Σ\_{k=0}^{9} 2^k = 2^10−1 = 1023 ≈ O(2^t); precision 1/2^10 = 1/1024; so precision improves as 1/(# operations)]

8. In Simon's algorithm for n=2 with s=11, list all measurement outcomes y that satisfy y·s=0 (mod 2). [Answer: y·(1,1) = y₁+y₂ = 0 mod 2; solutions: y=00 and y=11; these are the two allowed measurement outcomes from Simon's algorithm]

9. The QFT on a superposition of 3 equally-spaced states |0⟩+|3⟩+|6⟩ (unnormalised) in an 8-point register. After QFT, at which indices are the peaks? [Answer: spacing r=3, N=8. QFT peaks at multiples of N/r = 8/3 ≈ 2.67 — not integer, so not exact. Closest integers: 0, 3 (= round(8/3)), 5 (= round(16/3)), with additional probability spread. Exact calculation: amplitude at k = (1/√8)(1+e^(6πik/8)+e^(12πik/8)), peaks wherever this magnitude is maximised]

10. Derive the number of queries needed classically to solve the Deutsch-Jozsa problem with certainty for n=50 input bits. Compare with the quantum algorithm. [Answer: Classical deterministic: 2^49 + 1 ≈ 5.6×10^14 queries. Randomised with 99% confidence: ~7 queries (Chernoff bound). Quantum: exactly 1 query.]

### C. Multiple Choice Questions

Answers at end of section.

**Q1. The phase kickback mechanism states that O\_f|x⟩|−⟩ equals:**

(a)  |x⟩|f(x)⟩

(b)  (−1)^f(x)|x⟩|−⟩

(c)  |x⟩|−f(x)⟩

(d)  (−1)^f(x)|x⟩|0⟩

**Q2. Deutsch's algorithm uses how many oracle queries to determine if f is constant or balanced?**

(a)  2

(b)  n

(c)  1

(d)  log n

**Q3. The Deutsch-Jozsa algorithm measures |0⟩^n if and only if:**

(a)  f is balanced

(b)  f is constant

(c)  f(0) = 0

(d)  n is even

**Q4. Bernstein-Vazirani algorithm finds the hidden string s using:**

(a)  O(n) quantum queries

(b)  2 quantum queries

(c)  1 quantum query

(d)  O(√n) queries

**Q5. The QFT of an n-qubit register requires how many gates (excluding SWAP)?**

(a)  O(2^n)

(b)  O(n log n)

(c)  O(n²)

(d)  O(n)

**Q6. The QFT matrix F\_N has entries F\_{kj} = e^(2πijk/N)/√N. What is F\_N · F\_N†?**

(a)  2F\_N

(b)  I (identity)

(c)  N · I

(d)  0 (zero matrix)

**Q7. In QPE with t ancilla qubits, the phase estimate has precision:**

(a)  t bits (error ≤ 1/2^t)

(b)  log t bits

(c)  t/2 bits

(d)  2^t bits

**Q8. The QFT on a periodic state with period r (in an N-point register) produces peaks at:**

(a)  Random positions

(b)  Every r-th position

(c)  Multiples of N/r

(d)  Positions 0 and N/2 only

**Q9. Simon's algorithm finds the period s of f using:**

(a)  Exponentially many quantum queries

(b)  O(n) quantum queries followed by classical linear algebra over GF(2)

(c)  A single quantum query

(d)  O(√N) quantum queries

**Q10. Phase kickback is used to convert a bit-flip oracle into a phase oracle by initialising the ancilla qubit in:**

(a)  |0⟩

(b)  |+⟩ = (|0⟩+|1⟩)/√2

(c)  |−⟩ = (|0⟩−|1⟩)/√2

(d)  |1⟩

**Q11. QPE requires implementing controlled-U^(2^k) for k=0,1,...,t−1. The total number of controlled-U operations is:**

(a)  t

(b)  t²

(c)  2^t − 1

(d)  log t

**Q12. The QFT circuit ends with a SWAP network because:**

(a)  SWAPs reduce circuit depth

(b)  The rotation network produces output qubits in bit-reversed order

(c)  SWAP gates are cheaper than phase gates

(d)  The SWAP network implements the inverse QFT

**Q13. For QPE to efficiently solve a problem, what must be true about implementing U^(2^k)?**

(a)  U^(2^k) must be implementable in O(poly(n)) gates for all k

(b)  U^(2^k) must be a Clifford gate

(c)  U^(2^k) must be its own inverse

(d)  U must have integer eigenvalues

**Q14. Classical DFT on N=2^n points requires O(N log N) operations. The QFT uses O(n²) gates. What does this exponential gap mean practically?**

(a)  QFT speeds up classical signal processing exponentially

(b)  QFT gives exponential speedup only as a subroutine where input is already a quantum state

(c)  Classical computers cannot perform Fourier transforms

(d)  QFT requires exponentially more memory

**Q15. Quantum Phase Estimation is used in Shor's algorithm to:**

(a)  Generate entanglement between qubits

(b)  Find the period r of the function f(x) = a^x mod N

(c)  Factor the integer N directly

(d)  Prepare the initial superposition of all x values

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 5</strong></p>
<p>Q1: (b) (−1)^f(x)|x⟩|−⟩ — This is the phase kickback theorem: the phase (−1)^f(x) kicks back into the input register amplitude, and the ancilla |−⟩ remains unchanged.</p>
<p>Q2: (c) 1 — Deutsch's algorithm uses exactly 1 oracle query, compared to 2 classically for certain determination.</p>
<p>Q3: (b) f is constant — Amplitude at |0⟩^n after second H layer = (1/2^n)|Σ_x (−1)^f(x)|²; this equals 1 iff all (−1)^f(x) have the same sign, i.e., f is constant.</p>
<p>Q4: (c) 1 quantum query — BV algorithm finds all n bits of s with a single oracle query and measurement, versus n classical queries.</p>
<p>Q5: (c) O(n²) — The QFT uses n H gates + n(n-1)/2 controlled-R_k gates = n + n(n-1)/2 = O(n²) gates.</p>
<p>Q6: (b) I (identity) — The QFT matrix is unitary: F_N†F_N = I. Each column is a unit vector and columns are mutually orthogonal (orthogonality of Fourier characters).</p>
<p>Q7: (a) t bits (error ≤ 1/2^t) — With t ancilla, QPE produces a t-bit approximation to φ, with error ≤ 1/2^t = precision of a t-bit binary fraction.</p>
<p>Q8: (c) Multiples of N/r — The QFT of a state periodic with period r in an N-point register has probability concentrated at integer multiples of N/r.</p>
<p>Q9: (b) O(n) quantum queries + classical GF(2) linear algebra — Simon's algorithm makes O(n) quantum measurements, each giving a vector y with y·s=0; n such linearly independent vectors suffice to solve for s.</p>
<p>Q10: (c) |−⟩ = (|0⟩−|1⟩)/√2 — The phase kickback theorem requires the ancilla in |−⟩ (eigenstate of X with eigenvalue −1). Any other state does not produce the clean phase kickback.</p>
<p>Q11: (c) 2^t − 1 — Ancilla qubit k controls U^(2^k); total = 2^0+2^1+...+2^(t-1) = 2^t−1. This is why QPE runtime grows exponentially in t (measuring t bits of precision costs O(2^t) controlled-U operations).</p>
<p>Q12: (b) Output qubits in bit-reversed order — The product-form derivation naturally produces output qubit j holding bit n-j of the result; SWAP network corrects this to standard ordering.</p>
<p>Q13: (a) U^(2^k) implementable in O(poly(n)) gates — If U^(2^k) requires exponential resources, QPE would not be efficient. For Shor's algorithm, modular exponentiation can be done in O(n^2 log n) gates.</p>
<p>Q14: (b) Exponential speedup only as a subroutine where input is already quantum — The QFT cannot speed up classical signal processing because reading out the quantum state takes as long as the classical FFT. The speedup only materialises when the QFT is used as a module in a larger quantum algorithm.</p>
<p>Q15: (b) Find the period r of f(x) = a^x mod N — Shor's algorithm reduces factoring to period finding; QPE (applied to the modular exponentiation unitary) finds the period r, from which factors of N are extracted by GCD.</p>
</div>

### D. Theory Questions

- Explain the quantum oracle model. What is a query? Why is query complexity a useful measure of quantum advantage? How does it differ from time complexity? Give an example where query complexity gives a provable quantum advantage but time complexity comparison is less clear.

- State and prove the phase kickback theorem. Explain the physical mechanism: why does the phase "kick back" to the input register rather than remaining on the ancilla? What would happen if the ancilla were initialised in |0⟩ instead of |−⟩?

- Explain in detail why the Deutsch-Jozsa algorithm achieves exponential speedup. For what problems does this speedup apply? Does the D-J problem have classical randomised algorithms that are nearly as efficient? If so, is the quantum speedup still meaningful?

- Derive the product formula for the QFT: QFT|j₁j₂...jₙ⟩ = tensor product of states (|0⟩+e^(2πi 0.j\_k...j\_n)|1⟩)/√2 for k=1,...,n. Use this to explain the circuit structure.

- Compare the complexity of classical FFT and quantum QFT. Be precise: (a) what does each algorithm compute, (b) what are the input/output requirements, (c) in what contexts does the QFT provide an exponential speedup and in what contexts does it NOT?

- Describe the QPE circuit in complete detail. For each stage (H^⊗t, controlled-U^k, QFT†), state what transformation is applied, what the quantum state looks like after, and why that stage is necessary.

- Explain QPE precision analysis: how does the number of ancilla qubits t determine the precision, probability of success, and resource cost? If we want 10^-6 precision with 99% success probability, how many ancilla qubits are needed?

- Describe Simon's algorithm completely. What is the hidden subgroup problem? How does Simon's algorithm exploit quantum parallelism to solve it? Why does the algorithm require classical post-processing, and what classical algorithm is used?

- Explain how the Bernstein-Vazirani algorithm works as a special case of the Deutsch-Jozsa algorithm. What is the inner product function? Why does the BV algorithm output the hidden string s with certainty, while the D-J algorithm only makes a binary distinction (constant vs balanced)?

- Describe the connection between the QFT, QPE, and Shor's algorithm. Specifically: how does period finding reduce to eigenphase estimation, and how does eigenphase estimation use the QFT? Draw a high-level flowchart showing these connections.

### E. Programming Assignments

PA5.1. [Deutsch-Jozsa]  Implement a complete Deutsch-Jozsa experiment for n=1,2,3,4,5 qubits in Qiskit. For each n: (a) implement both a constant oracle (f=0 and f=1) and 3 different balanced oracles, (b) run on AerSimulator with 1 shot and verify the algorithm always gives the correct answer, (c) run on FakeSherbrooke (noisy) with 1000 shots and plot the histogram — does the algorithm still work? (d) measure circuit depth and gate count for each n and plot vs n. Submit code, histograms (noiseless and noisy), and analysis comparing ideal vs noisy results.

PA5.2. [QFT from Scratch and Verification]  Build a Qiskit QFT circuit for n=4,5,6 qubits from scratch (without the QFT library). (a) Verify correctness by comparing with Qiskit's built-in QFT using state fidelity. (b) Test on the periodic state |ψ⟩ = (1/√(2^n/r)) Σ\_k |kr⟩ for periods r=2,4,8,16 (choose appropriate n). (c) Simulate measurement outcomes and verify that peaks appear at multiples of N/r. (d) Implement continued fraction algorithm to extract r from a sampled peak. Submit code, circuit diagrams, probability histograms, and period extraction results.

PA5.3. [QPE Implementation]  Implement Quantum Phase Estimation in Qiskit for the following unitaries: (a) U=T gate (φ=1/8), (b) U=Rz(π/3) (φ=1/6, requires more ancilla for good precision), (c) U=S gate (φ=1/4). For each: (1) Build the QPE circuit with t=6 ancilla qubits, (2) simulate with AerSimulator, (3) decode the most probable measurement outcome, (4) compare with true φ, (5) plot estimated φ vs true φ for t=2,4,6,8,10 to show precision improvement. Demonstrate the 1/2^t precision scaling. Submit complete code and precision analysis plot.

### F. Project Suggestions

Project 5.A — Simon's Algorithm Full Implementation:  Implement Simon's algorithm in full: (a) Write a Qiskit implementation with a configurable oracle for any 3-qubit hidden string s ∈ {001,...,111}. (b) Run the quantum circuit n times to collect n measurement vectors y₁,...,yₙ. (c) Implement Gaussian elimination over GF(2) to find s from the linear system. (d) Analyse how many repetitions are needed to recover s with > 99% probability (theoretically and empirically). (e) Test all 7 non-trivial hidden strings and verify correctness. (f) Compare with classical brute force search. Write a 2000-word report including circuit diagrams, GF(2) algorithm description, and probability analysis.

Project 5.B — QFT Period Finding and Connection to Shor's Algorithm:  Study the period-finding step of Shor's algorithm without implementing full modular exponentiation. (a) For N=15 and a=2, compute f(x)=2^x mod 15 classically for x=0,...,31 and verify the period r=4. (b) Prepare the periodic quantum state |ψ⟩=(1/√8)Σ\_k|4k⟩ in a 5-qubit register directly (bypassing the oracle). (c) Apply QFT and simulate measurement. (d) Implement continued fraction algorithm to extract r from measured peaks. (e) Compute factors of 15 using gcd(2^2±1, 15). (f) Repeat for a=7 (period r=4) and a=4 (period r=2). Write a 2500-word technical report explaining each step with mathematical derivations and simulation results.

Project 5.C — Quantum Chemistry via QPE:  Use QPE to estimate the ground state energy of a simple molecule (H₂) using Qiskit and qiskit-nature. (a) Build the H₂ Hamiltonian as a sum of Pauli operators using qiskit\_nature.second\_q.mappers. (b) Implement e^(-iHt) using first-order Trotter decomposition. (c) Apply QPE with t=6 ancilla to estimate the ground state energy. (d) Compare with exact diagonalisation and VQE results. (e) Analyse the effect of Trotter step size on accuracy. Write a 3000-word report covering the molecular Hamiltonian construction, Trotter simulation, QPE circuit analysis, and numerical results. This project connects all three sections of this chapter in a real-world quantum chemistry context.

## References and Further Reading

1. Nielsen, M. A. & Chuang, I. L. (2000). Quantum Computation and Quantum Information. Cambridge. [Chapters 5 (QFT, QPE), 1.4 (algorithms overview)]

2. Deutsch, D. (1985). Quantum Theory, the Church-Turing Principle and the Universal Quantum Computer. Proc. Royal Society A 400, 97–117. [Original Deutsch algorithm]

3. Deutsch, D. & Jozsa, R. (1992). Rapid solution of problems by quantum computation. Proc. Royal Society A 439, 553–558. [Deutsch-Jozsa algorithm]

4. Bernstein, E. & Vazirani, U. (1993). Quantum Complexity Theory. STOC, pp. 11–20. [BV algorithm; quantum oracle complexity lower bounds]

5. Simon, D. R. (1994). On the Power of Quantum Computation. FOCS 1994, pp. 116–123. [Simon's algorithm; inspiration for Shor]

6. Shor, P. W. (1994). Algorithms for Quantum Computation: Discrete Logarithms and Factoring. FOCS 1994, pp. 124–134. [Shor's factoring algorithm using QFT+QPE]

7. Cleve, R., Ekert, A., Macchiavello, C., Mosca, M. (1998). Quantum algorithms revisited. Proc. Royal Society A 454, 339–354. [Clean QPE formulation; oracle query model]

8. Reiher, M. et al. (2017). Elucidating reaction mechanisms on quantum computers. PNAS 114, 7555. [QPE for quantum chemistry; FeMo cofactor resource estimates]

9. Qiskit Textbook (2024). Quantum Fourier Transform. https://learning.quantum.ibm.com. [Qiskit QFT implementation details]

10. Kitaev, A. Y. (1995). Quantum measurements and the Abelian stabilizer problem. arXiv:quant-ph/9511026. [Original QPE formulation]
