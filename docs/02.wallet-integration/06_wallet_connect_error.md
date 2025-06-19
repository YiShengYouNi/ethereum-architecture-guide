# 常见问题、坑点与最佳实践

实际开发过程中，我们总是会遇到各种各样的问题，遇到问题不可怕，可怕的是自己心慌。这儿先给大家列举若干高频、易忽略的常规问题，先做个积累，让自己在遇到问题时先有自查的方向和思路。

## 连接器 / wagmi 配置类问题

1. **`useAccount` 报错：未包裹 WagmiProvider**
    - 💥 表现：页面闪退 / Hook 报错
    - ✅ 解决：确保 React 组件**树**由 `<WagmiProvider>` 包裹，且传入 config 实例

    ```jsx
    // 正确用法：
    <WagmiProvider config={config}>
      <App />
    </WagmiProvider>
    
    // 否则，会报错：
    // Cannot call useAccount() outside of WagmiProvider
    ```

2. **AutoConnect 无效：刷新后状态丢失**
    - 💥 表现：刷新页面需重新连接
    - ✅ 原因：未使用持久化存储或上次连接器信息未保存

    ```jsx
    // 第一步：在wagmi config 中开启自动连接
    export const config = createConfig({
      autoConnect: true, // ✅ 启用自动连接
      connectors: [...],
      publicClient,
    })
    // 第二步：配合localstorage存储连接器信息
    localStorage.setItem('last-connector', connector.id)
    ```

3. **WalletConnect QR 二维码加载失败**
    - 💥 表现：未弹出二维码窗口
    - ✅ 解决：检查是否传入 `projectId`，`showQrModal` 设置为 `true`，检查网络代理阻拦

    ```jsx
    walletConnect({
      projectId: 'YOUR_PROJECT_ID', // 必填
      showQrModal: true,            // 必须为 true 否则不会弹窗
    })
    ```

4. **多个连接器优先级冲突**
    - 💥 表现：Injected Wallet 总是覆盖 WalletConnect
    - ✅ 解决：在 UI 层暴露连接器选择器，或记录上次连接器 (`localStorage.setItem('lastConnector')`)

    ```jsx
    // 在UI层暴露控制连接器
    const walletConnectConnector = connectors.find(c => c.id === 'walletConnect')
    connect({ connector: walletConnectConnector }) // 显式指定
    ```

5. **无法识别当前连接器名称**
    - 💥 表现：UI 显示“Unknown wallet”
    - ✅ 使用 `useAccount().connector?.name` 替代硬编码检测

    ```jsx
    const { connector } = useAccount()
    console.log(connector?.name) // 如 "MetaMask"、"WalletConnect"
    ```

---

## 🧩 链 ID / 网络环境问题

1. **链不匹配，导致 provider 调用失败**
    - 💥 报错：Unsupported chain ID
    - ✅ 使用 `useNetwork().chain.unsupported` 判断，并使用 `useSwitchChain` 自动切换
2. **部分钱包不支持自动切链（如 Rainbow）**
    - 💥 表现：`wallet_switchEthereumChain` 无反应
    - ✅ 提供链 ID + 手动操作提示：显示 chainId 与链名
3. **钱包连接成功但地址为 null**
    - 💥 原因：WalletConnect 未同步返回账户信息
    - ✅ 监听 `onConnect` 与 `accountsChanged`，或延迟渲染业务组件直到 `address` 可用
4. **连接钱包后网络变更未触发更新**
    - 💥 原因：未监听 `chainChanged` 事件
    - ✅ 使用 `wagmi` 自动更新链状态；如自定义 provider，需手动注册监听器

---

## 🧩 签名与合约钱包兼容问题

1. **合约钱包签名失败（Safe / Argent）**
    - 💥 报错：方法不支持 / 返回空签名
    - ✅ 使用 `EIP-1271` 验证合约签名
2. **EIP-712 签名在移动钱包中失败**
    - 💥 报错：无响应或内容被修改
    - ✅ 检查 domain 中 `chainId`、`name` 与类型结构一致性，推荐使用 `siwe` 工具包处理复杂结构
3. **签名返回地址不一致**
    - 💥 表现：验证失败，登录不生效
    - ✅ 解决：确认签名地址与 `signer.getAddress()` 一致，避免多账户状态混乱
4. **用户拒绝签名 / 连接无提示**
    - 💥 表现：UI 无响应
    - ✅ 使用 try-catch 包裹签名过程，提示“签名已取消 / 连接被拒绝”

---

### 🧩 WalletConnect / Session 管理问题

1. **WalletConnect 会话未持久化**
    - 💥 表现：刷新后连接断开，但 UI 显示已连接
    - ✅ 强制 session 检查：通过 `connector?.storage?.getItem()` 验证
2. **WalletConnect 重连触发重复连接请求**
    - 💥 原因：未正确处理 `wagmi autoConnect` 与 QR 会话共存逻辑
    - ✅ 方案：记录 `lastConnector`，页面刷新时仅尝试 reconnect（不重复触发 connect）
3. **WalletConnect 无法发送交易**
    - 💥 报错：`eth_sendTransaction` 拒绝
    - ✅ 排查是否缺少 gasLimit / from 字段，或是否连接的是只读 provider
4. **不同钱包返回签名结构不一致**
    - 💥 表现：同一签名结构，Wallet A 成功，Wallet B 验证失败
    - ✅ 使用 Ethers.js 的 `verifyMessage` / `TypedDataEncoder` 统一格式化验证

---

### 🧩 UI 交互与用户体验问题

1. **签名过程中用户误点其他按钮**
    - 💥 表现：用户重复点击导致多次签名请求
    - ✅ 禁用签名按钮 + 显示 `SignaturePrompt`
2. **连接中状态丢失（例如点击按钮后无动画）**
    - 💥 原因：未设置连接状态 loading 标志
    - ✅ `useConnect().isLoading` 用于控制按钮状态
3. **钱包未安装导致页面白屏 / 卡顿**
    - 💥 原因：直接使用 `window.ethereum` 报错
    - ✅ 判断是否存在注入对象：`if (!window.ethereum) { showInstallPrompt() }`
4. **未处理 `accountsChanged` 事件**
    - 💥 用户切换钱包后，仍显示旧地址
    - ✅ wagmi 自动处理该事件，手动 provider 需监听：

        ```tsx
        window.ethereum.on('accountsChanged', handleChange)
        ```

5. **签名过期/重放攻击未处理**
    - 💥 登录状态可被伪造
    - ✅ 使用 nonce + 服务端验证签名是否有效，并设定过期时间
