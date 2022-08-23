`环境:`
`macbook pro`
`Truffle v5.0.35 (core: 5.0.35)`
`Node v12.10.0`
`Solidity - 0.4.24 (solc-js)`

### 一、概述

&emsp;&emsp;Mythx为以太坊智能合约的安全检测工具,[SWC Registry](https://swcregistry.io/)可以看到具体的test cases

### 二、Mythx插件安装&运行

`在装插件前请把node环境和truffle装起来·`
利用npm进行安装:

```
npm install truffle-security
```

安装完，利用truffle创建一个项目，写一个检查的只能合约，运行就ok了

```
truffle run verify
```

看到以下运行打印的信息，如果没有MythX账号只是trial模式，分析项目有限，所以如果要跑全权限的话就要注册mythx的账号了

```
bigcherry_contract git:(master) ✗ truffle run verify        
You are currently running MythX in Trial mode. This mode reports only a partial analysis of your smart contracts, limited to three vulnerabilities. To get a complete analysis, sign up for a free MythX account at https://mythx.io.
```

### 三、注册Mythx账号

* 安装MetaMaskMythx注册需要关联metamask账号
* 注册Mythx这个按照注册提示走就好
* 邮件验证这步一定要做
* 修改密码
  最好修改咯

### 四、设置环境变量

把以下代码加到 `/etc/profile`末尾，然后执行 `source /etc/profile`

```
export MYTHX_ETH_ADDRESS=0x1234567891235678900000000000000000000000
export MYTHX_PASSWORD='Put your password in here!'
```

然后再运行 `truffle run verify`就看到检测的项目多了。

### 五、配置选项

当运行如下命令的时候就会看到选项，可以在运行的时候去配置，也可以写一个truffle-security.json文件把配置写在里面

```
truffle run verify --help
```

truffle-security.json文件内容:

```
{   
    "style": "table",
    "limit":1
}
```

### 六、问题查找

```
(py374) ➜  bigcherry_contract git:(master) ✗ truffle run verify contracts/new-supernode.sol:BCHC_Token


Welcome to MythX! You are currently running in Free mode.

BCHC_Token |**********************************************************************************| 100% || Elapsed: 9.1s ✓ completed

/Users/JC/Documents/project/grg_blockchain/contract/bigcherry_contract/contracts/new-supernode.sol

║ Line     │ Column   │ Type     │ Message                                                │ Rule ID              ║
╟──────────┼──────────┼──────────┼────────────────────────────────────────────────────────┼──────────────────────╢
║ -1       │ 0        │ warning  │ Upgrade to MythX Pro to unlock the ability to test     │ N/A                  ║
║          │          │          │ for even more vulnerabilities, perform deeper          │                      ║
║          │          │          │ security analysis, and more.                           │                      ║
║          │          │          │ https://mythx.io/plans                                 │                      ║
║ 1        │ 0        │ warning  │ A floating pragma is set.                              │ SWC-103              ║
║ 17       │ 27       │ warning  │ The state variable visibility is not set.              │ SWC-108              ║
║ 48       │ 28       │ warning  │ The state variable visibility is not set.              │ SWC-108              ║
```

这里有几个warning如果想知道某个warning的含义就拿RuleID去https://swcregistry.io/docs/SWC-103找相应的答案，还有正确的写法。

参考:
[mythx官网](https://mythx.io/)
[https://docs.mythx.io/en/latest/tools/truffle/](https://docs.mythx.io/en/latest/tools/truffle/)
