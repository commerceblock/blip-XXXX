```
bLIP: XXXX
Title: Zero-conf Statecoin Channels
Status: Draft
Author: 
Created: 2022-XX-XX
License: GPL 3.0
```

## Abstract

This document decribes how a Poon-Dryja lightning channel can be constructed without reliance upon on-chain transactions, using the statechain approach as implemented in MercuryWallet.  This approach will not require any changes to the lightning network protocol and would need only simple enhancements to the MercuryWallet statechain API.

## Motivation

The MercuryWallet statechain system facilitates the transfer of ownership of Bitcoin (or Elements based) unspent transaction outputs (UTXOs) between parties without performing on-chain transactions. This is acheieved by sharing the single key of the UTXO between the owner and a statechain entity (SE), and providing a time-locked backup transaction that returns the UTXO to the full custody of the owner.  

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
6. Optimistically generated pre-signed peg-out transaction is signed by both parties that has no timelock.
7. Enforcement by the SE to ensure a user pushes funds to one side of the channel in order for the channel to be closed.

## Specification

The following sequence specifies the process of the creation and closing of the 'state channel':

1. A statecoin is deposited by Alice (A) with the SE. A has one keyshare (`S_A`) and the SE has the other (`S_SE`) - the full private key is `S_A*S_SE`. The SE and A cooperate the co-sign Alice's backup transaction with an `nLocktime` of blockheight `b0`. 
2. A finds channel counterparty Bob (B) that supports `option_zeroconf_state_channel`. 
3. A and B generate secrets `A_T` and `B_T`
4. A and B interact to generate a shared secret, `T` (where `T = A_T*B_T`). 
5. A and B share the public values `A_T.G`, `B_T.G` and `T.G` with the SE (where `G` is the secp256k1 generator point). These are used to authenticate SE co-signing. 
6. A and the SE cooperatively generate a timelocked ZSC peg-out transaction (authorised by B, signing with `B_T`). 
7. A and B generate a ZSC peg-out transaction with no timelock. 
8. A, B and the SE complete the statecoin transfer to `SE+T`. 
9. A and B can now announce the channel open to the network. In the `open_channel` message the `temporary_channel_id` should specify the timelocked peg-out transaction and the `open_channel_tlvs` must reflect that this channel is an `option_zeroconf_state_channel`. 

At any time, A or B can request the SE to sign the peg-out transaction with no timelock. In the event that a non-timelocked peg-out transaction is broadcast to the network, Alice or Bob must specify the new channel outpoint in the `funding_created` message, as the `txid` has has changed.

In the event that Alice and Bob wish to virtually close their ZSC (and all funds have been pushed to B side of the channel), A and B can sign a statecoin transfer message with `A_T` and `B_T`. Once the SE has validated this message, the statecoin can be transferred from `SE+T` to `SE+B` using the normal statecoin transfer protocol. When B verified the transfer, B announces the closure of the `SE+T` ZSC to the public lightning network. 

Zeroconf State Channel (ZSC) steps:

```
     _______           S_A*S_SE              _____________
    |       |<----------------------------->|             |
    |       |          backup_tx (S_A)      |             |
    |   A   |<------------------------------|     SE      |       <-  A deposit statecoin
    |       |          nLocktime = b0       |             |
    |_______|<------------------------------|             |
                                            |             |
                                            |             |
     _______             T*S_SE             |             |              T*S_SE            _______ 
    |       |<----------------------------->|             |<----------------------------->|       |
    |       |        backup_tx (T)          |             |           backup_tx (T)       |       |
    |   A   |<------------------------------|             |------------------------------>|   B   |      <- create zeroconf_state_channel
    |       |      nLocktime = b0 - 6       |             |       nLocktime = b0 - 6      |       |
    |       |<------------------------------|_____________|------------------------------>|       |
    |       |                             pegout_tx (no timelock)                         |       |
    |_______|<--------------------------------------------------------------------------->|_______|    
    
    
     _______                                                                               _______ 
    |       |                                                                             |       |
    |       |                                                                             |       |
    |   A   |<--------------------------------------------------------------------------->|   B   |     <- update zeroconf_state_channel
    |       |                                                                             |       |
    |_______|                                                                             |_______|   
    
                                
     _______           Sig(A_T)              _____________            Sig(B_T)             _______ 
    |       |------------------------------>|             |<------------------------------|       |
    |       |          Sig peg out          |             |           Sig peg out.        |       |
    |   A   |<------------------------------|     SE      |------------------------------>|   B   |      <- close zeroconf_state_channel
    |       |                               |             |                               |       |
    |_______|                               |_____________|                               |_______|
```

## Universality

Most nodes are probably not interested in Zeroconf State Channel, but that is fine since
for the approach to be useful only two parties are needed to be interested in
establishing such a channel to support it, and they connect to each other directly.

## Backwards Compatibility

The Zeroconf State Channel protocol is not backwards compatible.

## Reference Implementation

[Original Statechains Paper]: https://github.com/RubenSomsen/rubensomsen.github.io/blob/master/img/statechains.pdf
