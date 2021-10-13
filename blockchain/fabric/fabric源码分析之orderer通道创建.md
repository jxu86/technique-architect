
`environment:`      
`fabric v2.2.0`


## 主流程
* load配置文件(默认orderer.yaml)
* 设置日志的级别
* 初始化本地MSP组件
* 创建监测器
* 初始化gRPC服务器
* 创建账本工厂
* 初始化系统通道

* 初始化多通道注册管理器
* 创建Orderer排序服务器
* 开始grpc服务







* 命令
    * start(默认)
    * version


* 使用到的包
    * kingpin


* 模块
    * Server


数据流向：peer节点→grpc→Server→broadcast/deliver→broadcastSupport/deliverSupport→multichain→chainSupport→kafka/solo→chain→BlockCutter→chainSupport.CreateNextBlock→chainSupport.WriteBlock→ledger账本→kafka→peer节点。










参考:   
[Fabric源码分析-Orderer启动](https://www.jianshu.com/p/b828d56af119)    
[Hyperledger Fabric 排序服务核心原理和工作过程](https://blog.csdn.net/yeasy/article/details/78798506)