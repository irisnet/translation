# 合并模块，完成应用开发

现在你的模块已经完成了，开以和[`auth`](https://godoc.org/github.com/cosmos/cosmos-sdk/x/auth) 与 [`bank`](https://godoc.org/github.com/cosmos/cosmos-sdk/x/bank)这两个模块合并:

_*注意*_: 您的应用需要导入您刚写的代码，教程所在代码仓库的导入路径是：(`github.com/cosmos/sdk-application-tutorial/x/nameservice`)。如果您是在您自己的代码仓库开发，那么这个导入路径的格式如下：(`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`).

```go
package app

import (
	"encoding/json"

	"github.com/tendermint/tendermint/libs/log"

	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/cosmos/cosmos-sdk/x/auth"
	"github.com/cosmos/cosmos-sdk/x/bank"
	"github.com/cosmos/cosmos-sdk/x/stake"
	"github.com/cosmos/sdk-application-tutorial/x/nameservice"

	bam "github.com/cosmos/cosmos-sdk/baseapp"
	sdk "github.com/cosmos/cosmos-sdk/types"
	abci "github.com/tendermint/tendermint/abci/types"
	cmn "github.com/tendermint/tendermint/libs/common"
	dbm "github.com/tendermint/tendermint/libs/db"
	tmtypes "github.com/tendermint/tendermint/types"
)

```
然后，你需要把存储的keys以及`Keepers`加到你`nameserviceApp`结构中，并且更新相应的构造函数。 

```go
const (
	appName = "nameservice"
)

type nameserviceApp struct {
	*bam.BaseApp
	cdc *codec.Codec

	keyMain          *sdk.KVStoreKey
	keyAccount       *sdk.KVStoreKey
	keyNSnames       *sdk.KVStoreKey
	keyNSowners      *sdk.KVStoreKey
	keyNSprices      *sdk.KVStoreKey
	keyFeeCollection *sdk.KVStoreKey

	accountKeeper       auth.AccountKeeper
	bankKeeper          bank.Keeper
	feeCollectionKeeper auth.FeeCollectionKeeper
	nsKeeper            nameservice.Keeper
}

func NewnameserviceApp(logger log.Logger, db dbm.DB) *nameserviceApp {

  // First define the top level codec that will be shared by the different modules
  cdc := MakeCodec()

  // BaseApp handles interactions with Tendermint through the ABCI protocol
  bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc))

  // Here you initialize your application with the store keys it requires
	var app = &nameserviceApp{
		BaseApp: bApp,
		cdc:     cdc,

		keyMain:          sdk.NewKVStoreKey("main"),
		keyAccount:       sdk.NewKVStoreKey("acc"),
		keyNSnames:       sdk.NewKVStoreKey("ns_names"),
		keyNSowners:      sdk.NewKVStoreKey("ns_owners"),
		keyNSprices:      sdk.NewKVStoreKey("ns_prices"),
		keyFeeCollection: sdk.NewKVStoreKey("fee_collection"),
	}

  return app
}
```

在这点上，构造函数仍然缺少重要的逻辑。也就是说，它需要：

- 从每个所需模块中将需要的`Keepers`实例化
- 生成`Keeper`所需的`storeKeys`
- 注册每个模块的`Handlers`，并且在结束时使用`baseapp`的`router`对象的`AddRoute()`方法
- 注册每个模块的`Queriers`，并且最后使用`baseapp`的`queryRouter`属性的`AddRoute()`方法加入Queriers的路由。 
- 将`KVStore`s安装到`baseApp` multistore中的keys上。
- 设置`initChainer`，定义初始应用状态。 

您最后完成的构造函数看上去应该时这样的：

```go
// NewnameserviceApp is a constructor function for nameserviceApp
func NewnameserviceApp(logger log.Logger, db dbm.DB) *nameserviceApp {

	// First define the top level codec that will be shared by the different modules
	cdc := MakeCodec()

	// BaseApp handles interactions with Tendermint through the ABCI protocol
	bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc))

	// Here you initialize your application with the store keys it requires
	var app = &nameserviceApp{
		BaseApp: bApp,
		cdc:     cdc,

		keyMain:          sdk.NewKVStoreKey("main"),
		keyAccount:       sdk.NewKVStoreKey("acc"),
		keyNSnames:       sdk.NewKVStoreKey("ns_names"),
		keyNSowners:      sdk.NewKVStoreKey("ns_owners"),
		keyNSprices:      sdk.NewKVStoreKey("ns_prices"),
		keyFeeCollection: sdk.NewKVStoreKey("fee_collection"),
	}

	// The AccountKeeper handles address -> account lookups
	app.accountKeeper = auth.NewAccountKeeper(
		app.cdc,
		app.keyAccount,
		auth.ProtoBaseAccount,
	)

	// The BankKeeper allows you perform sdk.Coins interactions
	app.bankKeeper = bank.NewBaseKeeper(app.accountKeeper)

	// The FeeCollectionKeeper collects transaction fees and renders them to the fee distribution module
	app.feeCollectionKeeper = auth.NewFeeCollectionKeeper(cdc, app.keyFeeCollection)

	// The NameserviceKeeper is the Keeper from the module for this tutorial
	// It handles interactions with the namestore
	app.nsKeeper = nameservice.NewKeeper(
		app.bankKeeper,
		app.keyNSnames,
		app.keyNSowners,
		app.keyNSprices,
		app.cdc,
	)

	// The AnteHandler handles signature verification and transaction pre-processing
	app.SetAnteHandler(auth.NewAnteHandler(app.accountKeeper, app.feeCollectionKeeper))

	// The app.Router is the main transaction router where each module registers its routes
	// Register the bank and nameservice routes here
	app.Router().
		AddRoute("bank", bank.NewHandler(app.bankKeeper)).
		AddRoute("nameservice", nameservice.NewHandler(app.nsKeeper))

	// The app.QueryRouter is the main query router where each module registers its routes
	app.QueryRouter().
		AddRoute("nameservice", nameservice.NewQuerier(app.nsKeeper))

	// The initChainer handles translating the genesis.json file into initial state for the network
	app.SetInitChainer(app.initChainer)

	app.MountStores(
		app.keyMain,
		app.keyAccount,
		app.keyNSnames,
		app.keyNSowners,
		app.keyNSprices,
	)

	err := app.LoadLatestVersion(app.keyMain)
	if err != nil {
		cmn.Exit(err.Error())
	}

	return app
}
```

`initChainer`定义了区块链初始化开始时，`genesis.json`中的账户如何映射到应用状态。`ExportAppStateAndValidators`函数帮助引导程序引导应用程序进入初始状态。你现在不需要过多的担心它们任何一件事。 

构造函数需要注册`initChainer`函数，但是这里还没有定义这个函数，接下来开始创建这个函数。

```go
// GenesisState represents chain state at the start of the chain. Any initial state (account balances) are stored here.
type GenesisState struct {
	Accounts []*auth.BaseAccount `json:"accounts"`
}

func (app *nameserviceApp) initChainer(ctx sdk.Context, req abci.RequestInitChain) abci.ResponseInitChain {
	stateJSON := req.AppStateBytes

	genesisState := new(GenesisState)
	err := app.cdc.UnmarshalJSON(stateJSON, genesisState)
	if err != nil {
		panic(err)
	}

	for _, acc := range genesisState.Accounts {
		acc.AccountNumber = app.accountKeeper.GetNextAccountNumber(ctx)
		app.accountKeeper.SetAccount(ctx, acc)
	}

	return abci.ResponseInitChain{}
}

// ExportAppStateAndValidators does the things
func (app *nameserviceApp) ExportAppStateAndValidators() (appState json.RawMessage, validators []tmtypes.GenesisValidator, err error) {
	ctx := app.NewContext(true, abci.Header{})
	accounts := []*auth.BaseAccount{}

	appendAccountsFn := func(acc auth.Account) bool {
		account := &auth.BaseAccount{
			Address: acc.GetAddress(),
			Coins:   acc.GetCoins(),
		}

		accounts = append(accounts, account)
		return false
	}

	app.accountKeeper.IterateAccounts(ctx, appendAccountsFn)

	genState := GenesisState{Accounts: accounts}
	appState, err = codec.MarshalJSONIndent(app.cdc, genState)
	if err != nil {
		return nil, nil, err
	}

	return appState, validators, err
}
```

最后增加一个helper函数生成一个amino [`*codec.Codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec#Codec)用于完整登记在这个应用中使用过的所有模块。 

```go
// MakeCodec generates the necessary codecs for Amino
func MakeCodec() *codec.Codec {
	var cdc = codec.New()
	auth.RegisterCodec(cdc)
	bank.RegisterCodec(cdc)
	nameservice.RegisterCodec(cdc)
	stake.RegisterCodec(cdc)
	sdk.RegisterCodec(cdc)
	codec.RegisterCrypto(cdc)
	return cdc
}
```

### 现在您已经创建了一个保护您开发模块的应用，现在开始[建立你的入口点](./entrypoint.md)了!
