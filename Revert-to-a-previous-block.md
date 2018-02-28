**Version 0.1.12** of the kernel offers the command line option to revert to a previous block. This functionality is meant for removing from the database all blocks greater than the one given as parameter. **The revert feature makes permanent changes to the database. If you are unsure about using it, back-up the database first.**

```
Usage: 
   ./aion.sh -r [block_number]
or
   ./aion.sh --revert [block_number]
```

### When to revert to a previous block?

#### Case 1: Failed database recovery
In case of data corruption, the kernel makes a few attempts at recovering the corrupted data.
When in recovery mode, it will display messages informing the user of the recovery status, as follows:
```
INFO  CONS [main]: Corrupt world state at block #1100. Rebuilding from block #1000.
INFO  CONS [main]: Rebuilding block #1001.
INFO  CONS [main]: Rebuilding block #1002.
INFO  CONS [main]: Rebuilding block #1003.
...
```
If these recovery attempts fail, you can stop the kernel and attempt reverting to the block that the rebuild started from, in the example above block 1000. The reason the recovery was started from this block is that the database was able to find a correct world state for that block, i.e. uncorrupted data.

#### Case 2: Stuck on sidechain

If your Aion kernel is running, but not sync-ing to the latest block from the network, there's a high likelihood that you are stuck on a side chain. You can verify this by enabling the DEBUG messages for SYNC in your `config.xml` file (to use the new configuration you need to restart the kernel after updating the config file).
If you see only messages with the error **no-parent** when trying to import blocks (like the ones below), it means that the kernel cannot find a common block between your chain and the network chain.
```
DEBUG SYNC [sync-import]: <try-import-block num=4881 hash=031ec0 parent=c42763>
DEBUG SYNC [sync-import]: <import-unsuccess err=no-parent num=4881 hash=031ec0>
DEBUG SYNC [sync-import]: <try-import-block num=4882 hash=885103 parent=031ec0>
DEBUG SYNC [sync-import]: <import-unsuccess err=no-parent num=4882 hash=885103>
...
DEBUG SYNC [sync-import]: <try-import-block num=5027 hash=51ed76 parent=b33346>
DEBUG SYNC [sync-import]: <import-unsuccess err=no-parent num=5027 hash=51ed76>
DEBUG SYNC [sync-import]: <try-import-block num=5028 hash=61b72e parent=51ed76>
DEBUG SYNC [sync-import]: <import-unsuccess err=no-parent num=5028 hash=61b72e>
```
In such cases, the common block between the two chains may be outside of the scope of the blocks considered by the sync manager. The revert feature is useful here because it allows you to adjust the scope, by moving to a block with a lower number. So in cases where you are stuck on a sidechain, select from the error messages the lowest block number for which you get a **no-parent** error (in this case 4881) and revert to it by calling:
```
./aion -r 4881
```
Restart the kernel to see if the new scope allows you to sync to the main chain. If not repeat the process by reverting to the new lowest block number without a parent block.
