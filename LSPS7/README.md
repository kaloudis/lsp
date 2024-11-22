# LSPS7 Channel Lease Extensions

| Name     | `channel_lease_extensions` |
|----------|-------------------------|
| Version  | 1                       |
| Status   | Draft                   |

LSPS7 requires [LSPS0](../LSPS0/) and therefore [BOLT8](https://github.com/lightning/bolts/blob/master/08-transport.md) as a transport layer. The payment object structure is taken from [LSPS1][].


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

- `original_order <object>` An *optional* property for the original order for the channel lease, including the `id <string>` to identify the order and the `service <string>` the purchase occured on.
- `extension_order_ids <string[]>` An *optional* list of order IDs for each time the channel lease has been extended.
- `short_channel_id` <[LSPS0.scid][]> Short Channel Identifier (SCID) of the channel.
- `max_channel_extension_expiry_blocks <uint32>` The maximum number of blocks a channel can be leased for.
- `expiration_block <uint32>` The block height at which the channel lease will expiry. May be marked as 0 if the user opened the channel to the LSP, and the LSP has no obligation to keep open.
  - MUST be 0 or greater.

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

The `payment` object returned by `lsps7.create_order` and `lsps7.get_order`, mirrors the formatting of [LSPS1.payment][].


[LSPS0.onchain_address]: ../LSPS0/common-schemas.md#link-lsps0onchain_address
[LSPS0.datetime]: ../LSPS0/common-schemas.md#link-lsps0datetime\
[LSPS0.scid]: ../LSPS0/common-schemas.md#link-lsps0scid
[LSPS0.client_rejected_error]: ../LSPS0/common-schemas.md#link-lsps0client_rejected_error
[LSPS1]: ../LSPS1
[LSPS1.payment]: ../LSPS1#3-payment