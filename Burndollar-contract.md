Documentation written by [drewlongshot](https://github.com/BostonTechie)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* [Requirements](#requirements)
  * [Fees](#fees)
* [Actions available](#actions-available)
  * [createTokenD](#createTokenD)
  * [updateBurnPair](#updateBurnPair)
  * [convert](#convert)
* [Tables available](#tables-available)
  * [params](#params)
  * [pendingAirdrops](#pendingairdrops)

# Introduction

The burndollar contract allows a token issuer of a who to create a related XXX.D token in a parent / child type relationship. The contract allows users to burn based on a percentage of the parent token in order to get the child "XXX.D" token. In addition, the token issuer can route has the choice to route the burn token to a DAO, or other accounts, if they so wish. The name of the child token will always be the name of the parent token with "stablecoin" appended to the end of the name. The symbol of the new token will always be the symbol of the parent token with ".D" appended to the symbol.

# Requirements

In order for a user to create a XXX.D token the user must be using a hive account that has already created a token, meaning they must be the token issuer. 

Once the .D token is created a token issuer will automatically be issued a set number of the new .D token. This automatic issuance is designed to be used so to create marketpools, to enhance the stability of said .D token. Two marketpools are required in order for all other users to be able to burn the parent token and receive a child token. 

One market pool must be created with one of the following stable coins:

 SWAP.HBD, SWAP.USDT, SWAP.DAI, or SWAP.USDC.  

 the other side of the market pool can be the parent or the child token. The value of the market swap must be approximately $500 as a minimum. Multiple pools can be created with stable coins but at least one pool must remain higher than $500 dollars of market value. 

The second market pool must be a pool of the parent token with the child XXX.D token. The value of this pool must stay at or above approximately $500 of market value.  

# Fees

To create a .D token the fee is 1000 BEED
For a user to update the parameters , after the initial creation is 100 BEED
To convert any quantity of a parent token to a .D token is 1 BEED

# Actions available:

## createTokenD:

Creates a new "XXX.D" token  A creation fee of 1000 BEED is required.

* requires active key: yes
* can be called by: the owner/ issuer hive account of a parent token that wishes to create a stablecoin token

* fields
   * symbol (string): The parent token to create the child XXX.D token, parent token must already exist. 
   * precision: The child .D token automatically receives the precision of the parent token upon creation.  
  
* parameters:
  * burnRouting (string): defaults to null, but can be changed to a valid hive account name
  * feePercentage (decimal as string): between 0 and 1, This parametersle controls how much a hive account 
    



* Mathematics of feePercentage:
  Setting a fee percentage of .90 will send 90% of the parent token to the "burnRouting" account, and 10% of the parent token to null. If a user were to burn 40 "parent tokens" this would mean 36 parent tokens would go to "burnRouting" account and 4 tokens would go to null. This only occurs if a token issuer changes the default burnRouting account. 



* examples:  
```
{
    "contractName": "burndollar",
    "contractAction": "createTokenD",
    "contractPayload": {
        "symbol": "TKN",         //parent token 
        "burnRouting": "whale",  // needed if null is not desired
        "feePercentage": ".9", 
        "isSignedWithActiveKey": true 
    }
}
```

A successful action will emit a "burndollar" event, e.g.
```
{
    "contract": "burndollar",
    "event": "issued new token dollar stablecoin",
    "data": {
        convertPercentage: '.9', feeRouting: 'whale', dSymbol: 'TKN.D' 
    }
}
```

After being a new .D token is successfully created the token hive account will automatically be issued 1000 units of the new .D token. Which is to be utilized in the creation of market pools as discussed in the #[Requirements](#requirements) section . 


## updateBurnPair:

After the initial creation of the child .D token. This action allows the token issuer to update two parameters if needed. The fee for doing so is 100 BEED.

* requires active key: yes

* fields
  * symbol = symbol of the parent token where the parameters are being updated

* parameters:
  * burnRouting (string): defaults to null, but can be change to a valid hive account name
  * feePercentage (decimal as string): between 0 and 1  see detailed explanation in ##[createTokenD](##createTokenD) section.

  * examples:  
```
{
    "contractName": "burndollar",
    "contractAction": "updateBurnPair",
    "contractPayload": {
        "symbol": "TKN",         //parent token
        "burnRouting": "mydaoaccount",  
        "feePercentage": ".01", 
        "isSignedWithActiveKey": true 
    }
}
```
  
## convert:

After a .D token is created and the token issue established appropriate market pools as discussed in the #[Requirements](#requirements) section . Other hive accounts will be able to convert the parent token into the new .D token. The fee for any size conversion is 1 BEED. The efficiency and the routing  is determined by the creator of the .D token, see detailed explanation in ##[createTokenD](##createTokenD) section. 



* requires active key: yes

* fields
  * symbol = symbol of the parent token to be burned, then hive account to be issued .D stable token

* parameters:
  * quantity (number as string): quantity of parent token the hive account wishes to be burned to receive .D token

* examples:  
```
{
    "contractName": "burndollar",
    "contractAction": "convert",
    "contractPayload": {
        "symbol": "TKN",
         "quantity" : "40", 
         "isSignedWithActiveKey": true 
    }
}
```


A successful action will emit a "burndollar" event, e.g.
```
{
    "contract": "burndollar",
    "event": "Converted token to dollar token",
    "data": {
      symbol: 'TKN.D',
      fee: '36.000',
      feeRouting: 'whale',
      parentSymbol: 'TKN',
      precision: 3,
      childIssued: '4.000',
      parentPriceInUSD: '1.000
    }
}

```
# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## params:
contains contract parameters such as the current fees
* fields
  * feePerTransaction = the cost in BEE per transaction
  * transactionPerBlock = number of maximum transaction in each block per airdrop
  * maxAirdropsPerBlock = number of maximum airdrops in each block

## pendingAirdrops:
contains information about pending airdrops
* fields
  * airdropId = id of airdrop (unique)
  * symbol = symbol of token to airdrop
  * type = distribution type
  * list = list of accounts to transact
  * blockNumber = block number in which airdrop was initiated
