# 非docker部署fabric2.2.0网络
`环境:`   
`mac`   
`docker 2.5.0.1`  
`golang 1.14.4`
`fabric 2.2.0`  

## 概述
一般我们部署fabric的网络都是基于docker容器上进行操作，如果想了解fabric docker网络的部署可以参考[Fabric之环境安装与网络启动流程](https://blog.csdn.net/u013938484/article/details/79867992),但如果是想对fabric进行深入的研究，源码的分析，docker的网络不利于调试，还是以非docker形式进行调试比较方便，当然这里说的非docker是orderer和peer节点的部署不用docker，链码还是用到docker。

## 环境安装
* 安装Docker及Docker-Compose
* 安装golang
* 下载和编译安装fabric 2.2.0    
    上github下周fabric 2.2.0后，进入目录执行`make release`,安装好的fabric工具请设置好环境变量
    ```
    git clone https://github.com/hyperledger/fabric.git
    cd fabric
    git checkout -b v2.2.0 v2.2.0
    make release
    ```

## 生成证书



## 生成创世区块


## 创建channel及锚点文件

## 启动Orderer节点

## 启动Peer节点启动

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
[Fabric 2.x 安装链码流程](https://blog.csdn.net/hello2mao/article/details/106083995)