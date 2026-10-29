---
topic: secure aggregation
date: "2026"
link: "[here](https://eprint.iacr.org/2026/346)"
file: "[[2026-USENIX-Lighthouse Single-Server Secure Aggregation with O(1) Server-Committee Communication at Scale.pdf]]"
tags:
  - "#TE"
---
# Summaary
---
## 1. Problem

Each client holds a private vector $x_i \in \mathbb{F}^{\ell}$. The server should learn only the aggregate:

$$
s = \sum_{i \in U} x_i,
$$

where $U$ is the accepted set of contributing clients.

A small **committee of decryptors** helps the server recover the sum while protecting individual inputs. Previous protocols can place substantial computation and communication costs on this committee.

## 2. Main Contribution

In the base protocol, Lighthouse achieves:

- $O(1)$ **computation** per committee member.
- $O(1)$ server–committee **communication** with respect to client count $n$ and vector dimension $\ell$, holding committee size and security parameters fixed.
- One message per ordinary client per aggregation, after which the client can disconnect.
- Two round trips per aggregation, excluding setup.

These constant-cost claims concern committee overhead, not total system computation or communication.

## 3. Main Techniques

- **Pairwise masking:** Masks shared between contributing clients cancel when their masked vectors are summed.
- **Batched threshold encryption:** The committee authorizes many mask decryptions through compact responses.
- **Succinct proofs:** The server proves that its committed participation set satisfies the required checks.
- **One-of-two decryption:** For each client, the server can recover either the individual mask or the relevant pairwise masks, according to the committed status, but cannot recover both.

## 4. Threat Model and Assumptions

- A malicious adversary with static corruption.
- The server, some clients, and some committee members may be corrupted.
- Client corruption and dropout fractions satisfy:

$$
\eta + \delta < \frac{1}{3}.
$$

- Committee corruption and dropout fractions satisfy:

$$
\eta_D + \delta_D < \frac{1}{3}.
$$

- Authenticated communication, a verifiable PKI, and public randomness.
- A one-time distributed key generation procedure for the committee.

## 5. Important Qualifications

- **Single-server still involves an auxiliary committee.**
- Security permits abort; guaranteed output delivery is not provided.
- Malicious-input validation and poisoning resistance are outside scope.
- The dynamic-participation extension introduces committee computation depending on newly joined clients and communication depending on vector dimension.
- Lower committee overhead comes with additional server work. Table 3 reports higher server computation than Flamingo.

## 6. Evaluation Setup

- 64–1024 clients.
- 16K-dimensional input vectors with 32-bit entries.
- 60 decryptors.
- 10% dropout rate.
- Direct experimental comparison with Flamingo.
- Python protocol logic with Rust cryptographic components.

## Next Reading Step

Understand how pairwise masks cancel, why dropouts leave residual masks, and how the committee enables their removal without exposing individual inputs.

---
## Main Scheme: Static-Participation Lighthouse




## Examples


## Questions
- 

## Summary


## Related Topics
- [[]]