From a terminal, you can interact with Aion though the **command line interface** which offers the following options:

```
Usage: ./aion.sh [options] [arguments]

  -h                           show help info
  -a create                    create a new account
  -a list                      list all existing accounts
  -a export [address]          export private key of an account
  -a import [private_key]      import private key
  -c                           create config with default values
  -c --id=uuid                 create config with customized node id
  -c --nodes=p2p0,p2p1,..      create config with customized boot nodes
  -c --p2p=ip:port             create config with customized p2p listening address
  -i                           show information
  -v                           show version
```