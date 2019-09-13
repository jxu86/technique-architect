

### **基础语法**


### **数据类型**

**一、布尔类型(Booleans)**

**布尔(bool)**:可能的取值为常量值**true**和**false**。

布尔类型支持的运算符有：

* ！逻辑非

* && 逻辑与

* || 逻辑或

* == 等于

* != 不等于

注意：运算符&&和||是短路运算符，如f(x)||g(y)，当f(x)为真时，则不会继续执行g(y)。

**二、整型(Integers)**

**int**/**uint**: 表示有符号和无符号不同位数整数。支持关键字**uint8** 到 **uint256** (以8步进)，

**uint** 和 **int** 默认对应的是 **uint256** 和 **int256**。


支持的运算符：

* 比较运算符： <=, < , ==, !=, >=, > (返回布尔值：true 或 false)

* 位操作符： &，|，^(异或)，~（位取反）

* 算术操作符：+，-，一元运算-，一元运算+，*，/, %(取余数), \***（幂）, << (左移位), >>(右移位)



### **关键字**
* internal： 内部函数。内部正常访问，外部无法访问，可继承。

* public：公共函数。内部正常访问，外部正常访问，可继承。

* external：外部函数。内部不能访问，外部正常访问，可继承。

* pure：当函数返回值是常量时。

* view：当函数返回值为变量时。

* constant：可以理解为view的旧版本，与其等价。

### **函数**
`function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]`


### **运算符**

### **控制语句**

### **错误处理**

参考:

[区块链技术学习指引](https://learnblockchain.cn/2018/01/11/guide/#more)

[第一集：Solidity语法讲解](https://blog.csdn.net/super_lixiang/article/details/83049719)

[solidity英文文档](https://solidity.readthedocs.io/en/v0.5.11/)

[solidity中文文档](https://solidity-cn.readthedocs.io/zh/develop/)

[cryptozombies](https://cryptozombies.io/zh/)