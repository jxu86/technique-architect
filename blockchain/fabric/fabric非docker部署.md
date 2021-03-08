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
创建并编辑config/cryptogen/crypto-config.yaml文件:
```
OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    EnableNodeOUs: true
    Specs:
      - Hostname: orderer
        SANS:
          - 127.0.0.1
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    Template:
      Count: 1
      SANS:
        - 127.0.0.1
    Users:
      Count: 1
```

生成证书文件:
```
cryptogen generate --config=./config/cryptogen/crypto-config.yaml --output=./config/crypto-config
```
把以下域名更新到hosts文件
```
127.0.0.1   peer0.org1.example.com
127.0.0.1   org1.example.com
127.0.0.1   example.com
127.0.0.1   orderer.example.com
```

## 生成创世区块
编辑config/configtx.yaml文件:
```
---
Organizations:

    # SampleOrg defines an MSP using the sampleconfig.  It should never be used
    # in production but may be used as a template for other definitions
    - &OrdererOrg
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrdererOrg

        # ID to load the MSP definition as
        ID: OrdererMSP

        # MSPDir is the filesystem path which contains the MSP configuration
        MSPDir: ./crypto-config/ordererOrganizations/example.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

        OrdererEndpoints:
            - 127.0.0.1:7050

    - &Org1
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org1MSP

        # ID to load the MSP definition as
        ID: Org1MSP

        MSPDir: ./crypto-config/peerOrganizations/org1.example.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org1MSP.peer')"

        # leave this flag set to true.
        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: 127.0.0.1
              Port: 7051

Capabilities:
    Channel: &ChannelCapabilities
        V2_0: true
    Orderer: &OrdererCapabilities
        V2_0: true
    Application: &ApplicationCapabilities
        V2_0: true

Application: &ApplicationDefaults
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        LifecycleEndorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"
        Endorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"

    Capabilities:
        <<: *ApplicationCapabilities
Orderer: &OrdererDefaults
    OrdererType: solo
    Addresses:
        - orderer.example.com:7050

    # Batch Timeout: The amount of time to wait before creating a batch
    BatchTimeout: 2s

    # Batch Size: Controls the number of messages batched into a block
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        # BlockValidation specifies what signatures must be included in the block
        # from the orderer for the peer to validate it.
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"

Channel: &ChannelDefaults
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ChannelCapabilities

Profiles:
    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
    TwoOrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
            Capabilities:
                <<: *ApplicationCapabilities
```
生成创世区块
```
export FABRIC_CFG_PATH=$(pwd)/config/
configtxgen -profile TwoOrgsOrdererGenesis  -channelID system-channel -outputBlock ./channel-artifacts/genesis.block
```
## 创建channel及锚点文件
创建channel文件
```
export FABRIC_CFG_PATH=$(pwd)/config/
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/mychannel.tx -channelID mychannel
```
创建锚点文件
```
export FABRIC_CFG_PATH=$(pwd)/config/
configtxgen -profile TwoOrgsChannel  -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID  mychannel -asOrg Org1MSP
```

## 启动Orderer节点
复制[orderer.yaml](https://github.com/jxu86/fabric-native-network/blob/master/config/orderer.yaml)到config目录下，打开终端并执行一下脚本

```
export FABRIC_CFG_PATH=$(pwd)/config/
export RDERER_HOST=orderer.example.com
export ORDERER_GENERAL_LOGLEVEL=DEBUG
export ORDERER_GENERAL_TLS_ENABLED=false
export ORDERER_GENERAL_TLS_PRIVATEKEY=$(pwd)/config/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
export ORDERER_GENERAL_TLS_CERTIFICATE=$(pwd)/config/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
export ORDERER_GENERAL_TLS_ROOTCAS=[$(pwd)/config/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt]
export ORDERER_GENERAL_ROOTCAS=[$(pwd)/config/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt]

export ORDERER_GENERAL_PROFILE_ENABLED=false
export ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
export ORDERER_GENERAL_LISTENPORT=7050
export ORDERER_GENERAL_GENESISMETHOD=file
export ORDERER_GENERAL_GENESISFILE=$(pwd)/channel-artifacts/genesis.block

export ORDERER_GENERAL_LOCALMSPDIR=$(pwd)/config/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp
export ORDERER_GENERAL_LOCALMSPID=OrdererMSP
export ORDERER_FILELEDGER_LOCATION=$(pwd)/data/orderer

export ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
export ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
export ORDERER_GENERAL_CLUSTER_ROOTCAS=crypto-config/ordererOrganizations/example.com/ca/ca.example.com-cert.pem

orderer

```

## 启动Peer节点启动
复制[core.yaml](https://github.com/jxu86/fabric-native-network/blob/master/config/core.yaml)到config目录下，新打开终端并执行一下脚本:
`注意: CORE_PEER_CHAINCODEADDRESS和CORE_PEER_CHAINCODELISTENADDRESS换成自己的电脑ip`

```
export FABRIC_CFG_PATH=$(pwd)/config/

export CORE_PEER_ID=peer0.org1.example.com
export CORE_CHAINCODE_MODE=dev
export CORE_PEER_CHAINCODEADDRESS=192.168.31.67:7052        //这里的ip换成自己电脑的ip
export CORE_PEER_CHAINCODELISTENADDRESS=192.168.31.67:7052  //这里的ip换成自己电脑的ip
export CORE_PEER_NETWORKID=dev
export CORE_PEER_TLS_ENABLED=false
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_ADDRESS=127.0.0.1:7051
export CORE_PEER_GOSSIP_BOOTSTRAP=127.0.0.1:7051
export CORE_PEER_GOSSIP_EXTERNALENDPOINT=127.0.0.1:7051
export CORE_PEER_LISTENADDRESS=0.0.0.0:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_CERT_FILE=$(pwd)/config/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=$(pwd)/config/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=$(pwd)/config/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE = $(pwd)/config/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/tlscacerts/tlsca.org1.example.com-cert.pem

export CORE_PEER_FILESYSTEMPATH=$(pwd)/data/peer
export FABRIC_LOGGING_SPEC=INFO

peer node start 
```


## 创建通道&加入通道&更新锚点
新打开终端，执行创建通道脚本
```
export FABRIC_CFG_PATH=$(pwd)/config/
export CHANNEL_NAME=mychannel
export CORE_CHAINCODE_MODE=dev
export CORE_PEER_ID=peer0.org1.example.com
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_MSPCONFIGPATH=$(pwd)/config/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer channel create -o localhost:7050 -c mychannel --ordererTLSHostnameOverride orderer.example.com -f ./channel-artifacts/mychannel.tx --outputBlock ./channel-artifacts/mychannel.block
```

执行加入通道脚本

```
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_MSPCONFIGPATH=$(pwd)/config/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer channel join -b ./channel-artifacts/mychannel.block
```


执行更新锚点脚本
```
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_MSPCONFIGPATH=$(pwd)/config/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer channel update -o localhost:7050 -c mychannel -f  $(pwd)/channel-artifacts/Org1MSPanchors.tx

```

##链码安装以及执行  
参考: https://github.com/jxu86/fabric-native-network/blob/master/deployCC.sh



所有的代码以及脚本请参考[我的github](https://github.com/jxu86/fabric-native-network)



参考:   
[安装及运行Fabirc1.2--非docker模式](https://blog.csdn.net/sitebus/article/details/104095858)    
[Native方式运行Fabric(非Docker方式)](https://blog.csdn.net/u013938484/article/details/79867992)     
[Fabric 2.x 安装链码流程](https://blog.csdn.net/hello2mao/article/details/106083995)    
[非Docker环境部署Fabric](http://xuyao.club/blog/2020/08/26/deploy-fabric-to-a-non-docker-environment/)