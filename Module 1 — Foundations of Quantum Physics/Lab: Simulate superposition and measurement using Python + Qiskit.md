## 0. Prerequisites

* Python 3.9 or later installed
* Basic terminal / command line familiarity
* Internet connection to install packages

We’ll use:

* `qiskit` for building circuits
* `qiskit-aer` for simulation

(consistent with the current Qiskit docs). ([Qiskit][1])

---

## 1. Environment Setup

### Step 1.1 – Create a project folder

Pick a location and create a folder, e.g.:

```bash
mkdir quantum-superposition-lab
cd quantum-superposition-lab
```

(Optional but recommended) Create a virtual environment:

```bash
python -m venv .venv
# Activate it:
# Windows:
.venv\Scripts\activate
# macOS / Linux:
source .venv/bin/activate
```

### Step 1.2 – Install Qiskit and Aer

In that folder / virtual environment:

```bash
pip install "qiskit[all]" qiskit-aer
```

Check the install:

```bash
python -c "import qiskit, qiskit_aer; print(qiskit.__qiskit_version__)"
```

You should see a dict of version numbers printed.

---

## 2. First Circuit: Superposition of |0⟩ and |1⟩

We will:

* Create a 1-qubit circuit
* Apply a Hadamard gate H to create superposition
* Measure the qubit
* Run many shots on a simulator and look at the statistics

### Step 2.1 – Create a Python file

Create `superposition_lab.py` and add the following code:

```python
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit.visualization import plot_histogram

# 1. Build a quantum circuit with 1 qubit and 1 classical bit
qc = QuantumCircuit(1, 1)

# 2. Put the qubit into superposition using a Hadamard gate
qc.h(0)

# 3. Measure the qubit into the classical bit
qc.measure(0, 0)

# 4. Print the circuit diagram
print("Quantum circuit:")
print(qc)

# 5. Choose a simulator backend
simulator = AerSimulator()

# 6. Transpile (compile) the circuit for the simulator
compiled_circuit = transpile(qc, simulator)

# 7. Run the circuit many times (shots) to collect statistics
job = simulator.run(compiled_circuit, shots=1000)

# 8. Get the result
result = job.result()
counts = result.get_counts()

print("\nMeasurement counts (1000 shots):")
print(counts)

# 9. Optionally, show a histogram (if running in a notebook or environment that can show plots)
try:
    histogram = plot_histogram(counts)
    # In a script, you need to explicitly show the plot
    import matplotlib.pyplot as plt
    plt.show()
except Exception as e:
    print("\nCould not display histogram (no GUI backend). Error:", e)
```

### Step 2.2 – Run the script

From the project folder:

```bash
python superposition_lab.py
```

You should see:

* An ASCII drawing of the circuit
* A counts dictionary, roughly like:

```text
Measurement counts (1000 shots):
{'0': 503, '1': 497}
```

The exact numbers will vary, but they should be close to 50/50.

---

## 3. Understanding What Just Happened

Line by line:

1. `qc = QuantumCircuit(1, 1)`

   * 1 quantum bit (qubit), 1 classical bit for measurement.

2. `qc.h(0)`

   * Applies the Hadamard gate to qubit 0:

     * Input: |0⟩
     * Output: (|0⟩ + |1⟩) / √2
   * This is a perfect equal superposition.

3. `qc.measure(0, 0)`

   * Measures the qubit in the computational (Z) basis.
   * Collapses the state to |0⟩ or |1⟩.
   * Result is stored in classical bit 0.

4. `AerSimulator()`

   * High-performance simulator backend from Qiskit Aer. ([Qiskit][1])

5. `simulator.run(..., shots=1000)`

   * Executes the circuit 1000 times and returns the measurement statistics.

The key concept:
Before measurement, the qubit is in superposition. After measurement, we only see classical outcomes, but the frequencies reflect the underlying quantum probabilities.

---

## 4. Variation 1: Start from |1⟩ Instead of |0⟩

Now let’s:

* Flip the qubit to |1⟩
* Then apply H
* See what happens

Modify the circuit part of your script:

```python
qc = QuantumCircuit(1, 1)

# Flip |0> to |1>
qc.x(0)

# Now apply Hadamard
qc.h(0)

qc.measure(0, 0)
```

Mathematically:

* Start: |0⟩
* After X gate: |1⟩
* After H: (|0⟩ − |1⟩) / √2

Run again:

```bash
python superposition_lab.py
```

You should still see roughly a 50/50 split between 0 and 1.
The difference here is in the phase (the minus sign), which does not show up in simple Z-basis measurement, but matters in interference when you add more gates.

This is a great talking point:

* Same measurement statistics
* Different underlying state (phase difference)

---

## 5. Variation 2: Compare “No Superposition” vs “Superposition”

To make superposition concrete, you can have students compare two circuits:

### Circuit A: No H gate

```python
qc = QuantumCircuit(1, 1)
qc.measure(0, 0)
```

Expected: almost all shots should return `0` (state |0⟩ is default).

### Circuit B: With H gate

```python
qc = QuantumCircuit(1, 1)
qc.h(0)
qc.measure(0, 0)
```

Expected: roughly 50% `0`, 50% `1`.

You can even put this in one script as two separate circuits:

```python
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator

simulator = AerSimulator()

# Circuit A: no superposition
qc_a = QuantumCircuit(1, 1)
qc_a.measure(0, 0)

compiled_a = transpile(qc_a, simulator)
job_a = simulator.run(compiled_a, shots=1000)
counts_a = job_a.result().get_counts()

print("Circuit A (no H gate) counts:", counts_a)

# Circuit B: with superposition
qc_b = QuantumCircuit(1, 1)
qc_b.h(0)
qc_b.measure(0, 0)

compiled_b = transpile(qc_b, simulator)
job_b = simulator.run(compiled_b, shots=1000)
counts_b = job_b.result().get_counts()

print("Circuit B (with H gate) counts:", counts_b)
```

Typical output:

```text
Circuit A (no H gate) counts: {'0': 1000}
Circuit B (with H gate) counts: {'0': 498, '1': 502}
```

Perfect for demonstrating that H really changes the quantum state.

---

## 6. Optional Extension: Two-Qubit Superposition

To bridge toward entanglement later, you can create a simple 2-qubit circuit:

```python
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit.visualization import plot_histogram
import matplotlib.pyplot as plt

qc = QuantumCircuit(2, 2)

# Put both qubits in superposition
qc.h(0)
qc.h(1)

qc.measure([0, 1], [0, 1])

print(qc)

simulator = AerSimulator()
compiled = transpile(qc, simulator)
job = simulator.run(compiled, shots=1000)
counts = job.result().get_counts()

print("Counts:", counts)
plot_histogram(counts)
plt.show()
```

Expected: the four outcomes `00`, `01`, `10`, `11` should appear with roughly equal probability (~25% each).

This demonstrates that:

* Superposition on each qubit multiplies into a superposition over all combinations.
* With 2 qubits, we have 4 basis states occupied; with n qubits, 2^n.


