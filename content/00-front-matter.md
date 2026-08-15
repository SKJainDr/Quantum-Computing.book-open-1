# Cover Page

# Dedicated to

# My beloved Mother

<img class="fig-img" src="content/images/image2.png" alt="figure">

**Late Smt. Satya Vati Jain**

**Q.C. Series | Volume I**

## QUANTUM COMPUTERS

## Principles, Qubits, Circuits, Algorithms & Hardware

A Comprehensive University Textbook for M.Sc. Physics for Specialization in Quantum Computing

**Dr. Sanjeev Kumar Jain**

Associate Professor and Ex-Head,

Department of Applied Sciences and Humanities  ·  Faculty of Science

Invertis University, Bareilly (U.P.), India

**An Overview**

| Programme | M.Sc. Physics — Quantum Computing Specialization |
|---|---|
| Semester | Semester III |
| This volume for Course | Quantum Computers |
| Chapter 1 | The Quantum Revolution: Computing at the Edge of Nature |
| Chapter 2 | The Language of Quantum Information: Mathematics and the Qubit |
| Chapter 3 | Quantum Gates: Single-Qubit Gates, Multi-Qubit Gates and Universality |
| Chapter 4 | Quantum Circuits: Design, Transpilation & Programming Frameworks |
| Chapter 5 | Quantum Algorithms: Oracle Models, QFT & Phase Estimation |
| Chapter 6 | Grover's Search and Variational Quantum Algorithms |
| Chapter 7 | Quantum Hardware and Noise |
| Chapter 8 | Qiskit Aer Noise Simulation and Hardware Benchmarking |
| Chapter 9 | Accessing IBM Quantum Hardware |
| Chapter 10 | Other Quantum Platforms: Cirq, Braket, and Azure Quantum |
| Platform | Python / Qiskit 1.x  ·  IBM Quantum Hardware |
| Features | Each Chapter has: 6 types of Information boxes, 15+ Recap Qs, 8+ Solved Examples · 15+ MCQs · 10+ Theory Qs · 10+ Problems · Figures, Programming Assignments and Project suggestions. |

## PREFACE

This textbook is written for students of M.Sc. Physics encountering quantum computing for the first time as part of the Quantum Computing Specialization. The goal is dual: to provide deep theoretical understanding firmly grounded in quantum mechanics, and to develop genuine practical programming ability using Qiskit on IBM Quantum hardware. Both goals are essential — theory without practice leads to abstract knowledge that cannot be applied; practice without theory leads to "quantum cargo cult" programming where one can run circuits without understanding why they work.

Quantum computing is one of the most exciting frontiers of twenty-first-century science. It sits at the intersection of quantum physics, computer science, mathematics, and engineering. For physics students, it offers the rare satisfaction of applying deep quantum mechanical principles — superposition, entanglement, interference, measurement — to solve computational problems that are genuinely intractable on the most powerful classical computers ever built. For India's M.Sc. students specifically, the National Quantum Mission (NQM, ₹6,003 crore, 2023–2031) has created extraordinary career opportunities: positions in quantum software development, hardware research, error correction, and algorithm design are opening at TCS, Wipro, Infosys, QpiAI, BosonQ Psi, and at the NQM hubs being established at IITs, IISc, and DRDO.

### How This Textbook Is Structured

The textbook is divided into five units spanning ten chapters, that build up the subject interestingly and thoroughly:

• **Unit I (Chapters 1–2): Foundations**. Chapter 1 introduces quantum computing historically and conceptually, with no mathematics — only ideas, stories, and intuitions. Chapter 2 builds the complete mathematical foundation: Hilbert spaces, Dirac notation, the qubit, Bloch sphere, density matrices, and the four fundamental principles of quantum information.

• **Unit II (Chapters 3–4): Gates and Circuits**. Chapter 3 develops the complete vocabulary of quantum gates — Pauli gates, Hadamard, phase gates, rotation gates, multi-qubit gates, and universality. Chapter 4 covers the art of quantum circuit design: depth, width, T-count, circuit identities, ancilla qubits, uncomputation, transpilation, and a survey of programming frameworks.

• **Unit III (Chapters 5–6): Algorithms**. Chapter 5 develops oracle-based algorithms (Deutsch, Deutsch-Jozsa, Bernstein-Vazirani, Simon), the Quantum Fourier Transform, and Quantum Phase Estimation. Chapter 6 covers Grover's search algorithm (with geometric proof) and variational quantum algorithms (VQE and QAOA) for the NISQ era.

• **Unit IV (Chapters 7–8): Hardware and Noise**. Chapter 7 surveys physical qubit platforms, develops the transmon qubit from Josephson junction physics through microwave control to circuit QED readout, and introduces quantum noise channels. Chapter 8 provides the complete treatment of noise simulation in Qiskit Aer, error mitigation, and hardware benchmarks (Quantum Volume, CLOPS).

• **Unit V (Chapters 9–10): Cloud Access**. Chapter 9 is a complete practical guide to accessing IBM Quantum hardware. Chapter 10 surveys other platforms: Google Cirq, Amazon Braket, and Microsoft Azure Quantum.

**The laboratory manual - Quantum Computing Lab I** - supports the laboratory part, and that practical course should be run together with the theory course.

### Pedagogical Features

Each chapter contains:

•	📜 Anecdote boxes: Historical stories and scientists — making the subject interesting.

•	🔑 Key Concept boxes: Formal definitions, precisely stated.

•	🌐 Real World boxes: Applications in industry, government, India's quantum mission.

•	**⚠** Warning boxes: Common misconceptions and pitfalls.

•	Example boxes: 8+ worked examples per chapter, step-by-step.

•	Equation boxes: Key mathematical formulas with physical interpretation.

•	Dark code blocks: Complete Qiskit programs with line-by-line commentary.

•	Figures: Labelled, captioned figures throughout.

- Each chapter closes with: 15+ Recap Qs  (with model answers) · 8+ Solved Problems · 10+ Unsolved Problems (answers in brackets) · 15+ MCQs (answers collected at the very end of the chapter) · 10+ Theory Questions · Programming Assignments · Project Suggestions — a focused review designed for strengthening understanding, application and for out of the classroom preparation.

The reader is encouraged to discuss among colleagues and try out all the examples, questions, program codes and the problems given at the end of a chapter. Consider them a part of the learning that can be gained from the chapter. Assignments and projects are designed for field learning.

### A note on honesty

Quantum computing is a field where media coverage often outpaces scientific reality. This textbook makes a deliberate effort to be precise about what quantum computers can do today (NISQ era, severely limited by noise), what they will likely do in 5–10 years (early error-corrected systems), and what they might do in 20+ years (large-scale fault-tolerant systems). Students who understand these distinctions will be better researchers, better engineers, and better communicators of science to the public.

The generation studying from this textbook will be among the first Indian-trained quantum scientists to work on nationally funded quantum hardware, quantum communication networks, and quantum software. It is hoped that this book contributes, in a small way, to prepare them for that responsibility.

— **Dr. S. K. Jain**

## The various information boxes in this textbook

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  KEY CONCEPT BOXES</strong></p>
<p>Contain the formal mathematical definitions, theorems, and circuit constructions that you must know for examinations. Read these carefully and ensure you can reproduce the key equations.</p>
</div>

<div class="box box-anecdote">
<p class="box-title"><strong>📜  ANECDOTE BOXES</strong></p>
<p>Provide historical context and the human stories behind the science. These are not examinable but are important for understanding how the field developed and for communicating science to non-specialists.</p>
</div>

<div class="box box-real-world">
<p class="box-title"><strong>🌍  REAL WORLD BOXES</strong></p>
<p>Connect theory to current industrial applications, national programmes (especially India's NQM), and career-relevant context. These appear in examination short-answer questions.</p>
</div>

<div class="box box-warning">
<p class="box-title"><strong>⚠️  WARNING BOXES</strong></p>
<p>Explicitly correct common misconceptions. If a concept appears in a warning box, it is almost certainly something that students — and even professionals — frequently get wrong.</p>
</div>

<div class="box box-math">
<p class="box-title"><strong>🧮  MATHEMATICS BOXES</strong></p>
<p>Contain worked derivations and numerical examples. Work through these step-by-step, covering the solution and attempting each step yourself first.</p>
</div>

<div class="box box-generic">
<p class="box-title"><strong># CODE BOXES (Dark background)</strong></p>
<p>Contain working Qiskit/Python code. Every code box can be run on IBM Quantum (free account at quantum.ibm.com). Running the code is the best way to develop intuition.</p>
</div>
