
`environment:`      
`fabric v2.2.0`

## 概述
Fabric账户是基于证书而不是传统的用户名密码形式,叫MSP。   
通过基于 PKI 的成员身份管理，Fabric 网络可以对接入的节点和用户的各种能力进行限制   
Fabric 设计中考虑了三种类型的证书：登记证书（Enrollment Certificate）、交易证书（Transaction Certificate），以及保障通信链路安全的 TLS 证书。证书的默认签名算法为 ECDSA，Hash 算法为 SHA-256。

* 登记证书（ECert）：颁发给提供了注册凭证的用户或节点等实体，代表网络中身份。一般长期有效。
* 交易证书（TCert）：颁发给用户，控制每个交易的权限，不同交易可以不同，实现匿名性。短期有效。
* 通信证书（TLSCert）：控制对网络层的接入访问，可以对远端实体身份进行校验，防止窃听。   
`目前，在实现上，主要通过 ECert 来对实体身份进行检验，通过检查签名来实现权限管理。TCert 功能暂未实现，用户可以使用 idemix 机制来实现部分匿名性。`

## 生成MSP的工具
* cryptogen
* fabric-ca

## MSP结构
根据证书角色来区分，如 Admin、Client、Peer、Orderer等；
```
|——org1.example.com   组织名称
    |
    |——-ca    组织的根证书
    |
    |——-msp   组织的msp
    |
    |——-peers 节点相关证书
    |
    |——-tlsca    组织内部的tlsca证书
    |
    |——-users   组织所属用户
```

## 签名策略

默认情况下，每个通道成员都定义了一组引用其组织的签名策略。当提案提交给Peer或交易提交给Orderer节点时，节点将读取附加到交易上的签名，并根据通道配置中定义的签名策略对它们进行评估。每个签名策略都有一个规则，该规则指定了一组签名可以满足该策略的组织和身份。您可以在下面configtx.yaml中的Organizations部分中看到由Org1定义的签名策略：
```
- &Org1
  ...
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
```
上面的所有策略都可以通过Org1的签名来满足。但是，每个策略列出了组织内部能够满足该策略的一组不同的角色。Admins策略只能由具有管理员角色的身份提交的交易满足，而只有具有peer的身份才能满足Endorsement策略。附加到单笔交易上的一组签名可以满足多个签名策略。例如，如果交易附加的背书由Org1和Org2共同提供，则此签名集将满足Org1和Org2的Endorsement策略。

## ImplicitMeta策略
如果您的通道使用默认策略，则每个组织的签名策略将由通道配置中更高层级的ImplicitMeta策略评估。ImplicitMeta策略不是直接评估提交给通道的签名，而是使用规则在通道配置中指定可以满足该策略的一组其他策略。 如果交易可以满足该策略引用的下层签名策略集合，则它可以满足ImplicitMeta策略。

您可以在下面的configtx.yaml文件的Application部分中看到定义的ImplicitMeta策略：
```
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
```
更详细请参考[【Fabric v2.3.2】教程 - 创建通道 - 通道策略](https://blog.csdn.net/weixin_44485744/article/details/120748270)      

参考:   
[fabric2.3版本源码记录_1](https://www.cnblogs.com/dongjl/p/14572593.html)       
[fabric源码分析之六MSP源码分析](https://blog.csdn.net/fpcc/article/details/105157222)   
[Hyperledger Fabrica 2.0 MSP & BCCSP](https://blog.csdn.net/DAOSHUXINDAN/article/details/104759019/)    
[【Fabric v2.3.2】教程 - 创建通道 - 通道策略](https://blog.csdn.net/weixin_44485744/article/details/120748270)      
[超级账本Fabric中的权限管理和策略](https://blog.csdn.net/yeasy/article/details/88536882)        
[超级账本Fabric链码背书策略及ACL配置教程](https://my.oschina.net/u/3843525/blog/3149159)
