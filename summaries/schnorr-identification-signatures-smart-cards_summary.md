# Summary: Efficient Identification and Signatures for Smart Cards

**Author:** C.P. Schnorr  
**Venue:** CRYPTO '89, LNCS 435, Springer-Verlag 1990  
**Paper file:** `0-387-34805-0_22.pdf`

---

## Summary

**"Efficient Identification and Signatures for Smart Cards"** introduces what is now known as the **Schnorr identification and signature scheme**. The motivation is to design a cryptographic scheme based on the **discrete logarithm problem** that is practical enough for the constrained processors found in smart cards of that era. Compared to prior discrete-log schemes (ElGamal, Beth, Günter, Chaum-Evertse-Graaf) and RSA/Fiat-Shamir, the new scheme dramatically reduces the number of modular multiplications at signing time and cuts signature/communication length roughly in half. The key innovation is a **preprocessing algorithm** that computes the expensive random exponentiation $x := a^r \pmod{p}$ during the smart card's idle time, so that actual signing costs only ~12 modular multiplications independent of the message.

---

## Math

**System parameters.** The Key Authentication Center (KAC) publishes:
- Primes $p$ and $q$ with $q \mid p-1$, $q \geq 2^{140}$, $p \geq 2^{512}$.
- A base $a \in \mathbb{Z}_p^*$ of order $q$, i.e. $a^q \equiv 1 \pmod{p}$, $a \neq 1$.
- A one-way hash function $h : \mathbb{Z}_p \times \mathbb{Z} \to \{0, \ldots, 2^t - 1\}$ with security parameter $t = 72$.

*Plain language: $p$ is a large prime, $q$ is a large prime factor of $p-1$, and $a$ generates a subgroup of order $q$ in $\mathbb{Z}_p^*$. All discrete logarithm computations are done modulo $q$.*

**Key generation.**
- Private key: $s \in \{1, \ldots, q\}$ chosen at random.
- Public key: $v = a^{-s} \pmod{p}$.

*Computing $v$ from $s$ is easy; recovering $s$ from $v$ requires computing $\log_a(v^{-1}) \pmod{p}$, assumed computationally infeasible.*

**Identification protocol (3-move).**
1. Prover $A$ picks random $r \in \{1, \ldots, q-1\}$, computes $x := a^r \pmod{p}$, sends $x$ to verifier $B$.
2. $B$ sends random challenge $e \in \{0, \ldots, 2^t - 1\}$.
3. $A$ sends $y := r + se \pmod{q}$.
4. $B$ accepts iff $x \equiv a^y v^e \pmod{p}$.

*Correctness: $a^y v^e = a^{r+se} \cdot a^{-se} = a^r = x \pmod{p}$.*

**Signature scheme.**
- Sign message $m$: pick random $r$, compute $x := a^r \pmod{p}$, set $e := h(x, m)$, output $(e,\, y)$ where $y := r + se \pmod{q}$.
- Verify $(e, y)$: compute $x' := a^y v^e \pmod{p}$, check $e = h(x', m)$.

*The signature is only 212 bits ($t + 140$ bits for $e$ and $y$), less than half the length of RSA or Fiat-Shamir signatures.*

**Security (Proposition 2.1).** If a probabilistic algorithm $\mathcal{AL}$ with time bound $|\mathcal{AL}|$ passes the identification test with probability $\varepsilon > 2^{-t+1}$ on straight exams, then the discrete logarithm of $v$ can be computed in time $\mathcal{O}(|\mathcal{AL}|/\varepsilon)$ with constant positive probability.

*Plain language: any successful forger can be turned into a discrete-log solver via a "rewinding" argument — run the prover twice on the same commitment $x$ with two different challenges $e', e''$, then extract $s = (y' - y'')/(e'' - e') \pmod{q}$. So breaking the scheme is as hard as discrete log.*

**Preprocessing algorithm.** The smart card stores $k$ random pairs $(r_i, x_i)$ where $x_i = a^{r_i} \pmod{p}$. Each round $v$ selects indices $a(0),\ldots,a(d-3)$ randomly from $\{1,\ldots,k\}$, with $a(d-2) := a(d) := v-1 \pmod{k}$ and $a(d-1) := v$, then updates:

$$r_v := \sum_{i=0}^{d} r_{a(i)} \cdot 2^i \pmod{q}, \qquad x_v := \prod_{i=0}^{d} x_{a(i)} \pmod{p}$$

The pair used for the next signature is:

$$r := r_v^{\text{old}} \cdot 2 + r_{v-1} \pmod{q}, \qquad x := x_v^{\text{old}\,2} \cdot x_{v-1} \pmod{p}$$

Step 2 costs only $2d$ multiplications modulo $p$, $d$ additions modulo $q$, and $d$ shifts — yielding ~12 multiplications per signature for $k=8$, $d=6$.

**Security of preprocessing (Theorem 4.2).** If the initial vector $(r_1,\ldots,r_k)$ is uniformly distributed over $\{1,\ldots,q\}^k$, then for all $j \geq 0$ the vector $(r_1^{k+j}, \ldots, r_k^{k+j})$ used for signatures is uniformly distributed over $\{1,\ldots,q\}^k$ for sufficiently large $q$.

*Plain language: any $k$ consecutive preprocessing outputs are jointly uniform, so observing signatures reveals nothing about the stored $r_i$ values beyond what is implied by the discrete logarithm assumption.*

**Supporting lemmas.**
- **Lemma 4.1**: The transformation matrix $T_v$ is invertible modulo $q$ (since $\det T_v$ is a nonzero integer less than $2^d < q$), so preprocessing preserves the uniform distribution on $(r_1,\ldots,r_k)$.
- **Lemma 4.3**: Pairwise distinct choices of indices $a(0),\ldots,a(d-3)$ over $v$ rounds generate, for sufficiently large $q$, pairwise distinct linear functions $r_v^* = r_v(r_1,\ldots,r_k)$ over $\mathbb{Z}_q$.
- **Theorem 4.4**: Pairwise distinct index vectors over the first $k$ rounds generate pairwise distinct linear functions for $r_{k+1}^*$, ensuring internal randomization is not repeated.

**Security parameter for $q$.** Baby-step giant-step finds discrete logs in $\mathcal{O}(\sqrt{q})$ steps, so $q \geq 2^{140}$ provides $2^{70}$ security. Combined with $t = 72$, the overall security level is $2^{72}$ operations.

---

## Experiments

The paper does not present computational or simulation experiments in the modern sense. Instead, it provides a **performance analysis and comparison** grounded in exact operation counts:

- All multiplication counts are computed analytically from the algorithms.
- The new scheme is compared against Fiat-Shamir ($k=8$, $t=9$), RSA, and the GQ scheme (Guillou-Quisquater).
- Memory requirements are computed for a 512-bit modulus $p$ and storage on smart card EEPROM/RAM/ROM.
- An optimized variant with $k=6$ pairs (including the key pair $(-s, v)$ as the 7th element) is analyzed for an average of 12.76 modular multiplications per preprocessing round.

---

## Results

| Scheme | Sig. generation (multiplications) | Preprocessing | Sig. verification | Sig. length |
|---|---|---|---|---|
| **Schnorr (new)** | **0** (with preprocessing) | **12\*** | 228\* | **212 bits** |
| Fiat-Shamir ($k=8$, $t=9$) | 451 | 0 | 45\* | ~450 bits |
| RSA | 750 | 0 | 12 | ~512 bits |
| GQ | 216\* | 0 | 108\* | ~512 bits |

*(\*) can be reduced by further optimization.*

Key findings:
- **Signature generation** with preprocessing costs only ~12 modular multiplications, done during idle time — the online signing cost is a single cheap multiplication $se \pmod{q}$.
- **Signature length** is 212 bits, less than half that of RSA or Fiat-Shamir.
- **Communication bits** for identification ($2t + 140$ bits) are less than half those of competing schemes.
- Total smart card storage: ~800 bytes EEPROM, ~192 bytes RAM, <500 bytes ROM for the full scheme.
- The optimized variant ($k=6$, using the key pair as the 7th element) averages **12.76 multiplications per preprocessing round** and requires only 26 bytes for system parameters.
- The security of the preprocessing is established by an information-theoretic argument (Theorem 4.2, Lemma 4.3, Theorem 4.4): an attacker observing more than $k$ consecutive signatures must exhaustively search over $(d-2)^k$ cases, which for $k=8$, $d=6$ is $2^{96} \gg 2^{72}$.
