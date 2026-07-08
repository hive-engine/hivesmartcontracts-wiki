There are two ways to interact with the sidechain.

### 1) Via `custom_json`

Broadcast a Hive `custom_json` operation with the sidechain id in the `id` field. On Hive Engine mainnet, the default sidechain id is `mainnet-hive`, so the custom JSON id is:

```text
ssc-mainnet-hive
```

The `json` field can contain a single contract call or an array of contract calls:

```js
[
  {
    contractName: "tokens",
    contractAction: "transfer",
    contractPayload: {
      symbol: "BEE",
      to: "alice",
      quantity: "1.00000000",
      memo: "optional memo",
    },
  },
];
```

Contract payloads that spend funds, issue assets, change ownership, or update privileged state normally require the operation to be signed with the Hive active key. The contract receives this as `isSignedWithActiveKey` in the action payload.

### 2) Via transfer memo

Some flows are initiated by transferring HIVE or HBD to a gateway account with a JSON memo. The memo wraps the sidechain call:

```js
{
  "id": "ssc-mainnet-hive",
  "json": {
    "contractName": "hivepegged",
    "contractAction": "buy",
    "contractPayload": {
      "recipient": "alice"
    }
  }
}
```

Transfer-driven contract actions receive transfer metadata in the payload, including the recipient and the transferred Hive asset amount. Use the target contract documentation before sending funds; not every contract action supports transfer memo execution.

Important: `CHAIN-ID` identifies the sidechain that should process the operation. It is configured as `chainId` in the node's `config.json`.
