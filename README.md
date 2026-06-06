<div align="center">

<img src="docs/ui/aegis-logo.svg" width="96" alt="AEGIS logo" />

# AEGIS Messenger

**A post-quantum, end-to-end encrypted messenger — built so only you and your contact can ever read a message.**

`Post-Quantum` · `End-to-End Encrypted` · `Rust core + Flutter` · `Android`

> ⚠️ **Prototype — not yet independently audited.** It works and is honest about its limits (see *Security*). Not for real high-risk use until an external crypto audit is done.

</div>

---

## Why AEGIS

Most messengers protect today's messages against today's computers. AEGIS protects them against **tomorrow's quantum computers too** — on *both* axes (confidentiality **and** authenticity) — and is engineered so that **no server, operator, or man-in-the-middle can ever read your chats**. Even on a seized phone, it is built to give up nothing.

## See it

| Chat | Settings |
|:---:|:---:|
| <img src="docs/ui/chat.png" width="240" alt="AEGIS chat" /> | <img src="docs/ui/settings.png" width="240" alt="AEGIS settings" /> |

## What it does

- 🔐 **Signal-style PQXDH** handshake (X3DH + Double Ratchet) with **ML-KEM-1024** — a fresh post-quantum secret is mixed into *every* ratchet step (forward secrecy + post-compromise security, even against a quantum adversary).
- 🪪 **Triple-hybrid identity signatures** — Ed25519 ∥ ML-DSA-65 ∥ SLH-DSA. Verification needs **all three**; unforgeable as long as *any one* of the classical / lattice / hash assumptions holds.
- 🛡️ **Hardened device security** — screenshot & screen-recording block (FLAG_SECURE), biometric/PIN app-lock + auto-lock, a **duress PIN** (covert wipe + decoy), StrongBox-backed keys, no cloud backup.
- 👻 **Amnesic by design** — chats live in RAM only (no chat database on disk), keys are zeroized on drop, optional wipe-everything-on-leave. Nothing lingers.
- 💬 **A real messenger** — multiple chats & profiles, **photos & voice messages** (each AEAD-sealed), reactions, replies, in-chat search, disappearing messages, safety-number verification. **DE / EN / TR**.
- 📡 **A "dumb" relay** — store-and-forward of opaque encrypted blobs only; no plaintext, no sender identity, minimal metadata.

## Verified — honestly

- **Crypto core:** 81 automated tests green — RFC/FIPS known-answer vectors, Wycheproof adversarial vectors, property tests, 250k+ fuzz inputs, relay DoS/abuse tests.
- **App:** widget/unit tested (chats, profile, amnesic media, localization), built and run on a real device, security-audited against OWASP MASVS (StrongBox confirmed, non-debuggable release, least-privilege permissions).
- **Guaranteed:** E2E confidentiality + integrity, forward secrecy, post-compromise security, replay/MITM protection, quantum resistance on both axes.
- **Best-effort / out of scope:** protections on a rooted or forensically-attacked device; metadata against a network observer without a mixnet; **independent crypto audit + bug bounty (planned, not yet done).**

## Tech stack

**Rust** (crypto core + relay, RustCrypto primitives) · **Flutter** + `flutter_rust_bridge` (Android app) · **ML-KEM-1024 / ML-DSA-65 / SLH-DSA** (FIPS 203 / 204 / 205) · ChaCha20-Poly1305 · X25519 · Ed25519.

## Status & roadmap

Working prototype (Android + Linux desktop). Next: independent crypto audit, encrypted persistence, media-over-relay transport, iOS, formal verification (Tamarin / ProVerif).

## License & commercial use

AEGIS is **proprietary software — © 2026 Ozan Küsmez. All rights reserved.**
You may view and evaluate it. **Running, deploying, or any commercial use requires a paid license.**
To license, deploy, or build on AEGIS, get in touch: **ozanks20@gmail.com** — see [`LICENSE`](LICENSE).

---

<div align="center"><sub>Built by <b>Ozan Küsmez</b> · ozanks20@gmail.com</sub></div>