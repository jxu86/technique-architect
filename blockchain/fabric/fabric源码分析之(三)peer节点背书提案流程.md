
`environment:`      
`fabric v1.4.2`


## 1. 概述
Endorser节点是peer节点所扮演的一种角色，在peer启动时会创建Endorser背书服务器，并注册到本地gRPC服务器（7051端口）上对外提供服务，对请求的签名提案消息执行启动链码容器、模拟执行链码、背书签名等流程。所有客户端提交到账本的调用交易都需要背书节点背书，当客户端收集到足够的背书信息之后，再将签名提案消息、模拟执行的结果以及背书信息打包成交易信息发给orderer节点排序出块
背书者Endorser在一个交易流中充当的作用如下：    
* 客户端发送一个背书申请（SignedProposal）到Endorser。    
* Endorser对申请进行背书，发送一个申请应答（ProposalResponse）到客户端。  
* 客户端将申请应答中的背书组装到一个交易请求（SignedTransaction）中。 

## 2. 背书服务初始化   
定位到`peer/node/start.go`的`serve`函数，这个是peer节点的启动初始化函数，下面为关键的背书节点启动语句:  
```golang
serverEndorser := endorser.NewEndorserServer(privDataDist, endorserSupport, pr, metricsProvider)
...
// start the peer server
auth := authHandler.ChainFilters(serverEndorser, authFilters...)
// Register the Endorser server
// 设置完之后注册背书服务
pb.RegisterEndorserServer(peerServer.Server(), auth)
```
背书服务最重要的接口为,位置为`protos\peer\peer.pb.go`:
```golang
// EndorserServer is the server API for Endorser service.
type EndorserServer interface {
	ProcessProposal(context.Context, *SignedProposal) (*ProposalResponse, error)
}
```
ProcessProposal()服务接口主要功能为接收和处理签名提案消息(SignedProposal)、启动链码容器、执行调用链码以及进行签名背书。     
函数定义的位置为`core/endorser/endorser.go`


## 3. 背书服务 
在ProcessProposal()服务中，主要存在以下流程：
* 首先对提案进行预处理`preProcess()`
    * 这一步主要就是对提案中的内容进行相关验证操作。
    * 验证Header信息
    * 验证证书信息
    * 判断调用的链码类型与通道信息。  
* 然后对提案进行模拟`SimulateProposal()`
    * 获取调用的链码的具体功能与参数。
    * 判断链码类型，用户链码需要检查实例化策略，系统链码只获取版本信息。
    * 创建Tx模拟器，调用`callChaincode()`方法进行模拟。
    * 记录模拟时间，执行链码，判断是否调用的是lscc，功能为upgrade或者为deploy。如果是的话进行链码的Init。
    * 对模拟完成的账本进行快照，返回模拟结果集。
* 最后进行背书操作`endorseProposal()`
    * 获取进行背书操作的链码
    * 获取链码事件与链码版本信息
    * 获取背书所需要的插件，获取调用链码的相关数据
    * 通过获取的插件进行背书操作
    * 返回背书响应


源码如下:   
```golang
// ProcessProposal process the Proposal
func (e *Endorser) ProcessProposal(ctx context.Context, signedProp *pb.SignedProposal) (*pb.ProposalResponse, error) {
	// start time for computing elapsed time metric for successfully endorsed proposals
	// 首先获取Peer节点处理提案开始的时间
	startTime := time.Now()
	// Peer节点接收到的提案数+1
	e.Metrics.ProposalsReceived.Add(1)
	// 从上下文中获取发起提案的地址
	addr := util.ExtractRemoteAddress(ctx)
	// 日志输出
	endorserLogger.Debug("Entering: request from", addr)

	// variables to capture proposal duration metric
	// 这个不是链码ID，是通道ID
	var chainID string
	var hdrExt *pb.ChaincodeHeaderExtension
	var success bool
	// 这个会在方法结束的时候调用
	defer func() {
		// capture proposal duration metric. hdrExt == nil indicates early failure
		// where we don't capture latency metric. But the ProposalValidationFailed
		// counter metric should shed light on those failures.
		// 判断chaincodeHeaderExtension是否为空，如果为空的话提案验证失败
		if hdrExt != nil {
			meterLabels := []string{
				"channel", chainID,
				"chaincode", hdrExt.ChaincodeId.Name + ":" + hdrExt.ChaincodeId.Version,
				"success", strconv.FormatBool(success),
			}
			e.Metrics.ProposalDuration.With(meterLabels...).Observe(time.Since(startTime).Seconds())
		}

		endorserLogger.Debug("Exit: request from", addr)
	}()

	// 0 -- check and validate
	// 到了第一个重要的方法，对已签名的提案进行预处理，点进行看一下
	vr, err := e.preProcess(signedProp)
	if err != nil {
		resp := vr.resp
		return resp, err
	}

	prop, hdrExt, chainID, txid := vr.prop, vr.hdrExt, vr.chainID, vr.txid

	// obtaining once the tx simulator for this proposal. This will be nil
	// for chainless proposals
	// Also obtain a history query executor for history queries, since tx simulator does not cover history
	// 这里定义了一个Tx模拟器，用于后面的模拟交易过程,如果通道Id为空，那么TxSimulator也是空
	var txsim ledger.TxSimulator
	// 定义一个历史记录查询器
	var historyQueryExecutor ledger.HistoryQueryExecutor
	// 判断是否需要Tx模拟
	if acquireTxSimulator(chainID, vr.hdrExt.ChaincodeId) {
		// 根据通道ID获取Tx模拟器
		if txsim, err = e.s.GetTxSimulator(chainID, txid); err != nil {
			return &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}, nil
		}

		// txsim acquires a shared lock on the stateDB. As this would impact the block commits (i.e., commit
		// of valid write-sets to the stateDB), we must release the lock as early as possible.
		// Hence, this txsim object is closed in simulateProposal() as soon as the tx is simulated and
		// rwset is collected before gossip dissemination if required for privateData. For safety, we
		// add the following defer statement and is useful when an error occur. Note that calling
		// txsim.Done() more than once does not cause any issue. If the txsim is already
		// released, the following txsim.Done() simply returns.
		defer txsim.Done()
		// 获取历史记录查询器
		if historyQueryExecutor, err = e.s.GetHistoryQueryExecutor(chainID); err != nil {
			return &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}, nil
		}
	}
	// 定义一个交易参数结构体，用于下面的方法,里面的字段之前都有说过
	txParams := &ccprovider.TransactionParams{
		ChannelID:            chainID,
		TxID:                 txid,
		SignedProp:           signedProp,
		Proposal:             prop,
		TXSimulator:          txsim,
		HistoryQueryExecutor: historyQueryExecutor,
	}
	// this could be a request to a chainless SysCC

	// TODO: if the proposal has an extension, it will be of type ChaincodeAction;
	//       if it's present it means that no simulation is to be performed because
	//       we're trying to emulate a submitting peer. On the other hand, we need
	//       to validate the supplied action before endorsing it

	// 1 -- simulate
	// 对交易进行模拟
	cd, res, simulationResult, ccevent, err := e.SimulateProposal(txParams, hdrExt.ChaincodeId)
	if err != nil {
		return &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}, nil
	}
	if res != nil {
		if res.Status >= shim.ERROR {
			endorserLogger.Errorf("[%s][%s] simulateProposal() resulted in chaincode %s response status %d for txid: %s", chainID, shorttxid(txid), hdrExt.ChaincodeId, res.Status, txid)
			var cceventBytes []byte
			if ccevent != nil {
				cceventBytes, err = putils.GetBytesChaincodeEvent(ccevent)
				if err != nil {
					return nil, errors.Wrap(err, "failed to marshal event bytes")
				}
			}
			pResp, err := putils.CreateProposalResponseFailure(prop.Header, prop.Payload, res, simulationResult, cceventBytes, hdrExt.ChaincodeId, hdrExt.PayloadVisibility)
			if err != nil {
				return &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}, nil
			}

			return pResp, nil
		}
	}

	// 2 -- endorse and get a marshalled ProposalResponse message
	var pResp *pb.ProposalResponse

	// TODO till we implement global ESCC, CSCC for system chaincodes
	// chainless proposals (such as CSCC) don't have to be endorsed
	if chainID == "" {
		pResp = &pb.ProposalResponse{Response: res}
	} else {
		// Note: To endorseProposal(), we pass the released txsim. Hence, an error would occur if we try to use this txsim
		// 开始背书
		pResp, err = e.endorseProposal(ctx, chainID, txid, signedProp, prop, res, simulationResult, ccevent, hdrExt.PayloadVisibility, hdrExt.ChaincodeId, txsim, cd)

		// if error, capture endorsement failure metric
		meterLabels := []string{
			"channel", chainID,
			"chaincode", hdrExt.ChaincodeId.Name + ":" + hdrExt.ChaincodeId.Version,
		}

		if err != nil {
			meterLabels = append(meterLabels, "chaincodeerror", strconv.FormatBool(false))
			e.Metrics.EndorsementsFailed.With(meterLabels...).Add(1)
			return &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}, nil
		}
		if pResp.Response.Status >= shim.ERRORTHRESHOLD {
			// the default ESCC treats all status codes about threshold as errors and fails endorsement
			// useful to track this as a separate metric
			meterLabels = append(meterLabels, "chaincodeerror", strconv.FormatBool(true))
			e.Metrics.EndorsementsFailed.With(meterLabels...).Add(1)
			endorserLogger.Debugf("[%s][%s] endorseProposal() resulted in chaincode %s error for txid: %s", chainID, shorttxid(txid), hdrExt.ChaincodeId, txid)
			return pResp, nil
		}
	}

	// Set the proposal response payload - it
	// contains the "return value" from the
	// chaincode invocation
	pResp.Response = res

	// total failed proposals = ProposalsReceived-SuccessfulProposals
	e.Metrics.SuccessfulProposals.Add(1)
	success = true

	return pResp, nil
}
```

### 3.1 检查和校验签名提案的合法性
`preProcess()`方法对签名提案消息进行预处理，主要包括验证消息格式和签名的合法性、验证提案消息对应链码检查是否是系统链码并且不为外部调用、交易的唯一性、验证是否满足对应通道的访问控制策略。
```golang
// preProcess checks the tx proposal headers, uniqueness and ACL
func (e *Endorser) preProcess(signedProp *pb.SignedProposal) (*validateResult, error) {
	vr := &validateResult{}
	// at first, we check whether the message is valid
	// 验证信息是否有效
	prop, hdr, hdrExt, err := validation.ValidateProposalMessage(signedProp)

	if err != nil {
		e.Metrics.ProposalValidationFailed.Add(1)
		vr.resp = &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}
		return vr, err
	}
	// 从提案的Header中获取通道Header信息
	chdr, err := putils.UnmarshalChannelHeader(hdr.ChannelHeader)
	if err != nil {
		vr.resp = &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}
		return vr, err
	}
	//获取签名域的Header
	shdr, err := putils.GetSignatureHeader(hdr.SignatureHeader)
	if err != nil {
		vr.resp = &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}
		return vr, err
	}

    // block invocations to security-sensitive system chaincodes
    // 根据提案消息头部hdrExt.ChaincodeId.Name链码名检查链码是否为允许外部调用的系统链码
	if e.s.IsSysCCAndNotInvokableExternal(hdrExt.ChaincodeId.Name) {
		endorserLogger.Errorf("Error: an attempt was made by %#v to invoke system chaincode %s", shdr.Creator, hdrExt.ChaincodeId.Name)
		err = errors.Errorf("chaincode %s cannot be invoked through a proposal", hdrExt.ChaincodeId.Name)
		vr.resp = &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}
		return vr, err
	}

	chainID := chdr.ChannelId
	txid := chdr.TxId
	endorserLogger.Debugf("[%s][%s] processing txid: %s", chainID, shorttxid(txid), txid)

	if chainID != "" {
		// labels that provide context for failure metrics
		meterLabels := []string{
			"channel", chainID,
			"chaincode", hdrExt.ChaincodeId.Name + ":" + hdrExt.ChaincodeId.Version,
		}

		// Here we handle uniqueness check and ACLs for proposals targeting a chain
		// Notice that ValidateProposalMessage has already verified that TxID is computed properly
		if _, err = e.s.GetTransactionByID(chainID, txid); err == nil {
			// increment failure due to duplicate transactions. Useful for catching replay attacks in
			// addition to benign retries
			e.Metrics.DuplicateTxsFailure.With(meterLabels...).Add(1)
			err = errors.Errorf("duplicate transaction found [%s]. Creator [%x]", txid, shdr.Creator)
			vr.resp = &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}
			return vr, err
		}

		// check ACL only for application chaincodes; ACLs
		// for system chaincodes are checked elsewhere
		if !e.s.IsSysCC(hdrExt.ChaincodeId.Name) {
			// check that the proposal complies with the Channel's writers
			if err = e.s.CheckACL(signedProp, chdr, shdr, hdrExt); err != nil {
				e.Metrics.ProposalACLCheckFailed.With(meterLabels...).Add(1)
				vr.resp = &pb.ProposalResponse{Response: &pb.Response{Status: 500, Message: err.Error()}}
				return vr, err
			}
		}
	} else {
		// chainless proposals do not/cannot affect ledger and cannot be submitted as transactions
		// ignore uniqueness checks; also, chainless proposals are not validated using the policies
		// of the chain since by definition there is no chain; they are validated against the local
		// MSP of the peer instead by the call to ValidateProposalMessage above
	}

	vr.prop, vr.hdrExt, vr.chainID, vr.txid = prop, hdrExt, chainID, txid
	return vr, nil
}
```

#### 3.1.1 验证消息格式和签名合法性
`preProcess()`调用`ValidateProposalMessage()`对消息进行验证，主要针对消息的格式、签名、交易id进行验证。     
在`core/common/validation/msgvalidation.go`找到`ValidateProposalMessage`函数    
```golang
// ValidateProposalMessage checks the validity of a SignedProposal message
// this function returns Header and ChaincodeHeaderExtension messages since they
// have been unmarshalled and validated
func ValidateProposalMessage(signedProp *pb.SignedProposal) (*pb.Proposal, *common.Header, *pb.ChaincodeHeaderExtension, error) {
	if signedProp == nil {
		return nil, nil, nil, errors.New("nil arguments")
	}

	putilsLogger.Debugf("ValidateProposalMessage starts for signed proposal %p", signedProp)

	// extract the Proposal message from signedProp
	// 从提案中获取Proposal内容
	prop, err := utils.GetProposal(signedProp.ProposalBytes)
	if err != nil {
		return nil, nil, nil, err
	}

	// 1) look at the ProposalHeader
	// 从Proposal中获取Header
	hdr, err := utils.GetHeader(prop.Header)
	if err != nil {
		return nil, nil, nil, err
	}

	// validate the header
	// 对header进行验证
	chdr, shdr, err := validateCommonHeader(hdr)
	if err != nil {
		return nil, nil, nil, err
	}

	// validate the signature
	// 验证签名
	err = checkSignatureFromCreator(shdr.Creator, signedProp.Signature, signedProp.ProposalBytes, chdr.ChannelId)
	if err != nil {
		// log the exact message on the peer but return a generic error message to
		// avoid malicious users scanning for channels
		putilsLogger.Warningf("channel [%s]: %s", chdr.ChannelId, err)
		sId := &msp.SerializedIdentity{}
		err := proto.Unmarshal(shdr.Creator, sId)
		if err != nil {
			// log the error here as well but still only return the generic error
			err = errors.Wrap(err, "could not deserialize a SerializedIdentity")
			putilsLogger.Warningf("channel [%s]: %s", chdr.ChannelId, err)
		}
		return nil, nil, nil, errors.Errorf("access denied: channel [%s] creator org [%s]", chdr.ChannelId, sId.Mspid)
	}

	// Verify that the transaction ID has been computed properly.
	// This check is needed to ensure that the lookup into the ledger
    // for the same TxID catches duplicates.
    // 对交易id进行验证，验证交易id是否与计算的交易id一致
	err = utils.CheckTxID(
		chdr.TxId,
		shdr.Nonce,
		shdr.Creator)
	if err != nil {
		return nil, nil, nil, err
	}

    // continue the validation in a way that depends on the type specified in the header
    // 根据消息类型进行分类处理
	switch common.HeaderType(chdr.Type) {
	case common.HeaderType_CONFIG:
		//which the types are different the validation is the same
		//viz, validate a proposal to a chaincode. If we need other
		//special validation for confguration, we would have to implement
		//special validation
		fallthrough
	case common.HeaderType_ENDORSER_TRANSACTION:
		// validation of the proposal message knowing it's of type CHAINCODE
		chaincodeHdrExt, err := validateChaincodeProposalMessage(prop, hdr)
		if err != nil {
			return nil, nil, nil, err
		}

		return prop, hdr, chaincodeHdrExt, err
	default:
		//NOTE : we proably need a case
		return nil, nil, nil, errors.Errorf("unsupported proposal type %d", common.HeaderType(chdr.Type))
	}
}

```
`validateCommonHeader()`校验`Proposal.Header`的合法性
```golang
// checks for a valid Header
func validateCommonHeader(hdr *common.Header) (*common.ChannelHeader, *common.SignatureHeader, error) {
	if hdr == nil {
		return nil, nil, errors.New("nil header")
	}

	chdr, err := utils.UnmarshalChannelHeader(hdr.ChannelHeader)
	if err != nil {
		return nil, nil, err
	}

	shdr, err := utils.GetSignatureHeader(hdr.SignatureHeader)
	if err != nil {
		return nil, nil, err
	}
	// 校验消息类型是否属于HeaderType_ENDORSER_TRANSACTION、HeaderType_CONFIG_UPDATE、HeaderType_CONFIG、HeaderType_TOKEN_TRANSACTION，并且校验Epoch是否为0
	err = validateChannelHeader(chdr)
	if err != nil {
		return nil, nil, err
	}
	// 校验shdr shdr.Nonce  shdr.Creator是否为nil，或长度是否为0
	err = validateSignatureHeader(shdr)
	if err != nil {
		return nil, nil, err
	}

	return chdr, shdr, nil
}
```
`checkSignatureFromCreator()`对签名进行校验
```golang
// given a creator, a message and a signature,
// this function returns nil if the creator
// is a valid cert and the signature is valid
func checkSignatureFromCreator(creatorBytes []byte, sig []byte, msg []byte, ChainID string) error {
	putilsLogger.Debugf("begin")

	// check for nil argument
	if creatorBytes == nil || sig == nil || msg == nil {
		return errors.New("nil arguments")
	}

	mspObj := mspmgmt.GetIdentityDeserializer(ChainID)
	if mspObj == nil {
		return errors.Errorf("could not get msp for channel [%s]", ChainID)
	}

	// get the identity of the creator
	creator, err := mspObj.DeserializeIdentity(creatorBytes)
	if err != nil {
		return errors.WithMessage(err, "MSP error")
	}

	putilsLogger.Debugf("creator is %s", creator.GetIdentifier())

	// ensure that creator is a valid certificate
	err = creator.Validate()
	if err != nil {
		return errors.WithMessage(err, "creator certificate is not valid")
	}

	putilsLogger.Debugf("creator is valid")

	// validate the signature
	err = creator.Verify(msg, sig)
	if err != nil {
		return errors.WithMessage(err, "creator's signature over the proposal is not valid")
	}

	putilsLogger.Debugf("exits successfully")

	return nil
}
```

#### 3.1.2 检查是否是系统链码并且不为外部调用
定位到`core/scc/sccproviderimpl.go`的`IsSysCCAndNotInvokableExternal`函数   
简单理解就是链码是否可以为外部调用
```golang
// IsSysCCAndNotInvokableExternal returns true if the chaincode
// is a system chaincode and *CANNOT* be invoked through
// a proposal to this peer
func (p *Provider) IsSysCCAndNotInvokableExternal(name string) bool {
	for _, sysCC := range p.SysCCs {
		if sysCC.Name() == name {
			return !sysCC.InvokableExternal()
		}
	}

	if isDeprecatedSysCC(name) {
		return true
	}

	return false
}

func isDeprecatedSysCC(name string) bool {
	return name == "vscc" || name == "escc"
}

```

#### 3.1.3 检查签名提案消息交易id的唯一性
首先查看是否存在该账本，然后查看账本是否存在该交易id。
```go
// GetTransactionByID retrieves a transaction by id
func (s *SupportImpl) GetTransactionByID(chid, txID string) (*pb.ProcessedTransaction, error) {
	lgr := s.Peer.GetLedger(chid)
	if lgr == nil {
		return nil, errors.Errorf("failed to look up the ledger for Channel %s", chid)
	}
	tx, err := lgr.GetTransactionByID(txID)
	if err != nil {
		return nil, errors.WithMessage(err, "GetTransactionByID failed")
	}
	return tx, nil
}
```


#### 3.1.4 验证是否满足对应通道的访问控制策略
背书节点在背书过程中会检查是否满足应用通道的`Writers`策略
```go
// CheckACL checks the ACL for the resource for the Channel using the
// SignedProposal from which an id can be extracted for testing against a policy
func (s *SupportImpl) CheckACL(signedProp *pb.SignedProposal, chdr *common.ChannelHeader, shdr *common.SignatureHeader, hdrext *pb.ChaincodeHeaderExtension) error {
	return s.ACLProvider.CheckACL(resources.Peer_Propose, chdr.ChannelId, signedProp)
}
```


### 3.2 调用链码并模拟执行提案
首先，`ProcessProposal()`方法调用方法`acquireTxSimulator()`根据链码判断是否需要创建交易模拟器`TxSimulator`，如果需要则创建交易模拟器`TxSimulator`（无法查询历史记录）以及历史记录查询器`HistoryQueryExecutor`，接着再调用`SimulateProposal()`模拟执行交易提案消息，并返回模拟执行结果。
其中，链码qscc、cscc不需要交易模拟器。
```
// determine whether or not a transaction simulator should be
// obtained for a proposal.
func acquireTxSimulator(chainID string, ccid *pb.ChaincodeID) bool {
	// 如果通道ID为空,就说明不需要进行Tx的模拟
	if chainID == "" {
		return false
	}

	// ¯\_(ツ)_/¯ locking.
	// Don't get a simulator for the query and config system chaincode.
	// These don't need the simulator and its read lock results in deadlocks.
	// 通道ID不为空，则判断链码的类型，如果是qscc(查询系统链码),cscc(配置系统链码)，则不需要进行Tx模拟
	switch ccid.Name {
	case "qscc", "cscc":
		return false
	default:
		return true
	}
}
```

```go
// SimulateProposal simulates the proposal by calling the chaincode
func (e *Endorser) SimulateProposal(txParams *ccprovider.TransactionParams, cid *pb.ChaincodeID) (ccprovider.ChaincodeDefinition, *pb.Response, []byte, *pb.ChaincodeEvent, error) {
	endorserLogger.Debugf("[%s][%s] Entry chaincode: %s", txParams.ChannelID, shorttxid(txParams.TxID), cid)
	defer endorserLogger.Debugf("[%s][%s] Exit", txParams.ChannelID, shorttxid(txParams.TxID))
	// we do expect the payload to be a ChaincodeInvocationSpec
	// if we are supporting other payloads in future, this be glaringly point
	// as something that should change
	// 获取链码调用的细节
	cis, err := putils.GetChaincodeInvocationSpec(txParams.Proposal)
	if err != nil {
		return nil, nil, nil, nil, err
	}

	var cdLedger ccprovider.ChaincodeDefinition
	var version string

	if !e.s.IsSysCC(cid.Name) { // 不是系统链码
		// 获取链码的标准数据结构
		cdLedger, err = e.s.GetChaincodeDefinition(cid.Name, txParams.TXSimulator)
		if err != nil {
			return nil, nil, nil, nil, errors.WithMessage(err, fmt.Sprintf("make sure the chaincode %s has been successfully instantiated and try again", cid.Name))
		}
		// 获取用户链码版本
		version = cdLedger.CCVersion()
		// 检查实例化策略以及获取版本
		err = e.s.CheckInstantiationPolicy(cid.Name, version, cdLedger)
		if err != nil {
			return nil, nil, nil, nil, err
		}
	} else {
		// 如果调用的是系统链码，仅仅获取系统链码的版本
		version = util.GetSysCCVersion()
	}

	// ---3. execute the proposal and get simulation results
	var simResult *ledger.TxSimulationResults	// 定义一个Tx模拟结果集
	var pubSimResBytes []byte					// 一个byte数组，保存public的模拟响应结果
	var res *pb.Response						// 响应信息
	var ccevent *pb.ChaincodeEvent				// 链码事件
	// 执行链码进行模拟
	res, ccevent, err = e.callChaincode(txParams, version, cis.ChaincodeSpec.Input, cid)
	if err != nil {
		endorserLogger.Errorf("[%s][%s] failed to invoke chaincode %s, error: %+v", txParams.ChannelID, shorttxid(txParams.TxID), cid, err)
		return nil, nil, nil, nil, err
	}

	if txParams.TXSimulator != nil {
		// GetTxSimulationResults()获取Tx模拟结果集
		if simResult, err = txParams.TXSimulator.GetTxSimulationResults(); err != nil {
			txParams.TXSimulator.Done()
			return nil, nil, nil, nil, err
		}
		// 之前提到Tx模拟结果集中不仅仅只有公共读写集，还有私有的读写集,接下来判断私有的读写集是否为空
		if simResult.PvtSimulationResults != nil {
			if cid.Name == "lscc" {
				// TODO: remove once we can store collection configuration outside of LSCC
				txParams.TXSimulator.Done()
				return nil, nil, nil, nil, errors.New("Private data is forbidden to be used in instantiate")
			}
			pvtDataWithConfig, err := e.AssemblePvtRWSet(simResult.PvtSimulationResults, txParams.TXSimulator)
			// To read collection config need to read collection updates before
			// releasing the lock, hence txParams.TXSimulator.Done()  moved down here
			txParams.TXSimulator.Done()

			if err != nil {
				return nil, nil, nil, nil, errors.WithMessage(err, "failed to obtain collections config")
			}
			endorsedAt, err := e.s.GetLedgerHeight(txParams.ChannelID)
			if err != nil {
				return nil, nil, nil, nil, errors.WithMessage(err, fmt.Sprint("failed to obtain ledger height for channel", txParams.ChannelID))
			}
			// Add ledger height at which transaction was endorsed,
			// `endorsedAt` is obtained from the block storage and at times this could be 'endorsement Height + 1'.
			// However, since we use this height only to select the configuration (3rd parameter in distributePrivateData) and
			// manage transient store purge for orphaned private writesets (4th parameter in distributePrivateData), this works for now.
			// Ideally, ledger should add support in the simulator as a first class function `GetHeight()`.
			pvtDataWithConfig.EndorsedAt = endorsedAt
			if err := e.distributePrivateData(txParams.ChannelID, txParams.TxID, pvtDataWithConfig, endorsedAt); err != nil {
				return nil, nil, nil, nil, err
			}
		}

		txParams.TXSimulator.Done()
		if pubSimResBytes, err = simResult.GetPubSimulationBytes(); err != nil {
			return nil, nil, nil, nil, err
		}
	}
	return cdLedger, res, pubSimResBytes, ccevent, nil
}
```

#### 3.2.1 检查实例化策略
`core/common/ccprovider/ccprovider.go/CheckInstantiationPolicy()`会调用`GetChaincodeData()`尝试从缓存或者本地文件系统获取已安装的链码包`CCPackage`，再解析成`ChaincodeData`对象`ccdata`。再与账本中保存的对应链码的实例化策略进行比较。  
```go
func CheckInstantiationPolicy(name, version string, cdLedger *ChaincodeData) error {
	ccdata, err := GetChaincodeData(name, version)
	if err != nil {
		return err
	}

	// we have the info from the fs, check that the policy
	// matches the one on the file system if one was specified;
	// this check is required because the admin of this peer
	// might have specified instantiation policies for their
	// chaincode, for example to make sure that the chaincode
	// is only instantiated on certain channels; a malicious
	// peer on the other hand might have created a deploy
	// transaction that attempts to bypass the instantiation
	// policy. This check is there to ensure that this will not
	// happen, i.e. that the peer will refuse to invoke the
	// chaincode under these conditions. More info on
	// https://jira.hyperledger.org/browse/FAB-3156
	if ccdata.InstantiationPolicy != nil {
		if !bytes.Equal(ccdata.InstantiationPolicy, cdLedger.InstantiationPolicy) {
			return fmt.Errorf("Instantiation policy mismatch for cc %s/%s", name, version)
		}
	}

	return nil
}
```

#### 3.2.2 调用链码
在`SimulateProposal()`方法中，会调用`callChaincode()`方法调用链码。    
```go
// call specified chaincode (system or user)
func (e *Endorser) callChaincode(txParams *ccprovider.TransactionParams, version string, input *pb.ChaincodeInput, cid *pb.ChaincodeID) (*pb.Response, *pb.ChaincodeEvent, error) {
	endorserLogger.Infof("[%s][%s] Entry chaincode: %s", txParams.ChannelID, shorttxid(txParams.TxID), cid)
	defer func(start time.Time) {
		logger := endorserLogger.WithOptions(zap.AddCallerSkip(1))
		elapsedMilliseconds := time.Since(start).Round(time.Millisecond) / time.Millisecond
		logger.Infof("[%s][%s] Exit chaincode: %s (%dms)", txParams.ChannelID, shorttxid(txParams.TxID), cid, elapsedMilliseconds)
	}(time.Now())

	var err error
	var res *pb.Response
	var ccevent *pb.ChaincodeEvent

	// is this a system chaincode
	// 执行链码，如果是用户链码具体怎么执行的要看用户写的链码逻辑，执行完毕后返回响应信息与链码事件
	res, ccevent, err = e.s.Execute(txParams, txParams.ChannelID, cid.Name, version, txParams.TxID, txParams.SignedProp, txParams.Proposal, input)
	if err != nil {
		return nil, nil, err
	}

	// per doc anything < 400 can be sent as TX.
	// fabric errors will always be >= 400 (ie, unambiguous errors )
	// "lscc" will respond with status 200 or 500 (ie, unambiguous OK or ERROR)
	// 状态常量一共有三个：OK = 200 ERRORTHRESHOLD = 400 ERROR = 500 大于等于400就是错误信息或者被背书节点拒绝。
	if res.Status >= shim.ERRORTHRESHOLD {
		return res, nil, nil
	}

	// ----- BEGIN -  SECTION THAT MAY NEED TO BE DONE IN LSCC ------
	// if this a call to deploy a chaincode, We need a mechanism
	// to pass TxSimulator into LSCC. Till that is worked out this
	// special code does the actual deploy, upgrade here so as to collect
	// all state under one TxSimulator
	//
	// NOTE that if there's an error all simulation, including the chaincode
	// table changes in lscc will be thrown away
    // 判断调用的链码是否为lscc,如果是lscc判断传入的参数是否大于等于3，并且调用的方法是否为deploy或者upgrade，如果是用户链码到这是方法就结束了。
    // 用户链码的实例化(deploy)和升级(upgrade)就会进来这里
	if cid.Name == "lscc" && len(input.Args) >= 3 && (string(input.Args[0]) == "deploy" || string(input.Args[0]) == "upgrade") {
		// 获取链码部署的基本结构,deploy与upgrade都需要对链码进行部署
		userCDS, err := putils.GetChaincodeDeploymentSpec(input.Args[2], e.PlatformRegistry)
		if err != nil {
			return nil, nil, err
		}

		var cds *pb.ChaincodeDeploymentSpec
		cds, err = e.SanitizeUserCDS(userCDS)
		if err != nil {
			return nil, nil, err
		}

		// this should not be a system chaincode
		if e.s.IsSysCC(cds.ChaincodeSpec.ChaincodeId.Name) {
			return nil, nil, errors.Errorf("attempting to deploy a system chaincode %s/%s", cds.ChaincodeSpec.ChaincodeId.Name, txParams.ChannelID)
		}
		// 执行链码的Init,具体如何执行的这里就不再看了,不然内容更多了
		_, _, err = e.s.ExecuteLegacyInit(txParams, txParams.ChannelID, cds.ChaincodeSpec.ChaincodeId.Name, cds.ChaincodeSpec.ChaincodeId.Version, txParams.TxID, txParams.SignedProp, txParams.Proposal, cds)
		if err != nil {
			// increment the failure to indicate instantion/upgrade failures
			meterLabels := []string{
				"channel", txParams.ChannelID,
				"chaincode", cds.ChaincodeSpec.ChaincodeId.Name + ":" + cds.ChaincodeSpec.ChaincodeId.Version,
			}
			e.Metrics.InitFailed.With(meterLabels...).Add(1)
			return nil, nil, err
		}
	}
	// ----- END -------

	return res, ccevent, err
}
```
执行`Execute()`方法调用链码，然后在针对`deploy`和`upgrade`操作进行处理。    
首先看看`Execute()`位于`core/chaincode/chaincode_support.go`    
```go
// Execute invokes chaincode and returns the original response.
func (cs *ChaincodeSupport) Execute(txParams *ccprovider.TransactionParams, cccid *ccprovider.CCContext, input *pb.ChaincodeInput) (*pb.Response, *pb.ChaincodeEvent, error) {
	// 主要是启动链码容器，调用链码
	resp, err := cs.Invoke(txParams, cccid, input)
	// 对链码执行结果进行处理
	return processChaincodeExecutionResult(txParams.TxID, cccid.Name, resp, err)
}
```
继续看看`Invoke`主要调用了`Launch`启动链码容器，和`execute`给链码容器grpc消息(`ChaincodeMessage_TRANSACTION`)进行通信，有兴趣的童鞋们可以跟踪下去   
```go
// Invoke will invoke chaincode and return the message containing the response.
// The chaincode will be launched if it is not already running.
func (cs *ChaincodeSupport) Invoke(txParams *ccprovider.TransactionParams, cccid *ccprovider.CCContext, input *pb.ChaincodeInput) (*pb.ChaincodeMessage, error) {
	// 启动链码容器
	h, err := cs.Launch(txParams.ChannelID, cccid.Name, cccid.Version, txParams.TXSimulator)
	if err != nil {
		return nil, err
	}

	// TODO add Init exactly once semantics here once new lifecycle
	// is available.  Enforced if the target channel is using the new lifecycle
	//
	// First, the function name of the chaincode to invoke should be checked.  If it is
	// "init", then consider this invocation to be of type pb.ChaincodeMessage_INIT,
	// otherwise consider it to be of type pb.ChaincodeMessage_TRANSACTION,
	//
	// Secondly, A check should be made whether the chaincode has been
	// inited, then, if true, only allow cctyp pb.ChaincodeMessage_TRANSACTION,
	// otherwise, only allow cctype pb.ChaincodeMessage_INIT,
	cctype := pb.ChaincodeMessage_TRANSACTION
	// 给链码发送消息
	return cs.execute(cctype, txParams, cccid, input, h)
}
```


#### 3.2.3 处理模拟执行结果
执行完链码后结果不会马上写到数据库，而是以读写集的形式返回给客户端，结果写入交易模拟器`TXSimulator`中。通过调用`GetTxSimulationResults()`方法可以获取模拟执行结果。`TxSimulationResults`包含公有数据读写集`PubSimulationResults`以及私有数据读写集`PvtSimulationResults`。      
`SimulateProposal()`方法会调用`GetTxSimulationResults()`方法获取模拟执行结果。那先看看此函数，位于`core/ledger/kvledger/txmgmt/rwsetutil/rwset_builder.go`      
```go
// GetTxSimulationResults returns the proto bytes of public rwset
// (public data + hashes of private data) and the private rwset for the transaction
func (b *RWSetBuilder) GetTxSimulationResults() (*ledger.TxSimulationResults, error) {
	// 获取交易模拟执行结果的交易私密数据读写集
	pvtData := b.getTxPvtReadWriteSet()
	var err error

	var pubDataProto *rwset.TxReadWriteSet
	var pvtDataProto *rwset.TxPvtReadWriteSet

	// Populate the collection-level hashes into pub rwset and compute the proto bytes for pvt rwset
	// 计算私密数据hash
	if pvtData != nil {
		if pvtDataProto, err = pvtData.toProtoMsg(); err != nil {
			return nil, err
		}
		// 遍历计算私密数据hash值
		for _, ns := range pvtDataProto.NsPvtRwset {
			for _, coll := range ns.CollectionPvtRwset {
				b.setPvtCollectionHash(ns.Namespace, coll.CollectionName, coll.Rwset)
			}
		}
	}
	// Compute the proto bytes for pub rwset
	// 获取交易模拟执行结果的公有数据读写集
	pubSet := b.GetTxReadWriteSet()
	if pubSet != nil {
		if pubDataProto, err = b.GetTxReadWriteSet().toProtoMsg(); err != nil {
			return nil, err
		}
	}
	// 构造交易模拟执行结果
	return &ledger.TxSimulationResults{
		PubSimulationResults: pubDataProto,
		PvtSimulationResults: pvtDataProto,
	}, nil
}
```


### 3.3 签名背书
在 `ProcessProposal()`方法中，首先会判断通道`id`是否为`nil`，如果为`nil`，则直接返回响应结果（例如`install`操作）。如果不为`nil`，会调用`endorseProposal()`方法对模拟执行结果进行签名和背书。在`endorseProposal()`方法中，会构造`Context`对象，再调用`EndorseWithPlugin()`里面会调用`getOrCreatePlugin()`创建plugin，然后调用`proposalResponsePayloadFromContext()`方法，在该方法中会计算背书结果hash以及封装模拟执行结果、链码`event`事件以及链码响应结果等（数据结构为`ProposalResponsePayload`），在序列化成`[]byte`数组，最后调用`Endorse()`方法执行签名背书操作（由于`escc`现在是插件形式执行，里面会进行判断。默认执行`escc`）        

```go
// endorse the proposal by calling the ESCC
func (e *Endorser) endorseProposal(_ context.Context, chainID string, txid string, signedProp *pb.SignedProposal, proposal *pb.Proposal, response *pb.Response, simRes []byte, event *pb.ChaincodeEvent, visibility []byte, ccid *pb.ChaincodeID, txsim ledger.TxSimulator, cd ccprovider.ChaincodeDefinition) (*pb.ProposalResponse, error) {
	endorserLogger.Debugf("[%s][%s] Entry chaincode: %s", chainID, shorttxid(txid), ccid)
	defer endorserLogger.Debugf("[%s][%s] Exit", chainID, shorttxid(txid))

	isSysCC := cd == nil
	// 1) extract the name of the escc that is requested to endorse this chaincode
	var escc string
	// ie, "lscc" or system chaincodes
	// 判断是否是系统链码
	if isSysCC { // 如果是系统链码，则使用escc进行背书
		escc = "escc"
	} else {
		escc = cd.Endorsement()
	}

	endorserLogger.Debugf("[%s][%s] escc for chaincode %s is %s", chainID, shorttxid(txid), ccid, escc)

	// marshalling event bytes
	var err error
	var eventBytes []byte
	if event != nil { // 如果链码事件不为空
		// 获取链码事件
		eventBytes, err = putils.GetBytesChaincodeEvent(event)
		if err != nil {
			return nil, errors.Wrap(err, "failed to marshal event bytes")
		}
	}

	// set version of executing chaincode
	if isSysCC {
		// if we want to allow mixed fabric levels we should
		// set syscc version to ""
		// 获取系统链码版本
		ccid.Version = util.GetSysCCVersion()
	} else {
		// 获取用户链码版本
		ccid.Version = cd.CCVersion()
	}

	ctx := Context{
		PluginName:     escc,
		Channel:        chainID,
		SignedProposal: signedProp,
		ChaincodeID:    ccid,
		Event:          eventBytes,
		SimRes:         simRes,
		Response:       response,
		Visibility:     visibility,
		Proposal:       proposal,
		TxID:           txid,
	}
	// 背书
	return e.s.EndorseWithPlugin(ctx)
}
```
接着看`EndorseWithPlugin`,位于`core/endorser/plugin_endorser.go`    
```go
// EndorseWithPlugin endorses the response with a plugin
func (pe *PluginEndorser) EndorseWithPlugin(ctx Context) (*pb.ProposalResponse, error) {
	endorserLogger.Debug("Entering endorsement for", ctx)

	if ctx.Response == nil {
		return nil, errors.New("response is nil")
	}

	if ctx.Response.Status >= shim.ERRORTHRESHOLD {
		return &pb.ProposalResponse{Response: ctx.Response}, nil
	}
	// 获取或者创建插件
	plugin, err := pe.getOrCreatePlugin(PluginName(ctx.PluginName), ctx.Channel)
	if err != nil {
		endorserLogger.Warning("Endorsement with plugin for", ctx, " failed:", err)
		return nil, errors.Errorf("plugin with name %s could not be used: %v", ctx.PluginName, err)
	}
	// 从上下文中获取提案byte数据
	prpBytes, err := proposalResponsePayloadFromContext(ctx)
	if err != nil {
		endorserLogger.Warning("Endorsement with plugin for", ctx, " failed:", err)
		return nil, errors.Wrap(err, "failed assembling proposal response payload")
	}
	// 进行背书操作
	endorsement, prpBytes, err := plugin.Endorse(prpBytes, ctx.SignedProposal)
	if err != nil {
		endorserLogger.Warning("Endorsement with plugin for", ctx, " failed:", err)
		return nil, errors.WithStack(err)
	}
	// 背书完成后，封装为提案响应结构体，最后将该结构体返回
	resp := &pb.ProposalResponse{
		Version:     1,
		Endorsement: endorsement,
		Payload:     prpBytes,
		Response:    ctx.Response,
	}
	endorserLogger.Debug("Exiting", ctx)
	return resp, nil
}
```
背书操作主要是在`Endorse`进行，位于`core/handlers/endorsement/plugin/plugin.go`
```go
// Endorse signs the given payload(ProposalResponsePayload bytes), and optionally mutates it.
// Returns:
// The Endorsement: A signature over the payload, and an identity that is used to verify the signature
// The payload that was given as input (could be modified within this function)
// Or error on failure
func (e *DefaultEndorsement) Endorse(prpBytes []byte, sp *peer.SignedProposal) (*peer.Endorsement, []byte, error) {
	signer, err := e.SigningIdentityForRequest(sp)
	if err != nil {
		return nil, nil, errors.New(fmt.Sprintf("failed fetching signing identity: %v", err))
	}
	// serialize the signing identity
	identityBytes, err := signer.Serialize()
	if err != nil {
		return nil, nil, errors.New(fmt.Sprintf("could not serialize the signing identity: %v", err))
	}

	// sign the concatenation of the proposal response and the serialized endorser identity with this endorser's key
	signature, err := signer.Sign(append(prpBytes, identityBytes...))
	if err != nil {
		return nil, nil, errors.New(fmt.Sprintf("could not sign the proposal response payload: %v", err))
	}
	endorsement := &peer.Endorsement{Signature: signature, Endorser: identityBytes}
	return endorsement, prpBytes, nil
}
```







参考:   
[Fabric1.4源码解析：Peer节点背书提案过程](https://www.cnblogs.com/cbkj-xd/p/11077490.html)  
[Fabric 1.4 源码分析 Endorser背书节点](https://www.cnblogs.com/i-dandan/p/12163377.html)