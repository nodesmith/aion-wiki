## Genesis Parameters

The genesis file provided as part of the [Aion kernel](https://github.com/aionnetwork/aion) installation bundle was modified specifically for the **test network**.

The complete file can be found in the `[aion_folder]/config/genesis.json`.

An abbreviated version of the genesis file is shown below, followed by details regarding each parameter and its meaning.

```json
{
  "alloc": {
    "0x8f30ce8eb81c57388bc25820b0f8d0612451c9f90091224028b9c562fc9c7036": {
      "balance": "314159000000000000000000"
    },
    ...
  },
  "networkBalanceAlloc": {
    "0": {
      "balance": "465934586660000000000000000"
    }
  },
  "energyLimit": "10000000",
  "nonce": "0x00",
  "difficulty": "0x64",
  "coinbase": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "1497536993",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "extraData": "0x386b1d6a6391fb17421315996c5ee8d5798cc96f8fb4afb8e1e5230cc911fb08"
}
```

### alloc

The first parameter describes account allocation. These are accounts that are pre-allocated with balances. For the purposes of the test network, 10 accounts have been pre-allocated with a balance of ``314159 AION`` each.

### networkBalanceAlloc

This field is intended to be used to record the total balances for multiple networks. The value can then be dynamically updated and used to calculate rewards. Currently, this value is unused.

### energyLimit

Each transaction in each block requires a dynamic amount of ``energy``. ``energyLimit`` is the upper bound that limits the number of transactions included in a block.

### nonce

This field indicates the value in which the ``PoW`` algorithm yields a valid solution. It is set arbitrarily to ``0x00`` for the genesis block.

### difficulty

The difficulty field allows the network to determine the block rate. It is initialized with a low value `0x64` so that the test network can find its natural difficulty level.

### coinbase

This field indicates the address of the account that successfully sealed the block. It is arbitrarily set to the ``0th`` address here.

### timestamp

The timestamp field is an arbitrarily set field. In our case, this timestamp corresponds to the first timestamp of the first commit in the Aion whitepaper repository.

### parentHash

This field indicates the hash of the parent (preceding) block that this block is referencing. For the purposes of the genesis, it is arbitrarily set to an all-zero hash.

###  extraData

An arbitrary field that may be used to set messages. Typically this ends up being the signature of some participating mining pool. With respect to the test network, this field is set to be the hash of a special message.


