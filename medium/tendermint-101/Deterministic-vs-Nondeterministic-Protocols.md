#### Deterministic vs. Nondeterministic Protocols

Nondeterministic protocols which solve consensus under the purely asynchronous case potentially rely on random oracles and generally incur high message complexity overhead, as they depend on reliable broadcasting for all communication. In the asynchronous environment, the overhead cost of a single reliable broadcast is about equivalent to [one ](http://tendermint.readthedocs.io/projects/tools/en/master/introduction.html#consensus-overview)`round`[ of Tendermint](http://tendermint.readthedocs.io/projects/tools/en/master/introduction.html#consensus-overview). Protocols like HoneyBadger BFT fall into this class of nondeterministic protocols under asynchrony. Normally, they require three instances of reliable broadcast for a single round of communication.

Tendermint instead is a totally deterministic protocol; there is no randomness in the protocol whatsoever. Leaders are elected deterministically, through a defined mathematical function in the implementation. As such, we are able to mathematically prove that the system is live and that the protocol is guaranteed to make decisions.

#### Rotating Leader Election

Tendermint rotates through the validator set, i.e. block proposers, in a weighted round-robin fashion. The more stake, i.e. voting power, that a validator has delegated to them, the more weight that they have, and the proportionally more times they will be elected as leaders. To illustrate, if one validator has the same amount of voting power as another validator, they will both be elected by the protocol an equal amount of times.

**The simplified explanation of how the algorithm works looks like this:**

1. Validator weight is established
2. Validator is elected, their turn to propose a block
3. Weight is recalculated, decreases some amount after round is complete
4. As each round progresses, weight increases incrementally in proportion to voting power
5. Validator is selected again

- And here is the actual code snippet on: ([GitHub](https://github.com/tendermint/tendermint/blob/master/types/validator_set.go#L50))

Because [the protocol selects block proposers deterministically](https://github.com/tendermint/tendermint/blob/master/docs/specification/new-spec/reactors/consensus/proposer-selection.md), given that you know the validator set and each validator’s voting power, you could compute exactly who the next block proposers will be in rounds `x`, `x + 1`,...,`x + n`. Because of this, critics argue that Tendermint isn't decentralized enough. When you can know predictably who the leaders will be, an attacker could target those leaders and launch a DDoS attack against them and potentially halt the chain from progressing. We mitigate this attack vector by implementing something called [Sentry Architecture](https://github.com/tendermint/tendermint/blob/master/docs/spec/p2p/node.md) in Tendermint.