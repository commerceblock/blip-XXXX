```
bLIP: XXXX
Title: Zero-conf Statecoin Channels
Status: Draft
Author: 
Created: 2022-XX-XX
License: GPL 3.0
```

## Abstract

Document decribes how a Poon-Dryja lightning channel can be constructed without reliance upon on-chain transactions, using the statechain approach as implemented in MercuryWallet.  This approach will not require any changes lightning network and would need only simple enhancements to MercuryWallet statechain API.

## Motivation

The MercuryWallet statechain system fascilitates the transfer of ownership of Bitcoin (or Elements based) unspent transaction outputs (UTXOs) between parties without performing on-chain transactions.

Bringing together these two protocols/approaches you get two benefits.
1. Capability to “enrol” a once fixed-size state coin UTXO.
2. The ability of transporting the lightning channel off-chain at any time.

## Rationale

A statechain’s security model rests on certain assumptions. You have a SE and a keyholder (Alice), neither of which has the full underlying key material. Before depositing funds into the statechain, Alice will cooperatively generate a peg-out transaction with the SE, so that in the event the SE goes offline, Alice can recover her funds. When Alice transfers her keyshare to Bob, Bob too will cooperatively generate a new peg-out transaction. With each new peg-out transaction, a timelock is decremented. While this does put a limit on the total number of times a statecoin can be transferred, this timelock ensures that Bob’s peg-out transaction can be confirmed before Alice’s. Ultimately, Bob trusts that the SE and Alice do not collude, or that the SE and Alice are not the same person.

This BLIP focuses on the model where a lightning channel is created inside a state coin peg-out transaction.  As this channel is not visible on-chain, this will be considered to exist with a subcategory of zero-conf channels.

A Zero-conf Channel will have the following attributes:
1. The state chain peg-out transaction will open a lightning channel.
2. The lightning channel will be timelocked.
3. Cooperative closure of the channel with or without the SE will require the timelock to expire.
4. Periodic updates of the peg-out transaction to reflect the exact amount each party is owed.
5. Each update to the peg-out transaction will result in the timelock being decremented.

## Specification

## Universality

## Reference Implementation
