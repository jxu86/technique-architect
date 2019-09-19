### **数据类型**
**Solidity 类型分为两类：**
* 值类型 (Value Type) - 变量在赋值或传参时，总是进行值拷贝。
    * 布尔类型 (Booleans)
    * 整型 (Integers)
    * 定长浮点型 (Fixed Point Numbers)
    * 定长字节数组 (Fixed-size byte arrays)
    * 有理数和整型常量 (Rational and Integer Literals)
    * 字符串常量（String literals）
    * 十六进制常量（Hexadecimal literals）
    * 枚举 (Enums)
    * 函数类型 (Function Types)
    * 地址类型 (Address)
    * 地址常量 (Address Literals)

* 引用类型 (Reference Types)
    * 不定长字节数组（bytes）
    * 字符串（string）
    * 数组（Array）
    * 结构体（Struts）

` 引用类型，赋值时，我们可以值传递，也可以引用即地址传递，如果是值传递，和值传递一样，修改新变量时，不会影响原来的变量值，如果是引用传递，那么当你修改新变量时，原来变量的值会跟着变化，这是因为新就变量同时指向同一个地址的原因。`

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









### **函数**
`function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]`


### **运算符**

### **控制语句**

### **错误处理**

参考:

[区块链：Solidity值传递&值类型与引用类型](https://blog.csdn.net/wtdask/article/details/81984103)

[区块链技术学习指引](https://learnblockchain.cn/2018/01/11/guide/#more)

[第一集：Solidity语法讲解](https://blog.csdn.net/super_lixiang/article/details/83049719)

[solidity英文文档](https://solidity.readthedocs.io/en/v0.5.11/)

[solidity中文文档](https://solidity-cn.readthedocs.io/zh/develop/)

[cryptozombies](https://cryptozombies.io/zh/)