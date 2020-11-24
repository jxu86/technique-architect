

```
fabric-ca-server init -b admin:adminpw --home  /root/fabric-ca-server --cfg.affiliations.allowremove --cfg.identities.allowremove
```
```
[root@jc fabric-ca-server]# tree
.
|-- ca-cert.pem
|-- fabric-ca-server-config.yaml
|-- fabric-ca-server.db
|-- IssuerPublicKey
|-- IssuerRevocationPublicKey
`-- msp
    `-- keystore
        |-- 04569ef502ba83d439d719bfa172c3d52ae9a910dc798a646a914a0fa6c73490_sk
        |-- IssuerRevocationPrivateKey
        `-- IssuerSecretKey
```

```
fabric-ca-server start -b admin:adminpw --home /root/fabric-ca-server
```

参考:   
[Hands-On Hyperledger Fabric——Fabric的证书（账号）体系](https://blog.csdn.net/No_Game_No_Life_/article/details/103102287)