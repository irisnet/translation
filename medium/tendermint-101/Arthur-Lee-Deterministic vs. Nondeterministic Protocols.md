# 确定性 与 非确定性 协议

在完全异步的情况下解决共识问题的非确定性协议潜在地依赖于随机预言和普遍地导致信息复杂度的开销，因为它们所有的通信都依赖于可靠的广播。在异步环境下，单个可靠的广播的开销大约和[Tendermint中的一轮](http://tendermint.readthedocs.io/projects/tools/en/master/introduction.html#consensus-overview)的花费是相当的。像HoneyBadger拜占庭容错这样的很多协议属于在异步环境下的非确定性协议一类。通常，它们要求在单轮的通信中要有三个可靠的广播实例。

相反Tendermint是一个完全确定性的协议；无论什么情况下在这个协议中都没有随机性。通过实现中的一个确定的数学函数，领导是很明确的被选举出来的。这样，我们能够在数学上证明这个系统是活的，并且这个协议能够保证做出决策。

## 轮流的领导选择制度

Tendermint 以加权的轮询方式在验证者集合(如区块的提案者们)中循环。一个验证者代理的其他人的权益(例如投票权)越多，它就有更多的权重，并且相应地它就会被更多的选为领导者。具体来说明一下，如果一个验证者和另一个验证者有着同样的投票权利，它们都会被协议以同样的次数选中。

### 对于这个算法如何工作的最简单的解释如下：

1.验证者的权重被设置

2.验证者被选择，轮到验证者来提议一个区块

3.权重被重新计算，在本轮结束后减少一定数量的权重

4.随着每一轮的进行，权重按照投票权利的相应比例逐渐递增

5.再一次选择验证者

. 这里是实际对应的代码片段[GitHub](https://github.com/tendermint/tendermint/blob/master/types/validator_set.go#L50)

因为[协议可以很明确地选择区块的提案人](https://github.com/tendermint/tendermint/blob/master/docs/specification/new-spec/reactors/consensus/proposer-selection.md)，鉴于你知道验证者集合和每个验证者的投票权重，你可以在x, x + 1,...,x + n 轮次中准确地计算出谁会是下一个区块的提案者。因此，有评论者争论说Tendermint去中心化的还不够。当你可以预知谁会是领导者的时候，一个攻击者可以以这些领导者为目标并对他们发动DDoS攻击，而且很有可能使得这个链停止向前发展。我们通过在Tendermint中实现某个叫做[哨兵架构](https://github.com/tendermint/tendermint/blob/master/docs/spec/p2p/node.md)的东西来减轻这个攻击路径的影响。
