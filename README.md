# Sietch

## What is Sietch?

Sietch is a new set of RPC functionality and consensus rules related to
using shielded funds. Shielded transaction creation is modified above
the level of the Zcash Protocol itself, no changes to that
layer are made. This preserves compatibility with all existing Zcash protocol
ecosystem software.

Sietch defines certain wallet and transction conditions that must be
upheld for transactions to be accepted by the network as well as making all clients have
new default behavior that is more privacy-preserving.

Sietch does not modify anything about Zcash internals such as zk-SNARKs,
cryptographic primitives or changing any of the deep math involved, such
as Elliptic Curves parameters or other deep cryptographic machinery.

Any Zcash Protocol coin could adopt this "upgrade" to plain Zcash Protocol,
though it should be noted that Hush was the first pure Sapling Zcash Protocol
coin and so our implementation ignores Sprout JoinSplits, since we thankfully
have none on our v3 mainnet. Supporting Sprout would not be complex and is
left as an exercise to the interested privacy coin.

## How is Sietch implemented?

Sietch is written in C++ and mostly lives inside the hushd full node, which means
all GUI wallets can benefit from it. Some features in the future
may b  implemented at the GUI wallet level. The goal is that the user does not need
to think or do anything different, to have more privacy when
making transactions.

Since Sietch works at the hushd level, all existing code, such as GUI wallets,
mining pools and exchanges, require no code changes. They will make a shielded
transaction as they always have, via the same exact RPC methods and parameters.

From the perspective of the average GUI wallet user, It Just Works.
There are no additional mandatory steps for the user when sending
each transaction. No mandatory wallet maintenance will be needed
to keep their privacy. The same goes for exchanges, mining pools and all Hush users.
No changes at all are required by end users to enable Sietch, other than upgrading
to compatible Hush software.

There will be some options for advanced users, to
tweak the settings, but using them will not be required for increased privacy.
Hush is dedicated to all future privacy features being defaulted to ON instead
of asking our users to opt-in to privacy, which we know is a fools errand.

Users will not have to opt-in to Sietch and there will be no way to turn off
the functionality.

## What are the goals of Sietch?

 * Make linkability analysis of fully shielded (z2z) transactions drastically more expensive
 * Allow people to make zaddr xtns safely/privately without thinking about
   metadata leakage or advanced techniques
 * Prevent average users from making some blockchain operations which give out too much metadata
 * Break some assumptions which many blockchain analyst software uses
 * Require blockchain analysts to write new software
 * Increase the privacy of all funds in the shielded pool
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
	z_shieldcoinbase
	z_mergetoaddress
```

These RPCs interact with zaddr xtns and may report different or additional info after this change:

```
	z_viewtransaction
	z_listtransactions
	listunspent
```

### z\_sendmany

