
`Environment:`  
`solc:0.5.11+commit.22be8592.Darwin.appleclang`

### 一、概述



### 二、EVM指令集  
* 算术操作
* 栈操作
* 处理流程操作
* 系统操作
* 逻辑操作
* 环境操作
* 区块操作


### 三、合约字节码结构  
智能合约编译后的字节码分为三部分:部署代码、runtime代码、auxdata。  
* 部署代码  
    以太坊虚拟机在创建合约的时候，会先创建合约账户，然后运行部署代码。运行完成后它会将runtime代码+auxdata 存储到区块链上。之后再把二者的存储地址跟合约账户关联起来(也就是把合约账户中的code hash字段用该地址赋值)，这样就完成了合约的部署。

* runtime代码
    就是合约创建成功后当它被调用运行的代码  

* auxdata
    紧跟着runtime代码后面被存储起来  
    将代码哈希地址作为swarm网络的一个地址，把代码的存入道swarm中；swarm类似于ipfs，便于以太坊的分布式存储。格式如，`{"bzzr1": <Swarm hash>, "solc": <compiler version>}`,所以auxdata是合约代码的校验码和solc版本的数据，并且运行合约时是没使用到。
    ```
    0xa2
    0x65 'b' 'z' 'z' 'r' '1' 0x58 0x20 <32 bytes swarm hash>
    0x64 's' 'o' 'l' 'c' 0x43 <3 byte version encoding>
    0x00 0x32
    ```
    `参考:https://solidity.readthedocs.io/en/latest/metadata.html#encoding-of-the-metadata-hash-in-the-bytecode`

HelloWorld.sol的代码为:
```
pragma solidity >=0.4.21 <0.6.0;

contract HelloWorld {
    address contractOwner;
    constructor() public {
    contractOwner = msg.sender;
  }
}
```
可以使用solc编译工具把合约代码编译成汇编代码和字节码  
执行: `solc --bin --asm HelloWorld.sol`得出以下结果:

```
======= HelloWorld.sol:HelloWorld =======
EVM assembly:
    /* "HelloWorld.sol":34:147  contract HelloWorld {... */
  mstore(0x40, 0x80)
    /* "HelloWorld.sol":87:145  constructor() public {... */
  callvalue
    /* "--CODEGEN--":8:17   */
  dup1
    /* "--CODEGEN--":5:7   */
  iszero
  tag_1
  jumpi
    /* "--CODEGEN--":30:31   */
  0x00
    /* "--CODEGEN--":27:28   */
  dup1
    /* "--CODEGEN--":20:32   */
  revert
    /* "--CODEGEN--":5:7   */
tag_1:
    /* "HelloWorld.sol":87:145  constructor() public {... */
  pop
    /* "HelloWorld.sol":130:140  msg.sender */
  caller
    /* "HelloWorld.sol":114:127  contractOwner */
  0x00
  dup1
    /* "HelloWorld.sol":114:140  contractOwner = msg.sender */
  0x0100
  exp
  dup2
  sload
  dup2
  0xffffffffffffffffffffffffffffffffffffffff
  mul
  not
  and
  swap1
  dup4
  0xffffffffffffffffffffffffffffffffffffffff
  and
  mul
  or
  swap1
  sstore
  pop
    /* "HelloWorld.sol":34:147  contract HelloWorld {... */
  dataSize(sub_0)
  dup1
  dataOffset(sub_0)
  0x00
  codecopy
  0x00
  return
stop

sub_0: assembly {
        /* "HelloWorld.sol":34:147  contract HelloWorld {... */
      mstore(0x40, 0x80)
      0x00
      dup1
      revert

    auxdata: 0xa265627a7a72315820a9fa9766256f0d8b91675866edc9d3a66b290f0e30e6563b79b1d73102128c4464736f6c634300050b0032
}

Binary:
6080604052348015600f57600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff160217905550603e80605d6000396000f3fe6080604052600080fdfea265627a7a72315820a9fa9766256f0d8b91675866edc9d3a66b290f0e30e6563b79b1d73102128c4464736f6c634300050b0032
```
智能合约编译生成的汇编指令也分为三部分：EVM assembly标签下的汇编指令对应的是部署代码；sub_0标签下的汇编指令对应的是runtime代码；sub_0标签下的auxdata和字节码中的auxdata完全相同。  
Binary为最后编译出来的字节码(部署代码+runtime代码+auxdata)。


### 四、合约创建流程


### 五、合约的运行流程


### 六、
```

pragma solidity ^0.4.0;

contract C {
    uint[] x; // x 的数据存储位置是 storage

    // memoryArray 的数据存储位置是 memory
    function f(uint[] memoryArray) public {
        x = memoryArray; // 将整个数组拷贝到 storage 中，可行
        var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
        y[7]; // 返回第 8 个元素，可行
        y.length = 2; // 通过 y 修改 x，可行
        delete x; // 清除数组，同时修改 y，可行
        // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
        // 但 storage 是“静态”分配的：
        // y = memoryArray;
        // 下面这一行也不可行，因为这会“重置”指针，
        // 但并没有可以让它指向的合适的存储位置。
        // delete y;
        g(x); // 调用 g 函数，同时移交对 x 的引用
        h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
```


### 三、GAS的计算


### 四、



参考：

[以太坊 EVM原理与实现](https://www.cnblogs.com/helloworld2018/p/8998926.html)

[以太坊虚拟机（EVM）底层原理及性能缺陷](https://abelsu7.top/2018/02/28/ethereum-virtual-machine/)

[《也来谈一谈以太坊虚拟机EVM的缺陷与不足》](https://bitkan.com/en/news/topic/35732)

[深入了解以太坊虚拟机](https://www.jianshu.com/p/1969f3761208)


[深入探索EVM : 编译和部署智能合约](https://www.arcblock.io/zh/post/2018/12/08/evm-part-1)

[从solc编译过程来理解solidity合约结构](https://www.anquanke.com/post/id/164567)

[Solidity中文文档](https://solidity-cn.readthedocs.io/zh/develop/)

[c/c++编译原理](https://zhuanlan.zhihu.com/p/44199755)

[EVM指令集](https://gist.github.com/hayeah/bd37a123c02fecffbe629bf98a8391df)

[以太坊智能合约虚拟机(EVM)原理与实现](https://blog.csdn.net/mongo_node/article/details/80216079)

[以太坊虚拟机(EVM)架构和源码简析](https://blog.csdn.net/SunnyWed/article/details/80696948)

[以太坊智能合约静态分析](https://paper.seebug.org/790/)  

[深入了解以太坊虚拟机第5部分——一个新合约被创建后会发生什么
](https://lilymoana.github.io/evm_part5.html)

[以太坊虚拟机介绍](https://blog.csdn.net/zxhoo/article/details/81865629)

[Ethereum 以太坊智能合约部署源码分析](https://www.jianshu.com/p/e5b56baad043)

[以太坊交易源码分析](https://blog.csdn.net/TurkeyCock/article/details/80485391)

