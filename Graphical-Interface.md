_**Note: This documentation is intended for Aion release v0.2.9 and onward**_
***

# Table of Contents
- [Overview](#overview)  
- [Known issues & Limitations](#known-issues--limitations)  
- [Set up](#set-up)  
- [Launch GUI](#launch-gui)  
- [Kernel Control](#kernel-control)  
  - [Launch/Terminate Kernel](#launchterminate-kernel)  
  - [Kernel Configuration](#kernel-configuration)  
- [Account Management](#account-management)  
  - [Add a New Account](#add-a-new-account)  
    - [Recover wallet](#recover-wallet)  
    - [Create new account](#create-new-account)  
  - [Import account](#import-account)  
    - [Import with Keystore File](#import-with-keystore-file)  
    - [Import with Private Key](#import-with-private-key)  
  - [Export Account](#export-account)  
- [Transactions](#transactions)  
  - [Send Aion](#send-aion)  
  - [Receive Aion](#receive-aion)  

# Overview

Aion core includes a graphical interface, which facilitates kernel management and provides basic wallet functionality.     In Aion v0.2.9, the GUI's first release is available.  As such, there are some [Known issues & Limitations](#known-issues--limitations); please refer to that section for details.

To run the GUI, first download and extract the Aion kernel as per the _Getting the Kernel_ step in the [Kernel section of the Aion Owner's Manual](https://github.com/aionnetwork/aion/wiki/Aion-Owner's-Manual#kernel).  Then, from the `aion` folder, run the following command:

```
./aion_gui.sh
```

# Known issues & Limitations

* Abnormal kernel termination (i.e. killed or crash) not gracefully handled
  * Issue: if a kernel process is launched by the GUI and then terminated by some mechanism other than the GUI, the GUI may not notice this termination and enter an inconsistent state.  
  * Resolution: Exit GUI, delete file `$HOME/.aion/kernel-pid`, then re-launch GUI
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
- Verify that in your config.xml (normally located at `config/config.xml` under the aion root directory) file, ensure Java API is enabled; i.e.:

```xml
<java active="true" ip="127.0.0.1" port="8547"/>
```

# Launch GUI

From the `aion` directory, run the following command in a terminal to launch the GUI:

`./aion.sh`

This window should open shortly:

[[/images/gui/dashboard1.png|Dashboard upon launch]]

# Kernel Control

## Launch/Terminate Kernel

The GUI can be used to launch an instance of the kernel by clicking _Launch kernel_ in the dashboard screen shown above.  After a kernel has been launched, it can be terminated by clicking _Terminate kernel_, as shown in the figure below.  The status of the kernel is indicated in the bottom-right corner.  There are four possible value:

- NOT RUNNING: Kernel is not running
- CONNECTING: Kernel is running; GUI has not yet connected to it, but a connection is in progress.
- CONNECTED: Kernel is running and GUI is connected to it.
- DISCONNECTED: Kernel is running; GUI is not connected to it.  This is the state when kernel termination is in progress, but kernel has not yet exited.

**Note: Upon kernel launch, it must run an integrity check for its database.  During this time, the GUI will be in CONNECTING state.  For large databases, this can take some time.**

The GUI can only terminate a kernel instance that it launched.  Furthermore, it is expected that a system runs only one instance of the kernel at one time; i.e. the user must ensure no other instance of the kernel is running before launching one from the GUI.

When the GUI exits, the kernel does not automatically exit.  In this case, upon re-launching the GUI, it will remember the instance that it had previously launched.

[[/images/gui/dashboard-kernel-launched.png|Dashboard with launched kernel]]

## Kernel Configuration

In the Settings screen, the GUI provides a built-in text editor for modifying the `config.xml` configuration file of the kernel.  The configuration can only be saved if the kernel is not currently running.

[[/images/gui/settings1.png|Settings page]]

Changes in the text editor are not saved until _Apply & save_ is clicked, which performs a basic XML validation before saving.

If undesirable changes were made in the text editor, the original saved state of the file can be restored by clicking `Reset`.

For details on config.xml, refer to the ([Configure your kernel section of the Installation wiki](https://github.com/aionnetwork/aion/wiki/Installation#configure-your-kernel).  Note: disregard the instructions about creating an account from the CLI, since you can do so using the GUI instead.

# Account Management

Account management is performed in the "Accounts" screen of the GUI.  Upon first usage, the wallet will not have any accounts and will look like this:

[[/images/gui/accounts-no-master-acct.png|Accounts screen with no accounts]]

Once a master account is created (see subsequent sections), the Accounts screen has an unlock button.  Click the Unlock button to unlock wallet.  After one minute of inactivity, the wallet automatically locks.

[[/images/gui/accounts-with-master-acct.png|Accounts screen with an account]]

## Add a New Account

There are two options to initialize your wallet by adding an account:

- [Recover previous wallet](#recover-wallet) - if you have an existing wallet from the Aion GUI with the corresponding mnemonic and password
- [Create new account](#create-new-account) - new Aion GUI user

### Recover wallet

1. Click "Add Account" from the Accounts screen
1. Under "Recover from seed," input mnemonic
1. Input corresponding wallet password
1. Click "Recover"

### Create new account

1. Click "Add Account" from the Accounts screen
1. Under "Create account," input an account name (this can be edited later)
1. Input a password
1. Confirm chosen password
1. Save and backup the seed mnemonic that appears - you will need it if you wish to recover your wallet later

**Note: Clicking on "Add account" after creating the first account will automatically generate an account in your wallet.  These accounts cannot be removed from the wallet.**

[[/images/gui/accounts-mnemonic-popup.png|Mnemonic popup]]

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

## Import with Private Key

1. Input your private key (tutorial to obtain private key [here](https://aion.readme.io/v1.0/docs/using-aion-web3-console#section-obtain-private-key))
1.  Create a password to use to unlock the account (Note: input this password correctly, currently there is no way to change this password)

[[/images/gui/accounts-import-privkey.png|Import private key popup]]

## Export Account

You may wish to save your accounts created on the Aion Wallet elsewhere. In this case, you will need to export the wallet (using the icon right of the accounts listing) and save the keystore file in your desired location. Note that the password you input here will be the new password to access the keystore file:

[[/images/gui/accounts-export.png|Export keystore popup]]

# Transactions

## Send Aion

You have the option to transact AION and send it to another wallet. Note that this wallet must accept native AION coins.

1. Make sure the account you wish to send AION from is unlocked under the "Accounts" listing (click on the lock icon to unlock an account)
1. Navigate to the "Send" option in the GUI, and verify your account information on the left
1. On the right panel, input the address you are sending to, and the amount to send in number of AION
1. Click on "Generate transaction" to send the AION, and you will be notified when the transaction finishes

[[/images/gui/send1.png|Send transaction screen]]

## Receive Aion

Under the "Receive" tab of the GUI, you can send your public wallet address by:

- scanning the QR code to display the wallet address
- copying the address to your desktop clipboard

[[/images/gui/receive1.png|Receive transaction screen]]