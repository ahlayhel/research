# Summary: Cryptographic Extraction and Key Derivation: The HKDF Scheme

**Author:** Hugo Krawczyk  
**Venue:** CRYPTO 2010, LNCS 6223, pp. 631–648  
**Paper file:** `978-3-642-14623-7_34.pdf`

---

## Summary

This paper provides the first formal treatment of general-purpose **Key Derivation Functions (KDFs)** and introduces **HKDF**, a practical KDF built on HMAC that was subsequently standardized as IETF RFC 5869. The core problem is that despite being ubiquitous in cryptographic protocols, KDFs lacked rigorous formal definitions and analysis. Existing standardized KDFs combined extract and expand phases in ad-hoc ways, requiring hash functions to behave as perfect random functions even when not necessary.

The paper's central contribution is the **extract-then-expand** paradigm: first use a randomness extractor to distill a pseudorandom key from imperfect source key material (SKM), then use a pseudorandom function (PRF) to expand that key into as many bits as needed. HMAC serves as both extractor and PRF in HKDF, and the paper proves security under a range of assumptions — from combinatorial (almost-universality) to idealized (random oracle) — depending on the application scenario. The paper also introduces the first formal definition of KDF security and the notion of **computational extractors**.

---

## Math

**Statistical extractor (Definition 2).** A function $\mathrm{ext} : \{0,1\}^t \times \{0,1\}^n \to \{0,1\}^{m'}$ is a $\delta$-statistical extractor w.r.t. distribution $X$ if:

$$(r,\, \mathrm{ext}_r(x)) \approx_\delta (r,\, z)$$

where $r \xleftarrow{R} \{0,1\}^t$, $x \leftarrow X$, and $z \xleftarrow{R} \{0,1\}^{m'}$. It is a $(m, \delta)$-statistical extractor if this holds for all $X$ with min-entropy $\geq m$.

*Plain language: A statistical extractor takes a "salt" $r$ and an imperfect random source, and outputs bits that are statistically close to uniform, regardless of the source's distribution — as long as the source has enough entropy.*

**Min-entropy (Definition 1).** A distribution $X$ has min-entropy $\geq m$ if $\Pr[X = a] \leq 2^{-m}$ for all $a$ in the support.

*Plain language: Min-entropy measures the worst-case unpredictability of a distribution. A source with min-entropy $m$ means no single value occurs with probability greater than $2^{-m}$.*

**Computational extractor (Definition 3).** Same as a statistical extractor but the closeness requirement is relaxed to $(t, \varepsilon)$-computational indistinguishability.

*Plain language: For cryptographic purposes, it suffices that no efficient attacker can distinguish the extractor's output from uniform — even if the distributions are not statistically close.*

**KDF security (Definitions 7/9).** A KDF is $(t, q, \varepsilon)$-$m$-entropy secure if no attacker $A$ running in time $t$ and making $q$ queries can win the following game with probability $> \frac{1}{2} + \varepsilon$: given side-information $\alpha$ and salt $r$, adaptively query the KDF on chosen contexts $c_i$, then distinguish $\mathrm{KDF}(\sigma, r, c, \ell)$ from a random string on a fresh context $c$.

**HKDF construction.** Given a Merkle-Damgård hash function underlying HMAC:

$$\mathrm{PRK} = \mathrm{HMAC}(\mathrm{XTS},\, \mathrm{SKM})$$

$$K(1) = \mathrm{HMAC}(\mathrm{PRK},\, \mathrm{CTXinfo} \,\|\, 0)$$

$$K(i+1) = \mathrm{HMAC}(\mathrm{PRK},\, K(i) \,\|\, \mathrm{CTXinfo} \,\|\, i), \quad 1 \leq i < t$$

$$\mathrm{KM} = K(1) \,\|\, K(2) \,\|\, \cdots \,\|\, K(t)$$

where $t = \lceil L/k \rceil$, $k$ is the HMAC output/key length, and the last block is truncated to $L \bmod k$ bits.

*Plain language: HKDF has two steps. First, HMAC "extracts" a fixed-length pseudorandom key PRK from the (possibly non-uniform) source material using a salt. Second, HMAC in feedback mode "expands" PRK into as many key bits as needed, binding each block to the context string CTXinfo.*

**Core security theorem (Theorem 1).** If XTR is a $(t_X, \varepsilon_X)$-computational extractor w.r.t. source $\Sigma$ and $\mathrm{PRF}^*$ is a $(t_P, q_P, \varepsilon_P)$-secure variable-length-output PRF, then the extract-then-expand KDF is $(\min\{t_X, t_P\},\, q_P,\, \varepsilon_X + \varepsilon_P)$-secure w.r.t. $\Sigma$.

*Plain language: The security of HKDF reduces cleanly to the security of its two components — the extractor and the PRF. Their error probabilities add, and the computational bound is the minimum of the two.*

**NMAC extraction lemmas** (adapted from Dodis et al. 2004):

- **Lemma 1** (RO outer, AU inner): If the outer HMAC function is modeled as a random oracle and the inner is $\delta$-almost-universal, then NMAC on an $m$-entropy source produces output $\sqrt{q(2^{-m} + \delta)}$-close to uniform, where $q$ bounds the number of random oracle queries.

- **Lemma 2** (non-idealized): If $h_\kappa$ is an $(m, \delta)$-statistical extractor and $\hat{h}_\kappa$ is a $(t, 1, \varepsilon)$-pseudorandom family, then NMAC is a $(t,\, n\delta + \varepsilon)$-computational extractor for $m$-blockwise sources with $n$ input blocks.

- **Lemma 3** (random compression functions): If $h_\kappa$ is a family of random compression functions with $k$-bit output, then NMAC truncated by $c$ bits is a $\bigl(k,\, \sqrt{(n+2)\,2^{-c}}\bigr)$-statistical extractor.

- **Lemma 4** (almost-universal): If the inner function is $\delta_1$-AU and the outer is $(2^{-k'} + \delta_2)$-AU, then NMAC truncated to $k'$ bits is a $\bigl(m,\, \sqrt{2^{k'}(2^{-m} + \delta_1 + \delta_2)}\bigr)$-statistical extractor.

- **Corollary 1**: If $h_\kappa$ is strongly universal and $H_\kappa$ is generically collision-resistant against linear-size circuits, then NMAC truncated by $c$ bits is a $(k,\, (n+2)\,2^{-c/2})$-statistical extractor on $n$-block inputs.

---

## Experiments

The paper is theoretical; there are no computational experiments. Analysis consists of:

- Formal proofs of HKDF's security under four progressively relaxed assumptions: (i) random oracle outer + almost-universal inner, (ii) blockwise extraction without idealization, (iii) truncated NMAC with random compression functions, (iv) random oracle compression function.
- Worked example: HMAC-SHA256 used to derive 288 bits (one 128-bit AES key + one 160-bit HMAC-SHA1 key), requiring $t = 2$ HMAC invocations with the second block truncated to 32 bits.
- Comparison with existing standardized KDFs (ANSI X9.42, X9.63, TLS PRF) documenting their ad-hoc structure and lack of formal security justification.

---

## Results

Key findings:

- **First formal KDF definition**: The paper introduces the first general, rigorous security definition for multi-purpose KDFs, capturing side-information leakage ($\alpha$), salting, context binding (CTXinfo), and adaptive multi-query security.
- **Extract-then-expand is provably sound** (Theorem 1): Security reduces modularly to the extractor and PRF, with additive error bounds.
- **HMAC is a good extractor** under a range of assumptions, including purely combinatorial ones (strong universality + collision resistance) — no random oracle needed for many practical scenarios such as IKE Diffie-Hellman groups.
- **Truncated NMAC achieves statistical extraction** (Lemmas 3/4, Corollary 1): e.g., NMAC-SHA512 truncated to 256 bits achieves statistical distance $(n+2) \cdot 2^{-128}$ under strong universality, giving a strong unconditional guarantee.
- **HMAC preserves the random oracle property** (Coron et al. 2005): modeling the compression function as a random oracle implies HMAC on variable-length inputs is also a random oracle, justifying the extraction step in the most constrained scenarios.
- **Practical impact**: Standardized as IETF RFC 5869 and widely deployed in TLS, IKE, Signal Protocol, and other cryptographic systems.
