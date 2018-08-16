---
oip: "xxxx"
title: eWallet Blockchain Integration
status: Active
type: eWallet
author: Thibault Denizet <thibault@omise.co>
created: 2018-08-08
---

# OIP-xxxx: eWallet Blockchain Integration

## Abstract

This OIP describes the changes required to plug an eWallet onto Ethereum and Plasma.

## Specification

### eWallet Ethereum Name Service

The eWallet Ethereum Name Service is a smart contract running on the Ethereum blockchain that allows eWallet providers to register their eWallet as being part of the network. It is a simple key/value storage where the key is the eWallet primary public key and the value their current domain name.

- Option #1: Using [ENS](https://ens.domains/)
- Option #2: Build a simple eWallet Name Service.

### Blockchain Wallets

Blockchain wallets will be the representations of Ethereum accounts in the eWallet. Two types of blockchain wallets will exist: `hot` and `cold`.

The eWallet saves the encrypted private keys of the `hot` wallets in the database, with the encryption key being stored in a file to which only the application has access. No private keys are stored with `cold` wallets and the administrators are responsible for managing them directly. Interaction with `cold` wallets will require manual action through [Metamask](https://metamask.io/) or similar tools.

See "Database Changes" for more information.

### Plasma Deposits and Exists

Every time funds are deposited on a plasma chain or exited, a deposit/exit will be recorded in the database.

### Ethereum / Plasma SDKs

In order to allow users owning their keys to access their funds from provider's mobile applications, we need to provide SDKs. Here are a few that we might consider building that would be able to handle deposits/exits when the user has his private key.

- JS SDK
- iOS SDK
- Android SDK
- Ruby SDK
- Elixir SDK

### eWallet Setup

The only mandatory step to plug an eWallet to Ethereum is to generate a new Ethereum account. The public key for that Ethereum account will be used as the main identifier for the eWallet. The private key will be encrypted and stored in the database, while the encryption key is stored in environment variables.

This value should be stored in the `Setting` table, under `primary_pub_key`

### Token creation

ERC-20 tokens will be creatable from the eWallet. Using a pre-defined Contract Template (see databases tables), a new token will be created on Ethereum by deploying the contract. Only ERC-20 will be supported at first. Providers will be able to create their own contract templates as long as they are compliant with ERC-20.

### Key Management Strategies

There are 3 different key management strategies that will be supported by the eWallet.

#### 1. Full Custody (providers own all the private keys)

Very close to the current state of things, except transactions between eWallets happen on the blockchain/Plasma. Users are not able to get their private keys.

#### 2. Optional User custody (providers own all the private keys by default, but users may request ownership of their private keys with multiple options)

Similar to Full Custody (and a transaction from that strategy is possible at anytime) with a twist: users are able to request ownership of their keys.

The major drawbacks for this strategy are:

- Users need to manage their private keys (backups, etc)
- Users need to deposit and exit their funds to/from Plasma themselves. This means making an actual OmiseGO wallet application makes a lot of sense to allow users to do exactly that (with notifications, etc.).

#### 3. Mandatory User custody (providers hold only their private keys, and all users must own their private keys)

Finally, with the "Mandatory User custody" strategy, all users will have to own and take care of their private keys themselves. This strategy needs to be set during the eWallet setup as a transaction from the other two strategies cannot be handled by the eWallet. A provider can, however, decide to switch to that strategy by handling the switching process with its users.

### Transaction Flows for one eWallet

The following flows are limited to transactions happening inside one eWallet.

#### Full custody

Full custody transactions inside an eWallet happen exactly in the same way as internal transactions. There is no interaction with the blockchain. Funds will be moved between internal wallets.

#### User custody (Optional or Mandatory)

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

### Transaction Flows for two eWallets

For the flows below, we'll use the following use-case:

```
A end-user wants to purchase something from eWallet A using a mobile application published by (and therefore connected to) eWallet B.
```

In the cases below, both eWallets have funds deposited in Plasma: eWallet A has `ABC` tokens while eWallet B has `ZYX` tokens.

#### eWallet A (Full custody) -> Ethereum address with regular transaction

1. Using eWallet B mobile app, a user wants to send funds to an Ethereum address.
2. The user submit the transaction to eWallet B, specifying how many tokens should be transferred and to which address.
3. eWallet B submits the transaction and let user knows about its progress/finality.

#### eWallet A (Full custody) -> eWallet B (Full custody) with regular transaction

1. Using eWallet B mobile app, a user wants to send funds to a user of eWallet A.
2. When submitting the transaction, the user needs to specify an eWallet's public key as well as an internal wallet address.
3. eWallet B sends the transaction to eWallet A on Plasma with some metadata containing the details of the transaction.
4. eWallet A receives the transaction and sends an event in the websocket channels for the given address, allowing eWallet A's user to be notified.

#### eWallet A (Full custody) -> eWallet B (Full custody) with transaction request

1. Using eWallet B mobile app, the user scans a QR code to pay for his purchase at eWallet A (10 `ABC`) using funds stored in eWallet B pooled wallets (`ZYX`). That QR code contains a string composed of an `ewallet_pub_key` and a transaction request ID.
2. eWallet B mobile application calls the `transaction_request.consume` endpoint sending the content of the QR code.
3. eWallet B detects that the received public key doesn't match its own, meaning the transaction request is hosted by another eWallet.
4. eWallet B then makes a call to the eWallet Ethereum Name Service sending the public key received in the consumption call. A domain name will be returned by the NS.
5. eWallet B sends a request to eWallet A to consume the request on behalf of the user. It receives a consumption ID back (with status `awaiting_funds`) and whether or not the tokens need to be converted (and if yes, to which currency) - this is defined by a conversion strategy inside the eWallet. In this case, eWallet A only wants to receive `ABC`. eWallet B connects to the websocket channel for the consumption.
6. eWallet B requests a quote from the DEX in order to know how many `ZYX` are needed to send 10 `ABC` to eWallet A. An event is sent to eWallet A about the progress.
7. The user receives the quote through websockets on eWallet B Mobile application and approves it. An event is sent to eWallet A about the progress.
8. eWallet B places the exchange order to get 10 `ABC`. Once the order has been matched, it sends the funds (10 `ABC`) to eWallet A address on Plasma. An event is sent to eWallet A about the progress.
9. eWallet A receives the Plasma event through the watcher, confirm the consumption and notifies eWallet B. eWallet B forward the event to the user's mobile application to let him know the transaction has been confirmed.
10. eWallet A POS/Admin panel receives an event in the websocket channel for the transaction request.
11. The user gets what he paid for.

#### eWallet's user (User custody) -> eWallet (Full custody)

1. Using eWallet B mobile app, the user scans a QR code to pay for his purchase at eWallet A (10 `ABC`) using funds stored in his Ethereum wallet (any currency). That QR code contains a string composed of an `ewallet_pub_key` and a transaction request ID.
2. eWallet B mobile application calls the `transaction_request.consume` endpoint sending the content of the QR code.
3. eWallet B detects that the received public key doesn't match its own, meaning the transaction request is hosted by another eWallet.
4. eWallet B then makes a call to the eWallet Ethereum Name Service sending the public key received in the consumption call. A domain name will be returned by the NS.
5. eWallet B sends a request to eWallet A to consume the request on behalf of the user. It receives a consumption ID back (with status `awaiting_funds`) and whether or not the tokens need to be converted (and if yes, to which currency) - this is defined by a conversion strategy inside the eWallet. In this case, eWallet A only wants to receive `ABC`. eWallet B connects to the websocket channel for the consumption.
6. eWallet B forward the details to the user's mobile application.
7. The user then needs to request a quote from the DEX directly in order to know how many of his choice of currency are needed to send 10 `ABC` to eWallet A. An event is sent to eWallet A/B about the progress.
8. The user receives the quote back from the DEX on the Mobile application and approves it. An event is sent to eWallet A/B about the progress.
9. The user deposits funds on Plasma in his choice of currency.
10. The user places the exchange order to get 10 `ABC` from the mobile application. Once the order has been matched, the user needs to send the funds (10 `ABC`) to eWallet A address on Plasma. An event is sent to eWallet A about the progress.
11. eWallet A receives the Plasma event through the watcher, confirm the consumption and notifies eWallet B. eWallet B forward the event to the user's mobile application to let him know the transaction has been confirmed.
12. eWallet A POS/Admin panel receives an event in the websocket channel for the transaction request.
13. The user gets what he paid for.

#### eWallet's user (User custody) -> eWallet's user (User custody)

1. Using eWallet B mobile app, the user scans a QR code to send tokens to his friends using eWallet A (10 `ABC`) using funds stored in his Ethereum wallet (any currency). That QR code contains a string composed of an `ewallet_pub_key` and a transaction request ID.
2. eWallet B mobile application calls the `transaction_request.consume` endpoint sending the content of the QR code.
3. eWallet B detects that the received public key doesn't match its own, meaning the transaction request is hosted by another eWallet.
4. eWallet B then makes a call to the eWallet Ethereum Name Service sending the public key received in the consumption call. A domain name will be returned by the NS.
5. eWallet B sends a request to eWallet A to consume the request on behalf of the user. It receives a consumption ID back (with status `awaiting_funds`) and whether or not the tokens need to be converted (and if yes, to which currency) - this is defined by a conversion strategy inside the eWallet. In this case, eWallet A's user only wants to receive `ABC`. eWallet B connects to the websocket channel for the consumption.
6. eWallet B forward the details to the user's mobile application, including the Ethereum address to which the funds need to be transferred.
7. The user then needs to request a quote from the DEX directly in order to know how many of his choice of currency are needed to send 10 `ABC` to eWallet A's user. An event is sent to eWallet A/B about the progress.
8. The user receives the quote back from the DEX on the Mobile application and approves it. An event is sent to eWallet A/B about the progress.
9. The user deposits funds on Plasma in his choice of currency. This should be done beforehand unless the blockchain can handle fast-enough transaction.
10. The user places the exchange order to get 10 `ABC` from the mobile application. Once the order has been matched, the user needs to send the funds (10 `ABC`) to eWallet A address on Plasma. An event is sent to eWallet A about the progress.
11. eWallet A receives the Plasma event through the watcher, confirm the consumption and notifies the user. eWallet B forward the event to the user's mobile application to let him know the transaction has been confirmed.
12. eWallet A's user receives an event in the websocket channel for the transaction request.

### Implementation

#### Code organization

Two new sub applications will be added to the eWallet project: the `blockchain_interface` and the `omg_adapter`.

##### `blockchain_interface`

The blockchain interface will define functions that need to be implemented in the adapters.

- `create_account`
- `create_token`
- `mint`
- `burn`
- `send_chain_transaction`
- `send_offchain_transaction`
- `get_quote`
- `place_exchange_order`
- `register_listener`

##### `omg_adapter`

- `create_account` (creates an Ethereum account)
- `create_eth_token` (publish a token smart contract on Ethereum)
- `mint` (adds more value to an existing token on Ethereum)
- `deposit` (deposit funds to the specified Plasma contract)
- `exit` (exit funds from the specified Plasma contract)
- `send_chain_transaction` (sends an Ethereum transaction)
- `send_offchain_transaction` (sends a Plasma transaction)
- `get_quote` (get a quote from Exchange)
- `place_exchange_order` (place an order for exchange)
- `register_listener` (register a GenServer as a listener to events from Ethereum and Plasma)

#### Database Changes

Table name: `contract_templates` (New)

- `uuid`
- `name`
- `content`
- `inserted_at`
- `updated_at`

Table name: `blockchain_wallet` (New)

- `uuid`
- `address`
- `pub_key`
- `encrypted_priv_key` (only for `hot` wallets, encrypted using a key defined in ENV variables)
- `type` (`hot`|`cold`)
- `primary` (`true`|`false`)
- `inserted_at`
- `updated_at`

Table name: `limits` (New)

- `uuid`
- `token_id`
- `low_amount`
- `high_amount`
- `limit_type` (`percent`|`value`)
- `blockchain_wallet_uuid`
- `inserted_at`
- `updated_at`

In case a blockchain wallet goes below or above the limits, a user will be notified that an action is required to keep the system secure (like moving funds in/out from/to a cold wallet.

Table name: `child_chain_contract` (New)

- `uuid`
- `contract_address`
- `active` (`true`|`false`)
- `inserted_at`
- `updated_at`

Table name: `deposit` (New)

- `uuid`
- `child_chain_contract_uuid`
- `amount`
- `token_id`
- `inserted_at`

Table name: `exit` (New)

- `uuid`
- `child_chain_contract_uuid`
- `inserted_at`

Table name: `exited_balances` (New)

- `uuid`
- `exit_uuid`
- `amount`
- `token_id`
- `inserted_at`

Table name: `wallet` (Existing)

- `blockchain_wallet_uuid` (`nullable`)

Table name: `token` (Existing)

- `contract_address` (`nullable`)
- `contract_template_uuid` (`nullable`)
- `status` (`pending`|`confirmed`)

Table name: `mint` (Existing)

- `status` (`pending`|`confirmed`)

Table name: `setting` (New)

- `key`
- `value`
- `inserted_at`
- `updated_at`

Table name: `transaction` (Existing)

- `type` (`internal`|`external`)

If the transaction is `external`, the addresses will be from blockchain wallets.

Table name: `transaction_request` (Existing)

- `blockchain_wallet_address`

Table name: `transaction_consumption` (Existing)

- `ewallet_pub_key`
- `blockchain_wallet_address`

Keys / Values:

`ewallet_identifier` : `pub_key` (`null` if not setup with blockchain)

#### Flows Implementations

This section describes how the flows explained earlier interact with the eWallet entities.

Coming soon.
