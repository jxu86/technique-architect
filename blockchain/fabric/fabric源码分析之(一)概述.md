



## 1.源码中的简拼   
```
MSP：Membership service provider 会员服务提供者
BCCSP：blockchain（前两个字母BC） cryptographic service provider 区域链加密服务提供者
ab：atomic broadcast原子（操作）广播
lscc：lifecycle(L) system(S) chaincode（CC）生命周期系统链码
Spec：Specification，规格标准，详细说明
KV：key-value 键-值
CDS：ChaincodeDeploymentSpec
CIS：ChaincodeInvocationSpec
mgmt：management
SW：software-based
AB：AtomicBroadcast
GB：genesis block，创世纪的block，也就是区域链中的第一个块
CC或cc：chaincode
SCC或scc：system chaincode
cscc：configer system chaincode
lscc：lifecycle system chaincode
escc：endorser system chaincode
vscc：validator system chaincode
qscc：querier system chaincode
alg：algorithm 算法
mcs：mspMessageCryptoService
mock：假装，学样子，模仿的意思，基本上是服务于xxx_test.go的，即用于测试的
Gossip：一种使分布结点达到状态最终一致的算法
attr：attribute
FsBlockStore：file system block store
vdb：versioned database 也就是状态数据库
RTEnv：runtime environment运行环境
pkcs11：pcks#11，一种公匙加密标准，有一套叫做Cryptoki的接口，是一组平台设备无关的API
MCS：mspMessageCryptoService，消息加密服务
sa：SecurityAdvisor
impl：implement，好多处XXX.go和XXXimpl.go是对应的，前者是用于接口或者定义的，后者是实现该接口或定义的
FSM：finite state machine 有限状态机
FS：filesystem 文件系统
blk：block
cli：command line interface 命令行界面
CFG：FABRIC_CFG_PATH中的，应该是config的意思
mgr：manager
cpinfo：checkpoint information，检查点信息
DevMode：development mode，开发模式
Reg：register，注册，登记
hdr：header
impl：implement
oid：ObjectIdentifier，对象标识符
ou或OU：organizational unit
CRL：certificate revocation list，废除证书列表
prop：proposal，申请，交易所发送的申请
ACL：Access Control List，访问控制列表
rwset：read/write set，读写集
tx，Tx：transaction，交易
CSP：cryptographic service provider，BCCSP的后三个字母，加密服务提供者
opt：option，选项
opts：options，多个选项
SKI：当前证书标识，所谓标识，一般是对公匙进行一下hash
AKI：签署方的SKI，也就是签署方的公匙标识
HSM：Hardware Security Modules
ks：KeyStore，Key存储，这个key指的是用于签名的公匙私匙
oid：OBJECT IDENTIFIER，对象身份标识
```
## 2.源码目录的基本结构     
* bcssp 加密服务代码目录
* common 全局公用代码目录
* core 核心功能代码目录
* docs 以.rst文件为核心，可编译生成文档。说明文档的目录
* events 事件代码目录，用于生产和消费信息
* examples 示例目录
* gossip 本意是绯闻的意思，是一种可达到去中心化，有一定容错能力且可达到最终一致的传播算法
* msp 会员服务代码目录
* orderer 就理解成orderer目录就好，orderer也算是区域链中的专用名词，用于消息的订阅与分发处理
* peer  peer节点入口    
* protos 原型目录，定义个各种原型和生成的对应的XXX.pb.go源码
* vendor 原意是商贩，在此就是存放go中使用的全部的各种第三方包





[fabric源码解析1——线头](https://blog.csdn.net/idsuf698987/article/details/74912362)     



