# Voice Wallet — 基于自然语言的 Web3 钱包

[English Version](./README.md)

---

## 项目资源与演示

### 项目仓库

- GitHub Repository  
  https://github.com/mxdu-tech/web3-portfolio/tree/main/voice-wallet

---

### 架构与设计

### Architecture & Knowledge Map

![Architecture](./images/architecture.png)

该图展示了系统的整体分层结构，以及各模块之间的关系。

---

### 演示视频

https://youtu.be/CS7R_RBFGTI

[![Voice Wallet Demo](https://img.youtube.com/vi/CS7R_RBFGTI/hqdefault.jpg)](https://youtu.be/CS7R_RBFGTI)

---

## 1. 项目简介

Voice Wallet 是一个浏览器端的自托管钱包（Self-Custodial Wallet），其核心目标是通过自然语言接口，将链上交易的构造与执行过程进行抽象。

在传统钱包中，用户需要直接操作底层交易参数（地址、金额、Gas 等），而本系统将这一过程转化为：

用户意图 → 结构化数据 → 标准交易 → 本地签名 → 链上广播

本项目关注的不仅是功能实现，更重要的是：

- 如何定义钱包系统的安全边界
- 如何在保证安全性的前提下降低用户认知成本
- 如何将协议复杂性从用户侧转移到系统侧

---

## 2. 问题背景与设计目标

现有 Web3 钱包普遍存在以下问题：

- 交互模型直接暴露底层协议结构
- 操作流程依赖多步骤人工处理
- 用户输入错误缺乏有效保护机制
- 学习成本较高

本项目的设计目标不是简单优化 UI，而是：

> 将“交易构造”本身视为一个可以被系统化处理的问题

在此基础上，系统需要同时满足：

- 用户资产完全自托管
- 私钥不离开本地执行环境
- 所有关键操作具备明确确认边界

---

## 3. 系统架构

![Architecture](./images/architecture.png)

系统采用分层架构，每一层承担明确职责：

- UI Layer：用户输入与确认
- Intent Processing Layer：自然语言解析与结构化转换
- Wallet Core Layer：私钥与账户管理
- Transaction Execution Layer：交易构造与签名
- Blockchain Interaction Layer：链上通信
- Extension Runtime Layer：运行环境与隔离

该结构的核心原则是：

> 将安全敏感模块与用户交互模块进行解耦，并明确数据流方向

---

## 4. 核心模块设计

### 4.1 Wallet Core（安全边界定义）

Wallet Core 是系统中最关键的模块，其职责不仅是管理密钥，更是定义信任边界（Trust Boundary）。

---

#### Key Management（私钥管理）

- 私钥由助记词（BIP39）派生生成
- 解密后仅存在于内存中
- 不进行明文持久化存储

这种设计本质上是在控制两个维度：

- **存储面（Storage Surface）**
- **暴露时间（Exposure Time）**

通过将私钥限制在内存中，可以显著降低持久化泄露风险。

---

#### Vault Encryption（加密机制）

系统采用浏览器原生加密能力：

- Web Crypto API
- PBKDF2（密码派生）
- AES-GCM（认证加密）

其中：

- PBKDF2 提高弱密码的抗破解能力
- AES-GCM 提供数据完整性校验

相比自定义加密实现，浏览器原生方案具有更低的实现风险和更高的兼容性。

---

#### Account Derivation（账户派生）

- 使用 HD Wallet 模型
- 标准路径：m/44'/60'/0'/0/0

该设计确保：

- 地址生成具备确定性
- 可与主流钱包（如 MetaMask）互操作

---

### 4.2 Transaction Execution（交易执行模型）

系统将交易执行抽象为一个明确的状态推进过程：

Created → Validated → Signed → Broadcast → Confirmed / Failed

---

#### Transaction Builder

使用：

- ethers.js / viem

负责：

- 构造标准交易对象
- 编码合约调用
- 适配 RPC 接口

---

#### Validation（前置校验）

在进入签名阶段之前执行：

- 地址合法性校验
- 余额检查
- 网络一致性验证

该阶段的作用在于：

> 在“不可逆操作（签名）”之前尽可能发现错误

---

#### Signing（本地签名）

- 使用内存中的私钥
- 在本地执行签名逻辑

该过程不涉及外部系统，从而确保私钥不离开信任边界。

---

#### Broadcast（广播）

- 通过 RPC Provider 调用 eth_sendRawTransaction

系统并不直接连接区块链网络，而是通过 RPC 节点完成交互，这也是当前钱包的标准模式。

---

### 4.3 Blockchain Interaction（链交互抽象）

该层封装所有链上通信：

- RPC 请求管理
- 状态查询（余额、交易）
- 广播处理

这一抽象使得：

链交互逻辑可以独立演进，而不影响上层模块。

---

### 4.4 Extension Runtime（运行环境）

系统运行于浏览器扩展环境，其特点是：

- 具备本地执行能力
- 与网页上下文隔离
- 支持持久化后台逻辑

---

#### Background Script

- 常驻执行
- 承载 Wallet Core 与 Transaction Logic

相比 UI 层，其生命周期更稳定，适合处理安全敏感逻辑。

---

#### Message Passing

UI 与后台通过：

chrome.runtime.sendMessage 进行通信

这种机制保证：

- UI 与核心逻辑解耦
- 所有敏感操作集中在受控环境中执行

---

#### Chrome Storage

- 使用 chrome.storage.local
- 仅存储加密后的数据

避免任何明文敏感信息落盘。

---

## 5. 交易流程

完整流程如下：

1. 用户输入自然语言
2. 系统解析用户意图
3. 提取交易参数
4. 参数校验
5. 构造交易
6. 用户确认
7. 本地签名
8. 广播交易
9. 返回交易结果

该流程中存在两个关键边界：

- **确认边界（User Confirmation）**
- **签名边界（Signing Boundary）**

所有不可逆操作均发生在用户确认之后。

---

## 6. 技术选型说明

### WDK（Wallet Development Kit）

提供钱包抽象能力：

- 钱包创建
- 账户管理
- 签名能力

选择 WDK 的原因在于：

底层钱包逻辑复杂且安全敏感，复用成熟组件可以降低实现风险。

---

### ethers / viem

用于 EVM 交互：

- 构造交易
- 调用合约
- 与 RPC 节点通信

相比直接使用 JSON-RPC：

SDK 提供了更高层抽象与类型安全。

---

### Web Crypto API

浏览器原生加密模块：

- 助记词加密
- 密钥派生

避免引入额外加密依赖。

---

### Chrome Extension API

提供：

- 本地存储
- 消息通信
- 后台执行环境

使钱包具备类似原生应用的运行模型。

---

## 7. 设计决策与权衡

### 自然语言接口

该设计将交易构造抽象为意图识别问题。

优点：

- 降低用户理解成本

挑战：

- 需要处理语义歧义
- 需要严格参数校验

---

### 自托管模型

优点：

- 用户完全控制资产
- 无需信任第三方

挑战：

- 密钥管理责任转移到用户

---

### 内存级私钥管理

优点：

- 减少长期暴露风险

挑战：

- 需要频繁解锁

---

### 扩展 vs 网页应用

选择浏览器扩展而非普通网页应用，主要原因在于：

- 扩展具备独立执行环境
- 可以隔离敏感逻辑
- 支持后台常驻

---

## 8. 安全模型补充

系统的安全模型基于以下假设：

- 执行环境为可信浏览器扩展
- 私钥仅存在于本地内存
- 所有签名操作需用户确认

潜在风险包括：

- 恶意网页诱导签名
- 扩展被篡改
- 用户输入错误

对应的控制措施：

- 明确的确认流程
- 参数校验机制
- 本地签名隔离

---

## 9. 后续优化方向

- 多链支持
- 更高精度的意图解析（LLM 优化）
- 交易模拟（Simulation）
- 状态管理优化（nonce / pending 管理）

---

## 10. 总结

该项目围绕一个核心问题展开：

> 在保持安全边界的前提下，是否可以重新定义 Web3 钱包的交互方式

Voice Wallet 作为一个实验性实现，展示了一种将自然语言引入钱包交互的可行路径，同时也体现了在安全性、可用性与系统复杂度之间的权衡。
