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

# Set up

# Usage

## Kernel Control

## Account Management

## 