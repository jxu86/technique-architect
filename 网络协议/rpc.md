

* 概述      
RPC(Remote Procedure Call，远程过程调用)是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络细节的应用程序通信协议。RPC协议构建于TCP或UDP,或者是HTTP上。允许开发者直接调用另一台服务器上的程序，而开发者无需另外的为这个调用过程编写网络通信相关代码，使得开发网络分布式程序在内的应用程序更加容易
RPC采用客户端-服务器端的工作模式，请求程序就是一个客户端，而服务提供程序就是一个服务器端。当执行一个远程过程调用时，客户端程序首先先发送一个带有参数的调用信息到服务端，然后等待服务端响应。在服务端，服务进程保持睡眠状态直到客户端的调用信息到达。当一个调用信息到达时，服务端获得进程参数，计算出结果，并向客户端发送应答信息。然后等待下一个调用。

* demo  
https://github.com/jxu86/technique-code/tree/master/rpc/go

参考:   
[以太坊RPC机制与API实例](https://www.cnblogs.com/Evsward/p/eth-rpc.html)    
[Golang中的RPC和gRPC](https://segmentfault.com/a/1190000016328212)      
[以太坊源码解析：rpc](https://www.jianshu.com/p/fadd34394ef1)
