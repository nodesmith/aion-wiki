### CLI Options
From a terminal, you can interact with Aion though the **command line interface** which offers the following options:

```
Usage: ./aion.sh [options] [arguments]

  -h                                            show help info

  -a create                                     create a new account
  -a list                                       list all existing accounts
  -a export [address]                           export private key of an account
  -a import [private_key]                       import private key

  -c [network]                                  create config with default values to select network

  -n, --network [network]                       execute kernel with selected network

  -d, --datadir [directory]                     execute kernel with selected database directory

  -i                                            show information

  -s create                                     create an ssl certificate for localhost
  -s create [[hostname] [ip]]                   create an ssl certificate for a custom hostname and ip

  -r                                            remove blocks on side chains and correct block info
  -r [block_number]                             revert db up to specific block number

  -v                                            show version
```
### Multi-Arguments CLI
Only the --network and --datadir option supports **multi-argument** CLI options:
```
./aion.sh --network conquest --datadir database_conquest
```
(The -n and -d option can be used instead and the order of the option tags do not matter)
> Current available networks:
> - mainnet
> - conquest