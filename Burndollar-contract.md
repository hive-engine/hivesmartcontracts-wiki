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
  * [burnpair](#burnpair)

# Introduction

The burndollar contract allows a token issuer to create a related stablecoin in a parent/child relationship. Users can burn parent tokens based on a percentage to receive the child "XXX.D" stable token. The issuer may route the fee portion of the parent token to any active Hive account, including a DAO.

* The name of the child token will be the parent token name with "stablecoin" appended.
* The symbol of the child token will be the parent token symbol with ".D" appended.

# Requirements

To create a XXX.D token, the user must have a Hive account that has already issued a token.

Once the .D token is created, the issuer is automatically granted a fixed number of the new .D token. This issuance is intended for creating market pools to stabilize the token. Two market pools are required for users to burn the parent token and receive the child token.

## Stablecoin Pool
 A market pool must be created with one of the following stable coins:

 SWAP.HBD, SWAP.USDT, SWAP.DAI, or SWAP.USDC.  

The other side of the pool can be either the parent or child token. The pool must have a minimum market value of approximately $500. Multiple pools may be created, but at least one must maintain this value. 

## Parent / StableCoin pool
A second market pool must pair the parent token with the child XXX.D token. This pool must also maintain a market value of approximately $500 or more.

# Fees

* To create a .D token the fee is 1000 BEED
* For a user to update the parameters , after the initial creation is 100 BEED
* To convert any quantity of a parent token to a .D token is 1 BEED

# Actions available:

## createTokenD:

Creates a new "XXX.D" token  A creation fee of 1000 BEED is required.

* requires active key: yes
* Callable by: Issuer of the parent token

* fields
   * symbol (string): The parent token symbol to create the child XXX.D token
   *  precision: Inherited from parent token
 
* parameters:
  * burnRouting (string): defaults to null, but can be changed to a valid Hive account name
  * feePercentage (decimal as string): between 0 and 1, This parameter controls how much of the stable coin will be issued to a Hive account. 
    
* Mathematics of feePercentage:
``` "feePercentage": ".9", ```
 - Burn 40 units of parent token → user receives 4 units of .D token
 - 36 units of parent token route to burnRouting (if any)
 - 4 units of parent token route to null
 
``` "feePercentage": ".1", ```
  - Burn 40 units of parent token → user receives 36 units of .D token
  - 4 units of parent token route to burnRouting (if any)
 - 36 units of parent token route to null

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

Upon successful creation, the issuer receives 1000 units of the new .D token for market pool creation as discussed in the 
#[Requirements](#requirements) section. 

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
        "burnRouting": "thedao",  
        "feePercentage": ".1", 
        "isSignedWithActiveKey": true 
    }
}
```
  
## convert:

Allows users to convert parent tokens into the child .D token. Requires a fee of 1 BEED.

* requires active key: yes
* market pools required // #[Requirements](#requirements)

* fields
  * symbol = symbol of the parent token to be burned, then hive account to be issued .D stable token

* parameters:
  * quantity (number as string): quantity of parent token the to be burned

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

A successful action will emit a "burndollar" event, e.g. // fee set to .9
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
All tables include an implicit _id field (unique identifier), omitted for brevity

## params:
contains contract parameters for the burndollar contract
* fields
  * issueDTokenFee = the cost in BEED for creating a XXX.d token
  * updateParamsFee = the cost in BEED of updating the burnRouting or feePercentage.  // fields available also available upon creation
  * maxAirdropsPerBlock = the cost in BEED of converting any amount of a parent token to the stable coin
  * minAmountConvertible = the minimal quantity a user must convert of a parent in the stable coin
  * dTokenToIssuer = the stable coins issued to the stable coin owner upon creation, intended to be used to create market pools
  * burnToken = the token to be burned to call actions

## burnpair:
contains information about the parent and the child token.
* fields
  * issuer = the hive account the owns the parent and newly created stable token
  * symbol = symbol of the stable token
  * precision = the precision of the parent token
  * parentSymbol = the symbol of the parent token
  * burnRouting = the hive account the fee will route to, if any
  * feePercentage = The fee which controls how much a user receives when the convert a parent token to the stable token
