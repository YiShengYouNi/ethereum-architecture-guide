# 📘智能合约生命周期与调用执行流程

在以太坊上，智能合约并非“上传即运行”，而是经过部署、创建账户、执行初始化代码、调用函数等多个阶段，最终形成可被调用的链上代码单元。理解这一流程，有助于开发者编写更可靠的合约，也有助于安全分析和调试优化。

## ✦ 1. 合约部署流程概述

部署合约是一笔特殊交易，其特点是：

- `to` 字段为 `null`（表示不向已有账户转账）
- `data` 字段中包含的是**合约字节码（含 constructor 初始化逻辑）**
- EVM 会将该字节码执行一次，执行结果（return 的字节码）将作为合约的**Runtime Code** 写入新账户中

📌 **部署流程图见图 1：合约部署流程图**

```
[EOA 构造交易]
     ↓
[交易签名与广播]
     ↓
[交易进入区块 → 被打包]
     ↓
[节点执行交易（含部署）]
     ↓
[EVM 执行部署代码（Constructor）]
     ↓
[返回 Runtime Bytecode]
     ↓
[将 Runtime Bytecode 写入新地址账户的 code 字段]
     ↓
[更新 StorageRoot 与 StateRoot]

解释：
1️⃣ 构造交易：由外部账户（EOA）发起，data 字段为合约部署字节码。
2️⃣ 签名与广播：本地签名 → 节点广播 → Mempool 等待打包。
3️⃣ 区块打包：交易被矿工/验证者打包进新区块。
4️⃣ 节点执行交易：启动 EVM → 执行字节码（constructor 部分）。
5️⃣ EVM 返回 Runtime Bytecode：通过 RETURN 指令返回 runtime code。
6️⃣ 创建新账户地址 → 存储 codeHash 与 runtime bytecode。
7️⃣ 状态树更新 → 写入 storageRoot、stateRoot。
```

## ✦ 2. 部署交易结构

合约部署交易的关键字段如下：

| 字段 | 含义 |
| --- | --- |
| `nonce` | 部署者账户的交易计数 |
| `to` | `null`，表示创建新合约 |
| `value` | 部署时附带的 ETH（可选） |
| `data` | 包含 constructor 初始化代码 |
| `gasLimit` | 执行合约初始化的资源限制 |
| `v`, `r`, `s` | 签名字段，确保交易合法性 |

---

## ✦ 3. 合约地址生成机制

合约部署后会生成一个**20 字节的合约地址**，通常有两种生成方式：

### ✅ 常规部署（CREATE）

```solidity
address = keccak256(rlp(sender, nonce))[-20:]
```

- 由部署者地址 + nonce 决定
- 地址无法提前预测（部署顺序影响地址）

### 🧪 特殊部署（CREATE2）

```solidity
address = keccak256(0xFF ++ sender ++ salt ++ keccak256(init_code))[-20:]
```

- 可用于构建“预部署合约”（先知地址）
- 被广泛应用于 DeFi 工厂、Proxy 合约体系

```text
+--------------------+             +--------------------+
|      CREATE        |             |     CREATE2        |
+--------------------+             +--------------------+
| address =          |             | address =          |
| keccak256(         |             | keccak256(         |
|   rlp(sender,nonce)|             |   0xff ++          |
| )                  |             |   sender ++ salt ++|
|                    |             |   keccak256(init)  |
+--------------------+             +--------------------+
  
- CREATE 地址依赖 nonce，受账户状态影响  
- CREATE2 地址可预测（与 initcode + salt 强相关）
```

---

## ✦ 4. Runtime Code 与初始化字节码的区别

部署合约的字节码结构如下：

```kotlin
完整字节码 = 初始化代码（constructor） + runtime 代码（return 输出）
```

- 初始化代码在部署时执行一次，用于计算和返回 Runtime Code
- Runtime Code 才是部署后**写入链上**、供后续调用的合约逻辑

```text
[Deployment Bytecode]
┌──────────────────────────────┐
│ 构造函数指令（constructor） │
│ ...                          │
│ RETURN <offset, length>      │ ← 指向 runtime code
└──────────────────────────────┘

             ↓ 执行后

[Runtime Bytecode]
┌──────────────────────────────┐
│ 实际合约执行逻辑             │
│ 包括函数调度器、业务代码等   │
└──────────────────────────────┘

📌 加注说明：部署完成后只保留 runtime code，constructor 不存在链上。
```

---

## ✦ 5. 合约调用流程详解

部署成功后，调用合约函数的流程如下：

1. 用户（或另一个合约）发起交易，指定目标合约地址 `to`
2. `data` 中编码为 `functionSelector + 参数`
3. 节点执行合约字节码，EVM 加载 runtime code 运行
4. 扣除 gas，更新状态树，生成 receipt

📌 对应 EVM CALL 指令，包括 `CALL` / `DELEGATECALL` / `STATICCALL`

```text
[EOA 构造交易]
  → to = 合约地址  
  → data = functionSelector + encodedArgs  
         ↓
[EVM 执行 CALL 指令]
         ↓
[合约 Dispatcher 解析 data]
         ↓
[跳转对应函数入口]
         ↓
[执行函数逻辑]
         ↓
[返回值写入 returndata buffer]
         ↓
[结果写入 Receipt / Logs / Storage]
```

📌 标注：

- `data[0:4]` 是函数选择器
- `data[4:]` 是 ABI 编码的参数
- 调用可为 `CALL`, `DELEGATECALL`, `STATICCALL`

---

## ✦ 6. 合约调用的行为分类

| 类型 | 指令 | 说明 | 常用场景 |
| --- | --- | --- | --- |
| 普通调用 | `CALL` | 使用被调用合约的上下文，允许转账 | 函数交互、value 调用 |
| 上下文复用 | `DELEGATECALL` | 使用调用者上下文，合约逻辑执行在当前存储空间中 | Proxy 模式 |
| 只读调用 | `STATICCALL` | 不允许状态变更，只能读链上数据 | View / Read Only 场景 |

---

## ✦ 7. 调用失败的常见原因

- `Out of Gas`：调用时预留 gas 不足
- `Revert`：逻辑中手动抛出错误
- `Invalid Opcode`：字节码错误或非法跳转
- `Fallback`/`Receive` 未定义时接收 ETH

---

## 8. 合约调用返回值与错误处理

| 结果 | 状态 | Gas 扣除 | 状态是否回滚 |
| --- | --- | --- | --- |
| `return` | 成功 | 按实际执行量扣 | 保留结果 |
| `revert` | 主动失败 | 扣部分 gas | 回滚状态 |
| `invalid` | 字节码错误 | 扣所有 gas | 回滚状态 |

---

## ✅ 小结

> 智能合约从部署到调用，是一个“状态驱动”的链上过程。
>
> 部署构建账户 + 写入代码，调用执行逻辑 + 改变状态。
>

理解这些流程，是掌握以太坊运行机制、编写合约系统与识别漏洞的基础能力。

## 🔄 导航

> ⬅️ 上一篇：[以太坊存储结构与Merkle Trie](./06_storage_and_mpt.md)
>
> ➡️ 下一篇：[节点类型、客户端结构与RPC通信机制](./08_nodes_and_rpc.md)
>

📚 作者：Henry

👨‍💻 受众：Web3 开发者 / 区块链学习者
