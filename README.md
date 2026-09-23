# ⚛️ Quantum OS Kernel & Simulator Engine

A full-stack, high-performance **Quantum Operating System Kernel and Gate-Based Simulator Engine** built from scratch in Rust.

This project bridges the gap between quantum physics and operating system architecture. It features a complete runtime pipeline: from OpenQASM parsing and Intermediate Representation (QIR) optimization to hardware pulse level simulation, error correction (ECC), quantum memory management (QMMU), and multi-tenant cloud security.

---

## 🔥 Key Features

### 💻 Quantum OS Kernel & Runtime Architecture
* **QMMU (Quantum Memory Management Unit):** Virtual memory manager handling quantum process swapping to disk (`.qswap`), state preservation, and page tables[cite: 1].
* **QIR Optimization Engine:** Gate fusion, cancellation, and depth reduction pipeline (achieving up to 80% instruction reduction)[cite: 1].
* **Quantum IPC & Teleportation:** Inter-Process Communication using quantum entanglement protocols to transfer states across process boundaries[cite: 1].
* **Context Switching & Scheduler:** Quantum state preservation and restoration during multi-process context switches[cite: 1].
* **Interactive CLI Shell (`qos_kernel>`):** Real-time command-line interface for executing programs, monitoring RAM usage, and running kernel benchmarks[cite: 2].

### 🔬 Multi-Engine Simulation Capabilities
* **State Vector Engine:** Exact $2^N$ state vector simulator parallelized with Rayon[cite: 2].
* **Gottesman-Knill Stabilizer Engine:** Clifford+T tableau simulator capable of scaling to 1,000+ qubits for non-universal operations.
* **Tensor Network Engines:** MPS (Matrix Product States) and 2D PEPS tensor contraction pipelines for low-entanglement large-scale circuits.
* **Dynamic Mid-Circuit Control Flow:** Real-time mid-circuit measurements with nanosecond feedback loops.

### 🛡️ Quantum Error Correction & Hardware Physics
* **Active QEC Systems:** 3-Qubit Bit-Flip Error Correction and Surface Code 17 (MWPM decoder)[cite: 1].
* **Zero-Noise Extrapolation (ZNE):** Mitigation technique for NISQ-era quantum hardware errors.
* **Pulse-Level HAL (Hardware Abstraction Layer):** Transmon qubit pulse simulation featuring Gaussian envelopes, DRAG (Derivative Removal by Adiabatic Gate) compensation, and Rabi oscillation calibrations.
* **Cryo-Thermal Drift Feedback:** Real-time pulse frequency adjustment loop compensating for sub-milliKelvin refrigerator temperature shifts.

### ☁️ Enterprise & Infrastructure Layer
* **Multi-Tenant Security:** Hardware quota enforcement and cross-tenant quantum memory sandbox isolation.
* **Distributed GPU Orchestrator:** Topology planner for splitting large state-vectors across MPI/NCCL multi-GPU clusters.

---

## 📊 Kernel Performance Benchmark

Benchmarks executed on the State Vector engine (Windows x86_64, Release Mode):

| Qubits ($N$) | Hilbert States ($2^N$) | RAM Usage | Execution Time |
| :---: | :---: | :---: | :---: |
| **2** | 4 | 0.06 KB | 41.8 µs |
| **4** | 16 | 0.25 KB | 75.9 µs |
| **8** | 256 | 4.00 KB | 303.5 µs |
| **10** | 1,024 | 16.00 KB | 900.0 µs |
| **12** | 4,096 | 64.00 KB | 2.18 ms |
| **14** | 16,384 | 256.00 KB | 6.26 ms |

---

## 🛠️ Getting Started

### Prerequisites
* [Rust](https://www.rust-lang.org/) (Edition 2024)
* `cargo` package manager

### Installation
Clone the repository:
```bash

cd quantum_os

```

---

Running the OS

To run the full diagnostic boot sequence, benchmarks, and interactive shell in debug mode:

cargo run

---

To run with maximum release optimizations (recommended for high-qubit counts):

cargo build --release
./target/release/quantum_os

---

## 60-second stress test

Run the sustained state-vector workload directly, without the diagnostic boot sequence:

```bash
cargo run --release -- --stress
```

The defaults are 60 seconds and 18 qubits. Override either value with
`--stress <seconds> <qubits>` (2–22 qubits), for example:

```bash
cargo run --release -- --stress 10 20
```

From the interactive `qos_kernel>` shell, run the same workload with:

```text
stress [seconds] [qubits]
```

---

## How to demo it

Here are the best things to try. The shell starts with **8 simulated qubits**, numbered `0` through `7`.

### 1. Basic “hello quantum world”

Start with one qubit:

```text
qos_kernel> run h 0
qos_kernel> run measure 0
```

Run the whole program again several times. After `h 0`, qubit 0 is supposed to be 50/50, so measurement should randomly give:

```text
[(0, 0)]
```

or:

```text
[(0, 1)]
```

Important: after you measure it, the state collapses. So doing this:

```text
run h 0
run measure 0
run measure 0
run measure 0
```

should give the **same answer repeatedly after the first measurement**.

---

### 2. Make two qubits entangled

This is the classic Bell pair:

```text
qos_kernel> run h 0
qos_kernel> run cnot 0 1
qos_kernel> run measure 0
qos_kernel> run measure 1
```

The interesting result is that you should get either:

```text
0
0
```

or:

```text
1
1
```

rather than independent random answers.

Restart the program and repeat it a bunch of times. You should see:

```text
00
11
00
00
11
11
...
```

but not normally:

```text
01
10
```

That's probably the coolest thing the little interactive shell can demonstrate.

---

### 3. Compare that with two independent random qubits

Restart, then:

```text
run h 0
run h 1
run measure 0
run measure 1
```

Now they're **not entangled**.

You can get all four combinations:

```text
00
01
10
11
```

That's a nice comparison:

```text
h 0
cnot 0 1
```

means correlated quantum pair.

Whereas:

```text
h 0
h 1
```

means two independent 50/50 qubits.

---

### 4. Play with its pretend OS scheduler

```text
qos_kernel> status
qos_kernel> alloc 100 2
qos_kernel> status
```

You'll see something like:

```text
Active processes: 1
 - PID 100: Allocated qubits [0, 1]
```

Then:

```text
alloc 200 3
status
```

You might get:

```text
PID 100: [0, 1]
PID 200: [2, 3, 4]
```

So now you've simulated two "quantum processes":

```text
PID 100 → q0 q1
PID 200 → q2 q3 q4
free    → q5 q6 q7
```

Try filling the machine:

```text
alloc 300 3
status
```

Now all eight have been allocated.

Then:

```text
alloc 400 1
```

should produce:

```text
Allocation failed: Not enough free quantum resources!
```

That's the "OS" part of the demo.

---

### 5. There's actually a scheduler bug you can discover

Restart first.

Do:

```text
alloc 100 2
status
```

Then **allocate to PID 100 again**:

```text
alloc 100 2
status
```

You'll likely see PID 100 change from:

```text
[0, 1]
```

to:

```text
[2, 3]
```

But qubits 0 and 1 weren't actually freed internally.

Do it repeatedly:

```text
alloc 100 2
alloc 100 2
alloc 100 2
alloc 100 2
alloc 100 1
```

Eventually you'll run out of qubits even though `status` appears to show only one process owning two of them.

That's a **real bug in the repo**: inserting the same PID replaces its entry in the `HashMap` without releasing its previous allocation.

---

### 6. There's an even nastier allocation bug

Do this only when you're happy to restart the program afterward:

```text
alloc 123 100
```

There are only 8 qubits, so you'd expect it to simply say "no."

It does return an error—but while trying, it marks the available qubits as occupied and **doesn't roll that operation back**.

So afterward try:

```text
alloc 1 1
```

It may fail too.

That's essentially:

> "I tried to reserve 100 hotel rooms, the hotel only had 8, the reservation failed... but somehow all 8 rooms still got marked occupied."

A good first GitHub issue/fix for this project.

---

### 7. Experiment with CNOT

CNOT means roughly:

> "If the control qubit is 1, flip the target qubit."

There isn't an `x` command for easily setting a qubit to 1, unfortunately, so the easiest way to see it is probabilistically:

```text
run h 0
run cnot 0 1
run measure 0
run measure 1
```

Again, that's why the outputs correlate.

Try reversing it:

```text
run h 0
run cnot 1 0
run measure 0
run measure 1
```

Since qubit 1 starts as `0`, that CNOT doesn't create the same entanglement. Qubit 0 will be random while qubit 1 remains 0.

So you'll see roughly:

```text
00
10
00
10
```

rather than only `00`/`11`.

---

### 8. Try a three-qubit GHZ state

This is the natural extension of the Bell pair:

```text
run h 0
run cnot 0 1
run cnot 1 2

run measure 0
run measure 1
run measure 2
```

You should get:

```text
000
```

or:

```text
111
```

That's a three-qubit entangled state.

Try four:

```text
run h 0
run cnot 0 1
run cnot 1 2
run cnot 2 3

run measure 0
run measure 1
run measure 2
run measure 3
```

Expected:

```text
0000
```

or:

```text
1111
```

This is probably the most satisfying demo available through the current shell.

---

### One thing to understand about `alloc`

There's currently a big hole in the "OS" illusion.

If you do:

```text
alloc 100 2
```

and it assigns:

```text
PID 100 → [0, 1]
```

nothing prevents you from doing:

```text
run h 7
```

The `run` command **isn't associated with a PID** and doesn't check ownership.

So today:

```text
alloc
```

is basically bookkeeping.

A much more interesting next version of this CLI would be:

```text
alloc 100 2
run 100 h 0
run 100 cnot 0 1
measure 100 0
```

where PID 100 sees its own **virtual qubits 0 and 1**, and the kernel maps those onto physical qubits. Then you'd actually start getting something resembling a tiny quantum OS.

**I'd play with the GHZ state first, then deliberately break the allocator.** Those two experiments show both the genuinely neat quantum-simulator part and the immature "OS kernel" part.
