# SDK应用教程

在本教程中，您将构建一个[Cosmos SDK](https://github.com/cosmos/cosmos-sdk/)的功能性应用，并学习SDK的基本概念和架构。该示例将教您如何基于Cosmos SDK快速**建立自己的区块链**。

在本教程结束后，您将拥有一个`nameservice`应用程序，该应用是实现一个字符串到其他字符串的映射（`map [string] string`），类似于模拟传统的DNS系统（`map [domain] zonefile`），如 [Namecoin](https://namecoin.org/), [ENS](https://ens.domains/) 或 [Handshake](https://handshake.org/)。 用户可以用nameservice应用购买未使用的名称，或出售/交易他们的名称。

本教程的所有最终源代码（及编译）都在此目录中。不过建议最好手动完成并尝试自己构建项目。

## 要求

- 安装[`golang` >1.11](https://golang.org/doc/install) 
- 一个可工作的[`$GOPATH`](https://github.com/golang/go/wiki/SettingGOPATH)目录
- 有意愿建立自己的区块链

## 教程

通过本教材，你将建立如下文件及自己的应用：

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

首先建立一个新的 github 代码仓库:

```bash
mkdir -p $GOPATH/src/github.com/{ .Username }/nameservice
cd $GOPATH/src/github.com/{ .Username }/nameservice
git init
```

然后按照如下步骤操作即可。第一步是应用的设计，如果你想直接跳到代码部分，可以直接进入[第二步](./keeper.md)。

### 教程章节

1. 应用[设计](./app-design.md)。
2. 在[`./app.go`](./app-init.md)中开始实现你的应用。
2. 开始用[`Keeper`](./keeper.md)建立模块。
3. 通过[`Msgs` 和 `Handlers`](./msgs-handlers.md)定义状态变换。
    * [`SetName`](./set-name.md)
    * [`BuyName`](./buy-name.md)
4. 用[`Queriers`](./queriers.md)在你的状态机上创建视图。
5. 用[`sdk.Codec`](./codec.md)注册类型的编码格式。
6. 为你的模块[创建CLI交互](./cli.md)。
7. [为客户端创建HTTP路由以访问你的nameservice](./rest.md)。
8. 导入模块并 [完成应用的建立](./app-complete.md)!
9. 为应用创建 [`nsd` 和 `nscli` 入口点](./entrypoint.md)。
10. [利用 `dep`](./dep.md)建立并管理依赖关系。
11. [编译并运行](./build-run.md) 示例。

## [点击这里](./app-design.md) 进入教程！
