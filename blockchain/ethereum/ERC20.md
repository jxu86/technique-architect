`openzeppelin-solidity 4.5.0`
`solidity 0.8.0`



### 状态变量
* name 返回 token 的名字
* symbol 返回令牌的符号，比如 LXT 
* decimals 返回 token 使用的小数点后几位
### 接口函数
* name 返回代币名字
* symbol 返回代币简称
* decimals 代币的最小分割量 token使用的小数点后几位。比如如果设置为3，就是支持0.001表示
* totalSupply ERC20 代币的总发行量，实现通常通过一个状态变量实现 
* balanceof 返回地址tokenOwner的账户余额 
* transfer 转移_value的 token 数量到的地址_to，并且必须触发Transfer事件。 如果_from帐户余额没有足够的令牌来支出，该函数应该被throw。 
* transferFrom 从地址_from发送数量为_value的 token 到地址_to,必须触发Transfer事件。 
* approve 允许_spender多次取回您的帐户，最高达_value金额。 如果再次调用此函数，它将以_value覆盖当前的余量。 
* allowance 返回_spender仍然被允许从_owner提取的金额。
### 事件
* Transfer 当 token 被转移 (包括 0 值)，必须被触发。 
* Approval 当任何成功调用approve(address _spender, uint256 _value)后，必须被触发。



参考:   
[以太坊开发之ERC20协议原理及实现](https://www.dazhuanlan.com/xiaodonging/topics/1229584)    
[以太坊ERC20代币合约案例 ](https://www.cnblogs.com/jameszou/p/10131443.html)    
[通过SOLIDITY智能合约转移和批准ERC-20通证](https://ethereum.org/zh/developers/tutorials/transfers-and-approval-of-erc-20-tokens-from-a-solidity-smart-contract/)   
[了解ERC-20通证智能合约](https://ethereum.org/zh/developers/tutorials/understand-the-erc-20-token-smart-contract/)