From a terminal, you can interact with Aion though the **command line interface** which offers the following options:

```
Usage: ./aion.sh [options] [arguments]
  -h
  --help                       show help info

  -a [arguments]
  --account [arguments]
  -a create                    create a new account
  -a list                      list all existing accounts
  -a export [address]          export private key of an account
  -a import [private_key]      import private key

  -c 
  --config                     create config with default values
  -c --id=uuid                 create config with customized node id
  -c --nodes=p2p0,p2p1,..      create config with customized boot nodes
  -c --p2p=ip:port             create config with customized p2p listening address

  -i 
  --info                       show information

  -r [block_number]
  --revert [block_number]      removes from the database all blocks greater than the one given

  -v 
  --version                    show version

```