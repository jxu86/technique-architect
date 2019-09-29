以下会说两种方法部署智能合约

第一种使用以太坊客户端 deploy方法，这种方法比较繁琐复杂，但是对应理解部署原理比较有用

第二种是使用trffle框架，这种方法基本是配置+命令，部署起来比较方便，开发效率会较第一种高。

## 一、以太坊客户端(geth)
`Environment:`

`Solidity v0.5.1`


### 1、合约编写
创建helloworld.sol, 内容如下:

```
pragma solidity >=0.4.21 <0.6.0;

contract HelloWorld{
    address payable creator;
    string greeting;
    constructor() public {
        creator = msg.sender;
        greeting = "hello world";
    }
    function greet() view public returns (string memory) {
        return greeting;
    }
    function setGreeting(string memory _newgreeting) public {
        greeting = _newgreeting;
    }
    function kill() public{
        if (msg.sender == creator)
            selfdestruct(creator);  // kills this contract and sends remaining funds back to creator
    }
}
```

### 2、合约编译
执行以下命令，得出abi文件HelloWorld.abi和bytecode文件HelloWorld.bin
`注意:我安装的是mac版的solidity编译器`
```
solc helloworld.sol --abi --bin -o ./
```

### 3、连接私链

```
geth attach ipc://Users/JC/Documents/project/grg_blockchain/tmp/test/data1/geth.ipc
Welcome to the Geth JavaScript console!

 instance: Geth/v1.7.4-stable-03c7946b/darwin-amd64/go1.10.3
validator: 0x564eec478efa01313d458b7b6a2edf124cc51fbf
 coinbase: 0x564eec478efa01313d458b7b6a2edf124cc51fbf
 at block: 33 (Fri, 20 Sep 2019 14:52:07 CST)
  datadir: /Users/JC/Documents/project/grg_blockchain/tmp/test/data0
  modules: admin:1.0 debug:1.0 eth:1.0 gdpos:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

>
```
解锁账户
```
personal.unlockAccount(eth.coinbase, "123456", 0)
```
### 4、部署合约

把HelloWorld.abi文件的内容copy给变量api
```
abi=[{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"string","name":"_newgreeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"internalType":"string","name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}]
```
把HelloWorld.bin文件的内容copy给变量bytecode
`注意: 前面要加'0x'`
```
bytecode="0x608060405234801561001057600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506040805190810160405280600b81526020017f68656c6c6f20776f726c640000000000000000000000000000000000000000008152506001908051906020019061009c9291906100a2565b50610147565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100e357805160ff1916838001178555610111565b82800160010185558215610111579182015b828111156101105782518255916020019190600101906100f5565b5b50905061011e9190610122565b5090565b61014491905b80821115610140576000816000905550600101610128565b5090565b90565b6103e3806101566000396000f3fe608060405260043610610051576000357c01000000000000000000000000000000000000000000000000000000009004806341c0e1b514610056578063a41368621461006d578063cfae321714610135575b600080fd5b34801561006257600080fd5b5061006b6101c5565b005b34801561007957600080fd5b506101336004803603602081101561009057600080fd5b81019080803590602001906401000000008111156100ad57600080fd5b8201836020820111156100bf57600080fd5b803590602001918460018302840111640100000000831117156100e157600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f820116905080830192505050505050509192919290505050610256565b005b34801561014157600080fd5b5061014a610270565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561018a57808201518184015260208101905061016f565b50505050905090810190601f1680156101b75780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161415610254576000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b565b806001908051906020019061026c929190610312565b5050565b606060018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156103085780601f106102dd57610100808354040283529160200191610308565b820191906000526020600020905b8154815290600101906020018083116102eb57829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061035357805160ff1916838001178555610381565b82800160010185558215610381579182015b82811115610380578251825591602001919060010190610365565b5b50905061038e9190610392565b5090565b6103b491905b808211156103b0576000816000905550600101610398565b5090565b9056fea165627a7a7230582049b29deb586ec1f5d654c4051b8d48cab7b2609190191cb5a1c39455b70ad0290029"
```

创建一个contact对象:
```
helloworldContract=eth.contract(abi)
```

创建一个实例:

```
helloworld = helloworldContract.new(
   {
     from: eth.coinbase, 
     data: bytecode, 
     gas: '4300000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
 ```
 得到的结果:
 ```
 > helloworld = helloworldContract.new(
...    {
......      from: eth.coinbase,
......      data: bytecode,
......      gas: '4300000'
......    }, function (e, contract){
......     console.log(e, contract);
......     if (typeof contract.address !== 'undefined') {
.........          console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
.........     }
......  })
null [object Object]
{
  abi: [{
      constant: false,
      inputs: [],
      name: "kill",
      outputs: [],
      payable: false,
      stateMutability: "nonpayable",
      type: "function"
  }, {
      constant: false,
      inputs: [{...}],
      name: "setGreeting",
      outputs: [],
      payable: false,
      stateMutability: "nonpayable",
      type: "function"
  }, {
      constant: true,
      inputs: [],
      name: "greet",
      outputs: [{...}],
      payable: false,
      stateMutability: "view",
      type: "function"
  }, {
      inputs: [],
      payable: false,
      stateMutability: "nonpayable",
      type: "constructor"
  }],
  address: undefined,
  transactionHash: "0xfaf6fa7c57ec8394da2e3650ec39da3ce681cb2182c26dc627c256f18cfb09e7"
}
> null [object Object]
Contract mined! address: 0xbc7384998a5453a41bc51c1aa0f252034b57b986 transactionHash: 0xfaf6fa7c57ec8394da2e3650ec39da3ce681cb2182c26dc627c256f18cfb09e7
```
结果打印了合约地址和交易hash

可以用实例调用合约函数
```
> helloworld.greet.call()
"hello world"
```
如果有上面结果，证明合约部署已经成功

### 5、其他账户调用合约
跟步骤3一样连接另外一个节点

执行一下命令：
```
> abi=[{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"string","name":"_newgreeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"internalType":"string","name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}]

> MyContract = eth.contract(abi)   #这时候就体现了abi的重要性，相当于接口的描述文档
> contractAddress = "0xbc7384998a5453a41bc51c1aa0f252034b57b986" #这个地址就是合约地址
> myContract = MyContract.at(contractAddress)
> myContract.greet.call()
"hello world"
 ```

如果出现上述结果，证明调用也成功了。
`注意: Solidity我这里用了v0.5.1, 开始我用v0.5.11没成功，原因我没去找`


## 二、使用Truffle框架
`Environment:`

`Truffle v5.0.35 (core: 5.0.35)`

`Solidity v0.5.8 (solc-js)`

`Node v12.10.0`

`Web3.js v1.2.1`
### 1、Truffle安装
```
npm install -g truffle@5.0.35
```
### 2、Truffle客户端
*   [EtherumJS TestRPC](https://github.com/ethereumjs/testrpc)
*   [Geth (go-ethereum)](https://github.com/ethereum/go-ethereum)

我这里选择的了第二种

### 3、 创建工程目录
打开你的terminal选择好你放合约工程的目录
```
mkdir HelloWorld
```

### 4、初始化工程
```
cd HelloWorld
truffle init
```
执行完后看到以下目录结构:
```
├── contracts
│   └── Migrations.sol
├── migrations
│   └── 1_initial_migration.js
├── test
└── truffle-config.js
```
* contract/ - Truffle默认的合约文件存放地址
* migrations/ - 存放发布脚本文件
* test/ - 用来测试应用和合约的测试文件
* truffle-config.js - Truffle的配置文件

### 5、编写自己的合约
在目录contracts里创建HelloWorld.sol文件，内容为：
```
pragma solidity >=0.4.21 <0.6.0;

contract HelloWorld{
    address payable creator;
    string greeting;
    constructor() public {
        creator = msg.sender;
        greeting = "hello world";
    }
    function greet() view public returns (string memory) {
        return greeting;
    }
    function setGreeting(string memory _newgreeting) public {
        greeting = _newgreeting;
    }
    function kill() public{
        if (msg.sender == creator)
            selfdestruct(creator);  // kills this contract and sends remaining funds back to creator
    }
}
```
### 6、增加发布脚本
在目录migrations里创建2_initial_helloworld.js文件，内容为
```
const HelloWorld = artifacts.require("HelloWorld");
module.exports = function(deployer) {
  deployer.deploy(HelloWorld);
};
```

### 7、编译
在工程目录HelloWorld目录下，执行：
```
truffle compile
```
打印一下信息代表编译通过:
```
➜  HelloWorld truffle compile

Compiling your contracts...
===========================
> Compiling ./contracts/HelloWorld.sol
> Compiling ./contracts/Migrations.sol
> Artifacts written to /Users/JC/Documents/project/grg_blockchain/tmp/test/contract/HelloWorld/build/contracts
> Compiled successfully using:
   - solc: 0.5.8+commit.23d335f2.Emscripten.clang
```

### 8、启动geth客户端
这是我启动的geth客户端，rpc端口为8100
`
geth_test --datadir ./data0 --targetgaslimit 0x5000000000 --nmsaddr 127.0.0.1:9100 --rpc --rpcaddr 127.0.0.1 --rpcport 8100 --port 30300 --rpccorsdomain data0 data1 data2 gdpos_genesis.json --rpcapi db,eth,net,web3,personal --networkid 10001
`

修改配置文件truffle-config.js
```
  networks: {
    // Useful for testing. The `development` name is special - truffle uses it by default
    // if it's defined here and no other network is specified at the command line.
    // You should run a client (like ganache-cli, geth or parity) in a separate terminal
    // tab if you use this network and you must also set the `host`, `port` and `network_id`
    // options below to some value.
    //
    development: {
      host: "127.0.0.1",     // Localhost (default: none)
      port: 8100,            // Standard Ethereum port (default: none)
      network_id: "*",       // Any network (default: none)
    },

```

### 9、部署合约
```
truffle migrate
```
出现以下错误:
```
HelloWorld truffle migrate

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      1357
> Block gas limit: 0x5000000000


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------

Error: Error: Error:  *** Deployment Failed ***

"Migrations" -- Returned error: authentication needed: password or unlock.

    at Object.run (/Users/JC/.nvm/versions/node/v12.10.0/lib/node_modules/truffle/build/webpack:/packages/migrate/index.js:96:1)
    at processTicksAndRejections (internal/process/task_queues.js:93:5)
Truffle v5.0.35 (core: 5.0.35)
Node v12.10.0
```
原因是账号没有解锁
解锁如下:
```
geth_test attach ipc://Users/JC/Documents/project/grg_blockchain/tmp/test/data0/geth.ipc
Welcome to the Geth JavaScript console!

 instance: Geth/v1.7.4-stable-03c7946b/darwin-amd64/go1.10.3
validator: 0x564eec478efa01313d458b7b6a2edf124cc51fbf
 coinbase: 0x564eec478efa01313d458b7b6a2edf124cc51fbf
 at block: 2674 (Sun, 08 Sep 2019 15:05:28 CST)
  datadir: /Users/JC/Documents/project/grg_blockchain/tmp/test/data0
  modules: admin:1.0 debug:1.0 eth:1.0 gdpos:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> personal.unlockAccount(eth.coinbase)
Unlock account 0x564eec478efa01313d458b7b6a2edf124cc51fbf
Passphrase:
true
```
账号解锁完后重新执行truffle migrate
```
HelloWorld truffle migrate

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      1357
> Block gas limit: 0x5000000000


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0x5383f376c5e4b8250720c420265892094ed4f840606b52a2528842cd4757a866
   > Blocks: 0            Seconds: 4
   > contract address:    0x9beed20097892e30918E211434aE75b85B58C871
   > block number:        2830
   > block timestamp:     1567926878
   > account:             0x76EFB705ccE44Eb65e684c4bFa8ebDa6EbF5baFd
   > balance:             4382.72737498
   > gas used:            251300
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.005026 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:            0.005026 ETH


2_initial_helloworld.js
=======================

   Deploying 'HelloWorld'
   ----------------------
   > transaction hash:    0xa6c1bcce50a03dd20b2d50f11221df0ef8dce5a1ebd3a50279489f8e04c30a01
   > Blocks: 0            Seconds: 0
   > contract address:    0x9171D0789DF9F488321D07C404A2A6F9b3d9737A
   > block number:        2832
   > block timestamp:     1567926880
   > account:             0x76EFB705ccE44Eb65e684c4bFa8ebDa6EbF5baFd
   > balance:             4382.71963578
   > gas used:            344932
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00689864 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00689864 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.01192464 ETH
```


### 10、运行合约
```
truffle console       # 进入命令行模式
truffle(development)> helloworld=await HelloWorld.deployed()
undefined
truffle(development)> helloworld.greet.call()
'hello world'
truffle(development)> helloworld.setGreeting("Hi world")
{
  tx: '0x4a518f5fad55e087cb058dc84aaa2e61d5c94fa735ea54a6439cf0e1cd36f68f',
  receipt: {
    blockHash: '0x6a2a41ac78f7f469ab4d4add98be0a5ab1b0079d7566ce2be37715e1877cb919',
    blockNumber: 2890,
    contractAddress: null,
    cumulativeGasUsed: 33464,
    from: '0x76efb705cce44eb65e684c4bfa8ebda6ebf5bafd',
    gasUsed: 33464,
    logs: [],
    logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
    status: true,
    to: '0x9171d0789df9f488321d07c404a2a6f9b3d9737a',
    transactionHash: '0x4a518f5fad55e087cb058dc84aaa2e61d5c94fa735ea54a6439cf0e1cd36f68f',
    transactionIndex: 0,
    type: 0,
    rawLogs: []
  },
  logs: []
}
truffle(development)> helloworld.greet.call()
'Hi world'
```

参考:

[以太坊智能合约Hello World示例程序](https://blog.csdn.net/CSDN_AF/article/details/77963841)

[以太坊私有链下智能合约部署](https://www.cnblogs.com/beyang/p/8469311.html)

[用Solidity在Truffle上构建一个HelloWorld智能合约](https://www.cnblogs.com/bugmaking/p/9211225.html)

[solidity中文文档](https://solidity-cn.readthedocs.io/zh/develop/introduction-to-smart-contracts.html)

[solidity英文版](https://solidity.readthedocs.io/en/v0.5.11/)
