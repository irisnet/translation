# Nameservice模型CLI

Cosmos SDK采用[`cobra`](https://github.com/spf13/cobra) 库实现CLI交互，通过这个库，各个模块可以非常简单的把自己的命令开放出来。创建以下文件，开始定义用户CLI与模块之间的交互:

- `./x/nameservice/client/cli/query.go`
- `./x/nameservice/client/cli/tx.go`
- `./x/nameservice/client/module_client.go`

### 查询

从`query.go`开始， 为每个模块`Queriers` (`resolve`, 和 `whois`)定义`cobra.Command`

```go
package cli

import (
	"fmt"

	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/spf13/cobra"
)

// GetCmdResolveName queries information about a name
func GetCmdResolveName(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "resolve [name]",
		Short: "resolve name",
		Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)
			name := args[0]

			res, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/resolve/%s", queryRoute, name), nil)
			if err != nil {
				fmt.Printf("could not resolve name - %s \n", string(name))
				return nil
			}

			fmt.Println(string(res))

			return nil
		},
	}
}

// GetCmdWhois queries information about a domain
func GetCmdWhois(queryRoute string, cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "whois [name]",
		Short: "Query whois info of name",
		Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc)
			name := args[0]

			res, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/whois/%s", queryRoute, name), nil)
			if err != nil {
				fmt.Printf("could not resolve whois - %s \n", string(name))
				return nil
			}

			fmt.Println(string(res))

			return nil
		},
	}
}
```

以上代码的注解:
- CLI引入一个新的实例 `context`: [`CLIContext`]，其中包括了CLI交互所需的用户输入与应用配置的数据。
- `cliCtx.QueryWithData()`请求的`path`直接映射到查询路径中的名称。
  * 路径中的第一部分用于区分SDK应用程序可能的查询类型： `Queriers`是`custom`。
  * 第二部分`nameservice`是查询指向的模块的名字。 
  * 最后将调用模块中特定的查询。
  * 在这个例子中，第四部分是查询，这是可以实现的，因为查询的参数是简单的字符串。如果要实现更复杂的查询，那么就需要用到[`.QueryWithData()`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/context#CLIContext.QueryWithData)函数的第二个参数来带入`data`。此类查询的样例可以参考[Staking模块中的查询样例](https://github.com/cosmos/cosmos-sdk/blob/develop/x/stake/querier/querier.go#L103)。
 

### 交易

现在，我们已经完成了查询交互的定义，现在转到`tx.go`生成交易。

> _*注意*_: 您的应用需要导入您刚写的代码，教程所在代码仓库的导入路径是：(`github.com/cosmos/sdk-application-tutorial/x/nameservice`)。如果您是在您自己的代码仓库开发，那么这个导入路径的格式如下：(`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`).

```go
package cli

import (
	"github.com/spf13/cobra"

	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/client/utils"
	"github.com/cosmos/cosmos-sdk/codec"
	"github.com/cosmos/sdk-application-tutorial/x/nameservice"

	sdk "github.com/cosmos/cosmos-sdk/types"
	authcmd "github.com/cosmos/cosmos-sdk/x/auth/client/cli"
	authtxb "github.com/cosmos/cosmos-sdk/x/auth/client/txbuilder"
)

// GetCmdBuyName is the CLI command for sending a BuyName transaction
func GetCmdBuyName(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "buy-name [name] [amount]",
		Short: "bid for existing name or claim new name",
		Args:  cobra.ExactArgs(2),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc).WithAccountDecoder(cdc)

			txBldr := authtxb.NewTxBuilderFromCLI().WithCodec(cdc)

			if err := cliCtx.EnsureAccountExists(); err != nil {
				return err
			}

			coins, err := sdk.ParseCoins(args[1])
			if err != nil {
				return err
			}

			account, err := cliCtx.GetFromAddress()
			if err != nil {
				return err
			}

			msg := nameservice.NewMsgBuyName(args[0], coins, account)
			err = msg.ValidateBasic()
			if err != nil {
				return err
			}

			cliCtx.PrintResponse = true

			return utils.CompleteAndBroadcastTxCli(txBldr, cliCtx, []sdk.Msg{msg})
		},
	}
}

// GetCmdSetName is the CLI command for sending a SetName transaction
func GetCmdSetName(cdc *codec.Codec) *cobra.Command {
	return &cobra.Command{
		Use:   "set-name [name] [value]",
		Short: "set the value associated with a name that you own",
		Args:  cobra.ExactArgs(2),
		RunE: func(cmd *cobra.Command, args []string) error {
			cliCtx := context.NewCLIContext().WithCodec(cdc).WithAccountDecoder(cdc)

			txBldr := authtxb.NewTxBuilderFromCLI().WithCodec(cdc)

			if err := cliCtx.EnsureAccountExists(); err != nil {
				return err
			}

			account, err := cliCtx.GetFromAddress()
			if err != nil {
				return err
			}

			msg := nameservice.NewMsgSetName(args[0], args[1], account)
			err = msg.ValidateBasic()
			if err != nil {
				return err
			}

			cliCtx.PrintResponse = true

			return utils.CompleteAndBroadcastTxCli(txBldr, cliCtx, []sdk.Msg{msg})
		},
	}
}
```

以上代码注解:
- 这里使用了`authcmd`数据包. [godocs提供了很多如何使用该数据包的方法](https://godoc.org/github.com/cosmos/cosmos-sdk/x/auth/client/cli#GetAccountDecoder). `authcmd`数据包提供了对CLI控制的账户的访问，并且便于对其签名。

### 模块客户端

实现这个功能应用的最后一部分工作叫做 `ModuleClient`. [Module clients](https://godoc.org/github.com/cosmos/cosmos-sdk/types#ModuleClients) 提供一个标准的方法，为模块导出它的客户端应用功能。 

> _*注意*_: 您的应用需要导入您刚写的代码，教程所在代码仓库的导入路径是：(`github.com/cosmos/sdk-application-tutorial/x/nameservice`)。如果您是在自己的代码仓库开发，那么这个导入路径的格式如下：(`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`).

```go
package client

import (
	"github.com/cosmos/cosmos-sdk/client"
	nameservicecmd "github.com/cosmos/sdk-application-tutorial/x/nameservice/client/cli"
	"github.com/spf13/cobra"
	amino "github.com/tendermint/go-amino"
)

// ModuleClient exports all client functionality from this module
type ModuleClient struct {
	storeKey string
	cdc      *amino.Codec
}

func NewModuleClient(storeKey string, cdc *amino.Codec) ModuleClient {
	return ModuleClient{storeKey, cdc}
}

// GetQueryCmd returns the cli query commands for this module
func (mc ModuleClient) GetQueryCmd() *cobra.Command {
	// Group gov queries under a subcommand
	govQueryCmd := &cobra.Command{
		Use:   "nameservice",
		Short: "Querying commands for the nameservice module",
	}

	govQueryCmd.AddCommand(client.GetCommands(
		nameservicecmd.GetCmdResolveName(mc.storeKey, mc.cdc),
		nameservicecmd.GetCmdWhois(mc.storeKey, mc.cdc),
	)...)

	return govQueryCmd
}

// GetTxCmd returns the transaction commands for this module
func (mc ModuleClient) GetTxCmd() *cobra.Command {
	govTxCmd := &cobra.Command{
		Use:   "nameservice",
		Short: "Nameservice transactions subcommands",
	}

	govTxCmd.AddCommand(client.PostCommands(
		nameservicecmd.GetCmdBuyName(mc.cdc),
		nameservicecmd.GetCmdSetName(mc.cdc),
	)...)

	return govTxCmd
}
```

以上代码的注解:
- 这个抽象结构允许客户端以标准方法从模块导入客户端功能。您将在[构建入口](./entrypoint.md)中看到这些。
- 这里一个有[尚未完成的功能](https://github.com/cosmos/cosmos-sdk/issues/2955) ，即在该接口添加Rest功能 (我们将在此教程的下一部分讨论这个问题) .

### 现在你可以[为Rest客户端定义与模块进行通讯的路径](./rest.md)了！
