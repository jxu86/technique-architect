









以太坊的节点发现基于类似的kademlia算法，源码中有两个版本，v4和v5。v4适用于全节点，通过discover.ListenUDP使用，v5适用于轻节点通过discv5.ListenUDP使用.





参考:   
[死磕以太坊源码分析之Kademlia算法](https://www.cnblogs.com/1314xf/p/14019453.html)   
[死磕以太坊源码分析之p2p节点发现](https://www.cnblogs.com/1314xf/p/14027186.html)       
[[以太坊源码分析][p2p网络04]：基于UDP的节点发现](https://www.jianshu.com/p/b232c870dcd2)     
[以太坊源码深入分析（6）-- 以太坊P2P协议接收广播的处理和Fetcher源码分析](https://www.jianshu.com/p/97289dbc469e)
