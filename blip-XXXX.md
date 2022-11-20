```
bLIP: XXXX
Title: Zero-conf Statecoin Channels
Status: Draft
Author: 
Created: 2022-XX-XX
License: CC0
```

## Abstract

Document decribes how a Poon-Dryja lightning channel can be constructed without reliance upon on-chain transactions, using the statechain approach as implemented in MercuryWallet.  This approach will not require any changes lightning network and would need only simple enhancements to MercuryWallet statechain API.

## Motivation

The MercuryWallet statechain system fascilitates the transfer of ownership of Bitcoin (or Elements based) unspent transaction outputs (UTXOs) between parties without performing on-chain transactions.

Bringing together these two protocols/approaches you get two benefits.
1. Capability to “enrol” a once fixed-size state coin UTXO.
2. The ability of transporting the lightning channel off-chain at any time.

## Rationale

## Specification

## Universality

## Reference Implementation
