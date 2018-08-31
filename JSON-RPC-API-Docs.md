# RPC Server Features and Settings

## 0. Contents

- [1. Introduction](#1-introduction)
- [2. Ethereum JSON RPC API Spec Deviations](#2-ethereum-json-rpc-api-spec-deviations)
   - [2.1 Unsupported Endpoints](#21-unsupported-endpoints)
   - [2.2 Implementation Deviations](#22-implementation-deviations)
   - [2.3 Deprecated Endpoints](#23-deprecated-endpoints)
   - [2.4 Filter Implementation](#24-filter-implementation)
   - [2.4 Curl Behaviour](#25-curl-behaviour)  
- [3. JSON RPC 2.0 (Google) Spec Deviations](#3-json-rpc-20-google-spec-deviations)
- [4. Configurations Overview](#4-configurations-overview)
   - [4.1 Connection Settings](#41-connection-settings)
   - [4.2 Feature Toggles](#42-feature-toggles)
   - [4.3 Advanced Settings](#43-advanced-settings)
- [5. APIs Enabled](#5-apis-enabled)
   - [5.1 API Groups Available](#51-api-groups-available)
- [6. RPC Server Vendor](#6-rpc-server-vendor)
- [7. RPC Server Performance & Resource Utilization (Advanced)](#7-rpc-server-performance--resource-utilization-advanced)
   - [7.1 Worker Threads](#71-worker-threads)
   - [7.2 IO Threads](#72-io-threads)
   - [7.3 Request Queue Size](#73-request-queue-size)
- [8 RPC Server Debugging Options](#8-rpc-server-debugging-options)
   - [8.1 Logger Settings](#81-logger-settings)
   - [8.2 Stuck Thread Detector (RPC Config Setting)](#82-stuck-thread-detector-rpc-config-setting)
- [9. Enabling Cross-Origin Requests](#9-enabling-cross-origin-requests)
- [10. Advanced Configurations](#10-advanced-configurations)
- [11. RPC Server over HTTPS](#11-rpc-server-over-https-secured-via-ssl)
   - [11.1 Generating Self Signed Certificates](#111-generating-self-signed-certificates)

## 1. Introduction

The RPC server embedded in the Aion Kernel is an HTTP server that implements the [Ethereum JSON RPC Spec](https://github.com/ethereum/wiki/wiki/JSON-RPC) with minor exceptions listed below. In addition, the Aion RPC server is compliant with the [JSON RPC 2.0 Spec](https://www.jsonrpc.org/specification), with minor exceptions listed below.

"Wbb3" clients like [web3.js](https://github.com/aionnetwork/aion_web3) and [web3.py](https://github.com/ethereum/web3.py) are essentially language-specific wrappers around this JSON RPC API. Therefore, if you are using Web3 to connect to Aion, you are essentially consuming the Aion JSON RPC API.

## 2. Ethereum JSON RPC API Spec Deviations

> This application note is current as of the Ethereum JSON RPC API Spec 2018-08-22 revision. 

Following are the details on the Aion Kernel RPC server's non-compliance & deviations with the [Ethereum JSON RPC API Spec](https://github.com/ethereum/wiki/wiki/JSON-RPC). **Apart from the below-mentioned exceptions, the Aion RPC server is in-compliance with all other requirements defined in the aforementioned specification.**

### 2.1 Unsupported Endpoints 

The following methods were not implemented. Calling these methods will result in a METHOD_NOT_FOUND (-32601) error. Rationale for non-implementation is provided. 

#### 2.1.1 Uncle-related Functions

The following Spec-defined functions deal with Uncles, which are not part of the Aion protocol:

* `eth_getUncleCountByBlockHash`
* `eth_getUncleCountByBlockNumber`
* `eth_getUncleByBlockHashAndIndex`
* `eth_getUncleByBlockNumberAndIndex`

#### 2.1.2 Unsupported Compilers

The following compilers are not supported. Currently, the only language supported on Aion is Solidity-on-Aion-FVM.   

* `eth_compileLLL`
* `eth_compileSerpent`

#### 2.1.3 Mining-related Functions

The functionality represented by the following functions is implemented as part of the `stratum` API submodule.

* `eth_getWork`
* `eth_submitWork`

#### 2.1.4 Direct DB Writes and Whisper

* `db_*`
   * The database APIs have been deprecated in the Ethereum Spec.
* `shh_*`
   * Aion currently does not support the Whisper protocol

### 2.2 Implementation Deviations

The following are implementation deviations from the Spec:  

#### 2.2.1 Pending Blocks and Transactions

* `pending` status is not supported for as a [default block parameter](https://github.com/ethereum/wiki/wiki/JSON-RPC#the-default-block-parameter). Please do not send it as an input argument where a block tag is accepted as an argument.
* eth_getBlockByHash does not return any pending blocks (only mainchain blocks)
* eth_getTransactionByHash does not return any pending transactions (only transactions sealed into mainchain blocks)

#### 2.2.2 Nrg / Gas Nomenclature 

The equivalent of Gas in Aion is Nrg. To promote usage of Aion nomenclature, in addition to the `gas*` field definitions, the user can interchably read the equivalent field definitions as `nrg*`. 

For all requests that respond with a [Transaction object](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionbyhash), in addition to the `gas*` field, the following `nrg*` fields are also returned (containing the same data)

* `gas` <-> `nrg` 
* `gasPrice` <-> `nrgPrice`

For all requests that respond with a [Transaction Receipt object](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionreceipt), in addition to the `gas*` field, the following `nrg*` fields are also returned (containing the same data)

* `gasUsed` <-> `nrgUsed`

### 2.3 Deprecated Endpoints

We do not recommend the use of the following endpoints, which may me removed in a later release:

* `eth_sign`
   * This method signs an arbitrary message using the Ethereum message signature format, as defined in the Spec. In the future, an implementation of an Aion message signature will be implemented and exposed (perhaps as `aion_sign`). The `eth_sign` method **should not be used to sign Aion transactions**.
* `web3_sha3`
   * Mostly there for spec-compliance. Does not make sense to be exposed any more. **You should not be sending data to get hashed to the kernel API server**. Please consider a client-side crypto library like Sodium to hash and sign messages on the client-side since that is more efficient and secure. 

### 2.4 Filter Implementation

Filters are the only stateful component of this otherwise stateless API. The spec provides no guidance on the following topics, therefore we outline the design decisions made in the course of Aion's implementation of the Ethereum JSON RPC filter functionality: 

#### 2.4.1 Filters Expiration

All attached filters have an expiration of 5 minutes; if the filter is not polled (by calling `eth_etFilterChanges`) for 5 minutes, the filter is removed from the "installed" set (ie. the filter object will be removed from memory). 

#### 2.4.2 Maximumm Events Returned

Each filter can accumulate a maximum of 1000 events at a time (ie. maximum events queue size = 1000), before it will stop accumulating any further events. Calls to `eth_etFilterChanges` flush the events queue. If you get 1000 events returned, please consider querying for a narrower range until you get less than 1000 events returned. 

#### 2.4.3 Filters Reliability

Attached filters are not persistent across restarts (i.e. upon kernel restart, all attached filters will be erased). Furthermore, the number of filters that can be "installed" is unbounded; be cautious exposing filter functionality publically.  

### 2.5 Curl Behaviour

If you are following the Curl examples in the JSON RPC Spec, note that when using the `nano` (NanoHttpd) server vendor, you need to put `-H "Content-Type: application/json"` at the start of the call, because the --data option sets the content type to `application/x-www-form-urlencoded`.

Curl calls work with `undertow` without the json header definition `-H "Content-Type: application/json"` (optional).

## 3. JSON RPC 2.0 (Google) Spec Deviations

> This application note is current as of the RPC 2.0 Spec 2013-01-04 revision. 

Following are the details on the Aion Kernel RPC server's non-compliance with the [RPC 2.0 Specification](https://www.jsonrpc.org/specification). Quoted are the spec section numbers, followed by spec deviations (non-compliance). **Apart from the below-mentioned exceptions, the Aion RPC server in in-compliance with all other specifications defined in the aforementioned spec.**

**Spec section 4: "Request object"**

* Aion RPC server ignores the `jsonrpc` member field altogether; user can omit this field or pass in any value they choose
* Aion RPC server relaxes the condition that `id` MUST be present; user can choose not to pass in an `id` and the object will still be treated as a normal "Request".

**Spec section 4.1: "Notification"**

* Request objects without an "id" member are treated as a normal "Request" (No notion of notifications in the Aion RPC server implementation)

**Spec section 4.2: "Parameter Structures"**

* Since the Ethereum RPC API Spec only defines the parameters "by-position", there is no formal definition of the parameter structure "by-name". Most (but not all) the RPC function call implementations internally check if the structured value for params is an object or array and checks for approprate parameters by name, but since we have not published the parameter names publically, **for all intents and purposes, parameters "by-name" are not publically supported**; in the future, we may publish the parameters "by-name" as part of our API docs.

**Spec section 6: "Batch"**

* Since the Aion RPC server has no notion of notifications, all guidance pertinent to handling notifications embedded in batch requests is not applicable. To reiterate, all objects contained in a batch are treated as a "Request". 

## 4. Configurations Overview

> **Note** that you should not expose the rpc port to the public. If you are running a public-facing rpc server, we strongly recommend that you reverse-proxy the Aion Kernel's RPC server port via a "front-end" web-server like Apache or Nginx, and configure settings like CORS, SSL and Rate Limiting on there, rather than through rpc settings. The CORS, SSL and Rate Limiting settings should only be used in testing environments or environments where you don't have to worry about public-facing webserver concerns which require more feature-rich configurations for CORS, SSL and Rate Limiting than are exposed through the RPC configurations. 

The following is an overview of all the setting exposed for the RPC server. For typical use-cases, you only need to worry about setting the ip and port correctly as the defaults out of the box should be sufficient. For advanced users or users running the RPC server in a production setting, the following sections provide details on exactly what each configuration setting does and when to use it.

**Note that if any of the settings are not declared in the config file, the defaults are used to configure the RPC server.** 

The default RPC configuration, with all settings explicitly declared (**Note:** only declare the settings you wish to explicitly set and control. Declaring all the settings with their defaults will increase the verbosity of your configuration file.):  
```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    <apis-enabled>web3,eth,ops,personal,stratum</apis-enabled>

    <vendor>undertow</vendor>
    
    <cors-enabled>true</cors-enabled>
    <cors-origin>*</cors-origin>

    <ssl>
        <enabled>false</enabled>
        <cert>ssl_keystore/aion.p12</cert>
        <unsecured-pass>mypassword</unsecured-pass>
    </ssl>

    <stuck-thread-detector-enabled>true</stuck-thread-detector-enabled>

    <worker-threads>-1</worker-threads>
    <io-threads>-1</io-threads>
    <request-queue-size>-1</request-queue-size>

    <filters-enabled>true</filters-enabled>
</rpc>
```

### 4.1 Connection Settings

* `active` - enables / disables the rpc server module. 
   * *possible values*: `false`, `true` (default)
* `ip` - ip address where the http server binds on
    * *possible values*: any valid ip address string
    * *default*: `127.0.0.1`
* `port` - port where the http server binds on
    * *possible values*: any integer in range [1, 65535]
    * *default*: `8545`

### 4.2 Feature Toggles

* `<apis-enabled>` - comma-separated list of enabled api submodules
    * *possible values*: `web3`,`net`,`eth`,`ops`,`personal`,`stratum`
    * *default*: "`web3, eth, ops, personal, stratum`"
* `<vendor>` - http server implementation for rpc server
    * *possible values*: `nano`,`undertow` (default)
* `<cors-enabled>` - enable / disable cross origin requests against rpc server
    * *possible values*: `true`,`false` (default)
* `<cors-origin>` - origin string populating `Access-Control-Allow-Origin` header
    * *possible values*: `true`,`false`
    * *default*: "`*`"
* `<ssl>.<enabled>` - enables / disables ssl on rpc port
   * *possible values*: `true`, `false` (default)
* `<ssl>.<cert>` - path to ssl certificate file
   * *possible values*: any valid file path
* `<ssl>.<unsecured-pass>` - password for ssl certificate file
   * *possible values*: any valid string

### 4.3 Advanced Settings

> **Note** that in most situations, there should be no reason to set or even declare these settings in your config file. Do not set these configurations unless you know exactly what you're doing.

* `<stuck-thread-detector-enabled>` - enable / disable stuck thread detector
   * *possible values*: `false`, `true` (default)
* `<worker-threads>` - maximum number of threads allocated for handling rpc requests
   * *possible values*: any integer in range [1, 65535] (inputs out-of-range sets value to default)
   * *default*: NanoHttpd & Undertow: `Math.max(Runtime.getRuntime().availableProcessors(), 2) * 8`
* `<io-threads>` - (*Undertow* only) number of io threads allocated for handling nio message reads / writes
   * *possible values*: any integer in range [1, 65535] (inputs out-of-range sets value to default)
   * *default*: Undertow: `Math.max(Runtime.getRuntime().availableProcessors(), 2)`
* `<request-queue-size>` - (*Undertow* only) size of queue for enqueuing requests if all worker threads have been dispatched
   * *possible values*: any integer in range [1, Integer.MAX_VALUE] (inputs out-of-range sets value to default)
   * *default*: NanoHttpd & Undertow: Unbounded
* `<filters-enabled>` - enable / disable filters in rpc server
   * *possible values*: `false`, `true` (default)
* `<api-methods-enabled>` - Comma delimited list of method names to enable in addition to those configured by the groups specified in the `<apis-enabled>` section
  * _possible values_: string listing the method names to enable. For example, the string `eth_blockNumber,eth_syncing` would enable the `eth_blockNumber` and `eth_syncing` methods.
  * _default_: Empty string
  * *note*: Currently only available in the `master-pre-merge` branch 
* `<api-methods-disabled>` - Comma delimited list of method names to disable from those configured by the groups specified in the `<apis-enabled>` section
  * _possible values_: string listing the method names to disable. For example, the string `personal_unlockAccount` would disable the `personal_unlockAccount` method if it was enabled by including `personal` in the `<apis-enabled>` section.
  * _default_: Empty string
  * *note*: Currently only available in the `master-pre-merge` branch


## 5. APIs Enabled

For security, the RPC server endpoints are grouped into "APIs" (collection of related methods, which require similar priviledges to kernel state and resources). You can choose to enabled the appropriate APIs based on the security restrictions you want to impose to the consumers of the RPC server. API groups can be enabled using the `<>` setting, providing a comma separated list of APIs you wish to enable.

```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    ...
    <apis-enabled>web3,eth,ops,personal,stratum</apis-enabled>
    ...
</rpc>
```

### 5.1 API Groups Available

* *web3* - all calls that start with *web3_* in the [Ethereum JSON RPC Spec](https://github.com/ethereum/wiki/wiki/JSON-RPC)
* *net* - all calls that start with *net_* in the [Ethereum JSON RPC Spec](https://github.com/ethereum/wiki/wiki/JSON-RPC)
* *eth* - all calls that start with *eth_* in the [Ethereum JSON RPC Spec](https://github.com/ethereum/wiki/wiki/JSON-RPC)
* *personal* - limited set of the [Ethereum Management APIs](https://github.com/ethereum/go-ethereum/wiki/Management-APIs) implemented by Aion Kernel:
   * personal_unlockAccount
   * personal_listAccounts
   * personal_lockAccount
   * personal_newAccount
* *ops* - api calls to be consumed by foundation supported software like [Aion Explorer](https://dashboard.aion.network). **Do not rely on these endpoints**, since they are subject to change.
* *stratum* - api calls used by mining clients

> **Note** for more granular control of which APIs are available, see the `<api-methods-enabled>` and `<api-methods-disabled>` options in [Advanced Settings](#43-advanced-settings)

## 6. RPC Server Vendor

Currently, the Aion kernel ships with two underlying http server implementations: [NanoHttpd](https://github.com/NanoHttpd/nanohttpd) and [Undertow](http://undertow.io). To switch between implementations, you can provide the `<vendor>` option to be either "*nano*" for NanoHttpd or "*undertow*" for Undertow. 

**Note** that we recommend using **Undertow** since it's an actively maintained codebase from JBoss and is widely used in production systems.

**Note** that the NanoHttpd implementation (`nano`) is slated to be deprecated in the future.

> **Note** that in the absence of a vendor option specified, the Aion Kernel defaults to **Undertow**.

```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    ...
    <vendor>undertow</vendor>
    ...
</rpc>
```

## 7. RPC Server Performance & Resource Utilization (Advanced)

> **Note** that unless you really understand how http servers are put together or you have experience configuring web servers like Tomcat, Apache Httpd, Nginx, etc, **you should not need to touch these parameters**

We expose settings for the underlying http server that let you control various performance and throughput parameters. Also note that some of the settings are implementation-specific (ie. only apply to if the appropriate vendor is specified), due to different http server architectures.

```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    ...
    <vendor>undertow</vendor>
    <worker-threads>16</worker-threads>
    <io-threads>2</io-threads>
    <request-queue-size>4096</request-queue-size>
    ...
</rpc>
```

### 7.1 Worker Threads

The `<worker-threads>` setting controls how many http requests the rpc server can concurrently handle. For both webserver vendor implementations, as requests arrive at the server, the processing of the request is dispatched to a thread pool. The `<worker-threads>` setting sizes the thread pool responsible for handling the http requests. 

Defaults (if unset):
* *NanoHttpd* & *Undertow*: `Math.max(Runtime.getRuntime().availableProcessors(), 2) * 8`

### 7.2 IO Threads

The `<io-threads>` setting affects throughput under heavy request load. This setting only applies to `Undertow`. If you are at all familiar with Netty or Grizzly, the Undertow threading model works very similarly (since it's based on Java NIO); all incoming requests are first read on the io thread, and then dispatched off to the worker thread for processing. Only under very heavy load (hundreds-to-thousands of ~simultaneously arriving requests or very large request bodies) does this setting start affecting performance, since nominal payload-sized request reads are reasonable efficient.

Defaults (if unset):
* *Undertow*: `Math.max(Runtime.getRuntime().availableProcessors(), 2)`
* *NanoHttpd*: Not Applicable

### 7.3 Request Queue Size

> Note that unless you really know what you're doing, **we DO NOT recommend you use this setting and leave the queue size unbounded**. If you really need to be rate-limiting requests, please use a "front-end" http server which exposes a more easy-to-use interface to do rate-limiting.

**Note** that this is an *Undertow* only setting. For *nano*, requests are always unbounded. 

The `<request-queue-size>` setting affects throughput under heavy constant and bursty request loads. If a request arrives at the RPC server and no worker threads are available to handle the request, the request gets enqueued in a queue until a worker is available for dispatch. This setting lets you size this queue. In general, you should leave this queue size unbounded, unless you are running a public facing api and are concerned about DOS attacks. If you do decide to set this, set this at a fairly large number (at least 4096) to account for occasional bursty traffic. In addition, if you are using *Undertow*, enabling this setting will give you mild performance penalty, since enabling this feature adds the [RequestLimitingHandler](http://undertow.io/javadoc/1.3.x/io/undertow/server/handlers/RequestLimitingHandler.html) in the handler chain. 

**Note** that if a request arrives at the RPC server and a queue size is provided and the queue becomes full, depending on the http client's timeout setting, the caller will start getting request failures, until the queue can be cleared out to enqueue more requests. 

Defaults (if unset):
* *Undertow*: Unbounded
* *NanoHttpd*: Unbounded

## 8 RPC Server Debugging Options

A few debugging options are available to you through the Logger and the Rpc settings in the config file.

### 8.1 Logger Settings

In the logger settings block (see Section 3 for details) in the config file, the `<API>` module is set to `INFO` by default. 

```xml
<log>
    ...
    <ROOT>INFO</ROOT>
    <API>DEBUG</API>
    ...
</log>
```

By setting the `<API>` module to `DEBUG`, you can log all inbound rpc requests, with information on rpc method, parameters input and the processing time of the request. In addition, you will also get any stack traces generated as in the course of failed input data validation or response fulfillment. Following is an example of a call to `eth_getBlockByNumber`.

```
18-08-23 15:02:52.181 DEBUG API  [XNIO-1 task-24]: <request mth=[eth_getBlockByNumber] params=["latest",false]>
18-08-23 15:02:52.181 DEBUG API  [XNIO-1 task-24]: <request mth=[eth_getBlockByNumber] rpc-process time: 0.0390307ms>
```

By setting the `<API>` module to `TRACE`, you can log the above, plus get a full dump of the http request (only available with vendor = *Undertow*). Note that you also have to set the `<ROOT>` logger to at least `INFO`, since this dump is performed by the Undertow library directly onto the logback root logger (the `<ROOT>` logger ships at `WARN` by default).

```
18-08-23 15:35:10.397 INFO  io.undertow.request.dump [XNIO-1 task-27]: 
----------------------------REQUEST---------------------------
               URI=/
 characterEncoding=null
     contentLength=85
       contentType=[application/json]
            header=accept=application/json
            header=Connection=close
            header=Content-Type=application/json
            header=content-length=85
            header=host=127.0.0.1:8545
            locale=[]
            method=POST
          protocol=HTTP/1.1
       queryString=
        remoteAddr=/127.0.0.1:55848
        remoteHost=localhost
            scheme=http
              host=127.0.0.1:8545
        serverPort=8545
          isSecure=false
--------------------------RESPONSE--------------------------
     contentLength=4269
       contentType=application/json
            header=Access-Control-Allow-Headers=origin,accept,content-type
            header=Date=Thu, 23 Aug 2018 19:35:10 GMT
            header=Connection=close
            header=Access-Control-Allow-Origin=*
            header=Access-Control-Allow-Credentials=true
            header=Content-Type=application/json
            header=Content-Length=4269
            header=Access-Control-Max-Age=86400
            header=Access-Control-Allow-Methods=POST,OPTIONS
            status=200

==============================================================
```

### 8.2 Stuck Thread Detector (RPC Config Setting)

> **Note** that by default, this feature is enabled to faciliate debugging of edge-case concurrency bugs that may manifest themselves in the future in the API server implementation. Please don't disable it unless you're running a heavily loaded server and can't tolerate the negligible overhead introduced by this feature. 

```xml
<log>
    ...
    <ROOT>WARN</ROOT>
    ...
</log>
<rpc active="true" ip="127.0.0.1" port="8545">
    ...
    <vendor>undertow</vendor>
    <stuck-thread-detector-enabled>true</stuck-thread-detector-enabled>
    ...
</rpc>
```

Similar to Tomcat's [Stuck Thread Detection Valve](https://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/valves/StuckThreadDetectionValve.html), our RPC server exposes a `<stuck-thread-detector-enabled>` setting (only available with vendor = *Undertow*). This generates a log message if any RPC request takes more than 600s (10 min) to respond, which implies an RPC request has been stuck, most likely due to a concurrency bug in the Aion RPC server implementation. **Note** that the `<ROOT>` logger in the `<log>` setting should be set to at least `WARN` (default) to see the stuck thread messages. 

```
18-08-23 16:07:52.694 WARN  io.undertow.request [XNIO-1 I/O-2]: UT005072: Thread XNIO-1 task-24 (id=105) has been active for 600876 milliseconds (since Thu Aug 23 16:07:50 EDT 2018) to serve the same request for / and may be stuck (configured threshold for this StuckThreadDetectionValve is 600 seconds). There is/are 1 thread(s) in total that are monitored by this Valve and may be stuck.
...
18-08-23 16:07:53.697 WARN  io.undertow.request [XNIO-1 I/O-2]: UT005073: Thread XNIO-1 task-24 (id=105) was previously reported to be stuck but has completed. It was active for approximately 120004 milliseconds. There is/are still 0 thread(s) that are monitored by this Valve and may be stuck.
```

## 9. Enabling Cross-Origin Requests 

> **Note** that you if you have enabled CORS on a "front-end" web-server like Apache or Nginx, please disable it here, since you will get duplicate CORS headers, which will break your CORS requests from the browser. 

```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    ...
    <cors-enabled>true</cors-enabled>
    <cors-origin>*</cors-origin>
    ...
</rpc>
```

If you are sending http requests to the RPC server from a browser, then you need to enable Cross Origin requests ([CORS](https://www.w3.org/TR/cors/)) via the `<cors-enabled>` setting in the `<rpc>` settings. 

In addition, if you need to specify an exact origin (instead of the default "`*`"), you may do so using the `<cors-origin>` setting. Please note that the value you provide in the `<cors-origin>` will be transparently passed into `Access-Control-Allow-Origin` header value and therefore for any value other than "`*`", the value you set needs to be an exact match to the origin you want to allow requestes from ([Web Origin Spec](https://tools.ietf.org/html/draft-abarth-origin-09)).

**Note** that you do not need to set the `<cors-origin>` setting if you want to enable all origins, since enabling cors defaults to allowing all origins ("`*`").

## 10. Advanced Configurations

> **Note** that unless you know exactly what you're doing, please don't disable filters, since client libraries like web3.js depend on filters to detect state changes.   

We expose an option to disable all filter implementation in the RPC server using the `<filters-enabled>` option. Filters are the only stateful part of an otherwise stateless API definition. Filters allow a user to make the RPC server subscribe to events such as new block & new pending transaction, and then the user can poll for filter changes to see a delta between polls. See the [Ethereum JSON RPC Spec](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getfilterchanges) for details.

Please note that disabling filters will cause all filter related method calls to return in error (`RpcError.NOT_ALLOWED`).

Disable filters saves you on a little bit of processing if you are in a resource constrained environment, since the RPC server no longer has to process every new block and pending transaction. Please only disable filters if you are not using a client library to call the JSON RPC api.

```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    ...
    <filters-enabled>true</filters-enabled>
    ...
</rpc>
```

## 11. RPC Server over HTTPS (Secured via SSL)

To secure the rpc server with SSL, you will first need a valid SSL certificate, stored in a java keystore format (pkcs12 or jks). To secure the rpc server port, simply add an `ssl` block in the `<rpc>` section of `config.xml`, with properties `<enabled>` set to true and `<cert>` set to the full path of yor ssl certificate. Note that the rpc port is served over http by default if SSL is not explicitly enabled. Furthermore, the rpc server will not start if you have `<enabled>` set to **true**, but fail to provide a valid ssl certificate.  

Java keystores require a password for applications to unlock and use the certificate. We provide two options to input the password for the keystore. The recommended way to provide the keystore password is to input it when prompted on the interactive console when you  start-up the kernel, if the `<unsecured-pass>` option is not set in the config file. If you are running the kernel non-interactively or can accept risks of putting the keystore password in a file (ie. test environments, etc.), then you can set the `<unsecured-pass>` setting with the keystore password and you won't be prompted to enter the password interactively. 

> **Note** that when ssl is enabled instead of the rpc server endpoint being available over HTTP (ex. `http:localhost:8545`), it will instead be available over HTTPS (ex. `https:localhost:8545`)

```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    ...
    <ssl>
        <!--toggle ssl on/off (if you to access json-rpc over https)-->
        <enabled>true</enabled>

        <!--path to jks or pkcs12 ssl certificate-->
        <cert>ssl_keystore/identity.jks</cert>

        <!--unsecured password to unlock java keystore file-->
        <unsecured-pass>mypassword</unsecured-pass>
    </ssl>
    ...
</rpc>
```

### 11.1 Generating Self Signed Certificates

The kernel provides a utility to generate your own self-signed SSL certificate: typing in the command: `./aion.sh -s create` and follow the prompts. This will create a certificate that is valid for the host `localhost` and the IP address `127.0.0.1`. To customize the host and IP address of the certificate, instead use the command `./aion.sh -s create <hostname> <ip>` and follow the prompts.

Once this step is complete, a new directory called `ssl_keystore` will be created if not already exists; this is where your new self-signed certificate will be found. 

> **Note** that in order to use the kernel self-signed ssl certificate generation utility, you will first need to install [OpenSSL](https://www.openssl.org/source) version 1.1.1 or above. Older versions may work but they have not been tested and there is no guarantee.












