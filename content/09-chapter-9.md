# Unit V - CHAPTER 9: Accessing IBM Quantum Hardware

# Account Setup · Backends · Calibration · Transpilation · Sampler Primitive · Job Management

<div class="box box-anecdote">
<p class="box-title"><strong>📜  The First Cloud Quantum Computer — IBM Quantum Experience, May 2016</strong></p>
<p>On 4 May 2016, IBM Research did something unprecedented: they put a 5-qubit quantum computer on the internet and made it free to use. Anyone in the world — student, hobbyist, researcher, or curious member of the public — could log in to the IBM Quantum Experience website, draw a quantum circuit using a drag-and-drop graphical interface, and run it on a real superconducting quantum processor in Yorktown Heights, New York. No dilution refrigerator required. No specialised lab. Just a web browser.</p>
<p>The response was astonishing. Within weeks, thousands of users from over 100 countries had run experiments. Within a year, 40,000 users had executed more than 300,000 quantum circuits. High school students were building Bell state generators. University physics departments were assigning homework on real quantum hardware. Researchers were publishing papers based on cloud quantum experiments. It was the democratisation of quantum computing — the moment when a technology that had existed only in a handful of elite physics labs became accessible to the world.</p>
<p>That first 5-qubit machine has evolved through generations: from the original IBM Quantum Experience, through Qiskit's launch in 2017, through the introduction of Qiskit Runtime in 2021, to today's 1,386-qubit IBM Condor and 133-qubit Heron processors with their SamplerV2 primitives, Sessions, and Python API. The democratisation of quantum computing is now a mature platform. This chapter teaches you how to use it — completely, correctly, and with sufficient depth to do real science.</p>
</div>

Chapter 9 is a practical companion to the theoretical foundations of Units I–IV. We now turn to the question that every M.Sc. Physics student asks after learning quantum circuits, gates, and noise: "How do I actually run this on a real quantum computer?" The answer involves several distinct steps: setting up an IBM Quantum account and connecting via the Qiskit IBM Runtime Python library, selecting and inspecting an appropriate backend, reading its calibration properties to understand its noise characteristics, transpiling your logical circuit to an Instruction Set Architecture (ISA) circuit compatible with the hardware, executing it using the SamplerV2 primitive, managing the job queue, and finally interpreting the hardware results against the ideal simulation.

Each of these steps is both a practical skill and a window into the physics of quantum computing. Reading calibration data teaches you about T1/T2 coherence and gate fidelity. Understanding transpilation teaches you about qubit connectivity and native gate sets. Interpreting hardware results against ideal simulation teaches you about the noise channels of Chapter 8 in action on real hardware. We develop each step fully, with complete working code, worked examples, and a clear understanding of what is happening under the hood.

## 9.1 IBM Quantum Ecosystem and Account Setup

Before writing a single line of Qiskit code, a student needs to understand what IBM Quantum is, what plans and access levels exist, and how to obtain and configure credentials. This section provides the complete setup walkthrough that works as of 2024–25, along with explanations of what each step does and why it is necessary.

### 9.1.1 The IBM Quantum Platform: Plans, Access, and What You Get

IBM Quantum operates the world's largest fleet of publicly accessible quantum computers. Access is provided through two channels, with different pricing models and capabilities:

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: IBM Quantum Access Plans (2024)</strong></p>
<p>IBM Quantum Open Plan (Free): Available to anyone with an IBM ID. Provides access to a selection of IBM Eagle (127-qubit) and Heron (133-qubit) processors with monthly usage limits. No reserved time; all jobs enter the shared public queue. Sufficient for learning, coursework, and small research experiments. Sign up at quantum.ibm.com.</p>
<p>IBM Quantum Premium Plan (Paid/Institutional): Available to IBM Quantum Network members (universities, research labs, companies). Provides access to the full processor fleet including latest-generation Heron R2 systems, reserved queue slots, Session priority, and dedicated support. Most Indian IITs with IBM partnership have Network access.</p>
<p>IBM Cloud (pay-as-you-go): Access IBM Quantum hardware through IBM Cloud using Cloud credentials rather than IBM Quantum credentials. Uses channel="ibm_cloud" instead of channel="ibm_quantum". Suitable for production workloads billed per circuit.</p>
<p>For this chapter: All code uses channel="ibm_quantum" (Open or Premium), which is the standard for academic use. The code is identical for both plans; only the available backends differ.</p>
</div>

<figure class="book-figure">
<img src="content/images/image92.png" alt="Figure 9.1: IBM Quantum ecosystem and account setup workflow. Left: The five-step account setup process from creating an IBM ID to listing available backends. Right: The Qiskit Runtime architecture stack — user Python code calls QiskitRuntimeService, which routes through the Primitives layer (SamplerV2/EstimatorV2), through the transpiler, to physical IBM Quantum hardware. The cloud layer handles authentication, queue management, and result storage.">
<figcaption>Figure 9.1: IBM Quantum ecosystem and account setup workflow. Left: The five-step account setup process from creating an IBM ID to listing available backends. Right: The Qiskit Runtime architecture stack — user Python code calls QiskitRuntimeService, which routes through the Primitives layer (SamplerV2/EstimatorV2), through the transpiler, to physical IBM Quantum hardware. The cloud layer handles authentication, queue management, and result storage.</figcaption>
</figure>

### 9.1.2 Creating an Account and Obtaining an API Token

The following steps are required exactly once — after completing them, your credentials are saved locally and you will not need to repeat the setup for future sessions.

- Go to https://quantum.ibm.com and click "Sign in". Create a free IBM ID (or use an existing one). If your institution has an IBM Quantum Network agreement, use your institutional email address — you may automatically receive Premium access.

- After logging in, click your profile icon in the top-right corner, then "Account settings" or "Profile". Find the "API token" section. Click "Copy token" to copy your personal API token — a 200-character alphanumeric string. This token authenticates all your Qiskit requests. Keep it secret: anyone with your token can use your quantum computing allocation.

- Note your "Instance" (also called "CRN" on IBM Cloud) if you have Premium access. The instance identifies which IBM Quantum Network hub, group, and project you belong to: e.g., "ibm-q/open/main" for the Open Plan or "ibm-q-research/your-university/main" for a Network instance.

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: API Token Security</strong></p>
<p>Your IBM Quantum API token is a secret credential equivalent to a password. Never commit it to a public GitHub repository, share it in a screenshot, or write it in plain text in a submitted assignment. If you accidentally expose your token, regenerate it immediately on the IBM Quantum dashboard. The save_account() call below stores the token in an encrypted local file (~/.qiskit/qiskit-ibm.json) — once saved, you do not need to pass it explicitly in code.</p>
</div>

### 9.1.3 Installing Qiskit IBM Runtime

Qiskit consists of multiple packages. For hardware access, the essential packages are qiskit (core circuit model and gate library) and qiskit-ibm-runtime (hardware connection and primitives). These should be installed in a Python 3.8+ virtual environment:

```python
# ─────────────────────────────────────────────────────────────────────────
# Installation (run once in terminal / command prompt)
# ─────────────────────────────────────────────────────────────────────────

# Recommended: create a virtual environment first
python3 -m venv qiskit_env
source qiskit_env/bin/activate          # macOS/Linux
# qiskit_env\Scripts\activate          # Windows

# Install core Qiskit and IBM Runtime
pip install qiskit                       # Core: circuit model, gates, transpiler
pip install qiskit-ibm-runtime           # IBM Quantum cloud access and primitives
pip install qiskit-aer                   # Local simulator (for comparison)

# Optional but useful
pip install matplotlib pylatexenc        # For circuit drawing and LaTeX output
pip install jupyter                      # For notebook-based development

# Verify installation
python3 -c "import qiskit; print(qiskit.__version__)"       # Should print 1.x.x
python3 -c "import qiskit_ibm_runtime; print(qiskit_ibm_runtime.__version__)"

# ─────────────────────────────────────────────────────────────────────────
# Version compatibility note (as of 2024):
# Qiskit 1.x uses SamplerV2, EstimatorV2 (PubResult API)
# qiskit-ibm-runtime >= 0.20 required for SamplerV2
# ─────────────────────────────────────────────────────────────────────────
```

### 9.1.4 QiskitRuntimeService: Connecting to the Cloud

The QiskitRuntimeService class is the central connection object. It handles authentication, backend discovery, job submission, and result retrieval. There are two usage patterns: saving credentials once (recommended) or passing them directly each session.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.1 — Setting up QiskitRuntimeService
# ─────────────────────────────────────────────────────────────────────────
from qiskit_ibm_runtime import QiskitRuntimeService

# ── STEP A: Save credentials (run ONCE; stores in ~/.qiskit/qiskit-ibm.json)
# Replace <YOUR_TOKEN> with your actual API token from quantum.ibm.com
QiskitRuntimeService.save_account(
    channel="ibm_quantum",
    token="<YOUR_TOKEN>",
    overwrite=True,          # overwrite if previously saved
    set_as_default=True,     # use as default account
)
# After this call, credentials are saved. Never call this again (or put in scripts).

# ── STEP B: Connect (every session — no token needed after save_account)
service = QiskitRuntimeService(channel="ibm_quantum")

# ── STEP C: Verify connection
print('Connection successful!')
print('Available backends:', [b.name for b in service.backends()])

# ── Alternative: IBM Cloud access (premium/pay-as-you-go)
# QiskitRuntimeService.save_account(
#     channel="ibm_cloud",
#     token="<IBM_CLOUD_API_KEY>",
#     instance="<CRN_STRING>",     # e.g. "crn:v1:bluemix:public:..."
# )
# service = QiskitRuntimeService(channel="ibm_cloud")

# ── STEP D: Check account details
print('Active account:', service.active_account())
# Output: {'channel': 'ibm_quantum', 'token': '...', 'url': 'https://...'} 
```

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: Qiskit IBM Runtime from Indian Institutions</strong></p>
<p>Students at IIT Madras, IIT Bombay, IISc Bangalore, IIT Delhi, TIFR Mumbai, and several other Indian institutions have access to IBM Quantum Premium Plan through the IBM Quantum Network. With Premium access, use instance="ibm-q-research/your-institution/main" (or the specific instance provided by your IBM Quantum coordinator). Contact your institution's quantum computing laboratory or physics department to check if your institution has Network membership. Students without institutional access can use the free Open Plan at quantum.ibm.com — fully functional for coursework, with access to 127-qubit Eagle processors.</p>
<p>Under India's National Quantum Mission (NQM, ₹6,003 crore, 2023–2031), IBM has announced partnerships to extend IBM Quantum Network access to additional Indian research institutions. The NQM technology hubs at IIT Madras and IISc are expected to receive dedicated queue allocations for quantum hardware experimentation.</p>
</div>

## 9.2 Discovering and Selecting Backends

A "backend" in Qiskit terminology is a quantum processor — a specific physical device accessible through the IBM Quantum cloud. Understanding how to list, filter, and select backends is the second step in the hardware access workflow. Different backends have different qubit counts, architectures, fidelities, and queue depths. Choosing the right backend is an important algorithmic decision.

### 9.2.1 service.backends(): Listing Available Processors

The service.backends() method returns a list of all backends accessible with your account. The Open Plan provides access to a rotating selection of production IBM Quantum systems; Premium Network access provides additional processors including the latest Heron generation.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.2 — Listing and filtering backends
# ─────────────────────────────────────────────────────────────────────────
from qiskit_ibm_runtime import QiskitRuntimeService

service = QiskitRuntimeService(channel="ibm_quantum")

# ── List ALL accessible backends ─────────────────────────────────────────
all_backends = service.backends()
print(f'Total accessible backends: {len(all_backends)}')
for b in all_backends:
    print(f'  {b.name:25s}  qubits={b.num_qubits:4d}  status={b.status().status_msg}')

# ── Filter: real hardware only (exclude simulators) ──────────────────────
real_backends = service.backends(
    simulator=False,       # exclude simulators (ibmq_qasm_simulator, etc.)
    operational=True,      # only currently operational (not in maintenance)
)

# ── Filter: minimum qubit count ──────────────────────────────────────────
large_backends = service.backends(
    min_num_qubits=127,    # at least 127 qubits
    simulator=False,
    operational=True,
)
print(f'\n127+ qubit backends: {[b.name for b in large_backends]}')

# ── Get backend object by name ────────────────────────────────────────────
backend = service.backend('ibm_sherbrooke')
print(f'\nSelected: {backend.name}')
print(f'  Qubits:            {backend.num_qubits}')
print(f'  Processor type:    {backend.processor_type}')
print(f'  Basis gates:       {backend.basis_gates}')
print(f'  Max shots:         {backend.max_shots}')
print(f'  Max circuits/job:  {backend.max_circuits}')

# ── Backend status: queue depth and availability ──────────────────────────
status = backend.status()
print(f'  Status:            {status.status_msg}')
print(f'  Jobs in queue:     {status.pending_jobs}')
print(f'  Operational:       {status.operational}')
```

<figure class="book-figure">
<img src="content/images/image93.png" alt="Figure 9.2: IBM Quantum backend overview. Left: Quantum Volume comparison for available IBM Quantum backends (2024) — ibm_torino (Heron R2, 133 qubits) leads with QV=4096; Eagle R3 processors share QV=128 or QV=256. Centre: T1/T2 coherence times for 10 sample qubits on a single backend (from backend.properties()), illustrating the per-qubit variability that motivates qubit selection in algorithm design. Right: Coupling map excerpt (7-qubit heavy-hex subgraph) coloured by CX gate error rate (green=low, red=high) — essential for choosing high-fidelity qubit pairs for two-qubit gates.">
<figcaption>Figure 9.2: IBM Quantum backend overview. Left: Quantum Volume comparison for available IBM Quantum backends (2024) — ibm_torino (Heron R2, 133 qubits) leads with QV=4096; Eagle R3 processors share QV=128 or QV=256. Centre: T1/T2 coherence times for 10 sample qubits on a single backend (from backend.properties()), illustrating the per-qubit variability that motivates qubit selection in algorithm design. Right: Coupling map excerpt (7-qubit heavy-hex subgraph) coloured by CX gate error rate (green=low, red=high) — essential for choosing high-fidelity qubit pairs for two-qubit gates.</figcaption>
</figure>

### 9.2.2 service.least\_busy(): Automatic Backend Selection

For students who want to run circuits quickly without manually monitoring queue depths, service.least\_busy() automatically selects the operational backend with the shortest current queue. This is the recommended approach for coursework and exploratory runs where the specific backend does not matter.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.3 — service.least_busy(): automatic backend selection
# ─────────────────────────────────────────────────────────────────────────
from qiskit_ibm_runtime import QiskitRuntimeService

service = QiskitRuntimeService(channel="ibm_quantum")

# ── Select the least busy real backend with ≥ 5 qubits ───────────────────
backend = service.least_busy(
    n_qubits=5,         # minimum qubit count
    operational=True,   # only currently running backends
    simulator=False,    # real hardware only
)
print(f'Least busy: {backend.name}  ({backend.num_qubits} qubits)')
print(f'Queue depth: {backend.status().pending_jobs} jobs')

# ── For algorithms needing 127 qubits: ───────────────────────────────────
backend_127 = service.least_busy(
    n_qubits=127,
    operational=True,
    simulator=False,
)
print(f'Least busy 127q: {backend_127.name}')

# ── Programmatic backend ranking by queue depth ───────────────────────────
real_ops = service.backends(simulator=False, operational=True)
ranked = sorted(real_ops, key=lambda b: b.status().pending_jobs)
print("\nBackend queue ranking (shortest first):")
for b in ranked[:5]:
    print(f'  {b.name:25s}  queue={b.status().pending_jobs:3d}  qubits={b.num_qubits}')
```

<div class="box box-example">
<p class="box-title"><strong>Example 9.1: Choosing Between Backends</strong></p>
<p>Problem: You want to run a 4-qubit Bell state experiment (2 qubits needed, 2 ancilla). You have three options: ibm_brisbane (queue=45 jobs), ibm_sherbrooke (queue=12 jobs, QV=128), ibm_torino (queue=8 jobs, QV=4096, Heron architecture). Which should you choose and why?</p>
<p><strong>Solution:</strong></p>
<p>For a simple 4-qubit Bell state circuit (shallow: 1 H + 3 CX + measurement ≈ 5 gates total):</p>
<p>(1) Circuit fidelity: All three backends have well above the ~5 CX gates needed with &gt;99% individual gate fidelity. The circuit is shallow enough that all three give similar noise outcomes.</p>
<p>(2) Queue time: ibm_torino has the shortest queue (8 jobs) but Heron architecture uses different native gates (ECR instead of CX) — transpilation will handle this automatically.</p>
<p>(3) Recommendation: ibm_torino (or ibm_sherbrooke). Use service.least_busy(n_qubits=4) to choose automatically. The tiny quality difference between Eagle and Heron is irrelevant for such a shallow circuit.</p>
<p>(4) For deep algorithms (VQE, QAOA, QFT on 10+ qubits): prefer ibm_torino (Heron) — the superior T1/T2 and lower gate errors make a measurable difference for circuits with 50+ gates.</p>
</div>

### 9.2.3 Backend Metadata: Name, Qubits, Coupling Map

Beyond queue depth and qubit count, each backend exposes rich metadata about its architecture. Understanding the coupling map — which qubits can directly interact — is essential for estimating transpilation overhead.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.4 — Exploring backend architecture metadata
# ─────────────────────────────────────────────────────────────────────────
from qiskit_ibm_runtime import QiskitRuntimeService
from qiskit.transpiler import CouplingMap

service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.backend('ibm_sherbrooke')   # 127-qubit Eagle R3

# ── Processor architecture ────────────────────────────────────────────────
print(f'Processor type:      {backend.processor_type}')
print(f'Basis gates:         {backend.basis_gates}')
# Eagle R3 basis: ['cx', 'id', 'rz', 'sx', 'x', 'measure', 'reset']
# Heron R2 basis: ['ecr', 'id', 'rz', 'sx', 'x', 'measure', 'reset']

# ── Coupling map: which qubits can have two-qubit gates ───────────────────
coupling_map = CouplingMap(backend.coupling_map)
print(f'Number of qubits:    {backend.num_qubits}')
print(f'Number of CX edges:  {coupling_map.size()}')
print(f'Coupling map type:   heavy-hex lattice')

# Display the neighbours of qubit 0
neighbors_0 = list(coupling_map.neighbors(0))
print(f'Qubit 0 neighbors:   {neighbors_0}')
# Heavy-hex: each qubit has 2-3 neighbours (degree 2 or 3)

# ── Distance matrix: SWAP cost between any two qubits ────────────────────
dist_matrix = coupling_map.distance_matrix
print(f'\nSWAP distance from Q0 to Q63: {int(dist_matrix[0,63])} SWAPs needed')
print(f'SWAP distance from Q0 to Q126: {int(dist_matrix[0,126])} SWAPs needed')
# Each SWAP adds 3 CX gates — long-range operations are expensive!

# ── Visualise the coupling map (if in Jupyter notebook) ──────────────────
# from qiskit.visualization import plot_gate_map
# plot_gate_map(backend)
```

## 9.3 Reading Hardware Calibration Data

Real quantum hardware is not uniformly noisy. Different qubits on the same chip can have T1 values ranging from 50 to 500 μs, CX gate errors from 0.2% to 2%, and readout errors from 0.5% to 5%. Reading and understanding this calibration data allows algorithm developers to make informed decisions: which qubits to use, which qubit pairs to avoid, how to estimate circuit fidelity before submitting, and whether noise mitigation is needed.

### 9.3.1 backend.properties(): T1, T2, Gate Errors, Readout Errors

The backend.properties() method returns a BackendProperties object containing the most recent calibration data for all qubits and gates. IBM updates this data approximately every 24 hours.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.5 — Reading and analysing backend calibration data
# ─────────────────────────────────────────────────────────────────────────
from qiskit_ibm_runtime import QiskitRuntimeService
import numpy as np

service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.backend('ibm_sherbrooke')
props = backend.properties()     # BackendProperties object

# ── Calibration timestamp ─────────────────────────────────────────────────
print(f'Calibration date: {props.last_update_date}')

# ── Per-qubit T1 and T2 (in seconds — convert to μs for readability) ──────
print('\nQubit   T1 (μs)   T2 (μs)   Readout Err')
print('-' * 45)
for q in range(min(10, backend.num_qubits)):
    t1  = props.t1(q) * 1e6      # convert s → μs
    t2  = props.t2(q) * 1e6
    ro  = props.readout_error(q)
    print(f'  Q{q:3d}   {t1:7.1f}   {t2:7.1f}   {ro:.4f} ({ro*100:.2f}%)')

# ── Gate errors ───────────────────────────────────────────────────────────
print('\nSingle-qubit SX gate errors:')
sx_errors = []
for q in range(min(10, backend.num_qubits)):
    err = props.gate_error('sx', q)
    sx_errors.append(err)
    print(f'  Q{q:3d}  SX error = {err:.5f} ({err*100:.3f}%)')
print(f'  Mean SX error: {np.mean(sx_errors)*100:.3f}%')

# ── Two-qubit CX/ECR errors (for each coupling edge) ─────────────────────
print('\nTwo-qubit gate errors (first 8 edges):')
cx_errors = []
gate_name = 'cx' if 'cx' in backend.basis_gates else 'ecr'  # Eagle=cx, Heron=ecr
for gate in props.gates:
    if gate.gate == gate_name and len(gate.qubits) == 2:
        q1, q2 = gate.qubits
        # gate_error returns the error for this specific qubit pair
        err = props.gate_error(gate_name, [q1, q2])
        cx_errors.append(err)
        if len(cx_errors) <= 8:
            print(f'  Q{q1}–Q{q2}: {gate_name.upper()} error = {err:.5f} ({err*100:.3f}%)')
print(f'  Mean {gate_name.upper()} error: {np.mean(cx_errors)*100:.3f}%')
print(f'  Best {gate_name.upper()} error: {np.min(cx_errors)*100:.3f}%')
print(f'  Worst {gate_name.upper()} error: {np.max(cx_errors)*100:.3f}%')

# ── Find the best qubit pair (lowest CX error) ────────────────────────────
best_err = float("inf"); best_pair = None
for gate in props.gates:
    if gate.gate == gate_name and len(gate.qubits) == 2:
        err = props.gate_error(gate_name, gate.qubits)
        if err < best_err:
            best_err = err; best_pair = gate.qubits
print(f'\nBest qubit pair: Q{best_pair[0]}-Q{best_pair[1]}  ({gate_name.upper()} err={best_err*100:.3f}%)')
```

<figure class="book-figure">
<img src="content/images/image94.png" alt="Figure 9.3: Transpilation pipeline and circuit overhead. Left: The four-pass transpilation pipeline — Layout (map logical to physical qubits), Routing (insert SWAP gates), Translation (decompose to native basis), and Optimisation (cancel redundant gates). Optimisation level 3 applies the most aggressive passes including Clifford resynthesis and Sabre-based layout+routing. Right: Circuit statistics before and after transpilation for a 5-qubit GHZ circuit on IBM Eagle — transpilation increases gate count (9→17) and depth (5→12) due to SWAP insertion and basis gate decomposition, illustrating the overhead of nearest-neighbour routing.">
<figcaption>Figure 9.3: Transpilation pipeline and circuit overhead. Left: The four-pass transpilation pipeline — Layout (map logical to physical qubits), Routing (insert SWAP gates), Translation (decompose to native basis), and Optimisation (cancel redundant gates). Optimisation level 3 applies the most aggressive passes including Clifford resynthesis and Sabre-based layout+routing. Right: Circuit statistics before and after transpilation for a 5-qubit GHZ circuit on IBM Eagle — transpilation increases gate count (9→17) and depth (5→12) due to SWAP insertion and basis gate decomposition, illustrating the overhead of nearest-neighbour routing.</figcaption>
</figure>

### 9.3.2 Coupling Maps and Native Gate Sets

Every IBM Quantum processor uses a "heavy-hex" lattice coupling map — a specific graph topology where each qubit has at most 3 neighbours (degree ≤ 3). This topology was chosen to minimise ZZ crosstalk between adjacent qubits: the heavy-hex lattice has a lower edge density than a grid, meaning fewer "accidental" interactions between neighbouring qubits during gates.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Heavy-Hex Lattice and Native Gate Sets</strong></p>
<p>The heavy-hex lattice is a graph where qubits sit on vertices and two-qubit gates can only be applied on edges. For a 127-qubit Eagle processor, the heavy-hex lattice has approximately 144 edges (2-qubit connections) out of a theoretical maximum of 127×126/2 = 8001 possible pairs. This means most qubit pairs are NOT directly connected — a two-qubit gate between non-adjacent qubits requires SWAP routing.</p>
<p>Native gate sets by processor generation:</p>
<p>IBM Eagle (127q): {CX, RZ, SX, X, ID, Reset, Measure} — CX is the native 2-qubit gate</p>
<p>IBM Heron (133q): {ECR, RZ, SX, X, ID, Reset, Measure} — ECR (Echoed Cross-Resonance) replaces CX</p>
<p>ECR vs CX: ECR has lower leakage errors but different gate decompositions; transpiler handles conversion automatically</p>
<p>Any gate (H, T, CNOT, CCX, etc.) not in the native set is automatically decomposed by the transpiler. For example: H = RZ(π/2) · SX · RZ(π/2); CNOT = CX (already native on Eagle); Toffoli (CCX) = 6 CX + 7 single-qubit gates (depth ≈ 14 native layers).</p>
</div>

### 9.3.3 Interpreting Calibration Data for Algorithm Design

Calibration data is not just interesting metadata — it directly informs algorithm design choices. The following checklist should be applied before any hardware submission:

- Check T2 vs circuit time: Estimate the total circuit execution time as (number of layers) × (gate time). For IBM Eagle, single-qubit gates ≈ 50 ns, CX ≈ 300 ns. A 20-layer circuit with mostly CX gates takes ≈ 20 × 300 ns = 6 μs. With T2 ≈ 150 μs, the coherence decay factor is exp(−6/150) ≈ 0.96 — 4% coherence loss from T1/T2 alone.

- Select best qubit subset: For algorithms using n ≤ 20 qubits, use calibration data to select the n qubits with lowest CX errors and highest T1/T2 on well-connected regions of the coupling map. The transpiler's SABRE layout pass can do this automatically.

- Avoid bad qubit pairs: If one CX edge has 1.5% error while another has 0.2%, routing algorithms that use the 1.5% edge produce 7× more error per gate. Explicitly constraining the qubit layout or using initial\_layout to route around bad edges can significantly improve results.

- Estimate total error: For a circuit with n\_1q single-qubit gates (error p\_1q each) and n\_2q two-qubit gates (error p\_2q each): F\_circuit ≈ (1-p\_1q)^n\_1q × (1-p\_2q)^n\_2q. Use the mean error rates from backend.properties() for a quick estimate.

<div class="box box-example">
<p class="box-title"><strong>Example 9.2: Pre-flight Circuit Fidelity Estimate</strong></p>
<p>Problem: A VQE ansatz for a 4-qubit molecule has 6 single-qubit RY gates (error 0.03% each) and 4 CX gates (error 0.35% each). The circuit uses 2 measurement qubits with readout error 1.8% each. Estimate total circuit fidelity.</p>
<p><strong>Solution:</strong></p>
<p>Gate fidelity contribution:</p>
<p>F_1q = (1−0.0003)^6 = 0.9997^6 ≈ 0.9982</p>
<p>F_2q = (1−0.0035)^4 = 0.9965^4 ≈ 0.9860</p>
<p>F_gate = F_1q × F_2q ≈ 0.9982 × 0.9860 ≈ 0.9842</p>
<p>Readout error contribution (2 qubits measured):</p>
<p>F_ro = (1−0.018)^2 = 0.982^2 ≈ 0.9643</p>
<p>Total estimated fidelity: F_total ≈ 0.9842 × 0.9643 ≈ 0.9493</p>
<p>Expected success rate: ~95%. This is good — readout errors are the dominant contribution (3.6% vs 1.6% gate errors). Using readout error mitigation (calibration matrix inversion) would recover most of the readout error, improving to ~98%.</p>
</div>

## 9.4 Transpilation: From Logical to ISA Circuit

Transpilation is the process of transforming a logical quantum circuit — written in terms of abstract gates like H, CNOT, T, and Toffoli on arbitrary qubit pairs — into an ISA (Instruction Set Architecture) circuit that uses only the native gates of the target hardware, with all two-qubit operations on directly connected qubit pairs. Without transpilation, a logical circuit cannot run on real hardware.

Transpilation in Qiskit involves four major passes: Layout (mapping logical to physical qubits), Routing (inserting SWAP gates), Translation (decomposing to native basis gates), and Optimisation (cancelling redundant gates). The overall quality of transpilation significantly affects circuit performance: poor transpilation adds unnecessary SWAP gates, increasing circuit depth and error rates.

### 9.4.1 What Is Transpilation and Why Is It Necessary?

Consider a simple CNOT gate between qubits 0 and 63 in a logical circuit. On IBM Eagle (127 qubits), qubit 0 and qubit 63 are separated by approximately 9 hops in the heavy-hex coupling graph. A direct CNOT is not possible — instead, the transpiler must insert 9 SWAP gates to "move" the quantum state of qubit 0 closer to qubit 63 before applying the CNOT. Each SWAP gate costs 3 CX gates, so this single logical CNOT becomes 9×3 + 1 = 28 CX gates after transpilation. The circuit depth increases from 1 to approximately 28 layers.

This overhead is inherent to hardware with limited connectivity and is one of the key reasons why algorithms designed for all-to-all connectivity (like some formulations of Shor's algorithm) are much harder to run on superconducting hardware than on trapped-ion systems with all-to-all coupling.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: ISA Circuit: Definition and Requirements</strong></p>
<p>An ISA (Instruction Set Architecture) circuit is a quantum circuit that satisfies all hardware constraints:</p>
<p>Only native basis gates: {CX, RZ, SX, X, Measure, Reset} for Eagle; {ECR, RZ, SX, X, Measure, Reset} for Heron</p>
<p>Only connected qubit pairs: every two-qubit gate operates on a directly coupled pair in the coupling map</p>
<p>Physical qubit indices: all qubit indices are physical hardware qubit numbers (0 to N-1)</p>
<p>Measurement at end: all measurements occur after all gates (for standard circuits — mid-circuit measurement requires specific backend support)</p>
<p>An ISA circuit is created by transpile() or generate_preset_pass_manager().run(). Attempting to run a non-ISA circuit on a backend will raise an error. The ISA circuit is what is actually executed on the quantum hardware — it is the "machine code" of quantum computing.</p>
</div>

### 9.4.2 generate\_preset\_pass\_manager() and optimization\_level

The recommended transpilation workflow uses generate\_preset\_pass\_manager() from qiskit.transpiler.preset\_passmanagers to create a PassManager configured for a specific backend and optimisation level, then call pm.run(circuit) to perform the transpilation.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.6 — Transpilation with generate_preset_pass_manager()
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService

service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.least_busy(n_qubits=5, simulator=False, operational=True)
print(f'Using backend: {backend.name}')

# ── Build logical circuit ──────────────────────────────────────────────────
# 4-qubit GHZ state: logical (any qubits, any connectivity)
qc = QuantumCircuit(4)
qc.h(0)
qc.cx(0, 1)
qc.cx(0, 2)
qc.cx(0, 3)
qc.measure_all()                    # adds ClassicalRegister automatically

print(f'Logical circuit: depth={qc.depth()}, gates={qc.count_ops()}')

# ── Create PassManager for backend ───────────────────────────────────────
# optimization_level: 0=no opt, 1=light, 2=medium, 3=heavy (recommended)
pm = generate_preset_pass_manager(
    optimization_level=3,
    backend=backend,
)

# ── Run transpilation ─────────────────────────────────────────────────────
isa_circuit = pm.run(qc)

# ── Inspect the ISA circuit ───────────────────────────────────────────────
print(f'\nISA circuit stats:')
print(f'  Depth:          {isa_circuit.depth()}')
print(f'  Total gates:    {sum(isa_circuit.count_ops().values())}')
print(f'  Gate breakdown: {isa_circuit.count_ops()}')
print(f'  Physical qubits used: {isa_circuit.num_qubits}')

# Layout: which physical qubits were assigned to logical qubits 0,1,2,3
layout = isa_circuit.layout.final_layout
print(f'  Layout (logical→physical): {layout}')

# ── Draw the ISA circuit (optional — useful for debugging) ────────────────
# isa_circuit.draw("mpl", fold=80)   # Matplotlib figure
# isa_circuit.draw("text")            # ASCII text
```

The four optimisation levels offer different trade-offs between transpilation time and circuit quality:

| Level | Name | Key Passes Applied | When to Use |
|---|---|---|---|
| 0 | No optimisation | TrivialLayout, BasicSwap, BasisTranslator | Debugging only — fastest, worst quality |
| 1 | Light | TrivialLayout, SabreSwap, InverseCancellation | Quick tests, prototype circuits |
| 2 | Medium | SabreLayout, SabreSwap, CommutativeCancellation | Balanced — good default for coursework |
| 3 | Heavy (Recommended) | SabreLayout, SabreSwap, Clifford resynthesis, 1Q consolidation | Production — best circuit quality, use for research |

### 9.4.3 Understanding the ISA Circuit

After transpilation, the ISA circuit looks very different from the logical input. Understanding the changes is important for debugging and for estimating execution performance. The key transformations are:

- Gate decomposition: H → RZ(π/2) · SX · RZ(π/2). T → RZ(π/4). Toffoli (CCX) → 6 CX + single-qubit gates. All multi-qubit gates are decomposed to the basis set.

- SWAP insertion: For each qubit pair that is not directly connected, SWAP gates are inserted. A SWAP decomposes to 3 CX gates. On a 127-qubit heavy-hex device, a circuit with long-range interactions can have significantly more CX gates after transpilation than the logical circuit.

- Layout assignment: Logical qubit 0 may be mapped to physical qubit 47, logical qubit 1 to physical qubit 48, etc. The SABRE layout algorithm chooses the initial mapping to minimise total SWAP cost for the full circuit.

- Gate cancellation: Optimisation level 3 cancels pairs of inverse gates (X·X = I, RZ(θ)·RZ(−θ) = I) and consolidates sequences of single-qubit gates into a single U3 gate, reducing depth.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.7 — Inspecting and verifying the ISA circuit
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService

service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.backend('ibm_sherbrooke')

# Build Bell state circuit
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

pm = generate_preset_pass_manager(optimization_level=3, backend=backend)
isa_circuit = pm.run(qc)

# ── Verify all gates are in basis set ────────────────────────────────────
basis_gates = set(backend.basis_gates)
circuit_gates = set(isa_circuit.count_ops().keys())
non_basis = circuit_gates - basis_gates
if non_basis:
    print(f'WARNING: Non-basis gates found: {non_basis}')  # should be empty
else:
    print('All gates are in native basis set ✓')

# ── Verify all 2Q gates are on connected pairs ────────────────────────────
from qiskit.transpiler import CouplingMap
cm = CouplingMap(backend.coupling_map)
all_connected = True
for instruction, qargs, _ in isa_circuit.data:
    if len(qargs) == 2:
        q0 = isa_circuit.find_bit(qargs[0]).index
        q1 = isa_circuit.find_bit(qargs[1]).index
        if not cm.distance(q0, q1) == 1:
            all_connected = False
            print(f'WARNING: Q{q0}-Q{q1} not directly connected!')
if all_connected:
    print('All 2Q gates on directly connected qubits ✓')

# ── Compare logical vs ISA ────────────────────────────────────────────────
print(f'\nLogical circuit: depth={qc.depth()}, CX gates={qc.count_ops().get("cx",0)}')
print(f'ISA circuit:     depth={isa_circuit.depth()}, native 2Q gates={sum(isa_circuit.count_ops().get(g,0) for g in ["cx","ecr"])}')
print(f'Physical qubits used: {[isa_circuit.layout.final_layout.get_physical_bits()]}')
```

### 9.4.4 Debugging Transpilation Issues

Transpilation can occasionally produce unexpected results. The most common issues are: (1) Unexpectedly deep circuits due to poor layout selection — try different random\_seed values in generate\_preset\_pass\_manager() to get different SABRE solutions. (2) Missing measurements — always call qc.measure\_all() or add individual measurements before transpilation. (3) Version incompatibility — the ISA circuit format changed between Qiskit 0.x and 1.x; ensure you use generate\_preset\_pass\_manager() rather than the legacy transpile() function for Qiskit 1.x hardware submission.

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: Use generate_preset_pass_manager(), Not transpile(), for Hardware</strong></p>
<p>In Qiskit 1.x, the recommended way to create ISA circuits for hardware is generate_preset_pass_manager(optimization_level=3, backend=backend).run(circuit). The legacy transpile(circuit, backend=backend) function still works but is deprecated for hardware use and may produce incorrect layouts in some edge cases. Always verify the ISA circuit with isa_circuit.draw() or count_ops() before submitting to avoid wasted queue time.</p>
</div>

## 9.5 The Sampler Primitive: Executing Circuits on Hardware

With an account connected, a backend selected, calibration data read, and an ISA circuit prepared, we are ready to run circuits on real quantum hardware. Qiskit Runtime provides this through "Primitives" — high-level abstractions that encapsulate the common computational patterns of quantum algorithms. There are two primitives: SamplerV2 (for measurement outcomes and probability distributions) and EstimatorV2 (for expectation values of observables). Chapter 9 focuses on SamplerV2 as it is the most fundamental and widely used.

### 9.5.1 Qiskit Runtime Primitives: Sampler and Estimator

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: SamplerV2 vs EstimatorV2: When to Use Each</strong></p>
<p>SamplerV2 runs circuits and returns shot-by-shot measurement outcomes (bitstrings). Use it when you need:</p>
<p>Probability distributions (histogram of measurement outcomes)</p>
<p>Grover search (measuring the target state probability)</p>
<p>Quantum teleportation, error correction codes (measuring syndrome bits)</p>
<p>Any algorithm whose output is a classical bitstring or probability distribution</p>
<p>EstimatorV2 runs circuits and returns the expectation value ⟨ψ|O|ψ⟩ of a specified Hermitian observable O. Use it when you need:</p>
<p>VQE (expectation value of Hamiltonian H)</p>
<p>QAOA (expectation value of cost Hamiltonian)</p>
<p>Quantum chemistry (energy estimation)</p>
<p>Any algorithm that optimises an objective function expressible as ⟨O⟩</p>
<p>Key difference: SamplerV2 returns BitArray objects containing raw measurement outcomes. EstimatorV2 returns PubResult objects with expectation values and standard errors. For Chapter 9, we use SamplerV2 exclusively; EstimatorV2 will be covered in Chapter 10 with VQE.</p>
</div>

### 9.5.2 SamplerV2: Setup, Run, and Results

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.8 — Complete Bell State Execution with SamplerV2
# Full workflow: build circuit → transpile → run → retrieve results
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler

# ── Step 1: Connect ────────────────────────────────────────────────────────
service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.least_busy(n_qubits=5, simulator=False, operational=True)
print(f'Using: {backend.name}  (queue: {backend.status().pending_jobs} jobs)')

# ── Step 2: Build logical circuit ─────────────────────────────────────────
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()         # adds classical register "meas" with 2 bits

# ── Step 3: Transpile to ISA circuit ─────────────────────────────────────
pm = generate_preset_pass_manager(optimization_level=3, backend=backend)
isa_circuit = pm.run(qc)
print(f'ISA circuit depth: {isa_circuit.depth()}')

# ── Step 4: Create SamplerV2 and run ─────────────────────────────────────
sampler = Sampler(backend)

# sampler.run() takes a list of (circuit, [parameters], shots) tuples (PUBs)
# PUB = Primitive Unified Bloc (Qiskit Runtime 0.20+ terminology)
job = sampler.run([isa_circuit], shots=1024)

print(f'Job submitted! Job ID: {job.job_id()}')
print(f'Job status: {job.status()}')

# ── Step 5: Wait for result (blocks until job completes) ─────────────────
result = job.result()    # This blocks — may take seconds to minutes in queue

# ── Step 6: Parse results ─────────────────────────────────────────────────
# result is a PrimitiveResult containing a list of PubResults
pub_result = result[0]   # first (and only) PUB

# BitArray contains all shot outcomes
bit_array = pub_result.data.meas   # "meas" is the name of the classical register
print(f'\nBitArray shape: {bit_array.shape}')
print(f'Number of shots: {bit_array.num_shots}')
print(f'Number of bits:  {bit_array.num_bits}')

# Convert to counts dictionary (most common usage)
counts = bit_array.get_counts()
print(f'\nMeasurement counts: {counts}')
# Expected (ideal): {'00': ~512, '11': ~512}
# Actual (hardware): some '01' and '10' from noise

# Compute Bell state fidelity estimate
total_shots = sum(counts.values())
correct = counts.get("00", 0) + counts.get("11", 0)
print(f'Bell fidelity estimate: {correct/total_shots:.4f}')

# ── Alternative: get bitstrings directly ─────────────────────────────────
bitstrings = bit_array.get_bitstrings()
print(f'First 10 shots: {bitstrings[:10]}')
# Each element like '00', '11', '01', '10'
```

<figure class="book-figure">
<img src="content/images/image95.png" alt="Figure 9.4: SamplerV2 execution model and job lifecycle. Left: Seven-step workflow from ISA circuit to measurement counts — each step corresponds to a specific Qiskit Runtime API call. The PubResult → BitArray → get_counts() chain is highlighted. Right: Job status state machine showing all possible transitions: INITIALIZING → QUEUED → RUNNING → DONE/ERROR/CANCELLED. The queue position indicator (accessible via backend.status().pending_jobs) helps estimate wait times; typical IBM Quantum jobs wait 1–30 minutes on public backends.">
<figcaption>Figure 9.4: SamplerV2 execution model and job lifecycle. Left: Seven-step workflow from ISA circuit to measurement counts — each step corresponds to a specific Qiskit Runtime API call. The PubResult → BitArray → get_counts() chain is highlighted. Right: Job status state machine showing all possible transitions: INITIALIZING → QUEUED → RUNNING → DONE/ERROR/CANCELLED. The queue position indicator (accessible via backend.status().pending_jobs) helps estimate wait times; typical IBM Quantum jobs wait 1–30 minutes on public backends.</figcaption>
</figure>

### 9.5.3 PubResult and BitArray: Parsing Sampler Output

The SamplerV2 output uses a new data model introduced in Qiskit Runtime 0.20 (the V2 API). Understanding this data model is essential for correctly parsing results, especially for circuits with multiple classical registers or parametrised circuits.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.9 — Parsing SamplerV2 PubResult: advanced usage
# Multiple classical registers, parametrised circuits, statistics
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.circuit import ParameterVector
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler
import numpy as np

service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.least_busy(n_qubits=3, simulator=False, operational=True)
pm = generate_preset_pass_manager(optimization_level=3, backend=backend)

# ── Circuit with two separate classical registers ─────────────────────────
from qiskit.circuit import ClassicalRegister
qc2 = QuantumCircuit(3)
creg_a = ClassicalRegister(2, "alice")   # 2-bit register for qubits 0,1
creg_b = ClassicalRegister(1, "bob")     # 1-bit register for qubit 2
qc2.add_register(creg_a)
qc2.add_register(creg_b)
qc2.h(0); qc2.cx(0,1); qc2.h(2)
qc2.measure([0,1], creg_a)
qc2.measure([2], creg_b)

isa2 = pm.run(qc2)
sampler = Sampler(backend)
job2 = sampler.run([isa2], shots=4096)
result2 = job2.result()

# Access separate registers by name
alice_bits = result2[0].data.alice    # BitArray for "alice" register
bob_bits   = result2[0].data.bob      # BitArray for "bob" register

print('Alice register counts:', alice_bits.get_counts())
print('Bob register counts:  ', bob_bits.get_counts())

# ── Parametrised circuit: run multiple parameter values in one job ─────────
theta = ParameterVector("theta", 1)
qc_p = QuantumCircuit(1)
qc_p.ry(theta[0], 0)
qc_p.measure_all()
isa_p = pm.run(qc_p)

# Submit multiple parameter values as separate PUBs in one job
theta_values = np.linspace(0, np.pi, 10)
pubs = [(isa_p, [[t]]) for t in theta_values]   # list of (circuit, params) PUBs
job_p = sampler.run(pubs, shots=1024)
result_p = job_p.result()

# Extract P(|1>) for each theta value
print('\nRabi-like rotation: P(|1>) vs theta')
for i, theta_val in enumerate(theta_values):
    counts_i = result_p[i].data.meas.get_counts()
    p1 = counts_i.get("1", 0) / 1024
    print(f'  theta={theta_val:.3f}: P(|1>)={p1:.3f}  (theory: {np.sin(theta_val/2)**2:.3f})')
```

### 9.5.4 Sessions and Batches: Managing Multiple Jobs

For algorithms requiring multiple sequential circuit evaluations — like VQE or QAOA optimisers — submitting each circuit as an independent job wastes time: each job goes through the queue independently. IBM Quantum Runtime Sessions allow multiple jobs to be submitted within a single "session", reserving a spot on the quantum computer and executing the jobs with minimal gap between them.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.10 — Sessions for VQE-style repeated circuit execution
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.circuit import ParameterVector
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler, Session
import numpy as np

service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.least_busy(n_qubits=2, simulator=False, operational=True)
pm = generate_preset_pass_manager(optimization_level=3, backend=backend)

# Build parametrised ansatz circuit
theta = ParameterVector("theta", 2)
qc = QuantumCircuit(2)
qc.ry(theta[0], 0)
qc.ry(theta[1], 1)
qc.cx(0, 1)
qc.measure_all()
isa = pm.run(qc)

# ── Using a Session: holds hardware reservation across multiple jobs ───────
# Jobs inside the session execute with high priority and minimal queue gap
with Session(backend=backend) as session:
    sampler = Sampler(session)
    
    results = []
    # Simulate 5 VQE optimiser iterations
    for iteration in range(5):
        params = np.random.uniform(0, 2*np.pi, 2)
        job = sampler.run([(isa, [params])], shots=512)
        result = job.result()
        counts = result[0].data.meas.get_counts()
        results.append(counts)
        print(f'Iteration {iteration+1}: {counts}')

# Session automatically closes after the with block
print('Session complete.')

# ── Batch mode: submit many INDEPENDENT circuits in parallel ─────────────
from qiskit_ibm_runtime import Batch

theta_sweep = np.linspace(0, 2*np.pi, 20)
with Batch(backend=backend) as batch:
    sampler_b = Sampler(batch)
    pubs = [(isa, [[t, 0.0]]) for t in theta_sweep]
    job_batch = sampler_b.run(pubs, shots=1024)

# After batch block closes, all circuits have been submitted
batch_result = job_batch.result()
print(f'Batch: {len(batch_result)} results received')
```

## 9.6 Job Management and Queue Strategies

After submitting a job to IBM Quantum, the circuit enters a shared queue along with jobs from users worldwide. Understanding how to manage jobs, monitor their status, retrieve results reliably, and choose strategies to minimise wait times is an important practical skill that separates efficient quantum computing practitioners from frustrated ones.

### 9.6.1 Job IDs, Status, and Retrieval

Every job submitted to IBM Quantum is assigned a unique job ID — a string identifier that persists even after the Python session ends. This allows you to retrieve job results hours or days later, share job IDs with collaborators, and check job status without keeping the connection open.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.11 — Job management: status, retrieval, and error handling
# ─────────────────────────────────────────────────────────────────────────
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler
from qiskit_ibm_runtime.exceptions import RuntimeJobFailureError
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
import time

service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.least_busy(n_qubits=2, simulator=False, operational=True)

# ── Submit a job ────────────────────────────────────────────────────────
qc = QuantumCircuit(2); qc.h(0); qc.cx(0,1); qc.measure_all()
isa = generate_preset_pass_manager(3, backend).run(qc)
sampler = Sampler(backend)
job = sampler.run([isa], shots=2048)

# ── Capture the job ID immediately ─────────────────────────────────────
jid = job.job_id()
print(f'Job submitted. ID: {jid}')
# Save this string! You can close Python and reconnect later using this ID.

# ── Monitor job status ──────────────────────────────────────────────────
# Possible statuses: INITIALIZING, QUEUED, RUNNING, DONE, ERROR, CANCELLED
while job.status().name not in ['DONE', 'ERROR', 'CANCELLED']:
    status = job.status()
    print(f'Status: {status.name}  |  Queue position: ~{backend.status().pending_jobs}')
    time.sleep(15)    # poll every 15 seconds (avoid hammering the API)

# ── Retrieve result safely ───────────────────────────────────────────────
try:
    result = job.result()
    counts = result[0].data.meas.get_counts()
    print(f'Result: {counts}')
except RuntimeJobFailureError as e:
    print(f'Job failed: {e}')

# ── Retrieve a previously submitted job by ID (different Python session) ─
old_job = service.job('<YOUR_JOB_ID_HERE>')
print(f'Previous job status: {old_job.status().name}')
if old_job.status().name == 'DONE':
    old_result = old_job.result()
    print('Counts from previous job:', old_result[0].data.meas.get_counts())

# ── List recent jobs ────────────────────────────────────────────────────
recent_jobs = service.jobs(limit=10)
print('\nRecent jobs:')
for j in recent_jobs:
    print(f'  {j.job_id()[:20]}...  {j.status().name:12s}  backend={j.backend().name}')

# ── Cancel a queued job ─────────────────────────────────────────────────
# job.cancel()   # cancels job if still QUEUED or RUNNING
```

<figure class="book-figure">
<img src="content/images/image96.png" alt="Figure 9.5: Bell state measurement results across three execution modes. (a) Ideal simulation (AerSimulator, no noise): perfect 50/50 split with Bell fidelity = 1.000. (b) Aer noisy simulation (depolarising 0.3% + readout 1.5%): dominant |00⟩ and |11⟩ with small |01⟩ and |10⟩ error counts, Bell fidelity ≈ 0.960. (c) Real IBM hardware (ibm_brisbane, 1024 shots): larger error fraction reflects real hardware noise including crosstalk, coherent errors, and readout misclassification not captured by simple depolarising models, Bell fidelity ≈ 0.938. The fidelity difference between (b) and (c) reveals model mismatch.">
<figcaption>Figure 9.5: Bell state measurement results across three execution modes. (a) Ideal simulation (AerSimulator, no noise): perfect 50/50 split with Bell fidelity = 1.000. (b) Aer noisy simulation (depolarising 0.3% + readout 1.5%): dominant |00⟩ and |11⟩ with small |01⟩ and |10⟩ error counts, Bell fidelity ≈ 0.960. (c) Real IBM hardware (ibm_brisbane, 1024 shots): larger error fraction reflects real hardware noise including crosstalk, coherent errors, and readout misclassification not captured by simple depolarising models, Bell fidelity ≈ 0.938. The fidelity difference between (b) and (c) reveals model mismatch.</figcaption>
</figure>

### 9.6.2 Queue Management: Timing, Backend Choice, and Priorities

The IBM Quantum public queue operates on a fair-share algorithm across all free-tier users worldwide. Jobs are processed approximately in order of submission, with Premium Network users receiving priority. Queue wait times range from seconds (for simulators or low-traffic periods) to several hours (for popular backends during peak hours UTC).

<figure class="book-figure">
<img src="content/images/image97.png" alt="Figure 9.6: IBM Quantum queue management. Left: Typical queue depth profile over 24 hours UTC for three backends — depth increases during US East Coast morning (UTC 13:00–20:00) and European business hours (UTC 08:00–16:00). For Indian users (IST = UTC+5:30), the quietest period is IST 04:00–10:00 (UTC 22:30–04:30) — submit jobs before sleeping for short queues. Right: Four access strategy options — Direct backend, Session (sequential related jobs), Batch (parallel independent jobs), and IBM Quantum Network (premium reserved access) — with recommended use cases.">
<figcaption>Figure 9.6: IBM Quantum queue management. Left: Typical queue depth profile over 24 hours UTC for three backends — depth increases during US East Coast morning (UTC 13:00–20:00) and European business hours (UTC 08:00–16:00). For Indian users (IST = UTC+5:30), the quietest period is IST 04:00–10:00 (UTC 22:30–04:30) — submit jobs before sleeping for short queues. Right: Four access strategy options — Direct backend, Session (sequential related jobs), Batch (parallel independent jobs), and IBM Quantum Network (premium reserved access) — with recommended use cases.</figcaption>
</figure>

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Queue Management Best Practices</strong></p>
<p>Strategies to minimise queue wait time and cost:</p>
<p>Choose the least busy backend: service.least_busy(n_qubits=required, simulator=False) selects the shortest queue automatically. This alone reduces wait time from 60 minutes to under 10 minutes in most cases.</p>
<p>Submit at low-traffic times (IST): IBM Quantum queue is lightest in the early morning IST (04:00–10:00 IST = 22:30–04:30 UTC) when US and Europe are asleep. Submit complex jobs before bed and retrieve results in the morning.</p>
<p>Use Sessions for iterative algorithms: Without a Session, a 100-iteration VQE submits 100 independent jobs, each queuing separately. With a Session, the 100 jobs execute back-to-back with minimal gaps, reducing total time from hours to minutes.</p>
<p>Validate with Aer first: Always simulate the full workflow (with a realistic noise model from FakeBackendV2) before submitting to real hardware. This catches circuit errors, result parsing mistakes, and parameter bugs without using queue time.</p>
<p>Set realistic shot counts: More shots = more accurate statistics but longer job time. For algorithm development, 1,024 shots is usually sufficient. For final results or publication-quality data, use 8,192–16,384 shots.</p>
<p>Use max_circuits per job: Each job can contain multiple circuits (up to backend.max_circuits, typically 100–300). Batching circuits into a single job reduces scheduling overhead. Use sampler.run(list_of_isa_circuits) to submit multiple circuits together.</p>
</div>

### 9.6.3 Interpreting Hardware Results vs Ideal Simulation

Comparing hardware results against ideal simulation is the essential diagnostic step that reveals what noise is doing to your circuit. The comparison should be systematic and quantitative.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 9.12 — Systematic comparison: hardware vs ideal vs noisy Aer
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler
from qiskit_aer import AerSimulator
from qiskit_aer.noise import NoiseModel
from qiskit_ibm_runtime.fake_provider import FakeSherbrookeV2
from qiskit.visualization import plot_histogram
import numpy as np

# ── Build and transpile the circuit ──────────────────────────────────────
service = QiskitRuntimeService(channel="ibm_quantum")
backend = service.backend('ibm_sherbrooke')

qc_ghz = QuantumCircuit(3)
qc_ghz.h(0)
qc_ghz.cx(0, 1)
qc_ghz.cx(1, 2)
qc_ghz.measure_all()

pm = generate_preset_pass_manager(optimization_level=3, backend=backend)
isa_ghz = pm.run(qc_ghz)

# ── (A) Ideal simulation (AerSimulator, no noise) ──────────────────────────
sim_ideal = AerSimulator()
# Run the ISA circuit on the ideal simulator (same transpiled circuit!)
from qiskit_aer.primitives import SamplerV2 as AerSampler
aer_sampler_ideal = AerSampler()
result_ideal = aer_sampler_ideal.run([isa_ghz], shots=8192).result()
counts_ideal = result_ideal[0].data.meas.get_counts()

# ── (B) Noisy Aer simulation (calibrated noise model from real backend) ─────
fake_backend = FakeSherbrookeV2()
noise_model = NoiseModel.from_backend(fake_backend)
sim_noisy = AerSimulator(noise_model=noise_model)
pm_fake = generate_preset_pass_manager(3, fake_backend)
isa_fake = pm_fake.run(qc_ghz)
result_noisy = aer_sampler_ideal.run([isa_fake], shots=8192).result()
counts_noisy = result_noisy[0].data.meas.get_counts()

# ── (C) Real hardware ─────────────────────────────────────────────────────
sampler_hw = Sampler(backend)
job_hw = sampler_hw.run([isa_ghz], shots=8192)
print(f'Hardware job submitted: {job_hw.job_id()}')
result_hw = job_hw.result()   # wait for result
counts_hw = result_hw[0].data.meas.get_counts()

# ── Quantitative comparison ────────────────────────────────────────────────
def ghz_fidelity(counts, n_shots):
    """GHZ fidelity estimate: P(|000>) + P(|111>)."""
    return (counts.get("000",0) + counts.get("111",0)) / n_shots

def total_variation_distance(p, q):
    """TVD between two count distributions."""
    all_states = set(p) | set(q)
    total = sum(p.values()); total_q = sum(q.values())
    return 0.5 * sum(abs(p.get(s,0)/total - q.get(s,0)/total_q) for s in all_states)

n = 8192
print(f'\nGHZ fidelity estimates:')
print(f'  Ideal:       {ghz_fidelity(counts_ideal, n):.4f}')
print(f'  Aer noisy:   {ghz_fidelity(counts_noisy, n):.4f}')
print(f'  Hardware:    {ghz_fidelity(counts_hw, n):.4f}')
print(f'\nTotal Variation Distance (lower=better):')
print(f'  TVD(Ideal, Aer noisy): {total_variation_distance(counts_ideal, counts_noisy):.4f}')
print(f'  TVD(Ideal, Hardware):  {total_variation_distance(counts_ideal, counts_hw):.4f}')
print(f'  TVD(Aer noisy, HW):   {total_variation_distance(counts_noisy, counts_hw):.4f}')
```

<figure class="book-figure">
<img src="content/images/image98.png" alt="Figure 9.7: Hardware vs ideal comparison across circuit types. Left: 3-qubit GHZ state counts for ideal (blue), Aer noisy simulation (green), and real hardware (red) — hardware shows ~16% degraded fidelity vs ideal, with noisy Aer capturing about 70% of the real noise. Centre: 2-qubit Grover (target |11⟩) with added error-mitigated hardware counts (purple) — readout correction improves hardware fidelity from 83% to 93%. Right: Fidelity comparison across six circuit types showing how fidelity degrades with circuit depth; the gap between Aer noisy simulation and real hardware reveals noise sources (crosstalk, coherent errors) not captured by simple noise models.">
<figcaption>Figure 9.7: Hardware vs ideal comparison across circuit types. Left: 3-qubit GHZ state counts for ideal (blue), Aer noisy simulation (green), and real hardware (red) — hardware shows ~16% degraded fidelity vs ideal, with noisy Aer capturing about 70% of the real noise. Centre: 2-qubit Grover (target |11⟩) with added error-mitigated hardware counts (purple) — readout correction improves hardware fidelity from 83% to 93%. Right: Fidelity comparison across six circuit types showing how fidelity degrades with circuit depth; the gap between Aer noisy simulation and real hardware reveals noise sources (crosstalk, coherent errors) not captured by simple noise models.</figcaption>
</figure>

The comparison between ideal, Aer noisy, and real hardware results reveals three important insights. First, the Aer noise model (built from calibration data) captures 60–80% of the observed hardware noise — the remaining gap is due to coherent errors, crosstalk, and time-dependent noise not captured in daily calibration snapshots. Second, as circuit depth increases, the hardware performance degrades faster than the Aer noisy simulation predicts — this is because real hardware has correlated errors and drift that simple independent noise models miss. Third, readout error mitigation (calibration matrix inversion) reliably recovers 50–80% of the readout error component, improving hardware fidelity by 2–5 percentage points for circuits with modest gate errors.

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 9: Accessing IBM Quantum Hardware

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  What is the IBM Quantum Open Plan? What backends are accessible and what are the usage limits? How does it differ from the Premium Plan?**

**Q2.  What is an API token in IBM Quantum and why must it be kept secret? Where is it stored after QiskitRuntimeService.save\_account() and what security precautions should you take?**

**Q3.  What does QiskitRuntimeService(channel='ibm\_quantum') do internally? What is the 'channel' parameter and why is it relevant?**

**Q4.  What does service.backends (simulator=False, operational=True, min\_num\_qubits=5) return? What filters does each argument apply?**

**Q5.  Explain service.least\_busy(). When would you use it and when would you prefer manual backend selection?**

**Q6.  What information does backend.properties() return? List five specific data fields and explain their practical importance for algorithm design.**

**Q7.  What is a coupling map? How does backend.coupling\_map differ from full connectivity? Give the example of IBM's heavy-hex topology.**

**Q8.  Describe the four transpilation stages. What does optimization\_level control? Which level is recommended for research-quality hardware runs and why?**

**Q9.  What is the difference between a logical circuit and an ISA circuit? Write Qiskit code to transpile a Bell state circuit to ISA and inspect the result.**

**Q10.  What is SamplerV2 and how does it differ from EstimatorV2? When would you use each? Write the code to execute a circuit using SamplerV2 with 4096 shots.**

**Q11.  What is a PubResult in Qiskit Runtime? How do you extract measurement counts from a PubResult? What is a BitArray?**

**Q12.  What is a Session in Qiskit Runtime? When should you use Sessions vs Batches vs direct job submission? What is the latency difference?**

**Q13.  How do you retrieve a previously submitted job by ID? Write the Qiskit code. Why is job ID management important for research workflows?**

**Q14.  Describe three strategies for managing the IBM Quantum queue. When is each strategy most appropriate?**

**Q15.  A Bell state circuit is run on IBM hardware and gives counts {'00':1876, '01':124, '10':98, '11':1898}. (a) Calculate P(00), P(11), P(01), P(10). (b) What is the total variation distance from ideal? (c) What noise processes likely caused the errors?**

**PART B — MODEL ANSWERS**

**Answer 1:**

IBM Quantum Open Plan (free tier, 2024): available to anyone with an IBM ID. Provides access to Eagle (127-qubit) and Heron (133-qubit) processors. Usage limits: monthly computation time limits (≥10 minutes/month for most users). All jobs enter the shared public queue with no priority. Sessions allowed but may time out during long queue waits. Sufficient for coursework, learning, and small research experiments. Premium Plan (institutional): IBM Quantum Network members. Full fleet access including latest-generation Heron R2, reserved queue slots (jobs execute within minutes, not hours), Session priority, dedicated support, and access to beta features. Most IITs with IBM partnership have Network membership.

**Answer 2:**

API token: a 200-character alphanumeric string that authenticates your IBM Quantum account to the cloud service — essentially a password for programmatic access. Must be kept secret because anyone with your token can use your quantum computing allocation, potentially exhausting your monthly limits. Storage: QiskitRuntimeService.save\_account() writes to ~/.qiskit/qiskit-ibm.json as a plaintext JSON file on your local filesystem. Security precautions: (1) chmod 600 ~/.qiskit/qiskit-ibm.json (owner read-only on Linux/Mac). (2) Never hardcode the token string in Python scripts that may be version-controlled. (3) Never commit ~/.qiskit/ to GitHub. (4) Regenerate the token if compromised (quantum.ibm.com → Account → API Token → Regenerate).

**Answer 3:**

QiskitRuntimeService(channel='ibm\_quantum'): loads credentials from ~/.qiskit/qiskit-ibm.json, establishes authenticated HTTPS connection to IBM Quantum cloud, fetches the list of available backends, and initialises the Runtime session manager. The 'channel' parameter specifies which IBM access model to use: 'ibm\_quantum' uses IBM Quantum credentials (for Open/Premium plan users, API endpoint quantum.ibm.com); 'ibm\_cloud' uses IBM Cloud credentials (for pay-as-you-go billing, API endpoint us-east.quantum-computing.cloud.ibm.com). Code structure differs only in the channel parameter and credentials format.

**Answer 4:**

Returns a list of Backend objects representing real quantum processors (simulator=False excludes AerSimulator); only currently operational (operational=True excludes processors in maintenance or offline); with at least 5 qubits (min\_num\_qubits=5 excludes smaller test systems). Each Backend object has: .name (string identifier), .num\_qubits (integer), .coupling\_map, .basis\_gates, .status() method (returns pending\_jobs, operational), .properties() method (calibration data). The returned list can be sorted, filtered, and iterated to find the best backend for a specific algorithm.

**Answer 5:**

service.least\_busy(): queries all matching backends (simulator=False, operational=True, specified min\_num\_qubits), retrieves status.pending\_jobs for each, and returns the Backend with the fewest queued jobs. Use when: you want quick results for exploratory/coursework circuits and the specific backend doesn't matter. Prefer manual selection when: (1) your algorithm requires specific hardware topology (e.g., linear connectivity for a specific circuit); (2) you need a specific QV level; (3) you want best calibration data (choose most recently calibrated); (4) your circuit requires specific native gates (ECR vs CX).

**Answer 6:**

backend.properties() fields: (1) T₁ per qubit: props.t1(q) in seconds — determines maximum single-qubit circuit time before relaxation. (2) T₂ per qubit: props.t2(q) in seconds — determines maximum useful coherent circuit time. (3) Gate error per gate: props.gate\_error('cx',[q1,q2]) — lets you identify low-error qubit pairs for two-qubit gates. (4) Readout error per qubit: props.readout\_error(q) — readout fidelity, important for measurement-heavy circuits. (5) Last calibration date: props.last\_update\_date — recent calibration = more reliable error estimates. Practical importance: choose qubits and qubit pairs with lowest errors; estimate circuit fidelity before running.

**Answer 7:**

Coupling map: graph where nodes are physical qubit indices and edges represent qubit pairs that can perform direct two-qubit gates. backend.coupling\_map returns a CouplingMap object; visualise with .draw() or extract edges. IBM heavy-hex topology: qubits in a hexagonal arrangement with each qubit having degree 2 or 3 (2 neighbours for 'linear' qubits, 3 for 'branching' qubits). Chosen because it minimises frequency crowding (fewer neighbours → fewer unwanted couplings) while providing sufficient connectivity. Non-adjacent qubit pairs require SWAP routing: each SWAP costs 3 CNOTs, so circuits on non-adjacent qubits have higher error than local circuits.

**Answer 8:**

Stage 1 — Basis translation: all gates decomposed into hardware native set {U(θ,φ,λ), CX} or {U, ECR} for Heron. Stage 2 — Circuit optimisation: gate cancellation, rotation merging, peephole optimisation reduce gate count. Stage 3 — SWAP routing: SABRE algorithm inserts SWAPs to satisfy coupling map; qubit assignment based on hardware error rates at level 3. Stage 4 — Scheduling: assigns parallel and sequential time slots. optimization\_level 0: stages 1,3 only (fast, minimal quality). Level 1: adds basic optimisation. Level 2: heavy optimisation. Level 3: noise-adaptive routing + maximum optimisation. Recommended for research: level 3 — minimises gate count AND chooses low-error qubits.

**Answer 9:**

Logical circuit: uses abstract gate names (H, CNOT, Toffoli), abstract qubit registers, any gate (including non-native ones). ISA circuit: uses ONLY the hardware's native gate set, physical qubit indices from the coupling map, and two-qubit gates only between adjacent qubits. Code: `from qiskit import QuantumCircuit; from qiskit.transpiler.preset\_passmanagers import generate\_preset\_pass\_manager; qc = QuantumCircuit(2,2); qc.h(0); qc.cx(0,1); qc.measure\_all(); pm = generate\_preset\_pass\_manager(backend=backend, optimization\_level=3); isa = pm.run(qc); print(isa.count\_ops()); isa.draw('text')`. Inspect: count\_ops() should show only native gates; num\_qubits shows physical qubit count.

**Answer 10:**

SamplerV2: executes parametrised circuits and returns MEASUREMENT OUTCOMES — bitstring count distributions. Used when: you need P(bitstring) statistics (Bell state experiments, Grover search, benchmarking). EstimatorV2: executes circuits and computes EXPECTATION VALUES ⟨ψ|O|ψ⟩ for Pauli observables O. Used when: you need energy values (VQE, QAOA). Sampler code: `with Session(backend=backend) as session: sampler = SamplerV2(mode=session); job = sampler.run([isa\_circuit], shots=4096); result = job.result(); pub\_result = result[0]; counts = pub\_result.data.c.get\_counts(); print(counts)`.

**Answer 11:**

PubResult (Primitive Unified Bloc Result): the result object for one circuit execution submitted as a 'Pub'. result[i] returns the PubResult for the i-th submitted circuit. Data extraction: `pub\_result.data` is a DataBin object; `pub\_result.data.c` accesses the classical register 'c'; `.get\_counts()` returns a dict {bitstring: count}; `.get\_bitstrings()` returns all individual bitstring outcomes; `.array` returns raw numpy array of shape (shots, num\_bits). BitArray: an efficient numpy-backed array storing all measurement outcomes, supporting slice indexing and multiple shots.

**Answer 12:**

Session: opens a dedicated backend reservation; all jobs in the session execute with minimal inter-job latency (~1ms vs ~30s public queue). Use for: iterative algorithms (VQE optimisation, QAOA) requiring many sequential circuit evaluations on the same backend. Session context manager automatically closes when the `with` block exits. Batch: `SamplerV2(mode='batch')` submits multiple independent circuits without continuous reservation; lower priority than Session but cheaper. Use for: non-iterative circuits that don't depend on each other's results. Direct submission (no Session/Batch): each job joins public queue independently; highest latency but no ongoing reservation.

**Answer 13:**

Code: `service = QiskitRuntimeService(channel='ibm\_quantum'); job = service.job('YOUR\_JOB\_ID'); print(job.status()); result = job.result() if job.status()=='DONE' else None`. Job ID management is important because: (1) IBM Quantum stores job results for 90 days — you can retrieve them without resubmitting. (2) Lab/project workflows: record job IDs in a notebook or database to reproduce or compare results later. (3) Collaboration: share job IDs with colleagues to review the same hardware results. (4) Debugging: reanalyse failed jobs to diagnose errors (job.error\_message()). Best practice: log all submitted job IDs with circuit description, backend, and timestamp.

**Answer 14:**

Strategy 1 — Use least\_busy(): automatically selects the backend with fewest pending jobs. Best for: exploratory runs where circuit is backend-agnostic, minimises waiting time. Strategy 2 — Submit multiple circuits per job: `sampler.run([circ1, circ2, circ3], shots=1000)` — all three circuits execute in one job submission, sharing one queue wait. Best for: circuits that don't depend on each other's results (Grover for multiple targets, calibration sets). Strategy 3 — Use Sessions for iterative algorithms: Session maintains reservation, reducing inter-job latency from ~30s to ~1ms. Best for: VQE with many gradient evaluations, QAOA optimisation loops, any algorithm requiring sequential dependent circuits.

**Answer 15:**

(a) Total shots = 1876+124+98+1898 = 3996. P(00) = 1876/3996 = 0.469. P(11) = 1898/3996 = 0.475. P(01) = 124/3996 = 0.031. P(10) = 98/3996 = 0.025. (b) Ideal: P(00)=P(11)=0.5, P(01)=P(10)=0. TVD = (1/2)Σ|P\_hw(x)−P\_ideal(x)| = (1/2)(|0.469−0.5|+|0.475−0.5|+|0.031−0|+|0.025−0|) = (1/2)(0.031+0.025+0.031+0.025) = 0.056 = 5.6%. (c) Noise causes: P(01) and P(10) arise from: (i) CNOT gate error (flip control or target with probability ~error rate per gate); (ii) Readout error (|00⟩ measured as |01⟩ or |10⟩); (iii) T₁ decay (|11⟩ → |10⟩ before measurement); (iv) Crosstalk.

## EXERCISES — CHAPTER 9

### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.1</strong></p>
<p>Problem: Explain the difference between channel="ibm_quantum" and channel="ibm_cloud" in QiskitRuntimeService. When would you use each? What does save_account() actually store on your machine?</p>
<p><strong>Solution:</strong></p>
<p>channel="ibm_quantum" uses IBM Quantum credentials (IBM ID + API token from quantum.ibm.com). It provides access to the IBM Quantum Open Plan (free tier) and IBM Quantum Network (institutional premium). This is the standard choice for academic and research use.</p>
<p>channel="ibm_cloud" uses IBM Cloud credentials (API key + CRN instance string from cloud.ibm.com). It provides pay-as-you-go access to IBM Quantum hardware through IBM Cloud billing, and is appropriate for production workloads or organisations already using IBM Cloud infrastructure.</p>
<p>save_account() creates the file ~/.qiskit/qiskit-ibm.json (on Linux/macOS) or %USERPROFILE%\.qiskit\qiskit-ibm.json (Windows). This JSON file stores the channel, token, and optional instance string in plaintext. The file is readable only by the current user (chmod 600 on Linux). After save_account(), subsequent calls to QiskitRuntimeService(channel="ibm_quantum") read from this file automatically — you never pass the token explicitly in code again.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.2</strong></p>
<p>Problem: You have access to three backends: A (queue=50 jobs, 127q, QV=128), B (queue=5 jobs, 127q, QV=256), C (queue=3 jobs, 27q, QV=64). Your circuit uses 15 qubits and has 40 CX gates. Which backend should you choose and why?</p>
<p><strong>Solution:</strong></p>
<p>Eliminate C immediately: 27 qubits &lt; 15 qubits required. C cannot run the circuit.</p>
<p>Compare A vs B: Both have 127 qubits (sufficient). Queue: B (5 jobs) &lt;&lt; A (50 jobs) — B is ~10× shorter wait. Quality: B has QV=256 = 2^8 vs A's QV=128 = 2^7 — B achieves one additional qubit/layer of reliable computation.</p>
<p>For a 40-CX-gate circuit: Circuit fidelity ≈ (1−p_CX)^40. For QV=128 backend (p_CX ≈ 0.4%): F ≈ 0.996^40 ≈ 0.852. For QV=256 backend (p_CX ≈ 0.3%): F ≈ 0.997^40 ≈ 0.887. The higher-QV backend B gives ~4% better fidelity AND shorter queue.</p>
<p>Recommendation: Backend B — shorter queue AND higher quality. Use service.least_busy(n_qubits=15) which would select B automatically.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.3</strong></p>
<p>Problem: A 3-qubit GHZ circuit is transpiled for ibm_sherbrooke (Eagle, heavy-hex). The logical circuit has: 1 H gate + 2 CX gates + 3 measurements = depth 3. After optimization_level=3 transpilation, what is the expected approximate depth, and what are the main sources of overhead? Give the actual ISA gate breakdown.</p>
<p><strong>Solution:</strong></p>
<p>Logical circuit: H(q0), CX(q0,q1), CX(q1,q2), measure all. Logical depth = 3, CX count = 2.</p>
<p>After transpilation to Eagle heavy-hex (basis: {CX, RZ, SX, X, Measure}):</p>
<p>(1) H gate decomposition: H = RZ(π/2) · SX · RZ(π/2) → 3 native gates (depth +3 vs depth +1)</p>
<p>(2) CX gate: CX is native → no decomposition overhead</p>
<p>(3) Qubit layout: if qubits 0,1,2 in the logical circuit map to physically adjacent qubits (e.g., Q0-Q1-Q2 on a heavy-hex chain), no SWAP gates are needed. The transpiler's SABRE layout algorithm finds this adjacent mapping.</p>
<p>(4) Gate cancellation: the transpiler may cancel adjacent RZ gates.</p>
<p>Expected ISA circuit: RZ·SX·RZ (H decomp) + CX + CX + 3×Measure ≈ 8 gates, depth ≈ 7-8.</p>
<p>Typical output: depth=8, gates={rz:3, sx:1, cx:2, measure:3} ≈ 9 gates total.</p>
<p>Note: optimization_level=3 applies 1Q gate consolidation, often reducing the H decomposition to a single U3 equivalent, giving depth=5-6 in practice.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.4</strong></p>
<p>Problem: Write complete Qiskit code to: (a) connect to IBM Quantum, (b) find and print the backend with the lowest CX gate error rate among 127-qubit backends, (c) print its best and worst qubit pair CX errors.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.5</strong></p>
<p>Problem: Explain the difference between a Qiskit Runtime Session and a Batch. Give an example algorithm where you would use each. What are the practical limitations of each approach?</p>
<p><strong>Solution:</strong></p>
<p>Session: A Session reserves a spot on the quantum computer for sequential execution of related jobs. Jobs submitted within a Session execute with low latency between them (typically &lt;10 seconds between consecutive jobs). The Session holds the reservation as long as it is active (up to a maximum time set by the backend, typically 1 hour for Open Plan).</p>
<p>Use Session for: VQE/QAOA optimisation loops where each iteration depends on the result of the previous one (e.g., COBYLA optimiser calling energy evaluation 100 times). Without a Session, each iteration re-queues from scratch — 100 iterations × 30 min/job = 50 hours. With a Session: 100 iterations × 30 seconds = 50 minutes.</p>
<p>Batch: A Batch submits multiple INDEPENDENT circuits in parallel — all circuits are queued together and executed as a batch, with results collected at the end. The Batch does NOT guarantee sequential execution or low inter-job latency.</p>
<p>Use Batch for: Parameter sweeps, comparing multiple circuit variants, training data generation. Example: sampling 50 different QAOA parameter combinations to build an initial optimiser landscape.</p>
<p>Limitations: Sessions count against your monthly quota even during idle time. On Open Plan, Session maximum is 10 minutes of active time. Batch jobs may be reordered by the scheduler. Neither Session nor Batch is available for the ibmq_qasm_simulator — only for real hardware backends.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.6</strong></p>
<p>Problem: A Bell state circuit run on ibm_brisbane with 4096 shots gives counts: {"00": 1921, "11": 1954, "01": 87, "10": 134}. (a) Compute the Bell fidelity estimate. (b) Compute the Total Variation Distance (TVD) from the ideal distribution. (c) Estimate the dominant error source.</p>
<p><strong>Solution:</strong></p>
<p>Total shots N = 4096. Error outcomes: n_err = 87+134 = 221.</p>
<p>(a) Bell fidelity = (n_00 + n_11)/N = (1921+1954)/4096 = 3875/4096 ≈ 0.946</p>
<p>(b) Ideal distribution: P_ideal(00)=0.5, P_ideal(11)=0.5, P_ideal(01)=P_ideal(10)=0.</p>
<p>Measured: p(00)=1921/4096≈0.469, p(11)=1954/4096≈0.477, p(01)=87/4096≈0.021, p(10)=134/4096≈0.033.</p>
<p>TVD = 0.5×(|0.469−0.5|+|0.477−0.5|+|0.021−0|+|0.033−0|)</p>
<p>= 0.5×(0.031+0.023+0.021+0.033) = 0.5×0.108 = 0.054</p>
<p>(c) Error analysis: |01⟩ + |10⟩ errors total 221/4096 = 5.4%. P(01) ≈ 2.1%, P(10) ≈ 3.3% — asymmetry (more 10 than 01) suggests a bit-flip error on qubit 1 (|1⟩ misread as |0⟩) is more common than on qubit 0. This is consistent with asymmetric readout error: readout error for |1⟩→|0⟩ is typically higher than |0⟩→|1⟩ on IBM hardware (typically 2-3% vs 0.5-1%). Dominant error: SPAM (state preparation and measurement).</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.7</strong></p>
<p>Problem: You submit 5 independent 2-qubit Bell state circuits as separate jobs (no Session). Each circuit takes 1 second to execute on hardware; queue wait time averages 25 minutes per job. How much total time is saved by using a Session?</p>
<p><strong>Solution:</strong></p>
<p>Without Session: Each of the 5 jobs re-enters the queue independently.</p>
<p>Total time = 5 × (queue wait + execution) = 5 × (25 min + 1 s) ≈ 5 × 25 min ≈ 125 minutes.</p>
<p>With Session: The first job waits in the queue once (25 minutes). Subsequent jobs execute back-to-back with inter-job latency ≈ 5–15 seconds (IBM's typical session latency).</p>
<p>Total time = 25 min + 4 × 15 s + 5 × 1 s = 25 min + 60 s + 5 s ≈ 27 minutes.</p>
<p>Time saved: 125 − 27 = 98 minutes ≈ 1.6 hours.</p>
<p>For n=100 iterations (a typical VQE run): Without Session: 100×25 = 2,500 minutes = 41.7 hours. With Session: 25 min + 99×15 s ≈ 49.8 minutes. Savings: 2,450 minutes ≈ 40.8 hours.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 9.8</strong></p>
<p>Problem: After running a 4-qubit GHZ circuit (target: 50% |0000⟩, 50% |1111⟩) on hardware with 8192 shots, you get: {"0000":3712, "1111":3618, and 24 other bitstrings with total count 862}. (a) Compute GHZ fidelity. (b) Estimate the single-qubit depolarising error rate p that would predict this fidelity for a 3-CX GHZ circuit. (c) Is this consistent with typical IBM hardware specs?</p>
<p><strong>Solution:</strong></p>
<p>(a) GHZ fidelity = (3712+3618)/8192 = 7330/8192 = 0.8948 ≈ 89.5%</p>
<p>(b) The 4-qubit GHZ circuit has 3 CX gates (H+CX+CX+CX). Error model: F ≈ (1-p_cx)^3.</p>
<p>0.8948 = (1-p_cx)^3 → p_cx = 1 − 0.8948^(1/3) = 1 − 0.9636 = 0.0364 = 3.64%</p>
<p>(c) Typical IBM Eagle CX error: 0.2–1.0% per gate. The estimated 3.64% is much higher than the median, suggesting: (i) the circuit used non-optimal qubit pairs with higher CX errors, (ii) SWAP routing added extra CX gates beyond the nominal 3, or (iii) readout errors contribute significantly (4 qubits measured, ~1.5% each ≈ (0.985)^4 ≈ 0.942 readout contribution).</p>
<p>Most likely: transpilation added SWAP gates (a 4-qubit GHZ on a linear chain topology needs only 3 CX, but a heavy-hex routing may add 1-2 SWAPs = 3-6 extra CX), bringing effective CX count to 6-9, giving F ≈ 0.99^6 to 0.99^9 ≈ 0.94 to 0.91. Combined with readout: 0.91 × 0.94 ≈ 0.855 — consistent with observed 0.895.</p>
</div>

### B. Unsolved Problems

Solve independently. Bracketed answers for self-checking.

1. Using QiskitRuntimeService, write code to find the backend with the highest QV among available real backends. Print its name, QV, and mean CX error. [Answer: Iterate service.backends(simulator=False), get b.properties() for each, compute mean CX error, sort by QV (from backend name/generation). ibm\_torino (Heron R2) with QV=4096 should rank highest on Open Plan.]

2. A quantum circuit has 8 logical qubits that need to run on a 127-qubit Eagle backend. The circuit has 15 CX gates between qubits that are, on average, 3 hops apart in the coupling map. Estimate the total number of CX gates after transpilation (accounting for SWAP routing) and the expected circuit fidelity at 0.4% per CX error. [Answer: Extra CX from SWAPs: 15 gates × (3-1) hops × 3 CX/SWAP = 90 additional CX gates. Total ≈ 105 CX gates. F ≈ (0.996)^105 ≈ 0.654 — approaching the noise floor]

3. Write Qiskit code to run the same 3-qubit GHZ circuit 5 times in a single Session with shots=1024 each. Collect all 5 results and compute the mean and standard deviation of the GHZ fidelity across runs. [Answer: Code exercise. Expected: mean fidelity ~85-92%, std ~1-3% reflecting shot noise and calibration drift between consecutive runs]

4. The backend.properties() returns T1 values as floats in seconds. Write a function best\_n\_qubits(backend, n) that returns the indices of the n qubits with the longest T1 values, along with their T1 values in μs. [Answer: props = backend.properties(); t1s = {q: props.t1(q)\*1e6 for q in range(backend.num\_qubits)}; return sorted(t1s, key=t1s.get, reverse=True)[:n]]

5. Compare the ISA circuit depth and gate count for a 5-qubit Quantum Fourier Transform (QFT) circuit transpiled at optimization\_level=1 vs optimization\_level=3 on a 127-qubit Eagle backend. Calculate the ratio of expected fidelities. [Answer: Level 1: depth ~30, CX gates ~25. Level 3: depth ~20, CX gates ~15 (after cancellations). Fidelity ratio: (0.996)^15 / (0.996)^25 ≈ 0.942/0.905 ≈ 1.04 — level 3 gives ~4% better fidelity for this circuit]

6. Explain why job.result() blocks the Python interpreter until the job is complete. Write alternative non-blocking code that allows you to submit 3 jobs simultaneously and retrieve results as each finishes. [Answer: job.result() calls an internal polling loop. Non-blocking: submit all 3 jobs first: j1,j2,j3 = sampler.run(c1), sampler.run(c2), sampler.run(c3). Then: results = {j.job\_id(): j.result() for j in [j1,j2,j3]} collects results as they arrive]

7. A Backend's coupling map has a maximum qubit-to-qubit distance of 11 hops on a 127-qubit heavy-hex graph. For a fully connected all-pairs algorithm (every logical qubit pair needs a gate), estimate the total number of SWAP gates needed for n=8 logical qubits. [Answer: n(n-1)/2 = 28 logical pairs. Average distance ~5 hops. SWAPs: 28 × (5-1) = 112 SWAPs × 3 CX each = 336 extra CX gates — this is why trapped-ion all-to-all connectivity is advantageous]

8. Write Qiskit code to submit a batch of 10 circuits (different numbers of CX gates: 1,2,...,10) and plot the measured Bell fidelity as a function of CX gate count. Use FakeSherbrookeV2 for simulation. [Answer: Code exercise. Expected result: exponential decay matching F≈(1-p\_CX)^n\_CX with p\_CX ≈ 0.3-0.5%]

9. What is the difference between backend.status().pending\_jobs and the actual time-until-execution? Why might a backend with 50 pending jobs execute your job faster than one with 5 pending jobs? [Answer: pending\_jobs counts jobs in the queue, not their execution time. A backend with 50 short single-qubit circuits (each 1 ms) queued executes faster than one with 5 long variational algorithms (each 10 minutes). Queue depth is a rough proxy; use circuit\_duration data from IBM for accurate estimates. Session priority can also bypass the queue for Network users]

10. Calculate the Total Variation Distance (TVD) between two 2-qubit distributions: ideal Bell state P={00:0.5, 11:0.5, 01:0, 10:0} and hardware result Q={00:0.46, 11:0.45, 01:0.04, 10:0.05}. Is this TVD acceptable for a quantum advantage demonstration? [Answer: TVD = 0.5×(|0.5-0.46|+|0.5-0.45|+|0-0.04|+|0-0.05|) = 0.5×(0.04+0.05+0.04+0.05) = 0.09. For quantum advantage demonstrations, TVD < 0.1 is generally required; this result (0.09) is marginal but acceptable for coursework purposes]

### C. Multiple Choice Questions

Circle the best answer. Answers at end of section.

**Q1. The function QiskitRuntimeService.save\_account() stores credentials in:**

- (a) An IBM Quantum cloud database only — not on your machine

- (b) A local file at ~/.qiskit/qiskit-ibm.json on your machine

- (c) An environment variable IBM\_QUANTUM\_TOKEN in your shell profile

- (d) A Python pickle file in your current working directory

**Q2. service.least\_busy(n\_qubits=5, simulator=False) selects:**

- (a) The backend with the fewest qubits ≥ 5

- (b) The real backend with the shortest queue depth AND ≥ 5 qubits

- (c) The backend with the highest QV and ≥ 5 qubits

- (d) The backend with the lowest CX error rate and ≥ 5 qubits

**Q3. An ISA (Instruction Set Architecture) circuit must satisfy which constraints?**

- (a) Only logical abstract gates; any connectivity

- (b) Native basis gates only; two-qubit gates on directly coupled pairs

- (c) Only single-qubit gates (no two-qubit gates)

- (d) Physical qubit indices 0 to N, but any gate set

**Q4. What is the primary purpose of a Qiskit Runtime Session?**

- (a) To run circuits faster by bypassing error correction

- (b) To store credentials across multiple Python sessions

- (c) To execute a sequence of related jobs with low inter-job latency on hardware

- (d) To parallelize a single circuit across multiple backends simultaneously

**Q5. The SamplerV2 primitive returns results as:**

- (a) A dictionary of {bitstring: count} directly

- (b) A PrimitiveResult containing PubResults with BitArray objects

- (c) A numpy array of complex amplitudes

- (d) A list of integers representing qubit measurement outcomes

**Q6. backend.properties().t1(qubit\_index) returns T1 in units of:**

- (a) Microseconds (μs)

- (b) Nanoseconds (ns)

- (c) Seconds (s)

- (d) Milliseconds (ms)

**Q7. On an IBM Eagle processor, the native two-qubit gate is:**

- (a) CNOT (CX)

- (b) ECR (Echoed Cross-Resonance)

- (c) SWAP

- (d) iSWAP

**Q8. The heavy-hex coupling map was chosen for IBM processors primarily to:**

- (a) Maximise the number of connections between qubits

- (b) Reduce ZZ crosstalk between adjacent qubits

- (c) Enable all-to-all connectivity

- (d) Minimise the total number of qubits required

**Q9. Which optimization\_level should be used for production hardware runs?**

- (a) 0 — fastest, minimal circuit changes

- (b) 1 — light optimisation, good default

- (c) 2 — medium optimisation, balanced

- (d) 3 — heavy optimisation, best circuit quality

**Q10. A SWAP gate on IBM Eagle hardware costs how many CX gates?**

- (a) 1 CX gate

- (b) 2 CX gates

- (c) 3 CX gates

- (d) 4 CX gates

**Q11. In SamplerV2, the shots parameter controls:**

- (a) The number of parallel circuits submitted

- (b) The number of times each circuit is executed and measured

- (c) The circuit optimization level

- (d) The number of qubits measured per circuit execution

**Q12. job.job\_id() returns:**

- (a) An integer from 0 to 999999

- (b) The qubit count of the backend used

- (c) A unique string identifier for the submitted job that persists after session ends

- (d) The position of the job in the current queue

**Q13. The Qiskit Runtime Batch mode is best for:**

- (a) Sequential VQE iterations where each result informs the next circuit

- (b) Submitting independent circuits in parallel for parameter sweeps

- (c) Running the same circuit repeatedly with identical parameters

- (d) Emergency submission bypassing the regular queue

**Q14. On IBM Heron (133-qubit) processors, the native two-qubit gate is:**

- (a) CX (CNOT)

- (b) ECR (Echoed Cross-Resonance)

- (c) CZ

- (d) iSWAP

**Q15. The Total Variation Distance (TVD) between an ideal distribution and a hardware distribution measures:**

- (a) The Euclidean distance between Bloch vectors

- (b) The sum of squared differences between corresponding probabilities

- (c) Half the sum of absolute differences between corresponding probabilities — L1 distance normalised to [0,1]

- (d) The number of incorrect bitstrings divided by total shots

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 9</strong></p>
<p><strong>Q1: (b)   Q2: (b)   Q3: (b)   Q4: (c)   Q5: (b)</strong></p>
<p><strong>Q6: (c)   Q7: (a)   Q8: (b)   Q9: (d)   Q10: (c)</strong></p>
<p><strong>Q11: (b)  Q12: (c)  Q13: (b)  Q14: (b)  Q15: (c)</strong></p>
<p>Q6 Detail: backend.properties().t1(q) returns a float in seconds — multiply by 1e6 to get μs. Q7 Detail: Eagle uses CX; Heron uses ECR. The transpiler automatically converts CX to ECR when targeting Heron. Q10 Detail: SWAP = CX·(I⊗CX)·CX which decomposes to exactly 3 CX gates — a fundamental identity in gate synthesis.</p>
</div>

### D. Theory Questions

- Explain the complete Qiskit Runtime execution model from Python code to hardware result. Describe each layer of the stack: QiskitRuntimeService, Primitives, transpiler, and hardware. What happens between sampler.run() and job.result()?

- What is the heavy-hex coupling map topology used in IBM Eagle processors? How many neighbours does each qubit have? Why was this topology chosen over a rectangular grid? Calculate the maximum distance (in hops) between any two qubits on a 127-qubit heavy-hex device.

- Describe the four passes in the Qiskit transpilation pipeline (Layout, Routing, Translation, Optimisation). For each pass, explain: what problem it solves, which algorithm is used at optimization\_level=3, and what the output looks like.

- What is an ISA (Instruction Set Architecture) circuit? List the three conditions a circuit must satisfy to be an ISA circuit for an IBM Eagle backend. Why does submitting a non-ISA circuit to a backend cause an error?

- Compare the SamplerV2 and EstimatorV2 Qiskit Runtime primitives. What output does each return? Give three examples of quantum algorithms best suited for each primitive.

- Explain the difference between a Qiskit Runtime Session and a Batch. Draw a timeline diagram showing how jobs execute in each mode. For a 100-iteration VQE with 30-minute average queue time and 5-second execution time per iteration, compute the total runtime with and without a Session.

- Explain the chain of objects in the SamplerV2 result: PrimitiveResult → PubResult → DataBin → BitArray. How do you extract a counts dictionary from this chain? What does the BitArray.shape attribute represent?

- Describe three practical strategies a student in India can use to minimise queue wait time when accessing IBM Quantum hardware. Include timing considerations (IST vs UTC), backend selection, and job batching strategies.

- What is Total Variation Distance (TVD)? Give the formula and explain what values of TVD indicate good vs poor agreement between ideal and hardware results. For a 3-qubit GHZ state, what TVD would you expect from a typical IBM Eagle backend?

- Why does the Aer noisy simulation (using FakeBackendV2 noise model) typically underpredict the actual hardware noise by 20–40%? List three noise sources in real hardware that are not captured by calibration-based noise models.

### E. Programming Assignments

PA9.1 — End-to-End Bell State Experiment. Implement a complete quantum experiment using IBM Quantum hardware: (a) Connect to IBM Quantum and use service.least\_busy(n\_qubits=2) to select a backend. Print its name, queue depth, T1/T2 median values (from properties()), and mean CX error. (b) Build a Bell state circuit: H(0) + CX(0,1) + measure\_all(). Transpile at optimization\_level=3. Print ISA circuit statistics (depth, gate breakdown, physical qubits used). (c) Run three experiments: ideal Aer simulation (4096 shots), noisy Aer simulation using FakeBackend noise model (4096 shots), and real hardware (4096 shots). (d) For each, compute: Bell fidelity estimate, TVD from ideal. (e) Plot all three histograms side by side using plot\_histogram. (f) Compute the readout-error-corrected hardware result using calibration matrix inversion and report the improvement. Submit: code, terminal output, histogram figure, and 500-word analysis comparing the three results and explaining observed discrepancies.

PA9.2 — Calibration-Based Qubit Selection. Write a Qiskit programme that systematically characterises a 127-qubit backend and selects the optimal qubit subset for a given algorithm: (a) Read backend.properties() and build a pandas DataFrame with columns: qubit\_index, T1\_us, T2\_us, readout\_error\_pct, sx\_error\_pct. (b) For each edge in the coupling map, add columns: edge (q1,q2), cx\_error\_pct, cx\_gate\_time\_ns. (c) Compute a "qubit quality score" Q = T2\_us / (cx\_error\_pct × readout\_error\_pct) for each qubit. Rank qubits by Q. (d) For a 5-qubit algorithm, select the top 5 qubits by score that form a connected subgraph on the coupling map. (e) Estimate the circuit fidelity for a 10-CX-gate circuit using the best vs worst 5-qubit subsets. Print the comparison. (f) Plot a qubit map with T2 and CX error colour-coded. Submit: code, DataFrame printout, qubit quality ranking, and map figure.

PA9.3 — Session-Based Ramsey Experiment Simulation. Implement a simulated Ramsey spectroscopy experiment using Qiskit Runtime Sessions on a real backend: (a) Build a Ramsey circuit template: Ry(π/2, q0) → delay(τ, q0) → Ry(π/2, q0) → measure. (b) Use 20 values of τ from 0 to 300 μs (in steps of 15 μs), converting each to circuit delays using qc.delay(). (c) Submit all 20 circuits in a single Session (no Batch — submit sequentially, capturing each result before submitting the next to simulate an online experiment). (d) From the 20 P(|1⟩) values, fit a decaying cosine A·cos(2π·Δf·τ)·exp(-τ/T2) + 0.5 to extract T2 and the detuning Δf. (e) Compare extracted T2 with the backend.properties().t2(qubit) value. Plot the Ramsey fringe with fit. Submit: code, extracted T2 and Δf values, Ramsey fringe plot, and comparison table.

### F. Project Suggestions

Project 9.A — Comprehensive Backend Comparison Study. Conduct a systematic comparison of two or three IBM Quantum backends available on your account: (a) For each backend, collect all calibration metrics (T1, T2, gate errors, readout errors) and compute statistics (mean, median, best 10%, worst 10%). (b) Run the same set of benchmark circuits on each backend: Bell state (2q), GHZ-3 (3q), GHZ-5 (5q), 2-qubit Grover (2q), QFT-4 (4q). Use 4096 shots each. (c) For each circuit on each backend, compute: fidelity, TVD, and estimated vs actual execution time including queue wait. (d) Build a comparison table and radar chart of all metrics across backends. (e) Write a 3,000-word report covering: which backend is best for which circuit type, how calibration metrics predict actual fidelity, whether QV is a reliable predictor of algorithm performance, and recommendations for algorithm developers choosing backends.

Project 9.B — Adaptive Backend Selection Tool. Build a Python tool that automates backend selection for a given algorithm: (a) Input: a QuantumCircuit (logical), a performance target (e.g., "circuit fidelity > 0.85"), and a maximum acceptable queue time. (b) For each accessible backend, the tool should: transpile the circuit, estimate post-transpilation fidelity using calibration data, check queue depth and estimate wait time. (c) Output: a ranked list of backends with estimated fidelity and wait time, and a recommendation. (d) Test the tool on 5 different circuits of varying depth. (e) Validate the tool's predictions against actual hardware runs on the top 2 recommended backends. Write a 2,500-word report on the tool's design, accuracy, and limitations.

Project 9.C — Quantum Error Analysis Pipeline. Build a complete pipeline that runs a quantum circuit on hardware and performs systematic error analysis: (a) Choose a 4-qubit circuit of educational interest (Grover, QFT, or VQE ansatz). (b) Run it on hardware at multiple shot counts (512, 1024, 2048, 4096, 8192) and plot fidelity vs shot count. (c) Apply readout error mitigation using calibration matrix inversion. Quantify improvement. (d) Apply Zero-Noise Extrapolation (ZNE) using gate folding (λ=1,2,3). Quantify improvement. (e) Compare: raw hardware, readout-mitigated, ZNE-mitigated, and ideal Aer simulation. (f) Analyse which error sources (readout vs gate vs coherence) dominate for your specific circuit and backend. Write a 3,500-word analysis covering: methodology, results, error budget breakdown, and practical recommendations for other students using IBM Quantum hardware.

## References and Further Reading

1. IBM Quantum Documentation (2024). Qiskit IBM Runtime. https://docs.quantum.ibm.com — The primary reference for all API details, authentication, primitives, and hardware specifications.

2. Qiskit IBM Runtime GitHub (2024). https://github.com/Qiskit/qiskit-ibm-runtime — Source code and latest release notes for SamplerV2, EstimatorV2, Sessions, and Batches.

3. Qiskit Transpiler Documentation (2024). https://docs.quantum.ibm.com/api/qiskit/transpiler — generate\_preset\_pass\_manager(), PassManager internals, optimization levels, layout and routing algorithms.

4. IBM Quantum Blog: "Introducing the Qiskit Primitives" (2022). https://research.ibm.com/blog/qiskit-primitives — Original introduction of Sampler and Estimator V1, with design rationale.

5. Gambetta, J. M. et al. (2012). Protocols for optimal readout of qubits using a continuous quantum nondemolition measurement. Physical Review A, 85, 042305. [Dispersive readout theory underlying IBM measurement circuits]

6. Cross, A. W. et al. (2019). Validating quantum computers using randomized model circuits. Physical Review A, 100, 032328. [Quantum Volume benchmark — context for backend quality metrics]

7. IBM Quantum Learning (2024). https://learning.quantum.ibm.com — Free interactive courses on Qiskit and IBM Quantum hardware; the "Basics of quantum information" and "Fundamentals of quantum algorithms" courses are recommended complements to this chapter.

8. Itoko, T. et al. (2023). Qiskit Runtime Sampler primitive for quantum circuits. IBM Technical Report. [SamplerV2 API design and PubResult structure]

9. National Quantum Mission India (2023). https://dst.gov.in/national-quantum-mission — Official NQM documentation with details of IBM Quantum partnerships with Indian institutions.

10. Qiskit Tutorials — IBM Quantum Experience (2024). https://github.com/Qiskit/qiskit-tutorials — Community tutorials including hardware experiments, noise characterisation, and error mitigation with Qiskit Runtime.
