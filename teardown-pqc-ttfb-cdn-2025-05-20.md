# Quantum Paper Teardown

## Thesis

Post-quantum certificate chains introduce discrete, bandwidth-threshold-governed latency penalties in TLS deployments, and the dominant mitigation levers are Merkle Tree Certificates (MTC) and session resumption — not propagation-delay reduction.

## Verdict

The paper succeeds in its core claim: it isolates certificate chain size as an independent latency variable, identifies specific flight-window thresholds (10 KB and 40 KB) where TTFB jumps by a full RTT, and shows that ML-DSA fits within those windows while SLH-DSA does not. It matters because it reframes PQC migration risk from a vague "bigger handshake = more latency" heuristic into a testable, architectable threshold problem with clear optimization paths.

## Tags

`post-quantum-cryptography`, `TLS-handshake`, `certificate-chain`, `CDN-latency`, `ML-DSA`, `Merkle-tree-certs`, `network-security`, `migration`

\---

## Paper Info

* **arXiv link:** https://arxiv.org/abs/2604.24869
* **Date:** 27 April 2026
* **Title:** Network Impact of Post-Quantum Certificate Chain Sizes on Time to First Byte in TLS Deployments
* **Authors:** Matthew Chou, Phuong Cao (University of Illinois at Urbana-Champaign / NCSA)

\---

## What the Paper Claims

PQC certificate chains (specifically ML-DSA and SLH-DSA) increase TLS handshake sizes by 5×–20× over classical schemes, and this size increase interacts with TCP's initial congestion window and slow-start thresholds to produce *discrete* RTT penalties rather than smooth latency degradation. The paper claims that ML-DSA stays within one bandwidth flight (≤10 KB chain), while SLH-DSA crosses a second threshold (\~40 KB), adding a full RTT. CDN session resumption (measured at 80–94% in real NCSA traffic data) largely bypasses these penalties in practice, and MTC can expand the "safe" chain size range by 2–3×.

## Mechanism in Plain Language

TCP's initial congestion window caps the first round-trip transmission at \~14 KB. If a TLS certificate chain exceeds this, the server must wait for an ACK before continuing, adding one RTT. A second threshold sits at \~42 KB (after TCP slow-start doubles the window). A certificate chain that crosses either boundary causes a step-function increase in Time to First Byte — not a gradual one. ML-DSA chains (\~8–15 KB) sit just inside the first threshold; SLH-DSA chains (\~60–150 KB) blow past both. Merkle Tree Certificates eliminate intermediate certificates entirely, replacing an N-certificate chain with a single compact proof (\~700–900 bytes), which keeps even large underlying certificates within one flight. Session resumption bypasses certificate transmission on repeated connections, reducing the problem to a one-time penalty on first contact.

The paper's experimental methodology is notable: rather than testing live PQC deployments end-to-end (which would conflate implementation overhead with network overhead), they inflate classical ECDSA certificates with non-critical extensions to match PQC sizes. This correctly isolates the *network* contribution of certificate size from cryptographic computation time. The OQS stack adds \~50–55 ms of implementation overhead independent of cert size, confirming that raw propagation delay is not the dominant term.

## What Matters Practically

The bandwidth flight-window threshold is the operative architectural constraint for PQC deployment, not RTT or raw bandwidth. Any PQC scheme whose certificate chain fits within \~10 KB (currently only ML-DSA in standard configurations) can be deployed in CDN environments without incurring meaningful TTFB penalties beyond the first, full handshake. SLH-DSA is effectively excluded from latency-sensitive infrastructure at current sizes. For non-CDN origin servers — where session resumption rates are \~35% vs CDN's \~80–94% — the penalty is fully exposed on the majority of connections and constitutes a genuine regression.

## Likely Misinterpretation

**Overshoot toward optimism:** "ML-DSA is fine, we can just migrate." The paper shows ML-DSA *currently* fits within one flight window with a leaf + one intermediate. Adding a second intermediate (11.9 KB) puts ML-DSA at the boundary. Hybrid ML-DSA (leaf + one intermediate \~9.5 KB, leaf + two \~14.3 KB) already crosses it. Anyone citing this paper to argue that PQC migration is smooth for hybrid schemes is misreading it.

**Overshoot toward pessimism:** "PQC latency is catastrophic." The session resumption data (94% for CDN TLS 1.3 connections) means the full-handshake penalty is rarely incurred in practice for CDN-fronted services. The paper does not claim that PQC makes CDN infrastructure unworkable — it claims it narrows the acceptable certificate chain design space.

**Category error:** Treating the \~50–55 ms OQS implementation overhead as a network result. It is not. That figure reflects the OpenSSL/OQS TLS stack, Docker containerization, and possibly key generation latency. It does not represent a fundamental PQC latency floor in optimized deployments.

## Bottom Line

Design PQC certificate chains to stay under the 10 KB first-flight threshold. ML-DSA with one intermediate does this; any hybrid or two-intermediate configuration needs MTC or CDN chain optimization to remain safe. Do not adopt SLH-DSA for any connection-latency-sensitive service without MTC. Treat session resumption rates as a *risk buffer*, not a solution — they disappear on cold connections, node restarts, and long-lived infrastructure where resumption state expires.

\---

## Scores

|Dimension|Score (1–5)|Rationale|
|-|-|-|
|Technical significance|4|First paper to isolate cert chain size as independent TTFB variable with controlled size-matching methodology; thresholds are concretely identified, not estimated|
|Industrial relevance|4|Directly actionable for CDN/TLS infrastructure design; real NCSA traffic dataset grounds the session resumption findings|
|Misinterpretation risk|4|The OQS overhead / network overhead conflation is easy to miss; "ML-DSA is fine" conclusion is one intermediate certificate away from being wrong|

\---

## Network-Task Extension: Validator \& Task Node Relevance

### Why This Matters for Task Node Grading

This paper is not about consensus or smart contract execution — it is about the authentication and identity layer underneath peer-to-peer TLS connections. For PFT validators, this is directly relevant to: (1) the TLS handshake cost when validators establish mutually authenticated connections, (2) the certificate size discipline required if PFT ever issues PQC-signed identity or signing certificates to validators, and (3) the latency exposure during cold-start events (new node joining, validator key rotation, restart after downtime) when session resumption is unavailable and the full certificate chain must be transmitted.

### Problem Analogy: PQC Deployment Constraints ↔ Validator Key Lifecycle Management

The core tension in the paper — that PQC algorithms are theoretically secure but their certificate structures break implicit assumptions in the transport layer (congestion window sizing, intermediate certificate count) — maps directly onto a validator identity problem. PFT validators make implicit assumptions about signing key size and handshake overhead when designing message throughput and peer connection budgets. Migrating validator signing to ML-DSA disrupts those assumptions in the same discrete, threshold-governed way the paper identifies for TLS: not a smooth regression but a step-function penalty when chain size crosses a flight limit. The paper's isolation methodology (size-matched ECDSA as a control) is a transferable design principle: any PFT PQC migration claim should separate cryptographic computation cost from network transmission cost from protocol implementation overhead — because conflating them, as OQS deployments currently do, obscures which bottleneck actually needs fixing.

### Shared Verification Methods or Transferable Heuristics

* **Threshold identification before migration:** The paper's 10 KB / 40 KB flight-window analysis should be replicated for PFT's peer connection protocol (likely libp2p or similar). Identify the initial send window for validator-to-validator TLS before assuming any PQC certificate scheme is viable.
* **Session resumption rate as a cold-start risk metric:** The paper's CDN vs. non-CDN resumption gap (94% vs. 46%) reveals that optimistic latency estimates assume warm connections. PFT validator nodes restart, rotate keys, and join fresh. Cold-start handshake cost under PQC is the relevant baseline, not steady-state cost.
* **Size-matched control methodology:** When evaluating PQC signing for transactions or validator messages, isolate size contribution from computation contribution by using padded classical signatures as a control. Do not report a single "PQC overhead" number.
* **MTC as a chain-compression primitive:** Merkle Tree Certificate logic — replacing N signed intermediates with a single compact proof — is architecturally available beyond TLS. For PFT governance or validator identity schemes, MTC-style proof structures could replace multi-hop certificate chains in validator credentialing.
* **Hybrid scheme penalty accounting:** The paper shows hybrid ML-DSA (classical + PQC) costs \~1 KB more per certificate and adds \~4 ms overhead attributable to implementation rather than size. PFT should not treat hybrid-PQC as a costless transition step.

### Failure Modes or Limits of the Analogy

The paper's CDN context assumes geographically distributed but relatively homogeneous infrastructure. PFT validators are geographically heterogeneous and may operate under significantly more variable RTT conditions than CDN edge nodes. The paper shows RTT doesn't affect *where* the thresholds occur, only the TTFB magnitude — so the threshold structure still applies — but the *cost* of crossing a threshold is proportionally larger for high-RTT validator pairs (international or poorly peered nodes). The paper also does not address UDP-based transports or QUIC, which many modern P2P networks use; the TCP congestion window model doesn't directly translate to QUIC's initial data limits, though analogous constraints exist.

### Implications for Validator Design, Scoring, or Review Workflow

PFT validators should not be granted credit for PQC readiness claims that cite ML-DSA adoption without specifying certificate chain depth and hybrid vs. pure mode. A hybrid ML-DSA validator certificate with two intermediates crosses the first flight threshold and incurs the same RTT penalty the paper documents for SLH-DSA — despite being nominally "ML-DSA." Task Node contributors making PQC migration claims should be expected to report: (1) certificate chain size including all intermediates, (2) expected session resumption rates for their node's connection pattern, and (3) whether their claim is evaluated under cold-start or warm-connection conditions.

For validator scoring purposes, "PQC-ready" should be graded on a spectrum: a node running pure ML-DSA with one intermediate and MTC-ready infrastructure is genuinely migration-ready; a node advertising hybrid ML-DSA with two intermediates and no session resumption optimization is in the penalty band. The paper's threshold framework provides the exact scoring axis.

\---

## PFT Implications: Accept / Defer / Reject Guidance

The following guidance applies to future contributor claims about PQC readiness, quantum-safe migration, or certificate-layer security on the PFT network.

### ACCEPT — Claims with evidence meeting this standard

|Claim Type|Accept Condition|
|-|-|
|"Validator X is ML-DSA ready"|Demonstrated: pure ML-DSA (not hybrid), one intermediate, chain ≤10 KB, tested on cold-start connection|
|"MTC reduces our PQC latency penalty"|Demonstrated: chain size measured before and after MTC, confirmed below 10 KB threshold in simulation or real deployment|
|"Our session resumption rate mitigates PQC overhead"|Demonstrated: resumption rate measured on actual validator connections, cold-start TTFB also reported|
|"PQC migration adds X ms to our validator handshake"|Accepted only if implementation overhead and network overhead are separately reported (size-matched control methodology used)|

### DEFER — Claims requiring more evidence before scoring

|Claim Type|Deferral Reason|
|-|-|
|"Hybrid ML-DSA is our PQC migration path"|Hybrid adds \~1 KB per cert and \~4 ms implementation overhead; chain size with two intermediates crosses the 10 KB threshold. Requires chain size audit before credit|
|"SLH-DSA is acceptable for low-frequency validator events"|The 40 KB threshold adds a full RTT even for infrequent connections; acceptable only if cold-start latency is contractually irrelevant for that use case — which must be argued, not assumed|
|"Our CDN fronts the validator, so PQC overhead doesn't apply"|CDN termination reduces, but does not eliminate, the problem for validator-to-validator (non-CDN) connections. Deferral pending evidence that all inter-validator paths are CDN-fronted|
|"TLS 1.3 adoption ensures session resumption benefit"|Paper shows TLS 1.3 adoption is necessary but not sufficient; non-CDN TLS 1.3 resumption is only 46%. Requires actual resumption rate measurement for this node's connection profile|

### REJECT — Claims that are contradicted by or ungrounded in the paper's findings

|Claim Type|Rejection Basis|
|-|-|
|"ML-DSA has negligible network overhead, PQC migration is transparent"|False at chain depth > 1 intermediate or in hybrid configurations; the paper explicitly documents threshold crossings. Blanket "negligible" claims are unsupported|
|"SLH-DSA is viable once networks upgrade their bandwidth"|Bandwidth is not the operative constraint. The congestion *window* (14 KB initial, not raw bandwidth) governs the threshold. Higher bandwidth does not move the threshold|
|"The \~50 ms OQS overhead is the PQC latency cost for PFT"|That figure is implementation overhead in an unoptimized Docker/OQS stack, not a fundamental PQC floor. Using it as a baseline for PFT latency projections is methodologically incorrect|
|"Post-quantum migration is complete once key exchange (ML-KEM) is upgraded"|Key exchange and certificate authentication are separate. The paper addresses the certificate layer specifically; ML-KEM adoption does not address certificate chain size penalties|
|"Q-Day is too far off to prioritize PQC migration now"|Out of scope for this paper's findings, but the "harvest now, decrypt later" threat documented in the paper background is active regardless of CRQC timeline estimates. Deferring certificate migration on timeline uncertainty ignores the asymmetric risk structure|

\---

## 

