# 构建应用

## `Makefile`

首先，让我们帮助开发者写一个包含常用命令的`Makefile`来构建您的应用：

> _*注释*_: 下面的Makefile文件中包含了一些命令，也用于Cosmos SDK 和 Tendermint Makefile中.

```make
DEP := $(shell command -v dep 2> /dev/null)

get_tools:
ifndef DEP
	@echo "Installing dep"
	go get -u -v github.com/golang/dep/cmd/dep
else
	@echo "Dep is already installed..."
endif

get_vendor_deps:
	@echo "--> Generating vendor directory via dep ensure"
	@rm -rf .vendor-new
	@dep ensure -v -vendor-only

update_vendor_deps:
	@echo "--> Running dep ensure"
	@rm -rf .vendor-new
	@dep ensure -v -update

install:
	go install ./cmd/nsd
	go install ./cmd/nscli
```

## `Gopkg.toml`

Golang提供了一些依赖性管理工具。本教程使用[`dep`](https://golang.github.io/dep/)来完成代码依赖性管理。`dep` 使用位于代码仓库根目录下的`Gopkg.toml`文件来定义应用所依赖的包。当前，Cosmos SDK依赖于一些特定版本的包，下面的清单包含了所须的版本。 让我们通过`constraints` 和 `overrides`命令来替换`./Gopkg.toml`下的内容。 

```toml
# Gopkg.toml example
#
# Refer to https://golang.github.io/dep/docs/Gopkg.toml.html
# for detailed Gopkg.toml documentation.
#
# required = ["github.com/user/thing/cmd/thing"]
# ignored = ["github.com/user/project/pkgX", "bitbucket.org/user/project/pkgA/pkgY"]
#
# [[constraint]]
#   name = "github.com/user/project"
#   version = "1.0.0"
#
# [[constraint]]
#   name = "github.com/user/project2"
#   branch = "dev"
#   source = "github.com/myfork/project2"
#
# [[override]]
#   name = "github.com/x/y"
#   version = "2.4.0"
#
# [prune]
#   non-go = false
#   go-tests = true
#   unused-packages = true

[[constraint]]
  name = "github.com/cosmos/cosmos-sdk"
  version = "v0.27.0"

[[override]]
  name = "github.com/golang/protobuf"
  version = "=1.1.0"

[[constraint]]
  name = "github.com/spf13/cobra"
  version = "~0.0.1"

[[constraint]]
  name = "github.com/spf13/viper"
  version = "~1.0.0"

[[override]]
  name = "github.com/tendermint/go-amino"
  version = "v0.14.1"

[[override]]
  name = "github.com/tendermint/iavl"
  version = "=v0.12.0"

[[override]]
  name = "github.com/tendermint/tendermint"
  version = "v0.27.0-dev1"

[[override]]
  name = "golang.org/x/crypto"
  source = "https://github.com/tendermint/crypto"
  revision = "3764759f34a542a3aef74d6b02e35be7ab893bba"

[prune]
  go-tests = true
  unused-packages = true
```

> _*注意*_: 如果您是在自己的代码仓库上从零开始建立应用，在运行`dep ensure -v`命令前，请确认您已经将此教程的导入目录`github.com/cosmos/sdk-application-tutorial`替换成您的代码仓库的目录（比如 `github.com/{ .Username }/{ .Project.Repo }`)。如果没有，您需要先删除`./vendor`目录(`rm -rf ./vendor`)和`Gopkg.lock` 文件 (`rm Gopkg.lock`)，然后重新运行`dep ensure -v`。 

架构已经建好，现在开始安装dep，以及相关的依赖包：

```bash
make get_tools
dep init -v
```

## 构建应用

```bash
# Update dependencies to match the constraints and overrides above
dep ensure -update -v

# Install the app into your $GOBIN
make install

# Now you should be able to run the following commands:
nsd help
nscli help
```

### 祝贺您已经完成了nameservice应用的开发！让我们来[运行并且体验和它的交互工作](./build-run.md)吧！
