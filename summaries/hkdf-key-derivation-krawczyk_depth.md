# Depth Background — *Cryptographic Extraction and Key Derivation: The HKDF Scheme*

**Author:** Hugo Krawczyk (IBM T.J. Watson Research Center)
**Venue:** CRYPTO 2010, LNCS 6223, pp. 631–648
**Standardized as:** IETF RFC 5869 (HKDF)

---

## The Main Concept

The paper is about **key derivation functions (KDFs)**: the cryptographic plumbing
that takes a *messy, imperfect source of secret randomness* and turns it into one or
more *clean, full-strength cryptographic keys*.

The central idea is the **extract-then-expand** paradigm:

1. **Extract** — Distill the unpredictability ("entropy") that is scattered, biased,
   or non-uniformly spread across the input source material into a single, short,
   uniformly-random-looking master key (called the PRK, the *pseudorandom key*).
2. **Expand** — Stretch that one good master key into as many output keys of whatever
   length the application needs, using a pseudorandom function.

Krawczyk's contributions are four-fold:

- A **rationale** for why a KDF should be split into two distinct modules.
- The **first rigorous, general definition** of a multi-purpose KDF and what it means
  for one to be secure — built on the new notion of a **computational extractor**.
- A **concrete, practical scheme (HKDF)** that implements both modules using only
  **HMAC**.
- A **security analysis** showing HKDF works across a spectrum of trust assumptions —
  from mild combinatorial properties (universal hashing) up to the idealized random
  oracle model — while *minimizing* the assumptions needed for each use case.

The key engineering virtue: HKDF uses the hash function **prudently**. Most prior
standardized KDFs blur extract and expand together and implicitly demand that the
hash behave like a *perfectly random function* — even when that strong assumption is
unnecessary. HKDF asks for only as much "magic" from the hash as each scenario truly
requires.

---

## Trace — Back to the Fundamental Axiom

Peeling the idea back, layer by layer, to its bedrock:

1. **HKDF (the artifact).** A specific construction: `PRK = HMAC(salt, SourceKeyMaterial)`,
   then iterated `HMAC(PRK, ...)` calls to produce output key bits.

2. **Extract-then-expand (the design pattern).** HKDF is one instance of separating
   "make the input uniform" from "stretch the uniform key."

3. **Extraction = randomness extractors (the abstraction).** The "extract" step is the
   cryptographic embodiment of a **randomness extractor** — a function that maps a
   high-entropy-but-non-uniform distribution to a near-uniform output. Krawczyk relaxes
   this to a **computational extractor**: the output need only be *computationally
   indistinguishable* from uniform (pseudorandom), not *statistically* close. This
   relaxation is what makes practical, hash-based extraction possible.

4. **Pseudorandomness (the deeper abstraction).** "Computationally indistinguishable
   from random" is the notion of **pseudorandomness**: a bounded adversary cannot tell
   the output apart from true coin flips. Both the expand step (a PRF) and the relaxed
   extract step rest on this.

5. **Entropy / min-entropy (the measurement).** Everything depends on the input having
   enough unpredictability. The right yardstick here is **min-entropy** — a worst-case
   measure (the probability of the *most likely* value), which is stricter and more
   appropriate than Shannon (average-case) entropy for security.

6. **The bedrock axiom — you cannot create randomness from nothing.** At the very
   bottom sits an information-theoretic truth: *a deterministic function cannot
   manufacture unpredictability that was not already present in its input.* For **any**
   single fixed function there exists some high-entropy input on which its output is
   badly non-uniform. This is *why* a generic extractor must be **keyed/randomized**
   (with a public, non-secret "salt"). The salt is the escape hatch that lets one fixed
   algorithm work across *all* sufficiently-random sources.

> **Fundamental axiom:** Randomness can be *concentrated and purified*, but never
> *conjured*. A KDF is an entropy-refinery, not an entropy-factory — and to be
> universal, the refinery needs a public random "catalyst" (the salt).

---

## Dumb Down — Plain English with Everyday Analogies

### What problem is being solved?

Imagine you've collected a secret that is *full of unpredictability* but *ugly and
lumpy*. For example:

- The static you record from a cheap microphone (lots of noise, but biased and patchy).
- The exact timing of your keystrokes over a minute.
- The shared secret two people compute in a Diffie–Hellman key exchange.

You can't use this raw, lumpy secret directly as an encryption key. Cryptographic
algorithms (AES, HMAC) demand keys that look like *perfectly fair coin flips* — every
bit unbiased and independent. Feeding them a lumpy secret is like trying to run a
precision engine on dirty fuel.

A **KDF is the fuel refinery.**

### The two stages, by analogy

**Extract = panning for gold.**
A prospector scoops up a pan of river silt. The silt contains real gold (the entropy),
but it's mixed with sand, gravel, and water (the bias and structure). By swirling the
pan, the worthless material washes away and a small amount of pure gold collects at the
bottom. The **extract** step swirls your lumpy secret and concentrates its scattered
randomness into one small nugget of pure, uniform-looking key (the PRK). You end up
with *less* material, but it's *pure*.

**The salt = the standardized pan.**
Why does the prospector need a (publicly known) standard pan? Because a single rigid
sieve might happen to be perfectly shaped to *let some particular river's gold fall
through*. By choosing the pan's shape randomly (and announcing it openly — the salt is
not secret), no adversary can pre-arrange a river that defeats it. One randomized tool
now works for *every* river. The salt doesn't need to be secret, only random — like the
serial number stamped on the pan, visible to everyone but unpredictable in advance.

**Expand = minting coins from a gold bar.**
Once you have your one pure gold nugget, you melt it and stamp out as many identical-
quality coins (output keys) as you need: one for encryption, one for authentication,
one for the next session, and so on. The **expand** step uses the master key to mint as
much key material as the application wants — and crucially, knowing one coin tells you
nothing about the others.

**Context info = stamping each coin with its purpose.**
Each minted coin gets engraved ("this key is for Alice↔Bob, session #42, for AES"). So
even if the *same* gold nugget is reused, the coins for different purposes come out
provably independent. Mix up the labels and you'd never confuse an encryption key with
an authentication key.

### Why not skip the refinery?

Older KDF standards basically threw the dirty fuel straight into the engine and *prayed*
the engine was magical enough to cope (i.e., they assumed the hash function was a
"perfectly random oracle"). HKDF says: refine first, demand magic only when you truly
must. That's the difference between a *hopeful hack* and an *engineered, analyzable
design*.

---

## Chronology — How the Idea Reached This Paper

A timeline of the concepts that converge in HKDF:

- **1948 — Shannon, information theory.** Formalizes *entropy* as a measure of
  uncertainty. Establishes the conceptual basis that randomness is a quantifiable,
  conserved resource — the foundation everything else stands on.

- **1979 — Carter & Wegman, universal hashing.** Introduce *universal hash function
  families*. These become the original workhorse for *statistical* randomness
  extraction and supply the "almost-universal (AU)" property HKDF's analysis later
  leans on.

- **1980s — Hard-core bits (Blum–Micali; Yao; Goldreich–Levin; Alexi–Chor–Goldreich–
  Schnorr).** Show that even a single bit of a one-way function can be made
  *pseudorandom* — the first "source-specific extractors." Establishes the notion of
  extracting *computational* randomness from cryptographic hardness, foreshadowing
  computational extractors.

- **1990s — The extractor abstraction matures.** Nisan–Zuckerman and others formalize
  *randomness extractors* and *min-entropy* in complexity theory, making precise the
  idea of mapping high-min-entropy sources to near-uniform outputs (and proving generic
  extractors must be randomized/keyed).

- **1996 — Bellare, Canetti & Krawczyk, HMAC.** Introduce **HMAC/NMAC** as a way to key
  hash functions for message authentication, with proofs that it is a secure PRF based
  on the compression function. This gives Krawczyk the versatile, already-deployed
  building block that HKDF will reuse for *both* extract and expand.

- **Late 1990s–2000s — IKE / IPsec KDF.** Krawczyk designs the KDF for the Internet Key
  Exchange protocols, where he first deploys the **extract-then-expand** paradigm in a
  practical setting (using nonces as salt and DH values as the source).

- **2003–2005 — Extraction meets practical RNGs.** Barak–Shaltiel–Tromer (true RNGs in
  a changing environment) and Barak–Halevi (a model/architecture for `/dev/random`)
  independently push extract-then-expand into system random-number generation,
  validating the paradigm's practicality.

- **2004–2005 — Hash functions under fire.** Wang et al.'s collision attacks on MD5 and
  SHA-1 spark "healthy skepticism" about treating hashes as ideal random functions.
  This directly motivates HKDF's guiding principle: *use the hash as conservatively as
  possible.*

- **2005 — Coron, Dodis, Malinaud & Puniya — "Merkle–Damgård Revisited."** Using the
  *indifferentiability* framework, they show HMAC's structure *preserves* random-oracle
  behavior (unlike plain Merkle–Damgård, which suffers length-extension). This is the
  result that lets HKDF safely model HMAC as a random oracle when a scenario demands it.

- **~2008 — Dodis, Gennaro, Håstad, Krawczyk, Rabin (referenced as [14]).** Lay the
  formal foundations analyzing HMAC/NMAC as a *randomness extractor*, including the
  blockwise-source and truncation results. This is the immediate theoretical scaffolding
  for HKDF's security proofs.

- **2010 — Krawczyk, this paper (CRYPTO 2010).** Synthesizes all of the above:
  introduces the **computational extractor** notion, gives the **first general formal
  definition** of a multi-purpose KDF and its security (including adversary side-
  information α and salt), specifies the concrete **HKDF** scheme over HMAC, and proves
  security across the full assumption spectrum (AU → blockwise → random oracle).

- **2010 — RFC 5869.** HKDF is published as an IETF standard, and goes on to become the
  default KDF in TLS 1.3, the Signal protocol, the Noise framework, and countless other
  systems — the practical payoff of the theory developed here.

---

## One-line Takeaway

HKDF is the disciplined refinery that turns lumpy secret randomness into pristine keys
by first **extracting** purity (with a public salt as catalyst) and then **expanding**
it into labeled output keys — all built from HMAC, and demanding only as much idealized
trust in the hash as each scenario strictly requires.
