---
topic: Hybrid Homomorphic Encryption
date: "2022"
link: "[here](https://eprint.iacr.org/2023/480)"
file: "[[2023-IACR-Hash Practical Homomorphic Evaluation of Block Cipher Based Hash Functions with Applications.pdf]]"
tags:
  - HE
  - HHE
  - Transciphering
---
---
# Summary

Adda Akram Bendoukha, Oana Stan, Renaud Sirdey, Nicolas Quero, and Luciano Freitas investigate **homomorphic hashing using block-cipher-based compression functions**. The contribution is an implementation study of familiar hash constructions under TFHE, motivated by integrity checks and protocols that need to hash secret data without revealing it.

The paper evaluates single-block-length Davies–Meyer and double-block-length Hirose/Tandem-DM constructions using PRINCE, SIMON, SPECK, and LowMC. The fastest reported double-length result is **1.28 minutes for a 128-bit digest of a 128-bit encrypted message**, using PRINCE/Hirose. A 128-bit digest does **not** imply 128-bit collision security.

Section and table references refer to the linked 16-page PDF.

## 1. Problem

Transciphering protects data confidentiality and reduces upload expansion, but it does not automatically detect corrupted data or supply every cryptographic operation required by a larger protocol. Homomorphic hashing is useful when both the message and its digest must remain hidden during processing.

Ordinary lightweight hash designs can still be expensive under TFHE. The authors therefore reuse block ciphers with comparatively manageable Boolean circuits and wrap them in established compression-function constructions.

## 2. Main Contribution

- A study of block-cipher-based hashes for FHE, considering both digest size and encrypted-domain execution cost.
- TFHE evaluation of Davies–Meyer, Hirose, and Tandem-DM instantiations with several lightweight/HE-oriented block ciphers.
- Four motivating use cases: encrypted integrity checking, single secret leader election, private database queries, and oblivious authentication (§3).
- A Cingulata-based implementation and multicore performance comparison (§4).

The construction-level security arguments are inherited from cited ideal-cipher/black-box analyses. The paper does not establish new concrete cryptanalysis bounds for every chosen cipher/hash pairing.

## 3. Main Techniques

| Technique | Purpose |
| --- | --- |
| Iterated compression function | Process multiple message blocks into a fixed-size digest |
| Single-block-length hashing | Obtain an $n$-bit digest from an $n$-bit block cipher |
| Double-block-length hashing | Increase the digest to $2n$ bits, improving the generic collision bound |
| Hirose's parallel calls | Permit parallel block-cipher evaluations within one compression step |
| Tandem-DM's dependent calls | Provide a comparison with a serial two-call construction |
| TFHE gate bootstrapping | Evaluate long Boolean circuits without fixing one shallow multiplicative-depth budget |
| Cingulata and ABC | Generate and optimize Boolean circuits, propagate constants, and schedule parallel execution |

LowMC's low multiplicative complexity does not guarantee good performance in the evaluated TFHE mode: its many XOR gates also incur substantial cost.

## 4. Threat Model and Assumptions

For the basic computation, a server evaluates a public hash function on HE-encrypted inputs without possessing the HE secret key. Confidentiality relies on the underlying FHE security and its parameterization.

Hash-security claims are analyzed in an **ideal-cipher/black-box model**, which treats the underlying keyed block cipher as a random permutation family. Applying such a generic result to PRINCE, SIMON, SPECK, or LowMC additionally requires confidence in the relevant concrete cipher and its use as a compression primitive.

The application sketches introduce additional assumptions:

- Encrypted integrity bits are evaluated correctly by the server; the paper does not give a general proof of malicious-server execution.
- The leader-election sketch relies on threshold FHE, adequate honest participation, common randomness, and the cited distributed protocol's assumptions.
- The authentication sketch needs a suitable keyed authentication primitive and challenge protocol, not merely collision resistance of an arbitrary unkeyed hash.

## 5. Important Qualifications

1. **Digest length and security strength differ.** For a $d$-bit digest, generic collision search takes about $2^{d/2}$ work. PRINCE's 128-bit double-length digest therefore offers at most roughly **64-bit generic collision security**, even when preimage resistance can target 128 bits (§4.1).
2. **Hashing is not automatically authentication.** Encrypting $m\parallel H(m)$ and checking the relation is a useful consistency check, but the paper does not prove that this generic composition gives AEAD security against active attackers.
3. **A server can compute the check without learning its outcome.** The integrity/authentication decision is itself encrypted. Enforcement requires encrypted selection, client decryption, or an additional protocol.
4. **No verification of arbitrary computation follows from the hash check.** A malicious evaluator could return an incorrect check or application result unless the larger protocol verifies it.
5. **Measurements are for short messages.** Table 1 hashes a 128-bit encrypted input; it is not a sustained bandwidth measurement on large files.
6. **SIMON results are estimates.** Footnote 12 explicitly derives them from gate counts and TFHE gate-bootstrap timing, rather than a complete measured SIMON hash implementation.
7. **Padding/domain details matter.** Merkle–Damgård security relies on a suitable message encoding/padding rule and correct compression instantiation. The short algorithm in §2.3 is not a complete specification for arbitrary-length input handling.
8. **The printed Davies–Meyer formula needs checking.** Its feed-forward term is inconsistent with the standard chaining-value form; see Questions before using that equation in an implementation.

## 6. Evaluation Setup

The paper uses **Cingulata in TFHE mode**, with OpenMP parallelism where possible, on an **Intel Xeon E3-1240 v5 at 3.50 GHz with 8 GB RAM** (§4.4).

**Table 1:** time in **minutes** to hash a **128-bit TFHE-encrypted message**. A cipher tuple denotes block length and key length in bits.

| Cipher | Davies–Meyer, SBL | Hirose, DBL | Tandem-DM, DBL |
| --- | --- | --- | --- |
| PRINCE (64,128) | — | 1.28 | 2.98 |
| SPECK (128,128) | 3.78 | — | — |
| SPECK (128,256) | — | 4.91 | 8.16 |
| SIMON (128,128), estimated | 2.14 | — | — |
| SIMON (128,256), estimated | — | 3.64 | 7.05 |
| LowMC (128,128) | 6.12 | — | — |
| LowMC (128,256) | — | 8.58 | 17.32 |

The PRINCE double-length rows perform **two iterations** for the 128-bit input and return a 128-bit digest. The other rows perform one iteration; their SBL digest is 128 bits and DBL digest is 256 bits (§4.4).

The table does not provide matched uncertainty intervals, a detailed global decryption-failure bound, or a common modern concrete-security reassessment. Its most direct result is the relative cost of these implementation choices on the stated setup.

---
# Main Scheme

## Encrypted Hash Computation and Consistency Checking

### 1. Notation

| Symbol | Meaning |
| --- | --- |
| $E_K$ | Underlying block-cipher permutation |
| $n$ | Block-cipher block length |
| $F$ | Compression function |
| $h_i$ | Chaining value after message block $i$ |
| $H(m)$ | Final hash digest |
| $[x]$ | FHE encryption of $x$ |
| $r$ | Encrypted equality/consistency decision |

### 2. Build and evaluate the hash circuit

Choose a compatible compression construction and cipher:

- SBL requires the appropriate block/key format and returns an $n$-bit chaining state.
- The evaluated DBL constructions use an $n$-bit block cipher with a $2n$-bit key and maintain a $2n$-bit chaining state.
- Hirose uses two independently evaluable cipher calls per compression step; Tandem-DM's second call depends on the first.

For encoded message blocks $m_1,\ldots,m_L$ and initialization value $h_0$:

$$
h_i=F(h_{i-1},m_i),\qquad H(m)=h_L.
$$

The server evaluates the same circuit on encrypted data:

$$
[h_i]=\operatorname{HE.Eval}(F,([h_{i-1}],[m_i])).
$$

Message-dependent cipher keys inside a compression function are also encrypted circuit inputs. This is different from simply hashing with one public fixed key.

The paper cites the original Hirose and Tandem-DM specifications rather than fully specifying their rounds; their security depends on the exact feed-forward and domain-separation structure, so this note does not replace those specifications with an informal two-call variant.

### 3. Integrity-check sketch after transciphering

The client computes $h=H(m)$, symmetrically encrypts $m\parallel h$, and sends that ciphertext plus the HE-encrypted symmetric key (§3.1).

After transciphering, the server has encryptions of a candidate message $m'$ and transmitted digest $h'$. It computes an encrypted digest $[H(m')]$ and the bitwise equality:

$$
[r]=\prod_{j=0}^{d-1}
\bigl(1\oplus[H(m')_j]\oplus[h'_j]\bigr).
$$

Each factor is one exactly when the two digest bits agree. The product is Boolean AND. Its meaning is **digest consistency**; collision resistance and the protocol assumptions are needed to relate that to tampering detection.

The server may return $[r]$ or select an application result under encryption:

$$
[y]=[r]\,[f(m')]+(1-[r])\,\bot.
$$

Here $\bot$ must have a suitable application encoding. This selection assumes the server actually evaluates the specified circuit.

### 4. Other application sketches

**Single secret leader election (§3.2):** peers register encrypted proofs $p_i=H(h\parallel t_i\parallel c)$. A threshold-FHE computation selects an encrypted proof and identifier, hashes them into a voucher, and threshold-decrypts that voucher. The selected peer recognizes it and later proves election by revealing the corresponding proof. Full consensus, adversarial, and threshold details belong to the cited leader-election protocol.

**Database queries (§3.3):** homomorphically hash a private query into an encrypted index, construct an encrypted one-hot vector of equality tests, and take a dot product with database records. Collision resolution remains part of the database design; a hash does not remove the encrypted-selection cost.

**Oblivious authentication (§3.4):** the server uses encrypted authentication key material to compute an expected challenge response, compares it with the supplied response under FHE, and gates the result with the encrypted success bit. The server need not learn the authentication outcome.

## Protocol

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>C: Compute h = H(m)
    C->>S: Symmetric encryption of m and h; HE encryption of symmetric key
    S->>S: Transcipher message and digest into HE
    S->>S: Evaluate H on the encrypted recovered message
    S->>S: Compare computed and received digests under HE
    S->>S: Optionally gate application output by encrypted check
    S->>C: Encrypted result and/or encrypted consistency bit
    C->>C: Decrypt and apply acceptance policy
```

---
# Questions

1. **Davies–Meyer transcription:** §2.3 prints $E_{M_i}(H_i)\oplus M_i$. The usual chaining-value feed-forward is $E_{M_i}(H_i)\oplus H_i$. Which circuit did the implementation evaluate? The printed equation should not be copied into a new implementation without checking.
2. **Integrity claim:** what active-adversary security is intended for encrypting $m\parallel H(m)$? A consistency relation and a collision-resistant hash alone are not a complete authenticated-encryption proof.
3. **Security target:** does the application need collision resistance or only preimage/second-preimage resistance? The answer changes whether a PRINCE-based 128-bit digest is adequate.
4. **Benchmark scope:** what padding, message-block handling, TFHE parameter set, and thread count produced each row? The short-message table leaves these important for reproduction.
5. **Leader-election composition:** how are malicious registrations, partial decryptions, and verification handled? The paper supplies a motivation sketch, not the entire distributed-security proof.
6. **Estimated versus measured:** the SIMON timings should remain labeled estimates, and Table 1 should not be presented as uniformly measured implementations.

---
# Related Topics

- [[000 - Transciphering HE]]
- [[009 - Optimized Stream-Cipher-Based Transciphering by Means of Functional-Bootstrapping]]
- [[025 - Ciphers for MPC and FHE]]
- [[010 - E2E near-standard and practical authenticated transciphering]]
