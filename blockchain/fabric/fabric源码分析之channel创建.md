
`environment:`      
`fabric v2.2.0`


# Orderer处理函数
`orderer/common/broadcast/broadcast.go`

```
// Handle reads requests from a Broadcast stream, processes them, and returns the responses to the stream
func (bh *Handler) Handle(srv ab.AtomicBroadcast_BroadcastServer) error {
	addr := util.ExtractRemoteAddress(srv.Context())
	logger.Debugf("Starting new broadcast loop for %s", addr)
	for {
		// 接收grpc消息
		msg, err := srv.Recv()
		if err == io.EOF {
			logger.Debugf("Received EOF from %s, hangup", addr)
			return nil
		}
		if err != nil {
			logger.Warningf("Error reading from %s: %s", addr, err)
			return err
		}
		// 处理消息
		resp := bh.ProcessMessage(msg, addr)
		// 返回处理结果
		err = srv.Send(resp)
		if resp.Status != cb.Status_SUCCESS {
			return err
		}

		if err != nil {
			logger.Warningf("Error sending to %s: %s", addr, err)
			return err
		}
	}

}
```

`orderer/common/broadcast/broadcast.go`
```
// ProcessMessage validates and enqueues a single message
func (bh *Handler) ProcessMessage(msg *cb.Envelope, addr string) (resp *ab.BroadcastResponse) {
	// 这个结构体应该理解为记录器，记录消息的相关信息
	tracker := &MetricsTracker{
		ChannelID: "unknown",
		TxType:    "unknown",
		Metrics:   bh.Metrics,
	}
	defer func() {
		// This looks a little unnecessary, but if done directly as
		// a defer, resp gets the (always nil) current state of resp
		// and not the return value
		tracker.Record(resp)
	}()
	// 记录处理消息的开始时间
	tracker.BeginValidate()
	// chdr: channelHeader,isConfig:是否为配置交易消息
	// 获取接收到的消息的Header、判断是否为配置信息、获取消息处理器
	chdr, isConfig, processor, err := bh.SupportRegistrar.BroadcastChannelSupport(msg)
	if chdr != nil {
		tracker.ChannelID = chdr.ChannelId
		tracker.TxType = cb.HeaderType(chdr.Type).String()
	}
	if err != nil {
		logger.Warningf("[channel: %s] Could not get message processor for serving %s: %s", tracker.ChannelID, addr, err)
		return &ab.BroadcastResponse{Status: cb.Status_BAD_REQUEST, Info: err.Error()}
	}

	if !isConfig {
		...
	} else { // isConfig
		logger.Debugf("[channel: %s] Broadcast is processing config update message from %s", chdr.ChannelId, addr)
		// 对于配置消息，使用处理器处理配置变更消息
		config, configSeq, err := processor.ProcessConfigUpdateMsg(msg)
		if err != nil {
			logger.Warningf("[channel: %s] Rejecting broadcast of config message from %s because of error: %s", chdr.ChannelId, addr, err)
			return &ab.BroadcastResponse{Status: ClassifyError(err), Info: err.Error()}
		}
		// 记录消息处理完毕时间
		tracker.EndValidate()
		// 开始进行入队操作
		tracker.BeginEnqueue()
		// waitReady()是一个阻塞方法，等待入队完成或出现异常
		if err = processor.WaitReady(); err != nil {
			logger.Warningf("[channel: %s] Rejecting broadcast of message from %s with SERVICE_UNAVAILABLE: rejected by Consenter: %s", chdr.ChannelId, addr, err)
			return &ab.BroadcastResponse{Status: cb.Status_SERVICE_UNAVAILABLE, Info: err.Error()}
		}
		// 把配置变更的交易进行全网共识,具体看看是哪种共识(raft、solo、kafka)
		err = processor.Configure(config, configSeq)
		if err != nil {
			logger.Warningf("[channel: %s] Rejecting broadcast of config message from %s with SERVICE_UNAVAILABLE: rejected by Configure: %s", chdr.ChannelId, addr, err)
			return &ab.BroadcastResponse{Status: cb.Status_SERVICE_UNAVAILABLE, Info: err.Error()}
		}
	}

	logger.Debugf("[channel: %s] Broadcast has successfully enqueued message of type %s from %s", chdr.ChannelId, cb.HeaderType_name[chdr.Type], addr)
	// 返回操作成功的响应
	return &ab.BroadcastResponse{Status: cb.Status_SUCCESS}
}
```

`orderer/common/broadcast/broadcast.go`

```
// ProcessConfigUpdateMsg handles messages of type CONFIG_UPDATE either for the system channel itself
// or, for channel creation.  In the channel creation case, the CONFIG_UPDATE is wrapped into a resulting
// ORDERER_TRANSACTION, and in the standard CONFIG_UPDATE case, a resulting CONFIG message
func (s *SystemChannel) ProcessConfigUpdateMsg(envConfigUpdate *cb.Envelope) (config *cb.Envelope, configSeq uint64, err error) {
	// 获取需要配置的channel id
	channelID, err := protoutil.ChannelID(envConfigUpdate)
	if err != nil {
		return nil, 0, err
	}

	logger.Debugf("Processing config update tx with system channel message processor for channel ID %s", channelID)
	// 判断获取到的通道ID是否为已经存在的用户通道ID，如果是的话转到StandardChannel中的ProcessConfigUpdateMsg()方法进行处理
	if channelID == s.support.ChannelID() {
		return s.StandardChannel.ProcessConfigUpdateMsg(envConfigUpdate)
	}

	// XXX we should check that the signature on the outer envelope is at least valid for some MSP in the system channel

	logger.Debugf("Processing channel create tx for channel %s on system channel %s", channelID, s.support.ChannelID())

	// If the channel ID does not match the system channel, then this must be a channel creation transaction
	// 返回Bundle结构体
	bundle, err := s.templator.NewChannelConfig(envConfigUpdate)
	if err != nil {
		return nil, 0, err
	}
	// 创建一个配置验证器对该方法的传入参数进行验证操作,返回实例化结构体ConfigEnvelope
	newChannelConfigEnv, err := bundle.ConfigtxValidator().ProposeConfigUpdate(envConfigUpdate)
	if err != nil {
		return nil, 0, errors.WithMessagef(err, "error validating channel creation transaction for new channel '%s', could not successfully apply update to template configuration", channelID)
	}
	// 创建一个签名的Envelope,此次为Header类型为HeaderType_CONFIG进行签名
	newChannelEnvConfig, err := protoutil.CreateSignedEnvelope(cb.HeaderType_CONFIG, channelID, s.support.Signer(), newChannelConfigEnv, msgVersion, epoch)
	if err != nil {
		return nil, 0, err
	}
	// 创建一个签名的Transaction,此次为Header类型为HeaderType_ORDERER_TRANSACTION进行签名
	wrappedOrdererTransaction, err := protoutil.CreateSignedEnvelope(cb.HeaderType_ORDERER_TRANSACTION, s.support.ChannelID(), s.support.Signer(), newChannelEnvConfig, msgVersion, epoch)
	if err != nil {
		return nil, 0, err
	}

	// We re-apply the filters here, especially for the size filter, to ensure that the transaction we
	// just constructed is not too large for our consenter.  It additionally reapplies the signature
	// check, which although not strictly necessary, is a good sanity check, in case the orderer
	// has not been configured with the right cert material.  The additional overhead of the signature
	// check is negligible, as this is the channel creation path and not the normal path.
	// 过滤器进行过滤，主要检查是否创建的Transaction过大，以及签名检查，确保Order节点使用正确的证书进行签名
	err = s.StandardChannel.filters.Apply(wrappedOrdererTransaction)
	if err != nil {
		return nil, 0, err
	}
	// 将Transaction返回
	return wrappedOrdererTransaction, s.support.Sequence(), nil
}
```



`orderer/consensus/solo/consensus.go`
```
// Configure accepts configuration update messages for ordering
func (ch *chain) Configure(config *cb.Envelope, configSeq uint64) error {
	select {
	case ch.sendChan <- &message{
		configSeq: configSeq,
		configMsg: config,
	}:
		return nil
	case <-ch.exitChan:
		return fmt.Errorf("Exiting")
	}
}

...

func (ch *chain) main() {
	var timer <-chan time.Time
	var err error

	for {
		seq := ch.support.Sequence()
		err = nil
		select {
		case msg := <-ch.sendChan:
			if msg.configMsg == nil {
				// NormalMsg
				...

			} else {
				// ConfigMsg
				// 消息中的配置序号小于通道最新配置序号，那么该消息就有可能会存在失效的问题
				if msg.configSeq < seq {
					msg.configMsg, _, err = ch.support.ProcessConfigMsg(msg.configMsg)
					if err != nil {
						logger.Warningf("Discarding bad config message: %s", err)
						continue
					}
				}
				batch := ch.support.BlockCutter().Cut()
				if batch != nil {
					block := ch.support.CreateNextBlock(batch)
					ch.support.WriteBlock(block, nil)
				}
				// 创建一个新的区块
				block := ch.support.CreateNextBlock([]*cb.Envelope{msg.configMsg})
				// 写区块
				ch.support.WriteConfigBlock(block, nil)
				timer = nil
			}
		case <-timer:
			...
		case <-ch.exitChan:
			...
		}
	}
}

```






参考:   
[Hyperledger Fabric原理详解与实战6](https://juejin.cn/post/6844904119379951624)     
[Fabric 1.4源码解读 7：Orderer架构解读](https://blog.csdn.net/m0_43499523/article/details/104882066)
