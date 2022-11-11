# 代码分析

本文件主要分析该项目代码的主体结构，目标是帮助该项目的初学者能够逐步深入该项目的各项细节。对于单个文件的更加详细的分析将会放在对应文件的开头位置的注释中，对于单个函数的较为详细的解释会在函数实现前给出。

建议您先阅读项目白皮书和官网文档，这将有助于您理解项目的代码实现。[该博客](https://liaoph.com
)上的 [Uniswap V3 系列分析文章](https://liaoph.com/uniswap-v3-1/)也将有助于您理解该项目。本项目代码中的添加的大部分中文注释来自于该系列文章，在此感谢。

## Overview
> 以下是核心(core)合约和外围(periphery)合约的总体概览。

Uniswap V3 由核心合约和外围合约共同组成。核心合约为与 Uniswap 交互的各方提供基本安全保障。它们定义了流动性池生成、池本身以及涉及其中的各个资产的交互。

外围合约与一个或多个核心合约交互，但并不是核心的一部分。它们旨在提供与核心交互的方法，以提高清晰度和用户安全性。

外部调用将主要调用外围接口。外部可访问函数都可以在参考文档中查看。内部函数可在 Uniswap V3 Github 仓库中查看。

### Core
Core 由一个工厂(factory)，一个池部署器(pool deployer)和工厂将会创建的许多池组成。

核心合约中的 gas 优化已经得到了极大的关注。结果是与 V2 相比，所有协议交互的 gas 成本大幅降低，但代价是代码清晰度降低。

#### Factory
> [Factory Reference](https://docs.uniswap.org/protocol/reference/core/UniswapV3Factory)

工厂定义了生成池的逻辑。池由构成资产对的两个代币和费用定义。同一资产对可以有多个池，仅通过交换费用(swap fee)来区分。

#### Pools
> [Pool Reference](https://docs.uniswap.org/protocol/reference/core/UniswapV3Pool)

池主要作为成对资产的自动做市商。此外，它们公开价格预言机数据，并可用作闪电交易的资产来源。

### Periphery
Periphery 是一组智能合约，旨在支持与核心合约的特定领域交互。由于 Uniswap 协议是一个无需许可的系统，以下描述的合约没有特殊权限，只是可能的类外围合约的一小部分。

#### SwapRouter
> [Swap Router Reference](https://docs.uniswap.org/protocol/reference/periphery/SwapRouter) | [Swap Router Interface](https://docs.uniswap.org/protocol/reference/periphery/interfaces/ISwapRouter)

交换路由支持前端提供交易的所有基本需求。它本身支持单个交易(x -> y)和多跳交易(例如 x -> y -> z)。

#### Nonfungible Position Manager
> [Nonfungible Position Manager Reference](https://docs.uniswap.org/protocol/reference/periphery/NonfungiblePositionManager) | [Nonfungible Position Manager Interface](https://docs.uniswap.org/protocol/reference/periphery/interfaces/INonfungiblePositionManager)

头寸管理器处理涉及创建、调整或退出头寸的逻辑交易。

#### Oracle
> [Oracle Reference](https://docs.uniswap.org/protocol/reference/core/libraries/Oracle)

预言机提供对各种系统设计有用的价格和流动性数据，并且在每个部署的池中都可用。

#### Periphery Libraries
> [Periphery Libraries](https://docs.uniswap.org/protocol/reference/periphery/libraries/Base64)

这些库提供了开发人员可能需要的各种帮助函数，例如计算池地址、安全传输函数等。

## Core Contracts

### [UniswapV3Factory](./contracts/UniswapV3Factory.sol)
部署 Uniswap V3 池、管理所有权并控制池协议费用。
- `createPool`: 根据给定的两种代币和费用创建池。
- `setOwner`: 更新工厂合约的所有者，须由当前所有者调用。
- `enableFeeAmount`: 使用给定的 `tickSpacing` 开启一个费用额。一旦启用，费用额将永远不能删除。

### [UniswapV3Pool](./contracts/UniswapV3Pool.sol)
- `_blockTimestamp`: 返回截断为 32 位的区块时间戳，及 mod 2**32。该方法在测试中被覆盖。
- `snapshotCumulativesInside`: 返回一个 tick 区间内的 `tickCumulativeInside`、`secondPerLiquidityInsideX128` 和 `secondsInside` 的快照。快照只能与其它快照进行比较，这些快照是在某个头寸存在的一段时间内截取的。例如，如果在第一个快照和第二个快照之间的整个时间段内都没有持有某个头寸，则无法比较快照。
- `observe`: 从当前区块时间戳返回每个时间戳 `secondsAgo`(uint32[]) 的累积刻度和流动性。要获得时间加权平均刻度或范围内的流动性，您必须使用两个值调用它，一个代表周期的开始，另一个代表周期的结束。例如，要获得最后一小时的时间加权平均刻度，您必须使用 secondsAgos = [3600, 0] 调用它。时间加权平均刻度表示池的几何时间加权平均价格，以 token1 / token0 的 log base sqrt(1.0001) 为单位。 TickMath 库可用于将刻度值转换为比率。
- `increaseObservationCardinalityNext`: 增加该池将存储的价格和流动性观察的最大数量。如果该池已经有一个大于或等于输入的 `observationCardinalityNext` 的 `observationCardinalityNext`，此方法是无法操作的。
- `initialize`: 设置池的初始价格。
- `mint`: 为给定的 `recipient`/`tickLower`/`tickUpper` 头寸添加流动性。`noDelegateCall` 通过 `_modifyPosition` 间接应用。
- `collect`: 收集某个头寸的代币。不重新计算赚取的费用，必须通过 mint 或 burn 任意数量的流动性完成。Collect 必须由头寸所有者调用。仅提取 token0 或仅提取 token1，可以将 `amount0Requested` 或 `amount1Requested` 设置为零。要提取所有欠的代币，调用者可以传递任何大于实际欠的代币值，例如 `type(uint128).max`。所欠代币可能来自累积的交换费用或burn的流动性。
- `burn`: 燃烧 sender 的流动性和头寸流动性所欠的账户代币。`noDelegateCall` 通过 `_modifyPosition` 间接应用。
- `swap`: 用 token0 兑换 token1，或用 token1 兑换 token0。该方法的调用者接收到回调，形式为 `IUniswapV3SwapCallback#uniswapV3SwapCallback`。
- `flash`: 在回调中接收 token0 和/或 token1 并支付费用。该方法的调用者收到回调，形式为 `IUniswapV3FlashCallback#uniswapV3FlashCallback`。对应闪电贷功能。
- `setFeeProtocol`: 设置协议的费用的%份额的分母。
- `collectProtocol`: 收取池中累积的协议费用。

### [UniswapV3PoolDeployer](./contracts/UniswapV3PoolDeployer.sol)
- `deploy`: 通过临时设置参数存储槽，然后在部署池后将其清除来部署具有给定参数的池。
