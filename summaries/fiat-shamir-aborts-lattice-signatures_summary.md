# Summary: Fiat-Shamir with Aborts: Applications to Lattice and Factoring-Based Signatures

**Author:** Vadim Lyubashevsky  
**Venue:** ASIACRYPT 2009, LNCS 5912, pp. 598–616  
**Paper file:** `978-3-642-10366-7_35.pdf`

---

## Summary

This paper introduces **"Fiat-Shamir with Aborts"**, a technique that transfers the algebraically-rich framework behind efficient number-theoretic identification (ID) and signature schemes (e.g., Schnorr, GQ, Okamoto) into the setting of **ideal lattices**. Prior lattice-based ID and signature schemes treated each challenge bit independently, resulting in signatures and communication of millions of bits. By exploiting the ring structure of **$(x^n + 1)$-cyclic lattices** (ideals in $\mathbb{Z}[x]/\langle x^n+1\rangle$), the paper constructs schemes where the entire challenge is treated as a single polynomial, dramatically reducing communication to ~65,000 bits for identification and ~50,000 bits for signatures.

The central technical contribution is the **aborting technique**: to protect the secret key during repeated use, the prover aborts and re-runs if the response would leak information, accepting only responses that fall into a carefully chosen "safe" range. This preserves perfect **witness-indistinguishability** while enabling efficient instantiation. The technique is also applied to **Girault's factoring-based signature scheme**, reducing its signature length from 488 to 425 bits. Security is based on the worst-case hardness of the approximate Shortest Vector Problem ($\mathrm{SVP}_\gamma$) in ideal lattices, with $\gamma = \tilde{O}(n^2)$.

---

## Math

**Ring and lattice setup.** Let $R = \mathbb{Z}_p[x]/\langle x^n + 1\rangle$ where $n$ is a power of 2. A lattice $\Lambda$ is an $(x^n+1)$-cyclic lattice (ideal in $\mathbb{Z}[x]/\langle x^n+1\rangle$) if $(v_0, \ldots, v_{n-1}) \in \Lambda \Rightarrow (-v_{n-1}, v_0, \ldots, v_{n-2}) \in \Lambda$.

*Plain language: Elements of this ring are polynomials of degree $\leq n-1$ with coefficients in $\mathbb{Z}_p$. Multiplying by $x$ corresponds to a cyclic shift with a sign flip. This algebraic structure enables $\tilde{O}(n)$-time polynomial multiplication via FFT.*

**Norm bound.** For $v, w \in R$:

$$\|vw\|_\infty \leq \|v\|_\infty \|w\|_1 \leq n \|v\|_\infty \|w\|_\infty$$

**Hash function family (Definition 1).** For $\hat{a} \in R^m$ and $\hat{z} \in D^m$ where $D \subseteq R$:

$$h_{\hat{a}}(\hat{z}) = a_1 z_1 + a_2 z_2 + \cdots + a_m z_m \in R$$

These functions are **homomorphic**:

$$h(\hat{y} + \hat{z}) = h(\hat{y}) + h(\hat{z}), \qquad h(\hat{y}c) = h(\hat{y})c$$

*Plain language: The hash is an inner product in the ring, with $\hat{a}$ as the key. Its homomorphic property is what makes the identification protocol work: the verifier can check $h(\hat{z}) = Sc + Y$ without knowing $\hat{y}$.*

**Worst-case to average-case reduction (Theorem 1).** Let $D = \{y \in R : \|y\|_\infty \leq d\}$, $m > \log p / \log 2d$, and $p \geq 4dmn^{1.5}\log n$. If there is a poly-time algorithm solving $\mathrm{Col}(h, D)$ for random $h \in H(R, D, m)$ with non-negligible probability, then there is a poly-time algorithm solving $\mathrm{SVP}_\gamma(\Lambda)$ for every $(x^n+1)$-cyclic lattice $\Lambda$, where $\gamma = 16dmn\log^2 n$.

*Plain language: Breaking the hash function (finding collisions) is as hard as finding short vectors in all ideal lattices. This is the security foundation of the scheme.*

**Key generation.** Secret key $\hat{s} \xleftarrow{R} D_s^m$ (polynomials with small $\ell_\infty$ norm $\leq \sigma$). Public key: random $h \in H(R, D, m)$ and $S = h(\hat{s})$.

**Lattice ID scheme (Figure 3).**
1. Prover picks $\hat{y} \xleftarrow{R} D_y^m$, sends $Y = h(\hat{y})$ to verifier.
2. Verifier sends challenge $c \xleftarrow{R} D_c$ where $D_c = \{g \in R : \|g\|_1 \leq \kappa\}$.
3. Prover computes $\hat{z} = \hat{s}c + \hat{y}$. If $\hat{z} \notin G^m$, abort ($\hat{z} \leftarrow \perp$).
4. Verifier accepts iff $\hat{z} \in G^m$ and $h(\hat{z}) = Sc + Y$.

where $G = \{g \in R : \|g\|_\infty \leq mn\sigma\kappa - \sigma\kappa\}$.

*Plain language: The prover masks $\hat{s}c$ by adding a random $\hat{y}$. The abort step ensures the response $\hat{z}$ doesn't fall in a region that could reveal which of multiple valid secret keys the prover is using — this is what makes the protocol witness-indistinguishable.*

**Why aborting enables witness-indistinguishability.** The range $G$ is chosen so that for any $\hat{s}, \hat{s}' \in D_s^m$ with $h(\hat{s}) = h(\hat{s}') = S$, and any $\hat{z} \in G^m$, the value $\hat{z} - \hat{s}c \in D_y$. Therefore, given the transcript $(Y, c, \hat{z})$, the view is identical whether the prover used $\hat{s}$ or $\hat{s}'$, since $h(\hat{y}) = h(\hat{y}') = Y$ and $\hat{y}' = \hat{z} - \hat{s}'c$.

**Completeness.** An honest prover is accepted with probability $1/e$ per attempt (aborting probability is $1 - 1/e$). Running 30 parallel instances gives acceptance probability $\approx 1 - 2^{-20}$.

**Security theorem for ID scheme (Theorem 2).** If the ID scheme is insecure against active attacks, there is a poly-time algorithm solving $\mathrm{SVP}_\gamma(\Lambda)$ for $\gamma = \tilde{O}(n^2)$ for every ideal lattice $\Lambda$ in $\mathbb{Z}[x]/\langle x^n+1\rangle$.

**Lattice signature scheme (Figure 4).** Uses the Fiat-Shamir transform with random oracle $H : \{0,1\}^* \to D_c$:

- **Sign($\mu$)**: pick $\hat{y} \xleftarrow{R} D_y^m$, set $e = H(h(\hat{y}), \mu)$, compute $\hat{z} = \hat{s}e + \hat{y}$. If $\hat{z} \notin G^m$, restart. Output $(\hat{z}, e)$.
- **Verify($\mu, \hat{z}, e$)**: accept iff $\hat{z} \in G^m$ and $e = H(h(\hat{z}) - Se, \mu)$.

**Security theorem for signature scheme (Theorem 3).** If the signature scheme is not strongly unforgeable, there is a poly-time algorithm solving $\mathrm{SVP}_\gamma(\Lambda)$ for $\gamma = \tilde{O}(n^2)$ for every ideal lattice $\Lambda$ in $\mathbb{Z}[x]/\langle x^n+1\rangle$.

**Factoring-based application.** For Girault/Pointcheval's scheme with $N = pq$, the response is $z = se + y$ over the integers. By picking $y \xleftarrow{R} \{0, \ldots, 2^{k+1}\sigma\}$ instead of $\{0, \ldots, 2^{k+k'}\sigma\}$ and aborting when $z \notin G = \{2^k\sigma, \ldots, 2^{k+1}\sigma\}$, the signature $(z, e)$ shrinks from 488 bits to 425 bits.

**Security of the aborting factoring scheme (Theorem 4).** An adversary who breaks the aborting signature scheme in $T$ steps can be used to factor $N$ in $\mathrm{poly}(T)$ steps.

---

## Experiments

The paper provides concrete parameter instantiations (Figure 2) guided by the lattice reduction analysis of Gama and Nguyen (2008), which estimated that current algorithms find vectors no shorter than $1.01^n$ times the shortest vector:

| $n$ | $m$ | $\sigma$ | $\kappa$ | Sig. size (bits) | Vector needed to break | Shortest vector findable |
|---|---|---|---|---|---|---|
| 512 | 4 | 127 | 24 | ~49,000 | $2^{23.5}$ | $2^{25.5}$ |
| 512 | 5 | 2047 | 24 | ~72,000 | $2^{27.9}$ | $2^{36.7}$ |
| 512 | 8 | 2047 | 24 | ~119,000 | $2^{28.6}$ | $2^{47.6}$ |
| 1024 | 8 | 2047 | 21 | ~246,000 | $2^{29.4}$ | $2^{69.4}$ |

Security is analyzed via the lattice $\Lambda^\perp_p(A) = \{u \in \mathbb{Z}^{mn} : Au = 0 \pmod{p}\}$ where $A = [\mathrm{Rot}(a_1) \,\|\, \cdots \,\|\, \mathrm{Rot}(a_m)]$. Breaking the scheme requires finding $u \in \Lambda^\perp_p(A)$ with $\ell_\infty$ norm $\leq 2mn\sigma\kappa$. The optimal attack uses only $\sqrt{n \log p / \log 1.01}$ dimensions, yielding a shortest vector of $\ell_\infty$ length approximately:

$$\min\!\left\{p,\; 2^{2\sqrt{n \log p \cdot \log 1.01}} \cdot \left(\frac{n \log p}{\log 1.01}\right)^{-1/4}\right\}$$

---

## Results

Key findings:

- **Communication complexity**: ~65,000 bits for identification and ~50,000 bits for signatures — orders of magnitude shorter than all prior lattice-based schemes (which required millions of bits), because the challenge is treated as a single polynomial rather than independent bits.
- **Signing time**: $\tilde{O}(n)$, compared to $\tilde{O}(n^4)$ for Gentry-Peikert-Vaikuntanathan hash-and-sign schemes.
- **Aborting overhead**: Expected 3 signing attempts before success ($\Pr[\text{success per try}] = 1/e$); aborted attempts are invisible in the final signature output.
- **Strong unforgeability**: A forger cannot produce a new valid signature even for a previously signed message.
- **Factoring-based improvement**: Signature length reduced from 488 to 425 bits in the Pointcheval/Girault scheme.
- **Limitation**: Hardness assumption ($\tilde{O}(n^2)$ approximation) is weaker than some competing schemes ($\tilde{O}(n^{1.5})$ or $\tilde{O}(n)$), though practical parameters remain sound given current lattice cryptanalysis.
- **Legacy**: This work directly inspired **CRYSTALS-Dilithium**, the NIST post-quantum signature standard, which extends these ideas with Gaussian rejection sampling.
