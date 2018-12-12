# SetName

## `Msg`

SDK `Msgs` 的命名一般为`Msg{ .Action }`。 要实现的第一个操作是`SetName`，它是一个允许地址所有者设置解析名称结果的`Msg`。 首先在名为`./x/nameservice/msgs.go`的新文件中定义`MsgSetName`：

```go
package nameservice

import (
	"encoding/json"

	sdk "github.com/cosmos/cosmos-sdk/types"
)

// MsgSetName defines a SetName message
type MsgSetName struct {
	NameID string
	Value  string
	Owner  sdk.AccAddress
}

// NewMsgSetName is a constructor function for MsgSetName
func NewMsgSetName(name string, value string, owner sdk.AccAddress) MsgSetName {
	return MsgSetName{
		NameID: name,
		Value:  value,
		Owner:  owner,
	}
}
```

`MsgSetName` 设置名称值涉及三个属性:
- `name` - 设置的名称
- `value` - 名称解析值
- `owner` - 名称所有者

> *注意*: 字段名称是 `NameID`而不是`Name`，因为`.Route()`是`Msg` 接口上方法的名称。 这将在[SDK未来的更新](https://github.com/cosmos/cosmos-sdk/issues/2456)中得到解决。

接下来，实现`Msg`接口:

```go
// Type should return the name of the module
func (msg MsgSetName) Route() string { return "nameservice" }

// Name should return the action
func (msg MsgSetName) Type() string { return "set_name"}
```

SDK使用上述函数将`Msgs`传递到适当的模块进行处理。它们还为用于索引的数据库标记添加了可读的名称。

```go
// ValdateBasic Implements Msg.
func (msg MsgSetName) ValidateBasic() sdk.Error {
	if msg.Owner.Empty() {
		return sdk.ErrInvalidAddress(msg.Owner.String())
	}
	if len(msg.NameID) == 0 || len(msg.Value) == 0 {
		return sdk.ErrUnknownRequest("Name and/or Value cannot be empty")
	}
	return nil
}
```

`ValidateBasic`用于提供对`Msg`是否有效的一些基本**无状态**检查。 在这种情况下，检查没有属性为空。请注意这里使用`sdk.Error` 类型。 SDK提供了一组开发人员经常遇到的错误类型。

```go
// GetSignBytes Implements Msg.
func (msg MsgSetName) GetSignBytes() []byte {
	b, err := json.Marshal(msg)
	if err != nil {
		panic(err)
	}
	return sdk.MustSortJSON(b)
}
```

`GetSignBytes` 定义了`Msg`如何编码以进行签名。多数情况下，这意味着要对排序的JSON进行编组，而不应修改输出。

```go
// GetSigners Implements Msg.
func (msg MsgSetName) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{msg.Owner}
}
```

`GetSigners` 定义在 `Tx` 上需要的签名以使其有效。例如，在这种情况下，`MsgSetName`要求`Owner`在尝试重置名称所指向的内容时签署该交易。

## `Handler`

现在指定了`MsgSetName`，下一步是定义收到此消息时需要采取的操作。 这就是`handler`的作用。

在新文件中(`./x/nameservice/handler.go`) 从下列代码开始:

```go
package nameservice

import (
	"fmt"

	sdk "github.com/cosmos/cosmos-sdk/types"
)

// NewHandler returns a handler for "nameservice" type messages.
func NewHandler(keeper Keeper) sdk.Handler {
	return func(ctx sdk.Context, msg sdk.Msg) sdk.Result {
		switch msg := msg.(type) {
		case MsgSetName:
			return handleMsgSetName(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type())
			return sdk.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```

`NewHandler`本质上是一个子路由器，它将进入该模块的消息定向到正确的处理程序Handler。 目前，只有一个`Msg`/`Handler`。

现在，您需要在`handleMsgSetName`中定义处理`MsgSetName`消息的实际逻辑：

> _*注意*_: SDK中handler的名称一般为 `handleMsg{ .Action }`

```go
// Handle MsgSetName
func handleMsgSetName(ctx sdk.Context, keeper Keeper, msg MsgSetName) sdk.Result {
	if !msg.Owner.Equals(keeper.GetOwner(ctx, msg.NameID)) { // Checks if the the msg sender is the same as the current owner
		return sdk.ErrUnauthorized("Incorrect Owner").Result() // If not, throw an error
	}
	keeper.SetName(ctx, msg.NameID, msg.Value) // If so, set the name to the value specified in the msg.
	return sdk.Result{}                      // return
}
```

在该函数中，检查`Msg`发件人是否是真正的名称所有者（`keeper.GetOwner`）。 如果是这样，他们可以通过调用`Keeper`上的函数来设置名称。如果没有，抛出错误并将其返回给用户。

### 太棒了，现在名称所有者可以`SetName`了！ 但是，如果一个名称还没有拥有者呢？模块中就需要一种方式让用户购买名称！接下来就让我们[定义BuyName消息](./buy-name.md)吧！
