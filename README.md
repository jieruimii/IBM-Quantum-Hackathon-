# Rescuing Quantum Spin Dynamics with Zero-Noise Extrapolation

**Quantum circuits produce results. Error mitigation improves estimates. Evidence determines whether those estimates deserve trust.**

A Qiskit-based investigation of how simulated gate noise distorts the magnetization dynamics of a four-spin transverse-field Ising model (TFIM), and whether zero-noise extrapolation can recover a more reliable signal. The project compares exact classical evolution, ideal Trotterized quantum evolution, noisy shot-based simulation, and a mitigated result while reporting both accuracy gains and computational cost.

Developed for **Qompute 2026 - Track 3: Can We Trust a Noisy Quantum Computer?**

## Goal

The project asks a focused question:

> How strongly does gate noise distort a small quantum simulation, and how much accuracy can zero-noise extrapolation recover relative to its additional sampling cost?

To answer it, the notebook establishes a complete evidence chain:

```text
Exact classical reference
        -> Ideal Qiskit Trotter circuit
        -> Noisy simulated execution
        -> Zero-noise extrapolation
        -> Accuracy and resource comparison
```

The objective is not simply to make the mitigated curve look better. It is to determine, quantitatively and transparently, whether the improvement is meaningful and what must be paid to obtain it.

## Why this topic matters

Quantum simulation is a promising application of quantum computing because molecules, materials, and many-body systems follow quantum-mechanical rules. In the future, more capable simulations could support research in batteries, catalysts, medicines, fertilizers, and energy systems.

Current quantum processors remain noisy. Every imperfect gate can move a computation away from its intended state, and errors accumulate as circuits become deeper. A visually plausible result may therefore be scientifically wrong. This creates a trust problem: before organizations use quantum outputs in research or decision-making, they need evidence that those outputs are stable, reproducible, and meaningfully better than an unmitigated baseline.

This project makes that trust problem measurable. Because the four-qubit system is small enough to solve exactly on a classical computer, the exact trajectory provides a known reference against which both noise and mitigation can be evaluated.

## Experiment

The notebook simulates an open four-spin transverse-field Ising chain:

$$
H=-J\sum_{i=0}^{N-2} Z_iZ_{i+1}-h\sum_{i=0}^{N-1}X_i,
$$

with $N=4$, $J=1$, $h=1$, a Trotter step size of $\Delta t=0.2$, and 31 observation points from $t=0$ through $t=6$.

Four trajectories are compared:

1. **Exact classical evolution** using matrix exponentiation.
2. **Ideal Trotter evolution** using Qiskit's `PauliEvolutionGate` and first-order `LieTrotter` synthesis.
3. **Noisy evolution** using Qiskit Aer with 0.1% single-qubit and 1.0% two-qubit depolarizing error probabilities.
4. **Mitigated evolution** using linear zero-noise extrapolation across noise scale factors 1, 2, and 3.

The primary observable is average magnetization along the $z$ direction:

$$
M_z(t)=\frac{1}{N}\sum_{i=0}^{N-1}\langle Z_i\rangle.
$$

## Results

| Comparison | RMSE |
| --- | ---: |
| Ideal Trotter vs. exact evolution | 0.031859 |
| Noisy vs. ideal Trotter | 0.237196 |
| Zero-noise extrapolation vs. ideal Trotter | 0.204861 |
| Noisy vs. exact evolution | 0.239799 |
| Zero-noise extrapolation vs. exact evolution | 0.206939 |

Zero-noise extrapolation reduced RMSE relative to the ideal Trotter trajectory by **13.63%**. The mitigation recovered part of the signal, particularly in the deeper portion of the evolution, but it did not restore ideal dynamics.

The improvement also required substantially more computation:

| Resource | Noisy baseline | Zero-noise extrapolation |
| --- | ---: | ---: |
| Shots per time point | 4,096 | 12,288 |
| Total shots | 126,976 | 380,928 |
| Sampling overhead | 1x | 3x |

The final transpiled circuit had a depth of **334** and contained **180 CX gates**. The growing error at later evolution times is consistent with noise accumulating as circuit depth and entangling-gate count increase.

The result is deliberately modest: a 13.63% improvement at three times the sampling cost. That tradeoff is more informative than claiming that mitigation simply "solves" noise.

## Figures

### Exact and ideal Trotter evolution

![Exact classical evolution compared with ideal Trotter evolution](figures/ideal_vs_exact.png)

### Noisy and mitigated evolution

![Noisy TFIM evolution compared with zero-noise extrapolation](figures/noisy_vs_mitigated.png)

### Circuit cost

![Circuit depth and CX gate count across evolution time](figures/circuit_cost.png)

## Run

Python **3.11-3.14** is supported. Python 3.11 is the conservative compatibility choice.

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install qiskit qiskit-aer numpy scipy matplotlib pylatexenc jupyter
jupyter lab
```

Open `IBM_Hackathon_Project_3_RECOVERED.ipynb`, select the environment created above, restart the kernel, and choose **Run All**.

No IBM Quantum account, API key, or real-hardware access is required. The notebook uses local statevector and noisy Aer simulations. Each run recreates the `figures/` directory and the three PNG outputs.

## Reproducibility checks

A successful full run should report:

```text
Ideal Trotter vs exact: 0.031859
Noisy vs ideal Trotter: 0.237196
ZNE vs ideal Trotter: 0.204861
Noisy vs exact: 0.239799
ZNE vs exact: 0.206939
ZNE improvement (%): 13.632307

Final circuit depth: 334
Final CX gate count: 180
Baseline shots per time point: 4096
ZNE shots per time point: 12288
Total baseline shots: 126976
Total ZNE shots: 380928
Sampling overhead: 3.0x
```

The simulation uses fixed transpiler and simulator seeds, so equivalent Qiskit versions should reproduce the recorded results. Small differences may appear across future Qiskit releases if transpilation behavior changes.

## Business implications

### 1. Quantum value must be evaluated as an accuracy-cost tradeoff

Mitigation consumes additional shots, runtime, queue capacity, and potentially money. A business case should therefore measure value per unit of verified accuracy rather than celebrate accuracy improvement in isolation. In this experiment, a 13.63% RMSE improvement costs three times as many shots.

### 2. Verification should be part of quantum procurement

Organizations purchasing quantum-computing services need benchmarks that separate algorithmic approximation error from hardware noise. Vendor evaluations and service-level objectives could include reference problems, uncertainty reporting, calibration context, mitigation overhead, and reproducibility requirements.

### 3. Trust infrastructure may be as valuable as raw hardware access

Enterprises in pharmaceuticals, chemicals, finance, energy, and advanced manufacturing will need tooling that records circuit versions, transpilation decisions, noise assumptions, mitigation settings, resource usage, and comparisons against classical baselines. Auditability can become a product capability rather than an afterthought.

### 4. Not every workload should be moved to quantum hardware

The four-qubit TFIM is easier and more accurate to solve classically. That is a feature of this benchmark, not a failure: it provides ground truth. Commercial teams should define explicit escalation criteria for when quantum execution adds scientific or economic value and when classical computation remains the more responsible choice.

### 5. Mitigation cost affects access and competition

If trustworthy results require several times more executions, well-funded organizations can afford more validation than smaller laboratories, startups, or public institutions. Cloud access alone does not guarantee equal access to reliable results. Pricing, research credits, and shared verification standards can influence who benefits from quantum progress.

### 6. High-stakes adoption requires calibrated claims

In medicine, financial risk, security, and infrastructure, a mitigated result should not be treated as automatically correct. Decision-makers need confidence intervals, sensitivity analysis, independent checks, and clear boundaries on how the output may be used.

## Boundaries and limitations

- The experiment uses a simulated depolarizing noise model, not a physical IBM Quantum processor.
- Real hardware includes qubit-specific errors, coherent errors, crosstalk, connectivity constraints, and calibration drift that are not represented here.
- Noise is amplified by changing simulator error probabilities. A hardware implementation would need an operational technique such as gate folding.
- Linear extrapolation may be inaccurate when the observable depends nonlinearly on noise strength.
- Finite-shot sampling introduces uncertainty; this prototype does not calculate confidence intervals for individual magnetization estimates.
- Zero-noise extrapolation addresses simulated hardware noise but does not remove Trotter approximation error.
- The four-qubit system is classically tractable and does not demonstrate quantum advantage.
- A 13.63% average RMSE improvement does not guarantee improvement at every time point, on another circuit, or under another noise model.

## Future work

- Add bootstrap confidence intervals to each magnetization estimate and to the reported RMSE improvement.
- Compare linear and higher-order extrapolation models.
- Implement gate folding so that noise scaling corresponds more closely to a deployable hardware workflow.
- Test measurement mitigation, Pauli twirling, and dynamical decoupling against the same baseline.
- Compare first-order Lie-Trotter and higher-order Suzuki-Trotter formulas at matched depth and accuracy.
- Run a small version on IBM Quantum hardware and connect observed behavior to device calibration data.
- Extend the experiment to correlations such as $\langle Z_iZ_j\rangle$ and larger spin systems where classical verification becomes more difficult.

## Project structure

```text
.
├── IBM_Hackathon_Project_3_RECOVERED.ipynb
├── README.md
└── figures/
    ├── ideal_vs_exact.png
    ├── noisy_vs_mitigated.png
    └── circuit_cost.png
```

## Attribution

This project extends:

> B. Tirado, *Simulating Quantum Dynamics: The Transverse-Field Ising Model*, BasQ Qiskit Fall Fest 2026 tutorial. Tutorial text and figures are licensed under CC BY 4.0; tutorial code is licensed under Apache License 2.0.

Additional references include Qiskit documentation for product-formula synthesis, primitives, transpilation, and Aer noise simulation; Smith, Kim, Pollmann, and Knolle, *Simulating quantum many-body dynamics on a current digital quantum computer* (2019); and Kim et al., *Evidence for the utility of quantum computing before fault tolerance* (2023).

## Core takeaway

**A mitigated quantum result is not automatically a trustworthy result. Trust comes from comparison, reproducibility, uncertainty, transparent cost, and honest limits.**
