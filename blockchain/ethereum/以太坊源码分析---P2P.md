









以太坊的节点发现基于类似的kademlia算法，源码中有两个版本，v4和v5。v4适用于全节点，通过discover.ListenUDP使用，v5适用于轻节点通过discv5.ListenUDP使用.

### k-桶初始化的过程
1、先新建table对象，连接本地database，如果本地没有database，则先新建一个空的database；
2、初始化K-桶，先获得k-桶信息的源节点：    
* 通过setFallbackNodes(bootnodes)来获得5个nursey节点；  
* 通过tab.loadSeedNodes()——>tab.db.querySeeds()来从本地database获得最多30个节点；     

3、把上面的节点存入seeds，进行for循环；   
4、在循环内执行tab.add(seed)，计算seed节点与本节点的距离，选择相应距离的bucket。如果bucket不满，则用bump()存入bucket；如果bucket已满，则放入replacements。   



参考:   
[死磕以太坊源码分析之Kademlia算法](https://www.cnblogs.com/1314xf/p/14019453.html)   
[死磕以太坊源码分析之p2p节点发现](https://www.cnblogs.com/1314xf/p/14027186.html)       
[[以太坊源码分析][p2p网络04]：基于UDP的节点发现](https://www.jianshu.com/p/b232c870dcd2)     
[以太坊源码深入分析（6）-- 以太坊P2P协议接收广播的处理和Fetcher源码分析](https://www.jianshu.com/p/97289dbc469e)    
[以太坊源码解读（8）以太坊的P2P模块解析——节点发现和K-桶维护](https://blog.csdn.net/lj900911/article/details/84138361)
