# CHAPTER 10: Other Quantum Platforms

# Google Cirq · Amazon Braket · Azure Quantum

# OpenQASM · Circuit Models · Multi-Provider Access · Q# · Cross-Platform Comparison · NISQ Outlook

<div class="box box-anecdote">
<p class="box-title"><strong>📜  The Quantum Race: IBM, Google, Amazon, Microsoft — and What It Means for Science</strong></p>
<p>On 23 October 2019, Google published a paper in Nature claiming "quantum supremacy" — the first computational task performed faster by a quantum computer than any classical computer could achieve in a reasonable time. The task was random circuit sampling on a 53-qubit processor named Sycamore. IBM, whose own quantum computer was publicly accessible on the cloud, immediately challenged the claim, arguing that an optimised classical simulation could complete the same task in 2.5 days, not 10,000 years. The scientific debate that followed was fierce and illuminating: it revealed just how hard it is to compare quantum and classical machines, how sensitive "supremacy" claims are to the choice of classical algorithm, and how different the two companies’ approaches to quantum computing were.</p>
<p>Amazon entered the quantum cloud market in 2019 with Amazon Braket — a cloud service that provides access not to Amazon’s own quantum hardware, but to hardware from multiple providers: IonQ (trapped ions), Rigetti (superconducting), Oxford Quantum Circuits, QuEra (neutral atoms), and IQM. Amazon’s strategy was to be the infrastructure layer — the AWS of quantum computing — rather than a hardware manufacturer. Microsoft took a different approach entirely: investing in topological qubits (a technology that has not yet been demonstrated at scale), developing Q#, a dedicated quantum programming language, and building Azure Quantum as a platform connecting Azure Cloud customers to third-party quantum hardware from IonQ, Quantinuum, and Pasqal.</p>
<p>For an M.Sc. Physics student in 2026, this diversity of platforms is both an opportunity and a challenge. IBM Quantum is the most accessible (free tier, excellent documentation, Qiskit is the dominant open-source framework). Google Cirq is important because Google publishes influential quantum computing research and Cirq is a reference implementation for many novel techniques. Amazon Braket matters because AWS is the dominant cloud platform for industry and data science, and quantum workloads may increasingly run alongside classical HPC workloads in AWS. Microsoft Q# represents a different paradigm — a quantum-native language rather than a quantum library embedded in Python — and the long-term bet on topological qubits, if it succeeds, could leapfrog current architectures. This chapter surveys all four, giving sufficient depth for practical use and informed comparison.</p>
</div>

Chapter 10 completes Unit V and the MPY305 course. Having mastered the theory of quantum computation (Units I–III), the physics of noise and hardware (Unit IV), and the practice of IBM Quantum access (Chapter 9), we now survey the broader ecosystem of quantum cloud platforms. We examine Google Cirq in sufficient depth to build and simulate circuits, understand its native gate model, and run XEB benchmarking. We cover Amazon Braket’s multi-provider architecture, its Python SDK, and its cost model. We introduce Microsoft Azure Quantum and its unique Q# quantum programming language. We then compare all four platforms systematically, discuss OpenQASM as the interoperability standard, and close with a frank assessment of where quantum computing stands today and what career opportunities exist for physics graduates.

## 10.1 The Quantum Cloud Ecosystem

The quantum computing landscape in 2025 is characterised by a handful of large cloud providers offering access to quantum hardware from multiple manufacturers, each with its own SDK, native gate set, and pricing model. Understanding the ecosystem — who the players are, what distinguishes them, and how they interoperate — is essential context before diving into the individual platforms.

<figure class="book-figure">
<img src="content/images/image99.png" alt="Figure 10.1: Quantum cloud platform comparison. Left: Radar chart comparing IBM Quantum, Google Cirq, Amazon Braket, and Azure Quantum across six key dimensions — IBM leads on access ease, free tier, and documentation; Google Cirq and Amazon Braket excel in hardware variety; all four have strong Python SDKs. Right: Feature comparison table showing SDK names, programming languages, hardware types, free tier availability, simulator options, and unique features for each platform.">
<figcaption>Figure 10.1: Quantum cloud platform comparison. Left: Radar chart comparing IBM Quantum, Google Cirq, Amazon Braket, and Azure Quantum across six key dimensions — IBM leads on access ease, free tier, and documentation; Google Cirq and Amazon Braket excel in hardware variety; all four have strong Python SDKs. Right: Feature comparison table showing SDK names, programming languages, hardware types, free tier availability, simulator options, and unique features for each platform.</figcaption>
</figure>

### 10.1.1 Why Multiple Platforms Exist: Different Hardware, Different SDKs

Each quantum cloud platform reflects its parent company’s strategic position in the quantum computing market:

- IBM Quantum + Qiskit: IBM has the largest fleet of publicly accessible quantum computers and the largest open-source quantum computing community (Qiskit has >500,000 users). IBM’s strategy is to make quantum computing as accessible as possible, following the model of open-source software development. Qiskit is open-source (Apache 2.0 licence) and the de facto standard for quantum software development.

- Google Quantum AI + Cirq: Google focuses on research-grade quantum computing, publishing landmark results (quantum supremacy 2019, below-threshold error correction 2024). Cirq is designed for researchers who want fine-grained control over circuits, noise models, and benchmarking. Google Sycamore hardware is not publicly accessible (research collaborators only); Cirq’s main value for most users is as a simulator and research tool.

- Amazon Braket: Amazon’s strategy is cloud infrastructure. Braket provides a unified API to access quantum hardware from multiple providers (IonQ, Rigetti, QuEra, Oxford QC, IQM) through the AWS ecosystem. Companies already using AWS can add quantum computation without learning new cloud infrastructure.

- Microsoft Azure Quantum + Q#: Microsoft’s long-term bet is on topological qubits (using non-Abelian anyons), which are predicted to have intrinsically lower error rates. Q# is a purpose-built quantum programming language that treats quantum operations as first-class language constructs, not library calls. Azure Quantum currently provides access to IonQ, Quantinuum, and Pasqal hardware while topological qubits remain in development.

### 10.1.2 OpenQASM as a Common Intermediate Representation

Despite the diversity of SDKs, the underlying quantum circuits from all platforms can be expressed in OpenQASM (Open Quantum Assembly Language) — a low-level, text-based language for quantum circuits. OpenQASM 2.0 (2017) and OpenQASM 3.0 (2022) serve as a common intermediate representation (IR) that enables circuit exchange between different frameworks.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: OpenQASM 2.0: Syntax and Usage</strong></p>
<p>A Bell state circuit in OpenQASM 2.0:</p>
<p>OpenQASM 3.0 adds: classical control flow (if/for/while), real-time pulse-level operations, mid-circuit measurements, and a richer type system. It is the language underlying IBM’s ISA circuits and is supported natively by Qiskit, Cirq (via cirq.to_qasm()), Braket (via braket.circuits.Circuit.to_openqasm()), and Azure QDK.</p>
</div>

The practical value of OpenQASM interoperability is significant: a circuit developed and debugged in Qiskit can be exported to OpenQASM 2.0 with qc.qasm(), then imported into Cirq (cirq.Circuit.from\_qasm\_str()), Braket, or compiled into Q# for Azure execution. This portability is important for research workflows that compare hardware performance across providers, and for students who want to translate textbook examples between platforms.

### 10.1.3 Platform Selection Criteria

The following checklist helps select the right platform for a given task:

| Use Case | Recommended Platform | Reason |
|---|---|---|
| Coursework / learning / free access | IBM Quantum (Open Plan) | Free, best documentation, largest community, Qiskit is the industry standard |
| Research comparing hardware across providers | Amazon Braket | Single API for IonQ, Rigetti, OQC, QuEra, IQM |
| High-fidelity small circuits (trapped ion) | IBM (IonQ via Braket) or Quantinuum | Best 2Q fidelity; all-to-all connectivity |
| Fine-grained circuit control + noise research | Google Cirq | DensityMatrixSimulator, custom noise channels, XEB tools |
| Integration with AWS HPC workloads | Amazon Braket | Native AWS SDK integration, hybrid classical-quantum workflows |
| Microsoft ecosystem / enterprise | Azure Quantum | Q# interop, Azure Active Directory auth, enterprise SLA |
| Neutral atom large-scale simulation | Amazon Braket (QuEra) or IBM | QuEra Aquila on Braket; 256 qubits neutral atom |
| Post-NISQ / fault-tolerant research | All platforms (simulators only) | No fault-tolerant hardware yet; use Aer, Cirq, or Braket sim |

## 10.2 Google Quantum AI and Cirq

Cirq is an open-source Python framework for designing, simulating, and running quantum circuits, developed by Google Quantum AI. While Qiskit is the dominant industry SDK for cloud quantum execution, Cirq occupies an important niche as the framework for Google’s own research — all Google Quantum AI papers since 2018 use Cirq internally. Cirq has a distinct design philosophy: it provides fine-grained, explicit control over every aspect of the circuit, making it the tool of choice for research that requires precise noise modelling, custom gate definitions, and cross-entropy benchmarking.

### 10.2.1 Cirq Architecture: Circuits, Moments, and GridQubits

Cirq’s circuit model is more explicit than Qiskit’s in one important respect: time. In Qiskit, a QuantumCircuit is a sequence of gate operations appended in order; the transpiler determines which operations can run in parallel. In Cirq, a Circuit is an explicit list of Moments, where each Moment represents a set of operations that execute simultaneously in one time step. This explicit moment structure gives the programmer direct control over parallelism.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Cirq Core Objects: Circuit, Moment, Operation, Gate, Qubit</strong></p>
<p>The Cirq object hierarchy has five levels:</p>
<p>cirq.Qubit: an abstract qubit. The most common concrete types are: cirq.GridQubit(row, col) — a qubit addressed by its position on a 2D grid (used for Sycamore and other 2D processors); cirq.LineQubit(index) — a qubit on a 1D line (used for simulation and linear-chain topologies); cirq.NamedQubit(name) — a qubit identified by a string name.</p>
<p>cirq.Gate: an abstract quantum operation. Examples: cirq.H (Hadamard), cirq.X (Pauli X), cirq.CNOT, cirq.CZ, cirq.PhasedXZGate. Gates are objects; to apply a gate to qubits, call gate.on(qubit1, qubit2, ...) which returns an Operation.</p>
<p>cirq.Operation: a gate applied to specific qubits. Example: cirq.H(q[0]) — H gate on qubit q[0]. Operations are the leaf elements of a circuit.</p>
<p>cirq.Moment: a set of Operations that are applied simultaneously in one clock cycle. All operations in a Moment must act on disjoint sets of qubits (no qubit appears twice).</p>
<p>cirq.Circuit: an ordered list of Moments. Can be created by appending operations (Cirq automatically groups compatible operations into Moments) or by explicit Moment construction.</p>
</div>

<figure class="book-figure">
<img src="content/images/image100.png" alt="Figure 10.2: Google Cirq circuit model and benchmarking. Left: Cirq object hierarchy — Circuit contains Moments, which contain GateOperations, applied to GridQubits and using Gates. Right: A 5×5 excerpt of the Sycamore GridQubit layout with nearest-neighbour connections; qubits are addressed as GridQubit(row,col). Far right: Cross-Entropy Benchmarking (XEB) fidelity decay as a function of circuit depth for ideal simulation (flat at 1.0), Sycamore hardware, and a noisy Cirq simulation — XEB is Google’s primary gate fidelity metric.">
<figcaption>Figure 10.2: Google Cirq circuit model and benchmarking. Left: Cirq object hierarchy — Circuit contains Moments, which contain GateOperations, applied to GridQubits and using Gates. Right: A 5×5 excerpt of the Sycamore GridQubit layout with nearest-neighbour connections; qubits are addressed as GridQubit(row,col). Far right: Cross-Entropy Benchmarking (XEB) fidelity decay as a function of circuit depth for ideal simulation (flat at 1.0), Sycamore hardware, and a noisy Cirq simulation — XEB is Google’s primary gate fidelity metric.</figcaption>
</figure>

### 10.2.2 Building and Simulating Circuits in Cirq

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 10.1 — Building Circuits in Cirq: Basic Operations
# pip install cirq     (includes cirq-core, cirq-google, cirq-aqt)
# ─────────────────────────────────────────────────────────────────────────
import cirq
import numpy as np

# ── Qubit types ───────────────────────────────────────────────────────────
# GridQubit: used for Sycamore and 2D hardware topologies
q0, q1 = cirq.GridQubit(0,0), cirq.GridQubit(0,1)

# LineQubit: simpler addressing for 1D chains and simulations
a, b, c = cirq.LineQubit.range(3)

# NamedQubit: string-addressed (useful for readability)
alice, bob = cirq.NamedQubit("alice"), cirq.NamedQubit("bob")

# ── Building a Bell state circuit ──────────────────────────────────────────
bell_circuit = cirq.Circuit()
bell_circuit.append(cirq.H(q0))         # Hadamard on q0
bell_circuit.append(cirq.CNOT(q0, q1))  # CNOT: control=q0, target=q1
bell_circuit.append(cirq.measure(q0, q1, key="result"))  # measure both

print('Bell state circuit:')
print(bell_circuit)
# Output:
# (0, 0): ──H──@──M(‘result’)─
# (0, 1): ─────X──M(‘result’)─

# ── Explicit Moment construction ───────────────────────────────────────────
explicit_circuit = cirq.Circuit([
    cirq.Moment([cirq.H(a), cirq.X(c)]),        # H on a, X on c simultaneously
    cirq.Moment([cirq.CNOT(a, b)]),              # CNOT in second moment
    cirq.Moment([cirq.measure(a, b, key="ab")]), # Measure in third moment
])
print('Explicit circuit moments:', len(explicit_circuit))

# ── Ideal simulation with Simulator ─────────────────────────────────────────
simulator = cirq.Simulator()     # statevector simulator

# Simulate without measurement (get statevector)
qc_nomeas = cirq.Circuit([cirq.H(q0), cirq.CNOT(q0, q1)])
result_sv = simulator.simulate(qc_nomeas)
print('\nStatevector:', result_sv.final_state_vector.round(4))
# Output: [0.707+0j, 0+0j, 0+0j, 0.707+0j]  (Bell state |00>+|11>)/sqrt(2)

# Simulate with measurement (sample shots)
result_shots = simulator.run(bell_circuit, repetitions=1024)
print('Bell state counts:', result_shots.measurements["result"][:5, :], '...')
print('Histogram:', result_shots.histogram(key='result'))
# Output: Counter({0: ~512, 3: ~512})  (00 and 11 in binary encoding)
```

### 10.2.3 Cirq Noise Models and the DensityMatrixSimulator

One of Cirq’s strengths is its flexible, explicit noise model system. Unlike Qiskit Aer (which attaches noise to specific gates through a NoiseModel object), Cirq inserts noise as explicit Channels inserted after gates using a NoiseModel class with insertable\_noise\_channel() methods. This gives finer control over noise and makes Cirq particularly useful for noise research.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 10.2 — Cirq Noise Models and DensityMatrixSimulator
# ─────────────────────────────────────────────────────────────────────────
import cirq
import numpy as np

# ── Define a depolarising noise model ────────────────────────────────────
# ConstantQubitNoiseModel: applies the same noise channel after every gate
p_depo = 0.003    # 0.3% depolarising error per gate operation
noise_model = cirq.ConstantQubitNoiseModel(cirq.depolarize(p_depo))

# ── DensityMatrixSimulator: tracks mixed states (required for noise) ──────
noisy_sim = cirq.DensityMatrixSimulator(noise=noise_model)

# ── Build a 3-qubit GHZ circuit ──────────────────────────────────────────
q = cirq.LineQubit.range(3)
ghz = cirq.Circuit([
    cirq.H(q[0]),
    cirq.CNOT(q[0], q[1]),
    cirq.CNOT(q[1], q[2]),
    cirq.measure(*q, key="result"),
])

# ── Ideal simulation ────────────────────────────────────────────────────
ideal_sim = cirq.DensityMatrixSimulator()   # no noise
ideal_result = ideal_sim.simulate(cirq.Circuit([cirq.H(q[0]),
                                  cirq.CNOT(q[0],q[1]), cirq.CNOT(q[1],q[2])]))
dm_ideal = ideal_result.final_density_matrix
fid_ideal = np.trace(dm_ideal @ dm_ideal).real   # purity (=1 for pure state)
print(f'Ideal purity: {fid_ideal:.4f}')

# ── Noisy simulation ─────────────────────────────────────────────────────
noisy_result = noisy_sim.simulate(cirq.Circuit([cirq.H(q[0]),
                                   cirq.CNOT(q[0],q[1]), cirq.CNOT(q[1],q[2])]))
dm_noisy = noisy_result.final_density_matrix
fid_noisy = np.trace(dm_noisy @ dm_noisy).real
print(f'Noisy purity: {fid_noisy:.4f}  (depolarising p={p_depo})')

# ── Custom per-qubit noise model ─────────────────────────────────────────
# For hardware-realistic simulation with different T1/T2 per qubit:
class PerQubitNoise(cirq.NoiseModel):
    def __init__(self, t1_map, t2_map, gate_time_ns):
        self.t1 = t1_map; self.t2 = t2_map; self.dt = gate_time_ns
    def noisy_operation(self, operation, moment_index):
        ops = [operation]
        for qubit in operation.qubits:
            t1 = self.t1.get(qubit, 100e3)  # default 100 μs in ns
            t2 = self.t2.get(qubit, 80e3)
            gamma = 1 - np.exp(-self.dt / t1)  # amplitude damping
            lam   = 1 - np.exp(-self.dt / t2)
            ops.append(cirq.amplitude_damp(gamma).on(qubit))
            ops.append(cirq.phase_damp(lam).on(qubit))
        return ops

t1_map = {q[0]: 200e3, q[1]: 150e3, q[2]: 100e3}  # T1 in ns
t2_map = {q[0]: 120e3, q[1]:  90e3, q[2]:  80e3}
per_qubit_noise = PerQubitNoise(t1_map, t2_map, gate_time_ns=50)
```

### 10.2.4 Cross-Entropy Benchmarking (XEB)

Cross-Entropy Benchmarking (XEB) is Google’s primary method for characterising quantum processor fidelity. Unlike randomised benchmarking (which measures average gate error per Clifford), XEB compares the probability distribution measured on hardware against the ideal distribution predicted by simulation, using the cross-entropy as the figure of merit. XEB was the metric used in Google’s quantum supremacy claim.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: XEB Fidelity: Definition and Interpretation</strong></p>
<p>For a random quantum circuit U with ideal output distribution P_ideal(x) = |&lt;x|U|0&gt;|^2:</p>
<p>Physical interpretation: if the hardware is perfectly noiseless, the bitstrings it produces are exactly those predicted to be most probable by the ideal simulation — so the average of P_ideal over those bitstrings equals 1/D + 1/D = 2/D, and F_XEB = D · 2/D – 1 = 1. If the hardware produces completely random bitstrings, they are uniformly distributed, E_x[P_ideal] = 1/D, and F_XEB = D · 1/D – 1 = 0.</p>
<p>Practical usage: run the same random circuit on both the simulator (to get P_ideal) and the hardware (to get the sample bitstrings), then compute F_XEB. For a random circuit with depth d and per-layer error rate e: F_XEB ≈ (1−2e)^d. Fitting to this model gives the average two-qubit gate error rate.</p>
</div>

### 10.2.5 Accessing Google Quantum Hardware

As of 2025, Google Sycamore and Willow processors are not publicly accessible. Access is available to academic and industry research partners through the Google Quantum AI partnership programme (https://quantumai.google/partners). Applications require a research proposal and are reviewed by Google. The programme is selective; most students and researchers use Cirq for simulation and algorithm development rather than hardware execution.

Alternatives for Cirq-based hardware execution: Cirq can submit circuits to Google’s hardware via the quantum\_engine module (for partners), or to IonQ and Rigetti processors via Amazon Braket’s Cirq integration (braket.circuits.Circuit.from\_cirq()), making Cirq circuits runnable on multiple providers without rewriting.

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: Cirq in Indian Quantum Research</strong></p>
<p>Several Indian research groups use Cirq for quantum algorithm research and noise characterisation. IISc Bangalore’s Centre for Quantum Information and Quantum Computing (CQIQC) uses Cirq for quantum error correction simulations. IIT Madras’ Quantum Computing Group employs Cirq’s DensityMatrixSimulator for superconducting qubit noise modelling. The Tata Institute of Fundamental Research (TIFR) uses Cirq alongside Qiskit for cross-platform algorithm benchmarking.</p>
<p>For M.Sc. Physics students: learning Cirq alongside Qiskit provides two major advantages. First, it deepens understanding of the quantum circuit model because Cirq’s explicit Moment structure forces you to think about which operations are parallel versus sequential. Second, the ability to read Cirq code is essential for understanding Google’s published research, which uses Cirq throughout. Many Nature and Physical Review Letters papers on quantum computing include Cirq code in their supplementary materials.</p>
</div>

## 10.3 Amazon Braket

Amazon Braket is a fully managed cloud service that enables access to quantum computers and simulators from multiple hardware providers through a unified API. Launched in 2019, it is part of Amazon Web Services (AWS) and integrates with the broader AWS ecosystem including S3 (for storing quantum results), IAM (for access control), CloudWatch (for monitoring), and EC2/Lambda (for classical post-processing). Braket’s architecture follows Amazon’s established cloud-services model: quantum circuits become Tasks that are submitted to a Device (simulator or QPU), results are stored in S3, and the workflow is managed through the standard AWS SDK.

<figure class="book-figure">
<img src="content/images/image101.png" alt="Figure 10.3: Amazon Braket architecture and provider comparison. Left: The Braket stack from user Python code through the Amazon Braket Service (AWS cloud), Task management, hardware provider selection, and down to individual simulator and QPU backends. IonQ Forte (trapped ion, 35 qubits), Rigetti Aspen-M (superconducting, 79 qubits), Oxford QC (spin qubit, 8 qubits), and QuEra Aquila (neutral atom, 256 qubits) are shown. Right: Physical qubit count and QV comparison for main Braket QPU providers.">
<figcaption>Figure 10.3: Amazon Braket architecture and provider comparison. Left: The Braket stack from user Python code through the Amazon Braket Service (AWS cloud), Task management, hardware provider selection, and down to individual simulator and QPU backends. IonQ Forte (trapped ion, 35 qubits), Rigetti Aspen-M (superconducting, 79 qubits), Oxford QC (spin qubit, 8 qubits), and QuEra Aquila (neutral atom, 256 qubits) are shown. Right: Physical qubit count and QV comparison for main Braket QPU providers.</figcaption>
</figure>

### 10.3.1 Braket Architecture: Tasks, Devices, and Managed Jobs

The fundamental Braket execution unit is a Task — a single circuit submission with a specified number of shots and a target device. Tasks are stateless: once submitted, they execute asynchronously; results are stored in S3 and retrieved later via the task ARN (Amazon Resource Name). For iterative algorithms (VQE, QAOA), Amazon Braket Hybrid Jobs provide a managed execution environment that runs classical code (in Python/PyTorch/TensorFlow) alongside quantum circuit calls in a dedicated EC2 instance.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Braket Execution Model: Tasks vs Hybrid Jobs</strong></p>
<p>Task (standard execution):</p>
<p>Submit: task = device.run(circuit, shots=1000)</p>
<p>Returns: QuantumTask object with task ARN</p>
<p>Results: task.result() blocks; alternatively retrieve later via task ARN from S3</p>
<p>Best for: single-circuit experiments, parameter sweeps, benchmarking</p>
<p>Hybrid Job (managed iterative execution):</p>
<p>Defines a Python function (entry_point) that can call device.run() inside an optimisation loop</p>
<p>Runs on dedicated AWS infrastructure (no queue for the QPU calls within the job)</p>
<p>Supports checkpointing and automatic restart</p>
<p>Best for: VQE, QAOA, quantum machine learning with many iterations</p>
<p>Local Simulator: cirq.DensityMatrixSimulator equivalent but using the Braket local SDK — runs on your laptop, no AWS account needed. Useful for development and testing.</p>
</div>

### 10.3.2 Building Circuits with the Braket SDK

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 10.3 — Amazon Braket SDK: Building and Running Circuits
# pip install amazon-braket-sdk
# AWS credentials required: aws configure  (or IAM role)
# ─────────────────────────────────────────────────────────────────────────
from braket.circuits import Circuit, Gate, Qubit
from braket.devices import LocalSimulator
from braket.aws import AwsDevice

# ── Build a Bell state circuit ─────────────────────────────────────────────
# Braket uses method chaining: circuit.gate(qubit)
bell = Circuit().h(0).cnot(0, 1)
print('Bell circuit:')
print(bell)
# Output:
# T  : |0|1|
# q0 : -H-C-
#          |
# q1 : ---X-

# ── Add measurement (Braket measures ALL qubits by default at end) ─────────
# Explicit probabilities instruction:
bell_probs = Circuit().h(0).cnot(0, 1).probability(target_indices=[0, 1])

# ── Build GHZ state ────────────────────────────────────────────────────────
ghz = Circuit().h(0).cnot(0, 1).cnot(1, 2)

# ── Native Braket gates: h, cnot, cz, rx, ry, rz, x, y, z, s, t, etc. ────
# Parametric gates:
from braket.circuits import FreeParameter
theta = FreeParameter("theta")
param_circuit = Circuit().ry(0, theta).cnot(0, 1)

# ── Run on Local Simulator (no AWS account needed) ──────────────────────────
device = LocalSimulator()         # runs on your machine
task = device.run(bell, shots=1024)
result = task.result()
print('Local simulator counts:', result.measurement_counts)
# Output: Counter({'00': ~512, '11': ~512})

# ── Exact statevector simulation ──────────────────────────────────────────
sv_device = LocalSimulator('default')   # statevector backend
sv_task = sv_device.run(Circuit().h(0).cnot(0,1))
sv_result = sv_task.result()
print('State vector:', sv_result.get_value_by_result_type(
    braket.circuits.result_types.StateVector()))
```

### 10.3.3 Running on Braket Simulators and QPU Devices

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 10.4 — Amazon Braket: Cloud Simulators and QPU Execution
# Requires: AWS account, boto3 configured, Braket permissions in IAM
# ─────────────────────────────────────────────────────────────────────────
from braket.aws import AwsDevice, AwsSession
from braket.circuits import Circuit
import boto3

# ── Braket managed simulators (run on AWS cloud) ──────────────────────────
# SV1: statevector simulator, up to 34 qubits
sv1 = AwsDevice('arn:aws:braket:::device/quantum-simulator/amazon/sv1')
# DM1: density matrix simulator (supports noise), up to 17 qubits
dm1 = AwsDevice('arn:aws:braket:::device/quantum-simulator/amazon/dm1')
# TN1: tensor network simulator (up to 50 qubits for certain circuits)
tn1 = AwsDevice('arn:aws:braket:::device/quantum-simulator/amazon/tn1')

bell = Circuit().h(0).cnot(0, 1)

# ── Submit to SV1 (cloud statevector simulator) ───────────────────────────
s3_folder = ('my-braket-bucket', 'results/')  # S3 bucket for results
task_sv1 = sv1.run(bell, s3_destination_folder=s3_folder, shots=1000)
print(f'SV1 Task ARN: {task_sv1.id}')
# Results stored in S3 automatically

# ── Run on QPU: IonQ Forte (trapped ion, 35 qubits) ──────────────────────
ionq_forte = AwsDevice('arn:aws:braket:us-east-1::device/qpu/ionq/Forte-1')

# Check device availability
print(f'IonQ Forte status: {ionq_forte.status}')
print(f'Execution windows: {ionq_forte.properties.service.executionWindows}')
# IonQ hardware has scheduled execution windows (not 24/7 like IBM)

# Submit circuit to real hardware
task_ionq = ionq_forte.run(bell,
    s3_destination_folder=s3_folder,
    shots=1000)
print(f'IonQ task ARN: {task_ionq.id}')

# ── Retrieve results (can be done in a separate Python session) ───────────
result = task_ionq.result()   # blocks until complete
print('IonQ counts:', result.measurement_counts)
print('IonQ result types:', result.result_types)

# Retrieve later by ARN (different session):
# from braket.aws import AwsQuantumTask
# task = AwsQuantumTask(arn='arn:aws:braket:...:quantum-task/...')
# result = task.result()

# ── Parametric task: parameter binding for VQE-style sweeps ──────────────
from braket.circuits import FreeParameter
import numpy as np

theta = FreeParameter("theta")
p_circuit = Circuit().ry(0, theta).cnot(0,1)
theta_values = np.linspace(0, 2*np.pi, 10)

# Submit batch of parametric tasks:
tasks = [sv1.run(p_circuit(theta=t), s3_destination_folder=s3_folder,
                 shots=500) for t in theta_values]
results = [t.result() for t in tasks]
p1_values = [r.measurement_counts.get('10', 0) / 500 for r in results]
print('P(|10>) vs theta:', [f'{p:.3f}' for p in p1_values])
```

### 10.3.4 Braket Pricing and Access Model

Amazon Braket pricing has two components: a per-task fee charged for each quantum circuit submission, and a per-shot fee charged for each circuit execution (repetition). Pricing varies by device type and provider. All managed simulator usage (SV1, DM1, TN1) has a per-minute compute cost.

| Device / Simulator | Per-task fee | Per-shot fee | Notes |
|---|---|---|---|
| Local Simulator | Free | Free | Runs on your machine, no AWS needed, up to 25 qubits |
| SV1 (cloud statevec) | $0.075/min | Free | Up to 34 qubits; cost ~$0.10 per typical circuit |
| DM1 (density matrix) | $0.075/min | Free | Up to 17 qubits, supports noise models |
| TN1 (tensor network) | $0.275/min | Free | Up to 50 qubits for low-entanglement circuits |
| IonQ Forte-1 (QPU) | $0.30/task | $0.00035/shot | 1000 shots = $0.65 total; not always available |
| Rigetti Aspen-M (QPU) | $0.30/task | $0.00035/shot | Same pricing as IonQ; execution windows apply |
| QuEra Aquila (QPU) | $0.01/task | $0.01/shot | Analog (AHS) mode; different pricing model |

Cost estimation for a 100-iteration QAOA with 2-qubit Bell circuits on IonQ Forte: 100 tasks × $0.30/task + 100,000 shots × $0.00035/shot = $30 + $35 = $65. Compare with IBM Quantum Open Plan: $0 (free). This cost difference makes IBM Quantum the clear choice for coursework and exploratory research; Braket becomes relevant when the algorithm requires multi-provider comparison or AWS ecosystem integration.

<div class="box box-warning">
<p class="box-title"><strong>⚠  Warning: Braket QPU Execution Windows</strong></p>
<p>Unlike IBM Quantum (which processes jobs 24/7 in a continuous queue), Amazon Braket QPU devices — IonQ, Rigetti, and others — have scheduled execution windows: specific hours of the day when the hardware accepts and processes quantum tasks. Outside these windows, submitted tasks queue and execute in the next available window. Check device.properties.service.executionWindows before submitting to avoid unexpected delays of 12–24 hours.</p>
</div>

## 10.4 Microsoft Azure Quantum

Microsoft’s approach to quantum computing is philosophically distinct from IBM, Google, and Amazon. While those companies all offer gate-model quantum computers running quantum circuits written in Python libraries, Microsoft has invested in two fundamentally different bets: (1) topological qubits — a hardware architecture expected to have intrinsically lower error rates than superconducting or trapped-ion qubits — and (2) Q#, a dedicated quantum programming language that treats quantum operations as first-class language constructs, not library calls embedded in Python.

<figure class="book-figure">
<img src="content/images/image102.png" alt="Figure 10.4: Microsoft Azure Quantum architecture and Q# language. Left: The Azure Quantum stack — from the Azure Quantum Portal through Python SDK and Q# language, Azure Quantum Workspace, hardware providers (IonQ, Quantinuum, Pasqal), and Azure resource management. Right: Side-by-side comparison of Q# (native quantum language with operation/measurement constructs) and Python SDK (azure-quantum with Cirq integration) for the same Bell state circuit, illustrating the two approaches to quantum programming on Azure.">
<figcaption>Figure 10.4: Microsoft Azure Quantum architecture and Q# language. Left: The Azure Quantum stack — from the Azure Quantum Portal through Python SDK and Q# language, Azure Quantum Workspace, hardware providers (IonQ, Quantinuum, Pasqal), and Azure resource management. Right: Side-by-side comparison of Q# (native quantum language with operation/measurement constructs) and Python SDK (azure-quantum with Cirq integration) for the same Bell state circuit, illustrating the two approaches to quantum programming on Azure.</figcaption>
</figure>

### 10.4.1 Azure Quantum Workspace and Providers

An Azure Quantum Workspace is an Azure cloud resource that groups together quantum hardware access, billing, and management. Creating a Workspace requires an Azure subscription (the first $500 in credits is free for new Azure accounts). The Workspace connects to hardware providers — IonQ (trapped ion), Quantinuum (trapped ion), and Pasqal (neutral atom) — through pre-negotiated provider agreements built into the Azure Quantum service.

- IonQ on Azure: IonQ Harmony (11q), IonQ Aria-1/Aria-2 (25q), IonQ Forte (36q). Pricing: $0.00097/qubit-gate + $0.000975/circuit shot.

- Quantinuum on Azure: H-Series processors (H1-1 20q, H2-1 32q). Pricing: quantum hardware usage credits (HQCs) at $1.30/HQC; typical circuit costs $3–10.

- Pasqal on Azure: neutral atom QPU, 100 qubits in analog mode. Pricing: per-hour reservation model.

- Azure Quantum Resource Estimator: a unique tool that estimates the number of physical and logical qubits, T gates, and runtime required to run an algorithm on a fault-tolerant quantum computer — without running on real hardware. Useful for assessing the feasibility of large-scale algorithms.

### 10.4.2 Q#: Microsoft’s Quantum Programming Language

Q# (pronounced "Q sharp") is a domain-specific language for quantum computing, developed by Microsoft as part of the Quantum Development Kit (QDK). Unlike Qiskit, Cirq, and Braket — which are Python libraries where quantum operations are function calls — Q# is a full programming language with its own type system, syntax, and compiler. This distinction matters because Q# can express algorithms at a higher level of abstraction, with the compiler handling circuit synthesis, qubit allocation, and resource estimation automatically.

<div class="box box-key-concept">
<p class="box-title"><strong>🔑  Key Concept: Q# Language Features</strong></p>
<p>Q# is a statically typed, functional-inspired quantum programming language. Key concepts:</p>
<p>Operations: the primary unit of quantum programming. An operation takes inputs and performs quantum computations. Example: operation H(q : Qubit) : Unit is Adj is the built-in Hadamard.</p>
<p>Functions: purely classical computations (no quantum operations). Pure functions are guaranteed to be side-effect-free.</p>
<p>Qubit management: qubits are allocated with use q = Qubit() inside a block and automatically returned. The programmer never assigns physical qubit indices — the compiler handles allocation.</p>
<p>Adjoint and Controlled: every quantum operation automatically generates its adjoint (inverse) and controlled versions using the is Adj and is Ctl qualifiers. This makes circuit inversion trivial.</p>
<p>Measurements: MResetZ(q) measures and resets in one operation; Result type is Zero or One.</p>
<p>Classical control: full classical control flow (if/for/while) based on measurement results, enabling mid-circuit classical processing.</p>
</div>

```python
// ─────────────────────────────────────────────────────────────────────────
// Code 10.5 — Q# Bell State Program
// Requires: QDK installed, dotnet SDK, or Azure Quantum portal notebook
// ─────────────────────────────────────────────────────────────────────────
namespace BellStateDemo {
    open Microsoft.Quantum.Canon;
    open Microsoft.Quantum.Intrinsic;
    open Microsoft.Quantum.Measurement;

    // Operation: prepare and measure Bell state
    // Returns (Result, Result) = two measurement outcomes
    @EntryPoint()
    operation RunBellState() : (Result, Result) {
        // Allocate two qubits (automatically initialised to |0>)
        use (q1, q2) = (Qubit(), Qubit());

        // Prepare Bell state |Phi+> = (|00> + |11>) / sqrt(2)
        H(q1);           // Hadamard on q1
        CNOT(q1, q2);    // CNOT: q1 = control, q2 = target

        // Measure both qubits and reset (REQUIRED before scope exit)
        let r1 = MResetZ(q1);  // measures and resets q1
        let r2 = MResetZ(q2);

        // Return measurement results
        return (r1, r2);
        // Note: qubits automatically released here by the runtime
    }

    // Run 1000 shots and count |00> vs |11> outcomes
    operation EstimateBellFidelity(nShots : Int) : Double {
        mutable correctCount = 0;
        for _ in 1..nShots {
            let (r1, r2) = RunBellState();
            // Correct outcomes: (Zero,Zero) = |00> or (One,One) = |11>
            if r1 == r2 { set correctCount += 1; }
        }
        return IntAsDouble(correctCount) / IntAsDouble(nShots);
    }
}

// ── To run on Azure Quantum IonQ backend (from Python): ───────────────
// workspace = Workspace(subscription_id="...", resource_group="...",
//                        name="my-workspace", location="eastus")
// from azure.quantum.target.ionq import IonQ
// target = workspace.get_targets("ionq.qpu.aria-1")
// job = target.submit(RunBellState, shots=1000)
// print(job.get_results())
```

### 10.4.3 Using Azure Quantum with the Python SDK

For students who prefer Python over Q#, Microsoft provides the azure-quantum Python package which integrates with Cirq, Qiskit, and the Braket SDK, allowing circuits written in any of these frameworks to be submitted to Azure Quantum hardware providers.

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 10.6 — Azure Quantum Python SDK with Cirq Integration
# pip install azure-quantum cirq-core
# Requires: Azure subscription with Quantum Workspace resource
# ─────────────────────────────────────────────────────────────────────────
from azure.quantum import Workspace
from azure.quantum.cirq import AzureQuantumService
import cirq

# ── Connect to Azure Quantum Workspace ───────────────────────────────────
workspace = Workspace(
    subscription_id  = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    resource_group   = "my-quantum-rg",
    name             = "my-quantum-workspace",
    location         = "eastus"
)

# ── Create service object and list available targets ─────────────────────
service = AzureQuantumService(workspace)
print('Available targets:')
for target in service.targets():
    print(f'  {target.name}  (status: {target.current_availability})')
# Expected output:
# ionq.qpu.aria-1  (status: Available)
# ionq.simulator   (status: Available)
# quantinuum.hqs-lt-s1-apival (status: Available)
# pasqal.sim.emu-tn  (status: Available)

# ── Build Bell state in Cirq ─────────────────────────────────────────────
q = cirq.LineQubit.range(2)
bell = cirq.Circuit([cirq.H(q[0]), cirq.CNOT(q[0], q[1]),
                     cirq.measure(*q, key="result")])

# ── Run on IonQ Simulator (free, no hardware queue) ──────────────────────
result_sim = service.run('ionq.simulator', bell, repetitions=1000)
print('IonQ simulator histogram:', result_sim.histogram(key='result'))

# ── Run on IonQ Aria QPU (real hardware, costs HQCs) ─────────────────────
# result_hw = service.run("ionq.qpu.aria-1", bell, repetitions=1000)
# print("IonQ Aria result:", result_hw.histogram(key="result"))

# ── Resource Estimation (unique Azure Quantum feature) ───────────────────
# Estimate resources for running a quantum algorithm at scale
# without executing on real hardware
re_target = service.get_target('microsoft.estimator')

# Build a Grover circuit (4 qubits) and estimate fault-tolerant resources
from qiskit import QuantumCircuit as QC
# ... build large circuit ...
# re_job = re_target.submit(large_circuit)
# re_result = re_job.get_results()
# print('Logical qubits:', re_result['physicalCounts']['breakdown']['logicalQubits'])
# print('Physical qubits:', re_result['physicalCounts']['physicalQubits'])
# print('Runtime (years):', re_result['physicalCounts']['runtime'])
```

## 10.5 Cross-Platform Comparison and Interoperability

<figure class="book-figure">
<img src="content/images/image103.png" alt="Figure 10.5: Platform selection guide and cost comparison. Left: Decision flowchart for selecting a quantum platform based on whether free access is required, multi-provider access is needed, and whether Microsoft/Q# tools are preferred. IBM Quantum is recommended for free academic use; Amazon Braket for AWS integration and multi-provider; Azure Quantum for Microsoft tools; Cirq for Google research. Right: Estimated cost per 1,000 shots for a 2-qubit Bell state circuit — IBM Open Plan and Network are free; commercial providers charge $0.50–$0.80 per 1,000 shots.">
<figcaption>Figure 10.5: Platform selection guide and cost comparison. Left: Decision flowchart for selecting a quantum platform based on whether free access is required, multi-provider access is needed, and whether Microsoft/Q# tools are preferred. IBM Quantum is recommended for free academic use; Amazon Braket for AWS integration and multi-provider; Azure Quantum for Microsoft tools; Cirq for Google research. Right: Estimated cost per 1,000 shots for a 2-qubit Bell state circuit — IBM Open Plan and Network are free; commercial providers charge $0.50–$0.80 per 1,000 shots.</figcaption>
</figure>

### 10.5.1 SDK Comparison: Qiskit vs Cirq vs Braket vs Q#

The four SDKs have genuinely different design philosophies that reflect their intended use cases. Understanding these differences helps students choose the right tool and read code across frameworks.

| Feature | Qiskit (IBM) | Cirq (Google) | Braket (Amazon) | Q# (Microsoft) |
|---|---|---|---|---|
| Language | Python | Python | Python | Q# (+ Python via IQ#) |
| Gate model | QuantumCircuit + .append() | Circuit + Moments | Circuit + method chain | Operations + auto-adjoint |
| Qubit type | int index | GridQubit LineQubit | int index | Qubit (abstract) |
| Native simulator | Qiskit Aer (local/cloud) | cirq.Simulator DensityMatrix | LocalSimulator SV1/DM1/TN1 | QuantumSimulator (resource est.) |
| Noise model | NoiseModel (Kraus channels) | ConstantQubitNoise Custom channels | Noise pragma DM1 device | Noise model via Q# API |
| Circuit export | qc.qasm() | cirq.to_qasm() | circuit.to_openqasm | Q# compiler → QASM/LLVM |
| Free hardware | Yes (Open Plan) | No (partners only) | No (pay-per-use) | $500 Azure credit |
| Unique strength | Ecosystem, QV/CLOPS tools | Moment control, XEB | Multi-provider AWS integration | Q# language, Resource Est. |

### 10.5.2 OpenQASM Interoperability and Circuit Translation

<figure class="book-figure">
<img src="content/images/image104.png" alt="Figure 10.6: Cross-platform circuit translation. Left: Bell state circuit code in all four SDKs — Qiskit, Cirq, Amazon Braket, and Azure Quantum Python SDK — showing different syntactic styles but equivalent quantum operations. Right: OpenQASM interoperability hub — OpenQASM 2.0/3.0 as the common IR enables circuit exchange between Qiskit, Cirq, Braket SDK, Azure QDK, PennyLane (Xanadu), and Pytket (Quantinuum). Two-way arrows indicate that circuits can be imported from and exported to each framework.">
<figcaption>Figure 10.6: Cross-platform circuit translation. Left: Bell state circuit code in all four SDKs — Qiskit, Cirq, Amazon Braket, and Azure Quantum Python SDK — showing different syntactic styles but equivalent quantum operations. Right: OpenQASM interoperability hub — OpenQASM 2.0/3.0 as the common IR enables circuit exchange between Qiskit, Cirq, Braket SDK, Azure QDK, PennyLane (Xanadu), and Pytket (Quantinuum). Two-way arrows indicate that circuits can be imported from and exported to each framework.</figcaption>
</figure>

```python
# ─────────────────────────────────────────────────────────────────────────
# Code 10.7 — Cross-Platform Circuit Translation via OpenQASM
# ─────────────────────────────────────────────────────────────────────────
from qiskit import QuantumCircuit
import cirq

# ── Step 1: Build Bell state in Qiskit ───────────────────────────────────
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

# ── Step 2: Export to OpenQASM 2.0 ──────────────────────────────────────
qasm_str = qc.qasm()     # or qc.qasm2_string() in newer Qiskit
print('OpenQASM 2.0 output:')
print(qasm_str)
# Output:
# OPENQASM 2.0;
# include "qelib1.inc";
# qreg q[2];  creg meas[2];
# h q[0];  cx q[0],q[1];  barrier q[0],q[1];
# measure q[0] -> meas[0];  measure q[1] -> meas[1];

# ── Step 3: Import into Cirq ─────────────────────────────────────────────
cirq_circuit = cirq.Circuit.from_qasm(qasm_str)
print('\nCirq circuit imported from QASM:')
print(cirq_circuit)

# ── Step 4: Export Cirq circuit back to QASM ─────────────────────────────
qasm_from_cirq = cirq.qasm(cirq_circuit)
print('\nQASM from Cirq:', qasm_from_cirq[:200], '...')

# ── Step 5: Import into Amazon Braket ────────────────────────────────────
from braket.circuits.serialization import OpenQASMSerializationProperties
from braket.circuits import Circuit as BraketCircuit

# Build same circuit natively in Braket for comparison:
braket_bell = BraketCircuit().h(0).cnot(0,1)

# Export Braket to QASM:
qasm_from_braket = braket_bell.to_unitary().__str__()  # via intermediate

# ── Step 6: PennyLane integration (cross-platform ML) ────────────────────
# pip install pennylane pennylane-qiskit
import pennylane as qml

# Convert Qiskit circuit to PennyLane:
qml_circuit = qml.from_qiskit(qc)
print('\nPennyLane circuit:', qml_circuit)

# PennyLane can execute on any backend (Qiskit, Braket, Cirq):
dev_qiskit = qml.device('qiskit.aer', wires=2)
dev_braket = qml.device('braket.local.qubit', wires=2)  
```

### 10.5.3 Choosing the Right Platform for Your Algorithm

The decision of which platform to use depends on five factors: budget (IBM Open Plan is free; others charge per shot), algorithm type (VQE on IBM; multi-provider comparison on Braket; noise research with Cirq; enterprise on Azure), hardware requirements (qubit count, connectivity, fidelity), ecosystem integration (AWS, Azure, or standalone), and team expertise. The figure 10.5 decision tree summarises these criteria. For most students:

- Start with IBM Quantum: free access, best documentation, Qiskit is the industry standard, and IBM provides the most accessible quantum hardware for coursework and research.

- Add Cirq for simulation research: when you need fine-grained noise control, custom gate definitions, or are reading Google quantum research papers that use Cirq.

- Add Braket when moving to industry or AWS: if your organisation uses AWS or you need to access multiple hardware providers through a single API.

- Add Azure Quantum for Q# and resource estimation: the Azure Quantum Resource Estimator is the best available tool for assessing the feasibility of fault-tolerant algorithms before they can be run on real hardware.

## 10.6 The NISQ Era: Present State and Future Outlook

Having studied the theory of quantum computation, the physics of noise and hardware, the practice of IBM Quantum access, and the broader ecosystem of quantum cloud platforms, we conclude this book with a frank assessment of where quantum computing stands today, where it is going, and what this means for M.Sc. Physics students entering the field.

<figure class="book-figure">
<img src="content/images/image105.png" alt="Figure 10.7: Technology readiness and quantum computing timeline. Left: Technology Readiness Level (TRL 1–9) for different quantum application domains across superconducting, trapped-ion, and photonic platforms. Quantum key distribution and sensing are most mature (TRL 6–9); fault-tolerant algorithms for RSA breaking remain at TRL 1. Right: Timeline of key milestones from IBM’s first cloud QC (2016) through Google Willow (2024) to projected fault-tolerant universal QC (2030–2033). Solid markers = achieved; open markers = projected.">
<figcaption>Figure 10.7: Technology readiness and quantum computing timeline. Left: Technology Readiness Level (TRL 1–9) for different quantum application domains across superconducting, trapped-ion, and photonic platforms. Quantum key distribution and sensing are most mature (TRL 6–9); fault-tolerant algorithms for RSA breaking remain at TRL 1. Right: Timeline of key milestones from IBM’s first cloud QC (2016) through Google Willow (2024) to projected fault-tolerant universal QC (2030–2033). Solid markers = achieved; open markers = projected.</figcaption>
</figure>

### 10.6.1 What NISQ Computers Can (and Cannot) Do Today

The NISQ era is characterised by a genuine tension: quantum processors are large and capable enough to perform computations that are classically infeasible in principle, but noisy enough that proving computational advantage over optimised classical algorithms remains difficult in practice. Here is an honest assessment:

#### Demonstrated and Near-Term Achievable

- Quantum supremacy (specific tasks): Google Sycamore (2019) and Willow (2024) performed random circuit sampling tasks that are classically intractable. The tasks are artificial — designed to be hard for classical computers — but the demonstrations are scientifically valid.

- Quantum error correction below threshold: Google Willow (2024) demonstrated below-threshold surface code error correction for the first time — the key theoretical requirement for fault-tolerant quantum computing. This is a landmark achievement even though fault-tolerant quantum computing at scale remains years away.

- Quantum simulation of specific systems: IBM and Google have demonstrated quantum simulation of small lattice gauge theories, quantum chemistry models, and condensed matter systems at scales (50–100 qubits) where classical simulation is difficult. These are the first demonstrations of potential practical quantum advantage.

- Quantum sensing and communication: these applications do not require NISQ-era gate-model computers at all. Quantum key distribution (QKD) is commercially deployed (China Micius satellite, Toshiba, ID Quantique). Quantum sensors (atomic clocks, gravimeters, magnetometers) already outperform classical technology and are being deployed in defence and geophysics.

#### Credibly Expected in 5–10 Years

- Quantum advantage for quantum chemistry: VQE on 100–1,000 logical qubits could simulate molecules (catalysts, drug targets) beyond classical reach. Requires error correction below threshold and ~100,000 physical qubits. IBM targets this by 2033.

- Quantum optimisation advantage: QAOA on specialised hardware may provide advantage for some combinatorial problems. Still theoretical — no unconditional proof of advantage for any practical problem.

- Fault-tolerant demonstrations: first logical qubit with error rate << physical error rate at practically useful scale. Requires ~1,000–5,000 physical qubits per logical qubit.

#### Long-Term (10+ Years) or Speculative

- Breaking RSA-2048 with Shor’s algorithm: requires ~20 million physical qubits at current error rates. This is a 2035+ target on optimistic projections.

- Topological qubits (Microsoft): if successful, would dramatically reduce the overhead for fault-tolerant computation. Technology is not yet proven at scale despite decades of research.

- Quantum machine learning at scale: most QML proposals require fault-tolerant hardware and face dequantization — classical algorithms can match the performance of many QML algorithms — making genuine QML advantage very uncertain.

### 10.6.2 The Road to Fault-Tolerant Quantum Computing

The path from today’s NISQ processors to fault-tolerant quantum computers is long but increasingly well-mapped. The key milestones and their prerequisites:

<div class="box box-equation">
<p><strong>Key Equation</strong></p>
<p>Physical error rate today (IBM Heron, best qubits): ~0.1–0.3% per CX gate</p>
<p>Surface code fault-tolerance threshold:             ~1% physical error rate</p>
<p>Required physical error rate for efficient FTQC:    ~0.01% (10x improvement needed)</p>
<p>Physical qubits per logical qubit (at 0.1% error):  ~1,000–5,000</p>
<p>Logical qubits for Shor RSA-2048:                   ~4,000</p>
<p>Physical qubits for Shor RSA-2048:                  ~4–20 million</p>
</div>

Google’s Willow demonstration (2024) showed that the surface code is working as theory predicts — the logical error rate decreases as code distance increases when physical error rates are below threshold. This is the crucial validation of the entire fault-tolerant quantum computing programme. The remaining challenges are engineering: scaling the number of physical qubits while maintaining below-threshold error rates, developing fast and accurate qubit readout, reducing crosstalk in large arrays, and building the classical control electronics for millions of simultaneous qubit operations.

### 10.6.3 Career Pathways in Quantum Computing

For M.Sc. Physics graduates completing their course in 2027, the quantum computing job market offers several distinct pathways. Understanding which skills correspond to which careers helps focus your learning priorities:

| Career Path | Key Skills Required | Where to Look |
|---|---|---|
| Quantum Software Engineer | Qiskit, Python, algorithms (VQE, QAOA, Grover), circuit optimisation, noise mitigation | IBM, Google, Rigetti, IonQ, QpiAI (India), startups |
| Quantum Hardware Engineer | Superconducting circuits, dilution refrigerators, microwave electronics, signal processing | IBM, Google, Intel, TIFR, IIT labs, national labs |
| Quantum Algorithms Researcher | QC theory, linear algebra, complexity theory, quantum error correction, variational methods | Universities, IBM Research, Microsoft Research, Caltech, MIT |
| Quantum Application Scientist | Domain chemistry/finance/optimisation + quantum, Python, VQE/QAOA, business communication | Pharma, finance (JPMorgan, Goldman), McKinsey Quantum |
| Quantum Educator / Technical Writer | All of the above + pedagogy, curriculum design, science communication | IBM, Qiskit Advocates programme, NPTEL, academic institutions |
| Quantum Policy & Strategy | Quantum literacy + policy understanding, geopolitics of technology | NQM India, DST, NITI Aayog, EU Quantum Flagship, consulting firms |

<div class="box box-real-world">
<p class="box-title"><strong>🌐  Real World: Quantum Computing Careers in India — 2025 Outlook</strong></p>
<p>India’s National Quantum Mission (NQM, 2023–2031, ₹6,003 crore) is creating a wave of quantum computing job opportunities across academia, government, and industry. Current hiring organisations include: QpiAI (Bangalore — quantum software and NISQ algorithms), BosonQ Psi (Pune — quantum simulation for engineering), Tata Consultancy Services (quantum computing practice), Wipro (quantum AI lab), IBM India Research (Bangalore), Microsoft Research India (Bangalore). Public sector: DRDO, ISRO, C-DAC, and the National Quantum Technology and Application Hubs at IIT Madras, IISc Bangalore, and IISER Pune are actively recruiting quantum-trained physicists. Typical entry-level roles: quantum software developer (CTC: ₹8–15 lakhs), quantum algorithms researcher (postdoc: ₹12–20 lakhs + benefits), quantum hardware engineer (CTC: ₹10–18 lakhs). Premium senior roles at IBM, Google, or QpiAI can reach ₹30–60+ lakhs. The NQM targets training 25,000 quantum professionals by 2031 — the supply of trained graduates is currently far below demand.</p>
</div>

## RECAP — SHORT ANSWER QUESTIONS & MODEL ANSWERS

Chapter 10: Other Quantum Platforms: Cirq, Braket, and Azure Quantum

Instructions: Answer each question in 3–6 lines. Each question carries equal marks.

**PART A — QUESTIONS**

**Q1.  Why do multiple quantum cloud platforms exist? Describe the strategic position of IBM Quantum, Google Quantum AI, Amazon Braket, and Microsoft Azure Quantum in the quantum market.**

**Q2.  What is OpenQASM? Compare OpenQASM 2.0 and 3.0. What capabilities does 3.0 add that are essential for dynamic circuits?**

**Q3.  Describe Cirq's circuit model. What are Moments and GridQubits? How does Cirq's Moment structure differ from Qiskit's circuit representation?**

**Q4.  What is the DensityMatrixSimulator in Cirq? How does it differ from the statevector simulator? Write Cirq code to simulate a noisy Bell state using ConstantQubitNoiseModel.**

**Q5.  Define Cross-Entropy Benchmarking (XEB). Write the linear XEB fidelity formula. What does F\_XEB = 0 indicate about the hardware?**

**Q6.  Compare XEB and randomised benchmarking (RB). For which purpose is each more appropriate? What are the limitations of XEB?**

**Q7.  Describe the Amazon Braket architecture. What is the difference between a Task and a Hybrid Job? What simulators does Braket provide and what are their qubit limits?**

**Q8.  Write Braket Python code to build a Bell state circuit and execute it on the LocalSimulator. How would you modify this to run on the IonQ QPU?**

**Q9.  What is Azure Quantum Workspace? What hardware providers are accessible? What is the role of Q# in the Azure Quantum ecosystem?**

**Q10.  Describe Q# as a quantum programming language. What makes it different from Qiskit/Cirq? What are Adjoint and Controlled functors?**

**Q11.  Compare Qiskit, Cirq, Braket SDK, and Q# on: (a) programming model, (b) free tier, (c) simulator options, (d) hardware access, (e) best use case.**

**Q12.  What is platform selection criteria for a quantum algorithm? For each scenario, recommend a platform: (a) M.Sc. coursework in India, (b) VQE research needing auto-differentiation, (c) comparing superconducting vs trapped-ion hardware, (d) developing a Q#-native quantum algorithm.**

**Q13.  What is OpenQASM 3.0's classical control flow syntax? Give an example of an if-statement in QASM 3.0 used in a dynamic circuit (e.g., teleportation feedforward).**

**Q14.  What can NISQ computers do in 2025 that provides genuine value? Be specific. What cannot they do yet that many people incorrectly claim?**

**Q15.  Describe three career pathways for M.Sc. Physics graduates in quantum computing in India by 2031. What specific skills and qualifications are required for each?**

**PART B — MODEL ANSWERS**

**Answer 1:**

Multiple platforms exist because of different hardware technologies, strategic positions, and ecosystems. IBM Quantum: democratisation strategy — free tier, open-source Qiskit, >500K users, largest fleet. Focuses on accessibility and IBM Cloud integration. Google Quantum AI: research-focused — landmark results (supremacy 2019, below-threshold error correction 2024), Cirq designed for research control, hardware restricted to collaborators. Revenue not primary motive. Amazon Braket: infrastructure play — AWS of quantum computing, multi-provider access (IonQ, Rigetti, QuEra, OQC, IQM) through AWS ecosystem. Companies already in AWS can add quantum without new infrastructure. Microsoft Azure Quantum: long-term topological qubit bet (non-Abelian anyons for intrinsically low error rates) + Q# quantum-native language + Azure integration. Currently accessing IonQ, Quantinuum, Pasqal hardware.

**Answer 2:**

OpenQASM (Open Quantum Assembly Language): text-based low-level language for quantum circuits serving as a common intermediate representation (IR) for circuit portability between frameworks. QASM 2.0 (2017): basic gate definitions (qreg, creg, U, CX, custom gates), measurement, barrier. No classical control, no loops, static circuits only. QASM 3.0 (2022) adds: classical control flow (if/else/while/for loops in real-time during execution), mid-circuit measurement with classical register feedback, real-time pulse-level gate definitions, rich classical data types (bool, int, float, angle, complex), subroutines. Essential for dynamic circuits: 3.0 enables if-based feedforward needed for quantum error correction and teleportation.

**Answer 3:**

Cirq circuit model: a sequence of Moments. Moment: a set of operations (gates) that can all be applied simultaneously because they act on disjoint qubits. A Circuit is an ordered list of Moments. GridQubit(row,col): qubit with 2D position on a grid, matching Sycamore's physical layout. LineQubit(i): qubit on a 1D chain. NamedQubit('a'): abstract named qubit. Difference from Qiskit: Qiskit circuits are defined as a sequence of gate instructions with implicit timing (layers computed by scheduler). Cirq explicitly defines Moments, giving direct control over parallelism. This is useful for noise modelling (applying noise after each Moment) and hardware-specific optimisation.

**Answer 4:**

DensityMatrixSimulator: evolves the full 2^n × 2^n density matrix, tracking mixed states. Required for: noise simulation (decoherence produces mixed states), open quantum system dynamics, accurate partial trace calculations. Statevector simulator: only valid for pure states, tracks 2^n-component vector — faster but cannot represent noise. Code: `import cirq; q0,q1 = cirq.LineQubit.range(2); bell = cirq.Circuit([cirq.H(q0), cirq.CNOT(q0,q1), cirq.measure(q0,q1,key='res')]); noise = cirq.ConstantQubitNoiseModel(cirq.depolarize(p=0.01)); sim = cirq.DensityMatrixSimulator(noise=noise); result = sim.run(bell, repetitions=1000); print(result.histogram(key='res'))`.

**Answer 5:**

XEB (Cross-Entropy Benchmarking): benchmarks hardware by comparing measured output distribution of random circuits with ideal simulated distribution. Linear XEB fidelity: F\_XEB = N·⟨p\_ideal(x)⟩\_measured − 1, where N=2^n, p\_ideal(x) = ideal probability of measured bitstring x, and ⟨·⟩ is average over measured samples. F\_XEB = 1: perfect hardware (measured distribution matches ideal). F\_XEB = 0: completely depolarised hardware — output is uniformly random, no quantum coherence. For F\_XEB = 0: the quantum computer produces bitstrings randomly with no correlation to the ideal quantum computation — equivalent to a classical random bit generator.

**Answer 6:**

RB: applies random Clifford sequences of increasing length k, measures fidelity decay F(k) = A·p^k + B. Gives average Clifford gate error r = 1−p. SPAM-robust (A,B absorb errors). Appropriate for: characterising average gate quality, comparing hardware generations, certification. Limitation: gives only average; doesn't capture circuit-specific errors or non-Markovian noise. XEB: compares specific random circuit outputs to ideal. Appropriate for: certifying hardware fidelity for specific circuit types, as used in Google's quantum supremacy experiments. Limitation: requires classical simulation of ideal circuits (hard for n>50); can give inflated fidelity for circuits with non-random structure; sensitive to choice of circuits.

**Answer 7:**

Braket architecture: Task = single circuit execution on a specific Device. Device: LocalSimulator (classical, free), AwsDevice (SV1/DM1/TN1 managed simulators, QPUs). Hybrid Job: iterative VQA on AWS EC2 classical compute, submitting Tasks in a loop. Simulators: SV1 (statevector, up to 34 qubits, free for first tier then $0.075/task-min); DM1 (density matrix with noise channels, up to 17 qubits); TN1 (tensor network for structured circuits, up to 50 qubits but limited topology). QPU providers: IonQ Forte (35 trapped-ion qubits, ~$0.01/shot), Rigetti (79 SC qubits, ~$0.00035/shot), QuEra Aquila (256 neutral atoms), OQC Lucy.

**Answer 8:**

Code: `from braket.circuits import Circuit; from braket.devices import LocalSimulator; c = Circuit(); c.h(0); c.cnot(0,1); c.measure([0,1]); device = LocalSimulator(); task = device.run(c, shots=1000); result = task.result(); print(result.measurement\_counts)`. For IonQ QPU: replace LocalSimulator with `from braket.aws import AwsDevice; device = AwsDevice('arn:aws:braket:us-east-1::device/qpu/ionq/Forte-1')` and set `shots=100` (minimum billed shots). Note: requires AWS account credentials (`aws configure`), and costs approximately 100 × $0.01 + $0.30 = $1.30 per run.

**Answer 9:**

Azure Quantum Workspace: a Microsoft Azure resource group containing quantum providers, classical compute, and storage. Access via Azure portal: create 'Quantum Workspace' resource, select providers (IonQ, Quantinuum, Pasqal), add to workspace. Authentication: Azure Active Directory (AAD) credentials. Python SDK: `pip install azure-quantum qiskit-azure-quantum`. Hardware providers: IonQ (trapped-ion, Forte, Aria), Quantinuum (H1, H2 trapped-ion, best fidelity), Pasqal (neutral atoms). Q# role: Microsoft's quantum language with first-class quantum types; compiles to QIR (LLVM-based Quantum Intermediate Representation), which targets simulators, hardware, and future topological qubits.

**Answer 10:**

Q# is a statically-typed, quantum-native domain-specific language. Key differences from Qiskit/Cirq: (1) Qubits are a first-class type: `use q = Qubit()` allocates qubits, automatically released at scope end. (2) Quantum operations are language constructs with Adjoint and Controlled functors auto-generated: any operation `Op` automatically has `Adjoint Op` (inverse) and `Controlled Op` (adds control qubit) without manual matrix computation. (3) Type system enforces quantum validity at compile time: cannot copy qubits (no-cloning enforced syntactically). (4) Compiles to QIR for hardware-independent execution. Adjoint functor: `Adjoint H(q)` ≡ H† (since H is self-adjoint, same as H); `Adjoint T(q)` ≡ T†. Controlled functor: `Controlled CNOT([control], [target1, target2])` adds an extra control layer.

**Answer 11:**

Qiskit: Python library embedding quantum in classical code; free IBM Quantum access; Aer simulator (up to 32 qubits noise-free, noisy models); IBM Eagle/Heron hardware; best for coursework, IBM hardware, algorithm development. Cirq: Python library with explicit Moment structure; no free QPU access (research collaborators only); DensityMatrixSimulator; research and noise modelling. Braket SDK: Python library with multi-provider interface; no free QPU access ($0.30/task min); LocalSimulator free, SV1/DM1/TN1 low cost; IonQ/Rigetti/QuEra/OQC; AWS integration. Q#: quantum-native language with Adjoint/Controlled; no free QPU access; built-in simulators; IonQ/Quantinuum/Pasqal via Azure; best for long-term quantum programming, Azure ecosystem.

**Answer 12:**

(a) M.Sc. coursework: IBM Quantum — free tier, best documentation, Qiskit widely taught, course assignments run on real hardware. (b) VQE with auto-differentiation: PennyLane — built-in parameter-shift gradients, PyTorch/JAX integration, backend-agnostic (can use IBM hardware as backend). (c) Comparing superconducting vs trapped-ion: Amazon Braket — single SDK accesses both Rigetti (SC) and IonQ (TI) with same circuit code; run same circuit on both, compare results. (d) Q#-native algorithm development: Azure Quantum — Q# development environment, Visual Studio integration, Azure Quantum simulation + IonQ/Quantinuum hardware access.

**Answer 13:**

QASM 3.0 if-statement syntax: `if (bit\_register == value) { gate\_operations; }`. Teleportation feedforward example: `OPENQASM 3.0; qubit[3] q; bit[2] c; h q[0]; cx q[0], q[1]; h q[2]; cx q[2], q[1]; h q[2]; c[0] = measure q[1]; c[1] = measure q[2]; if (c[0] == 1) { x q[0]; }  // Apply X correction if Bell measurement bit 0 = 1; if (c[1] == 1) { z q[0]; }  // Apply Z correction if Bell measurement bit 1 = 1`. This QASM 3.0 code directly describes the classical feedforward required for quantum teleportation — impossible in QASM 2.0 which had no if-statements.

**Answer 14:**

What NISQ computers CAN do in 2025 with genuine value: (1) Quantum sensing: atomic clocks, gravimeters, magnetometers — already surpassing classical limits, commercially deployed. (2) Quantum key distribution (QKD): secure communication using entangled photons — commercially deployed by ID Quantique, Toshiba, QuantumCTek. (3) Quantum simulation benchmarking: testing and validating quantum chemistry Hamiltonians on small molecules (H₂, LiH) with VQE. (4) Quantum error correction demonstrations: IBM and Google have demonstrated below-threshold logical qubits. (5) Algorithm research: VQE, QAOA, quantum ML — active research yielding genuine insights even without classical advantage. What they CANNOT do: break RSA, simulate large molecules with chemical accuracy, provide proven classical advantage on commercially relevant optimisation.

**Answer 15:**

(a) Quantum Software Engineer (NQM hub or industry): requires Python, Qiskit, algorithm knowledge (Grover, VQE, QAOA), linear algebra, quantum mechanics fundamentals. Qualification: M.Sc. Physics + Qiskit certification. Companies: TCS, QpiAI, Wipro Quantum. (b) Quantum Hardware Researcher (IIT/IISc NQM hub): requires quantum optics, microwave engineering, cryogenics, solid-state physics, transmon qubit design. Qualification: M.Sc. Physics + PhD in experimental quantum physics. Role: building 50–1000 qubit processors for NQM 2031 target. (c) Quantum Communication/QKD Engineer (DRDO/ISRO/private): requires quantum optics, photonics, free-space and fibre optics, BB84/BBM92 protocols, FPGA programming. Qualification: M.Sc. Physics + experience with quantum optics experiments. Role: NQM satellite QKD programme, ground station deployment.

## EXERCISES — CHAPTER 10

### A. Solved Problems

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.1</strong></p>
<p>Problem: Write the Bell state (|00⟩+|11⟩)/√2 circuit in (a) Qiskit, (b) Cirq, (c) Amazon Braket SDK. For each, identify the type of qubit object used and how the circuit is visualised.</p>
<p><strong>Solution:</strong></p>
<p>(a) Qiskit:</p>
<p>(b) Cirq:</p>
<p>(c) Amazon Braket:</p>
<p>Key differences: Qiskit uses append() or direct method calls; Cirq uses explicit list construction with Gate.on(qubit) syntax; Braket uses method chaining (.h(0).cnot(0,1)). Qiskit and Braket use integer qubit indices; Cirq uses GridQubit/LineQubit objects that carry spatial position information.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.2</strong></p>
<p>Problem: Explain the difference between cirq.Moment and a Qiskit QuantumCircuit layer. Why does Cirq’s explicit Moment structure matter for hardware execution and noise simulation?</p>
<p><strong>Solution:</strong></p>
<p>In Qiskit, a QuantumCircuit is a directed acyclic graph (DAG) of gate operations with implicit parallelism — the transpiler automatically groups compatible operations into simultaneous layers. The programmer appends operations in any order; the circuit compiles to a schedule when transpiled.</p>
<p>In Cirq, a Circuit is an ordered list of Moments. A Moment is an explicit set of operations that execute simultaneously. Each Moment must contain operations on disjoint qubits. The programmer explicitly controls which operations are parallel.</p>
<p>Why explicit Moments matter: (1) For hardware execution on devices with strict timing requirements (like Google Sycamore), explicit Moments map directly to hardware clock cycles — there is no ambiguity about what executes when. (2) For noise simulation, the DensityMatrixSimulator applies noise after each Moment (not each gate), allowing separate noise models for single-qubit and two-qubit Moments. (3) For circuit analysis, the depth (number of Moments) is immediately visible without transpilation. (4) For custom gate scheduling, the programmer can insert identity gates (padding) in specific Moments to control timing — important for dynamical decoupling sequences.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.3</strong></p>
<p>Problem: Convert the following Qiskit circuit to OpenQASM 2.0 by hand, then show how to import it into Cirq. The circuit: H(0), CX(0,1), T(1), measure all.</p>
<p><strong>Solution:</strong></p>
<p>OpenQASM 2.0 (written by hand):</p>
<p>Import into Cirq:</p>
<p>Note: Cirq’s from_qasm() supports most OpenQASM 2.0 gates from qelib1.inc. The T gate in Cirq is cirq.T = cirq.ZPowGate(exponent=0.25). The resulting Cirq circuit has the same structure: Moment[H(q(0))], Moment[CNOT(q(0),q(1))], Moment[T(q(1))], Moment[measure(q(0),q(1)].</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.4</strong></p>
<p>Problem: Compute the estimated cost of running a 50-iteration VQE optimisation on Amazon Braket using IonQ Forte-1, where each iteration requires 2 circuits × 500 shots. Compare with IBM Quantum.</p>
<p><strong>Solution:</strong></p>
<p>Amazon Braket (IonQ Forte-1):</p>
<p>Total tasks: 50 iterations × 2 circuits = 100 tasks</p>
<p>Total shots: 100 circuits × 500 shots = 50,000 shots</p>
<p>Task fee: 100 × $0.30 = $30.00</p>
<p>Shot fee: 50,000 × $0.00035 = $17.50</p>
<p>Total Braket cost: $30.00 + $17.50 = $47.50</p>
<p>IBM Quantum (Open Plan):</p>
<p>Total cost: $0.00 (free, subject to monthly usage limits)</p>
<p>IBM Quantum (Premium Network): $0.00 (institutional subscription covers usage)</p>
<p>Conclusion: IBM Quantum is significantly more cost-effective for academic research and coursework. Amazon Braket’s value lies in multi-provider access, AWS ecosystem integration, and availability for production workloads where the cost is justified by business requirements. For a student’s 50-iteration VQE, IBM Quantum is the clear choice.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.5</strong></p>
<p>Problem: Write a Cirq programme that builds a 3-qubit GHZ circuit, simulates it with 1% depolarising noise using DensityMatrixSimulator, and computes the GHZ fidelity from the density matrix.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.6</strong></p>
<p>Problem: Explain the XEB (Cross-Entropy Benchmarking) metric. For a 5-qubit random circuit that achieves F_XEB = 0.72, what does this imply about the hardware? If the per-layer error rate is e, and F_XEB ≈ (1−2e)^d where d=10 layers, compute e.</p>
<p><strong>Solution:</strong></p>
<p>XEB measures how well the hardware output distribution matches the ideal distribution from classical simulation. F_XEB = 1 means perfect agreement; F_XEB = 0 means the hardware output is completely random (pure noise). F_XEB is computed as: F_XEB = D · E_x[P_ideal(x)] − 1, where D = 2^n and E_x is the expectation over hardware-sampled bitstrings.</p>
<p>For F_XEB = 0.72 on 5 qubits (D=32): this means the hardware output distribution retains 72% of the ideal fidelity. The circuit is performing significantly better than random (F=0) but not perfectly (F=1). This is consistent with a 5-qubit circuit with ~5-10% total error, as would be expected for a ~10-gate depth circuit with 0.5-1% per-gate error.</p>
<p>Computing e from F_XEB = (1−2e)^d:</p>
<p>0.72 = (1−2e)^10</p>
<p>(1−2e) = 0.72^(1/10) = 0.72^0.1 = 0.9662</p>
<p>2e = 1 − 0.9662 = 0.0338</p>
<p>e = 0.0169 ≈ 1.7% per layer error rate</p>
<p>For a typical circuit with 2 CX gates per layer: per-gate CX error ≈ 0.85% — consistent with a moderate-quality superconducting or trapped-ion processor.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.7</strong></p>
<p>Problem: What is the Azure Quantum Resource Estimator and why is it useful? A researcher wants to run Grover’s algorithm on a database of 2^20 ≈ 1 million items. Estimate the number of logical and physical qubits needed for a fault-tolerant surface code implementation.</p>
<p><strong>Solution:</strong></p>
<p>The Azure Quantum Resource Estimator (microsoft.estimator target) estimates the computational resources — logical qubits, physical qubits, T gates, Toffoli gates, runtime — required to run a specified quantum algorithm on a fault-tolerant surface code quantum computer, without actually running on hardware. It is particularly valuable for assessing whether large-scale algorithms are feasible with near-future hardware.</p>
<p>Grover’s algorithm for N=2^20 items:</p>
<p>Circuit structure: n=20 qubits; approximately π√N/4 = π × 1024/4 ≈ 805 Grover iterations</p>
<p>Each iteration: ~20 Toffoli gates (for the oracle) + 20 diffusion gates</p>
<p>Total gates: 805 × 40 ≈ 32,200 logical gate operations</p>
<p>Logical qubits: 20 data + ~10 ancilla = 30 logical qubits</p>
<p>Physical qubits (surface code, d=15 at p=0.1%): 2×15² = 450 physical per logical × 30 = 13,500 physical qubits</p>
<p>This is well within the reach of near-future quantum hardware (IBM targets ~100,000 qubits by 2030). The Resource Estimator would also output the runtime: ~32,200 logical gates × 1μs/gate ≈ 32 ms.</p>
</div>

<div class="box box-solved-problem">
<p class="box-title"><strong>Solved Problem 10.8</strong></p>
<p>Problem: Write Q# code for a 2-qubit Grover’s search with oracle for target state |11⟩. Show: (a) the oracle operation, (b) the diffusion operator, (c) the full Grover iteration, (d) how to call it from Q#.</p>
</div>

### B. Unsolved Problems

Solve independently. Bracketed answers for self-checking.

1. Write a Cirq programme to build and simulate a 3-qubit Grover’s search circuit for target |101⟩. Use cirq.Simulator() to find the output probability distribution and verify that P(|101⟩) is close to 1. [Answer: 3-qubit Grover needs π√8/4 ≈ 2.2 iterations; use 2 iterations. Oracle flips phase of |101⟩. P(|101⟩) after 2 iterations ≈ 0.945 for n=3.]

2. Compare the Cirq Circuit and Qiskit QuantumCircuit representations of a 3-qubit QFT circuit. How many Moments does the Cirq circuit have? How many layers does the Qiskit circuit have after transpilation with optimization\_level=3? [Answer: QFT-3 has H+CP+SWAP gates. Cirq Moments ≈ 6 (H, CP, CP, H, CP, H+SWAP). Qiskit transpiled depth on Eagle ≈ 10–15 native layers due to CX decomposition of CP gates.]

3. Estimate the total cost on Amazon Braket to benchmark a 5-qubit VQE circuit using Rigetti Aspen-M: 20 optimiser iterations, 3 circuits per iteration, 1,000 shots each. [Answer: 20 × 3 = 60 tasks × $0.30 + 60,000 shots × $0.00035 = $18.00 + $21.00 = $39.00]

4. In Q#, what does the is Adj qualifier on an operation mean? Give an example of when the automatic adjoint generation is useful in a quantum algorithm. [Answer: is Adj means the compiler automatically generates the inverse (adjoint, or Hermitian conjugate) of the operation. Useful in Grover’s diffusion operator (needs H·X·…·X·H, which is the adjoint of the oracle preparation); in phase kickback circuits; and in quantum signal processing where circuit inversion appears repeatedly.]

5. Explain why Google Sycamore qubits are addressed as GridQubit(row, col) in Cirq rather than by integer index. What hardware property does this represent? [Answer: Sycamore uses a 2D grid coupling topology where two-qubit gates are only possible between nearest-neighbour qubits on the grid. The (row, col) addressing reflects the physical placement of qubits on the 2D chip. Cirq uses this to define which gate operations are physically valid (gates between non-adjacent GridQubits raise errors), forcing circuit designs to respect hardware connectivity.]

6. Write the OpenQASM 2.0 representation of the Toffoli (CCX) gate decomposition and show how to import it into both Qiskit and Cirq. [Answer: OpenQASM standard library qelib1.inc defines ccx directly. Qiskit: qc.ccx(0,1,2) then qc.qasm() exports ccx gate. Cirq: cirq.CCX or cirq.CCNOT; from\_qasm() handles ccx from qelib1.inc. The Toffoli has a 15-gate CX decomposition in native basis.]

7. The Azure Quantum Resource Estimator predicts that running Shor’s algorithm on RSA-1024 requires 3.7 million physical qubits at a physical error rate of 0.1%. IBM Condor has 1,121 physical qubits. How many IBM Condor-class processors would need to be linked to run this algorithm? [Answer: 3.7×10⁶ / 1,121 ≈ 3,300 processors. IBM’s modular architecture (Flamingo) plans to link processors via quantum interconnects; current inter-chip fidelity makes large-scale linking still a research challenge.]

8. Compare XEB fidelity with Quantum Volume as metrics. For what types of circuits is XEB more informative than QV, and vice versa? [Answer: XEB measures fidelity for random circuits of any depth, providing a continuous fidelity vs depth curve — better for characterising error rates per circuit layer. QV is a binary pass/fail on square random circuits — better as a single hardware quality number for comparison. XEB is more informative for deep circuit benchmarking; QV is better for hardware generation comparisons.]

9. Write Braket SDK code to compare the Bell state fidelity on LocalSimulator vs the IonQ simulator device (arn:aws:braket:us-east-1::device/quantum-simulator/ionq/ionq\_qpu). What errors would you expect to see, and why? [Answer: Code exercise. LocalSimulator is ideal (perfect fidelity). IonQ simulator on Braket adds realistic IonQ noise (T1≥8s, T2≥1s, 2Q fidelity 99.5%). Expected: Bell fidelity ≈ 0.995 on IonQ sim vs 1.000 on LocalSimulator. The difference comes from IonQ’s simulated gate errors.]

10. Explain the difference between Amazon Braket’s Task and Hybrid Job execution models. For a 100-iteration QAOA, which is preferable and why? Estimate the queue overhead for each approach. [Answer: Task: each circuit submitted independently to QPU queue (typically 1–12 hour window between submissions). 100 Tasks = 100 × waiting for execution window ≈ possible 100-day delay in worst case. Hybrid Job: runs in dedicated AWS environment, QPU calls have priority access within a single execution window. 100 Hybrid Job QPU calls ≈ single execution session. Hybrid Job strongly preferred for iterative algorithms.]

### C. Multiple Choice Questions

Circle the best answer. Answers at end of section.

**Q1. OpenQASM (Open Quantum Assembly Language) serves primarily as:**

- (a) A simulator for quantum circuits

- (b) A common intermediate representation enabling circuit exchange between frameworks

- (c) A hardware description language for qubit fabrication

- (d) An error correction protocol for quantum circuits

**Q2. In Cirq, a Moment represents:**

- (a) The time taken to execute one gate

- (b) A set of simultaneously executed operations on disjoint qubits

- (c) A single qubit measurement event

- (d) A noise channel applied after each gate

**Q3. Google Sycamore quantum hardware is accessible via Cirq to:**

- (a) All users with a Google account

- (b) Only Google Research Quantum AI partners (not publicly accessible)

- (c) Amazon Braket customers

- (d) IBM Quantum Network members

**Q4. In Amazon Braket, a Task refers to:**

- (a) The quantum hardware device (QPU or simulator)

- (b) A single circuit submission with specified shots and target device

- (c) The Python SDK package for building circuits

- (d) A classical post-processing job on AWS EC2

**Q5. Q# (Microsoft quantum language) differs from Qiskit/Cirq/Braket because:**

- (a) It only works on classical computers

- (b) It is a dedicated quantum programming language with quantum operations as first-class constructs

- (c) It does not support two-qubit gates

- (d) It requires CUDA GPU hardware to run

**Q6. The XEB (Cross-Entropy Benchmarking) fidelity F\_XEB = 0 implies:**

- (a) The circuit executed with perfect fidelity

- (b) The hardware output is completely random (pure noise)

- (c) The circuit depth is zero

- (d) The hardware has zero gate errors

**Q7. Amazon Braket QPU devices (IonQ, Rigetti) differ from IBM Quantum in that:**

- (a) They process jobs 24/7 in a continuous queue like IBM

- (b) They have scheduled execution windows; jobs queue until the next window opens

- (c) They are free to access with no charges

- (d) They only support single-qubit gates

**Q8. In Q#, the is Adj qualifier on an operation means:**

- (a) The operation is adjacent to another in the circuit

- (b) The compiler automatically generates the adjoint (inverse) of the operation

- (c) The operation uses adjacency-list qubit routing

- (d) The operation performs adjacent qubit SWAP gates

**Q9. Which Amazon Braket simulator supports noise (density matrix) simulation?**

- (a) SV1 (statevector simulator)

- (b) TN1 (tensor network)

- (c) DM1 (density matrix simulator)

- (d) Local Simulator (default mode)

**Q10. The Azure Quantum Resource Estimator is used to:**

- (a) Measure T1 and T2 on Azure Quantum hardware

- (b) Estimate physical/logical qubits and runtime for fault-tolerant algorithms without running on hardware

- (c) Calculate the financial cost of quantum jobs on Azure

- (d) Estimate the entropy of a quantum circuit output distribution

**Q11. Which of the following is true about Q# qubit allocation?**

- (a) The programmer assigns specific physical qubit indices (e.g., Q47, Q63)

- (b) Qubits are allocated with use q = Qubit() and the compiler assigns physical indices

- (c) Q# uses the same integer indexing as Qiskit

- (d) Q# only supports single-qubit operations, not multi-qubit circuits

**Q12. Braket Hybrid Jobs are preferred over individual Tasks for VQE because:**

- (a) Hybrid Jobs execute at lower cost per shot

- (b) Hybrid Jobs allow iterative QPU calls within a single execution session, avoiding repeated queue waits

- (c) Hybrid Jobs have larger shot limits per circuit

- (d) Hybrid Jobs run faster gates than standard Tasks

**Q13. The cirq.ConstantQubitNoiseModel(cirq.depolarize(p)) applies noise:**

- (a) Only to two-qubit gates

- (b) After every gate operation on every qubit

- (c) Only to the final measurement step

- (d) Only when using the statevector simulator (not DensityMatrixSimulator)

**Q14. OpenQASM 3.0 extends OpenQASM 2.0 primarily by adding:**

- (a) Support for quantum teleportation only

- (b) Classical control flow (if/for/while), real-time pulse control, and mid-circuit measurements

- (c) A graphical drag-and-drop circuit editor

- (d) Native support for 1000+ qubit circuits

**Q15. For a student in India working on an M.Sc. project with free access constraints, which platform is most appropriate?**

- (a) Amazon Braket (pay-per-shot)

- (b) Google Cirq (no public hardware access)

- (c) IBM Quantum Open Plan (free, 127-qubit Eagle processors, full Qiskit ecosystem)

- (d) Azure Quantum (requires Azure subscription)

<div class="box box-generic">
<p class="box-title"><strong>MCQ ANSWERS — CHAPTER 10</strong></p>
<p><strong>Q1: (b)   Q2: (b)   Q3: (b)   Q4: (b)   Q5: (b)</strong></p>
<p><strong>Q6: (b)   Q7: (b)   Q8: (b)   Q9: (c)   Q10: (b)</strong></p>
<p><strong>Q11: (b)  Q12: (b)  Q13: (b)  Q14: (b)  Q15: (c)</strong></p>
<p>Q2 Detail: A Moment in Cirq is an explicit parallel timestep — unlike Qiskit’s implicit parallelism. Q7 Detail: IonQ and Rigetti on Braket have scheduled windows (typically 2–6 hours/day); jobs submitted outside windows wait. Q9 Detail: SV1 is ideal statevector (no noise); DM1 supports density matrix noise; TN1 is tensor network. Q11 Detail: Q# uses high-level qubit allocation (use q = Qubit[n]()) — the QDK compiler handles physical mapping.</p>
</div>

### D. Theory Questions

- Explain the role of OpenQASM 2.0 as a cross-platform intermediate representation. Write the Bell state circuit in valid OpenQASM 2.0 syntax. What are the key additions in OpenQASM 3.0 that extend its capabilities beyond OpenQASM 2.0?

- Compare the Cirq Circuit/Moment model with the Qiskit QuantumCircuit/DAG model for representing quantum circuits. What are the advantages of Cirq’s explicit Moment structure for noise simulation and hardware-level programming? When is Qiskit’s implicit parallelism more convenient?

- Explain the Cross-Entropy Benchmarking (XEB) protocol used by Google to characterise quantum processor fidelity. How is F\_XEB computed? What does F\_XEB = 0.75 imply about the hardware error rate, assuming F\_XEB ≈ (1−2e)^d for a depth-12 circuit?

- Describe the Amazon Braket execution model: what is a Task, how do execution windows work for QPU devices, and what is a Hybrid Job? For a 200-iteration QAOA requiring 500 shots per circuit, compare the expected runtime (including queue waits) with and without Hybrid Jobs.

- What is Q# and how does it differ fundamentally from Qiskit, Cirq, and the Braket SDK? Explain the role of the is Adj qualifier, the use q = Qubit() allocation syntax, and the MResetZ measurement pattern. What advantage does Q# offer for expressing fault-tolerant algorithms?

- Explain the Azure Quantum Resource Estimator. What inputs does it require, what outputs does it produce, and why is it useful even when real fault-tolerant hardware does not yet exist? Use Grover’s algorithm on N=2^20 as an example.

- Compare the four quantum cloud platforms (IBM, Google, Amazon, Microsoft) in terms of: free access, hardware variety, SDK design philosophy, unique features, and appropriate use cases. Which would you recommend for each of the following users: (a) a student doing coursework, (b) a pharma company comparing hardware providers, (c) a Microsoft research partner?

- What is the Technology Readiness Level (TRL) scale? Assign approximate TRL values for: (a) quantum key distribution, (b) VQE for drug discovery, (c) Shor’s algorithm for RSA-2048, (d) quantum sensing with nitrogen-vacancy centres. Justify each assignment.

- Describe three specific career pathways in quantum computing that an M.Sc. Physics graduate could pursue in India in 2025. For each, list the key technical skills required, the types of organisations hiring, and the approximate salary range. How does the National Quantum Mission create new opportunities?

- Google Cirq’s ConstantQubitNoiseModel applies noise after every gate operation. How does this differ from Qiskit Aer’s NoiseModel, which assigns specific error channels to specific gate types? What are the advantages and disadvantages of each approach for modelling realistic hardware noise?

### E. Programming Assignments

PA10.1 — Cirq Full Simulation Suite. Install Cirq (pip install cirq) and implement a complete simulation programme: (a) Build a 4-qubit GHZ state circuit using cirq.GridQubit objects arranged in a 2×2 grid. (b) Simulate ideally with cirq.Simulator() and print the statevector; verify |0000⟩ and |1111⟩ have amplitude 1/√2. (c) Simulate with cirq.DensityMatrixSimulator() with 0.5% depolarising noise (cirq.ConstantQubitNoiseModel(cirq.depolarize(0.005))). Compute the GHZ fidelity F = ⟨GHZ|ρ|GHZ⟩ from the density matrix. (d) Sweep the depolarising error p from 0 to 0.05 in steps of 0.005 and plot GHZ fidelity vs p. (e) Export the circuit to OpenQASM 2.0 using cirq.qasm() and verify it is parseable by Qiskit (QuantumCircuit.from\_qasm\_str()). Submit: code, statevector output, density matrix fidelity, fidelity vs p plot, and QASM string.

PA10.2 — Amazon Braket Local Simulation. Using the Amazon Braket Local Simulator (no AWS account needed): (a) Install amazon-braket-sdk (pip install amazon-braket-sdk). (b) Build the following circuits: Bell state (2q), GHZ (3q), 2-qubit Grover (target |11⟩), QFT-3 (3-qubit QFT). (c) Run each on LocalSimulator with 4096 shots. (d) For each circuit, compute: success probability (fraction of shots in ideal target state), and compare with theoretical expectation. (e) Use the noise pragma (from braket.circuits import Circuit; circuit.circuit\_instructions) to add a simple depolarising noise channel and compare results. (f) Export each circuit to OpenQASM 3.0 using circuit.to\_openqasm() and verify each is syntactically valid. Submit: code, counts tables, fidelity comparison, and QASM outputs.

PA10.3 — Cross-Platform Bell State Comparison. Implement the Bell state experiment across all four frameworks and compare: (a) Build the Bell state (H+CNOT+measure) in Qiskit, Cirq, and Amazon Braket SDK. (b) Simulate each with their respective ideal simulators (Qiskit Aer statevector, cirq.Simulator, Braket LocalSimulator). (c) For each simulator, run 8192 shots and record counts. (d) Compute Bell fidelity (P(|00⟩) + P(|11⟩)) for each. (e) Convert each circuit to OpenQASM 2.0 and verify the QASM strings are equivalent. (f) (Optional, requires Azure account) Submit the Cirq Bell circuit to Azure Quantum IonQ simulator and compare results. Write a 1,000-word analysis comparing the SDK syntax, simulation results, QASM output quality, and any observed differences across platforms.

### F. Project Suggestions

Project 10.A — Comprehensive Cross-Platform VQE Study. Implement the Variational Quantum Eigensolver (VQE) for the H₂ molecule ground state energy across two platforms: (a) IBM Quantum (Qiskit): implement the full VQE pipeline from Chapter 9 (ansatz, EstimatorV2, COBYLA optimiser) on real IBM hardware. (b) Amazon Braket (IonQ Forte or Rigetti): implement the same VQE circuit using the Braket SDK and Hybrid Jobs on the IonQ simulator (no hardware required for this project). (c) Google Cirq: implement VQE using Cirq’s variational algorithms and DensityMatrixSimulator with a realistic noise model. (d) For each platform, report: the optimised ground state energy, convergence behaviour, circuit depth, and estimated/actual noise impact. (e) Compare the three implementations: code complexity, SDK expressiveness, simulation accuracy, and practical challenges. Write a 4,000-word comparative report covering: methodology, results, SDK comparison, noise analysis, and recommendations for future VQE implementations.

Project 10.B — OpenQASM Interoperability Pipeline. Build an automated cross-platform circuit translation and benchmarking pipeline: (a) Define a set of 10 test circuits of increasing complexity: Bell state (2q), GHZ-3, GHZ-5, QFT-3, QFT-5, Grover-2, Grover-3, random Clifford-5, parametric ansatz-4, teleportation circuit. (b) Implement each circuit in Qiskit, then export to OpenQASM 2.0. (c) Import each QASM circuit into Cirq and Braket SDK. Verify correctness by comparing simulation results across platforms. (d) For any circuits where translation fails or introduces errors, document the failure mode and propose a fix. (e) Measure translation fidelity: for each circuit, compute the TVD between Qiskit and Cirq simulation outputs. (f) Write a 3,000-word report on OpenQASM interoperability: which circuit constructs translate perfectly, which ones fail, and what this means for the standardisation of quantum software.

Project 10.C — India Quantum Computing Ecosystem Report. Conduct a survey of the quantum computing ecosystem relevant to Indian students and researchers: (a) Identify 10 Indian organisations (academic, government, private) working on quantum computing. For each: research focus, quantum platforms used (IBM/Cirq/Braket/Azure), open positions, and contact information. (b) Survey the IBM Quantum Network membership for Indian institutions: which IITs, IIScs, and national labs have Premium access? What are the terms? (c) Survey National Quantum Mission funding: which projects are funded, what platforms are specified, what are the deliverables by 2031? (d) Interview (or research published work from) at least two Indian quantum computing researchers; summarise their platform choices, research results, and advice for M.Sc. students. (e) Create a platform access guide specifically for students at Indian institutions: step-by-step instructions for IBM Quantum, Braket, and Cirq access with IST-specific queue timing advice. Write a 5,000-word report combining all of the above into a practical resource for Indian quantum computing students.

## References and Further Reading

1. Cirq Documentation (2024). https://quantumai.google/cirq — Official Cirq documentation including circuit model, noise channels, XEB, and Sycamore device specifications.

2. Arute, F. et al. (2019). Quantum supremacy using a programmable superconducting processor. Nature, 574, 505. [Google Sycamore quantum supremacy paper — original XEB methodology]

3. Google Quantum AI (2024). Quantum error correction below the surface code threshold. Nature, 614, 676. [Willow processor below-threshold QEC]

4. Amazon Braket Documentation (2024). https://docs.aws.amazon.com/braket — SDK reference, device documentation, pricing, and Hybrid Jobs.

5. Svore, K. et al. (2018). Q#: Enabling scalable quantum computing and development with a high-level DSL. Proceedings of the Real World Domain Specific Languages Workshop. [Q# language design and philosophy]

6. Microsoft Azure Quantum Documentation (2024). https://learn.microsoft.com/azure/quantum — Q# language reference, workspace setup, provider documentation, Resource Estimator.

7. Cross, A. M. et al. (2022). OpenQASM 3: A broader and deeper quantum assembly language. ACM Transactions on Quantum Computing, 3, 1. [OpenQASM 3.0 language specification]

8. Bergholm, V. et al. (2022). PennyLane: Automatic differentiation of hybrid quantum–classical computations. arXiv:1811.04968. [Cross-platform ML framework built on Cirq/Qiskit/Braket]

9. National Quantum Mission India (2023). https://dst.gov.in/national-quantum-mission — Official NQM programme document: targets, hubs, funding allocation.

10. IBM Quantum Documentation (2024). https://docs.quantum.ibm.com — Qiskit 1.x reference, Qiskit IBM Runtime, backend specifications.

### Course Conclusion: Quantum Computers

This concludes **Quantum Computers**. Across five units and ten chapters, you have built a complete foundation — from the mathematical language of qubits and quantum gates (Unit I), through the major quantum algorithms (Units II–III), the physics of hardware and noise (Unit IV), and the practice of accessing real quantum computers (Unit V). The field is moving rapidly: some of the specific API calls and hardware specifications in this textbook will have been superseded by the time you read it. What endures is the theoretical framework: the mathematics of quantum channels, the structure of quantum algorithms, the physics of coherence and decoherence, and the engineering principles of quantum error correction. These provide the foundation from which any practitioner can quickly learn new tools, new platforms, and new algorithms as they emerge.

The invitation of quantum computing is not merely technical — it is scientific. Every time you run a circuit on real hardware and see the gap between ideal simulation and noisy reality, you are observing quantum decoherence in action. Every time you implement a quantum algorithm and measure its performance, you are doing quantum information science. The boundary between textbook quantum mechanics and cutting-edge research has never been thinner. The quantum computers described in Chapter 9 are the same devices used to produce the results in Nature and Physical Review Letters. The code you have learned to write in this book is the same code used by researchers at IBM, Google, IonQ, and the Indian national laboratories.

**Go build something quantum** !!

**— Dr. S. K. Jain**
