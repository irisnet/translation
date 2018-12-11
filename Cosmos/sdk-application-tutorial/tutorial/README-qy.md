# SDK应用开发教程

此教程通过建立一个用[Cosmos SDK](https://github.com/cosmos/cosmos-sdk/)开发的功能应用。通过完成这个开发过程，开发者将初步了解Cosmos SDK的基本原理和结构。同时，开发者将领略到在Cosmos SDK上**开发区块链应用是多么的快捷、简单**。 

当完成这个教程时，开发者将完成`nameservice`应用的开发，nameservice的主要功能是将一些字符串映射到另外的一些字符串上。这个功能与[Namecoin](https://namecoin.org/), [ENS](https://ens.domains/), 或者[Handshake](https://handshake.org/)非常相似，它们都是基于传统DNS系统的模型。在这些应用上，用户可以自由买卖、交易域名。 

此教程所有的的源代码都已经保存在此路径下（且已经编译好），但建议开发者最好能尝试自己动手完成此项目的开发。 

## 需求

- [`golang` >1.11](https://golang.org/doc/install) installed
- 一个工作的 [`$GOPATH`](https://github.com/golang/go/wiki/SettingGOPATH)目录
- 想要尝试建立自己的区块链应用！

## 教程

完整的应用将由下面这些文件组成，我们将在完成此教程的过程中逐个建立


```bash
./nameservice
├── Gopkg.toml
├── Makefile
├── app.go
├── cmd
│   ├── nscli
│   │   └── main.go
│   └── nsd
│       └── main.go
└── x
    └── nameservice
        ├── client
        │   └── cli
        │       ├── query.go
        │       └── tx.go
        ├── codec.go
        ├── handler.go
        ├── keeper.go
        ├── msgs.go
        └── querier.go
```

让我们从建立一个新的git仓库开始:

```bash
mkdir -p $GOPATH/src/github.com/{ .Username }/nameservice
cd $GOPATH/src/github.com/{ .Username }/nameservice
git init
```

接下来让我们逐一完成下面的开发步骤！下面的第一步描述了我们如何设计`nameservice`这一应用,如果您希望直接开始代码开发，则可以从[第二步](./keeper.md)开始.


### 实现步骤

1. 应用[设计](./app-design.md)。
2. 从 [`./app.go`](./app-init.md)开始实现应用.
3. 开始用[`Keeper`](./keeper.md)开发模型.
4. 用[`Msgs` 和 `Handlers`](./msgs-handlers.md)定义状态转移过程.
    * [`SetName`](./set-name.md)
    * [`BuyName`](./buy-name.md)
5. 通过[`Queriers`](./queriers.md)建立你的状态机视图.
6. 用[`sdk.Codec`](./codec.md) 在编码格式中注册你的类型.
7. 创建[你的模型的CLI 交互](./cli.md).
8. 创建[客户端调用nameservice应用的HTTP路由](./rest.md)
9. 引入你的模型并且[完成建立你的应用](./app-complete.md)!
10. 为你的应用创建 [`nsd` and `nscli` 入口](./entrypoint.md).
11. 用[`dep`管理依赖关系](./dep.md).
12. [建立并运行](./build-run.md) 样例.

## [请双击这里](./app-design.md)开始我们的教程!
