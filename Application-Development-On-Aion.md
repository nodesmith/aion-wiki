This tutorial is designed to facilitate learning how to make projects that interact with the Aion kernel through its available APIs.
Two APIs are provided to developers, the [Java API](https://github.com/aionnetwork/aion_api) and the [JSON-RPC (Web3) API](https://github.com/aionnetwork/aion_web3).

The contents below show code examples by functionality and by JSON-RPC method.

<details>
<summary><i><b>Contents by Topic</b></i></summary>
<br/>

- [Java API](#java)
    - [Setup](#java-setup)
    - [Dependencies](#java-dep)
    - [Use](#java-use)
- [JSON-RPC (Web3) API](#web3)
    - [Setup](#web3-setup)
    - [Console](#web3-console)
    - [Use](#web3-use)
- [Examples](#examples)
    - [Account Functionality](#account)
    - [Block Functionality](#block)
    - [Transaction Functionality](#transaction)
    - [Contract Functionality](#contract)
    - [Network Functionality](#network)
</details>
<br/>

<details>
<summary><i><b>Contents by JSON-RPC Method</b></i></summary>
<br/>

- Examples for supported methods:
    - [`eth_accounts`](#all-acc)
    - [`eth_blockNumber`](#get-block)
    - [`eth_call`](#call)
    - [`eth_coinbase`](#miner-acc)
    - [`eth_compileSolidity`](#comp-sol)
    - [`eth_estimateGas`](#nrg-estimate)
    - [`eth_gasPrice`](#nrg-price)
    - [`eth_getBalance`](#acc-balance)
    - [`eth_getBlockByHash`](#block-hash)
    - [`eth_getBlockByNumber`](#block-number)
    - [`eth_getBlockTransactionCountByHash`](#tx-hash)
    - [`eth_getBlockTransactionCountByNumber`](#tx-number)
    - [`eth_getCode`](#get-code)
    - [`eth_getCompilers`](#get-comp)
    - [`eth_getStorageAt`](#get-storage)
    - [`eth_getTransactionByBlockHashAndIndex`](#tx-hash-idx)
    - [`eth_getTransactionByBlockNumberAndIndex`](#tx-number-idx)
    - [`eth_getTransactionByHash`](#tx-hash)
    - [`eth_getTransactionCount`](#acc-txs)
    - [`eth_getTransactionReceipt`](#tx-receipt)
    - [`eth_sendTransaction`](#send-tx)
    - [`eth_sign`](#sign)
    - [`eth_sendRawTransaction`](#raw-tx)
    - [`eth_syncing`](#sync-status)
    - [`net_listening`](#net-listen)
    - [`net_peerCount`](#peer-count)

- Details on methods that are not supported:
    - [`eth_compileLLL`](#get-comp)
    - [`eth_compileSerpent`](#get-comp)
    - [`eth_getUncleByBlockHashAndIndex`](#block)
    - [`eth_getUncleByBlockNumberAndIndex`](#block)
    - [`eth_getUncleCountByBlockHash`](#block)
    - [`eth_getUncleCountByBlockNumber`](#block)
    - [`net_version`](#network)

</details>
<br/>

For functionality related to mining please review our reference miners implementations in the [aion_miner](https://github.com/aionnetwork/aion_miner) project.

Additional API use examples are available in:
* [Java API introduction and tutorial](https://github.com/aionnetwork/aion_api/wiki/Aion-Java-API-introduction-and-tutorial);
* [Java API functionality tests](https://github.com/aionnetwork/aion_api/blob/master/test/org/aion/api/test/BaseAPITests.java);
* [Java API contract tests](https://github.com/aionnetwork/aion_api/blob/master/test/org/aion/api/test/ContractTests.java);
* [Aion Web3 wiki](https://github.com/aionnetwork/aion_web3/wiki) and [examples](https://github.com/aionnetwork/aion_web3/tree/master/example);
* [Aion Web3 functionality tests](https://github.com/aionnetwork/aion_qa/tree/master/Web3/test);
* [Aion Web3 contract storage test](https://github.com/aionnetwork/aion_qa/tree/master/TestBench/contractStorageTestWeb3).

## <a name="java"></a>Java API

The Java API provides high throughput connection to the kernel and is ideal for Java applications.

### <a name="java-setup"></a>Setup

The use of the Java API is enabled from the kernel configuration file `config.xml`.
The `active` attribute must be set to `"true"` before starting the kernel to allow the API to interact with the Aion network through the local kernel.

```xml
<api>
  <java active="true" ip="127.0.0.1" port="8547"></java>
  <!-- other API settings -->
</api>
```

### <a name="java-dep"></a>Dependencies

An application interacting with the Aion kernel through the Java API must include the following `jar` dependencies:
* `modAionBase.jar`
* `modAionApi-[version].jar`

The `modAionBase.jar` contains the classes in [`modAionBase`](https://github.com/aionnetwork/aion/tree/master/modAionBase).
The jar can be found in the `mod` folder delivered with every [kernel release](https://github.com/aionnetwork/aion/releases), or it can be compiled from the sources available online inside the module.

The latest release for `modAionApi-[version].jar` is available online in the [`aion_api`](https://github.com/aionnetwork/aion_api/releases) project.
Alternatively, the file can be created by compiling the source files.

### <a name="java-use"></a>Use

To use the Java API in an application, first the `IAionAPI api` object must be initialized.
Then a connection must be establised to a kernel, as shown below:

```java
// connect to Java API
IAionAPI api = IAionAPI.init();
ApiMsg apiMsg = api.connect(IAionAPI.LOCALHOST_URL);

// failed connection
if (apiMsg.isError()) {
    System.out.format("Could not connect due to <%s>%n", apiMsg.getErrString());
    System.exit(-1);
}

// code to use API
// ...

// disconnect from api  
api.destroyApi();
```

If the kernel is running on the same machine as the application using the API, the connection is established through the call:
```java
api.connect(IAionAPI.LOCALHOST_URL);
```
Otherwise, the host and port where the kernel is running must be passed as parameters, for example, assuming the IP is `123.123.123.123` and the port is `4567`:
```java
api.connect("tcp://123.123.123.123:4567");
```

## <a name="web3"></a>JSON-RPC (Web3) API

The JSON-RPC API was designed to be highly compatible with existing Ethereum client interfaces and allows for easy migration of existing DApps built on Ethereum to Aion.

### <a name="web3-setup"></a>Setup

The use of the JSON-RPC API is enabled from the kernel configuration file `config.xml`.
The `active` attribute must be set to `"true"` before starting the kernel to allow the API to interact with the Aion network through the local kernel.

```xml
<api>
  <rpc active="true" ip="127.0.0.1" port="8545">
    <!--boolean, enable/disable cross origin requests (browser enforced)-->
    <cors-enabled>false</cors-enabled>
    <!--comma-separated list, APIs available: web3,net,debug,personal,eth,stratum-->
    <apis-enabled>web3,eth,personal,stratum,net</apis-enabled>
    <!--size of thread pool allocated for rpc requests-->
    <threads>1</threads>
  </rpc>
  <!-- other API settings -->
</api>
```

### <a name="web3-console"></a>Console

To interact with the kernel using a console first ensure that you have all the [requirements](https://github.com/aionnetwork/aion_web3#requirements) installed.
Next, either clone the `aion_web3` project or use the `web3` folder provided with your [kernel release](https://github.com/aionnetwork/aion/releases) and execute the required installation steps from [setup](https://github.com/aionnetwork/aion_web3#setup).
Finally, run the following in a terminal:

```bash
cd /path/to/aion/web3
node console.js
```
To exit the console run:

```js
process.exit()
```

### <a name="web3-use"></a>Use

For JavaScript applications, add the following lines to your code to interract with the API:

```js
const Web3 = require('/path/to/aion/web3');
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
```

Note that if the kernel is running on a different machine or using a different port the string `http://localhost:8545` must be replaced with the correct configuration.

## <a name="examples"></a>Examples

In each subsection below we provide code snippets illustrating the specific interaction with the API.

To jump directly to the complete examples, click the links below:
- [account example](#acc-example)
- [block example](#blk-example)
- [transaction example](#tx-example)
- [contract example](#ctr-example)
- [network example](#net-example)

To run the JavaScript code examples in a terminal, navigate to the test folder and run the script using:
```bash
mocha example.js
```
To install Mocha follow the instructions given [here](https://mochajs.org/#installation).

### <a name="account"></a>Account Functionality

Next we illustrate the API calls for querying the following information:
* [all client accounts](#all-acc)
* [coinbase account](#miner-acc)
* [account balance](#acc-balance)
* [account transaction count](#acc-txs)

The [final subsection](#acc-example) contains code illustrating all of the above interactions.

#### <a name="all-acc"></a>Retrieve All Client Accounts

The examples below show how to query the APIs for the list of addresses owned by the client.
The functionality is compatible with [`eth_accounts`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_accounts).
In each code snippet, the addresses are retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get accounts from API
List<Address> accounts = api.getWallet().getAccounts().getObject();

// print accounts to standard output
System.out.format("the keystore contains %d accounts, as follow:%n", accounts.size());
for (Address account : accounts) {
    System.out.format("\t%s%n", account.toString());
}
```

* Sample output:

```
the keystore contains 4 accounts, as follow:
	a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e
	a04164b007f75fb77e79ebfd90ace768e5d8ab26855035df4f2b9e03c1ca40f4
	a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5
	a0eb00973749a0f7b7f8e66e2a13bc2725e3a02804d44a0b88c1666512031591
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get accounts from API
let accounts = web3.eth.accounts;

// print accounts to standard output
console.log("the keystore contains " + accounts.length + " accounts, as follow:");
console.log(accounts);
```

* Alternative way to get the accounts:

```js
// get accounts from API
let accounts = web3.personal.listAccounts;

// print accounts to standard output
console.log("the keystore contains " + accounts.length + " accounts, as follow:");
console.log(accounts);
```

* Sample output:
```
the keystore contains 4 accounts, as follow:
[ '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  '0xa04164b007f75fb77e79ebfd90ace768e5d8ab26855035df4f2b9e03c1ca40f4',
  '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  '0xa0eb00973749a0f7b7f8e66e2a13bc2725e3a02804d44a0b88c1666512031591' ]
```

</details>

#### <a name="miner-acc"></a>Retrieve Coinbase Account

The examples below show how to query the APIs for the address configured to receive the mining rewards.
The functionality is compatible with [`eth_coinbase`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_coinbase).
In each code snippet, the address is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get miner account
Address account = api.getWallet().getMinerAccount().getObject();

// print retrieved value
System.out.format("coinbase account = %s%n", account.toString());
```

* Sample output:

```
coinbase account = a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get miner account
let miner = web3.eth.coinbase;

// print retrieved value
console.log("coinbase account = " + miner);
```

* Sample output:

```
coinbase account = 0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e
```

</details>

#### <a name="acc-balance"></a>Retrieve Account Balance

The examples below show how to query the APIs for the balance of a given address.
The functionality is compatible with [`eth_getBalance`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getbalance).
In each code snippet, the balance is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// interpret string as address
Address account = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");

// get account balance
BigInteger balance = api.getChain().getBalance(account).getObject();

// print balance
System.out.format("%s has balance = %d nAmp (over %d AION)%n",
                  account.toString(),
                  balance,
                  balance.divide(BigInteger.TEN.pow(18)));
```

* Sample output:

```
a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 has balance = 1003070000000000000 nAmp (over 1 AION)
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// set address
let account = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';

// get account balance
let balance = web3.eth.getBalance(account);

// print balance
console.log(account + " has balance = " + balance + " nAmp (" + balance.shiftedBy(-18).toString() + " AION)");
```

* Sample output:

```
a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 has balance = 1003070000000000000 nAmp (1.00307 AION)
```

</details>

#### <a name="acc-txs"></a>Retrieve Account Transaction Count

The examples below show how to query the APIs for the number of transactions sent by a given address.
The functionality is compatible with [`eth_getTransactionCount`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactioncount).
In each code snippet, the transaction count is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// interpret string as address
// note that hex prefix '0x' is optional
Address account = Address.wrap("0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");

// get number of transactions sent by account
BigInteger txCount = api.getChain().getNonce(account).getObject();

// print performed transactions
System.out.format("%s performed %d transactions%n", account.toString(), txCount);
```

* Sample output:

```
a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 performed 33 transactions
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// set address
// note that hex prefix '0x' is optional
account = '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';

// get number of transactions sent by account
let txCount = web3.eth.getTransactionCount(account);

// print performed transactions
console.log(account + " performed " + txCount + " transactions");
```

* Sample output:

```
a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 performed 33 transactions
```

</details>

#### <a name="acc-example"></a>Complete Examples

Each code example below retrieves and prints to the standard output the following:
1. all keystore addresses,
2. the coinbase (miner) address,
3. the balance for a given account,
4. the number of transactions performed by a given account, and
5. the balance and number of transactions performed by each account in the keystore.

<details>
<summary><i>Java Code</i></summary>
<br/>

```java
package org.aion.tutorials;

import org.aion.api.IAionAPI;
import org.aion.api.type.ApiMsg;
import org.aion.base.type.Address;

import java.math.BigInteger;
import java.util.List;

public class AccountExample {

    public static void main(String[] args) {

        // connect to Java API
        IAionAPI api = IAionAPI.init();
        ApiMsg apiMsg = api.connect(IAionAPI.LOCALHOST_URL);

        // failed connection
        if (apiMsg.isError()) {
            System.out.format("Could not connect due to <%s>%n", apiMsg.getErrString());
            System.exit(-1);
        }

        // 1. eth_accounts

        // get accounts from API
        List<Address> accounts = api.getWallet().getAccounts().getObject();

        // print accounts to standard output
        System.out.format("the keystore contains %d accounts, as follow:%n", accounts.size());
        for (Address account : accounts) {
            System.out.format("\t%s%n", account.toString());
        }

        // 2. eth_coinbase

        // get miner account
        Address account = api.getWallet().getMinerAccount().getObject();

        // print retrieved value
        System.out.format("%ncoinbase account = %s%n", account.toString());

        // 3. eth_getBalance

        // interpret string as address
        account = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");

        // get account balance
        BigInteger balance = api.getChain().getBalance(account).getObject();

        // print balance
        System.out.format("%n%s has balance = %d nAmp (over %d AION)%n",
                          account.toString(),
                          balance,
                          balance.divide(BigInteger.TEN.pow(18)));

        // 4. eth_getTransactionCount

        // interpret string as address
        // note that hex prefix '0x' is optional
        account = Address.wrap("0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");

        // get number of transactions sent by account
        BigInteger txCount = api.getChain().getNonce(account).getObject();

        // print performed transactions
        System.out.format("%n%s performed %d transactions%n", account.toString(), txCount);

        // 5. eth_getBalance & eth_getTransactionCount for each keystore account

        // print balance & tx count to standard output
        System.out.format("%nthe keystore contains %d accounts, as follow:%n", accounts.size());
        for (Address acc : accounts) {
            System.out.format("\t%s balance = %22d nAmp, tx count = %2d%n",
                              acc.toString(),
                              api.getChain().getBalance(acc).getObject(),
                              api.getChain().getNonce(acc).getObject());
        }

        // disconnect from api
        api.destroyApi();

        System.exit(0);
    }
}
```

* Sample output:

```
the keystore contains 4 accounts, as follow:
	a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e
	a04164b007f75fb77e79ebfd90ace768e5d8ab26855035df4f2b9e03c1ca40f4
	a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5
	a0eb00973749a0f7b7f8e66e2a13bc2725e3a02804d44a0b88c1666512031591

coinbase account = a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e

a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 has balance = 1003070000000000000 nAmp (over 1 AION)

a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 performed 33 transactions

the keystore contains 4 accounts, as follow:
	a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e balance = 1251613319407197678222 nAmp, tx count = 35
	a04164b007f75fb77e79ebfd90ace768e5d8ab26855035df4f2b9e03c1ca40f4 balance =    5000000000000000000 nAmp, tx count =  0
	a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 balance =    1003070000000000000 nAmp, tx count = 33
	a0eb00973749a0f7b7f8e66e2a13bc2725e3a02804d44a0b88c1666512031591 balance =                      0 nAmp, tx count =  0
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

```js
const Web3 = require('/path/to/aion/web3');
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

// 1.a) eth_accounts

// get accounts from API
let accounts = web3.eth.accounts;

// print accounts to standard output
console.log("the keystore contains " + accounts.length + " accounts, as follow:");
console.log(accounts);

// 1.b) eth_accounts

// get accounts from API
accounts = web3.personal.listAccounts;

// print accounts to standard output
console.log("\nthe keystore contains " + accounts.length + " accounts, as follow:");
console.log(accounts);

// 2. eth_coinbase

// get miner account
let miner = web3.eth.coinbase;

// print retrieved value
console.log("\ncoinbase account = " + miner);

// 3. eth_getBalance

// set address
let account = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';

// get account balance
let balance = web3.eth.getBalance(account);

// print balance
console.log("\n" + account + " has balance = " + balance + " nAmp (" + balance.shiftedBy(-18).toString() + " AION)");

// 4. eth_getTransactionCount

// set address
// note that hex prefix '0x' is optional
account = '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';

// get number of transactions sent by account
let txCount = web3.eth.getTransactionCount(account);

// print performed transactions
console.log("\n" + account + " performed " + txCount + " transactions");

// 5. eth_getBalance & eth_getTransactionCount for each keystore account

// print balance & tx count to standard output
console.log("\nthe keystore contains " + accounts.length + " accounts, as follow:");
accounts.forEach(print);

function print(acc, index) {
    console.log("\t" + acc + " balance = " + web3.eth.getBalance(acc) + " nAmp, tx count = " + web3.eth.getTransactionCount(acc));
}
```

* Sample output:

```
the keystore contains 4 accounts, as follow:
[ '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  '0xa04164b007f75fb77e79ebfd90ace768e5d8ab26855035df4f2b9e03c1ca40f4',
  '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  '0xa0eb00973749a0f7b7f8e66e2a13bc2725e3a02804d44a0b88c1666512031591' ]

the keystore contains 4 accounts, as follow:
[ '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  '0xa04164b007f75fb77e79ebfd90ace768e5d8ab26855035df4f2b9e03c1ca40f4',
  '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  '0xa0eb00973749a0f7b7f8e66e2a13bc2725e3a02804d44a0b88c1666512031591' ]

coinbase account = 0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e

a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 has balance = 1003070000000000000 nAmp (1.00307 AION)

0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 performed 33 transactions

the keystore contains 4 accounts, as follow:
	0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e balance = 1.253111308690440988407e+21 nAmp, tx count = 35
	0xa04164b007f75fb77e79ebfd90ace768e5d8ab26855035df4f2b9e03c1ca40f4 balance = 5000000000000000000 nAmp, tx count = 0
	0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5 balance = 1003070000000000000 nAmp, tx count = 33
	0xa0eb00973749a0f7b7f8e66e2a13bc2725e3a02804d44a0b88c1666512031591 balance = 0 nAmp, tx count = 0
```

</details>

### <a name="block"></a>Block Functionality

Next we illustrate the API calls for querying the following information:
* [recent block number](#get-block)
* [block information given its hash](#block-hash)
* [block information given its number](#block-number)
* [number of transactions in block given its hash](#tx-hash)
* [number of transactions in block given its number](#tx-number)

Uncles are not a supported feature in the Aion blockchain. Information about side chain blocks can be requested using the block hash, however the following functionality does not have a direct implementation:
* [`eth_getUncleByBlockHashAndIndex`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getunclebyblockhashandindex)
* [`eth_getUncleByBlockNumberAndIndex`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getunclebyblocknumberandindex)
* [`eth_getUncleCountByBlockHash`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getunclecountbyblockhash)
* [`eth_getUncleCountByBlockNumber`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getunclecountbyblocknumber)

The [final subsection](#blk-example) contains code illustrating all of the above interactions.

#### <a name="get-block"></a>Retrieve Recent Block Number

The examples below show how to query the APIs for the number of the most recent block. The functionality is compatible with [`eth_blockNumber`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_blocknumber). In each code snippet, the block number is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get block number from API  
long blockNumber = api.getChain().blockNumber().getObject();  

// print block number  
System.out.format("current block = %d%n", blockNumber);
```

* Sample output:

```
current block = 245973
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get block number from API
let blockNumber = web3.eth.blockNumber;

// print block number
console.log("current block = " + blockNumber);
```

* Sample output:
```
current block = 245973
```

</details>

#### <a name="block-hash"></a>Retrieve Block by Hash

The examples below show how to query the APIs for information about a block given its hash. The functionality is compatible with [`eth_getBlockByHash`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getblockbyhash) except for the information on uncles since uncles are not supported in the Aion blockchain. In each code snippet, the block information is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify hash  
Hash256 hash = Hash256.wrap("0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab");  

// get block with given hash  
Block block = api.getChain().getBlockByHash(hash).getObject();  

// print block information  
System.out.format("%nblock with transaction hashes:%n%s%n", block.toString());  

// get block details with given hash  
// with full transaction information  
BlockDetails details = api.getAdmin().getBlockDetailsByHash(hash).getObject();  

// print block information  
System.out.format("%nblock with transaction information:%n%s%n", details.toString());
```

* Sample output:

```
block with transaction hashes:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions: ,
[
  0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80
],
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674


block with transaction information:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions:
[
  nrgPrice: 10000000000,
  nrg: 84107,
  transactionIndex: 0,
  nonce: 367,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 84047,
  transactionIndex: 1,
  nonce: 368,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320982,
  transactionIndex: 2,
  nonce: 446,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1ed,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25b,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x91ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db,
                  0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76
                ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320922,
  transactionIndex: 3,
  nonce: 447,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dad,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201a,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x9ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa,
                  0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe
                ]
  ]
]
,
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify hash
let hash = '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab';

// get block with given hash
let block = web3.eth.getBlock(hash, false);

// print block information
console.log("block with transaction hashes:");
console.log(block);

// get block with given hash
// with full transaction information
block = web3.eth.getBlock(hash, true);

// print block information
console.log("\nblock with transaction information:");
console.log(block);
```

* Sample output:

```
block with transaction hashes:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
     '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
     '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
     '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80' ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }

block with transaction information:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 0,
       nonce: 367,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 1,
       nonce: 368,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 2,
       nonce: 446,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 3,
       nonce: 447,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 } ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }
```

</details>

#### <a name="block-number"></a>Retrieve Block by Number

The examples below show how to query the APIs for information about a block from the main chain given its number. The functionality is compatible with [`eth_getBlockByNumber`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getblockbynumber) except for the information on uncles since uncles are not supported in the Aion blockchain. In each code snippet, the block information is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify number  
long number = 247726;  

// get main chain block with given number  
Block block = api.getChain().getBlockByNumber(number).getObject();  

// print block information  
System.out.format("%nblock with transaction hashes:%n%s%n", block.toString());  

// get main chain block with given number  
// with full transaction information  
BlockDetails details = api.getAdmin().getBlockDetailsByNumber(number).getObject();  

// print block information  
System.out.format("%nblock with transaction information:%n%s%n", details.toString());
```

* Sample output:

```
block with transaction hashes:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions: ,
[
  0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80
],
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674


block with transaction information:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions:
[
  nrgPrice: 10000000000,
  nrg: 84107,
  transactionIndex: 0,
  nonce: 367,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 84047,
  transactionIndex: 1,
  nonce: 368,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320982,
  transactionIndex: 2,
  nonce: 446,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1ed,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25b,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x91ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db,
                  0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76
                ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320922,
  transactionIndex: 3,
  nonce: 447,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dad,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201a,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x9ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa,
                  0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe
                ]
  ]
]
,
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify number
let number = 247726;

// get main chain block with given number
let block = web3.eth.getBlock(number, false);

// print block information
console.log("\nblock with transaction hashes:");
console.log(block);

// get main chain block with given number
// with full transaction information
block = web3.eth.getBlock(number, true);

// print block information
console.log("\nblock with transaction information:");
console.log(block);
```

* Sample output:

```
block with transaction hashes:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
     '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
     '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
     '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80' ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }

block with transaction information:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 0,
       nonce: 367,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 1,
       nonce: 368,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 2,
       nonce: 446,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 3,
       nonce: 447,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 } ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }
```

</details>

#### <a name="tx-hash"></a>Retrieve Block Transaction Count by Hash

The examples below show how to query the APIs for the number of transactions sealed in a block given the hash for the block of interest. The functionality is compatible with [`eth_getBlockTransactionCountByHash`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getblocktransactioncountbyhash). In each code snippet, the transaction count is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify hash  
Hash256 hash = Hash256.wrap("0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab");  

// get tx count given hash  
int txCount = api.getChain().getBlockTransactionCountByHash(hash).getObject();  

// print information  
System.out.format("%d transactions in block %s%n", txCount, hash.toString());
```

* Sample output:

```
4 transactions in block 50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify hash
let hash = '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab';

// get tx count given hash
let txCount = web3.eth.getBlockTransactionCount(hash);

// print information
console.log(txCount + " transactions in block " + hash);
```

* Sample output:

```
4 transactions in block 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab
```

</details>

#### <a name="tx-number"></a>Retrieve Block Transaction Count by Number

The examples below show how to query the APIs for the number of transactions sealed in a main chain block given the number for the block of interest. The functionality is compatible with [`eth_getBlockTransactionCountByNumber`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getblocktransactioncountbynumber). In each code snippet, the transaction count is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify number  
long number = 247726;  

// get tx count given block number  
int txCount = api.getChain().getBlockTransactionCountByNumber(number).getObject();  

// print information  
System.out.format("%d transactions in block #%s%n", txCount, number);
```

* Sample output:

```
4 transactions in block #247726
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify number
let number = 247726;

// get tx count given block number
let txCount = web3.eth.getBlockTransactionCount(number);

// print information
console.log(txCount + " transactions in block #" + number);
```

* Sample output:

```
4 transactions in block #247726
```

</details>

#### <a name="blk-example"></a>Complete Examples

Each code example below retrieves and prints to the standard output the following:
1. the recent block number,
2. the block information given its hash,
3. the block information given its number,
4. the number of transactions in a block given its hash, and
5. the number of transactions in a block given its number.

<details>
<summary><i>Java Code</i></summary>
<br/>

```java
package org.aion.tutorials;

import org.aion.api.IAionAPI;
import org.aion.api.type.Block;
import org.aion.api.type.BlockDetails;
import org.aion.base.type.Hash256;

public class BlockExample {

    public static void main(String[] args) {

        // connect to Java API
        IAionAPI api = IAionAPI.init();
        api.connect(IAionAPI.LOCALHOST_URL);

        // 1. eth_blockNumber

        // get block number from API
        long blockNumber = api.getChain().blockNumber().getObject();

        // print block number
        System.out.format("current block = %d%n", blockNumber);

        // 2. eth_getBlockByHash

        // specify hash
        Hash256 hash = Hash256.wrap("0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab");

        // get block with given hash
        Block block = api.getChain().getBlockByHash(hash).getObject();

        // print block information
        System.out.format("%nblock with transaction hashes:%n%s%n", block.toString());

        // get block details with given hash
        // with full transaction information
        BlockDetails details = api.getAdmin().getBlockDetailsByHash(hash).getObject();

        // print block information
        System.out.format("%nblock with transaction information:%n%s%n", details.toString());

        // 3. eth_getBlockByNumber

        // specify number
        long number = 247726;

        // get main chain block with given number
        block = api.getChain().getBlockByNumber(number).getObject();

        // print block information
        System.out.format("%nblock with transaction hashes:%n%s%n", block.toString());

        // get main chain block with given number
        // with full transaction information
        details = api.getAdmin().getBlockDetailsByNumber(number).getObject();

        // print block information
        System.out.format("%nblock with transaction information:%n%s%n", details.toString());

        // 4. eth_getBlockTransactionCountByHash

        // get tx count given hash
        int txCount = api.getChain().getBlockTransactionCountByHash(hash).getObject();

        // print information
        System.out.format("%n%d transactions in block %s%n", txCount, hash.toString());

        // 5. eth_getBlockTransactionCountByNumber

        // get tx count given block number
        txCount = api.getChain().getBlockTransactionCountByNumber(number).getObject();

        // print information
        System.out.format("%n%d transactions in block #%s%n", txCount, number);

        // disconnect from api
        api.destroyApi();

        System.exit(0);
    }
}
```

* Sample output:

```
current block = 245973

block with transaction hashes:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions: ,
[
  0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80
],
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674


block with transaction information:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions:
[
  nrgPrice: 10000000000,
  nrg: 84107,
  transactionIndex: 0,
  nonce: 367,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 84047,
  transactionIndex: 1,
  nonce: 368,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320982,
  transactionIndex: 2,
  nonce: 446,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1ed,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25b,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x91ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db,
                  0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76
                ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320922,
  transactionIndex: 3,
  nonce: 447,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dad,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201a,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x9ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa,
                  0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe
                ]
  ]
]
,
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674


block with transaction hashes:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions: ,
[
  0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80
],
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674


block with transaction information:
logsBloom: 0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480,
totalDifficulty: 250890205,
receiptsRoot: 0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0,
extraData: 0x41494f4e00000000000000000000000000000000000000000000000000000000,
nrgUsed: 810058,
transactions:
[
  nrgPrice: 10000000000,
  nrg: 84107,
  transactionIndex: 0,
  nonce: 367,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 84047,
  transactionIndex: 1,
  nonce: 368,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000004,
      0x0x0000000000000000000000000000000000000000000000000000000000000000
    ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320982,
  transactionIndex: 2,
  nonce: 446,
  input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1ed,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25b,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x91ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db,
                  0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76
                ]
  ]
],
[
  nrgPrice: 10000000000,
  nrg: 320922,
  transactionIndex: 3,
  nonce: 447,
  input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
  from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
  to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  value: 0,
  hash: 0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80,
  timestamp: 0,
  error: ,
  log:
  [
      address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
    data: 0x,
    topics:
    [
      0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
      0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe,
      0x0x0000000000000000000000000000000000000000000000000000000000000006,
      0x0x0000000000000000000000000000000000000000000000000000000000000001
    ]
  ],
  [
        address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
      data: 0x,
      topics:
      [
        0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
        0x0xa082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429,
        0x0x00000000000000000000000000000000000000000000000000000002540be400
      ]
  ],
  [
          address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
        data: 0x,
        topics:
        [
          0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
          0x0xa0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50,
          0x0x00000000000000000000000000000000000000000000000000000002540be400
        ]
  ],
  [
            address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
          data: 0x,
          topics:
          [
            0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
            0x0xa044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dad,
            0x0x00000000000000000000000000000000000000000000000000000002540be400
          ]
  ],
  [
              address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
            data: 0x,
            topics:
            [
              0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
              0x0xa03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201a,
              0x0x00000000000000000000000000000000000000000000000000000002540be400
            ]
  ],
  [
                address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
              data: 0x,
              topics:
              [
                0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
                0x0xa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d,
                0x0x00000000000000000000000000000000000000000000000000000002540be400
              ]
  ],
  [
                  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
                data: 0x,
                topics:
                [
                  0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
                  0x0x9ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa,
                  0x0x6c075256f5c821d5b2398b3e29f46829792e15c87ed06634ca9b931c1c9d3abe
                ]
  ]
]
,
nonce: 0x49556905044475223817598209627374924441515029559814195533311904064399585837056,
miner: 0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b,
difficulty: 1012,
number: 247726,
nrgLimit: 15002969,
solution: 0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3,
size: 3957,
transactionsRoot: 0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c,
stateRoot: 0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1,
parentHash: 0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27,
hash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
timeStamp: 1529590674


4 transactions in block 50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab

4 transactions in block #247726
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

```js
const Web3 = require('/path/to/aion/web3');
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

// 1. eth_blockNumber

// get block number from API
let blockNumber = web3.eth.blockNumber;

// print block number
console.log("current block = " + blockNumber);

// 2. eth_getBlockByHash

// specify hash
let hash = '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab';

// get block with given hash
let block = web3.eth.getBlock(hash, false);

// print block information
console.log("\nblock with transaction hashes:");
console.log(block);

// get block with given hash
// with full transaction information
block = web3.eth.getBlock(hash, true);

// print block information
console.log("\nblock with transaction information:");
console.log(block);

// 3. eth_getBlockByNumber

// specify number
let number = 247726;

// get main chain block with given number
block = web3.eth.getBlock(number, false);

// print block information
console.log("\nblock with transaction hashes:");
console.log(block);

// get main chain block with given number
// with full transaction information
block = web3.eth.getBlock(number, true);

// print block information
console.log("\nblock with transaction information:");
console.log(block);

// 4. eth_getBlockTransactionCountByHash

// get tx count given hash
let txCount = web3.eth.getBlockTransactionCount(hash);

// print information
console.log("\n" + txCount + " transactions in block " + hash);

// 5. eth_getBlockTransactionCountByNumber

// get tx count given block number
txCount = web3.eth.getBlockTransactionCount(number);

// print information
console.log("\n" + txCount + " transactions in block #" + number);
```

* Sample output:

```
current block = 249189

block with transaction hashes:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
     '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
     '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
     '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80' ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }

block with transaction information:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 0,
       nonce: 367,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 1,
       nonce: 368,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 2,
       nonce: 446,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 3,
       nonce: 447,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 } ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }

block with transaction hashes:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
     '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
     '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
     '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80' ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }

block with transaction information:
{ logsBloom: '0x00000000002002000000004001000000000000000000000000000000010000000000000004000000000400000000002000000400040100000000000000000000000000000000000400400000100000000000040000000000000800000040000800000040001040000000000018008000000000000000000000084000010008000000005000000808000000200102000000000000000008000000000000000000020000040080000000000000000010000800020026000008040000000048000000800820000000000000000401000000002000200000000002000000000000280001000000200040000000000000002200000002000000000000000010000480',
  totalDifficulty: BigNumber { s: 1, e: 8, c: [ 250890205 ] },
  receiptsRoot: '0x17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  extraData: '0x41494f4e00000000000000000000000000000000000000000000000000000000',
  nrgUsed: 810058,
  transactions:
   [ { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 0,
       nonce: 367,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 1,
       nonce: 368,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 2,
       nonce: 446,
       input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 },
     { nrgPrice: [Object],
       nrg: 2000000,
       transactionIndex: 3,
       nonce: 447,
       input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
       blockNumber: 247726,
       gas: 2000000,
       from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
       to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       value: [Object],
       hash: '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80',
       gasPrice: '0x2540be400',
       timestamp: 1529590674 } ],
  nonce: '0x6d9036da00000001f00000000000000000000000000000000000000000000000',
  miner: '0xa0b1b3f651990669c031866ec68a4debfece1d3ffb9015b2876eda2a9716160b',
  difficulty: BigNumber { s: 1, e: 3, c: [ 1012 ] },
  number: 247726,
  gasLimit: '0xe4ed59',
  gasUsed: '0xc5c4a',
  nrgLimit: 15002969,
  solution: '0x0084673100b1b79054efbb10d5abc24b511f163a1d88183cffbd419861c3a531a15dbbd29e4d97f69dec036a0a995e9975a1033c72fbfe6a10d2d7d387cf76bc9ba4349655a300d5cc89e407037d6e37f555a84e6fe43a3f09cf754dd57adf367b707426f718f2312790b925e2300ba67e28f79caec03ac6d293112a717a4ae8257bd3071797628b9475b4e0b92d004e87be33f3389e2478c4ee24b5e0e153285daa70f7b550db4b6cd449e089ef079100db330f2674dc7221ac2d195bbbae4701e01e1662960aaf3ba2ea789c8632ed6536683ea61bf5348bf915d1139ceceab6c2b9880c6f1e3e12722a403713692a22e9394e1716bc94fc69ef9f664113c5eb1f09e216f1438f11521692cc173c6ee21e002bca676aecfb82cb3493d028a061b8bb18292fbd3b607a33f26f7c4c8284be2a0c17c2ab9e1d4969ae3ef1374de1d578b98c9698347f1a25100f7fdd790e9835586131e9d63f6d95154671f4b603507e9c8bb16ee82fcec104a5994554f18414ceea2016866e11af7e7b3bbc2e2828d5f63b1cc296b4bde30f0c71df04fb67f8b96b5a732a0a6f240a14085ed22e1a267e83388c64403e7099f6a82a36cced5ac6a8fbf65f03f4b0d57c03a23194c08829e6e76fa0133d78a3773920ee1ca56775499b1ddf0d26f23e7dafa52c50fb4b6507df1abe0058c4872a9c1c28a502b93ee6f50de2e5fa11cd2f8aad17f3f626e9e34dd755b81ff7e23cf1ba71072634367f261b02f9ac240dc6977e61924f489438310cc7eeeb8c743ad168266f1b9229320c149ef3930b2f2e5e8ab04d45f40321ef415558df06911aed1ab335ed304647fd136bec483c0a266d9a568ba5b7b5f8a50a680baa4dd3600645a8f8bea12d133d219194fd3c5e2ca9296186cdce35c380e8b773ac5493f502eb4d95ad66c30db95d2f3723bdf231a89d1d578361fcf786dbf0a08914c4a1675f529967cf46505f72e5ceb99af0853c195100c8aa6e5d91e2ce15d89215d5905fc9e17c4f35f0c25535a5b6fc89c706f709a3691df6b072fa98aaaaa71818252d843899df40fb5a8941bc7e63339541249e985546addda852250d04b79093c64793fe878d66ed3b040b03dcc117eec5ca2c67e78c257d3b97299b511cedfb081e0b94d4d2e7176beb064b2d59966373a454b763bf521bd6bd1f0c5b91cbb22ea83c9581f289a609f1b801dc32ccf27e4dd783bf7d0fd75321475d752c210db4c15505b2144cc0c4598f36702d2f87c13ab5b55ff1eb559c0976a7540421b8074e7daf8ef7afb234bd22f73c96420b0ff78b69348c0fd61e9971ed8f91cee7ecad210f1c2bb548bfe30337685ccf153b5331a78a07f76e2a9dd8065f4dc30e0b1dff733e3b2c4e2f4fe765f737effb280cf30f537689d330b8480841ef33951ce499c7941b4608091238a1d4f2755571f10fc3d8a77e37bfefb5f48926ced6181e54a04b71f1054e15ba65363820e5f9b2c204ea107a9a463512b982a3276453f8258c7320bb8d313b51ea2d223739015d06cca5b2d70603dbe652733c9c11432c59724af7f7fba236312e3b973783f088afda7b28801a3ea8447b7026d87bb108ef53024d9730f779a80cfd4e9b20cd67a33f5974155b4d0c8fd412362d9b4b290a8e344392f04719a633311e9d4b32f76c5121496917a84354a1643a36a100afba701334827ed0087ecfb11eb030811e96f33c71e6ce5a539e0425581778bbc46d0970046b49c3dfc29b04b61782ae47c727e032e6c6a1261bb6e3e52a0abe3ef38f2e73b74f3f032fd40f64340d0864926c823763ba83432c978749fdcb852431248c242c23865df7423d32ea534d4d515de1a77462eefd7a16b14e889b536abb20b3c6b2c8bf7b029f1127bee6e22bc774f07078c1ac79fcb132fc8ffccfe5b275a04a1f1e15fb7bd114b9aa972e652fd611a8d3ac6ebe3808a62b50ed8a09e48d802b549aaa49c2f6da6b08a7f808c3',
  size: 3957,
  transactionsRoot: '0xec38da7cfa82cffe541ecfb6fdb0ef9f693875cc368fd90fb3ca10467d48395c',
  stateRoot: '0x25c39deb4ac0e3ae31703c55eb99077e33e2f8e150c5bdd647f53d1c07af2cd1',
  parentHash: '0x1e013016b58cb7a0fd6828308ebe5344e56276c052219f7ff65f21c576089d27',
  hash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  timestamp: 1529590674 }

4 transactions in block 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab

4 transactions in block #247726
```

</details>


### <a name="transaction"></a>Transaction Functionality

Next we illustrate the API calls for querying the following information:
* [transaction information given its hash](#tx-hash)
* [transaction information given its index and block hash](#tx-hash-idx)
* [transaction information given its index and block number](#tx-number-idx)
* [transaction receipt given its hash](#tx-receipt)
* [sign data](#sign)

and performing the following actions:
* [send transaction](#send-tx)
* [send raw transaction](#raw-tx)

The [final subsection](#tx-example) contains code illustrating all of the above interactions.

#### <a name="tx-hash"></a>Retrieve Transaction By Hash

The examples below show how to query the APIs for the detailed information regarding a transaction given its hash. The functionality is compatible with [`eth_getTransactionByHash`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionbyhash). In each code snippet, the transaction details are retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify hash
Hash256 hash = Hash256.wrap("0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80");

// get tx with given hash
Transaction tx = api.getChain().getTransactionByHash(hash).getObject();

// print tx information
System.out.format("transaction details:%n%s%n", tx);
```

* Sample output:

```
transaction details:
nrgPrice: 10000000000,
nrg: 320922,
nonce: 447,
transactionIndex: 0,
input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
blockNumber: 247726,
from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
value: 0,
hash: 0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80,
timestamp: 1529590672338000
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify hash
let hash = '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80';

// get tx with given hash
let tx = web3.eth.getTransaction(hash);

// print tx information
console.log("transaction details:");
console.log(tx);
```

* Sample output:

```
transaction details:
{ nrgPrice: BigNumber { s: 1, e: 10, c: [ 10000000000 ] },
  blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrg: 2000000,
  transactionIndex: 3,
  nonce: 447,
  input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
  blockNumber: 247726,
  gas: 2000000,
  from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  value: BigNumber { s: 1, e: 0, c: [ 0 ] },
  hash: '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80',
  gasPrice: '0x2540be400',
  timestamp: 1529590674 }
```

</details>

#### <a name="tx-hash-idx"></a>Retrieve Transaction By Block Hash and Index

The examples below show how to query the APIs for the detailed information regarding a transaction given its index and the block hash where it occurred. The functionality is compatible with [`eth_getTransactionByBlockHashAndIndex`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionbyblockhashandindex). In each code snippet, the transaction details are retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify block hash
Hash256 hash = Hash256.wrap("0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab");
// specify tx index = 0 -> first tx
int index = 0;

// get tx with given hash & index
Transaction tx = api.getChain().getTransactionByBlockHashAndIndex(hash, index).getObject();

// print tx information
System.out.format("transaction details:%n%s%n", tx);
```

* Sample output:

```
transaction details:
nrgPrice: 10000000000,
nrg: 84107,
nonce: 367,
transactionIndex: 0,
input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
blockNumber: 247726,
from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
value: 0,
hash: 0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
timestamp: 1529590657509000
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify block hash
let hash = '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab';
// specify tx index = 0 -> first tx
let index = 0;

// get tx with given hash & index
let tx = web3.eth.getTransactionFromBlock(hash, index);

// print tx information
console.log("transaction details:");
console.log(tx);
```

* Sample output:

```
transaction details:
{ nrgPrice: BigNumber { s: 1, e: 10, c: [ 10000000000 ] },
  blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrg: 2000000,
  transactionIndex: 0,
  nonce: 367,
  input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
  blockNumber: 247726,
  gas: 2000000,
  from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  value: BigNumber { s: 1, e: 0, c: [ 0 ] },
  hash: '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
  gasPrice: '0x2540be400',
  timestamp: 1529590674 }
```

</details>

#### <a name="tx-number-idx"></a>Retrieve Transaction By Block Number and Index

The examples below show how to query the APIs for the detailed information regarding a transaction given its index and the block number where it occurred. The functionality is compatible with [`eth_getTransactionByBlockNumberAndIndex`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionbyblocknumberandindex). In each code snippet, the transaction details are retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify block number
long number = 247726L;
// specify tx index = 1 -> second tx
int index = 1;

// get tx with given number & index
Transaction tx = api.getChain().getTransactionByBlockNumberAndIndex(number, index).getObject();

// print tx information
System.out.format("transaction details:%n%s%n", tx);
```

* Sample output:

```
transaction details:
nrgPrice: 10000000000,
nrg: 84047,
nonce: 368,
transactionIndex: 0,
input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
blockNumber: 247726,
from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
value: 0,
hash: 0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
timestamp: 1529590672530000
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify block number
let number = 247726;
// specify tx index = 1 -> second tx
let index = 1;

// get tx with given number & index
let tx = web3.eth.getTransactionFromBlock(number, index);

// print tx information
console.log("transaction details:");
console.log(tx);
```

* Sample output:

```
transaction details:
{ nrgPrice: BigNumber { s: 1, e: 10, c: [ 10000000000 ] },
  blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrg: 2000000,
  transactionIndex: 1,
  nonce: 368,
  input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
  blockNumber: 247726,
  gas: 2000000,
  from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  value: BigNumber { s: 1, e: 0, c: [ 0 ] },
  hash: '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
  gasPrice: '0x2540be400',
  timestamp: 1529590674 }
```

</details>

#### <a name="tx-receipt"></a>Retrieve Transaction Receipt

The examples below show how to query the APIs for the receipt for a transaction given its hash. The functionality is compatible with [`eth_getTransactionReceipt`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionreceipt). In each code snippet, the transaction receipt is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify tx hash
Hash256 hash = Hash256.wrap("0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f");

// get receipt for given tx hash
TxReceipt txReceipt = api.getTx().getTxReceipt(hash).getObject();

// print tx receipt
System.out.format("transaction receipt:%n%s%n", txReceipt);
```

* Sample output:

```
transaction receipt:
txIndex: 2,
blockNumber: 247726,
nrg: 320982,
nrgCumulativeUsed: 489136,
blockHash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
txHash: 0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
contractAddress: ,
log:
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
    0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
    0x0x0000000000000000000000000000000000000000000000000000000000000006,
    0x0x0000000000000000000000000000000000000000000000000000000000000001
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1ed,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25b,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
    0x0x91ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db,
    0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76
  ]
]
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify tx hash
let hash = '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f';

// get receipt for given tx hash
let txReceipt = web3.eth.getTransactionReceipt(hash);

// print tx receipt
console.log("transaction receipt:");
console.log(txReceipt);
```

* Sample output:

```
transaction receipt:
{ blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000002000000000004000000000000000000000000000000000010000000000000000000000000000000000002000000400040000000000000000000000000000000000000400400000000000000000000000000000000800000040000800000000001040000000000008000000000000000000000000084000010008000000005000000008000000200102000000000000000008000000000000000000020000040080000000000000000000000800000006000008000000000008000000000020000000000000000400000000002000200000000000000000000000080001000000200040000000000000000000000000000000000000000000000400',
  nrgUsed: 320982,
  contractAddress: null,
  transactionIndex: 2,
  transactionHash: '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
  gasLimit: '0x1e8480',
  cumulativeNrgUsed: 489136,
  gasUsed: '0x04e5d6',
  blockNumber: 247726,
  root: '17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  cumulativeGasUsed: '0x776b0',
  from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  logs:
   [ { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 0,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 1,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 2,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 3,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 4,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 5,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 6,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 } ],
  gasPrice: '0x02540be400',
  status: '0x1' }
```

</details>

#### <a name="send-tx"></a>Perform Transactions

The examples below show how to use the APIs to send a transaction. The functionality is compatible with [`eth_sendTransaction`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sendtransaction). In each code snippet, the transaction receipt is retrieved from the API and printed to the standard output. Other possible transaction parameters are energy price, energy limit, and data.

Note that these examples only consider best case scenarios where the account can be unlocked, the transaction execution does not produce any errors and the transaction is eventually included in a block. A separate tutorial will be provided with the recommended sanity checks to ensure the sent transaction was included in the main chain.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// specify accounts and amount
Address sender = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");
Address receiver = Address.wrap("a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e");
BigInteger amount = BigInteger.valueOf(1_000_000_000_000_000_000L); // = 1 AION

// unlock sender
boolean isUnlocked = api.getWallet().unlockAccount(sender, "password", 100).getObject();
System.out.format("sender account %s%n", isUnlocked ? "unlocked" : "locked");

// create transaction
TxArgs.TxArgsBuilder builder = new TxArgs.TxArgsBuilder().from(sender).to(receiver).value(amount);

// perform transaction
Hash256 txHash = ((MsgRsp) api.getTx().sendTransaction(builder.createTxArgs()).getObject()).getTxHash();
System.out.format("%ntransaction hash: %s%n", txHash);

// print receipt
TxReceipt txReceipt = api.getTx().getTxReceipt(txHash).getObject();
// repeat till tx processed
while (txReceipt == null) {
    // wait 10 sec
    sleep(10000);
    txReceipt = api.getTx().getTxReceipt(txHash).getObject();
}
System.out.format("%ntransaction receipt:%n%s%n", txReceipt);
```

* Sample output:

```
sender account unlocked

transaction hash: 16b43e02eec679db7adccb1587183c6012879bb3ab753ecd324e6eab0c2b802a

transaction receipt:
txIndex: 0,
blockNumber: 306854,
nrg: 21000,
nrgCumulativeUsed: 21000,
blockHash: 0xf00500c28fdf2027b0151a0a75ca667d6f9b91a2a60a8c472ce6a94a256fc688,
txHash: 0x16b43e02eec679db7adccb1587183c6012879bb3ab753ecd324e6eab0c2b802a,
from: 0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5,
to: 0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e,
contractAddress: ,
log:
```
</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify accounts and amount
let sender = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';
let receiver = 'a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e';
let amount = 1000000000000000000; // = 1 AION

// unlock sender
let isUnlocked = web3.personal.unlockAccount(sender, "password", 100)
let status = isUnlocked ? "unlocked" : "locked";
console.log("sender account " + status);

// perform transaction
let txHash = web3.eth.sendTransaction({from: sender, to: receiver, value: amount});
console.log("\ntransaction hash: " + txHash);

// print receipt
let txReceipt = web3.eth.getTransactionReceipt(txHash);
// repeat till tx processed
while (txReceipt == null) {
  // wait 10 sec
  sleep(10000);
  txReceipt = web3.eth.getTransactionReceipt(txHash);
}
console.log("\ntransaction receipt:");
console.log(txReceipt);

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

* Sample output:

```
sender account unlocked

transaction hash: 0x652109fcfdce09008df6a805c0dfb6294d0a1bb004268675c1402cc2819f01ea

transaction receipt:
{ blockHash: '0x863811cc0169e477375bafb2c223fb5e14f3b2965fe328143678dcb764b16298',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 21000,
  contractAddress: null,
  transactionIndex: 0,
  transactionHash: '0x652109fcfdce09008df6a805c0dfb6294d0a1bb004268675c1402cc2819f01ea',
  gasLimit: '0x07a120',
  cumulativeNrgUsed: 21000,
  gasUsed: '0x5208',
  blockNumber: 257956,
  root: '3267660f820bef9d7848a9324316be6e1db003165c8512c138ffa38a4d8fd374',
  cumulativeGasUsed: '0x5208',
  from: '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  to: '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  logs: [],
  gasPrice: '0x02540be400',
  status: '0x1' }
```

</details>


#### <a name="sign"></a>Perform signature from the given data

The examples below show how to use the APIs to use the user account to sign the given data and get the signature. The functionality is compatible with [`eth_sign`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sign). In each code snippet, the signature of the signed data is retrieved from the API and printed to the standard output.

Note that these examples only consider best case scenarios where the account can be unlocked.

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// specify accounts and amount
let signer = 'a0290daf95c1ba93e1930a8ec6a82f1ca8f52ab2e0f3b2ef3f34ee90ae033504';
let data = '0x9dd2c369a187b4e6b9c402f030e50743e619301ea62aa4c0737d4ef7e10a3d49';//web3.sha3("xyz")

// unlock signer
let isUnlocked = web3.personal.unlockAccount(signer, "password", 100)
let status = isUnlocked ? "unlocked" : "locked";
console.log("signer account " + status);

// perform transaction
let signature = web3.eth.sign(signer, data);
console.log("\signature of the data: " + signature);
```

* Sample output:

```
signer account unlocked
signature of the data: 0x5354a32ccf6e4cfc990f03681119454d431fd0440c30836f5201d05c875a81c5ad178d2b5f52e13e8d2ab91876c410a5abff3d21ea58ac992e38ecda86cdc607
```

</details>

#### <a name="tx-example"></a>Complete Examples

Each code example below retrieves and prints to the standard output the following:
1. the transaction details given its hash,
2. the transaction details for a given block hash and transaction index,
3. the transaction details for a given block number and transaction index,
4. the receipt for a given transaction hash, and
5. the transaction hash and receipt for a newly performed transaction.

<details>
<summary><i>Java Code</i></summary>
<br/>

```java
package org.aion.tutorials;

import org.aion.api.IAionAPI;
import org.aion.api.type.*;
import org.aion.base.type.Address;
import org.aion.base.type.Hash256;

import java.math.BigInteger;

import static java.lang.Thread.sleep;

public class TransactionExample {

    public static void main(String[] args) throws InterruptedException {

        // connect to Java API
        IAionAPI api = IAionAPI.init();
        ApiMsg apiMsg = api.connect(IAionAPI.LOCALHOST_URL);

        // failed connection
        if (apiMsg.isError()) {
            System.out.format("Could not connect due to <%s>%n", apiMsg.getErrString());
            System.exit(-1);
        }

        // 1. eth_getTransactionByHash

        // specify hash
        Hash256 hash = Hash256.wrap("0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80");

        // get tx with given hash
        Transaction tx = api.getChain().getTransactionByHash(hash).getObject();

        // print tx information
        System.out.format("transaction details:%n%s%n", tx);

        // 2. eth_getTransactionByBlockHashAndIndex

        // specify block hash
        hash = Hash256.wrap("0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab");
        // specify tx index = 0 -> first tx
        int index = 0;

        // get tx with given hash & index
        tx = api.getChain().getTransactionByBlockHashAndIndex(hash, index).getObject();

        // print tx information
        System.out.format("transaction details:%n%s%n", tx);

        // 3. eth_getTransactionByBlockNumberAndIndex

        // specify block number
        long number = 247726L;
        // specify tx index = 1 -> second tx
        index = 1;

        // get tx with given number & index
        tx = api.getChain().getTransactionByBlockNumberAndIndex(number, index).getObject();

        // print tx information
        System.out.format("transaction details:%n%s%n", tx);

        // 4. eth_getTransactionReceipt

        // specify tx hash
        hash = Hash256.wrap("0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f");

        // get receipt for given tx hash
        TxReceipt txReceipt = api.getTx().getTxReceipt(hash).getObject();

        // print tx receipt
        System.out.format("transaction receipt:%n%s%n", txReceipt);

        // 5. eth_sendTransaction

        // specify accounts and amount
        Address sender = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");
        Address receiver = Address.wrap("a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e");
        BigInteger amount = BigInteger.valueOf(1_000_000_000_000_000_000L); // = 1 AION

        // unlock sender
        boolean isUnlocked = api.getWallet().unlockAccount(sender, "password", 100).getObject();
        System.out.format("sender account %s%n", isUnlocked ? "unlocked" : "locked");

        // create transaction
        TxArgs.TxArgsBuilder builder = new TxArgs.TxArgsBuilder().from(sender).to(receiver).value(amount);

        // perform transaction
        Hash256 txHash = ((MsgRsp) api.getTx().sendTransaction(builder.createTxArgs()).getObject()).getTxHash();
        System.out.format("%ntransaction hash: %s%n", txHash);

        // print receipt
        txReceipt = api.getTx().getTxReceipt(txHash).getObject();
        // repeat till tx processed
        while (txReceipt == null) {
            // wait 10 sec
            sleep(10000);
            txReceipt = api.getTx().getTxReceipt(txHash).getObject();
        }
        System.out.format("%ntransaction receipt:%n%s%n", txReceipt);

        // disconnect from api
        api.destroyApi();

        System.exit(0);
    }
}
```

* Sample output:

```
transaction details:
nrgPrice: 10000000000,
nrg: 320922,
nonce: 447,
transactionIndex: 0,
input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
blockNumber: 247726,
from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
value: 0,
hash: 0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80,
timestamp: 1529590672338000

transaction details:
nrgPrice: 10000000000,
nrg: 84107,
nonce: 367,
transactionIndex: 0,
input: 0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
blockNumber: 247726,
from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
value: 0,
hash: 0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045,
timestamp: 1529590657509000

transaction details:
nrgPrice: 10000000000,
nrg: 84047,
nonce: 368,
transactionIndex: 0,
input: 0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400,
blockNumber: 247726,
from: 0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
value: 0,
hash: 0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0,
timestamp: 1529590672530000

transaction receipt:
txIndex: 2,
blockNumber: 247726,
nrg: 320982,
nrgCumulativeUsed: 489136,
blockHash: 0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab,
txHash: 0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f,
from: 0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c,
to: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
contractAddress: ,
log:
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xfd24c246254d45bcfec939758f58d63bd1565596fdc53994ae974bb54f83bb69,
    0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76,
    0x0x0000000000000000000000000000000000000000000000000000000000000006,
    0x0x0000000000000000000000000000000000000000000000000000000000000001
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1ed,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25b,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0xdc3b8ebc415c945740a70187f1d472ad2d64a9e7a87047f38023aec56516976b,
    0x0xa0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad,
    0x0x00000000000000000000000000000000000000000000000000000002540be400
  ]
],
[
  address: 0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a,
  data: 0x,
  topics:
  [
    0x0x1fa305c7f8521af161de570532762ed7a60199cde79e18e1d259af3459562521,
    0x0x91ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db,
    0x0x7f21f0710cfd7a24883f3d41c47ad324b49e484b56d3010250110a9cd6876c76
  ]
]

sender account unlocked

transaction hash: 027412b1717f161efdec7199a4d7b697705dac72c4af6217f2bcb9ed807c30aa

transaction receipt:
txIndex: 0,
blockNumber: 306878,
nrg: 21000,
nrgCumulativeUsed: 21000,
blockHash: 0xa7d7f03626384670e7c2a161ddeceed4949f408f428d54e4e9172c9a1545e0fa,
txHash: 0x027412b1717f161efdec7199a4d7b697705dac72c4af6217f2bcb9ed807c30aa,
from: 0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5,
to: 0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e,
contractAddress: ,
log:
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

```js
const Web3 = require('/path/to/aion/web3');
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

// 1. eth_getTransactionByHash

// specify hash
let hash = '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80';

// get tx with given hash
let tx = web3.eth.getTransaction(hash);

// print tx information
console.log("\ntransaction details:");
console.log(tx);

// 2. eth_getTransactionByBlockHashAndIndex

// specify block hash
hash = '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab';
// specify tx index = 0 -> first tx
let index = 0;

// get tx with given hash & index
tx = web3.eth.getTransactionFromBlock(hash, index);

// print tx information
console.log("\ntransaction details:");
console.log(tx);

// 3. eth_getTransactionByBlockNumberAndIndex

// specify block number
let number = 247726;
// specify tx index = 1 -> second tx
index = 1;

// get tx with given number & index
tx = web3.eth.getTransactionFromBlock(number, index);

// print tx information
console.log("\ntransaction details:");
console.log(tx);

// 4. eth_getTransactionReceipt

// specify tx hash
hash = '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f';

// get receipt for given tx hash
let txReceipt = web3.eth.getTransactionReceipt(hash);

// print tx receipt
console.log("\ntransaction receipt:");
console.log(txReceipt);

// 5. eth_sendTransaction

// specify accounts and amount
let sender = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';
let receiver = 'a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e';
let amount = 1000000000000000000; // = 1 AION

// unlock sender
let isUnlocked = web3.personal.unlockAccount(sender, "password", 100)
let status = isUnlocked ? "unlocked" : "locked";
console.log("\nsender account " + status);

// perform transaction
let txHash = web3.eth.sendTransaction({from: sender, to: receiver, value: amount});
console.log("\ntransaction hash: " + txHash);

// print receipt
txReceipt = web3.eth.getTransactionReceipt(txHash);
// repeat till tx processed
while (txReceipt == null) {
  // wait 10 sec
  sleep(10000);
  txReceipt = web3.eth.getTransactionReceipt(txHash);
}
console.log("\ntransaction receipt:");
console.log(txReceipt);

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

* Sample output:

```
transaction details:
{ nrgPrice: BigNumber { s: 1, e: 10, c: [ 10000000000 ] },
  blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrg: 2000000,
  transactionIndex: 3,
  nonce: 447,
  input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
  blockNumber: 247726,
  gas: 2000000,
  from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  value: BigNumber { s: 1, e: 0, c: [ 0 ] },
  hash: '0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80',
  gasPrice: '0x2540be400',
  timestamp: 1529590674 }

transaction details:
{ nrgPrice: BigNumber { s: 1, e: 10, c: [ 10000000000 ] },
  blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrg: 2000000,
  transactionIndex: 0,
  nonce: 367,
  input: '0x1fec4cc691ae034f8976efbe9c89fa3a78c19d663bf61c76c6d3dd528259fb9cb565e8db00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a064ff11a1563ef09fd5d4671272275dd962d46f7498f8d297d60bf3e4bee1eda07e5b667332c1af7a14bb51b86fe92e1cc0d2a5f0de70613c26ff3584c312c7a0a3afd832901ba4e1bc947721babb13039194ac1b780f2b13b9b86b2ed08639a09bb6e70f50e6f512e0aa3033f2477ad04f75704bcf14ce1c9aca37492cf25ba0ce3ce58c28307850315f934cda934e65252b5115aaeca58ead64e88d9569ad00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
  blockNumber: 247726,
  gas: 2000000,
  from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  value: BigNumber { s: 1, e: 0, c: [ 0 ] },
  hash: '0xdb50c83faad497dc281df5a7ae5e2aa3294431d64a7868134895d33838882045',
  gasPrice: '0x2540be400',
  timestamp: 1529590674 }

transaction details:
{ nrgPrice: BigNumber { s: 1, e: 10, c: [ 10000000000 ] },
  blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrg: 2000000,
  transactionIndex: 1,
  nonce: 368,
  input: '0x1fec4cc69ce22f4a90d3d5ee88b4750e1c4ad0fbf9b2981ca82132742a48002f7e85e2fa00000000000000000000000000000040000000000000000000000000000000f000000000000000000000000000000005a082abceefa73078541d577ad576719c3f475b4ad0bd136918da43f0bce30429a0b612e3b6be803768451b7a331b1837face5be52b2b1fd253a31516bbdc1b50a044ba079d30fa1cdea965e93ab7cbaa9d18f8c02e569707bb320129a1840dada03cefb0ad4b6effd63825a1d6e345fc5b708f197b85c0a4f5583b125f52201aa0ee6827c6f05bd9192d444cb4fa0dec304829edf5c3c99d3df30d9618ebad1d00000000000000000000000000000005000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400000000000000000000000002540be400',
  blockNumber: 247726,
  gas: 2000000,
  from: '0xa05a3889b106e75baa621b8cc719679a3dbdd799afac1ca6b42d03dc93a23687',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  value: BigNumber { s: 1, e: 0, c: [ 0 ] },
  hash: '0x92aef634e0194a3c70a39163c45c2f2747ec86a8e40a9c6ddb93009741be8cb0',
  gasPrice: '0x2540be400',
  timestamp: 1529590674 }

transaction receipt:
{ blockHash: '0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000002000000000004000000000000000000000000000000000010000000000000000000000000000000000002000000400040000000000000000000000000000000000000400400000000000000000000000000000000800000040000800000000001040000000000008000000000000000000000000084000010008000000005000000008000000200102000000000000000008000000000000000000020000040080000000000000000000000800000006000008000000000008000000000020000000000000000400000000002000200000000000000000000000080001000000200040000000000000000000000000000000000000000000000400',
  nrgUsed: 320982,
  contractAddress: null,
  transactionIndex: 2,
  transactionHash: '0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f',
  gasLimit: '0x1e8480',
  cumulativeNrgUsed: 489136,
  gasUsed: '0x04e5d6',
  blockNumber: 247726,
  root: '17e8253caf42481001da608479fa8250557b2a76226b6f3cbc9924c6768c4bd0',
  cumulativeGasUsed: '0x776b0',
  from: '0xa0dd16394f16ea21c8b45c00b2e43850ae7e8f00fe54789ddd1881d33b21df0c',
  to: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
  logs:
   [ { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 0,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 1,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 2,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 3,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 4,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 5,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 },
     { address: '0xa0e1cca4fe786118c0abb1fdf45c04e44354f971b25c04ed77ac46f13cae179a',
       logIndex: 6,
       data: '0x',
       topics: [Array],
       blockNumber: 247726,
       transactionIndex: 2 } ],
  gasPrice: '0x02540be400',
  status: '0x1' }

sender account unlocked

transaction hash: 0x3f66966ad038c8fbc5d31722d077e79675811f6c8d0d86fd9bc9445d18c7b07c

transaction receipt:
{ blockHash: '0x76944e5c82875b34e06c88634fcff023967f1c28be7a3809b3e191c6df1ef8ce',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 21000,
  contractAddress: null,
  transactionIndex: 0,
  transactionHash: '0x3f66966ad038c8fbc5d31722d077e79675811f6c8d0d86fd9bc9445d18c7b07c',
  gasLimit: '0x1e8480',
  cumulativeNrgUsed: 21000,
  gasUsed: '0x5208',
  blockNumber: 306911,
  root: 'a3710bbf0c04c6f66bf4b268365ffe38ad78429a778f63ffdbb3fd4aa7c66d53',
  cumulativeGasUsed: '0x5208',
  from: '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  to: '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  logs: [],
  gasPrice: '0x02540be400',
  status: '0x1' }
```

</details>


### <a name="contract"></a>Contract Functionality

Next, we illustrate the API calls for querying the following information:
* [available compilers](#get-comp)
* [current energy price](#nrg-price)
* [energy estimate](#nrg-estimate)
* [contract code](#get-code)
* [contract storage](#get-storage)

and performing the following actions:
* [compile Solidity code](#comp-sol)
* [deploy a contract](#ctr-deploy)
* [execute a contract function](#ctr-execute)
* [call a contract function](#call)

The [final subsection](#ctr-example) contains code illustrating all of the above interactions.

#### <a name="get-comp"></a>Retrieve Available Compilers

The JavaScript example below shows how to query the APIs for the list of available code compilers.
The functionality is compatible with [`eth_getCompilers`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getcompilers).
Only the Solidity compiler is currently supported.

The Aion blockchain currently does not support:
* [`eth_compileLLL`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_compilelll)
* [`eth_compileSerpent`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_compileserpent)

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get list of available compilers
let compilers = web3.eth.getCompilers();

// print
console.log("available compilers:");
console.log(compilers);
```

* Sample output:

```
available compilers:
[ 'solidity' ]
```

</details>

#### <a name="comp-sol"></a>Compile Solidity Code

The examples below show how to use the APIs to compile solidity code.
The functionality is compatible with [`eth_compileSolidity`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_compilesolidity).
In each code snippet, the given contract is compiled and the output is printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// contract source code
String source_ticker = "contract ticker { uint public val; function tick () { val+= 1; } }";

// compile code
Map<String, CompileResponse> result = api.getTx().compile(source_ticker).getObject();

// print result
System.out.println(result);
```

* Sample output:

```
{ticker=code :0x605060405234156100105760006000fd5b610015565b60c2806100236000396000f30060506040526000356c01000000000000000000000000900463ffffffff1680633c6bb43614603b5780633eaf5d9f146062576035565b60006000fd5b341560465760006000fd5b604c6075565b6040518082815260100191505060405180910390f35b3415606d5760006000fd5b6073607e565b005b60006000505481565b6001600060008282825054019250508190909055505b5600a165627a7a723058201c1817d957da90680b7c55720aff16e7a4407b31f7480f63210748a5cc4efb0c0029,
source :contract ticker { uint public val; function tick () { val+= 1; } },
language :,
languageVersion: ,
compilerVersion: ,
compilerOption: ,
abiDefString: [{"outputs":[{"name":"","type":"uint128"}],"constant":true,"payable":false,"inputs":[],"name":"val","type":"function"},{"outputs":[],"constant":false,"payable":false,"inputs":[],"name":"tick","type":"function"}],
abiDefinition:
[
constant: true,
anonymous: false,
payable: false,
type: function,
name: val,
inputs:
outputs:
[
indexed: false,
type: uint128,
name: ,
paramLengths:
[
]

]
isEvent: false,
isConstructor: false,
isFallback: false,
hashed: 3c6bb436

],
[
constant: false,
anonymous: false,
payable: false,
type: function,
name: tick,
inputs:
outputs:
isEvent: false,
isConstructor: false,
isFallback: false,
hashed: 3eaf5d9f

]
userDoc: constant: false,
name: ,
type: ,
inputs:
outputs:
,
developerDocconstant: false,
name: ,
type: ,
inputs:
outputs:

}
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// contract source code
const source_ticker = 'contract ticker { uint public val; function tick () { val+= 1; } }';

// compile code
let result = web3.eth.compile.solidity(source_ticker);

// print result
console.log(result);
```

* Sample output:

```
{ ticker:
   { code: '0x605060405234156100105760006000fd5b610015565b60c2806100236000396000f30060506040526000356c01000000000000000000000000900463ffffffff1680633c6bb43614603b5780633eaf5d9f146062576035565b60006000fd5b341560465760006000fd5b604c6075565b6040518082815260100191505060405180910390f35b3415606d5760006000fd5b6073607e565b005b60006000505481565b6001600060008282825054019250508190909055505b5600a165627a7a723058201c1817d957da90680b7c55720aff16e7a4407b31f7480f63210748a5cc4efb0c0029',
     info:
      { abiDefinition: [Array],
        languageVersion: '0',
        language: 'Solidity',
        compilerVersion: '0.4.15+commit.ecf81ee5.Linux.g++',
        source: 'contract ticker { uint public val; function tick () { val+= 1; } }' } } }
```

</details>

#### <a name="nrg-price"></a>Retrieve Energy Price

The examples below show how to query the APIs for the current price per NRG in [nAmp](https://github.com/aionnetwork/aion/wiki/Aion-Terminology).
The functionality is compatible with [`eth_gasPrice`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gasprice).
In each code snippet, the NRG price is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java  
// get NRG price
long price = api.getTx().getNrgPrice().getObject();

// print price
System.out.println("current NRG price = " + price + " nAmp");
```

* Sample output:

```
current NRG price = 10000000000 nAmp
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get NRG price
let price = web3.eth.gasPrice

// print price
console.log("current NRG price = " + price + " nAmp");
```

* Sample output:

```
current NRG price = 10000000000 nAmp
```

</details>

#### <a name="nrg-estimate"></a>Estimate Energy Required

The examples below show how to use the APIs to estimate the NRG required to allow the transaction to complete.
The functionality is compatible with [`eth_estimateGas`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_estimategas).
In each code snippet, the NRG estimate is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// compile contract source code
String source_ticker = "contract ticker { uint public val; function tick () { val+= 1; } }";
Map<String, CompileResponse> result = api.getTx().compile(source_ticker).getObject();

// get NRG estimate for contract
long estimate = api.getTx().estimateNrg(result.get("ticker").getCode()).getObject();

// print estimate
System.out.println("NRG estimate for contract = " + estimate + " NRG");

// transaction data
Address sender = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");
Address receiver = Address.wrap("a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e");
BigInteger amount = BigInteger.valueOf(1_000_000_000_000_000_000L); // = 1 AION
ByteArrayWrapper data = ByteArrayWrapper.wrap("test.message".getBytes());

// prepare transaction
TxArgs tx = new TxArgs.TxArgsBuilder().data(data).from(sender).to(receiver).value(amount).createTxArgs();

// get NRG estimate for transaction
estimate = api.getTx().estimateNrg(tx).getObject();

// print estimate
System.out.println("NRG estimate for transaction = " + estimate + " NRG");
```

* Sample output:

```
NRG estimate for contract = 233661 NRG
NRG estimate for transaction = 21768 NRG
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// compile contract source code
const source_ticker = 'contract ticker { uint public val; function tick () { val+= 1; } }';
let result = web3.eth.compile.solidity(source_ticker);

// get NRG estimate for contract
let estimate = web3.eth.estimateGas({data:result.ticker.code});

// print estimate
console.log("NRG estimate for contract = " + estimate + " NRG");

// transaction data
let sender = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';
let receiver = 'a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e';
let amount = 1000000000000000000; // = 1 AION
let data = '0x746573742e6d657373616765'; // hex for "test.message"

// get NRG estimate for transaction
estimate = web3.eth.estimateGas({data: data, from: sender, to: receiver, value: amount})

// print estimate
console.log("NRG estimate for transaction = " + estimate + " NRG");
```

* Sample output:

```
NRG estimate for contract = 233661 NRG
NRG estimate for transaction = 21768 NRG
```

</details>

#### <a name="get-code"></a>Retrieve Contract Code

The examples below show how to use the APIs to retrieve the code for a contract.
The functionality is compatible with [`eth_getCode`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getcode).
In each code snippet, the contract code is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java  
// set contract account
Address contractAccount = Address.wrap("a0960fcb7d6423a0446243916c7c6360543b3d2f9c5e1c5ff7badb472b782b79");

// get code from latest block
long blockNumber = -1L; // indicates latest block
byte[] code = api.getTx().getCode(contractAccount, blockNumber).getObject();

// print code
System.out.println("0x" + IUtils.bytes2Hex(code));
```

* Sample output:

```
0x60506040526000356c01000000000000000000000000900463ffffffff1680630fd5f53f146100545780634985e85c146100e25780638da5cb5b1461015f578063ceb35b0f146101905761004e565b60006000fd5b34156100605760006000fd5b6100e06004808035906010019082018035906010019191908080600f01601080910402601001604051908101604052809392919081815260100183838082843782019150505050505090909190808060100135903590916020019091929080356effffffffffffffffffffffffffffff1916906010019091905050610203565b005b34156100ee5760006000fd5b6101426004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919050506103af565b604051808383825281601001526020019250505060405180910390f35b341561016b5760006000fd5b610173610457565b604051808383825281601001526020019250505060405180910390f35b341561019c5760006000fd5b6102016004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919080806010013590359091602001909192905050610466565b005b600060005080600101549054339091149190141615156102235760006000fd5b828260026000506000876040518082805190601001908083835b60108310151561026357805182525b60108201915060108101905060108303925061023d565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508282909180600101839055555050508060036000506000858582528160100152602001908152601001600020905060006101000a81548160ff02191690836f01000000000000000000000000000000900402179055507fdb8f9fd4bfba5ae615ae02e8b4d281b887225fd3d34a5ac7b8d78df768bb63e7856040518080601001828103825283818151815260100191508051906010019080838360005b8381101561036d5780820151818401525b601081019050610351565b50505050905090810190600f16801561039a5780820380516001836010036101000a031916815260100191505b509250505060405180910390a15b5b50505050565b6000600060026000506000846040518082805190601001908083835b6010831015156103f157805182525b6010820191506010810190506010830392506103cb565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508060010154905491509150610452565b915091565b60006000508060010154905482565b600060005080600101549054339091149190141615156104865760006000fd5b818160026000506000866040518082805190601001908083835b6010831015156104c657805182525b6010820191506010810190506010830392506104a0565b6001836010036101000a038019825116818451168082178552505050505050905001915050604051809103902090600019169090600019169082528160100152602001908152601001600020905060005082829091806001018390555550505081817fa226db3f664042183ee0281230bba26cbf7b5057e50aee7f25a175ff45ce4d7f60405160405180910390a25b5b5050505600a165627a7a723058201a36e96ba95136c7bf35a644bf8d3382c49dafc64e9362b025ed6f4eb99ed0640029
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// set contract account
let ctAcc = 'a0960fcb7d6423a0446243916c7c6360543b3d2f9c5e1c5ff7badb472b782b79';

// get code from latest block
let code = web3.eth.getCode(ctAcc, 'latest');

// print code
console.log(code);
```

* Sample output:

```
0x60506040526000356c01000000000000000000000000900463ffffffff1680630fd5f53f146100545780634985e85c146100e25780638da5cb5b1461015f578063ceb35b0f146101905761004e565b60006000fd5b34156100605760006000fd5b6100e06004808035906010019082018035906010019191908080600f01601080910402601001604051908101604052809392919081815260100183838082843782019150505050505090909190808060100135903590916020019091929080356effffffffffffffffffffffffffffff1916906010019091905050610203565b005b34156100ee5760006000fd5b6101426004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919050506103af565b604051808383825281601001526020019250505060405180910390f35b341561016b5760006000fd5b610173610457565b604051808383825281601001526020019250505060405180910390f35b341561019c5760006000fd5b6102016004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919080806010013590359091602001909192905050610466565b005b600060005080600101549054339091149190141615156102235760006000fd5b828260026000506000876040518082805190601001908083835b60108310151561026357805182525b60108201915060108101905060108303925061023d565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508282909180600101839055555050508060036000506000858582528160100152602001908152601001600020905060006101000a81548160ff02191690836f01000000000000000000000000000000900402179055507fdb8f9fd4bfba5ae615ae02e8b4d281b887225fd3d34a5ac7b8d78df768bb63e7856040518080601001828103825283818151815260100191508051906010019080838360005b8381101561036d5780820151818401525b601081019050610351565b50505050905090810190600f16801561039a5780820380516001836010036101000a031916815260100191505b509250505060405180910390a15b5b50505050565b6000600060026000506000846040518082805190601001908083835b6010831015156103f157805182525b6010820191506010810190506010830392506103cb565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508060010154905491509150610452565b915091565b60006000508060010154905482565b600060005080600101549054339091149190141615156104865760006000fd5b818160026000506000866040518082805190601001908083835b6010831015156104c657805182525b6010820191506010810190506010830392506104a0565b6001836010036101000a038019825116818451168082178552505050505050905001915050604051809103902090600019169090600019169082528160100152602001908152601001600020905060005082829091806001018390555550505081817fa226db3f664042183ee0281230bba26cbf7b5057e50aee7f25a175ff45ce4d7f60405160405180910390a25b5b5050505600a165627a7a723058201a36e96ba95136c7bf35a644bf8d3382c49dafc64e9362b025ed6f4eb99ed0640029
```

</details>

#### <a name="get-storage"></a>Retrieve Stored Value

The examples below show how to use the APIs to retrieve stored values from a contract.
The functionality is compatible with [`eth_getStorageAt`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getstorageat).
In each code snippet, the stored values are retrieved from the API and printed to the standard output.
The contract for which the values are retrieved is [personnel.sol](https://github.com/aionnetwork/aion_qa/blob/master/Web3/test/contracts/personnel.sol) which uses the first two values to store the owner address.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// view contract creation tx
Hash256 txHash = Hash256.wrap("0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48");
TxReceipt receipt = api.getTx().getTxReceipt(txHash).getObject();
System.out.println(receipt);

// set contract account
Address contractAccount = receipt.getContractAddress();

// get value from storage
long blockNumber = -1L; // code for latest
String valuePos0 = api.getChain().getStorageAt(contractAccount, 0, blockNumber).getObject();
String valuePos1 = api.getChain().getStorageAt(contractAccount, 1, blockNumber).getObject();

// print values
// in this case the first two values are the contract owner
System.out.println("concatenated values = " + valuePos0 + valuePos1);
```

* Sample output:

```
txIndex: 0,
blockNumber: 298287,
nrg: 349674,
nrgCumulativeUsed: 349674,
blockHash: 0x2e9b90a21f9702fde90807e42db0107ce85b8c14a99c1f9f76182ec20599eebc,
txHash: 0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48,
from: 0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e,
to: 0x,
contractAddress: a076ddb4cf37f7cd1360de9cf2336f75f06c38655da09d4e3f9b690acf57c2e1,
log:

concatenated values = a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// view contract creation tx
let txHash = '0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48';
let receipt = web3.eth.getTransactionReceipt(txHash);
console.log(receipt);

// set contract account
let contractAccount = receipt.contractAddress;

// get value from storage
let valuePos0 = web3.eth.getStorageAt(contractAccount, 0, 'latest');
let valuePos1 = web3.eth.getStorageAt(contractAccount, 1, 'latest');

// print values
// in this case the first two values are the contract owner
console.log("\nconcatenated values = " + valuePos0 + valuePos1);
```

* Sample output:

```
{ blockHash: '0x2e9b90a21f9702fde90807e42db0107ce85b8c14a99c1f9f76182ec20599eebc',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 349674,
  contractAddress: '0xa076ddb4cf37f7cd1360de9cf2336f75f06c38655da09d4e3f9b690acf57c2e1',
  transactionIndex: 0,
  transactionHash: '0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48',
  gasLimit: '0x4c4b40',
  cumulativeNrgUsed: 349674,
  gasUsed: '0x0555ea',
  blockNumber: 298287,
  root: 'e433bbbc1714c2301faf7ac447c5881f345c32f8853e121694e430219cacb9d5',
  cumulativeGasUsed: '0x555ea',
  from: '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  to: '0x',
  logs: [],
  gasPrice: '0x02540be400',
  status: '0x1' }

concatenated values = 0xa0bd0ef93902d9e123521a67bef7391e0x9487e963b2346ef3b3ff78208835545e
```

</details>

#### <a name="ctr-deploy"></a>Deploy Contract

The examples below show how to use the APIs to deploy a contract.
In each code snippet, after deploying the contract the resulting account address, transaction hash and receipt are printed to the standard output.

Note that these examples only consider best case scenarios where the account can be unlocked, the transaction execution does not produce any errors and the transaction is eventually included in a block. A separate tutorial will be provided with the recommended sanity checks to ensure the deployed contract is included in the main chain.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// contract source code
String source_personnel =
        "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";

// compile code
Map<String, CompileResponse> result = api.getTx().compile(source_personnel).getObject();
CompileResponse contract = result.get("Personnel");

// unlock owner
Address owner = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");
boolean isUnlocked = api.getWallet().unlockAccount(owner, "password", 100).getObject();
System.out.format("owner account %s%n", isUnlocked ? "unlocked" : "locked");

// deploy contract
ContractDeploy.ContractDeployBuilder builder = new ContractDeploy.ContractDeployBuilder()
        .compileResponse(contract).value(BigInteger.ZERO).nrgPrice(NRG_PRICE_MIN)
        .nrgLimit(NRG_LIMIT_CONTRACT_CREATE_MAX).from(owner).data(ByteArrayWrapper.wrap(Bytesable.NULL_BYTE));

DeployResponse contractResponse = api.getTx().contractDeploy(builder.createContractDeploy()).getObject();

// print response
Hash256 txHash = contractResponse.getTxid();
Address contractAccount = contractResponse.getAddress();
System.out.format("%ntransaction hash:%n\t%s%ncontract address: %n\t%s%n", txHash, contractAccount);

// get & print receipt
TxReceipt txReceipt = api.getTx().getTxReceipt(txHash).getObject();
System.out.format("%ntransaction receipt:%n%s%n", txReceipt);
```

Alternative deploy using contract controller:

```java
// contract source code
String source_personnel =
        "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";

// unlock owner
Address owner = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");
boolean isUnlocked = api.getWallet().unlockAccount(owner, "password", 100).getObject();
System.out.format("owner account %s%n", isUnlocked ? "unlocked" : "locked");

// clear old deploy
api.getContractController().clear();

// deploy contract
ApiMsg msg = api.getContractController()
        .createFromSource(source_personnel, owner, NRG_LIMIT_CONTRACT_CREATE_MAX, NRG_PRICE_MIN);

if (msg.isError()) {
    System.out.println("deploy contract failed! " + msg.getErrString());
} else {
    // get contract
    IContract contractResponse = api.getContractController().getContract();

    // print response
    Hash256 txHash = contractResponse.getDeployTxId();
    Address contractAccount = contractResponse.getContractAddress();
    System.out.format("%ntransaction hash:%n\t%s%ncontract address: %n\t%s%n", txHash, contractAccount);

    // get & print receipt
    TxReceipt txReceipt = api.getTx().getTxReceipt(txHash).getObject();
    System.out.format("%ntransaction receipt:%n%s%n", txReceipt);
}
```

* Sample output:

```
owner account unlocked

transaction hash:
	226b57f1b342892718e3776c6d9e53256e26b53714e291c4aff892b5ab9c259b
contract address:
	a0c751cd789de21265f16d1c2cf6e9c5bf56fd1cbeb30704fd2b017e4ef65e8f

transaction receipt:
txIndex: 0,
blockNumber: 307647,
nrg: 349674,
nrgCumulativeUsed: 349674,
blockHash: 0x8882810ca8481b5532055c00beb44a121e0ca47c77fca099f4aa204649024282,
txHash: 0x226b57f1b342892718e3776c6d9e53256e26b53714e291c4aff892b5ab9c259b,
from: 0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5,
to: 0x,
contractAddress: a0c751cd789de21265f16d1c2cf6e9c5bf56fd1cbeb30704fd2b017e4ef65e8f,
log:
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// contract source code
const source_personnel = "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";

// compile code
let result = web3.eth.compile.solidity(source_personnel);
let abi = result.Personnel.info.abiDefinition;
let code = result.Personnel.code;

// unlock owner
let owner = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';
let isUnlocked = web3.personal.unlockAccount(owner, "password", 100);
console.log("owner account " + (isUnlocked ? "unlocked" : "locked"));

// deploy contract
let response = web3.eth.contract(abi).new({from: owner, data: code, gasPrice: 10000000000, gas: 5000000});

// print response
let txHash = response.transactionHash;
let contractAccount = response.address;
// note that the address is not defined in the response
console.log("\ntransaction hash:\n\t" + txHash + "\ncontract address:\n\t" + contractAccount);

// get & print receipt
txReceipt = web3.eth.getTransactionReceipt(txHash);
// repeat till tx processed
while (txReceipt == null) {
  // wait 10 sec
  sleep(10000);
  txReceipt = web3.eth.getTransactionReceipt(txHash);
}
// getting the address from the receipt
contractAccount = txReceipt.contractAddress;
console.log("\ncontract address:\n\t" + contractAccount);
// print full receipt
console.log("\ntransaction receipt:");
console.log(txReceipt);

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

* Sample output:

```
owner account unlocked

transaction hash:
	0x1957290572a6785f550adcaaeedccc6c71c78ba60f7e236b23ae8ae0ccb798a1
contract address:
	undefined

contract address:
	0xa0effc1986a633667d32cccc3402457a6badfed493dcb545c11ef5fd3decdc0e

transaction receipt:
{ blockHash: '0x574b8ffef09084283aa3febfca060ba33724f5d8cd028877941e56dff782c318',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 349674,
  contractAddress: '0xa0effc1986a633667d32cccc3402457a6badfed493dcb545c11ef5fd3decdc0e',
  transactionIndex: 0,
  transactionHash: '0x1957290572a6785f550adcaaeedccc6c71c78ba60f7e236b23ae8ae0ccb798a1',
  gasLimit: '0x4c4b40',
  cumulativeNrgUsed: 349674,
  gasUsed: '0x0555ea',
  blockNumber: 313456,
  root: '0bb58744440f2cd6951081956dfbe0e447dec5e34b80324ff1ccce0950242b94',
  cumulativeGasUsed: '0x555ea',
  from: '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  to: '0x',
  logs: [],
  gasPrice: '0x02540be400',
  status: '0x1' }
```

</details>

#### <a name="ctr-execute"></a>Execute Contract Function

The examples below show how to use the APIs to execute a contract function.
In each code snippet, a function to add information to the contract is executed after which we use a get function to verify the value was added.
The transaction hashes for both calls are printed to the standard output.
Note that the get call does not create a transaction.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// input values
Hash256 txHash = Hash256.wrap("0xb35c28a10bc996f1cdd81425e6c90d4c841ed6ba6c7f039e76d448a6c869d7bc");
String source_personnel =
        "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";
String addressToAdd = "a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab";
String keyToAdd = "key-ab123";

// get contract object parameters
TxReceipt txReceipt = api.getTx().getTxReceipt(txHash).getObject();
Address contractAccount = txReceipt.getContractAddress();
Address ownerAddress = txReceipt.getFrom();
String abiDefinition = ((Map<String, CompileResponse>) api.getTx().compile(source_personnel).getObject())
        .get("Personnel").getAbiDefString();

// get contract object using ownerAddress & contractAccount & abiDefinition
IContract ctr = api.getContractController().getContractAt(ownerAddress, contractAccount, abiDefinition);

// unlock account
api.getWallet().unlockAccount(ownerAddress, "password");

// execute function: adding a user address
ContractResponse rsp = ctr.newFunction("addAddress").setFrom(ownerAddress).setParam(ISString.copyFrom(keyToAdd))
        .setParam(IAddress.copyFrom(addressToAdd)).setTxNrgLimit(NRG_LIMIT_TX_MAX).setTxNrgPrice(NRG_PRICE_MIN)
        .build().execute().getObject();

System.out.println("ADD response:\n" + rsp);

// wait for tx to be processed
Thread.sleep(30000L);
// get & print receipt
txReceipt = api.getTx().getTxReceipt(rsp.getTxHash()).getObject();
System.out.format("ADD transaction receipt:%n%s%n", txReceipt);

// call function: getting a user address
rsp = ctr.newFunction("getUserAddress").setParam(ISString.copyFrom(keyToAdd)).build().execute().getObject();

System.out.println("GET response:\n" + rsp);

// checking that received value matches input
String received = Address.wrap((byte[]) rsp.getData().get(0)).toString();
if (!received.equals(addressToAdd)) {
    System.out.format("%nThe received value:%n%s%ndoes not match the given parameter:%n%s%n",
                      received,
                      addressToAdd);
} else {
    System.out.format("%nThe received value:%n%s%nmatches the given parameter:%n%s%n", received, addressToAdd);
}
```

* Sample output:

```
ADD response:
constant: false,
data:
transactionHash: 8e2f8e2b18033aed17bead6dddca8ec2a8290bdb3e317ec82cc5ed89eea6c4b4,
status: Transaction Done.,
error: null

ADD transaction receipt:
txIndex: 0,
blockNumber: 314761,
nrg: 44010,
nrgCumulativeUsed: 44010,
blockHash: 0xe6a5d4fbde342bc11f3e6b9c9bd97ab8a0db12b60a2581dbc330fcc9510ba6cc,
txHash: 0x8e2f8e2b18033aed17bead6dddca8ec2a8290bdb3e317ec82cc5ed89eea6c4b4,
from: 0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5,
to: 0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5,
contractAddress: ,
log:
[
  address: 0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5,
  data: 0x,
  topics:
  [
    0x0xa226db3f664042183ee0281230bba26cbf7b5057e50aee7f25a175ff45ce4d7f,
    0x0xa0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab
  ]
]

GET response:
constant: true,
data:
[a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab]
transactionHash: 0000000000000000000000000000000000000000000000000000000000000000,
status: Api failed.,
error: null


The received value:
a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab
matches the given parameter:
a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// input values
let txHash = '0xb35c28a10bc996f1cdd81425e6c90d4c841ed6ba6c7f039e76d448a6c869d7bc';
const source_personnel = "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";
let addressToAdd = "0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab";
let keyToAdd = "key-ab456";

// get contract object parameters
let txReceipt = web3.eth.getTransactionReceipt(txHash);
let contractAccount = txReceipt.contractAddress;
let ownerAddress = txReceipt.from;
let abiDefinition = web3.eth.compile.solidity(source_personnel).Personnel.info.abiDefinition;

// get contract object using ownerAddress & contractAccount & abiDefinition
let ctr = web3.eth.contract(abiDefinition).at(contractAccount);

// unlock account
web3.personal.unlockAccount(ownerAddress, "password", 100);

// execute function: adding a user address
let rsp = ctr.addAddress(keyToAdd, addressToAdd, {from: ownerAddress, gas: 2000000, gasPrice: 10000000000});

console.log("ADD response: " + rsp);

// get & print receipt
txReceipt = web3.eth.getTransactionReceipt(rsp);
// repeat till tx processed
while (txReceipt == null) {
  // wait 10 sec
  sleep(10000);
  txReceipt = web3.eth.getTransactionReceipt(rsp);
}
console.log("\nADD transaction receipt:")
console.log(txReceipt);

// call function: getting a user address
rsp = ctr.getUserAddress(keyToAdd);

console.log("\nGET response: " + rsp);

// checking that received value matches input
if (!rsp == addressToAdd) {
    console.log("\nThe received value:\n%s\n does not match the given parameter:\n%s\n", rsp, addressToAdd);
} else {
    console.log("\nThe received value:\n%s\nmatches the given parameter:\n%s\n", rsp, addressToAdd);
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

* Sample output:

```
ADD response: 0xb1bdd53910ae2fb4155f48bfc0b65f02faf3221f90a929168526e334f828f771

ADD transaction receipt:
{ blockHash: '0x544f3af1d55207705a1c38cb89cc0a5295a2d874993159f3edf3389397a8f094',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000004000000400000000000000000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000020000000000400000000000000000000000000000000000000000000002000000000000000000000200000000400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 68010,
  contractAddress: null,
  transactionIndex: 0,
  transactionHash: '0xb1bdd53910ae2fb4155f48bfc0b65f02faf3221f90a929168526e334f828f771',
  gasLimit: '0x1e8480',
  cumulativeNrgUsed: 68010,
  gasUsed: '0x0109aa',
  blockNumber: 314695,
  root: '97bf8fa45395ce9da3304152b6d7379036bbfafa0a26241916ac6f821f075a73',
  cumulativeGasUsed: '0x109aa',
  from: '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  to: '0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5',
  logs:
   [ { address: '0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5',
       logIndex: 0,
       data: '0x',
       topics: [Array],
       blockNumber: 314695,
       transactionIndex: 0 } ],
  gasPrice: '0x02540be400',
  status: '0x1' }

GET response: 0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab

The received value:
0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab
matches the given parameter:
0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab
```

</details>

#### <a name="call"></a>Call Contract Function

The examples below show how to use the APIs to call a contract function.
The functionality is compatible with [`eth_call`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_call).

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).
* The example in [Execute Contract Function](#ctr-execute) also applies where `ctr.newFunction("getUserAddress").setParam(ISString.copyFrom(keyToAdd)).build().execute()` defaults to a call (does not create a transaction).

```java
// input values
Hash256 txHash = Hash256.wrap("0xb35c28a10bc996f1cdd81425e6c90d4c841ed6ba6c7f039e76d448a6c869d7bc");
String source_personnel =
        "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";
String addressToAdd = "a0ab234ab234ab234ab234ab234ab234ab234ab234ab234ab234ab234ab234ab";
String keyToAdd = "key-ab123";

// get contract object parameters
TxReceipt txReceipt = api.getTx().getTxReceipt(txHash).getObject();
Address contractAccount = txReceipt.getContractAddress();
Address ownerAddress = txReceipt.getFrom();
String abiDefinition = ((Map<String, CompileResponse>) api.getTx().compile(source_personnel).getObject())
        .get("Personnel").getAbiDefString();

// get contract object using ownerAddress & contractAccount & abiDefinition
IContract ctr = api.getContractController().getContractAt(ownerAddress, contractAccount, abiDefinition);

// unlock account
api.getWallet().unlockAccount(ownerAddress, "password");

// call function: adding a user address
ContractResponse rsp = ctr.newFunction("addAddress").setFrom(ownerAddress).setParam(ISString.copyFrom(keyToAdd))
        .setParam(IAddress.copyFrom(addressToAdd)).setTxNrgLimit(NRG_LIMIT_TX_MAX).setTxNrgPrice(NRG_PRICE_MIN)
        .build().call().getObject();

System.out.println("ADD response:\n" + rsp);

// call function: getting a user address
rsp = ctr.newFunction("getUserAddress").setParam(ISString.copyFrom(keyToAdd)).build().call().getObject();

System.out.println("GET response:\n" + rsp);

// checking that received value matches input
String received = Address.wrap((byte[]) rsp.getData().get(0)).toString();
if (!received.equals(addressToAdd)) {
    System.out.format("The received value:%n%s%ndoes not match the given parameter:%n%s%n",
                      received,
                      addressToAdd);
} else {
    System.out.format("The received value:%n%s%nmatches the given parameter:%n%s%n", received, addressToAdd);
}
```

* Sample output:

```
ADD response:
constant: true,
data:
transactionHash: 0000000000000000000000000000000000000000000000000000000000000000,
status: Api failed.,
error: null

GET response:
constant: true,
data:
[a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab]
transactionHash: 0000000000000000000000000000000000000000000000000000000000000000,
status: Api failed.,
error: null

The received value:
a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab
does not match the given parameter:
a0ab234ab234ab234ab234ab234ab234ab234ab234ab234ab234ab234ab234ab
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The same example as in [Execute Contract Function](#ctr-execute) applies where `ctr.getUserAddress(keyToAdd)` defaults to a call (does not create a transaction).

</details>

#### <a name="ctr-example"></a>Complete Examples

Each code example below illustrates APIs interactions for the following use cases:
1. get available compilers,
2. compile solidity code,
3. get current NRG price,
4. estimate needed NRG,
5. get contract code,
6. get stored value,
7. deploy a contract,
8. execute a contract function, and
9. call a contract function.

<details>
<summary><i>Java Code</i></summary>
<br/>

```java
package org.aion.tutorials;

import org.aion.api.IAionAPI;
import org.aion.api.IContract;
import org.aion.api.IUtils;
import org.aion.api.sol.IAddress;
import org.aion.api.sol.ISString;
import org.aion.api.type.*;
import org.aion.base.type.Address;
import org.aion.base.type.Hash256;
import org.aion.base.util.ByteArrayWrapper;
import org.aion.base.util.Bytesable;

import java.math.BigInteger;
import java.util.Map;

import static org.aion.api.ITx.*;

public class ContractExample {

    public static void main(String[] args) throws InterruptedException {

        // connect to Java API
        IAionAPI api = IAionAPI.init();
        ApiMsg apiMsg = api.connect(IAionAPI.LOCALHOST_URL);

        // failed connection
        if (apiMsg.isError()) {
            System.out.format("Could not connect due to <%s>%n", apiMsg.getErrString());
            System.exit(-1);
        }

        // 1. eth_getCompilers

        // not available/needed at present

        // 2. eth_compileSolidity

        // contract source code
        String source_ticker = "contract ticker { uint public val; function tick () { val+= 1; } }";

        // compile code
        Map<String, CompileResponse> result = api.getTx().compile(source_ticker).getObject();

        // print result
        System.out.println(result);

        // 3. eth_gasPrice

        // get NRG price
        long price = api.getTx().getNrgPrice().getObject();

        // print price
        System.out.println("\ncurrent NRG price = " + price + " nAmp");

        // 4. eth_estimateGas

        // get NRG estimate for contract
        long estimate = api.getTx().estimateNrg(result.get("ticker").getCode()).getObject();

        // print estimate
        System.out.println("\nNRG estimate for contract = " + estimate + " NRG");

        // transaction data
        Address sender = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");
        Address receiver = Address.wrap("a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e");
        BigInteger amount = BigInteger.valueOf(1_000_000_000_000_000_000L); // = 1 AION
        ByteArrayWrapper data = ByteArrayWrapper.wrap("test.message".getBytes());

        // prepare transaction
        TxArgs tx = new TxArgs.TxArgsBuilder().data(data).from(sender).to(receiver).value(amount).createTxArgs();

        // get NRG estimate for transaction
        estimate = api.getTx().estimateNrg(tx).getObject();

        // print estimate
        System.out.println("\nNRG estimate for transaction = " + estimate + " NRG");

        // 5. eth_getCode

        // set contract account
        Address contractAccount = Address.wrap("a0960fcb7d6423a0446243916c7c6360543b3d2f9c5e1c5ff7badb472b782b79");

        // get code from latest block
        long blockNumber = -1L; // indicates latest block
        byte[] code = api.getTx().getCode(contractAccount, blockNumber).getObject();

        // print code
        System.out.println("\n0x" + IUtils.bytes2Hex(code));

        // 6. eth_getStorageAt

        // view contract creation tx
        Hash256 txHash = Hash256.wrap("0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48");
        TxReceipt receipt = api.getTx().getTxReceipt(txHash).getObject();
        System.out.println("\n" + receipt);

        // set contract account
        contractAccount = receipt.getContractAddress();

        // get value from storage
        String valuePos0 = api.getChain().getStorageAt(contractAccount, 0, blockNumber).getObject();
        String valuePos1 = api.getChain().getStorageAt(contractAccount, 1, blockNumber).getObject();

        // print values
        // in this case the first two values are the contract owner
        System.out.println("concatenated values = " + valuePos0 + valuePos1);

        // 7.a) deploy contract

        // contract source code
        String source_personnel =
                "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                        + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                        + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                        + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                        + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                        + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                        + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                        + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                        + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";

        // compile code
        result = api.getTx().compile(source_personnel).getObject();
        CompileResponse contract = result.get("Personnel");

        // unlock owner
        Address owner = Address.wrap("a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5");
        boolean isUnlocked = api.getWallet().unlockAccount(owner, "password", 100).getObject();
        System.out.format("\nowner account %s%n", isUnlocked ? "unlocked" : "locked");

        // deploy contract
        ContractDeploy.ContractDeployBuilder builder = new ContractDeploy.ContractDeployBuilder()
                .compileResponse(contract).value(BigInteger.ZERO).nrgPrice(NRG_PRICE_MIN)
                .nrgLimit(NRG_LIMIT_CONTRACT_CREATE_MAX).from(owner).data(ByteArrayWrapper.wrap(Bytesable.NULL_BYTE));

        DeployResponse contractResponse = api.getTx().contractDeploy(builder.createContractDeploy()).getObject();

        // print response
        txHash = contractResponse.getTxid();
        contractAccount = contractResponse.getAddress();
        System.out.format("%ntransaction hash:%n\t%s%ncontract address: %n\t%s%n", txHash, contractAccount);

        // get & print receipt
        TxReceipt txReceipt = api.getTx().getTxReceipt(txHash).getObject();
        System.out.format("%ntransaction receipt:%n%s%n", txReceipt);

        // 7.b) deploy contract

        isUnlocked = api.getWallet().unlockAccount(owner, "password", 100).getObject();
        System.out.format("%nowner account %s%n", isUnlocked ? "unlocked" : "locked");

        // clear old deploy
        api.getContractController().clear();

        // deploy contract
        ApiMsg msg = api.getContractController()
                .createFromSource(source_personnel, owner, NRG_LIMIT_CONTRACT_CREATE_MAX, NRG_PRICE_MIN);

        if (msg.isError()) {
            System.out.println("deploy contract failed! " + msg.getErrString());
        } else {
            // get contract
            IContract contractRsp = api.getContractController().getContract();

            // print response
            txHash = contractRsp.getDeployTxId();
            contractAccount = contractRsp.getContractAddress();
            System.out.format("%ntransaction hash:%n\t%s%ncontract address: %n\t%s%n", txHash, contractAccount);

            // get & print receipt
            txReceipt = api.getTx().getTxReceipt(txHash).getObject();
            System.out.format("%ntransaction receipt:%n%s%n", txReceipt);
        }

        // 8. execute a contract function

        // input values
        txHash = Hash256.wrap("0xb35c28a10bc996f1cdd81425e6c90d4c841ed6ba6c7f039e76d448a6c869d7bc");
        String addressToAdd = "a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab";
        String keyToAdd = "key-ab123";

        // get contract object parameters
        txReceipt = api.getTx().getTxReceipt(txHash).getObject();
        contractAccount = txReceipt.getContractAddress();
        Address ownerAddress = txReceipt.getFrom();
        String abiDefinition = ((Map<String, CompileResponse>) api.getTx().compile(source_personnel).getObject())
                .get("Personnel").getAbiDefString();

        // get contract object using ownerAddress & contractAccount & abiDefinition
        IContract ctr = api.getContractController().getContractAt(ownerAddress, contractAccount, abiDefinition);

        // unlock account
        api.getWallet().unlockAccount(ownerAddress, "password");

        // execute function: adding a user address
        ContractResponse rsp = ctr.newFunction("addAddress").setFrom(ownerAddress).setParam(ISString.copyFrom(keyToAdd))
                .setParam(IAddress.copyFrom(addressToAdd)).setTxNrgLimit(NRG_LIMIT_TX_MAX).setTxNrgPrice(NRG_PRICE_MIN)
                .build().execute().getObject();

        System.out.println("ADD response:\n" + rsp);

        // wait for tx to be processed
        Thread.sleep(30000L);
        // get & print receipt
        txReceipt = api.getTx().getTxReceipt(rsp.getTxHash()).getObject();
        System.out.format("ADD transaction receipt:%n%s%n", txReceipt);

        // 9. eth_call

        // call function: getting a user address
        rsp = ctr.newFunction("getUserAddress").setParam(ISString.copyFrom(keyToAdd)).build().call().getObject();

        System.out.println("GET response:\n" + rsp);

        // checking that received value matches input
        String received = Address.wrap((byte[]) rsp.getData().get(0)).toString();
        if (!received.equals(addressToAdd)) {
            System.out.format("The received value:%n%s%ndoes not match the given parameter:%n%s%n",
                              received,
                              addressToAdd);
        } else {
            System.out.format("The received value:%n%s%nmatches the given parameter:%n%s%n", received, addressToAdd);
        }

        // disconnect from api
        api.destroyApi();

        System.exit(0);
    }
}
```

* Sample output:

```
{ticker=code :0x605060405234156100105760006000fd5b610015565b60c2806100236000396000f30060506040526000356c01000000000000000000000000900463ffffffff1680633c6bb43614603b5780633eaf5d9f146062576035565b60006000fd5b341560465760006000fd5b604c6075565b6040518082815260100191505060405180910390f35b3415606d5760006000fd5b6073607e565b005b60006000505481565b6001600060008282825054019250508190909055505b5600a165627a7a723058201c1817d957da90680b7c55720aff16e7a4407b31f7480f63210748a5cc4efb0c0029,
source :contract ticker { uint public val; function tick () { val+= 1; } },
language :,
languageVersion: ,
compilerVersion: ,
compilerOption: ,
abiDefString: [{"outputs":[{"name":"","type":"uint128"}],"constant":true,"payable":false,"inputs":[],"name":"val","type":"function"},{"outputs":[],"constant":false,"payable":false,"inputs":[],"name":"tick","type":"function"}],
abiDefinition:
[
constant: true,
anonymous: false,
payable: false,
type: function,
name: val,
inputs:
outputs:
[
indexed: false,
type: uint128,
name: ,
paramLengths:
[
]

]
isEvent: false,
isConstructor: false,
isFallback: false,
hashed: 3c6bb436

],
[
constant: false,
anonymous: false,
payable: false,
type: function,
name: tick,
inputs:
outputs:
isEvent: false,
isConstructor: false,
isFallback: false,
hashed: 3eaf5d9f

]
userDoc: constant: false,
name: ,
type: ,
inputs:
outputs:
,
developerDocconstant: false,
name: ,
type: ,
inputs:
outputs:

}

current NRG price = 10000000000 nAmp

NRG estimate for contract = 233661 NRG

NRG estimate for transaction = 21768 NRG
0x60506040526000356c01000000000000000000000000900463ffffffff1680630fd5f53f146100545780634985e85c146100e25780638da5cb5b1461015f578063ceb35b0f146101905761004e565b60006000fd5b34156100605760006000fd5b6100e06004808035906010019082018035906010019191908080600f01601080910402601001604051908101604052809392919081815260100183838082843782019150505050505090909190808060100135903590916020019091929080356effffffffffffffffffffffffffffff1916906010019091905050610203565b005b34156100ee5760006000fd5b6101426004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919050506103af565b604051808383825281601001526020019250505060405180910390f35b341561016b5760006000fd5b610173610457565b604051808383825281601001526020019250505060405180910390f35b341561019c5760006000fd5b6102016004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919080806010013590359091602001909192905050610466565b005b600060005080600101549054339091149190141615156102235760006000fd5b828260026000506000876040518082805190601001908083835b60108310151561026357805182525b60108201915060108101905060108303925061023d565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508282909180600101839055555050508060036000506000858582528160100152602001908152601001600020905060006101000a81548160ff02191690836f01000000000000000000000000000000900402179055507fdb8f9fd4bfba5ae615ae02e8b4d281b887225fd3d34a5ac7b8d78df768bb63e7856040518080601001828103825283818151815260100191508051906010019080838360005b8381101561036d5780820151818401525b601081019050610351565b50505050905090810190600f16801561039a5780820380516001836010036101000a031916815260100191505b509250505060405180910390a15b5b50505050565b6000600060026000506000846040518082805190601001908083835b6010831015156103f157805182525b6010820191506010810190506010830392506103cb565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508060010154905491509150610452565b915091565b60006000508060010154905482565b600060005080600101549054339091149190141615156104865760006000fd5b818160026000506000866040518082805190601001908083835b6010831015156104c657805182525b6010820191506010810190506010830392506104a0565b6001836010036101000a038019825116818451168082178552505050505050905001915050604051809103902090600019169090600019169082528160100152602001908152601001600020905060005082829091806001018390555550505081817fa226db3f664042183ee0281230bba26cbf7b5057e50aee7f25a175ff45ce4d7f60405160405180910390a25b5b5050505600a165627a7a723058201a36e96ba95136c7bf35a644bf8d3382c49dafc64e9362b025ed6f4eb99ed0640029
txIndex: 0,
blockNumber: 298287,
nrg: 349674,
nrgCumulativeUsed: 349674,
blockHash: 0x2e9b90a21f9702fde90807e42db0107ce85b8c14a99c1f9f76182ec20599eebc,
txHash: 0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48,
from: 0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e,
to: 0x,
contractAddress: a076ddb4cf37f7cd1360de9cf2336f75f06c38655da09d4e3f9b690acf57c2e1,
log:


concatenated values = a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e
owner account unlocked

transaction hash:
	bc3863dbc07a0020778a632622ebcd4ffea02f8790682a7bc70581c850aaf30f
contract address:
	a0ae5633cd573960ca4d92b5127e49071214440bd070fc67bce495f214b7557e

transaction receipt:
txIndex: 0,
blockNumber: 314963,
nrg: 349674,
nrgCumulativeUsed: 349674,
blockHash: 0x331eae747c79f23d57bd28bf52257e8da35f65f4bfd5390e1831b66e4eea3c41,
txHash: 0xbc3863dbc07a0020778a632622ebcd4ffea02f8790682a7bc70581c850aaf30f,
from: 0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5,
to: 0x,
contractAddress: a0ae5633cd573960ca4d92b5127e49071214440bd070fc67bce495f214b7557e,
log:


owner account unlocked

transaction hash:
	bf9627f42dcb942f8f06cd82a8b8764452d3fb1fe73d19ccf10526aa8ce5ecd0
contract address:
	a0764b1ca325a15656e87216f6c4d80dc8d9e22fb99e4aa5e2529d1c8c8f4c25

transaction receipt:
txIndex: 0,
blockNumber: 314965,
nrg: 349674,
nrgCumulativeUsed: 349674,
blockHash: 0xbdc4b952822bf7091a55dad2f232f8ca11a6ba5146c404f5714f64447512931c,
txHash: 0xbf9627f42dcb942f8f06cd82a8b8764452d3fb1fe73d19ccf10526aa8ce5ecd0,
from: 0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5,
to: 0x,
contractAddress: a0764b1ca325a15656e87216f6c4d80dc8d9e22fb99e4aa5e2529d1c8c8f4c25,
log:

ADD response:
constant: false,
data:
transactionHash: db546561c8cef16a81fea509053a9917e65269bd7261191b8dd9272a9fa2fcc3,
status: Transaction Done.,
error: null

ADD transaction receipt:
txIndex: 0,
blockNumber: 314967,
nrg: 44010,
nrgCumulativeUsed: 44010,
blockHash: 0x52ef01f0938a8dd29b67ba18bd62ad3312f172e78580ad51034994878a5865e5,
txHash: 0xdb546561c8cef16a81fea509053a9917e65269bd7261191b8dd9272a9fa2fcc3,
from: 0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5,
to: 0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5,
contractAddress: ,
log:
[
  address: 0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5,
  data: 0x,
  topics:
  [
    0x0xa226db3f664042183ee0281230bba26cbf7b5057e50aee7f25a175ff45ce4d7f,
    0x0xa0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab
  ]
]

GET response:
constant: true,
data:
[a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab]
transactionHash: 0000000000000000000000000000000000000000000000000000000000000000,
status: Api failed.,
error: null

The received value:
a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab
matches the given parameter:
a0ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab123ab
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

```js
const Web3 = require('/path/to/aion/web3');
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

// 1. eth_getCompilers

// get list of available compilers
let compilers = web3.eth.getCompilers();

// print
console.log("available compilers: " + compilers + "\n");

// 2. eth_compileSolidity

// contract source code
const source_ticker = 'contract ticker { uint public val; function tick () { val+= 1; } }';

// compile code
let result = web3.eth.compile.solidity(source_ticker);

// print result
console.log(result);

// 3. eth_gasPrice

// get NRG price
let price = web3.eth.gasPrice

// print price
console.log("\ncurrent NRG price = " + price + " nAmp");

// 4. eth_estimateGas

// compile contract source code
result = web3.eth.compile.solidity(source_ticker);

// get NRG estimate for contract
let estimate = web3.eth.estimateGas({data:result.ticker.code});

// print estimate
console.log("\nNRG estimate for contract = " + estimate + " NRG");

// transaction data
let sender = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';
let receiver = 'a0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e';
let amount = 1000000000000000000; // = 1 AION
let data = '0x746573742e6d657373616765'; // hex for "test.message"

// get NRG estimate for transaction
estimate = web3.eth.estimateGas({data: data, from: sender, to: receiver, value: amount})

// print estimate
console.log("\nNRG estimate for transaction = " + estimate + " NRG");

// 5. eth_getCode

// set contract account
let ctAcc = 'a0960fcb7d6423a0446243916c7c6360543b3d2f9c5e1c5ff7badb472b782b79';

// get code from latest block
let code = web3.eth.getCode(ctAcc, 'latest');

// print code
console.log("\n" + code + "\n");

// 6. eth_getStorageAt

// view contract creation tx
let txHash = '0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48';
let receipt = web3.eth.getTransactionReceipt(txHash);
console.log(receipt);

// set contract account
let contractAccount = receipt.contractAddress;

// get value from storage
let valuePos0 = web3.eth.getStorageAt(contractAccount, 0, 'latest');
let valuePos1 = web3.eth.getStorageAt(contractAccount, 1, 'latest');

// print values
// in this case the first two values are the contract owner
console.log("\nconcatenated values = " + valuePos0 + valuePos1);

// 7. deploy contract

// contract source code
const source_personnel = "contract Personnel { address public owner; modifier onlyOwner() { require(msg.sender == owner); _;} "
                + "mapping(bytes32 => address) private userList; /** 3 LSB bits for each privilege type */ "
                + "mapping(address => bytes1) private userPrivilege; function Personnel(){ owner = msg.sender; } "
                + "event UserAdded(string _stamp); event AddressAdded(address indexed _addr); "
                + "function getUserAddress(string _stamp) constant returns (address){ return userList[sha3(_stamp)]; } "
                + "function addUser(string _stamp, address _addr, bytes1 _userPrivilege) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; userPrivilege[_addr] = _userPrivilege; "
                + "UserAdded(_stamp); } function addAddress(string _stamp, address _addr) "
                + "onlyOwner{ userList[sha3(_stamp)] = _addr; AddressAdded(_addr); } }";

// compile code
result = web3.eth.compile.solidity(source_personnel);
let abi = result.Personnel.info.abiDefinition;
code = result.Personnel.code;

// unlock owner
let owner = 'a06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5';
let isUnlocked = web3.personal.unlockAccount(owner, "password", 100);
console.log("\nowner account " + (isUnlocked ? "unlocked" : "locked"));

// deploy contract
let response = web3.eth.contract(abi).new({from: owner, data: code, gasPrice: 10000000000, gas: 5000000});

// print response
txHash = response.transactionHash;
contractAccount = response.address;
// note that the address is not defined in the response
console.log("\ntransaction hash:\n\t" + txHash + "\ncontract address:\n\t" + contractAccount);

// get & print receipt
txReceipt = web3.eth.getTransactionReceipt(txHash);
// repeat till tx processed
while (txReceipt == null) {
  // wait 10 sec
  sleep(10000);
  txReceipt = web3.eth.getTransactionReceipt(txHash);
}
// getting the address from the receipt
contractAccount = txReceipt.contractAddress;
console.log("\ncontract address:\n\t" + contractAccount);
// print full receipt
console.log("\ntransaction receipt:");
console.log(txReceipt);

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// 8. execute a contract function

// input values
txHash = '0xb35c28a10bc996f1cdd81425e6c90d4c841ed6ba6c7f039e76d448a6c869d7bc';
let addressToAdd = "0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab";
let keyToAdd = "key-ab456";

// get contract object parameters
txReceipt = web3.eth.getTransactionReceipt(txHash);
contractAccount = txReceipt.contractAddress;
let ownerAddress = txReceipt.from;
let abiDefinition = web3.eth.compile.solidity(source_personnel).Personnel.info.abiDefinition;

// get contract object using ownerAddress & contractAccount & abiDefinition
let ctr = web3.eth.contract(abiDefinition).at(contractAccount);

// unlock account
web3.personal.unlockAccount(ownerAddress, "password", 100);

// execute function: adding a user address
let rsp = ctr.addAddress(keyToAdd, addressToAdd, {from: ownerAddress, gas: 2000000, gasPrice: 10000000000});

console.log("\nADD response: " + rsp);

// get & print receipt
txReceipt = web3.eth.getTransactionReceipt(rsp);
// repeat till tx processed
while (txReceipt == null) {
  // wait 10 sec
  sleep(10000);
  txReceipt = web3.eth.getTransactionReceipt(rsp);
}
console.log("\nADD transaction receipt:")
console.log(txReceipt);

// 9. eth_call

// call function: getting a user address
rsp = ctr.getUserAddress(keyToAdd);

console.log("\nGET response: " + rsp);

// checking that received value matches input
if (!rsp == addressToAdd) {
    console.log("\nThe received value:\n%s\n does not match the given parameter:\n%s\n", rsp, addressToAdd);
} else {
    console.log("\nThe received value:\n%s\nmatches the given parameter:\n%s\n", rsp, addressToAdd);
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

* Sample output:

```
available compilers: solidity

{ ticker:
   { code: '0x605060405234156100105760006000fd5b610015565b60c2806100236000396000f30060506040526000356c01000000000000000000000000900463ffffffff1680633c6bb43614603b5780633eaf5d9f146062576035565b60006000fd5b341560465760006000fd5b604c6075565b6040518082815260100191505060405180910390f35b3415606d5760006000fd5b6073607e565b005b60006000505481565b6001600060008282825054019250508190909055505b5600a165627a7a723058201c1817d957da90680b7c55720aff16e7a4407b31f7480f63210748a5cc4efb0c0029',
     info:
      { abiDefinition: [Array],
        languageVersion: '0',
        language: 'Solidity',
        compilerVersion: '0.4.15+commit.ecf81ee5.Linux.g++',
        source: 'contract ticker { uint public val; function tick () { val+= 1; } }' } } }

current NRG price = 10000000000 nAmp

NRG estimate for contract = 233661 NRG

NRG estimate for transaction = 21768 NRG

0x60506040526000356c01000000000000000000000000900463ffffffff1680630fd5f53f146100545780634985e85c146100e25780638da5cb5b1461015f578063ceb35b0f146101905761004e565b60006000fd5b34156100605760006000fd5b6100e06004808035906010019082018035906010019191908080600f01601080910402601001604051908101604052809392919081815260100183838082843782019150505050505090909190808060100135903590916020019091929080356effffffffffffffffffffffffffffff1916906010019091905050610203565b005b34156100ee5760006000fd5b6101426004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919050506103af565b604051808383825281601001526020019250505060405180910390f35b341561016b5760006000fd5b610173610457565b604051808383825281601001526020019250505060405180910390f35b341561019c5760006000fd5b6102016004808035906010019082018035906010019191908080600f0160108091040260100160405190810160405280939291908181526010018383808284378201915050505050509090919080806010013590359091602001909192905050610466565b005b600060005080600101549054339091149190141615156102235760006000fd5b828260026000506000876040518082805190601001908083835b60108310151561026357805182525b60108201915060108101905060108303925061023d565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508282909180600101839055555050508060036000506000858582528160100152602001908152601001600020905060006101000a81548160ff02191690836f01000000000000000000000000000000900402179055507fdb8f9fd4bfba5ae615ae02e8b4d281b887225fd3d34a5ac7b8d78df768bb63e7856040518080601001828103825283818151815260100191508051906010019080838360005b8381101561036d5780820151818401525b601081019050610351565b50505050905090810190600f16801561039a5780820380516001836010036101000a031916815260100191505b509250505060405180910390a15b5b50505050565b6000600060026000506000846040518082805190601001908083835b6010831015156103f157805182525b6010820191506010810190506010830392506103cb565b6001836010036101000a03801982511681845116808217855250505050505090500191505060405180910390209060001916909060001916908252816010015260200190815260100160002090506000508060010154905491509150610452565b915091565b60006000508060010154905482565b600060005080600101549054339091149190141615156104865760006000fd5b818160026000506000866040518082805190601001908083835b6010831015156104c657805182525b6010820191506010810190506010830392506104a0565b6001836010036101000a038019825116818451168082178552505050505050905001915050604051809103902090600019169090600019169082528160100152602001908152601001600020905060005082829091806001018390555550505081817fa226db3f664042183ee0281230bba26cbf7b5057e50aee7f25a175ff45ce4d7f60405160405180910390a25b5b5050505600a165627a7a723058201a36e96ba95136c7bf35a644bf8d3382c49dafc64e9362b025ed6f4eb99ed0640029

{ blockHash: '0x2e9b90a21f9702fde90807e42db0107ce85b8c14a99c1f9f76182ec20599eebc',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 349674,
  contractAddress: '0xa076ddb4cf37f7cd1360de9cf2336f75f06c38655da09d4e3f9b690acf57c2e1',
  transactionIndex: 0,
  transactionHash: '0xb42a5f995450531f66e7db40efdfad2c310fa0f8dbca2a88c31fdc4837368e48',
  gasLimit: '0x4c4b40',
  cumulativeNrgUsed: 349674,
  gasUsed: '0x0555ea',
  blockNumber: 298287,
  root: 'e433bbbc1714c2301faf7ac447c5881f345c32f8853e121694e430219cacb9d5',
  cumulativeGasUsed: '0x555ea',
  from: '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  to: '0x',
  logs: [],
  gasPrice: '0x02540be400',
  status: '0x1' }

concatenated values = 0xa0bd0ef93902d9e123521a67bef7391e0x9487e963b2346ef3b3ff78208835545e

owner account unlocked

transaction hash:
	0xc62efed241bca72bbf04bf018e86f55ddf76c37c90734e70eb4029821eb3d371
contract address:
	undefined

contract address:
	0xa0c12a380d25c48eb1ef57902ab9e9d88a260ea6d0c984269debd911e8ce6770

transaction receipt:
{ blockHash: '0xebc27704b64416ff78aa53141652612220ff603fbd0ead7b9ec52e692137f897',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 349674,
  contractAddress: '0xa0c12a380d25c48eb1ef57902ab9e9d88a260ea6d0c984269debd911e8ce6770',
  transactionIndex: 0,
  transactionHash: '0xc62efed241bca72bbf04bf018e86f55ddf76c37c90734e70eb4029821eb3d371',
  gasLimit: '0x4c4b40',
  cumulativeNrgUsed: 349674,
  gasUsed: '0x0555ea',
  blockNumber: 315024,
  root: '262b17599aae7369a19c08880a4164eb329ae2ff93f3c047ec6a7693782307cb',
  cumulativeGasUsed: '0x555ea',
  from: '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  to: '0x',
  logs: [],
  gasPrice: '0x02540be400',
  status: '0x1' }

ADD response: 0xb4636988594e8c76f5a666c8fb9d9fd85eee62bc41b0c6376050150665dc07f2

ADD transaction receipt:
{ blockHash: '0x01c1363c109ef49b735603141fbb6c30f8ecb7bf4d9c51e27fd8ce8090cfb493',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000004000000400000000000000000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000020000000000400000000000000000000000000000000000000000000002000000000000000000000200000000400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 44010,
  contractAddress: null,
  transactionIndex: 0,
  transactionHash: '0xb4636988594e8c76f5a666c8fb9d9fd85eee62bc41b0c6376050150665dc07f2',
  gasLimit: '0x1e8480',
  cumulativeNrgUsed: 44010,
  gasUsed: '0x00abea',
  blockNumber: 315026,
  root: 'c24aa18c7b77d72d2a554feada58228fe9938f6de3cc6124e3bf103eff0bbc10',
  cumulativeGasUsed: '0xabea',
  from: '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  to: '0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5',
  logs:
   [ { address: '0xa0f2bc28cc71bf81e38e91bcfdd1a89e8b67af39b65d3a046b1627482a0a72e5',
       logIndex: 0,
       data: '0x',
       topics: [Array],
       blockNumber: 315026,
       transactionIndex: 0 } ],
  gasPrice: '0x02540be400',
  status: '0x1' }

GET response: 0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab

The received value:
0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab
matches the given parameter:
0xa0ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab456ab
```

</details>

### <a name="network"></a>Network Functionality

Next we illustrate the API calls for querying the following information:
* [syncing status](#sync-status)
* [current peer count](#peer-count)
* [network listening status](#net-listen)

The APIs at present do not support querying for the network identifier similar to [`net_version`](https://github.com/ethereum/wiki/wiki/JSON-RPC#net_version).

The [final subsection](#net-example) contains code illustrating all of the above interactions.

#### <a name="sync-status"></a>Retrieve Syncing Status

The examples below show how to query the APIs for the kernel's sync status.
The functionality is compatible with [`eth_syncing`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_syncing).
In each code snippet, the sync status is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get sync status
SyncInfo status = api.getNet().syncInfo().getObject();

// print status
System.out.format("{ currentBlock: %d,%n  highestBlock: %d,%n  startingBlock: %d }%n",
                  status.getChainBestBlock(),
                  status.getNetworkBestBlock(),
                  status.getStartingBlock());
```

* Sample output:

```
{ currentBlock: 306442,
  highestBlock: 306442,
  startingBlock: 306413 }
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get sync status
let status = web3.eth.syncing;

// print status
console.log(status);
```

* Sample output:

```
{ currentBlock: 306442,
  highestBlock: 306442,
  startingBlock: 306413 }
```

</details>

#### <a name="peer-count"></a>Retrieve Current Number of Peers

The examples below show how to query the APIs for the number of active peers.
The functionality is compatible with [`net_peerCount`](https://github.com/ethereum/wiki/wiki/JSON-RPC#net_peercount).
In each code snippet, the number of active peers is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get peer count
int peerCount = api.getNet().getPeerCount().getObject();

// print count
System.out.format("number of active peers = %d%n", peerCount);
```

Alternative implementation with access to peer details:

```java
// get peer information
List<Node> peers = api.getNet().getActiveNodes().getObject();

// print count
System.out.format("number of active peers = %d%n", peers.size());
```

* Sample output:

```
number of active peers = 6
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get peer count
let peers = web3.net.peerCount;

// print count
console.log("number of active peers = " + peers);
```

* Sample output:

```
number of active peers = 6
```

</details>

#### <a name="net-listen"></a>Retrieve Network Listening Status

The examples below show how to query the APIs to determine if the kernel is actively listening for network connections.
The functionality is compatible with [`net_listening`](https://github.com/ethereum/wiki/wiki/JSON-RPC#net_listening).
In each code snippet, the listening status is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get listening status
boolean listening = api.getNet().isListening().getObject();

// print status
System.out.format("%slistening for connections%n", (listening ? "" : "not "));
```

* Sample output:

```
listening for connections
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// get listening status
let listening = web3.net.listening;

// print status
console.log((listening ? "" : "not ") + "listening for connections",);
```

* Sample output:

```
listening for connections
```

</details>

#### <a name="net-example"></a>Complete Examples

Each code example below retrieves and prints to the standard output the following:
1. the syncing status,
2. the number of active peers, and
3. if the client is listening for connections.


<details>
<summary><i>Java Code</i></summary>
<br/>

```java
package org.aion.tutorials;

import org.aion.api.IAionAPI;
import org.aion.api.type.ApiMsg;
import org.aion.api.type.Node;
import org.aion.api.type.SyncInfo;

import java.util.List;

public class NetworkExample {

    public static void main(String[] args) {

        // connect to Java API
        IAionAPI api = IAionAPI.init();
        ApiMsg apiMsg = api.connect(IAionAPI.LOCALHOST_URL);

        // failed connection
        if (apiMsg.isError()) {
            System.out.format("Could not connect due to <%s>%n", apiMsg.getErrString());
            System.exit(-1);
        }

        // 1. eth_syncing

        // get sync status
        SyncInfo status = api.getNet().syncInfo().getObject();

        // print status
        System.out.format("{ currentBlock: %d,%n  highestBlock: %d,%n  startingBlock: %d }%n",
                          status.getChainBestBlock(),
                          status.getNetworkBestBlock(),
                          status.getStartingBlock());

        // 2.a) net_peerCount

        // get peer count
        int peerCount = api.getNet().getPeerCount().getObject();

        // print count
        System.out.format("%nnumber of active peers = %d%n", peerCount);

        // 2.b) net_peerCount

        // get peer information
        List<Node> peers = api.getNet().getActiveNodes().getObject();

        // print count
        System.out.format("%nnumber of active peers = %d%n", peers.size());

        // 3. net_listening

        // get listening status
        boolean listening = api.getNet().isListening().getObject();

        // print status
        System.out.format("%n%slistening for connections%n", (listening ? "" : "not "));

        // disconnect from api
        api.destroyApi();

        System.exit(0);
    }
}
```

* Sample output:

```
{ currentBlock: 306442,
  highestBlock: 306442,
  startingBlock: 306413 }

number of active peers = 6

number of active peers = 6

listening for connections
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

```js
const Web3 = require('/path/to/aion/web3');
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

// 1. eth_syncing

// get sync status
let status = web3.eth.syncing;

// print status
console.log(status);

// 2. net_peerCount

// get peer count
let peers = web3.net.peerCount;

// print count
console.log("\nnumber of active peers = " + peers);

// 3. net_listening

// get listening status
let listening = web3.net.listening;

// print status
console.log("\n" +  (listening ? "" : "not ") + "listening for connections",);
```

* Sample output:

```
{ currentBlock: 306442,
  highestBlock: 306442,
  startingBlock: 306413 }

number of active peers = 6

listening for connections
```

</details>