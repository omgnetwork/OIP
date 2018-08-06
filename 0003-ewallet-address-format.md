---
oip: "0003"
title: eWallet Address Format
status: Draft
type: eWallet
author: Thibault Denizet (thibault@omise.co)
created: 2018-07-26
---

# OIP-0003: eWallet Address Format

## Summary

Currently addresses are stored as UUID for wallets in the eWallet. This format is not really suited and was only meant as a temporary format. This OIP explores a few solutions to replace it with something more appropriate.

## Specification

A wallet address consists of 16 characters. The 16 characters comprise of 4 letters followed by 12 numeric digits.

There may be separators to help its readbility. The separator can be any printable character that are not `[A-Za-z0-9]`. The separator can be placed at any position in the address.

To ensure interoperability, the separators are stripped before usage and/or storage. Hence the followings all refer to the same wallet address:

- `abcd111122223333`
- `abcd_111122223333`
- `abcd-1111-2222-3333`
- `abcd-1111_2222_3333`
- `a-bcd-111-122-223-333`
- `abc-d11-112-222-333-3`

## Rationale

Allowing 4 letters gives the possibility to customize wallet addresses to suit the eWallet provider. It distinguishes the wallet address from credit card numbers to prevent confusion and any impact from existing policies and regulations regarding storing, transmitting and processing credit card numbers. It also opens up more available wallet addresses.

16 numeric digits = 10^16 = 1E16 possibilities
4 letters + 12 numeric digits = 4^26 * 10^16 = 4.56976E21 possibilities

Other explored options include:

### 1. UUID

The first option is to keep it as is, but UUID are long, hard to type (including the dashes, and all the alphanumeric characters) and can't be copied with one click (which is something I'd like to see in the new format).

### 2. ULID

The second option is pretty simple: we use the same format used for every external ID in the eWallet. The format for addresses would look like `adr_01ca2p8jqans5aty5gj5etmjcf`.

Still hard to type for a user and a bit too long, also contains both letters and numbers.

### 3. `this-is-my-address-yo`

Another option is to use either an auto-generated "passphrase:" with 3 to 5 words that would be used as an address. An added feature or alternative would be to allow users to enter their own passphrase address (but we'd probably end up with stupid and obcene ones...)

### 4. vvvv-mmmm-uuuu-uuu-c (3324-3239-8328-8438)

Credit card style, with digits for vendor identifier, user identifier, and check digit.

### 5. 324-3-234-1344

Bank account style (Thailand). What people are used to in Thailand.
