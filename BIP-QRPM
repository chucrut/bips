# BIP: TBD
**Title:** Quantum-Resistant Protocol Migration (QRPM)
**Author:** Agustin Cruz agustin.cruz@gmail.com
**Status:** Draft
**Type:** Standards Track (Consensus)
**Created:** 2025-03-09
**License:** BSD-2-Clause  

---

## Abstract

This proposal outlines a plan to transition Bitcoin’s address and signature scheme to a quantum-resistant protocol. It introduces new address types secured by post-quantum cryptography (PQC), alongside a **turnstile mechanism** to migrate existing funds from legacy addresses without confiscation. The transition is implemented in phases, prioritizing the most vulnerable address types first and providing long-term incentives rather than imposing a sudden cut-off. This approach is designed to protect Bitcoin from an eventual quantum computing threat while respecting user sovereignty and minimizing disruptive impacts on the ecosystem.

## Motivation

Bitcoin relies on ECDSA (and Schnorr) for signatures, both of which could be threatened by large-scale quantum computers running Shor’s algorithm. While exact timelines for practical quantum computers remain uncertain, proactively preparing is essential to avoid a crisis scenario (a “Q-day”) in which large numbers of coins might be stolen.

However, an abrupt, forced migration that invalidates legacy coins after a certain date would be contentious, risking chain splits and accusations of confiscation. This BIP proposes a more gradual approach—introducing quantum-safe address types, allowing extended voluntary migration, and eventually restricting old signatures *only* to a turnstile that moves funds to new PQC-based outputs.

## Design Goals

1. **Future-Proofing Against Quantum Threats**  
   Transition to cryptographic primitives believed to be secure against quantum attacks, ensuring the long-term security of unspent outputs (UTXOs).

2. **No Forced Confiscation**  
   Avoid any protocol rule that outright invalidates or burns users’ funds if they fail to migrate by a certain deadline. Instead, restrict legacy spends so that they must transition through a PQC “turnstile,” but remain spendable indefinitely by the rightful owner.

3. **Phased Migration**  
   Prioritize the most vulnerable outputs first (e.g., P2PK, P2TR), followed by address types that still rely on ECC. Provide sufficient notice and incentives for all to migrate over a multi-year timeline, preventing congestion and fee spikes.

4. **Network Stability**  
   Mitigate potential fee manipulation, network spam, or chain splits. Retain backward compatibility wherever possible and use known soft-fork activation methods.

5. **Emergency Mechanisms**  
   Include a rollback option should the chosen quantum-resistant algorithms be found flawed, and a fast-activation path if a “Q-day” scenario arrives sooner than expected.

## Overview

### 1. Introducing Quantum-Resistant Addresses

A new output script type (e.g., using a new SegWit version) will allow spending from a quantum-safe public key. Example: **P2QPKH** (“Pay-to-Quantum-PubKey-Hash”), which mirrors P2PKH but uses a PQC key. Such addresses hide the large PQC public key behind a hash, revealed only in the spending transaction’s witness.

- **Algorithm Choices:** Potential candidates include lattice-based (e.g., Falcon) or hash-based (XMSS/LMS) signatures. An initial implementation may use **hybrid** signatures that require both ECDSA/Schnorr *and* PQC to reduce risks of undiscovered flaws.

- **Address Encoding:** A distinct Bech32 version (e.g., version 2 or higher) to ensure older wallets treat it as unknown. PQC public keys and signatures may be large; storing only a hash in the UTXO set preserves scalability.

### 2. Turnstile Mechanism

After a specified activation, spending from legacy outputs (P2PK, P2TR, etc.) may require transitioning to a PQC-based address with a valid PQC signature. This ensures that even if an attacker has a quantum computer capable of breaking ECC, they would still need to forge PQC to steal funds.

- **One-Way Transition:** Once coins move to a PQC address, they remain under PQC rules. Users cannot revert to ECC-only addresses, preventing regression to weaker security.

- **No Confiscation:** UTXOs remain spendable via a special *turnstile transaction*, which proves ownership with the old key plus a new PQC key. Lost coins remain locked, but are not automatically seized or destroyed.

### 3. Multi-Year, Phased Migration

Rather than imposing a single deadline, migration proceeds through stages to minimize disruption:

- **Phase A:** Within ~1-2 years of initial deployment, require P2PK/P2TR outputs to use the turnstile. P2PKH/P2WPKH remain spendable under ECC for now.
- **Phase B:** Extend the restriction to reused addresses and other known-exposed public keys.
- **Phase C:** After ~5+ years, require *all* ECC-based spends (including P2PKH) to go through the turnstile. Timelines can be shortened or extended by community consensus based on quantum threat assessments.

### 4. Fee and Congestion Mitigation

Migration surges can strain network capacity. Proposed measures include:

- **Miner Policy Coordination:** Miners voluntarily dedicate block space for turnstile transactions, keeping fees predictable.
- **Batching & Consolidation:** Wallets should merge multiple UTXOs into single transactions, reducing on-chain load.
- **Staggered Deadlines:** Migrating first the smaller set of high-risk UTXOs before the majority lowers the risk of fee spikes.

### 5. Activation and Rollback

Deployment follows a similar path to SegWit/Taproot soft forks:

1. **Soft Fork A:** Introduce P2QPKH and PQC signature rules (voluntary use).
2. **Soft Fork B:** Enforce turnstile spending for certain address types from block *N*, with further rule expansions in subsequent phases.
3. **Emergency Halt/Rollback:** If a serious flaw is discovered in the PQC scheme, an additional soft fork can suspend the new rules, safeguarding user funds.

## Reference Implementation

An initial implementation can be published as a pull request to Bitcoin Core, adding:

- **Consensus Code:** Recognize a new witness version for PQC. Validate hybrid or PQC signatures in `CheckInputScripts`.
- **Wallet Support:** Generate and store PQC keypairs. Provide migration tools to sweep legacy addresses through the turnstile.
- **Policy Adjustments:** Optionally introduce standardness rules (e.g., fee discounts, block space allocation) for turnstile transactions.

Extensive tests on regtest and testnet/signet will precede mainnet activation.

## Security Considerations

- **PQC Algorithm Strength:** Must be reviewed by cryptography experts. Hybrid mode is recommended initially.
- **Legacy UTXOs:** UTXOs in old formats remain valid but require PQC-based redemption post-activation, thwarting quantum thieves.
- **Network Attacks:** Potential DoS via large PQC signatures is mitigated by witness size limits and partial block space reservations.
- **Lost Coins:** Coins whose owners never migrate effectively become locked. This is more secure than letting quantum attackers seize them.

## Economic Impact

- **Fees & Block Space:** Expected short-term rise in transactions as users migrate. Batching/consolidation mitigates fee pressure.
- **Supply Integrity:** Prevents sudden quantum-driven reintroduction of dormant coins, safeguarding market confidence.
- **Long-term Adoption:** Demonstrates Bitcoin’s capacity to adapt without violating property rights, potentially strengthening investor trust.

## Deployment & Testing Strategy

1. **Reference Implementation & Code Review:** Community and cryptographic audits.  
2. **Testnet Activation:** Trial a compressed migration schedule, verifying performance, security, and user experience.  
3. **Mainnet Soft Fork A:** Activate new PQC outputs. Voluntary usage.  
4. **Mainnet Soft Fork B:** Gradual enforcement of the turnstile, phased by address type.  
5. **Continuous Monitoring:** Track UTXO migration rates. Provide an emergency path for major issues.

## Conclusion

This proposal provides a structured pathway to quantum-safe Bitcoin without sacrificing user sovereignty. By introducing PQC addresses, enforcing a turnstile mechanism over time, and offering phased deployment, it ensures that funds remain secure against future quantum attacks while mitigating network, economic, and social disruptions.
