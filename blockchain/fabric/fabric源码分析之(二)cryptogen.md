`环境:`  
`fabric: v1.4.2`  



### 1.概述
cryptogen是hyperleder fabric提供的为网络实体生成加密材料（公私钥/证书等）的实用程序。简单来说就是一个生成认证证书(x509 certs)的工具。这些证书代表一个身份，并允许在网络实体间通信和交易时进行签名和身份认证。           
cryptogen使用一个包含网络拓扑的crypto-config.yaml文件，为文件中定义的组织和属于这些组织的实体生成一组证书和密钥。每个组织都配置唯一的根证书（ca-cert），并包含了特定的实体（peer和orders），这就形成李一种典型的网络结构--每个成员都有所属杜CA。hyperleder fabric网络中的交易和通信都使用实体杜私钥签名，使用公钥验证。


### 2.`common/tools/cryptogen`目录结构
cryptogen的代码目录如下:    
```bash
├── ca
│   ├── ca_test.go
│   └── generator.go            # CA证书生成
├── csp
│   ├── csp.go                  # 公私钥生成，调用了bccsp
│   └── csp_test.go
├── main.go                     # main函数入口，命令解析，命令执行
├── metadata
│   ├── metadata.go             # 版本信息
│   └── metadata_test.go
└── msp
    ├── generator.go            # MSP结构生成
    ├── msp_internal_test.go
    └── msp_test.go
```

### 3.流程图
[cryptogen函数流程图](https://www.processon.com/view/link/5ddc9aece4b0fcce5b59f01e)	

### 4.配置文件
```yaml
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    EnableNodeOUs: true
    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs:
      - Hostname: orderer
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

### 5.主要函数

* generatePeerOrg
```go
func generatePeerOrg(baseDir string, orgSpec OrgSpec) {

	orgName := orgSpec.Domain

	fmt.Println(orgName)
    // generate CAs
    // 生成目录结构
	orgDir := filepath.Join(baseDir, "peerOrganizations", orgName)
	caDir := filepath.Join(orgDir, "ca")
	tlsCADir := filepath.Join(orgDir, "tlsca")
	mspDir := filepath.Join(orgDir, "msp")
	peersDir := filepath.Join(orgDir, "peers")
	usersDir := filepath.Join(orgDir, "users")
	adminCertsDir := filepath.Join(mspDir, "admincerts")
    // generate signing CA
    // 生成Ca证书，ca目录下pem和_sk文件
	signCA, err := ca.NewCA(caDir, orgName, orgSpec.CA.CommonName, orgSpec.CA.Country, orgSpec.CA.Province, orgSpec.CA.Locality, orgSpec.CA.OrganizationalUnit, orgSpec.CA.StreetAddress, orgSpec.CA.PostalCode)
	if err != nil {
		fmt.Printf("Error generating signCA for org %s:\n%v\n", orgName, err)
		os.Exit(1)
	}
	// generate TLS CA
	// 生成ca证书，tlsca目录下pem和_sk文件
	tlsCA, err := ca.NewCA(tlsCADir, orgName, "tls"+orgSpec.CA.CommonName, orgSpec.CA.Country, orgSpec.CA.Province, orgSpec.CA.Locality, orgSpec.CA.OrganizationalUnit, orgSpec.CA.StreetAddress, orgSpec.CA.PostalCode)
	if err != nil {
		fmt.Printf("Error generating tlsCA for org %s:\n%v\n", orgName, err)
		os.Exit(1)
	}
	// 生成msp下的admincerts、cacerts、tlscacerts目录及对应的文件
	err = msp.GenerateVerifyingMSP(mspDir, signCA, tlsCA, orgSpec.EnableNodeOUs)
	if err != nil {
		fmt.Printf("Error generating MSP for org %s:\n%v\n", orgName, err)
		os.Exit(1)
	}
	// 遍历所有的peer生成对应的msp、tls以及其目录下的文件
	generateNodes(peersDir, orgSpec.Specs, signCA, tlsCA, msp.PEER, orgSpec.EnableNodeOUs)

	// TODO: add ability to specify usernames
	users := []NodeSpec{}
	for j := 1; j <= orgSpec.Users.Count; j++ {
		user := NodeSpec{
			CommonName: fmt.Sprintf("%s%d@%s", userBaseName, j, orgName),
		}

		users = append(users, user)
	}
	// add an admin user
	adminUser := NodeSpec{
		CommonName: fmt.Sprintf("%s@%s", adminBaseName, orgName),
	}

	users = append(users, adminUser)
	// 遍历所有的users生成对应的msp、tls以及其目录下的文件
	generateNodes(usersDir, users, signCA, tlsCA, msp.CLIENT, orgSpec.EnableNodeOUs)

	// copy the admin cert to the org's MSP admincerts
	// 复制users/admin/msp/admincerts下的证书到msp/admincerts
	err = copyAdminCert(usersDir, adminCertsDir, adminUser.CommonName)
	if err != nil {
		fmt.Printf("Error copying admin cert for org %s:\n%v\n",
			orgName, err)
		os.Exit(1)
	}

	// copy the admin cert to each of the org's peer's MSP admincerts
	// 复制users/admin/msp/admincerts下的证书到所有节点下msp/admincerts
	for _, spec := range orgSpec.Specs {
		err = copyAdminCert(usersDir,
			filepath.Join(peersDir, spec.CommonName, "msp", "admincerts"), adminUser.CommonName)
		if err != nil {
			fmt.Printf("Error copying admin cert for org %s peer %s:\n%v\n",
				orgName, spec.CommonName, err)
			os.Exit(1)
		}
	}
}
```

* NewCA
```go
// NewCA creates an instance of CA and saves the signing key pair in
// baseDir/name
func NewCA(baseDir, org, name, country, province, locality, orgUnit, streetAddress, postalCode string) (*CA, error) {

	var response error
	var ca *CA

	err := os.MkdirAll(baseDir, 0755) // 创建证书目录，并设置权限
	if err == nil {
		priv, signer, err := csp.GeneratePrivateKey(baseDir) // 生成私钥
		response = err
		if err == nil {
			// get public signing certificate
			ecPubKey, err := csp.GetECPublicKey(priv)       // 生成公钥
			response = err
			if err == nil {
				template := x509Template()                  // 创建X509Template模板
				//this is a CA
				template.IsCA = true
				template.KeyUsage |= x509.KeyUsageDigitalSignature |
					x509.KeyUsageKeyEncipherment | x509.KeyUsageCertSign |
					x509.KeyUsageCRLSign
				template.ExtKeyUsage = []x509.ExtKeyUsage{
					x509.ExtKeyUsageClientAuth,
					x509.ExtKeyUsageServerAuth,
				}

                //set the organization for the subject
                // 生成证书主题
				subject := subjectTemplateAdditional(country, province, locality, orgUnit, streetAddress, postalCode)
				subject.Organization = []string{org}    // 设置证书组织机构
				subject.CommonName = name               // 设置证书的服务器的主机名

				template.Subject = subject // 主题
				template.SubjectKeyId = priv.SKI()      // 获得私钥的SKI

				x509Cert, err := genCertificateECDSA(baseDir, name, &template, &template,
					ecPubKey, signer)                   // 获得证书
				response = err
				if err == nil {
					ca = &CA{                           // 证书对象赋值
						Name:               name,
						Signer:             signer,
						SignCert:           x509Cert,
						Country:            country,
						Province:           province,
						Locality:           locality,
						OrganizationalUnit: orgUnit,
						StreetAddress:      streetAddress,
						PostalCode:         postalCode,
					}
				}
			}
		}
	}
	return ca, response
}
```

* genCertificateECDSA
```go
// generate a signed X509 certificate using ECDSA
func genCertificateECDSA(baseDir, name string, template, parent *x509.Certificate, pub *ecdsa.PublicKey,
	priv interface{}) (*x509.Certificate, error) {

	//create the x509 public cert
	// 生成x509证书
	certBytes, err := x509.CreateCertificate(rand.Reader, template, parent, pub, priv)
	if err != nil {
		return nil, err
	}

	//write cert out to file
	fileName := filepath.Join(baseDir, name+"-cert.pem")
	certFile, err := os.Create(fileName)
	if err != nil {
		return nil, err
	}
	//pem encode the cert
	// 证书编码成pem格式文件
	err = pem.Encode(certFile, &pem.Block{Type: "CERTIFICATE", Bytes: certBytes})
	certFile.Close()
	if err != nil {
		return nil, err
	}

	x509Cert, err := x509.ParseCertificate(certBytes)
	if err != nil {
		return nil, err
	}
	return x509Cert, nil
}
```


* generateNodes
```go
// 遍历所有的peer生成对应的msp、tls以及其目录下的文件
func generateNodes(baseDir string, nodes []NodeSpec, signCA *ca.CA, tlsCA *ca.CA, nodeType int, nodeOUs bool) {

	for _, node := range nodes {
		nodeDir := filepath.Join(baseDir, node.CommonName)
		if _, err := os.Stat(nodeDir); os.IsNotExist(err) {
			err := msp.GenerateLocalMSP(nodeDir, node.CommonName, node.SANS, signCA, tlsCA, nodeType, nodeOUs)
			if err != nil {
				fmt.Printf("Error generating local MSP for %s:\n%v\n", node, err)
				os.Exit(1)
			}
		}
	}
}
```


* GenerateLocalMSP

```go
// 创建节点下msp和tls目录以及子目录和证书
func GenerateLocalMSP(baseDir, name string, sans []string, signCA *ca.CA,
	tlsCA *ca.CA, nodeType int, nodeOUs bool) error {

	// create folder structure
	mspDir := filepath.Join(baseDir, "msp")
	tlsDir := filepath.Join(baseDir, "tls")
	// 创建 msp/admincerts, msp/cacerts, msp/tlscacerts ,msp/keystore and msp/signcerts 目录
	err := createFolderStructure(mspDir, true)  
	if err != nil {
		return err
	}
    // 创建tls目录
	err = os.MkdirAll(tlsDir, 0755)
	if err != nil {
		return err
	}

	/*
		Create the MSP identity artifacts
	*/
	// get keystore path
	keystore := filepath.Join(mspDir, "keystore")

    // generate private key
    // 生成private key和保存到keystore目录下
	priv, _, err := csp.GeneratePrivateKey(keystore)
	if err != nil {
		return err
	}

	// get public key
	// 生成publickey
	ecPubKey, err := csp.GetECPublicKey(priv)
	if err != nil {
		return err
	}
	// generate X509 certificate using signing CA
	var ous []string
	if nodeOUs {
		ous = []string{nodeOUMap[nodeType]}
    }
    // 生成CA证书，并保存在msp/signcerts目录下
	cert, err := signCA.SignCertificate(filepath.Join(mspDir, "signcerts"),
		name, ous, nil, ecPubKey, x509.KeyUsageDigitalSignature, []x509.ExtKeyUsage{})
	if err != nil {
		return err
	}

	// write artifacts to MSP folders

	// the signing CA certificate goes into cacerts
	// 创建msp/cacerts/ca.org1.example.com-cert.pem
	err = x509Export(filepath.Join(mspDir, "cacerts", x509Filename(signCA.Name)), signCA.SignCert)
	if err != nil {
		return err
	}
	// the TLS CA certificate goes into tlscacerts
	// 创建msp/tlscacerts/tlsca.org1.example.com-cert.pem
	err = x509Export(filepath.Join(mspDir, "tlscacerts", x509Filename(tlsCA.Name)), tlsCA.SignCert)
	if err != nil {
		return err
	}

	// generate config.yaml if required
	if nodeOUs && nodeType == PEER {
		exportConfig(mspDir, "cacerts/"+x509Filename(signCA.Name), true)
	}

	// the signing identity goes into admincerts.
	// This means that the signing identity
	// of this MSP is also an admin of this MSP
	// NOTE: the admincerts folder is going to be
	// cleared up anyway by copyAdminCert, but
	// we leave a valid admin for now for the sake
	// of unit tests
	// 创建msp/admincerts/Admin@org1.example.com-cert.pem
	err = x509Export(filepath.Join(mspDir, "admincerts", x509Filename(name)), cert)
	if err != nil {
		return err
	}

	/*
		Generate the TLS artifacts in the TLS folder
	*/

	// generate private key
	// 生成私钥
	tlsPrivKey, _, err := csp.GeneratePrivateKey(tlsDir)
	if err != nil {
		return err
	}
	// get public key
	// 生成公钥
	tlsPubKey, err := csp.GetECPublicKey(tlsPrivKey)
	if err != nil {
		return err
	}
	// generate X509 certificate using TLS CA
	// 使用tls ca生成证书
	_, err = tlsCA.SignCertificate(filepath.Join(tlsDir),
		name, nil, sans, tlsPubKey, x509.KeyUsageDigitalSignature|x509.KeyUsageKeyEncipherment,
		[]x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth, x509.ExtKeyUsageClientAuth})
	if err != nil {
		return err
	}
	// 生成tls下ca.crt
	err = x509Export(filepath.Join(tlsDir, "ca.crt"), tlsCA.SignCert)
	if err != nil {
		return err
	}

	// rename the generated TLS X509 cert
	tlsFilePrefix := "server"
	if nodeType == CLIENT {
		tlsFilePrefix = "client"
	}
	// 生成tls下server.crt
	err = os.Rename(filepath.Join(tlsDir, x509Filename(name)),
		filepath.Join(tlsDir, tlsFilePrefix+".crt"))
	if err != nil {
		return err
	}
	// 生成tls下server.key
	err = keyExport(tlsDir, filepath.Join(tlsDir, tlsFilePrefix+".key"), tlsPrivKey)
	if err != nil {
		return err
	}

	return nil
}
```


* GeneratePrivateKey
```go
// GeneratePrivateKey creates a private key and stores it in keystorePath
func GeneratePrivateKey(keystorePath string) (bccsp.Key,
	crypto.Signer, error) {

	var err error
	var priv bccsp.Key
	var s crypto.Signer

	opts := &factory.FactoryOpts{
		ProviderName: "SW",         // 设置提供者的别名
		SwOpts: &factory.SwOpts{
			HashFamily: "SHA2",     // hash的算法
			SecLevel:   256,        // 长度

			FileKeystore: &factory.FileKeystoreOpts{
				KeyStorePath: keystorePath,             // 文件keyStore的存放位置
			},
		},
    }
    // 根据参数获得具体的算法对象
	csp, err := factory.GetBCCSPFromOpts(opts)
	if err == nil {
        // generate a key
        // 取得ECDSA算法 256长度的可以，Temporary 表明是否是临时key，false表明不是临时的。
		priv, err = csp.KeyGen(&bccsp.ECDSAP256KeyGenOpts{Temporary: false})
		if err == nil {
            // create a crypto.Signer
            // 获取私钥签名
			s, err = signer.New(csp, priv)
		}
	}
	return priv, s, err
}
```


### 6.生成证书目录结构
```bash
.
|-- ordererOrganizations
|   `-- example.com
|       |-- ca
|       |   |-- 159a7bd324dfcbacfbd81de4494c6794ef1db7b95044c12992669c605e4c448c_sk
|       |   `-- ca.example.com-cert.pem
|       |-- msp
|       |   |-- admincerts
|       |   |   `-- Admin@example.com-cert.pem
|       |   |-- cacerts
|       |   |   `-- ca.example.com-cert.pem
|       |   `-- tlscacerts
|       |       `-- tlsca.example.com-cert.pem
|       |-- orderers
|       |   |-- orderer2.example.com
|       |   |   |-- msp
|       |   |   |   |-- admincerts
|       |   |   |   |   `-- Admin@example.com-cert.pem
|       |   |   |   |-- cacerts
|       |   |   |   |   `-- ca.example.com-cert.pem
|       |   |   |   |-- keystore
|       |   |   |   |   `-- c3ef14f04019072094718940ea07a9db747c01acc3da4529960f219e6289bed0_sk
|       |   |   |   |-- signcerts
|       |   |   |   |   `-- orderer2.example.com-cert.pem
|       |   |   |   `-- tlscacerts
|       |   |   |       `-- tlsca.example.com-cert.pem
|       |   |   `-- tls
|       |   |       |-- ca.crt
|       |   |       |-- server.crt
|       |   |       `-- server.key
|       |   |-- orderer3.example.com
|       |   |   |-- msp
|       |   |   |   |-- admincerts
|       |   |   |   |   `-- Admin@example.com-cert.pem
|       |   |   |   |-- cacerts
|       |   |   |   |   `-- ca.example.com-cert.pem
|       |   |   |   |-- keystore
|       |   |   |   |   `-- 6756427b8bd7b63dc4ec9f1fa43810862fb2ffc61a312b7819c1779bf5d1c5cd_sk
|       |   |   |   |-- signcerts
|       |   |   |   |   `-- orderer3.example.com-cert.pem
|       |   |   |   `-- tlscacerts
|       |   |   |       `-- tlsca.example.com-cert.pem
|       |   |   `-- tls
|       |   |       |-- ca.crt
|       |   |       |-- server.crt
|       |   |       `-- server.key
|       |   |-- orderer4.example.com
|       |   |   |-- msp
|       |   |   |   |-- admincerts
|       |   |   |   |   `-- Admin@example.com-cert.pem
|       |   |   |   |-- cacerts
|       |   |   |   |   `-- ca.example.com-cert.pem
|       |   |   |   |-- keystore
|       |   |   |   |   `-- b90988974424013728038af20b3fed21ec5ba4d620e20e5ff91e7bd56b599aee_sk
|       |   |   |   |-- signcerts
|       |   |   |   |   `-- orderer4.example.com-cert.pem
|       |   |   |   `-- tlscacerts
|       |   |   |       `-- tlsca.example.com-cert.pem
|       |   |   `-- tls
|       |   |       |-- ca.crt
|       |   |       |-- server.crt
|       |   |       `-- server.key
|       |   |-- orderer5.example.com
|       |   |   |-- msp
|       |   |   |   |-- admincerts
|       |   |   |   |   `-- Admin@example.com-cert.pem
|       |   |   |   |-- cacerts
|       |   |   |   |   `-- ca.example.com-cert.pem
|       |   |   |   |-- keystore
|       |   |   |   |   `-- b0cc6b9070b74893edb3703432a3bb96c83651636bb97ea13de5a95760789da9_sk
|       |   |   |   |-- signcerts
|       |   |   |   |   `-- orderer5.example.com-cert.pem
|       |   |   |   `-- tlscacerts
|       |   |   |       `-- tlsca.example.com-cert.pem
|       |   |   `-- tls
|       |   |       |-- ca.crt
|       |   |       |-- server.crt
|       |   |       `-- server.key
|       |   `-- orderer.example.com
|       |       |-- msp
|       |       |   |-- admincerts
|       |       |   |   `-- Admin@example.com-cert.pem
|       |       |   |-- cacerts
|       |       |   |   `-- ca.example.com-cert.pem
|       |       |   |-- keystore
|       |       |   |   `-- 021316baae49052e4c741c16f2d79f9ce49938d15d15e23f9c78f8b80cda9fca_sk
|       |       |   |-- signcerts
|       |       |   |   `-- orderer.example.com-cert.pem
|       |       |   `-- tlscacerts
|       |       |       `-- tlsca.example.com-cert.pem
|       |       `-- tls
|       |           |-- ca.crt
|       |           |-- server.crt
|       |           `-- server.key
|       |-- tlsca
|       |   |-- e08dc7c2ed7a816f3db9e2e97de02a665b7e8803e905cae88b780ff5383c6b01_sk
|       |   `-- tlsca.example.com-cert.pem
|       `-- users
|           `-- Admin@example.com
|               |-- msp
|               |   |-- admincerts
|               |   |   `-- Admin@example.com-cert.pem
|               |   |-- cacerts
|               |   |   `-- ca.example.com-cert.pem
|               |   |-- keystore
|               |   |   `-- 1205e162b707676db8011c3743db8b5fb1ac2e674a9cac69927b9ba07ea4f9df_sk
|               |   |-- signcerts
|               |   |   `-- Admin@example.com-cert.pem
|               |   `-- tlscacerts
|               |       `-- tlsca.example.com-cert.pem
|               `-- tls
|                   |-- ca.crt
|                   |-- client.crt
|                   `-- client.key
`-- peerOrganizations
    |-- org1.example.com
    |   |-- ca	# 存放组织Org1的根证书和对应的私钥文件，默认采用EC算法，证书为自签名。组织内的实体将基于该根证书作为证书根。
    |   |   |-- ca.org1.example.com-cert.pem
    |   |   `-- d2d37c565c8480f03f17c702bf1156e005b2591aa87f510cabc11a476965f417_sk
    |   |-- msp 				# 存放代表该组织的身份信息
    |   |   |-- admincerts		# 组织管理员的身份验证证书，被根证书签名。
    |   |   |   `-- Admin@org1.example.com-cert.pem
    |   |   |-- cacerts			# 组织的根证书，同ca目录下文件。
    |   |   |   `-- ca.org1.example.com-cert.pem
    |   |   |-- config.yaml
    |   |   `-- tlscacerts		# 用于TLS的CA证书，自签名。
    |   |       `-- tlsca.org1.example.com-cert.pem
    |   |-- peers						# 存放属于该组织的所有Peer节点
    |   |   |-- peer0.org1.example.com	# 第一个peer的信息，包括其msp证书和tls证书两类。
    |   |   |   |-- msp					# msp相关证书   
    |   |   |   |   |-- admincerts		# 组织管理员的身份验证证书。Peer将基于这些证书来认证交易签署者是否为管理员身份。
    |   |   |   |   |   `-- Admin@org1.example.com-cert.pem
    |   |   |   |   |-- cacerts			# 存放组织的根证书
    |   |   |   |   |   `-- ca.org1.example.com-cert.pem
    |   |   |   |   |-- config.yaml
    |   |   |   |   |-- keystore		# 本节点的身份私钥，用来签名
    |   |   |   |   |   `-- cb7fed83abec4f4e4c38ab6e02d58bf17498da88ff9890db62022b3ab0e895d0_sk
    |   |   |   |   |-- signcerts		# 验证本节点签名的证书，被组织根证书签名 
    |   |   |   |   |   `-- peer0.org1.example.com-cert.pem
    |   |   |   |   `-- tlscacerts		# TLS连接用到身份证书，即组织TLS证书
    |   |   |   |       `-- tlsca.org1.example.com-cert.pem
    |   |   |   `-- tls					# tls相关证书
    |   |   |       |-- ca.crt			# 组织的根证书
    |   |   |       |-- server.crt		# 验证本节点签名的证书，被组织根证书签名
    |   |   |       `-- server.key		# 本节点的身份私钥，用来签名
    |   |   `-- peer1.org1.example.com	# 第二个peer的信息，结构类似。（此处省略。）
    |   |       |-- msp
    |   |       |   |-- admincerts
    |   |       |   |   `-- Admin@org1.example.com-cert.pem
    |   |       |   |-- cacerts
    |   |       |   |   `-- ca.org1.example.com-cert.pem
    |   |       |   |-- config.yaml
    |   |       |   |-- keystore
    |   |       |   |   `-- 9bd682210c20c745f965b3edce4bcff7af649c4791942e475ea92b5e4f2c2a2c_sk
    |   |       |   |-- signcerts
    |   |       |   |   `-- peer1.org1.example.com-cert.pem
    |   |       |   `-- tlscacerts
    |   |       |       `-- tlsca.org1.example.com-cert.pem
    |   |       `-- tls
    |   |           |-- ca.crt
    |   |           |-- server.crt
    |   |           `-- server.key
    |   |-- tlsca						# 存放tls相关的证书和私钥。
    |   |   |-- 17b3da18b1f78c4baa09c52984d8146a47e2c5ecc43f1c1e8c0a72c2fd21245f_sk
    |   |   `-- tlsca.org1.example.com-cert.pem
    |   `-- users						# 存放属于该组织的用户的实体
    |       |-- Admin@org1.example.com	# 管理员用户的信息，其中包括msp证书和tls证书两类。
    |       |   |-- msp					# msp相关证书
    |       |   |   |-- admincerts		# 组织根证书作为管理员身份验证证书 
    |       |   |   |   `-- Admin@org1.example.com-cert.pem
    |       |   |   |-- cacerts			# 存放组织的根证书
    |       |   |   |   `-- ca.org1.example.com-cert.pem
    |       |   |   |-- keystore		# 本用户的身份私钥，用来签名
    |       |   |   |   `-- d93142f02dca95a481af2c80b347e734dbc5835f4dbd4e6c9739e8f3a4e62669_sk
    |       |   |   |-- signcerts		# 管理员用户的身份验证证书，被组织根证书签名。要被某个Peer认可，则必须放到该Peer的msp/admincerts目录下
    |       |   |   |   `-- Admin@org1.example.com-cert.pem
    |       |   |   `-- tlscacerts		# TLS连接用的身份证书，即组织TLS证书
    |       |   |       `-- tlsca.org1.example.com-cert.pem
    |       |   `-- tls					# 存放tls相关的证书和私钥。
    |       |       |-- ca.crt			# 组织的根证书
    |       |       |-- client.crt		# 管理员的用户身份验证证书，被组织根证书签名
    |       |       `-- client.key		# 管理员用户的身份私钥，被组织根证书签名
    |       `-- User1@org1.example.com	# 第一个用户的信息，包括msp证书和tls证书两类
    |           |-- msp					# msp证书相关信息
    |           |   |-- admincerts		# 组织根证书作为管理者身份验证证书。
    |           |   |   `-- User1@org1.example.com-cert.pem
    |           |   |-- cacerts			# 存放组织的根证书
    |           |   |   `-- ca.org1.example.com-cert.pem
    |           |   |-- keystore		# 本用户的身份私钥，用来签名
    |           |   |   `-- eaaaa98f8f37d22373838a15eb85fb1816a28cb6dc611cd1852a97eb7808c79a_sk
    |           |   |-- signcerts		# 验证本用户签名的身份证书，被组织根证书签名
    |           |   |   `-- User1@org1.example.com-cert.pem
    |           |   `-- tlscacerts		# TLS连接用的身份证书，被组织根证书签名。
    |           |       `-- tlsca.org1.example.com-cert.pem
    |           `-- tls					# 组织的根证书
    |               |-- ca.crt			# 组织的根证书
    |               |-- client.crt		# 验证用户签名的身份证书，被根组织证书签名
    |               `-- client.key		# 用户的身份私钥用来签名。
    `-- org2.example.com				# 跟org1.example.com类似
        |-- ca
        |   |-- ca.org2.example.com-cert.pem
        |   `-- dfeac21a81040e73ea7ffe97c9bad347b71bc6182b0ab538a76681400d0e480a_sk
        |-- msp
        |   |-- admincerts
        |   |   `-- Admin@org2.example.com-cert.pem
        |   |-- cacerts
        |   |   `-- ca.org2.example.com-cert.pem
        |   |-- config.yaml
        |   `-- tlscacerts
        |       `-- tlsca.org2.example.com-cert.pem
        |-- peers
        |   |-- peer0.org2.example.com
        |   |   |-- msp
        |   |   |   |-- admincerts
        |   |   |   |   `-- Admin@org2.example.com-cert.pem
        |   |   |   |-- cacerts
        |   |   |   |   `-- ca.org2.example.com-cert.pem
        |   |   |   |-- config.yaml
        |   |   |   |-- keystore
        |   |   |   |   `-- 042b26104990a0819c15cee318f52b86068efa9d53912f4fcd6bc50a8bd92c27_sk
        |   |   |   |-- signcerts
        |   |   |   |   `-- peer0.org2.example.com-cert.pem
        |   |   |   `-- tlscacerts
        |   |   |       `-- tlsca.org2.example.com-cert.pem
        |   |   `-- tls
        |   |       |-- ca.crt
        |   |       |-- server.crt
        |   |       `-- server.key
        |   `-- peer1.org2.example.com
        |       |-- msp
        |       |   |-- admincerts
        |       |   |   `-- Admin@org2.example.com-cert.pem
        |       |   |-- cacerts
        |       |   |   `-- ca.org2.example.com-cert.pem
        |       |   |-- config.yaml
        |       |   |-- keystore
        |       |   |   `-- 204765709153d413947e63f5bdc2941593a8a30c482dc4789487e17f07316a18_sk
        |       |   |-- signcerts
        |       |   |   `-- peer1.org2.example.com-cert.pem
        |       |   `-- tlscacerts
        |       |       `-- tlsca.org2.example.com-cert.pem
        |       `-- tls
        |           |-- ca.crt
        |           |-- server.crt
        |           `-- server.key
        |-- tlsca
        |   |-- fad3a1cd6bd78d6e6d29cf14d5f5699adee559ba785ceb3915e5fdd90c4553a3_sk
        |   `-- tlsca.org2.example.com-cert.pem
        `-- users
            |-- Admin@org2.example.com
            |   |-- msp
            |   |   |-- admincerts
            |   |   |   `-- Admin@org2.example.com-cert.pem
            |   |   |-- cacerts
            |   |   |   `-- ca.org2.example.com-cert.pem
            |   |   |-- keystore
            |   |   |   `-- bed9ee88a142cda5250023b799d75630606eb46b6403ad123db57ab4577b59f0_sk
            |   |   |-- signcerts
            |   |   |   `-- Admin@org2.example.com-cert.pem
            |   |   `-- tlscacerts
            |   |       `-- tlsca.org2.example.com-cert.pem
            |   `-- tls
            |       |-- ca.crt
            |       |-- client.crt
            |       `-- client.key
            `-- User1@org2.example.com
                |-- msp
                |   |-- admincerts
                |   |   `-- User1@org2.example.com-cert.pem
                |   |-- cacerts
                |   |   `-- ca.org2.example.com-cert.pem
                |   |-- keystore
                |   |   `-- 52529321de2d1f39fc94de726c91dee6dd81668aad94f5a2530df6164eb11670_sk
                |   |-- signcerts
                |   |   `-- User1@org2.example.com-cert.pem
                |   `-- tlscacerts
                |       `-- tlsca.org2.example.com-cert.pem
                `-- tls
                    |-- ca.crt
                    |-- client.crt
                    `-- client.key
```


参考:   
[Hyperledger-Fabric源码分析（MSP-证书生成）](https://www.jianshu.com/p/026fecf07e8a)       
[fabric源码阅读第一篇中](https://blog.csdn.net/qq_15693861/article/details/81133418)      
[fabric源码阅读第一篇](https://blog.csdn.net/qq_15693861/article/details/81121295)
