---
oip: "0002"
title: eWallet Internal Exchange Mechanism
status: Draft
type: eWallet
author: Unnawut Leepaisalsuwanna <unnawut@omise.co>
created: 2018-07-20
---

# OIP-0002: eWallet Internal Exchange Mechanism

## Simple Summary
Enables an eWallet server to perform exchanges and transfers from/to different minted tokens within its own eWallet environment.

## Abstract
This RFC describes the standard endpoints, flows, request/response formats, error codes needed to allow an eWallet server to perform cross-token exchanges within its own eWallet environment.

It is expected that there will be following RFCs for exchanges across eWallet environments and across the OMG network.

## Motivation
Prior to this RFC, the eWallet server is able to transfer values between accounts and/or users, if and only if the `from` and `to` are of the same minted token.

One of the main purposes of the eWallet is to enable exchanges of different minted tokens. Since, different mechanisms could be used and potentially impact future integrations with centralized & decentralized exchanges, it is important for this mechanism to be put up as an RFC and discussed before implementing.

## Specification

### Definitions

- **Internal exchange:** An exchange transaction that happens within the same eWallet environment.

- **External exchange:** An exchange transaction that involves a third-party, e.g. exchanging with a centralized or decentralized exchange services. The external exchange is out of scope of this RFC.

- **Exchange pair:** A pair of minted tokens that are allowed to be exchanged.

- **Same-token transfer:** A transaction whose its `from_address` and `to_address` are different, but `from_token_uuid` and `to_token_uuid` are the same.

- **Exchange transaction:** A transaction whose its `from_address` and `to_address` are the same, but the `from_token_uuid` and `to_token_uuid` are different.

- **Exchange transfer:** A transaction whose its `from_address` and `to_address` are different, and the `from_token_uuid` and `to_token_uuid` are also different.

### Data schemas

**EWalletDB.ExchangePair:** Contains the data about a pair of minted tokens that are allowed to be exchanged.

  - `uuid` (uuid)
  - `id` (string)
  - `name` (string)
  - `from_token_uuid` (uuid)
  - `to_token_uuid` (uuid)
  - `rate` (float)
  - `inserted_at` (datetime)
  - `updated_at` (datetime)
  - `deleted_at` (datetime)

For example, an exchange pair of `from_token: AAA`, `to_token: BBB`, `rate: 10.0` means that `1 AAA = 10 BBB`.

*The exchange rate stored is the rate of a token's main unit to another token's main unit, not subunits.*

**EWalletDB.Transfer:** Contains the data about each transaction that happened.

  - To add:
    - `from_amount` (integer)
    - `from_token_uuid` (uuid)
    - `to_amount` (integer)
    - `to_token_uuid` (uuid)
    - `exchange_pair_uuid` (uuid) - references `exchange_pair.uuid`
  - To migrate:
    - `from_amount` <- `amount`
    - `from_token_uuid` <- `token_uuid`
    - `to_amount` <- `amount`
    - `to_token_uuid` <- `token_uuid`
  - To remove:
    - `amount` (use `from_amount` and `to_amount` instead)
    - `token_uuid` (use `from_token_uuid` and `to_token_uuid` instead)

**LocalLedgerDB.Transaction:** A logical group of ledger entries that represents a single business event (e.g. a same-token transfer, an exchange transaction, an exchange transfer).

Each `EWalletDB.Transfer` item is one `LocalLedgerDB.Transaction`. One `LocalLedgerDB.Transaction` for an exchange has at least 4 `LocalLedgerDB.Entry`. For example:

  - transaction.type = “internal_exchange”
  - transaction.exchange_pair_id, e.g. "BTC/ETH"
  - transaction.exchange_rate, e.g. "1.00"
  - transaction.entry[0], e.g. credit user 10 OMG
  - transaction.entry[1], e.g. debit exchange account 10 OMG
  - transaction.entry[2], e.g. credit exchange account 1 ETH
  - transaction.entry[3], e.g. debit user 1 ETH

### Mechanisms

For internal exchanges, the operation happens at the DB's transaction level so it should be seamlessly handled within the eWallet server as long as the exchange operations are wrapped inside a single DB transaction.

At the API level, this can be represented as a typical `/transaction.create` endpoint, where the only difference is that for exchanges, the `from_token_id` and `to_token_id` have different values.

#### Same-token transfers (existing feature)

![same-token transfer diagram](https://user-images.githubusercontent.com/921194/41035018-995015b6-69b5-11e8-9d67-578659b864de.png)

1. Alice has 100 AAA
2. Alice wants to send 10 AAA to Bob
2. Alice sends 10 AAA to Bob
3. Bob receives the 10 AAA from Alice

##### Example:

```json
(Assuming subunit_to_unit = 1 for simplicity)

# EWalletDB.Transfer
{
  "id": "tfr_1234567890",
  "type": "internal",
  "from": "bal_alice_1111",
  "from_amount": 10,
  "from_token_id": "tok_AAA_1234",
  "to": "bal_bob_2222",
  "to_amount": 10,
  "to_token_id": "tok_AAA_1234"
}

# LocalLedgerDB.Transaction
{
  "uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310"
}

# LocalLedgerDB.Entry
[
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "debit",
    "amount": 10,
    "wallet_address": "bal_alice_1111",
    "token_id": "tok_AAA_1234"
  },
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "credit",
    "amount": 10,
    "wallet_address": "bal_bob_2222",
    "token_id": "tok_AAA_1234"
  }
]
```

#### Exchange transaction (proposed by this RFC)

This is when a user wants to exchange a token for another, where the sender and the receiver is the same address.

![exchange transaction diagram](https://user-images.githubusercontent.com/921194/41035308-7241b8d4-69b6-11e8-9ff7-ad109a3c0159.png)

1. Alice has 100 AAA
2. Alice wants to exchange 10 AAA for 1 BBB with ExchangeAccount
3. Alice sends 10 AAA to ExchangeAccount
4. ExchangeAccount sends 1 BBB to Alice
5. If 3. or 4. fails, the transaction is reverted

##### Example:

```json
(Assuming subunit_to_unit = 1 for simplicity)

# EWalletDB.Transfer
{
  "id": "tfr_1234567890",
  "type": "exchange",
  "from": "bal_alice_1111",
  "from_amount": 10,
  "from_token_id": "tok_AAA_1234",
  "to": "bal_alice_1111",
  "to_amount": 1,
  "to_token_id": "tok_BBB_1234"
}

# LocalLedgerDB.Transaction
{
  "uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310"
}

# LocalLedgerDB.Entry
[
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "debit",
    "amount": 10,
    "wallet_address": "bal_alice_1111",
    "token_id": "tok_AAA_1234"
  },
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "credit",
    "amount": 10,
    "wallet_address": "bal_exchange_2222",
    "token_id": "tok_AAA_1234"
  },
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "debit",
    "amount": 1,
    "wallet_address": "bal_exchange_2222",
    "token_id": "tok_BBB_1234"
  },
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "credit",
    "amount": 1,
    "wallet_address": "bal_alice_1111",
    "token_id": "tok_BBB_1234"
  }
]
```

#### Exchange transfers (proposed by this RFC)

This is when a sender wants to transfer a token and expects the receiver at a different address to receive a different token.

![exchange transfer diagram](https://user-images.githubusercontent.com/921194/41035234-399ca232-69b6-11e8-8a68-a7cb98b3c32e.png)

1. Alice has 100 AAA
2. Alice wants to send 1 BBB to Bob using his/her funds of 100 AAA via ExchangeAccount that has liquidity
3. Bob receives the 10 OMG from Alice (via ExchangeAccount)

##### Example:

```json
(Assuming subunit_to_unit = 1 for simplicity)

# EWalletDB.Transfer
{
  "id": "tfr_1234567890",
  "type": "exchange_transfer",
  "from": "bal_alice_1111",
  "from_amount": 10,
  "from_token_id": "tok_AAA_1234",
  "to": "bal_bob_2222",
  "to_amount": 1,
  "to_token_id": "tok_BBB_1234"
}

# LocalLedgerDB.Transaction
{
  "uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310"
}

# LocalLedgerDB.Entry
[
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "debit",
    "amount": 10,
    "wallet_address": "bal_alice_1111",
    "token_id": "tok_AAA_1234"
  },
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "credit",
    "amount": 10,
    "wallet_address": "bal_exchange_3333",
    "token_id": "tok_AAA_1234"
  },
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "debit",
    "amount": 1,
    "wallet_address": "bal_exchange_3333",
    "token_id": "tok_BBB_1234"
  },
  {
    "transaction_uuid": "44d7fc38-b250-4456-a4e8-0d23dc52f310",
    "type": "credit",
    "amount": 1,
    "wallet_address": "bal_bob_2222",
    "token_id": "tok_BBB_1234"
  }
]
```

### Rounding method

The rounding method to use is "Round half to even", as known as the banker's rounding.

### Exchange rates

When doing the exchange, the actual rate to use for calculation needs to take into account the difference in each token's `subunit_to_unit` value, and scale the result by the tokens' difference in subunit scale. For example:

```
exchanged_value = original_value * exchange_rate * (target_token.subunit_to_unit / source_token.subunit_to_unit)
```

Transactions are always made with the exchange amounts, not the rate. Although an exchange transaction is based on the exchange rate, the parties agree on the exchanging amounts, not the rate. This is important because the division remainders and the rounding will almost always cause the exchanging parties to get more or less tokens depending on the rounding method used.

Consequently, the UI must display the exchanging amounts and asks user to confirm the amount, not the rate. The rate can be displayed as a complimenting information.

### `/transaction.calculate` endpoint

Calculate the amount of minted tokens to be exchanged, used for confirming the exact amount with the user before executing the actual exchange.

  - Use this endpoint to avoid discrepancies between how endpoint consumer and the server calculates the exchange amount.
  - Errors are returned if it is anticipated to happen in an actual transfer.

#### Request

##### Fields

- `from_amount` (integer, optional)
- `from_token_id` (string, required)
- `to_amount` (integer, optional)
- `to_token_id` (string, required)

###### Conditions

- If `from_amount` is provided, `to_amount` will be calculated based on the exchange rate and rounding method.
- If `to_amount` is provided, `from_amount` will be calculated based on the exchange rate and rounding method.
- If both `from_amount` and `to_amount` are provided, and the amounts are the same as the server's calculation, a successful response is returned.
- If both `from_amount` and `to_amount` are provided, but the amounts are different from the server's calculation, an `exchange:invalid_rate` response is returned.

##### Example

```json
{
  "from_amount": 100,
  "from_token_id": "tok_AAA_1234",
  "to_amount": 100,
  "to_token_id": "tok_BBB_1234"
}
```

#### Response

##### Fields

- `version` (integer)
- `success` (boolean)
- `data` (object)
  - `object` (string)
  - `from_amount` (integer)
  - `from_token_id` (string)
  - `to_amount` (integer)
  - `to_token_id` (string)
  - `exchange_pair` (object)
    - `object` (string)
    - `id` (string)
    - `name` (string)
    - `rate` (float)
  - `calculated_at` (datetime)

##### Example

```json
{
  "version": "1",
  "success": true,
  "data": {
    "object": "exchange_calculation",
    "from_amount": 100,
    "from_token_id": "tok_AAA_1234",
    "to_amount": 100,
    "to_token_id": "tok_BBB_1234",
    "exchange_pair": {
      "object": "exchange_pair",
      "id": "exg_AAA_BBB_123456",
      "name": "AAA/BBB",
      "rate": "1.00",
    },
    "calculated_at": "2018-01-01T00:00:00Z"
  }
}
```

### `/transaction.create` endpoint

Exchange a certain amount of minted tokens between two addresses.

#### Request

##### Fields

- `from_amount` (integer, optional)
- `from_token_id` (string, required)
- `from_address` (string, required)
- `to_amount` (integer, optional)
- `to_token_id` (string, required)
- `to_address` (string, required)
- `exchange_account_id` (string, required)
- `sync_opposite` (boolean, optional)

The request may be sent without `from_amount` or `to_amount` and subjects to the same conditions as `/exchange.calculate`.

##### Example

```json
{
  "from_token_id": "tok_AAA_1234",
  "from_address": "81e75f46-ee14-4e4c-a1e5-cddcb26dce9c",
  "to_amount": 100,
  "to_token_id": "tok_BBB_1234",
  "to_address": "4aa07691-2f99-4cb1-b36c-50763e2d2ba8",
  "exchange_account_id": "acc_01ca2p8jqans5aty5gj5etmjcf",
  "metadata": {},
  "encrypted_metadata": {}
  "sync_opposite": true
}
```

#### Response

##### Fields

- `version` (integer)
- `success` (boolean)
- `data` (object)
  - `object` (string)
  - `id` (string)
  - `idempotency_token` (string)
  - `from` (object)
    - `object` (string)
    - `address` (string)
    - `amount` (integer)
    - `token` (object)
      - `object` (string)
      - `id` (string)
      - `symbol` (string)
      - `name` (string)
      - `subunit_to_unit` (integer)
  - `to` (object)
    - `object` (string)
    - `address` (string)
    - `amount` (integer)
    - `token` (object)
      - `object` (string)
      - `id` (string)
      - `symbol` (string)
      - `name` (string)
      - `subunit_to_unit` (integer)
  - `exchange` (object)
    - `object` (string)
    - `id` (string)
    - `from_token_id` (string)
    - `to_token_id` (string)
    - `rate` (float)
  - `metadata` (object)
  - `encrypted_metadata` (object)
  - `status` (string)
  - `created_at` (datetime)
  - `updated_at` (datetime)

##### Example

```json
{
  "version": "1",
  "success": true,
  "data": {
    "object": "transaction",
    "id": "txn_01cbfg7hq1v0v0fq1csn2w2n29",
    "idempotency_token": "12345",
    "from": {
      "object": "transaction_source",
      "address": "XXX123",
      "amount": 1000,
      "token": {
        "object": "token",
        "id": "tok_AAA_01cbfg349rb8akcvs7vx20a3g0",
        "symbol": "AAA",
        "name": "AAA Point",
        "subunit_to_unit": 100
      }
    },
    "to": {
      "object": "transaction_source",
      "address": "XXX123",
      "amount": 100,
      "token": {
        "object": "token",
        "id": "tok_ZZZ_01cbfg349rb8akcvs7hq1vzv0d31",
        "symbol": "ZZZ",
        "name": "ZZZ Point",
        "subunit_to_unit": 100
      }
    },
    "exchange": {
      "object": "exchange_pair",
      "from_token_id": "tok_AAA_01cbfg349rb8akcvs7vx20a3g0",
      "to_token_id": "tok_ZZZ_01cbfg349rb8akcvs7hq1vzv0d31",
      "rate": 10,
    },
    "metadata": {},
    "encrypted_metadata": {},
    "status": "confirmed",
    "created_at": "2018-01-01T00:00:00Z",
    "updated_at": "2018-01-01T10:00:00Z"
  }
}
```

### Error codes

- `transaction:insufficient_funds` - when the requestor does not have enough funds to exchange.
- `exchange:insufficient_funds` - when the counterparty does not have enough funds to exchange back. It is important to distinguish between the user and the counterparty so that user-friendly error message can be displayed, and the user knows if its the counterparty's funds or theirs that is insufficient.
- `exchange:pair_not_found` - when the request requires an exchange pair that does not exist.
- `exchange:invalid_rate` - when the provided exchange amounts or the exchange rate is invalid.
- `exchange:pair_already_exists` - when the request attempts to create a pair that already exists.
- `exchange:opposite_pair_not_found` - when the request asks for `sync_opposite: true` but the opposite pair could not be found.

## Rationale

1. Round-half-to-even or banker's rounding is used as it is commonly used in financial systems. It is also the default rounding mode for IEEE 754 (IEEE Standard for Floating-Point Arithmetic).

2. An `exchange_account_id` needs to be specified for separation of responsibility of each account. This `exchange_account_id` would also allow for more use cases, e.g. dedicate an account as the exchange account and put funds in there. It is then possible to limit the amount of tokens that can be exchanged by limiting the funds that the account has.

3. This RFC chose to reuse the `/transaction.create` endpoint because of its similarities to an exchange transaction and exchange transfer. An exchange transaction is basically a transfer of different tokens to the same address, while an exchange transfer can be represented by a transfer that has different from/to tokens.

4. There could be a discrepancy between the time `/transaction.calculate` is made and when `/transaction.create` happens. Given that this is an internal exchange and exchange rates are fixed, unchanged for most of the time. The discrepancy will be very rare to occur.

5. The exchange rate is stored as a main unit to main unit ratio, not subunits. This is for readability and is subunit-agnostic. So in order to exchange between tokens with different `subunit_to_unit` values, one need to take into account the difference in scale of the subunits.
