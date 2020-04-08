
`Environment:`  
`fabric v1.4.4`


### 一、概述

在fabric中联盟不能为空，必须包含一个组织机构，所有在创建联盟的时候必须有一个组织机构，能够添加进去，
fabric中的联盟和通道是一对一的关系，联盟必须和通道channel并存，而所有的配置都是记录在区块中的，包括有哪些联盟，有哪些org，所以要添加联盟就必须修改区块中的数据，更新配置。

### 二、创建联盟
1、配置configtx.yaml    
2、生成新的创世区块文件     
3、把新的创世区块文件抽取新联盟的配置生成json格式文件       
4、获取现在创世区块文件并用jd转化成json格式文件     
5、经过增量计算和添加相应的头设置生成最新的.pb文件      
6、把.pb文件update到orderer上更新创世区块


* 修改配置文件configtx.yaml        
向`configtx.yaml`的`Section: Profile`中创建Orderer创世区块的配置profile中添加新联盟（以TestConsortium联盟为例）

```yaml
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
           - *Org2
     TestConsortium:
     	Organizations:
           - *Org1
```

* 根据配置文件生成包含新联盟的新创世区块
```
configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/sys-channel.block
```

* 将其内容转换成JSON并抽取出新联盟的配置材料
```
configtxlator proto_decode --input ./channel-artifacts/sys-channel.block --type common.Block | jq .data.data[0].payload.data.config.channel_group.groups.Consortiums.groups.TestConsortium > ./channel-artifacts/TestConsortium.json
```

* 进入cli容器
```
docker exec -it cli bash
```
* 设置`ORDERER_CA`环境变量
```
 export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

* 切换到`OrdererOrgs`的admin用户
```bash
    export CORE_PEER_LOCALMSPID="OrdererMSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=$ORDERER_CA
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/users/Admin@example.com/msp
```
`原因：系统通道的相关操作必须由`OrdererOrgs`的admin用户来执行`

* 获取系统通道的创世区块
```bash
    peer channel fetch config ./channel-artifacts/sys_config_block.pb -o orderer.example.com:7050 -c byfn-sys-channel --tls --cafile $ORDERER_CA
```

其中`-c`参数是`--channelID`的简写，此处需要使用系统通道ID，即在调用`configtxgen`创建orderer创世区块时所指定的channelID。

* 将创世区块中的内容转换成JSON并对其进行修剪    
```bash
    exit
    configtxlator proto_decode --input ./channel-artifacts/sys_config_block.pb --type common.Block | jq .data.data[0].payload.data.config > ./channel-artifacts/sys_config.json
```

* 将新联盟TestConsortium配置定义`TestConsortium.json`添加到channel的`Consortiums`的`TestConsortium`中，并将其写入`sys_updated_config.json`

```bash
    jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups": {"TestConsortium": .[1]}}}}}' ./channel-artifacts/sys_config.json ./channel-artifacts/TestConsortium.json >& ./channel-artifacts/sys_updated_config.json
```

* 将原始的配置sys_config.json编码成protobuf
```
configtxlator proto_encode --input ./channel-artifacts/sys_config.json --type common.Config --output ./channel-artifacts/sys_config.pb
```

* 将更新后的配置sys_updated_config.json编码成protobuf
```
configtxlator proto_encode --input ./channel-artifacts/sys_updated_config.json --type common.Config --output ./channel-artifacts/sys_updated_config.pb
```

* 配置增量计算
```
configtxlator compute_update --channel_id byfn-sys-channel --original ./channel-artifacts/sys_config.pb --updated ./channel-artifacts/sys_updated_config.pb --output ./channel-artifacts/sys_config_update.pb
```

* 将sys_config_update.pb编码成json
```
configtxlator proto_decode --input ./channel-artifacts/sys_config_update.pb --type common.ConfigUpdate | jq . > ./channel-artifacts/sys_config_update.json
```

* 生成sys_config_update_in_envelope.json
```
echo '{"payload":{"header":{"channel_header":{"channel_id":"byfn-sys-channel", "type":2}},"data":{"config_update":'$(cat ./channel-artifacts/sys_config_update.json)'}}}' | jq . > ./channel-artifacts/sys_config_update_in_envelope.json
```

* 将sys_config_update_in_envelope.json编码成protobuf
```
configtxlator proto_encode --input ./channel-artifacts/sys_config_update_in_envelope.json --type common.Envelope --output ./channel-artifacts/sys_config_update_in_envelope.pb
```

* 向orderer发送配置更新（<u>必须使用OrdererOrg的admin用户</u>)

```bash
    docker exec -it cli bash
    
    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    export CORE_PEER_LOCALMSPID="OrdererMSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=$ORDERER_CA
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/users/Admin@example.com/msp
    
    peer channel update -f ./channel-artifacts/sys_config_update_in_envelope.pb -c byfn-sys-channel -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA
```

### 三、创建通道        
创建channel，只要是联盟的成员的admin都可以创建channel       

* 编辑configtx.yaml找到channel创建的配置文件的位置，编写channel配置文件
```
TestChannel:
        Consortium: TestConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
            Capabilities:
                <<: *ApplicationCapabilities
```


```
configtxgen -profile TestChannel -outputCreateChannelTx ./channel-artifacts/testchannel.tx -channelID testchannel
```

```
docker exec -it cli bash
```


此处我们testConsortium里面的是org1，所有无需切换环境变量，如果是其他org,则必须切换到该org的admin用户

```
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

peer channel create -o orderer.example.com:7050 -c testchannel -f ./channel-artifacts/testchannel.tx --tls --cafile $ORDERER_CA

peer channel fetch 0 testchannel.block -o orderer.example.com:7050 -c testchannel --tls --cafile $ORDERER_CA

peer channel join -b testchannel.block
peer channel list #查看

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer1.org1.example.com:7051

peer channel join -b testchannel.block
peer channel list #查看
```


参考:   
[hyperledge工具-configtxlator](https://www.cnblogs.com/wanghui-garcia/p/10497415.html)      
[Linux系统下使用jq工具处理json](https://www.jianshu.com/p/3522fe70de19)         
[linux工具之jq](https://blog.csdn.net/weixin_44398879/article/details/85774977)