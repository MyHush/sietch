# Sietch

## What is Sietch?

Sietch is the codename for a new set of RPC functionality and consensus rules related to
using shielded funds. Shielded transaction creation is modified above
the level of the Zcash Protocol itself, no changes to that
layer are made. Additionally, there are no changes at how the RPC layer
is used, i.e. Sietch is implemented in hushd internals and existing software
which uses `hush-cli` and/or the RPC interface are not required to change
in any way. This preserves compatibility with all existing Zcash protocol
ecosystem software at a very low level.

Sietch defines certain wallet and transaction conditions that must be
upheld for transactions to be accepted by the network as well as making all clients have
new default behavior that is more privacy-preserving. This involves both
new wallet behavior that is non-consensus (currently implemented) and also future consensus rules
to enforce this new default behavior.

Sietch does not modify anything about Zcash internals such as zk-SNARKs,
cryptographic primitives or changing any of the deep zero knowledge math involved, such
as Elliptic Curves parameters, datatypes or other cryptographic machinery.

Any Zcash Protocol coin could adopt this "upgrade" to Sietch-enabled Hush Protocol,
though it should be noted that Hush was the first pure Sapling Zcash Protocol
coin and so our implementation ignores Sprout zaddrs. This is because we thankfully
have no Sprout zaddrs or transactions on our v3 mainnet.
Sietch is compatible with older sprout zaddrs and supporting
it is left as an exercise to the interested privacy coin.

## How is Sietch implemented?

Sietch is written in C++ and inside the hushd full node code, which means
all existing code that interacts with hush can benefit *with no changes*.
Some optional/advanced features in the future
may be implemented at the GUI wallet level. The goal is that the user does not need
to think or do anything different, to have more privacy when
making transactions.

Since Sietch works at the hushd level, all existing code, such as GUI wallets,
mining pools and exchanges, require no code changes. They will make a shielded
transaction as they always have, via the same exact RPC methods and parameters.

From the perspective of the average GUI wallet user, It Just Works.
There are no additional mandatory steps for the user when sending
each transaction. No mandatory wallet maintenance will be needed
to keep their privacy. No confusing options to "get right" on each transaction.
The same goes for exchanges, mining pools and all Hush users.
No changes at all are required by end users to enable Sietch, other than upgrading
to compatible Hush software.

There will be some options for advanced users, to
tweak the settings, but using them will not be required for increased privacy.
Hush is dedicated to all future privacy features being defaulted to ON instead
of asking our users to opt-in to privacy, which we know is a fools errand.

Users will not have to opt-in to Sietch and there will be no way to turn off
the functionality. Additionally, Sietch will be enabled on mainnet as an opt-in feature
at first, as a first wave of adoption and for testing. When all Hush users have updated
to software with Sietch-enabled Hush protocol, a new consensus rule will go into effect
which will prevent older non-Sietch clients. Additionally, Hush will increase it's
PROTOCOL\_VERSION with the Sietch feature, which will also allow Sietch-enabled Hush nodes
to effeciently block potentially malicious non-Sietch hushd clients at the p2p level.

## What are the goals of Sietch?

 * Make linkability analysis of fully shielded (z2z) transactions drastically more expensive
 * Allow people to make zaddr xtns safely/privately without thinking about
   metadata leakage or advanced techniques
 * Prevent average users from making some blockchain operations which give out too much metadata
 * Break assumptions in blockchain analyst software
 * Require blockchain analysts to write new software
 * Increase computational cost of current blockchain analysis
 * Increase the privacy of all funds in the Hush shielded pool
 * Introduce non-determinism to counter ITM/Metaverse style metadata attacks

## What are the limitations/downsides of Sietch?

The goal of Sietch is increased privacy at the cost of increased blockspace usage per transaction,
increased CPU time and RAM to validate transcations and as a secondary effect, increased sync times.

Sietch potentially adds inputs and outputs to transactions to increase their privacy,
and so the maximum number of actual recipients is lower when Sietch is enabled. In practice,
transactions can still send to over 1000 recipients even with Sietch protections, so it
doesn't effect normal usage.

How much longer will average zxtn take? Research: How many zouts while staying under <10s?

Potentially more txfees if the transaction becomes so large to require more than the default fee.

No increased RAM usage, as creating JoinSplits is a serial process, but shielded transactions
will take longer in CPU seconds, since there will be more recipients by default.

## Implementation Details

These RPCs are modified directly:

```
	z_sendmany
```

These RPCs are currently not modified, but may be in the future:

```
	z_shieldcoinbase
	z_mergetoaddress
```

These RPCs interact with zaddr xtns and may report different or additional info after this change:

```
	z_viewtransaction
	z_listtransactions
	listunspent
```

These new RPCs were added in Sietch:

```
	z_listnullifiers
```

### z\_sendmany Rule of Seven

Our goal is to be non-deterministic while also inserting enough zouts such that we have at least N=7 zouts.

Since the normal case is to have 2 zouts (one recipient for a z2z and one change zout back to sending address),
the normal case will be to add 5 zouts to a normal z2z xtn.

  * For a z=>t, we must add 7 zouts
  * For a t=>z we must add 6 zouts
  * For a t=>(z,z) we must add 5 zouts
  * For a z=>z we must add 6 zouts
  
The reason N=7 is chosen is because of the simple fact that `6!=720` while `7!=5040`. This parameter is chosen
in response to the ITM attack, which relies on a small number of zouts and doing combinatorial algorithms on all
possibilities. These combinatorial algorithms increase in state space for each link in a long chain of transactions.
Traditionally each xtn only has a few zouts and so because `2!=2` and `3!=6`, the combinatorial explosion does not
have a chance to slow down the ITM attack.

Now, consider 5040 choices at each link in a chain, and say we are studying a chain of length L=10.
Compared to pre-Sietch, we would have `2^10=1024` possibilities versus

```
5040^10 = 10575608481180064985917685760000000000
````

possibilities. Even for short chains of xtns, the combinatorial explosion of possibility renders the ITM attack
extremely expensive. It limits it to searching for linkability metadata in only short chains with lots of
additional supporting metadata, effectively raising the bar for attack and removing many potential attackers
who do not have sufficient resources. In pre-Sietch code, the ITM attack can potentially research very long chains,
dozens, hundreds and perhaps thousands of transactions in length, with commodity hardware.
Sietch exponenentially increases the cost of doing this, in RAM, CPU and running-time. Either an attacker would
need botnet or supercomputer-level resources to attack the same length chains as pre-Sietch code, or more likely,
de-anonymizing attackers will focus on studying short linkability chains with a lot of additional metadata from
timing analysis, amount analysis and potentially passive or active dust attacks.

## Non-determinism

Combinatorial explosion can only protect us so much. It is only one layer of defense and surely will not save us
from the inevitable quantum computers which are being optimized every day.

When adding zutxos, to break the ITM/Metaverse metadata attacks at a deep level, we must break a deep assumption
that is baked deep into Bitcoin wallet behavior: determinism.

Given the exact same wallet.dat, Bitcoin, Zcash and Hush will act in *exactly the same way* when sending a transaction, every
single time. Given all input data, one can predict the behavior of what will happen. Obviously, this is a very good idea in Bitcoin,
for total supply and issuance accountability. But this also contributes to Bitcoin being a "surveillance coin", that perfectly
preserves metadata until the end of time.

Zcash had no reason to change this and they did not, and so this behavior is baked in deep to all Bitcoin and Zcash forks.
In light of the ITM/Metaverse attack, this determinism is considered dangerous by the author. The reason is that the
ITM/Metaverse attack utilizes the fact that wallet operations are predictable to extract more metadata than previously thought possible from z2z and t=>z xtns. The only way to prevent that is to break the assumption of predictable wallet behavior.

## Downsides/Issues/Attacks

Code which uses purely raw transactions and broadcasts them to the network needs to do extra work to support Sietch.
This is why SDL has it's own implementation of Sietch, and any kind of hardware wallet support that uses raw transactions
in the future, would need to learn about Sietch.

### Metadata Attacks Against Sietch

Yes, Sietch can be metadata-attacked itself!
TLDR: Worst case is that an attacker steals wallet.dat files and does an immense amount of work to reduce privacy back to pre-Sietch levels, using the ITM attack.

At first there was a single Sietch implementation of 200 zaddrs that were fixed. If the wallet.dat owning those zaddrs were stolen, that could be used to delete all the Sietch "privacy dust" from the transaction graph, making the job of blockchain analysts and/or ITM attackers much easier. This theoretical attack is what prompted dynamic Sietch addresses and also a better way of generating zaddrs inside SDL: using BIP39 seed-phrases to generate a single zaddr and then delete the seed phrase. This method leaves no wallet.dat on disk to steal and the private key material for the zaddr only existed in memory a short time.

Currently in production are 200 static zaddrs in `hushd` and 10,000 static (BIP39-derived) zaddrs in `SDL`. The dynamic
Sietch zaddr code for `hushd` is complete and can be viewed here: https://github.com/MyHush/hush3/tree/sietch_dynamic
It's currently being performance tested as it does some exotic things.

There is no wallet.dat to steal to recover data about Sietch zoutputs for 10,000 of the 10,200 zaddrs currently in the combined Sietch zaddr pool, so this attack is no longer viable. Dynamic Sietch zaddrs will make the entire process much more secure by preventing analysts/attackers from even knowing the zaddrs that could potentially be a Sietch output. These dynamic Sietch zaddrs will be generated at run-time and private keys never even written to disk, nor part of the `hdseed` of any wallet.dat in the case of `SDL`.

## Conclusions

Sietch-enabled Hush Protocol can be thought of as using the ideas of combinatorial explosion and non-determinism to thwart brand-new
blockchain analysis techniques. Non-determinism is the stronger weapon, but it does not add enough privacy unless we add in the appropriate amount of combinatorial explosion to linkability analysis. Together they are a potent weapon which also give us a knob
to turn to increase future security, i.e. the minimum number of zaddr outputs allowed in a transaction.

This is why both techniques complement each other and have a greater privacy improvement when used together.

## Current Implementations

There are currently 4(!) implementations of Sietch in Hush world, 2 inside of `hushd` internals and 2 for `SilentDragonLite` which uses raw transactions and not the RPC interface of `z_sendmany`. Each of the 2 implementations has a static (drawing from a fixed pool of Sietch zaddrs) and a dynamic version (dynamically generating Sietch zaddrs at run-time). Currently the static implementations are in production as of `Hush 3.3.0` and `SilentDragonLite 1.1.3` and dynamic versions are mostly complete and undergoing performance testing.
