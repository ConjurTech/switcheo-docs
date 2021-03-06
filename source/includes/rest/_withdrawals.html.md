## Withdrawals

Withdrawals are a transfer of tokens from the Switcheo smart contract into your wallet.

### Overview

Once a withdrawal has been executed, tokens in your contract balance would be deducted.
Tokens that are put on hold by orders cannot be withdrawn.

Withdrawals are not instantaneous.
Tokens deducted from your contract balance will be added to your wallet balance but put on hold until
the withdrawal has been fully executed.

### POST Create Withdrawal

> Create a withdrawal

```js
function createWithdrawal ({ blockchain, address, assetID, amount, privateKey }) {
  const signableParams = { blockchain, assetID, amount,
                           contractHash: CONTRACT_HASH, timestamp: getTimestamp() }
  const signature = signParams(signableParams, privateKey,)
  const apiParams = { ...signableParams, address, signature }
  return api.post(API_URL + '/withdrawals', apiParams)
}

// NOTE: in this example, the parameters can be in camel case because
// the `signParams` and `api.post` method automatically convert param keys
// to snake case
createWithdrawal({
  blockchain: 'neo',
  address: user.address,
  assetID: 'SWTH',
  amount: (toNeoAssetAmount(1)),
  privateKey: user.privateKey
})
```

> Example request

```js
{
  "blockchain": "neo",
  "asset_id": "SWTH",
  "amount": "100000000",
  "timestamp": 1531544358560,
  "contract_hash": "<contract hash>",
  "signature": "<signature>",
  "address": "87cf67daa0c1e9b6caa1443cf5555b09cb3f8e5f"
}
```

> Example response

```js
{
  "id": "e0f56e23-2e11-4848-b749-a147c872cbe6"
}
```

This endpoint creates a withdrawal which can be executed through [Execute Withdrawal](#execute-withdrawal).
To be able to make a withdrawal, sufficient funds are required in the contract balance.

A [signature](#authentication) of the request payload has to be provided for this API call.

<aside class="notice">
  IMPORTANT: After calling this endpoint, the Execute Withdrawal endpoint has to be called for the withdrawal to be executed.
</aside>

#### HTTP Request

`POST /v2/withdrawals`

#### Request Parameters

 Parameter         | Type       | Required | Description
------------------ | ---------- | -------- | ------------
 blockchain        | **string** | yes       | Blockchain that the token to withdraw is on. Possible values are: `neo`.
 asset_id          | **string** | yes       | The [asset symbol or ID](#supported-assets) to withdraw.
 amount            | [amount](#amounts) | yes       | [Amount](#amounts) of tokens to withdraw.
 timestamp         | [timestamp](#timestamp)    | yes       | The exchange's timestamp to be used as a nonce.
 contract_hash     | **string** | yes       | Switcheo Exchange [contract hash](#contracts) to execute the withdraw on.
 signature         | **string** | yes       | Signature of the request payload. See [Authentication](#authentication) for more details.
 address           | [address](#addresses) | yes       | The withdrawer's [address](#addresses). **Do not include this in the parameters to be signed.**

#### Example

[Full create withdrawal example](https://github.com/ConjurTech/switcheo-api-examples/blob/master/src/examples/withdrawals/createWithdrawalExample.js)

### POST Execute Withdrawal

> Executing a withdrawal

```js
function executeWithdrawal ({ withdrawal, privateKey }) {
    const signableParams = { id: withdrawal.id, timestamp: getTimestamp() }
    const signature = signParams(signableParams, privateKey)
    const url = `${API_URL}/withdrawals/${withdrawal.id}/broadcast`
    return api.post(url, { ...signableParams, signature })
}
```

> Example request

```js
{
  "id": "998f1bdc-f26f-44f2-b204-403e24d5777f",
  "timestamp": 1531545149715,
  "signature": "<signature>"
}
```

> Example response

```js
{
  "event_type": "withdrawal",
  "amount": -100000000,
  "asset_id": "ab38352559b8b203bde5fddfa0b07d8b2525e132",
  "status": "confirming",
  "id": "998f1bdc-f26f-44f2-b204-403e24d5777f",
  "blockchain": "neo",
  "reason_code": 9,
  "address": "87cf67daa0c1e9b6caa1443cf5555b09cb3f8e5f",
  "transaction_hash": null,
  "created_at": "2018-07-14T05:12:29.699Z",
  "updated_at": "2018-07-14T05:12:29.765Z",
  "contract_hash": "<contract hash>"
}
```

This is the second endpoint required to execute a withdrawal.
After using the [Create Withdrawal](#create-withdrawal) endpoint,
you will receive a response which requires additional signing.

#### HTTP Request

`POST /v2/withdrawals/:id/broadcast`

#### Request Parameters

 Parameter  | Type       | Required | Description
 ---------- | ---------- | -------- | -----------
 id         | **string** | yes       | `id` parameter in the response from create withdrawal endpoint.
 timestamp  | [timestamp](#timestamp)    | yes       | The exchange's timestamp to be used as a nonce.
 signature  | **string** | yes       | Signature of the request payload. See [Authentication](#authentication) for more details.

#### Example

[Full execute withdrawal example](https://github.com/ConjurTech/switcheo-api-examples/blob/master/src/examples/withdrawals/executeWithdrawalExample.js)
