# Quantum Paper Teardown

## Thesis
LWE-based post-quantum cryptography is not unconditionally quantum-secure; its perceived hardness is entirely bottlenecked by transient engineering constraints on quantum state preparation, not by any proven theoretical lower bound.

## Verdict
The paper succeeds at exposing a real and underappreciated conceptual gap in how the cryptographic community frames LWE security, but it does not demonstrate a practical attack — and the gap between the theoretical argument and an operational exploit remains enormous. It matters because it sets the correct epistemic baseline for any organization claiming PQC-readiness today.

## Tags
`post-quantum-cryptography`, `LWE`, `lattice-cryptography`, `quantum-learning`, `GKP-error-correction`, `complexity-theory`, `security-assumptions`

---

## Paper Info
- **arXiv link:** https://arxiv.org/abs/2605.04582
- **Date:** May 6, 2026
- **Title:** Fundamental Limitations of Post-Quantum Cryptographic Architectures
- **Authors:** Jiho Jung, Donghwa Ji, Mingyu Lee, Kabgyun Jeong (Team QST, Seoul National University)

---

## What the Paper Claims

The paper argues across four interconnected domains that lattice-based cryptography — specifically the Learning with Errors (LWE) paradigm underpinning NIST-standardized schemes like ML-KEM (Kyber) and ML-DSA (Dilithium) — provides no unconditional quantum security guarantee. Its four claims are: (1) LWE's hardness rests on unproven complexity-theoretic assumptions that could collapse into BQP; (2) Shannon's theorem and Fano's inequality prove that the injected Gaussian noise does not thermodynamically erase the cryptographic secret; (3) LWE noise is structurally equivalent to continuous-variable displacement errors correctable by GKP quantum error correction codes; and (4) quantum learning models such as the GKZ algorithm can extract the underlying secret in polynomial time given access to quantum samples. The paper concludes that calling any LWE-based scheme "post-quantum" represents a premature classification.

## Mechanism in Plain Language

LWE security works by injecting artificial Gaussian noise into a system of linear equations — the idea is that even a quantum adversary cannot distinguish the noisy equation from a random one and therefore cannot recover the hidden secret vector. The GKZ algorithm (Grilo-Kerenidis-Zijlstra, 2019) shows this assumption breaks down if the adversary can prepare coherent quantum superpositions over the classical sample space: applying the quantum Fourier transform to this superposition amplifies the signal corresponding to the secret, reducing the noise to a manageable phase perturbation rather than an insurmountable barrier. The critical obstacle is not mathematical but physical — preparing that superposition requires a quantum random access memory (QRAM) capable of loading exponentially many classical data points coherently, and no such hardware exists at scale. The paper extends this by noting that Shannon's channel capacity theorem already proves the noise does not permanently destroy information (the legitimate receiver must be able to decrypt, bounding the residual entropy of the secret), and that the Gottesman-Kitaev-Preskill (GKP) error-correcting code is formally designed to remove exactly the kind of Gaussian displacement errors that LWE injects as its security mechanism — meaning the same tools being developed to make quantum computers reliable could, in principle, be repurposed to break the cryptosystem.

## What Matters Practically

The paper confirms that LWE security is not grounded in any unconditional physical law — it is grounded in the cost of QRAM state preparation, which is a rapidly evolving engineering problem rather than a fixed ceiling. Organizations that adopt NIST-standardized PQC schemes (ML-KEM, ML-DSA, SLH-DSA) and treat migration as a solved problem are miscategorizing residual risk as eliminated risk. The distinction matters most for long-lived key material and high-value secrets that must remain secure on a 10–20 year horizon: if fault-tolerant QRAM overhead decreases significantly before key material is rotated, the security model degrades without warning. The paper also signals that hybrid quantum-classical variational algorithms on current NISQ hardware are already beginning to probe these structures, meaning the threat is not purely hypothetical future hardware.

## Likely Misinterpretation

The most dangerous misread is that this paper demonstrates an attack. It does not — it identifies where an attack would live and argues the defense is engineering-contingent rather than theoretically permanent. Readers who conclude "LWE is broken" have overcredited the paper; readers who dismiss it because no practical exploit exists have missed the point entirely. A second misread is treating this as a reason to prefer alternative PQC families such as hash-based (SLH-DSA) or code-based schemes without noting that the structural equivalence argument between GKP error correction and LWE noise would need to be replicated for those families independently before they can be declared safer by comparison. Finally, the paper's use of Shannon's theorem and Fano's inequality is sound in spirit but operationally incomplete: the fact that the secret's entropy is bounded does not imply that a polynomial-time extractor exists, only that the information is present in principle.

## Bottom Line

LWE-based PQC is the right transitional standard for today, but any claim of unconditional post-quantum security is false on its face. Systems with key material or signatures that must survive 10+ years should be architected for cryptographic agility — not as a defensive afterthought, but as a primary design requirement. The QRAM bottleneck is the only thing standing between the GKZ attack and a practical break, and that bottleneck is an active research target.

---

## Scores

| Dimension | Score (1–5) | Rationale |
|---|---|---|
| Technical significance | 4 | The four-domain synthesis (complexity, thermodynamics, QEC, quantum learning) is a genuinely integrated argument that goes beyond typical PQC criticism. The GKP/LWE structural equivalence and the Shannon/Fano entropy bound are non-trivial contributions to framing. Docked one point because no new algorithmic result is proven — the paper synthesizes known results rather than deriving new bounds. |
| Industrial relevance | 5 | Directly relevant to any system migrating to NIST-standardized PQC (ML-KEM, ML-DSA). The claim that "PQC-ready" is a premature classification has immediate consequences for procurement, compliance, and security architecture documentation. |
| Misinterpretation risk | 5 | Extremely high. The paper's rhetorical framing is aggressive enough that a non-specialist will conclude LWE is broken today, while a specialist might dismiss it as purely theoretical with no near-term consequence. Both readings miss the actionable middle ground. |

---

## Network-Task Extension: PFT Security Implications

### Why This Paper Is Relevant to the Post Fiat Network

Post Fiat's validator and node communication architecture depends on the security of its underlying cryptographic primitives for authentication, key exchange, and signature verification. If PQC migration is in scope — or if contributors make PQC-readiness claims about protocol components — this paper establishes the correct epistemic baseline for evaluating those claims. Specifically, any argument of the form "we use an LWE-based scheme and are therefore quantum-secure" is formally overclaiming under the framework this paper establishes: LWE security is QRAM-contingent, not theoretically unconditional.

### Network Risk Surfaces: What the Paper Maps Directly

**1. Key Lifecycle Risk**
LWE-based key encapsulation mechanisms (ML-KEM / Kyber) protect session keys in transit. The paper's thermodynamic argument proves the secret is not destroyed by noise — it is present in the ciphertext and extractable given sufficient quantum resources. For Post Fiat, this means any long-lived key material (validator identity keys, node bootstrapping keys, treasury-adjacent signing keys) secured under LWE-based KEM has a theoretical extraction path that depends solely on QRAM maturity. Keys must be treated as having a finite quantum-security shelf life, not a permanent one.

**2. Signature Assumption Risk**
ML-DSA (Dilithium) and FALCON are both LWE/NTRU-based signature schemes. If validator or node signatures migrate to these schemes under the belief that they provide unconditional post-quantum security, the paper's BQP argument applies directly: there is no mathematical proof these problems sit outside BQP, and their algebraic structure resembles the noisy hidden subgroup problem already known to be vulnerable to quantum Fourier methods. Any PFT contributor claiming that adopting ML-DSA closes the quantum signature risk is citing an unproven assumption as a guarantee.

**3. Validator and Node Communication Channels**
The paper's Shannon/Fano section proves that an LWE-protected channel does not thermodynamically erase the plaintext — it distributes it across noise variables that remain bounded and, in principle, recoverable. For validator-to-validator and node-to-orchestrator communication, this means channel security degrades gracefully from the perspective of an adversary with maturing quantum hardware: the adversary does not need to solve the problem today, only need to record ciphertexts now and break them when QRAM overhead decreases. This is the classical "harvest now, decrypt later" threat, but the paper provides the formal entropy bound that makes it concrete rather than speculative.

**4. Migration Readiness and Contributor Claims**
This is the highest-priority PFT implication. Reviewers and validators will encounter contributor proposals claiming PQC-readiness as a binary property — either a protocol is PQC-ready or it is not. The paper establishes that this binary framing is wrong. PQC-readiness is a function of QRAM advancement timelines, key rotation schedules, cryptographic agility infrastructure, and the specific security parameter regimes in use. A contributor who asserts "we adopted ML-KEM, therefore we are post-quantum secure" is making a claim that cannot be validated against an unconditional standard, only against a provisional engineering snapshot.

---

### Accept / Defer / Reject Framework for PQC-Readiness Claims

**ACCEPT** (claims that are epistemically sound under this paper):
- "LWE-based schemes are the best available transitional standard today given current QRAM constraints."
- "ML-KEM and ML-DSA provide strong computational hardness against known classical and near-term quantum adversaries."
- "Adopting NIST FIPS 203/204 is the correct baseline migration step for PQC preparation."
- "Cryptographic agility is required to respond to QRAM advances without protocol redesign."

**DEFER** (claims that require additional evidence or timeline assumptions before acceptance):
- "Our key material is secure for X years under post-quantum assumptions." — Requires explicit QRAM advancement timeline assumptions, not just reference to current hardware limits.
- "Adopting PQC eliminates the quantum threat to our signature scheme." — Requires clarification that this holds under current QRAM constraints only.
- "LWE hardness has been validated by NIST, therefore the security is proven." — NIST standardization reflects best available knowledge, not a mathematical impossibility proof. Defer until the contributor distinguishes empirical confidence from provable lower bounds.
- Any claim about NISQ-era safety margins for high-value secrets: the paper's hybrid variational quantum algorithm section documents that NISQ probing of LWE structure is already underway.

**REJECT** (claims that are directly falsified by this paper):
- "Post-quantum cryptography is unconditionally secure against quantum computers." — Directly contradicted; security is QRAM-contingent.
- "Injecting Gaussian noise into LWE permanently destroys the cryptographic secret." — Shannon's theorem and Fano's inequality formally disprove this.
- "No quantum algorithm can break LWE." — The GKZ algorithm breaks LWE in polynomial time given quantum samples; the constraint is state preparation cost, not mathematical impossibility.
- "Adopting a NIST-standardized PQC scheme closes the quantum risk surface." — It narrows the surface, it does not close it.
- "The quantum threat to LWE is purely a future concern with no near-term relevance." — NISQ hybrid attacks on LWE structure are documented in the paper's cited literature (Zeng et al., 2025).

---

### What Validator Reviewers Should Do Differently Because This Paper Exists

1. **Require explicit QRAM assumptions in any PQC security claim.** A contributor who cannot articulate what QRAM advancement scenario would invalidate their security argument does not understand the threat model they are claiming to address.

2. **Distinguish cryptographic agility from PQC migration.** Migration to ML-KEM/ML-DSA is necessary but not sufficient. The network needs key rotation infrastructure, algorithm negotiation protocols, and a process for responding to QRAM milestone events — contributors who deliver scheme replacement without agility scaffolding are solving the wrong problem.

3. **Flag "harvest now, decrypt later" exposure for any long-lived key material.** The paper's entropy bounds make this threat concrete: validator identity keys or any key material with a multi-year lifespan should be evaluated against the scenario where today's ciphertexts are stored by an adversary and broken retroactively.

4. **Treat the GKP/LWE equivalence as a forward-looking threat signal.** As fault-tolerant quantum error correction matures (GKP codes in particular), the same engineering progress that makes quantum computers reliable will reduce the cost of the attack described in this paper. Treat QEC hardware milestones as PFT cryptographic risk events.

5. **Do not allow the QRAM bottleneck to substitute for a security proof in contributor documentation.** The QRAM barrier is the current defense. It is shrinking. Contributors who cite it as a permanent safety margin are writing documentation that will become false on a hardware timeline, not a theoretical one.

---

## Verification
- **Public post / GitHub URL:** *[Publish to PFT public teardown repository — placeholder pending merge]*
- **Commit / post date:** 2026-05-07
