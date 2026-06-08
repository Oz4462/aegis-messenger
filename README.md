<div align="center">

<img src="docs/ui/aegis-logo.svg" width="120" alt="AEGIS logo" />

# AEGIS Messenger

### The post-quantum, end-to-end encrypted messenger — built so that *only you and your contact* can ever read a message.

<p>
<img src="https://img.shields.io/badge/status-working%20prototype-orange" alt="status" />
<img src="https://img.shields.io/badge/2--device%20E2E-proven%20live-22C55E" alt="proven live" />
<img src="https://img.shields.io/badge/encryption-end--to--end-4F46E5" alt="e2ee" />
<img src="https://img.shields.io/badge/crypto-post--quantum-6D5DF6" alt="post-quantum" />
<img src="https://img.shields.io/badge/FIPS-203%20%C2%B7%20204%20%C2%B7%20205-2BB6A3" alt="fips" />
</p>
<p>
<img src="https://img.shields.io/badge/tests-105%20Rust%20%2B%2028%20Flutter-brightgreen" alt="tests" />
<img src="https://img.shields.io/badge/fuzz-250k%2B%20inputs%20%C2%B7%200%20panics-brightgreen" alt="fuzz" />
<img src="https://img.shields.io/badge/Rust-000000?logo=rust&logoColor=white" alt="rust" />
<img src="https://img.shields.io/badge/Flutter-02569B?logo=flutter&logoColor=white" alt="flutter" />
<img src="https://img.shields.io/badge/platform-Android-3DDC84?logo=android&logoColor=white" alt="android" />
<img src="https://img.shields.io/badge/license-proprietary-E5483D" alt="license" />
</p>

<b>No server, no operator, no man-in-the-middle can ever read your chats — not even a future quantum computer.</b>

</div>

> [!WARNING]
> **AEGIS is a working prototype that has *not yet* had an independent crypto audit.** Everything below is real, built, and tested — and every limit is stated honestly. Do not rely on it for genuine high-risk communication until an external audit is complete.

---

## 📱 Two real phones. End-to-end. Post-quantum. Proven live.

<div align="center">
<table>
<tr>
<td align="center"><sub><b>Phone A · Xiaomi 15 Ultra</b></sub></td>
<td align="center"><sub><b>Phone B · Samsung Z Flip5</b></sub></td>
</tr>
<tr>
<td><img src="docs/ui/device-a.png" width="250" alt="AEGIS on phone A" /></td>
<td><img src="docs/ui/device-b.png" width="250" alt="AEGIS on phone B" /></td>
</tr>
</table>
<sub>The <b>same conversation</b> on two physical phones. Each message is encrypted on the sender → crosses the relay as an <b>opaque blob</b> → and is decrypted <b>only</b> on the recipient. Verified live, both directions.</sub>
</div>

---

## Why AEGIS exists

Most messengers protect *today's* messages against *today's* computers. The day a large quantum computer arrives, ciphertext harvested years ago can be decrypted retroactively — **"harvest-now, decrypt-later."**

**AEGIS is built for that day.** It is post-quantum on **both** axes — confidentiality *and* authenticity — and engineered so that **no server, no operator, and no man-in-the-middle can ever read your chats.** Even on a seized phone, it is designed to give up nothing.

## How it fits together

```mermaid
flowchart LR
    subgraph A["📱 You"]
      UA["Flutter UI"] -->|FFI| CA["aegis-core · Rust"]
    end
    R[("aegis-relay<br/>opaque blobs only")]
    subgraph B["📱 Contact"]
      CB["aegis-core · Rust"] -->|FFI| UB["Flutter UI"]
    end
    CA -->|"sealed blob"| R
    R -->|"sealed blob"| CB
    style R fill:#4F46E5,stroke:#312E81,color:#ffffff
```

All cryptography lives in a **Rust core** (`aegis-core`); the Flutter app calls it over FFI. The **relay** only moves opaque, encrypted blobs — it never sees plaintext, and with sealed-sender it does not even learn who sent a message. Your **contact code is just a 44-character mailbox id** — the heavy post-quantum key bundle is published to the relay, signed by your mailbox key so no one can impersonate you.

## 🔐 Cryptography we use

AEGIS does **not** invent primitives. It composes audited building blocks (RustCrypto / dalek) into the Signal-style protocol, and validates the composition against official test vectors.

| Purpose | Primitive | Standard |
|---|---|---|
| Wire AEAD | **ChaCha20-Poly1305** | RFC 8439 |
| At-rest AEAD | **XChaCha20-Poly1305** (192-bit nonce) | RFC 8439 / draft-irtf |
| Post-quantum KEM | **ML-KEM-1024** | FIPS 203 |
| Post-quantum signature | **ML-DSA-65** | FIPS 204 |
| Post-quantum signature (hash-based) | **SLH-DSA-SHAKE-128f** | FIPS 205 |
| Classical key agreement | **X25519** | RFC 7748 |
| Classical signature | **Ed25519** | RFC 8032 |
| Hashing | **SHA-256/512, SHA-3 / SHAKE** | FIPS 180-4 / 202 |
| Key derivation | **HKDF** | RFC 5869 |
| At-rest key from PIN | **Argon2id** (memory-hard) | RFC 9106 |
| Key hygiene | **zeroize-on-drop** + **mlock** (keys wiped from memory, locked against swap) | — |

**Handshake — PQXDH:** a Signal X3DH secret **‖** an ML-KEM-1024 shared secret, merged via HKDF. Secure as long as *either* the classical *or* the post-quantum assumption holds.

**Ratchet — continuous post-quantum:** every Double-Ratchet DH step mixes in a **fresh** ML-KEM-1024 secret → forward secrecy *and* post-compromise (self-healing) security, even against a quantum adversary. Text **and** media ride this same ratchet.

**Identity — triple-hybrid signatures:** identity & pre-keys are signed with **Ed25519 ‖ ML-DSA-65 ‖ SLH-DSA** at once. Verification requires **all three** — forgery means breaking classical *and* lattice *and* hash-based crypto simultaneously.

**Metadata — post-quantum sealed sender:** the sealed-sender layer that hides *who* sent each message is itself hybrid **X25519 ‖ ML-KEM-1024** with per-message forward secrecy — so the sender metadata is protected against a future quantum adversary too, not only the message body. The relay sees an opaque blob with no sender identity.

**One-time prekeys:** every new conversation consumes a **real single-use prekey** the relay dispenses from your published pool (never reused, consumed once on receipt) — exactly as X3DH/PQXDH intends, so two contacts never share prekey material.

## 🧨 We attacked it ourselves — hard

Authorized adversarial testing against our own code and devices. Highlights:

**Protocol attacks — all rejected:** identity-swap · ciphertext tamper · header tamper · replay · reorder-then-replay · forged signed-prekey · forged PQ-prekey · cross-session · malformed-wire · crafted-injection desync. *(11 protocol attacks + Google Wycheproof adversarial vectors.)*

**Fuzzing & DoS:** **250,000+** hostile/garbage inputs to the wire / envelope / relay parsers → **zero panics, zero overflows** (length-bounded, cap-before-allocation, memory-safe Rust).

**Live relay pentest (against the running server):**

| Attack | Result |
|---|---|
| Malformed opcode | connection closed, relay alive |
| 4 GiB length-prefix (OOM attempt) | rejected **before** allocation |
| Drain someone else's mailbox with a forged signature (BOLA) | **0 bytes delivered** |
| Substitute a victim's key bundle (impersonation) | **rejected** — bundles are owner-signed |
| 1024-message flood per mailbox | bounded (excess dropped) |
| 1100 concurrent connections | relay survived |

**Device pentest:** non-debuggable release (`run-as` blocked) · no exported components · malicious-intent fuzz → no crash · 2000 monkey events → no ANR · no plaintext in logcat · **no hardcoded secrets** in the native library.

**Standards audit:** mapped to **OWASP MASVS** · **MobSF** static scan (0 trackers, 0 secrets) · **StrongBox** hardware key-backing confirmed on a real device · minSDK raised to the StrongBox API floor.

**Result:** the only real finding was a *LOW* clipboard-plaintext issue — **fixed**. No cryptographic weakness found.

## 👻 Device & "amnesic" security

- **End-to-end only** — keys never leave the two phones; the relay carries opaque blobs.
- **Amnesic by design** — chats live in **RAM only** (no chat database on disk); media bytes stay in memory and temp files are securely shredded; keys are zeroized on drop; optional **wipe-everything-on-leave**.
- **Duress PIN** — entering it covertly wipes everything and shows an empty decoy account.
- **FLAG_SECURE** — screenshots, screen-recording, and the app-switcher preview are blocked.
- **Hardware-backed keys** — Android **StrongBox** (→ TEE fallback), biometric/PIN app-lock + auto-lock, no cloud backup. Without a secure element, the at-rest key can instead be **derived from your PIN via Argon2id** (only a salt is stored; the key is re-derived on unlock, never written to disk).
- **Swap-resistant secrets** — long-lived secret keys are **mlock**ed into RAM (best-effort) so they are not paged out to swap or hibernation.

## 💬 Features

<div align="center">
<table>
<tr><td valign="top">

🔒 **Real 2-device E2E** over the relay
📷 **Photos & voice notes** — AEAD-sealed, same ratchet
👥 Multiple chats & profiles
⏱️ Disappearing messages

</td><td valign="top">

😀 Emoji reactions · replies / quotes
🔎 In-chat search
🔢 Safety-number verification (anti-MITM)
🌍 **DE / EN / TR**

</td></tr>
</table>
</div>

<div align="center">
<img src="docs/ui/chat.png" width="220" alt="chat" />&nbsp;&nbsp;
<img src="docs/ui/contact-code.png" width="220" alt="add a real relay contact" />&nbsp;&nbsp;
<img src="docs/ui/settings.png" width="220" alt="settings" />
<br/><sub>Chat · adding a real contact (44-char code) · settings</sub>
</div>

## ✅ Verified, honestly

- **Crypto core:** **105** Rust tests green — RFC/FIPS KATs (incl. Argon2id RFC 9106), Wycheproof, adversarial tests, 250k+ fuzz, relay DoS & impersonation, one-time-prekey consumption, PQ-metadata tamper.
- **App:** **28** Flutter widget/unit tests green; built & run on real devices.
- **Live:** real **two-device** end-to-end messaging proven on two physical phones, both directions.
- **Guaranteed:** E2E confidentiality + integrity · forward secrecy · post-compromise security · replay/MITM protection · quantum resistance on **both axes + the sender-metadata layer**.
- **Best-effort / out of scope:** rooted or forensically-attacked devices · metadata vs. a network observer without a mixnet · **independent crypto audit + bug bounty (planned, not yet done)** · iOS.

## 🛠️ Tech stack

**Rust** (crypto core + relay) · **Flutter** + `flutter_rust_bridge` (Android) · RustCrypto / dalek primitives · FIPS 203 / 204 / 205 PQC.

## 📈 Status & roadmap

Working prototype (Android + Linux desktop), with **real 2-device relay messaging proven live.** The latest hardening — post-quantum sealed-sender metadata, one-time prekey pool, Argon2id at-rest key, and secret-`mlock` — is Rust/FFI-verified; a 2-device live re-test of the new wire format is the immediate next step.
**Next:** 2-device re-test of the new wire · independent crypto audit · encrypted persistence (Argon2id-backed) · iOS · formal verification (Tamarin / ProVerif).

## 📜 License & commercial use

AEGIS is **proprietary software — © 2026 Ozan Küsmez. All rights reserved.**
You may **view and evaluate** it. **Running, deploying, or any commercial use requires a paid license.**
To license, deploy, or build on AEGIS → **ozanks20@gmail.com** · see [`LICENSE`](LICENSE).

---

<div align="center">
<sub>Built by <b>Ozan Küsmez</b> · ozanks20@gmail.com</sub>
<br/>
<sub>#cryptography #postquantum #e2ee #securemessaging #mlkem #signalprotocol #rust #flutter #android #privacy #infosec</sub>
</div>
