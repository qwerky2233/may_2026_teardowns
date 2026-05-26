# Quantum Paper Teardown

## Thesis
Two hybrid quantum-classical models — a Kernelized Quantum Reservoir with Repeated Measurement (KQRC-RM) and a Projected Quantum Kernel Gaussian Process (QGP) — can perform multi-output energy time-series forecasting at the 100+ qubit scale on real NISQ hardware, outperforming classical analogues on a small smart meter dataset.

## Verdict
The paper succeeds as an engineering feasibility demonstration on 100+ qubits, but it does not establish QML advantage: the "improvement" over classical baselines is measured against weak, non-competitive comparators, on a tiny dataset with a favorable correlation structure, with training done entirely on a simulator, and the hardware results are substantially degraded — making this a careful empirical study, not a capabilities milestone.

## Tags
`quantum-machine-learning`, `quantum-reservoir-computing`, `quantum-kernel-methods`, `NISQ`, `time-series-forecasting`, `hybrid-classical-quantum`, `energy-systems`

---

## Paper Info
- **arXiv link:** https://arxiv.org/abs/2605.24252
- **Date:** 22 May 2026
- **Title:** Hybrid Quantum-Classical Machine Learning Algorithms for Multi-Output Time-Series Forecasting at Utility Scale
- **Authors:** Mackenson Polché, Varun Puram, Aditi Lal, Weronika Golletz, Joan Etude Arrow, Vardaan Sahgal, Kumar Ghosh, Giorgio Cortiana, Corey O'Meara

---

## What the Paper Claims

The paper presents two hybrid QML frameworks for multi-stream electricity demand forecasting and evaluates them on IBM's 156-qubit `ibm_marrakesh` superconducting processor. KQRC-RM achieves a 36.92% MAE reduction over its classical analogue (Echo State Network with kernel ridge regression) on a simulator but degrades to 107% *worse* than that baseline on hardware. QGP achieves 62.01% MAE reduction over a classical multi-output GP on simulator and 40.37% on hardware, with 49% of outputs in a low-error regime in a 100-qubit run. The paper frames these as evidence of "feasibility" for quantum-assisted forecasting at utility scale.

## Mechanism in Plain Language

**KQRC-RM:** Each of three customer time-series is mapped to its own register of 15 system qubits. Input values are encoded as single-qubit rotations; CNOT gates entangle neighboring registers to propagate cross-stream information; ancilla qubits are measured at each timestep (and reset) to extract probability distributions without collapsing system memory. These probability vectors build a feature matrix, on which a classical RBF kernel ridge regression is trained. The quantum circuit acts as a fixed, non-trained nonlinear dynamics reservoir — only the readout layer is learned.

**QGP:** Each customer maps to a single qubit in a hardware-efficient variational circuit (Hadamard init → R_Y encoding → CNOT chain → R_X re-uploading → trainable R_Y). Instead of comparing full quantum state fidelities (which requires exponentially deep compute–uncompute circuits), similarity between inputs is computed from 2-qubit reduced density matrices (projected kernel). This projected kernel replaces the covariance function of a classical Gaussian Process, giving probabilistic multi-output predictions with uncertainty bands. Circuit parameters are trained by maximizing the GP marginal log-likelihood — on a simulator only — then the fixed parameters are transferred to hardware for inference.

The theoretical framing uses the Huang et al. (2021) geometric-difference / model-complexity diagnostic to argue the smart meter data lies in a regime where quantum kernels *may* provide advantage. This is a theoretical precondition check, not a proof.

## What Matters Practically

The most practically important result is the 100-qubit QGP experiment: a topology-aware shallow ansatz (circuit depth 19, two-qubit depth 4) produces useful predictions for 80% of customers on real hardware, with the 20% failure rate traceable directly to qubit-quality heterogeneity on the device rather than to the model. This is a concrete operational constraint — prediction quality is gated on qubit T2 time and CZ gate error, which means hardware-aware qubit selection and transpilation are first-class concerns, not optimizations. The KQRC-RM result at 114 qubits is less practically encouraging: hardware performance degrades below the naïve persistence baseline, and the resource cost (30 qubits per stream) scales poorly.

For hybrid quantum-classical AI strategy, the QGP architecture's linear qubit scaling with outputs — versus KQRC-RM's linear scaling with both streams and time — is a material architectural difference that should guide near-term deployment scoping.

## Likely Misinterpretation

The framing "reduces average MAE by 62% relative to classical GP baseline" will be read as quantum models outperforming classical forecasting. It does not: the classical baseline is a standard multi-output GP with an RBF kernel, not a competitive modern time-series model. Against ARIMA, N-BEATS, Temporal Fusion Transformer, or even a well-tuned linear model on this small dataset, the quantum advantage narrative likely collapses. The paper also trains QGP entirely on a simulator and transfers frozen parameters to hardware — this sidesteps trainability costs on real hardware entirely and should not be confused with end-to-end quantum training. The phrase "utility scale" in the title refers only to the number of qubits and customers in a single batch, not to production-grade forecasting infrastructure.

## Bottom Line

Track QGP-style projected-kernel architectures as the more viable near-term QML pattern — shallow circuits, linear qubit scaling, Bayesian uncertainty outputs, and noise tolerance through local reduced-state statistics. Do not interpret the benchmark numbers as evidence of practical forecasting advantage. The next material test of this approach is performance against modern classical time-series baselines (e.g., N-BEATS, TFT, or even LightGBM) on a larger, less curated dataset — and that experiment is absent here. Until it exists, treat this as architecture feasibility, not advantage evidence.

---

## Scores

| Dimension | Score (1–5) | Rationale |
|---|---|---|
| Technical significance | 3 | Projected-kernel QGP architecture at 100 qubits with noise characterization is a genuine engineering contribution; the KQRC-RM hardware failure tempers the score. |
| Industrial relevance | 3 | Energy demand forecasting is a high-value domain, and the hardware scalability result matters for deployment planning, but the classical baseline gap makes the score conditional on future benchmarking. |
| Misinterpretation risk | 5 | "Utility scale," "62% MAE reduction," and "feasibility demonstration" will routinely be cited as near-term QML advantage evidence — the paper provides insufficient inoculation against this reading. |

---

## Verification
- **Public post / GitHub URL:** [placeholder — not yet published]
- **Commit / post date:** 2026-05-26

---

---

# QML Claims Audit

*This section provides an adversarial review of the paper's QML claims across five domains: dataset realism, classical baseline strength, trainability and barren plateau risk, hardware and noise assumptions, and generalizability.*

---

## 1. Dataset Realism

**What the paper uses:** 103 residential smart meter time-series, winter season only, hourly granularity, all from a single anonymized dataset. Experimental subsets are chosen by *maximum pairwise correlation* — meaning the models are always evaluated on the most favorable, most structured slice of the data.

**Problems:**

- **Dataset size is tiny for benchmarking QML.** Training windows are 15 hours; test horizons are 5 hours. This is an extremely short-horizon, low-data regime. Classical GP baselines are known to perform competitively in exactly this regime; the comparison is therefore structurally biased toward making the quantum model look good.
- **Correlation filtering is a latent confound.** By selecting the 15 most correlated customers and then constructing triplets/groups from that subset, the paper stacks the deck: correlated signals are precisely the setting where quantum entanglement structure *could* provide a structural inductive bias. The "quantum advantage" may be entirely an artifact of this selection criterion.
- **Single season, single dataset.** There is no winter/summer transfer, no year-over-year generalization, and no heterogeneous demand patterns (e.g., commercial customers, EV charging). The model may be exploiting winter-season temporal periodicity, not quantum-enhanced feature extraction.
- **No ablation on correlation threshold.** The paper does not test what happens when less-correlated customers are included. If performance degrades sharply, the correlation filtering is doing most of the work.

**Verdict:** Dataset is adequate for a feasibility demonstration but insufficient to support any claim about real-world forecasting advantage. The correlation-selection methodology needs explicit sensitivity analysis before deployment implications can be drawn.

---

## 2. Classical Baseline Strength

**KQRC-RM baseline — Echo State Network with KRR (ESN-KRR):**

- ESN-KRR is a reasonable structural analogue for reservoir computing, but it is **not a competitive modern baseline** for energy time-series forecasting. Specifically:
  - ESN is trained *per-customer* (no cross-stream joint modeling), whereas KQRC-RM is jointly trained — this is an architectural asymmetry that advantages the quantum model independently of quantumness.
  - No comparison against N-BEATS, Temporal Fusion Transformer, DeepAR, or even a simple LSTM with cross-series inputs.
  - No comparison against a multi-stream classical reservoir with cross-stream coupling (which would isolate whether the quantum dynamics or just the cross-stream joint modeling is doing the work).

**QGP baseline — Classical multi-output GP with RBF kernel (LMC coregionalization):**

- The classical GP baseline is arguably the weakest possible choice for a modern multi-output forecasting comparison. LMC with RBF is a 2007-era method.
- No comparison against MOGP with spectral mixture kernels, which are competitive on periodic time-series.
- No comparison against variational sparse GPs, which scale better and are often more accurate on small-data regimes.
- The naïve persistence baseline (last-observation carry-forward) is included, and QGP hardware results are only marginally better than persistence on Group A (0.24 vs. 0.28 MAE) — this is a damning detail that is not foregrounded.

**Bottom line:** The "62% MAE improvement" over classical GP is measured against a strawman. A 2026 paper should compare against at least one modern neural forecasting model (TFT, N-BEATS, or PatchTST) and one competitive kernel method. The absence of these comparisons is the single largest gap in the experimental section.

---

## 3. Trainability and Barren Plateau Risk

**What the paper does:** KQRC-RM uses no trained quantum parameters — the reservoir is fixed and only the classical kernel readout is trained. QGP trains circuit parameters using the parameter-shift rule to maximize GP log-likelihood, but this training is done exclusively on a simulator (5-qubit configuration), and parameters are then frozen for hardware inference.

**Issues:**

- **KQRC-RM avoids the trainability problem by construction.** A fixed, non-trained reservoir sidesteps barren plateaus entirely. This is a deliberate design choice, but it also means the model cannot adapt its quantum feature map to the data — the expressivity is fixed at initialization. The "36.92% improvement" on simulator is therefore entirely attributable to the inductive bias of the specific fixed reservoir architecture, not to trained quantum representations.

- **QGP trainability is only validated at 5 qubits.** The convergence curve (Fig. 11a) shows clean loss convergence for the 5-qubit model on a simulator. The 100-qubit experiment uses *transferred* parameters — the parameters were optimized for 5 qubits and scaled up, not trained at 100 qubits. This is a critical gap: there is no evidence that a 100-qubit projected quantum kernel GP is trainable from scratch. The paper assumes the 5-qubit solution generalizes, but this is unvalidated.

- **Barren plateau risk for QGP at scale:** Even with projected kernels (which reduce concentration relative to fidelity kernels), the gradient landscape of a 100-qubit variational circuit with multi-layer data re-uploading may exhibit exponentially vanishing gradients. The paper does not present gradient variance as a function of qubit count, which would be the standard diagnostic. This is a significant omission for a paper claiming 100-qubit scalability.

- **No expressibility or entanglement capacity analysis.** The paper does not characterize whether the hardware-efficient ansatz used in QGP is expressive enough to distinguish the customer time-series at 100 qubits, or whether it is operating in an over-entangled regime that suppresses distinguishability (a known failure mode for projected kernels).

**Verdict:** The trainability story is incomplete and partially circular. KQRC-RM avoids the problem by using a fixed reservoir; QGP scales to 100 qubits by parameter transfer, not by training at 100 qubits. Neither result establishes that QML training at utility scale is tractable.

---

## 4. Hardware and Noise Assumptions

**Strengths — the paper is above-average in hardware honesty:**
- Experiments run on real IBM hardware (`ibm_marrakesh`, Heron r2), not solely on simulators.
- Dynamical decoupling (XpXm) and gate twirling (32 randomizations) are applied.
- Per-qubit T2 times and CZ error rates are reported and correlated with prediction accuracy tiers.
- The 20% high-error rate in the 100-qubit QGP run is explicitly attributed to specific hardware regions (physical qubits 41–55 and 121–155, T2 as low as 14–24 μs).

**Gaps and concerns:**

- **Training on simulator, inference on hardware.** The QGP model is never trained on hardware. The parameter-shift rule is applied entirely in simulation (MPS simulator). The practical question — can a projected quantum kernel GP be trained end-to-end on near-term hardware within a reasonable shot budget — is not answered.

- **Shot budget is not reported.** The number of circuit shots per kernel evaluation, and the resulting wall-clock time and cost of the 100-qubit experiment, are not disclosed. For real-world feasibility, this is a first-order constraint. A classical GP on this dataset trains in milliseconds.

- **MPS simulator fidelity.** The Matrix Product State simulator is used as the "noiseless" reference, but MPS truncates quantum entanglement for computational tractability. The gap between MPS simulator and ideal statevector simulation is not characterized, which means the "simulator MAE" may itself be a lower bound on noiseless performance rather than exact.

- **Hardware noise is quasi-static.** The experiments were run at a specific calibration snapshot. T2 and CZ error rates drift on timescales of hours to days. The paper reports calibration data at execution time but does not characterize prediction stability across recalibrations — a critical concern for any operational deployment.

- **No error mitigation on KQRC-RM for ancilla readout.** Readout error mitigation (e.g., matrix inversion on the ancilla measurement distributions) is not applied, even though readout errors directly corrupt the probability vectors used as features. For a 114-qubit circuit with repeated mid-circuit measurement, this is likely a significant noise source.

---

## 5. Generalizability Beyond the Reported Benchmark

**Structural dependencies that limit generalizability:**

| Dependency | Paper Assumption | Risk if Violated |
|---|---|---|
| High inter-series correlation | Subsets selected by maximum Pearson correlation | Performance likely degrades substantially on uncorrelated or anti-correlated series |
| Short training window (15 hrs) | Favors non-parametric methods (GP, KRR) over neural models | Neural baselines become competitive or dominant with >100 training samples |
| Fixed seasonal regime | Winter only, single dataset | Non-stationarity across seasons untested; residential demand seasonality is strong |
| Small output dimensionality | 3 streams (KQRC-RM), 5–100 streams (QGP) | KQRC-RM scales quadratically in qubit count; untested beyond 3 streams |
| Parameter transfer from 5→100 qubits | Assumed to generalize | No theoretical or empirical validation of transfer quality |
| Single hardware platform | `ibm_marrakesh` Heron r2 | Different connectivity topologies (e.g., Google Sycamore, trapped-ion) may produce very different results |

**What would make this result generalize:**
1. Reproduce on at least one other quantum platform (different gate set and topology)
2. Test with uncorrelated customer subsets drawn randomly rather than by correlation rank
3. Train QGP end-to-end at 20+ qubits on hardware (not simulator) and report trainability diagnostics
4. Compare against at least one competitive modern neural baseline (TFT, N-BEATS, or LightGBM)
5. Report shot counts, wall-clock times, and energy-cost-per-prediction to enable fair resource comparison

**What the paper should not be cited as:**
- Evidence that quantum models outperform deep learning on time-series forecasting
- Evidence that NISQ quantum hardware provides practical advantage for energy systems
- Evidence that projected quantum kernels are trainable at 100 qubits
- A "utility scale" result in the operational energy-industry sense

**What the paper should be cited as:**
- Evidence that a projected-kernel QGP can produce structured, useful outputs from 100-qubit circuits under realistic hardware noise
- Evidence that topology-aware circuit design materially reduces circuit depth and preserves prediction quality at scale
- A concrete instantiation of how hardware noise heterogeneity maps to prediction accuracy tiers — a useful operational diagnostic for future QML deployment planning
