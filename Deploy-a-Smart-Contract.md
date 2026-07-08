To deploy a smart contract, broadcast a Hive `custom_json` operation to the sidechain id, usually `ssc-mainnet-hive` on Hive Engine mainnet. The custom JSON payload calls the system `contract` contract's `deploy` action:

```js
{
    "contractName": "contract",
    "contractAction": "deploy",
    "contractPayload": {
        "name": "MySmartContractName",
        "code": "BASE64EncodedJavascriptCode"
    }
}
```

- name: this is the name of the Smart Contract you want to deploy, it has to be unique on the sidechain
- code: this is the JavaScript source of your smart contract encoded as Base64

Deployment must be signed by the Hive account that will own the contract. Large contracts may require chunking or helper tooling because Hive custom JSON operations have size limits.

Example deployment payload:

```js
{
    "contractName": "contract",
    "contractAction": "deploy",
    "contractPayload": {
        "name": "users_test_contract",
        "code": "YWN0aW9ucy5jcmVhdGUgPSBmdW5jdGlvbiAocGF5bG9hZCkgew0KICAgICAgZGIuY3JlYXRlVGFibGUoJ3VzZXJzJyk7DQogICAgfQ0KDQogICAgYWN0aW9ucy5hZGRVc2VyID0gZnVuY3Rpb24gKHBheWxvYWQpIHsNCiAgICAgIGNvbnN0IHsgdXNlcm5hbWUgfSA9IHBheWxvYWQ7DQogICAgICBpZiAodXNlcm5hbWUgJiYgdHlwZW9mIHVzZXJuYW1lID09PSAnc3RyaW5nJyl7DQoNCiAgICAgICAgbGV0IHVzZXJzID0gZGIuZ2V0VGFibGUoJ3VzZXJzJyk7DQoNCiAgICAgICAgbGV0IHVzZXIgPSB1c2Vycy5maW5kT25lKHsgJ2lkJzogc2VuZGVyIH0pOw0KDQogICAgICAgIGlmICh1c2VyID09PSBudWxsKSB7DQogICAgICAgICAgY29uc3QgbmV3VXNlciA9IHsNCiAgICAgICAgICAgICdpZCc6IHNlbmRlciwNCiAgICAgICAgICAgICd1c2VybmFtZSc6IHVzZXJuYW1lLA0KICAgICAgICAgICAgJ3ZlcmlmaWVkJzogZmFsc2UsDQogICAgICAgICAgfTsNCg0KICAgICAgICAgIHVzZXJzLmluc2VydChuZXdVc2VyKTsNCiAgICAgICAgICBlbWl0KCduZXdVc2VyQ3JlYXRlZCcsIG5ld1VzZXIpOw0KICAgICAgICB9IA0KICAgICAgfQ0KICAgIH0NCg0KICAgIGFjdGlvbnMudXBkYXRlVXNlciA9IGZ1bmN0aW9uIChwYXlsb2FkKSB7DQogICAgICBjb25zdCB7IHVzZXJuYW1lIH0gPSBwYXlsb2FkOw0KICAgICAgaWYgKHVzZXJuYW1lICYmIHR5cGVvZiB1c2VybmFtZSA9PT0gJ3N0cmluZycpew0KDQogICAgICAgIGxldCB1c2VycyA9IGRiLmdldFRhYmxlKCd1c2VycycpOw0KICAgICAgICBsZXQgdXNlciA9IHVzZXJzLmZpbmRPbmUoeyAnaWQnOiBzZW5kZXIgfSk7DQogICAgICAgIGlmICh1c2VyKSB7DQogICAgICAgICAgdXNlci51c2VybmFtZSA9IHVzZXJuYW1lOw0KICAgICAgICAgIHVzZXJzLnVwZGF0ZSh1c2VyKTsNCiAgICAgICAgfQ0KICAgICAgfQ0KICAgIH0="
    }
}
```
