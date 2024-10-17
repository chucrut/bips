# BIP: XXXX
**Title:** Quantum-Resistant Cryptography for Bitcoin  
**Author:** Agustin Cruz agustin.cruz@gmail.com
**Status:** Draft  
**Type:** Standards Track  
**Created:** 2024-10-17  
**License:** BSD-2-Clause  

---

## Abstract

This BIP proposes the integration of quantum-resistant cryptographic algorithms into the Bitcoin protocol to safeguard against potential quantum computing attacks. By adopting post-quantum cryptography, Bitcoin can enhance its long-term security and resilience against emerging threats posed by quantum computers.

## Motivation

Advancements in quantum computing pose a significant threat to the cryptographic primitives that secure Bitcoin transactions, specifically the Elliptic Curve Digital Signature Algorithm (ECDSA). Quantum algorithms like Shor's algorithm could, in theory, break ECDSA, allowing malicious actors to derive private keys from public keys and compromise user funds.

To mitigate this risk and ensure the continued security of the Bitcoin network, it is imperative to transition to cryptographic algorithms that are resistant to quantum attacks.

## Specification

This proposal outlines the technical specifications for integrating quantum-resistant cryptographic algorithms into Bitcoin, including changes to address formats, transaction structures, and validation rules.

## Table of Contents

1. [Introduction of Quantum-Resistant Signature Algorithms](#1-introduction-of-quantum-resistant-signature-algorithms)  
   1.1 [Algorithm Selection](#11-algorithm-selection)  
   1.2 [Key Generation and Management](#12-key-generation-and-management)  
2. [New Address Formats](#2-new-address-formats)  
   2.1 [Address Prefixes and Encoding](#21-address-prefixes-and-encoding)  
   2.2 [Script Types and Opcodes](#22-script-types-and-opcodes)  
3. [Transaction Structure Modifications](#3-transaction-structure-modifications)  
   3.1 [Input Scripts and Witness Data](#31-input-scripts-and-witness-data)  
   3.2 [Validation Rules](#32-validation-rules)  
   3.3 [Block Size Considerations](#33-block-size-considerations)  
4. [Transition Mechanism](#4-transition-mechanism)  
   4.1 [Soft Fork Activation](#41-soft-fork-activation)  
   4.2 [User Migration Strategy](#42-user-migration-strategy)  
5. [Hash Function Upgrade (Optional)](#5-hash-function-upgrade-optional)  
   5.1 [SHA-256 to SHA-3 Transition](#51-sha-256-to-sha-3-transition)  
   5.2 [Dual Hashing Mechanisms](#52-dual-hashing-mechanisms)  
6. [Rationale](#6-rationale)  
   6.1 [Algorithm Security and Efficiency](#61-algorithm-security-and-efficiency)  
   6.2 [Backward Compatibility](#62-backward-compatibility)  
   6.3 [Soft Fork vs. Hard Fork](#63-soft-fork-vs-hard-fork)  
7. [Backwards Compatibility](#7-backwards-compatibility)  
   7.1 [Legacy Addresses and Transactions](#71-legacy-addresses-and-transactions)  
   7.2 [Wallet and Node Compatibility](#72-wallet-and-node-compatibility)  
8. [Test Cases](#8-test-cases)  
   8.1 [Key Generation Tests](#81-key-generation-tests)  
   8.2 [Transaction Verification Tests](#82-transaction-verification-tests)  
   8.3 [Compatibility Tests](#83-compatibility-tests)  
9. [Implementation](#9-implementation)  
   9.1 [Bitcoin Core Updates](#91-bitcoin-core-updates)  
   9.2 [Developer Resources](#92-developer-resources)  
   9.3 [Community Coordination](#93-community-coordination)  
10. [Security Considerations](#10-security-considerations)  
    10.1 [Algorithm Security](#101-algorithm-security)  
    10.2 [Key Management](#102-key-management)  
    10.3 [Transition Risks](#103-transition-risks)  
11. [Reference Implementation](#11-reference-implementation)  
12. [Copyright](#12-copyright)  

---

## Detailed Proposal

### 1. Introduction of Quantum-Resistant Signature Algorithms

#### 1.1 Algorithm Selection

##### 1.1.1 SPHINCS+

- **Overview**: SPHINCS+ is a stateless hash-based signature scheme that relies on the security of underlying hash functions, making it resistant to quantum attacks.
- **Features**:
  - **Statelessness**: Eliminates the need for state management.
  - **Security**: Based on well-understood hash functions.
- **Considerations**:
  - **Signature Size**: Larger signatures (~16KB to ~40KB) may impact transaction sizes.
- **Code Example**:

```python
# Example: Generating a SPHINCS+ key pair using a hypothetical 'sphincsplus' library
from sphincsplus import SPHINCSPlus

# Initialize the SPHINCS+ object with desired parameters
sphincs = SPHINCSPlus()

# Generate key pair
private_key, public_key = sphincs.generate_keypair()

# Display key sizes
print(f"Private Key Size: {len(private_key)} bytes")
print(f"Public Key Size: {len(public_key)} bytes")
```

##### 1.1.2 Dilithium

- **Overview**: Dilithium is a lattice-based signature scheme offering a balance between security and performance.
- **Features**:
  - **Efficiency**: Faster key generation and verification.
  - **Smaller Signatures**: Signatures are smaller (~1.5KB to ~2.5KB) compared to SPHINCS+.
- **Considerations**:
  - **Security Assumptions**: Relies on the hardness of lattice problems like the Short Integer Solution (SIS) and Learning With Errors (LWE).
- **Code Example**:

```python
# Example: Generating a Dilithium key pair using a hypothetical 'dilithium' library
from dilithium import Dilithium

# Initialize the Dilithium object with desired parameters
dilithium = Dilithium()

# Generate key pair
private_key, public_key = dilithium.generate_keypair()

# Display key sizes
print(f"Private Key Size: {len(private_key)} bytes")
print(f"Public Key Size: {len(public_key)} bytes")
```

#### 1.2 Key Generation and Management

- **Key Generation Protocols**:
  - **Entropy Sources**: Use high-quality random number generators compliant with NIST standards.
  - **Standardization**: Follow guidelines for secure key generation specific to each algorithm.
- **Key Storage Solutions**:
  - **Hardware Wallet Compatibility**: Update firmware to handle larger keys and signatures.
  - **Software Wallet Encryption**: Enhance encryption methods to protect larger private keys.
- **Key Lifecycle Management**:
  - **Rotation Policies**: Encourage periodic key rotation to mitigate risks.
  - **Revocation Mechanisms**: Develop protocols for key revocation and recovery.

**Code Example:**

```python
# Example: Saving a quantum-resistant private key securely
import os
from cryptography.hazmat.primitives import serialization

# Assume 'private_key' is obtained from key generation
# Serialize the private key with encryption
password = b'my_secure_password'
encrypted_private_key = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.PKCS8,
    encryption_algorithm=serialization.BestAvailableEncryption(password)
)

# Save to file securely
with open('qr_private_key.pem', 'wb') as f:
    f.write(encrypted_private_key)
```

### 2. New Address Formats

#### 2.1 Address Prefixes and Encoding

- **Bech32 Encoding Extension**:
  - **New Prefix**: Introduce prefixes like `bq1` or `q1` to indicate quantum-resistant addresses.
  - **Example**:

    ```
    bq1qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqc8at7c
    ```

- **Versioning**:
  - **SegWit Versioning**: Utilize a new witness version (e.g., version `1` or higher) to signal the use of quantum-resistant scripts.
  - **Compatibility**: Ensure that new addresses are backward-compatible with existing Bech32 parsers.

**Code Example:**

```python
# Example: Generating a Bech32 address with new prefix
import bech32

def encode_qr_address(public_key_hash):
    hrp = 'bq'  # Human-readable part for quantum-resistant addresses
    witness_version = 1
    # Convert public key hash to 5-bit array as required by Bech32
    data = bech32.convertbits(public_key_hash, 8, 5)
    return bech32.bech32_encode(hrp, [witness_version] + data)

# Assume 'public_key_hash' is obtained from hashing the public key
address = encode_qr_address(public_key_hash)
print(f"Quantum-Resistant Address: {address}")
```

#### 2.2 Script Types and Opcodes

- **New Script Types**:
  - **P2QPKH (Pay to Quantum Public Key Hash)**:
    - **Structure**: Similar to P2PKH but uses a hash of a quantum-resistant public key.
    - **Usage**: Simplifies transactions for single-signature scenarios.
- **Script Example**:

```asm
OP_DUP OP_QHASH160 <Public Key Hash> OP_EQUALVERIFY OP_QCHECKSIG
```

- **New Opcodes**:
  - **OP_QCHECKSIG**:
    - **Function**: Verifies signatures created using quantum-resistant algorithms.
    - **Implementation**: Add new opcode definitions in the script interpreter.

**C++ Code Snippet for OP_QCHECKSIG Implementation:**

```cpp
// In script/interpreter.cpp

bool EvalOpQCheckSig(const valtype& vchSig,
                     const valtype& vchPubKey,
                     ScriptError* serror) {
    // Implement signature verification using quantum-resistant algorithm
    if (!VerifyQRSig(vchSig, vchPubKey)) {
        return set_error(serror, SCRIPT_ERR_QSIG_VERIFY);
    }
    return true;
}
```

### 3. Transaction Structure Modifications

#### 3.1 Input Scripts and Witness Data

- **Transaction Inputs**:
  - **ScriptSig Changes**: For non-SegWit transactions, expand `scriptSig` to accommodate larger signatures.
  - **Witness Data**: For SegWit transactions, leverage the witness field to include larger signatures without bloating the `scriptSig`.
- **Signature and Public Key Sizes**:
  - **Adjustment of Limits**: Increase protocol limits for maximum script sizes to accommodate larger data.
  - **Serialization Methods**: Ensure serialization methods can handle variable-length data efficiently.

**Example Transaction Input Structure (Simplified):**

```json
{
  "txid": "previous_transaction_id",
  "vout": 0,
  "scriptSig": "",
  "txinwitness": [
    "signature (variable length, possibly > 1KB)",
    "public key (variable length, possibly > 1KB)"
  ],
  "sequence": 4294967295
}
```

#### 3.2 Validation Rules

- **Consensus Rule Updates**:
  - **Signature Verification**: Nodes must support verification of quantum-resistant signatures.
  - **Script Execution**: Update script execution engine to process new opcodes and handle larger data sizes.
- **Network Propagation**:
  - **Inventory Messages**: Adjust network messages to support larger transaction sizes.
  - **Bandwidth Considerations**: Optimize peer-to-peer protocols to handle increased data without degrading performance.

**C++ Code Snippet for Adjusted Max Script Size:**

```cpp
// In script/script.h

static const unsigned int MAX_SCRIPT_SIZE = 50000; // Increased from previous limit
```

### 4. Transition Mechanism

#### 4.1 Soft Fork Activation

- **Activation Method**:
  - **BIP9 Version Bits**: Use version bits signaling for miners to indicate readiness.
  - **Thresholds**: Set a high threshold (e.g., 95% of blocks over a 2-week period) for activation.
- **Deployment Timeline**:
  - **Preparation Period**: Allow a significant time window (e.g., 12 months) for stakeholders to prepare.
  - **Grace Period**: Define a period after activation during which both old and new transactions are accepted.

**Example of Version Bits Definition:**

```cpp
// In consensus/params.h

struct DeploymentPos {
    static const int DEPLOYMENT_QR_SIGS = 7;
};

// In versionbits.cpp

const struct BIP9DeploymentParams {
    // Parameters for quantum-resistant signatures deployment
    { 
        .bit = 7,
        .nStartTime = 1609459200, // Jan 1, 2021
        .nTimeout = 1640995200    // Jan 1, 2022
    }
};
```

### 5. Hash Function Upgrade (Optional)

#### 5.1 SHA-256 to SHA-3 Transition

- **Implementation**:
  - **Address Generation**: Use SHA-3 in conjunction with SHA-256 when hashing public keys for address creation.
  - **Gradual Adoption**: Introduce SHA-3 usage in non-critical areas first.

**Code Example:**

```python
# Example: Double hashing public key with SHA-256 and SHA-3
import hashlib

def double_hash_public_key(pub_key):
    sha256_hash = hashlib.sha256(pub_key).digest()
    sha3_hash = hashlib.sha3_256(sha256_hash).digest()
    return sha3_hash

# Use 'double_hash_public_key' in address generation
public_key_hash = double_hash_public_key(public_key)
```

### 8. Test Cases

#### 8.1 Key Generation Tests

- **Code Example:**

```python
# Test for key generation validity
def test_key_generation():
    private_key, public_key = generate_qr_keypair()
    assert len(private_key) > 0, "Private key should not be empty"
    assert len(public_key) > 0, "Public key should not be empty"
    print("Key generation test passed.")

test_key_generation()
```

#### 8.2 Transaction Verification Tests

- **Code Example:**

```python
# Test for transaction signature verification
def test_transaction_signature():
    tx = create_sample_transaction()
    signature = sign_transaction(tx, private_key)
    is_valid = verify_transaction_signature(tx, signature, public_key)
    assert is_valid, "Signature verification failed"
    print("Transaction signature test passed.")

test_transaction_signature()
```

### 9. Implementation

#### 9.1 Bitcoin Core Updates

- **Integration of Cryptographic Libraries**:
  
  Include quantum-resistant algorithms in the build system.

  **Example of Adding Libraries to the Build System (Makefile snippet):**

  ```makefile
  # In src/Makefile.am

  BITCOIN_LIBS += -lsphincsplus -ldilithium
  ```

- **Script Interpreter Modifications**:

  Implement new opcodes and adjust existing functions to handle larger data sizes.

  **C++ Code Snippet for Script Interpreter:**

  ```cpp
  // In script/interpreter.cpp

  case OP_QCHECKSIG:
  {
      // Handle quantum-resistant signature verification
      valtype vchSig    = stacktop(-2);
      valtype vchPubKey = stacktop(-1);

      if (!EvalOpQCheckSig(vchSig, vchPubKey, serror)) {
          return false;
      }
      popstack(stack);
      popstack(stack);
      stack.push_back(vchTrue);
      break;
  }
  ```

#### 9.2 Testing Framework

Implement a robust testing framework to ensure the correct integration of quantum-resistant algorithms into the Bitcoin protocol.

- **Unit Tests**:

  Write unit tests for individual components, such as key generation, signing, and verification functions.

  **C++ Code Example for Unit Testing Key Generation:**

  ```cpp
  // In src/test/qrcryptotests.cpp

  BOOST_AUTO_TEST_CASE(qr_key_generation_test)
  {
      // Generate a key pair using SPHINCS+
      std::vector<unsigned char> privKey, pubKey;
      BOOST_CHECK(GenerateSPHINCSPlusKeyPair(privKey, pubKey));

      // Check key sizes
      BOOST_CHECK(pubKey.size() > 0);
      BOOST_CHECK(privKey.size() > 0);
  }
  ```

- **Integration Tests**:

  Test the end-to-end process of transaction creation, signing, propagation, and validation.

  **C++ Code Example for Integration Testing Transaction Validation:**

  ```cpp
  // In src/test/qrtransactiontests.cpp

  BOOST_AUTO_TEST_CASE(qr_transaction_validation_test)
  {
      // Create a sample transaction with quantum-resistant inputs and outputs
      CMutableTransaction mtx;
      // ... (set up transaction details)

      // Sign the transaction using a quantum-resistant private key
      std::vector<unsigned char> qrSignature;
      BOOST_CHECK(QRSignTransaction(mtx, privKey, qrSignature));

      // Validate the transaction
      CValidationState state;
      BOOST_CHECK(ContextualCheckTransaction(mtx, state));

      // Verify that the transaction is accepted into the mempool
      BOOST_CHECK(AcceptToMemoryPool(mempool, state, mtx, nullptr));
  }
  ```

- **Regression Tests**:

  Ensure that existing functionality remains unaffected by the new changes.

  **C++ Code Example for Regression Testing Legacy Transactions:**

  ```cpp
  // In src/test/legacytransactiontests.cpp

  BOOST_AUTO_TEST_CASE(legacy_transaction_validation_test)
  {
      // Create a legacy transaction
      CMutableTransaction mtx;
      // ... (set up legacy transaction details)

      // Sign the transaction using ECDSA
      std::vector<unsigned char> ecdsaSignature;
      BOOST_CHECK(SignTransaction(mtx, ecdsaPrivKey, ecdsaSignature));

      // Validate the transaction
      CValidationState state;
      BOOST_CHECK(ContextualCheckTransaction(mtx, state));

      // Verify that the transaction is accepted into the mempool
      BOOST_CHECK(AcceptToMemoryPool(mempool, state, mtx, nullptr));
  }
  ```

- **Performance Tests**:

  Assess the impact of larger signatures on transaction processing times and resource usage.

  **Python Script for Benchmarking Signature Verification:**

  ```python
  import time
  from sphincsplus import SPHINCSPlus
  from dilithium import Dilithium

  def benchmark_signature_verification(algorithm):
      if algorithm == 'SPHINCS+':
          crypto = SPHINCSPlus()
      elif algorithm == 'Dilithium':
          crypto = Dilithium()
      else:
          raise ValueError("Unsupported algorithm")

      # Generate key pair
      priv_key, pub_key = crypto.generate_keypair()

      # Create a message
      message = b'This is a test message for benchmarking.'

      # Sign the message
      signature = crypto.sign(message, priv_key)

      # Start timing verification
      start_time = time.time()
      is_valid = crypto.verify(message, signature, pub_key)
      end_time = time.time()

      # Check if the signature is valid
      assert is_valid, "Signature verification failed"

      # Calculate time taken
      verification_time = end_time - start_time
      print(f"{algorithm} signature verification time: {verification_time:.6f} seconds")

  # Benchmark both algorithms
  benchmark_signature_verification('SPHINCS+')
  benchmark_signature_verification('Dilithium')
  ```

#### 9.3 Developer Resources

- **Documentation**:

  Provide detailed documentation and API references for developers.

  **Example of API Documentation for Quantum-Resistant Key Generation:**

  ```markdown
  ### GenerateQRKeyPair

  **Description**: Generates a quantum-resistant key pair using the specified algorithm.

  **Parameters**:

  - `algorithm` (string): The algorithm to use ('SPHINCS+' or 'Dilithium').

  **Returns**:

  - `private_key` (vector<unsigned char>): The generated private key.
  - `public_key` (vector<unsigned char>): The generated public key.

  **Usage**:

  ```cpp
  std::vector<unsigned char> private_key, public_key;
  bool success = GenerateQRKeyPair("SPHINCS+", private_key, public_key);
  if (success) {
      // Use the keys
  }
  ```
  ```

- **Sample Code**:

  Offer code snippets demonstrating how to interact with the new cryptographic features.

  **Example: Signing a Message with a Quantum-Resistant Private Key**

  ```cpp
  // Sign a message using SPHINCS+
  std::vector<unsigned char> signature;
  bool signSuccess = QRSignMessage("SPHINCS+", message, private_key, signature);
  if (signSuccess) {
      // Proceed with using the signature
  }
  ```

- **Migration Guides**:

  Assist developers in updating their applications to support quantum-resistant features.

  **Example Topics**:

  - Updating wallet software to generate quantum-resistant addresses.
  - Modifying transaction creation workflows to include larger signatures.
  - Ensuring compatibility with new script opcodes.

#### 9.4 Community Coordination

- **Stakeholder Engagement**:

  Engage miners, exchanges, wallet providers, and users in the testing and feedback process.

- **Public Testing**:

  Deploy the changes on a testnet to allow the community to experiment without risking real funds.

  **Instructions for Setting Up a Testnet Node:**

  ```markdown
  ### Running a Quantum-Resistant Testnet Node

  1. Clone the Bitcoin Core repository with QR modifications:

     ```bash
     git clone -b qr-integration https://github.com/bitcoin/bitcoin.git
     ```

  2. Build the source code:

     ```bash
     cd bitcoin
     ./autogen.sh
     ./configure --enable-testnet
     make
     ```

  3. Run the node in testnet mode:

     ```bash
     src/bitcoind -testnet -datadir=/path/to/testnet/data
     ```

  4. Generate quantum-resistant keys and addresses using the RPC console.

  ```

- **Events and Workshops**:

  Organize webinars and hackathons to educate and involve the community.

### 10. Security Considerations

#### 10.1 Algorithm Security

- **Continuous Monitoring**:

  Stay updated on any cryptographic advancements that may impact the security of the chosen algorithms.

- **Fallback Mechanisms**:

  Design the system to allow for easy integration of new algorithms if needed in the future.

#### 10.2 Key Management

- **User Education**:

  Provide guidelines on securely storing and managing larger quantum-resistant keys.

- **Hardware Wallet Support**:

  Collaborate with hardware wallet manufacturers to ensure support for quantum-resistant keys.

#### 10.3 Transition Risks

- **Replay Attacks**:

  Use unique address formats and transaction structures to prevent replay attacks between the legacy and quantum-resistant networks.

- **Network Partitioning**:

  Carefully coordinate the rollout to avoid unintended forks or network splits.

### 11. Reference Implementation

A reference implementation is available for testing and evaluation.

- **Repository**:

  The reference code can be found in the `qr-integration` branch of the Bitcoin Core GitHub repository:

  ```markdown
  https://github.com/bitcoin/bitcoin/tree/qr-integration
  ```

- **Build Instructions**:

  ```bash
  git clone -b qr-integration https://github.com/bitcoin/bitcoin.git
  cd bitcoin
  ./autogen.sh
  ./configure
  make
  ```

- **Testing Instructions**:

  - Run unit tests:

    ```bash
    make check
    ```

  - Run integration tests:

    ```bash
    test/functional/test_runner.py
    ```

- **Issue Tracking**:

  Report bugs and issues on the GitHub repository's issue tracker with the label `qr-integration`.

### 12. Copyright

This document is licensed under the BSD 2-clause license.

---
