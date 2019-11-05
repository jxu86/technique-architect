
`Environment:`  
`solc:0.5.11+commit.22be8592.Darwin.appleclang`

### 一、概述
 &emsp;&emsp;以下介绍的是以太坊智能合约编译和部署过程，没涉及到EVM具体的指令执行过程。我们知道EVM运行的是合约的字节码，那么合约编译完后到底是怎样的？里面包括除了真正的合约代码还有哪些功能，下面会说到。


### 二、智能合约的编译(字节码的生成)  
我们先看一个简单的合约例子:  
HelloWorld.sol内容为:
```
pragma solidity >=0.4.21 <0.6.0;
contract HelloWorld {
    uint a;
    constructor() public {
      a = 0x55;
    }

    function sayHello() public pure returns(string memory) {
      return "helloworld";
    }
}
```
可以使用solc编译工具把合约代码编译成汇编代码和字节码  
执行: `solc --optimize --asm --bin HelloWorld.sol`得出以下结果:
```
======= HelloWorld.sol:HelloWorld =======
EVM assembly:
    /* "HelloWorld.sol":34:214  contract HelloWorld {... */
  mstore(0x40, 0x80)
    /* "HelloWorld.sol":72:116  constructor() public {... */
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
  pop
    /* "HelloWorld.sol":105:109  0x55 */
  0x55
    /* "HelloWorld.sol":101:102  a */
  0x00
    /* "HelloWorld.sol":101:109  a = 0x55 */
  sstore
    /* "HelloWorld.sol":34:214  contract HelloWorld {... */
  dataSize(sub_0)
  dup1
  dataOffset(sub_0)
  0x00
  codecopy
  0x00
  return
stop

sub_0: assembly {
        /* "HelloWorld.sol":34:214  contract HelloWorld {... */
      mstore(0x40, 0x80)
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
        /* "HelloWorld.sol":34:214  contract HelloWorld {... */
      pop
      jumpi(tag_2, lt(calldatasize, 0x04))
      shr(0xe0, calldataload(0x00))
      dup1
      0xef5fb05b
      eq
      tag_3
      jumpi
    tag_2:
      0x00
      dup1
      revert
        /* "HelloWorld.sol":122:211  function sayHello() public pure returns(string memory) {... */
    tag_3:
      tag_4
      tag_5
      jump	// in
    tag_4:
      0x40
      dup1
      mload
      0x20
      dup1
      dup3
      mstore
      dup4
      mload
      dup2
      dup4
      add
      mstore
      dup4
      mload
      swap2
      swap3
      dup4
      swap3
      swap1
      dup4
      add
      swap2
      dup6
      add
      swap1
      dup1
      dup4
      dup4
      0x00
        /* "--CODEGEN--":8:108   */
    tag_6:
        /* "--CODEGEN--":33:36   */
      dup4
        /* "--CODEGEN--":30:31   */
      dup2
        /* "--CODEGEN--":27:37   */
      lt
        /* "--CODEGEN--":8:108   */
      iszero
      tag_8
      jumpi
        /* "--CODEGEN--":90:101   */
      dup2
      dup2
      add
        /* "--CODEGEN--":84:102   */
      mload
        /* "--CODEGEN--":71:82   */
      dup4
      dup3
      add
        /* "--CODEGEN--":64:103   */
      mstore
        /* "--CODEGEN--":52:54   */
      0x20
        /* "--CODEGEN--":45:55   */
      add
        /* "--CODEGEN--":8:108   */
      jump(tag_6)
    tag_8:
        /* "--CODEGEN--":12:26   */
      pop
        /* "HelloWorld.sol":122:211  function sayHello() public pure returns(string memory) {... */
      pop
      pop
      pop
      swap1
      pop
      swap1
      dup2
      add
      swap1
      0x1f
      and
      dup1
      iszero
      tag_9
      jumpi
      dup1
      dup3
      sub
      dup1
      mload
      0x01
      dup4
      0x20
      sub
      0x0100
      exp
      sub
      not
      and
      dup2
      mstore
      0x20
      add
      swap2
      pop
    tag_9:
      pop
      swap3
      pop
      pop
      pop
      mload(0x40)
      dup1
      swap2
      sub
      swap1
      return
    tag_5:
        /* "HelloWorld.sol":185:204  return "helloworld" */
      0x40
      dup1
      mload
      dup1
      dup3
      add
      swap1
      swap2
      mstore
      0x0a
      dup2
      mstore
      shl(0xb2, 0x1a195b1b1bdddbdc9b19)
      0x20
      dup3
      add
      mstore
        /* "HelloWorld.sol":122:211  function sayHello() public pure returns(string memory) {... */
      swap1
      jump	// out

    auxdata: 0xa265627a7a723158204ff415e78e8ff1320ec8d86a414f3ad584e200a186c072b0643ee6c2a166745a64736f6c634300050b0032
}

Binary:
608060405234801561001057600080fd5b50605560005560fe806100246000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c8063ef5fb05b14602d575b600080fd5b603360a5565b6040805160208082528351818301528351919283929083019185019080838360005b83811015606b5781810151838201526020016055565b50505050905090810190601f16801560975780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b60408051808201909152600a8152691a195b1b1bdddbdc9b1960b21b60208201529056fea265627a7a723158204ff415e78e8ff1320ec8d86a414f3ad584e200a186c072b0643ee6c2a166745a64736f6c634300050b0032
```
可以看到编译出来得到两大部分:汇编代码和字节码(Binary),其实这两部分是等价的，汇编代码通过opcode可以对应回字节码，汇编只是方便我们阅读，最后送到EVM上运行的是Binary。  

智能合约编译后的字节码分为三部分:部署代码、runtime代码、auxdata。  
EVM assembly标签下的汇编指令对应的是部署代码；    
sub_0标签下的汇编指令对应的是runtime代码；  
sub_0标签下的auxdata和字节码中的auxdata完全相同。  
Binary为最后编译出来的字节码(部署代码+runtime代码+auxdata)。  
* 部署代码  
    以太坊虚拟机在创建合约的时候，会先创建合约账户，然后运行部署代码。运行完成后它会将runtime代码+auxdata 存储到区块链上。之后再把二者的存储地址跟合约账户关联起来(也就是把合约账户中的code hash字段用该地址赋值)，这样就完成了合约的部署。  
    部署代码有两个主要作用：
    * Payable检查
    * 运行合约构造器函数，并设置初始化内存变量（就像合约的拥有者）
    * 复制代码，并将其返回给内存

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


### 三、智能合约的部署    
EVM在创建合约的时候需要执行合约，合约的执行就是从第一个字节码开始执行。也就是说，部署代码会先给执行。  
而部署代码分别做了三件事(Payable检查、运行合约构造器函数、复制代码，并将其返回给内存)，那接下来我们分析一下部署代码这三部分的代码,我们先用反汇编工具把字节码变的简单易懂点:
```
solc --optimize --bin  HelloWorld.sol -o ./   # 生成HelloWorld.bin
evmasm -d -i HelloWorld.bin -o ./helloworld.asm  # 反汇编生成汇编代码
```
反汇编的工具我用的是[pyevmasm](https://github.com/crytic/pyevmasm)  
helloworld.asm的内容如下:

```
00000000: PUSH1 0x80
00000002: PUSH1 0x40
00000004: MSTORE
00000005: CALLVALUE
00000006: DUP1
00000007: ISZERO
00000008: PUSH2 0x10
0000000b: JUMPI
0000000c: PUSH1 0x0
0000000e: DUP1
0000000f: REVERT
00000010: JUMPDEST
00000011: POP
00000012: PUSH1 0x55
00000014: PUSH1 0x0
00000016: SSTORE
00000017: PUSH1 0xfe
00000019: DUP1
0000001a: PUSH2 0x24
0000001d: PUSH1 0x0
0000001f: CODECOPY
00000020: PUSH1 0x0
00000022: RETURN
00000023: INVALID
00000024: PUSH1 0x80
00000026: PUSH1 0x40
00000028: MSTORE
00000029: CALLVALUE
0000002a: DUP1
0000002b: ISZERO
0000002c: PUSH1 0xf
0000002e: JUMPI
0000002f: PUSH1 0x0
00000031: DUP1
00000032: REVERT
00000033: JUMPDEST
00000034: POP
00000035: PUSH1 0x4
00000037: CALLDATASIZE
00000038: LT
00000039: PUSH1 0x28
0000003b: JUMPI
0000003c: PUSH1 0x0
0000003e: CALLDATALOAD
0000003f: PUSH1 0xe0
00000041: SHR
00000042: DUP1
00000043: PUSH4 0xef5fb05b
00000048: EQ
00000049: PUSH1 0x2d
0000004b: JUMPI
0000004c: JUMPDEST
0000004d: PUSH1 0x0
0000004f: DUP1
00000050: REVERT
00000051: JUMPDEST
00000052: PUSH1 0x33
00000054: PUSH1 0xa5
00000056: JUMP
00000057: JUMPDEST
00000058: PUSH1 0x40
0000005a: DUP1
0000005b: MLOAD
0000005c: PUSH1 0x20
0000005e: DUP1
0000005f: DUP3
00000060: MSTORE
00000061: DUP4
00000062: MLOAD
00000063: DUP2
00000064: DUP4
00000065: ADD
00000066: MSTORE
00000067: DUP4
00000068: MLOAD
00000069: SWAP2
0000006a: SWAP3
0000006b: DUP4
0000006c: SWAP3
0000006d: SWAP1
0000006e: DUP4
0000006f: ADD
00000070: SWAP2
00000071: DUP6
00000072: ADD
00000073: SWAP1
00000074: DUP1
00000075: DUP4
00000076: DUP4
00000077: PUSH1 0x0
00000079: JUMPDEST
0000007a: DUP4
0000007b: DUP2
0000007c: LT
0000007d: ISZERO
0000007e: PUSH1 0x6b
00000080: JUMPI
00000081: DUP2
00000082: DUP2
00000083: ADD
00000084: MLOAD
00000085: DUP4
00000086: DUP3
00000087: ADD
00000088: MSTORE
00000089: PUSH1 0x20
0000008b: ADD
0000008c: PUSH1 0x55
0000008e: JUMP
0000008f: JUMPDEST
00000090: POP
00000091: POP
00000092: POP
00000093: POP
00000094: SWAP1
00000095: POP
00000096: SWAP1
00000097: DUP2
00000098: ADD
00000099: SWAP1
0000009a: PUSH1 0x1f
0000009c: AND
0000009d: DUP1
0000009e: ISZERO
0000009f: PUSH1 0x97
000000a1: JUMPI
000000a2: DUP1
000000a3: DUP3
000000a4: SUB
000000a5: DUP1
000000a6: MLOAD
000000a7: PUSH1 0x1
000000a9: DUP4
000000aa: PUSH1 0x20
000000ac: SUB
000000ad: PUSH2 0x100
000000b0: EXP
000000b1: SUB
000000b2: NOT
000000b3: AND
000000b4: DUP2
000000b5: MSTORE
000000b6: PUSH1 0x20
000000b8: ADD
000000b9: SWAP2
000000ba: POP
000000bb: JUMPDEST
000000bc: POP
000000bd: SWAP3
000000be: POP
000000bf: POP
000000c0: POP
000000c1: PUSH1 0x40
000000c3: MLOAD
000000c4: DUP1
000000c5: SWAP2
000000c6: SUB
000000c7: SWAP1
000000c8: RETURN
000000c9: JUMPDEST
000000ca: PUSH1 0x40
000000cc: DUP1
000000cd: MLOAD
000000ce: DUP1
000000cf: DUP3
000000d0: ADD
000000d1: SWAP1
000000d2: SWAP2
000000d3: MSTORE
000000d4: PUSH1 0xa
000000d6: DUP2
000000d7: MSTORE
000000d8: PUSH10 0x1a195b1b1bdddbdc9b19
000000e3: PUSH1 0xb2
000000e5: SHL
000000e6: PUSH1 0x20
000000e8: DUP3
000000e9: ADD
000000ea: MSTORE
000000eb: SWAP1
000000ec: JUMP
000000ed: INVALID
000000ee: LOG2
000000ef: PUSH6 0x627a7a723158
000000f6: SHA3
000000f7: DUP3
000000f8: PUSH9 0xb09c0270a7b58338b4
00000102: INVALID
00000103: INVALID
00000104: SWAP9
00000105: INVALID
00000106: INVALID
00000107: INVALID
00000108: PUSH9 0x93e1ba09b014ac1a8d
00000112: BYTE
00000113: INVALID
00000114: INVALID
00000115: INVALID
00000116: GT
00000117: PUSH5 0x736f6c6343
0000011d: STOP
0000011e: SDIV
0000011f: SIGNEXTEND
00000120: STOP
00000121: ORIGIN
``` 
  * Payable检查
    ```
    00000000: PUSH1 0x80      // stack=[0x80]
    00000002: PUSH1 0x40      // stack=[0x40, 0x80]
    00000004: MSTORE          // 把0x80(32字节)这个值放在0x40这个内存里, stack=[]
    00000005: CALLVALUE       // 压入创建合约的这笔交易携带的eth数量,没有就压入0, stack=[value]
    00000006: DUP1            // 复制栈第一个值到栈顶, stack=[value,value]
    00000007: ISZERO          // 判断栈顶的值是否为0, 如果是0就压入1, 否则压入0: stack=[0x1/0x0, value]
    00000008: PUSH2 0x10      // 把0x0010压到栈顶,stack=[0x10,0x1/0x0,value]
    0000000b: JUMPI           // 如果stack[1]不为零,则跳转到0x10(JUMPDEST这条指令那), stack=[value]
    0000000c: PUSH1 0x0       // stack=[0, value]
    0000000e: DUP1            // 回滚状态，终止程序
    0000000f: REVERT
    00000010: JUMPDEST
    ...
    ```
  &emsp;&emsp;这段程序的意思是检查合约有没有携带eth，如果有就结束退出，如果没有就跳到0x00000010位置继续执行。  
  &emsp;&emsp;payable是Solidity的一个关键字，如果一个函数被其标记，那么用户在调用该函数的同时还可以发送以太币到该智能合约。而这部分字节码的意义就在于阻止用户在调用没有被payable标记的函数时，向该智能合约发送以太币。

  * 运行合约构造器函数
    我们继续来看看0x00000010的程序,这段就是执行合约的构造函数，把0x55保存到storage的0地址去。
    ```
    ...
    00000010: JUMPDEST            // 没有含义, 表示跳转的目标位置
    00000011: POP                 // 弹出栈的第一个元素，stack=[]
    00000012: PUSH1 0x55          // stack=[0x55]
    00000014: PUSH1 0x0           // stack=[0,0x55]
    00000016: SSTORE              // 把0x55存放在storage的0处
    ...
    ```

  * 复制代码，并将其返回给内存
    这段就是把0x00000024开始长度为0xfe的代码，也就是真正合约的代码保存到内存返回。
    ```
    ...
    00000017: PUSH1 0xfe          // stack=[0xfe]
    00000019: DUP1                // stack=[0xfe,0xfe]
    0000001a: PUSH2 0x24          // stack=[0x24,0xfe,0xfe]
    0000001d: PUSH1 0x0           // stack=[0,0x24,0xfe,0xfe]
    0000001f: CODECOPY            // 复制从0x24开始长度为0xfe的代码到地址为0的内存里, stack=[0xfe]
    00000020: PUSH1 0x0           // stack=[0,0xfe]
    00000022: RETURN              // 返回
    00000023: INVALID
    =======真正的合约代码========
    00000024: PUSH1 0x80
    00000026: PUSH1 0x40
    ...
    ```
  当部署合约代码执行完之后，真正合约那部分的代码会保存在数据库，下次调用就根据合约地址调用真正的合约代码。

参考：

[以太坊 EVM原理与实现](https://www.cnblogs.com/helloworld2018/p/8998926.html)

[以太坊虚拟机（EVM）底层原理及性能缺陷](https://abelsu7.top/2018/02/28/ethereum-virtual-machine/)

[《也来谈一谈以太坊虚拟机EVM的缺陷与不足》](https://bitkan.com/en/news/topic/35732)

[深入了解以太坊虚拟机](https://www.jianshu.com/p/1969f3761208)


[深入探索EVM : 编译和部署智能合约](https://www.arcblock.io/zh/post/2018/12/08/evm-part-1)

[从solc编译过程来理解solidity合约结构](https://www.anquanke.com/post/id/164567)

[Solidity中文文档](https://solidity-cn.readthedocs.io/zh/develop/)


[EVM指令集](https://gist.github.com/hayeah/bd37a123c02fecffbe629bf98a8391df)

[以太坊智能合约虚拟机(EVM)原理与实现](https://blog.csdn.net/mongo_node/article/details/80216079)

[以太坊虚拟机(EVM)架构和源码简析](https://blog.csdn.net/SunnyWed/article/details/80696948)

[以太坊智能合约静态分析](https://paper.seebug.org/790/)  

[深入了解以太坊虚拟机第5部分——一个新合约被创建后会发生什么](https://lilymoana.github.io/evm_part5.html)

[以太坊虚拟机介绍](https://blog.csdn.net/zxhoo/article/details/81865629)

[Ethereum 以太坊智能合约部署源码分析](https://www.jianshu.com/p/e5b56baad043)

[以太坊交易源码分析](https://blog.csdn.net/TurkeyCock/article/details/80485391)

