# Hive Smart Contracts

Hive Smart Contracts (Hive Engine) is a sidechain that runs JavaScript contracts when it sees Hive transactions. Reads use a node's JSON-RPC API. Writes are Hive `custom_json` operations, or a transfer memo for the few contracts that support that path.

## Agent and developer quickstart

Use this when calling the chain from an app or an agent. Action payloads are defined on the contract pages linked below.

### Read (JSON-RPC)

Public Hive Engine endpoints already used in this wiki:

- Contracts: `https://api.hive-engine.com/rpc/contracts`
- Blockchain: `https://api.hive-engine.com/rpc/blockchain`

Send an HTTP POST with `Content-Type: application/json`. These reads do not need a Hive signature. A local node exposes the same methods at `http://localhost:5000/contracts` and `http://localhost:5000/blockchain`. Method list: [JSON RPC server](JSON-RPC-server.md). The contracts URL is also the example in [Querying the Engine API](Smart-Contracts-Guide.md#querying-the-engine-api).

Example, the `tokens` row for `BEE`:

```json
{
  "jsonrpc": "2.0",
  "method": "findOne",
  "params": {
    "contract": "tokens",
    "table": "tokens",
    "query": { "symbol": "BEE" }
  },
  "id": 1
}
```

Call `getStatus` on the blockchain endpoint before relying on a method. The result includes `chainId` (Hive Engine mainnet is `mainnet-hive`) and `disabledMethods`. Checked 2026-09-29, that public node disabled the blockchain methods `getBlockRangeInfo` and `getBlockInfoByHiveBlock`.

The companion portal [Hive Engine developer docs](https://hive-engine.github.io/engine-docs/) documents the same `custom_json` and transfer-memo shapes. It does not list a separate RPC host; use the endpoints above.

### Write (`custom_json`)

Broadcast a Hive `custom_json` operation. On Hive Engine mainnet the operation `id` is `ssc-mainnet-hive` (sidechain id `mainnet-hive`). The `json` field is one call or an array of calls. Token transfer, from the [Tokens contract](Tokens-Contract.md#transfer) page:

```json
{
  "contractName": "tokens",
  "contractAction": "transfer",
  "contractPayload": {
    "symbol": "TKN",
    "to": "harpagon",
    "quantity": "15.21548745"
  }
}
```

Actions that spend funds, issue assets, or change privileged state must be signed with the Hive active key. The node records that as `isSignedWithActiveKey` on the payload. Details: [Interact with the sidechain](Interact-with-the-sidechain.md).

**Historical:** Steem Engine mainnet used the custom JSON id `ssc-mainnet1`. The [developer portal](https://hive-engine.github.io/engine-docs/) still mentions that older id. Hive Engine mainnet uses `ssc-mainnet-hive`, as shown above.

### Signing

Sign on a machine you control, with a wallet or library that keeps the private key local. Do not put an active, owner, or posting private key, or a wallet password, in a prompt, log, ticket, or repository. The public RPC URLs above do not need a key. The [Beem broadcast example](Smart-Contracts-Guide.md#broadcasting-custom-json-with-beem) is a local-wallet sketch; its key and password strings are placeholders.

### Where to go next

- [Interact with the sidechain](Interact-with-the-sidechain.md) — `custom_json` and transfer memo
- [JSON RPC server](JSON-RPC-server.md) — `getStatus`, `find`, `findOne`, blocks, transactions
- [Contract Action Index](Contract-Action-Index.md) — action and table names
- [Tokens](Tokens-Contract.md), [Market](Market-Smart-Contract.md), [NFT](NFT-Contracts.md)
- [Deploy a Smart Contract](Deploy-a-Smart-Contract.md)
- [Smart Contracts Guide](Smart-Contracts-Guide.md) — contract development and testing

## 1. What is it?

Hive Smart Contracts is a sidechain powered by Hive. It allows users and contracts to perform actions on a decentralized database through JavaScript smart contracts.

## 2. How does it work?

You need a Hive account. To interact with the sidechain, broadcast a Hive `custom_json` operation using the sidechain id, contract name, contract action, and payload format expected by the target contract. The Hive Smart Contracts node streams Hive blocks, extracts those transactions, executes the requested contract actions, and stores the resulting sidechain blocks and contract tables in MongoDB.

## 3. Sidechain specifications

- run on [node.js](https://nodejs.org)
- database layer powered by [MongoDB](https://www.mongodb.com/)
- Smart Contracts developed in Javascript
- Smart Contracts run in a sandboxed JavaScript virtual machine using `isolated-vm`
- a block on the sidechain is produced only when transactions are parsed in a Hive block

## 4. Current node implementation

The current reference implementation is the `hive-engine/hivesmartcontracts` node. It includes JSON-RPC APIs, MongoDB-backed chain and contract storage, witness support, P2P networking, light-node mode, replay/restore tooling, and the deployed Hive Engine contract set.
