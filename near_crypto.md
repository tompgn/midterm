## Digital Signatures

The accounts are protected by the use of digital signatures. Near supports accounts derived from a public/private key scheme based on either curve SECP256K1 or Ed25519 (dalek library).
The support of SECP256K1 is useful to maintain some compatibility layer with bitcoin/ethereum, but most of the chain operations - such as staking - only support Ed25519 accounts.
Those digital signatures are essential for authenticating that the origin of a transaction spending funds or authoring a block, for instance, really is in charge of the account allowed to perfom this transaction, hence in control of the private key.
Since it is computationally infeasible to compute the private key from the public information (one-wayness), anybody able to sign a transaction with a give account as origin has to be in control of some private key linked to this account.

### Accounts

Unlike many other blockchain projects, the accounts are not identified primarily by public key hashes that represent an address. Instead, Near defines the notion of an "account ID" that's a dot-separated, human-readable, string of characters, formatted in a similar way as URLs are. They also follow a hierarchy where postfixes represent a higher domain than prefixes.
For instance, the "near" domain is active on the mainnet, and allows for accounts of the type: "alice.near". The owner of a given account ID could then define sub-accounts by prefixing them, e.g. tipping.alice.near could be used by Alice only to tip her friends.
Human readable accounts are a very nice feature of the protocol that improves usability by hiding addresses and obscure byte strings from users.
However, it comes with various costs: in particular, there is no direct cryptographic binding between a given public/private keypair and the account ID, without access to onchain data, unlike what's possible with addresses, that can be all derived offchain from a given keypair. Breaking this relationship also breaks the possibility to easily accommodate primitives like hierarchical key derivation schemes.
Last the main drawback is maybe not so much cryptographic, but social: while it could appear nice to reserve account IDs following a given name on the near domain, the set of name is limited and cannot be re-used, forcing late users to rely on account IDs that - although human-readable - would be harder to relate to a given individual.
On the contrary, public key hashes are Global and Unique IDs that can then be linked to a human readable identifying string through a registrar. This combines the advantages of both uniqueness with human-readability, even though it forces the user to perform the registration.

Since the link between account IDs and keypairs is defined onchain, rather by cryptographic hashing, it becomes possible to link multiple keypairs to a given account ID[[1]](#1). By default, Near defines a full-access key that can perform any operation on an account, including sending funds. It is possible for users to define one or many so-called "FunctionCall" keys that are registered to perform calls to given functions only, with specific token allowances. This is not dissimilar to the distinction that substrate makes between stash and controller keys, excepted that the FunctionCall keys are more specialised hence that many of them can be defined depending on the functions to be called as part of a given users routine and the allowances needed.

## Hash functions

Like any blockchain, Near makes extensive uses hash functions and hash-based data structures.
For most of its operations, Near relies on a single hash function, chosen to be SHA2-256, which is used for most of the primitives including hashing for the Merkle tree representations, hasing of contract code, etc.
They also use blake2b in a specific case related to how they generate a randomness beacon. This is discussed in more details below.
The possible downsides of the choice for SHA2-256 as the main hash function This is not as efficient as the Blake2/Blake3 family and even more compared to algeabraic hashing functions in case the cryptographic guarantees are not needed.
It is also an older hash function that doesn't likely provide the same level of security as Blake2/3 or Keccak, but the bitcoin network in particular has been instrumental in popularizing implementation and hardware research for this specific hash function, in a way that probably makes it the most standard to date.

Cryptographic hashing is at the base of two sets of data structures in Near:

### Merkle Tries

### Hash lists

Besides those basic primitives that can be found at the foundation of most blockchains, Near uses also a set of bleeding edge primitives.

## Erasure coding

In Near, erasure coding is baked into the consensus itself to enshrine data availability: each validator of a shard sends erasure coded data to validators from other shards. This part is crucial to ensure that if a given shard gets taken over by a malicious set of actors, the rest of the chain remains able to detect that the proofs are fake.
The scheme used ensures that only $1/3^\textrm{rd}$ of the validators can reconstruct data for all shards, which remains robust in case $1/3^\textrm{rd}$ of the nodes are split from the network and another $1/3^\textrm{rd}$ is malicious.
The data availability is enforced at consensus level, by forcing validators to check whether the data is available before they accept a block.

## Verifiable random functions

Near's new random beacon makes use of a VRF to generate randomness that's both unpredictable and unbiasable[[2]](#2). A reliable randomness beacon is an important piece of tool of modern blockchains, and can be used both in the consensus level (e.g. to select validators in an unpredictable way, preventing them to get targeted in advance by malicious actors), or as a tool for higher level applications, for instance as a seed to be used whenever smart contracts require some unbiasable and unpredictable source of randomness.

Their new randomness beacon is based on BLS signatures, and allows a given threshold $k$ of nodes to produce unpredictable and unbiasable randomness without any set of $k-1$ nodes being able to learn any information about the randomness beacon.
Those properties are trivial consequences of the features of BLS signatures and cryptographic pairings, that require a distributed key generation protocol.
The way this protocol is conducted is worth some note: Near is relying on the linearity of polynomials to aggregate point evaluations of polynomials of degree $k$ by each participants into point evaluations of a sum polynomial of degree $k$ that the protocol agrees on. However there is a caveat in that each recipient of an evaluation from a validator is responsible for checking its correctness and challenge it within a given time period (half a day, or one epoch) if incorrect. This is weaker than directly enforcing the correctness of the resulting polynomial onchain (see [[2]](#2)).
The distributed key generation protocol is conducted once per epoch and the VRF is jointly evaluated at every block height to generate a random number.

# References

<a id="1">[1]</a> https://www.near-sdk.io/zero-to-hero/beginner/logging-in
<a id="2">[2]</a> https://near.org/blog/randomness-threshold-signatures/