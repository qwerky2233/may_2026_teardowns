# Quantum Paper Teardown

## Thesis

Quantum Error Mitigation benchmarks systematically overstate technique effectiveness because they ignore two compounding artefact sources — parameter sensitivity and hardware temporal drift — both of which individually, and severely jointly, can flip experimental conclusions from "significant improvement" to "significant degradation."

## Verdict

The paper succeeds and matters substantially: it provides empirical proof, not just theoretical concern, that the evidentiary base for near-term QEM claims is structurally compromised, and it names the exact failure modes precisely enough to act on them.

## Tags

`error-mitigation`, `benchmarking`, `statistical-rigour`, `ZNE`, `NISQ`, `reproducibility`, `quantum-advantage`, `hardware-drift`

\---

## Paper Info

* **arXiv link:** https://arxiv.org/abs/2605.29872
* **Date:** 28 May 2026
* **Title:** Claim against Measurement: Statistical Artefacts in Quantum Error Mitigation Benchmarks
* **Authors:** Dominik Koester, Wolfgang Mauerer (Technical University of Applied Science Regensburg; Siemens AG)

\---

## What the Paper Claims

A systematic review of 81 QEM papers (2022–2026) finds that 75% use only descriptive statistics and only 30% control for hardware drift — the two criteria that matter most when effects are small and conditions are variable. A replication study across 132 parameter configurations shows that implicitly assumed ZNE choices (scale factors, extrapolation method, folding strategy, calibration snapshot) are *active* parameters: their variation shifts outcomes from statistically significant improvement to statistically significant degradation in 12% of tested configurations even on physics-grounded noise models. A 72-hour longitudinal study on real hardware (IQM Euro-Q-Exa) shows temporal drift alone induces a **3.4× variation in apparent ZNE effectiveness** — larger than the 2.7× variation produced by switching the entire noise model — and reduces 30 nominal repetitions to as few as 1.8 effective independent observations. The paper does not claim QEM is intrinsically unsound; it claims current evaluation practice cannot distinguish genuine mitigation effects from statistical and experimental confounds.

\---

## Mechanism in Plain Language

**Parameter sensitivity.** A ZNE experiment requires choosing scale factors (the set of noise amplification levels), a folding strategy (how gate repetitions are inserted), an extrapolation model (linear, polynomial, exponential, Richardson), a transpiler setting, and a hardware calibration snapshot. None of these are typically reported as a sensitivity-tested range — papers pick one set and report the result. This paper shows that varying just those unchosen parameters across reasonable literature-standard values moves the same underlying experiment across three distinct outcome zones: significantly better, not significant, and significantly worse. Scale factor choice alone can change the variance bound by a factor of 194×.

**Drift-induced effectiveness illusion.** Superconducting processors recalibrate on timescales of minutes to hours, and the noise environment at circuit execution time determines the true signal level. When ZNE is run at different times on the same device with the same code, the apparent effect size (Cohen's d) varies by more than 3× over 48 hours — not because ZNE changed, but because the hardware's noise floor shifted. Worse, consecutive runs are strongly autocorrelated (lag-1 r₁ = 0.83 in the weekend session), meaning 30 repeated measurements carry roughly the statistical weight of fewer than 2 independent observations. Standard paired t-tests silently assume independence, so significance conclusions derived from them are formally invalid in the presence of such autocorrelation.

**Compounding.** The two effects interact multiplicatively. A paper that reports a single configuration at a single point in time conflates a potentially fortunate parameter choice with a potentially favourable calibration window. Neither source alone would be damning in a field with adequate statistical practice, but together, without inferential testing, they allow artefact to read as discovery.

\---

## Core Analytical Results: What Is Demonstrated vs. What Is Implied

### Demonstrated (hard results with statistical backing)

|Result|Evidence|
|-|-|
|75% of 81 reviewed QEM papers use only descriptive statistics|Systematic 8-criterion review with inter-rater agreement and LLM-assisted baseline|
|70% do not control for temporal drift|Same review|
|Scale factor choice produces a 194× difference in ZNE variance bound (analytically)|Derived from Richardson coefficient formula — no experiment needed|
|Outcome classification (better/worse/n.s.) shifts across 12% of 66 non-negative-control configurations|132-configuration sweep, paired t-tests with Bonferroni and BH correction — robust|
|Temporal drift induces 3.4× variation in apparent ZNE effect size within 48 hours on a single device|147-time-point longitudinal study with autocorrelation and ICC analysis|
|Drift reduces nominal 30 repetitions to \~1.8 effective independent observations|Kish design-effect formula applied to one-way ANOVA ICC within time points|
|Drift-induced effectiveness illusion is not IQM-specific: confirmed on IBM ibm\_brussels|Independent 12-hour session|
|ZNE is counterproductive for shallow circuits on low-error hardware (IBM Marrakesh TC1, d = −0.75)|Direct hardware run — clean result with clear mechanistic explanation|

### Implied but Not Fully Established

|Implication|Caveat|
|-|-|
|Most published QEM improvements are artefacts|The paper shows evaluation is *insufficient to distinguish* artefact from genuine effect — it does not demonstrate that all reported improvements are false positives. The 83% significant-improvement rate on physics-grounded noise models actually supports ZNE's existence as a genuine technique|
|The critique generalises to PEC, Clifford data regression, and other QEM|The paper explicitly targets ZNE with Richardson extrapolation as "the most widely reported method." The parameter-sensitivity argument is structurally applicable to any method with unchosen hyperparameters, but drift severity and parameter interaction profiles will differ|
|Near-term quantum advantage claims built on QEM are invalidated|The critique establishes that prior claims cannot be *re-verified* with current evidence standards; it does not establish they were wrong at the time of execution|

\---

## Claims Audit: Prior Quantum Advantage Work Most Affected

### Category 1 — IBM "Quantum Utility" (Kim et al. 2023, *Nature*)

The most prominent QEM-dependent advantage claim in the corpus period. That paper applied ZNE (among other methods) to a 127-qubit Ising model simulation and argued classical simulation was impractical at that scale. The paper under review directly cites it (\[33]) as an example of ZNE deployed at scale with framework-default parameters inherited silently. This critique binds hardest here because: (a) the experiment ran on a single calibration window with no drift randomisation; (b) scale factors and folding strategy were not sensitivity-tested; (c) no paired inferential statistics were reported for the mitigation step specifically. **The critique does not prove the utility claim is false — the classical hardness argument is independent of whether ZNE added value. But it does establish that the ZNE-improved vs. raw comparison presented cannot support the reliability attributed to it.**

### Category 2 — Variational Quantum Eigensolvers (VQE) with ZNE

A large class of near-term chemistry and optimisation papers applies ZNE as a routine post-processing step before reporting energy estimates. Across this class, the critique binds in three ways simultaneously: (a) circuits are typically shallow enough that ZNE can be counterproductive on low-error hardware (the IBM Marrakesh TC1 result is a direct demonstration); (b) VQE experiments are run in batches over cloud access with non-deterministic timing — exactly the scenario where drift-induced effectiveness illusion operates; (c) energy comparisons to classical benchmarks are reported as single-point results without parameter robustness checks. **Papers claiming chemical accuracy improvements attributable to ZNE are most exposed: the improvement margins in near-term VQE are typically small (sub-10 mHartree), well within the variation ranges demonstrated here.**

### Category 3 — Shot-Optimisation and Overhead Papers

A secondary category of papers claims to identify optimal shot counts or sampling strategies to maximise mitigation benefit (e.g., the Desdentado et al. case study examined in the paper). The critique demonstrates that execution-time confounding with calibration drift makes it systematically impossible to isolate shot count effects from temporal hardware effects without randomised execution order. Papers reporting non-monotonic relationships between shot count and circuit fidelity should be re-examined for drift as an alternative explanation.

### Least-Affected Categories

* **Purely simulated benchmarks** (no real hardware): parameter sensitivity critique still applies, but temporal drift does not.
* **PEC with explicit noise characterisation**: more constrained parameter space if the noise model is separately validated; drift sensitivity depends on how recently the noise model was calibrated.
* **Fault-tolerant threshold experiments**: outside the paper's scope; QEC overhead studies are not QEM evaluations.

\---

## Assessing the Critique's Own Limits

### Where the Critique Is Strong

The 8-criterion review is methodologically sound (inter-rater protocol, LLM false-positive filtering, 77% automated agreement). The parameter sweep uses standard statistical corrections and is robust to multiple comparisons. The temporal autocorrelation finding has clean mathematical grounding via the Kish design-effect formula. The cross-vendor validation (IBM Brussels confirming IQM drift pattern) meaningfully limits hardware-specificity as a counter.

### Where the Critique May Overreach or Leave Gaps

**1. ZNE-only scope.** The paper is scoped to ZNE with Richardson extrapolation. This is the most common method, but probabilistic error cancellation (PEC) is analytically distinct — it has explicit overhead bounds derived from noise characterisation and is less sensitive to scale factor choices because it does not perform extrapolation. The parameter-sensitivity critique does not apply to PEC in the same form. A reader who concludes "all QEM is as unreliable as ZNE evaluated here" is overreading.

**2. No hybrid mitigation-correction analysis.** The paper does not address hybrid QEM-QEC architectures, where error correction handles dominant error channels and mitigation addresses residual noise. In these regimes — increasingly relevant as device quality improves — the signal-to-noise rationale changes: drift variance at the QEM layer is smaller relative to the corrected baseline. The critique's drift severity is, in part, a function of operating at raw error rates where noise is dominant; it will be less severe in the hybrid setting.

**3. Static representation of mitigation literature.** The review covers 2022–2026 but makes no distinction between early and recent work. Mitigation practice has been evolving: recent papers from IBM and Mitiq have begun including scale factor sensitivity analysis as standard. The 25% inferential statistics rate may be improving at the tail of the review window, though the paper does not test for trend.

**4. Effect-size inflation acknowledged but not fully resolved.** The paper correctly notes that Cohen's d is inflated by shot count and uses Cliff's δ as a non-parametric alternative. But the longitudinal study focuses primarily on d for the drift narrative (citing 3.4× variation). The equivalent Cliff's δ variation over time is not reported for the longitudinal study, which would have made the drift-induced illusion more directly comparable to the cross-configuration analysis.

**5. Financial constraints framing.** The paper notes that cloud access costs constrain experiment repetitions — this is valid but also argues for the field to develop cost-efficient drift-control designs (randomised execution, interleaved controls), not necessarily to dismiss experiments that couldn't afford comprehensive randomisation. The critique is tougher on published work than on the structural constraints that produced it.

\---

## What Matters Practically

Any future quantum result that cites ZNE-improved values as evidence of genuine computational advance needs to answer three questions the prior literature does not: (1) Are the reported improvements robust across at least a small parameter grid — specifically scale factors and extrapolation method? (2) Was hardware drift randomised out or measured? (3) Were inferential statistics applied, not just error bars? If the answer to any of these is no, the result should be read as existence-proof that the technique *can* improve results under those conditions, not as evidence of reliable, reproducible improvement. For system architects evaluating quantum advantage timelines: error mitigation is not a dependable substitute for error correction as a path to useful computation. The paper quantifies exactly why — the improvement margin is real but fragile, and its fragility is currently invisible in the published record.

\---

## Likely Misinterpretation

**Overread dismissal:** Some readers will conclude that ZNE doesn't work and that all prior QEM-based advantage claims are fabricated. This is not what the paper shows. On physics-grounded noise models, 83% of configurations show statistically significant improvement. The critique is about *evaluation quality*, not *method validity*. ZNE improves results under the right conditions; the problem is that current practice cannot reliably identify what those conditions are or reproduce them.

**Underread continuation:** Others will note that the paper confirms ZNE works in most tested configurations and continue publishing single-point, no-drift-control evaluations with error bars but no hypothesis tests. This ignores the central finding: in the regimes where near-term advantage is being claimed — small effects, cloud hardware, shallow circuits — the probability of an artefact masquerading as a result is not negligible, it is structural.

**ZNE-only generalisation risk:** The paper's results should not be mechanically extended to PEC or learning-based mitigation without separate analysis. These methods have different hyperparameter structures and different relationships to hardware noise characterisation. Applying the drift critique uniformly conflates methods with distinct properties.

\---

## Bottom Line

Treat any published QEM improvement claimed without parameter robustness checks, drift control, and inferential statistics as an existence result, not a reliability result. This paper makes it impossible to read such claims the old way in good faith — not because QEM fails, but because the evidence was never adequate to support the confidence placed in it. Future advantage work built on mitigation needs to budget for drift randomisation and parameter sweeps, or explicitly scope its claims to the tested point in configuration space.

\---

## Scores

|Dimension|Score (1–5)|Rationale|
|-|-|-|
|Technical significance|5|The drift-induced effectiveness illusion and 1.8 effective observation result are empirically novel and analytically rigorous. The 194× variance-bound difference from scale factor choice is an analytic result that should have been in every ZNE tutorial already.|
|Industrial relevance|4|Directly affects any organisation making resource allocation decisions based on published QEM benchmarks. Slightly below 5 because the immediate industrial path — hybrid QEM-QEC at higher gate fidelity — is partially outside the critique's scope.|
|Misinterpretation risk|5|High in both directions simultaneously. The paper's own careful caveats ("we do not claim QEM is unsound") will be lost in secondary citation. Any paper that merely cites this as "ZNE doesn't work" or dismisses it as "just a stats complaint" is misreading it.|

\---

## 

