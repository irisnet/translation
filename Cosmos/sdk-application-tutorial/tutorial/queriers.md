# 建立查询

首先，我们开始创建文件`./x/nameservice/querier.go` 。这个文件用于定义应用提供哪些查询功能给客户使用。你们的`nameservice`模块将开放2种查询：

- `resolve`: 用于查询`name`对应`nameservice`保存的`value`，这种查询方式与DNS查询很像。 
- `whois`: 	用于查询`name`对应的`price`, `value`, 和name的`owner`。主要用于帮助用户查询所需购买name的成本。 

我们先来定义`NewQuerier`函数定义将查询传递到应用模块的路径（就像`NewHandler`函数一样）。 由于查询并没有定义一个类似与`Msg`的接口，您需要手工定义不同状态分支的操作（它们不能从query的`.Route()`函数中删除）：

```go
package nameservice

import (
	"github.com/cosmos/cosmos-sdk/codec"

	sdk "github.com/cosmos/cosmos-sdk/types"
	abci "github.com/tendermint/tendermint/abci/types"
)

// query endpoints supported by the governance Querier
const (
	QueryResolve = "resolve"
	QueryWhois   = "whois"
)

// NewQuerier is the module level router for state queries
func NewQuerier(keeper Keeper) sdk.Querier {
	return func(ctx sdk.Context, path []string, req abci.RequestQuery) (res []byte, err sdk.Error) {
		switch path[0] {
		case QueryResolve:
			return queryResolve(ctx, path[1:], req, keeper)
		case QueryWhois:
			return queryWhois(ctx, path[1:], req, keeper)
		default:
			return nil, sdk.ErrUnknownRequest("unknown nameservice query endpoint")
		}
	}
}
```

现在，查询路径已经定义好了，让我们开始定义每个查询对应的输入与输出相应。 

```go
// nolint: unparam
func queryResolve(ctx sdk.Context, path []string, req abci.RequestQuery, keeper Keeper) (res []byte, err sdk.Error) {
	name := path[0]

	value := keeper.ResolveName(ctx, name)

	if value == "" {
		return []byte{}, sdk.ErrUnknownRequest("could not resolve name")
	}

	return []byte(value), nil
}

// nolint: unparam
func queryWhois(ctx sdk.Context, path []string, req abci.RequestQuery, keeper Keeper) (res []byte, err sdk.Error) {
	name := path[0]

	whois := Whois{}

	whois.Value = keeper.ResolveName(ctx, name)
	whois.Owner = keeper.GetOwner(ctx, name)
	whois.Price = keeper.GetPrice(ctx, name)

	bz, err2 := codec.MarshalJSONIndent(keeper.cdc, whois)
	if err2 != nil {
		panic("could not marshal result to JSON")
	}

	return bz, nil
}

// Whois represents a name -> value lookup
type Whois struct {
	Value string         `json:"value"`
	Owner sdk.AccAddress `json:"owner"`
	Price sdk.Coins      `json:"price"`
}
```

以上代码注解:
- 这里，你的`Keeper`的get和set类型的函数会被大量使用。当您在建立一个新的应用时，您需要回到前面的模型中，根据需要增添相应的get和set类型的函数。 
- 如果您的应用需要更多的客户响应类型（比如`Whois`），可以在这里定义这些新的相应类型。 
  
### 现在您已经可以改变和看到您的模块的状态，我们需要继续完善它。下一步，我们将在[Animo编码格式中注册您定义的类型](./codec.md)!
