# 附录：WalletConnect v2 与 Web3Modal v3 区别与用法

Web3Modal v3 是基于 WalletConnect v2 协议实现的 UI 接入层，不是协议版本升级。

### 🔍 核心区别对比表：

| 项目 | WalletConnect v2 | Web3Modal v3 |
| --- | --- | --- |
| 类型 | 协议标准 | UI 工具包 |
| 发布方 | WalletConnect 团队 | WalletConnect 团队 |
| 功能定位 | 建立 DApp 与钱包之间的通信桥 | 提供钱包连接 UI 与统一入口 |
| 通信协议 | JSON-RPC over WebSocket | 基于 WalletConnect v2 |
| 会话管理 | 多链支持、长连接、中继服务器 | 封装连接器管理、状态展示、QR 窗口 |
| 插件系统 | ❌（非模块化） | ✅（支持 AuthKit、Passkey 等扩展） |
| 集成方式 | 需手动构建连接器（如 wagmi） | 提供组件 + 自动注入连接器 |

---

### 🧱 典型使用方式

### ✅ WalletConnect v2（用于 wagmi 配置）

```tsx
import { walletConnect } from 'wagmi/connectors'

walletConnect({
  projectId: 'YOUR_PROJECT_ID',
  metadata: { name: 'MyDapp', ... },
  showQrModal: false, // 自定义 QR 由 Web3Modal 控制
})
```

### ✅ Web3Modal v3（快速 UI 集成）

```tsx
import { EthereumClient, w3mConnectors, w3mProvider } from '@web3modal/ethereum'
import { Web3Modal } from '@web3modal/react'

<Web3Modal
  projectId="YOUR_PROJECT_ID"
  ethereumClient={ethereumClient}
/>
```

> Web3Modal 提供了完整 UI、状态管理与连接器封装，适合快速集成。
> 

---

### ✅ 适用建议

| 使用场景 | 推荐方式 |
| --- | --- |
| 需要自定义连接按钮、状态管理 | wagmi + WalletConnect v2 |
| 追求快速集成、官方 UI 体验 | Web3Modal v3 |
| 多连接器统一管理（如 Injected + WalletConnect） | Web3Modal v3（内建组合器） |
| 想接入邮箱 / Passkey 登录（AuthKit） | Web3Modal v3 + AuthKit |

---

### 📘 参考资料

- WalletConnect 文档
- Web3Modal v3 文档
- wagmi 官方连接器文档