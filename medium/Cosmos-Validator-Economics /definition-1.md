### I. Cosmos Economic Design

Cosmos is an upcoming network of blockchains that could revolutionize the blockchain landscape by providing much-needed scalability and interoperability. The Cosmos Network will be secured jointly by the holders of atoms (the staking token of Cosmos) and validators, who will run consensus nodes to validate transactions and maintain network consensus.

#### Definitions:

We subsequently define key parts of the Cosmos system, as even people with a good understanding of Cosmos may not be aware of the current plans. We start by breaking down key components of the economic design.

#### Cosmos Hub

The Cosmos Hub is the first blockchain of the Cosmos Network. The hub will connect to many other blockchains, effectively functioning as a light client of those chains. This will allow token transfers across different chains to be routed via the hub.

#### Atoms

The atom is the native token of the Cosmos Hub. The core utility of atoms is to act as a staking mechanism to secure the Hub as deposits in the staking process. You can think of the atom as a virtualized ASIC of sorts, to loosely frame it in Proof-of-Work mining terms. The amount of atoms staked towards a validator defines the frequency by which the validator may propose a new block and its weight in votes to commit a block. For an deep dive into how weighted voting for validators works, read [this blog post](https://blog.cosmos.network/tendermint-explained-bringing-bft-based-pos-to-the-public-blockchain-domain-f22e274a0fdb). In return for bonding (staking) atoms with their chosen validator, an atom holder, who has delegated (see below) becomes eligible for block rewards paid in atoms and photons, as well as transaction fees paid in any of the whitelisted tokens (see below).

#### Validators

Validators secure the Cosmos Hub by validating and relaying transactions, proposing, verifying and finalizing blocks. Validators can stake their own atoms or be delegated tokens from other atom holders. There will be a limited number of validators, initially 100, who will be individually required to operate highly reliable automated signing infrastructure. Validators must also keep their validation keys secure while connected to the P2P network to sign blocks. Validators will be able to charge delegators a commission in atoms for their work in securing the network.

#### Delegators

Delegators of the Cosmos Hub are holders of the staking token, atoms, who use some or all of their atoms to secure the Cosmos Hub. There is no minimum amount of atoms required in order to stake atoms. They do so by selecting one or more validators and delegating their voting power to them by putting up atoms as collateral. In the case of misbehavior by the validator (for example, signing two different blocks at the same block height), part of the collateral deposited by both the errant validator and delegator will get slashed. In return, delegators can earn a proportion of the transaction fees as well as block rewards.

#### Inflationary Atoms

New atoms are created every block and distributed to validators and delegators participating in the consensus process. This provides an incentive to atom holders to not just passively hold their tokens in wallets, but to put them at risk in order to secure the network. The number of new atoms created per block is variable and depends on the percentage of the atom supply that is staked in the network. The target rate of atoms put up as collateral to secure the network is at least ⅔ of the total atom supply. If less atoms are staked, atom supply via block rewards increases up to a ceiling of 20% annualized inflation of the total supply. If more than ⅔ are being staked, atom block rewards decrease gradually down to a floor of 7% annualized inflation.

#### Transactions Fees

Transactions on the Cosmos Network will be subject to transaction fees similar to other existing blockchains. Unlike current blockchains, the Cosmos Network plans to accept a multitude of tokens for the purpose of payment of transaction fees (see whitelisting tokens below for more information). Resulting transaction fees, minus a network tax that goes into a reserve pool, are split among validators and delegators based on their stake. Funds from the reserve pool are planned to be used to increase the security and value of the Cosmos Network and may also be distributed in accordance with decisions from the governance system.

#### Governance

The Cosmos Network employs on-chain governance by letting token holders vote on proposals that can adjust parameters of the system, implement upgrades or change the written constitution governing the Cosmos Hub. (No written constitution governing the Cosmos Hub has yet been implemented.) Each zone in the network can have its own constitution and governance mechanism. Delegators implicitly take part in the voting process by inheriting the vote of the validator they are delegating to. Delegators are given an option to override the vote of the validator they delegated atoms to for a vote on a specific issue.

