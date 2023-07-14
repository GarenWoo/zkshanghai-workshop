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

```shell
garenwoo@GuanyudeMBP my-app % cd apps/contracts
garenwoo@GuanyudeMBP contracts % npm run compile

> monorepo-ethers-contracts@1.0.0 compile
> hardhat compile

Downloading compiler 0.8.4
Generating typings for: 12 artifacts in dir: ./build/typechain for target: ethers-v5
Successfully generated 40 typings!
Compiled 14 Solidity files successfully
```

### 合约测试
运行my-app中的测试用例：

```shell
garenwoo@GuanyudeMBP contracts % npm test

> monorepo-ethers-contracts@1.0.0 test
> hardhat run scripts/download-snark-artifacts.ts && hardhat test



  Feedback
    # joinGroup
      ✔ Should allow users to join the group (382ms)
    # sendFeedback
      ✔ Should allow users to send feedback anonymously (1111ms)


  2 passing (3s)
```

### 生成测试代码覆盖率（Code Coverage）
```shell
garenwoo@GuanyudeMacBook-Pro contracts % npm run test:coverage        

> monorepo-ethers-contracts@1.0.0 test:coverage
> hardhat coverage

Version
=======
> solidity-coverage: v0.7.22

Instrumenting for coverage...
=============================

> Feedback.sol

Compilation:
============

Generating typings for: 12 artifacts in dir: ./build/typechain for target: ethers-v5
Successfully generated 40 typings!
Compiled 14 Solidity files successfully

Network Info
============
> HardhatEVM: v2.14.0
> network:    hardhat



  Feedback
    # joinGroup
      √ Should allow users to join the group (1298ms)
    # sendFeedback
      √ Should allow users to send feedback anonymously (3399ms)


  2 passing (4s)

---------------|----------|----------|----------|----------|----------------|
File           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
---------------|----------|----------|----------|----------|----------------|
 contracts\    |      100 |      100 |      100 |      100 |                |
  Feedback.sol |      100 |      100 |      100 |      100 |                |
---------------|----------|----------|----------|----------|----------------|
All files      |      100 |      100 |      100 |      100 |                |
---------------|----------|----------|----------|----------|----------------|

> Istanbul reports written to ./coverage/ and ./coverage.json
```


### gas reporter
```
> monorepo-ethers-contracts@1.0.0 test:report-gas
> REPORT_GAS=true hardhat test

Compiled 14 Solidity files successfully

  Feedback
    # joinGroup
      ✓ Should allow users to join the group
    # sendFeedback
      ✓ Should allow users to send feedback anonymously

·-----------------------------|----------------------------|-------------|-----------------------------·
|     Solc version: 0.8.4     ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
······························|····························|·············|······························
|  Methods                                                                                             │
·············|················|·············|··············|·············|···············|··············
|  Contract  ·  Method        ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·············|················|·············|··············|·············|···············|··············
|  Feedback  ·  joinGroup     ·     896154  ·     1656004  ·    1276079  ·            4  ·          -  │
·············|················|·············|··············|·············|···············|··············
|  Feedback  ·  sendFeedback  ·          -  ·           -  ·     396055  ·            2  ·          -  │
·············|················|·············|··············|·············|···············|··············
|  Deployments                ·                                          ·  % of limit   ·             │
······························|·············|··············|·············|···············|··············
|  Feedback                   ·          -  ·           -  ·    1514107  ·          5 %  ·          -  │
······························|·············|··············|·············|···············|··············
|  IncrementalBinaryTree      ·          -  ·           -  ·    1667044  ·        5.6 %  ·          -  │
······························|·············|··············|·············|···············|··············
|  Pairing                    ·          -  ·           -  ·    1204971  ·          4 %  ·          -  │
······························|·············|··············|·············|···············|··············
|  Semaphore                  ·          -  ·           -  ·    1846388  ·        6.2 %  ·          -  │
······························|·············|··············|·············|···············|··············
|  SemaphoreVerifier          ·          -  ·           -  ·    7357877  ·       24.5 %  ·          -  │
·-----------------------------|-------------|--------------|-------------|---------------|-------------·

  2 passing (4s)
```
