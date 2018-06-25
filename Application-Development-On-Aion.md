This tutorial is designed to facilitate learning how to make projects that interact with the Aion kernel through its available APIs.
Two APIs are provided to developers, the [Java API](https://github.com/aionnetwork/aion_api) and the [JSON-RPC (Web3) API](https://github.com/aionnetwork/aion_web3).

The contents below show code examples by functionality and by JSON-RPC method. More examples will be gradually added.

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
</details>
<br/>

<details>
<summary><i><b>Contents by JSON-RPC Method</b></i></summary>
<br/>

- Examples for supported methods:
    - [`eth_accounts`](#all-acc)
    - [`eth_blockNumber`](#get-block)
    - [`eth_coinbase`](#miner-acc)
    - [`eth_getBalance`](#acc-balance)
    - [`eth_getBlockByHash`](#block-hash)
    - [`eth_getBlockByNumber`](#block-number)
    - [`eth_getBlockTransactionCountByHash`](#tx-hash)
    - [`eth_getBlockTransactionCountByNumber`](#tx-number)
    - [`eth_getTransactionByBlockHashAndIndex`](#tx-hash-idx)
    - [`eth_getTransactionByBlockNumberAndIndex`](#tx-number-idx)
    - [`eth_getTransactionByHash`](#tx-hash)
    - [`eth_getTransactionCount`](#acc-txs)
    - [`eth_getTransactionReceipt`](#tx-receipt)
    - [`eth_sendTransaction`](#send-tx)
    - [`eth_getCompilers`](#get-comp)
    - [`eth_compileSolidity`](#comp-sol)

- Details on methods that are not supported:
    - [`eth_compileLLL`](#get-comp)
    - [`eth_compileSerpent`](#get-comp)
    - [`eth_getUncleByBlockHashAndIndex`](#block)
    - [`eth_getUncleByBlockNumberAndIndex`](#block)
    - [`eth_getUncleCountByBlockHash`](#block)
    - [`eth_getUncleCountByBlockNumber`](#block)

</details>


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
api.connect(IAionAPI.LOCALHOST_URL);

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
    <apis-enabled>web3,eth,personal,stratum</apis-enabled>
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

### <a name="account"></a>Account Functionality

Next we illustrate the API calls for querying the following information:
* [all client accounts](#all-acc)
* [coinbase account](#miner-acc)
* [account balance](#acc-balance)
* [account transaction count](#acc-txs)

The [final subsection](#acc-example) contains code illustrating all of the above interactions.

#### <a name="all-acc"></a>Retrieve All Client Accounts

The examples below show how to query the APIs for the list of addresses owned by the client. The functionality is compatible with [`eth_accounts`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_accounts). In each code snippet, the addresses are retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get accounts from API
List<Address> accounts = api.getWallet().getAccounts().getObject();

// print accounts to standard output
System.out.format("the keystore contains %d accounts, as follow:%n",
                  accounts.size());
for (Address account : accounts) {
    System.out.format("\t%s%n", account.toString());
}
```

* Sample output:

```
the keystore contains 4 accounts, as follow:
	a0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
	a0bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
	a0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
	a0dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd
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
console.log(accounts);
```

* Alternative way to get the accounts:

```js
// get accounts from API
let accounts = web3.personal.listAccounts;

// print accounts to standard output
console.log(accounts2);
```

* Sample output:
```
[ '0xa0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa',
  '0xa0bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb',
  '0xa0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc',
  '0xa0dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd' ]
```

</details>

#### <a name="miner-acc"></a>Retrieve Coinbase Account

The examples below show how to query the APIs for the address configured to receive the mining rewards. The functionality is compatible with [`eth_coinbase`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_coinbase). In each code snippet, the address is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// get miner account
Address account = api.getWallet().getMinerAccount().getObject();

// print retrieved value
System.out.format("coinbase account = %s%n",
                  account.toString());
```

* Sample output:

```
coinbase account = a0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
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
console.log(miner);
```

* Sample output:

```
0xa0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
```

</details>

#### <a name="acc-balance"></a>Retrieve Account Balance

The examples below show how to query the APIs for the balance of a given address. The functionality is compatible with [`eth_getBalance`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getbalance). In each code snippet, the balance is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// interpret string as address
account = Address.wrap("a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab");

// get account balance
BigInteger balance = api.getChain().getBalance(account).getObject();

// print balance
System.out.format("%s has a balance of %d nAmp (over %d AION)%n",
                  account.toString(),
                  balance,
                  balance.divide(BigInteger.TEN.pow(18)));
```

* Sample output:

```
a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab has a balance of 225773315865192624951 nAmp (over 225 AION)
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

* The `web3` constant is initialized as in the [example above](#web3-use).

```js
// set address
let account = 'a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab';

// get account balance
let balance = web3.eth.getBalance(account);

// print balance
console.log(balance + " nAmp");
console.log(balance.shiftedBy(-18).toString() + " AION");
```

* Sample output:

```
225773315865192624951 nAmp
225.773315865192624951 AION
```

</details>

#### <a name="acc-txs"></a>Retrieve Account Transaction Count

The examples below show how to query the APIs for the number of transactions sent by a given address. The functionality is compatible with [`eth_getTransactionCount`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactioncount). In each code snippet, the transaction count is retrieved from the API and printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// interpret string as address
// note that hex prefix '0x' is optional
account = Address.wrap("0xa0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab");

// get number of transactions sent by account
BigInteger txCount = api.getChain().getNonce(account).getObject();

// print performed transactions
System.out.format("%s performed %d transactions%n",
                  account.toString(),
                  txCount);
```

* Sample output:

```
a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab performed 5 transactions
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
let account = '0xa0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab';

// get number of transactions sent by account
let txCount = web3.eth.getTransactionCount(account);

// print performed transactions
console.log(account + " performed " +txCount  + " transactions");
```

* Sample output:
```
a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab performed 5 transactions
```

</details>

#### <a name="acc-example"></a>Complete Examples

<details>
<summary><i>Java Code</i></summary>
<br/>

```java
package org.aion.tutorials;  

import org.aion.api.IAionAPI;  
import org.aion.base.type.Address;  

import java.math.BigInteger;  
import java.util.List;  

public class AccountExample {  

  public static void main(String[] args) {  

    // connect to Java API  
    IAionAPI api = IAionAPI.init();  
    api.connect(IAionAPI.LOCALHOST_URL);  

    // 1. eth_accounts

    // get accounts from API
    List<Address> accounts = api.getWallet().getAccounts().getObject();  

    // print accounts to standard output  
    System.out.format("the keystore contains %d accounts, as follow:%n",
                      accounts.size());  
    for (Address account : accounts) {  
      System.out.format("\t%s%n", account.toString());  
    }

    // 2. eth_coinbase

    // get miner account
    Address account = api.getWallet().getMinerAccount().getObject();  

    // print retrieved value  
    System.out.format("coinbase account = %s%n",
                      account.toString());  

    // 3. eth_getBalance

    // interpret string as address
    account = Address.wrap("a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab");  

    // get account balance  
    BigInteger balance = api.getChain().getBalance(account).getObject();  

    // print balance  
    System.out.format("%s has a balance of %d nAmp (over %d AION)%n",  
                      account.toString(),  
                      balance,  
                      balance.divide(BigInteger.TEN.pow(18)));  

    // 4. eth_getTransactionCount

    // interpret string as address
    // note that hex prefix '0x' is optional
    account = Address.wrap("0xa0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab");  

    // get number of transactions sent by account  
    BigInteger txCount = api.getChain().getNonce(account).getObject();  

    // print performed transactions  
    System.out.format("%s performed %d transactions%n",
                      account.toString(),
                      txCount);  

    // disconnect from api  
    api.destroyApi();  

    System.exit(0);  
  }  
}
```

* Sample output:

```
the keystore contains 4 accounts, as follow:
	a0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
	a0bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
	a0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
	a0dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd
coinbase account = a0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab has a balance of 225773315865192624951 nAmp (over 225 AION)
a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab performed 5 transactions
```

</details>
<br/>

<details>
<summary><i>JavaScript Code</i></summary>
<br/>

```js
const Web3 = require('/path/to/aion/web3');
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

// 1. eth_accounts

// get accounts from API
let accounts = web3.eth.accounts;

// print accounts to standard output
console.log(accounts);

// 2. eth_coinbase

// get miner account
let miner = web3.eth.coinbase;

// print retrieved value
console.log(miner);

// 3. eth_getBalance

// set address
let account = 'a0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab';

// get account balance
let balance = web3.eth.getBalance(account);

// print balance
console.log(balance + " nAmp");
console.log(balance.shiftedBy(-18).toString() + " AION");

// 4. eth_getTransactionCount

// set address
// note that hex prefix '0x' is optional
account = '0xa0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab';

// get number of transactions sent by account
let txCount = web3.eth.getTransactionCount(account);

// print performed transactions
console.log(account + " performed " + txCount + " transactions");

```

* Sample output:
```
[ '0xa0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa',
  '0xa0bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb',
  '0xa0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc',
  '0xa0dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd' ]
0xa0cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
225773315865192624951 nAmp
225.773315865192624951 AION
0xa0abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcab performed 5 transactions
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
System.out.format("current block is %d%n", blockNumber);
```

* Sample output:

```
current block is 245973
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
console.log("current block is " + blockNumber);
```

* Sample output:
```
current block is 245973
```

</details>

#### <a name="block-hash"></a>Retrieve Block by Hash

The examples below show how to query the APIs for information about a block given its hash. The functionality is compatible with [`eth_getBlockByHash`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getblockbyhash) except for the uncles information, since uncles are not supported in the Aion blockchain. In each code snippet, the block information is retrieved from the API and printed to the standard output.

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

The examples below show how to query the APIs for information about a block from the main chain given its number. The functionality is compatible with [`eth_getBlockByNumber`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getblockbynumber) except for the uncles information, since uncles are not supported in the Aion blockchain. In each code snippet, the block information is retrieved from the API and printed to the standard output.

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
        System.out.format("current block is %d%n", blockNumber);

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
console.log("current block is " + blockNumber);

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
current block is 249189

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

and performing the following actions:
* [send transaction](#send-tx)
<!--TODO * [send raw transaction](#raw-tx)-->

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
System.out.format("transaction details:%n%s%n", tx.toString());
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
System.out.format("transaction details:%n%s%n", tx.toString());
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
System.out.format("transaction details:%n%s%n", tx.toString());
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
System.out.format("transaction details:%n%s%n", txReceipt.toString());
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

The examples below show how to use the APIs to send a transaction. The functionality is compatible with [`eth_sendTransaction`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sendtransaction). In each code snippet, the transaction receipt is retrieved from the API and printed to the standard output. Other possible transaction parameters are energy price, energy limit and data.

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
System.out.format("%ntransaction hash:%n%s%n", txHash.toString());

// print receipt
TxReceipt txReceipt = api.getTx().getTxReceipt(txHash).getObject();
// repeat till tx processed
while (txReceipt == null) {
    // wait 10 sec
    sleep(10000);
    txReceipt = api.getTx().getTxReceipt(txHash).getObject();
}
System.out.format("%ntransaction receipt:%n%s%n", txReceipt.toString());
```

* Sample output:

```
sender account unlocked

transaction hash:
e9a7f92509597547be528155948994e80df1e9b318fc3dba8149201ccd00cb71

transaction receipt:
txIndex: 0,
blockNumber: 257876,
nrg: 21000,
nrgCumulativeUsed: 21000,
blockHash: 0x3535d4a23d78c185d87482c0248a3897d59021386c64beb752e1ec77ec9e28a0,
txHash: 0xe9a7f92509597547be528155948994e80df1e9b318fc3dba8149201ccd00cb71,
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

#### <a name="tx-example"></a>Complete Examples

<details>
<summary><i>Java Code</i></summary>
<br/>

```java
package org.aion.tutorials;

import org.aion.api.IAionAPI;
import org.aion.api.type.MsgRsp;
import org.aion.api.type.Transaction;
import org.aion.api.type.TxArgs;
import org.aion.api.type.TxReceipt;
import org.aion.base.type.Address;
import org.aion.base.type.Hash256;

import java.math.BigInteger;

import static java.lang.Thread.sleep;

public class TransactionExample {

    public static void main(String[] args) throws InterruptedException {

        // connect to Java API
        IAionAPI api = IAionAPI.init();
        api.connect(IAionAPI.LOCALHOST_URL);

        // 1. eth_getTransactionByHash

        // specify hash
        Hash256 hash = Hash256.wrap("0x5f2e74ade04ab9f6e8d4acd394f7f51832d4706d7268eea0ecc6391f94185b80");

        // get tx with given hash
        Transaction tx = api.getChain().getTransactionByHash(hash).getObject();

        // print tx information
        System.out.format("transaction details:%n%s%n", tx.toString());

        // 2. eth_getTransactionByBlockHashAndIndex

        // specify block hash
        hash = Hash256.wrap("0x50a906f4ccaf05a3ebca69cc4f84a116e6aec881e3c4d080c4df505fea65afab");
        // specify tx index = 0 -> first tx
        int index = 0;

        // get tx with given hash & index
        tx = api.getChain().getTransactionByBlockHashAndIndex(hash, index).getObject();

        // print tx information
        System.out.format("transaction details:%n%s%n", tx.toString());

        // 3. eth_getTransactionByBlockNumberAndIndex

        // specify block number
        long number = 247726L;
        // specify tx index = 1 -> second tx
        index = 1;

        // get tx with given number & index
        tx = api.getChain().getTransactionByBlockNumberAndIndex(number, index).getObject();

        // print tx information
        System.out.format("transaction details:%n%s%n", tx.toString());

        // 4. eth_getTransactionReceipt

        // specify tx hash
        hash = Hash256.wrap("0x2a1d8dc09f2c6670690bbda74377f59c8737df9812cf7c794fa35409d200fe3f");

        // get receipt for given tx hash
        TxReceipt txReceipt = api.getTx().getTxReceipt(hash).getObject();

        // print tx receipt
        System.out.format("transaction receipt:%n%s%n", txReceipt.toString());

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
        System.out.format("%ntransaction hash:%n%s%n", txHash.toString());

        // print receipt
        txReceipt = api.getTx().getTxReceipt(txHash).getObject();
        // repeat till tx processed
        while (txReceipt == null) {
            // wait 10 sec
            sleep(10000);
            txReceipt = api.getTx().getTxReceipt(txHash).getObject();
        }
        System.out.format("%ntransaction receipt:%n%s%n", txReceipt.toString());

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

transaction hash:
3e8610afdf8208f2f25dbb09659b0fc60d5435dac1e1e747842e1039f2d9d428

transaction receipt:
txIndex: 0,
blockNumber: 257920,
nrg: 21000,
nrgCumulativeUsed: 21000,
blockHash: 0xc4f3603317abfd91a8e589c92fbf2037c660a2710d01353c5afe03f8182a801a,
txHash: 0x3e8610afdf8208f2f25dbb09659b0fc60d5435dac1e1e747842e1039f2d9d428,
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

transaction hash: 0x0c75e5f34d168fb61fc56ef89723b436d6cd8e5f765ef0ba630879b1564d63f2

transaction receipt:
{ blockHash: '0x7528f809a0a41689f8e592ee5ca5fbc1cac22d8c64756affa9588f56d26e5dfb',
  nrgPrice: '0x02540be400',
  logsBloom: '00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  nrgUsed: 21000,
  contractAddress: null,
  transactionIndex: 0,
  transactionHash: '0x0c75e5f34d168fb61fc56ef89723b436d6cd8e5f765ef0ba630879b1564d63f2',
  gasLimit: '0x07a120',
  cumulativeNrgUsed: 21000,
  gasUsed: '0x5208',
  blockNumber: 257972,
  root: 'f89df590b60b9c71bc2c185ffec4bf0ed1e3665b6e3c95e412d4dbfb43a2ec32',
  cumulativeGasUsed: '0x5208',
  from: '0xa06f02e986965ddd3398c4de87e3708072ad58d96e9c53e87c31c8c970b211e5',
  to: '0xa0bd0ef93902d9e123521a67bef7391e9487e963b2346ef3b3ff78208835545e',
  logs: [],
  gasPrice: '0x02540be400',
  status: '0x1' }
```

</details>


### <a name="contract"></a>Contract Functionality

Next we illustrate the API calls for querying the following information:
* [available compilers](#get-comp)

and performing the following actions:
* [compile Solidity code](#comp-sol)

<!-- The [final subsection](#contract-example) contains code illustrating all of the above interactions.-->

#### <a name="get-comp"></a>Retrieve Available Compilers

The JavaScript example below shows how to query the APIs for the list of available code compilers. The functionality is compatible with [`eth_getCompilers`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getcompilers). Only the Solidity compiler is currently supported.

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
console.log("avaliable compilers:");
console.log(compilers);
```

* Sample output:

```
avaliable compilers:
[ 'solidity' ]
```

</details>

#### <a name="comp-sol"></a>Compile Solidity Code

The examples below show how to use the APIs to compile solidity code. The functionality is compatible with [`eth_compileSolidity`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_compilesolidity). In each code snippet, the given contract is compiled and the output is printed to the standard output.

<details>
<summary><i>Java Code</i></summary>
<br/>

* The `api` object is initialized as in the [example above](#java-use).

```java
// contract source code
String contract = "contract ticker { uint public val; function tick () { val+= 1; } }";

// compile code
ApiMsg result = api.getTx().compile(contract);

// print result
System.out.println(result.getObject().toString());
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
const sol = 'contract ticker { uint public val; function tick () { val+= 1; } }';

// compile code
let result = web3.eth.compile.solidity(sol);

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
