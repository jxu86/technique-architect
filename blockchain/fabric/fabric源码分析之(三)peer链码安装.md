




## 1.概述       
在Fabric中交易的处理过程，客户端将提案首先发送到背书节点，背书节点检提案的合法性。如果合法的话，背书节点将通过交易所属的链码临时执行一个交易，并执行背书节点在本地持有的状态副本。  
Chaincode应该仅仅被安装于chaincode所有者的背书节点上，链码运行在节点上的沙盒（Docker容器）中，并通过gRPC协议与相应的Peer节点进行交互，以使该chaincode逻辑对整个网络的其他成员保密。     
请务必在一条channel上每一个要运行你chaincode的背书节点上安装你的chaincode       
其他没有chaincode的成员将无权成为chaincode影响下的交易的认证节点（endorser）。也就是说，他们不能执行chaincode。不过，他们仍可以验证交易并提交到账本上。     
ChainCode要在区块链网络中运行，需要经过链码安装和链码实例化两个步骤。

[多次安装，一次实例化]      
在一个区块链子链中，该网络是由“1账本+1通道+N个peer节点”组成。           
如果我们要手动来搭建Fabric网络的话，即通过命令行的形式来进行ChainCode的安装与实例化。我们需要多次install，一次instance。            
也就是说，对于整个Fabric网络来说，假设有X个背书节点，那么，我们需要给每个背书节点安装ChainCode，但是在整个网络搭建过程中只需要instance ChainCode一次。          
因为install 针对的是背书节点，instance 针对的是通道。           
install 链码的对象是背书节点，主要目的是方便背书节点对运行链码，对交易进行模拟。        
instance 链码的对象是channel，主要目的是为了将安装过的链码在指定通道上进行实例化调用，在节点上创建容器启动，并执行初始化操作。实例化的过程中，需要指定背书策略，来确定通道上哪些节点执行的交易才能添加到账本中。        
安装的过程其实就是对指定的代码进行编译打包，并把打包好的文件发送到Peer，等待接下来的实例化。其实，这就是我们接下来要讲解的链码的生命周期这一部分。      
实例化链上代码主要是在Peer所在的机器上对前面安装好的链上代码进行包装，生成对应Channel的Docker镜像和Docker容器。并且在实例化时我们可以指定背书策略。     
1. Chaincode运行在一个受保护的Docker容器当中，与背书节点的运行互相隔离。
2. Chaincode可通过应用提交的交易对账本状态初始化并进行管理。




![peer chaincode install](../../file/fabric/chaincode_install.png)










参考:   
[5-ChainCode生命周期、分类及安装、实例化命令解析](https://zhuanlan.zhihu.com/p/35419439)