# Nameservice模型的Rest接口

你的模型也可以提供一个REST接口用于程序方法模型提供的功能。为此让我们先建立一个文件来负责HTTP处理程序。


- `./x/nameservice/client/rest/rest.go`

首先加入 `imports` 和 `const`s:

```go
package rest

import (
	"fmt"
	"net/http"

	"github.com/cosmos/cosmos-sdk/client/context"
	"github.com/cosmos/cosmos-sdk/client/utils"
	"github.com/cosmos/cosmos-sdk/codec"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/sdk-application-tutorial/x/nameservice"

	"github.com/gorilla/mux"
)

const (
	restName = "name"
)
```

### 注册路径

首先，在`RegisterRoutes`函数中定义模型的REST客户端接口。路径以模块名称开始以避免与其他模块的路径冲突：

```go
// RegisterRoutes - Central function to define routes that get registered by the main application
func RegisterRoutes(cliCtx context.CLIContext, r *mux.Router, cdc *codec.Codec, storeName string) {
	r.HandleFunc(fmt.Sprintf("/%s/names", storeName), buyNameHandler(cdc, cliCtx)).Methods("POST")
	r.HandleFunc(fmt.Sprintf("/%s/names", storeName), setNameHandler(cdc, cliCtx)).Methods("PUT")
	r.HandleFunc(fmt.Sprintf("/%s/names/{%s}", storeName, restName), resolveNameHandler(cdc, cliCtx, storeName)).Methods("GET")
	r.HandleFunc(fmt.Sprintf("/%s/names/{%s}/whois", storeName, restName), whoIsHandler(cdc, cliCtx, storeName)).Methods("GET")
}
```

### 查询处理程序

然后，开始建立我们前面提到的HTTP处理程序。这和前面定义CLI方法很相似。从`whois` 和 `resolve`查询开始：

```go
func resolveNameHandler(cdc *codec.Codec, cliCtx context.CLIContext, storeName string) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		paramType := vars[restName]

		res, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/resolve/%s", storeName, paramType), nil)
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusNotFound, err.Error())
			return
		}

		utils.PostProcessResponse(w, cdc, res, cliCtx.Indent)
	}
}

func whoIsHandler(cdc *codec.Codec, cliCtx context.CLIContext, storeName string) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		paramType := vars[restName]

		res, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/whois/%s", storeName, paramType), nil)
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusNotFound, err.Error())
			return
		}

		utils.PostProcessResponse(w, cdc, res, cliCtx.Indent)
	}
}
```

以上代码注解:
- 注意，我在这里同样采用`cliCtx.QueryWithData`函数获得数据
- 上面的函数和相应的CLI功能项基本一样

### Tx处理程序

现在定义`buyName` and `setName`交易处理程序路径

```go
type buyNameReq struct {
	BaseReq utils.BaseReq `json:"base_req"`
	Name    string        `json:"name"`
	Amount  string        `json:"amount"`
	Buyer   string        `json:"buyer"`
}

func buyNameHandler(cdc *codec.Codec, cliCtx context.CLIContext) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		var req buyNameReq
		err := utils.ReadRESTReq(w, r, cdc, &req)
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		baseReq := req.BaseReq.Sanitize()
		if !baseReq.ValidateBasic(w) {
			return
		}

		addr, err := sdk.AccAddressFromBech32(req.Buyer)
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		coins, err := sdk.ParseCoins(req.Amount)
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		// create the message
		msg := nameservice.NewMsgBuyName(req.Name, coins, addr)
		err = msg.ValidateBasic()
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		utils.CompleteAndBroadcastTxREST(w, r, cliCtx, baseReq, []sdk.Msg{msg}, cdc)
	}
}

type setNameReq struct {
	BaseReq utils.BaseReq `json:"base_req"`
	Name    string        `json:"name"`
	Value   string        `json:"value"`
	Owner   string        `json:"owner"`
}

func setNameHandler(cdc *codec.Codec, cliCtx context.CLIContext) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		var req setNameReq
		err := utils.ReadRESTReq(w, r, cdc, &req)
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		baseReq := req.BaseReq.Sanitize()
		if !baseReq.ValidateBasic(w) {
			return
		}

		addr, err := sdk.AccAddressFromBech32(req.Owner)
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		// create the message
		msg := nameservice.NewMsgSetName(req.Name, req.Value, acc)
		err = msg.ValidateBasic()
		if err != nil {
			utils.WriteErrorResponse(w, http.StatusBadRequest, err.Error())
			return
		}

		utils.CompleteAndBroadcastTxREST(w, r, cliCtx, baseReq, []sdk.Msg{msg}, cdc)
	}
}
```

以上代码注解:
- [`BaseReq`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/utils#BaseReq) 函数包含了完成交易需要的基本项（用哪个key，如何编码，应用在那个区块链上，等。。。）并且如上面展示的那样嵌入到程序中
- `baseReq.ValidateBasic` 和 `utils.CompleteAndBroadcastTxREST`为您设置响应代码，使用这些函数您不需要为处理操作错误或者成功而担心。 
  
### 现在你的模型已经有了与Cosmos SDK应用[合并所需要的所有东西(./app-complete.md)!
