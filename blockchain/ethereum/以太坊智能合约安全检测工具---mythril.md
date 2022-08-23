
`环境:`  
`Python 3.10.4`


### 概述





### mythril安装

```
pip install mythril

```
or docker 
```
docker pull mythril/myth
```



```
(py3104) ➜  contract-secure docker run -v /Users/jc/Documents/project/contract-secure:/tmp mythril/myth analyze /tmp/bec.sol --solv 0.4.24
==== Integer Arithmetic Bugs ====
SWC ID: 101
Severity: High
Contract: OverflowAdd
Function name: fallback
PC address: 143
Estimated Gas Usage: 5954 - 26049
The arithmetic operator can overflow.
It is possible to cause an integer overflow or underflow in the arithmetic operation.
--------------------
In file: /tmp/bec.sol:6

balance += deposit

--------------------
Initial State:

Account: [CREATOR], balance: 0x0, nonce:0, storage:{}
Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], calldata: , value: 0x0
Caller: [CREATOR], function: add(uint256), txdata: 0x1003e2d2ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff, value: 0x0
```

参考:    
[mythril](https://github.com/ConsenSys/mythril)           
[Mythril教程【智能合约安全分析】](http://blog.hubwiz.com/2020/05/11/mythril-tutorial/)       
[Mythril智能合约安全分析入门](https://zhuanlan.zhihu.com/p/139837676)