
### Introduction
Aion network p2p protocols define set of messages which provide foundation rules of peer to peer communication fit for aion network and development progresses. 

### <b>Node</b> <sup>[example](https://github.com/aionnetwork/aion/wiki/Aion-Seed-nodes)</sup>
* endpoint: p2p://{uuid}@{ip}:{port}

### <b>Header</b> <sup>[src](https://github.com/aionnetwork/aion/blob/dev/modP2p/src/org/aion/p2p/Header.java)</sup>
Header specific how to encode / decode message. For each protocol there would be pair of message and handler to be constructed by same header instance which indicates same version, control and action.    
* length - byte[8]
* includes
  1. version - byte[2]
     * Aion test net zero is using version 0.
  2. control - byte
  3. action - byte
  4. body len - byte[4]
     * Max body length is 400 MB per message on current setting. Any message exceeds max length should not be delivered to body read routine.   
``` 
|<-    version    ->|<- ctrl->|<- act ->|<-             body len              ->|       
---------------------------------------------------------------------------------
|    0    |    1    |    2    |    3    |    4    |    5    |    6    |    7    |  
---------------------------------------------------------------------------------
```

### Message <sup>[src](https://github.com/aionnetwork/aion/blob/dev/modP2p/src/org/aion/p2p/Msg.java)</sup>
Message exchanged by peers contains 2 part, header and body. 
* header byte[8]
* body byte[]

### Route Table
route function `return (ver << 16) | (ctrl << 8) | action;`

m - module, v - version, c - control, a - action

| m | v | c | a | message | attributes | description | 
| --- | --- | --- | --- | --- | --- | --- |
| p2p | 0 | 0 | 0 | `DISCONNECT` |  | disconnect |
|     | 0 | 0 | 1 | `REQ_HANDSHAKE` | `nodeId` byte[36]<br>`version` byte[4]<br>`ip` byte[8]<br>`port` byte[4]<br>`revisionLen` byte<br>`revision` byte[] | request handshake<int>version - self supported version |
|     | 0 | 0 | 2 | `RES_HANDSHAKE` | byte  | response handshake<br>`0x01` true<br>`0x00` false |
|     | 0 | 0 | 3 | `PING` | | ping |
|     | 0 | 0 | 4 | `PONG` | | pong |
|     | 0 | 0 | 5 | `REQ_ACTIVE_NODES` | | request active nodes |
|     | 0 | 0 | 6 | `RES_ACTIVE_NODES` | `List<Node>` [src](https://github.com/aionnetwork/aion/blob/dev/modP2pImpl/src/org/aion/p2p/impl/Node.java)| response active nodes |
|     | 0 | 0 | 127 | `UNKNOWN` | | drop message |
| sync | 0 | 1 | 0 | `REQ_STATUS` | | request status |
|      | 0 | 1 | 1 | `RES_STATUS` | `bestBlockNumber` byte[8]<br>`totalDifficultyLen` byte<br>`totalDifficulty` byte[]<br>`bestHash` byte[32]<br>`genesisHash` byte[32]<br> | response status |
|      | 0 | 1 | 2 | `REQ_BLOCKS_HEADERS` | `fromBlock` byte[8]<br>`take` byte[4] | request blocks headers |
|      | 0 | 1 | 3 | `RES_BLOCKS_HEADERS` | `List<A0BlockHeader>` (rlp) [src](https://github.com/aionnetwork/aion/blob/dev/modAion/src/org/aion/zero/types/A0BlockHeader.java) | response blocks headers |
|      | 0 | 1 | 4 | `REQ_BLOCKS_BODIES` | `List<byte[32]>` | request blocks bodies |
|      | 0 | 1 | 5 | `RES_BLOCKS_BODIES` | `List<byte[]>` (rlp) | response blocks bodies |
|      | 0 | 1 | 6 | `BROADCAST_TX` | `List<ITransaction>` (rlp) [src](https://github.com/aionnetwork/aion/blob/dev/modAion/src/org/aion/zero/types/AionTransaction.java) | broadcast new transactions |
|      | 0 | 1 | 6 | `BROADCAST_NEWBLOCK` | `IBlock` (rlp) [src](https://github.com/aionnetwork/aion/blob/dev/modAionImpl/src/org/aion/zero/impl/types/AionBlock.java) | broadcast new block |
|      | 0 | 1 | 127 | `UNKNOWN` | | drop message | 

!! Except table cells with label '(rlp)', messages are encoded / decoded as from top to bottom in each attributes cells on table above. 

### Handshake Flow

| |node-a | node-b |
| --- | --- | --- |
|  0  | fire connection to node-b | receive connection from node-a |
|  1  | create record on outbound collection | create connection on inbound collection |
|  2  | fire request handshake to node-b | receive request handshake<br>if handshake rule passed send node-a response handshake<br>else no response till node-a get timeout from inbound collection |

### Connections and Timeout

* p2p groups 3 collections of connections: inbound, outbound, active.
  * Inbound collection stores incoming connection before handshake process
  * Outbound collection stores outgoing connections before handshake process 
  * Active collection stores connections and node info after handshake process succeed.

* Node time stamp will be refresh on RES_HANDSHAKE, RES_ACTIVE_NODES and so far sync protocols. Connections will be closed if records expired
  


