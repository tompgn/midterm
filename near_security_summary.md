# Byzantine generals problem

* _In a distributed environment where participants don't know each other, a consensus needs to be reached even in the presence of a set of unknown malicious actors, without the help of a central authority._
* Nightshade is effectively the part that addresses the problem of the Byzantine generals, and like many proof of stake finality mechanisms, relies on a logic founded on PBFT
* Nightshade, provides similar guarantees as PBFT, but also suffers the same drawbacks
  1. The protocol is secure as long as $2/3^\textrm{rd}$ of the validators are honest, but a complexity of $O(n^2)$ in message passing for $n$ of validators.
  2. If $1/3^\textrm{rd}$ of the participants of the protocol drop out, the protocol stalls.
* The segregation of the block production part (Doomslug) from the finality part (Nightshade) preserves liveness and still offers guarantees of PBFT for users happy to accept the longer finality period.

# Sybil attacks

* _In a pseudonymous environment, nothing prevents an adversary to create multiple accounts._
* The block production mechanism, in which at least $51\%$ of the block producters need to be honest
* The finality gadget, which requires a threshold of $2/3^\textrm{rd}$ honest validators to be online.
* This makes it crucial to prevent a malicious actor to secure many validator slots.
* Near relies on the fact that a sufficient amount of tokens need to be bounded for a validator to get a slot.
* As long as the token maintains sufficient value, it is expensive to create accounts that are funded enough to participate to the protocol.

# $51\%$ Attacks

> **NB:_**  By $51\%$ attack we mean the procedure bywhich an adversary could take over the network, not necessarily that a threshold of $51\%$ would be required to do so, as we know that the real threshold in most blockchains is in fact lower, e.g. selfish mining in bitcoin etc.

* Near uses a proof-of-stake algorithm where the security of the network depends on the amount of tokens that get staked (bonded) by the validators and the validator candidates.
* The security of the system can then be assessed in terms of a financial quantity in FIAT currency (e.g. USD) depending on the price of the token.

## Finality

* In Near, any attack that takes over the finality mechanism is fatal.
* Nightshade requires $2/3^\textrm{rd}$ of the validators to be honest to finalize a block.
* The theoretical security threshold is half in case of a network partition of the honest validators (c.f. PBFT).
* __The theoretical security is then equal to the cost of taking other the $1/3^\textrm{rd}+1$ least bonded validators.__

## Block production

* Doomslug ensures the chain will go on as long as at least $51\%$ of the validators are online to endorse a block.
* A malicious actor with $51\%$ of the stake, could produce an invalid block and get it endorsed, but it would not be sufficient to get it finalized.
* Such a block producer would be exposed to fishermen submitting fraud proofs to get him slashed.
* Before finalization the security level is equal to the stash value of the block producer.
* After finalization, it is equal to $1/3^\textrm{rd}$ of the stake.
* The liveness and security of the protocol can also be badly impacted by the fact that there is no punishment for being offline during block production, other than the loss of the block reward.

## Shard security

* Within one shard, if the majority of the validators gets corrupted, the shard itself can be fully taken over.
* Validators on other shard are only operating a light-client equivalent verification on the corrupted shard.
* Inter-shard transactions from a corrupted shard are dangerous and propagate state corruption.
* Fishermen are incentivized to publish onchain challenges attesting from the invalidity of a given block shard.
* In case of a successful challenge, the state is reverted to the one just before the shard corruption.

## Adaptive corruption

* _The possibility for a malicious party to bribe or corrupt only a specific subset of the actors in order to take over the network, or grief other participants._
* A randomness beacon to assign validators to shards makes adapting corruption more difficult.
* "Hidden validators" whose assignment to any given shard is not meant to be released publicly further prevent adaptive behavior.
* At the main chain level, block producers are operating in following a simple round robin
* This leaves them open to adaptive attacks!
* Validators are rotated once every epoch (15 hours), so any block that would be challenged within that period would let the set of validators exposed for the rest of the epoch.

## More practical considerations

* Near currently operates only on one shard and with 100 validators.
* Nothing is put in place to ensure that the stake are evenly distributed so there are huge differences between stakes
* The most bonded validator getting bonded for 25 000 000 Near tokens and the least bonded being worth only 180 000 Near tokens.
* Taking over the least slots relatively cheap if considering the security of a 4B USD network (at the time of writing, the price of the Near token is around 4.2USD)
* Taking over the last 34% of the validators and being in a theoretically able to take over the whole network requires only 11.5M Near tokens, which is less than 50M USD.

#### What We Like

1. Near is aware of and has addresses the different security challenges posed by proof-of-stake consensus based blockchains, albeit not always in an optimal manner.
2. Consensus separation in block production and finality that favours liveness while still maintaining PBFT security for mission critical usecases that can trade a higher speed for added security.
3. The problem of shard corruption spreading out being addressed through a clever use of randomness and hidden validators, with Fishermen being the end guardians.

#### What We Don't Like

1. Nothing is done to ensure the stakes of the validators are homogenized, leading to as system where the worst case security bound is practically very low even though the total bonded is reasonably high.
2. The use of round robin for the main block production, allowing for a malicious actor to adaptively bribe, grief, DDos, of more generally attack validators following their turns.
3. In case a bad block is challenged, the validators would be uncovered till the end of the epoch.
4. No offline penalties apart from the missing reward.