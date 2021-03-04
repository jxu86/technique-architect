# 非docker部署fabric2.2.0网络
`环境:`   
`mac`   
`docker 2.5.0.1`  
`golang 1.14.4`
`fabric 2.2.0`  

## 环境安装
下载fabric 2.2.0源码进行安装和编译

## 配置docker chaincode的地址
修改脚本`peer-org1-startup.sh`以下环境变量
```
export CORE_PEER_CHAINCODEADDRESS=192.168.31.67:7052
export CORE_PEER_CHAINCODELISTENADDRESS=192.168.31.67:7052
```
把`192.168.31.67`换成自己主机的ip

## 执行命令
* 执行`sh generate-config-file.sh`创建证书，创世块、通道文件和锚点文件等
* 执行`sh orderer-startup.sh`启动orderer节点
* 执行`sh peer-org1-startup.sh`启动peer节点
* 执行`sh create-channel.sh`创建通道
* 执行`sh deployCC.sh`安装链码



参考:   
[安装及运行Fabirc1.2--非docker模式](https://blog.csdn.net/sitebus/article/details/104095858)    
[Native方式运行Fabric(非Docker方式)](https://blog.csdn.net/u013938484/article/details/79867992)