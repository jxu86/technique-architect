
`环境:`  
`Python 3.10.4`


### 概述
Slither是一个用Python3编写的智能合约静态分析框架，提供如下功能：
* 自动化漏洞检测。提供超30多项的漏洞检查模型，模型列表详见：https://github.com/crytic/slither#detectors or https://github.com/crytic/slither/wiki/Detector-Documentation
。
* 自动优化检测。Slither可以检测编译器遗漏的代码优化项并给出优化建议。
* 代码理解。Slither能够绘制合约的继承拓扑图，合约方法调用关系图等，帮助开发者理解代码。
* 辅助代码审查。用户可以通过API与Slither进行交互。


### slither安装
参考: https://github.com/crytic/slither
```
pip install slither-analyzer
```

### 检查智能合约zz

```
slither ./contracts
```
如果用到openzeppelin库:
```
slither ./contracts --solc-remaps "@openzeppelin=node_modules/@openzeppelin"
```

### 查看合约继承关系
```
slither ./contracts/ERC20T.sol --solc-remaps "@openzeppelin=node_modules/@openzeppelin" --print inheritance-graph
```
生成.dot文件可以使用graphviz工具转成png
```
dot -Tpng InputFile.dot -o OutputFile.png
```




### solidity版本管理工具
[solc-select](https://github.com/crytic/solc-select)    
[以太坊合约静态分析工具Slither简介与使用](https://blog.csdn.net/weixin_43587332/article/details/107548222)














参考:      
[slither](https://github.com/crytic/slither)  