# 开始创建你的应用程序

首先创建新文件`./app.go`，该文件是确定性状态机的核心。

在`app.go`中，您可以定义应用在接收交易时执行的操作。 但首先，它需以正确的顺序接收交易，这就是[Tendermint共识引擎](https://github.com/tendermint/tendermint)的作用。

首先导入必要的依赖包:

```go
package app

import (
  "github.com/tendermint/tendermint/libs/log"
	"github.com/cosmos/cosmos-sdk/x/auth"

	bam "github.com/cosmos/cosmos-sdk/baseapp"
	dbm "github.com/tendermint/tendermint/libs/db"
)
```

导入的每个模块和包的godoc链接:

- [`log`](https://godoc.org/github.com/tendermint/tendermint/libs/log): Tendermint的日志模块。
- [`auth`](https://godoc.org/github.com/cosmos/cosmos-sdk/x/auth): Cosmos SDK的`auth` 模块。
- [`dbm`](https://godoc.org/github.com/tendermint/tendermint/libs/db): Tendermint数据库的代码。
- [`baseapp`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp): 见下文

这里有几个`tendermint` 数据包。Tendermint通为[ABCI](https://github.com/tendermint/tendermint/tree/master/abci)接口将交易从网络层传到应用层。 您正在构建的区块链节点体系结构如下所示：

```
+---------------------+
|                     |
|     Application     |
|                     |
+--------+---+--------+
         ^   |
         |   | ABCI
         |   v
+--------+---+--------+
|                     |
|                     |
|     Tendermint      |
|                     |
|                     |
+---------------------+
```

很方便的是，您不必实现ABCI接口，Cosmos SDK以[`baseapp`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp)的形式提供了实现样例。

以下是`baseapp` 的功能:
- 解码从Tendermint共识引擎收到的交易。
- 从交易中提取消息并执行基本的完备性检查。
- 将消息传送到适当的模块以便处理。 请注意，`baseapp` 并不了解您要使用的特定模块。 这需要您在`app.go`中声明此类模块，教程后面将会详述。`baseapp`仅实现可应用于任何模块的核心路由逻辑。
- 如果ABCI消息是[`DeliverTx`](https://tendermint.com/docs/spec/abci/abci.html#delivertx) ([`CheckTx`](https://tendermint.com/docs/spec/abci/abci.html#checktx) 并不会持续变化)则提交。
- 帮助设置[`Beginblock`](https://tendermint.com/docs/spec/abci/abci.html#beginblock)和[`Endblock`](https://tendermint.com/docs/spec/abci/abci.html#endblock)，这两条消息使您能够在处理每个区块的开头和结尾定义执行的逻辑。 实际上，每个模块都实现了自己的`BeginBlock`和`EndBlock`子逻辑，应用程序的作用是将所有内容聚合在一起（_*注意*_：您不会在应用程序中使用这些消息）。
- 帮助初始化您的状态。
- 帮助设置查询。

现在，您需要为您的应用程序创建一个新的自定义类型`nameserviceApp`。 这种类型中插入 `baseapp`（在Go中嵌入类似于其他语言的继承），可以访问所有`baseapp`的方法。

```go
const (
    appName = "nameservice"
)

type nameserviceApp struct {
    *bam.BaseApp
}
```

为您的应用添加一个简单的构造函数：

```go
func NewnameserviceApp(logger log.Logger, db dbm.DB) *nameserviceApp {

    // First define the top level codec that will be shared by the different modules
    cdc := MakeCodec()

    // BaseApp handles interactions with Tendermint through the ABCI protocol
    bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc))

    var app = &nameserviceApp{
        BaseApp: bApp,
        cdc:     cdc,
    }

    return app
}
```

很好，现在你的应用已经有了整体的框架，但是还缺乏一些功能。

`baseapp`不了解您要在应用程序中使用的消息传递路径或用户交互。 您的应用程序的主要作用就是定义这些路由，同时定义初始状态。这两件事都要求您在应用程序中添加模块。

正如您在[应用设计](./app-design.md)中看到的，在nameservice中您需要三个模块：`auth`，`bank`和`nameservice`。 前两个已经存在，但最后一个还没有。`nameservice`模块将定义大部分状态机，下一步就是构建它。

### 为了完成应用程序，您需要构建一些模块。[开始构建您的nameservice模块](./keeper.md)，稍后你会回到`app.go`.
