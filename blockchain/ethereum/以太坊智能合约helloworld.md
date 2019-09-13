## 使用Truffle框架
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

[用Solidity在Truffle上构建一个HelloWorld智能合约](https://www.cnblogs.com/bugmaking/p/9211225.html)

[solidity中文文档](https://solidity-cn.readthedocs.io/zh/develop/introduction-to-smart-contracts.html)

[solidity英文版](https://solidity.readthedocs.io/en/v0.5.11/)
