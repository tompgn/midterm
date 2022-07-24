Near has been designed with careful consideration of the threat model of blockchains in mind, and as such offers an attractive set of security guarantees. However practical considerations and shortcuts have sometimes been used and are weakening up the overall security design.

The security of the user account balances relies on digital signatures as most of the blockchains do, we already reviewed the different schemes supported and analyzed the way accounts are used and generated in part 1, dedicated to cryptography.
However, as soon as a user submits a transaction, the thread model changes since the transaction is then executed on behalf of the user by a distributed state machine, and it is crucial to ensure that all possible attacks or issues that could prevent a correct execution of the protocol are addressed. In this part, we are focusing only on this, and not on issues arising from a misuse of the protocol by a given user, or by the security of its personal setup.

The origin of the transaction is enforced by a digital signature associated with the account that initiated the transaction. When such transaction is submitted to the protocol, it can be included by a chunk producer and executed on a given shard. Many treat can affect a trustless system when performing those operations:

# Byzantine generals problem

In a distributed environment where participants don't know each other, a consensus needs to be reached even in the presence of a set of unknown malicious actors, without the help of a central authority.
This coordination problem is sometimes referred to as the _Byzantine generals problem_. Consensus protocols in blockchains are primarily designed to tackle it.
As previously reviewed, Near's consensus agreement relies on two different parts for their protocol: a block production mechanism called __"Doomslug"__ , which ensures reliable block candidates can get produced without too much overhead, and a finality gadget called __"Nightshade"__. Nightshade is effectively the part that addresses the problem of the Byzantine generals, and like many proof of stake finality mechanisms, relies on a logic founded on PBFT[[7]](#7). A subset of participants called "validators" is responsible for reaching a consensus on the state of the system at a given time, and those validators are untrusted and distributed. Nightshade, provides similar guarantees as PBFT, but also suffers the same drawbacks, mamely: the protocol is secure as long as $2/3^\textrm{rd}$ of the validators are honest, but relies on two rounds of message passing with a complexity on in $O(n^2)$ for a number $n$ of validators, making it increasignly difficult to keep a low block time as the number of validators gets higher.
More importantly, if $1/3^\textrm{rd}$ of the participants of the protocol drop out, the protocol stalls.
This is the main reason behind the separation of the block production function from the finality function: it allows for the chain to keep on producing blocks, albeit with lower security guarantees, and to finalize them independently when possible. If too many validators go offline, the finality of the chain would stall without being critical for the liveness. Users accepting to deal with a lower security threshold (only the block producer would get slashed in case of malicious behaviour) would be mostly unaffected, while others happy to wait for finalization could still submot their transactions and get them processed, providing they would wait for finalization to get triggered again.

# Sybil attacks

The protocol relies on the assumption that a sufficient number of participants are honest. This affects both the block production mechanism, in which at least $51\%$ of the block producters need to be honest, and the finality gadget, which requires a threshold of $2/3^\textrm{rd}$ honest validators to be online. This makes it crucial to prevent a malicious actor to secure many validator slots (in Near, block producers are chosen within the set of validators, so it is sufficient to ensure the set of validators is difficult to get corrupted). Yet, as the system is pseudonymous, nothing prevents an adversary to create multiple accounts.
To prevent Sybil attacks, Near relies on the fact that a sufficient amount of tokens need to be bounded for a validator to get a slot. As long as the token maintains sufficient value, it becomes expensive to create accounts that are funded enough to participate to the protocol.
At the moment of writing, the lower staked validator slot has got for 180 000 Near tokens, so around $750 000 equivalent.

# $51\%$ Attacks

> **NB:_**  By $51\%$ attack we mean the procedure bywhich an adversary could take over the network, not necessarily that a threshold of $51\%$ would be required to do so, as we know that the real threshold in most blockchains is in fact lower, e.g. selfish mining in bitcoin etc.

Near uses a proof-of-stake algorithm where the security of the network depends on the amount of tokens that get staked (bonded) by the validators and the validator candidates. The security of the system can then be assessed in terms of a financial quantity in FIAT currency (e.g. USD) depending on the price of the token at a given point in time.

As the proof-of-stake protocol design in Near relies on different algorithms with different properties, we can look at them individually to assess the security of all parts, then most critical applications should base their threat model on the weakest part of the protocol, while less important or financially attractive transactions could be happy to consider a less conservative tradeoff.

## Finality

In Near, any attack that takes over the finality mechanism is fatal, therefore it is sufficient to take over the nightshade finality algorithm to fully compromize the chain. As previously discussed, Nightshade offers the same guarantees as PBFT, which means that $2/3^\textrm{rd}$ of the validators need to be honest to finalize a block. However, we know that the theoretical security threshold is half of this since, in case of a perfect network partition of honest validators, $1/3^\textrm{rd}+1$ malicious actors could join any of the two (remaining $1/3^\textrm{rd}$) parts of the segregated honest validator pool and be in a majority there.
__The theoretical security is then equal to the cost of taking other the $1/3^\textrm{rd}+1$ least bonded validators.__

## Block production

As previously exposed, the doomslug algorithm is ensuring the chain will go on as long as at least $51\%$ of the validators are online to endorse a block, since each block producer needs to collect sufficient endorsement messages (or skipping messages if the previous block was not produced) to be allowed to produce its own one. It means that a block producer controlling less than $51\%$ of the stake would not be allowed to produce blocks that are accepted by other participants. Over $51\%$ of the stake, a malicious block producer could produce an invalid block and get it endorsed, but it would not be sufficient to get it finalized, since $67\%$ of the validators are then required to finalize it. The block producer would then be exposed to fishermen that submit fraud proofs and get him slashed. Before finalization, the level of security of a single block is effectively only equal to the stash value of the block producer. After finalization, it is equal to $1/3^\textrm{rd}$ of the stake (since no mechanism is ensuring the stake of different validators is homogeneous), the lower bound is actually only the total stake of the least bonded validator. 

## Shard security

Near is a shared blockchain (although at the time of writing, the network still operates on a single shard, we analyze the situation from a theoretical standpoint following the specifications in the nightshade paper XOX as if multiple shards were in use), so the security guarantees are different.
Within one shard (defined in Near as a sequence of chunks within the main block), if the majority of the validators gets corrupted, the shard itself can be fully taken over, and validators could endorse an invalid state. Effectively, other validators on other shard are only operating a light-client equivalent verification on the corrupted shard, so they run the risk of getting an inter-shard transactions coming from a corrupted shard, spreading the state corruption to other shards and eventually to the whole network.
Cross-chain communication security is handled by fishermen, that should be able to detect and produce a proof that a given chunk is invalid. However, most participants don't maintain the state for all shards, so it should be ensured that sufficient information is available to produce such proofs.
To address this problem, Near limits the cumulative state that a transaction can read or write to a small number of bytes $L_s$ and forces every chunk producer to decompose the execution of transactions into intermediate states that are less than or equal to $L_s$. The fishermen only need to hold $L_s$ bytes of the state from the mismatching state root to produce a challenge.
In order to provide instant cross-shard (cross-chunk) transactions, Near optimistically allows for receipts to be executed without waiting for the challenge period to expire, at the cost of having to revert the whole block in case the state is declared invalid by a fisherman later.

## Adaptive corruption

One important attack vector on blockchains that cannot scale infinitely the number of validators is the possibility for adaptive corruption, i.e. the possibility for a malicious party to bribe or corrupt only a specific subset of the actors in order to take over the network, or grief other participants.
In Near, this problem is addressed partly by using the randomness beacon to assign validators to shards, and also by the use of "hidden validators" whose assignment to any given shard is never released publicly, making it impractical to find out who should be corrupted.
However, at the full chain level, the block producers are operating in following a round robin, so attacks such as DDOSing a particular producer becomes easier, and the validators are rotated once every epoch (15 hours), so any block that would be challenged within that period would let the set of validators exposed for the rest of the epoch.

## More practical considerations

Near currently operates only on one shard and with 100 validators. Nothing is put in place to ensure that the stake are evenly distributed so there are huge differences between stakes, with the most bonded validator getting bonded for 25 000 000 Near tokens and the least bonded being worth only 180 000 Near tokens. It makes taking over the least slots relatively cheap if considering the security of a 4B USD network (at the time of writing, the price of the Near token is around 4.2USD)
Worse, taking over the last 34% of the validators and being in a theoretically able to take over the whole network requires only 11.5M Near tokens, which is less than 50M USD.
The liveness and security of the protocol can also be badly impacted by the fact that there is no punishment for being offline during block production, other than the loss of the block reward.

#### What We Like

1. Near is aware of and has addresses the different security challenges posed by proof-of-stake consensus based blockchains, albeit not always in an optimal manner.
2. Consensus separation in block production and finality that favours liveness while still maintaining PBFT security for mission critical usecases that can trade a higher speed for added security.
3. The problem of shard corruption spreading out being addressed through a clever use of randomness and hidden validators, with Fishermen being the end guardians.

#### What We Don't Like

1. Nothing is done to ensure the stakes of the validators are homogenized, leading to as system where the worst case security bound is practically very low even though the total bonded is reasonably high.
2. The use of round robin for the main block production, allowing for a malicious actor to adaptively bribe, grief, DDos, of more generally attack validators following their turns.
3. The fast cross-shard transaction favours liveness over immutability in that if a malicious actor manages to take over a shard and starts to issue a cross-shard transactions, the full block state would be eventually reverted to a past state before the attack if challenged by a fisherman during the challenge period.
4. In case a bad block is challenged, the validators would be uncovered (the assigment to their chunks would be public) till the end of the epoch.
5. No offline penalties apart from the missing reward.

<a id="7">[7]</a> Miguel Castro and Barbara Liskov: _Practical Byzantine Fault Tolerance_  https://pmg.csail.mit.edu/papers/osdi99.pdf