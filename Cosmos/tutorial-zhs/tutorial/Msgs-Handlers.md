# Msgs and Handlers

现在您已经有了`Keeper`设置，现在就可以构建真正允许用户购买名称并设置值的`Msgs` 和`Handlers`了。

## `Msgs`

`Msgs` 触发状态转换。`Msgs`包含在客户端提交给网络的[`Txs`](https://github.com/cosmos/cosmos-sdk/blob/develop/types/tx_msg.go#L34-L38) 中。 Cosmos SDK包装并解包来自`Txs`的`Msgs`，这意味着，作为开发人员，您只需要定义`Msgs`。 `Msgs`必须满足以下接口：

```go
// Transactions messages must fulfill the Msg
type Msg interface {
	// Return the message type.
	// Must be alphanumeric or empty.
	Type() string

	// Returns a human-readable string for the message, intended for utilization
	// within tags
	Route() string

	// ValidateBasic does a simple validation check that
	// doesn't require access to any other information.
	ValidateBasic() Error

	// Get the canonical byte representation of the Msg.
	GetSignBytes() []byte

	// Signers returns the addrs of signers that must sign.
	// CONTRACT: All signatures must be present to be valid.
	// CONTRACT: Returns addrs in some deterministic order.
	GetSigners() []AccAddress
}
```

## `Handlers`

`Handlers`定义收到给定`Msg`时需要采取的操作（哪些存储需要更新，如何以及在什么条件下进行更新）。

在此模块中，您有两种类型的`Msgs`，用户可以发送这些消息以与应用程序状态进行交互：[`SetName`](./set-name.md) 和[`BuyName`](./buy-name.md)。 每种类型都有一个相关的`Handler`。

### 现在您已经更好地了解了 `Msgs`和`Handlers`，可以开始构建您的第一条消息了：[`SetName`](./set-name.md)。
