# LSPS7 Channel Lease Extensions

| Name     | `channel_lease_extensions` |
|----------|-------------------------|
| Version  | 1                       |
| Status   | Draft                   |

LSPS7 requires [LSPS0](../LSPS0/) and therefore [BOLT8](https://github.com/lightning/bolts/blob/master/08-transport.md) as a transport layer. Much of the design here takes from that of [LSPS1][], in the spirit of uniformity.


## Motivation

The goal of this specification is to provide a standardized LSP API for wallets to extend the lifetime of a channel purchased from an LSP.

## Order Flow Overview

* Client calls `lsps7.get_extendable_channels` to get the LSP's options.
* Client calls `lsps7.create_order` to create an order.
* Client pays the order either on-chain or off-chain.
* LSP extends the expiry time of the channel as soon as the payment is confirmed.


## API

### 1. lsps7.get_extendable_channels

| JSON-RPC Method | lsps7.get_extendable_channels |
|---------------- |----------- |
| Idempotent      | Yes        |


`lsps7.get_extendable_channels` is the entrypoint for each client using the API. It lists all options in a dictionary. 

The client SHOULD call `lsps7.get_extendable_channels` first.

**Request** No parameters needed.

**Response**

```JSON
{
  "extendable_channels": [
    {
        "original_order": {
            "id": "3c2ef2b-195f-0695-b72ef-3929b9138e04",
            "service": "LSPS1"
        },
        "extension_order_ids": [
            "142b3ef8-2eca-6135-7f74-7d1afda8b835",
            "39afg2a3-7cf7-f3b6-056e-43f0d5b61f3f"
        ],
        "short_channel_id": "871428x964x0",
        "max_channel_extension_expiry_blocks": 300,
        "expiration_block": 839230
    }
  ]
}
```

`extendable_channels` is an array of channels the LSP will allow you to extend the expiry of. These channels have the following properties:

- `original_order <object>` The original order for the channel lease, including the `id <string>` to identify the order and the `service <string>` the purchase occured on.
- `extension_order_ids <string[]>` A list of order ids for each time the channel lease has been extended.
- `short_channel_id` <[LSPS0.scid][]> Short Channel Identifier (SCID) of the channel.
- `max_channel_extension_expiry_blocks <uint32>` The maximum number of blocks a channel can be leased for.
- `expiration_block <uint32>` The block height at which the channel lease will expiry.
  - MUST be 1 or greater.

> **Rationale `original_order`** The client MAY want to look up the original order for reference.

> **Rationale `extension_order_ids`** The client MAY want to look up any one of the previous extensions orders for reference.


**Errors** No additional errors are defined for this method.

### 2. lsps7.create_order 

| JSON-RPC Method     | lsps7.create_order |
|-------------------- |------------------- |
| Idempotent          | No                 |


The request is constructed depending on the client's needs. 

**Request**

```json
{
  "short_channel_id": "871428x964x0",
  "channel_extension_expiry_blocks": 144,
  "token": "",
  "refund_onchain_address": "bc1qvmsy0f3yyes6z9jvddk8xqwznndmdwapvrc0xrmhd3vqj5rhdrrq6hz49h"
}
```

- `short_channel_id` <[LSPS0.scid][]> Short Channel Identifier (SCID) of the channel to extend.
- `channel_extension_expiry_blocks <uint32>` How long to extend the lease of the channel in block time.
  - MUST be 1 or greater. 
  - MUST be below or equal to `max_channel_extension_expiry_blocks` returned from the `get_extendable_channels` call.
- `token <string>` Field for arbitrary data like a coupon code or a authentication token.
  - Client MAY omit this field.
- `refund_onchain_address <string>` <LSPS0.onchain_address> Address where the LSP will send the funds if the order fails.
  - Client MAY omit this field.

**Response**

```json
{
  "order_id": "bb4b5d0a-8334-49d8-9463-90a6d413af7c",
  "short_channel_id": "871428x964x0",
  "channel_extension_expiry_blocks": 144,
  "new_channel_expiry_blocks": 839374,
  "token": "",
  "created_at": "2012-04-23T18:25:43.511Z",
  "order_state": "CREATED",
  "payment": {
    "bolt11": {
      "state": "EXPECT_PAYMENT",
      "expires_at": "2015-01-25T19:29:44.612Z",
      "fee_total_sat": "8888",
      "order_total_sat": "2008888",
      "invoice" : "lnbc252u1p3aht9ysp580g4633gd2x9lc5al0wd8wx0mpn9748jeyz46kqjrpxn52uhfpjqpp5qgf67tcqmuqehzgjm8mzya90h73deafvr4m5705l5u5l4r05l8cqdpud3h8ymm4w3jhytnpwpczqmt0de6xsmre2pkxzm3qydmkzdjrdev9s7zhgfaqxqyjw5qcqpjrzjqt6xptnd85lpqnu2lefq4cx070v5cdwzh2xlvmdgnu7gqp4zvkus5zapryqqx9qqqyqqqqqqqqqqqcsq9q9qyysgqen77vu8xqjelum24hgjpgfdgfgx4q0nehhalcmuggt32japhjuksq9jv6eksjfnppm4hrzsgyxt8y8xacxut9qv3fpyetz8t7tsymygq8yzn05"
    },
    "onchain": {
      "state": "EXPECT_PAYMENT",
      "expires_at": "2015-01-25T19:29:44.612Z",
      "fee_total_sat": "9999",
      "order_total_sat": "2009999",
      "address" : "bc1p5uvtaxzkjwvey2tfy49k5vtqfpjmrgm09cvs88ezyy8h2zv7jhas9tu4yr",
      "min_fee_for_0conf": 253,
      "min_onchain_payment_confirmations": 0,
      "refund_onchain_address": "bc1qvmsy0f3yyes6z9jvddk8xqwznndmdwapvrc0xrmhd3vqj5rhdrrq6hz49h"
    }
  },
  "channel": {
    "short_channel_id": "871428x964x0",
    "expires_at": "2024-08-19T19:43:20.529Z",
    "funded_at": "2024-05-21T19:43:20.529Z",
  },
}
```

- `order_id <string>` Id of this specific order.
  - MUST be unique.
  - MUST be at most 64 characters long.
  - SHOULD be a valid [UUID version 4](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_(random)) (aka random UUID).
- `short_channel_id` <[LSPS0.scid][]> Mirrored from the request.
- `funding_confirms_within_blocks <uint16>` Mirrored from the request.
- `token <string>` Mirrored from the request.
  - MUST be an empty string if the token was not provided.
- `created_at` <[LSPS0.datetime][]> Datetime when the order was created.
- `order_state <string enum>` Current state of the order.
  - `CREATED` Order has been created. Default value.
  - `COMPLETED` LSP has published funding transaction.
  - `FAILED` Order failed.
- `payment <object>` Contains everything about payments, see [3. Payment](#3-payment).
- `channel <object>` Contains information about the channel, see [4 Channel](#4-channel).


**Client**
- SHOULD validate that `channel_extension_expiry_blocks` + `channel_extension_expiry_blocks` = `new_channel_expiry_blocks`.
- SHOULD validate the `fee_total_sat` is reasonable.
- MAY abort the flow here.

**Errors**

| Code   | Message         | Data | Description |
| ----   | -------         | ----------- | ---- |
| -32602 | Invalid params  | {"property": %invalid_property%, "message": %human_message% }            | Invalid method parameter(s). |
| 001    | Client rejected | {"message": %human_message% }                                            | [LSPS0.client_rejected_error][] |
| 100    | Option mismatch | {"property": %option_mismatch_property%, "message": %human_message% }    | The order doesnt match the options defined in `lsps7.get_extendable_channels` channel object. |


- LSP MUST validate the order against the options defined in the `lsps7.get_extendable_channels` channel object. LSP MUST return an `100` error in case of a mismatch.
  - `%option_mismatch_property%` MUST be one of the fields in the `lsps7.get_extendable_channels` channel object.
  - Example: `{ "property": "max_channel_extension_expiry_blocks" }`.

- LSP MUST validate the request fields. LSP MUST return a `-32602` error in case of an invalid request field.
  - `%invalid_property%` MUST be one of the fields in the request body. MUST use `.` to separate nested fields.
  - Example: `{ "property": "channel_extension_expiry_blocks", "message": "Not an integer" }`.

- LSP MUST validate the `token` field and return an error if the token is invalid.

> **Rationale `token` validation** The client should be informed if the token is invalid. Ignoring the invalid token and creating an order without the potentially discount or other side effect is not good UX. Ignoring the invalid token will also NOT prevent anybody bruteforcing the token because the client will still detect if the LSP has given a discount.

> **Rationale Client rejected** LSPs can reject a client for example for misbehaviour. LSPs can reject a node on two levels: Prevent a peer connection OR disable order creation. Preventing a peer connection might not work in case you still want to allow other functions to keep working, for example an existing channel.


### 2.1 lsps7.get_order

| JSON-RPC Method | lsps7.get_order |
|---------------- |---------------- |
| Idempotent      | Yes             |

The client MAY check the current status of the order at any point.

**Request**

```json
{
  "order_id": "bb4b5d0a-8334-49d8-9463-90a6d413af7c"
}
```

**Response** is the same as defined in `lsps7.create_order`.

**Errors**

| Code   | Message   | Data    | Description                                           |
| ------ | --------- | ------- | ----------------------------------------------------- |
| 101    | Not found | {}      | Order with the requested order_id has not been found. |


### 3. Payment

This section describes the `payment` object returned by `lsps7.create_order` and `lsps7.get_order`. It mirrors the formatting of the `payment` object in [LSPS1][].

```json
{
  "bolt11": {
    "state" : "EXPECT_PAYMENT",
    "expires_at": "2025-01-01T00:00:00Z",
    "fee_total_sat": "8888",
    "order_total_sat": "200888",
    "invoice": "lnbc252u1p3aht9ysp580g4633gd2x9lc5al0wd8wx0mpn9748jeyz46kqjrpxn52uhfpjqpp5qgf67tcqmuqehzgjm8mzya90h73deafvr4m5705l5u5l4r05l8cqdpud3h8ymm4w3jhytnpwpczqmt0de6xsmre2pkxzm3qydmkzdjrdev9s7zhgfaqxqyjw5qcqpjrzjqt6xptnd85lpqnu2lefq4cx070v5cdwzh2xlvmdgnu7gqp4zvkus5zapryqqx9qqqyqqqqqqqqqqqcsq9q9qyysgqen77vu8xqjelum24hgjpgfdgfgx4q0nehhalcmuggt32japhjuksq9jv6eksjfnppm4hrzsgyxt8y8xacxut9qv3fpyetz8t7tsymygq8yzn05"
  },
  "onchain": {
    "state": "EXPECT_PAYMENT",
    "expires_at": "2025-01-01T00:00:00Z",
    "fee_total_sat": "9999",
    "order_total_sat": "200999",
    "address": "bc1p5uvtaxzkjwvey2tfy49k5vtqfpjmrgm09cvs88ezyy8h2zv7jhas9tu4yr",
    "min_onchain_payment_confirmations": 1,
    "min_fee_for_0conf": 253,
    "refund_onchain_address": "bc1qvmsy0f3yyes6z9jvddk8xqwznndmdwapvrc0xrmhd3vqj5rhdrrq6hz49h"
  }
}
```

The Client MUST ignore unkown payment options.

The LSP MAY omit payment options.

> **Rationale**: E.g.: The LSP might not support onchain payments.

> **Rationale**:
> Fees and expiry dates are defined on the level of the payment-option.
> This allows an LSP to provide the cheapest service for each payment option.
>
> An LSP might choose to have a lower fee for lightning than for onchain payments.
> Onchain payments might
> - be more costly for the LSP as they might have to spend a small UTXO in the future
> - be more risky for the LSP in case of 0-conf channels
> - require longer confirmation times than a lightning payment. (Risk of changing on-chain fees)


#### 3.1 Lightning Payments using BOLT-11

```json
{
    "state" : "EXPECT_PAYMENT",
    "expires_at": "2025-01-01T00:00:00Z",
    "fee_total_sat": "8888",
    "order_total_sat": "200888",
    "invoice": "lnbc252u1p3aht9ysp580g4633gd2x9lc5al0wd8wx0mpn9748jeyz46kqjrpxn52uhfpjqpp5qgf67tcqmuqehzgjm8mzya90h73deafvr4m5705l5u5l4r05l8cqdpud3h8ymm4w3jhytnpwpczqmt0de6xsmre2pkxzm3qydmkzdjrdev9s7zhgfaqxqyjw5qcqpjrzjqt6xptnd85lpqnu2lefq4cx070v5cdwzh2xlvmdgnu7gqp4zvkus5zapryqqx9qqqyqqqqqqqqqqqcsq9q9qyysgqen77vu8xqjelum24hgjpgfdgfgx4q0nehhalcmuggt32japhjuksq9jv6eksjfnppm4hrzsgyxt8y8xacxut9qv3fpyetz8t7tsymygq8yzn05"
}
```

- `state`
    - `EXPECT_PAYMENT` Payment expected.
    - `HOLD` Lighting payment arrived, preimage NOT released.
    - `PAID`  When the has been preimage released
    - `REFUNDED` Lightning payment has been refunded.
    - <del>`CANCELLED` Lightning payment has been cancelled.</del> This state has been deprecated.
      - The LSP MUST use `REFUNDED` for a cancelled or refunded invoice.
      - Earlier versions of this spec MAY still use this state. 
      - Clients SHOULD still support this state for backwards compatibility.
- `expires_at` <[LSPS0.datetime][]> The timestamp at which the payment option for this order expires
- `fee_total_sat` <[LSPS0.sat][]> The total fee the LSP will charge to extend the lease of this channel in satoshi.
- `order_total_sat` <[LSPS0.sat][]> What the client needs to pay in total to  extend the lease of the requested channel.
    - MUST be the `fee_total_sat` plus the `client_balance_sat` requested in satoshi.
- `invoice <string>`
    - MUST be a [Lightning BOLT 11 invoice](https://github.com/lightning/bolts/blob/master/11-payment-encoding.md) for the number of `order_total_sat`. 
    - Invoice MUST be a [HOLD invoice](https://bitcoinops.org/en/topics/hold-invoices/).
    - MUST be at most 2048 characters long.

**Client**

- MAY pay the `invoice`.
- MAY pull `lsps7.get_order` to check the success of the payment.
- The client gets refunded automatically in case the channel lease extension failed, the order expires, or just before the payment times out.

**LSP**

- MUST change the `payment.state` to `HOLD` when the payment arrived.
- If the channel lease updated successfully
    - MUST release the preimage and therefore complete the payment.
    - MUST set the `payment.state` to `PAID`.
- If the channel lease failed to update or the order expired or shortly before the payment times out:
    - MUST reject the payment.
    - MUST set the `payment.state` to `CANCELLED`.

#### 3.2 Onchain payments

Onchain payments dictionary MUST be omitted if the client didn't provide a `refund_onchain_address` 
and/or the LSP disabled onchain payments.

```json
{
    "state": "EXPECT_PAYMENT",
    "expires_at": "2025-01-01T00:00:00Z",
    "fee_total_sat": "9999",
    "order_total_sat": "200999",
    "address": "bc1p5uvtaxzkjwvey2tfy49k5vtqfpjmrgm09cvs88ezyy8h2zv7jhas9tu4yr",
    "min_onchain_payment_confirmations": 1,
    "min_fee_for_0conf": 253,
    "refund_onchain_address": "bc1qvmsy0f3yyes6z9jvddk8xqwznndmdwapvrc0xrmhd3vqj5rhdrrq6hz49h"
}
```

- `state`
  - `EXPECT_PAYMENT`: Payment expected
  - `PAID`: Onchain payment is confirmed
  - `REFUNDED`: Onchain payment has been refunded
- `expires_at` <[LSPS0.datetime][]> The timestamp at which the payment option for this order expires
- `fee_total_sat` <[LSPS0.sat][]> The total fee the LSP will charge to extend this channel lease in satoshi.
- `order_total_sat` <[LSPS0.sat][]> What the client needs to pay in total to extend the lease of the requested channel.
    - MUST be the `fee_total_sat` plus the `client_balance_sat` requested in satoshi.
- `address` <[LSPS0.onchain_address][]> On-chain address the client can pay the `order_total_sat` to
- `min_onchain_payment_confirmations <uint16>` Minimum number of block confirmations that are required for the on-chain payment to be considered confirmed.
    - MUST be equal or greater than `options.min_onchain_payment_confirmations`.
- `min_fee_for_0conf <LSPS0.onchain_fee>` Fee rate for on-chain payment in case the client wants the payment to be confirmed without a confirmation.
    - MUST be `null` or absent if `min_onchain_payment_confirmations` is greater than 0.
    - SHOULD choose a high enough fee to lower the risk of a double spend.
- `refund_onchain_address` <[LSPS0.onchain_address][]> Client supplied refund address.
    - LSP SHOULD set this to mirror the order creation request.
    - LSP MAY omit this field as it wasn't present in earlier versions of this specification.

> **Rationale `min_onchain_payment_confirmations`** The main risk for an LSP is that the client pays the on-chain payment and then double spends the transaction. This is especially critical in case the client requested a high `client_balance`. Opening a 0conf channel alone has no risk attached to it IF the on-chain payment is confirmed. Therefore, the LSP can mitigate this risk by waiting for a certain number of block confirmations before extending the channel lease.

> **Rationale `min_fee_for_0conf`** The client MAY want to have instant confirmation of the on-chain payment. The LSP can mitigate the risk of a double spend by requiring a high fee rate. The client can then decide if he wants to pay the high fee rate or wait for the on-chain payment to be confirmed once.

**Client**

- MUST pay `order_total_sat` to `address`.
- MAY pull `lsps7.get_order` to check the success of the payment.

**LSP** Payment confirmation

- MUST monitor the blockchain and update `onchain_payment`.
- IF `min_onchain_payment_confirmations	` is 0 and incoming transaction fee is greater than `min_fee_for_0conf`:
  - SHOULD set the transaction as confirmed.
- IF `min_onchain_payment_confirmations	` is equal or greater than 1:
  - SHOULD set the transaction as confirmed after `min_onchain_payment_confirmations` confirmations.
- MAY always set the transaction as confirmed before `min_onchain_payment_confirmations` confirmations.
- In rare circumstances, MAY take longer than `min_onchain_payment_confirmations` confirmations in case the LSP doesn't trust the transaction.
- MUST always set the transaction as confirmed after 6 block confirmations.


**LSP** State change

- MUST change the `payment.state` to `PAID` when the payment for the order is confirmed.
- If the order expired and the channel lease has NOT been updated, OR the channel lease update failed.
    - MUST refund the client to `refund_onchain_address`.
      - The number of satoshi to refund 
        - MUST be `order_total_sat` MINUS transaction size * onchain fee rate. The LSP MUST choose a reasonable fee rate.
        - MAY overprovision.
      - The LSP MUST bump the fee in case the transaction doesn't resolve within 6 hours.
    - MUST set the payment `payment.state` to `REFUNDED`.


### 4. Channel

**Channel object**

```json
{
  "short_channel_id": "871428x964x0",
  "funded_at": "2012-04-23T18:25:43.511Z",
  "expires_at": "2012-04-23T18:25:43.511Z"
}
```

- `channel <object>` Contains channel information. 
  - `short_channel_id` <[LSPS0.scid][]> Short Channel Identifier (SCID) of the channel.
  - `funded_at` <[LSPS0.datetime][]> Datetime when the funding transaction has been published.
  - `expires_at` <[LSPS0.datetime][]> Earliest datetime when the channel MAY be closed by the LSP.
    - MUST respect `channel_expiry_blocks`. 
    - MAY overprovision.

#### 4.1 Channel successfully extended

The LSP MUST update the channel channel lease (extending the expiry) under the following conditions:
- `payment.state` is `HOLD` (lightning) or `PAID` (on-chain).

**LSP**
- MUST attempt a channel lease extension.
    - MUST update `expiration_block` on the channel (`lsps7.get_extendable_channels`) and `expires_at` on the `channel` object where applicable.
        - MAY overprovision.

In case the channel lease extension succeeds
- MUST set `order_state` to `COMPLETED`.
- MUST update the channel object.

In case the channel lease extension failed
- MUST set `order_state` to `FAILED`.
- MUST issue a refund.




[LSPS0.common_schemas]: ../LSPS0/common-schemas.md
[LSPS0.sat]: ../LSPS0/common-schemas.md#link-lsps0sat
[LSPS0.onchain_address]: ../LSPS0/common-schemas.md#link-lsps0onchain_address
[LSPS0.datetime]: ../LSPS0/common-schemas.md#link-lsps0datetime
[LSPS0.outpoint]: ../LSPS0/common-schemas.md#link-lsps0outpoint
[LSPS0.scid]: ../LSPS0/common-schemas.md#link-lsps0scid
[LSPS0.txid]: ../LSPS0/common-schemas.md#link-lsps0txid
[LSPS0.client_rejected_error]: ../LSPS0/common-schemas.md#link-lsps0client_rejected_error
[LSPS1]: ../LSPS1/README.md
