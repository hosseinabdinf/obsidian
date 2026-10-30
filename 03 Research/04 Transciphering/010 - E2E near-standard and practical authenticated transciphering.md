---
topic: Hybrid Homomorphic Encryption
date: "2023"
link: "[here](https://eprint.iacr.org/2023/1040)"
file: "[[2024-IACR-E2E E2E Near Standard and Practical Authenticated Transciphering.pdf]]"
tags:
  - HE
  - HHE
  - Transciphering
---
---
# Summary

Ehud Aharoni, Nir Drucker, Gilad Ezov, Eyal Kushnir, Hayim Shaul, and Omri Soceanu implement **AES-CTR, AES-GCM, and Ascon under CKKS**, then connect authenticated transciphering to encrypted **ResNet-50 inference on ImageNet-sized inputs**.

The main idea is to use CKKS slots for approximate bits, control their error with BLEACH cleanup polynomials, and exploit large batches. The important security distinction is explicit: **authenticated transciphering protects inputs from malicious storage, while the FHE computation service remains semi-honest**. Correct execution by a malicious evaluator is left to a stronger proposed notion, verifiable authenticated transciphering.

References below use the linked 17-page PDF. The local manuscript contains placeholder publication fields; its performance and design statements are those of this version.

## 1. Problem

Standard symmetric ciphers provide compatibility and established analysis, but their Boolean operations are difficult to evaluate efficiently with approximate CKKS arithmetic. Meanwhile, a transciphered output must still support a substantial application, not merely decrypt successfully after conversion.

Adding AEAD introduces another problem: the server can calculate a tag comparison only under encryption, and CKKS's approximate validity bit does not behave like an exact selector when suppressing invalid plaintexts.

## 2. Main Contribution

- A bit-sliced AES-CTR implementation using CKKS SIMD, oblivious S-box lookup, and selected bootstrap/cleanup placement (§§4–5).
- A BLEACH-based error analysis supporting cleanup after several gates rather than after every gate (§5).
- An informal **authenticated transciphering (AT)** threat model and discussion of the stronger **VAT** goal (§6).
- AES-GCM with homomorphic GHASH, plus an Ascon implementation with a different packing arrangement (§§7–8).
- GPU/CPU measurements and an end-to-end path from AES-GCM-encrypted image data to encrypted ResNet-50 inference (§§9–10).
- Discussion of approximate invalid-data suppression, numerical reconstruction, packing, and key management (Appendix C and §§10–11).

## 3. Main Techniques

| Technique | Why it helps |
| --- | --- |
| AES bit slicing across CKKS ciphertexts | One ciphertext per bit position, with many independent AES blocks in its slots |
| Pointer permutation for ShiftRows | Avoid CKKS slot rotations for this AES step |
| Boolean identities in arithmetic | Express AND and XOR as polynomials on approximately Boolean slots |
| S-box indicators | Compute table outputs by summing indicators selected by public output bits |
| BLEACH cleanup $h(x)=3x^2-2x^3$ | Contract small errors near zero and one |
| Bootstrapping after SubBytes | Refresh where the state is compact, before expensive derived work expands it |
| Packed powers of the GCM hash key | Parallelize GHASH products across blocks |
| Five-ciphertext Ascon state | Reduce state memory compared with a 320-ciphertext bit-sliced version |

## 4. Threat Model and Assumptions

§6 separates the cloud into:

- a **semi-honest FHE service** that follows the prescribed computation and does not learn plaintexts;
- **malicious external storage or other services** that may alter the client's stored symmetric ciphertexts.

The client authenticates record identifiers through AEAD associated data so substitution of records can be detected under the intended binding. The client retains the HE secret key. The service receives the necessary encrypted symmetric key material and HE evaluation keys.

AT propagates the AEAD validity decision to the client under HE. It does **not** prove that a malicious FHE service used the intended function, computed the tag correctly, or returned the correct validity bit. The paper introduces **VAT** as a future combination with verifiable computation.

The security of keys and their bindings is essential (§11). Replacing both a ciphertext and its associated encrypted symmetric key can defeat an intended authenticity policy if the service accepts untrusted key material. The paper calls for KMS/PKI support; this machinery is outside the demonstrated cryptographic computation.

For CKKS, use of CPA/CPAD-secure configurations and controlled decryption feedback is a separate requirement from the AEAD property. Approximation cleanup is a correctness mechanism, not a proof that ordinary CKKS is automatically secure against all decryption-feedback attacks.

## 5. Important Qualifications

1. **Throughput is heavily batched.** The AES state holds 32,768 blocks, representing 512 KiB. An amortized 3 ms per block does not mean a single block is available after 3 ms; the full GPU batch takes 1.65 minutes.
2. **The client expands AES keys offline.** Generation, clear key expansion, and HE encryption of the round keys occur once before the measured flow (§4.1, Remark 1).
3. **GPU and CPU rankings are not matched-hardware algorithmic comparisons.** Figure 2 applies assumed CPU-to-GPU normalization factors to older work; those dotted curves are estimates, not rerun competitor measurements.
4. **Reported runtime is the minimum of ten runs.** It is not an average, median, or uncertainty interval (§9).
5. **Approximate correctness must survive every stage.** A cleaned bit can still gain error during GHASH, integer reconstruction, packing, and inference. Bounds must account for CKKS operation error as well as ideal polynomial behavior.
6. **AT is not a complete replay policy.** Binding record identifiers in AAD helps prevent substitution, but freshness/version information needs its own application binding when required.
7. **Ascon batching is across independent streams.** Its evolving state prevents treating successive blocks of one stream as independent slots. The implementation fills slots with different keys or nonces (§8).
8. **The Ascon description is version-sensitive.** This paper uses the then-existing Ascon specification. Its prose about variant rounds contains ambiguities and should not be used as a present-day standardized implementation specification.
9. **Invalid-data suppression is subtle in CKKS.** Multiplying by an approximately zero validity selector can leave plaintext-dependent residuals. Appendix C proposes randomized masking, but the exact bounded-domain security/error guarantee is not proved as a formal theorem here.
10. **End-to-end does not mean a fully reported product benchmark.** The paper describes the integrated inference flow and says accuracy/latency align with the cited inference work, but does not give one complete stage-by-stage timing table for the demonstrated image.

## 6. Evaluation Setup

HElayers uses a **bootstrappable CryptoLab HEaaN context**, targeting **128-bit security**, with **32,768 slots**, depth **12**, scale **$2^{42}$**, 18 extra bits in the first prime, and bootstrap chain index restored to 12 with minimum index 3 (§9).

- **GPU:** NVIDIA A100 SXM4 with 80 GB GPU memory; host AMD EPYC 7763 at 2.45 GHz, 64 cores/128 threads. Host computation is restricted using `OMP_NUM_THREADS=1`.
- **CPU:** Intel Xeon E5-2699 v4 at 2.20 GHz, 44 cores/88 threads, 750 GB RAM.
- Each experiment is repeated ten times; the table reports the **minimum** runtime.

Selected **Table 2** results:

| Platform | Symmetric key length | Mode | Payload | Total latency (min) | Amortized latency (ms/block) | Peak RAM (GB) |
| --- | --- | --- | --- | --- | --- | --- |
| CPU | 128 | AES-CTR | 512 KiB | 20 | 37 | 69 |
| CPU | 256 | AES-CTR | 512 KiB | 28.58 | 52 | 74 |
| GPU | 128 | AES-CTR | 512 KiB | 1.65 | 3 | 14 |
| GPU | 128 | AES-GCM | 512 KiB | 6.95 | 13 | 14 |
| GPU | 256 | AES-CTR | 512 KiB | 2.3 | 4.2 | 14 |
| GPU | 256 | AES-GCM | 512 KiB | 7.66 | 14 | 15 |
| GPU | 128 | Ascon | 4 KiB | 0.43 | 50.8 | 22 |
| GPU | 128 | Ascon | 512 KiB | 11.5 | 10.49 | 23 |

AES blocks are 128 bits; the evaluated Ascon payload blocks are 64 bits. Thus the Ascon rate per eight-byte block should not be compared directly with AES's per-sixteen-byte-block rate. The 512 KiB Ascon case amortizes initialization/finalization over longer independent streams.

The inference demonstration uses **$224\times224\times3$ 32-bit values**, totaling **602,112 bytes = 588 KiB**. It therefore needs **two AES batches**, rather than the one 512 KiB batch used in the headline timings. The paper reports average post-AES-GCM absolute reconstruction noise around **$1.16\times10^{-10}$** for this experiment (§10), before the subsequent data-type/packing stages.

---
# Main Scheme

## CKKS Boolean Evaluation and Authenticated Transciphering

### 1. Notation

| Symbol | Meaning |
| --- | --- |
| $K$ | AES/AEAD secret key |
| $\widehat K$ | HE-encrypted symmetric key/round-key material |
| $A,N,C,T$ | Associated data, nonce, symmetric ciphertext, authentication tag |
| $H=\operatorname{AES}_K(0^{128})$ | GCM authentication subkey |
| $h(x)=3x^2-2x^3$ | BLEACH cleanup polynomial |
| $s$ | Encrypted approximate tag-validity selector |
| $\widehat m$ | CKKS ciphertext representing the recovered plaintext |

### 2. Bits and AES in approximate arithmetic

For ideal bits $x,y\in\{0,1\}$:

$$
x\land y=xy,\qquad x\oplus y=(x-y)^2,
\qquad x\lor y=x+y-xy.
$$

CKKS slots contain these values plus small errors. The paper uses cleanup to restore proximity to Boolean values after groups of gates.

AES is bit-sliced: one of 128 ciphertexts holds the same bit position from each of 32,768 independent blocks. The state is roughly 1.5 GB in the paper's ciphertext-size estimate. ShiftRows reorders pointers; it does not rotate slots inside ciphertexts.

For an S-box input byte with encrypted bits $x_i$, an ideal indicator for public table index $j$ is:

$$
I_j(\mathbf x)=\prod_{i=0}^{7}
\begin{cases}
x_i,&j_i=1,\\
1-x_i,&j_i=0.
\end{cases}
$$

Output bit $k$ is the sum of indicators whose public S-box entry has bit $k=1$. The implementation shares intermediate work; it reports **272 multiplications at depth three** for its chosen indicator construction (§4.2). The alternative appendix optimization was not used in the experiments.

A round's reported depth is nine: AddKey one, MixColumns three, ShiftRows zero, SubBytes three, cleanup two. Bootstrapping occurs after SubBytes, followed by cleanup. AES-CTR/GCM use AES *encryption* of public counter blocks even during symmetric decryption.

### 3. Clean errors after several gates

Near a Boolean value, ideal cleanup gives:

$$
h(e)=3e^2-2e^3,\qquad
h(1+e)=1-3e^2-2e^3.
$$

The linear error term disappears. Lemma 5.3 propagates a gate error bound $B$ through depth $d$, then bounds the cleaned deviation by:

$$
3(B+0.25)^{2d}e^2+2(B+0.25)^{3d}e^3.
$$

The paper uses this to select cleanup intervals, reporting that its mixed AES circuit can be cleaned once per round when the initial bound is below $2^{-29}$. A scale of $2^{42}$ is used in the experiments. This bound should be applied with the stated intermediate-error assumptions and with the numerical error of evaluating cleanup itself accounted for.

### 4. AES-GCM authentication under HE

The server generates the encrypted CTR keystream using encrypted round keys, removes the public ciphertext mask, and evaluates GHASH over AAD and ciphertext blocks using the encrypted subkey $H$.

GHASH operates in:

$$
\mathbb F_2[X]/(X^{128}+X^7+X^2+X+1).
$$

A safe specification-level description of the authentication recurrence is:

$$
Y_0=0,\qquad Y_i=(Y_{i-1}\oplus X_i)\,H,
$$

where the block sequence includes the prescribed AAD/ciphertext padding and length block. The expected tag combines the resulting hash with the AES-derived tag mask. Public ciphertext/AAD blocks are plaintext operands relative to HE; powers of $H$ remain encrypted.

To exploit SIMD, the implementation packs precomputed powers of $H$ in **HTBL**, roughly the size of another 128-ciphertext AES state. These powers can be prepared by the client, generated homomorphically by the server, or split between them (§7.2). The exact polynomial exponents and length handling must follow GCM's specification; the abbreviated printed summation is not a complete tag algorithm.

### 5. Carry the validity decision to the client

The server homomorphically compares the recomputed and received tags. The client receives either the relevant tag material or an encrypted validity indicator and releases the result only after accepting the check (§6).

A naive invalid-data gate,

$$
\widehat m_{\mathrm{gated}}=s\,\widehat m,
$$

is insufficient for exact suppression under CKKS: if an invalid selector decrypts to a small nonzero $\epsilon$, the product still contains $\epsilon m$.

Appendix C instead proposes constructing a random mask bit by bit: multiply mask bits by the encrypted invalidity decision, clean those bits, reconstruct the mask, and add it to the candidate plaintext. The proposed result is approximately unmodified data for a valid tag and randomized data for an invalid tag. Its concrete leakage guarantee depends on the domain, mask distribution, and approximation bounds. A validity indicator must still be returned; random output is not inherently distinguishable from a valid application value.

### 6. Numerical reconstruction and inference

After transciphering, data is in a bit-sliced representation. The server:

1. reconstructs values in the agreed numerical type;
2. repacks them to match the model's required tensor layout using rotations and masks;
3. evaluates the HE-compatible ResNet-50 model;
4. returns the encrypted prediction and authentication decision.

Scaling a bit by $2^i$ also scales its error. The paper uses additional powers in its reconstruction procedure to suppress errors associated with zero bits before weighted recombination. Correctness still depends on the original input's range and the CKKS integer/fractional precision; arbitrary 32/64-bit types cannot automatically be recovered exactly.

Ascon uses a separate packing strategy: five ciphertexts represent the five 64-bit state words, with independent streams packed across slots. This saves memory while introducing within-word rotations and preserving the serial dependency between blocks of each stream.

## Protocol

```mermaid
sequenceDiagram
    participant C as Client
    participant D as Potentially malicious storage
    participant S as Semi-honest FHE service
    C->>D: AEAD ciphertext, nonce, AAD, tag
    C->>S: Authenticated key binding and HE-encrypted symmetric key material
    C->>S: Request for records and application
    S->>D: Fetch requested records
    D->>S: Stored AEAD data
    S->>S: Transcipher and recompute tags under CKKS
    S->>S: Compute encrypted validity decision
    S->>S: Apply required invalid-data handling
    S->>S: Reconstruct numbers, repack, run encrypted inference
    S->>C: Encrypted prediction and validity information
    C->>C: Decrypt; release prediction only after acceptance
```

---
# Questions

1. **Formal AT definition:** §6 explicitly gives an informal threat model. What precise forgery/leakage game captures the allowed input modifications, encrypted-key access, and acceptance policy?
2. **Failure handling:** what statistical or computational guarantee does Appendix C's finite-range randomized mask provide for the actual plaintext range? Approximate masking should not be called exact AEAD rejection without that analysis.
3. **Cleanup proof:** the displayed Lemma 5.3 calculation analyzes ideal $h(f(\mathbf x))$. How is CKKS error from computing $h$, rescaling, and bootstrapping included in a complete bound?
4. **Version/specification checks:** the printed CTR nonce/counter split and Ascon variant-round description require verification against the intended primary specifications before implementation. The main idea does not depend on copying those ambiguous details.
5. **GHASH ordering:** the abbreviated powers-of-$H$ summation and Horner expression in §7.2 should be reconciled with the actual block order and length block. Public test vectors are needed to validate a concrete implementation.
6. **Packing security:** if different clients' keys share slots, how are key ownership, permissions, and output recipients bound to each slot?
7. **Complete application cost:** how much do two-batch transciphering, tag handling, number reconstruction, and model-layout permutations add to the demonstrated 588 KiB image's inference time?
8. **Future work:** VAT, efficient scheme switching, other AEADs, and automated joint packing are proposals here, not experimentally completed security guarantees.

---
# Related Topics

- [[000 - Transciphering HE]]
- [[011 - Efficient AES-GCM Decryption Under Homomorphic Encryption]]
- [[005 - SoK - FHE-Friendly Symmetric Ciphers and Transciphering]]
- [[016 - High-Throughput AES Transciphering using CKKS - Less than 1ms]]
- [[020 - XBOOT - Free-XOR Gates for CKKS with Applications to Transciphering]]
- [[032 - Transciphering Framework for Approximate Homomorphic Encryption (Full Version)]]
