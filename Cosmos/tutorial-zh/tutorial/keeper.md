The Keeper

Cosmos SDK模块的主要核心是Keeper。 它处理与存储器之间的的交互，引用其他管理员进行跨模块交互，并包含Cosmos SDK模块的大部分核心功能。

首先创建文件./x/nameservice/keeper.go来保存模块的keeper。 在Cosmos SDK应用程序中，模块一般位于./x/文件夹中。

Keeper架构

请在./x/nameservice/keeper.go文件中定义nameservice.Keeper ，以启动SDK模块：

package nameservice

import (
	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/cosmos/cosmos-sdk/x/bank"

	sdk "github.com/cosmos/cosmos-sdk/types"
)

// Keeper maintains the link to data storage and exposes getter/setter methods for the various parts of the state machine
type Keeper struct {
	coinKeeper bank.Keeper

	namesStoreKey  sdk.StoreKey // Unexposed key to access name store from sdk.Context
	ownersStoreKey sdk.StoreKey // Unexposed key to access owners store from sdk.Context
	pricesStoreKey sdk.StoreKey // Unexposed key to access prices store from sdk.Context

	cdc *codec.Codec // The wire codec for binary encoding/decoding.
}
以下是关于上述代码的几点注意事项:

导入了3种不同的cosmos-sdk数据包:

codec	- codec提供了使用Cosmos编码格式的工具 Amino。

bank -bank模块控制账户及token的转移。

types - types 包含整个SDK中的常用类型。

Keeper 的架构。在keeper中有几个关键部分:

bank.Keeper -这是bank模块中对Keeper 的引用，允许该模块中的代码从bank模块调用函数。 SDK使用对象功能 来访问应用程序状态的各个部分。 这是为了允许开发人员采用最少权限的方法，限制故障或恶意模块的功能影响它不需要访问的部分状态。

*codec.Codec -这是Amino用于指向编码和解码二进制结构的codec指针。

sdk.StoreKey -可以访问一个长期保存应用程序状态的sdk.KVStore 。

该模块有3个存储密钥:

namesStoreKey - 这是用于存储由name指向的值字符串的主存储器 (i.e. map[name]value)
ownersStoreKey - 该存储器中包含所有给定名称的当前所有者(i.e. map[sdk_address]name)
priceStoreKey - 该存储器包含当前所有者为给定名称支付的价格。 任何购买此名称的人都必须以高于当前所有者的价格来购买。 (i.e. map[name]price)
Getters and Setters

现在来通过Keeper添加与存储器交互的方法。 首先，添加一个函数来设置给定名称解析的字符串：

// SetName - sets the value string that a name resolves to
func (k Keeper) SetName(ctx sdk.Context, name string, value string) {
	store := ctx.KVStore(k.namesStoreKey)
	store.Set([]byte(name), []byte(value))
}
在这个方法中，首先使用Keeper中的 namesStoreKey 获取map[name]value的存储对象。

注意: 此函数使用sdk.Context。 该对象包含很多访问状态的重要功能，如blockHeight和chainID。
接下来，使用 .Set([]byte, []byte)方法将<name, value>导入存储。 由于只能存储[]byte格式，因此首先将string转换为[]byte，然后将它们作为参数用于Set方法。

接下来，添加一个方法来解析名称（i.e. 查找name的value）：

// ResolveName - returns the string that the name resolves to
func (k Keeper) ResolveName(ctx sdk.Context, name string) string {
	store := ctx.KVStore(k.namesStoreKey)
	bz := store.Get([]byte(name))
	return string(bz)
}
这里，与SetName方法一样，首先使用 StoreKey访问存储器。 接下来，不使用存储键上的Set方法，而是使用.Get([]byte) []byte的方法，作为函数的参数，传递密钥，即name字符串转换为[]byte，并以[]byte的形式返回结果，再将[]byte转换为string并返回结果。

添加类似功能以获取并设置名称所有者:

// HasOwner - returns whether or not the name already has an owner
func (k Keeper) HasOwner(ctx sdk.Context, name string) bool {
	store := ctx.KVStore(k.ownersStoreKey)
	bz := store.Get([]byte(name))
	return bz != nil
}

// GetOwner - get the current owner of a name
func (k Keeper) GetOwner(ctx sdk.Context, name string) sdk.AccAddress {
	store := ctx.KVStore(k.ownersStoreKey)
	bz := store.Get([]byte(name))
	return bz
}

// SetOwner - sets the current owner of a name
func (k Keeper) SetOwner(ctx sdk.Context, name string, owner sdk.AccAddress) {
	store := ctx.KVStore(k.ownersStoreKey)
	store.Set([]byte(name), owner)
}
上述代码的注意事项:

从存储器ownersStoreKey得到数据，而不是从namesStoreKey。
sdk.AccAddress 是 []byte类型的别名, 两者可以互相转换。
还有一个额外的函数HasOwner ，可方便用于条件语句。
最后，为名称的价格添加一个getter和setter：

// GetPrice - gets the current price of a name.  If price doesn't exist yet, set to 1steak.
func (k Keeper) GetPrice(ctx sdk.Context, name string) sdk.Coins {
	if !k.HasOwner(ctx, name) {
		return sdk.Coins{sdk.NewInt64Coin("mycoin", 1)}
	}
	store := ctx.KVStore(k.pricesStoreKey)
	bz := store.Get([]byte(name))
	var price sdk.Coins
	k.cdc.MustUnmarshalBinaryBare(bz, &price)
	return price
}

// SetPrice - sets the current price of a name
func (k Keeper) SetPrice(ctx sdk.Context, name string, price sdk.Coins) {
	store := ctx.KVStore(k.pricesStoreKey)
	store.Set([]byte(name), k.cdc.MustMarshalBinaryBare(price))
}
上述代码的注意事项:

sdk.Coins没有自己的字节编码，这意味着需要使用Amino对价格进行编组和解组，以从存储器中插入或删除。
获取没有所有者的名称价格（没有名称所以没有价格）时，返回 1steak 作为价格。
./x/nameservice/keeper.go文件中所需的最后一段代码是Keeper的构造函数：

// NewKeeper creates new instances of the nameservice Keeper
func NewKeeper(coinKeeper bank.Keeper, namesStoreKey sdk.StoreKey, ownersStoreKey sdk.StoreKey, priceStoreKey sdk.StoreKey, cdc *codec.Codec) Keeper {
	return Keeper{
		coinKeeper:     coinKeeper,
		namesStoreKey:  namesStoreKey,
		ownersStoreKey: ownersStoreKey,
		pricesStoreKey: priceStoreKey,
		cdc:            cdc,
	}
}
接下来我们将了解用户如何使用Msgs 和 Handlers与新存储器进行交互
