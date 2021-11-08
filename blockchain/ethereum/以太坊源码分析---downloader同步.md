




### full sync
full 模式会在数据库中保存所有区块数据，同步时从远程节点同步 header 和 body 数据，而state 和 receipt 数据则是在本地计算出来的。

在 full 模式下，downloader 会同步区块的 header 和 body 数据组成一个区块，然后通过 blockchain 模块的 BlockChain.InsertChain 向数据库中插入区块。在 BlockChain.InsertChain 中，会逐个计算和验证每个块的 state 和 recepit 等数据，如果一切正常就将区块数据以及自己计算得到的 state、recepit 数据一起写入到数据库中。

### fast sync
fast 模式下，recepit 不再由本地计算，而是和区块数据一样，直接由 downloader 从其它节点中同步；state 数据并不会全部计算和下载，而是选一个较新的区块（称之为 pivot）的 state 进行下载，以这个区块为分界，之前的区块是没有 state 数据的，之后的区块会像 full 模式下一样在本地计算 state。因此在 fast 模式下，同步的数据除了 header 和 body，还有 receipt，以及 pivot 区块的 state。

因此 fast 模式忽略了大部分 state 数据，并且使用网络直接同步 receipt 数据的方式替换了 full 模式下的本地计算，所以比较快。

### light sync
light 模式也叫做轻模式，它只对区块头进行同步，而不同步其它的数据。

SyncMode:
* FullSync:从完整区块同步整个区块链历史
* FastSync:快速下载标题，仅在链头处完全同步
* LightSync:仅下载标题，然后终止










参考:   
[死磕以太坊源码分析之downloader同步](https://www.cnblogs.com/1314xf/p/14177133.html)       
[死磕以太坊源码分析之区块上链入库](https://blog.csdn.net/pulong0748/article/details/114008162)