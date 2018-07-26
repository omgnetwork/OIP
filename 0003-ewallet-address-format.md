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

I'm not sure which one is best, but here are 7 formats that @unnawut and I came up with:

### 1. UUID

The first option is to keep it as is, but UUID are long, hard to type (including the dashes, and all the alphanumeric characters) and can't be copied with one click (which is something I'd like to see in the new format).

### 2. ULID

The second option is pretty simple: we use the same format used for every external ID in the eWallet. The format for addresses would look like `adr_01ca2p8jqans5aty5gj5etmjcf`.

Still hard to type for a user and a bit too long, also contains both letters and numbers.

### 3. xxxx_332493849383

Third option is using 16 characters (4 letters + 12 digits) to have a format that looks like `abcd_111122223333` and could be shown to the user as `abcd-1111-2222-3333` with 4 cases to fill (giving the same feeling as credit card, minus the 4 characters at the beginning).

Why use those 4 characters? To increase the potential number of addresses.

10^16 = 1E16 possibilities
4^26 * 10^16 = 4.56976E21 possibilities

4 characters are easy to remember and 12 digits is just two more than a phone number so with this, I believe anyone should be able to remember their address.

### 4. `this-is-my-address-yo`

Another option is to use either an auto-generated "passphrase:" with 3 to 5 words that would be used as an address. An added feature or alternative would be to allow users to enter their own passphrase address (but we'd probably end up with stupid and obcene ones...)

### 5. vvvv-mmmm-uuuu-uuu-c (3324-3239-8328-8438)

Credit card style, with digits for vendor identifier, user identifier, and check digit.

### 6. 324-3-234-1344

Bank account style (Thailand). What people are used to in Thailand.

## More options?

If you guys have more ideas and think we should go with one of the format, let me know! I don't think there's a right answer, they all have their pros and cons, but let's try to find the best one for us!
