_**Work-in-progress: not yet ready for public consumption!**_
***

# Overview

Aion core includes a graphical interface, which facilitates kernel management and provides basic wallet functionality.     In Aion v0.2.9, the GUI's first release is available.  As such, there are some [Known issues & Limitations](#known-issues--limitations); please refer to that section for details.

To run the GUI, first download and extract the Aion kernel as per the _Getting the Kernel_ step in the [Kernel section of the Aion Owner's Manual](https://github.com/aionnetwork/aion/wiki/Aion-Owner's-Manual#kernel).  Then, from the `aion` folder, run the following command:

```
./aion_gui.sh
```

# Known issues & Limitations

* Abnormal kernel termination (i.e. killed or crash) not gracefully handled
  * Issue: if a kernel process is launched by the GUI and then terminated by some mechanism other than the GUI, the GUI may not notice this termination and enter an inconsistent state.  
  * Resolution: Exit GUI, delete file $HOME/.aion/kernel-pid, re-launch GUI
* Transaction history updates 
  * Issue: recently-sent transactions sometimes are slow to appear 
  * Resolution: the transaction will show up given some time; for forcing a "refresh," can exit the GUI and re-open it and unlock the master account and once the transaction history loads, it will show the complete list
* GUI slow to connect to kernel API when launching kernel with large database
  * Issue: Upon launching the kernel process, it will run an integrity check to verify the blocks in its database.  During this time, the kernel is running, but the GUI cannot yet connect to it.  Therefore, with a large database, it will stay in the "CONNECTING..." state for some time with no feedback until this check is complete.
  * Resolution: Wait for the integrity check to complete; its progress can be monitored from the log file of the kernel.
* Only Aion mainnet is supported currently
  * Issue: GUI does not support launching the kernel to run on the testnet
  * Resolution: None currently; support for other networks will be introduced in a subsequent release.

# Set up

If setting up Aion for the first time:
- Download and extract the Aion kernel as per the _Getting the Kernel_ step in the [Kernel section of the Aion Owner's Manual](https://github.com/aionnetwork/aion/wiki/Aion-Owner's-Manual#kernel).  

If you have an already-configured copy of Aion v.0.2.9+:
- Verify that in your config.xml file, ensure Java API is enabled [TODO: Put full path of mainnet config file]; i.e.:

```xml
<java active="true" ip="127.0.0.1" port="8547"></java>
```

# Usage

## Kernel Control

## Account Management

## 