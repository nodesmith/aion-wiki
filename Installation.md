## Install the Aion Kernel

1. First, download the latest Aion kernel release from the [releases page](https://github.com/aionnetwork/aion/releases) or build your own by following the instructions in the [build page](https://github.com/aionnetwork/aion/wiki/Build-your-Aion-network).

2. Next, unarchive the downloaded file by right-clicking on it and selecting `Extract Here` from the drop-down menu. 
The `aion` folder will be generated in the current folder. 
    
    Alternatively, to extract the file contents, run the following command in a terminal: 
    
    ```
    tar xvjf aion-{@version}.tar.bz2
    ```

3. Finally, navigate to the `aion` folder and configure and start the kernel.

## Launch the Kernel

The aion kernel is ready to launch as soon as the release is unpacked. **However, if you wish to used the built-in miner or connect to known peers you should first edit the configuration file as explained in the next section.**

Open a terminal and navigate to the aion directory using `cd aion`. Then start the kernel by executing: 

```
./aion.sh
```

When the kernel starts up, you should see it trying to sync with the network and importing blocks. 

**Optional:** To check which peers you are connected to, open another terminal and run the command below:

```
netstat -antp | grep java
```

Please check the [owner's manual wiki](https://github.com/aionnetwork/aion/wiki/Aion-Owner's-Manual) for further instructions on working with the kernel. 

## Configure your Kernel

The current kernel configuration can be found inside `aion/config/config.xml`.
This file is read only once when the kernel is started. Any updates made in the `config.xml` file while the kernel is running will not take effect until you stop the kernel (`Ctrl+C`) and restart it (`./aion.sh`).

The following are some frequent use cases when the configurations should be modified. Peruse the [wiki](https://github.com/aionnetwork/aion/wiki) for other configuration settings.

1. **Mining**: To receive tokens for mining blocks, you first need to create an account using:
    
```
./aion.sh -a create
```

The [mining wiki](https://github.com/aionnetwork/aion/wiki/Internal-Miner) illustrates how to set this account to be able to receive tokens for mining.

To import accounts created using a different binary on the same network follow the tutorial on [exporting and importing accounts](https://github.com/aionnetwork/aion/wiki/Importing-Accounts).

2. **Adding known peers**: The default configuration from the release contains the seed nodes by default. Do not remove these nodes from the configuration. To include additional peers (e.g. friends that are also connected to the network) update the `config.xml` file by adding nodes using the **permanent peer id**, IP and port of the computers you wish to connect to:
    
```xml
<net>
    <p2p>
        <ip>0.0.0.0</ip>
        <port>30303</port>
    </p2p>
    <nodes>
        <node>p2p://PEER_ID@IP:PORT</node>
    </nodes>
</net>
```
    
> **Note:** Make sure to keep your configuration IP at **127.0.0.1** to connect locally. To allow peers to connect to you, set it to **0.0.0.0**.

After running the kernel once, your configuration file will be updated with a permanent peer id. This id can be found at the top of the `config.xml` file.

```xml
<aion>
...
        <id>your_permanet_id</id>
...
</aion>
```

If instead of a permanent id your configuration has an id placeholder, you only need to start the kernel to be assigned a permanent id. You can share this id with peers that want to add your node to their configuration file.
During kernel run time your peer list will expand to include other active nodes. 

To view your active peers in the kernel output enable the following: (not exist after v0.2.8, this has been migrated into log system P2P module DEBUG level.)

```xml
<net>
    <p2p>
        <show-status>true</show-status>
    </p2p>
</net>
```

If you accidentally delete the seed nodes from your configuration, you can find them on the [seed nodes page](https://github.com/aionnetwork/aion/wiki/Aion-Seed-nodes). **Make sure to add the seed nodes for the network you want to connect to.**

3. **Log system settings**: 
You can set different log level in the log modules, you can select 5 type ERROR, WARN, INFO, DEBUG, TRACE as the log level. The TRACE level can get every log printout. If you don't want the log to store into your disk (just print it on the terminal), you can disable it in the <log-file> by set it to false.

```xml
<log>
        <!--Enable/Disable logback service; if disabled, output will not be logged -->
        <log-file>true</log-file>
        <!--Sets the physical location on disk where log files will be stored.-->
        <log-path>log</log-path>
        <GEN>INFO</GEN>
        <VM>ERROR</VM>
        <SYNC>INFO</SYNC>
        <CONS>INFO</CONS>
        <DB>ERROR</DB>
        <API>INFO</API>
        <P2P>INFO</P2P>
        <TX>INFO</TX>
        <TXPOOL>INFO</TXPOOL>
</log>
```
4. **Connecting via SSL**: 
To connect to the kernel via SSL, you will first need to download `OpenSSL` <a href="https://www.openssl.org/source/">here</a>, version 1.1.1 or above. Older versions may work but they have not been tested and there is no guarantee.

The next step is generating your own self-signed SSL certificate. The Aion kernel provides two easy means of generating this certificate. The first is by typing the command `./aion.sh -s create` and following the prompts. This will create a certificate that is valid for the host `localhost` and the IP address `127.0.0.1`. To customize the host and IP address of the certificate, instead use the command `./aion.sh -s create <hostname> <ip>` and follow the prompts.

Once this step is complete a new directory called `ssl_keystore` will be created and your new self-signed certificate will be placed inside it. To use this certificate the following lines must be copy and pasted into the `config.xml` file:
```xml
<ssl>
    <!--toggle ssl on/off (if you to access json-rpc over https)-->
    <enabled>true</enabled>
    <!--path to jks or pkcs12 ssl certificate-->
    <cert>ssl_keystore/certificate</cert>
</ssl>
```
`certificate` should be changed to match the file name of the certificate you wish to use inside the `ssl_keystore` directory. If you wish to store the certificate in a different directory altogether then modify the whole path appropriately. Just note that the kernel commands to create a new certificate will always create the new certificate inside the `ssl_keystore` directory. To disable the SSL connection simply set the `enabled` value to `false` instead.

The code above should be pasted inside the `rpc` tag in the `config.xml` file so that your file should look something like the following:
```xml
<rpc active="true" ip="127.0.0.1" port="8545">
    <!--boolean, enable/disable cross origin requests (browser enforced)-->
    <cors-enabled>false</cors-enabled>
    <!--comma-separated list, APIs available: web3,net,debug,personal,eth,stratum-->
    <apis-enabled>web3,eth,personal,stratum</apis-enabled>
    <ssl>
        <!--toggle ssl on/off (if you to access json-rpc over https)-->
        <enabled>true</enabled>
        <!--path to jks or pkcs12 ssl certificate-->
        <cert>ssl_keystore/certificate</cert>
    </ssl>
</rpc>
```

Now run the kernel using `./aion.sh` to connect via your new self-signed SSL certificate.