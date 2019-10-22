
`环境:`  
`macbook pro`  
`node:v12.10.0`  
`ganache-cli:v6.7.0 (ganache-core: 2.8.0)`  
`openzeppelin:v2.5.3`  


### 一、概述
OpenZeppelin SDK是基于EVM做的一个智能合约框架，集成开发、部署、交互。  
接下来主要是演示智能合约的升级部分

### 二、环境安装
nodejs和ganache-cli的安装就不说了，自己去查教程安装去

```
npm install --global @openzeppelin/cli
```
安装后查看一下版本:
```
contract openzeppelin --version
2.5.3
```
### 三、创建项目  
接着创建项目文件夹，npm init一下创建一个package.json文件
```
mkdir openzeppelin-demo
cd openzeppelin-demo
npm init -y
```
现在用openzeppelin init命令去创建一个OpenZeppelin SDK项目:

```
➜  openzeppelin-demo openzeppelin init
? Welcome to the OpenZeppelin SDK! Choose a name for your project openzeppelin-demo
? Initial project version 1.0.0
Project initialized. Write a new contract in the contracts folder and run 'openzeppelin create' to deploy it.
```
看看目录的结构
```
➜  openzeppelin-demo tree -a
.
├── .openzeppelin
│   └── project.json
├── contracts
│   └── .gitkeep
├── networks.js
└── package.json
```
* 其中.openzeppelin默认是隐藏的文件，里面是project.json配置文件
* networks.js为网络节点的一些配置

project.json内容为:
```
{
  "manifestVersion": "2.2",
  "contracts": {},
  "dependencies": {},
  "name": "openzeppelin-demo",
  "version": "1.0.0"
}
```
### 四、编写智能合约
创建contracts/Counter.sol内容如下:
```
pragma solidity ^0.5.0;

contract Counter {
  uint256 public value;

  function increase() public {
    value++;
  }
}
```


### 五、编译智能合约
执行:
```
openzeppelin compile
```
完了后你会发现目录会多了一个build的文件夹，里面Counter.json就是编译的结果(ABI和bytecode)
另外project.json增加了compiler项,之后你可以改变solcVersion的版本来编译项目:
```
{
  "manifestVersion": "2.2",
  "contracts": {},
  "dependencies": {},
  "name": "openzeppelin-demo",
  "version": "1.0.0",
  "compiler": {
    "manager": "openzeppelin",
    "solcVersion": "0.5.12",
    "compilerSettings": {
      "optimizer": {}
    }
  }
}
```
### 六、部署智能合约
首先把以太坊RPC客户端ganache-cli启动起来:
```
ganache-cli --deterministic

```

执行`openzeppelin create`部署合约，然后一直enter就好,最后在选择要不要执行函数，可以No:
```
openzeppelin-demo openzeppelin create  
Nothing to compile, all contracts are up to date.  
? Pick a contract to instantiate Counter  
? Pick a network development    
✓ Added contract Counter   
✓ Contract Counter deployed   
All contracts have been deployed   
? Do you want to call a function on the instance after creating it? No  
✓ Setting everything up to create contract instances  
✓ Instance created at 0xCfEB869F69431e42cdB54A4F4f105C19C080A601  
0xCfEB869F69431e42cdB54A4F4f105C19C080A601   
```
然后合约就部署完了，合约的地址为`0xCfEB869F69431e42cdB54A4F4f105C19C080A601`  
接下来通过命令`openzeppelin send-tx`调用increase方法:
```
➜  openzeppelin-demo openzeppelin send-tx  
? Pick a network development  
? Pick an instance Counter at 0xCfEB869F69431e42cdB54A4F4f105C19C080A601  
? Select which function increase()  
✓ Transaction successful. Transaction hash:   0x1993a8b6774ce05f2f2da0c5fc1174de46a3630e642fac81cf71bec28864e451  
```
通过命令`openzeppelin call`访问value变量:
```
openzeppelin-demo openzeppelin call  
? Pick a network development  
? Pick an instance Counter at 0xCfEB869F69431e42cdB54A4F4f105C19C080A601  
? Select which function value()  
✓ Method 'value()' returned: 1  
1
```
可以看到value的值变成1，也就是说increase方法调用成功

###  七、升级合约
我们把contracts/Counter.sol的代码修改成一下内容:
```
pragma solidity ^0.5.0;

contract Counter {
  uint256 public value;

  function increase(uint256 amount) public {
    value += amount;
  }
}
```
执行`openzeppelin upgrade`升级合约:
```
openzeppelin-demo openzeppelin upgrade  
? Pick a network development  

✓ Compiled contracts with solc 0.5.12 (commit.7709ece9)  
✓ Contract Counter deployed  
All contracts have been deployed  
? Which instances would you like to upgrade? All instances  
✓ Instance upgraded at 0xCfEB869F69431e42cdB54A4F4f105C19C080A601. Transaction receipt: 0xa2ca324b3d99031dd444eb0c21baf42adeb86cf
8f2785734ed083d8bd54ee5fc  
✓ Instance at 0xCfEB869F69431e42cdB54A4F4f105C19C080A601 upgraded
```
升级完可以看到合约的地址是不变的
然后我们调用increase方法，这时候我们就要输入参数了，这里我输了100

```
openzeppelin-demo openzeppelin send-tx  
? Pick a network development  
? Pick an instance Counter at 0xCfEB869F69431e42cdB54A4F4f105C19C080A601  
? Select which function increase(amount: uint256)  
? amount (uint256): 100  
✓ Transaction successful. Transaction hash: 0x9ca605fc6529637c04bd6d98b91b094cf2f20acaecc86caa990e787afe40e3ea
```
查看value的值变成101了，因为升级前value的值为1，所以最后的结果是101
```
openzeppelin-demo openzeppelin call  
? Pick a network development  
? Pick an instance Counter at 0xCfEB869F69431e42cdB54A4F4f105C19C080A601  
? Select which function value()  
✓ Method 'value()' returned: 101  
101
```



参考:  
[大硕杂谈：ZeppelinOS 分析简报](https://www.jianshu.com/p/1e0fa2ef73f3)  
[zeppelin_os_whitepaper](https://ethfans.org/posts/zeppelin_os_whitepaper)  
https://docs.openzeppelin.com/sdk/2.5/first