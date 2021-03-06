`环境:`     
`fabric v1.4.4`     
`go version go1.13.5 darwin/amd64`          
`node v12.10.0`         
`npm v6.10.3`         
`git version 2.17.2 (Apple Git-113)`            
`Docker version 19.03.5, build 633a0ea`         
`docker-compose version 1.24.1, build 4667896b`         

### **一、Hyperledger Fabric环境搭建**
```
mkdir hyfa && cd hyfa
vim bootstrap.sh
```
把`https://github.com/hyperledger/fabric/blob/v1.4.4/scripts/bootstrap.sh`脚本内容copy到刚创建的bootstrap.sh文件里面去
然后执行
```
sh bootstrap.sh
```
这个脚本自动化的把所有的环境和文件编译都帮忙做了，执行完后也就是fabric环境安装完成，不过注意的是这个过程有点慢，需要vpn出去，也可以切换一下docker的镜像源。
这个脚本主要是做:
* 下载fabric-samples
* 下载相关的docker镜像文件
* 编译平台特定的二进制文件和配置文件安装到fabric-samples存储库的根目录中

脚本执行完后，执行`docker images`可以看到相关的镜像如下
```
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-javaenv     1.4.4               4648059d209e        5 weeks ago         1.7GB
hyperledger/fabric-javaenv     latest              4648059d209e        5 weeks ago         1.7GB
hyperledger/fabric-ca          1.4.4               62a60c5459ae        5 weeks ago         150MB
hyperledger/fabric-ca          latest              62a60c5459ae        5 weeks ago         150MB
hyperledger/fabric-tools       1.4.4               7552e1968c0b        5 weeks ago         1.49GB
hyperledger/fabric-tools       latest              7552e1968c0b        5 weeks ago         1.49GB
hyperledger/fabric-ccenv       1.4.4               ca4780293e4c        5 weeks ago         1.37GB
hyperledger/fabric-ccenv       latest              ca4780293e4c        5 weeks ago         1.37GB
hyperledger/fabric-orderer     1.4.4               dbc9f65443aa        5 weeks ago         120MB
hyperledger/fabric-orderer     latest              dbc9f65443aa        5 weeks ago         120MB
hyperledger/fabric-peer        1.4.4               9756aed98c6b        5 weeks ago         128MB
hyperledger/fabric-peer        latest              9756aed98c6b        5 weeks ago         128MB
hyperledger/fabric-zookeeper   0.4.18              ede9389347db        7 weeks ago         276MB
hyperledger/fabric-zookeeper   latest              ede9389347db        7 weeks ago         276MB
hyperledger/fabric-kafka       0.4.18              caaae0474ef2        7 weeks ago         270MB
hyperledger/fabric-kafka       latest              caaae0474ef2        7 weeks ago         270MB
hyperledger/fabric-couchdb     0.4.18              d369d4eaa0fd        7 weeks ago         261MB
hyperledger/fabric-couchdb     latest              d369d4eaa0fd        7 weeks ago         261MB
```

再进去fabric-samples/bin目录可以看到生成的执行文件:
```
configtxgen      cryptogen        fabric-ca-client orderer
configtxlator    discover         idemixgen        peer
```

如果看到以上结果，那就证明安装完成了

### **二、Hyperledger Fabric网络搭建**
`注意：下面涉及到的配置文件都可以在fabric-samples/first-network下找到`         
`以下是操作可以明白整个网络的启动流程，如果想快速启动网络直接进去fabric-samples/first-network 执行sh byfn.sh -m up就搞定了`

#### **1、crypto-config.yaml配置文件**
crypto-config.yaml 文件主要指定整个网络中相关组织的详细信息

```
# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes
# ---------------------------------------------------------------------------
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer               # Orderer组织的名称
    Domain: example.com         # 指定Orderer组织的域名
    EnableNodeOUs: true         # 指定是否生成config.yaml文件
    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs:
      - Hostname: orderer       # Specs.Hostname属性值+Domain属性值构成了Orderer组织的完整域名。
      - Hostname: orderer2
      - Hostname: orderer3
      - Hostname: orderer4
      - Hostname: orderer5

# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ---------------------------------------------------------------------------
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
```
&emsp;&emsp;该配置文件指定了OrdererOrgs及PeerOrgs两个组织信息。PeerOrgs配置信息指定创建了Org1与Org2两个组织。每个Org组织使用Template属性下的Count指定创建了两个Peer节点，Users属性下的Count指定了在各个Peer节点中创建一个用户。   
&emsp;&emsp;Peer节点的域名名称组成结构为“peer+起始数字0+Domain属性的值”。例如，Org1组织中的Domain属性值为`org1.example.com`，Template.Count属性值为2，则两个Peer节点的完整域名为`peer0.org1.example.com`和`peer1.org1.example.com`。    


#### **2、生成组织结构与身份证书**
&emsp;&emsp;Hyperledger Fabric提供一个工具cryptogen可以生成组织结构和身份证书信息
```
# 执行命令
cryptogen generate --config=./crypto-config.yaml

# 输出
org1.example.com
org2.example.com
```
&emsp;&emsp;证书和密钥（即MSP材料）将被输出到当前路径下的一个名为crypto-config的目录中，该目录下会根据crypto-config.yaml配置文件中指定的结构产生两个子目录.     
·ordererOrganizations子目录下包括构成Orderer组织（1个Orderer节点）的身份信息。     
·peerOrganizations子目录下为所有的Peer节点组织（2个Org组织，每个Org组织包含2个Peer节点）的相关身份信息。其中最关键的是MSP目录，代表了实体的身份信息。

#### **3、configtx.yaml配置文件**
&emsp;&emsp;生成组织结构与身份证书、密钥之后，需要为区块链创建一个GenesisBlock（初始区块或称之为创世区块）与Channel（通道），这些内容需要通过配置定义来指定相关的信息，如指定Orderer服务的相关配置，以及当前的联盟信息、联盟中所包含的组织信息，这些信息的配置被定义在configtx.yaml文件中。核心配置内容如下。   
* Organizations部分指定OrdererOrg与PeerOrg的组织信息，其核心目的是指定各组织的名称、唯一ID及MSP的目录所在路径。
* Capabilities部分指定通道的权限信息。
* Application部分指定初始加入通道的组织。
* Orderer部分指定Orderer节点的信息。
具体文件可参考: 
https://github.com/hyperledger/fabric-samples/blob/v1.4.4/first-network/configtx.yaml

#### **4、Order服务创建创世区块文件**
&emsp;&emsp;指定使用configtx.yaml文件中定义的TwoOrgsOrdererGenesis模板，生成Orderer服务系统通道的初始区块文件
```
export SYS_CHANNEL= byfn-sys-channel

configtxgen -profile TwoOrgsOrdererGenesis -channelID $SYS_CHANNEL -outputBlock ./channel-artifacts/genesis.block
```

#### **3、创建应用通道交易配置文件**
&emsp;&emsp;指定使用configtx.yaml配置文件中的TwoOrgsChannel模板来生成新建通道的配置交易文件（TwoOrgsChannel模板指定的Org1和Org2两个组织都属于应用通道中的成员）
```
export CHANNEL_NAME=mychannel

configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```


#### **4、生成锚节点更新配置文件**
&emsp;&emsp;同样基于configtx.yaml配置文件的TwoOrgsChannel模板，为每个组织分别生成锚节点更新配置，且注意指定对应的组织名称。
```
export CHANNEL_NAME=mychannel

configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```


#### **5、docker-compose-cli.yaml配置文件**
```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

volumes:
  orderer.example.com:
  peer0.org1.example.com:
  peer1.org1.example.com:
  peer0.org2.example.com:
  peer1.org2.example.com:

networks:
  byfn:

services:

  orderer.example.com:
    extends:
      file:   base/docker-compose-base.yaml
      service: orderer.example.com
    container_name: orderer.example.com
    networks:
      - byfn

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org1.example.com
    networks:
      - byfn

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org1.example.com
    networks:
      - byfn

  peer0.org2.example.com:
    container_name: peer0.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org2.example.com
    networks:
      - byfn

  peer1.org2.example.com:
    container_name: peer1.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org2.example.com
    networks:
      - byfn

  cli:
    container_name: cli
    image: hyperledger/fabric-tools:$IMAGE_TAG
    tty: true
    stdin_open: true
    environment:
      - SYS_CHANNEL=$SYS_CHANNEL
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      #- FABRIC_LOGGING_SPEC=DEBUG
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - orderer.example.com
      - peer0.org1.example.com
      - peer1.org1.example.com
      - peer0.org2.example.com
      - peer1.org2.example.com
    networks:
      - byfn
```
&emsp;&emsp;由以上配置信息可以看出，该配置文件指定了网络中各个节点容器（共计6个容器，即1个Orderer、属于2个Orgs组织的4个Peer、1个CLI）的信息。仔细观察会发现，Orderer与各Peer容器都设置了container_name与networks信息；其他信息都由extends指向了base/docker-compose-base.yaml文件。
&emsp;&emsp;CLI容器指定了所代表的Peer节点（CORE_PEER_ADDRESS=peer0.org1.example.com：7051），通过volumes指定了将系统中的链码、组织结构及证书、生成的配置文件映射到容器中指定的目录下，且通过depends_on属性指定了所依赖的相关容器。
关联配置文件:   
docker-compose-base.yaml    
peer-base.yaml

#### **6启动网络** 
```
docker-compose -f docker-compose-cli.yaml up -d
```
* -f 指定启动容器时所使用的docker-compose配置文件
* -d 指定是否显示网络启动过程中的实时日志信息，如果需要查看详细网络启动日志

启动成功后执行`docker ps`可以查看有6个容器在跑。
如果要关闭网络可以执行`docker-compose -f docker-compose-cli.yaml down`

#### **5、创建应用通道**
执行
```
docker exec -it cli bash
```
进入cli容器
创建环境变量:
```
export CHANNEL_NAME=mychannel
```
```
peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```


#### **6、将节点加入应用通道**  
```
peer channel join -b mychannel.block
```


#### **7、更新锚节点**
使用Org1的管理员身份更新锚节点配置:
```
peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

使用Org2的管理员身份更新锚节点配置
```
peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```


参考:   
`<<Hyperledger Fabric菜鸟进阶攻略>> 第一章、第三章`     
[Hyperledger Fabric1.4安装](https://www.cnblogs.com/zongmin/p/11635686.html)    
[Hyperledger中文文档](https://hyperledgercn.github.io/hyperledgerDocs/)
[hyperledger fabric 测试环境调试智能合约](https://www.jianshu.com/p/1db60adf134b)