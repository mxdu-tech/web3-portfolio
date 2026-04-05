# Voice Wallet — Natural Language Web3 Wallet

[中文版本](./README_CN.md)

---

## Project Resources & Demo

### Repository

- GitHub Repository  
  https://github.com/mxdu-tech/web3-portfolio/tree/main/voice-wallet

---

### Architecture & Design

### Architecture & Knowledge Map

![Architecture](./images/architecture.png)

This diagram illustrates the system's layered architecture and the relationships between core modules.

---

### Demo Video

https://youtu.be/CS7R_RBFGTI

[![Voice Wallet Demo](https://img.youtube.com/vi/CS7R_RBFGTI/hqdefault.jpg)](https://youtu.be/CS7R_RBFGTI)

---

## 1. Overview

Voice Wallet is a browser-based self-custodial wallet that enables users to initiate blockchain transactions using natural language.

Instead of manually specifying parameters such as address, amount, and gas, this system abstracts the transaction construction process through a natural language interface.

The system transforms user input into structured transaction data and executes signing and broadcasting locally.

The core focus of this project includes:

- Abstracting wallet interaction
- Systematizing transaction construction
- Designing secure boundaries under self-custody constraints

---

## 2. Problem & Motivation

Current Web3 wallets present several usability challenges:

- High complexity (address, network, gas management)
- Fragmented workflows (copy/paste across contexts)
- Poor accessibility for non-technical users
- Irreversible errors in user input

---

This project aims to:

- Shift transaction construction from users to the system
- Provide a more intuitive interaction model
- Maintain full user control over assets (self-custody)

---

## 3. System Architecture

![Architecture](./images/architecture.png)

The system follows a layered architecture:

- UI Layer
- Intent Processing Layer
- Wallet Core Layer
- Transaction Execution Layer
- Blockchain Interaction Layer
- Extension Runtime Layer

Each layer is decoupled through well-defined data flows.

---

## 4. Core Components

### 4.1 Wallet Core

The Wallet Core defines the system’s security boundary, handling key management and account state.

---

#### Key Management

- Private keys are derived from mnemonic phrases (BIP39)
- Decrypted keys exist only in memory
- No plaintext persistence

This minimizes exposure and reduces attack surface.

---

#### Vault Encryption

Encryption is implemented using browser-native cryptography:

- Web Crypto API
- PBKDF2 (password-based key derivation)
- AES-GCM (authenticated encryption)

PBKDF2 strengthens resistance against brute-force attacks, while AES-GCM ensures both confidentiality and integrity.

Choosing Web Crypto API:

- Native browser support
- Reduced dependency risk

---

#### Account Derivation

- Based on HD Wallet structure
- Standard derivation path: m/44'/60'/0'/0/0

Ensures compatibility with mainstream wallets such as MetaMask.

---

### 4.2 Transaction Execution

Core pipeline:

User Input → Validation → Signing → Broadcast

---

#### Transaction Builder

Implemented using:

- ethers.js
- viem

Responsibilities:

- Construct transaction payload (to, value, data, chainId)
- Encode contract calls
- Interact with RPC providers

---

#### Validation

Performed before transaction construction:

- Address format validation
- Balance check
- Network compatibility check

Prevents invalid inputs from reaching the signing stage.

---

#### Signing

- Executed locally
- Uses in-memory private keys

No private key is transmitted externally.

---

#### Broadcast

- Uses RPC provider
- Calls JSON-RPC method `eth_sendRawTransaction`

---

### 4.3 Blockchain Interaction

Handles communication with blockchain nodes:

- RPC request handling
- Balance queries
- Transaction broadcasting

The wallet interacts with the blockchain via RPC endpoints rather than direct node connections.

---

### 4.4 Extension Runtime

The system is implemented as a browser extension.

---

#### Background Script

- Persistent execution environment
- Hosts wallet core logic

---

#### Message Passing

Communication between UI and background:

- chrome.runtime.sendMessage

Ensures separation of concerns.

---

#### Chrome Storage

- Uses chrome.storage.local
- Stores only encrypted vault data

---

## 5. Transaction Flow

End-to-end pipeline:

1. User input (natural language)
2. Intent parsing
3. Parameter extraction
4. Validation
5. Transaction construction
6. User confirmation
7. Local signing
8. Broadcast
9. Result return (transaction hash)

---

## 6. Tech Stack

### WDK (Wallet Development Kit)

Provides wallet abstraction:

- Wallet creation
- Account management
- Signing capabilities

Avoids reimplementing low-level wallet primitives.

---

### ethers / viem

EVM interaction libraries:

- Transaction construction
- Smart contract calls
- RPC communication

---

### Web Crypto API

Browser-native cryptography used for:

- Mnemonic encryption
- Key derivation

---

### Chrome Extension API

Provides:

- Local storage (chrome.storage)
- Message passing
- Background execution

---

### Frontend

- React
- TypeScript
- Tailwind CSS

---

## 7. Design Decisions & Trade-offs

### Natural Language Interface

Advantages:

- Improves usability
- Reduces cognitive load

Challenges:

- Requires parsing layer
- Potential ambiguity

---

### Self-Custody Model

Advantages:

- Full user control over assets
- Trustless model

Challenges:

- User responsibility for key management

---

### In-Memory Key Handling

Advantages:

- Reduced risk of key leakage

Challenges:

- Requires frequent unlock operations

---

### Browser Extension Form

Advantages:

- Familiar UX
- Easy distribution

Challenges:

- Browser environment constraints

---

## 8. Future Work

- Multi-chain support
- Improved intent parsing (LLM optimization)
- Transaction simulation
- Gas optimization

---

## 9. Conclusion

This project explores:

- Wallet architecture design
- Abstraction of user interaction
- Balancing security and usability

Voice Wallet serves as a practical exploration of the design space for next-generation Web3 wallets.
