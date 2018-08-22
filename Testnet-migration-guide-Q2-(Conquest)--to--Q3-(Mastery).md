**This migration requires resetting the database. Please backup the `database` folder if you still want to test your data on the Q2 testnet (Conquest)**

Before the migration, you should backup the `keystore` and `config` folders from the aion kernel for the Conquest network.

1. Download the release build for the Mastery testnet from the releases page. [See here](https://github.com/aionnetwork/aion/releases/tag/v0.3.0.q)
2. Extract the files and move them to the path where you want to execute the aion kernel.
3. If you changed the **config.xml** from the `config` folder in the aion kernel for the Conquest network, please copy the changes to the config.xml file in the config folder in the aion kernel for the Mastery network. 
4. Remove the **\<threads\>1\</threads\>** inside the config.xml file if you are using rpc as your client API connection.
5. Copy the `keystore` folder from the aion kernel folder of the Conquest network to the aion kernel folder for the Mastery network.
6. If your configuration settings are broken and you do not know how to correct them, you can remove the config.xml file and execute `./aion.sh -c` to get the default config settings, or copy the contents of [this file](https://github.com/aionnetwork/aion/blob/testnet_q3_mastery/modBoot/resource/config.xml). Make sure you have the seed nodes from the linked file in your configuration before starting the kernel.
7. The dashbard for the Mastery testnet is [here](https://mastery.aion.network/#/dashboard).
