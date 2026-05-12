# QAOA for Max-Cut on NISQ Devices

A systematic study of the Quantum Approximate Optimization Algorithm (QAOA) applied to the Max-Cut problem, comparing performance across noiseless simulation, noisy simulation, and IBM Quantum hardware.

## Overview

We evaluate how effectively QAOA approximates optimal Max-Cut solutions on small graphs (4–8 nodes) and analyze how quantum noise impacts performance. Experiments sweep circuit depths p ∈ {1, 2, 4, 8, 16} across three backends:

- **Noiseless simulator** — ideal quantum execution via `StatevectorSampler`
- **Noisy simulator** — custom noise model (depolarizing + thermal relaxation + readout error) via `AerSimulator`
- **IBM Quantum hardware** — real device execution via `qiskit-ibm-runtime`

### Key Findings

- Noiseless QAOA achieves approximation ratio ≈ 1.0 across all graph types
- Noise degrades average solution quality but best samples often remain near-optimal
- Optimal NISQ depth is p ≈ 2–4; deeper circuits degrade under noise

## Setup

### Prerequisites

- Python 3.10+
- IBM Quantum account (for hardware runs)

### Installation

```bash
git clone https://github.com/<your-username>/qaoa-maxcut-nisq.git
cd qaoa-maxcut-nisq
pip install -r requirements.txt
```

### IBM Quantum (optional, for hardware runs)

```python
from qiskit_ibm_runtime import QiskitRuntimeService
QiskitRuntimeService.save_account(channel="ibm_quantum", token="YOUR_TOKEN")
```

## Usage

### Run QAOA on a single graph

```python
from src.qaoa import build_example_graph_4, build_maxcut_hamiltonian, optimize_qaoa_parameters

G = build_example_graph_4()
hamiltonian = build_maxcut_hamiltonian(G)
result = optimize_qaoa_parameters(hamiltonian, depth=2)
```

### Run full experiment suite

```python
from src.qaoa4 import run_full_experiment

run_full_experiment(
    depths=[1, 2, 4, 8, 16],
    settings=["noiseless", "noisy", "hardware"]
)
```

## Noise Model

Custom noise model parameters (representative of IBM superconducting qubits):

| Parameter | Value | Description |
|-----------|-------|-------------|
| 1Q depolarizing | 0.1% | Per single-qubit gate |
| 2Q depolarizing | 1.0% | Per CX gate |
| T₁ | 50 μs | Energy relaxation time |
| T₂ | 70 μs | Dephasing time |
| 1Q gate time | 50 ns | Single-qubit gate duration |
| 2Q gate time | 300 ns | CX gate duration |
| Readout error | 2.0% | Symmetric bit-flip on measurement |

## Benchmark Suite

22 graphs across 5 families:

- **Complete** (K₄–K₈) — dense, symmetric
- **Cycle** (4–8 nodes) — sparse, regular
- **Regular** (6, 8 nodes, 3-regular) — balanced connectivity
- **Random** (4–8 nodes, Erdős–Rényi p=0.7) — irregular
- **Weighted** (5, 6, 8 nodes) — non-uniform edge importance
