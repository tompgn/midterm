## Digital Signatures

* Near supports accounts derived from a public/private key scheme based on either curve SECP256K1 or Ed25519 (dalek library).
* The support of SECP256K1 is useful to maintain some compatibility layer with bitcoin/ethereum.
* Most protocol operations effectively require Ed25519 to be used.

### Accounts

* Near hides the address information and instead uses human-readable account IDs similar to what the ENS provides on ethereum, within the __.near__ domain (e.g. alice.near).
* Several public/private keypairs can be associated to an account.
* _Full access_ keys allow for all operations to be performed, in particular token transfers.
* _FunctionCall_ keys that can only call certain functions and cannot use more than a predefined token allowance.

## Hash functions

* Near uses SHA2-256 for most of the chain data structures.
* This hash function is battle tested, but slow and arguably less secure than Blake2/3 or Keccak variants.
* Blake2b is used in the new randomness beacon production. It is unclear why they do not leverage the speed of this hash function within their data structures too.

### Merkle-Patricia Tries

Near uses various Merkle-Patricia tries to record the chain state.

<p align = "center">
<img src = "images/mt.svg">
</p>
<p align = "center"><b>
The Merkle path for datum #2 consists of the datum node hash (purple) together with the path itself in green and the root.
</b></p>
<p align = "center"><b>
The Merkle proof for datum #2 consists of the datum node hash (purple) together with the path conodes in pink and the root.
</b></p>

//new slide maybe?

* The merkleized structure ensures that the state cannot be tampered with without changing the root of the tree.
* The tree roots are included into the blocks, producing a compact commitment for the whole state data.
* A short ($log(n)$ in a balanced trie, where $n$ is the number of nodes) inclusion proof can be produced for every datum(or node in the tree).
* In practice the state in Near is represented by multiple Merkle tries. And all the receipts, challenges transaction outcomes, etc. are also committed to the blocks through a Merkle trie root.

### Hash lists

* The chain itself is a hash list, where each block refers to its parent by including the parent's header hash in its own header. 
* It ensures that an older block cannot be tampered with without invalidating all the subsequent blocks.

<p align = "center">
<img src = "images/hl.svg">
</p>

## Erasure coding

* Erasure coding is baked into the consensus itself to enshrine data availability.
* Each validator of a shard sends erasure coded data to validators from other shards.
* The scheme used ensures that only $1/3^\textrm{rd}$ of the validators can reconstruct data for all shards.
* The data availability is enforced at consensus level, by forcing validators to check whether the data is available before they accept a block.

## Signature Agregation: BLS signatures and Multisignatures

* Verifying the signatures of all validators for a block would be prohibitively expensive.
* Instead, __BLS signatures__ are used, and block producers are collecting those signatures and aggregate them with BLS aggregation.
* The block producer then publishes the aggregated signature together with a Merkle root of the individual signatures organized into a Merkle tree, and signs this using a cheap ECDSA scheme.
* When syncing the chain, participants can then choose to skip the validation of all individual signatures and verify only the ECDSA signature from the validator, relying on the fact that he has neither been challenged and slashed.
* Optionally, Schorr aggrgation could be present into a block together with the BLS aggregation for easier compatibility with chains like Ethereum.

## Verifiable random functions

* Near's new random beacon makes use of a VRF to generate randomness that's both unpredictable and unbiasable.
* Their new randomness beacon is based on BLS signatures, and allows any given subset $k < n$ of $n$ nodes to produce randomness without any node set of size $k-1$ being able to learn any information about the randomness beacon.
* In practice, Near choses $k = (2/3) . n$ to increase the difficulty for a subset of malicious and colluding participants to reveal the randomness, as long as at least $1/3^\textrm{rd}$ of the participants are honest.

## Succing Non-interactive ARguments of Knowledge

* The current implementation of Near still relies on fishermen to ensure the security of shards (refer to the description of the consensus). 
* This is a __fraud proof approach__, that is cheap and easy to implement but bears the downside of negatively impacting liveness and speed of the protocol due to the challenge period, as well as introducing the questions about incentivization of the fishermen.
* Near is planning to switch to a __validity proof approach__ where each chunk producer would produce a zero knowledge succint proof attesting of the chunk validatity.

#### What We Like

1. Near uses (or has a roadmap for the use of) a full set of powerful and bleeding edge primitives. This allows them to remain competitive with the best technical projects in the space, even though exotic cryptographic primitives may be a double-edge sword as they are less battle-tested.
2. Near's use of erasure coding to solve the data availability issue, over the use of some set of notaries that are trusted to keep long term data.
3. The new randomness beacon based on threshold randomness that allows the protocol to use unbiasable and unpredictable randomness for every validator selection, as opposed to round-robin based mathods that could be biased by non-revealing.
4. The use of BLS aggregation of the signatures, which is elegant and efficient for a large number of validators. The use of Schnorr to ensure maximal compatibility with popular chains like Ethereum.

#### What We Don't Like

1. Most of the structures rely on SHA2-256 as a hash function, which is older and slower than Blake2b, while Blake is already used by the protocol for Hash-to-point functions in the new randomness beacon implementation.
2. Unlear path towards the use of zkSNARKs to provide security based on validity proofs as opposed to fraud proofs. The current challenge/response mechanism affects the protocol liveness and can be gamed by using briberies to modify the incentives to report fraud.
3. At the moment, the protocol is in a phase where some of the design decisions are half baked: the use of BLS aggregation is nice as it would allow for a potentially very large pool of validators to get their signatures aggregated and then validated cheaply. However, with only 100 validators, at the time of writing, it may not really be optimal. Yet if the number of validators increases, the Schnorr interactive aggregation phase may become unpractical on the other hand.
