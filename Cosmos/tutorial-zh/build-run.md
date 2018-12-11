# 构建和执行应用程序

## 构建 `nameservice` 应用程序

如果您想在这里构建`nameservice`应用，并使用它提供的功能，您首先需要安装`dep`.

> _*注释*_: 如果您想在您自己的代码仓库上开始建立此应用，您需将前面所写的代码倒入您的代码仓库。本教程设置的代码倒入路径是(`github.com/cosmos/sdk-application-tutorial`)，如果你是在您自己的代码仓库上开发，那么您需要先将代码倒入的路径设置为您代码仓库的路径(`github.com/{ .Username }/{ .Project.Repo }`)，然后再编译您的应用。 

```bash
# Initialize dep and install dependencies
make get_tools && make get_vendor_deps

# Install the app into your $GOBIN
make install

# Now you should be able to run the following commands:
nsd help
nscli help
```

## Running the live network and using the commands

To initialize configuration and a `genesis.json` file for your application and an account for the transactions, start by running:

> _*NOTE*_: In the below commands addresses are are pulled using terminal utilities. You can also just input the raw strings saved from creating keys, shown below. The commands require [`jq`](https://stedolan.github.io/jq/download/) to be installed on your machine.

```bash
# Initialize configuration files and genesis file
nsd init --chain-id testchain

# Copy the `Address` output here and save it for later use
nscli keys add jack

# Copy the `Address` output here and save it for later use
nscli keys add alice

# Add both accounts, with coins to the genesis file
nsd add-genesis-account $(nscli keys show jack --address) 1000mycoin,1000jackCoin
nsd add-genesis-account $(nscli keys show alice --address) 1000mycoin,1000aliceCoin
```

现在您可以通过执行`nsd start`来启动`nsd`,您将看到日志开始流出，这表示区块已经开始产生。这个过程可能需要等待数秒钟时间。 

打开另一个终端来运行您刚刚创建的网络。 

```bash
# First check the accounts to ensure they have funds
nscli query account $(nscli keys show jack --address) \
    --indent --chain-id testchain
nscli query account $(nscli keys show alice --address) \
    --indent --chain-id testchain

# Buy your first name using your coins from the genesis file
nscli tx nameservice buy-name jack.id 5mycoin \
    --from     $(nscli keys show jack --address) \
    --chain-id testchain

# Set the value for the name you just bought
nscli tx nameservice set-name jack.id 8.8.8.8 \
    --from     $(nscli keys show jack --address) \
    --chain-id testchain

# Try out a resolve query against the name you registered
nscli query nameservice resolve jack.id --chain-id testchain
# > 8.8.8.8

# Try out a whois query against the name you just registered
nscli query nameservice whois jack.id --chain-id testchain
# > {"value":"8.8.8.8","owner":"cosmos1l7k5tdt2qam0zecxrx78yuw447ga54dsmtpk2s","price":[{"denom":"mycoin","amount":"5"}]}

# Alice buys name from jack
nscli tx nameservice buy-name jack.id 10mycoin \
    --from     $(nscli keys show alice --address) \
    --chain-id testchain
```

### 恭喜您已经构建了一个Cosmos SDK应用！我们的教程也即将结束。如果您希望了解如何采用REST服务来运行相同的命令, 可以[双击这里](./run-rest.md)。 

