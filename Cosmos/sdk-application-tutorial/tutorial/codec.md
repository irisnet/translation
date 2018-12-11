# 编码文件

[用Amino注册你的类型](https://github.com/tendermint/go-amino#registering-types) 以便对数据做编解码，这里我们需要在`./x/nameservice/codec.go`中加入一些代码。所有开发者创建的接口以及实现接口所需的数据结构都需要在`RegisterCodec`函数中声明。在这个模块中，我们需要在这里注册`Msg`实现的两个接口（`SetName`和`BuyName`），但是，在这里还不需要注册`Whois`查询的返回类型

```go
package nameservice

import (
	"github.com/cosmos/cosmos-sdk/codec"
)

// RegisterCodec registers concrete types on wire codec
func RegisterCodec(cdc *codec.Codec) {
	cdc.RegisterConcrete(MsgSetName{}, "nameservice/SetName", nil)
	cdc.RegisterConcrete(MsgBuyName{}, "nameservice/BuyName", nil)
}
```

### 下一步我们将定义您模块中的[CLI交互](./cli.md)部分

