---
topic: Transciphering notes source audit
date: "2026-10-30"
tags:
  - HHE
  - Transciphering
  - Research-Review
---
# Transciphering notes — source audit

**Assessment:** the 21 substantive paper notes are broadly consistent with their linked local PDFs after the corrections below. Their benchmark qualifications and distinctions between confidentiality, authentication, and verified computation are important to retain. Several source formulas/counts remain unresolved; the affected notes are reading notes, not validated implementation specifications.

**Scope:** all 61 paper pages were checked for content and PDF availability. There are **21 substantive notes**, **35 template-only pages with PDFs**, and **five template-only pages without local PDFs** (003, 007, 013, 019, 022). All 21 substantive notes were read and cross-checked against the relevant source sections, definitions and selected benchmark tables. This audit does not complete the remaining paper readings.

The two PDFs physically in `PDFs/04-Transciphering` are HHE Land and HHE Kombat. The other 54 PDFs referenced by this collection are in `PDFs`; the audit followed those links too. See [[000 - Transciphering HE]] for the paper-to-PDF links and reading status.

## Corrections and clarifications

1. **CPA composition (001, 026).** The notes now distinguish flaws/missing arguments in a paper's proof from failure of the underlying construction. Under independent key generation, HE IND-CPA including its public evaluation material, and the symmetric scheme's CPA/nonce assumptions, an encryption-of-zero hybrid supports the basic CPA composition. The argument is explicitly labeled independent review reasoning. It establishes neither CCA/CPAD security nor malicious-server correctness. In particular, the Kreyvium paper's historical statement is not presented as a general impossibility result.
2. **Multiplexer example (006, 014).** The printed weights $(1,1,2)$ modulo seven fail the separation condition. For the printed function $f(a,b,c)=a$ if $c=1$, and $b$ otherwise, replacing the weights by **$(1,3,2)$** gives exactly the printed classes $P_0=\{0,1,2,5\}$ and $P_1=\{3,4,6\}$. All eight inputs were checked. This repairs the mathematical example; it does not establish which weights the authors' implementation uses.
3. **LUT coefficient search (006).** The note now states that the chain and coefficients $d_{i,k}$ are fixed before solving for $\beta_{i,j}$. The resulting system is linear in $\beta$; solving for both coefficient families simultaneously would be bilinear. Source: thesis §7.3, printed pp. 101–103, PDF pp. 125–127.
4. **Sub-millisecond normalization (016).** The displayed batch total gives $1000\cdot32.52/32768=0.992431640625$ ms/block and $512/32.52\approx15.74$ KiB/s. It rounds to Table 5's 0.99 ms, but not exactly to the abstract's separately reported 0.994 ms. The note now preserves that small discrepancy instead of claiming exact compatibility. This is amortized throughput; the batch still takes 32.52 s.
5. **PRINCE classification (018).** Removed a criticism that was inappropriate for the journal version: footnote 3 on PDF p. 3 explicitly says PRINCE is an exception to its standardization criterion.
6. **Attack scope and initialization (009, 025).** The masking counterexample is explicitly about the changed wrapper in §3.4, not standard Grain128-AEAD or digit packing. The LowMC public-generator description now identifies the discarded range $a_{80},\ldots,a_{239}$ and first used bit $a_{240}$, removing initialization ambiguity.

## Source issues that remain open

These are already identified in the individual notes; reproducing a source claim does not resolve it.

- **HHE Land (001):** Theorem 1's printed game hops replace HE outputs with uniform ciphertext-space elements. Ordinary IND-CPA does not imply that property. The independent CPA hybrid above avoids it; it is not an endorsement of the printed reduction.
- **HHE Kombat (002):** Table 16 labels server rates KB/s although they match bit/s values in Tables 10–14. The Table 14 TFHE baseline's time differs by about a factor of 1000 from its cycles/throughput. Its latency needs clarification. Smaller memory/payload ratios are better.
- **Hash functions (008):** the printed Davies–Meyer equation feeds forward the message instead of the chaining value. The note flags this and does not assume the implementation made the same error. Digest consistency is not by itself an AEAD proof.
- **Packed Grain (009):** with the printed modified mask $c=m+z\bmod2B^2$, $m,z\in\{0,\ldots,B-1\}$, ciphertext supports depend on the message even for uniform $z$. The performance analysis does not justify the claim of unchanged security.
- **ChaCha (023):** the proposed scaled-XOR and LUT-packing expressions need a consistent encoding/conversion construction. Correct adder truth tables and operation totals do not validate those transitions. Only the binary baseline is measured as a full cipher.
- **Hippogryph (024):** the note's repaired recomposition is valid at the ideal residue level. Accumulator construction, discretization, noise, and implementation behavior were not reproduced. The published Thunderbird comparison is not a completely reproduced common-host measurement.
- **Block-cipher comparison (017, 018):** several prose counts disagree with the comparison table. Keep the reported runtimes and count discrepancies distinct. The two PDFs substantially overlap and are not independent evidence.
- **Kreyvium (026):** displayed depth/length formulas need the 1152-clock initialization offset when interpreting usable output; optimized BGV/FV workloads also differ. The 64-versus-66 initialization-ones discrepancy is recorded.

## Per-note review record

“Reviewed” means the note was checked against the relevant local source and mathematical reasoning where applicable. It does not certify every theorem or current cryptanalytic security claim.

| ID | Checks | Result / qualification |
| --- | --- | --- |
| 001 | HHE interface, Table 5, security game and proof | Clarified the independent-key CPA hybrid; printed uniform-ciphertext game hops remain unsupported. |
| 002 | Benchmark definitions, security classes, Tables 10–16 | Confirmed bit/s versus KB/s discrepancy, memory-ratio direction, and inconsistent TFHE baseline time. |
| 004 | Taxonomy, RtF path, Tables 2–4 | Selected figures match the survey; preserve original-source and variant-specific security caveats. |
| 005 | Method taxonomy, Tables 2–4, applications | Selected figures and application sizes match; retain conversion exclusions and Trivium security exception. |
| 006 | Encoding gadgets, AES, Transistor, LUTs, ORPHEUS | Added a checked multiplexer repair and specified which LUT coefficients are fixed before the linear solve. |
| 008 | Compression constructions, Table 1, integrity sketch | Selected timings match; SIMON is estimated and the printed Davies–Meyer feed-forward remains suspect. |
| 009 | Packed operators, masking adaptation, Table 1 | Confirmed mask-support counterexample; explicitly limited it to the modified wrapper. |
| 010 | Boolean arithmetic, cleanup, GHASH, AT, Table 2 | Selected timings and 588 KiB image accounting match; AT assumes a semi-honest FHE service. |
| 011 | Poster scope and Table 2 | Confirmed poster-specific timings, minimum-of-ten methodology, and amortized units. |
| 012 | LUT/MVB scheme, parameters, Tables 3–5 | Confirmed 28.73 s is extrapolated; measured AMD 16-thread result is 36.39 s. |
| 014 | Gadget criterion, examples, primitive counts/timings | Verified five-input gadget; added corrected multiplexer weights with all eight inputs checked. |
| 015 | Trace preprocessing, HalfCBS, Tables 3/5/7–9 | Selected figures match, including 95.03 ms NTT and 56.12 ms integer-input conversion; keep local-failure scope. |
| 016 | Context conversion, layouts, Tables 1–6 | Corrected normalization to 0.99243 ms/block and separated the abstract’s 0.994 ms claim. |
| 017 | BR/packing-KS model, cipher sections, Table 1 | Confirmed overlapping version and prose/table count discrepancies; Hippogryph column is estimated. |
| 018 | Journal version, footnote 3, cipher counts, Table 1 | Corrected unfair PRINCE-standardization criticism; journal explicitly supplies the exception. |
| 020 | Parity/scale mechanism, AES/Rasta, benchmark and GWAS scope | Ideal parity identities and reported selected results check out; client performs final statistic division. |
| 021 | SM4 recurrence, inversion, packing, timing | Inversion chain verified on all 256 field elements; selected batch and timing figures match. |
| 023 | Quarter round, adders, scaled XOR, Table 1 | Adder truth table and counts check out; optimized encoding transitions and timings remain proposals. |
| 024 | Representation conversion, recomposer, Tables 1–4 | Ideal recomposition checked for every nibble; retain printed-formula and incomplete-baseline caveats. |
| 025 | LowMC v2, S-box, parameters, MPC/HE results, Appendix G | S-box/count checks pass; made the first used public-generator bit a240 explicit. |
| 026 | Kreyvium state/taps, depth lengths, Tables 1–2, composition | Separated historical proof-gap claim from independent CPA reasoning; state and offset accounting check out. |

## Independent checks and limits

Exhaustive small-domain checks covered the repaired multiplexer (8 inputs), SIMON/Trivium gadget (32 inputs), AND modulo three, base-four AND, byte-shift semantics, field-product identities in $\mathbb F_3,\mathbb F_5,\mathbb F_{17}$, Hippogryph recomposition (16 nibbles), LowMC S-box (8 inputs), SM4 inversion (256 values), and full-adder sum/carry (8 inputs). Additional arithmetic checks covered cipher operation totals, XBOOT's ideal parity reduction, BLEACH expansions, encrypted-state sizes, warm-up offsets, image size and headline amortization.

These checks operate on the notes' ideal mathematical descriptions. They do not simulate homomorphic noise, establish extremely small failure probabilities, or reproduce the published benchmarks. External implementations, current paper revisions and later cryptanalysis were not independently checked in this local-source audit. Source-PDF links, note structure and preservation of existing metadata were also checked.
