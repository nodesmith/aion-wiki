## Configuration

The database configuration can be set from the **config** folder by editing the **config.xml** file.

If you do not have a configuration file, first open a terminal and execute:
```bash
./aion.sh -c
```
Note that you may need to add the [seed nodes](https://github.com/aionnetwork/aion/wiki/Aion-Seed-nodes) to the file before starting the kernel.
The current default database configuration is:
```xml
<db>
    <path>database</path>
    <vendor>leveldb</vendor>
</db>
```
---
1. The **path** tag is used for setting the physical location on disk where data will be stored.

The path is relative with respect to the current folder. 

> For example if you set the path to `<path>database</path>`, it will create a new folder called **database** where it will store the data. If the path is set to `<path>database/db1</path>`, it will create two new folders called **database** inside the current folder and **db1** inside the **database** folder. The data will be stored in **db1**.

When setting the path for the database please make sure that:

* Your application has writing privileges on that path.
* There is no other database (using a different vendor) already using the path.
---
2. The **vendor** tag is used for choosing the database driver/implementation.

The current vendor options are:
* **leveldb**
* **h2**
* **rocksdb**

If you are changing vendors, you must either:
* delete/move the previous database folder 

or
* changes the path of the database to a new valid location. 

### Additional configuration options
```xml
<db>
    <path>database</path>
    <vendor>leveldb</vendor>
    <enable_db_cache>false</enable_db_cache>
    <enable_db_compression>false</enable_db_compression>
    <cache_size>128mB</cache_size>
    <block_size>16mB</block_size>
    <max_fd_alloc_size>1024</max_fd_alloc_size>
    <write_buffer_size>64mB</write_buffer_size>
    <read_buffer_size>64mB</read_buffer_size>
</db>
```
The **enable_db_cache** and **enable_db_compression** tags customize the behavior of the database to use or not use its internal caching and compression implementations. The remaining tags customize different database options.

---
## Contents

The Aion blockchain data is stored in several folders each representing a key-value database:
- `block` and `index` store the information about mined blocks
- `details` and `storage` keep information on contract details and data storage
- `state` is used for the state trie
- `transaction` keeps the data related to transactions

When the program is started it will print out its topmost known block number. In the case of an empty database, this will be the genesis block with a message like:

```
INFO GEN - loaded genesis block <num=0, root=4f512c6f...>
```

For a database that has some stored data, the message will look something like the one below:

```
INFO GEN - loaded block <num=33, root=a9182787... l=32>
```

To check that data was correctly written to the database stop the program (Ctrl+C) and restart it (./aion.sh). Compare the starting block number between the two runs. If any blocks were mined or synchronized while the program was running, the block number at the start of the program should reflect this change.

---
## Recovery

Aion already has some built-in methods for attempting to recover from corrupted data inside the database. If the logs show that database recovery is in progress, please allow the program to complete the recovery process.

If the recovery process failed, or you prefer to restart from genesis, do the following:
1. Delete the database folder.
2. Restart the program. It will automatically sync with the network and populate the database.

---
## Submitting Issues

If you can replicate the issue you experienced please enable the debugging feature of the database by editing the **config.xml** file. Between the **log** tags update (or add) the following line
`<DB>DEBUG</DB>`. After enabling the debug messages, run the program again and add the log information to the issue description. 

If you cannot replicate the issue please submit the old log information.

When possible you can also add a copy of the corrupted database to the issue description.