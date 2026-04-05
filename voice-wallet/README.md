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

This diagram illustrates the system’s layered architecture and the relationships between core components.

---

### Demo Video

https://youtu.be/CS7R_RBFGTI

[![Voice Wallet Demo](https://img.youtube.com/vi/CS7R_RBFGTI/hqdefault.jpg)](https://youtu.be/CS7R_RBFGTI)

---

## 1. Overview

Voice Wallet is a browser-based self-custodial wallet designed to abstract blockchain transaction execution through a natural language interface.

Instead of requiring users to manually construct transactions (address, amount, gas), the system transforms user intent into structured transaction data, executes signing locally, and broadcasts the transaction to the network.

This project focuses on three core aspects:

- Abstracting wallet interaction away from protocol complexity  
- Systematizing transaction construction  
- Defining clear security boundaries under a self-custody model  

---

## 2. Problem & Design Goals

Most existing Web3 wallets expose protocol-level complexity directly to users:

- Manual handling of addresses, networks, and gas parameters  
- Multi-step fragmented workflows  
- High cognitive load for non-technical users  
- Irreversible consequences of input errors  

This project reframes transaction execution as a system problem rather than a user responsibility.

The design goals are:

- Move transaction construction from users to the system  
- Provide a more intuitive interaction model  
- Preserve full user control over assets (self-custody)  

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

A key design principle is:

> Security-sensitive components are isolated from user interaction layers, with explicit data flow boundaries.

---

## 4. Core Components

### 4.1 Wallet Core (Security Boundary)

The Wallet Core defines the trust boundary of the system. It is responsible for key management and account state.

---

#### Key Management

- Private keys are derived from mnemonic phrases (BIP39)  
- Decrypted keys exist only in memory  
- No plaintext persistence  

This design minimizes:

- Storage surface  
- Exposure time  

---

#### Vault Encryption

Encryption is implemented using browser-native cryptography:

- Web Crypto API  
- PBKDF2 (password-based key derivation)  
- AES-GCM (authenticated encryption)  

PBKDF2 improves resistance to brute-force attacks, while AES-GCM ensures both confidentiality and integrity.

Choosing Web Crypto API:

- Native browser support  
- Reduced dependency risk  

---

#### Account Derivation

- HD Wallet structure  
- Standard derivation path: m/44'/60'/0'/0/0  

This ensures compatibility with mainstream wallets such as MetaMask.

---

### 4.2 Transaction Execution (State Model)

Transaction execution is modeled as a state transition process:

Created → Validated → Signed → Broadcast → Confirmed / Failed

---

#### Transaction Builder

Implemented using:

- ethers.js  
- viem  

Responsibilities:

- Construct transaction payload (to, value, data, chainId)  
- Encode contract calls  
- Interface with RPC providers  

---

#### Validation

Executed before signing:

- Address format validation  
- Balance check  
- Network consistency validation  

This stage ensures that invalid transactions are filtered before irreversible operations.

---

#### Signing

- Executed locally  
- Uses in-memory private keys  

Private keys never leave the trust boundary.

---

#### Broadcast

- Uses RPC provider  
- Calls `eth_sendRawTransaction`  

The wallet communicates with the blockchain through RPC endpoints rather than direct node connections.

---

### 4.3 Blockchain Interaction

This layer abstracts all on-chain communication:

- RPC request handling  
- State queries (balance, transaction status)  
- Transaction broadcasting  

This abstraction allows independent evolution of interaction logic.

---

### 4.4 Extension Runtime

The system runs as a browser extension, providing:

- Isolated execution environment  
- Persistent background processes  
- Secure local storage  

---

#### Background Script

- Long-lived execution environment  
- Hosts wallet core and transaction logic  

Compared to UI components, it provides greater stability for sensitive operations.

---

#### Message Passing

Communication between UI and background:

- chrome.runtime.sendMessage  

This ensures separation between presentation and execution layers.

---

#### Chrome Storage

- Uses chrome.storage.local  
- Stores only encrypted vault data  

No sensitive data is stored in plaintext.

---

## 5. Transaction Flow

End-to-end process:

1. User input (natural language)  
2. Intent parsing  
3. Parameter extraction  
4. Validation  
5. Transaction construction  
6. User confirmation  
7. Local signing  
8. Broadcast  
9. Result return (transaction hash)  

Two important boundaries exist in this flow:

- User confirmation boundary  
- Signing boundary  

All irreversible actions occur after explicit user confirmation.

---

## 6. Technology Stack

### WDK (Wallet Development Kit)

Provides wallet abstraction:

- Wallet creation  
- Account management  
- Signing capabilities  

Using WDK avoids reimplementing complex and security-critical wallet primitives.

---

### ethers / viem

EVM interaction libraries:

- Transaction construction  
- Smart contract interaction  
- RPC communication  

Compared to raw JSON-RPC, these libraries provide higher-level abstractions and better developer ergonomics.

---

### Web Crypto API

Browser-native cryptography used for:

- Mnemonic encryption  
- Key derivation  

Reduces reliance on external cryptographic libraries.

---

### Chrome Extension API

Provides:

- Local storage  
- Message passing  
- Background execution  

Enables a wallet-like runtime model within the browser.

---

### Frontend

- React  
- TypeScript  
- Tailwind CSS  

---

## 7. Design Decisions & Trade-offs

### Natural Language Interface

Transforms transaction construction into an intent recognition problem.

Advantages:

- Lower cognitive load  
- More intuitive interaction  

Challenges:

- Ambiguity in user input  
- Requires strict validation  

---

### Self-Custody Model

Advantages:

- Full user control over assets  
- Trustless design  

Challenges:

- Key management responsibility shifts to users  

---

### In-Memory Key Handling

Advantages:

- Reduced long-term exposure  

Challenges:

- Requires frequent unlocking  

---

### Extension vs Web Application

The extension model is chosen because:

- Provides an isolated execution environment  
- Supports persistent background processes  
- Enables secure handling of sensitive logic  

---

## 8. Security Considerations

The system assumes:

- Trusted browser extension environment  
- Private keys exist only in memory  
- All signing operations require user confirmation  

Potential risks include:

- Malicious dApp interactions  
- Extension compromise  
- User input errors  

Mitigations include:

- Explicit confirmation flows  
- Pre-signing validation  
- Local signing isolation  

---

## 9. Future Work

- Multi-chain support  
- Improved intent parsing (LLM-based)  
- Transaction simulation  
- Advanced state handling (nonce / pending management)  

---

## 10. Conclusion

This project explores a fundamental question:

> Can Web3 wallet interaction be redefined without compromising security boundaries?

Voice Wallet demonstrates a possible direction where:

- Interaction is abstracted  
- Security remains localized  
- System design balances usability and trust  

It serves as a practical exploration of the design space for next-generation Web3 wallets.
