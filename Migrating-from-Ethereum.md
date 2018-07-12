Resources for migrating from Ethereum:
1. Video tutorial on [Application Development on Aion](https://www.youtube.com/watch?v=WbFpVf6R1pc&list=PL7eVtcQCTlGM7_o7G5pvpUciB0zSfe8CD&index=6).
2. Video tutorial on [Migrating Your DApp](https://www.youtube.com/watch?v=mBD6AqyXhJs&list=PL7eVtcQCTlGM7_o7G5pvpUciB0zSfe8CD&index=7).
3.  [Code examples](https://github.com/aionnetwork/aion/wiki/Application-Development-On-Aion) for frequently used functionality.

The following table shows a list of differences between Aion and Ethereum that should be taken into consideration when migrating already implemented smart contracts from Ethereum to Aion:

|  | Ethereum | Aion |
|--|--|--|
| data word | 256 bits | 128 bits |
| local variable count | 16 data word (of 256 bits) | 16 data word (of 128 bits) |
| int size | int8 – int256 | int8 – int128 |
| uint size | uint8 – uint256 | uint8 – uint128 |
| inline assembly | supported | not supported (currently) |
| address | 20 bytes | 32 bytes |
| hash function (signatures & wallet) | Keccak-256 | Blake2b |
| signature function | ECDSA – curve secp256k1 | ECDSA - curve ED25519 |
| compilers | Solidity, LLL, Serpent | Solidity |
