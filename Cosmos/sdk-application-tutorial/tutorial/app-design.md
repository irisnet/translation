# 应用开发目标


构建该应用的目的是让用户购买名称并设置这些名称解析的值。给定名称的所有者将是当前最高出价者。在本节中，您将了解这些简单要求如何转化为应用的设计。

区块链应用实际就是[复制了一个确定性状态机](https://en.wikipedia.org/wiki/State_machine_replication)。作为开发人员，你只需要定义状态机（即在什么状态，启动状态和触发状态转换的消息），[*Tendermint*](https://tendermint.com/docs/introduction/introduction.html)将为您处理网络上的复制过程。

>Tendermint是一个与应用无关的引擎，负责处理区块链的*网络层*和*共识层*。也就是说，Tendermint负责传播和排序交易。Tendermint Core依赖于同名的拜占庭容错（BFT）算法来达成交易顺序的共识。 有关Tendermint的更多信息，请单击[此处](https://tendermint.com/docs/introduction/introduction.html)。

[Cosmos SDK](https://github.com/cosmos/cosmos-sdk/) 旨在帮助您构建状态机。SDK采用**模块化框架**，也就是说应用程序是通过聚集一组可互操作的模块构建的。 每个模块都包含自己的消息/事务处理器，而SDK负责将每个消息路由到其各自的模块。

以下是nameservice应用程序所需的模块:
- `auth`: 该模块用于定义帐户和费用，并为应用程序的其功能提供访问入口。
- `bank`: 该模块用于建立并管理token及token账户。
- `nameservice`: 该模块还在完善中。它将处理您正在构建的`nameservice`应用的核心逻辑，是您在构建应用时需要完成的主要软件。

>你可能会问为什么没有模块来处理验证人集的更改。 实际上，Tendermint依赖于一组验证人对将要添加到区块链中的下一个有效交易块[达成共识](https://tendermint.com/docs/introduction/introduction.html#consensus-overview) 。 默认情况下，如果没有模块处理验证人集的更改，验证人集将与`genesis.json`中定义的验证人集保持一致。这个应用就是这种情况。 如果要允许更改应用的验证人集，可以使用SDK的[staking 模块](https://github.com/cosmos/cosmos-sdk/tree/develop/x/stake)，或自己编写模块！

现在，看一下应用的两个主要部分：状态和消息类型。

## 状态

状态代表你在特定时刻的申请。它将表明每个帐户拥有多少通证，每个名称的所有者和价格，以及每个名称解析的值。

通证和帐户的状态由`auth`和`bank`模块定义，你现在不必关心它。你需要做的是定义与您的`nameservice`模块特定相关的状态部分。

在SDK中，所有内容都存储在为`multistore`。 可以在multistore中创建任意数量的键/值存储对（在Cosmos SDK中称为[`KVStores`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#KVStore)）。 对于您的应用，您需要存储：

- 从`name` 到 `value` 的映射。在`multistore` 中创建`nameStore`以保存该数据。
- 从`name` 到 `owner` 的映射。在`multistore` 中创建`ownerStore` 以保存该数据。
- 从`name` 到 `price`的映射。在`multistore` 中创建`priceStore` 以保存该数据。

## 消息

消息包含在交易中。它们会触发状态转换。每个模块都定义了一个消息列表以及其处理者。 以下是为nameservice应用实现其功能所需的消息：

- `MsgSetName`: 该消息允许名称所有者在`nameStore`中为特定的名称设定一个值。
- `MsgBuyName`: 该消息允许客户购买一个名称并在`ownerStore`中成为其所有者。

当交易（包含在块中）到达Tendermint节点时，它将通过[ABCI](https://github.com/tendermint/tendermint/tree/master/abci) 传递给应用程序并解码以获取消息。 然后将消息发送到适当的模块，并根据`Handler`中定义的逻辑进行处理。 如果需要更新状态，则`Handler`会调用`Keeper`来执行更新。 本教程的后续步骤可了解更多相关概念的信息。

### 现在您已经从设计层面了解了应用的整体功能框架，那么接下来就开始[实现](./app-init.md)它吧
