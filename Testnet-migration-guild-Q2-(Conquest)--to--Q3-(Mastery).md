**This migration requires to reset the database. Please backup the database if you still want to test your data on the Q2 testnet (Conquest)**

Before the migrate, you should backup the keystore folder, config folder in the aion kernel for the Conquest.

1. Download the release build for the Mastery testnet on the release page. [See here](https://github.com/aionnetwork/aion/releases/tag/v0.3.0.q)
2. Extract the file and move to the path you want to execute the aion kernel.
3. If you changed the **config.xml** in the config folder in the aion kernel for the Conquest. Please copy the changes to the config.xml in the config folder in the aion kernel for the Mastery. 
4. Remove the **\<threads\>1\</threads\>** in side the config.xml if you are using the rpc as your client API connection. 
5. copy the keystore folder inside the aion kernel folder of the Conquest to the aion kernel folder for the Mastery.
6. If you massed up your config settings. You can remove the config.xml and execute ./aion.sh -c to get the default config settings or find the files in [here](https://github.com/aionnetwork/aion/blob/testnet_q3_mastery/modBoot/resource/config.xml)

7. the dashbard of the Mastery testnet is [here](https://mastery.aion.network/#/dashboard)