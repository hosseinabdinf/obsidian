---
topic: Hybrid Homomorphic Encryption
date: "2025"
link: "[here](https://eprint.iacr.org/2024/1702)"
file: "[[2025-IACR-MPC Secure and Efficient Transciphering for FHE Based MPC.pdf]]"
tags:
  - HE
  - HHE
  - Transciphering
---
---
# Summary

## 1. Problem

FHE-based MPC may use a cipher under jointly generated keys whose differences are known to malicious participants. Ordinary single-key confidentiality does not cover this situation. **Secure and efficient transciphering for FHE-based MPC**, Aranha, Guimarães, Hoffmann and Méaux, analyzes this related-keystream problem and accelerates mixed-filter-permutator evaluation using leveled lookup tables. Source: local 36-page paper, §§2–5 and the stated supplementary analysis.

## 2. Main Contribution

The paper defines the **two-related-keystream setting** K and K⊕Δ, where the attacker chooses Δ, and studies Margrethe-18-4 in that setting. It relates the attack surface to first derivatives of the filter and gives concrete Boolean-function analyses, with explicit residual assumptions. It also explains how more related keys can expose higher-order derivatives and degrade security.

The evaluation contribution replaces chains of small bootstrapped functions with a single larger vertical-packing lookup wherever their inputs are shared. For Margrethe this gives fourteen independent 18-bit-input/four-bit-output LUT evaluations, followed by a sum. Native binary key whitening is handled by selecting a ciphertext or its complement. These operations avoid recurrent circuit bootstrapping when the key bits already have suitable RGSW encryptions.

## 3. Main Techniques

Margrethe's 2,048-bit key supplies 308 selected/whitened bits per nibble. Fourteen 22-bit groups each contribute one fixed 18-to-4 lookup plus a four-bit integer assembled from the remaining bits. The contributions sum modulo 16. The large table is public and the key selectors are encrypted.

For Elisabeth-style modular whitening, the analogous method modifies the **public LUT** to absorb whitening, avoiding an encrypted modular-addition circuit. Original Elisabeth's speed does not restore its broken security; the paper distinguishes patched variants.

Multi-party key mixing uses a gadget-domain XOR, supporting different gadget decompositions but requiring the same HE secret key. For bits a,b, evaluate a(1−2b)+b row by row through external products. Optional packed-key compression and expansion is a separate setup/latency tradeoff.

## 4. Threat Model and Assumptions

The MPC motivation includes malicious parties and jointly contributed encrypted symmetric keys with proofs of correct encryption. In a static protocol, a known input encrypted under one honest party's key and later re-encrypted under the joint key can reveal keystreams under K and K⊕Δ. The single-honest-party example makes Δ the XOR of malicious contributions. Simply choosing different IVs is not a general RKA countermeasure.

The concrete analysis uses the stronger same-IV comparison and a single chosen XOR relation. **It does not prove general RKA security for arbitrary key transformations or arbitrary numbers of related keys.** The full MPC protocol additionally needs distributed key generation, threshold decryption, proofs and its own adversary/availability assumptions. Dishonest-majority decryption requiring all parties is not automatically guaranteed-output-delivery security.

## 5. Important Qualifications

The two-stream key-recovery analysis examines derivatives of the Boolean filter and group combinations. The reported minimum derivative degree/nonlinearity are concrete computations, but higher guessed-bit cases rely on an **assumed algebraic-immunity lower bound of six** for derivative components. Dense monomial behavior is sampled for many derivatives, not exhaustively proved for every derived equation system. Thus “out-of-the-box secure” means supported against the analyzed strategies under those assumptions, not a reduction proving all malicious MPC uses secure.

For a t-dimensional XOR space of offsets, **2^t related streams** expose a t-th derivative; degree can drop by t. Dynamic membership/key resharing may create such additional relations. The two-stream conclusion cannot be reused without counting the actual relations a protocol exposes.

The differential-fault connection is restricted to a key-register fault producing the same known XOR-offset observations. It does not establish general physical fault resistance. Native Margrethe evaluation has no bootstrap/circuit-bootstrap phase, but obtaining the large RGSW-encrypted key still costs communication or conversion work.

Output failure probabilities use a Gaussian tail calculation from **measured noise standard deviations**, not a rigorous empirical verification of tails as small as 2^-1000. Aggregation, error correlation and later PBS correctness need separate budgets. The paper's abstract says 3.9 ms, whereas the detailed fastest parallel table gives 3.12 ms; the detailed values are retained below.

## 6. Evaluation Setup

MOSFHET on AWS c6i.metal, Intel Xeon 8375C at 3.5 GHz. At least 100 executions per timing; generated streams are checked against a clear Python Margrethe implementation. Parallel evaluation uses up to fourteen threads **within one output**, not multiple independent output streams. Code: [MARGRETHE](https://github.com/antoniocgj/MARGRETHE).

**Tables 4–5:**

| Parameters | RGSW-key size | Nibble latency, one thread | Nibble latency, parallel | Parallel output rate |
|---|---:|---:|---:|---:|
| Set 1, N=2,048, q=2^52, one base-2^23 digit | 128 MB | 27.2 ms | 3.12 ms | 1,282.05 bit/s |
| Set 2, two base-2^17 digits | 256 MB | 54.2 ms | 4.21 ms | 950.11 bit/s |

One-thread rates are 147.06/73.8 bit/s. The separate plaintext-independent comparison Table 6 uses **25.2 ms** per four-bit unit for Set 1; that differs from Table 5's 27.2 ms and is not silently normalized here. Extending to more precision combines nibble streams and is constrained by output noise.

Two full 2,048-bit keys can be mixed using MP1/MP2 parameters in **430 ms on one thread**; subsequent cipher evaluation uses the Set-2 cost. Compression discussion projects around 287 seconds to expand 2,048 selectors using an external method; that projection is not included in native evaluation timings. Comparator ciphers often run on different hardware or use different failure targets. The fastest single-bit FiLIP row, the larger-integer FiLIP rows and Margrethe's four-bit rows have different payloads.

---
# Main Scheme

## Title

Leveled Margrethe evaluation with explicitly limited related-key exposure

### 1. Notation

K∈F₂^2048. Public subset/permutation/whitening yields x∈F₂^308. Let g:F₂^18→Z₁₆ be the specified fixed table; each group consists of eighteen table-input bits and four additional nibble bits. Define

$$
f(x)=\sum_{b=0}^{13}\left(g(x_{22b},\ldots,x_{22b+17})
+\sum_{j=0}^{3}2^j x_{22b+18+j}\right)\pmod {16}.
$$

### 2. Homomorphic evaluation

Provision RGSW encryptions of key bits. Apply public selection/permutation by indexing and whitening by C↦RGSW(1)−C when the whitening bit is one. For each group, vertical-pack the large public table using the first eighteen encrypted selectors. Convert the remaining four selectors to the required weighted LWE nibble, then add all fourteen contributions modulo 16. Client sends c=m+f(x); subtract the encrypted mask to recover the encrypted message.

### 3. Related-key analysis and protocol use

For a binary component F of the nibble filter, matched related outputs expose

$$
D_\Delta F(x)=F(x)\oplus F(x\oplus\Delta).
$$

Public subset/permutation maps Δ into the corresponding selected direction. Analyze degrees, nonlinearity, algebraic immunity and guessed-bit descendants of these derivatives. For multiple offsets forming a vector space, replace the first derivative by the corresponding higher-order derivative and reassess security. Correct encryption proofs for key contributions do not prevent such deliberate chosen-offset relations.

## Protocol

```mermaid
flowchart LR
    K[Proved encrypted party key contributions] --> M[Gadget-domain XOR key mixing]
    M --> S[Public selection and binary whitening]
    S --> V[Fourteen independent vertical-packed LUTs]
    V --> A[Add weighted nibble terms modulo 16]
    A --> T[Transcipher input or stored MPC value]
    T --> P[Threshold-FHE MPC with its own security protocol]
```

# Questions

- Exactly how many related keys can static or changing membership expose?
- Which assumptions support derivative algebraic immunity after concentrated guesses?
- Does selector compression save enough bandwidth to justify its setup time and changed noise?

# Related Topics

- [[000 - Transciphering HE]]
- Margrethe, mixed filter permutators, related-key attacks, higher-order derivatives, threshold FHE, vertical packing.
