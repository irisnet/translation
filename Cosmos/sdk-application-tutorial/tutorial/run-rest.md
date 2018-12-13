# 运行REST服务路由

现在，您已经测试了你的CLI查询命令和交易，可以开始测试REST服务了。退出我们前面启动的`nsd`，并收集用户的地址。 

```bash
$ nscli keys show jack --address
$ nscli keys show alice --address
```

现在可以在另一个终端窗口中启动`rest-server`

```bash
$ nscli rest-server --chain-id testchain --trust-node
```

然后您可以构造并运行下列查询:

> 注意: 确定用您的密码和对应购买者/拥有者的地址替代下面地址!

```bash
# Get the sequence and account numbers for jack to construct the below requests
$ curl -s -k https://localhost:1317/auth/accounts/cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6
# > {"type":"auth/Account","value":{"address":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","coins":[{"denom":"jackCoin","amount":"1000"},{"denom":"mycoin","amount":"1010"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"A9YxyEbSWzLr+IdK/PuMUYmYToKYQ3P/pM8SI1Bxx3wu"},"account_number":"0","sequence":"1"}}

# Get the sequence and account numbers for alice to construct the below requests
$ curl -s -k https://localhost:1317/auth/accounts/cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl
# > {"type":"auth/Account","value":{"address":"cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl","coins":[{"denom":"aliceCoin","amount":"1000"},{"denom":"mycoin","amount":"980"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"Avc7qwecLHz5qb1EKDuSTLJfVOjBQezk0KSPDNybLONJ"},"account_number":"1","sequence":"1"}}

# Buy another name for jack
$ curl -XPOST -s -k https://localhost:1317/nameservice/names --data-binary '{"base_req":{"name":"jack","password":"foobarbaz","chain_id":"testchain","sequence":"2","account_number":"0"},"name":"jack1.id","amount":"5mycoin","buyer":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6"}'
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1242"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"2986","tags":[{"key":"YWN0aW9u","value":"YnV5X25hbWU="}]},"hash":"098996CD7ED4323561AC9011DEA24C70C8FAED2A4A10BC8DE2CE35C1977C3B7A","height":"23"}

# Set the data for that name that jack just bought
$ curl -XPUT -s -k https://localhost:1317/nameservice/names --data-binary '{"base_req":{"name":"jack","password":"foobarbaz","chain_id":"testchain","sequence":"3","account_number":"0"},"name":"jack1.id","value":"8.8.4.4","owner":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6"}'
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1242"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"1352","tags":[{"key":"YWN0aW9u","value":"c2V0X25hbWU="}]},"hash":"B4DF0105D57380D60524664A2E818428321A0DCA1B6B2F091FB3BEC54D68FAD7","height":"26"}

# Query the value for the name jack just set
$ curl -s -k https://localhost:1317/nameservice/names/jack1.id
# 8.8.4.4

# Query whois for the name jack just bought
$ curl -s -k https://localhost:1317/nameservice/names/jack1.id/whois
# > {"value":"8.8.8.8","owner":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","price":[{"denom":"STAKE","amount":"10"}]}

# Alice buys name from jack
$ curl -XPOST -s -k https://localhost:1317/nameservice/names --data-binary '{"base_req":{"name":"alice","password":"foobarbaz","chain_id":"testchain","sequence":"1","account_number":"1"},"name":"jack1.id","amount":"10mycoin","buyer":"cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl"}'
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1264"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"4509","tags":[{"key":"YWN0aW9u","value":"YnV5X25hbWU="}]},"hash":"81A371392B52F703266257D524538085F8C749EE3CBC1C579873632EFBAFA40C","height":"70"}
```

### 请求样式:

#### `POST /nameservice/names` BuyName请求的主体：
```json
{
  "base_req": {
    "name": "string",
    "password": "string",
    "chain_id": "string",
    "sequence": "number",
    "account_number": "number",
    "gas": "string,not_req",
    "gas_adjustment": "string,not_req",
  },
  "name": "string",
  "amount": "string",
  "buyer": "string"
}
```

#### `PUT /nameservice/names` SetName请求的主体:
```json
{
  "base_req": {
    "name": "string",
    "password": "string",
    "chain_id": "string",
    "sequence": "number",
    "account_number": "number",
    "gas": "string,not_req",
    "gas_adjustment": "strin,not_reqg"
  },
  "name": "string",
  "value": "string",
  "owner": "string"
}
```
