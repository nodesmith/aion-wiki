
## Node

* endpoint: p2p://{uuid}@{ipv4}:{port}

## Message

#### Header 
* length    byte[8]
* includes
  1. version - byte[2]
  2. control - byte
  3. action - byte
  4. body len - byte[4]

``` 
|<-    version    ->|<- ctrl->|<- act ->|<-             body len              ->|       
---------------------------------------------------------------------------------
|    0    |    1    |    2    |    3    |    4    |    5    |    6    |    7    |  
---------------------------------------------------------------------------------
```

#### Routes

Based on decoded header we have route table. Routes to handlers is 1 to many relation.

`m - modules, v - version, c - control, a - action`
| m | v | c | a | msg | handler |
| --- | --- | --- | --- | --- | --- |
| p2p | 0 | 0 | 0 | Disconnect |  |
| | | | 1 | ReqHandshake |  |
| | | | 2 | ResHandshake |  |
| | | | 3 | Ping |  |
| | | | 4 | Pong |  |
| | | | 5 | ReqActiveNodes |  |
| | | | 6 | ResActiveNodes |  |

#### 4. rules
* outbound connections, drop if request handshake reach max 3 times
