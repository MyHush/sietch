# Byrsa 

A walled citadel on a hill for your shielded funds, made from tiny bits of shielded
dust or "zdust", like the bits of oxhide that Dido used in founding the citadel
of Carthage:

```
		In Virgil's account of Dido's founding of Carthage,
		when Dido and her party were encamped at Byrsa,
		the local Berber chieftain offered them as much land
		as could be covered with a single oxhide.
		Therefore, Dido cut an oxhide into tiny strips and set
		them on the ground end to end until she had completely
		encircled the hilltop of Byrsa (Greek: βύρσα, "oxhide"). 
```

## What is Byrsa?

Byrsa is a new set of RPC functionality and consensus rules for
using shielded funds, which do not currently modify the internals of the
Zcash Protocol, but sit on top of it. It defines certain wallet
and transction conditions that must be upheld for transactions
to be accepted by the network.

Any Zcash Protocol coin could adopt this "upgrade" to plain Zcash Protocol,
though it should be noted that Hush was the first pure Sapling Zcash Protocol
coin and so our implementation ignores Sprout JoinSplits, since we thankfully
have none on our v3 mainnet. Supporting Sprout would not be complex and is
left as an exercise to the interested privacy coin.

## How is Byrsa implemented?

Byrsa is written in C++ the hushd full node, which means
all GUI wallets can benefit from it. Some features are implemented
at the GUI wallet level. The goal is that the user does not need
to think or do anything different, to have more privacy when
making transactions.

From the perspective of the average GUI wallet user, It Just Works.
There are no additional mandatory steps for the user when sending
each transaction. There will be some options for advanced users, to
tweak the settings, but using them is not required for increased privacy.
They will mostly serve as ways for developers to experiment and optimize
things.

## What are the goals of Byrsa?

 * Make linkability analysis of zaddrs drastically more expensive
 * Allow people to make zaddr xtns safely/privately without thinking about
   metadata leakage or advanced techniques
 * Prevent average users from making some blockchain operations which give out too much metadata
 * Break some assumptions which many blockchain analyst software uses
 * Require blockchain analysts to write new software
 * Increase the privacy of all funds in the shielded pool
 * Introduce non-determinism to counter ITM/Metaverse style metadata attacks

## What are the limitations/downsides of Byrsa?

The goal of Byrsa is increased privacy and the cost is increased blockspace usage per transaction,
increased CPU time and RAM to validate transcations and as a secondary effect, increased sync times.

Byrsa potentially adds inputs and outputs to transactions to increase their privacy,
and so the maximum number of actual recipients is lower when Byrsa is enabled. In practice,
transactions can still send to over 1000 recipients even with Byrsa protections, so it
doesn't actually effect any current users, as far as we know.

How much longer will average xtn take?

No increased RAM usage, as creating JoinSplits is a serial process, but shielded transactions
will take longer in CPU seconds, since there will be more recipients by default.

## Implementation Details

These RPCs are modified:

```
	z_sendmany
	z_shieldcoinbase
	z_mergetoaddress
```

