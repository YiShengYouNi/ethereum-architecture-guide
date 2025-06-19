# 区块链钱包基础概念

区块链钱包是用户进入 Web3 世界的第一站。要理解钱包的功能，首先要**抛开**“钱包=资产”的传统观念，真正认识它在去中心化世界中的角色：**私钥管理器 + 用户身份认证器**。

## 钱包的本质：不是“装币”的，而是“控权”的

> 区块链钱包 ≠ 存放资产
>
>
> 区块链钱包 = 控制私钥 + 发起签名
>

区块链资产（如 ETH、ERC-20）是“记录在链上的数据”，不在你的钱包里，而是在一个地址上。

**钱包的功能核心在于：**

- 保存并使用私钥
- 发起链上签名操作（如转账、授权、登录）
- 允许你证明“我是这个地址的控制者”

![钱包职责](../../assets/02/01_wallet_responsibility.png)

> 🔍 私钥生成地址，地址持有资产，钱包控制私钥，因此钱包可控制资产。

## 账户类型：EOA 与 合约钱包的本质区别

### ✅ EOA（Externally Owned Account）

- 有私钥控制
- 默认类型，由钱包（如 MetaMask） 创建
- 可直接签名与发起交易

### ✅ 合约钱包（Smart Contract Account）

- 无私钥，由合约逻辑控制
- 支持多签、限额等定制行为
- 签名流程复杂（需 EIP-1271 验证）

| 项目 | EOA | 合约钱包 |
| --- | --- | --- |
| 控制方式 | 私钥 | 智能合约逻辑 |
| 是否支持签名 | ✅ | 需合约实现 EIP-1271 |
| 是否可被升级 | ❌ | ✅（可 Proxy） |
| Gas 支付方式 | 自付 | 可由他人代付（Paymaster） |

![账户类型](../../assets/02/01_eoa_vs_contract_wallet.png)

## 钱包分类与生态

### ✅ 热钱包（Hot Wallet）

- 在线使用，安装即用
- 易于操作但风险较高（如浏览器插件）
- 典型产品：MetaMask、Coinbase Wallet

### ✅ 冷钱包（Cold Wallet）

- 私钥离线保存，不易被攻击
- 需硬件支持，如 Ledger、Trezor
- 适合大额资产长期保存

### ✅ 托管钱包（Custodial）

- 由平台托管私钥，用户只需账号密码
- 用户无需记住助记词
- 适合新手，安全依赖平台

### ✅ 非托管钱包（Non-custodial）

- 用户自持私钥与助记词
- 完全去中心化，丢失私钥即资产无法找回

| 类别 | 是否联网 | 是否持私钥 | 推荐场景 |
| --- | --- | --- | --- |
| 热钱包 | 是 | ✅ | 高频交易、日常交互，风险高 |
| 冷钱包 | 否 | ✅ | 长期存储、高安全性 |
| 托管钱包 | 是 | ❌ | 新用户、小额场景 |
| 非托管钱包 | 是/否 | ✅ | Web3 爱好者、链上开发者 |

## 主流钱包产品速览

| 钱包 | 类型 | 适用链 | 特点 |
| --- | --- | --- | --- |
| MetaMask | 热钱包（浏览器插件） | EVM | 最广泛使用，支持 Injected Provider |
| Rainbow | 热钱包（移动端） | EVM | UI 优雅，支持 WalletConnect |
| Safe | 合约钱包 | EVM | 多签控制，适合 DAO / 机构 |
| Phantom | 热钱包 | Solana | 支持 NFT 展示与 Swap |
| Coinbase Wallet | 热钱包 | EVM / L2 | 支持 ENS、身份系统 |

## 安全机制解析：助记词、签名、授权与撤销

### ✅ 助记词（Mnemonic）

- 一组 12/24 个单词，**用于生成私钥**
- 推荐使用硬件钱包或安全插件保存
- 不应上传至任何云服务或公开网络

### ✅ 签名（Signature）

- 用户对**消息或交易**进行私钥签名
- 签名后数据无法被篡改，可验证身份

### ✅ 授权（Approve）机制

- ERC20 Token 使用 `approve` + `transferFrom` 实现转账
- 授权容易被滥用，务必限制数量与对象

### ✅ 撤销授权

- 可使用 [Revoke.cash](https://revoke.cash/) 或 Etherscan 的 Token Approvals 工具
- 定期清理授权是良好习惯

## 示例代码（Ethers.js v6 版本）

```tsx
import { Wallet } from 'ethers';

const wallet = Wallet.createRandom();

console.log('地址:', wallet.address);
console.log('助记词:', wallet.mnemonic?.phrase);  // 注意：v6 中为可选属性
console.log('私钥:', wallet.privateKey);

```

### 📌 说明

- `Wallet.createRandom()` 用于生成新钱包
- `mnemonic` 现在是 `Mnemonic | undefined`，建议使用可选链操作符访问
- 默认导入方式已基于 ESM 结构，适配现代构建工具（如 Vite、Next.js）
