

### 启动流程
`入口函数cmd/geth/main.go>main()>geth()`
主要做了以下几件事：
* 准备操作内存缓存配额并设置度量系统
* 加载配置和注册服务
* 启动节点
  * p2p server启动
  * rpc服务启动
* 守护当前线程(等待结束信号)

### 加载配置
* 首先加载默认配置(作为主网节点启动)
* 接着加载自定义配置（适用私有链）
* 最后加载命令窗口参数（开发阶段）


参考:   
[Geth启动流程代码解析(完整解析）](https://blog.csdn.net/jiang_xinxing/article/details/80289619)     
[以太坊启动源码分析](https://blog.csdn.net/TurkeyCock/article/details/80399203)        
[以太坊启动过程源码解析](https://www.jianshu.com/p/8776c13fcd30)