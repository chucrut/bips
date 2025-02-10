# BIP: XXXX
**Title:** Quantum-Resistant Address Migration Protocol (QRAMP)
**Author:** Agustin Cruz agustin.cruz@gmail.com
**Status:** Draft  
**Type:** Standards Track  
**Created:** 2025-02-10
**License:** BSD-2-Clause  

---

## Abstract

This BIP proposes a consensus change that enforces a mandatory migration period for funds held in legacy Bitcoin addresses (i.e. addresses secured by ECDSA as defined in current protocol rules) to quantum-resistant addresses. After a specified migration deadline, any transaction attempting to spend funds from a legacy address will be considered invalid. The goal is to mitigate the risk of quantum computing attacks which could, in a worst-case scenario, compromise private keys and allow unauthorized spending.

## Motivation

Quantum computing poses a theoretical risk to the cryptographic primitives underlying Bitcoin’s ECDSA signatures. While many legacy addresses (especially those that have not revealed their public key) are protected by additional layers (such as address hashing), addresses that have exposed their public keys may become vulnerable if sufficiently powerful quantum computers emerge.

Rather than relying solely on user-initiated migrations, this proposal enforces a protocol-level migration deadline to:
- Prevent potential large-scale unauthorized spending of funds from legacy addresses.
- Encourage and accelerate the adoption of quantum-resistant signature schemes.
- Protect the overall value and security of the Bitcoin network in the event of quantum threats.

## Specification


### Definitions

- **Legacy Address:** An address that uses the currently deployed ECDSA-based cryptography (for example, P2PKH or P2SH addresses that do not support quantum-resistant schemes).  
  *Example:* A Bitcoin address starting with “1” or “3” on mainnet.

- **Quantum-Resistant Address (QRA):** An address that employs a quantum-resistant signature scheme. Examples include:
  - **Hash-based signature schemes:** Such as XMSS (eXtended Merkle Signature Scheme) or LMS (Leighton-Micali Signature System).
  - **Lattice-based cryptography:** Signatures based on hard lattice problems.
  
  *Note:* The specific quantum-resistant scheme(s) will be detailed in a separate BIP or referenced technical document.

- **Migration Period:** A predefined time window (specified in either block heights or timestamps) during which users are encouraged to migrate funds.  
  *Example:* A migration period lasting 6 months or 100,000 blocks, during which legacy transactions remain valid.

- **Migration Deadline:** A specific block height or timestamp after which any transaction spending funds from a legacy address is rejected by nodes running the upgraded software.  
  *Example:* If `T_deadline` is set to block height 700,000, any transaction included in block 700,000 or later that attempts to spend from a legacy address will be invalid.

### Protocol Rules

1. **Activation**
   - **Hard Fork Trigger:**  
     The upgrade activates via a hard fork at a predetermined block height or timestamp.  
     *Example:* Activation at block height 680,000 with a migration period running until block height 700,000.
   - **Grace Period:**  
     During the migration period (blocks 680,000 to 699,999 in our example), both legacy and quantum-resistant addresses are valid. Wallets and explorers should prominently display warnings to users that funds held in legacy addresses must be migrated before block 700,000.

2. **Enforcement of Migration Deadline**
   - Define `T_deadline` as the cutoff point (block height or timestamp). For any transaction `tx` included in a block with timestamp (or height) ≥ `T_deadline`:
     - **If** `tx` spends funds from an output locked by a legacy address, the transaction is rejected.
     - **Else** (if funds are from a quantum-resistant address), the transaction is processed as usual.
   - *Example:* A transaction attempting to use a legacy P2PKH input at block height 701,000 would be rejected, while a similar transaction using an XMSS-based QRA would be accepted.

3. **Script Verification Changes**
   - The script interpreter is modified to add a check based on the block’s timestamp/height:
     - **If** the output’s locking script is associated with a legacy address **and** the current block time/height ≥ `T_deadline`, **then** the script evaluation fails immediately.
     - **Else,** the standard script evaluation process continues.
   - This change must be implemented with careful testing to avoid accidental rejections near the deadline boundary.

4. **Transition Considerations**
   - **Node and Miner Upgrades:**  
     All nodes and miners must upgrade their software to enforce the new rules after `T_deadline`.
   - **User Notifications:**  
     Wallet providers, block explorers, and other infrastructure must display clear warnings. For example, a wallet might show a banner reading: “Warning: Funds in legacy addresses will be unspendable after block 700,000. Please migrate your funds to a quantum-resistant address.”
   - **Emergency Migration Mechanisms:**  
     Consider providing a mechanism (such as an incentivized migration bonus or automated migration tool) during the migration period to assist less technically inclined users.

### Pseudocode Example

Below is a simplified pseudocode snippet that illustrates the concept. (Note: This is not production code, and actual implementation details will vary.)

```plaintext
function verifyTransaction(tx, currentBlockHeight):
    for each input in tx.inputs:
        UTXO = lookupUTXO(input)
        
        // Check if the output is locked by a legacy address.
        if UTXO.addressType == LEGACY:
            // Enforce the migration deadline.
            if currentBlockHeight >= MIGRATION_DEADLINE:
                errorMessage = "Transaction rejected: Funds from legacy addresses are locked post-migration deadline."
                logError(errorMessage, tx, UTXO)
                return VERIFICATION_FAILED(errorMessage)
        
        // Proceed with standard script evaluation.
        if not evaluateScript(input.scriptSig, UTXO.scriptPubKey):
            errorMessage = "Transaction rejected: Script evaluation failed for input."
            logError(errorMessage, tx, UTXO)
            return VERIFICATION_FAILED(errorMessage)
    
    return VERIFICATION_SUCCESS

function logError(message, tx, UTXO):
    // Detailed logging for debugging and audit purposes.
    print("Verification Error:", message, "Transaction ID:", tx.id, "UTXO Address:", UTXO.address)
```


## Rationale

The core rationale for this proposal is to safeguard Bitcoin against future quantum attacks by:

- **Reducing Vulnerabilities:** Transitioning funds to quantum-resistant schemes preemptively eliminates the risk of a quantum attack on exposed public keys.
- **Enforcing Timelines:** A hard deadline forces timely action rather than relying on a gradual, voluntary migration that might leave many users at risk.
- **Balancing Risks:** While the risk of funds being permanently locked is non-trivial, this risk is weighed against the potential catastrophic damage of a quantum attack on Bitcoin’s security.

## Addressing Criticisms

- **Risk of Permanent Fund Loss:**  
  To mitigate this, the migration period will be set generously based on extensive community and academic research. Multiple safeguards (user notifications, auto-migration tools, and possibly a temporary grace period extension mechanism) may be implemented.

- **Uncertain Quantum Timelines:**  
  Even if the viability of quantum computers remains uncertain, proactive measures ensure the network’s long-term security. The migration period can be adjusted based on evolving threat assessments.

- **Potential Chain Splits:**  
  Coordination with major stakeholders (miners, exchanges, wallet providers) will be critical. A detailed communication plan and phased rollouts (perhaps first on testnets) will help minimize disruption.

## Backwards Compatibility

This proposal is a hard fork with two distinct phases:

### Pre-Migration Deadline (Before `T_deadline`)
- Legacy and quantum-resistant transactions are both valid.
- Nodes operate under the old script validation rules.

### Post-Migration Deadline (After `T_deadline`)
- Transactions spending outputs from legacy addresses are rejected.
- Only transactions using quantum-resistant addresses will be accepted by upgraded nodes.

**Example:**  
Imagine a network where 60% of nodes have upgraded. In the transition period, legacy transactions are accepted by all. Post-deadline, if non-upgraded nodes continue to accept legacy transactions, a network split might occur. Therefore, coordination and a sufficiently high upgrade threshold are imperative.

## Security Considerations

- **Risk of Premature Activation:**  
  If the migration deadline is set too early, users might lose access to funds they have not migrated. A thorough analysis and broad community consensus are required to set an appropriate deadline.

- **Quantum Threat Timing:**  
  Should quantum breakthroughs occur earlier than anticipated, even a generous migration period might prove insufficient. Regular reviews and the possibility of an emergency upgrade mechanism are advised.

- **Attack Vectors During Migration:**  
  An attacker might publicize the deadline to induce panic selling or rushed migrations, potentially causing network instability. Robust communication and educational campaigns are crucial to manage user behavior.

## Test Cases

The following test cases should be executed in both regtest and testnet environments to ensure correct behavior:

### Before Migration Deadline

- **Test 1A:**  
  **Scenario:** A transaction spending funds from a legacy address at block height 680,500.  
  **Expected Outcome:** The transaction is valid.

- **Test 1B:**  
  **Scenario:** A transaction spending funds from a quantum-resistant address at block height 680,500.  
  **Expected Outcome:** The transaction is valid.

### After Migration Deadline

- **Test 2A:**  
  **Scenario:** A transaction spending funds from a legacy address at block height 700,100.  
  **Expected Outcome:** The transaction is rejected with an error message indicating that legacy address funds are locked.

- **Test 2B:**  
  **Scenario:** A transaction spending funds from a quantum-resistant address at block height 700,100.  
  **Expected Outcome:** The transaction is valid.

### Boundary Conditions

- **Test 3A:**  
  **Scenario:** A transaction included in a block exactly at the migration deadline (block height 700,000) that spends from a legacy address.  
  **Expected Outcome:** The transaction is evaluated based on the block’s consensus (e.g., if the rule is enforced on the block in which the deadline is reached, it should be rejected).

- **Test 3B:**  
  **Scenario:** A transaction that includes multiple inputs, some from legacy and some from quantum-resistant addresses.  
  **Expected Outcome:** The entire transaction is rejected if any legacy address output is spent post-deadline.

## Implementation

### Reference Implementation

A reference implementation should include:

- **Script Interpreter Changes:**  
  Modifications to the Bitcoin Core script interpreter to enforce the migration rule. For example, adding a condition check against `MIGRATION_DEADLINE` before processing legacy address inputs.

- **Wallet Software Updates:**  
  Integration of warning messages and automatic migration tools in wallet software.  
  **Example:** A wallet could show a progress meter or a “Migrate Now” button next to legacy addresses.

- **Network Monitoring Tools:**  
  Tools that monitor the rate of migrations and alert developers to potential issues near the deadline.

### Deployment Strategy

- **Testnet Rollouts:**  
  Deploy the changes on testnet to gather data, test fallback scenarios, and validate user behavior.

- **Phased Activation:**  
  Consider a phased activation where the migration rule is first enforced on a subset of nodes, followed by full activation post-deadline.

- **Emergency Rollback Procedures:**  
  Define clear rollback procedures in case the enforced migration causes unforeseen problems.




