<div align="center">

<img src="docs/ui/aegis-logo.svg" width="112" alt="AEGIS logo" />

# AEGIS Messenger

### The post‑quantum, end‑to‑end encrypted messenger — engineered so that *only you and your contact* can ever read a message.

**Ultimate client policy (Ω1–Ω4) · Phases 0–5 shipped · audit freeze `audit-freeze-2026-08-07` · August 2026**

<p>
<img src="https://img.shields.io/badge/status-working%20prototype-orange" alt="status" />
<img src="https://img.shields.io/badge/Ultimate-Ω1–Ω4%20client-0EA5E9" alt="ultimate" />
<img src="https://img.shields.io/badge/encryption-end--to--end-4F46E5" alt="e2ee" />
<img src="https://img.shields.io/badge/post--quantum-both%20axes-6D5DF6" alt="post-quantum" />
<img src="https://img.shields.io/badge/FIPS-203%20%C2%B7%20204%20%C2%B7%20205-2BB6A3" alt="fips" />
<img src="https://img.shields.io/badge/formal-ProVerif%20%C2%B7%20Tamarin%20%C2%B7%20CryptoVerif-8B5CF6" alt="formal" />
<img src="https://img.shields.io/badge/audit-freeze%202026--08--07-yellow" alt="audit-freeze" />
<img src="https://img.shields.io/badge/tests-Rust%20%2B%20Flutter%20%2B%20Maestro-brightgreen" alt="tests" />
</p>
<p>
<img src="https://img.shields.io/badge/Rust-000000?logo=rust&logoColor=white" alt="rust" />
<img src="https://img.shields.io/badge/Flutter-02569B?logo=flutter&logoColor=white" alt="flutter" />
<img src="https://img.shields.io/badge/platform-Android-3DDC84?logo=android&logoColor=white" alt="android" />
<img src="https://img.shields.io/badge/unsafe-forbidden-success" alt="forbid-unsafe" />
<img src="https://img.shields.io/badge/license-proprietary-E5483D" alt="license" />
</p>

</div>

> [!WARNING]
> **AEGIS is a working prototype that has *not yet* had an independent crypto audit.** Everything below is real, tested, and — where it matters — *machine‑checked*. The limits are stated honestly. **Do not rely on it for genuine high‑risk communication** until an external audit is complete.

> [!NOTE]
> **This public repository is the showcase surface** (README, license, UI assets). Full engineering source lives in the private development tree. Evaluation & commercial licensing: **ozanks20@gmail.com**.

---

## ✨ Why AEGIS exists

Most messengers protect *today's* messages against *today's* computers. The day a large quantum computer arrives, ciphertext harvested years ago can be decrypted retroactively — the **"harvest‑now, decrypt‑later"** attack.

**AEGIS is built for that day.** It is post‑quantum on **both** axes — *confidentiality* **and** *authenticity* — and engineered so that **no server, no operator, and no man‑in‑the‑middle can ever read your chats.** Even on a seized phone, **Ultimate** is designed so the adversary gets **neither** the full account **nor** a cryptographic proof that no further world exists — within the residual ends we name honestly (R‑A6, R‑A5∞, R‑A10, R‑GLOBAL).

What makes it different from "yet another secure messenger":

- 🔭 **Post‑quantum on both axes** — most products (incl. today's Signal) are PQ for *confidentiality* only. AEGIS signs identities with **three** disjoint signature schemes too. *(Honest caveat: the signature axis is the least quantum‑urgent one — see [Where AEGIS honestly stands](#-where-aegis-honestly-stands-august-2026).)*
- 🧮 **Math‑first, machine‑checked** — the handshake's secrecy, authentication and no‑replay properties are **formally verified in ProVerif**; the ratchet's forward secrecy in **Tamarin**; and the post‑quantum **KEM‑combiner** (the "two disjoint KEMs" guarantee) both symbolically *and* **computationally in CryptoVerif** — with anti‑vacuity controls that *must* fail.
- 🧬 **A second, non‑lattice confidentiality axis (HQC‑256).** Beyond ML‑KEM, AEGIS wires a **code‑based** KEM (HQC‑256, NIST's 2025 backup pick) as a *disjoint* second post‑quantum leg, combined so confidentiality survives **even a total break of the entire lattice family**. It is validated bit‑exact against the **official NIST/PQClean KAT** and is opt‑in behind a feature flag (default‑off to keep the reproducible build pure‑Rust).
- 🌳 **Key Transparency + client self‑audit** — every key the relay serves is committed into a **signed, append‑only Merkle directory** (CONIKS/CT model). A swapped key fails the proof; the directory key is **pinned persistently**; and the client **monitors its own view over time** — a relay that forks, rolls back, or silently re‑keys its directory between observations is caught, across restarts.
- 🧅 **Tor, mixnet & cover traffic (opt‑in)** — route every relay connection through Tor — an external **Orbot** *or* an in‑process **embedded Tor** that needs no Orbot — so the relay never sees your IP (`.onion` supported); optionally tunnel through the **Nym mixnet** (Sphinx packets reordered by independent mix nodes — the only one of these that resists a *global* observer); hide *when* you type behind an always‑on Poisson stream of decoys (Loopix); and **rotate mailbox queues** so the relay can't follow one address across a session.
- 🏗️ **Reproducible builds + binary transparency** — the relay rebuilds **bit‑identical** in a digest‑pinned, self‑verifying container; release hashes go into a signed append‑only log. A backdoored binary is either absent from the log or public evidence.
- 🙈 **Amnesic & anti‑forensic** — RAM‑only chats by default, duress PIN (and under Ultimate a **duress *world***), panic‑burn, gaze‑lock, screenshot block, hardware‑backed keys.
- 🛡️ **AEGIS Ultimate (Ω1–Ω4)** — Split Identity · Multi‑World Memory · Blind Pipe · Continuous Proof. **Phases 0–5 client policy shipped** on the development mainline; see the full section below.
- 🚫 **No invented crypto** — audited classical primitives (RustCrypto / dalek) plus a **formally‑verified ML‑KEM** (`libcrux`, the implementation family Signal ships) and the vetted **PQClean** HQC reference, all validated against official FIPS/RFC/ACVP/NIST‑KAT vectors + Google Wycheproof + 250k+ fuzz.


---

## 🛡️ AEGIS Ultimate — four Ω that are not optional

Most “secure messengers” stop at content encryption. **Ultimate** is a harder product bar: the system is designed so that a realistic adversary — the relay, the network, a quantum harvest of ciphertext, a supply‑chain backdoor, **theft of one device**, or **coercion with one shown world** — gets **neither** the full account **nor** a proof that no further world exists. And every claimed success is either **publicly refutable** or ends **fail‑closed**.

That is not marketing language for “military grade.” It is a single target architecture with four mandatory user promises. If any Ω is missing, the product is a prior grade (ASP / Hardened) — **not** Ultimate. Full living Ultimate specification and audit-freeze package ship with the engineering tree under evaluation license.

| Ω | User promise (plain language) | Engineering name | Client status |
|---|---|---|:---:|
| **Ω1** | *Even with my phone in the enemy’s hand, the account is not fully dead and not fully readable.* | **Split Identity** | ✅ **shipped** |
| **Ω2** | *Under coercion I can show a world, and nobody can prove it is the only one.* | **Multi‑World Memory** | ✅ **shipped** |
| **Ω3** | *The state / relay should not even see that I talk or with whom.* | **Blind Infrastructure** | ✅ **shipped** |
| **Ω4** | *The world can prove it — not take my word.* | **Continuous Proof** | ✅ **shipped** |

### How Ultimate was built (Phases 0 → 5)

Ultimate was not a single PR. It was shipped as a **documented phase ladder** on `main`, each phase with an exit gate:

| Phase | Focus | Primary Ω | What actually landed |
|:---:|---|:---:|---|
| **0** | Living specification | all | Ultimate definition, adversary×promise matrix, residual ends R‑A6 / R‑A5∞ / R‑A10 / R‑GLOBAL |
| **1** | Proof spine + profile skeleton | **Ω4** | Ultimate Settings flag, continuous‑proof gate skeleton, release‑log hooks |
| **2** | **Blind Pipe** | **Ω3** | Ultimate **refuses clear TCP** to the relay. Dial is fail‑closed unless proxy (Tor/Nym‑class) + cover policy + pin requirements are met. Ephemeral receive mailboxes rotate every **5** real messages under Ultimate (far tighter than the default cadence). |
| **3** | **Split Identity** | **Ω1** | Long‑term provision secret **I** sealed under wrap key **W**. **W** is Shamir **2‑of‑3** (phone · token/SE · offline share). High‑value send **and** decrypt/poll require the split factors when enrolled. Discovery at rest no longer trusts a monodevice identity AEAD alone — it uses an **AETH** marker. The discovery mailbox seed is sealed under **W** as **AEMS**. When hardware is present, the token share is **SE‑wrapped**; revoke shreds that wrap for the enrollment. |
| **4** | **Multi‑World Memory** | **Ω2** | ≥ 2 sealed worlds at rest; only one unlocked per session. Duress PIN selects a **non‑primary** world when multi‑world is enrolled (else the legacy wipe path). Distinct per‑world AEAD keys + sealed primary/decoy snapshots. World keys SE‑wrapped when hardware allows. Enabling Ultimate **auto‑enrolls** multi‑world so the decoy path is not an expert‑only checkbox. |
| **5** | **Continuous Proof** + audit freeze | **Ω4** | Every release artifact can enter a signed append‑only **ReleaseLog**. A CLI **monitor** emits **ForkProof** on fork/rollback/equivocation. The client runs a startup / Ultimate proof gate (layout + release membership / `kt_verify_release`). Operator scripts publish and validate CDN layouts. Annotated tag **`audit-freeze-2026-08-07`** freezes a reviewable surface for an external firm — **not** a completed audit. |

```mermaid
flowchart TB
    subgraph U["AEGIS Ultimate profile — one Settings posture"]
      O1["Ω1 Split Identity<br/>phone · token/SE · offline"]
      O2["Ω2 Multi-World<br/>primary + decoy AEAD"]
      O3["Ω3 Blind Pipe<br/>proxy · pin · cover"]
      O4["Ω4 Continuous Proof<br/>release-log · monitor"]
    end
    O1 --> HV["High-value traffic gated"]
    O3 --> DIAL["Dial fail-closed if pipe incomplete"]
    O4 --> PROOF["Startup / release verify"]
    O2 --> DURESS["Duress → non-primary world"]
    HV --> CORE["Rust core · sealed blobs only"]
    DIAL --> RELAY["Opaque store-and-forward"]
    CORE --> RELAY
    PROOF --> LOG["Signed ReleaseLog + ForkProof"]
```

### Ω1 — Split Identity (phone alone is incomplete)

Under Ultimate, **the phone is not the whole account**. The long‑term identity material **I** is sealed under a wrap key **W**. **W** itself is never stored as a single blob on one factor: it is split with **Shamir secret sharing (2‑of‑3)** across phone · hardware token / Secure Element · offline share. Sending and decrypting high‑value traffic while enrolled requires the live factors — a seized locked phone that only holds one share does not hand the adversary a complete monodevice identity.

Discovery was hardened the same way: at rest the client uses threshold‑aware markers (**AETH** for identity, **AEMS** for the mailbox seed under **W**) so “wipe the phone disk and read the discovery identity” is no longer a one‑device complete story. Token revoke is real for this enrollment (the SE wrap is shredded); a full network‑wide protocol‑epoch fan‑out revoke remains an honest residual.

### Ω2 — Multi‑World Memory (coercion meets a shown world)

Coercion is not solved by crypto — **total** coercion of a human with every factor and every world is residual **R‑A5∞**. What Ultimate *does* ship is the next best engineering bar: **at least two cryptographically sealed worlds**, only one unlocked per session, with **distinct AEAD keys** and sealed primary/decoy snapshots. Entering the duress PIN does **not** necessarily wipe everything into an empty decoy account — when multi‑world is enrolled it unlocks a **non‑primary world** that can look like a real (but limited) life. From disk alone, an examiner should not be able to *prove* that no other world exists.

Ultimate auto‑enrolls multi‑world when the profile is turned on, so the feature is not buried behind expert toggles. Cover/traffic policy stays independent of which world is active.

### Ω3 — Blind Pipe (no “just TCP to the relay”)

Ultimate is not “Tor recommended.” Ultimate is **fail‑closed**:

- no direct clear TCP to the relay;
- Tor and/or a Nym‑class path required;
- cover traffic policy on;
- TLS certificate pinning where configured;
- mailbox rotation on a **tight** cadence (every **5** real messages);
- still **no delivery receipts** on the relay path (the NDSS sealed‑sender traffic‑analysis amplifier).

If the pipe is incomplete, the dial **does not silently fall back to plain** — it refuses. That is the difference between a privacy checklist and a posture.

### Ω4 — Continuous Proof (don’t take our word for the binary)

Supply‑chain claims without public evidence are theater. Ultimate’s proof spine:

- publish each release artifact into a signed, append‑only **ReleaseLog**;
- client gate checks layout / membership (and FRB `kt_verify_release` where codegen is live);
- operator **monitor** path emits **ForkProof** if the log forks or rolls back;
- CDN publish + layout validate scripts for operators who host the public log;
- **`audit-freeze-2026-08-07`** packages the reviewable client surface for an external engagement.

What is still residual (and stated as such): a **publicly hosted** log URL you can point strangers at, a **signed external audit report**, and a funded bug bounty. The machinery is in tree; the social / operational half is not magic.

### Hard residual ends (the only accepted “losses”)

Ultimate **accepts** exactly these residual ends — marketing must never claim them closed:

| ID | Residual | Why software alone cannot “solve” it |
|----|----------|--------------------------------------|
| **R‑A6** | Full malware / rooted OS reading app RAM while the user is unlocked | Cleartext must exist somewhere to be *read* |
| **R‑A5∞** | Total coercion of the human + all factors + all worlds + all shares | Human, not crypto |
| **R‑A10** | Flash / FTL forensics after crypto‑erase | Crypto‑erase ≠ physical media wipe |
| **R‑GLOBAL** | Global passive observer **without** mix‑class transport | Timing and volume correlation |

---

## 📱 See it

| Chat | Settings |
|:---:|:---:|
| <img src="docs/ui/chat.png" width="250" alt="AEGIS chat" /> | <img src="docs/ui/settings.png" width="250" alt="AEGIS settings" /> |

The UI is a **Flutter** Android client (Linux desktop path for development). Emulator smoke and the **Maestro** full UI suite (launch · settings · gaze · advanced TLS · save · new‑chat sheet · devices) last ran **green** on an `aegis_api34` AVD — crypto and threshold logic stay on unit tests, not on UI automation.

## 🧩 How it fits together

```mermaid
flowchart LR
    You["📱 You<br/>(Flutter UI)"] -->|FFI| Core1["🦀 aegis-core<br/>(Rust crypto)"]
    Core1 -->|"sealed blob"| Relay[("☁️ aegis-relay")]
    Relay -->|"sealed blob"| Core2["🦀 aegis-core<br/>(Rust crypto)"]
    Core2 -->|FFI| Peer["📱 Contact<br/>(Flutter UI)"]
    Relay -.->|"sees only ciphertext"| Note["🚫 no plaintext<br/>🚫 no sender id<br/>routed by mailbox key"]
```

All cryptography lives in a **Rust core** (`aegis-core`, `#![forbid(unsafe_code)]`); the Flutter app calls it over FFI. The **relay** only moves opaque, encrypted blobs — it never sees plaintext, and it routes by an opaque recipient **mailbox key** (no sender field on the wire). Payload‑level **Sealed Sender** is wired into the app send path: the initial handshake *and* every ratchet message are wrapped in a sealed‑sender envelope, so the relay sees **neither** the sender identity **nor** the cleartext ratchet header. The sealed‑sender / metadata layer is itself **PQ‑hybrid** (X25519 ∥ ML‑KEM‑1024) with per‑message forward secrecy — so even metadata resists harvest‑now‑decrypt‑later.

> **Honest limit:** sealed‑sender‑*class* designs are academically known to be statistically deanonymizable by a malicious server over a conversation (Martiny, Kaptchuk, Aviv, Roche & Wustrow, NDSS 2021). AEGIS narrows that attack surface — there are **no delivery receipts on the relay path** (the NDSS attack's main amplifier), and opt‑in **Tor (external Orbot or embedded)**, an opt‑in **Nym‑mixnet transport**, **always‑on cover traffic** and **mailbox‑queue rotation** raise the bar further. Full unlinkability against a determined relay or a *global* observer still depends on those opt‑in transports being enabled end‑to‑end — the mixnet in particular needs a running Nym client and both endpoints over it — so AEGIS treats metadata privacy as **defense‑in‑depth, not a guarantee**.

---

## 🔐 Cryptography

AEGIS does **not** invent primitives. It composes audited building blocks (RustCrypto / dalek) plus a **formally‑verified ML‑KEM** (`libcrux`) and the vetted **PQClean HQC** reference into the Signal‑style protocol, and validates the composition against official test vectors.

| Purpose | Primitive | Standard |
|---|---|---|
| Wire AEAD | **ChaCha20‑Poly1305** | RFC 8439 |
| At‑rest AEAD (key‑committing) | **XChaCha20‑Poly1305** (192‑bit nonce) | RFC 8439 / draft‑irtf |
| Post‑quantum KEM (lattice) | **ML‑KEM‑1024** *(libcrux — formally verified, hax/F\*)* | FIPS 203 |
| Post‑quantum KEM (code‑based, disjoint) | **HQC‑256** *(PQClean reference, NIST L5; opt‑in `hqc` feature)* | NIST PQC round 4 / 2025 selection |
| Post‑quantum signature (lattice) | **ML‑DSA‑87** *(NIST L5 / CNSA 2.0)* | FIPS 204 |
| Post‑quantum signature (hash‑based) | **SLH‑DSA‑SHAKE‑128f** | FIPS 205 |
| Classical key agreement | **X25519** | RFC 7748 |
| Classical signature | **Ed25519** | RFC 8032 |
| At‑rest PIN KDF | **Argon2id** (m=64 MiB, t=3, p=4) | RFC 9106 |
| Hashing | **SHA‑256/512, SHA‑3 / SHAKE** | FIPS 180‑4 / 202 |
| Key derivation | **HKDF** | RFC 5869 |
| Threshold wrap (Ultimate Ω1) | **Shamir secret sharing** 2‑of‑3 over wrap key **W** | Classical SSS + AEAD seal of **I** |
| World stores (Ultimate Ω2) | Per‑world **XChaCha20‑Poly1305** keys (SE‑wrapped when HW available) | At‑rest multi‑world |
| Key hygiene | **zeroize‑on‑drop** + best‑effort `mlock` | — |

**🤝 Handshake — PQXDH:** a Signal X3DH secret **‖** an ML‑KEM‑1024 shared secret, merged via HKDF. The session root stays secret as long as *either* the classical *or* the post‑quantum assumption holds.

**🔄 Ratchet — continuous post‑quantum:** every Double‑Ratchet DH step mixes in a **fresh** ML‑KEM‑1024 secret → forward secrecy *and* post‑compromise (self‑healing) security, even against a quantum adversary. This is the same *continuous*‑PQ cadence Signal shipped in its 2025 Triple Ratchet and Apple in PQ3 — quantum resistance for the whole conversation, not just the handshake.

**🪪 Identity — triple‑hybrid signatures:** identity & pre‑keys are signed with **Ed25519 ‖ ML‑DSA‑87 ‖ SLH‑DSA** at once. Verification requires **all three** — forgery needs breaking classical *and* lattice *and* hash‑based crypto simultaneously.

**🧬 Crypto‑agility + a non‑lattice second axis (`SUITE_V3`):** the root derivation and the bundle/initial wire are version‑negotiated through a **suite registry**. `SUITE_V2` (X25519 ∥ ML‑KEM‑1024) is the shipped default; **`SUITE_V3` = X25519 ∥ ML‑KEM‑1024 ∥ HQC‑256** is wired end‑to‑end (real encap/decap in the handshake) behind the opt‑in `hqc` feature. The two post‑quantum legs rest on **disjoint hard problems** (module‑lattice vs. syndrome decoding of quasi‑cyclic codes), composed as a **robust KEM‑combiner** (Giacon–Heuer–Poettering, PKC 2018): the root stays pseudorandom while **either** ML‑KEM **or** HQC holds — so confidentiality survives even a *total* break of the lattice family. The HQC leg is validated bit‑exact against the **official NIST/PQClean round‑4 KAT** (its `.rsp` sha256 reproduced equal to PQClean's pinned `META.yml` value) and its decapsulation is `catch_unwind`‑guarded against the HQC decode‑failure path. *Default build stays `[SUITE_V2]`, pure‑Rust and byte‑identical — HQC is not in the shipped APK unless the feature is enabled.*

**🛡️ Downgrade resistance:** the negotiated suite id and both advertised capability lists are **bound into the transcript / root** for the v3 path; a message‑level downgrade is rejected fail‑fast. The remaining bundle‑strip vector (a relay stripping the responder's unsigned advertised list) is closed by an **EDHOC SUITES_I "always‑send‑I"** mechanism — the initiator always carries its real capability list and it is **bound into the root**, so a strip *or* a truncation of that list makes the two sides derive divergent roots (fail‑closed). This is built and proven (see below) behind an opt‑in `strict_downgrade` feature; enabling it by default trades v2 wire byte‑compatibility (a deliberate owner decision). Grounded in RFC 8446 §4.1.3, RFC 9528 (EDHOC) and Bhargavan et al. 2016.

**🔐 Ultimate at rest (Ω1 + Ω2):** content crypto is not enough if a single device disk holds a complete account. Under Ultimate the long‑term provision secret is sealed under wrap key **W**, and **W** is split 2‑of‑3. Discovery identity and mailbox seed follow the same threshold story (**AETH** / **AEMS**). Separately, multi‑world memory keeps **primary** and **decoy** sealed under distinct AEAD keys — duress unlocks a non‑primary world when enrolled. This is complementary to PQXDH/ratchet: those protect the *wire*; these protect *possession* and *coercion with one PIN*.

---

## 🧮 Formally verified (machine‑checked)

The headline differentiator. AEGIS's protocol composition is modelled in **ProVerif 2.05** (symbolic), **Tamarin 1.12** (ratchet), and **CryptoVerif 2.12** (computational), and the security goals are *proven*, not just asserted. Each model ships with its committed result in the engineering tree, and the symbolic models run as a **reproducible regression gate** (proof runner diffs every query outcome against a committed golden).

| Model | Tool | Property | Result |
|---|---|---|:---:|
| `pqxdh-m1.pv` | ProVerif | **Root‑key secrecy (G1)** + **hybrid (G6)**: root stays secret if X25519 **or** ML‑KEM holds | ✅ **TRUE** (3 worlds) |
| `pqxdh-m1b.pv` | ProVerif | **Handshake authentication (G2/G5)** — responder‑auth via signed prekey; implicit initiator‑auth | ✅ **TRUE** |
| `pqxdh-m1c.pv` | ProVerif | **One‑time‑prekey single‑use ⇒ injective (no replay)** + falsification control | ✅ **TRUE** (control ❌) |
| `pqxdh-m1d.pv` | ProVerif | **Signed one‑time‑prekey dispensing ⇒ agreement (relay/MITM cannot substitute an OPK)** + control | ✅ **TRUE** (control ❌) |
| `pqxdh-m2-hqc.pv` | ProVerif | **ML‑KEM ∥ HQC combiner secrecy** — root secret even if the **whole lattice family falls**, while the disjoint HQC leg holds + control | ✅ **TRUE** (control ❌) |
| `cv-pqxdh-m2-combiner.cv` | **CryptoVerif** | **Combiner secrecy, computational** — `root` indistinguishable from random up to the *single honest HQC leg's* IND‑CCA2 advantage, with the ML‑KEM secret key *handed to the adversary* + non‑vacuity control | ✅ **PROVED** (control unprovable) |
| `dr-m2-fs.spthy` | Tamarin | **Double‑Ratchet forward secrecy** (one‑way chain, bounded) | ✅ **VERIFIED** |

Modelled exactly as the academic state of the art (Bhargavan–Jacomme–Kiefer–Schmidt, *PQXDH*, USENIX Security 2024; Cohn‑Gordon et al., *Signal*, EuroS&P 2017; Giacon–Heuer–Poettering, *KEM Combiners*, PKC 2018). The CryptoVerif combiner bound depends **only** on the HQC leg — proving the robust‑combiner theorem in the computational model; its **non‑vacuity control** (both KEM secrets leaked → root *not* provable) is wired into the gate and *must* flip, so the proof cannot be vacuous. Symbolic proofs hold under idealized primitives; the residual (full computational handshake, unbounded PCS) is stated honestly in the formal-verification plan (engineering tree).

---

## 🧨 We attacked it ourselves — hard

Authorized adversarial testing against our own code and device:

**Protocol attacks — all rejected:** identity‑swap, ciphertext tamper, header tamper, replay, reorder‑then‑replay, forged signed‑prekey, forged PQ‑prekey, cross‑session, malformed‑wire, crafted‑injection desync, suite‑downgrade (message‑rewrite *and* bundle‑strip). *(Protocol attacks + Google Wycheproof adversarial vectors.)*

**Fuzzing & DoS:** **250,000+** hostile/garbage inputs to the wire / envelope / relay parsers → **zero panics, zero overflows** (length‑bounded, cap‑before‑allocation, memory‑safe Rust). The HQC encapsulation path was separately hammered with **200,078** adversarial public keys → **0 panics** (the unsigned, relay‑served HQC prekey cannot crash the initiator).

**Live relay pentest (against the running server):**

| Attack | Result |
|---|---|
| Malformed opcode | connection closed, relay alive |
| 4 GiB length‑prefix (OOM attempt) | rejected **before** allocation |
| Drain someone else's mailbox with a forged signature (BOLA) | **0 bytes delivered** |
| 1024‑message flood per mailbox | bounded (excess dropped) |
| 1100 concurrent connections | relay survived (cap 1024 + idle timeout) |

**Device pentest:** non‑debuggable release · no exported components · malicious‑intent fuzz → no crash · 2000 monkey events → no ANR · no plaintext in logcat · **no hardcoded secrets** in the native library.

**Standards audit:** mapped to **OWASP MASVS**; **MobSF** static scan (0 trackers, 0 secrets); **StrongBox** hardware key‑backing confirmed on a real device.

**Recursive self‑audit loop (4 cycles):** an adversarial *read → audit → fix → re‑audit* loop run against our own code found and fixed **28 issues** (1 P0 · 2 P1 · 9 P2 · 16 P3) — and the re‑audit even **caught 2 regressions in its own earlier fixes** — before converging clean. The two most serious were both in the anti‑forensic machinery: a **P0** — the duress/lockout wipe was a **no‑op while the app was locked** — and a **P1** — the duress wipe could **flush one buffered plaintext frame** before aborting. Both fixed and regression‑tested. A later honesty‑audit pass corrected a *documentation* over‑statement (a v3‑live claim that overstated downgrade coverage) — and that residual is now the closed‑behind‑a‑flag `strict_downgrade` mechanism above.

**`@username` discovery hardening:** an adversarial pass on the new contact‑by‑name path closed **5 findings** + added **11 tests** (unbounded inbound growth → bounded buffer + FIFO eviction · reorder strand · mailbox‑rotate abuse · dual‑poller race · wipe TOCTOU).

> **Result:** **no cryptographic weakness found** — no protocol break and no broken primitive. Every finding above was an implementation bug in the surrounding machinery — **all fixed and regression‑tested.**

---

## 👻 Device & "amnesic" security

- 🔒 **End‑to‑end only** — keys never leave the two phones; the relay carries opaque blobs.
- 🧠 **Amnesic by design** — chats live in **RAM only** (no chat database on disk); media bytes stay in memory and temp files are securely shredded; keys are zeroized on drop; optional **wipe‑everything‑on‑leave**. (Encrypted persistence is strictly **opt‑in**.)
- 🆘 **Duress PIN** — classic path: covert wipe + empty decoy account. **Ultimate multi‑world path:** when multi‑world is enrolled, duress unlocks a **non‑primary sealed world** instead of (or in addition to) a blank slate — so coercion can be answered with a *shown* life, not only with destruction. Total coercion of every factor and every world remains residual **R‑A5∞**.
- 🧩 **Ultimate profile** — a single Settings posture that hard‑enables Blind Pipe (fail‑closed dial), multi‑world auto‑enroll, high‑value traffic gates, and continuous‑proof checks together. It is not a pile of unrelated expert toggles.
- 👁️ **Gaze‑lock** — the app locks the instant the front camera stops seeing your face. Frames are processed **on‑device** and never stored or sent. *(Convenience feature — there is no published security research validating gaze‑based locking; do not treat it as a proven control.)*
- 📵 **FLAG_SECURE** — screenshots, screen‑recording, and the app‑switcher preview are blocked.
- 🔑 **Hardware‑backed keys** — Android **StrongBox** (→ TEE fallback), biometric/PIN app‑lock + auto‑lock, no cloud backup. Under Ultimate, **token shares and world keys are SE‑wrapped when hardware is available**; on desktop without SE the wrap is soft (honest residual).

## 🪪 Find a contact by name — without a server address book

Pasting a 44‑character key is the surest way to lose a non‑technical user. AEGIS lets you add a contact by typing **`@username`** or tapping a **share‑link** — **no phone number, no email, no contact upload** — without giving up the security model:

- 🌳 **KT‑verified resolution.** A username resolves through the **same pinned Key‑Transparency directory** as every other key. The claim is **Ed25519‑signed under the mailbox** and **first‑claim‑wins**, so the relay cannot forge or repoint a name; the client checks signed **inclusion *and* absence** proofs. (Wire: relay opcodes `0x08` claim / `0x09` resolve; KT tags `0x22/0x23/0x24`.)
- 🕸️ **The relay learns the minimum.** With an opt‑in **discovery mailbox**, the relay sees only that *a name exists and is being polled* — **not the conversation graph**. Discovery is opt‑in; the default stays amnesic.
- 🔀 **Then it disappears.** After the first handshake both sides **migrate off the shared discovery mailbox onto ephemeral per‑contact mailboxes**, and **sealed sender** hides who initiated — so the durable handle is used once, not for every message.
- 🔐 **Threshold‑aware at rest (Ultimate Ω1).** When split identity is enrolled, discovery no longer rests on a monodevice AEAD of the full identity: the client stores an **AETH** marker, and the discovery mailbox seed is sealed under wrap key **W** as **AEMS**. A disk image of the phone alone is an incomplete discovery identity.
- ✋ **First contact is authenticated** by a **safety‑number dialog**.

> **Honest limit:** a username is a **discovery** handle, *not* an identity proof — the **safety number is what authenticates**. Whoever looks a name up *first* trusts‑on‑first‑use; a first‑lookup squat is an unresolvable residual, the same TOFU caveat every such system has. The whole flow is **proven live across two real phones** (see *Verified, honestly* below).

---

## 💬 Features

**Messaging.** Multiple chats & profiles · **photos & voice messages** (each AEAD‑sealed, **chunked over the relay** so large media works) · emoji reactions · replies/quotes · in‑chat search · disappearing messages · smooth message‑entrance animations.

**Identity & devices.** Safety‑number verification · account re‑key · paper / Shamir backup · multi‑device link path (Sesame‑class provisioning in the app) · **Ultimate Split Identity** (2‑of‑3 wrap, SE token when HW) · **Ultimate Multi‑World** (primary + decoy sealed stores, duress world select).

**Discovery.** **Add a contact by `@username` or a share‑link** — no 44‑char code to copy/paste, no phone number. Opt‑in discovery mailbox: the relay learns only that a name exists & is polled, *not* who talks to whom; after the first handshake both sides migrate to ephemeral per‑contact mailboxes; a safety‑number dialog authenticates first contact. Threshold markers **AETH/AEMS** when Ω1 is enrolled.

**Transparency & supply chain.** **Key‑transparency check on contact add** (warning dialog on a swapped key) · client self‑audit for fork/rollback/re‑key · KT gossip entry points · **Continuous Proof** release‑log publish / verify / monitor with **ForkProof** · audit freeze tag for external engagement.

**Transport & metadata.** **Tor routing (Orbot *or* embedded)** · **Nym‑mixnet transport** · always‑on cover traffic · mailbox‑queue rotation · dual‑pin TLS (advanced) · **Ultimate Blind Pipe** (fail‑closed: no clear TCP, proxy+pin+cover required, rotate every 5 messages).

**Local & UX.** Opt‑in encrypted persistence (default: **amnesic**) · gaze‑lock · `FLAG_SECURE` · **DE / EN / TR**.

---

## ✅ Verified, honestly

All Rust numbers below were **freshly re‑measured** (WSL toolchain, `cargo 1.95`, `--no-fail-fast`); every crate is green with 0 failures.

- **🦀 Rust — 243 workspace tests green** across four crates:
  - `aegis-core` **172** — RFC/FIPS KATs (incl. a cross‑impl libcrux≡RustCrypto≡NIST ML‑KEM KAT), Wycheproof vectors, adversarial unit tests, the protocol‑v2 hardening (key‑committing AEAD, domain‑separated combiner, signed‑Merkle OPK pools), the encrypted‑backup envelope, the **crypto‑agility suite registry**, and 250k+ fuzz.
  - `aegis-relay` **40** (DoS / BOLA / flood / KT‑restart) · `aegis-kt` **28** (Merkle proofs, golden root, equivocation, rollback/fork monitor, tamper rejection) · `aegis-memlock` **3**.
  - **Opt‑in feature builds gate their own code so it cannot bit‑rot:** `aegis-core --features hqc` → **186** (**+14**: real HQC‑256 encap/decap, the ML‑KEM∥HQC combiner, the **external NIST/PQClean HQC KAT**, and the full v3 handshake over the wire) · `aegis-relay --features tls` → **42** (**+2**: rustls + cert‑pin transport) · `aegis-core --features hqc,strict_downgrade` → **186** (the EDHOC‑SUITES_I downgrade‑closure tests). All default‑OFF to keep the reproducible build pure‑Rust.
  - **Non‑vacuity is proven, not assumed:** the two downgrade‑closure tests were verified by **mutation** — disabling the always‑send‑I wire emit turns the bundle‑strip test **RED**; disabling the v2‑root binding turns the truncation test **RED**; restoring makes both **GREEN** again.
- **📱 Flutter** — large widget/unit suite (historically **164+** and still growing with Ultimate): `@username` discovery‑identity store + inbound‑mailbox hardening (bounded growth/FIFO eviction, all‑zero‑rotate rejection, wipe‑generation guard), contact‑link / `aegis://` parser, embedded‑Tor `effectiveProxy` precedence + fail‑closed invariant, encrypted‑backup round‑trip, **plus Ultimate modules** — ultimate profile dial gate, threshold / split identity, multi‑world + world store AEAD, continuous‑proof gate. Built & run on emulator and real devices.
- **🎬 Maestro UI suite** — declarative flows for smoke launch, settings, gaze, advanced TLS, settings‑save, home new‑chat sheet, devices, and a full orchestrator flow. Last full run on emulator **`aegis_api34`**: **green** (crypto stays on unit tests; UI proves the screens still open and save).
- **📡 Live 2‑device `@name` chat — proven end‑to‑end on real hardware** (Xiaomi `@ozantest1` ↔ Samsung): typing a username resolves it under the pinned KT directory, handshakes, and migrates to ephemeral mailboxes. Confidentiality was checked on the wire — a known plaintext was sent + received while a `tcpdump` capture of the relay port recorded **1 418 packets / ~24 KB relayed with 0 plaintext matches** (exact / case‑insensitive / token / strings) — the relay carries ciphertext only.
- **🧮 Formal — 7 machine‑checked models** (5 ProVerif + 1 CryptoVerif + 1 Tamarin) **+ 4 falsification/anti‑vacuity controls** (3 ProVerif + 1 CryptoVerif) — see table above. The 8 ProVerif models + the Tamarin model run reproducibly as a regression gate (`PROOFS OK (8 ProVerif + 1 Tamarin)`); the CryptoVerif combiner proof + its non‑vacuity control are gated alongside (`CRYPTOVERIF OK`).
- **🏷️ Audit freeze** — annotated tag **`audit-freeze-2026-08-07`** packages the Ultimate client surface (Phases 0–5 + residuals) for external engagement. **This is a freeze for auditors, not a signed audit report.**
- **🔒 Guaranteed (design intent — self‑reported until external audit):** E2E confidentiality + integrity · forward secrecy · post‑compromise security · replay/MITM protection · quantum resistance on both axes (with a *disjoint* non‑lattice confidentiality backup once `hqc` is enabled) · relay accountability on served keys (KT + client self‑audit for fork/rollback/re‑key) · bit‑identical rebuilds of the relay artifact · Ultimate fail‑closed dial and high‑value traffic gates when the profile is on · multi‑world sealed stores when enrolled · release‑log verify/monitor path for binaries that claim to be AEGIS.
- **⚠️ Best‑effort / out of scope / residual ends:** rooted/forensically‑attacked devices (**R‑A6**) · total coercion of human+all factors+all worlds (**R‑A5∞**) · flash forensics after crypto‑erase (**R‑A10**) · a *global* network observer without mix‑class transport (**R‑GLOBAL** — Nym path is runtime‑gated and needs a running Nym client + both endpoints over it) · public third‑party KT monitors not yet universally hosted · **independent crypto audit + bug bounty (planned, not yet done)** · groups productization (foundational `aegis-groups` crate exists) · iOS polish · a production multi‑region hosted relay as default SaaS · publicly hosted release‑log CDN URL (scripts in tree; operator residual).

## 🧭 Where AEGIS honestly stands (August 2026)

Per‑axis position against the shipping field, cross‑checked **August 2026** against the product surface and against the public state of the art — **Signal's Sparse PQ Ratchet (SPQR / "Triple Ratchet")** and **Apple iMessage PQ3**. The verdicts below are deliberately conservative: where a shipping competitor matches or beats us, we say so.

| Axis | Position |
|---|---|
| PQ **identity** authentication (triple‑hybrid incl. hash‑based SLH‑DSA hedge) | **Ahead — genuinely unique.** No production messenger ships PQ signatures on identity today. Honest caveat: this is the *least* quantum‑urgent axis (forgery needs a quantum computer *at signing time*; harvest‑now‑decrypt‑later does not apply to signatures). |
| PQ **confidentiality** + continuous PQ ratchet | **At parity, not ahead** for the ML‑KEM ratchet (Signal SPQR/Triple Ratchet, Apple PQ3 ship the same class — with computational proofs and better ratchet bandwidth). **A disjoint second axis (code‑based HQC‑256) is wired** behind a flag — that *specific* "survive a total lattice break" hedge is unusual in shipping messengers. |
| **Crypto‑agility, downgrade resistance & non‑lattice hedge** | **Unusual.** A versioned suite registry negotiates ML‑KEM vs. ML‑KEM ∥ HQC‑256 with the suite ids + both advertised capability lists **bound into the transcript**; an opt‑in EDHOC‑SUITES_I "always‑send‑I" path closes the bundle‑strip downgrade with a root binding that *also* defeats list‑truncation (proven by mutation). Few shipping messengers expose a *disjoint code‑based* PQ fallback **or** a binding‑based downgrade defence at this granularity. Honest caveat: both are opt‑in (default‑off) until the v2‑deprecation decision, so the shipped build is ML‑KEM‑only on the confidentiality axis. |
| Key Transparency with client self‑audit | **Ahead of most** (WhatsApp‑class; SimpleX/Briar don't have it). Gossip/third‑party monitors still missing. |
| Formal proof maturity | **Mixed‑strong.** Symbolic (ProVerif/Tamarin) + a **computational CryptoVerif combiner proof**; forward secrecy still bounded. PQ3 has Tamarin over unbounded loops *plus* a full computational handshake proof. Residual: unbounded PCS + full CryptoVerif handshake. |
| **Ultimate client posture (Ω1–Ω4)** | **Shipped as client policy (Phases 0–5).** Split identity, multi‑world sealed stores, blind‑pipe fail‑closed dial, continuous‑proof gate + audit freeze. No major consumer messenger ships this *combined* possession/coercion/pipe/proof stack. Honest caveat: residuals R‑A6 / R‑A5∞ / R‑A10 / R‑GLOBAL remain **explicit**; public CDN host + external firm sign‑off are still open. |
| Independent audit | **Behind every named competitor** — they are all independently audited; AEGIS is not yet. The freeze tag packages the surface; it does **not** replace a firm report. Until then, treat all claims as vendor self‑reports. |
| Metadata in production | **Behind SimpleX/Briar** (no‑identifier design / Tor‑P2P by construction) *unless* Ultimate + mix‑class path is on end‑to‑end. AEGIS: sealed‑sender class (known weak per NDSS 2021) + Tor (external/embedded) + Nym + cover + mailbox rotation + Ultimate **hard‑require** of the blind pipe — still defense‑in‑depth, not by‑construction anonymity. |
| Product breadth (groups, multi‑device, iOS, backup, SaaS relay) | **Still closing.** Android prototype + Linux desktop path; multi‑device link path and groups crate exist; **iOS product** and **global hosted relay defaults** remain open. |

Gap‑closing continues on the formal, product, and operational tracks in parallel with Ultimate hardening.


## 🛠️ Tech stack

**Rust** (crypto core + relay, `forbid(unsafe_code)`) · **Flutter** + `flutter_rust_bridge` (Android) · **ML Kit** on‑device face detection (gaze‑lock) · RustCrypto / dalek primitives · `libcrux` ML‑KEM · PQClean HQC · FIPS 203/204/205 PQC. Builds run under **WSL2**.

## 📈 Status & roadmap

**Working prototype (Android + Linux desktop)** shipping today:

- handshake & continuous PQ ratchet **formally modelled**
- media‑over‑relay (chunked AEAD)
- **contact‑by‑`@username` discovery** (opt‑in mailbox, live‑proven across two real phones)
- **key transparency + client self‑audit**
- **Tor (Orbot or embedded) + Nym‑mixnet transport + always‑on cover + mailbox‑queue rotation**
- **reproducible builds + binary transparency** (release log + monitor / ForkProof)
- **offline delivery** (background mailbox drain + unread badges)
- ML‑KEM on a **formally‑verified** implementation family
- **AEGIS Ultimate Phases 0–5 client policy** (Ω1–Ω4) on the development mainline
- Maestro full UI suite green on emulator

**Protocol v2 hardening complete:** key‑committing transport AEAD · domain‑separated hybrid‑signature combiner · signed PQ one‑time prekeys (machine‑checked in ProVerif).

**Crypto‑agility + a second confidentiality axis — wired (opt‑in):** a versioned **suite registry** with **`SUITE_V3` = X25519 ∥ ML‑KEM‑1024 ∥ HQC‑256`** is live behind the default‑OFF `hqc` feature — real HQC‑256 encap/decap in the handshake, the ML‑KEM∥HQC combiner, **external NIST/PQClean KAT conformance**, and the full v3 handshake e2e over the wire, with the combiner secrecy proven symbolically (ProVerif) *and* computationally (CryptoVerif). The bundle‑strip downgrade residual is closed by the **EDHOC‑SUITES_I `strict_downgrade`** mechanism (built + mutation‑proven; default‑off because turning it on deprecates the v2 wire — an owner decision). The default shipped build stays `[SUITE_V2]`, pure‑Rust and byte‑identical.

**Ultimate closeout residuals (honest):** public release‑log CDN host · external firm contract & signed report · SE soft on desktop · protocol‑epoch fan‑out revoke · declare residual ends in UI everywhere · ASP path‑length upgrades.

**Next (product + formal + ops):** release keystore + SLSA/sigstore provenance · **independent crypto audit + bug bounty** · metadata/anonymity to full strength (mixnet path over *both* endpoints · on‑device runtime of the embedded‑Tor build · background cover while suspended) · ProVerif model of the downgrade closure · unbounded‑PCS proof (Tamarin) + full CryptoVerif handshake · groups productization (MLS/TreeKEM) · multi‑device polish · **iOS** · a production‑hosted relay + public KT monitors.

## 📜 License & commercial use

AEGIS is **proprietary software — © 2026 Ozan Küsmez. All rights reserved.**

| You may (free) | You may **not** without a paid commercial license |
|---|---|
| View this showcase repository | Run or deploy in production or for any commercial purpose |
| Evaluate for personal, non‑commercial study | Provide AEGIS as a product or service to others |
| Contact the owner about licensing | Copy, redistribute, sublicense, or sell the Software or derivatives |

**In short:** it belongs to the Owner. You may look. Anyone who wants to **run** or **use** it beyond private evaluation needs a **paid license**.

**To license, deploy, or build on AEGIS → [ozanks20@gmail.com](mailto:ozanks20@gmail.com)** · full terms in [`LICENSE`](LICENSE).

**NO WARRANTY.** The Software is provided “AS IS”. It is an **UNAUDITED** prototype and must not be relied upon for real high‑risk communication until an independent security audit has been completed.

**Security honesty:** no independent external crypto audit yet — treat claims as vendor self‑reports. Metadata privacy is defense‑in‑depth, not a global‑observer guarantee. Residual ends R‑A6 / R‑A5∞ / R‑A10 / R‑GLOBAL remain explicit under Ultimate.

---

<div align="center">

**AEGIS** — post‑quantum · end‑to‑end · Ultimate Ω1–Ω4

Built by <b>Ozan Küsmez</b> · [ozanks20@gmail.com](mailto:ozanks20@gmail.com)

<sub><code>#cryptography #post-quantum #e2ee #secure-messaging #rust #flutter #key-transparency #formal-methods</code></sub>

</div>
