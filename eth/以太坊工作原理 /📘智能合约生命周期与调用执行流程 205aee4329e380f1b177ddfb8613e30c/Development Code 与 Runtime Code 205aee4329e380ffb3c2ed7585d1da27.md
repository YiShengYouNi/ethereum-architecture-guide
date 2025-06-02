# Development Code 与 Runtime Code

在合约部署时，提到了 Development Bytecode 和  Runtime Bytecode，有点懵。关于这个，在展开了解下。

## 🧱 背景回顾：以太坊合约是如何部署的？

在以太坊中，部署合约不是简单地“把代码复制粘贴到链上”，而是：

1. 你发送一笔 **to 为 null 的交易**
2. 在这个交易的 `data` 字段里，放的是一段 **部署 Bytecode**
3. **EVM 执行部署 Bytecode**，这段代码的执行“结果”会变成你合约的实际代码（Runtime Code），存储在你新合约地址的 `code` 区域中

## ✅ 一、什么是部署 Bytecode？什么是 Runtime Bytecode？

用 Solidity 写一个合约，比如：

```solidity
contract MyContract {
    uint public x;

    function set(uint _x) public {
        x = _x;
    }
}
```

使用 `solc` 编译器编译时，会产生两个重要的字节码（Bytecode）：

| 类型 | 作用 | 被谁执行 | 存在哪里 |
| --- | --- | --- | --- |
| **Deployment Bytecode**（部署代码） | 构造合约部署逻辑，最终返回 Runtime Code | **部署时被 EVM 执行一次** | 不存储在链上 |
| **Runtime Bytecode**（运行代码） | 合约真正的执行逻辑（函数、变量等） | **交易执行时被 EVM 多次执行** | 存储在合约账户中 |

---

## ✅ 二、部署 Bytecode 是如何“写入” Runtime Bytecode 的？

你发起一笔部署合约的交易（`to: null`），此交易携带的是 **Deployment Bytecode**。

**Deployment Bytecode 的作用是：**

> 在部署阶段由 EVM 执行，它的“最终返回值”会被作为合约的 Runtime Code，写入新生成的合约账户。
> 

### 🔁 类比理解：

你可以把部署 Bytecode 看成是「启动器」，它的最后一行代码类似：

```
return(pointer_to_runtime_code, size)
```

这段代码会把内存中已经准备好的 Runtime Code 片段“返回”给 EVM，EVM 就会把它写入到合约账户中，成为该合约真正的逻辑代码。

---

## ✅ 三、用 Solidity 举个例子

你写：

```solidity
contract A {
    function hello() public pure returns (string memory) {
        return "world";
    }
}
```

编译器会生成 Deployment Bytecode，大概结构如下（伪逻辑）：

```
// Constructor logic
STORE runtime_code at memory[0x00]
RETURN memory[0x00:len(runtime_code)]
```

而实际被写入合约账户的，是这段 Runtime Code：

```
// 实际执行 hello() 的逻辑
PUSH1 0x60
PUSH1 0x40
MSTORE
...
```

你调用合约的时候，执行的是 **这段被返回的 Runtime Bytecode**，而不是那段部署代码本身。

![ChatGPT Image 2025年6月1日 17_21_35.png](Development%20Code%20%E4%B8%8E%20Runtime%20Code%20205aee4329e380ffb3c2ed7585d1da27/ChatGPT_Image_2025%E5%B9%B46%E6%9C%881%E6%97%A5_17_21_35.png)

## ✅ 四、总结

> 合约部署时，交易中携带的字节码不是直接运行的代码，而是一段创建逻辑（Deployment Bytecode），它在 EVM 中运行后，会返回真正的 Runtime Bytecode。这段返回值会被存入新创建的合约账户，用于后续每次调用时执行。
>