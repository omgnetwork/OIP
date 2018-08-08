---
oip: "xxxx"
title: eWallet Blockchain Integration
status: Active
type: eWallet
author: Thibault Denizet <thibault@omise.co>
created: 2018-08-08
---

# OIP-0004: eWallet Blockchain Integration

## Abstract



## Overview

## Specification

### Terminology

- Internal Wallet
- Blockchain Wallet
- Plasma Wallet

### eWallet Ethereum Name Service

The eWallet Ethereum Name Service is a smart contract running on the Ethereum blockchain that allows eWallet providers to register their eWallet as being part of the network. It is a simple key/value storage where the key is the eWallet primary public key and the value their current domain name.

Option #1: Using [ENS](https://ens.domains/)
Option #2: Build a simple eWallet Name Service.

### Blockchain Wallets

Blockchain wallets will be the representations of Ethereum accounts in the eWallet. Two types of blockchain wallets will exist: `hot` and `cold`.

The eWallet saves the encrypted private keys of the `hot` wallets in the database, with the encryption key being stored in an environment variable. No private keys are stored with `cold` wallets and the administrators are responsible for managing them directly. Interaction with `cold` wallets will require manual action through [Metamask](https://metamask.io/) or similar tools.

See "Database Changes" for more information.

### Plasma Deposits

Every time funds are deposited on a plasma chain, a deposit will be recorded in the database.

### eWallet Setup

The only mandatory step to plug an eWallet to Ethereum is to generate a new Ethereum account. The public key for that Ethereum account will be used as the main identifier for the eWallet. The private key will be encrypted and stored in the database, while the encryption key is stored in environment variables.

This value should be stored in the `Setting` table, under `primary_pub_key`

### Token creation

ERC-20 tokens will be creatable from the eWallet. Using a pre-defined Contract Template (see databases tables), a new token will be created on Ethereum by deploying the contract. Only ERC-20 will be supported at first. Providers will be able to create their own contract templates as long as they are compliant with ERC-20.

### Key Management Strategies

There are 3 different key management strategies that will be supported by the eWallet.

#### 1. Full Custody (providers own all the private keys)

Very close to the current state of things, except transactions between eWallets happen on the blockchain. Users are not able to get their private keys.

#### 2. Partial custody (providers own all the private keys by default, but users may request ownership of their private keys)

Similar to Full Custody (and a transaction from that strategy is possible at anytime) with a twist: users are able to request ownership of their keys.

#### 3. Non Custodial (providers hold only their private keys, and all users must own their private keys)

Finally, with the "Non Custodial" strategy, all users will have to own and take care of their private keys themselves. This strategy needs to be set during the eWallet setup as a transaction from the other two strategies cannot be handled by the eWallet. A provider can, however, decide to switch to that strategy by handling the switching process with its users.

### Transaction Flows for one eWallet

The following flows are limited to transactions happening inside one eWallet.

#### Full custody

Full custody transactions inside an eWallet happen exactly in the same way as internal transactions. There is no interaction with the blockchain. Funds will be moved between internal wallets.

#### Partial custody

For partial custody, unless a user has requested ownership of his/her private keys, a transaction will happen in the same way as the "Full custody" strategies.

However, if a user owns the private key, the flows changes.

##### From a user to an eWallet

1. A user owning the private key wants to send funds to an eWallet in order to purchase something.
2. The eWallet specify the amount it expects to receive to the user.
3. The user signs and sends a transaction with the appropriate amount to one of the eWallet addresses with some metadata to correlate with the eWallet internal data. Websockets are available to track the progress of the transaction from it being received until confirmed.
4. Once the eWallet is satisfied with the level of confirmation for the transaction, it can deliver the goods.

##### From an eWallet to a user

1. A user purchases something through a provider, and that provider wnats to reward him with some loyalty ETC-20 tokens.
2. The user specifies the address where he wishes to receive funds. This can be selected from a list of addresses the eWallet knows belong to the user, or an entirely different address.
2. Once the purchase is settled, a transaction is signed and sent from the eWallet to the address given by the user.
3. The eWallet will keep an eye out on the transaction to see that it proceeds as expected, reporting the status to the user through websockets.

#### Non custodial

With the "Non Custodial" strategy, the flows are the same as the "Partial Custody" strategy where a user owns the private keys.

### Transaction Flows for two eWallets

For the flows below, we'll use the following use-case:

```
A end-user wants to purchase something from eWallet A using a mobile application published by (and therefore connected to) eWallet B.
```

In the cases below, both eWallets have funds deposited in Plasma: eWallet A has `ABC` tokens while eWallet B has `ZYX` tokens.

#### eWallet (Full custody) -> eWallet (Full custody)

1. Using eWallet B mobile app, the user scans a QR code to pay for his purchase at eWallet A (10 `ABC`) using funds stored in eWallet B pooled wallets (`ZYX`). That QR code contains a string composed of an `ewallet_address` and a transaction request ID.
2. eWallet B mobile application calls the `transaction_request.consume` endpoint sending the content of the QR code.
3. eWallet B detects that the received public key doesn't match its own, meaning the transaction request is hosted by another eWallet.
4. eWallet B then makes a call to the eWallet Ethereum Name Service sending the public key received in the consumption call. A domain name will be returned by the NS.
5. eWallet B sends a request to eWallet A to consume the request on behalf of the user. It receives a consumption ID back (with status `awaiting_funds`) and wether or not the tokens need to be converted (and if yes, to which currency) - this is defined by a conversion strategy inside the eWallet. In this case, eWallet A only wants to receive `ABC`. eWallet B connects to the websocket channel for the consumption.
6. eWallet B requests a quote from the DEX in order to know how many `ZYX` are needed to send 10 `ABC` to eWallet A. An event is sent to eWallet A about the progress.
7. The user receives the quote through websockets on eWallet B Mobile application and approves it. An event is sent to eWallet A about the progress.
8. eWallet B places an exchange order to get 10 `ABC`. Once the order has been matched, it sends the funds (10 `ABC`) to eWallet A address on Plasma. An event is sent to eWallet A about the progress.
9. eWallet A receives the Plasma event through the watcher, confirm the consumption and notifies eWallet B. eWallet B forward the event to the user's mobile application to let him know the transaction has been confirmed.
10. eWallet A POS/Admin panel receives an event in the websocket channel for the transaction request.

#### eWallet (Full custody) -> eWallet (Partial custody)

#### eWallet (Full custody) -> eWallet (Non custodial)


#### eWallet (Partial custody) -> eWallet (Full custody)

#### eWallet (Partial custody) -> eWallet (Partial custody)

#### eWallet (Partial custody) -> eWallet (Non custodial)


#### eWallet (Non custodial) -> eWallet (Full custody)

#### eWallet (Non custodial) -> eWallet (Partial custody)

#### eWallet (Non custodial) -> eWallet (Non custodial)

### Implementation

#### Code organization

#### Database Changes

Table name: `contract_templates`

- `uuid`
- `name`
- `content`
- `inserted_at`
- `updated_at`

Table name: `blockchain_wallet`

- `uuid`
- `address`
- `pub_key`
- `encrypted_priv_key` (only for `hot` wallets, encrypted using a key defined in ENV variables)
- `type` (`hot`|`cold`)
- `primary` (`true`|`false`)
- `inserted_at`
- `updated_at`

Table name: `limits`

- `uuid`
- `token_id`
- `low_amount`
- `high_amount`
- `limit_type` (`percent`|`value`)
- `blockchain_wallet_uuid`
- `inserted_at`
- `updated_at`

In case a blockchain wallet goes below or above the limits, a user will be notified that an action is required to keep the system secure (like moving funds in/out from/to a cold wallet.

Table name: `child_chain_contract`

- `uuid`
- `contract_address`
- `active` (`true`|`false`)
- `inserted_at`
- `updated_at`

Table name: `deposit`

- `uuid`
- `child_chain_contract_uuid`
- `amount`
- `token_id`
- `inserted_at`

Table name: `exit`

- `uuid`
- `child_chain_contract_uuid`
- `inserted_at`

Table name: `exited_balances`

- `uuid`
- `exit_uuid`
- `amount`
- `token_id`
- `inserted_at`

Table name: `wallet`

- `blockchain_wallet_uuid` (`nullable`)

Table name: `token`

- `contract_address` (`nullable`)
- `contract_template_uuid` (`nullable`)
- `status` (`pending`|`confirmed`)

Table name: `mint`

- `status` (`pending`|`confirmed`)

Table name: `setting`

- `key`
- `value`
- `inserted_at`
- `updated_at`

Keys / Values:

`ewallet_identifier` : `pub_key` (`null` if not setup with blockchain)
