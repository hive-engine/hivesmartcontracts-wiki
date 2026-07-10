# Hive Smart Contracts

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
