
`environment:`      
`fabric v2.2.0`


* gossip/comm ：负责 Peer 节点间的 RPC 通信
* gossip/discovery ：Fabric 网络成员发现、管理
* gossip/election ：组织内 Peer 节点选举
* gossip/gossip ：Gossip 协议实现
* gossip/identity ：节点身份标识管理
* gossip/privdata ：私有数据处理部分
* gossip/service ：封装 Gossip 功能对位不提供服务
* gossip/state ：节点参与的 Channel 状态管理、区块数据、私有数据与本地账本的对接


参考:   
[fabric gossip 源码解析](https://www.jianshu.com/p/0334bd18e882)  
[Fabric中数据同步的实现](https://www.chaindesk.cn/witbook/11/170)   
[fabric 2.0, Gossip Service](https://blog.csdn.net/m0_37889044/article/details/104917109)   
[Hyperledger Fabric源码解析 Gossip-Emitter](https://blog.csdn.net/qq_30145355/article/details/110409009)
