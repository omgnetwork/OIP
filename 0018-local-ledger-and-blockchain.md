---
oip: "0018"
title: Local ledger behaviour for fund transfers between internal and blockchain wallets
status: Draft
type: eWallet
author: Unnawut Leepaisalsuwanna <unnawut@omise.co>
created: 2019-08-01
---

# OIP-0018: Potterhat's fallback mechanism when relying on multiple Ethereum clients

# Simple Summary

This OIP complements [OIP-12: Blockchain integration](https://github.com/omisego/OIP/pull/12) by going into the nitty-gritty details of how the blockchain funds should be reflected inside the eWallet's local ledger.

# Motivation

With blockchain integration, we have, in a way, two sources of truth (local ledger vs. blockchain) that behave very differently but still need to be kept in sync.

This OIP lays down the behaviour for the eWallet's local ledger when dealing with transactions that cross between internal wallets and blockchain wallets, so that the integrity of both the local ledger and blockchain can be kept.

# Specification

## Part 1: Importing or generating the ERC-20 token

A blockchain-enabled token could appear in the eWallet in 2 ways:

### 1. Importing an existing ERC-20 token

1. An ERC-20 token called "ABC" was deployed externally, with total supply of 1,000,000 ABC.
2. An admin imports the token into the eWallet. A new `EWallet.Token` is created, with 0 minted tokens. This token is also reflected in the local ledger with 0 total supply.
3. Someone sends 1,000 ABC from the blockchain into the eWallet's hot wallet.

### 2. Generating a new ERC-20 token

1. An admin generates a `EWallet.Token` with blockchain enabled and configured to mint 1,000 ABC on creation.
2. A new ERC-20 contract named "ABC" gets deployed, with total supply of 1,000.
3. Since the eWallet deploys the contract, the eWallet's hot wallet receives the freshly-minted 1,000 ABC from the contract.

As you can see in both cases, some amount of ABC arrives in the eWallet's hot wallet. This arrived funds will be used in the next part.

At this point, the local ledger is unaware of the events, but some other part of the eWallet must be monitoring for these received funds.

## Part 2: Accounting for the received blockchain funds

1. The eWallet sees that it received 1,000 ABC. This means the eWallet has gained 1,000 ABC-worth of value that it has control of. In other words, the eWallet has 1,000 ABC-worth of collateral that it can use freely within its internal system.
2. So in order to utilize that 10 ABC-worth of value, the eWallet mints 10 ABC into its internal wallet via the local ledger.
3. The eWallet can now freely move this locally minted 10 ABC within its system, be it another account's or a user's internal wallet.

## Part 3: Accounting for the blockchain funds leaving the eWallet (normal mode)

Funds may leave the eWallet via 2 originating actions:

### 1. Transferring from the hot wallet to an external blockchain address

### 2. Transferring from an internal wallet to an external blockchain address

## Part 4: Accounting for the blockchain funds leaving the eWallet (panic mode)

In unusual cases, the funds may have left the hot wallet without the eWallet's knowledge. E.g. the private key holder imported the mnemonic phrases into a hardware wallet then transfer funds from there. In this case:

1. The eWallet should notify the admins that funds have left the hot wallet that were not accounted for.

Question:
- Should we block further transactions until the funds are refilled or adjusted?
- How to handle funds entering the system during this discrepancy? We don't know if this is new funds or it is the funds intended to fill up the missing amount.

## Other scenarios
- eWallet has a locally minted token, then converted to a blockchain token

## Notes

Observe that this way, the local ledger token's total supply is always equal to the token amount in the eWallet's designated hot wallet.
