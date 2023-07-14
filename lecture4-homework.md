# 第4课 课后作业

## 作业题目
Check out https://semaphore.appliedzkp.org/ and set up a starter project!

## 我的作业
### 什么是 Semaphore ？
Semaphore 是一种零知识协议，允许使用者在不暴露身份的情况下使用一个信号作为可证明的群组成员。此外，Semaphore 提供一种简单的机制防止重放信号。

（当前版本：v3）

### Semaphore 的特性
1. 可以为用户创建 Semaphore 身份。
   
1. 可以将用户的 Semaphore 身份添加到一个群组内（以 Merkle Tree 结构）。
   
1. 可以发送可验证且匿名的信号（例如投票和许可）。

用户将信号广播出来，Semaphore 零知识证明可以确保用户已经加入一个群组且尚未使用 nullifier 发出信号。


Semaphore 使用协同工作的链上 Solidity 合约和链下 JavaScript 库：

- 在链下，JavaScript 库可用于创建身份、管理组和生成证明。
- 在链上，Solidity 合约可用于管理群组和验证证明。

### Semaphore 用途
作为 dapp，开发者通过 Semaphore ，定制链上和链下组件，以实现所需的功能。

### Semaphore 安装（Quick Setup）
CLI 键入`npx @semaphore-protocol/cli@latest`

### 创建一个 Semaphore 工程目录
CLI 键入`semaphore create my-app --template monorepo-ethers`

（其中 `--template` 后面接 3 种模板之一：contracts-hardhat / monorepo-ethers / monorepo-subgraph）

安装依赖项：
```
cd my-app 
npm i
```

### 合约编译
`cd apps/contracts`

`npm run compile`

### 合约测试
运行my-app中的测试用例：

### 部署合约
