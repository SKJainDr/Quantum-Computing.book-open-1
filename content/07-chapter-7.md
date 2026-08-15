# Unit IV - CHAPTER 7: Quantum Hardware and Noise

# Physical Platforms · Transmon Circuit · Coherence · Quantum Channels · Noise Models · Benchmarks

<div class="box box-anecdote">
<p class="box-title"><strong>📜  The Josephson Junction — Cambridge, 1962</strong></p>
<p>In 1962, Brian Josephson was a 22-year-old PhD student at Cambridge University when he predicted that Cooper pairs of electrons could tunnel quantum mechanically through an insulating barrier between two superconductors — without any applied voltage. His supervisor, Philip Anderson, initially doubted the result; Nobel laureate John Bardeen publicly argued against it. Josephson was right. The Josephson effect was experimentally confirmed within months, and Josephson received the Nobel Prize in Physics in 1973.</p>
<p>The Josephson junction became the fundamental building block of superconducting quantum computing. Researchers in the 1980s and 1990s discovered that Josephson junction circuits could behave as quantum two-level systems — artificial atoms with quantised, controllable energy levels. In 1999, Nakamura, Pashkin, and Tsai at NEC Japan demonstrated the first superconducting qubit. The transmon qubit, developed at Yale in 2007 by Koch, Charge, and Schoelkopf, addressed the critical weakness of early designs (sensitivity to charge noise) and became the dominant architecture used in IBM Quantum, Google Sycamore, and dozens of other platforms worldwide.</p>
<p>This chapter bridges the gap between the abstract circuit model and physical quantum hardware. Understanding hardware is not optional: gate times, error rates, connectivity topologies, and noise channels all flow directly from the physics of the chosen qubit platform. Every algorithm developed in Units II and III must ultimately be mapped onto the physical constraints studied here.</p>
</div>

We examine five qubit platforms in detail — superconducting, trapped-ion, photonic, spin, and neutral atom — and develop the transmon qubit from its Josephson junction physics through microwave control to circuit QED readout. We then build the mathematical framework for quantum noise: coherence times T1 and T2, the Kraus operator formalism for quantum channels, and Qiskit Aer noise models for realistic circuit simulation. The chapter concludes with Quantum Volume and CLOPS — the two benchmark metrics that quantify and compare hardware capability — and a frank assessment of NISQ era limitations and the road to fault tolerance.

## 7.1 Physical Qubit Platforms

A qubit can be realised in any quantum system with two distinguishable energy levels that can be prepared, controlled, and measured with sufficient precision. The challenge is simultaneously achieving: (i) long coherence times so the quantum state survives long enough for the computation to complete; (ii) high-fidelity gate operations with low error rates; and (iii) scalability to the thousands or millions of qubits needed for fault-tolerant applications. No platform perfectly satisfies all three — each makes different engineering trade-offs, and the choice of platform fundamentally shapes what algorithms can run and with what performance.

<figure class="book-figure">
<img src="content/images/image79.png" alt="Figure 7.1: Physical qubit platform comparison (2024). Left: Radar chart of capability across six key metrics — superconducting qubits lead in scale and speed; trapped ions in fidelity and connectivity; neutral atoms in reconfigurability. Right: T2 coherence times by platform on a logarithmic scale, illustrating the seven-orders-of-magnitude range from superconducting (~300 μs) to trapped-ion (~10⁶ μs = ~1 s). Bar heights reflect best reported values; typical production devices are 3–5× lower.">
<figcaption>Figure 7.1: Physical qubit platform comparison (2024). Left: Radar chart of capability across six key metrics — superconducting qubits lead in scale and speed; trapped ions in fidelity and connectivity; neutral atoms in reconfigurability. Right: T2 coherence times by platform on a logarithmic scale, illustrating the seven-orders-of-magnitude range from superconducting (~300 μs) to trapped-ion (~10⁶ μs = ~1 s). Bar heights reflect best reported values; typical production devices are 3–5× lower.</figcaption>
</figure>

### 7.1.1 Superconducting Qubits (IBM, Google)

Superconducting qubits are the most advanced and widely deployed quantum technology today. They operate at millikelvin temperatures (10–20 mK, colder than interstellar space) inside dilution refrigerators, where the electrical resistance of metallic circuits vanishes completely. The qubit is encoded in the quantised energy levels of a superconducting LC circuit containing one or more Josephson junctions.

#### Physical Implementation

The circuit consists of a superconducting loop (typically aluminium or niobium) approximately 100–500 micrometres in size, patterned on a silicon or sapphire chip using electron-beam lithography. The Josephson junction — a nanometre-thin aluminium oxide tunnel barrier between two Al electrodes — provides an essential nonlinearity. Without this nonlinearity, the circuit would behave as a harmonic oscillator with equally-spaced energy levels; a microwave drive resonant with the |0⟩→|1⟩ transition would simultaneously drive |1⟩→|2⟩, preventing selective qubit control. The Josephson junction creates an anharmonic energy level structure: the |0⟩→|1⟩ spacing differs from |1⟩→|2⟩ by the anharmonicity α ≈ −200 to −350 MHz, which is large enough that a narrowband microwave pulse can selectively address only the qubit transition.

#### Control and Readout

Qubits are controlled by microwave pulses at 4–8 GHz, delivered through coaxial lines from room-temperature electronics. Single-qubit gates are implemented by shaped microwave pulses (Gaussian or DRAG envelopes) of duration 10–50 ns. Two-qubit gates (cross-resonance on IBM, parametric coupling on Google) operate in 100–300 ns. Qubit state is read out by probing a coupled microwave resonator — see Section 7.2.4 for the circuit QED readout mechanism.

#### Performance Parameters (2024)

- T1 relaxation: 50–500 μs (IBM Heron R2 median ~200 μs; best qubits ~500 μs)

- T2 dephasing: 50–300 μs (limited by 1/f charge and flux noise)

- Single-qubit gate fidelity: >99.9% (errors <0.1%)

- Two-qubit (CX/CNOT) gate fidelity: 99.0–99.8%

- Readout fidelity: 97–99.5%

- Gate time: single-qubit 10–50 ns; two-qubit 100–300 ns; readout 300–800 ns

- Scale: IBM Condor (1,121 qubits, 2023); Google Willow (105 qubits, 2024)

- Connectivity: heavy-hex lattice (IBM) — each qubit couples to 2–3 neighbours

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: IBM Quantum Network and the Indian Quantum Ecosystem</strong></p>
<p>IBM Quantum provides cloud-based access to over 100 processors through the IBM Quantum Network, which includes 220+ member organisations across 30+ countries. Indian institutions with IBM Quantum Premium Access include IIT Madras, IIT Bombay, IISc Bangalore, and TIFR Mumbai. IBM's 10-year quantum roadmap targets "quantum-centric supercomputing" by 2033: modular systems with 100,000+ qubits linked by quantum communication.</p>
<p>India's National Quantum Mission (NQM, ₹6,003 crore, 2023–2031) specifically targets indigenous superconducting qubit development at IIT Madras, IIT Bombay, and TIFR. IIT Madras has demonstrated a 2-qubit superconducting processor (2022); TIFR Mumbai operates a dilution refrigerator and is developing flux-tunable transmon qubits. For M.Sc. students at Indian institutions, cloud access to IBM Quantum through department agreements provides hands-on quantum programming experience without needing local hardware.</p>
</div>

### 7.1.2 Trapped-Ion Qubits (IonQ, Quantinuum)

Trapped-ion quantum computers encode qubits in the internal hyperfine energy levels of individual atomic ions suspended in vacuum by electromagnetic fields. These are among the oldest and highest-fidelity quantum computing platforms. The qubit arises from natural atomic physics: specific internal states of individual atoms, which are identical to each other by the laws of quantum mechanics. This reproducibility — every ytterbium-171 ion in the universe is exactly the same — is a fundamental advantage over manufactured solid-state qubits, which suffer from fabrication variability.

#### Physical Implementation

Ions — typically ytterbium-171 (¹⁷¹Yb⁺) at IonQ, or calcium-40 (⁴⁰Ca⁺) at Quantinuum — are produced by laser-ionising a neutral atom beam and trapped in a Paul trap: a combination of static and oscillating radiofrequency electric fields that confines the ions in a linear chain inside an ultra-high vacuum chamber (pressure ~10⁻¹¹ Torr). The trap operates at room temperature, but the ions are laser-cooled to millikelvin temperatures, where their motional state approaches the quantum ground state.

Qubits are encoded in two hyperfine ground states separated by a microwave or optical transition. For ¹⁷¹Yb⁺ at IonQ, the qubit transition is at 12.6 GHz (microwave); for ⁴⁰Ca⁺ at Quantinuum, it is at 729 nm (optical). Individual ions are addressed and controlled by tightly focused laser beams — typically 355 nm UV for ¹⁷¹Yb⁺. Readout is performed by fluorescence detection: one qubit state scatters photons brightly under a detection laser, the other does not.

#### Two-Qubit Gates via Phonon Bus

The key mechanism for two-qubit gates in trapped ions is the shared motional modes of the ion chain. The Coulomb repulsion between ions means they are coupled: displacing one ion shifts the equilibrium positions of all others. This creates normal modes of collective vibration (phonon modes) that can serve as a quantum bus connecting any two ions in the chain. By exciting one ion into a superposition of motional states and then transferring this motional qubit to a second ion, a two-qubit entangling gate (Mølmer-Sørensen gate) can be implemented between any pair of ions — giving full all-to-all connectivity without any SWAP routing.

#### Performance Parameters (2024)

- T1 relaxation: >10 seconds (limited only by background gas collisions and off-resonant photon scattering)

- T2 dephasing: 1–10 seconds (Quantinuum H2-2 reports T2 > 1 minute for nuclear spin qubits)

- Single-qubit gate fidelity: 99.99%+ (best in any platform)

- Two-qubit (MS) gate fidelity: 99.5–99.9%

- Connectivity: all-to-all (any pair of ions can be entangled directly)

- Gate time: single-qubit ~1 μs; two-qubit ~100 μs–1 ms (much slower than superconducting)

- Scale: IonQ Forte (35 qubits); Quantinuum H2-2 (56 qubits, 2024)

- Readout fidelity: ~99.8%

The all-to-all connectivity is transformative: a 56-qubit trapped-ion device can execute circuits requiring fewer total gates than a 56-qubit superconducting device with nearest-neighbour connectivity, because no SWAP gates are needed to route interactions between non-adjacent qubits. This connectivity advantage partially compensates for the slower gate speeds. IonQ reported QV = 512 in 2022 with only 25 physical qubits — demonstrating that qubit quality, not quantity, determines computational power.

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: Gate Speed vs. Coherence Time: Context Matters</strong></p>
<p>Trapped-ion gates are 1,000–10,000× slower than superconducting gates, but this does not mean they are inherently worse. What matters for algorithm execution is the number of gate operations that can be completed within one coherence time. A trapped-ion qubit with T2 = 1 second and gate time = 1 ms can execute 1,000 two-qubit gates/T2. A superconducting qubit with T2 = 100 μs and gate time = 300 ns executes ~333 gates/T2. Despite being 3,333× slower in absolute gate speed, the trapped-ion device achieves 3× more operations per coherence time. For algorithms requiring high fidelity on tens of qubits with limited depth, trapped ions can outperform superconducting systems despite their slower gates. The comparison is always algorithm-specific.</p>
</div>

### 7.1.3 Photonic Qubits

Photonic quantum computing encodes qubits in the properties of individual photons: polarisation (horizontal |H⟩ = |0⟩, vertical |V⟩ = |1⟩), path (which waveguide the photon travels), or time-bin (early or late arrival within a time window). Photons have extraordinary advantages: they travel at the speed of light, interact almost not at all with the environment at room temperature (essentially infinite coherence time), and are natural carriers of quantum information over long distances — the same photons used for qubits can carry quantum information through optical fibre networks.

#### Physical Implementation

Photonic quantum computers use integrated photonic chips containing beam splitters, phase shifters, and single-photon detectors fabricated in silicon or silicon nitride waveguides. Single photons are generated by quantum dot emitters, spontaneous parametric down-conversion (SPDC) in nonlinear crystals, or on-chip sources. The fundamental optical operations — beam splitters, phase shifters, and polarising optics — implement linear transformations on the photon field.

#### Key Properties and Fundamental Challenge

- Coherence: effectively infinite — photons do not decohere in vacuum or in optical fibre at room temperature

- Single-qubit gates: trivially implemented by beam splitters and phase shifters (linear optics)

- Critical limitation: linear optics CANNOT implement deterministic two-qubit gates (KLM theorem, Knill, Laflamme, Milburn 2001). Any linear optical two-qubit gate has success probability ≤ 1/2 per attempt, scaling down exponentially with circuit depth

- Solutions: measurement-based computing (cluster states), Gaussian Boson Sampling (Xanadu), error-corrected photonic computing (PsiQ, Xanadu Borealis)

- Best current use: quantum communication (QKD), boson sampling (quantum advantage demonstrations), quantum networking

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: Photonic Qubits: Specialised, Not Universal</strong></p>
<p>Photonic quantum computers as currently deployed (Xanadu Borealis, PsiQ) are NOT universal gate-model computers equivalent to IBM or IonQ systems. Standard linear optical quantum computing requires exponentially large ancilla resources for reliable universal gates. Current photonic devices are optimised for Gaussian Boson Sampling — a specific problem that demonstrates quantum advantage but does not directly solve practical optimisation or simulation problems. Universal fault-tolerant photonic quantum computing using topological cluster states and measurement-based computation is an active research direction but is not yet competitive with superconducting or trapped-ion systems for general algorithms.</p>
</div>

### 7.1.4 Spin Qubits

Spin qubits encode quantum information in the spin state of an individual electron or nucleus confined in a semiconductor quantum dot or in a crystal defect. The qubit is the simplest possible quantum system: spin-up |↑⟩ = |0⟩ and spin-down |↓⟩ = |1⟩. Spin qubits are attractive because they are extraordinarily small (nanometre scale), potentially manufacturable using the same CMOS fabrication processes as classical computer chips, and can exploit nuclear spin as an ultra-long-lived quantum memory.

#### Types of Spin Qubits

- Electron spin qubit in Si/SiGe quantum dot (Intel, TU Delft): ~50 nm per qubit; operates at 1–4 K

- Electron spin in GaAs quantum dot (early platform; limited by nuclear spin bath causing fast dephasing)

- Nitrogen-vacancy (NV) centre in diamond: defect qubit operable at room temperature; T2 ~ms

- Phosphorus-31 donor in silicon (Kane proposal): nuclear spin T1 > 30 minutes in isotopically purified ²⁸Si

#### Performance Parameters (Silicon Spin, 2024)

- Size: ~50 nm per qubit — 1,000× smaller than superconducting qubits

- Operating temperature: 1–4 K (potentially compatible with cryo-CMOS control circuits)

- T1 (electron spin, Si): ~10–100 ms; T1 (nuclear spin, ²⁸Si): >10 hours

- T2 (electron, Si): 1–10 ms with dynamical decoupling; T2\* ~1 μs without

- Single-qubit fidelity: 99.8%+ (Si electron spin, 2022)

- Two-qubit fidelity: 96–99% (improving rapidly; TU Delft 99.5% in 2022)

- Scale: 6-qubit Si processor (Intel/TU Delft, 2023); scaling remains challenging

The long-term vision for spin qubits is compelling: a silicon quantum chip manufactured using standard semiconductor equipment could one day have millions of qubits. The NV-centre in diamond is uniquely suited for room-temperature quantum sensing (magnetic field detection, biological imaging) while simultaneously serving as a quantum register for small-scale computation.

### 7.1.5 Platform Comparison and Selection Guide

Choosing a qubit platform for a given application requires analysing the algorithmic requirements (circuit depth, qubit count, connectivity pattern) against the hardware capabilities. A useful figure of merit is the number of two-qubit operations per coherence time (T2 / t\_2Q\_gate):

| Platform | T₂/Gate (ops/T₂) | Advantage | Disadvantage | Best Application | Leaders |
|---|---|---|---|---|---|
| Superconducting | ~3×10⁵ ops/T₂ | Scale, speed, maturity | Connectivity, cryo, crosstalk | NISQ VQE/QAOA; large-scale | IBM, Google, Rigetti |
| Trapped Ion | ~10⁶ ops/T₂ | Fidelity, all-to-all | Slow gates, scale limit | High-fidelity small circuits | IonQ, Quantinuum |
| Neutral Atom | ~10⁴ ops/T₂ | Scale, reconfigurable | Mid-circuit measure difficult | Analog simulation, 1000+ q | QuEra, Pasqal, Atom Computing |
| Spin Qubit | ~10⁶ ops/T₂ (nuclear) | CMOS compatible, small | Fabrication uniformity | Long-term fault-tolerance | Intel, TU Delft, QuTech |
| Photonic | N/A (no T₂ decay) | Room temp, networking | No det. 2Q gate (linear) | QKD, quantum networks | Xanadu, PsiQ, Quandela |

## 7.2 The Transmon Qubit: Circuit QED Deep Dive

The transmon qubit is the most widely deployed superconducting qubit architecture, used in IBM Quantum, Google Sycamore, Rigetti, IQM, and most other superconducting quantum computers. Understanding the transmon at a physical level — from Josephson junction through circuit Hamiltonian, microwave control, and dispersive readout — is essential for any quantum computing practitioner. It explains concretely why qubits have the properties they have: why the qubit frequency is 5 GHz rather than 50 GHz or 50 MHz, why gates take 50 ns rather than 50 ps or 50 μs, why coherence times are in the microsecond range rather than nanoseconds or seconds.

The transmon was developed at Yale University in 2007 by Jens Koch, Terri Yu, Jay Gamble, David DeMille, Michel Devoret, and Robert Schoelkopf. The name is a portmanteau of "transmission line shunted plasma oscillation qubit" — though most practitioners simply call it the transmon. Its key innovation over the earlier Cooper pair box (CPB) was adding a large shunt capacitor that exponentially suppressed sensitivity to charge noise while maintaining the anharmonicity needed for qubit operation.

### 7.2.1 The Josephson Junction

The Josephson junction is the heart of any superconducting qubit. It is a nanoscale device consisting of two superconducting electrodes (typically aluminium, Al) separated by an ultra-thin (~1–3 nm) insulating barrier of aluminium oxide (Al₂O₃). At temperatures below the superconducting critical temperature (Tc ≈ 1.2 K for Al), electrons near the Fermi level bind together into Cooper pairs — bound pairs with zero net spin and charge 2e — that condense into a macroscopic quantum ground state described by a single complex order parameter Ψ = |Ψ|e^(iφ), where φ is the quantum phase.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Josephson Relations</strong></p>
<p>When two superconductors are separated by a thin tunnel barrier (the Josephson junction), Cooper pairs tunnel coherently through the barrier. The junction is characterised by two fundamental equations — the Josephson relations:</p>
<p>DC Josephson effect: A supercurrent I_c·sin(φ) flows across the junction at zero voltage — current without resistance. This is pure quantum tunnelling of Cooper pairs. DC Josephson effect: A voltage V across the junction causes the phase to evolve at the Josephson frequency: dφ/dt = 2eV/ħ = 2π × 483.6 MHz/μV. For V = 1 μV, the frequency is 483.6 MHz; for V = 10 μV, it is 4.836 GHz — squarely in the microwave qubit range.</p>
<p>The Josephson junction behaves as a nonlinear inductor with inductance L_J = ħ/(2e · I_c · cos(φ)). This nonlinearity — L_J depending on the current state — is what makes the energy levels anharmonic, enabling selective qubit addressing.</p>
</div>

The Josephson energy is the characteristic energy scale of the junction: E\_J = I\_c · ħ/(2e) = I\_c · Φ₀/(2π), where Φ₀ = h/(2e) = 2.068 × 10⁻¹⁵ Wb is the magnetic flux quantum. For a transmon junction with I\_c = 40 nA: E\_J/h ≈ 40×10⁻⁹ × 2.068×10⁻¹⁵ / (2π × 6.626×10⁻³⁴) ≈ 20 GHz. This sets the overall energy scale of the circuit.

### 7.2.2 Transmon Circuit and Hamiltonian

The transmon circuit consists of a Josephson junction (providing nonlinear inductance E\_J) in parallel with a large shunt capacitance C\_s (providing charging energy E\_C = e²/(2C\_Σ), where C\_Σ = C\_J + C\_s is the total capacitance including the junction capacitance C\_J). This elegant circuit was the solution to a critical problem with early Cooper pair box qubits: sensitivity to offset charge noise from fluctuating background charges in the substrate.

<figure class="book-figure">
<img src="content/images/image80.png" alt="Figure 7.2: Transmon qubit circuit and energy level structure. Left: The transmon circuit — a Josephson junction (cross symbol, E_J) shunted by capacitor C_s (contributing to E_C), capacitively coupled to a microwave readout resonator (ωᵣ) through coupling capacitor C_g. The full system is cooled to ~15 mK in a dilution refrigerator. Right: Anharmonic energy level structure. The |0⟩→|1⟩ spacing is ω₀₁/2π = 5.15 GHz; the |1⟩→|2⟩ spacing is ω₁₂/2π = 4.80 GHz; the anharmonicity α/2π = ω₁₂ − ω₀₁ = −350 MHz prevents accidental |2⟩ excitation when driving |0⟩→|1⟩.">
<figcaption>Figure 7.2: Transmon qubit circuit and energy level structure. Left: The transmon circuit — a Josephson junction (cross symbol, E_J) shunted by capacitor C_s (contributing to E_C), capacitively coupled to a microwave readout resonator (ωᵣ) through coupling capacitor C_g. The full system is cooled to ~15 mK in a dilution refrigerator. Right: Anharmonic energy level structure. The |0⟩→|1⟩ spacing is ω₀₁/2π = 5.15 GHz; the |1⟩→|2⟩ spacing is ω₁₂/2π = 4.80 GHz; the anharmonicity α/2π = ω₁₂ − ω₀₁ = −350 MHz prevents accidental |2⟩ excitation when driving |0⟩→|1⟩.</figcaption>
</figure>

#### The Transmon Hamiltonian

The quantum Hamiltonian of the Cooper pair box (which becomes the transmon in the large E\_J/E\_C limit) is derived by treating n (the number of Cooper pairs on the superconducting island) as the quantum charge operator, and φ (the junction phase) as the quantum phase operator, conjugate to n: [φ, n] = i. The Hamiltonian is:

<div class="box box-equation">
<p><strong>Equation 7.2</strong></p>
<p><strong>H_CPB  =  4E_C(n − n_g)²  −  E_J cos(φ)</strong></p>
<p>E_C = e²/(2C_Σ) = charging energy per Cooper pair (typically 200–350 MHz)</p>
<p>E_J = I_c·ħ/(2e) = Josephson energy (typically 10–40 GHz)</p>
<p>n_g = C_g·V_g/(2e) = offset charge (gate-induced charge, source of charge noise)</p>
<p><strong>Transmon regime: E_J/E_C &gt;&gt; 1 (typically 50–100)</strong></p>
</div>

#### Why Charge Noise Is Suppressed Exponentially

In the Cooper pair box (E\_J/E\_C ~ 1), the energy levels depend strongly on n\_g — charge noise from the substrate shifts the qubit frequency unpredictably. The key insight of the transmon is that in the limit E\_J/E\_C >> 1, the energy levels become essentially independent of n\_g. The charge dispersion (variation of E\_k with n\_g) decreases exponentially with E\_J/E\_C:

<div class="box box-equation">
<p><strong>Equation 7.3</strong></p>
<p>ε_k ∝ exp(−√(8E_J/E_C))   [charge dispersion of level k]</p>
<p>For E_J/E_C = 50: ε ~ exp(−√400) = exp(−20) ≈ 2×10⁻⁹ — negligibly small</p>
</div>

#### Anharmonicity and Why It Matters

The price of reduced charge sensitivity is reduced anharmonicity. In the transmon limit, expanding the cosine potential around its minimum, the energy levels are approximately those of an anharmonic oscillator:

<div class="box box-equation">
<p><strong>Equation 7.4</strong></p>
<p>E_k ≈ −E_J + √(8E_J·E_C)·(k + 1/2) + α·(k+1/2)² + ...</p>
<p>ω₀₁/2π ≈ √(8E_J·E_C)/h − E_C/h  [qubit frequency]</p>
<p><strong>α/2π  = ω₁₂/2π − ω₀₁/2π ≈ −E_C/h  [anharmonicity]</strong></p>
</div>

Typical values: E\_C/h ≈ 250–350 MHz, giving α/2π ≈ −250 to −350 MHz. This anharmonicity is critical: it ensures a microwave pulse resonant with the |0⟩→|1⟩ transition (at ω₀₁) does not also drive the |1⟩→|2⟩ transition (at ω₁₂ = ω₀₁ + α), which would leak population out of the qubit subspace. The anharmonicity must be large compared to the pulse bandwidth — see Solved Problem 7.7.

### 7.2.3 Microwave Control and Gate Operations

Qubit gates are implemented by applying shaped microwave pulses at the qubit frequency. The physics is straightforward: the qubit is a two-level system with transition frequency ω₀₁/(2π) in the 4–8 GHz range. A resonant microwave drive creates a time-dependent electric field that couples to the qubit through the gate capacitor C\_g, driving Rabi oscillations between |0⟩ and |1⟩.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Microwave Drive Hamiltonian (Rotating Frame)</strong></p>
<p>In the rotating frame at the drive frequency ω_d, a microwave pulse with amplitude Ω(t), phase φ_d drives the qubit Hamiltonian:</p>
<p>Gate implementations: A Gaussian pulse with area ∫Ω(t)dt = π implements an X gate (π-pulse, rotates |0⟩ to |1⟩). Area π/2 implements a √X (Ry(π/2)) gate. The drive phase φ_d sets the rotation axis: φ_d = 0 gives rotation about X; φ_d = π/2 gives rotation about Y. All single-qubit gates (U3 on IBM) are composed from at most two pulses.</p>
<p>DRAG pulses (Derivative Removal via Adiabatic Gate): Standard Gaussian pulses have spectral tails that extend to the |1⟩→|2⟩ transition frequency, causing leakage errors. DRAG pulses add a derivative component Ω_Y(t) = -λ·dΩ_X/dt (with λ ≈ 1/(2α)) that cancels the |1⟩→|2⟩ drive while preserving the |0⟩→|1⟩ transition. DRAG reduces leakage errors from ~10⁻³ to ~10⁻⁵ per gate and is the standard pulse shape in IBM hardware.</p>
</div>

#### Two-Qubit Gates: Cross-Resonance (IBM)

IBM's primary two-qubit gate is the cross-resonance (CR) gate, which implements the CNOT (CX) gate without requiring direct tunable coupling. The CR gate works as follows: the control qubit is driven at the target qubit's resonance frequency ω\_target. Because the control and target are coupled (through a fixed capacitor or resonator bus), this drive has a qubit-state-dependent effect on the target qubit — it drives the target at a rate that depends on whether the control is in |0⟩ or |1⟩. This creates a ZX interaction that, combined with single-qubit rotations, implements CNOT. CR gate time: 200–400 ns on IBM hardware; two-qubit error rate ~0.1–0.5%.

#### Two-Qubit Gates: Parametric Coupling (Google, Rigetti)

Google's Sycamore and Rigetti's Aspen processors use flux-tunable coupling: the coupling between two qubits is controlled by a tunable SQUID (Superconducting Quantum Interference Device) that can switch the coupling on and off by threading magnetic flux through the loop. When the coupling is turned on at the right frequency, the qubits exchange energy at a tunable rate, implementing a parametric iSWAP gate in ~12–30 ns. The advantage over CR is faster gate time; the disadvantage is the need for an additional flux-tunable element per coupling with its own noise sources.

### 7.2.4 Readout via Circuit QED

Measuring the state of a superconducting qubit uses the principle of circuit quantum electrodynamics (cQED). Each transmon qubit is capacitively coupled to a microwave readout resonator — a coplanar waveguide resonator with resonance frequency ω\_r/(2π) ≈ 6–8 GHz, different from the qubit frequency ω\_q. The qubit-resonator coupling strength g/(2π) ≈ 50–300 MHz is large compared to the decay rates but smaller than the qubit-resonator detuning Δ = ω\_q − ω\_r, putting the system in the dispersive coupling regime.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Dispersive Readout: cQED Architecture</strong></p>
<p>In the dispersive limit (g/Δ &lt;&lt; 1), the qubit and resonator are far off-resonance and cannot directly exchange energy. Instead, the coupling produces a state-dependent frequency shift of the resonator — the dispersive shift χ:</p>
<p>Readout procedure: Send a microwave probe tone at ω_r. Measure the phase or amplitude of the reflected (or transmitted) signal using a homodyne or heterodyne detector. The reflected phase shifts by ±arctan(2χ/κ) depending on the qubit state, where κ is the resonator linewidth. By demodulating the output with an analogue-to-digital converter (ADC) at room temperature and comparing to a threshold, the qubit state |0⟩ or |1⟩ is determined.</p>
<p>This is a quantum non-demolition (QND) measurement in the dispersive limit: reading out the qubit does not directly disturb its state (though photon shot noise in the resonator can induce measurement-induced dephasing). Readout fidelity on IBM Heron: 97–99.5%.</p>
</div>

## 7.3 Coherence Times: T1, T2, and T2*

Coherence times are the most practically important parameters of a quantum processor. They impose hard limits on the depth of quantum circuits that can be executed faithfully. A qubit that can maintain its quantum state for 100 μs while gates take 300 ns can, in principle, execute ~333 two-qubit gates before coherence is significantly degraded — setting the practical circuit depth ceiling. Understanding the physical mechanisms that limit coherence is essential for both hardware engineers (who try to extend T1 and T2) and algorithm designers (who must work within these limits).

### 7.3.1 Amplitude Damping and T1 Relaxation

T1, the longitudinal relaxation time or energy relaxation time, quantifies how long the excited state |1⟩ population survives before spontaneously decaying to the ground state |0⟩. This process is irreversible: the qubit loses energy to the environment (phonons, photons, quasiparticles) and returns to thermal equilibrium.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: T1 Relaxation: Definition and Dynamics</strong></p>
<p>If the qubit is prepared in |1⟩ at time t = 0, the probability of finding it still in |1⟩ at time t is:</p>
<p>Physical mechanisms causing T1 decay in superconducting transmon qubits:</p>
<p>Dielectric loss: Two-level system (TLS) defects in the substrate (Si, SiO₂) and in the junction oxide (Al₂O₃) absorb energy from the qubit. This is the dominant T1 limitation in most devices. Mitigated by clean fabrication, surface treatment (HF etching), and using low-loss substrates (crystalline sapphire).</p>
<p>Quasiparticle poisoning: Thermally excited or radiation-generated unpaired electrons (quasiparticles) tunnel through the Josephson junction, disrupting the Cooper pair condensate and absorbing qubit energy. Mitigated by shielding from ionising radiation, quasiparticle traps (normal metal patches near the junction), and very low cryostat temperatures.</p>
<p>Purcell effect: The qubit antenna (coupling capacitor, transmission lines) radiates microwave energy into the environment. The Purcell decay rate Γ_Purcell = (g/Δ)²·κ where κ is the resonator linewidth. Mitigated by Purcell filters and optimised resonator design.</p>
<p>Current best T1 (IBM Heron R2, 2024): median ~200 μs; best individual qubits ~500 μs.</p>
</div>

#### T1 Measurement: Inversion Recovery

The standard T1 measurement is the inversion recovery sequence: (1) Apply an X gate to prepare |1⟩; (2) Wait for time τ (free evolution); (3) Measure in the computational basis. Repeat for many values of τ from 0 to ~5T1. The measured P(|1⟩) as a function of τ follows exp(−τ/T1). Fitting this exponential gives T1 directly. On IBM Quantum hardware, this can be performed using the Qiskit Experiments library (T1Experiment class) which automates the sweep and fit.

<figure class="book-figure">
<img src="content/images/image81.png" alt="Figure 7.3: Coherence time measurements. Left: T1 inversion recovery — exponential decay of P(|1⟩) for four T1 values (50, 100, 200, 500 μs) representing device generations from 2019 to 2024. The dashed line marks 1/e ≈ 0.368. Right: T2 Ramsey fringe — oscillating signal with decaying envelope for three T2 values. The oscillation frequency equals the drive detuning Δf from the qubit; the envelope decay constant is T2. The Ramsey protocol (π/2 → delay → π/2 → measure) is illustrated.">
<figcaption>Figure 7.3: Coherence time measurements. Left: T1 inversion recovery — exponential decay of P(|1⟩) for four T1 values (50, 100, 200, 500 μs) representing device generations from 2019 to 2024. The dashed line marks 1/e ≈ 0.368. Right: T2 Ramsey fringe — oscillating signal with decaying envelope for three T2 values. The oscillation frequency equals the drive detuning Δf from the qubit; the envelope decay constant is T2. The Ramsey protocol (π/2 → delay → π/2 → measure) is illustrated.</figcaption>
</figure>

### 7.3.2 Phase Damping and T2 Dephasing

T2, the transverse relaxation time or dephasing time, quantifies how long the off-diagonal elements (coherences) of the qubit density matrix survive. While T1 measures energy loss, T2 measures phase randomisation: the qubit "forgets" the relative phase between its |0⟩ and |1⟩ components. Dephasing can occur without any energy exchange — through random fluctuations of the qubit frequency ω₀₁ caused by noise in the environment.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: T2 Dephasing: Definition and Fundamental Bound</strong></p>
<p>Starting from a superposition state at t=0, the off-diagonal density matrix element evolves as:</p>
<p>Physical origin of the 2T1 bound: Even if there were no pure dephasing at all (no external phase noise), T1 decay still randomises the qubit phase by collapsing the superposition. The maximum possible T2 is twice T1 — achieved only in the ideal case of no pure dephasing.</p>
<p>Physical mechanisms causing T2 &lt; 2T1 in transmon qubits:</p>
<p>Magnetic flux noise: 1/f-spectrum noise from TLS defects near the qubit loop area fluctuates the effective flux Φ through tunable qubits, shifting ω₀₁. Dominant in flux-tunable designs; mitigated by operating at the flux "sweet spot" where dω₀₁/dΦ = 0.</p>
<p>Charge noise: Random background charges from impurities in the substrate fluctuate n_g, shifting ω₀₁. Exponentially suppressed in transmons (vs. CPBs) but still present at the ~10⁻⁶ e/√Hz level.</p>
<p>Photon shot noise: Thermal photons in the readout resonator (from residual blackbody radiation at ~20–50 mK) cause measurement-induced dephasing at a rate Γ_meas = 2χ²n̄/κ.</p>
<p>Control electronics noise: Phase noise in the microwave oscillators drifts the effective frame of reference.</p>
</div>

#### The T2 Decomposition: Pure Dephasing Time T\_φ

The total T2 dephasing rate is the sum of the T1 contribution and a pure dephasing contribution:

<div class="box box-equation">
<p><strong>Equation 7.9</strong></p>
<p><strong>1/T2  =  1/(2T1)  +  1/T_φ</strong></p>
<p>T_φ = pure dephasing time (phase kicks without energy change)</p>
<p>If T_φ &gt;&gt; 2T1: then T2 ≈ 2T1 (T1-limited)</p>
<p>If T_φ &lt;&lt; 2T1: then T2 ≈ T_φ (pure-dephasing limited)</p>
</div>

#### T2 Measurement: Ramsey Sequence

T2 is measured using the Ramsey free induction decay sequence: (1) Apply Ry(π/2) to prepare |+⟩ = (|0⟩+|1⟩)/√2; (2) Wait for time τ (free evolution); (3) Apply Ry(π/2) again; (4) Measure in Z basis. The result oscillates at the detuning frequency Δω = ω\_drive − ω₀₁ and decays with envelope exp(−τ/T2). Fitting the decaying oscillation gives both T2 and the qubit frequency offset. A deliberate detuning Δω ≈ 100 kHz is often introduced to make the fringes visible even at short times.

<figure class="book-figure">
<img src="content/images/image82.png" alt="Figure 7.4: Qubit state evolution on the Bloch sphere under decoherence. Left: T1 relaxation — the Bloch vector for a qubit starting in |1⟩ (south pole) decays toward the ground state |0⟩ (north pole), tracing a path along the meridian; successive vectors at t = 0, 0.3T1, 0.7T1, T1 show shrinking z-component. Centre: T2 dephasing alone — the Bloch vector starting in |+⟩ at the equator loses its transverse component while z remains approximately zero. Right: Combined T1+T2 decay — the vector spirals inward toward the centre (maximally mixed state at origin), losing both energy and phase information.">
<figcaption>Figure 7.4: Qubit state evolution on the Bloch sphere under decoherence. Left: T1 relaxation — the Bloch vector for a qubit starting in |1⟩ (south pole) decays toward the ground state |0⟩ (north pole), tracing a path along the meridian; successive vectors at t = 0, 0.3T1, 0.7T1, T1 show shrinking z-component. Centre: T2 dephasing alone — the Bloch vector starting in |+⟩ at the equator loses its transverse component while z remains approximately zero. Right: Combined T1+T2 decay — the vector spirals inward toward the centre (maximally mixed state at origin), losing both energy and phase information.</figcaption>
</figure>

### 7.3.3 T2\* and Inhomogeneous Broadening

T2\* ("T2 star") is an additional, shorter coherence time that arises from slow, quasi-static frequency fluctuations that do not average out over a single Ramsey experiment. These slow drifts (varying on timescales longer than the gate sequence but shorter than the experiment repetition rate) cause inhomogeneous broadening — different repetitions of the same circuit experience different qubit frequencies, producing a dephased average:

<div class="box box-equation">
<p><strong>Equation 7.10</strong></p>
<p><strong>1/T2*  =  1/T2  +  1/T_inhom</strong></p>
<p>T_inhom accounts for quasi-static frequency noise (longer-timescale drift)</p>
<p>T2* ≤ T2 ≤ 2T1  [hierarchy of coherence times]</p>
</div>

T2\* is measured by a simple Ramsey sequence with many repetitions. T2 (the intrinsic dephasing time) is longer and can be measured using a Hahn echo sequence: X gate → wait τ/2 → X (π-pulse) → wait τ/2 → Ry(π/2) → measure. The π-pulse "refocuses" slow quasi-static noise, effectively cancelling inhomogeneous broadening and revealing T2 > T2\*. For modern IBM transmon qubits, T2\* ≈ T2 ≈ 50–300 μs because low-frequency noise is well-controlled by careful chip design. For silicon spin qubits in natural silicon (with nuclear spin bath), T2\* ~ 1 μs while T2 (with dynamical decoupling) ~ 1 ms — a factor of 1,000 difference.

### 7.3.4 Coherence Time and Circuit Depth

<div class="box box-example">
<p class="box-title"><strong>Example 7.1: Maximum Circuit Depth from Coherence Time</strong></p>
<p>Problem: A superconducting qubit has T2 = 150 μs and two-qubit CX gate time t_2Q = 300 ns. (a) What is the maximum number of sequential two-qubit gates executable before coherence-limited fidelity drops below 50%? (b) If the qubit also has per-gate depolarising error p = 0.003, what limits the circuit depth more severely — coherence or gate errors?</p>
<p><strong>Solution:</strong></p>
<p>(a) Coherence-limited depth: The density matrix element decays as exp(−t/T2). Fidelity drops below 50% when exp(−t/T2) = 0.5, i.e., t = T2 × ln(2) ≈ 150 μs × 0.693 = 103.9 μs.</p>
<p>Maximum gate count (coherence): n_coh = 103.9 μs / 0.3 μs ≈ 346 two-qubit gates.</p>
<p>(b) Depolarising-limited depth: F_depo = (1-p)^n. For F &gt; 0.5: n &lt; log(0.5)/log(0.997) = −0.693/(−0.003005) ≈ 231 gates.</p>
<p>Gate errors (231 gates) limit depth more than coherence (346 gates). This is typical: on modern hardware, per-gate calibration errors usually dominate over coherence decay for circuits up to ~200 two-qubit gates. Both effects worsen simultaneously in longer circuits.</p>
</div>

## 7.4 Quantum Channels and Kraus Operators

The unitary evolution model U|ψ⟩ is insufficient for describing real quantum hardware. In practice, the quantum state of a qubit is not a pure state but a mixed state (density matrix), and its evolution under noise and measurement involves irreversible processes. Quantum channels provide the most general, rigorous mathematical description of any physical process acting on a quantum system — encompassing unitary evolution, decoherence, measurement, and thermalization as special cases.

### 7.4.1 Quantum Channel Formalism

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Quantum Channel (CPTP Map)</strong></p>
<p>A quantum channel is a completely positive, trace-preserving (CPTP) linear map ε: L(H) → L(H) acting on density matrices. The conditions are:</p>
<p>Trace-preserving: Tr[ε(ρ)] = 1 for all valid density matrices ρ (probability conservation)</p>
<p>Completely positive: (I_n ⊗ ε)(ρ_AB) ≥ 0 for any n and any valid joint state ρ_AB (even if the channel acts on only part of a larger system, it cannot create negative probabilities)</p>
<p>Every CPTP map has a Kraus operator decomposition (operator-sum representation):</p>
<p>Physical derivation: If the environment starts in |0_E⟩ and the joint system+environment evolves by unitary U_SE, then tracing out the environment gives Kraus operators K_k = ⟨k_E|U_SE|0_E⟩ for any orthonormal basis {|k_E⟩} of the environment. The choice of basis gives different Kraus representations of the same channel (the Kraus representation is not unique).</p>
<p>Special cases: Unitary evolution has a single Kraus operator K_0 = U (one term, K_0†K_0 = U†U = I ✓). Measurement has Kraus operators K_k = |k⟩⟨k| (projectors). Complete dephasing has K_0=|0⟩⟨0|, K_1=|1⟩⟨1|.</p>
</div>

<figure class="book-figure">
<img src="content/images/image83.png" alt="Figure 7.5: Quantum noise channels and their action on the Bloch sphere. From left: (1) Bit-flip channel — sphere is compressed along x-axis by factor (1-2p); pure x-eigenstates |±⟩ are unaffected while |0⟩ and |1⟩ are most vulnerable. (2) Phase-flip channel — sphere is compressed in the equatorial plane (x,y shrink by 1-2p) while the z-axis is preserved; |0⟩ and |1⟩ are unaffected, equatorial superpositions are most vulnerable. (3) Depolarising channel — sphere contracts uniformly by factor (1−4p/3); at p = 3/4 the state becomes maximally mixed (sphere collapses to origin). (4) Amplitude damping — asymmetric contraction toward the north pole (|0⟩); models T1 relaxation.">
<figcaption>Figure 7.5: Quantum noise channels and their action on the Bloch sphere. From left: (1) Bit-flip channel — sphere is compressed along x-axis by factor (1-2p); pure x-eigenstates |±⟩ are unaffected while |0⟩ and |1⟩ are most vulnerable. (2) Phase-flip channel — sphere is compressed in the equatorial plane (x,y shrink by 1-2p) while the z-axis is preserved; |0⟩ and |1⟩ are unaffected, equatorial superpositions are most vulnerable. (3) Depolarising channel — sphere contracts uniformly by factor (1−4p/3); at p = 3/4 the state becomes maximally mixed (sphere collapses to origin). (4) Amplitude damping — asymmetric contraction toward the north pole (|0⟩); models T1 relaxation.</figcaption>
</figure>

### 7.4.2 Bit-Flip, Phase-Flip, and Bit-Phase-Flip Channels

#### Bit-Flip Channel (X-type Error)

The bit-flip channel models the classical analogue of a noise process: with probability p, a Pauli X (NOT gate) is applied, flipping |0⟩↔|1⟩. With probability (1−p), the state is unchanged.

<div class="box box-equation">
<p><strong>Equation 7.12</strong></p>
<p><strong>ε_BF(ρ) = (1-p)·ρ + p·X·ρ·X†</strong></p>
<p>Kraus operators: K₀ = √(1-p)·I,  K₁ = √p·X</p>
<p>Verify: K₀†K₀ + K₁†K₁ = (1-p)I + pX†X = (1-p)I + pI = I ✓</p>
<p>Bloch sphere action: r_x → r_x (unchanged);  r_y → (1-2p)·r_y;  r_z → (1-2p)·r_z</p>
</div>

The bit-flip channel is important for understanding the principle of quantum error correction: it models the simplest type of hardware error. The 3-qubit bit-flip code (|0⟩\_L = |000⟩, |1⟩\_L = |111⟩) can detect and correct a single bit-flip error by majority vote — the foundation of all quantum error correction.

#### Phase-Flip Channel (Z-type Error)

The phase-flip channel applies Pauli Z with probability p, inverting the relative phase between |0⟩ and |1⟩ without changing populations. It is the quantum analogue of the dephasing process — it destroys coherences while leaving diagonal elements of ρ unchanged.

<div class="box box-equation">
<p><strong>Equation 7.13</strong></p>
<p><strong>ε_PF(ρ) = (1-p)·ρ + p·Z·ρ·Z†</strong></p>
<p>For ρ = [[a, b], [c, d]]:   ε_PF(ρ) = [[a, (1-2p)b], [(1-2p)c, d]]</p>
<p><strong>Off-diagonal elements decay by factor (1-2p) → models pure dephasing!</strong></p>
<p>Bloch sphere action: r_x → (1-2p)·r_x;  r_y → (1-2p)·r_y;  r_z → r_z (unchanged)</p>
</div>

#### Bit-Phase-Flip Channel (Y-type Error)

The bit-phase-flip channel applies Pauli Y = iXZ with probability p, combining both bit flip and phase flip. It can be thought of as simultaneously flipping the state and flipping the phase.

<div class="box box-equation">
<p><strong>Equation 7.14</strong></p>
<p>ε_BPF(ρ) = (1-p)·ρ + p·Y·ρ·Y†</p>
<p>Kraus: K₀ = √(1-p)·I,  K₁ = √p·Y</p>
</div>

<div class="box box-example">
<p class="box-title"><strong>Example 7.2: Bit-Flip Channel Acting on |+⟩ and |0⟩</strong></p>
<p>Problem: Apply the bit-flip channel with p = 0.3 to (a) |+⟩ = (|0⟩+|1⟩)/√2, and (b) |0⟩. Compute output density matrices and fidelities with the input.</p>
<p><strong>Solution (a) — Input |+⟩:</strong></p>
<p>ρ = |+⟩⟨+| = (1/2)[[1,1],[1,1]]</p>
<p>X·ρ·X = X·(1/2)[[1,1],[1,1]]·X = (1/2)[[0,1],[1,0]][[1,1],[1,1]][[0,1],[1,0]]</p>
<p>= (1/2)[[1,1],[1,1]] = ρ   [since |+⟩ is X eigenstate: X|+⟩ = |+⟩]</p>
<p>ε_BF(|+⟩⟨+|) = (1-p)ρ + p·ρ = ρ = |+⟩⟨+|</p>
<p>Fidelity F = ⟨+|ε(ρ)|+⟩ = 1.0 — the |+⟩ state is completely immune to bit flips!</p>
<p><strong>Solution (b) — Input |0⟩:</strong></p>
<p>ρ = |0⟩⟨0| = [[1,0],[0,0]]</p>
<p>X·ρ·X = [[0,1],[1,0]][[1,0],[0,0]][[0,1],[1,0]] = [[0,0],[0,1]] = |1⟩⟨1|</p>
<p>ε_BF(|0⟩⟨0|) = (1-0.3)[[1,0],[0,0]] + 0.3[[0,0],[0,1]] = [[0.7,0],[0,0.3]]</p>
<p>This is a mixed state: 70% probability of |0⟩, 30% probability of |1⟩.</p>
<p>Fidelity F = ⟨0|ε(ρ)|0⟩ = 0.7  [probability of measuring |0⟩ in output]</p>
<p>Lesson: Basis choice matters. Bit-flip errors destroy |0⟩/|1⟩ states but preserve |±⟩ states. This is why quantum error correction must protect against all error types.</p>
</div>

### 7.4.3 Depolarising and Amplitude Damping Channels

#### Depolarising Channel

The depolarising channel is the most symmetric and commonly used noise model in quantum computing. It replaces the qubit state with the maximally mixed state I/2 with probability p, and leaves it unchanged with probability (1−p). Equivalently, it applies X, Y, or Z with equal probability p/3 each, and I with probability (1−p). This is the noise model used in Qiskit's randomised benchmarking and depolarising error in Aer.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Depolarising Channel</strong></p>
<p>Bloch sphere action: The Bloch vector contracts uniformly:  r → (1 − 4p/3)·r.</p>
<p>At p = 3/4: contraction factor = 1 − 4(3/4)/3 = 1 − 1 = 0 → state becomes I/2 (maximally mixed, Bloch vector = 0). For p &gt; 3/4 the channel is still valid but the Bloch vector direction can flip (reversal).</p>
<p>The depolarising channel is often used to model gate errors in randomised benchmarking: each Clifford gate is followed by depolarisation with parameter p. The RB decay curve p^k (k Clifford gates) gives the average Clifford gate error rate 1-p directly.</p>
</div>

#### Amplitude Damping Channel

The amplitude damping channel is the most physically accurate model for T1 energy relaxation. It describes the irreversible process in which the qubit spontaneously emits energy to the environment, transitioning from the excited state |1⟩ to the ground state |0⟩. Unlike the other channels, amplitude damping is not symmetric: only |1⟩ decays to |0⟩, not vice versa (at zero temperature).

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Amplitude Damping Channel</strong></p>
<p>Action on density matrix ρ = [[ρ₀₀, ρ₀₁],[ρ₁₀, ρ₁₁]]:</p>
<p>Interpretation: (1) The |1⟩ population ρ₁₁ decays by factor (1−γ) — the excited state empties. (2) The |0⟩ population grows by γ·ρ₁₁ — population flows from |1⟩ to |0⟩. (3) Coherences (off-diagonal) decay by factor √(1−γ) = exp(−t/2T1) — this matches the T1 contribution to T2: 1/(2T1) in the T2 decomposition.</p>
<p>Verify completeness: K₀†K₀ + K₁†K₁ = [[1,0],[0,1−γ]] + [[0,0],[0,γ]] = [[1,0],[0,1]] = I ✓</p>
</div>

## 7.5 Qiskit Aer Noise Models

Qiskit Aer is the high-performance quantum circuit simulator included in the Qiskit ecosystem. Beyond ideal (noiseless) simulation, Aer provides a comprehensive noise simulation framework that can model the full range of physical noise processes — thermal relaxation, gate errors, readout errors, crosstalk, and custom user-defined noise channels. Noisy simulation is essential for NISQ algorithm development: it allows researchers to assess circuit performance on realistic hardware before consuming limited quantum cloud time.

The central object is the NoiseModel class. A NoiseModel specifies which quantum error channel is applied after each gate on each qubit. When a circuit is simulated with a NoiseModel, the simulator samples from the Kraus operators of each error channel after each gate, propagating realistic mixed states through the circuit.

<figure class="book-figure">
<img src="content/images/image84.png" alt="Figure 7.6: Qiskit Aer noise model architecture and circuit fidelity. Left: NoiseModel architecture — thermal_relaxation_error, depolarizing_error, ReadoutError, and pauli_error objects are created and attached to specific gates and qubits using .add_all_qubit_quantum_error() or .add_quantum_error(). The AerSimulator then applies these error channels after each gate during simulation. Right: Circuit fidelity F = (1−p)^n vs. circuit depth (number of two-qubit gates) for five depolarising error rates. The 90% fidelity line (dashed) shows the maximum circuit depth for practically useful output.">
<figcaption>Figure 7.6: Qiskit Aer noise model architecture and circuit fidelity. Left: NoiseModel architecture — thermal_relaxation_error, depolarizing_error, ReadoutError, and pauli_error objects are created and attached to specific gates and qubits using .add_all_qubit_quantum_error() or .add_quantum_error(). The AerSimulator then applies these error channels after each gate during simulation. Right: Circuit fidelity F = (1−p)^n vs. circuit depth (number of two-qubit gates) for five depolarising error rates. The 90% fidelity line (dashed) shows the maximum circuit depth for practically useful output.</figcaption>
</figure>

### 7.5.1 thermal\_relaxation\_error

The thermal\_relaxation\_error function is the most physically accurate noise model for superconducting qubits. It simultaneously models both T1 amplitude damping and T2 dephasing for a gate of specified duration, constructing the corresponding Kraus operators analytically.

```python
# ────────────────────────────────────────────────────────────────────────
# Code 7.1 — Thermal Relaxation Noise Model in Qiskit Aer
# Models T1 (amplitude damping) + T2 (dephasing) for realistic superconducting
# qubit simulation. Uses typical IBM Falcon qubit parameters.
# ────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit_aer.noise import (NoiseModel, thermal_relaxation_error,
                               depolarizing_error, ReadoutError)
import numpy as np

# ── Step 1: Define physical hardware parameters ─────────────────────────
T1      = 100e-6    # T1 relaxation time = 100 μs
T2      = 80e-6     # T2 dephasing time  = 80 μs  (must satisfy T2 ≤ 2T1)
t_1q    = 50e-9     # Single-qubit gate duration = 50 ns
t_2q    = 300e-9    # Two-qubit CX gate duration  = 300 ns
t_meas  = 600e-9    # Measurement duration = 600 ns

# ── Step 2: Create thermal relaxation errors for each gate type ──────────
# thermal_relaxation_error(T1, T2, time) returns a Kraus channel
# modelling T1 + T2 decay during a gate of specified duration.
error_1q   = thermal_relaxation_error(T1, T2, t_1q)    # single-qubit gate
error_2q   = thermal_relaxation_error(T1, T2, t_2q)    # two-qubit gate (per qubit)
error_meas = thermal_relaxation_error(T1, T2, t_meas)  # measurement window

# ── Step 3: Add optional gate calibration errors (depolarising) ──────────
# Real gates have both coherence decay AND imperfect calibration.
# Compose the two error channels: error = thermal THEN depolarising.
depo_1q     = depolarizing_error(3e-4, 1)   # 0.03% single-qubit error rate
depo_2q     = depolarizing_error(3e-3, 2)   # 0.30% two-qubit error rate
combined_1q = error_1q.compose(depo_1q)     # apply T1/T2 then depolarise
combined_2q = error_2q.compose(depo_2q)

# ── Step 4: Build the NoiseModel ─────────────────────────────────────────
noise_model = NoiseModel()

# Add single-qubit error to all standard gates on all qubits
noise_model.add_all_qubit_quantum_error(combined_1q, ['h', 'x', 'y', 'z', 's', 't',
                                                       'rx', 'ry', 'rz', 'u1', 'u2', 'u3'])

# Add two-qubit error to CX/CZ gates on all qubit pairs
noise_model.add_all_qubit_quantum_error(combined_2q, ['cx', 'cz'])

# Add state-preparation-and-measurement (SPAM) error
p_meas = 0.015   # 1.5% readout error rate
# ReadoutError([[P(report 0 | prepared 0), P(report 1 | prepared 0)],
#               [P(report 0 | prepared 1), P(report 1 | prepared 1)]])
readout_err = ReadoutError([[1-p_meas, p_meas], [p_meas, 1-p_meas]])
noise_model.add_all_qubit_readout_error(readout_err)

# ── Step 5: Simulate Bell state circuit with and without noise ───────────
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

sim_noisy = AerSimulator(noise_model=noise_model)
sim_ideal = AerSimulator()
shots = 10000

counts_noisy = sim_noisy.run(transpile(qc, sim_noisy), shots=shots).result().get_counts()
counts_ideal = sim_ideal.run(transpile(qc, sim_ideal), shots=shots).result().get_counts()

print('Ideal Bell state:  ', counts_ideal)   # {'00': ~5000, '11': ~5000}
print('Noisy Bell state:  ', counts_noisy)   # {'00': ~4800, '11': ~4700, '01': ~250, '10': ~250}

# Compute Bell state fidelity estimate
n_correct = counts_noisy.get("00",0) + counts_noisy.get("11",0)
print(f'Bell fidelity estimate: {n_correct/shots:.3f}')  # typically ~0.95
```

### 7.5.2 depolarizing\_error and Custom NoiseModel

Beyond thermal relaxation, Qiskit Aer supports a rich set of noise channels. The depolarizing\_error is the simplest and most commonly used model for gate benchmarking. For advanced use, NoiseModel.from\_backend() constructs a complete noise model from a real IBM hardware backend's calibration data — the most accurate approach for predicting real hardware performance.

```python
# ────────────────────────────────────────────────────────────────────────
# Code 7.2 — Custom Per-Qubit Noise Model and Backend-Based Model
# Demonstrates both manual calibration-based noise and automatic backend model.
# ────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit_aer.noise import (NoiseModel, depolarizing_error,
                               thermal_relaxation_error)

# ── Method A: Automatic noise model from fake IBM backend ───────────────
from qiskit_ibm_runtime.fake_provider import FakeSherbrookeV2

backend = FakeSherbrookeV2()   # 127-qubit Eagle R3 calibration data
noise_auto = NoiseModel.from_backend(backend)
print('Basis gates with errors:', noise_auto.basis_gates)
print('Noisy qubit list:', noise_auto.noise_qubits[:5], '...')

# ── Method B: Manual per-qubit noise model ───────────────────────────────
# Typical IBM Falcon qubit parameters — vary across the chip
qubit_data = {
    0: {"T1": 150e-6, "T2":  80e-6, "t1q":  50e-9, "p1q": 3e-4, "p2q": 4e-3},
    1: {"T1": 200e-6, "T2": 100e-6, "t1q":  45e-9, "p1q": 2e-4, "p2q": 3.5e-3},
    2: {"T1":  90e-6, "T2":  60e-6, "t1q":  55e-9, "p1q": 5e-4, "p2q": 6.0e-3},
}

nm_manual = NoiseModel()
for q, d in qubit_data.items():
    # Single-qubit: thermal relaxation + depolarising, composed
    e_th_1q  = thermal_relaxation_error(d["T1"], d["T2"], d["t1q"])
    e_dp_1q  = depolarizing_error(d["p1q"], 1)
    e_1q     = e_th_1q.compose(e_dp_1q)
    nm_manual.add_quantum_error(e_1q, ['h','x','y','z','s','t'], [q])

    # Two-qubit: thermal for worst qubit in pair + depolarising
    if q + 1 in qubit_data:
        q2 = q + 1
        T1m = min(d["T1"], qubit_data[q2]["T1"])
        T2m = min(d["T2"], qubit_data[q2]["T2"])
        e_th_2q  = thermal_relaxation_error(T1m, T2m, 300e-9)
        e_dp_2q  = depolarizing_error(d["p2q"], 2)
        # Expand single-qubit thermal to 2-qubit: tensor product
        e_2q     = e_th_2q.expand(e_th_2q).compose(e_dp_2q)
        nm_manual.add_quantum_error(e_2q, ['cx'], [q, q2])
        nm_manual.add_quantum_error(e_2q, ['cx'], [q2, q])

# ── Test: 3-qubit GHZ circuit with manual noise model ────────────────────
qc_ghz = QuantumCircuit(3, 3)
qc_ghz.h(0)
qc_ghz.cx(0, 1)
qc_ghz.cx(1, 2)
qc_ghz.measure([0, 1, 2], [0, 1, 2])

sim_m = AerSimulator(noise_model=nm_manual)
counts_ghz = sim_m.run(transpile(qc_ghz, sim_m), shots=8192).result().get_counts()
print('GHZ noisy counts:', counts_ghz)
# Expected: ~000 and ~111 dominate, ~5-10% error outcomes 001, 010, etc.

# ── Compute GHZ fidelity estimate ────────────────────────────────────────
n_ghz = counts_ghz.get("000",0) + counts_ghz.get("111",0)
print(f'GHZ fidelity: {n_ghz/8192:.3f}')   # typically ~0.88-0.93
```

## 7.6 Hardware Benchmarks: Quantum Volume and CLOPS

Comparing quantum computers is non-trivial. A processor with more qubits is not necessarily more capable if those qubits have poor fidelity. A processor with faster gates may underperform a slower one if the faster gates have higher error rates. Individual metrics — qubit count, gate fidelity, T1, T2, connectivity — each tell part of the story but none tell the whole story. Benchmark metrics provide standardised, single-number comparisons that capture the interplay of multiple hardware properties.

### 7.6.1 Quantum Volume (QV)

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Quantum Volume: Definition</strong></p>
<p>Quantum Volume (QV) is a single-number hardware benchmark introduced by IBM in 2019. It is defined as the maximum size of a "square" random quantum circuit that can be executed with heavy-output probability greater than 2/3 at the 97.5% confidence level:</p>
<p>Protocol step by step:</p>
<p>Select n qubits from the processor (choosing the best n qubits if more are available).</p>
<p>Generate 100 random "model circuits": n qubits × n layers, where each layer consists of a random permutation of qubits into pairs, with a random 2-qubit SU(4) gate applied to each pair. These are then decomposed into the native gate set.</p>
<p>Classically simulate each circuit to find the ideal output distribution P_ideal(x) for each bitstring x.</p>
<p>Identify "heavy outputs": bitstrings x with P_ideal(x) &gt; median(P_ideal). Exactly half of all 2^n bitstrings are heavy.</p>
<p>Execute each circuit on hardware with ≥1000 shots. Compute heavy-output generation probability (HOGP): fraction of measured shots that are heavy outputs.</p>
<p>If HOGP &gt; 2/3 for all 100 circuits with 97.5% statistical confidence, the processor achieves QV = 2^n. Try n+1.</p>
</div>

<figure class="book-figure">
<img src="content/images/image85.png" alt="Figure 7.7: Quantum Volume progress and CLOPS metric. Left: QV growth for IBM, IonQ, and Quantinuum from 2019–2024. IBM grew from QV=8 (2019) to QV=4096 (2024 Heron R2) — roughly doubling every 12–18 months. Quantinuum achieved QV=2048 in 2021 with only 20 physical qubits, demonstrating that qubit quality (not count) drives QV. IonQ reached QV=512 in 2022. The dashed trend line shows the ~2× per year improvement trajectory. Right: CLOPS values across IBM processor generations showing 17× improvement from Falcon (2021) to Heron R2 (2024), driven by faster gates, improved electronics, and better compilation.">
<figcaption>Figure 7.7: Quantum Volume progress and CLOPS metric. Left: QV growth for IBM, IonQ, and Quantinuum from 2019–2024. IBM grew from QV=8 (2019) to QV=4096 (2024 Heron R2) — roughly doubling every 12–18 months. Quantinuum achieved QV=2048 in 2021 with only 20 physical qubits, demonstrating that qubit quality (not count) drives QV. IonQ reached QV=512 in 2022. The dashed trend line shows the ~2× per year improvement trajectory. Right: CLOPS values across IBM processor generations showing 17× improvement from Falcon (2021) to Heron R2 (2024), driven by faster gates, improved electronics, and better compilation.</figcaption>
</figure>

#### Interpreting QV

QV = 64 = 2^6 means the processor reliably executes random 6-qubit, 6-layer circuits with HOGP > 2/3. A 6-qubit, 6-layer random circuit contains approximately 6×6/2 = 18 SU(4) entangling gates. Achieving QV=64 therefore requires: at least 6 high-quality qubits with sufficient connectivity, gate fidelity high enough that 18 entangling gates maintain >2/3 heavy-output probability, and connectivity sufficient that the entangling structure of random circuits can be implemented with minimal SWAP overhead.

The QV metric elegantly combines all of these factors. Note that QV = 4096 = 2^12 from IBM's Heron R2 does NOT mean the processor reliably executes arbitrary 12-qubit × 12-layer circuits — it means this specific test (random circuits with optimal qubit selection and transpilation) passes at this scale.

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: QV Is Not a Universal Circuit Capability Metric</strong></p>
<p>QV measures performance on a specific type of random circuit with random SU(4) gates. It is an excellent measure of overall processor quality but does not directly predict performance on all algorithms. A processor with QV=128 may excel at shallow variational circuits (VQE, QAOA) but underperform on deep Grover circuits, or vice versa, depending on the specific connectivity and gate error distribution. For algorithm-specific benchmarking, use error mitigation + direct algorithm execution benchmarks.</p>
</div>

### 7.6.2 CLOPS: Circuit Layer Operations Per Second

While QV measures the quality of computation (how reliably circuits execute), CLOPS (Circuit Layer Operations Per Second) measures the throughput — how fast circuits are processed end-to-end in a real workload. Introduced by IBM in 2021, CLOPS accounts for all real-world latencies: gate execution time, classical control latency, circuit compilation time, and cloud interface overhead.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: CLOPS Definition</strong></p>
<p>CLOPS measures how many QV-circuit-equivalent layers are completed per second in a realistic workload. For a variational algorithm (VQE, QAOA) requiring repeated circuit execution with updated parameters, CLOPS directly determines the wall-clock time per optimisation iteration.</p>
<p>Example: A processor with CLOPS = 10,000 running a QAOA circuit with D = 10 layers, 1,000 shots per iteration, 500 optimiser iterations: total time ≈ 1000 × 500 × 10 / 10,000 = 500 seconds ≈ 8 minutes. The same calculation on a Falcon processor (CLOPS = 850) would take ≈ 5,882 seconds ≈ 98 minutes — nearly 12× longer.</p>
</div>

### 7.6.3 NISQ Era Limitations

The NISQ (Noisy Intermediate-Scale Quantum) era — coined by John Preskill in 2018 — describes quantum hardware with 50–1,000+ qubits that is too noisy for full fault-tolerant operation. Understanding NISQ limitations is not pessimism; it is essential engineering knowledge for designing algorithms and applications that work within these constraints.

#### The Fundamental Tension: Scale vs. Quality

The central challenge of the NISQ era is that increasing qubit count without proportional improvement in gate fidelity does not produce more useful computation. A 1,000-qubit processor with 0.5% two-qubit gate error cannot execute an algorithm requiring 1,000 qubits of reliable quantum computation — after 200 gates, fidelity is below 37% regardless of how many qubits are available.

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>Coherence limit:   d_max ~ T2 / t_gate  ≈ 100,000 μs / 0.3 μs = 333 layers</p>
<p>Gate-error limit:  d_max = log(F_thresh)/log(1-p)  ≈ log(0.5)/log(0.997) ≈ 231</p>
<p>For p=0.3%, circuit fidelity F = 0.5 after ~231 CX gates (gate errors dominate)</p>
</div>

#### Specific NISQ Limitations

- No fault-tolerant error correction: QEC requires ~1,000–10,000 physical qubits per logical qubit and gate fidelities >99.9% (the fault-tolerance threshold for the surface code). Current devices have far too few qubits and too many errors for fully fault-tolerant computation.

- Restricted connectivity: IBM heavy-hex lattice — each qubit has 2–3 neighbours; implementing a gate between non-adjacent qubits requires SWAP routing, tripling circuit depth. Algorithms designed for all-to-all connectivity (e.g., Shor's algorithm) are particularly penalised.

- Readout errors (SPAM): 1–3% readout errors on typical qubits; mitigated by calibration matrix inversion but residual errors affect measurement statistics.

- Crosstalk: Nearby qubits interact through residual couplings even when not gating; simultaneous gates on neighbouring qubits can produce correlated errors not captured by independent noise models.

- Calibration drift: Qubit frequencies and gate calibrations drift on timescales of hours due to temperature fluctuations and TLS instabilities; circuits executed after calibration may have different error rates than expected.

- Limited circuit depth: For current NISQ hardware, practical algorithms are limited to ~50–200 two-qubit gates before decoherence and gate errors overwhelm the signal.

#### Near-Term Paths to Practical Advantage

- Error mitigation: Zero-Noise Extrapolation (ZNE) — run circuits at multiple noise levels and extrapolate to zero noise. Probabilistic Error Cancellation (PEC) — invert the noise channel probabilistically. These add 10–1000× classical overhead but can halve effective error rates without full QEC.

- Shallow variational algorithms: VQE and QAOA use circuits of depth 2–100 that are feasible on current hardware; their heuristic nature allows partial results even with noise.

- Analog quantum simulation: Use the quantum system itself as a simulator for specific Hamiltonians (Hubbard model, Ising model), bypassing the digital gate model entirely.

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: The Race to Fault Tolerance: 2023–2024 Milestones</strong></p>
<p>Two landmark results mark the transition from NISQ to fault-tolerant quantum computing: (1) Google Willow (2024): 105-qubit superconducting processor achieved below-threshold surface code error correction — the first experimental demonstration that adding more physical qubits reduces (not worsens) logical error rates, with logical error rates decreasing by factor 2 per code distance increase. (2) Microsoft + Quantinuum (2024): Demonstrated 4 logical qubits with logical error rates 800× better than physical error rates using Quantinuum H2 hardware and Microsoft's topological qubit compilation. Timeline to fault-tolerant quantum advantage for commercially relevant problems: most analysts estimate 2028–2035, requiring 10,000–1,000,000+ physical qubits depending on the application.</p>
</div>

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 7: Quantum Hardware and Noise

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Describe the five main physical qubit platforms. For each, give the qubit type, operating temperature, control method, and one key advantage and disadvantage.**

**Q2.  What is a Josephson junction? Describe the Josephson effect and write the two Josephson relations (current-phase and voltage-phase). Why is the Josephson junction essential for superconducting qubits?**

**Q3.  Describe the transmon qubit Hamiltonian H = 4E\_C(n−n\_g)² − E\_J cos(φ). What is the transmon regime (E\_J/E\_C >> 1)? What problem does the large shunt capacitor solve?**

**Q4.  Explain the anharmonicity α = E\_{12} − E\_{01} of the transmon. Why is anharmonicity essential for qubit control? What typical value does it take and what limits it?**

**Q5.  Explain the cross-resonance two-qubit gate used by IBM. What is the effective interaction Hamiltonian? How is CNOT implemented from it?**

**Q6.  Define T₁ (relaxation time) and write the amplitude damping channel Kraus operators. How do K₀ and K₁ physically describe the T₁ process?**

**Q7.  Define T₂ (dephasing time). Write the relation 1/T₂ = 1/(2T₁) + 1/T₂\_pure. What is the difference between T₂ measured by free induction decay (T₂\*) and Hahn echo (T₂)?**

**Q8.  Derive the Kraus operators for the bit-flip channel ε(ρ) = (1−p)ρ + p·XρX. Verify the completeness condition Σ\_k K\_k†K\_k = I. Show the Bloch vector effect.**

**Q9.  Write the single-qubit depolarising channel. Derive that the Bloch vector undergoes uniform contraction r → (1−4p/3)·r. At what value of p does the channel produce the maximally mixed state?**

**Q10.  What is Quantum Volume? Write the protocol step by step. What does 'heavy output generation probability > 2/3' mean physically?**

**Q11.  What is CLOPS? Write the formula. Why is CLOPS important for variational algorithms like VQE and QAOA?**

**Q12.  Describe the Circuit QED readout mechanism. What is the dispersive coupling shift ±χ? How is the qubit state encoded in the resonator response?**

**Q13.  A qubit has T₁ = 150 μs and T₂ = 80 μs. A circuit requires 60 two-qubit gates at 300 ns each. Estimate the fidelity loss due to decoherence alone.**

**Q14.  Compare trapped-ion and superconducting qubits on: T₂, gate time, gate fidelity, qubit count (2024), connectivity, and cooling requirements.**

**Q15.  What is the surface code? What is its error threshold? Why does achieving below-threshold error rates not immediately enable fault-tolerant quantum computing?**

**PART B — MODEL ANSWERS**

**Answer 1:**

Superconducting: Josephson junction circuit; 10–20 mK; microwave pulses; advantage: fast gates (10–300 ns), mature, large scale; disadvantage: requires dilution refrigerator. Trapped-ion: hyperfine/optical levels of ¹⁷¹Yb⁺; room temperature vacuum chamber; laser pulses; advantage: best fidelity (>99.9%), all-to-all connectivity, long T₂; disadvantage: slow gates (ms), complex laser infrastructure. Neutral atom: Rb/Cs Rydberg states; MOT/tweezer at μK; laser excitation; advantage: programmable topology, large arrays; disadvantage: lower gate fidelity. Photonic: polarisation/path modes; room temperature; beam splitters/detectors; advantage: room temperature operation; disadvantage: photon loss, non-deterministic gates. Spin: electron/nuclear spin; mK–room; microwave/RF; advantage: CMOS compatible, long coherence potential; disadvantage: difficult 2-qubit gates, slow readout.

**Answer 2:**

A Josephson junction consists of a thin (~1-3 nm) insulating Al₂O₃ barrier between two superconducting aluminium electrodes. Cooper pairs (electron pairs) tunnel quantum mechanically through this barrier. Josephson relations: Current-phase: I = I\_c sin(φ) where I\_c is the critical current and φ is the phase difference. Voltage-phase: V = (Φ₀/2π)·dφ/dt where Φ₀ = h/2e ≈ 2.07×10⁻¹⁵ Wb is the flux quantum. Essential because: the nonlinear inductance L\_J(φ) = Φ₀/(2πI\_c cos(φ)) creates an anharmonic energy level structure, allowing selective microwave driving of only the |0⟩→|1⟩ qubit transition without simultaneously driving |1⟩→|2⟩.

**Answer 3:**

H\_transmon = 4E\_C(n−n\_g)² − E\_J cos(φ), where E\_C = e²/(2C\_Σ) is the charging energy and E\_J = I\_c Φ₀/(2π) is the Josephson energy. Transmon regime: E\_J/E\_C ≫ 1 (achieved by large shunt capacitor C\_sh → small E\_C). Charge dispersion (sensitivity to offset charge n\_g) scales as exp(−√(8E\_J/E\_C)). For E\_J/E\_C ≈ 50: charge dispersion ∝ exp(−20) ≈ 10⁻⁹ — reduced by ~10⁵ compared to Cooper Pair Box. The large shunt capacitor effectively averages over charge fluctuations, eliminating the dominant noise source in earlier designs.

**Answer 4:**

Anharmonicity α = E\_{12} − E\_{01} < 0, typically α/2π ≈ −200 to −350 MHz. E\_{01} = ω\_q ≈ 5 GHz is the qubit transition; E\_{12} ≈ E\_{01} + α. Essential: microwave pulses must selectively drive |0⟩→|1⟩ without simultaneously driving |1⟩→|2⟩ (leakage). A narrowband pulse with bandwidth Δν ≪ |α|/2π (e.g., DRAG pulses of 10–50 ns duration, bandwidth ~20 MHz) can selectively address the qubit. Limit: making |α| larger requires smaller E\_J/E\_C (more anharmonic) which increases charge sensitivity. Typical compromise: E\_J/E\_C ≈ 50, giving α/2π ≈ −250 MHz.

**Answer 5:**

Cross-resonance (CR) gate: drive qubit A (control) at the resonant frequency ω\_B of qubit B (target). Due to qubit-qubit coupling, the effective Hamiltonian becomes H\_CR = (Ω/2) Z\_A ⊗ X\_B — a ZX interaction. After a calibrated CR pulse of duration t = π/(2Ω): exp(−iπ Z\_A⊗X\_B/4) implements an echoed cross-resonance gate. Combined with single-qubit corrections, this produces a CNOT gate. CR gate advantage: requires no physical tuning of qubit frequencies — only microwave drives — making it compatible with fixed-frequency transmon qubits (which have lower charge sensitivity than tunable qubits).

**Answer 6:**

T₁ = energy relaxation time — time for |1⟩→|0⟩ decay. P₁(t) = P₁(0)·exp(−t/T₁). Kraus operators for amplitude damping (γ = 1−exp(−t/T₁)): K₀ = [[1,0],[0,√(1−γ)]] (no jump — system stays in its state, but |1⟩ amplitude decreases); K₁ = [[0,√γ],[0,0]] (jump — |1⟩ decays to |0⟩ with probability γ). Completeness: K₀†K₀+K₁†K₁ = diag(1,1−γ)+diag(0,γ) = I ✓. K₀ describes the continuous decay of |1⟩ amplitude; K₁ corresponds to the spontaneous emission event (energy exchange with environment).

**Answer 7:**

T₂ = phase coherence time — decay of off-diagonal density matrix elements. ρ₀₁(t) = ρ₀₁(0)·exp(−t/T₂). Relation: 1/T₂ = 1/(2T₁) + 1/T₂\_pure, where T₁ contributes to dephasing through energy relaxation (mixing populations) and T₂\_pure accounts for pure dephasing (random frequency fluctuations with no energy exchange). T₂\* (free induction decay): shorter than T₂ due to inhomogeneous broadening — qubit frequency varies between experimental runs (1/f charge noise, magnetic flux noise). Hahn echo: π-pulse at t/2 refocuses slow (low-frequency) noise, recovering T₂ > T₂\*. High-order dynamical decoupling sequences (CPMG, XY-8) can further extend coherence.

**Answer 8:**

Bit-flip channel derivation: probability (1−p): no error, apply I; probability p: bit-flip, apply X. Kraus: K₀ = √(1−p)·I, K₁ = √p·X. Completeness: K₀†K₀ + K₁†K₁ = (1−p)I + p·X†X = (1−p)I + p·I = I ✓. Bloch vector effect: ε(ρ) = (1−p)ρ + p·XρX. For ρ = (I + r·σ)/2: XρX = (I − r\_x·X + r\_y·Y·? ... X(r·σ)X: X·X=I (r\_x unchanged), X·Y·X = −Y (r\_y → −r\_y), X·Z·X = −Z (r\_z → −r\_z). Result: r → (r\_x, −(1−2p)r\_y... wait, more carefully: ε(ρ) has Bloch vector (x, (1−2p)y, (1−2p)z). The x-component is preserved (X-direction), while y,z contract by factor (1−2p).

**Answer 9:**

Depolarising: ε\_D(ρ) = (1−p)ρ + (p/3)(XρX+YρY+ZρZ). Bloch vector derivation: ε\_D(ρ) acts on (I+r·σ)/2. XρX contracts r to (−r\_x, r\_y, −r\_z); YρY to (−r\_x,−r\_y,r\_z); ZρZ to (r\_x,−r\_y,−r\_z)... wait: the sum (1−p)r + (p/3)(−r\_x+−r\_x+r\_x, r\_y+−r\_y+−r\_y, −r\_z+r\_z+−r\_z) = (1−p)r + (p/3)(−r\_x, −r\_y, −r\_z) = [1−p−p/3]r = (1−4p/3)r ✓. Maximally mixed at p=3/4: Bloch vector → (1−4·3/4/3)r = (1−1)r = 0, giving ρ = I/2 ✓.

**Answer 10:**

QV protocol: (1) Choose model size n. (2) Generate 100+ random n-qubit depth-n circuits (Haar-random SU(4) on random qubit pairs + random permutations). (3) Classically simulate each to get ideal probability distribution {p(x)}. (4) Define heavy set H(C): bitstrings with p(x) > median. (5) Run each circuit on hardware (≥100 shots). (6) Compute heavy-output generation probability: fraction of measured bitstrings that are in H(C). (7) If mean over all circuits > 2/3 with ≥97.7% statistical confidence: QV = 2^n. Physical meaning of 2/3: the threshold is midway between a perfect device (≈0.847) and a completely random device (0.5), ensuring the hardware provides meaningful quantum coherence beyond random noise.

**Answer 11:**

CLOPS = (N\_updates × N\_circuits × Layers) / T\_total, where T\_total is wall-clock time for all executions. Measures execution throughput in circuit layer operations per second. Important for VQE/QAOA: these algorithms require many circuit evaluations per optimisation step. With 50 parameters, parameter-shift needs 100 circuit evaluations per gradient. At 10,000 CLOPS and 50 layers per circuit: 100 circuits × 50 layers = 5,000 layer ops. Time per gradient step = 5,000/10,000 = 0.5 seconds. Higher CLOPS → faster gradient steps → faster convergence → more experiments per session.

**Answer 12:**

Circuit QED readout: qubit coupled to readout resonator via coupling g. In dispersive limit |Δ| = |ω\_q − ω\_r| ≫ g: effective Hamiltonian H\_disp = ω\_r a†a + (ω\_q ± χ a†a)σ\_z/2, where χ = g²/Δ. Qubit state shifts resonator: qubit |0⟩ → resonator at ω\_r − χ; qubit |1⟩ → resonator at ω\_r + χ. Probing resonator at ω\_r with microwave tone: transmission amplitude and phase differ by ±arctan(2χ/κ) depending on qubit state. Room-temperature heterodyne detection: IQ mixer extracts quadratures; discriminator classifies as 0 or 1. QND: dispersive measurement does not drive qubit transitions (to first order in χ/Δ).

**Answer 13:**

Circuit time: 60 × 300 ns = 18,000 ns = 18 μs. T₁ decoherence factor: exp(−t/T₁) = exp(−18/150) = exp(−0.12) ≈ 0.887. T₂ decoherence factor: exp(−t/T₂) = exp(−18/80) = exp(−0.225) ≈ 0.799. Combined fidelity from decoherence: ≈ 0.887 × 0.799 ≈ 0.708. Combined with gate errors (0.3% per CNOT): (1−0.003)^{60} = 0.997^{60} ≈ 0.835. Total estimated fidelity: 0.708 × 0.835 ≈ 0.591 ≈ 59%. This illustrates that even with T₁,T₂ well above circuit time, decoherence + gate errors together significantly degrade fidelity.

**Answer 14:**

T₂: Trapped-ion: 1–10 seconds (hours for nuclear spin). Superconducting: 50–300 μs. Gate time: Trapped-ion 2Q: ~ms (laser pulse, phonon mediation). Superconducting 2Q: 100–300 ns (microwave, 1000× faster). Gate fidelity (2Q): Trapped-ion: >99.9%. Superconducting: 99.0–99.8%. Qubit count (2024): Trapped-ion: 56 (Quantinuum H2-2), 32 (IonQ Forte). Superconducting: 1121 (IBM Condor), 133 (Heron). Connectivity: Trapped-ion: all-to-all via phonon bus. Superconducting: limited (heavy-hex, 2–3 neighbours). Cooling: Trapped-ion: vacuum system at room temperature (ion itself at ~mK via laser cooling). Superconducting: dilution refrigerator at 15–20 mK.

**Answer 15:**

Surface code: a 2D topological quantum error-correcting code that encodes 1 logical qubit in a d×d grid of physical qubits (d² + (d-1)² physical qubits per logical qubit). Error threshold: ~1% physical gate error rate. Below threshold, logical error rate decreases exponentially with code distance d. Above threshold: adding more physical qubits makes things worse. Achieving below-threshold error is necessary but not sufficient for practical fault-tolerant QC: (1) Need code distance d large enough to achieve desired logical error rate (e.g., d=27 for 10⁻¹² logical error per gate at physical error 0.1%). (2) Each logical qubit requires ~1000 physical qubits at 0.1% physical error. (3) For RSA-2048 via Shor: ~4000 logical qubits × 1000 physical = ~4 million physical qubits.

## EXERCISES — CHAPTER 7

### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.1</strong></p>
<p>Problem: Verify the Kraus completeness condition for the depolarising channel with Kraus operators K₀=√(1-p)·I, K₁=√(p/3)·X, K₂=√(p/3)·Y, K₃=√(p/3)·Z.</p>
<p><strong>Solution:</strong></p>
<p>K₀†K₀ = (√(1-p))² · I†I = (1-p) · I</p>
<p>K₁†K₁ = (p/3) · X†X = (p/3) · I   [X is Hermitian and X² = I]</p>
<p>K₂†K₂ = (p/3) · Y†Y = (p/3) · I   [Y is Hermitian and Y² = I]</p>
<p>K₃†K₃ = (p/3) · Z†Z = (p/3) · I   [Z is Hermitian and Z² = I]</p>
<p>Σ Kₖ†Kₖ = (1-p)I + (p/3)I + (p/3)I + (p/3)I = (1-p+p)I = I  ✓</p>
<p>The completeness condition is satisfied, confirming the depolarising channel is trace-preserving.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.2</strong></p>
<p>Problem: A superconducting qubit has T1 = 200 μs and T2 = 120 μs. (a) Find the pure dephasing time T_φ. (b) For a gate time of 50 ns, compute the error probability per gate from T1 and T2 decay alone.</p>
<p><strong>Solution:</strong></p>
<p>(a) From 1/T2 = 1/(2T1) + 1/T_φ:</p>
<p>1/T_φ = 1/T2 − 1/(2T1) = 1/120 − 1/400  [all times in μs]</p>
<p>= 400/(120×400) − 120/(120×400) = 280/48000</p>
<p>T_φ = 48000/280 ≈ 171.4 μs</p>
<p>Pure dephasing is significant: T_φ ≈ 171 μs, meaning flux/charge noise contributes ~41% of total dephasing.</p>
<p>(b) Error probability per 50 ns gate (coherence contribution only):</p>
<p>Amplitude damping error: γ = 1 − exp(−t/T1) ≈ t/T1 = 50×10⁻⁹/200×10⁻⁶ = 2.5×10⁻⁴</p>
<p>Dephasing error: 1 − exp(−t/T2) ≈ t/T2 = 50×10⁻⁹/120×10⁻⁶ = 4.2×10⁻⁴</p>
<p>Combined coherence error ≈ 3(t/T1)/4 + t/T2 ≈ 6×10⁻⁴ per gate ≈ 0.06%.</p>
<p>This is smaller than typical gate calibration errors (~0.03–0.1%), confirming that coherence is NOT the dominant error for short gates on modern devices.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.3</strong></p>
<p>Problem: Compute the action of the amplitude damping channel with γ = 0.1 on ρ = |1⟩⟨1| = [[0,0],[0,1]]. Verify trace preservation.</p>
<p><strong>Solution:</strong></p>
<p>K₀ = [[1, 0], [0, √(1−γ)]] = [[1,0],[0,√0.9]] = [[1,0],[0,0.9487]]</p>
<p>K₁ = [[0, √γ], [0, 0]] = [[0, 0.3162], [0, 0]]</p>
<p>K₀·ρ·K₀† = [[1,0],[0,0.9487]]·[[0,0],[0,1]]·[[1,0],[0,0.9487]]</p>
<p>= [[0,0],[0,0.9487]]·[[1,0],[0,0.9487]] = [[0,0],[0,0.9]]</p>
<p>K₁·ρ·K₁† = [[0,0.3162],[0,0]]·[[0,0],[0,1]]·[[0,0],[0.3162,0]]</p>
<p>= [[0,0.3162],[0,0]]·[[0,0],[0.3162,0]] = [[0.1,0],[0,0]]</p>
<p>ε_AD(ρ) = K₀ρK₀† + K₁ρK₁† = [[0,0],[0,0.9]] + [[0.1,0],[0,0]] = [[0.1,0],[0,0.9]]</p>
<p>Trace check: 0.1 + 0.9 = 1.0 ✓</p>
<p>Interpretation: Started in |1⟩; after damping with γ=0.1, the probability of |1⟩ decreased from 1.0 to 0.9 (10% decayed to |0⟩), probability of |0⟩ increased from 0 to 0.1.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.4</strong></p>
<p>Problem: A quantum processor has Quantum Volume QV = 64. (a) Precisely interpret what this means. (b) How does QV = 128 compare?</p>
<p><strong>Solution:</strong></p>
<p>(a) QV = 64 = 2^6. This means: the processor can execute a random 6-qubit, 6-layer circuit with heavy-output generation probability &gt; 2/3, verified at 97.5% statistical confidence. Concretely, the best 6 qubits form a subgraph that supports ~18 entangling gates (6 layers × ~3 SU4 pairs per layer) with sufficient fidelity that the output distribution remains &gt;2/3 heavy-output correct.</p>
<p>(b) QV = 128 = 2^7: same test passes for 7-qubit, 7-layer circuits. The "volume" doubles: one additional qubit AND one additional layer. This is a harder test — 7-qubit random circuits contain ~24 entangling gates, require 7-qubit connectivity, and require 17% more gate fidelity headroom. Going from QV=64 to QV=128 requires a genuine hardware improvement, not just more qubits.</p>
<p>Caution: QV is a lower bound on capability — a processor may execute specific structured circuits (like shallow VQE) better than QV suggests, because QV uses worst-case random circuits.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.5</strong></p>
<p>Problem: Show that the depolarising channel with p = 3/4 maps any input state ρ to the maximally mixed state I/2.</p>
<p><strong>Solution:</strong></p>
<p>Using the equivalent form: ε_D(ρ) = (1 − 4p/3)ρ + (4p/3)(I/2)</p>
<p>At p = 3/4: coefficient of ρ = 1 − 4(3/4)/3 = 1 − 1 = 0</p>
<p>Coefficient of I/2 = 4(3/4)/3 = 1</p>
<p>Therefore: ε_D(ρ) = 0·ρ + 1·(I/2) = I/2 for ANY input ρ. ✓</p>
<p>Alternatively, using Pauli form: X·ρ·X + Y·ρ·Y + Z·ρ·Z for ρ = [[a,b],[c,d]]:</p>
<p>= [[d,c],[b,a]] + [[d,−c],[−b,a]] + [[a,−b],[−c,d]] ... [expanding each term]</p>
<p>... After simplification: X·ρ·X + Y·ρ·Y + Z·ρ·Z = 2I − 2ρ·Tr[ρ]/Tr[I] ... at p=3/4 total = I/2 ✓</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.6</strong></p>
<p>Problem: Write Qiskit code to compare the GHZ state fidelity under (a) no noise, (b) depolarising p=0.01 per CX gate, (c) thermal relaxation (T1=100μs, T2=80μs, t_gate=300ns).</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.7</strong></p>
<p>Problem: A transmon has anharmonicity α/(2π) = −230 MHz and qubit frequency ω₀₁/(2π) = 5.05 GHz. A Gaussian π-pulse of duration t_pulse = 20 ns is applied. Verify selectivity: confirm the pulse does NOT accidentally drive the |1⟩→|2⟩ transition.</p>
<p><strong>Solution:</strong></p>
<p>Bandwidth of Gaussian pulse (FWHM in frequency): Δf ≈ 0.44 / t_pulse = 0.44 / (20×10⁻⁹) = 22 MHz</p>
<p>Frequency of |0⟩→|1⟩ transition: f₀₁ = 5.05 GHz  (drive is resonant here)</p>
<p>Frequency of |1⟩→|2⟩ transition: f₁₂ = f₀₁ + α/(2π) = 5.05 − 0.230 = 4.82 GHz</p>
<p>Separation: |f₁₂ − f₀₁| = 230 MHz</p>
<p>Selectivity condition: Δf &lt;&lt; |α/(2π)|</p>
<p>22 MHz &lt;&lt; 230 MHz  ✓  (ratio ~1:10.5 — excellent selectivity)</p>
<p>The pulse bandwidth 22 MHz is 10× narrower than the anharmonicity 230 MHz. The pulse spectrum at f₁₂ = 4.82 GHz is ~exp(−(230/22)²/2) ≈ exp(−54) ≈ 10⁻²³ of the peak — completely negligible. DRAG pulses suppress leakage further to below 10⁻⁴ per gate.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 7.8</strong></p>
<p>Problem: Calculate CLOPS for a system running 100 template circuits with log₂(QV) = 8 layers, 10 parameter updates each, 100 shots per execution, completing in 25 seconds. Compare with IBM Falcon (850 CLOPS).</p>
<p><strong>Solution:</strong></p>
<p>CLOPS = (M × K × S × D) / T_total</p>
<p>M = 100 templates, K = 10 parameter updates, S = 100 shots, D = 8 layers, T = 25 s</p>
<p>CLOPS = (100 × 10 × 100 × 8) / 25 = 8,000,000 / 25 = 3,200 CLOPS</p>
<p>Comparison with IBM Falcon (850 CLOPS):</p>
<p>Speedup ratio: 3,200 / 850 ≈ 3.8× faster throughput.</p>
<p>For a VQE optimisation with 500 iterations × 200 shots × 8 circuit layers:</p>
<p>Time on this system: (500×200×8) / 3200 ≈ 250 s ≈ 4.2 minutes</p>
<p>Time on IBM Falcon:  (500×200×8) / 850  ≈ 941 s ≈ 15.7 minutes</p>
<p>The 3.8× CLOPS improvement reduces wall-clock time from ~16 minutes to ~4 minutes — making the difference between a productive and an impractical optimisation loop.</p>
</div>

### B. Unsolved Problems

Solve independently. Bracketed answers for self-checking.

1. A qubit has T1 = 120 μs. Compute the fraction P(|1⟩,t)/P(|1⟩,0) remaining in the excited state at t = 60 μs, t = 120 μs, and t = 300 μs. [Answer: P(60μs) = e^(−1/2) ≈ 0.607; P(120μs) = e^(−1) ≈ 0.368; P(300μs) = e^(−2.5) ≈ 0.082]

2. A qubit has T1 = 300 μs and T2 = 200 μs. (a) Is T2 ≤ 2T1? (b) Compute T\_φ. (c) Which dephasing mechanism dominates? [Answer: (a) 200 ≤ 600 ✓; (b) 1/T\_φ = 1/200 − 1/600 = 3/600 − 1/600 = 2/600, T\_φ = 300 μs; (c) T\_φ = 300 μs ≈ T1: T1 and pure dephasing contribute equally]

3. Apply the phase-flip channel with p = 0.5 to ρ = |+⟩⟨+| = (1/2)[[1,1],[1,1]]. Compute the output and interpret. [Answer: ε\_PF(|+⟩⟨+|) = [[1/2,(1−2·0.5)/2],[(1−2·0.5)/2,1/2]] = [[1/2,0],[0,1/2]] = I/2 — maximally mixed state; at p=0.5 the phase-flip channel completely destroys the equatorial coherence]

4. Verify the completeness condition for the amplitude damping channel with γ = 0.3. Explicitly compute K₀†K₀ + K₁†K₁. [Answer: K₀†K₀ = [[1,0],[0,0.7]], K₁†K₁ = [[0,0],[0,0.3]], sum = I ✓]

5. A 25-gate CX circuit runs on a processor with per-gate depolarising error p = 0.004. (a) What is the circuit fidelity F? (b) For F > 0.90, what is the maximum number of CX gates allowed? [Answer: (a) F = (1−0.004)^25 = 0.996^25 ≈ 0.905; (b) n < log(0.9)/log(0.996) ≈ 26 gates — the 25-gate circuit barely clears the 90% threshold]

6. A trapped-ion processor has T2 = 2 s and two-qubit gate time = 0.8 ms. A superconducting processor has T2 = 150 μs and gate time = 300 ns. Compute gates-per-T2 for each. Which has more operations within one coherence time? [Answer: Ion: 2/(0.8×10⁻³) = 2500 gates/T2; SC: 150×10⁻⁶/300×10⁻⁹ = 500 gates/T2. Trapped ion has 5× more operations per T2 despite being 2667× slower per gate]

7. A transmon qubit has E\_J/h = 20 GHz and E\_C/h = 280 MHz. (a) Compute the transmon qubit frequency ω₀₁/2π ≈ √(8E\_JE\_C)/h − E\_C/h. (b) Compute the anharmonicity α/2π ≈ −E\_C/h. (c) Is the transmon condition satisfied? [Answer: (a) √(8×20000×280)/h − 280 MHz = √44,800,000 − 280 ≈ 6,693 − 280 ≈ 6,413 MHz ≈ 6.4 GHz; (b) α/2π = −280 MHz; (c) E\_J/E\_C = 20000/280 ≈ 71 >> 1 ✓]

8. QV = 256 is achieved by a processor. (a) What is n\_eff? (b) Approximately how many two-qubit gates does the test circuit contain? (c) If two-qubit fidelity is 99.6%, estimate the expected HOGP. [Answer: (a) n=8 (2^8=256); (b) 8 layers × 4 pairs per layer × 1 CX each ≈ 32 CX gates; (c) (0.996)^32 ≈ 0.879; since 0.879 > 2/3, QV=256 is consistent with 99.6% fidelity]

9. For the bit-phase-flip channel with p = 0.25, compute the Kraus operators and verify completeness. Then compute the action on ρ = |0⟩⟨0|. [Answer: K₀ = √0.75 I, K₁ = √0.25 Y; K₀†K₀+K₁†K₁ = 0.75I+0.25Y²=0.75I+0.25I=I ✓; Y|0⟩ = i|1⟩ so ε(|0⟩⟨0|) = 0.75|0⟩⟨0|+0.25|1⟩⟨1|]

10. Compute CLOPS for: M=100, K=10, S=100, D = log₂(512) = 9 layers, T=60 s. Is this more or less than IBM Eagle (~1500 CLOPS)? [Answer: CLOPS = (100×10×100×9)/60 = 900,000/60 = 1,500 CLOPS — exactly equal to IBM Eagle. This is the expected result: the QV protocol with QV=512, D=9, using IBM Eagle parameters.]

### C. Multiple Choice Questions

Circle the best answer. Answers at end of section.

**Q1. Which platform achieves the highest two-qubit gate fidelity as of 2024?**

- (a) Superconducting (IBM Heron)

- (b) Trapped ion (IonQ/Quantinuum)

- (c) Neutral atom (QuEra)

- (d) Spin qubit (Intel)

**Q2. The transmon qubit uses a large shunt capacitor primarily to:**

- (a) Increase qubit frequency above 10 GHz

- (b) Exponentially suppress sensitivity to charge noise

- (c) Increase anharmonicity to >1 GHz

- (d) Enable direct readout without a resonator

**Q3. T1 relaxation (amplitude damping) describes:**

- (a) Phase randomisation from flux noise

- (b) Spontaneous decay of |1⟩ to |0⟩ (energy relaxation)

- (c) Time between error correction cycles

- (d) Gate operation time for single-qubit gates

**Q4. The fundamental upper bound on T2 relative to T1 is:**

- (a) T2 ≥ T1

- (b) T2 ≤ T1

- (c) T2 ≤ 2T1

- (d) T2 = T1 always

**Q5. A Kraus representation of a quantum channel must satisfy:**

- (a) Σ Kₖ = I

- (b) Σ Kₖ†Kₖ = I

- (c) Each Kₖ is unitary

- (d) At most 2 Kraus operators for a qubit

**Q6. The depolarising channel with p = 3/4 maps any state ρ to:**

- (a) The ground state |0⟩⟨0|

- (b) The excited state |1⟩⟨1|

- (c) The maximally mixed state I/2

- (d) The original state ρ (identity)

**Q7. In circuit QED dispersive readout, the qubit state is inferred from:**

- (a) Direct measurement of qubit fluorescence

- (b) Frequency shift of the coupled readout resonator

- (c) Tunnelling current through the Josephson junction

- (d) Microwave amplitude absorbed by the qubit

**Q8. The Josephson energy E\_J = I\_c·ħ/(2e) for I\_c = 30 nA is approximately:**

- (a) 0.5 GHz

- (b) 5 GHz

- (c) 15 GHz

- (d) 50 GHz

**Q9. Quantum Volume QV = 128 means:**

- (a) The processor has 128 qubits

- (b) The processor achieves 128 CLOPS

- (c) A random 7-qubit, 7-layer circuit passes with HOGP > 2/3

- (d) The processor runs 128 gates per microsecond

**Q10. CLOPS measures:**

- (a) Gate fidelity per circuit layer

- (b) Circuit layer throughput including all classical overhead

- (c) Number of logical qubits per physical qubit

- (d) Decoherence rate per circuit layer

**Q11. Trapped-ion qubits have all-to-all connectivity because:**

- (a) Each ion has direct capacitive coupling to all others

- (b) Shared phonon modes of the ion chain can mediate gates between any pair

- (c) Laser beams can reach any ion regardless of position

- (d) The Coulomb interaction is infinite-range, bypassing the trap geometry

**Q12. The DRAG pulse technique reduces:**

- (a) Readout errors by improving dispersive coupling

- (b) T1 decay by reducing quasiparticle tunnelling

- (c) Leakage to the |2⟩ state by adding a derivative component

- (d) Crosstalk between adjacent qubits during two-qubit gates

**Q13. In the bit-flip channel ε(ρ) = (1-p)ρ + pXρX, which state is completely UNAFFECTED?**

- (a) |0⟩

- (b) |1⟩

- (c) |+⟩ = (|0⟩+|1⟩)/√2

- (d) |i⟩ = (|0⟩+i|1⟩)/√2

**Q14. Qiskit's thermal\_relaxation\_error function requires which input parameters?**

- (a) Gate error rate p only

- (b) T1, T2, and gate time

- (c) T1 and T2 only (gate time inferred)

- (d) E\_J, E\_C, and qubit frequency

**Q15. A NISQ limitation that does NOT apply to fully fault-tolerant quantum computers is:**

- (a) Qubits exist

- (b) Decoherence — all qubits have T1 and T2

- (c) Accumulating uncorrected gate errors limiting practical circuit depth

- (d) Qubit state measurement is required for algorithm output

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 7</strong></p>
<p><strong>Q1: (b)   Q2: (b)   Q3: (b)   Q4: (c)   Q5: (b)</strong></p>
<p><strong>Q6: (c)   Q7: (b)   Q8: (c)   Q9: (c)   Q10: (b)</strong></p>
<p><strong>Q11: (b)  Q12: (c)  Q13: (c)  Q14: (b)  Q15: (c)</strong></p>
<p>Q8 Detail: E_J/h = I_c·Φ₀/(2πh) = I_c/(2e/h) = 30×10⁻⁹ × 2.068×10⁻¹⁵ / (2π × 6.626×10⁻³⁴) ≈ 14.9 GHz ≈ 15 GHz.</p>
</div>

### D. Theory Questions

- Explain, with equations, why the transmon qubit requires E\_J/E\_C >> 1 for low charge noise. What property is exponentially suppressed in this limit, and what property is sacrificed?

- Derive the relationship 1/T2 = 1/(2T1) + 1/T\_φ from first principles. What physical processes contribute to each term, and what is the maximum possible T2?

- State the Kraus operator representation theorem for quantum channels. What are the conditions that Kraus operators must satisfy? Give one example showing why a non-CPTP map fails physically.

- Explain the circuit QED dispersive readout mechanism in detail. What is the dispersive shift χ? How does the resonator frequency change with qubit state, and how is this used to infer the qubit state?

- Compare superconducting and trapped-ion qubit platforms across: (a) gate speed, (b) coherence time, (c) operations per T2, (d) connectivity, (e) scale. Under what circumstances would you choose each platform for a given algorithm?

- Define Quantum Volume (QV) precisely. Explain each step of the QV measurement protocol. Why does QV capture more information about hardware quality than individual metrics like qubit count or gate fidelity alone?

- Explain why the depolarising channel is a special case of a quantum channel. Show that at p = 3/4 it produces the maximally mixed state for any input. What is the action of the depolarising channel on the Bloch sphere?

- What is the Purcell effect in superconducting qubits, and how does it contribute to T1 decay? How do Purcell filters mitigate this? Write the Purcell decay rate formula and explain each parameter.

- Describe three NISQ-era error mitigation techniques (not correction). For each, explain: the physical principle, the classical overhead it requires, and its limitation.

- Explain the all-to-all connectivity advantage of trapped-ion qubits via the phonon bus mechanism. Why does this reduce the number of gates required compared to nearest-neighbour superconducting qubits? Quantify the SWAP overhead difference for a 5-qubit all-to-all vs linear-chain circuit.

### E. Programming Assignments

PA7.1 — Coherence Time Measurement Simulation. Build a Qiskit Aer program to simulate T1 and T2 measurements: (a) T1 inversion recovery: prepare |1⟩, add delay(τ), measure — sweep τ from 0 to 500 μs and fit P(1) to exp(−τ/T1) using scipy.optimize.curve\_fit. Use AerSimulator with thermal\_relaxation\_error(T1=100μs, T2=80μs). (b) T2 Ramsey: prepare |+⟩ with Ry(π/2), add delay(τ) with deliberate 50 kHz detuning, apply Ry(π/2), measure — sweep τ from 0 to 200 μs and fit to A·cos(2π·Δf·τ)·exp(−τ/T2) + C. (c) Compare extracted T1, T2 with input values; assess fit accuracy. Submit: code, decay/fringe plots with fits, table comparing input vs extracted parameters.

PA7.2 — Noise Model Comparison for Grover's Algorithm. Implement and compare three noise scenarios for a 4-qubit Grover search (target |1011⟩, 3 iterations): (a) Ideal (no noise), (b) Depolarising: 0.2% single-qubit, 0.8% CX gate, (c) Full thermal relaxation: T1=100μs, T2=80μs, t\_1q=50ns, t\_2q=300ns, plus ReadoutError 1.5%. For each: measure success probability (P(finding |1011⟩)), circuit depth, and total gate count after transpilation. Produce: bar chart of success probabilities, discussion of which noise source dominates (hint: compare depolarising vs thermal error rates per gate), and a 500-word analysis.

PA7.3 — Quantum Volume Estimation. Implement a simplified QV estimation in Qiskit Aer: (a) For n = 2, 3, 4, 5 qubits, generate 50 random n-qubit, n-layer circuits (use qiskit.circuit.random.random\_circuit(n, n, measure=False) with max\_operands=2). (b) For each circuit, compute the ideal heavy-output set by classically simulating with AerSimulator(method="statevector"). (c) Run the same circuits on AerSimulator with FakeSherbrookeV2 noise model. (d) Compute the heavy-output generation probability (HOGP) for each n. (e) Determine the maximum n where HOGP > 2/3 — this is your estimated QV exponent for Sherbrooke. Compare with IBM's reported QV for this backend. Submit: code, HOGP vs n plot with the 2/3 threshold line, and brief analysis.

### F. Project Suggestions

Project 7.A — Transmon Qubit Characterisation on Real IBM Hardware. Connect to a real IBM Quantum backend via Qiskit IBM Runtime. (a) Use backend.properties() to retrieve T1, T2, and gate error rates for all qubits; plot a qubit map with T1/T2 colour-coded. (b) Run inversion recovery experiments on 3 different qubits using the Qiskit Experiments library (T1Experiment) to measure T1 directly. (c) Run Ramsey experiments (T2RamseyExperiment) to measure T2. (d) Run Clifford randomised benchmarking (StandardRB) on 3 single-qubit gates and 2 two-qubit pairs. (e) Compute the pure dephasing time T\_φ from your T1 and T2 measurements. Write a 2,500-word hardware characterisation report comparing measured values with IBM's reported values, explaining sources of discrepancy, and ranking the available qubits by quality for algorithm use.

Project 7.B — Quantum Channel Tomography. Implement quantum process tomography (QPT) to characterise a noisy channel in Qiskit Aer: (a) Define a composite noise channel using thermal\_relaxation\_error + depolarizing\_error for a single qubit. (b) Implement QPT: prepare each of the 4 Pauli eigenstates (|0⟩, |1⟩, |+⟩, |+i⟩), apply the channel, measure in all 3 Pauli bases (12 total experiments, 16 data points). (c) Use the linear inversion method to reconstruct the process matrix χ (4×4 complex). (d) Extract the Kraus operators from χ via eigendecomposition. (e) Compare your reconstructed χ with the theoretical expectation. Write a 2,000-word report explaining QPT methodology, your results, and the accuracy of reconstruction.

Project 7.C — Cross-Platform Algorithm Performance Comparison. Implement the same quantum algorithm (Grover search, 4-qubit target) on three simulated platforms: (a) IBM superconducting (FakeSherbrookeV2 noise model — realistic nearest-neighbour CX topology), (b) Trapped-ion simulator (custom noise model: T1=10s, T2=1s, all-to-all connectivity with MS gate error 0.2%), (c) Neutral atom simulator (custom model: Rydberg CZ gate 0.4% error, programmable connectivity). For each: transpile to native gates, simulate, measure Grover success probability and total circuit depth. Write a 3,000-word comparative analysis explaining how platform connectivity and error rates interact to determine algorithm performance, which platform is most suitable for this specific problem, and how your results change as the target problem scales to 8 qubits.

## References and Further Reading

1. Koch, J. et al. (2007). Charge-insensitive qubit design derived from the Cooper pair box. Physical Review A, 76, 042319. [Original transmon paper]

2. Krantz, P. et al. (2019). A quantum engineer's guide to superconducting qubits. Applied Physics Reviews, 6, 021318. [Comprehensive 81-page review of superconducting qubits and cQED]

3. Bruzewicz, C. D. et al. (2019). Trapped-ion quantum computing: Progress and challenges. Applied Physics Reviews, 6, 021314.

4. Nielsen, M. A. & Chuang, I. L. (2000). Quantum Computation and Quantum Information. Cambridge University Press. [Chapter 8: quantum noise and quantum operations]

5. Cross, A. W. et al. (2019). Validating quantum computers using randomized model circuits. Physical Review A, 100, 032328. [Quantum Volume benchmark]

6. Wack, A. et al. (2021). Quality, Speed, and Scale: Three Key Attributes to Measure the Performance of Near-Term Quantum Computers. arXiv:2110.14108. [CLOPS definition]

7. Josephson, B. D. (1962). Possible new effects in superconductive tunnelling. Physics Letters, 1, 251. [Nobel-winning prediction]

8. Preskill, J. (2018). Quantum Computing in the NISQ Era and Beyond. Quantum, 2, 79. [Coined "NISQ"]

9. Google Quantum AI (2024). Quantum error correction below the surface code threshold. Nature, 614, 676. [Below-threshold demonstration]

10. IBM Quantum (2024). IBM Quantum documentation and hardware specifications. https://docs.quantum.ibm.com
