# Quantum Paper Teardown

## Thesis

Microsoft's Reichardt, Aasen, and Chao argue that concatenating the five-qubit Laflamme code onto the four-qubit Iceberg (⟦4,2,2⟧) error-detecting code yields small, resource-efficient quantum codes that — while explicitly not fault tolerant — achieve logical error rates competitive with much larger codes at realistic near-term noise levels (p \~ 10⁻³).

## Verdict

The paper succeeds on its own terms and matters significantly: it is an honest, simulation-backed case that **partial fault tolerance with selective postselection is a viable architectural path**, not a compromise — but only within a tightly bounded noise and circuit-size operating window that the paper both defines and, in places, understates the difficulty of reaching.

## Tags

`error-correction`, `partial-fault-tolerance`, `iceberg-code`, `concatenation`, `near-term-hardware`, `postselection`, `steane-ec`, `trapped-ion`

\---

## Paper Info

* **arXiv link:** https://arxiv.org/abs/2605.15344
* **Date:** 14 May 2026
* **Title:** Fire and ice: Partially fault-tolerant quantum computing with selective state filtering
* **Authors:** Ben W. Reichardt, David Aasen, Rui Chao (Microsoft Quantum)

\---

## What the Paper Claims

The paper claims that small CSS codes built by concatenating the ⟦5,1,3⟧ Laflamme code (and related codes) onto the ⟦4,2,2⟧ Iceberg error-detecting code — yielding ⟦20,2,6⟧ and ⟦36,2,8⟧ CSS codes — achieve logical error rates of \~2×10⁻⁶ and \~10⁻⁷ per round at p = 10⁻³, respectively, under circuit-level noise. These codes are explicitly not fault tolerant: second-order faults can cause undetected logical errors. The paper argues this is acceptable because, at realistic noise rates, the statistical dominance of higher-order errors means the non-fault-tolerant circuits behave *effectively* like d/2+1 codes in practice. The key enabling mechanism is **selective filtering (postselection)** on encoded ancilla states before use in Steane-style error correction, which suppresses the worst errors at the cost of rejecting some prepared states. The paper provides a quantitative criterion (pR/pL < \~300) under which rejection overhead remains manageable.

\---

## Mechanism in Plain Language

The ⟦4,2,2⟧ Iceberg code encodes **two logical qubits** in four physical qubits and detects (but does not correct) any single-qubit error. On its own this is useful but limited — detection without correction means you just discard bad states. The insight is to use this as the *inner* layer of a concatenated code, with a small outer code (e.g., the 5-qubit Laflamme code) handling correction across Iceberg blocks. The resulting ⟦20,2,6⟧ code has distance 6: it takes at least 4 simultaneous faults to cause an undetected logical error.

Error correction uses **Steane's method**: an ancilla block prepared in an encoded |0⟩ or |+⟩ state is coupled to the data via transversal CNOT gates and measured, extracting an error syndrome without directly touching the data. The catch is that preparing these ancilla states reliably is hard — the circuits for doing so are *not* fault tolerant, meaning a cleverly placed second-order fault can sneak through. The solution is **filtering**: after preparing an ancilla, the scheme measures a subset of stabilizers as a sanity check. If anything looks wrong, the ancilla is discarded and preparation restarts. Because the ancilla hasn't yet touched the data, discarding it costs only time, not computation fidelity.

The "paired support" property of GF(4)-linear outer codes (e.g., ⟦5,1,3⟧, ⟦8,2,3⟧) enables efficient simultaneous construction of two X stabilizers per transversal CNOT step, which is what makes the state-preparation circuits compact enough to be practical. This structural property is what distinguishes the Iceberg-concatenated codes from arbitrary small codes and is the key technical enabler behind the overhead numbers in Table I.

\---

## What Matters Practically

**1. The operating regime is real and near-term.** The paper targets p ∈ \[10⁻⁴, 4×10⁻³], matching announced trapped-ion and neutral-atom two-qubit gate fidelities. This is not a theoretical limit result — it is calibrated to hardware that exists or is 1–2 generations away. At p = 10⁻³, the ⟦20,2,6⟧ code delivers \~2×10⁻⁶ logical error rates with \~100–200 CNOTs per accepted ancilla, which is a practical number for near-term systems.

**2. Partial FT reframes the overhead calculus.** Full fault tolerance guarantees that *any* single fault in a preparation circuit cannot cause a logical error. Achieving this forces larger, slower circuits. The paper shows rigorously that at p \~ 10⁻³, the non-fault-tolerant second-order errors are statistically negligible compared to the dominant d/2+1-order errors — so you can often skip the overhead of full FT without paying a visible penalty in logical error rate. This changes the architectural tradeoff: for near-term systems with limited qubit counts, partial FT + filtering may dominate full FT in useful logical operations per unit time.

**3. Rejection rate management requires adaptive routing infrastructure.** The pR/pL < 300 criterion is central to the paper's practicality claim. At p = 10⁻³ this holds comfortably for both codes. But rejection means discarding ancilla states — and those ancilla states must be replenished from a factory running in parallel. This demands **adaptive routing of ancilla states to data blocks**, which the paper notes is a real engineering requirement, not a footnote. Hybrid quantum-classical systems that ignore ancilla factory throughput in their resource models will underestimate overhead.

**4. The two-stage ancilla factory is architecturally important.** For the distance-8 codes (⟦36,2,8⟧ and ⟦48,4,8⟧), a single-stage factory wastes significant work when a sub-block fails. The two-stage design — where the inner ⟦12,2,4⟧ or ⟦16,4,4⟧ blocks are validated first, and only accepted blocks proceed to the outer stage — reduces expected CNOT cost by 30–38% at p = 0.4%. This is a workflow optimization, not just a circuit trick, and has direct implications for how ancilla factories should be scheduled on real hardware.

**5. Transversal CNOT logical error rates differ substantially from memory error rates.** The paper's Table I and Fig. 16 show that CNOT gates between code blocks can increase logical error rates by 4–10× over repeated error correction alone. A correlated decoder that jointly decodes both blocks reduces this, but adds latency. Systems architects who benchmark only memory (repeated EC) error rates and use those to project computation error rates will be significantly optimistic.

\---

## Likely Misinterpretation

**Misread 1: "Microsoft has demonstrated fault-tolerant quantum computation."** They have not. The paper is explicit that the circuits are not fault tolerant. A single bad second-order fault in the ancilla preparation circuits can pass through the filter and cause a logical error. The paper's contribution is showing that, *statistically*, this doesn't dominate at p \~ 10⁻³. At lower noise (p < \~5×10⁻⁴), the second-order errors begin to show, and the ⟦20,2,6⟧ circuit (a) curve in Fig. 7 visibly flattens — exactly as the paper warns. Anyone who cites this paper as proof of full fault tolerance has not read Section VII.

**Misread 2: "The rejection overhead is acceptable for arbitrary algorithms."** The pR/pL < 300 criterion limits the *number of logical operations* you can run before rejections dominate circuit acceptance probability. For the 5% acceptance rate case (10× time overhead), the bound is \~300 operations limited by logical error rate rather than rejection rate. This is fine for small circuits but does not scale to deep fault-tolerant algorithms like Shor factoring at commercial scale. The paper is honest about this; press coverage will not be.

**Misread 3: "Small codes beat the surface code."** The comparison to surface codes in Fig. 7 is framed carefully by the authors (they note it "unfairly advantages" surface codes by not accounting for the d syndrome cycles needed per logical operation in fault-tolerant surface code execution). The takeaway is not that Iceberg-concatenated codes are universally superior to surface codes — it is that for non-local (all-to-all) connectivity platforms, they are competitive at moderate qubit counts, whereas surface codes remain better-suited for geometrically constrained superconducting architectures.

**Misread 4: "Non-Clifford gates are handled."** The paper explicitly leaves non-Clifford (T gate / arbitrary Z rotation) overhead to future work. The full universality stack — magic state distillation or cultivation overhead combined with this partial FT scheme — has not been analyzed. A hybrid quantum-classical strategy built around this paper's codes cannot yet be costed end-to-end.

\---

## Bottom Line

This paper establishes a **concrete, practically calibrated alternative to full fault tolerance for near-term non-local qubit platforms**: concatenate onto the Iceberg code, use Steane-style EC with filtered ancillas, and accept pR/pL < 300 as a workflow constraint. Any near-term quantum system architecture targeting trapped-ion or neutral-atom hardware at p \~ 10⁻³ should benchmark against these codes before defaulting to surface-code assumptions. The open gap — non-Clifford gate overhead at these noise levels — is now the critical unknown for end-to-end resource estimation.

\---

## What This Changes for Partial-FT Workflow Evaluation vs. Full-FT Claims

### The partial-FT workflow is a regime, not a fallback

The standard framing treats partial fault tolerance as a concession made while waiting for full FT. This paper reframes it as a **deliberately chosen regime** with its own parameter space: distance d ≤ 8, p \~ 10⁻³, pR/pL < 300, non-local connectivity. Within this regime, partial FT + filtering is strictly better than full FT for small logical circuits because it reduces overhead without paying a detectable logical error penalty. The evaluation question is not "when will we achieve full FT?" but "what is the circuit depth budget before partial FT's second-order errors begin to dominate, and does my application fit inside it?"

### The rejection-to-error ratio is the new threshold

Full FT literature defines a **pseudothreshold** (noise rate below which the code improves over physical qubits) as the primary figure of merit. For partial FT schemes with postselection, pR/pL < C (where C ≈ 300 for 5% acceptance tolerance) is the binding constraint for circuit length. Workflow evaluation must track *both* quantities across all circuit layers, not just the logical error rate. Current resource estimators that output only "logical error rate per gate" are insufficient for evaluating partial FT schemes.

### Ancilla factory throughput must be in the critical path

In full FT workflow evaluation, magic state distillation overhead is the standard bottleneck to model. In this partial FT scheme, the analogous bottleneck is **ancilla state factory throughput**: how many accepted ⟦4,2,2⟧-encoded |0⟩ states can be delivered per unit time to the data blocks running Steane EC? The two-stage factory design (Fig. 13) provides the right abstraction, but system architects need to model adaptive routing latency explicitly. The paper shows this matters at p = 0.4% (30–38% CNOT savings from adaptive staging); at higher p or with tighter latency budgets, the gap will be larger.

### CNOT logical error rates must be measured separately from memory error rates

Full FT benchmarking frequently quotes memory (repeated EC) error rates as the primary logical performance metric. This paper's Fig. 16 shows that transversal CNOT gates between code blocks increase logical error rates by 4–10× over repeated EC, with a correlated decoder closing much of the gap. **Any partial FT workflow evaluation that benchmarks only memory error rates and projects CNOT performance from those will be significantly optimistic.** The evaluation protocol must include CNOT + EC round benchmarks as a distinct metric, not a derived quantity.

### The non-Clifford gap is now the critical unknown

This paper closes the loop on Clifford operations (CNOT, Hadamard, EC rounds) for this code family. Non-Clifford gates — T gates, arbitrary Z rotations — remain unanalyzed. Until the T gate overhead is costed within this partial FT framework (whether via physical Z rotations + error detection per \[CCED23], magic state distillation \[BK05], or cultivation \[GSJ24]), end-to-end resource estimation for any application requiring non-Clifford operations cannot be completed. The partial FT workflow is **conditionally validated for Clifford circuits only**. Claiming quantum advantage for any non-trivially non-Clifford algorithm on the basis of this paper alone is premature.

\---

## Scores

|Dimension|Score (1–5)|Rationale|
|-|-|-|
|Technical significance|4|Rigorous simulation-backed case for a practical, honest partial-FT code family. The paired-support insight and two-stage ancilla factory are genuine contributions. Held back from 5 by the unresolved non-Clifford gap and the simplified noise model (no idle qubit noise, no leakage, no bias).|
|Industrial relevance|4|Directly calibrated to trapped-ion and neutral-atom hardware specs. The ⟦20,2,6⟧ and ⟦36,2,8⟧ codes are specific, simulatable, and buildable with near-term qubit counts. Minus one point because the comparison to surface codes undersells how different the connectivity assumptions are, which matters for superconducting-centric organizations.|
|Misinterpretation risk|5|Very high. The phrase "not fault tolerant" will be ignored in press coverage. The restriction to Clifford circuits will be glossed over. The noise model simplifications will be treated as conservative rather than as potentially-significant omissions. The pR/pL < 300 constraint on circuit depth will not make it into headlines.|

\---

## 

