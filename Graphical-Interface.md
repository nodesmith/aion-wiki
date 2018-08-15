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
* Transaction history updates may be slow 
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
- You should now have a directory named `aion` containing the Aion kernel and GUI

If you have an already-configured copy of Aion v.0.2.9+:
- Verify that in your config.xml file, ensure Java API is enabled [TODO: Put full path of mainnet config file]; i.e.:

```xml
<java active="true" ip="127.0.0.1" port="8547"></java>
```

# Launch GUI

From the `aion` directory, run the following command in a terminal to launch the GUI:

`./aion.sh`

This window should open shortly:

[[/images/gui/dashboard1.png|Dashboard upon launch]]

# Kernel Control

# Account Management

Account management is performed in the "Accounts" screen of the GUI.  Upon first usage, the wallet will not have any accounts and will look like this:

[[/images/gui/accounts-no-master-acct.png|Accounts screen with no accounts]]

## Add a New Account

There are two options to initialize your wallet by adding an account:

- [Recover previous wallet](#recover-wallet) - if you have an existing wallet from the Aion GUI with the corresponding mnemonic and password
- [Create new account](#create-new-account) - new Aion GUI user

### Recover wallet

- Click "Add Account" from the Accounts screen
- Under "Recover from seed," input mnemonic
- Input corresponding wallet password
- Click "Recover"

### Create new account

- Click "Add Account" from the Accounts screen
- Under "Create account," input an account name (this can be edited later)
- Input a password
- Confirm chosen password
- Save and backup the seed mnemonic that appears - you will need it if you wish to recover your wallet later:

[[/images/gui/accounts-mnemonic-popup.png|Mnemonic popup]]

- **Note: Clicking on "Add account" after creating the first account will automatically generate an account in your wallet.  These accounts cannot be removed from the wallet.**

## Import account

There are two options to import an existing account:

- Import using a keystore file and password
- Import using a private key (you will be asked to _create_ a password for it)

**Remember Me**
- There is a "Remember Me" option when importing accounts. Selecting this will display your imported accounts even if you re-launch your wallet. If this option is not selected, you will have to reimport these accounts if you re-launch the Aion Desktop Wallet.

### Import with Keystore File

1. Click on the keystore space, and navigate to select your desired Keystore UTC File
1. Input corresponding keystore password:

[[/images/gui/accounts-import-keystore.png|Import keystore popup]]

Import with Private Key

1. Input your private key (tutorial to obtain private key here)
1.  Create a password to use to unlock the account (Note: input this password correctly, currently there is no way to change this password)

[[/images/gui/accounts-import-privkey.png|Import private key popup]]
