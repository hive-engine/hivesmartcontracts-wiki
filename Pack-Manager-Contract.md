Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

# Table of Contents

* [Introduction](#introduction)
* [Creating an NFT Collectable](#creating-an-nft-collectable)
  * actions:
  * [createNft](#createnft)
* [Registering Packs](#registering-packs)
  * actions:
  * [registerPack](#registerpack)
  * [updatePack](#updatepack)
  * [updateEdition](#updateedition)
* [Defining NFT Instance Types](#defining-nft-instance-types)
  * actions:
  * [addType](#addtype)
  * [updateType](#updatetype)
  * [deleteType](#deletetype)
* [Setting Names](#setting-names)
  * actions:
  * [setTraitName](#settraitname)
* [Opening Packs](#opening-packs)
  * actions:
  * [deposit](#deposit)
  * [open](#open)
* [Tables available](#tables-available)
  * [params](#params)
  * [managedNfts](#managednfts)
  * [types](#types)
  * [packs](#packs)
  * [foils](#foils)
  * [categories](#categories)
  * [rarities](#rarities)
  * [teams](#teams)

# Introduction

The Pack Manager is a smart contract designed for managing the issuance of NFT collectables. Consider a [Splinterlands](https://splinterlands.com/) style issuance mechanism: in Splinterlands you buy pack tokens (ALPHA, BETA, UNTAMED, ORB, SLDICE on Hive Engine). Then you open these packs in the game to generate Monster & Summoner card NFTs. Each pack generates 5 cards. The Pack Manager provides a similar issuance mechanism for your own projects. It's a generalized, more customizable version of what Splinterlands does.

As a user, you feed fungible pack tokens to the Pack Manager, and it "opens" them, sending generated NFT instances back to you.

As an app creator, you create your NFT and register pack -> NFT mappings with the Pack Manager. You set configuration that determines how many NFT instances are issued per pack, their edition, rarity, and various other properties.

For app creators, a recommended basic usage flow is:

1. Create a new pack token (optional, can use any existing fungible token as a pack)
2. Create a new NFT ([createNft action](#createnft))
3. Register settings for a pack ([registerPack action](#registerpack))
4. Set names for things like foils, categories, rarities, and teams ([setTraitName](#settraitname))
5. Add NFT instance types (repeated use of the [addType action](#addtype))
6. (optional) Finalize settings, effectively making configuration read only ([updatePack](#updatepack) and [updateEdition](#updateedition))
7. Deposit BEE to cover NFT issuance costs ([deposit action](#deposit))
8. Now your users can open packs! ([open action](#open))

For details of the above process, keep reading:

# Actions available:
## Creating an NFT Collectable

Not just any old NFT is compatible with the Pack Manager. In order for the Pack Manager to issue NFTs, an NFT symbol has to be **under management** with the Pack Manager. We may add a way later on for existing NFTs to be placed under management if they satisfy certain criteria, but currently the only way to have an NFT under management is to create it through the following action:

### createNft:
Creates a new NFT definition and places it under management of the packmanager smart contract. A creation fee of 100 BEE is required. Three data properties will be defined as follows:

Name | Type | Read Only | Authorized Editing Accounts | Authorized Editing Contracts | Description
--- | --- | --- | --- | --- | ---
edition | integer >= 0 | Yes | none | packmanager | A number that indicates which edition an NFT instance belongs to (analogous to Alpha, Beta, Untamed, etc in Splinterlands)
foil | integer >= 0 | Optional, up to creator | Hive account that creates the NFT | packmanager | A number that indicates the foil of an NFT instance (analogous to Gold or Regular in Splinterlands)
type | integer >= 0 | Optional, up to creator | Hive account that creates the NFT | packmanager | A number that indicates the specific type of an NFT instance (analogous to the type of card in Splinterlands, for example Goblin Shaman or Alric Stormbringer)

Note that the max supply of the NFT will be technically unlimited. Instead of capping supply with a specific number, supply is determined indirectly based on the number of associated pack tokens in existence.

In addition, the NFT's groupBy will be set to ['edition', 'foil', 'type']. For more information about the purpose of groupBy, see [NFT contract documentation](https://github.com/hive-engine/hivesmartcontracts-wiki/blob/master/NFT-Contracts.md#setgroupby).
* requires active key: yes

* can be called by: Hive account

* parameters:
  * name (string): name of the token (letters, numbers, whitespace only, max length of 50)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * **(optional)** orgName (string): name of the company/organization that created this NFT (letters, numbers, whitespace only, max length of 50)
  * **(optional)** productName (string): product/brand that this NFT is associated with (letters, numbers, whitespace only, max length of 50)
  * **(optional)** url (string): url of the project (max length of 255)
  * **(optional)** isFoilReadOnly (boolean): if true, then the foil data property will be created as read-only. The default value is true if this parameter is not specified.
  * **(optional)** isTypeReadOnly (boolean): if true, then the type data property will be created as read-only. The default value is true if this parameter is not specified.

* examples:
```
{
    "contractName": "packmanager",
    "contractAction": "createNft",
    "contractPayload": {
        "symbol": "WAR",
        "name": "War Game Military Units"
    }
}

{
    "contractName": "packmanager",
    "contractAction": "createNft",
    "contractPayload": {
        "symbol": "WAR",
        "name": "War Game Military Units",
        "orgName": "Wars R Us Inc",
        "productName": "War Game Name Here",
        "url": "https://mywargame.com"
    }
}

{
    "contractName": "packmanager",
    "contractAction": "createNft",
    "contractPayload": {
        "symbol": "WAR",
        "name": "War Game Military Units",
        "orgName": "Wars R Us Inc",
        "productName": "War Game Name Here",
        "url": "https://mywargame.com",
        "isFoilReadOnly": false,
        "isTypeReadOnly": false
    }
}
```

A successful action will emit a "createNft" event: ``symbol``
example:
```
{
    "contract": "packmanager",
    "event": "createNft",
    "data": {
        "symbol": "WAR"
    }
}
```

## Registering Packs

Once you have an NFT under management, the next step is to register pack tokens & corresponding settings that will be associated with the NFT. This is a many-to-many mapping: you can have multiple pack tokens that map to multiple NFTs. For example, you could have small chest, medium chest, and large chest packs that all map to the same NFT and issue 5, 10, and 15 NFT instances per pack, respectively. You could also have a chest token that maps to 2 or more NFTs, so the player can choose which type of NFT it opens as.

Any already existing fungible token can be used as a pack token; it doesn't have to be a new token that was created specifically for this purpose (although for most new projects making a new token is sensible). For example, you could give Splinterlands ALPHA tokens new utility by having them used to generate military units for your new War Game. Or have dCity SIM used as a pack token for your own city simulator.

The following actions are available to manage pack registrations:

### registerPack:
Register settings for a new pack token / NFT pair. New editions for existing NFTs under management can also be registered using this action. Note that you cannot register settings twice for the same pack token & NFT. After settings are registered, they can be edited using the updatePack action. A 1000 BEE fee must be paid every time registerPack is used.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * packSymbol (string): symbol of the fungible pack token to be registered
  * nftSymbol (string): symbol of the NFT that the pack token should be linked to
  * edition (integer >= 0): what edition does this pack open (In Splinterlands there is Alpha, Beta, Untamed; other projects might have a 1st Edition, 2nd Edition, etc)?
  * **(optional)** editionName (string): name for the new edition that this pack opens (letters, numbers, whitespace only, max length of 100)
  * cardsPerPack (integer >= 1 and <= 30): how many NFT instances should be generated for each pack opened?
  * foilChance (array of integer): percentage chances for determining the foil of an opened NFT instance (see notes below)
  * categoryChance (array of integer): percentage chances for determining the foil of an opened NFT instance (see notes below)
  * rarityChance (array of integer): percentage chances for determining the foil of an opened NFT instance (see notes below)
  * teamChance (array of integer): percentage chances for determining the foil of an opened NFT instance (see notes below)
  * numRolls (integer >= 1 and <= 10): maximum possible number of rolls if a random category / rarity / team throw results in no NFT instance types to choose from (see notes below). numRolls = 1 means there will only ever be a single roll (never any re-rolls).

**Editions:** when you call registerPack for the first time with a particular edition number, the editionName parameter is required. If you then call registerPack again for the same edition (it's perfectly OK to have many different packs that all open the same edition), the editionName parameter will be ignored (you don't have to provide it, and if you do it won't have any effect). To change an edition name later on, use the [updateEdition](#updateedition) action. Edition numbers, by convention, should start at 0 and be incremented by 1 for each new edition (although you are not forced to follow this convention).

**Pack opening:** the following procedure is followed when a pack is opened, to generate each NFT instance contained within the pack:

1. Pick a random foil, category, team, and rarity, according to the configured percent chances.
2. Randomly select an instance type that has the chosen category, team, and rarity.
3. If there are no instance types available that match the chosen category, team, and rarity, repeat steps 1-2 up to the maximum number of times defined by the numRolls parameter. If no valid instance type is selected after the maximum number of re-rolls, then select type 0, which must exist.
4. Issue an NFT of the selected instance type.

Foil is analogous to Gold or Regular cards in Splinterlands, and is independent of the other NFT instance type characteristics. Meaning that if your app has 3 foils (say Regular, Gold, and Platinum), and 10 defined NFT instance types, then you've actually got 30 types total (10 Regular, 10 Gold, and 10 Platinum). For more info on NFT instance types, and the meaning of category, team, and rarity, refer to the section on [defining NFT instance types](#defining-nft-instance-types).

**Percent chances:** foilChance, categoryChance, rarityChance, and teamChance are all defined in terms of a partition array. For example, if you want 5 rarities (say Common, Uncommon, Rare, Epic, and Legendary), you might use this partition:

```[700, 900, 980, 995, 1000]```

This means that when a random roll is done to determine rarity, a random number from 1 to 1000 (inclusive) is picked, and where that number falls in the partition indicates the rarity. A roll of 700 or below maps to rarity 0 (Common), 701 to 900 maps to rarity 1 (Uncommon), 901 to 980 maps to rarity 2 (Rare), 981 to 995 maps to rarity 3 (Epic), and 996 to 1000 maps to rarity 4 (Legendary). So the length of the array, 5, indicates how many rarities we have. This partition means there is a 70% chance to roll a Common, 20% chance for an Uncommon, 8% chance for a Rare, 1.5% chance for an Epic, and 0.5% chance for a Legendary.

As another example, let's say you want an NFT with 2 teams (Black and White), with an equal chance (50/50) of getting one team or the other. You could define a teamChance partition like this:

```[50, 100]``` or ```[5, 10]``` or ```[1, 2]```

So we see there are multiple ways to define the same percent chances. The exact numbers don't matter so much as how the numbers divide the partition to determine the percentages.

As a final example, let's say you only have 1 foil type. Maybe you don't care at all about foil and don't want to use that concept in your game. Then you would define foilChance as a partition with just a single number:

```[1]```

Partition arrays must consist only of positive integers, and they must form an ascending sequence. The following are examples of invalid partitions that would cause your registerPack call to fail:

* ```[0, 5, 10]``` <--- all numbers must be positive
* ```[50.1, 60.5, 100]``` <--- all numbers must be integers
* ```[100, 150, 150, 200]``` <--- numbers cannot repeat
* ```[50, 60, 20, 10, 80, 100]``` <--- numbers must form an ascending sequence (i.e. next number in the array must always be greater than the previous)
* ```[]``` <--- must have at least 1 number, can't be empty

Each partition array can have a maximum of 100 numbers (thus there is a maximum limit of 100 types of foils, categories, rarities, and teams for any given NFT & edition).

**Example action call:**
```
{
    "contractName": "packmanager",
    "contractAction": "registerPack",
    "contractPayload": {
        "packSymbol": "PACK",
        "nftSymbol": "WAR",
        "edition": 0,
        "editionName": "Ultimate War Edition",
        "cardsPerPack": 3,
        "numRolls": 5,
        "foilChance": [50, 100],
        "categoryChance": [33, 66, 100],
        "rarityChance": [300, 1000],
        "teamChance": [1, 3]
    }
}
```

A successful action will emit a "registerPack" event: ``account, symbol, nft``
example:
```
{
    "contract": "packmanager",
    "event": "registerPack",
    "data": {
        "account": "cryptomancer",
        "symbol": "PACK",
        "nft": "WAR"
    }
}
```

### updatePack:
Edit settings for a previously registered pack token / NFT pair. Note that settings can only be changed if not yet finalized (isFinalized parameter has not been set). If the pack settings have already been finalized (turned to read-only), then this action will result in an error.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * packSymbol (string): symbol of the fungible pack token for this registration
  * nftSymbol (string): symbol of the NFT that the pack token is linked to
  * **(optional)** edition (integer >= 0): updated edition value; note that the edition can only be changed to a value already registered through previous use of the registerPack action.
  * **(optional)** cardsPerPack (integer >= 1 and <= 30): new value for how many NFT instances should be generated per pack opened
  * **(optional)** foilChance (array of integer): new percentage chances for determining the foil of an opened NFT instance
  * **(optional)** categoryChance (array of integer): new percentage chances for determining the foil of an opened NFT instance
  * **(optional)** rarityChance (array of integer): new percentage chances for determining the foil of an opened NFT instance
  * **(optional)** teamChance (array of integer): new percentage chances for determining the foil of an opened NFT instance
  * **(optional)** numRolls (integer >= 1 and <= 10): new maximum possible number of rolls if a random category / rarity / team throw results in no NFT instance types to choose from. numRolls = 1 means there will only ever be a single roll (never any re-rolls).
  * **(optional)** isFinalized (boolean): if present must be set to true, indicates these settings are final and can never be updated again

The isFinalized parameter is intended to reassure your users that NFT instance generation settings for a particular pack can never be changed once your application is live. Although it's not required to ever call updatePack with this parameter set, it is good form to always do so once you have finished setting up your pack and consider it ready for production release.

For details on the meaning of other parameters, see [registerPack](#registerpack) above.

* examples:
```
// If you make a mistake with registerPack, use updatePack to fix it:
{
    "contractName": "packmanager",
    "contractAction": "updatePack",
    "contractPayload": {
        "packSymbol": "PACK",
        "nftSymbol": "WAR",
        "cardsPerPack": 10,
        "numRolls": 1,
        "foilChance": [99, 100],
        "categoryChance": [6000, 9000, 9750, 9995, 10000]
    }
}

// Finalizing your pack settings before production launch is good etiquette:
{
    "contractName": "packmanager",
    "contractAction": "updatePack",
    "contractPayload": {
        "packSymbol": "PACK",
        "nftSymbol": "WAR",
        "isFinalized": true
    }
}
```

A successful action will emit an "updatePack" event: ``symbol, nft, old<SettingName1>, new<SettingName1>, ..., isFinalized (only if set)``
example:
```
{
    "contract": "packmanager",
    "event": "updatePack",
    "data": {
        "symbol": "PACK",
        "nft": "WAR",
        "oldCardsPerPack": 3,
        "newCardsPerPack": 10,
        "oldNumRolls": 5,
        "newNumRolls": 1,
        "oldFoilChance": [50, 100],
        "newFoilChance": [99, 100],
        "oldCategoryChance": [33, 66, 100],
        "newCategoryChance": [6000, 9000, 9750, 9995, 10000],
        "isFinalized": true
    }
}
```

### updateEdition:
Update properties of an edition previously defined through calling the [registerPack](#registerpack) action.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * nftSymbol (string): symbol of the NFT under management that the edition number is linked to
  * edition (integer >= 0): identifying number of the edition to update, which must have been previously used in a call to the registerPack action
  * **(optional)** editionName (string): new name for the edition (letters, numbers, whitespace only, max length of 100)
  * **(optional)** categoryRO (boolean): if present must be set to true, indicates categories for NFT instance types of this edition may no longer be edited
  * **(optional)** rarityRO (boolean): if present must be set to true, indicates rarities for NFT instance types of this edition may no longer be edited
  * **(optional)** teamRO (boolean): if present must be set to true, indicates teams for NFT instance types of this edition may no longer be edited
  * **(optional)** nameRO (boolean): if present must be set to true, indicates names for NFT instance types of this edition may no longer be edited

The categoryRO, rarityRO, teamRO, and nameRO settings determine whether or not those characteristics can be updated using the [updateType](#updatetype) action. Although not required, it is considered good form to set these Read Only flags once you are certain you will not need to make any edits to instance type definitions (after all, players will be very unhappy if you suddenly change their Mighty Dragons to Worthless Fleas). See below section on [defining NFT instance types](#defining-nft-instance-types) for more information.

Note that once you set a Read Only flag, it cannot be unset (because something wouldn't be read-only if you could make it editable again)!

* examples:
```
// Changing an edition name:
{
    "contractName": "packmanager",
    "contractAction": "updateEdition",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 0,
        "editionName": "Mega Uber War Edition"
    }
}

// It's good etiquette to make NFT instance type definitions read-only like this (here we leave the name editable):
{
    "contractName": "packmanager",
    "contractAction": "updateEdition",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 0,
        "categoryRO": true,
        "rarityRO": true,
        "teamRO": true
    }
}
```

A successful action will emit an "updateEdition" event: ``nft, edition, oldEditionName, newEditionName, categoryRO (only if set), rarityRO (only if set), teamRO (only if set), nameRO (only if set) ``
example:
```
{
    "contract": "packmanager",
    "event": "updateEdition",
    "data": {
        "nft": "WAR",
        "edition": 0,
        "oldEditionName": "Ultimate War Edition",
        "newEditionName": "Mega Uber War Edition",
        "categoryRO": true,
        "rarityRO": true,
        "teamRO": true
    }
}
```

## Defining NFT Instance Types

The next step, after placing an NFT under management and setting up packs, is to define all the different types of NFT instances that may be generated from the packs. Using Splinterlands as an example again, that game has hundreds of different cards you can mix & match to form teams for battle. There's Yodin Zaku, Xander Foxwood, Pirate Captain, and Naga Warrior, to name a few. This is what we mean when we talk about **types** in the context of the Pack Manager. All of these different cards are encompassed by the same NFT definition; they are differentiated internally by a number that represents the type of the card.

To give another example, if you used the Pack Manager to create an NFT for a war game, your types might be the military units used by your game: soldier, general, destroyer, tank, cruise missile, marine, cannon, howitzer, etc.

All types have the following properties, which must be integers >= 0 and <= 99 (the text name of the property is stored elsewhere and linked to the numeric value):

* **category**: what category of thing is this type? ex: boat (types can be canoe, sailboat, yacht, tanker), plane (types can be Boeing 747, F-22A Raptor, Japanese Zero), land (types can be swamp, mountain, forest). Splinterlands cards can be Monsters or Summoners.
* **rarity**: how rare is this type? ex: common, uncommon, epic, legendary
* **team**: what grouping, if any, should this type be part of? ex: Splinterlands has cards grouped into Fire, Water, Earth, Life, Death, Dragon, and Neutrals. For sports cards, you might use the names of various sports teams.

New types can be added at any time, even after NFTs have been issued (maybe you want to add a new type to be airdropped once a certain number of packs have been opened, for example). Existing types can also be edited. However, types can only be deleted if no NFT instances have been issued yet (i.e. if the NFT creator is still working on getting everything setup). Once NFTs have been issued, it wouldn't make sense to be able to delete types (you don't want that expensive Dragon you bought to suddenly be removed from the game and made worthless, right?).

The following actions are available to manage types:

### addType:
Adds a new type for an NFT currently under management. A 1 BEE fee is required every time this contract action is used. When added, new types are assigned an ID number that starts at 0 and goes up by 1 for each new type added. So the first type for a given NFT and edition will have ID 0, the 2nd will have ID 1, the 3rd will have ID 2, and so on.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * nftSymbol (string): symbol of the NFT (uppercase letters only, max length of 10)
  * edition (integer >= 0): what edition does this type belong to (In Splinterlands there is Alpha, Beta, Untamed; other projects might have a 1st Edition, 2nd Edition, etc)? Note that type values are distinct per NFT per edition, and the edition must have been previously registered with the registerPack action.
  * category (integer >= 0 and <= 99): what is the category of the new type?
  * rarity (integer >= 0 and <= 99): what is the rarity of the new type?
  * team (integer >= 0 and <= 99): what is the team of the new type?
  * name (string): what is the name of the type? Names must consist of letters, numbers, and whitespaces only, with a maximum length of 100 characters.

**Important Note:** although this action will allow you to set any positive integer for the above parameters, by convention you MUST start at 0 for the first category, rarity, and team, or you may get unexpected behavior when packs are opened. Subsequent categories, rarities, and teams should be incremented by 1. Thus if you have 5 rarities (say Common, Uncommon, Rare, Epic, and Legendary), you should represent them here as 0 (Common), 1 (Uncommon), 2 (Rare), 3 (Epic), and 4 (Legendary). This is so that random NFT generation via percent partition arrays works properly (see "percent chances" sub-section of the [registerPack](#registerpack) action).

Also, it's best to ensure that you define at least one NFT instance type for EVERY POSSIBLE combination of category, team, and rarity. This allows you to safely set numRolls = 1 when calling the [registerPack](#registerpack) action).

Note that the number of types may be unlimited, as long as you have enough BEE to pay the addType fees. However, you are limited to a maximum of 100 different categories, rarities, and teams (this limitation applies to foils as well).

* example:
```
{
    "contractName": "packmanager",
    "contractAction": "addType",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 0,
        "category": 1,
        "rarity": 2,
        "team": 0,
        "name": "F22A Raptor"
    }
}
```

A successful action will emit an "addType" event: ``nft, edition, typeId (ID number of the newly added type), rowId (internal db ID number from the implicit _id field)``
example:
```
{
    "contract": "packmanager",
    "event": "addType",
    "data": {
        "nft": "WAR",
        "edition": 0,
        "typeId": 0,
        "rowId": 567
    }
}
```

Any given type can be uniquely identified by the combination of nft, edition, and typeId. For direct db queries, the rowId also uniquely identifies the type.

### updateType:
Edits an existing type for an NFT under management.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * nftSymbol (string): symbol of the NFT (uppercase letters only, max length of 10)
  * edition (integer >= 0): what edition does this type belong to?
  * typeId (integer >= 0): what is the ID number of the type to update? The combination of nftSymbol, edition, and typeId uniquely identify the type to update. These 3 things cannot be changed.
  * **(optional)** category (integer >= 0 and <= 99): what should be the type's new category?
  * **(optional)** rarity (integer >= 0 and <= 99): what should be the type's new rarity?
  * **(optional)** team (integer >= 0 and <= 99): what should be the type's new team?
  * **(optional)** name: what should be the type's new name? Names must consist of letters, numbers, and whitespaces only, with a maximum length of 100 characters.

Note that this action will result in an error if you try to edit a read-only property (i.e. if categoryRO, rarityRO, teamRO, or nameRO has been set for the property in question through the [updateEdition](#updateedition) action).

* examples:
```
{
    "contractName": "packmanager",
    "contractAction": "updateType",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 0,
        "typeId": 23,
        "team": 5
    }
}

{
    "contractName": "packmanager",
    "contractAction": "updateType",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 0,
        "typeId": 23,
        "category": 2,
        "rarity": 0,
        "team": 4,
        "name": "Russian Su57"
    }
}
```

A successful action will emit an "updateType" event: ``nft, edition, typeId, old field value 1, new field value 1, old field value 2, new field value 2, ...``
example:
```
{
    "contract": "packmanager",
    "event": "updateType",
    "data": {
        "nft": "WAR",
        "edition": 0,
        "typeId": 23,
        "oldTeam": 4,
        "newTeam": 5
    }
}
```

### deleteType:
Deletes a previously created type for an NFT under management. Only possible when an NFT's circulating supply is 0. Calling this action when there is a non-zero circulating supply will result in an error. The 1 BEE fee burned by the original addType action will not be refunded.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * nftSymbol (string): symbol of the NFT (uppercase letters only, max length of 10)
  * edition (integer >= 0): what edition does this type belong to?
  * typeId (integer >= 0): what is the ID number of the type to delete? The combination of nftSymbol, edition, and typeId uniquely identify the type to delete.

* example:
```
{
    "contractName": "packmanager",
    "contractAction": "deleteType",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 3,
        "typeId": 111
    }
}
```

A successful action will emit a "deleteType" event: ``nft, edition, typeId``
example:
```
{
    "contract": "packmanager",
    "event": "deleteType",
    "data": {
        "nft": "WAR",
        "edition": 3,
        "typeId": 111,
    }
}
```

## Setting Names

These actions allow you to setup mappings from edition, foil, category, rarity, and team ID numbers to corresponding text names/labels. 

* **Edition name:** use [registerPack](#registerpack) to set an edition name, and [updateEdition](#updateedition) to change an edition name.
* **NFT instance type name:** use [addType](#addtype) to set a type name, and [updateType](#updatetype) to change a type name.
* **Foil, category, rarity, and team names:** use [setTraitName](#setTraitName) to both set and update these things.

Note that setting edition & NFT instance type names is mandatory, but using [setTraitName](#setTraitName) is entirely optional. However it's strongly encouraged if you want your names to be on-chain and easily available for third party tools to reference.

### setTraitName:
Sets or updates a human readable name for a foil, category, rarity, or team ID number. Essentially these numbers act as look ups into a big on-chain name table.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * nftSymbol (string): symbol of the NFT (uppercase letters only, max length of 10)
  * edition (integer >= 0): what edition does this trait belong to?
  * trait (string): which trait are we setting a name for? Must be one of "foil", "category", "rarity", or "team".
  * index (integer >= 0 and <= 99): ID number of the trait we are setting a name for, should have been previously used in a call to the [addType](#addtype) action
  * name (string): what is the (new) name of the trait? Names must consist of letters, numbers, and whitespaces only, with a maximum length of 100 characters.

* example:
```
{
    "contractName": "packmanager",
    "contractAction": "setTraitName",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 3,
        "trait": "category",
        "index": 4,
        "name": "Amphibious Assault Ship"
    }
}
```

A successful insert (setting a name for the first time) will emit a "setTraitName" event: ``nft, edition, trait, index, name``. A successful update (changing a previously set name) will emit an "updateTraitName" event: ``nft, edition, trait, index, oldName, newName``.
examples:
```
{
    "contract": "packmanager",
    "event": "setTraitName",
    "data": {
        "nft": "WAR",
        "edition": 3,
        "trait": "category",
        "index": 4,
        "name": "Amphibious Assault Ship"
    }
}

{
    "contract": "packmanager",
    "event": "updateTraitName",
    "data": {
        "nft": "WAR",
        "edition": 3,
        "trait": "category",
        "index": 4,
        "oldName": "Aphimbious Assault Ships",    // correcting a spelling & pluralization error
        "newName": "Amphibious Assault Ship"
    }
}
```

## Opening Packs

Once all setup is complete, it's time to open some packs! Note that opening packs has a BEE cost; the contract needs to pay the NFT issuance fees (see [NFT documentation](https://github.com/hive-engine/hivesmartcontracts-wiki/blob/master/NFT-Contracts.md#fees) for more details). The NFT creator is responsible for making sure the packmanager smart contract always has enough BEE on hand for this purpose, and must periodically add more BEE as needed. You can do this using the deposit action.

The following actions are available:

### deposit:
Adds more BEE to the fee pool reserved for a particular NFT under management. This action should be used to fill up the fee pool prior to opening any packs; pools always start at 0 balance. You CANNOT add BEE simply by sending it to the contract; such transactions will result in permanently lost BEE. You MUST use this action instead.
* requires active key: yes

* can be called by: any Hive account

* parameters:
  * nftSymbol (string): symbol of the NFT under management for which you want to reserve BEE for issuance fees
  * amount (string): the quantity of BEE to deposit

* example:
```
{
    "contractName": "packmanager",
    "contractAction": "deposit",
    "contractPayload": {
        "nftSymbol": "WAR",
        "amount": "300.123"
    }
}
```

A successful action will emit a "deposit" event: ``nft, newFeePool (new total balance stored in the contract)``
example:
```
{
    "contract": "packmanager",
    "event": "deleteType",
    "data": {
        "nft": "WAR",
        "newFeePool": "1800.5"
    }
}
```

### open:
Opens one or more packs.
* requires active key: yes

* can be called by: any Hive account

* parameters:
  * packSymbol (string): symbol of the fungible pack token to be opened
  * nftSymbol (string): symbol of the NFT that the pack token opens (it is possible for a pack token to open more than one type of NFT)
  * packs (integer >= 1 and <= 999): number of packs to open, calling account must have at least this much packSymbol in its balance

The number of NFT instances that will be generated by this action is **packs X cardsPerPack (for this particular pack / NFT pair)**. This number must be less than or equal to 60; an attempt to open too many packs will result in an error. For example, if you want to open 5 packs that contain 10 NFT instances each, you'll get 50 NFT instances. Or you could open 2 packs that contain 30 NFT instances each. But you can't open 3 packs that contain 30 NFT instances each, because that would put you over the maximum limit of 60 in one action.

This action will also result in an error if the contract's fee pool balance is too low to pay the NFT issuance fees.

* example:
```
{
    "contractName": "packmanager",
    "contractAction": "open",
    "contractPayload": {
         "packSymbol": "PACK",
         "nftSymbol": "WAR",
         "packs": 5
    }
}
```

A successful action will emit NFT issuance events for each NFT instance generated. See [NFT documentation](https://github.com/hive-engine/hivesmartcontracts-wiki/blob/master/NFT-Contracts.md#issue) for more details.

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.
## params:
contains contract parameters such as the current fees
* fields
  * registerFee = the cost in BEE to register a pack token / NFT settings pair
  * typeAddFee = the cost in BEE to add an NFT instance type

## managedNfts:
contains information about NFTs under management of the packmanager smart contract
* fields
  * nft = symbol of the NFT under management
  * feePool = current BEE balance available to the contract for issuing NFT instances
  * editionMapping = information about what editions exist for this NFT

editionMapping is a dictionary that maps edition numbers to information about the edition as follows:
  * nextTypeId = the next type ID that will be assigned for this NFT & edition when the [addType action](#addtype) is called
  * editionName = name of this edition
  * categoryRO = boolean flag indicating if categories of NFT instance types should be editable
  * rarityRO = boolean flag indicating if rarity of NFT instance types should be editable
  * teamRO = boolean flag indicating if team of NFT instance types should be editable
  * nameRO = boolean flag indicating if name of NFT instance types should be editable

example query results:
```
[ { _id: 1,
    nft: 'WAR',
    feePool: '1000',
    editionMapping: { '0': {
        nextTypeId: 33,
        editionName: 'Ultimate War Edition',
        categoryRO: false,
        rarityRO: false,
        teamRO: false,
        nameRO: false
    }, '1': {
        nextTypeId: 12,
        editionName: 'War Modern Expansion',
        categoryRO: false,
        rarityRO: false,
        teamRO: true,
        nameRO: true
    } } } ]
```

There is a database index on the nft field, which serves as a unique primary key.

## types
contains details about NFT instance types added through the [addType action](#addtype)
* fields
  * nft = symbol of the NFT under management
  * edition = edition number that this type belongs to
  * typeId = the type's ID number
  * category = the type's category ID number
  * rarity = the type's rarity ID number
  * team = the type's team ID number
  * name = human friendly text label identifying this type
  
There are database indexes on the nft, edition, and typeId fields. The combination of these fields can be used as a unique primary key.

example query results:
```
[ { _id: 1,
    nft: 'WAR',
    edition: 0,
    typeId: 0,
    category: 1,
    rarity: 1,
    team: 3,
    name: 'Tank' },
  { _id: 2,
    nft: 'WAR',
    edition: 0,
    typeId: 1,
    category: 2,
    rarity: 3,
    team: 3,
    name: 'Submarine' },
  { _id: 3,
    nft: 'WAR',
    edition: 1,
    typeId: 0,
    category: 3,
    rarity: 4,
    team: 0,
    name: 'B52 Bomber' },
  { _id: 4,
    nft: 'WAR',
    edition: 1,
    typeId: 1,
    category: 3,
    rarity: 5,
    team: 3,
    name: 'Japanese Zero Fighter' } ]
```

## packs
contains pack settings linking fungible pack tokens to corresponding NFTs, which determine how NFTs are generated from the packs
* fields
  * account = Hive account that created these settings and has permission to edit them
  * symbol = symbol of the pack token
  * nft = symbol of the NFT to associate with the pack token
  * edition = edition number of the NFT instances to be generated from the pack token
  * cardsPerPack = number of NFT instances generated from opening 1 pack
  * foilChance = partition array defining percent chances for determining foil when opening packs
  * categoryChance = partition array defining percent chances for determining category when opening packs
  * rarityChance = partition array defining percent chances for determining rarity when opening packs
  * teamChance = partition array defining percent chances for determining team when opening packs
  * numRolls = maximum number of random rolls to be used when generating NFT instances (re-rolls happen when there is not at least one defined NFT instance type for a given combination of category / rarity / team)
  * isFinalized = boolean flag indicating if these pack settings are read only or not (defaults to false)

There are database indexes on the account, symbol, and nft fields. The combination of symbol and nft fields can be used as a unique primary key.

example query results:
```
[ { _id: 1,
    account: 'cryptomancer',
    symbol: 'PACK',
    nft: 'WAR',
    edition: 0,
    cardsPerPack: 5,
    foilChance: [ 50, 100 ],
    categoryChance: [ 70, 90, 100 ],
    rarityChance: [ 600, 800, 900, 975, 1000 ],
    teamChance: [ 1000, 2800, 3000 ],
    numRolls: 10,
    isFinalized: false } ]
```

## foils
contains a mapping of foil ID numbers to text labels, essentially a big string look-up table
* fields
  * nft = symbol of the NFT under management
  * edition = edition number that this foil belongs to
  * index = ID number of the foil
  * name = human friendly text label identifying this foil

There are database indexes on the nft, edition, and index fields. The combination of these fields can be used as a unique primary key.

example query results:
```
[ { _id: 1, nft: 'WAR', edition: 0, index: 0, name: 'Standard' },
  { _id: 2, nft: 'WAR', edition: 0, index: 1, name: 'Gold' } ]
```

## categories
contains a mapping of category ID numbers to text labels, essentially a big string look-up table
* fields
  * nft = symbol of the NFT under management
  * edition = edition number that this category belongs to
  * index = ID number of the category
  * name = human friendly text label identifying this category

There are database indexes on the nft, edition, and index fields. The combination of these fields can be used as a unique primary key.

example query results:
```
[ { _id: 1, nft: 'WAR', edition: 0, index: 0, name: 'Air' },
  { _id: 2, nft: 'WAR', edition: 0, index: 1, name: 'Ground' },
  { _id: 3, nft: 'WAR', edition: 0, index: 2, name: 'Naval' } ]
```

## rarities
contains a mapping of rarity ID numbers to text labels, essentially a big string look-up table
* fields
  * nft = symbol of the NFT under management
  * edition = edition number that this rarity belongs to
  * index = ID number of the rarity
  * name = human friendly text label identifying this rarity

There are database indexes on the nft, edition, and index fields. The combination of these fields can be used as a unique primary key.

example query results:
```
[ { _id: 1, nft: 'WAR', edition: 0, index: 0, name: 'Common' },
  { _id: 2, nft: 'WAR', edition: 0, index: 1, name: 'Uncommon' },
  { _id: 3, nft: 'WAR', edition: 0, index: 2, name: 'Rare' },
  { _id: 4, nft: 'WAR', edition: 0, index: 3, name: 'Legendary' } ]
```

## teams
contains a mapping of team ID numbers to text labels, essentially a big string look-up table
* fields
  * nft = symbol of the NFT under management
  * edition = edition number that this team belongs to
  * index = ID number of the team
  * name = human friendly text label identifying this team

There are database indexes on the nft, edition, and index fields. The combination of these fields can be used as a unique primary key.

example query results:
```
[ { _id: 1, nft: 'WAR', edition: 0, index: 0, name: 'Marines' },
  { _id: 2, nft: 'WAR', edition: 0, index: 1, name: 'Rogue Squadron' },
  { _id: 3, nft: 'WAR', edition: 0, index: 2, name: 'Naval Strike Group' },
  { _id: 4, nft: 'WAR', edition: 0, index: 3, name: 'Light Brigade' } ]
```
