
`Environment:`

`fabric v1.4.4`


### byfn.sh -m generate
gengerate 作用为生成网络初始化配置，包括MSP证书、私钥、创世区块和配置交易等文件，MSP相关的信息在crypto-config文件夹中，创世区块在channel-artifacts文件夹中，MSP证书和私钥用于不同的网络实体，创世块用于启动排序服务，配置交易文件用于配置通道。
```
... ...

elif [ "${MODE}" == "generate" ]; then ## Generate Artifacts
  generateCerts
  replacePrivateKey
  generateChannelArtifacts

... ...
```
在byfn.sh脚本上，依次执行了 generateCerts、  replacePrivateKey、  generateChannelArtifacts三个函数。        
其中：generateCerts执行下面述(1)中操作；replacePrivateKey将Org1、Org2的证书私钥文件配置到docker-compose-e2e.yaml文件中，以便于后续网络启动时候使用；generateChannelArtifacts执行下面(2)-(5)中的操作。

```
➜  first-network git:(bc72f3e) ✗ ./byfn.sh -m generate
Generating certs and genesis block for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
/Users/JC/Documents/project/hyba/fabric-samples/first-network/../bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########  (1) 创建证书
##########################################################
+ cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
+ res=0
+ set +x

Generate CCP files for Org1 and Org2
/Users/JC/Documents/project/hyba/fabric-samples/first-network/../bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############  (2) 生成创世块
##########################################################
CONSENSUS_TYPE=solo
+ '[' solo == solo ']'
+ configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
2019-12-19 08:05:04.105 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-12-19 08:05:04.354 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: solo
2019-12-19 08:05:04.354 CST [common.tools.configtxgen.localconfig] Load -> INFO 003 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:04.494 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 004 orderer type: solo
2019-12-19 08:05:04.494 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 005 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:04.498 CST [common.tools.configtxgen] doOutputBlock -> INFO 006 Generating genesis block
2019-12-19 08:05:04.499 CST [common.tools.configtxgen] doOutputBlock -> INFO 007 Writing genesis block
+ res=0
+ set +x

#################################################################
### Generating channel configuration transaction 'channel.tx' ###   (3) 生成通道配置交易文件
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
2019-12-19 08:05:04.544 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-12-19 08:05:04.699 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:04.910 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-12-19 08:05:04.910 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:04.910 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 005 Generating new channel configtx
2019-12-19 08:05:04.919 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 006 Writing new channel tx
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########   (4) 在通道上为Org1定义一个锚节点
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
2019-12-19 08:05:04.992 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-12-19 08:05:05.225 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:05.584 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-12-19 08:05:05.584 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:05.584 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
2019-12-19 08:05:05.584 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########   (5) 在通道上为Org2定义一个锚节点
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
2019-12-19 08:05:05.641 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-12-19 08:05:05.811 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:05.997 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-12-19 08:05:05.997 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /Users/JC/Documents/project/hyba/fabric-samples/first-network/configtx.yaml
2019-12-19 08:05:05.997 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
2019-12-19 08:05:05.998 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
+ res=0
+ set +x
```



参考:   
[Hyperledger Fabric笔记3--BYFN启动流程分析](https://blog.csdn.net/u011127242/article/details/80038177)