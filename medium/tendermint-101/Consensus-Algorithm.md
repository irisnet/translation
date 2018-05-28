### Consensus Algorithm

#### Partially Synchronous, Synchronous, and Asynchronous Communication

Let’s explore the synchronous case, using a well-known protocol for reference: the Bitcoin protocol. In Bitcoin, there is a “known fixed upper bound”. This is referring to the 10 minute block time. In order for the Bitcoin network to move forward with creating blocks, the protocol artificially imposes a timing assumption, giving the entire network of nodes 10 whole minutes to listen for, collect, validate, and gossip transactions propagating across its peer network.

Ethereum is another example of a protocol that makes synchrony assumptions with block times averaging 15 seconds. While 15 seconds is much faster than 10 minutes — giving the Ethereum network the benefit of higher throughput — this comes at the expense of its miners, as this results in more orphaned blocks; there is less time for transactions to propagate through its network.

Tendermint belongs to a class of protocols which solve consensus under partially synchronous communication, wherein, a partially synchronous system model alternates between periods of synchrony and asynchrony; we sometimes refer to this model as “weakly synchronous”. What this means is, Tendermint does rely on timing assumptions in order to make progress. However, in contrast to synchronous systems, the speed of progress does not depend on system parameters, but instead depends on real network speed.

#### Liveness & Termination

Defined: “*The termination property is just that each correct processor should eventually make a decision.*”

Those algorithms that of Nakamoto consensus, Peercoin, NXT, Snow White, Ouroboros, etc., in the synchronous system model, rely on synchrony assumptions not just for process termination but for safety. Those algorithms designed for synchronous systems always have fixed bounds which are known and always hold. And in the event that the synchrony bounds do not hold, consensus is broken, and the chain will fork. Therefore, Bitcoin’s 10 minute block time, for example, is duly conservative, as to preserve safety.

By way of contrast, Tendermint never forks in the presence of asynchrony if less than 1/3 of processes are faulty. This property is what makes Tendermint a BFT-based PoS protocol, in which it strictly prefers safety over liveness (see: [CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem)). As a result, a Tendermint blockchain will halt momentarily until a supermajority, i.e. more than 2/3, of the validator set comes to consensus.

Now, there *are* consensus protocols that work in purely asynchronous networks, but, adhering to the FLP Impossibility Theorem, they cannot simultaneously be deterministic.