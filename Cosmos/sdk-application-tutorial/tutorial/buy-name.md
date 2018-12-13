# 名称购买消息

## Msg消息

现在我们来定义购买名称的`Msg`消息，并把它加入到`./x/nameservice/msgs.go`文件中，这部分代码和`SetName`很相似。 

```go
// MsgBuyName defines the BuyName message
type MsgBuyName struct {
	NameID string
	Bid    sdk.Coins
	Buyer  sdk.AccAddress
}

// NewMsgBuyName is the constructor function for MsgBuyName
func NewMsgBuyName(name string, bid sdk.Coins, buyer sdk.AccAddress) MsgBuyName {
	return MsgBuyName{
		NameID: name,
		Bid:    bid,
		Buyer:  buyer,
	}
}

// Type Implements Msg.
func (msg MsgBuyName) Route() string { return "nameservice" }

// Name Implements Msg.
func (msg MsgBuyName) Type() string { return "buy_name" }

// ValidateBasic Implements Msg.
func (msg MsgBuyName) ValidateBasic() sdk.Error {
	if msg.Buyer.Empty() {
		return sdk.ErrInvalidAddress(msg.Buyer.String())
	}
	if len(msg.NameID) == 0 {
		return sdk.ErrUnknownRequest("Name cannot be empty")
	}
	if !msg.Bid.IsPositive() {
		return sdk.ErrInsufficientCoins("Bids must be positive")
	}
	return nil
}

// GetSignBytes Implements Msg.
func (msg MsgBuyName) GetSignBytes() []byte {
	b, err := json.Marshal(msg)
	if err != nil {
		panic(err)
	}
	return sdk.MustSortJSON(b)
}

// GetSigners Implements Msg.
func (msg MsgBuyName) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{msg.Buyer}
}
```

接下来，在./x/nameservice/handler.go`文件中将`MsgBuyName`的处理函数加入到模块的路由函数中中：

```go
// NewHandler returns a handler for "nameservice" type messages.
func NewHandler(keeper Keeper) sdk.Handler {
	return func(ctx sdk.Context, msg sdk.Msg) sdk.Result {
		switch msg := msg.(type) {
		case MsgSetName:
			return handleMsgSetName(ctx, keeper, msg)
		case MsgBuyName:
			return handleMsgBuyName(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type())
			return sdk.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```

最后，让我们来定义`BuyName` `handler`函数用于执行由消息引发的状态转换。开发者需要记住，这时候消息的`ValidateBasic`函数仍然在执行，因此，输入数据会被验证。然而，`ValidateBasic`并不能查询应用状态和验证逻辑中与网络状态相关的部分（例如，账户余额），所以需要在`handler`函数中完成这部分响应。 

```go
// Handle MsgBuyName
func handleMsgBuyName(ctx sdk.Context, keeper Keeper, msg MsgBuyName) sdk.Result {
	if keeper.GetPrice(ctx, msg.NameID).IsAllGT(msg.Bid) { // Checks if the the bid price is greater than the price paid by the current owner
		return sdk.ErrInsufficientCoins("Bid not high enough").Result() // If not, throw an error
	}
	if keeper.HasOwner(ctx, msg.NameID) {
		_, err := keeper.coinKeeper.SendCoins(ctx, msg.Buyer, keeper.GetOwner(ctx, msg.NameID), msg.Bid)
		if err != nil {
			return sdk.ErrInsufficientCoins("Buyer does not have enough coins").Result()
		}
	} else {
		_, _, err := keeper.coinKeeper.SubtractCoins(ctx, msg.Buyer, msg.Bid) // If so, deduct the Bid amount from the sender
		if err != nil {
			return sdk.ErrInsufficientCoins("Buyer does not have enough coins").Result()
		}
	}
	keeper.SetOwner(ctx, msg.NameID, msg.Buyer)
	keeper.SetPrice(ctx, msg.NameID, msg.Bid)
	return sdk.Result{}
}
```

首先，需要检查购买者的出价是否比现在的价格高。然后需要检查此域名是否已经有了主人，如果已经有主人了，那么域名的主人将收到购买者支付的费用。 

如果没有主人，那么您的`nameservice`模块将销毁购买者支付的通证（比如，将通证发到一个不可恢复的地址）。 

如果`SubtractCoins` 或 `SendCoins`返回一个non-nil error错误，处理程序将扔出错误提示，并且恢复交易状态。如果没有收到错误反馈，就调用前面`Keeper`中定义的get和set类型函数将购买者设置为新的主人，并将价格设置为新的交易报价。 

> _*注意*_: 这个处理程序调用`coinKeeper`提供的函数来处理通证操作。如果您想在应用中实现通证操作，希望了解更多可用的函数，请[参考这个模块的介绍](https://godoc.org/github.com/cosmos/cosmos-sdk/x/bank#BaseKeeper) .

### 现在您已经完成了`Msgs`消息和`Handlers`消息处理函数的定义，下面来学习如何在[可查询交易](./queriers.md)上生成数据!
