




```
-> geth
    -> startNode
        -> Start
            -> openEndpoints
                -> startRPC
                    -> n.http.start
                        -> net.Listen("tcp", h.endpoint)
                        -> go h.server.Serve(listener)
                            -> httpServer.ServeHTTP
```






参考:  
[以太坊源码解析：rpc](https://yangzhe.me/2019/07/05/ethereum-rpc/)