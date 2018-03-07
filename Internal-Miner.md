## Internal Mining Guide

Internal mining is configured by modifying the `config.xml` file located within `[aion_folder]/config`.
An example configuration enabling internal mining is shown below:

```xml
<consensus>
    <mining>true</mining>
    <miner-address>0000000000000000000000000000000000000000000000000000000000000111</miner-address>
    <cpu-mine-threads>2</cpu-mine-threads>
    <extra-data>MyAion</extra-data>
</consensus>
```
In order to ensure the kernel is mining the following fields must be configured:
* **mining**: _true_ to enable mining, _false_ to disable it
* **miner-address**: The wallet address that will collect AION for mining blocks

    To create a new wallet, run `./aion.sh -a create`. To import an existing wallet, please refer to [Importing Accounts](https://github.com/aionnetwork/aion/wiki/Importing-Accounts).
* **cpu-mine-threads**: 1 OR up to 75% of your max CPU logical cores

    The number of logical cores may be seen in either task manager (Windows) or system monitor (Ubuntu). It is not recommended to go above 75% of the total logical cores.

Optional field:
* **extraData**: any hex string up to 32 bytes

Once the config file has been setup the kernel should pick up the mining settings next time it is started.

**Mining is normally delayed 10s to allow time for the kernel to fully start.**
