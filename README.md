# Byrsa 

A walled citadel on a hill for your shielded funds.

## What is Byrsa?

Byrsa is a new set of consensus rules for using shielded funds, which does
not modify the internals of the Zcash Protocol, but sits on top
of it. It defines certain wallet and transction conditions that
must be upheld for transactions to be accepted by the network.
These conditions

## How is Byrsa implemented?

Mostly it's done directly in C++ the hushd full node, which means
all GUI wallets can benefit from it. Some features are implemented
at the GUI wallet level. The goal is that the user does not need
to think or do anything different, to have more privacy when
making transactions.

From the perspective of the average GUI wallet user, It Just Works.
There are no additional mandatory steps for the user when sending
each transaction.

## What are the goals of Byrsa?

 * Make linkability analysis of zaddrs drastically more expensive
 * Allow people to make zaddr xtns safely/privately without thinking about
   metadata leakage or advanced techniques
 * Prevent some blockchain operations which give out too much metadata
 * Break some assumptions which many blockchain analyst software uses
 * Require blockchain analysts to write new software
 * Increase the privacy of all funds in the shielded pool

