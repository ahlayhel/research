# Summary: Random Oracles Are Practical: A Paradigm for Designing Efficient Protocols

**Authors:** Mihir Bellare, Phillip Rogaway  
**Venue:** ACM CCS 1993, pp. 62–73  
**Paper file:** `168588.168596.pdf`

---

## Summary

This paper introduces and advocates the **random oracle model** as a formal framework for designing and proving the security of practical cryptographic protocols. A **random oracle** is an idealized function $R : \{0,1\}^* \to \{0,1\}^\infty$ chosen uniformly at random — every party, including the adversary, can query it, but no one can predict its outputs without querying. The paper's thesis is: design a protocol that is provably secure when $R$ is a true random oracle, then instantiate $R$ with a concrete hash function (e.g., a truncated or restricted variant of MD5 or SHA).

The paper makes three kinds of contributions. First, it provides **new efficient constructions** for public-key encryption (achieving semantic and chosen-ciphertext security), digital signatures, and non-interactive zero-knowledge proofs in the random oracle model — all dramatically more efficient than prior provably-secure schemes in the standard model. Second, it provides **formal justifications for known heuristics**, such as the "hash-then-sign" signature paradigm and the Fiat-Shamir transform that converts interactive proofs into non-interactive ones. Third, it proves **theoretical results** in the random oracle model, including that any NP language has an efficient non-interactive zero-knowledge proof in this model.

---

## Math

**Random oracle.** A random oracle $R$ is a uniformly random map $R : \{0,1\}^* \to \{0,1\}^\infty$, where each bit $R(x)_i$ is chosen independently and uniformly. All parties — honest and adversarial — have oracle access to $R$. In practice, $R$ is instantiated by a concrete hash function.

*Plain language: Think of $R$ as an infinite table filled with random bits: given any input string, you get a deterministic but unpredictable output. It's like a perfect random function that everyone can look up but no one can predict without querying.*

**Trapdoor permutation.** A trapdoor permutation generator $\mathcal{G}$ outputs a triple $(f, f^{-1}, d)$ where $f$ is a permutation on $\{0,1\}^k$, $f^{-1}$ is its inverse, and $d$ samples from the domain. The one-wayness requirement: for all non-uniform PPT adversaries $M$,

$$\varepsilon(k) = \Pr\!\left[(f, f^{-1}, d) \leftarrow \mathcal{G}(1^k);\; y \leftarrow d(1^k) :\; M(f, d, y) = f^{-1}(y)\right]$$

is negligible. Examples: RSA, modular squaring.

**Encryption scheme 1 — polynomial security.** Let $G : \{0,1\}^* \to \{0,1\}^\infty$ be a random generator (derived from $R$). Encrypt message $x$ as:

$$E_G(x) = f(r) \,\|\, G(r) \oplus x$$

where $r$ is random from the domain of $f$ and $\oplus$ denotes XOR of $x$ with the first $|x|$ bits of $G(r)$.

*Plain language: The message is masked by a pseudorandom pad derived from a random seed $r$, and only the encryption of $r$ under the trapdoor permutation is transmitted. Without knowing $f^{-1}$, recovering $r$ (and thus $G(r)$) is as hard as inverting $f$.*

**Encryption scheme 2 — chosen-ciphertext security and non-malleability.** Let $G, H$ be independent random functions. Encrypt as:

$$E_{G,H}(x) = f(r) \,\|\, G(r) \oplus x \,\|\, H(r \,\|\, x)$$

Decryption of $y = a \,\|\, \tilde{x} \,\|\, b$ with $|a| = k$: compute $r = f^{-1}(a)$, $x = G(r) \oplus \tilde{x}$; accept iff $H(r \,\|\, x) = b$.

*Plain language: The extra hash $H(r \,\|\, x)$ acts as a "commitment" that binds the ciphertext together, preventing an adversary from modifying one part without invalidating the others — this achieves non-malleability.*

**Signature scheme — hash-then-invert.** Fix a trapdoor permutation $f$ and random hash $H : \{0,1\}^* \to \{0,1\}^k$. The signature of message $m$ is $\sigma = f^{-1}(H(m))$; verification checks $f(\sigma) = H(m)$.

*Plain language: The signer inverts the trapdoor permutation on the hash of the message. Security relies on the fact that without the trapdoor, computing $f^{-1}$ on a random-looking hash value is infeasible.*

**Security proof sketch (signatures).** Suppose forger $F$ succeeds with non-negligible probability $\lambda(k)$ making $n(k)$ queries to $H$. Construct inverter $M(f, d, y)$: pick a random index $t \in \{1, \ldots, n(k)\}$, answer the $t$-th $H$-query with $y$ (instead of a fresh $f(r)$), and answer all other $H$-queries with $f(r_i)$ for fresh random $r_i$. If $F$'s forgery $(m, \sigma)$ uses $m = m_t$, then $\sigma = f^{-1}(y)$. The success probability of $M$ is at least:

$$\frac{\lambda(k)}{n(k)} - \frac{1}{2^k}$$

which is non-negligible, contradicting the one-wayness of $f$.

**Non-interactive zero-knowledge via Fiat-Shamir.** Given a 3-move interactive ZK proof $(P', V')$ for $L \in \mathrm{NP}$ with challenge $b \in \{0,1\}$, the transformation uses random hash $H : \{0,1\}^* \to \{0,1\}^{2k}$. The prover computes $2k$ commitments $a_1, \ldots, a_{2k}$, derives all challenges as $b_i = i\text{-th bit of } H(a_1 \cdots a_{2k})$, and sends all responses. Soundness error drops from $1/2$ to at most $T(n) \cdot 2^{-k(n)}$ where $T(n)$ is the number of oracle queries a cheating prover can make.

*Plain language: The prover replaces the verifier's random challenge with the hash of the transcript so far — since the hash is unpredictable, the prover cannot "cheat" by choosing their commitment after seeing the challenge. This makes any 3-move ZK proof non-interactive in one shot.*

---

## Experiments

The paper is theoretical; there are no implementation experiments. Analysis consists of:

- Efficiency comparisons of the new encryption schemes against prior provably-secure schemes (Goldwasser-Micali, Blum-Goldwasser, Naor-Yung, Rackoff-Simon), showing dramatic improvements:
  - Ciphertext size reduced to $O(|x| + k)$ vs. $O(k \cdot |x|)$ for semantic security.
  - Encryption cost: one application of $f$ (e.g., one modular squaring) vs. $O(|x|)$ applications.
- Concrete guidance on instantiating random oracles with hash functions, including analysis of why raw MD5 and SHA fail (length-extension attacks, known compression-function collisions), and which constructions are acceptable candidates:
  - Truncated hash output: $h_1(x) = $ first 64 bits of $\mathrm{MD5}(x)$.
  - Input-restricted hash: $h_2(x) = \mathrm{MD5}(x)$ for $|x| < 400$.
  - Non-standard usage: $h_3(x) = \mathrm{MD5}(xx)$.
  - First-block compression: $h_4 : \{0,1\}^{512} \to \{0,1\}^{128}$.

---

## Results

Key findings:

- **Efficient semantically-secure encryption** (Scheme 1): ciphertext size $O(|x| + k)$, one trapdoor evaluation to encrypt and one to decrypt — far more practical than all prior provably-secure schemes.
- **Efficient chosen-ciphertext-secure and non-malleable encryption** (Scheme 2): same efficiency, simultaneously achieving the strongest notions of encryption security (Rackoff-Simon CCA and Dolev-Dwork-Naor non-malleability).
- **Formal justification of hash-then-sign**: the "classical" $\sigma = f^{-1}(H(m))$ signature scheme is proven secure against adaptive chosen-message attack in the random oracle model.
- **Efficient non-interactive ZK**: any language in NP with a 3-move interactive ZK proof can be made non-interactive in the random oracle model with soundness error $2^{-k(n)}$, at only an $O(k(n))$ blowup in communication and computation.
- **Foundational paradigm**: the paper establishes the random oracle methodology as a principled middle ground between ad-hoc design (no security proof) and standard-model provable security (too inefficient). It explicitly warns that standard hash functions (MD5, SHA used naively) are not sound instantiations.
- **Open problem noted**: there is no known complexity-theoretic assumption that fully captures all properties of a public random oracle, leaving open the question of sound instantiation in the standard model — a gap later explored (and shown to be provably unbridgeable in general) by Canetti, Goldreich, and Halevi (1998).
