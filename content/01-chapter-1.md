# UNIT I - CHAPTER 1: The Quantum Revolution Computing at the Edge of Nature

<div class="box box-anecdote">
<p class="box-title"><strong>📜  The Feynman Provocation — MIT, May 1981</strong></p>
<p>At a Physics of Computation conference at MIT, Richard Feynman rose to challenge his audience with characteristic irreverence: 'Nature isn't classical, dammit, and if you want to make a simulation of nature, you'd better make it quantum mechanical, and by golly it's a wonderful problem because it doesn't look so easy.' He was pointing to something profound: the state space of a quantum system grows exponentially with its size. To simulate 300 coupled quantum particles exactly, a classical computer would need to store more amplitudes than there are atoms in the observable universe. Feynman was right — and the full implications of that insight are still unfolding.</p>
</div>

We live at a remarkable moment in the history of computation. The classical computer — arguably the most transformative invention of the twentieth century — is approaching the fundamental physical limits of its miniaturisation. At the same time, physicists have spent four decades developing a completely different paradigm: computation using the very quantum effects that doom classical miniaturisation. This chapter tells that story.

This opening chapter sets the scene for the entire course. Before we encounter a single equation, we must first understand why quantum computing was conceived, what problems it addresses, and what it can — and cannot — achieve. We survey the history of the field, explore the landmark algorithms that defined it, and situate it within the broader global effort now underway to build practical quantum machines.

### Why Classical Computers Are Not Enough

To appreciate why an entirely new computational paradigm is being developed — one that costs billions of dollars and requires cooling machines to temperatures colder than outer space — we must first understand the limits of classical computers. These limits are not merely engineering challenges that will be solved by better chips or more memory. Some of them are mathematical and physical boundaries that cannot be crossed, no matter how advanced our silicon technology becomes.

#### 1.1.1 The Triumph of Moore's Law

In 1965, Gordon Moore, co-founder of Intel, made a remarkable empirical observation: the number of transistors that could be economically placed on a microchip was doubling approximately every two years. This observation — now known as Moore's Law — proved to be one of the most accurately predictive statements in the history of technology.

The numbers are staggering. The Intel 4004 processor of 1971, the world's first single-chip CPU, contained 2,300 transistors. Each transistor was 10,000 nanometres (nm) wide. By 2023, Apple's M3 chip contained approximately 25 billion transistors, each just 3 nm wide — roughly the width of 15 silicon atoms arranged in a line. From 2,300 to 25,000,000,000 transistors over 52 years represents an increase by a factor of roughly 11 billion.

This exponential progress gave us personal computers, smartphones, the internet, artificial intelligence, and modern medicine. Every decade, computers became roughly 100 times more powerful, enabling applications that had previously been impossible. The world was transformed.

📜  Gordon Moore's Famous Prediction

Gordon Moore published his now-legendary paper 'Cramming More Components onto Integrated Circuits' in Electronics Magazine in April 1965. He predicted — almost offhandedly — that the number of components per circuit would double each year for the foreseeable future. He later revised this to every two years. Remarkably, this prediction held for over half a century, guiding the entire semiconductor industry as a self-fulfilling prophecy. Intel famously used Moore's Law as a planning tool, setting internal targets based on it. The law has now largely ended, not because of any failure of engineering ambition, but because the laws of physics have intervened. You can only make a transistor so small before it stops behaving like a classical on/off switch and starts behaving like a quantum object.

Today, that exponential progress is ending. At 3 nm transistor widths, quantum mechanical effects become dominant. Electrons — which should stay inside the transistor channel — begin to quantum tunnel through the insulating barrier, causing leakage current and bit errors. As transistors are packed ever more densely, heat dissipation becomes catastrophic; modern processor cores can dissipate over 100 watts per square centimetre, comparable to a rocket engine nozzle.

The physical limit of classical transistor scaling is now widely acknowledged to be approaching within this decade. TSMC, Intel, and Samsung continue to make incremental improvements, but the era of doubling performance every two years by simply shrinking transistors is effectively over. A transistor in the Intel 4004 processor (1971) was 10,000 nanometres wide; in Apple's M3 chip (2023), transistors measure just 3 nanometres — roughly 15 silicon atoms.

<figure class="book-figure">
<img src="content/images/image3.png" alt="Figure 1.1: Moore&#x27;s Law: Transistor Count in Microprocessors (1971–2023)">
<figcaption>Figure 1.1: Moore&#x27;s Law: Transistor Count in Microprocessors (1971–2023)</figcaption>
</figure>

This progress has transformed every domain of human activity. But the exponential progress of Moore's Law is ending. Quantum tunnelling causes electrons to leak through barriers. Heat dissipation becomes catastrophic — modern chips dissipate over 100 W/cm², more than a rocket engine nozzle per unit area. The physical limit is near.

#### 1.1.2 The Exponential Complexity Wall

Even if transistors could be made arbitrarily small and fast, classical computers would still face an insurmountable barrier for certain problems: the problems themselves are intrinsically exponential. No matter how fast the computer, the computational resources required grow exponentially with problem size. No classical algorithm can solve them efficiently for large inputs.

Consider four categories of important problems

| Problem | Classical Complexity | Practical Consequence |
|---|---|---|
| Integer factorisation | Sub-exponential | RSA-2048 takes ~10¹³ years to crack classically |
| Quantum simulation | Exponential O(2ⁿ) | 50-electron molecule already intractable classically |
| Unstructured search | Linear O(N) | 10¹² items: 500 seconds at 10⁹ checks/second |
| Travelling salesman | Factorial O(N!) | 20 cities: 2.4×10¹⁸ routes — exact solution impossible |

<div class="box box-real-world">
<p class="box-title"><strong>🌐  The Drug Discovery Crisis</strong></p>
<p>An exact classical simulation of a molecule with n electrons requires memory and time scaling as O(2ⁿ). Penicillin has approximately 1,200 atoms; no classical computer in existence or foreseeable can simulate it exactly. As a result, drug development relies on approximations and costly laboratory trial-and-error, taking 10–15 years and $2–3 billion per drug on average. Quantum computers could simulate molecular quantum dynamics exactly and efficiently, potentially compressing drug discovery timelines from years to weeks. This is one of the most powerful arguments for why quantum computing matters.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 1.1  Classical vs Quantum Search Time</strong></p>
<p>Problem: A database has 10¹² items. A classical computer checks 10⁹ items/second. A quantum computer runs 10⁶ Grover iterations/second. Compare search times. Classical (average): N/2 = 5×10¹¹ checks / 10⁹/s = 500 seconds ≈ 8.3 minutes. Grover (quantum): √N × π/4 ≈ √(10¹²) × 0.785 ≈ 7.85×10⁵ iterations / 10⁶/s ≈ 0.79 seconds. Speedup factor: 500 / 0.79 ≈ 633×. Despite running 1000× slower per step, the quantum computer is over 600× faster overall.</p>
</div>

### 1.2 The Birth of Quantum Computing

#### 1.2.1 Paul Benioff's Quantum Turing Machine (1980)

The first rigorous quantum computing model was proposed by Paul Benioff at Argonne National Laboratory in 1980. Benioff showed that a quantum-mechanical system could simulate a classical Turing machine reversibly, without violating the laws of quantum mechanics. However, Benioff's machine offered no computational advantage — it was quantum mechanically implemented but classically powered.

#### 1.2.2 Feynman's Universal Quantum Simulator (1982)

Richard Feynman's 1982 paper made the crucial theoretical step: classical computers cannot efficiently simulate general quantum systems. More powerfully, he proposed that a computer that itself operates by quantum rules would naturally simulate other quantum systems with polynomial overhead. This was simultaneously a fundamental limitation (classical computers cannot do quantum simulation efficiently) and a founding vision (quantum computers can).

#### 1.2.3 David Deutsch's Universal Quantum Computer (1985)

Oxford physicist David Deutsch defined the first universal quantum computer in 1985 — a quantum device capable of simulating any physical process with polynomial overhead. He also constructed the first quantum algorithm, proving that quantum computation is not just a different implementation of classical computation, but a fundamentally superior computational model for specific problem classes.

<div class="box box-anecdote">
<p class="box-title"><strong>📜  Deutsch and the Many-Worlds Interpretation</strong></p>
<p>David Deutsch is a committed advocate of Hugh Everett's many-worlds interpretation of quantum mechanics, and he views quantum parallelism quite literally as computation happening in parallel universes. In his view, when a quantum computer evaluates a function in superposition, it genuinely performs 2ⁿ computations in 2ⁿ parallel universes, and quantum interference allows the universes to collaborate to produce the correct answer. Whether or not you accept many-worlds, the mathematics is identical and the algorithms work the same. But Deutsch's perspective gives a vivid intuition for why quantum computers can be so powerful: they harness an exponentially large computational resource that exists in nature but that classical computers cannot access.</p>
</div>

<figure class="book-figure">
<img src="content/images/image4.png" alt="Figure 1.2: Key Milestones in Quantum Computing History (1981–2024)">
<figcaption>Figure 1.2: Key Milestones in Quantum Computing History (1981–2024)</figcaption>
</figure>

### 1.3 Shor's Algorithm: When Quantum Broke Cryptography

#### 1.3.1 The RSA Problem

RSA public-key cryptography, published in 1977 by Rivest, Shamir, and Adleman, has underpinned internet security for nearly fifty years. Its security rests on the apparent computational hardness of factoring large integers: multiplying two large primes p and q is easy (polynomial time), but given only N = p×q, recovering p and q appears computationally hard classically.

In April 1994, Peter Shor of Bell Laboratories announced a polynomial-time quantum algorithm for integer factorisation. The announcement was staggering — RSA encryption protecting global banking, email, and government communications could, in principle, be broken by a sufficiently large quantum computer.

<div class="box box-generic">
<p class="box-title"><strong>Shor's Algorithm Complexity: Classical GNFS exp(O(n^{1/3} log^{2/3} n))  →  Quantum O(n³)</strong></p>
<p>n = number of bits. Exponential quantum speedup — polynomial vs sub-exponential classical.</p>
</div>

<div class="box box-anecdote">
<p class="box-title"><strong>📜  The Day Cryptography Held Its Breath — 1994</strong></p>
<p>When Shor announced his algorithm at a workshop, the reaction was immediate. NSA cryptographers reportedly read his preprint within hours. Companies selling RSA encryption software watched their stock prices fluctuate. Shor later said: 'I was very excited. I thought: this is going to change things.' Ron Rivest (the 'R' in RSA) has since said: 'Shor's result was completely unexpected. It showed that quantum computers could do something fundamentally different from classical computers.' The internet security community immediately began planning for post-quantum cryptography — a process now culminating in NIST's 2024 standardization of quantum-safe algorithms (CRYSTALS-Kyber, CRYSTALS-Dilithium, SPHINCS+).</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 1.2  Shor's Algorithm Speedup Estimate</strong></p>
<p>Classical GNFS time to factor RSA-2048 (n=2048 bits): Time ∝ exp(c·n^(1/3)·(ln n)^(2/3)) where c≈1.923. n^(1/3) = 2048^(1/3) ≈ 12.70. ln(2048) ≈ 7.624. (7.624)^(2/3) ≈ 3.87. Exponent ≈ 1.923 × 12.70 × 3.87 ≈ 94.6. exp(94.6) ≈ 1.4×10⁴¹ operations. At 10¹⁸ ops/sec (Frontier supercomputer): time ≈ 10²³ seconds ≈ 3×10¹⁵ years (300,000× age of universe). Quantum (Shor): O(n³) = O(2048³) ≈ 8.6×10⁹ gate operations. At 10⁶ gates/sec: ≈ 8600 seconds ≈ 2.4 hours. Conclusion: Exponential speedup — from 10¹⁵ years to 2.4 hours.</p>
</div>

<figure class="book-figure">
<img src="content/images/image5.png" alt="Figure 1.3: Quantum Computational Advantage: Grover Search and Shor Factoring">
<figcaption>Figure 1.3: Quantum Computational Advantage: Grover Search and Shor Factoring</figcaption>
</figure>

### 1.4 Grover's Search and the Power of Amplitude Amplification

In 1996, Lov Grover published an algorithm for searching an unsorted database of N items in O(√N) quantum operations. The quadratic speedup is universal — it applies to any problem reducible to searching — and is provably optimal for unstructured quantum search (BBBV theorem, 1994).

<div class="box box-generic">
<p class="box-title"><strong>Grover's Algorithm: O(√N) queries  vs  Classical O(N) average search</strong></p>
<p>Grover optimal iterations: k_opt = round(π√N/4). Success probability peaks at P(target) ≈ 1.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 1.3  Grover's Algorithm — Step-by-Step</strong></p>
<p>For a database of N=16 items with 1 marked item: 1. Prepare equal superposition of all 16 states using 4 Hadamard gates: (1/4)Σ|x⟩ 2. Apply Phase Oracle: negate amplitude of target state |x*⟩ 3. Apply Diffusion Operator (inversion about mean): amplifies |x*⟩ amplitude 4. Repeat steps 2–3 for k_opt = round(π√16/4) = round(π) = 3 iterations 5. Measure: probability of finding |x*⟩ ≈ sin²((2×3+1)×arcsin(1/4)) ≈ 0.961 (96.1%) Classical average: 8 checks. Grover: 3 oracle calls. Speedup: ~2.7× for N=16; grows as √N.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 1.4  Why Grover Cannot Solve NP Problems Easily</strong></p>
<p>Consider 3-SAT with N=2ⁿ possible truth assignments to n boolean variables. Grover gives O(√(2ⁿ)) = O(2^(n/2)) quantum speedup — this is exponential in n/2, still exponential! For n=100 variables: classical brute-force ≈ 2¹⁰⁰ checks; Grover ≈ 2⁵⁰ checks. At 10¹⁵ checks/second: Grover takes 2⁵⁰/10¹⁵ ≈ 1 year. Still intractable. Grover helps but does NOT make NP problems polynomial-time. The BBBV theorem proves O(√N) is optimal for quantum unstructured search — no better quantum algorithm can exist for general unstructured search.</p>
</div>

### 1.5 Quantum Supremacy and the NISQ Era

#### 1.5.1 Google's Sycamore Experiment (2019)

On October 23, 2019, Google's 53-qubit Sycamore processor completed a random quantum circuit sampling task in 200 seconds. Google claimed the same computation would take Summit (IBM's 200 petaflop supercomputer) approximately 10,000 years. IBM immediately contested this, claiming Summit could complete it in 2.5 days using optimised classical simulation. Both claims are technically defensible — the true figure depends on classical algorithm choice and output fidelity requirements.

<div class="box box-anecdote">
<p class="box-title"><strong>📜  The Supremacy Controversy</strong></p>
<p>The word 'supremacy' in Google's paper title proved controversial. IBM — a direct competitor — argued the claim was exaggerated and that the classical comparison was unfair. The Chinese team at USTC subsequently claimed their own quantum advantage via Boson Sampling (2020) and with their Zuchongzhi superconducting processor (2021). The scientific debate highlights a fundamental difficulty: comparing quantum and classical machines is not straightforward when the quantum computer is doing a task specifically designed to make it look good. What is clear is that quantum computers can now perform at least one task that is impractical for classical machines, even if that task was designed to showcase quantum hardware.</p>
</div>

#### 1.5.2 The NISQ Era — Where We Are Now

John Preskill coined the term NISQ (Noisy Intermediate-Scale Quantum) in 2018 to characterise quantum hardware with 50–1000 qubits, too noisy for full error correction but potentially useful for specific near-term applications. The NISQ era is defined by its constraints:

- Gate errors: typically 0.1–1% per two-qubit gate (vs <10⁻¹⁵ for classical logic).

- Limited circuit depth: decoherence limits the number of gates before the quantum state is corrupted.

- No fault tolerance: errors accumulate without correction.

- Limited qubit connectivity: not all qubit pairs can interact directly.

<div class="box box-example">
<p class="box-title"><strong>Example 1.5  NISQ vs Fault-Tolerant Requirements for Shor's Algorithm</strong></p>
<p>Running Shor's algorithm on RSA-2048 requires: • Logical qubits needed: ~4000 logical qubits • Physical qubits per logical qubit (surface code): ~5000 (at 0.1% physical error rate) • Total physical qubits: ~20 million • Current state (2024): IBM has 133-qubit Heron processor • Gap: factor of ~150,000× in qubit count • Gap: factor of ~100× in error rate (need &lt;0.1% vs current ~0.3%) Conclusion: Cryptographically relevant quantum computing requires ~10–15 more years of hardware improvement.</p>
</div>

### 1.6 The Global Quantum Computing Ecosystem

#### 1.6.1 Hardware Technology Comparison

Multiple physical platforms are being developed for quantum computing, each with distinct advantages and challenges. The radar chart below shows a comprehensive performance comparison:

<figure class="book-figure">
<img src="content/images/image6.png" alt="Figure 1.4: Quantum Hardware Technology Comparison (2024)">
<figcaption>Figure 1.4: Quantum Hardware Technology Comparison (2024)</figcaption>
</figure>

<div class="box box-example">
<p class="box-title"><strong>Example 1.6  Physical Qubit Technologies — Detailed Comparison</strong></p>
<p>Superconducting (IBM, Google): Qubit = Josephson junction circuit at 15 mK. Control = microwave pulses at 5 GHz. T₁ = 100–500 μs. 2-qubit gate fidelity = 99.5–99.9%. Gate time = 50 ns. Advantage: fast, mature technology. Disadvantage: requires extreme cooling. Trapped Ion (IonQ, Quantinuum): Qubit = ¹⁷¹Yb⁺ atom. Control = laser pulses. T₂ = seconds to hours. 2-qubit fidelity = 99.9%+. Gate time = 1–10 ms. Advantage: exceptional fidelity, all-to-all connectivity. Disadvantage: slow gates. Neutral Atom (QuEra, Pasqal): Qubit = Rb or Cs atom in optical tweezer. Control = laser pulses. T₁ = ~1s. Advantage: programmable connectivity, large arrays. Disadvantage: mid-circuit measurement difficult.</p>
</div>

<div class="box box-real-world">
<p class="box-title"><strong>🌐  India's National Quantum Mission — A Career Opportunity</strong></p>
<p>India's National Quantum Mission (NQM), approved by Cabinet in April 2023 with a budget of ₹6,003 crore (~$730 million) over 8 years, targets: quantum computers with 50–1000 qubits by 2031; quantum communication networks between major cities; satellite-based QKD; and training 25,000 quantum professionals. The NQM will establish National Quantum Technology and Application Hubs at IITs, IISc, DRDO, and other leading institutes. Private companies including TCS, Wipro, Infosys, and start-ups (QpiAI, BosonQ Psi) are actively hiring quantum-trained physicists. For M.Sc. Physics students graduating in the next 5 years, this is one of the most exciting career opportunities in Indian science and technology.</p>
</div>

### 1.7 What Quantum Computers Can and Cannot Do

Quantum computing advantage is problem-specific, not universal:

| Algorithm | Speedup Type | Problem | Application |
|---|---|---|---|
| Shor | Exponential | Factoring/discrete log | Break RSA, ECC cryptography |
| Grover | Quadratic (√N) | Unstructured search | Database search, NP verification |
| HHL | Exponential (caveat) | Linear systems Ax=b | Finance, ML, fluid dynamics |
| VQE/QAOA | Heuristic | Chemistry, optimisation | Drug discovery, logistics |
| Simulation | Polynomial | Quantum systems | Materials, condensed matter |

<div class="box box-warning">
<p class="box-title"><strong>⚠  Quantum Computers Are NOT Universal Speedup Machines</strong></p>
<p>Quantum computers do NOT speed up every computation. Running email, video games, or word processing on a quantum computer would be SLOWER than classical, not faster. The quantum advantage is problem-specific and depends on mathematical structure (periodicity for Shor, amplitude interference for Grover). For the vast majority of everyday computing tasks, classical computers remain optimal. Even for quantum-advantaged problems, the speedup only matters for large problem sizes — and the overhead of error correction is substantial.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 1.7  The Holevo Bound — A Fundamental Information Limit</strong></p>
<p>Despite n qubits having a 2ⁿ-dimensional state space, the Holevo bound (1973) limits accessible classical information:  χ ≤ S(ρ) − Σₓ pₓ S(ρₓ) ≤ log₂ d = n bits For n=10 qubits: state space = 2¹⁰ = 1024 amplitudes, but maximum extractable information = 10 classical bits. Consequence: Quantum parallelism computes on 2ⁿ amplitudes simultaneously but measurement collapses this to n bits. Quantum speedup requires algorithms that use interference to make the correct answer highly probable in a small number of measurements — not brute-force readout of all parallel computations.</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 1.8  Estimating Practical Quantum Advantage Threshold</strong></p>
<p>For Grover's algorithm to offer a real-world speedup over classical on a particular task: • Classical: N/2 checks at 10⁹/second • Quantum: (π/4)√N Grover iterations at rate R gates/second • Break-even when: N/(2×10⁹) = (π√N/4)/R → N = (π×10⁹/(2R))² For R = 10⁶ (realistic NISQ): N = (π×10³/2)² ≈ 2.5×10⁶ items For R = 10⁹ (fault-tolerant): N = (π×1/2)² ≈ 2.5 items (meaningless — classical wins for small N) Conclusion: Grover's algorithm is practically useful only for very large databases AND when the quantum computer is fast enough. For current NISQ hardware, the overhead is often too large.</p>
</div>

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 1: The Quantum Revolution: Computing at the Edge of Nature

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  What was Richard Feynman's key argument at the 1982 MIT conference regarding the simulation of quantum systems on classical computers?**

**Q2.  Define Moore's Law. At what transistor size (year ≈ 2023) has miniaturisation reached its physical limits, and what quantum-mechanical effect causes electrons to leak through transistor barriers?**

**Q3.  State the classical and quantum computational complexities for integer factorisation. What is the name of the classical algorithm used to factor large numbers, and how long would it take to factor RSA-2048 on a petaflop supercomputer?**

**Q4.  Write Grover's optimal iteration formula. For a database of N = 10⁶ items, how many oracle calls does Grover's algorithm require? Why is this speedup described as 'quadratic'?**

**Q5.  Who proposed the first quantum computing model, and in what year? What was its fundamental limitation compared to later quantum computing proposals?**

**Q6.  What does the acronym NISQ stand for? List four specific hardware limitations that characterise NISQ-era devices and explain how they prevent executing Shor's algorithm on RSA-2048 today.**

**Q7.  State the Holevo bound. If a quantum computer has 10 qubits with a 2¹⁰-dimensional state space, how many classical bits of information can be extracted from a single measurement? Explain the implication for quantum speedup.**

**Q8.  Describe the RSA public-key cryptography system in one paragraph. Which mathematical problem underpins its security, and why does Shor's algorithm threaten it? Name the three NIST post-quantum cryptographic standards (2024).**

**Q9.  Compare Shor's algorithm and Grover's algorithm on: (a) type of speedup, (b) class of problem addressed, (c) quantum mechanism exploited, and (d) practical impact on cryptography.**

**Q10.  What is India's National Quantum Mission (NQM)? State its budget (in INR and USD), timeline, qubit targets, and the number of quantum professionals it aims to train. Name two Indian private-sector companies actively hiring quantum-trained physicists.**

**Q11.  Google's Sycamore processor completed a task in 200 seconds in 2019. (a) How many qubits did it use? (b) What was Google's claim about classical computation time? (c) What revised time did IBM claim? (d) What does this controversy reveal about quantum supremacy claims?**

**Q12.  Why does Grover's algorithm NOT make NP-complete problems tractable? Use the example of 3-SAT with n = 100 variables to support your answer with an explicit time estimate.**

**Q13.  State the computational complexity of the four problems listed in the textbook's 'Exponential Complexity Wall' table. For each, give the problem name, classical complexity, and one practical consequence of that complexity.**

**Q14.  Explain the drug discovery crisis using quantum simulation as a solution. Why is classical simulation of penicillin's quantum dynamics infeasible? What scaling advantage does quantum simulation offer?**

**Q15.  Describe the three physical qubit technologies: superconducting, trapped ion, and neutral atom. For each, state the qubit type, control method, coherence time, and one key advantage and one disadvantage.**

**PART B — MODEL ANSWERS**

**Answer 1:**

At the 1981 MIT Physics of Computation conference, Feynman argued that the quantum state space of a many-particle system grows exponentially with the number of particles — O(2^n) — making exact classical simulation fundamentally intractable. To simulate 300 coupled quantum particles, a classical computer would need more memory than there are atoms in the observable universe. His constructive proposal was that a quantum computer, operating by quantum rules, could simulate quantum systems with only polynomial overhead, matching the structure of the simulated system.

**Answer 2:**

Moore's Law (Gordon Moore, 1965) states that the number of transistors on a microchip doubles approximately every two years. By 2023, Apple's M3 chip reached 3 nm transistors — roughly 15 silicon atoms wide, the current physical miniaturisation limit. The key quantum-mechanical effect causing failure at this scale is quantum tunnelling: electrons leak through transistor barrier walls because their quantum wavefunctions extend beyond the barrier, creating leakage currents even when the transistor is nominally 'off' — making logic unreliable and power dissipation unacceptable.

**Answer 3:**

Classical: GNFS (General Number Field Sieve), complexity O(exp(1.923 × n^{1/3} × (ln n)^{2/3})). For RSA-2048: approximately 10^{41} operations → ~3×10^{18} years on a petaflop (10^{15} ops/sec) supercomputer. Quantum: Shor's algorithm, O(n³). For RSA-2048: ~8.6×10^9 gates → approximately 2.4 hours at 10^6 gates/second. The exponential vs polynomial contrast is the entire basis for post-quantum cryptography urgency.

**Answer 4:**

Grover's optimal iterations: k\_opt = round(π√N / 4). For N = 10^6: k\_opt = round(π × 1000/4) = round(785.4) = 785 oracle calls. Classical average: N/2 = 500,000 checks. The speedup is 'quadratic' because the number of queries scales as √N rather than N — the ratio of classical to quantum calls is √N, growing as the square root of the database size.

**Answer 5:**

Paul Benioff at Argonne National Laboratory proposed the first quantum computing model in 1980 — a quantum-mechanical Turing machine. Its fundamental limitation was offering no computational advantage over classical computation: it operated quantum mechanically but achieved only classical computational power, not exploiting superposition or interference for speedup. The first genuinely advantageous models came from Feynman (1982, quantum simulation) and Deutsch (1985, universal quantum speedup).

**Answer 6:**

NISQ = Noisy Intermediate-Scale Quantum (coined by John Preskill, 2018). Four hardware limitations: (1) Gate errors: ~0.1–1% per two-qubit gate — errors accumulate with circuit depth. (2) Limited coherence time: T₁, T₂ ≈ 100–500 μs limits circuit depth to ~50–200 layers. (3) No fault tolerance: errors are not corrected — accumulate throughout computation. (4) Limited connectivity: heavy-hex topology means only 2–3 qubit neighbours, requiring costly SWAP routing. Shor's on RSA-2048 requires ~20 million physical qubits vs current ~133.

**Answer 7:**

The Holevo bound (1973): χ ≤ log₂ d = n bits, where d = 2^n is the Hilbert space dimension. For 10 qubits: despite a 2^{10} = 1024-dimensional state space, only 10 classical bits can be extracted per measurement. Implication: quantum parallelism computes on 2^n amplitudes simultaneously, but measurement collapses this to n bits. Quantum speedup requires interference to amplify the correct answer's amplitude before measurement — algorithms cannot simply 'read out' all 2^n results at once.

**Answer 8:**

RSA (Rivest–Shamir–Adleman, 1977): public key N = p × q is a product of two large primes. Encryption uses N; decryption requires knowing p and q. Security relies on the apparent hardness of factoring N. Shor's algorithm (1994) factors N in polynomial time O(n³), directly breaking RSA. The three NIST 2024 post-quantum standards are: CRYSTALS-Kyber (key encapsulation, lattice-based), CRYSTALS-Dilithium (digital signature, lattice-based), and SPHINCS+ (digital signature, hash-based).

**Answer 9:**

(a) Speedup: Shor — exponential (polynomial vs sub-exponential classical); Grover — quadratic (√N vs N). (b) Problem: Shor — integer factorisation and discrete logarithm; Grover — unstructured search over any database. (c) Mechanism: Shor — QFT to extract periodicity of modular exponentiation; Grover — amplitude amplification via phase oracle + diffusion operator. (d) Cryptography: Shor completely breaks RSA and elliptic-curve cryptography; Grover halves symmetric key security (AES-128 → 64-bit equivalent), requiring AES-256 for long-term security.

**Answer 10:**

India's NQM, approved April 2023: budget ₹6,003 crore (~$723 million USD) over 8 years (2023–2031). Qubit targets: 50 qubits by 2026, 1,000 qubits by 2031. Satellite-based QKD by 2028. Quantum communication networks between major cities. Training 25,000 quantum professionals. NQM hubs at IIT Madras, IIT Bombay, IISc Bangalore, TIFR Mumbai. Two companies: TCS (Tata Consultancy Services, has dedicated quantum computing lab) and QpiAI (quantum hardware and software startup, Bengaluru).

**Answer 11:**

(a) 53 qubits. (b) Google claimed Summit supercomputer would need ~10,000 years. (c) IBM's team argued optimised tensor network simulation could complete it in ~2.5 days. (d) The controversy reveals: 'quantum supremacy' claims are highly sensitive to which classical algorithm is chosen for comparison. The task was specifically designed to favour quantum hardware. The 4,000× difference between Google's and IBM's classical estimates shows how difficult it is to bound classical simulation cost, making absolute supremacy claims premature.

**Answer 12:**

Grover reduces search space from N to √N operations — but for NP-complete problems, N = 2^n (exponential search space). Grover gives O(√(2^n)) = O(2^{n/2}) operations — still exponential in n, just with n halved. For 3-SAT with n = 100: Grover = 2^50 ≈ 10^15 oracle calls. At 10^{15} calls/second, Grover still takes ~1 year — completely intractable. The BBBV theorem proves O(√N) is optimal for unstructured search, so NP-complete problems remain exponentially hard even for quantum computers.

**Answer 13:**

Integer Factorisation: GNFS complexity exp(O(n^{1/3} log^{2/3} n)) — RSA-2048 requires ~10^{13} years classically, underpinning internet security. Quantum Simulation: O(2^n) for n-electron system — 50-electron molecule is computationally intractable, bottlenecking drug discovery. Unstructured Search: O(N) classically — searching 10^{12} items takes ~500 seconds, limiting database lookup performance. Travelling Salesman: O(N!) factorial — 20 cities gives 2.4×10^{18} routes, exact solution impossible for large logistics networks.

**Answer 14:**

Penicillin has ~1,200 atoms and hundreds of electrons. Exact classical simulation requires memory and time scaling as O(2^n) where n = number of electrons — fundamentally infeasible even on the most powerful classical supercomputers. Drug development therefore relies on expensive approximate classical methods and 10–15 year, $2–3 billion laboratory trial-and-error. A quantum computer simulates molecular quantum dynamics in polynomial time O(poly(n)), because the quantum processor's state space naturally matches the molecular quantum state space — potentially compressing drug discovery from years to weeks.

**Answer 15:**

Superconducting (IBM, Google): Josephson junction circuit at 15 mK; microwave pulse control (10–50 ns); T₁ = 100–500 μs; advantage: fast gates, mature technology, large qubit counts; disadvantage: requires extreme cooling. Trapped Ion (IonQ, Quantinuum): ¹⁷¹Yb⁺ hyperfine levels; laser pulse control (~ms gates); T₂ = 1–10 seconds; advantage: exceptional fidelity (>99.9%), all-to-all connectivity; disadvantage: slow gates, complex laser infrastructure. Neutral Atom (QuEra, Pasqal): Rb/Cs in optical tweezers; laser excitation to Rydberg states; T₁ ≈ 1 s; advantage: programmable topology, large arrays (256 qubits); disadvantage: lower gate fidelity, mid-circuit measurement challenging.

<div class="box box-generic">
<p class="box-title"><strong>END OF CHAPTER 1 — EXERCISES &amp; PROBLEMS</strong></p>

</div>

#### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.1</strong></p>
<p><strong>Problem:</strong> Estimate the number of transistors per square millimetre in Intel's 4004 (1971, transistor width 10 μm) and Apple's M3 (2023, transistor width 3 nm). What is the ratio?</p>
<p><strong>Solution:</strong> Intel 4004 (10 μm = 10,000 nm): Area of one transistor ≈ (10,000 nm)² = 10⁸ nm² = 0.1 mm². Density ≈ 1/0.1 = 10 transistors/mm². Apple M3 (3 nm): Area ≈ (3 nm)² = 9 nm². Density ≈ 1/(9×10⁻¹² mm²) ≈ 1.1×10¹¹ transistors/mm². Ratio: 1.1×10¹¹ / 10 = 1.1×10¹⁰ — approximately 11 billion times more dense. This perfectly illustrates Moore's Law over 52 years.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.2</strong></p>
<p><strong>Problem:</strong> A classical computer running the GNFS algorithm at 10¹⁵ operations/second is used to factor RSA-2048 (n=2048 bits). Estimate the computation time in years.</p>
<p><strong>Solution:</strong> GNFS operations ≈ exp(1.923 × n^(1/3) × (ln n)^(2/3)) with n=2048. n^(1/3) = 2048^(1/3) ≈ 12.70. ln(2048) = ln(2¹¹) = 11 ln 2 ≈ 7.624. (7.624)^(2/3) ≈ 3.87. Exponent = 1.923 × 12.70 × 3.87 ≈ 94.6. Operations ≈ e^94.6 ≈ 10^41. Time = 10^41/10^15 = 10^26 seconds. One year ≈ 3.15×10⁷ seconds. Time ≈ 10^26/3.15×10⁷ ≈ 3.2×10¹⁸ years ≈ 2.3×10⁸ times the age of the universe. Completely infeasible.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.3</strong></p>
<p><strong>Problem:</strong> Grover's algorithm is run on a quantum computer to search a database of 4 million (4×10⁶) items. How many oracle calls are needed? If each oracle call takes 1 μs, what is the total search time? Compare with classical average.</p>
<p><strong>Solution:</strong> Grover iterations: k_opt = round(π/4 × √N) = round(π/4 × √(4×10⁶)) = round(π/4 × 2000) = round(1570.8) = 1571 iterations. Quantum search time: 1571 × 1 μs = 1571 μs ≈ 1.57 ms. Classical average: N/2 = 2×10⁶ checks. At 10⁹ checks/second: 2×10⁶/10⁹ = 2 ms. Grover is ~1.27× faster. Speedup grows with N — for N=4×10¹², speedup would be ~1270×.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.4</strong></p>
<p><strong>Problem:</strong> Compare the number of qubits required to run Shor's algorithm on RSA-512 vs RSA-4096, using the approximation that ~5n physical qubits are needed where n is the key size.</p>
<p><strong>Solution:</strong> RSA-512: Physical qubits ≈ 5 × 512 = 2,560 (for a fault-tolerant implementation at 0.1% error rate). RSA-4096: Physical qubits ≈ 5 × 4096 = 20,480. Ratio: 4096/512 = 8. Physical qubits scale linearly with key size (as expected for O(n³) algorithm). Note: More careful estimates give ~20 million physical qubits for RSA-2048 (including full surface code overhead). Current hardware: 133 qubits (IBM Heron, 2024). Gap: factor of ~150,000.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.5</strong></p>
<p><strong>Problem:</strong> India's National Quantum Mission has a budget of ₹6,003 crore over 8 years. Convert this to US dollars and compare with the US National Quantum Initiative annual budget ($900 million/year).</p>
<p><strong>Solution:</strong> ₹6,003 crore = ₹60,030,000,000 = ₹6.003×10¹⁰. At ₹83/$ exchange rate: $6.003×10¹⁰/83 ≈ $7.23×10⁸ ≈ $723 million over 8 years. Annual NQM budget: $723M/8 = ~$90 million/year. US NQI budget: $900 million/year — approximately 10× larger annually. However, India's NQM is 90% focused on domestic research and training, creating proportionally more career opportunities for Indian students and researchers.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.6</strong></p>
<p><strong>Problem:</strong> The AES-128 symmetric encryption key length effectively becomes 64-bit secure against Grover's attack. If cracking AES-128 classically takes 10³⁸ years, how long does it take with Grover's algorithm?</p>
<p><strong>Solution:</strong> Grover gives a quadratic speedup: classical O(2¹²⁸) → quantum O(√(2¹²⁸)) = O(2⁶⁴) operations. Speedup factor: 2¹²⁸/2⁶⁴ = 2⁶⁴ ≈ 1.8×10¹⁹. Quantum time estimate: 10³⁸ years / 1.8×10¹⁹ ≈ 5.6×10¹⁸ years. This is still astronomically large! Even with Grover, AES-128 requires 2⁶⁴ operations — about 1.8×10¹⁹ operations. At 10⁶ Grover operations/second: 1.8×10¹⁹/10⁶ = 1.8×10¹³ seconds ≈ 570,000 years. AES-128 is NOT broken by quantum computers — though AES-256 is recommended for post-quantum security.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.7</strong></p>
<p><strong>Problem:</strong> A NISQ device has 100 qubits with 0.3% two-qubit gate error rate and maximum circuit depth of 100 gates before fidelity drops below 50%. What is the maximum number of two-qubit gates that can be applied?</p>
<p><strong>Solution:</strong> The fidelity after k two-qubit gates each with error rate p = 0.003 is approximately: F ≈ (1-p)^k = (0.997)^k. For F = 0.5: 0.997^k = 0.5 → k × ln(0.997) = ln(0.5) → k = ln(0.5)/ln(0.997) = −0.693/−0.003004 ≈ 231 gates. So approximately 231 two-qubit gates maintain &gt;50% fidelity — consistent with the stated depth limit of ~100 for more complex circuits with additional sources of error. This limits the class of algorithms executable on NISQ hardware to those with very shallow circuits.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 1.8</strong></p>
<p><strong>Problem:</strong> Google's Sycamore performed a task in 200 seconds that a classical computer will take 10,000 years. IBM claimed Summit could do it in 2.5 days (216,000 seconds). Calculate the 'quantum advantage' ratio claimed by Google and the revised ratio by IBM.</p>
<p><strong>Solution:</strong> Google's claimed speedup: t_classical/t_quantum = (10,000 × 365.25 × 24 × 3600) / 200 = 3.16×10¹¹ / 200 ≈ 1.58×10⁹× (about 1.6 billion times faster). IBM's revised speedup: 216,000 / 200 = 1,080× — about 1000 times faster. Both are quantum advantages but differ by a factor of ~1.5 million. The discrepancy shows how sensitive 'quantum supremacy' claims are to the choice of classical algorithm and simulation method. IBM used tensor network contraction methods not available in 2019. The true speedup for practical applications remains an open and actively debated question.</p>
</div>

#### B. Unsolved Problems

Solve the following problems independently. Answers are provided in brackets for self-checking.

**1.** A classical computer performs 10¹² floating-point operations per second. How many years would it take to check all 2⁸⁰ possible AES-80-bit keys? How many Grover iterations would a quantum computer require?  [Answer: Classical: ~3.8×10⁷ years; Quantum: ~6×10¹¹ Grover iterations]

**2.** The first quantum algorithm was proposed by Deutsch for n=1 input bit. How many evaluations does it save compared to classical? What is the saving for the Deutsch-Jozsa problem with n=50?  [Answer: n=1: saves 1 query (1 vs 2); n=50: saves 2⁴⁹ queries (1 vs 2⁴⁹+1 = ~5.6×10¹⁴)]

**3.** A quantum computer has 127 qubits (IBM Eagle). What is the size of its Hilbert space? How much classical memory (in bytes) would be needed to store the full quantum state?  [Answer: 2¹²⁷ ≈ 1.7×10³⁸ complex amplitudes; memory ≈ 2.7×10³⁹ bytes — incomprehensibly larger than all classical storage on Earth]

**4.** If Grover's algorithm has optimal iterations k\_opt = π√N/4, compute k\_opt for N = 1, 4, 16, 64, 256. What pattern do you notice?  [Answer: k\_opt = 0, 1, 3, 6, 12 respectively; doubles roughly for every 4× increase in N]

**5.** The CNOT gate has a 2-qubit error rate of 0.5% on a NISQ device. What is the fidelity of a circuit using 10, 50, and 200 CNOT gates?  [Answer: F = (0.995)^k: 10 gates: 95.1%; 50 gates: 77.8%; 200 gates: 36.7%]

**6.** India's NQM budget of ₹6,003 crore is spread over 8 years to train 25,000 quantum professionals. What is the average investment per quantum professional trained?  [Answer: ₹6,003 crore / 25,000 ≈ ₹2.4 lakh per professional (~$2,900 — extremely cost-effective)]

**7.** Estimate how many bits of classical information a 50-qubit quantum state encodes. How many classical bits can be extracted from a single measurement of this state?  [Answer: Classical description needs 2×2⁵⁰ real numbers ≈ 10¹⁵ parameters; only 50 bits extractable per measurement]

**8.** The NQM aims to build a 50-qubit quantum computer by 2028 and a 1000-qubit system by 2031. If error rates decrease by 50% every 2 years from 1% in 2024, what will the error rate be in 2031?  [Answer: After 3.5 doublings (7 years): 1% × (0.5)^3.5 ≈ 0.088% — approaching the fault-tolerance threshold of ~0.1%]

**9.** A quantum computer running Shor's algorithm requires O(n²) qubits for an n-bit integer. How many qubits are needed for RSA-1024, RSA-2048, RSA-4096?  [Answer: RSA-1024: ~1 million; RSA-2048: ~4 million; RSA-4096: ~16 million (including error correction overhead)]

**10.** The Holevo bound states at most n bits can be extracted from n qubits. A communications protocol sends 10 qubits. What is the maximum number of classical bits that can be transmitted? If superdense coding (using entanglement) is used, how many bits can be sent?  [Answer: Without entanglement: 10 bits maximum (Holevo); With superdense coding (1 ebit shared): 20 classical bits from 10 qubits — Holevo does not apply since shared entanglement acts as extra resource]

#### C. Multiple Choice Questions

Note: Answers to all MCQs are given at the end of this section.

**Q1.** Which physicist first proposed using quantum systems to simulate other quantum systems in 1982?

(a)  Charles Bennett (IBM)

(b)  Peter Shor (Bell Labs)

(c)  Richard Feynman (Caltech)

(d)  David Deutsch (Oxford)

**Q2.** Shor's algorithm solves integer factorisation in polynomial time. Its time complexity for an n-bit number is:

(a)  O(n log n)

(b)  O(n²)

(c)  O(n³)

(d)  O(2ⁿ)

**Q3.** Grover's search algorithm achieves what type of speedup over classical unstructured search?

(a)  Exponential (2ⁿ to polynomial)

(b)  Quadratic (N to √N)

(c)  Logarithmic

(d)  No speedup — same complexity

**Q4.** The BBBV theorem (1994) proves that O(√N) is:

(a)  An upper bound for quantum search

(b)  The exact optimal for Grover's algorithm

(c)  The optimal quantum query complexity for unstructured search

(d)  A lower bound for classical search

**Q5.** The NISQ era refers to quantum devices with approximately how many qubits?

(a)  1–10 qubits

(b)  50–1000 noisy qubits

(c)  1 million fault-tolerant logical qubits

(d)  Unlimited — NISQ refers to error-free operation

**Q6.** The Holevo bound states that from n qubits, at most how many classical bits can be accessed?

(a)  2ⁿ bits

(b)  n² bits

(c)  n bits

(d)  log₂(n) bits

**Q7.** Which type of cryptography is directly threatened by Shor's algorithm?

(a)  AES-256 symmetric encryption

(b)  RSA and ECC public-key cryptography

(c)  Hash-based cryptography (SHA-256)

(d)  Post-quantum cryptography (CRYSTALS-Kyber)

**Q8.** India's National Quantum Mission (NQM) was approved in:

(a)  2019

(b)  2021

(c)  2022

(d)  2023

**Q9.** Google's Sycamore quantum supremacy experiment (2019) used how many qubits?

(a)  20

(b)  53

(c)  100

(d)  433

**Q10.** Which classical computation is best suited for quantum computing advantage?

(a)  Video encoding (H.264)

(b)  Email encryption using AES-256

(c)  Integer factorisation (RSA keys)

(d)  Sorting a list of 10⁹ numbers

**Q11.** The drug penicillin has ~1200 atoms. Classical exact simulation of its quantum dynamics scales as:

(a)  O(1200²)

(b)  O(1200)

(c)  O(2^1200)

(d)  O(log 1200)

**Q12.** Which of these is NOT an example of quantum advantage?

(a)  Factoring large integers (Shor)

(b)  Searching an unsorted database (Grover)

(c)  Sorting a classical list (quicksort)

(d)  Simulating quantum chemistry (VQE)

**Q13.** The first NISQ-era hardware available for public use via the cloud was:

(a)  Google Sycamore (2019)

(b)  IBM Quantum Experience (2016)

(c)  D-Wave Advantage (2020)

(d)  IonQ (2021)

**Q14.** If a quantum computer has a 2-qubit gate error rate of 1%, what is the maximum fidelity after 50 such gates?

(a)  99%

(b)  61% approximately

(c)  50%

(d)  0%

**Q15.** Which quantum algorithm has the broadest applicability despite only giving a quadratic speedup?

(a)  Shor's algorithm

(b)  HHL linear systems

(c)  Grover's amplitude amplification

(d)  Quantum Fourier Transform

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 1</strong></p>
<p><strong>Q1: (c) Richard Feynman  —</strong> 1982 MIT conference lecture 'Simulating Physics with Computers'</p>
<p><strong>Q2: (c) O(n³)  —</strong> Polynomial in the number of bits — exponential speedup over GNFS</p>
<p><strong>Q3: (b) Quadratic (N to √N)  —</strong> Searches N items in O(√N) oracle calls, provably optimal</p>
<p><strong>Q4: (c) The optimal quantum query complexity for unstructured search  —</strong> No quantum algorithm can beat O(√N) for unstructured search</p>
<p><strong>Q5: (b) 50–1000 noisy qubits  —</strong> As defined by John Preskill in 2018</p>
<p><strong>Q6: (c) n bits  —</strong> Despite 2ⁿ-dimensional state space — Holevo's theorem 1973</p>
<p><strong>Q7: (b) RSA and ECC public-key cryptography  —</strong> Both rely on factoring or discrete logarithm — solved by Shor</p>
<p><strong>Q8: (d) 2023  —</strong> Approved by Indian Cabinet in April 2023 with ₹6,003 crore budget</p>
<p><strong>Q9: (b) 53  —</strong> 53-qubit Sycamore processor, task completed in 200 seconds</p>
<p><strong>Q10: (c) Integer factorisation (RSA keys)  —</strong> Shor's algorithm gives exponential speedup for this problem</p>
<p><strong>Q11: (c) O(2^1200)  —</strong> Exponential in number of electrons — exactly Feynman's argument for quantum simulation</p>
<p><strong>Q12: (c) Sorting a classical list  —</strong> Sorting has no quantum speedup — it's a classical data manipulation task</p>
<p><strong>Q13: (b) IBM Quantum Experience (2016)  —</strong> IBM made quantum hardware publicly available via cloud in May 2016</p>
<p><strong>Q14: (b) ~61%  —</strong> F = (1-0.01)^50 = 0.99^50 ≈ 0.605 ≈ 61%</p>
<p><strong>Q15: (c) Grover's amplitude amplification  —</strong> Applies to any search/verification problem — universally applicable quadratic speedup</p>
</div>

#### D. Theory Questions

- Explain in your own words why Feynman argued in 1982 that classical computers cannot efficiently simulate quantum systems. What is the fundamental bottleneck, and how does quantum computing overcome it?

- Compare and contrast Shor's and Grover's algorithms on four dimensions: (a) type of speedup (exponential vs quadratic), (b) class of problems addressed, (c) quantum mechanism exploited (QFT vs amplitude amplification), (d) practical implications for cryptography and security.

- What is the NISQ era? List four specific limitations of NISQ devices. How do these limitations prevent current hardware from running Shor's algorithm on RSA-2048?

- Explain the Holevo bound. Why does it imply that quantum parallelism alone does not give exponential advantage in general computation? Give a specific example to illustrate.

- Describe India's National Quantum Mission: its goals, budget, timeline, and key career opportunities it creates for M.Sc. Physics graduates. Compare with quantum programmes of USA, China, and EU.

- What is quantum supremacy (or quantum advantage)? Describe the Google Sycamore experiment of 2019. Why was it controversial, and what does it definitively prove about quantum computers?

- Explain why quantum computers cannot solve all NP-hard problems efficiently. Use Grover's algorithm applied to 3-SAT as a specific example to show that even with quadratic speedup, the problem remains exponential.

- What is post-quantum cryptography? Name the three NIST-standardised post-quantum algorithms (2024). Why does the existence of Shor's algorithm not immediately make RSA insecure today?

- Describe Moore's Law. What physical effects are causing it to slow down at the nanometre scale? How does quantum computing offer an alternative path forward?

- Explain the concept of quantum simulation using the drug penicillin as an example. Why is exact classical simulation of its quantum mechanics infeasible? What could a quantum computer do that no classical computer can?

#### E. Programming Assignments

- [Qiskit — Basic] Set up your IBM Quantum account. Write a 10-line Qiskit program that creates a 2-qubit Bell state, simulates 2048 shots, plots the histogram, and verifies that only |00⟩ and |11⟩ appear. Submit code and histogram.

- [Research] Using quantum.ibm.com, find the current least-busy IBM Quantum processor with at least 5 qubits. Record: processor name, qubit count, quantum volume, T₁ and T₂ for 3 qubits, and 2-qubit gate error rate. Write a half-page summary of what these metrics mean.

- [Analysis] Using the Grover formula k\_opt = π√N/4, calculate the number of Grover iterations for N = 10², 10⁴, 10⁶, 10⁸, 10¹², 10¹⁸, 10²⁴. For each, compute the classical average search time (N/2 at 10¹⁰ checks/second) and quantum search time (k\_opt at 10⁶ iterations/second). Plot both on a log-log graph. At what N does the quantum computer first win?

#### F. Project Suggestions

- Project 1.A — India's Quantum Ecosystem: Research and map India's quantum computing landscape. Identify: (a) all academic groups working on quantum computing (IITs, IISc, TIFR, HRI), (b) government programmes (NQM, C-DAC, DRDO), (c) private companies (TCS, Wipro, start-ups), (d) international collaborations. Create a visual map and a 2500-word report on career opportunities, funding sources, and India's strengths and gaps compared to USA and China.

- Project 1.B — Post-Quantum Cryptography Implementation: Study the CRYSTALS-Kyber key encapsulation mechanism. Implement it in Python using the open-source liboqs library. Benchmark its performance (key generation, encryption, decryption speed) against RSA-2048 and RSA-4096. Evaluate the trade-offs in key size, speed, and security. Write a 2000-word technical report.

- Project 1.C — Quantum Advantage Analysis: For 5 specific real-world problems (choose from: drug simulation, portfolio optimisation, traffic routing, protein folding, weather forecasting), research: (a) what is the best known classical algorithm complexity, (b) is there a known quantum speedup, (c) what scale of quantum hardware would be needed for practical advantage, (d) what is the current state of quantum approaches. Write a 3000-word comparative report.
