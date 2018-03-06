
## Node

* endpoint: p2p://{uuid}@{ipv4}:{port}

## Message

#### Header 

* length    byte[8]
* includes
  1. version - byte[2]
     * current version is 0 for test net 0
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

Header encode / decode design is 

Based on decoded header we have route table. Routes to handlers is 1 to many relation.
route function<br>
` return (ver << 16) | (ctrl << 8) | action; `


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

## Development

P2p module (interface & implementation) is built on top and standard JDK9 NIO package. The network layer is a high-performance, dependency free, low level networking stack built from the ground up to work with the AION-0 P2P protocol. The breadth of control we exercise over the library will be used to expand the features of the library as our P2P protocol progresses.
