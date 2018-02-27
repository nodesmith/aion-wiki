## Welcome to the Aion Wiki!

This wiki is a guide to using the Aion blockchain network.

## Repository layout

The Aion implementation is distributed across multiple modules to allow for code reuse:
* The [**Aion Core**](https://github.com/aionnetwork/aion) contains the main functionality of the Aion network and can be downloaded from the [releases page](https://github.com/aionnetwork/aion/releases).
* The [**Aion FastVM**](https://github.com/aionnetwork/aion_fastvm) is an enhanced Ethereum Virtual Machine provided as a separate module.
* The [**Aion Miner**](https://github.com/aionnetwork/aion_miner) repository contains code and documentation for mining blocks on the Aion blockchain.
* The [**Aion Interchain**](https://github.com/aionnetwork/aion_interchain) repository will provide information regarding our bridging protocol.
* The [**Aion Compatible Web3 Api**](https://github.com/aionnetwork/aion_web3) repository provides a Web3 application programming interface for interacting with the Aion blockchain.
* The [**Aion Java Api**](https://github.com/aionnetwork/aion_api) repository provides a java programming interface for interacting with the Aion blockchain.



## Modules

The Aion Core is jdk9 module based implementation:
* The [**modMcf**] generic multi-chain framework as fundamental blockchain building blocks.
* The [**modAion modAionImpl**] aion core interface and aion zero implementation (POW) .
* The [**modAionBase**] aion core common library.
* The [**modApiServer**] aion core api daemon for aion binary api protocol and web3 protocol client.
* The [**modBoot**] the boostrap lib for aion core.
* The [**modCrypto**] the crypto library, include blake2b, sha, edcsa, ed25519 etc.
* The [**modDbImpl**] blockchain key value database implementation, support level db and h2.
* The [**modEvtMgr modEvtMgrImpl**] event support framework for blockchain kernel.
* The [**modP2p modP2pImpl**] peer to peer library support aion wire protocol.
* The [**modTxpool modTxpoolImpl**] blockchain transaction pool interface and implementation.
* The [**modRlp**] RLP implemention , which is serialization solution for aion core..
* The [**modLogger**] logger module.



## Resources

The Aion installation guide is available in the repository [README.md](https://github.com/aionnetwork/aion/blob/master/README.md) file.

This wiki also contains documentation on:
* [Genesis Block](https://github.com/aionnetwork/aion/wiki/Genesis-Block) - description of the network genesis block
* [Database](https://github.com/aionnetwork/aion/wiki/Database) - storage configuration options
* [Internal Miner](https://github.com/aionnetwork/aion/wiki/Internal-Miner) - enabling and disabling the internal miner and collecting AION
* [Command Line Interface](https://github.com/aionnetwork/aion/wiki/Command-Line-Interface) - using the command line options to interact with the network