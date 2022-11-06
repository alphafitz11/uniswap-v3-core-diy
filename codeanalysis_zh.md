# 代码分析

本文件主要分析该项目代码的主体结构，目标是帮助该项目的初学者能够逐步深入该项目的各项细节。对于单个文件的更加详细的分析将会放在对应文件的开头位置的注释中，对于单个函数的较为详细的解释会在函数实现前给出。

建议您先阅读项目白皮书和官网文档，这将有助于您理解项目的代码实现。[该博客](https://liaoph.com
)上的 [Uniswap V3 系列分析文章](https://liaoph.com/uniswap-v3-1/)也将有助于您理解该项目。

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



### [UniswapV3Pool](./contracts/UniswapV3Pool.sol)



### [UniswapV3PoolDeployer](./contracts/UniswapV3PoolDeployer.sol)



### [interfaces](./contracts/interfaces/)
#### [IERC20Minimal](./contracts/interfaces/IERC20Minimal.sol)



#### [IUniswapV3Factory](./contracts/interfaces/IUniswapV3Factory.sol)


#### [IUniswapV3Pool](./contracts/interfaces/IUniswapV3Pool.sol)


#### [IUniswapV3PoolDeveloper](./contracts/interfaces/IUniswapV3PoolDeployer.sol)


#### [callback](./contracts/interfaces/callback/)
##### [IUniswapV3FlashCallback](./contracts/interfaces/callback/IUniswapV3FlashCallback.sol)


##### [IUniswapV3MintCallback](./contracts/interfaces/callback/IUniswapV3MintCallback.sol)


##### [IUniswapV3SwapCallback](./contracts/interfaces/callback/IUniswapV3SwapCallback.sol)


#### [pool](./contracts/interfaces/pool/)
##### [IUniswapV3PoolActions](./contracts/interfaces/pool/IUniswapV3PoolActions.sol)


##### [IUniswapV3PoolDerivedState](./contracts/interfaces/pool/IUniswapV3PoolDerivedState.sol)


##### [IUniswapV3PoolEvents](./contracts/interfaces/pool/IUniswapV3PoolEvents.sol)


##### [IUniswapV3PoolImmutables](./contracts/interfaces/pool/IUniswapV3PoolImmutables.sol)


##### [IUniswapV3PoolOwnerActions](./contracts/interfaces/pool/IUniswapV3PoolOwnerActions.sol)


##### [IUniswapV3PoolState](./contracts/interfaces/pool/IUniswapV3PoolState.sol)


### [libraries](./contracts/libraries/)
#### [BitMath](./contracts/libraries/BitMath.sol)


#### [FixedPoint128](./contracts/libraries/FixedPoint128.sol)


#### [FixedPoint96](./contracts/libraries/FixedPoint96.sol)


#### [FullMath](./contracts/libraries/FullMath.sol)


#### [LiquidityMath](./contracts/libraries/LiquidityMath.sol)


#### [LowGasSafeMath](./contracts/libraries/LowGasSafeMath.sol)


#### [Oracle](./contracts/libraries/Oracle.sol)


#### [Position](./contracts/libraries/Position.sol)


#### [SafeCast](./contracts/libraries/SafeCast.sol)


#### [SqrtPriceMath](./contracts/libraries/SqrtPriceMath.sol)



#### [SwapMath](./contracts/libraries/SwapMath.sol)


#### [Tick](./contracts/libraries/Tick.sol)


#### [TickBitmap](./contracts/libraries/TickBitmap.sol)


#### [TickMath](./contracts/libraries/TickMath.sol)


#### [TransferHelper](./contracts/libraries/TransferHelper.sol)


#### [UnsafeMath](./contracts/libraries/UnsafeMath.sol)

