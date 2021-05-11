# In this tutorial we will be covering

web reference : [Creating a Truffle project using a Truffle Box](https://www.trufflesuite.com/tutorial)

1. Setting up the development environment
2. Creating a Truffle project using a Truffle Box
3. Writing the smart contract
4. Compiling and migrating the smart contract
5. Testing the smart contract
6. Creating a user interface to interact with the smart contract (not fully documented )
7. Interacting with the dapp in a browser

## Background

`Pete Scandlon` of Pete's Pet Shop is interested in using Ethereum as an efficient way to handle their pet adoptions.
The store has space for `16 pets` at a given time, and they already have a database of pets.
As an initial proof of concept, Pete wants to see a dapp which associates an Ethereum address with a pet to be adopted.

**Install Truffle:**

> npm install -g truffle
> $truffle version

    truffle v5.0.17 (core: 5.0.16)\
    Solidity v0.5.0 (solc-js)\
    Node v12.16.2\
    Web3.js v1.0.0-beta.37

## Truffle Directory Structure

The default Truffle directory structure contains the following:

### contracts

Contains the `Solidity` source files for our smart contracts. There is an important contract in here called `Migrations.sol`.

### migrations

Truffle uses a migration system to handle smart contract deployments. A migration is an additional special smart contract that keeps track of changes.

### test

Contains both JavaScript and Solidity tests for our smart contracts

### truffle-config.js

Truffle `configuration` file

## Writing the smart contract

We'll start our dapp by writing the smart contract that acts as the back-end logic and storage.

1. Create a new file named `Adoption.sol` in the `contracts/` directory.

1. variables

```java
//This is an array of Ethereum addresses
{
   address[16] public adopters;
```

1. functions

```java
}
//1. Adopting a pet
function adopt(uint petId) public returns (uint){

    require(petId >= 0 && petId <=  15);

    adopters[petId]=msg.sender;
    return petId;
}
//2. Retrieving the adopters
function getAdopters() public view returns (address[16] memory) {
  return adopters;
}

```

## Compilation

`truffle compile`

```
$truffle compile

Compiling your contracts...
===========================
> Compiling ./contracts/Migrations.sol
> Compiling ./contracts/adoption.sol
> Artifacts written to /Users/mac/Documents/Development/Blockchain/2021/tracking-system-pet-shop/build/contracts
> Compiled successfully using:
   - solc: 0.5.0+commit.1d4f565a.Emscripten.clang
```

## Migration

Now that we've successfully compiled our contracts, it's time to migrate them to the blockchain!

1. Create a new file named `2_deploy_contracts.js` in the `migrations/` directory.

1. Add the following content to the `2_deploy_contracts.js` file:

   ```java
   var Adoption = artifacts.require("Adoption");

   module.exports = function(deployer) {
   deployer.deploy(Adoption);
   };
   ```

1. Before we can migrate our contract to the blockchain, we need to have a blockchain running. For this tutorial, we're going to use `Ganache`, a `personal blockchain` for Ethereum development you can use to\

   - deploy contracts,
   - develop applications,
   - and run tests.

1. Back in our terminal, migrate the contract to the blockchain.
   \
   `truffle migrate`

```
$truffle migrate

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.


Starting migrations...
======================
> Network name:    'development'
> Network id:      5777
> Block gas limit: 0x6691b7


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0x178054ae79857e56f1830313a33269a22c4db5aba79901c3ed999842ab5cc61d
   > Blocks: 0            Seconds: 0
   > contract address:    0x4A247CE60ed2DAEd0018a99A474B26b436c8C263
   > block number:        1
   > block timestamp:     1620595007
   > account:             0x68F48429F451934fD1032ba63be0f72eB10424EB
   > balance:             99.99524454
   > gas used:            237773
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00475546 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00475546 ETH


2_deploy_contracts.js
=====================

   Deploying 'Adoption'
   --------------------
   > transaction hash:    0x80618ebd4b7229240b5876232b8cbbb486c851604638629984b46c7abd3ac00b
   > Blocks: 0            Seconds: 0
   > contract address:    0xb8D9F6e4ED8a8f8d673EF71735B78b07B7715dca
   > block number:        3
   > block timestamp:     1620595010
   > account:             0x68F48429F451934fD1032ba63be0f72eB10424EB
   > balance:             99.98932784
   > gas used:            253820
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.0050764 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:           0.0050764 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.00983186 ETH
```

### ACCOUNT

> ADDRESS:
> `0x68F48429F451934fD1032ba63be0f72eB10424EB`

> BALANCE:
> 99.99 ETH

### BLOCK 4

> GAS USED:
> 27015

> GAS LIMIT:
> 6721975

> MINED ON:
> 2021-05-10 00:16:51

> BLOCK HASH:
> 0xff3b3ced7aaf850867535310c1f7ec2101d1e171a24aa039903be680a17d0c88

> TX HASH:
> 0x9e7caba27f0f8b3265a56985577b7f2659768954bce6f555872d1b998d289757

> CONTRACT CALL
> FROM ADDRESS:
> `0x68F48429F451934fD1032ba63be0f72eB10424EB`

> TO CONTRACT ADDRESS:
> `0x4A247CE60ed2DAEd0018a99A474B26b436c8C263`

> GAS USED:
> 27015

- In `Ganache`, note that
  - the state of the blockchain has changed. The blockchain now shows that the current block, previously `0`, is now `4`.
  - first account originally had `100` ether, it is now lower `99.99` ether

## Testing the smart contract using Solidity

1. Create a new file named `TestAdoption.sol` in the `test/` directory.

2. Add the following content to the `TestAdoption.sol` file:

```java
pragma solidity ^0.5.0;

import "truffle/Assert.sol";//Gives us various assertions to use in our tests.
import "truffle/DeployedAddresses.sol";//This smart contract gets the address of the deployed contract.
import "../contracts/Adoption.sol";//The smart contract we want to test.

contract TestAdoption {

    // The address of the adoption contract to be tested
    Adoption adoption = Adoption(DeployedAddresses.Adoption());

    // The id of the pet that will be used for testing
    uint expectedPetId = 8;

    // The expected owner of adopted pet is this contract
    address expectedAdopter = address(this);





    // Testing the adopt() function
    function testUserCanAdoptPet() public {

        uint returnedId = adoption.adopt(expectedPetId);

        Assert.equal(returnedId, expectedPetId,"Adoption of the expected pet should match what is returned.");
    }

    // Testing retrieval of a single pet's owner
    function testGetAdopterAddressByPetId() public {
        address adopter = adoption.adopters(expectedPetId);

        Assert.equal(adopter, expectedAdopter, "Owner of the expected pet should be this contract");
    }

    // Testing retrieval of all pet owners
    function testGetAdopterAddressByPetIdInArray() public {
        // Store adopters in memory rather than contract's storage
        address[16] memory adopters = adoption.getAdopters();

        Assert.equal(adopters[expectedPetId], expectedAdopter, "Owner of the expected pet should be this contract");
    }
}
```

## Running the tests

`truffle test`

```
$truffle test
Using network 'development'.


Compiling your contracts...
===========================
> Compiling ./test/TestAdoption.sol
> Artifacts written to /var/folders/_q/hrsdkrv93t96fxcxd_wmzcp80000gn/T/test-121410-3502-1795zxv.8ij6
> Compiled successfully using:
   - solc: 0.5.0+commit.1d4f565a.Emscripten.clang



  TestAdoption
    ✓ testUserCanAdoptPet (249ms)
    ✓ testGetAdopterAddressByPetId (465ms)
    ✓ testGetAdopterAddressByPetIdInArray (539ms)


  3 passing (25s)
```

## Creating a user interface to interact with the smart contract

it's time to create a UI so that Pete has something to use for his pet shop!

### Instantiating web3

1. Open `/src/js/app.js` in a text editor.
1. Note that

   1. global App object to manage our application,
   1. load in the pet data `pets.json` in `init()` petTemplate/petsRow
   1. then call the function `initWeb3()`.
      ```js
      // Request account access
      await window.ethereum.enable();
      ```
   1. The web3 JavaScript library interacts with the Ethereum blockchain. It can
      1. retrieve user accounts,
      1. send transactions,
      1. interact with smart contracts, and more.

      Remove the multi-line comment from within initWeb3 and replace it with the following:

      ```js
      // Modern dapp browser...
      if (window.ethereum) {
      App.web3Provider = window.ethereum;

      try {
         //Request account access
         await window.ethereum.enable();
      } catch (error) {
         //User denied account access
         console.log("User denied account access");
      }
      }

      // Legacy dapp browsers..
      else if (window.web3) {
      App.web3Provider = window.web3.currentProvider;
      }
      // if no injected web3 instance is detected
      else {
      App.web3Provider = new Web3.providers.HttpProvider("http://localhost:8545");
      }

      web3 = new Web3(App.web3Provider);
      ```
