### **关键字**
* private : 私有函数。内部正常访问，外部无法访问，子类无法继承。

* internal： 内部函数。内部正常访问，外部无法访问，子类可继承。

* public：公共函数。内部正常访问，外部正常访问，子类可继承。

* external：外部函数。内部不能访问，外部正常访问，子类可继承。

* pure：当函数返回值为自变量而非变量时，使用 pure；不能改也不能读状态变量

* view：当函数返回值为全局变量或属性时，使用 view；可以读取状态变量但是不能改

* constant：可以理解为 view 的旧版本，与 view 是等价的


```
contract constantViewPure{
    string name;
    uint public age;
    
    function constantViewPure() public{
        name = "liushiming";
        age = 29;
    }
    
    function getAgeByConstant() public constant returns(uint){
        age += 1;  //声明为constant，在函数体中又试图去改变状态变量的值，编译会报warning, 但是可以通过
        return age;  // return 30, 但是！状态变量age的值不会改变，仍然为29！
    } 
    
    function getAgeByView() public view returns(uint){
        age += 1; //view和constant效果一致，编译会报warning，但是可以通过
        return age; // return 30，但是！状态变量age的值不会改变，仍然为29！
    }
    
    function getAgeByPure() public pure returns(uint){
        return age; //编译报错！pure比constant和view都要严格，pure完全禁止读写状态变量！
        return 1;
    }
}
```

* Assert `assert(bool condition)`如果条件不满足，抛出异常，合约中断，一般使用在内部错误
* Require `require(bool condition)`如果条件不满足，抛出异常，合约中断，在函数输入参数和外部函数使用,`require(bool condition, string message)`，提供一个错误信息
* Revert 中断合约执行，回滚状态改变,`revert(string reason)`，提供一个回滚说明
* Exceptions


参考:      
[【易错概念】Solidity语法constant/view/pure关键字定义](https://cloud.tencent.com/developer/article/1347304)