### CLI Options
From a terminal, you can interact with Aion though the **command line interface** which offers the following options:

```
Usage: ./aion.sh [options] [arguments]

  -h                                  show help info

  -a create                           create a new account
  -a list                             list all existing accounts
  -a export [address]                 export private key of an account
  -a import [private_key]             import private key

  -c [network]                        create config with default values to select network

  -n, --network [network]             execute kernel with selected network

  -d, --datadir [directory]           execute kernel with selected database directory

  -i                                  show information

  -s create                           create an ssl certificate for localhost
  -s create [[hostname] [ip]]         create an ssl certificate for a custom hostname and ip

  -r                                  remove blocks on side chains and correct block info
  -r [block_number]                   revert db up to specific block number

  -v                                  show version
```
### Multi-Arguments CLI

**[1] Specifying datadir (-d) && network (-n)**
```
./aion.sh -n [valid network] -d [valid datadir]

OR

./aion.sh -d [valid datadir] -n [valid network]
```
Creates new directory and copies configuration files in (config.xml & genesis.json), with subpaths;
```
/home/aion/[datadir]/[network]/config/config.xml
/home/aion/[datadir]/[network]/config/genesis.json
/home/aion/[datadir]/[network]/keystore
/home/aion/[datadir]/[network]/database
/home/aion/[datadir]/[network]/log
```
> - Invalid parameters will not execute the kernel

**[2] Generating new config file (-c)**
```
./aion.sh -c [valid network]
```
Generates new config.xml file at base directory, either one of;
```
/home/aion/config/mainnet/config.xml
/home/aion/config/conquest/config.xml
```
> - Invalid parameters will not generate new config file
> - If the config folder does't exist, the config folder and config file will be generated at base

**[3] Creating new accounts (-a)**
```
./aion.sh -a create -n [valid network] -d [valid datadir]
./aion.sh -a list -n [valid network] -d [valid datadir]
./aion.sh -a export [valid address] -n [valid network] -d [valid datadir]
./aion.sh -a import [private key] -n [valid network] -d [valid datadir]
```
Creates a new account using the specified datadir and network path. For example;
```
/home/aion/[datadir]/[network]/keystore
```
> - Invalid parameters - or no parameters - will set to default keystore folder
> - If the datadir folder doesn't exist, the keystore account will generate the new directory for it